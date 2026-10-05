# Ticket 02: Unauthorized Privilege Escalation (Domain Admins)

## Summary
A user account (`jit`, "John IT") was added to the Domain Admins group on
DC01 by the account `HOMELAB\MC`. This was a simulated escalation to practice
detecting and investigating high-severity group membership changes.

## What I Observed
DC01 is Server Core (no GUI), so I queried the Security log with PowerShell:

    Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4728} -MaxEvents 1

Event ID 4728 ("A member was added to a security-enabled global group"):

| Field | Value |
|---|---|
| Time | 10/5/2026 7:07:13 AM |
| Changed by (Subject) | HOMELAB\MC (Logon ID 0x5DFE1) |
| Member added | CN=John IT,OU=IT,DC=homelab,DC=local (`jit`) |
| Group | Domain Admins (SID ending -512) |

I confirmed the change with `Get-ADGroupMember -Identity "Domain Admins"`,
which listed `jit` alongside the built-in Administrator account.

## Why This Matters
Domain Admins has full control of every system in the domain. A new member in
this group is one of the highest-severity changes a SOC watches for, because an
attacker with these rights can create accounts, disable security tooling, and
reach any machine. In a real environment this alert should trigger immediate
investigation, not a routine ticket.

## Next Steps
1. Contact the account owner (`MC`) and the requester to confirm the change was
   approved and ticketed.
2. If not approved, remove `jit` from Domain Admins immediately and reset
   credentials for both `jit` and `MC`.
3. Use the Logon ID (0x5DFE1) to find the originating logon (Event ID 4624) and
   see which machine and logon type `MC` used.
4. Review other changes made in that session for signs of wider compromise.

## Cleanup
Removed `jit` from Domain Admins after the exercise and verified with
`Get-ADGroupMember`.<img width="502" height="375" alt="VirtualBoxVM_Z4lz20T8oC" src="https://github.com/user-attachments/assets/6a989703-4afc-4f1f-bd52-1a9bf31d0a1f" />
