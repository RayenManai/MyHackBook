# Post Domain Compromise Attacks

## Starting Position and Goals

At this stage, the attacker has achieved complete control over the Active Directory domain. This means:

- Domain Administrator (DA) access has been obtained.
- Full compromise of the AD infrastructure is possible.
- Access to critical assets like domain controllers, file servers, and user accounts is available.

The goals from this position are to maximize the value delivered to the client, establish persistence to maintain access, and demonstrate the full scope of domain compromise.

## Methodology

### Maximize Value to the Client

With domain compromise, the penetration test enters its final phase of impact assessment. The focus shifts to extracting and demonstrating the maximum value to the client — showing what a true attacker could do with full domain control.

**How?**

- **Put your blinders on and do it again**: Treat the network as a new hunting ground. Continue enumerating and exploiting, as if this were a fresh engagement, to uncover additional vulnerabilities and high-value targets.

- **Dump NTDS.dit and crack passwords**: Extract the Active Directory database to recover password hashes. Offline cracking can reveal weak passwords and provide further access vectors.

- **Enumerate shares for sensitive information**: Search file servers and shared resources for sensitive data (intellectual property, financial records, credentials, personal information) to demonstrate the scope of compromise.

### Establish Persistence

Persistence mechanisms are backup access methods installed to maintain control even if the primary access vector is discovered and remediated.

**Why is it needed?**

With domain compromise, maintaining access is critical. If your Domain Administrator account is discovered and disabled, persistence ensures you can regain entry. This demonstrates the true severity of domain compromise and the difficulty of eradicating an attacker with full AD control.

**Techniques:**

- **Create a Domain Administrator account**: A dedicated DA account serves as a backup entry point. This can be useful for demonstrating persistence capabilities. However, **DO NOT FORGET to delete it**. Remember that persistence mechanisms should be removed after testing, and all persistence activities should be documented with explicit client approval. If persistence goes undetected, this must be reported as a finding.

- **Create a Golden Ticket**: A Golden Ticket (a forged Kerberos TGT) provides long-term, difficult-to-detect access even if credentials are changed or accounts are disabled. This demonstrates the severity of domain compromise.

```{toctree}
:maxdepth: 1
ntds-dumping
golden-ticket
```
