## Ticket 1: New hire (Sam Rivera) — DONE

**Ticket:** Make an account for new hire Sam Rivera in a new Sales Organizational Unit, with a Sales group. He has to change his password the first time he signs in.

**What I did on DC01 (PowerShell):**

1. Made the Organizational Unit, which is a folder in Active Directory: `New-ADOrganizationalUnit -Name "Sales" -Path "DC=homelab,DC=local"`
2. Made the group: `New-ADGroup -Name "Sales-Team" -GroupScope Global -Path "OU=Sales,DC=homelab,DC=local"`
3. Made Sam's account: `New-ADUser -Name "Sam Rivera" -GivenName "Sam" -Surname "Rivera" -SamAccountName "srivera" -UserPrincipalName "srivera@homelab.local" -Path "OU=Sales,DC=homelab,DC=local" -AccountPassword (Read-Host -AsSecureString "Temporary password") -ChangePasswordAtLogon $true -Enabled $true`
4. Added Sam to the group: `Add-ADGroupMember -Identity "Sales-Team" -Members "srivera"`

**How I checked it:**

- `Get-ADOrganizationalUnit -Identity "OU=Sales,DC=homelab,DC=local"` showed Sales.
- `Get-ADGroup -Identity "Sales-Team"` showed a Global Security group.
- `Get-ADUser -Identity srivera` showed Enabled: True and Sales in the path.
- `Get-ADGroupMember -Identity "Sales-Team"` listed Sam Rivera.

**What went wrong:**

- I left out a required part, so PowerShell asked me for it, and I ended up making a group and an account in the wrong place. I found them with `Get-ADGroup` and `Get-ADUser`, deleted them with `Remove-ADGroup` and `Remove-ADUser`, and made them again correctly.
- I mixed up `-Name` and `-SamAccountName`. `-Name` is the full name and `-SamAccountName` is the sign-in name.
- I had spaces after dashes (`- Path`) and a missing dash. PowerShell rejects the whole command for that, so I read the error and fixed the line.

**What I learned:**

- When a command prints nothing, it usually worked, but I always check with a `Get-` command.
- Every `New-` command has a matching `Get-` command with the same noun.
- Brackets in the syntax mean optional. I never type the brackets or `<string>`.

(screenshots below)
<img width="447" height="384" alt="VirtualBoxVM_9v0agtoSku" src="https://github.com/user-attachments/assets/eaeaa0f0-d6d4-44b6-be3d-7eff43b85073" />
<img width="447" height="384" alt="VirtualBoxVM_ihadCajYd2" src="https://github.com/user-attachments/assets/f0b6361c-01fc-41bd-b7a2-56cef1edb24e" />


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

## Ticket 3: Shared Sales folder — DONE

**Ticket:** The sales team needs a shared folder. Only people in Sales-Team can add and change files. Everyone else is locked out.

**What I did on DC01 (PowerShell):**

1. Made the folder: `New-Item -Path "C:\Shares\Sales" -ItemType Directory`. I checked it with `Get-Item`.
2. Shared the folder with `New-SmbShare -Name "Sales" -Path "C:\Shares\Sales" -ChangeAccess "HOMELAB\Sales-Team"`. I checked it with `Get-SmbShare` and `Get-SmbShareAccess -Name "Sales"`. It showed Sales-Team with Change.
3. Set the folder permissions with `icacls "C:\Shares\Sales" /grant "HOMELAB\Sales-Team:(OI)(CI)M"`. I checked it with `icacls "C:\Shares\Sales"`. It showed Sales-Team with Modify.

**What I tested on the client:**

- Signed in as Sam (`HOMELAB\srivera`), opened `\\DC01\Sales`, and created a text file. It worked.
- Signed in as jit, opened `\\DC01\Sales`, and got "You do not have permission to access \\DC01\Sales." That's what I wanted.

**What I learned:**

- There are two locks. The share is the front door, and the folder permissions (NTFS) are the lock on the cabinet inside. A person needs both to get in. The share is what stopped jit.
- `Get-Command *Share*` helped me find `New-SmbShare` and tell it apart from `New-FileShare` and `New-NfsShare`. SMB is the one for Windows.
- Access levels: Read is look only, Change is add, edit, and delete, Full is everything.

**What went wrong:**

- I put `-WhatIf` on the share command to practice, but it actually created the share. When I ran it again I got "The name has already been shared". Some commands ignore `-WhatIf`, so I always check with a `Get-` command.
- When I tried to open the share on the client, I typed `\\DC01\Sales` in the search box instead of the address bar. I used the Windows key + R instead.

<img width="512" height="384" alt="VirtualBoxVM_tgpvlGK53Z" src="https://github.com/user-attachments/assets/cf8543ab-4a9c-4277-95c0-59a49b5465d9" /><img width="512" height="384" alt="VirtualBoxVM_TFXs9Qzjtl" src="https://github.com/user-attachments/assets/fb64e5d6-e268-4d8c-ac9f-b971338cda75" />
<img width="512" height="384" alt="VirtualBoxVM_N1rFoSCMZW" src="https://github.com/user-attachments/assets/4f6c8e6a-23fa-4525-b766-9fc156a13511" />
<img width="512" height="384" alt="VirtualBoxVM_QVJ6tjwbeO" src="https://github.com/user-attachments/assets/88faa15f-0811-4384-a1ac-a8c489a3be7e" />
<img width="512" height="384" alt="VirtualBoxVM_CPc8VCLcqY" src="https://github.com/user-attachments/assets/2ea1f6bc-6366-49ff-bee2-fcab6b333755" />
<img width="512" height="384" alt="VirtualBoxVM_3dC3x1Z4Og" src="https://github.com/user-attachments/assets/ee118776-c7d6-4f9f-937a-c2d95f677316" />

## Ticket 4: Leaver — (not started)
