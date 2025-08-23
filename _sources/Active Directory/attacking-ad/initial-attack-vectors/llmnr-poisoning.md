# LLMNR Poisoning

### What is LLMNR?

LLMNR (Link-Local Multicast Name Resolution) is a protocol used in Windows environments to resolve hostnames on a local network when DNS queries fail. It allows systems to perform name resolution without relying on a DNS server by broadcasting queries to nearby machines. While intended as a fallback mechanism for convenience, it introduces significant security risks.

### What can go wrong here?

LLMNR poisoning is an attack technique often used as an initial vector in Active Directory compromises. An attacker connected to the same network listens for LLMNR queries and responds with false information, tricking victims into sending authentication requests to the attacker’s machine. This enables the capture of NTLMv2 hashes, which can then be brute-forced offline or relayed to other services for lateral movement and privilege escalation.

### Attack Requirements

The main requirements for LLMNR poisoning are that LLMNR (or its predecessor, NBT-NS) is enabled in the network, that DNS resolution fails for the queried hostname, and that the attacker has local network access to intercept and respond to the broadcast traffic. These conditions make it a common and effective entry point in poorly hardened Windows environments.

### LLMNR Poisoning in AD

#### Step 1: Attacker runs responder

```
sudo responder -I eth0 -dwP
```

#### Step 2: An Event Occurs in the Network and Triggers LLMNR

When a LLMNR event occurs in the network and is maliciously responded to, the attacker will obtain sensitive information, including IP adress, domain, username and password hash of the victim.

In our lab scenario we simulated this by looking for `\\10.0.2.15` (attacker IP address) inside the windows machine top bar in files

#### Step 3: Cracking the hash offline

We can attempt to take the victim's captured hash offline and crack it.

```
hashcat –m 5600 <hashfile.txt> <wordlist.txt> --show
```

5600 is for NTLMv2

### Mitigations

The primary mitigation strategy is to disable LLMNR and its predecessor NBT-NS across the environment, since they are rarely necessary in well-managed networks. Enforcing strong DNS infrastructure ensures that hosts do not fall back to insecure name resolution methods. Additionally, enforcing SMB signing and enabling Extended Protection for Authentication can reduce the effectiveness of relay attacks. Using strong password policies and multifactor authentication limits the value of captured hashes, while monitoring for suspicious authentication attempts and unusual broadcast traffic helps detect ongoing poisoning attempts. Network segmentation can also reduce the attacker’s ability to intercept and respond to LLMNR queries.

#### References

- [https://tcm-sec.com/llmnr-poisoning-and-how-to-prevent-it/](https://tcm-sec.com/llmnr-poisoning-and-how-to-prevent-it/)
