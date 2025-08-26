# Enumeration

## Starting Position and Goals

The starting position for this step is that the attacker has already obtained a foothold in the internal network, by compromising one machine or account. At this stage, the attacker has the possibility to enumerate various details about the Domain even with low-priveleged access.

The goal of this step is to map the Active Directory environment: identify users, groups, computers, domain controllers, trusts, and privilege delegation paths. Enumeration provides the roadmap for lateral movement and privilege escalation by uncovering high-value targets (e.g., Domain Admins, service accounts, ...). Moreover you never know what low hanging fruit can just be waiting there (credentials in description ...)

## Methodology

- Identify Domain Information
- Enumerate Users and Groups
- Enumerate Computers and Domain Controllers
- Enumerate Trust Relationships
- Enumerate Service Accounts
- Use Automated Enumeration Tools
- Document Findings and Paths

```{toctree}
:maxdepth: 1

credentials-injection
manual-enumeration
enumeration-with-ldapdomaindump
enumeration-with-bloodhound
```
