# Active Directory HomeLab – Linux Integration

## Overview
This project documents an isolated Active Directory homelab where an Ubuntu Linux client is joined to a Windows Server 2022 domain using Kerberos and SSSD.

The lab simulates a realistic enterprise environment with:
- Isolated networking
- Centralized authentication
- Linux ↔ Active Directory integration
- Structured troubleshooting

---

## Network Topology
- Virtualization: VirtualBox
- Network type: Internal Network (VM ↔ VM only)
- IP range: 192.168.100.0/24

## Network Diagram
![Network Diagram](diagrams/network-topology.png)

---

### Machines
**DC01 – Windows Server 2022**
- IP: 192.168.100.10
- Roles:
  - Active Directory Domain Services
  - DNS
- Domain: lab.local

**Ubuntu Client**
- IP: 192.168.100.20
- DNS: DC01 only

---

## Domain Controller Setup
- New AD forest: lab.local
- DNS configured with required A and SRV records
- Administrator account used only for domain administration

---

## Linux AD Join
- Static IP configured via netplan
- DNS configured manually (no systemd-resolved)
- Joined domain using `realmd`
- Kerberos realm: LAB.LOCAL
- Authentication handled via SSSD and PAM

### Key Configuration (sssd.conf)
```ini
[sssd]
services = nss, pam
domains = lab.local

[domain/lab.local]
id_provider = ad
use_fully_qualified_names = False
fallback_homedir = /home/%u
