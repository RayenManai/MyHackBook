# Credentials Injection

## What is it?

Credential injection is the technique of using discovered Active Directory credentials (username and password) on a system that is not joined to the domain, in order to authenticate against AD resources. Instead of needing a domain-joined machine or Kerberos ticketing setup, the credentials are “injected” into a Windows process so that all network connections from that process use the supplied AD account.

The most common method is using the built-in Windows tool runas.exe with the /netonly flag. This allows attackers to “borrow” the identity of a domain account for network authentication while still running locally as their own user.

```
runas.exe /netonly /user:<domain>\<username> cmd.exe
```

## Use Case

- You compromise an internal web app and recover plaintext AD credentials for a user.
- You use your attacker-controlled Windows machine (or a local admin foothold box) and inject those credentials with runas.exe /netonly
- Now you can enumerate AD, query LDAP, browse file shares, and access services like SQL or intranet portals as that user.

## Why should I care?

The main advantage of using Windows credential injection (runas.exe /netonly) over Linux is that it provides a transparent and native way to load credentials into memory so that any application launched from that session automatically uses them for network authentication. This makes enumeration and exploitation more seamless and realistic, closely mimicking how a real domain user would operate. In contrast, on Linux you typically have to pass credentials explicitly to each tool.
