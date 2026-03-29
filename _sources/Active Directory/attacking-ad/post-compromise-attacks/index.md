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

**Situation and Use Case**

Consider a practical scenario: You have compromised a machine in the `10.10.155.0/24` network, but this compromised machine has two network interfaces — one on your accessible network and another on an isolated `10.10.10.0/24` network that you cannot directly reach from your attacker machine. This segmentation is intentional and designed to protect critical systems.

Rather than installing offensive tools on the compromised machine (which increases the risk of detection), you can utilize it as a proxy to pivot into the isolated network. This approach allows you to:

- Run reconnaissance and attacks from your attacker machine
- Minimize the footprint of tools on compromised systems
- Leverage your familiar tools and techniques
- Work around network segmentation and firewall rules

**Methods and Tools**

#### Proxychains

Proxychains is a tool that intercepts network calls in Linux and routes them through a SOCKS proxy. When you prepend `proxychains` to any command, it forces that command's traffic through your configured proxy instead of using your direct network connection. This allows you to route commands through the compromised machine as if you were attacking from inside the isolated network.

**How it works:**

1. You establish an SSH tunnel with a SOCKS proxy on the compromised machine
2. Proxychains intercepts traffic from your tools and routes it through this tunnel
3. The compromised machine forwards the traffic to the isolated network
4. Responses come back through the same tunnel

**Setup:**

```bash
# First, view/edit the proxychains configuration file
cat /etc/proxychains4.conf

# Ensure this SOCKS proxy line is present (or add it):
# socks4 127.0.0.1 9050
```

The configuration file tells proxychains where to send the traffic. `127.0.0.1:9050` is your local machine (localhost on port 9050).

```bash
# Establish an SSH tunnel with dynamic port forwarding from the compromised machine
ssh -f -N -D 9050 -i pivot root@10.10.155.5
```

**Breaking down this SSH command:**

- `ssh` - SSH client
- `-f` - Run in background (forks after authentication)
- `-N` - Don't execute any remote command (just forward ports)
- `-D 9050` - Create a SOCKS proxy on localhost port 9050 that tunnels to the remote machine
- `-i pivot` - Use the private key file named "pivot" for authentication
- `root@10.10.155.5` - Connect as root user to the compromised machine at 10.10.155.5

Once this tunnel is established, anything you send to `127.0.0.1:9050` goes through the SSH tunnel to the compromised machine, where it can then access the `10.10.10.0/24` network.

**Usage Examples:**

```bash
# Network reconnaissance
proxychains nmap -p88 10.10.10.255
proxychains nmap 10.10.10.255 -sT

# Active Directory attacks
proxychains impacket-GetUserSPNs MARVEL.local/fcastle:Password1! -dc-ip 10.10.10.255 -request

# Remote Desktop access
proxychains xfreerdp /u:administrator /p:'Hacker321!' /v:10.10.10.255

# Web applications
proxychains firefox
```

#### sshuttle

Sshuttle creates a transparent proxy tunnel via SSH, making the pivot network feel like a directly connected subnet. This approach is often more seamless than proxychains for traditional commands.

**Setup:**

```bash
# Establish transparent tunnel
sshuttle -r root@10.10.155.5 10.10.10.0/24 --ssh-cmd "ssh -i pivot"
```

**Usage:**

```bash
# With sshuttle established, commands communicate directly with the new network
nmap 10.10.10.255 -p88

# All traffic to 10.10.10.0/24 transparently tunnels through the pivot point
```

```{toctree}
:maxdepth: 1
pass-attacks
mimikatz
kerberoasting
token-impersonation
lnk-file-attacks
gpp-cPassword-attacks
```
