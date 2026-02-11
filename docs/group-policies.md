# Group Policy Implementation

## Overview

Three Group Policy Objects have been implemented in the lab.local domain to enforce security policies and automate user environment configuration.

---

## Implemented Policies

### 1. Security - Password and Lockout Policy

**Scope:** Domain-wide  
**Purpose:** Enforce strong password requirements and protect against brute-force attacks

| Setting | Value | Rationale |
|---------|-------|-----------|
| Minimum password length | 12 characters | Industry standard for strong passwords |
| Password complexity requirements | Enabled | Requires uppercase, lowercase, numbers, and symbols |
| Maximum password age | 90 days | Regular rotation reduces risk of compromised credentials |
| Minimum password age | 1 day | Prevents immediate password reuse |
| Enforce password history | 10 passwords | Prevents cycling through old passwords |
| Account lockout threshold | 5 invalid attempts | Balances security with user experience |
| Account lockout duration | 15 minutes | Auto-unlock reduces helpdesk burden |
| Reset lockout counter after | 15 minutes | Aligns with lockout duration |

---

### 2. Security - Login Banner

**Scope:** Domain-wide  
**Purpose:** Legal notice and authorized access warning

**Configuration:**
- **Title:** `LAB.LOCAL - Authorized Access Only`
- **Message:** `This system is part of the lab.local domain and is for authorized users only. All activity may be monitored and logged. Unauthorized access is strictly prohibited.`

**Implementation:**
- Windows clients: Display via GPO (Interactive logon policies)
- Linux clients: Configured manually via `/etc/issue` and SSH banner

---

### 3. User - Map Network Drives

**Scope:** Domain Users  
**Purpose:** Automatically map shared storage for domain users

| Setting | Value |
|---------|-------|
| Drive letter | H: |
| Share path | `\\DC01\UserData` |
| Label | UserData |
| Reconnect | Enabled |

**Prerequisites:**
- SMB share `UserData` created on DC01
- NTFS permissions: Domain Users (Read/Change)

---

## Verification

Group Policy application can be verified on client machines:

**Windows:**
```powershell
gpupdate /force
gpresult /r
```

**Linux (Ubuntu):**
- Password policies enforced via SSSD integration with AD
- Login banner visible at `/etc/issue`
- Domain authentication handled by Kerberos/PAM

---

## Future Enhancements

Potential policy additions:
- Organizational Unit (OU) structure for targeted policy application
- Windows Firewall rules via GPO
- Software deployment and restrictions
- Audit policy for security logging
- Screen lock timeout policies
