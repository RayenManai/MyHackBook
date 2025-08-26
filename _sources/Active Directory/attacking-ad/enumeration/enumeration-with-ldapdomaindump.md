# Enumeration with ldapdomaindump

Once valid Active Directory credentials are obtained, one of the fastest ways to extract a wealth of domain information is using ldapdomaindump. This tool queries LDAP and generates organized reports of AD objects such as users, groups, computers, and policies.

```
sudo ldapdomaindump ldap(s)://10.0.2.18 -u 'MARVEL\fcastle' -p Password1!
```

## Use Case

- Quickly identify interesting accounts (e.g., service accounts, admin groups).
- Spot weaknesses in descriptions (like passwords accidentally stored in the “Description” field).
- Build a map of group memberships to target privileged users.
