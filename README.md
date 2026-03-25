<div align="center">

![Header](https://img.shields.io/badge/Aaron%20O'Neill-IT%20Support%20%26%20Cybersecurity-007ACC?style=for-the-badge&logo=microsoft&logoColor=white)

[![GitHub followers](https://img.shields.io/github/followers/YOURUSERNAME?label=follow&style=social)](https://github.com/YOURUSERNAME)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=social&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/aaron-oneill-ab3a421a4/)
[![Email](https://img.shields.io/badge/Email-Contact%20Me-EA4335?style=social&logo=gmail&logoColor=white)](mailto:aarononeill11cfc@gmail.com)

</div>

---

## 👨‍💻 About Me

**Technical Deployment Specialist at CompNow (Sydney, NSW)**  
Hands-on IT professional specializing in Windows 11 enterprise rollouts, Microsoft ecosystem management, and L1/L2 support. Currently building a home lab portfolio to accelerate path to NOC/Desktop Support roles.

- **Experience:** Managed Autopilot deployments for NSW Department of Education; Apple Business Manager for AFL Corporate fleet.
- **Education:** Diploma of Cyber Security (2025) | Cert IV IT (2025) | Google IT Support Cert (in progress).
- **Targets:** AZ-900 (Q2 2026) | CCNA (June 2026). Open to Sydney-based Helpdesk/NOC opportunities.

<div align="center">

| **Core Skills** | **Level** | **Tools/Tech** |
|-----------------|-----------|----------------|
| Active Directory | 🟢 Advanced | ADUC, PowerShell, GPO |
| Microsoft 365 | 🟢 Advanced | Intune, Entra ID, Autopilot |
| Networking (L1/L2) | 🟡 Intermediate | TCP/IP, Packet Tracer, ping/tracert |
| ITSM | 🟡 Intermediate | ServiceNow workflows |
| Scripting | 🟡 Intermediate | PowerShell |

</div>

---

## 🛠️ Home Lab Portfolio

Hands-on labs demonstrating real-world L1/L2 skills. Each includes setup instructions, challenges overcome, and verifiable outputs.

| # | Lab | Key Skills | Status | 📊 Metrics |
|---|-----|------------|--------|------------|
| 1 | [Active Directory & Server 2022](./lab1-active-directory/) | AD DS, DHCP, GPO, PowerShell | ✅ Complete | 50+ users/groups created |
| 2 | [Microsoft 365 & Intune](./lab2-m365-intune/) | MDM, MFA, Compliance Policies | ✅ Complete | 10+ policies/devices |
| 3 | [ServiceNow ITSM](./lab3-servicenow/) | Incidents/Changes/Problems | ✅ Complete | NOC dashboard built |
| 4 | [Cisco Networking](./lab4-networking/) | Subnetting, Routing, DHCP/DNS | 🔨 In Progress | 2-subnet topology |
| 5 | [L1 Network Troubleshooting](./lab5-network-troubleshooting/) | Ping/DNS/Gateway triage | 🆕 Planned | 5 common scenarios |

---

## Lab Highlights

### Lab 1: Active Directory Domain Controller
**Built:** Windows Server 2022 DC + Windows 11 client on VirtualBox. Full OU structure, GPO enforcement, domain join.

**Achievements:**
- Automated 20+ user provisioning via PowerShell.
- Implemented lockout policies reducing simulated brute-force risks.

**PowerShell Example:**
```powershell
New-ADUser -Name "Jane Smith" -SamAccountName "jsmith" -Path "OU=IT,DC=lab,DC=local" -Enabled $true -AccountPassword (ConvertTo-SecureString "P@ssw0rd123" -AsPlainText -Force)
```

[Runbook](lab1-active-directory/runbook.md) | [Screenshots](lab1-active-directory/screenshots/)

### Lab 5: L1 Network Troubleshooting (New!)
**Scenarios:** Simulated tickets for ping failures, DNS issues, gateway problems using Packet Tracer.

**L1 Triage Decision Tree:**
