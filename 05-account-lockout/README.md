# Lab 05: Account Lockout Investigation

**Type:** Simulated help desk ticket
**Environment:** Windows Server domain `homelab.local` (DC01 domain controller, CLIENT01 domain client)
**Tools:** `net accounts`, PowerShell `Get-WinEvent`, Active Directory PowerShell module
**Key event:** Security Event ID 4740 (account locked out)

## 1. Ticket report

> "I'm locked out of my account and can't sign in."
> User: `jit` (John IT)

## 2. Setup (simulated)

To reproduce the problem, I set the domain lockout threshold to 3 failed attempts (`net accounts /lockoutthreshold:3`), then entered a wrong password for `jit` three times on CLIENT01.

<img width="502" height="375" alt="VirtualBoxVM_RkegCdIlQg" src="https://github.com/user-attachments/assets/d73b91b0-0482-4ab6-85b3-3c3120fbffca" />

Threshold is 3. Lockout duration and observation window are both 30 minutes.

## 3. Symptom confirmed

![Uploading VirtualBoxVM_NR9sXgQtkW.png…]()

Windows shows: "The referenced account is currently locked out and may not be logged on to."

## 4. Investigation

I queried the Security log on the domain controller for the latest Event ID 4740:

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4740} -MaxEvents 1 | Format-List TimeCreated, Message
```

![Uploading VirtualBoxVM_mynCz8XezX.png…]()

| Field | Value | Meaning |
|---|---|---|
| TimeCreated | 10/8/2026 9:15:10 AM | When the lockout happened |
| Account Name (locked out) | jit | The affected user |
| Caller Computer Name | DESKTOP-KP8DMJL | The computer the failed attempts came from |
| Subject | DC01$ | The domain controller recorded the event |

## 5. Root cause

Three consecutive failed password attempts for `jit` from the client computer reached the lockout threshold of 3, so Active Directory locked the account. In a real environment, the Caller Computer Name tells you which machine to check next (a stale saved password, a mapped drive or a phone with old credentials are common causes).

## 6. Resolution

```powershell
Unlock-ADAccount -Identity jit
Get-ADUser jit -Properties LockedOut | Select Name, LockedOut
```

![Uploading explorer_R8uyDSq3GX.png…]()

`LockedOut` returned `False`. The account was unlocked and verified.

## 7. What I learned

- Event 4740 shows who was locked out and which computer caused it.
- The "Subject" field is the domain controller, not the person who caused the lockout.
- A lockout threshold limits password guessing, and the same log trail helps tell a forgetful user from an attack.

## Next steps

Pair this with the failed-logon events (4625) from Lab 01 to see the individual bad attempts behind a lockout.
