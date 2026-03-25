
# Lab 2: Microsoft 365 & Intune MDM

## Overview
M365 E5 Developer tenant (free). Configured users, MFA, Intune policies, device enrollment.

**Resources:**
- [M365 Dev Program](https://developer.microsoft.com/microsoft-365/dev-program) → 90-day E5 sandbox.

## Setup Instructions (45 mins)

### 1. Create Tenant
developer.microsoft.com → Join → New Tenant: labintune.onmicrosoft.com
→ Download E5 subscription → 25 users unlocked.



### 2. Add Users & MFA
admin.microsoft.com → Users → New User: jane.doe@labintune.onmicrosoft.com
→ Security → Authentication Methods → Enable MFA



### 3. Intune Compliance Policy
endpoint.microsoft.com → Devices → Compliance → Policies → Create

Require: BitLocker, Win11+, Defender AV

Non-compliant: Block access after 1 day



### 4. Enroll Win11 Device
CLIENT01 → Settings → Accounts → Access Work → Enroll only in device management
→ Verify: endpoint.microsoft.com → Devices → Compliant ✓


