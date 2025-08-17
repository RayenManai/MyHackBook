# Active Directory

Active Directory (AD) is a directory service used in Windows environments to manage users, groups, computers, and resources.  
Enumeration is the **first step** in attacking AD.

---

## 🧭 Key Enumeration Techniques

| Technique            | Tools               | Notes                              |
| -------------------- | ------------------- | ---------------------------------- |
| LDAP Queries         | ldapsearch, adfind  | Extract user/computer info         |
| Kerberos Enumeration | kerbrute, Rubeus    | Identify valid users via AS-REP    |
| SMB/NetBIOS          | smbmap, nbtscan     | Find shares and sessions           |
| PowerShell Recon     | PowerView, ADModule | Run directly on a compromised host |

---

## 🔍 Example Commands

### Kerberos Enumeration (AS-REP Roasting)

```bash
impacket-GetNPUsers domain.local/ -dc-ip 10.0.0.5 -usersfile users.txt -format john
```

```{figure} ../_static/logo.jpg
:alt: Active Directory Enumeration Diagram
:width: 70%
:align: center

Active Directory Enumeration Workflow
```
