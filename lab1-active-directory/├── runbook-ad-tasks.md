Lab 1 Runbook - Copy this
text
# AD Helpdesk Runbook — L1 Tasks

## Password Reset
Unlock-ADAccount jsmith
Set-ADAccountPassword jsmith -Reset -NewPassword (ConvertTo-SecureString "NewP@ss123" -AsPlainText -Force)

text
**Ticket Note:** "Reset password, forced change on next login."

## Find Locked Accounts
Search-ADAccount -LockedOut | Select Name, LockedOutOn | Export-Csv locked-users.csv

text

## Move User to New OU
Move-ADObject -Identity "CN=jsmith,OU=IT,DC=lab,DC=local" -TargetPath "OU=HR,DC=lab,DC=local"

text

## Common GPO: Password Policy
gpupdate /force # On client
rsop.msc # Verify applied

text

