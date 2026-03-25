
# Lab 1: Active Directory & Windows Server 2022

## Overview
Built production-like AD domain (lab.local) with DC01 (Server 2022) + CLIENT01 (Win11) on VirtualBox. Configured users, OUs, GPOs, DHCP.

**Environment Specs:**
- Hypervisor: VirtualBox 7.x (free)
- DC01: Win Server 2022 Eval | IP: 192.168.56.101 | RAM: 4GB | Disk: 60GB
- CLIENT01: Win11 | DHCP client | RAM: 2GB | Disk: 40GB

## Setup Instructions (30-45 mins)

### 1. Create VMs
VirtualBox → New → DC01

Type: Microsoft Windows | Version: Windows 2022 (64-bit)

RAM: 4096MB | CPU: 2 cores

Disk: VDI 60GB Dynamic | Network: Host-Only Adapter (vboxnet0)

text
Attach Server 2022 ISO → Install (Desktop Experience).

### 2. Promote DC01 to Domain Controller
Server Manager → Add Roles → Active Directory Domain Services
→ Install → Promote to DC → New Forest: lab.local
→ DSRM Password: P@ssw0rd123
Reboot.

text

### 3. Configure DHCP & DNS
Server Manager → Tools → DHCP → IPv4 → New Scope

Scope: 192.168.56.100-200 | Router: 192.168.56.101 | DNS: 192.168.56.101
→ Activate Scope

text

### 4. Create OUs & Test Users
Active Directory Users → New OU: IT_Department, Finance, HR
PowerShell (Admin):
New-ADUser jsmith -Path "OU=IT_Department,DC=lab,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force) -Enabled $true

text

### 5. Join CLIENT01 to Domain
CLIENT01 Settings → Network: vboxnet0 → ipconfig /renew
System Properties → Change → Domain: lab.local → Reboot
Login: lab\jsmith

text

## Verification
DC01: Get-ADUser -Filter * | Measure → 20+ users
CLIENT01: whoami → lab\jsmith

text
