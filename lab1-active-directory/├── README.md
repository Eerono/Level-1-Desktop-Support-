# Lab 1 — Active Directory & Windows Server 2022

Set up a basic Active Directory domain using two virtual machines — no commands needed, everything through the graphical interface.

**Domain:** `lab.local` · **Time:** 45–60 min

---

## What You Need to Download First

| Download | Link |
|----------|------|
| VirtualBox (free) | [virtualbox.org](https://www.virtualbox.org/wiki/Downloads) |
| Windows Server 2022 (free 180-day trial) | [Microsoft Eval Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) |
| Windows 11 ISO | [microsoft.com](https://www.microsoft.com/software-download/windows11) |

---

## Part 1 — Create Your Two Virtual Machines in VirtualBox

### VM 1: DC01 (the server)

1. Open VirtualBox → click **New**
2. Name: `DC01` · Type: **Microsoft Windows** · Version: **Windows 2022 (64-bit)** → Next
3. RAM: **4096 MB** · CPU: **2** → Next
4. Disk size: **60 GB** → Next → Finish
5. Click **DC01** → **Settings → Network → Adapter 1** → change *Attached to* to **Host-only Adapter** → OK
6. **Settings → Storage** → click the empty disc icon → click the disc icon on the right → **Choose a disk file** → select your Server 2022 ISO → OK
7. Click **Start** → Windows setup will launch

**During Windows setup:**
- Language/keyboard → Next → Install Now
- Select **Windows Server 2022 Standard Evaluation (Desktop Experience)** → Next
- Accept the licence → **Custom install** → select Disk 0 → Next
- Windows installs and restarts — set an Administrator password when prompted

---

### VM 2: CLIENT01 (the client PC)

1. Click **New** → Name: `CLIENT01` · Version: **Windows 11 (64-bit)**
2. RAM: **2048 MB** · CPU: **2** · Disk: **40 GB** → Finish
3. **Settings → System → Motherboard** → tick **Enable EFI** → under the **TPM** section enable **TPM 2.0**
4. **Settings → Network → Adapter 1** → **Host-only Adapter** → OK
5. **Settings → Storage** → attach the Windows 11 ISO
6. Click **Start** → follow Windows 11 setup

**Skipping the Microsoft account requirement:**
- When setup asks to connect to Wi-Fi → press `Shift + F10` → a black window opens → type `OOBE\BYPASSNRO` → press Enter → VM reboots
- Go through setup again → when asked about internet → click **"I don't have internet"** → **"Continue with limited setup"** → create a local username and password


<img width="832" height="491" alt="image" src="https://github.com/user-attachments/assets/94c7b23e-b196-404f-b52a-0dc9f97d32ef" />

---

## Part 2 — Install Active Directory on DC01

1. Log into **DC01** → open **Server Manager** (it opens automatically)
2. Click **Manage** (top right) → **Add Roles and Features**
3. Click **Next** three times until you reach *Server Roles*
4. Tick **Active Directory Domain Services** → click **Add Features** → Next → Next → Next → **Install**
5. Wait for it to finish — do **not** close the window early
<img width="857" height="550" alt="step 3 2" src="https://github.com/user-attachments/assets/48d594ee-15e7-4c66-a301-081be78db1f1" />

**Promote the server to a Domain Controller:**

6. In Server Manager, click the **yellow flag** (top right) → **Promote this server to a domain controller**
7. Select **Add a new forest** → Root domain name: `lab.local` → Next
8. Set a **DSRM password** (e.g. `P@ssw0rd123!`) → Next → Next → Next → Next → Next → **Install**
9. The server restarts automatically — log back in as `LAB\Administrator`
<img width="796" height="641" alt="step 5 1" src="https://github.com/user-attachments/assets/e06a408e-7c3a-488a-8924-756572c8de7c" />

---

## Part 3 — Create Organisational Units and Users

### Create OUs

1. In Server Manager → **Tools** → **Active Directory Users and Computers**
2. Expand `lab.local` in the left panel
3. Right-click `lab.local` → **New → Organizational Unit**
4. Create these four OUs one at a time:
   - `IT_Department`
   - `Finance`
   - `HR`
   - `Computers`
<img width="1920" height="1200" alt="Screenshot (272)" src="https://github.com/user-attachments/assets/0de5c838-74cf-4e66-ad9b-bb40d7bf2238" />

### Create Users

1. Click the `IT_Department` OU → right-click in the right panel → **New → User**
2. Fill in:
   - First name: `John` · Last name: `Smith` · User logon name: `jsmith` → Next
   - Password: `P@ssw0rd123!`
   - Untick **User must change password at next logon** · Tick **Password never expires** → Next → Finish
3. Repeat for:
   - `Alice Wong` / `awong` → place in `Finance`
   - `Bob Patel` / `bpatel` → place in `HR`
<img width="1920" height="1200" alt="Screenshot (274)" src="https://github.com/user-attachments/assets/97cab8d3-b49d-45d7-95a4-9da7efb03ba2" />

---

## Part 4 — Create a Security Policy (GPO)

1. In Server Manager → **Tools** → **Group Policy Management**
2. Expand **Forest: lab.local → Domains → lab.local**
3. Right-click `lab.local` → **Create a GPO in this domain and link it here**
4. Name it `Security Baseline` → OK
5. Right-click **Security Baseline** → **Edit**
6. Navigate to: **Computer Configuration → Policies → Windows Settings → Security Settings → Local Policies → Security Options**
7. Double-click **Interactive logon: Message title for users attempting to log on** → tick *Define* → type `Authorised Access Only` → OK
8. Double-click **Interactive logon: Message text for users attempting to log on** → tick *Define* → type `This system is for authorised lab use only.` → OK
9. Close the editor
<img width="1920" height="1200" alt="Screenshot (282)" src="https://github.com/user-attachments/assets/55410c14-74dc-4696-a9bd-e008ed072bc7" />

---

## Part 5 — Join CLIENT01 to the Domain

1. Log into **CLIENT01** with your local account
2. Right-click the **Start menu** → **System**
3. Scroll down → click **Advanced system settings** → **Computer Name** tab
4. Click **Change** → select **Domain** → type `lab.local` → OK
5. Enter credentials when prompted:
   - Username: `LAB\Administrator`
   - Password: your DC01 admin password
6. You'll see *"Welcome to the lab.local domain"* → OK → restart when prompted
7. At the login screen click **Other user** → log in as `LAB\jsmith` with password `P@ssw0rd123!`
<img width="1920" height="1200" alt="Screenshot (283)" src="https://github.com/user-attachments/assets/f8283fca-aba3-4969-81f6-ec19b9af84e2" />

---

## Quick Checks — Did It Work?

| What to check | Where to look | Expected result |
|---------------|---------------|-----------------|
| Users exist | AD Users and Computers on DC01 | jsmith, awong, bpatel visible |
| CLIENT01 joined | AD Users and Computers → Computers OU | CLIENT01 listed |
| Domain login works | Log into CLIENT01 as `LAB\jsmith` | Desktop loads successfully |
| GPO applied | CLIENT01 login screen | Warning banner appears before login |

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| CLIENT01 can't find the domain | Both VMs must be set to **Host-only Adapter** in VirtualBox network settings |
| Windows 11 won't skip Microsoft account | Press `Shift + F10` → type `OOBE\BYPASSNRO` → Enter |
| GPO banner not showing | On DC01 → Group Policy Management → right-click **Security Baseline** → **GPUpdate** |
| Can't log in as domain user | Click **Other user** on the login screen and type `LAB\jsmith` |
