
# Lab 1 — Active Directory & Windows Server 2022
 
## Overview
 
Build a production-realistic Active Directory domain with a Domain Controller and a domain-joined Windows 11 client. You'll configure DHCP, DNS, Organisational Units, Group Policy Objects, and user accounts — skills that map directly to enterprise sysadmin and IT support roles.
 
**Domain:** `lab.local` | **Estimated time:** 45–60 minutes | **Difficulty:** Intermediate
 
---
 
## Environment Specs
 
| Component | Detail |
|-----------|--------|
| Hypervisor | VirtualBox 7.x (free) |
| **DC01** — Domain Controller | Windows Server 2022 Eval |
| DC01 IP | `192.168.56.101` (static) |
| DC01 Resources | 4 GB RAM · 2 CPU cores · 60 GB VDI |
| **CLIENT01** — Domain Client | Windows 11 (22H2+) |
| CLIENT01 IP | DHCP from DC01 |
| CLIENT01 Resources | 2 GB RAM · 2 CPU cores · 40 GB VDI |
| Network | VirtualBox Host-only Adapter (`vboxnet0`) |
 
> **Note:** Download the free 180-day Server 2022 evaluation ISO from [Microsoft's Evaluation Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022). No product key is required for lab use.
 
---
 
## Step 1 — Create the DC01 Virtual Machine
 
~10 minutes
 
1. VirtualBox → **New** → Name: `DC01`
2. Type: **Microsoft Windows** · Version: **Windows 2022 (64-bit)**
3. RAM: `4096 MB` · CPU: `2 cores` · Enable EFI: **off**
4. Disk: **VDI, dynamically allocated, 60 GB**
5. After creation → Settings → **Network** → Adapter 1: **Host-only Adapter** → `vboxnet0`
6. Settings → **Storage** → attach the Server 2022 ISO to the optical drive
7. Boot DC01 → select **Windows Server 2022 Standard (Desktop Experience)** → Custom install on Disk 0
 
---
 
## Step 2 — Set a Static IP on DC01
 
~5 minutes
 
A Domain Controller must have a static IP — it cannot be a DHCP client for the services it hosts. Open **PowerShell as Administrator**:
 
```powershell
# Identify the interface name
Get-NetAdapter
 
# Set static IP, subnet, gateway, and DNS (pointing to itself)
New-NetIPAddress -InterfaceAlias "Ethernet" `
  -IPAddress "192.168.56.101" `
  -PrefixLength 24 `
  -DefaultGateway "192.168.56.1"
 
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" `
  -ServerAddresses "192.168.56.101"
 
# Verify
ipconfig /all
```
 
> ⚠️ Replace `"Ethernet"` with the actual adapter name shown by `Get-NetAdapter`. It may be `"Ethernet 2"` or similar.
 
---
 
## Step 3 — Install AD DS and Promote to Domain Controller
 
~15 minutes
 
Install the role, then run the promotion. The server reboots automatically.
 
```powershell
# Install the AD DS role and management tools
Install-WindowsFeature -Name AD-Domain-Services `
  -IncludeManagementTools
 
# Promote to DC — creates new forest lab.local
Import-Module ADDSDeployment
 
Install-ADDSForest `
  -DomainName "lab.local" `
  -DomainNetbiosName "LAB" `
  -ForestMode "WinThreshold" `
  -DomainMode "WinThreshold" `
  -InstallDns:$true `
  -SafeModeAdministratorPassword `
    (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force) `
  -Force:$true
```
 
After the reboot, log back in as `LAB\Administrator`, then verify:
 
```powershell
Get-ADDomain
```
 
> 💡 The DSRM password is your break-glass recovery credential for the AD database. In a real environment, store it in a password manager.
 
---
 
## Step 4 — Configure DHCP Server
 
~5 minutes
 
DHCP lets CLIENT01 receive an IP automatically. The scope starts at `.110` to avoid conflicting with DC01 at `.101`.
 
```powershell
# Install DHCP role
Install-WindowsFeature -Name DHCP -IncludeManagementTools
 
# Authorize the DHCP server in AD (required — without this, DHCP silently refuses to hand out leases)
Add-DhcpServerInDC -DnsName "dc01.lab.local" `
  -IPAddress "192.168.56.101"
 
# Create a scope
Add-DhcpServerv4Scope -Name "Lab Scope" `
  -StartRange "192.168.56.110" `
  -EndRange "192.168.56.200" `
  -SubnetMask "255.255.255.0" `
  -State Active
 
# Set gateway and DNS options for clients
Set-DhcpServerv4OptionValue -ScopeId "192.168.56.0" `
  -Router "192.168.56.101" `
  -DnsServer "192.168.56.101" `
  -DnsDomain "lab.local"
```
 
---
 
## Step 5 — Create OUs and Test User Accounts
 
~5 minutes
 
Organisational Units mirror real business departments and are the foundation for applying GPOs to specific groups of users or computers.
 
```powershell
# Create department OUs
$base = "DC=lab,DC=local"
foreach ($ou in "IT_Department","Finance","HR","Computers") {
  New-ADOrganizationalUnit -Name $ou -Path $base
}
 
# Create test users
$pass = ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force
 
$users = @(
  @{Name="John Smith";  SamAccountName="jsmith"; OU="IT_Department"},
  @{Name="Alice Wong";  SamAccountName="awong";  OU="Finance"},
  @{Name="Bob Patel";   SamAccountName="bpatel"; OU="HR"}
)
 
foreach ($u in $users) {
  New-ADUser `
    -Name $u.Name `
    -SamAccountName $u.SamAccountName `
    -UserPrincipalName "$($u.SamAccountName)@lab.local" `
    -Path "OU=$($u.OU),DC=lab,DC=local" `
    -AccountPassword $pass `
    -Enabled $true `
    -PasswordNeverExpires $true
}
 
# Confirm users were created
Get-ADUser -Filter * -Properties Department | Select Name, SamAccountName
```
 
---
 
## Step 6 — Create and Link a GPO
 
~5 minutes
 
Group Policy is one of the most critical AD skills. This starter GPO sets a logon banner — a real compliance requirement in most organisations.
 
```powershell
# Create a GPO and link it to the domain root
New-GPO -Name "Security Baseline" | `
  New-GPLink -Target "DC=lab,DC=local"
 
# Set a logon warning banner via registry policy
Set-GPRegistryValue -Name "Security Baseline" `
  -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ValueName "LegalNoticeCaption" `
  -Type String -Value "Authorised Access Only"
 
Set-GPRegistryValue -Name "Security Baseline" `
  -Key "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -ValueName "LegalNoticeText" `
  -Type String -Value "This system is for authorised lab use only."
```
 
---
 
## Step 7 — Create CLIENT01 and Join the Domain
 
~10 minutes
 
**Create the VM:**
 
1. New VM → Name: `CLIENT01` · Windows 11 (64-bit) · 2 GB RAM · 40 GB VDI
2. Settings → System → enable **TPM 2.0** (required for Windows 11)
3. Network Adapter 1: **Host-only** → `vboxnet0`
4. Install Windows 11. At the network screen, press `Shift+F10` → run `OOBE\BYPASSNRO` → reboot → choose **"I don't have internet"** to create a local account and skip the Microsoft account requirement.
 
**Join the domain** — open PowerShell as local Administrator on CLIENT01:
 
```powershell
# Confirm DHCP lease and DNS resolution
ipconfig /all
Resolve-DnsName dc01.lab.local
 
# Join the domain (prompts for domain admin credentials)
Add-Computer -DomainName "lab.local" `
  -OUPath "OU=Computers,DC=lab,DC=local" `
  -Credential (Get-Credential) `
  -Restart
```
 
When prompted, enter `LAB\Administrator` and the admin password. After the reboot, log in as `LAB\jsmith` to test a domain user login.
 
---
 
## Verification Checklist
 
Run these on DC01 and CLIENT01 to confirm everything is working.
 
| Check | Command | Expected Result |
|-------|---------|----------------|
| AD users exist | `Get-ADUser -Filter * \| Measure-Object` | Count ≥ 5 |
| DHCP scope is active | `Get-DhcpServerv4Scope` | State: Active · range 192.168.56.110–200 |
| Domain login works | `whoami` (on CLIENT01) | `lab\jsmith` |
| GPO applied | `gpresult /r` (on CLIENT01) | "Security Baseline" listed under Computer Settings |
| DNS resolving | `Resolve-DnsName client01.lab.local` (on DC01) | Returns CLIENT01's IP |
| Computer object in AD | `Get-ADComputer CLIENT01` (on DC01) | Object exists in OU=Computers |
 
---
 
## Troubleshooting
 
**CLIENT01 can't resolve `lab.local` / domain join fails**
- Verify DNS points to DC01: `Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.56.101"`
- Then test: `Resolve-DnsName dc01.lab.local`
 
**DHCP not handing out addresses**
- Confirm the server is authorised in AD: `Get-DhcpServerInDC`
- If missing, run: `Add-DhcpServerInDC -DnsName "dc01.lab.local" -IPAddress "192.168.56.101"`
 
**Windows 11 forces Microsoft account during setup**
- At the network screen: `Shift+F10` → `OOBE\BYPASSNRO` → reboot → "I don't have internet"
 
**GPO not applying on CLIENT01**
- Force a refresh: `gpupdate /force`
- Check for errors: `gpresult /r`
 
**AD promotion fails with DNS conflict**
- Before promotion, set DNS to `127.0.0.1` temporarily, then switch back to the static IP post-reboot
 
---
