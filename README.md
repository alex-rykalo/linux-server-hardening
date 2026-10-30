# linux-server-hardening
Production-ready Ubuntu Server hardening, validated with real attacks from Kali Linux

## Overview

This project demonstrates the end-to-end security hardening of a fresh
Ubuntu Server installation, following the principle of least privilege.
Each control was not only configured but **verified through real attacks**
launched from a dedicated Kali Linux machine on an isolated lab network.

## Architecture
```
┌─────────────────┐         Host-only network          ┌──────────────────────┐
│   Kali Linux    │         192.168.56.0/24            │   Ubuntu Server      │
│  (attacker)     │ ─────────────────────────────────► │   (hardened target)  │
│  192.168.56.2   │         SSH over port 2222         │   192.168.56.4       │
└─────────────────┘                                    └──────────────────────┘
                                                         NAT adapter → Internet
```


## Technologies Used

- Ubuntu Server 26.10 (ARM64)
- OpenSSH (key-based authentication, custom port)
- UFW (Uncomplicated Firewall)
- Fail2Ban (brute-force protection)
- systemd (service management)
- unattended-upgrades (automatic security patches)

## What Was Implemented

### 1. User & Privilege Management
- Non-root user with sudo access (principle of least privilege)
- Daily operations run as unprivileged user; root access only via sudo

### 2. SSH Hardening
- Key-based authentication (ED25519), password login disabled
- Root login over SSH disabled
- Default port changed from 22 to 2222
- Resolved Ubuntu's `ssh.socket` activation overriding the config port

### 3. Firewall (UFW)
- Default policy: deny all incoming, allow outgoing
- Only the SSH port (2222) is exposed; all other ports closed

### 4. Brute-Force Protection (Fail2Ban)
- Monitors SSH logs, bans IPs after 3 failed attempts
- Ban time: 1 hour, detection window: 10 minutes

### 5. Automatic Security Updates
- unattended-upgrades enabled for automatic security patching

## Testing & Validation

Every control was validated with real attacks from Kali Linux:

| Test | Expected result | Status |
|------|----------------|--------|
| Key-based login on port 2222 | Access granted without password | ✅ |
| Password authentication attempt | `Permission denied (publickey)` | ✅ |
| Root login attempt | Denied | ✅ |
| Brute-force attack (4 attempts) | Attacker IP auto-banned by Fail2Ban | ✅ |
| Access after reboot | All controls persisted | ✅ |

## Challenges & Solutions

- **ssh.socket overriding the SSH port** — modern Ubuntu uses socket
  activation that held port 22 regardless of `sshd_config`. Disabled
  `ssh.socket` so the service listens on the configured port 2222.
- **Dual default routes breaking Internet access** — the Host-only
  interface received a default route with higher priority than NAT,
  sending Internet traffic to an isolated network. Corrected the routing
  so outbound traffic uses the NAT interface.
- **Fail2Ban banning the gateway IP** — in the NAT/lab network, SSH
  traffic appeared to originate from the gateway address. Noted the
  real-world implication for proxies/load balancers (X-Forwarded-For).

## Skills Demonstrated

- Linux system administration (users, permissions, systemd, networking)
- SSH security hardening and key-based authentication
- Firewall configuration (UFW/iptables)
- Intrusion prevention (Fail2Ban)
- Attack simulation and security validation
- Troubleshooting (routing, socket activation, service debugging)

## Screenshots

### Firewall active — only SSH port exposed
`sudo ufw status verbose` — default deny incoming, only port 2222 allowed.

<img width="267" height="106" alt="Знімок екрана 2026-10-29 о 00 06 15" src="https://github.com/user-attachments/assets/d0a43cbe-ae8f-443f-a0b9-159628babe51" />


### SSH listening on custom port 2222
`sudo ss -tulnp | grep ssh` — SSH moved from port 22 to 2222.

<img width="461" height="36" alt="Знімок екрана 2026-10-29 о 00 06 55" src="https://github.com/user-attachments/assets/03e4cbcb-d488-474f-8579-1fadc4aebbf8" />


### SSH hardening configuration
`/etc/ssh/sshd_config.d/99-hardening.conf` — root login disabled, password auth disabled, custom port.

<img width="305" height="42" alt="Знімок екрана 2026-10-29 о 00 07 58" src="https://github.com/user-attachments/assets/6f1f59a0-a0c5-4333-a781-be9b5a2d7512" />


### Key-based login works (no password prompt)
Successful SSH login from Kali using the key pair — no password required.

<img width="282" height="227" alt="Знімок екрана 2026-10-29 о 00 09 21" src="https://github.com/user-attachments/assets/e4fd5ab1-7c05-4e90-ae9c-7818a4c88d10" />


### Password authentication rejected
Attempt to log in with a password is denied — `Permission denied (publickey)`.

<img width="282" height="74" alt="Знімок екрана 2026-10-29 о 00 10 26" src="https://github.com/user-attachments/assets/c8b24541-7200-4186-9577-50b8f60f794d" />


### Root login blocked
Direct root login over SSH is denied.

<img width="224" height="34" alt="Знімок екрана 2026-10-29 о 00 10 47" src="https://github.com/user-attachments/assets/68fbbc31-45cc-4f1d-82a4-acb5a269fa33" />


### Fail2Ban bans the attacker in real time
After repeated failed login attempts from Kali, Fail2Ban automatically bans the source IP. Subsequent connections are refused.


<img width="416" height="421" alt="Знімок екрана 2026-10-29 о 00 12 21" src="https://github.com/user-attachments/assets/b534bc7c-53ca-428c-a52a-f2c140e056c2" />

### Automatic security updates enabled
`unattended-upgrades` configured for daily automatic security patching.

<img width="280" height="42" alt="Знімок екрана 2026-10-29 о 00 13 13" src="https://github.com/user-attachments/assets/f2ccd3bd-b2b9-4e3f-86ac-359269a9102c" />


### All services enabled on boot
`systemctl is-enabled ssh fail2ban ufw` — all security services persist across reboots.

<img width="284" height="35" alt="Знімок екрана 2026-10-29 о 00 14 07" src="https://github.com/user-attachments/assets/36dcfd58-c605-41b4-9478-fd68cfcb7b5e" />
