# Enumeration with Bloodhound

### What is it?

BloodHound is a great AD enumeration tool available. With graph-based visualisation, every AD object (users, groups, computers, ACLs, trusts) becomes a node, and every relationship becomes an edge, allowing attackers and defenders alike to map out potential attack paths.

Sharphound is the data collection component of BloodHound. It gathers AD information via LDAP queries, SMB, and other protocols, then outputs it as JSON files. These are later ingested into the BloodHound GUI, which uses the Neo4j graph database to display attack paths visually.

### Use case

It allows attackers (and defenders) to see attack paths through the domain, showing which accounts can escalate privileges or access high-value targets. Instead of manually correlating users, groups, and permissions, BloodHound provides a visual map, saving time and reducing errors.

### Methodology

**Starting Position:** From a Windows domain-joined machine (noisy and likely detected by AV/EDR) or a Windows machine with injected AD credentials via runas /netonly (preferred option as we control this machine and can easily deactivate AV or make exceptions)

1. Run Sharphound on the Windows machine using desired collection methods (e.g., All or Session) to enumerate AD objects.

2. Retrieve the ZIP output file containing JSON data.

3. Import the ZIP into BloodHound GUI connected to Neo4j.

4. Visualize nodes and edges, explore group memberships, sessions, and privileges.

5. Identify attack paths from your current position to high-value targets (e.g., Domain Admins).
