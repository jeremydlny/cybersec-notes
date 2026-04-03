# Active Directory Basics

## Windows Domains

A Windows domain is a group of users and computers managed centrally by a server called a **Domain Controller (DC)**.

Key advantages:
- Centralised identity management — all users configured from Active Directory
- Security policies can be pushed to all machines from the DC

## Active Directory

The **Active Directory Domain Service (AD DS)** is the core service running on the Domain Controller.

It stores information about:
- Users
- Computers
- Groups
- Policies

### Key objects

**Users** — can represent people or service accounts (e.g. a user running a database service)

**Machines** — every computer that joins the domain gets a machine account (name = computer name + `$`, e.g. `DC01$`). Machine accounts are local admins on their own machine.

**Security Groups** — used to assign permissions to multiple users at once.

| Group | Description |
|---|---|
| Domain Admins | Full control over the domain |
| Server Operators | Can manage DCs, cannot change admin memberships |
| Backup Operators | Can access any file regardless of permissions |
| Account Operators | Can create/modify accounts in the domain |
| Domain Users | All user accounts in the domain |
| Domain Computers | All machines in the domain |
| Domain Controllers | All DCs in the domain |

## Managing Users in AD

Users and objects are organised in **Organisational Units (OUs)** — like folders inside AD DS.

Default OUs created by Windows:
- `Builtin` — default groups
- `Computers` — machines joining the domain land here by default
- `Domain Controllers` — DCs
- `Users` — default users and groups
- `Managed Service Accounts` — service accounts

### Delegation
You can give specific users control over specific OUs without making them Domain Admins.  
Example: give the IT support OU the right to reset passwords in other OUs.

## Managing Computers in AD

Best practice is to separate machines into at least 3 categories:

- **Workstations** — end user machines, should not have privileged accounts logged in
- **Servers** — provide services to users or other servers
- **Domain Controllers** — most sensitive machines, contain hashed passwords for all domain users

## Authentication Methods

### Kerberos Authentication

Default authentication protocol for modern Windows domains.

How it works:
1. User sends username + encrypted timestamp to the **Key Distribution Center (KDC)** running on the DC
2. KDC returns a **Ticket Granting Ticket (TGT)** — proves the user is authenticated without sending the password again
3. User uses the TGT to request a **Ticket Granting Service (TGS)** for a specific service
4. The service validates the TGS and grants access

Key concepts:
- `KDC` — Key Distribution Center, runs on the DC
- `TGT` — Ticket Granting Ticket, proves identity
- `TGS` — Ticket Granting Service, grants access to a specific service
- Tickets are time-limited — reduces risk if intercepted

### NetNTLM Authentication

Older protocol, kept for compatibility. Challenge-response based.

How it works:
1. Client sends authentication request to server
2. Server sends a random **challenge**
3. Client encrypts the challenge using the user's password hash and sends it back
4. Server forwards everything to the DC for validation
5. DC confirms or denies — server grants or refuses access

Key difference from Kerberos: the password hash is used directly, no tickets involved.

## Trees, Forests and Trusts

### Trees
Multiple domains sharing the same namespace can be joined into a **tree**.  
Example: `thm.local` → `uk.thm.local` + `us.thm.local`

Each subdomain can have its own DCs and policies, while sharing the root domain.

### Forests
A **forest** is a collection of trees with different namespaces joined together.  
Example: `thm.local` + `mhtstudios.local` grouped into one forest.

### Trust Relationships
Trusts allow users in one domain to access resources in another.

| Type | Description |
|---|---|
| One-way trust | Domain A trusts Domain B — users from B can access A, not the other way |
| Two-way trust | Both domains trust each other — users from both can access either |

By default, joining domains into a tree or forest creates two-way trusts between them.

## Key Commands

Reset a user password via PowerShell:
```powershell
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose
```

Force password change at next login:
```powershell
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```
