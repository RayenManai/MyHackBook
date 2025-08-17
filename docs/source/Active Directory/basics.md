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

1. **Kerberos**: This is the default protocol. It relies on tickets and mutual authentication.

2. **NetNTLM**: Legacy protocol based on a challenge-response mechanism.
