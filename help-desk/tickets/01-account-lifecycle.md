# Session 1: Account Lifecycle

**Environment:** Windows Server domain `homelab.local`, domain controller DC01, one Windows client.

## Ticket 1: New hire (Sam Rivera) — DONE

**Request:** Create an account for new hire Sam Rivera in a new Sales Organizational Unit, with a Sales group. He must change his password at first sign-in.

**What I did (on DC01, in PowerShell):**

1. Created the Organizational Unit:
   `New-ADOrganizationalUnit -Name "Sales" -Path "DC=homelab,DC=local"`
2. Created the group:
   `New-ADGroup -Name "Sales-Team" -GroupScope Global -Path "OU=Sales,DC=homelab,DC=local"`
3. Created the user (full name, sign-in name `srivera`, enabled, must change password at first sign-in):
   `New-ADUser -Name "Sam Rivera" -GivenName "Sam" -Surname "Rivera" -SamAccountName "srivera" -UserPrincipalName "srivera@homelab.local" -Path "OU=Sales,DC=homelab,DC=local" -AccountPassword (Read-Host -AsSecureString "Temporary password") -ChangePasswordAtLogon $true -Enabled $true`
4. Added Sam to the group:
   `Add-ADGroupMember -Identity "Sales-Team" -Members "srivera"`

**Verification:**
- `Get-ADOrganizationalUnit -Filter *` shows Sales.
- `Get-ADUser -Identity srivera` shows Enabled: True, in OU=Sales.
- `Get-ADGroupMember -Identity "Sales-Team"` lists Sam Rivera.

<img width="447" height="384" alt="VirtualBoxVM_9v0agtoSku" src="https://github.com/user-attachments/assets/eaeaa0f0-d6d4-44b6-be3d-7eff43b85073" />
<img width="447" height="384" alt="VirtualBoxVM_ihadCajYd2" src="https://github.com/user-attachments/assets/f0b6361c-01fc-41bd-b7a2-56cef1edb24e" />


## What went wrong and how I fixed it

- **Created a stray group and user by accident.** A command with a missing value made PowerShell prompt for the required parameter, and it created objects in the wrong place. I found them with `Get-ADGroup` and `Get-ADUser`, removed them with `Remove-ADGroup` and `Remove-ADUser`, and recreated them correctly.
- **Swapped `-Name` and `-SamAccountName`.** `-Name` is the full display name. `-SamAccountName` is the sign-in name.
- **Typos with dashes.** A space after a dash, or a missing dash, makes PowerShell reject the whole command. I fixed them by reading the error and editing the line.
- **Lesson:** a command that prints nothing succeeded or silently did the wrong thing. Always verify with a `Get-` command.

## Ticket 2: Password reset — (in progress)
## Ticket 3: Shared folder — (not started)
## Ticket 4: Leaver — (not started)
