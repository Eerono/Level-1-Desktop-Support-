# M365 & Intune Helpdesk Runbook — L1 Tasks

A quick-reference guide for common Level 1 Microsoft 365 and Intune support tasks. All commands use the **Microsoft Graph PowerShell SDK** — install it once with `Install-Module Microsoft.Graph -Scope CurrentUser -Force`.

---

## Table of Contents

- [Reset a User's Password](#reset-a-users-password)
- [Reset MFA / Re-register Authenticator](#reset-mfa--re-register-authenticator)
- [Assign or Remove a Licence](#assign-or-remove-a-licence)
- [Disable and Offboard a User](#disable-and-offboard-a-user)
- [Force a Device Sync with Intune](#force-a-device-sync-with-intune)
- [Remotely Wipe or Retire a Device](#remotely-wipe-or-retire-a-device)
- [Check Device Compliance Status](#check-device-compliance-status)
- [Ticket Note Templates](#ticket-note-templates)
- [Escalation Checklist](#escalation-checklist)

---

## Reset a User's Password

Use when a user is locked out of their M365 account or has forgotten their password.

### Via Admin Centre (GUI)

1. [admin.microsoft.com](https://admin.microsoft.com) → **Users** → **Active users**
2. Click the user → **Reset password**
3. Auto-generate or set a temporary password
4. Check **Require this user to change their password when they first sign in**
5. Deliver the temporary password via a secure channel — never email or Teams in plaintext

### Via PowerShell

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Reset password and force change on next login
$passwordProfile = @{
  Password                      = "TempP@ss2024!"
  ForceChangePasswordNextSignIn = $true
}

Update-MgUser -UserId "jane.doe@labintune.onmicrosoft.com" `
  -PasswordProfile $passwordProfile

# Confirm
Get-MgUser -UserId "jane.doe@labintune.onmicrosoft.com" |
  Select DisplayName, UserPrincipalName
```

> ⚠️ Always pair a password reset with `ForceChangePasswordNextSignIn = $true`. Never leave a temporary password permanently set.

**Ticket note:**
```
Password reset for jane.doe@labintune.onmicrosoft.com per user request.
Temporary password issued via [secure channel]. Forced change on next login.
User confirmed successful login. Ticket #[XXXX].
```

---

## Reset MFA / Re-register Authenticator

Use when a user gets a new phone, loses access to their Authenticator app, or is stuck in an MFA loop.

### Via Admin Centre

1. [admin.microsoft.com](https://admin.microsoft.com) → **Users** → **Active users** → select user
2. **Authentication methods** tab → **Require re-register multifactor authentication**
3. Optionally delete existing methods listed under Authentication methods

### Via PowerShell

```powershell
Connect-MgGraph -Scopes "UserAuthenticationMethod.ReadWrite.All"

# List current authentication methods for the user
Get-MgUserAuthenticationMethod -UserId "jane.doe@labintune.onmicrosoft.com"

# Delete a specific method (replace {methodId} with the actual ID from above)
Remove-MgUserAuthenticationMethod `
  -UserId "jane.doe@labintune.onmicrosoft.com" `
  -AuthenticationMethodId "{methodId}"
```

After resetting, direct the user to [aka.ms/mfasetup](https://aka.ms/mfasetup) to re-register their authenticator app.

**Ticket note:**
```
MFA reset for jane.doe@labintune.onmicrosoft.com — user reported new phone.
Authentication methods cleared. User directed to aka.ms/mfasetup to re-register.
Re-registration confirmed by user. Ticket #[XXXX].
```

---

## Assign or Remove a Licence

Use during onboarding (assign) or offboarding (remove), or when a user needs access to an additional M365 service.

### Via Admin Centre

1. [admin.microsoft.com](https://admin.microsoft.com) → **Users** → **Active users** → select user
2. **Licences and apps** tab → check/uncheck the required licence → **Save changes**

### Via PowerShell

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All"

# Find the SKU ID for the licence you want to assign
Get-MgSubscribedSku | Select SkuPartNumber, SkuId

# Assign a licence (replace <SkuId-here> with the value from above)
Set-MgUserLicense -UserId "jane.doe@labintune.onmicrosoft.com" `
  -AddLicenses @(@{ SkuId = "<SkuId-here>" }) `
  -RemoveLicenses @()

# Remove a licence
Set-MgUserLicense -UserId "jane.doe@labintune.onmicrosoft.com" `
  -AddLicenses @() `
  -RemoveLicenses @("<SkuId-here>")

# Confirm current licence assignment
Get-MgUserLicenseDetail -UserId "jane.doe@labintune.onmicrosoft.com" |
  Select SkuPartNumber
```

**Ticket note:**
```
M365 E5 licence assigned to jane.doe@labintune.onmicrosoft.com per onboarding
request from [Manager]. Access confirmed. Ticket #[XXXX].
```

---

## Disable and Offboard a User

Use when an employee leaves or is suspended. Follow this order — blocking sign-in first prevents any access gap.

```powershell
Connect-MgGraph -Scopes "User.ReadWrite.All", "Directory.ReadWrite.All"

$upn = "jane.doe@labintune.onmicrosoft.com"

# Step 1 — Block sign-in immediately
Update-MgUser -UserId $upn -AccountEnabled:$false

# Step 2 — Revoke all active sessions and tokens
Revoke-MgUserSignInSession -UserId $upn

# Step 3 — Remove all licences to free up seats
$skus = (Get-MgUserLicenseDetail -UserId $upn).SkuId
Set-MgUserLicense -UserId $upn `
  -AddLicenses @() `
  -RemoveLicenses $skus

# Step 4 — Confirm account is disabled
Get-MgUser -UserId $upn | Select DisplayName, AccountEnabled
```

**Post-offboarding checklist — flag these in the ticket for L2/manager action:**

- [ ] Mailbox delegation or auto-forwarding configured
- [ ] OneDrive access transferred to manager
- [ ] Shared mailboxes and distribution lists reviewed
- [ ] Device wiped via Intune (see below)
- [ ] Physical access cards/keys returned

**Ticket note:**
```
Offboarding completed for jane.doe@labintune.onmicrosoft.com.
Sign-in blocked, sessions revoked, licences removed per [Manager] request.
Device wipe initiated via Intune. Mailbox delegation pending manager review.
Ticket #[XXXX].
```

---

## Force a Device Sync with Intune

Use after pushing a new policy or configuration profile and wanting it to apply immediately rather than waiting for the next scheduled check-in (default is every 8 hours).

### On the device

```powershell
# Method 1 — via Settings UI
# Settings → Accounts → Access work or school → [account] → Info → Sync

# Method 2 — via PowerShell (run as Administrator)
Start-Process "C:\Windows\System32\deviceenroller.exe" -ArgumentList "/c /AutoEnrollMDM"

# Method 3 — trigger the MDM sync scheduled task
Get-ScheduledTask | Where-Object { $_.TaskName -like "*Enterprise*" } |
  Start-ScheduledTask
```

### From the Intune portal

1. [endpoint.microsoft.com](https://endpoint.microsoft.com) → **Devices** → **All devices**
2. Click the device → **Sync** (top action bar) → confirm the prompt
3. The device will check in within a few minutes

**Ticket note:**
```
Manual Intune sync triggered for [device name] following policy update.
Sync confirmed in portal. Policy status updated to Compliant. Ticket #[XXXX].
```

---

## Remotely Wipe or Retire a Device

Use during offboarding (retire) or when a device is lost or stolen (wipe).

> ⚠️ **Wipe is irreversible.** All data is deleted and the OS resets to factory. Always confirm with a manager before initiating. **Retire** is the safer option for BYOD — it removes only company data.

| Action | What it does | When to use |
|--------|-------------|-------------|
| **Retire** | Removes company apps, profiles, and data. Personal data untouched. | Offboarding, BYOD |
| **Wipe** | Full factory reset. All data deleted. | Lost/stolen device, corporate-owned |
| **Delete** | Removes the device record from Intune only. | Already wiped or decommissioned |

### Via Intune portal

1. [endpoint.microsoft.com](https://endpoint.microsoft.com) → **Devices** → **All devices**
2. Click the device → **Wipe** or **Retire** from the top action bar
3. Confirm the action — note the timestamp for your ticket

### Via PowerShell

```powershell
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.PrivilegedOperations.All"

# Find the managed device ID
$device = Get-MgDeviceManagementManagedDevice -Filter "deviceName eq 'CLIENT01'"
$deviceId = $device.Id

# Retire — removes company data only
Invoke-MgRetireDeviceManagementManagedDevice -ManagedDeviceId $deviceId

# Wipe — full factory reset (destructive, use with caution)
Invoke-MgCleanDeviceManagementManagedDeviceWindowsDevice -ManagedDeviceId $deviceId
```

**Ticket note (lost device):**
```
Remote wipe initiated for [device name] (serial: [XXXX]) — reported lost
by [username] on [date]. Wipe confirmed in Intune portal at [time].
Device record retained pending insurance/police report. Manager [name] notified.
Ticket #[XXXX].
```

---

## Check Device Compliance Status

Use when a user reports being blocked from M365 resources, or during routine audits.

### Via Intune portal

1. [endpoint.microsoft.com](https://endpoint.microsoft.com) → **Devices** → **All devices**
2. Use the **Compliance** column for a quick overview
3. Click a device → **Device compliance** → review each policy and its pass/fail status

### Via PowerShell

```powershell
Connect-MgGraph -Scopes "DeviceManagementManagedDevices.Read.All"

# Get compliance state for all managed devices
Get-MgDeviceManagementManagedDevice |
  Select DeviceName, ComplianceState, LastSyncDateTime, OperatingSystem |
  Sort-Object ComplianceState

# Export non-compliant devices to CSV for audit or manager review
Get-MgDeviceManagementManagedDevice -Filter "complianceState eq 'noncompliant'" |
  Select DeviceName, UserPrincipalName, LastSyncDateTime |
  Export-Csv ".\non-compliant-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

**Ticket note:**
```
Compliance check for [device name]. Status: [Compliant/Non-compliant].
Non-compliant settings: [list settings]. User advised to [remediation steps].
Follow-up scheduled for [date]. Ticket #[XXXX].
```

---

## Ticket Note Templates

| Scenario | Template |
|----------|----------|
| Password reset | `Password reset for [UPN] per user request. Temp password via [channel]. Forced change on next login. Ticket #[XXXX].` |
| MFA reset | `MFA methods cleared for [UPN] — reason: [new phone/lost access]. User directed to aka.ms/mfasetup. Re-registration confirmed. Ticket #[XXXX].` |
| Licence assigned | `[Licence name] assigned to [UPN] per request from [manager]. Access confirmed. Ticket #[XXXX].` |
| Licence removed | `Licence removed from [UPN] — reason: [offboarding/role change]. Seat freed. Ticket #[XXXX].` |
| Account disabled | `Account disabled for [UPN] per offboarding request from [manager]. Sign-in blocked, sessions revoked, licences removed. Ticket #[XXXX].` |
| Device wipe | `Remote wipe for [device] — reason: [lost/stolen/offboarding]. Confirmed in Intune at [time]. Ticket #[XXXX].` |
| Non-compliant device | `Compliance review for [device]. Non-compliant: [settings]. User advised: [steps]. Follow-up: [date]. Ticket #[XXXX].` |
| Device sync | `Manual sync triggered for [device] after policy update. Status updated. Ticket #[XXXX].` |

---

## Escalation Checklist

Escalate to L2/L3 if any of the following apply:

- [ ] Conditional Access is blocking all users — pull the **Sign-in logs** from Entra before touching any policy
- [ ] MFA reset completed but user still cannot sign in — may be a token cache issue; escalate with sign-in log export
- [ ] Device stuck in **"Pending"** compliance state for more than 30 minutes — possible MDM enrollment corruption
- [ ] Wipe or retire action does not reach the device within 24 hours — device may be offline; document the attempt with timestamp
- [ ] User reports being blocked despite the device showing Compliant — Conditional Access evaluation issue; pull sign-in logs
- [ ] Licence assignment fails with "no licences available" — L2 to review licence pool with billing

```powershell
# Useful diagnostics to gather before escalating

# Export recent sign-in logs for a specific user
Connect-MgGraph -Scopes "AuditLog.Read.All"
Get-MgAuditLogSignIn `
  -Filter "userPrincipalName eq 'jane.doe@labintune.onmicrosoft.com'" `
  -Top 20 |
  Select CreatedDateTime, AppDisplayName, ConditionalAccessStatus, Status |
  Format-Table -AutoSize

# Check remaining licence availability
Get-MgSubscribedSku |
  Select SkuPartNumber,
    @{N="Assigned"; E={$_.ConsumedUnits}},
    @{N="Total";    E={$_.PrepaidUnits.Enabled}} |
  Format-Table -AutoSize

# Get full device details for the escalation ticket
Get-MgDeviceManagementManagedDevice -Filter "deviceName eq 'CLIENT01'" |
  Select DeviceName, SerialNumber, EnrolledDateTime, LastSyncDateTime,
         ComplianceState, OperatingSystem, OsVersion, UserPrincipalName
```
