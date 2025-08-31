# Post Compromise Attacks

## Starting Position and Goals

The starting position here is that the attacker already has some form of access within the domain — for example:

- A low-privileged domain user account.

- A compromised workstation.

- Cleartext credentials or password hashes obtained through initial attacks (LLMNR poisoning, SMB relay, config file credentials, etc.).

The goal is to expand this access across the environment, either to reach more valuable assets (file servers, SQL databases, domain controllers) or to escalate privileges toward full domain dominance. Two key techniques come into play here: lateral movement and pivoting.

## Methodology

### Lateral Movement

**What is it?**

Lateral movement is the process of using a compromised account or system to access and control additional systems within the same environment. It’s about “moving sideways” within the domain.

**Why is it needed?**

A single compromised user or machine rarely gives full control. Attackers need to move laterally to collect more credentials, locate high-value targets, and escalate privileges.

### Pivoting

**What is it?**

Pivoting is using a compromised system as a stepping stone to reach otherwise inaccessible parts of the network. Unlike lateral movement, which typically stays within the same security zone, pivoting allows attackers to access new network segments or restricted environments.

**Why is it needed?**

Corporate networks are often segmented. An attacker may compromise a user workstation in the corporate VLAN but needs to pivot through it to reach the data center subnet, OT/ICS networks, or domain controllers behind firewalls.

```{toctree}
:maxdepth: 1
pass-attacks
mimikatz
kerberoasting
token-impersonation
```
