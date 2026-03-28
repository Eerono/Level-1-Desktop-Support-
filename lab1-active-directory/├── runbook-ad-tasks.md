# AD Helpdesk Runbook — L1 Tasks

A quick-reference guide for common Level 1 Active Directory support tasks. All commands run on the Domain Controller or any machine with RSAT installed, in **PowerShell as Administrator**.

---

## Table of Contents

- [Password Reset](#password-reset)
- [Unlock a Locked Account](#unlock-a-locked-account)
- [Find All Locked Accounts](#find-all-locked-accounts)
- [Move a User to a Different OU](#move-a-user-to-a-different-ou)
- [Disable / Enable a User Account](#disable--enable-a-user-account)
- [Check Group Membership](#check-group-membership)
- [GPO — Force Update and Verify](#gpo--force-update-and-verify)
- [Ticket Note Templates](#ticket-note-templates)
- [Escalation Checklist](#escalation-checklist)

---

## Password Reset

Use when a user has forgotten their password or their account was compromised.

```powershell
# Step 1 — Reset the password
Set-ADAccountPassword -Identity "jsmith" `
  -Reset `
  -NewPassword (ConvertTo-SecureString "NewP@ss123!" -AsPlainText -Force)

# Step 2 — Force the user to change it on next login (always do this)
Set-ADUser -Identity "jsmith" -ChangePasswordAtLogon $true

# Step 3 — Confirm account is enabled and not still locked
Get-ADUser -Identity "jsmith" -Properties LockedOut, Enabled, PasswordLastSet |
  Select Name, Enabled, LockedOut, PasswordLastSet
```

> ⚠️ Never leave a temporary password permanently set. Always pair `Set-ADAccountPassword` with `-ChangePasswordAtLogon $true`.

**Ticket note:**
```
Reset password for jsmith per user request. Temporary password issued.
Forced change on next login. User advised to update password immediately.
```

---

## Unlock a Locked Account

Use when a user is locked out after too many failed login attempts.

```powershell
# Step 1 — Confirm the account is actually locked
Get-ADUser -Identity "jsmith" -Properties LockedOut, BadLogonCount, LastBadPasswordAttempt |
  Select Name, LockedOut, BadLogonCount, LastBadPasswordAttempt

# Step 2 — Unlock the account
Unlock-ADAccount -Identity "jsmith"

# Step 3 — Verify it's unlocked
Get-ADUser -Identity "jsmith" -Properties LockedOut | Select Name, LockedOut
```

> 💡 If the same user is locking out repeatedly, check for a saved password on a mobile device or mapped drive. Do not keep unlocking without investigating the root cause.

**Ticket note:**
```
Account unlocked for jsmith. BadLogonCount reviewed. User advised to check
saved credentials on mobile/shared drives if issue recurs.
```

---

## Find All Locked Accounts

Use for a sweep during morning checks or when multiple users report access issues.

```powershell
# List all locked accounts with lockout timestamp
Search-ADAccount -LockedOut |
  Select Name, SamAccountName, LockedOut, LastLogonDate |
  Sort-Object Name

# Export to CSV for ticket logging or manager review
Search-ADAccount -LockedOut |
  Select Name, SamAccountName, LockedOut, LastLogonDate |
  Export-Csv -Path ".\locked-users-$(Get-Date -Format 'yyyy-MM-dd').csv" -NoTypeInformation
```

> 💡 Dating the CSV filename (`yyyy-MM-dd`) keeps exports organised if you run this regularly.

**Ticket note:**
```
Lockout sweep performed. X accounts found locked. See attached CSV.
Accounts unlocked after manager approval. Root cause under investigation.
```

---

## Move a User to a Different OU

Use when a user changes department or role and needs to inherit a different set of GPOs.

```powershell
# Step 1 — Confirm the user's current location
Get-ADUser -Identity "jsmith" -Properties DistinguishedName |
  Select Name, DistinguishedName

# Step 2 — Move to the target OU
Move-ADObject `
  -Identity "CN=jsmith,OU=IT_Department,DC=lab,DC=local" `
  -TargetPath "OU=HR,DC=lab,DC=local"

# Step 3 — Confirm the move
Get-ADUser -Identity "jsmith" -Properties DistinguishedName |
  Select Name, DistinguishedName
```

> ⚠️ Moving a user to a new OU changes which GPOs apply to them. Confirm with the manager that the target OU has the correct policies before moving.

**Ticket note:**
```
Moved jsmith from OU=IT_Department to OU=HR per ticket #XXXX and
manager approval from [Name]. GPO inheritance change confirmed.
```

---

## Disable / Enable a User Account

Use when an employee is offboarded, on extended leave, or being re-onboarded.

```powershell
# Disable an account (offboarding / suspension)
Disable-ADAccount -Identity "jsmith"

# Move disabled accounts to a dedicated OU to keep AD tidy
Move-ADObject `
  -Identity "CN=jsmith,OU=HR,DC=lab,DC=local" `
  -TargetPath "OU=Disabled_Users,DC=lab,DC=local"

# Re-enable an account (returning employee)
Enable-ADAccount -Identity "jsmith"

# Confirm current state
Get-ADUser -Identity "jsmith" -Properties Enabled | Select Name, Enabled
```

**Ticket note (disable):**
```
Account disabled for jsmith per offboarding request from [Manager].
Account moved to OU=Disabled_Users. Mailbox delegation pending IT review.
```

---

## Check Group Membership

Use when troubleshooting access issues — a user missing a group is the most common cause of permission problems.

```powershell
# List all groups a user belongs to
Get-ADPrincipalGroupMembership -Identity "jsmith" |
  Select Name, GroupCategory, GroupScope |
  Sort-Object Name

# Add a user to a group
Add-ADGroupMember -Identity "Finance_ReadOnly" -Members "jsmith"

# Remove a user from a group
Remove-ADGroupMember -Identity "Finance_ReadOnly" -Members "jsmith" -Confirm:$false

# List all members of a specific group
Get-ADGroupMember -Identity "Finance_ReadOnly" | Select Name, SamAccountName
```

**Ticket note:**
```
Added jsmith to Finance_ReadOnly group per access request #XXXX.
Approved by [Manager]. Access verified by user.
```

---

## GPO — Force Update and Verify

Use after making GPO changes, or when a user reports that a policy isn't applying correctly.

```powershell
# On the client machine — force an immediate GPO refresh
gpupdate /force

# Generate a detailed policy report for the current user and computer
gpresult /r

# Generate a full HTML report (easier to read for complex policies)
gpresult /h "C:\Temp\gpo-report.html" /f
Start-Process "C:\Temp\gpo-report.html"
```

**On DC01 — check which GPOs are linked and their status:**

```powershell
# List all GPOs in the domain
Get-GPO -All | Select DisplayName, GpoStatus, CreationTime | Sort-Object DisplayName

# Check which GPOs are linked to a specific OU
Get-GPInheritance -Target "OU=IT_Department,DC=lab,DC=local"
```

> 💡 `rsop.msc` (Resultant Set of Policy) is a GUI alternative to `gpresult /r` — run it on the client to see the effective policy applied to that machine and user.

**Ticket note:**
```
GPO refresh forced on [hostname] after policy change. Verified "Security Baseline"
applying correctly via gpresult /r. Issue resolved.
```

---

## Ticket Note Templates

Copy-paste templates for common scenarios. Replace bracketed fields before submitting.

| Scenario | Template |
|----------|----------|
| Password reset | `Reset password for [username] per user request. Forced change on next login. Ticket #[XXXX].` |
| Account unlock | `Unlocked account for [username]. BadLogonCount: [N]. User advised to check saved credentials.` |
| Account disabled | `Disabled account for [username] per [manager] request. Reason: [offboarding/suspension]. Ticket #[XXXX].` |
| OU move | `Moved [username] from [source OU] to [target OU] per manager approval from [name]. Ticket #[XXXX].` |
| Group access added | `Added [username] to [group] per access request approved by [manager]. Ticket #[XXXX].` |
| GPO issue | `GPO refresh forced on [hostname]. Policy [GPO name] confirmed applying via gpresult. Ticket #[XXXX].` |

---

## Escalation Checklist

Escalate to L2/L3 if any of the following apply:

- [ ] User is locking out repeatedly (3+ times in one day) — possible credential loop or compromised account
- [ ] `Unlock-ADAccount` succeeds but the account locks again within minutes — investigate via Event ID 4740 in Event Viewer
- [ ] GPO is linked but `gpresult /r` shows it is not applying — may be a WMI filter, security filtering, or replication issue
- [ ] AD replication errors — run `repadmin /showrepl` and escalate if failures are present
- [ ] Password reset fails with an "Access Denied" error — permissions issue on the OU, escalate to AD admin
- [ ] User reports access issues after an OU move — GPO inheritance change may require L2 review

```powershell
# Useful commands to gather info before escalating
Get-EventLog -LogName Security -InstanceId 4740 -Newest 10  # Lockout events
repadmin /showrepl                                           # Replication health
netdom query fsmo                                            # FSMO role holders
```
