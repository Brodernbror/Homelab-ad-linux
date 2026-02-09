# Troubleshooting Notes

This document summarizes key issues encountered during the setup and how they were resolved.

## Networking
- Initial connectivity issues in VirtualBox Internal Network
- Resolved by configuring static IP and correct routing via netplan

## DNS
- `systemd-resolved` caused name resolution failures in isolated environment
- Fixed by using static `/etc/resolv.conf` pointing to the Domain Controller

## Kerberos / realmd
- Realm discovery failed when using lowercase domain name
- Resolved by using uppercase Kerberos realm (`LAB.LOCAL`)
- Manual `realm join` used when automatic discovery failed

## SSSD / PAM
- Login failures caused by incorrect `sssd.conf` configuration
- Fixed:
  - `domains` vs `domain` key
  - `%u` vs `%U` in `fallback_homedir`
- Correct file permissions required (`600`)

## SSH
- Differentiated between:
  - `Connection timed out` (network issue)
  - `Connection refused` (sshd not listening)
  - `Permission denied` (PAM / SSSD authorization)
- Domain Admin login intentionally restricted
- Regular domain users authenticated successfully
