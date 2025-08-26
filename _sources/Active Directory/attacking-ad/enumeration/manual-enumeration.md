# Manual Enumeration Techniques

These techniques leverage built-in Windows tools for enumerating Active Directory, without relying on external scripts. They are especially valuable in stealthier engagements.

## Microsoft Management Console

**Starting Position:** From a Windows domain-joined machine or a Windows machine with injected credentials (via runas /netonly)

The MMC provides snap-ins such as Active Directory Users and Computers (ADUC) or Group Policy Management. With valid credentials, an attacker can browse AD objects, group memberships, and policies directly through a GUI.

## Command Prompt

**Starting Position:** From a Windows domain-joined machine as the net command must be executed from a domain-joined machine.

Command Prompt supports classic tools like net command for enumeration.

```
net user /domain
net user "<username>" /domain

net group /domain
net group "Domain Admins" /domain

-- enumerate password policy
net accounts /domain
```

## PowerShell

**Starting Position:** From a Windows domain-joined machine or a Windows machine with injected credentials (via runas /netonly)

PowerShell is far more powerful and scriptable than cmd.exe, offering access to modern enumeration cmdlets.

```
Get-ADDomain
Get-ADUser -Identity <username> -Properties *
Get-ADGroupMember "Domain Admins"
```
