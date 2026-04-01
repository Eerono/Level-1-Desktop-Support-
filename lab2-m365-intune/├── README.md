# Lab 2 — Microsoft 365 & Intune MDM

## Overview

Configure a Microsoft 365 tenant from scratch with real enterprise tooling — user provisioning, multi-factor authentication, Intune device compliance policies, and Windows 11 device enrollment. These skills map directly to cloud sysadmin, IT support, and endpoint management roles.

**Tenant:** `labintune.onmicrosoft.com` | **Estimated time:** 45–60 minutes | **Difficulty:** Intermediate

---

## Environment Specs

| Component | Detail |
|-----------|--------|
| M365 Plan | Microsoft 365 Developer E5 (free 90-day sandbox) |
| Users available | 25 licensed users |
| Admin portal | [admin.microsoft.com](https://admin.microsoft.com) |
| Endpoint portal | [endpoint.microsoft.com](https://endpoint.microsoft.com) |
| Entra ID portal | [entra.microsoft.com](https://entra.microsoft.com) |
| Test device | CLIENT01 (Windows 11 — from Lab 1, or a fresh VM) |

> 💡 **Free sandbox:** Go to [developer.microsoft.com/microsoft-365/dev-program](https://developer.microsoft.com/en-us/microsoft-365/dev-program) → Join → set up a new tenant. This gives you a **90-day renewable E5 sandbox** with 25 users — far better than the standard 30-day Business trial. No credit card required.

---

## Step 1 — Create the M365 Tenant

~5 minutes

1. Go to [developer.microsoft.com/microsoft-365/dev-program](https://developer.microsoft.com/en-us/microsoft-365/dev-program)
2. Sign in with a personal Microsoft account → **Join Now**
3. Choose **Instant sandbox** (recommended) or **Configurable sandbox**
4. Your tenant domain will be assigned as `<yourname>.onmicrosoft.com` — for this lab we use `labintune.onmicrosoft.com`
5. Note your **Global Administrator** credentials — you'll need these throughout

**Verify tenant is active:**

1. Go to [admin.microsoft.com](https://admin.microsoft.com)
2. Sign in as the Global Admin
3. Confirm **Microsoft 365 E5** licence is showing under **Billing → Your products**
<img width="2910" height="1206" alt="image" src="https://github.com/user-attachments/assets/143b45a6-a705-4d23-a9f2-88ca6b179a9e" />

---

## Step 2 — Add Users and Assign Licences

~10 minutes

### Create users via the Admin Centre

1. [admin.microsoft.com](https://admin.microsoft.com) → **Users** → **Active users** → **Add a user**
2. Fill in the details:

| Field | Value |
|-------|-------|
| First name | Jane |
| Last name | Doe |
| Username | `jane.doe@labintune.onmicrosoft.com` |
| Password | Auto-generate (note it down) |
| Licences | Microsoft 365 E5 |
| Roles | No admin access (standard user) |

3. Repeat to create a second test user: `john.smith@labintune.onmicrosoft.com`

4. <img width="977" height="999" alt="image" src="https://github.com/user-attachments/assets/4da14a9b-3a98-455b-ad77-52e7c1eb3bd3" />


### Create users in bulk via PowerShell (optional but good practice)

```powershell
# Install the Microsoft Graph PowerShell module if not already present
Install-Module Microsoft.Graph -Scope CurrentUser -Force

# Connect to your tenant
Connect-MgGraph -Scopes "User.ReadWrite.All", "Directory.ReadWrite.All"

# Create a user
$passwordProfile = @{
  Password                      = "TempP@ss2024!"
  ForceChangePasswordNextSignIn = $true
}

New-MgUser `
  -DisplayName "Jane Doe" `
  -UserPrincipalName "jane.doe@labintune.onmicrosoft.com" `
  -MailNickname "janedoe" `
  -AccountEnabled `
  -PasswordProfile $passwordProfile `
  -UsageLocation "AU"

# Confirm user was created
Get-MgUser -Filter "displayName eq 'Jane Doe'" | Select DisplayName, UserPrincipalName, Id
```

---

## Step 3 — Enable and Configure MFA

~10 minutes

### Enable MFA per user (basic method)

1. [admin.microsoft.com](https://admin.microsoft.com) → **Users** → **Active users**
2. Select `jane.doe` → **Manage multifactor authentication**
3. Check the box next to the user → **Enable**
4. Repeat for all test users

   <img width="2107" height="455" alt="image" src="https://github.com/user-attachments/assets/0884ebe9-ce9f-4cb0-a235-96a873da14a0" />


### Enable MFA via Conditional Access (recommended — enterprise approach)

Conditional Access lets you apply MFA based on conditions (location, device state, risk level).

1. [entra.microsoft.com](https://entra.microsoft.com) → **Protection** → **Conditional Access** → **New policy**
2. Configure:

| Setting | Value |
|---------|-------|
| Name | `Require MFA — All Users` |
| Users | All users (exclude your break-glass admin account) |
| Cloud apps | All cloud apps |
| Conditions | Any location |
| Grant | Require multifactor authentication |
| Policy state | **On** |

3. Save the policy

> ⚠️ Always exclude at least one break-glass admin account from Conditional Access policies. If MFA breaks, you need a way back in. Create a dedicated account (e.g., `breakglass@labintune.onmicrosoft.com`) and store its credentials securely offline.

**Test MFA is working:**

1. Open a private/incognito browser window
2. Sign in as `jane.doe@labintune.onmicrosoft.com`
3. You should be prompted to set up Microsoft Authenticator

---

## Step 4 — Configure Intune Compliance Policy

~10 minutes

A compliance policy defines the minimum security requirements a device must meet. Non-compliant devices can be blocked from accessing company resources via Conditional Access.

1. [endpoint.microsoft.com](https://endpoint.microsoft.com) → **Devices** → **Compliance** → **Policies** → **Create policy**
2. Platform: **Windows 10 and later**
3. Name: `Win11 Security Baseline`
4. Configure the following settings:

**Device Health:**

| Setting | Value |
|---------|-------|
| Require BitLocker | Yes |
| Require Secure Boot | Yes |
| Require code integrity | Yes |

**Device Properties:**

| Setting | Value |
|---------|-------|
| Minimum OS version | `10.0.22000` (Windows 11 minimum build) |

**System Security:**

| Setting | Value |
|---------|-------|
| Require a password | Yes |
| Minimum password length | 8 |
| Microsoft Defender Antimalware | Require |
| Real-time protection | Require |
| Firewall | Require |

**Actions for noncompliance:**

| Action | Schedule |
|--------|----------|
| Mark device noncompliant | Immediately |
| Send email to end user | Day 1 |
| Remotely lock device | Day 3 |
| Retire device | Day 30 |

5. **Assignments** → Assign to **All Devices** (or a test group)
6. Review and **Create**

---

## Step 5 — Create a Configuration Profile

~5 minutes

Configuration profiles push settings to devices — separate from compliance. Use these to configure things like disabling USB storage, enforcing a wallpaper, or setting Windows Update rings.

1. [endpoint.microsoft.com](https://endpoint.microsoft.com) → **Devices** → **Configuration** → **Create** → **New policy**
2. Platform: **Windows 10 and later** · Profile type: **Settings catalog**
3. Name: `Win11 Baseline Config`
4. Add these settings:

| Category | Setting | Value |
|----------|---------|-------|
| Windows Update for Business | Branch readiness level | General Availability Channel |
| Windows Update for Business | Defer feature updates (days) | 14 |
| Defender | Allow real-time monitoring | Allowed |
| Experience | Allow Cortana | Block |
| Storage | Removable disk deny write access | Enabled |

5. Assign to **All Devices** → **Create**

---

## Step 6 — Enroll Windows 11 Device into Intune

~10 minutes

### Method A — Manual enrollment (Settings UI)

On CLIENT01 (Windows 11):

1. **Settings** → **Accounts** → **Access work or school** → **Connect**
2. Click **"Enroll only in device management"** (at the bottom of the sign-in screen)
3. Sign in as `jane.doe@labintune.onmicrosoft.com`
4. Follow the enrollment wizard

### Method B — Enroll via PowerShell (for automation practice)

```powershell
# Trigger MDM enrollment via the enrollment URI
$enrollmentServer = "https://enrollment.manage.microsoft.com/enrollmentserver/discovery.svc"

# Check if already enrolled
Get-Item "HKLM:\SOFTWARE\Microsoft\Enrollments" -ErrorAction SilentlyContinue

# Force a sync after enrollment to pull down policies immediately
$session = New-Object -ComObject Microsoft.MDM.PolicyManager
Start-Process "C:\Windows\System32\deviceenroller.exe" -ArgumentList "/c /AutoEnrollMDM"
```

### Verify enrollment in Intune

1. [endpoint.microsoft.com](https://endpoint.microsoft.com) → **Devices** → **All devices**
2. CLIENT01 should appear within 5–10 minutes
3. Check the **Compliance** column — it will show **Checking** initially, then **Compliant** or **Not compliant**
4. Click the device → **Device compliance** to see which policies passed or failed

> 💡 If CLIENT01 shows **Not compliant**, it's likely because BitLocker isn't enabled on a VM (no TPM). This is expected in a lab. Click the non-compliant setting to see the exact reason — this is the same workflow you'd use to diagnose real endpoints.

---

## Step 7 — Link Compliance to Conditional Access

~5 minutes

This is what makes compliance enforceable — non-compliant devices get blocked from M365 resources.

1. [entra.microsoft.com](https://entra.microsoft.com) → **Protection** → **Conditional Access** → **New policy**
2. Configure:

| Setting | Value |
|---------|-------|
| Name | `Block Non-Compliant Devices` |
| Users | All users |
| Cloud apps | All cloud apps |
| Conditions → Device platforms | Windows |
| Conditions → Filter for devices | `device.isCompliant -eq False` |
| Grant | Block access |
| Policy state | **Report-only** (test first before enforcing) |

3. Run in **Report-only** mode for 24–48 hours and review the **Insights and reporting** tab before switching to **On**

---

## Verification Checklist

| Check | Where | Expected Result |
|-------|-------|----------------|
| Tenant active with E5 licences | admin.microsoft.com → Billing | 25 E5 licences assigned |
| Users created and licensed | admin.microsoft.com → Active users | jane.doe and john.smith visible |
| MFA enforced | Sign in as jane.doe in incognito | Prompted for Authenticator setup |
| Compliance policy created | endpoint.microsoft.com → Compliance | `Win11 Security Baseline` active |
| Device enrolled | endpoint.microsoft.com → All devices | CLIENT01 visible |
| Compliance status | endpoint.microsoft.com → CLIENT01 | Compliant or Not compliant with reasons |
| Config profile assigned | endpoint.microsoft.com → Configuration | `Win11 Baseline Config` assigned to device |

---

## Troubleshooting

**Device not appearing in Intune after enrollment**
- Wait 10–15 minutes — initial sync can be slow
- On CLIENT01, force a sync: **Settings** → **Accounts** → **Access work or school** → click the account → **Info** → **Sync**
- Or via PowerShell: `Start-Process "C:\Windows\System32\deviceenroller.exe" -ArgumentList "/c /AutoEnrollMDM"`

**Compliance shows "Not evaluated"**
- The policy may not be assigned to the device or user group. Check **endpoint.microsoft.com → Compliance → Policies → [policy name] → Device status**

**BitLocker marked non-compliant on VM**
- Expected — VMs without a TPM chip cannot enable BitLocker. In the lab, note this as a known exception. In production, use a hardware TPM or enable virtual TPM in Hyper-V/VMware.

**MFA Conditional Access locked out all users**
- This is why you exclude a break-glass account. Sign in with the break-glass admin → Entra → Conditional Access → disable the policy temporarily.

**User can't sign in after MFA is enabled**
- They need to complete MFA registration first. Direct them to [aka.ms/mfasetup](https://aka.ms/mfasetup) while on a trusted network, before the Conditional Access policy enforces it.

---

---

