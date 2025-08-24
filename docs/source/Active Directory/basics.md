# Basics

Active Directory is the most used solution for the management of devices and users in corporate environments, as it allows for a centralised administration.

### Terminology

A group of users and computers represent a **Windows Domain** (an administrative boundary). These components are managed in a repository called **Active Directory (AD)**. **The Domain Controller (DC)** is the server that runs the AD services.
**Active Directory Domain Services (AD DS)** is like a phone book that store information about all the components of the network (users, groups, machines ...)

- **Users**: Generally persons in the organisation (employees), or services (MSSQL)

- **Machines**: Each computer that joins the domain is respresented through a machine object. They are assigned a machine account, its name is the machine name followed by a dollar sign (local administrator on the computer). Machines are often devided into the following categories: Workstations (device utilized by users for daily work, no privileged user should sign), Servers (provide services) and Domain Controllers (manage the domain)

- **Security Groups**: user and/or machine groups that can be granted specific privileges. The most important ones are:

| Security Group     | Description                                      |
| ------------------ | ------------------------------------------------ |
| Domain Admins      | administrative privileges over the entire domain |
| Domain Controllers | all DCs of the domain                            |
| Domain Users       | all user accounts in the domain                  |
| Domain Computers   | all computers in the domain                      |

### AD Components

#### Physical Components

1. **Domain Controllers**: a server running windows server Operating System and having active directory installed. It hosts the "phone book". Their main functionalities are:
   - Hosting a copy of the AD DS store
   - Providing authentication and authorization services
   - Allowing administrative access to manage resources
2. **AD DS Data Store**: the directory store stores all the data of its domain controller. It contains the `NTDS.dit` file. It is only accessed through the DC.

#### Logical Components

1. **AD DS schema**: defines every type of object that can be stored in the directory.

2. **Organizational Units**: AD containers that can group users, groups, computers and other OUs. They are mainly used to apply policies, whereas security groups are used to grant permissions. Policies are managed via **Group Policy Objects (GPO)**.

3. **Trees**: A hierarchy of domains in AD DS that share the same namespace with the parent domain. This comes handy for large organizations that manage multiple domains independently (for example different countries/regulations). Domains in a tree create by default a two-way transitive trust.

4. **Forests**: A collection of one or more domain trees. This comes handy when two companies merge for example or need to cooperate (each with its domain tree). This union of trees with different namespaces into the same network creates the forest.

### Example

```{figure} ../_static/AD/AD_structure.png
:alt: Active Directory Forest Example
:width: 100%
:align: center

Active Directory Forest Example

```

### Authentication

Credentials are stored in the Domain Controllers. Inorder to authenticate a user, two protocols can be mainly used:

#### Kerberos

This is the default protocol. It relies on tickets and mutual authentication.

```{figure} ../_static/AD/krb-auth.png
:alt: Sequence Diagram of Kerberos Authentication
:width: 100%
:align: center

Kerberos Authentication Protocol

```

1. Request TGT (AS-REQ): The user sends their username and a timestamp encrypted with a key derived from their password to the KDC’s Authentication Service (AS), requesting a Ticket Granting Ticket (TGT).

2. Receive TGT + Session Key (AS-REP): The KDC validates the request and responds with a TGT (encrypted with the krbtgt account hash) and a Session Key (encrypted with the user’s key), allowing the user to request service tickets without resending their password.

3. Request TGS (TGS-REQ): The user wants to access a specific service, so they send the TGT, the Service Principal Name (SPN) of the target service, and an authenticator encrypted with the Session Key to the KDC’s Ticket Granting Service (TGS).

4. Receive TGS + Service Session Key (TGS-REP): The KDC verifies the TGT and authenticator, then issues a service ticket (TGS) encrypted with the service account’s key, along with a Service Session Key encrypted with the user’s Session Key.

5. Authenticate to Service (AP-REQ / AP-REP): The user presents the service ticket and an authenticator to the target service. The service decrypts the ticket using its account key, verifies the authenticator, and if valid, grants access. Optionally, the service can reply with an AP-REP for mutual authentication.

#### NetNTLM

Legacy protocol based on a challenge-response mechanism.

```{figure} ../_static/AD/ntlm-auth.png
:alt: Sequence Diagram of NTLM Authentication
:width: 100%
:align: center

NTLM Authentication Protocol

```

1. User initiates authentication
   The client (user’s machine) wants to access a resource on a server. It sends the username to the server in plaintext (no password yet).

2. Server issues a challenge
   The server generates a random challenge (nonce) and sends it to the client.

3. Client computes the response
   The client takes the user’s password, applies the MD4 hash function to produce the NT hash, and then encrypts the challenge using this NT hash. This encrypted value is the NTLM response.

4. Server forwards to the Domain Controller (DC)
   The server doesn’t know if the response is valid, so it forwards the username, the challenge it issued, and the client’s response to a domain controller.

5. Domain Controller verifies
   The DC looks up the user in Active Directory, retrieves the stored NT hash, and uses it to encrypt the same challenge. If the DC’s result matches the client’s response, authentication succeeds.

- Passwords are never sent over the wire, but the NT hash acts as a “password equivalent.” If an attacker steals the NT hash (via dumping or interception), they can use it directly in pass-the-hash attacks.
- Unlike Kerberos, NTLM does not bind authentication to a specific service or server, which is why it’s vulnerable to relay attacks.

### LDAP
