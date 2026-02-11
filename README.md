# Active Directory HomeLab – Linux Integration

## Overview

This project documents an isolated Active Directory homelab where an Ubuntu Linux client is joined to a Windows Server 2022 domain using Kerberos and SSSD.

The lab simulates a realistic enterprise environment with:

* Isolated networking
* Centralized authentication via Active Directory
* Linux ↔ Active Directory integration
* Group Policy enforcement
* Structured troubleshooting

---

## Network Topology

* **Virtualization:** Microsoft Hyper-V (migrated from VirtualBox)
* **Network type:** Internal Virtual Switch (isolated VM-to-VM communication)
* **IP range:** 192.168.100.0/24

## Network Diagram

[![Network Diagram](diagrams/network-topology.png)](diagrams/network-topology.png)

---

### Machines

**DC01 – Windows Server 2022**

* IP: 192.168.100.10
* Roles:
  + Active Directory Domain Services
  + DNS Server
  + Group Policy Management
* Domain: lab.local
* Network: Lab-Internal only (no internet access)

**Ubuntu Client**

* IP: 192.168.100.20 (static, via netplan)
* DNS: DC01 (192.168.100.10)
* Network adapters:
  + eth0: Lab-Internal (domain communication)
  + eth1: Lab-External (internet access, SSH)
* Domain-joined via `realmd`

---

## Domain Controller Setup

* New AD forest: **lab.local**
* DNS configured with required A and SRV records
* Administrator account used only for domain administration

### Group Policies Configured

Three GPOs have been created and applied to the domain:

1. **Security - Password and Lockout Policy**
   * Minimum password length: 12 characters
   * Password complexity requirements: Enabled
   * Account lockout threshold: 5 invalid attempts
   * Account lockout duration: 15 minutes

2. **Security - Login Banner**
   * Interactive logon message displayed before authentication
   * Legal notice for authorized access only

3. **User - Map Network Drives**
   * Automatically maps H: drive to `\\DC01\UserData`
   * Configured via Group Policy Preferences

---

## Linux AD Join

* Static IP configured via netplan
* DNS configured manually (no systemd-resolved)
* Joined domain using `realmd`
* Kerberos realm: LAB.LOCAL
* Authentication handled via SSSD and PAM

### Key Configuration (sssd.conf)

```
[sssd]
services = nss, pam
domains = lab.local

[domain/lab.local]
id_provider = ad
use_fully_qualified_names = False
fallback_homedir = /home/%u
```

### Netplan Configuration

The Ubuntu client uses dual network interfaces:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses: [192.168.100.20/24]
      nameservers:
        addresses: [192.168.100.10]
    eth1:
      dhcp4: true
```

* **eth0:** Lab-Internal – communicates with DC01 and domain services
* **eth1:** Lab-External – provides internet access and SSH connectivity

---

## Hyper-V Migration

This lab was originally built in VirtualBox and has been successfully migrated to Microsoft Hyper-V for improved performance.

### Virtual Switches

Two virtual switches were configured:

* **Lab-Internal:** Internal network for isolated VM communication (simulates enterprise LAN)
* **Lab-External:** External network providing internet access and SSH connectivity to host machine

### Migration Process

1. Converted VirtualBox `.vdi` disks to Hyper-V `.vhdx` format using VBoxManage and Convert-VHD
2. Created Generation 1 VMs in Hyper-V (required for BIOS-based boot)
3. Attached converted virtual hard disks
4. Reconfigured network adapters to use Hyper-V virtual switches
5. Updated netplan configuration on Ubuntu to match new interface names

---

## Remote Access

* **SSH to Ubuntu:** Connect via eth1 IP address (assigned by DHCP on Lab-External)
* **RDP to DC01:** Not recommended for production scenarios; DC should remain isolated

---

## Next Steps

Potential enhancements for the lab:

* Add a Windows 10/11 client joined to the domain
* Implement centralized logging (Syslog/Wazuh)
* Configure file shares with NTFS permissions
* Set up Organizational Units (OUs) for structured GPO application
* Deploy additional domain services (DHCP, Certificate Services)

---

## Documentation

* [GPO Configuration Guide](docs/GPO-guide.md)
* [Troubleshooting](troubleshooting.md)
* [Network Configuration Examples](config-examples/)

---

## About

This homelab serves as a learning environment for:

* Active Directory administration
* Linux integration in Windows environments
* Group Policy management
* Network isolation and segmentation
* Virtualization platform migration

**Platform:** Microsoft Hyper-V (Windows 11 Pro)  
**Domain:** lab.local  
**Authentication:** Kerberos + SSSD
