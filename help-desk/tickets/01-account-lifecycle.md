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

## Ticket 2: Password reset — DONE

**Ticket:** Sam Rivera forgot his password. Reset it, make him choose a new one at his next sign-in, and sign in as Sam on the client.

**What I did on DC01 (PowerShell):**

1. Searched for the right command with `Get-Command *Password*` and found `Set-ADAccountPassword`.
2. Read the syntax with `Get-Help Set-ADAccountPassword` and found the `-Reset` option, which sets a new password without needing the old one.
3. Reset the password: `Set-ADAccountPassword -Identity srivera -Reset`. PowerShell asked me to type the new password twice, hidden.
4. Made Sam change it at his next sign-in: `Set-ADUser -Identity srivera -ChangePasswordAtLogon $true`.
5. Checked it with `Get-ADUser -Identity srivera -Properties *`. It showed `PasswordExpired : True`.

**What I did on the client:**

1. Signed in as `HOMELAB\srivera` with the temporary password.
2. Windows said the password must be changed before signing in, and I set a new one.
3. Ran `whoami` and it showed `homelab\srivera`.

**What went wrong:**

- My first try was `Set-ADAccountPassword srivera` without `-Reset`. It asked for Sam's current password, which means it was a password change, not a reset. A technician can't ask for a password the user forgot, so I redid it with `-Reset`.
- My first `Set-ADUser` command had `-ChangePasswordAtLogon` with nothing after it, which gave a "missing an argument" error. The syntax showed `<bool>`, which means a yes or no value, so I added `$true`.

**What I learned:**

- A password change needs the old password. A reset does not.
- `<bool>` means true or false, written `$true` or `$false`.
- After a change, I check it with a `Get-` command.

<img width="512" height="384" alt="VirtualBoxVM_JTznk9QoRH" src="https://github.com/user-attachments/assets/8bb36ca8-21f3-4f69-b5c6-d7bccd05155c" />
<img width="512" height="384" alt="VirtualBoxVM_Eunof3sJyP" src="https://github.com/user-attachments/assets/f33a2e67-d807-4422-9266-4982883278f2" />
<img width="512" height="384" alt="VirtualBoxVM_xFsHRaKwUH" src="https://github.com/user-attachments/assets/3b474ffb-1dc9-4cc1-a525-dee264d98be4" />
<img width="512" height="384" alt="VirtualBoxVM_yjStclF5KT" src="https://github.com/user-attachments/assets/18562ad4-5d05-4ad5-b4d1-ff675824f0a3" />
<img width="512" height="384" alt="VirtualBoxVM_Vz5yOfivy6" src="https://github.com/user-attachments/assets/e1c9c7bc-663a-43cc-8082-e11884eb4689" />

## Ticket 3: Shared folder — (not started)
## Ticket 4: Leaver — (not started)
