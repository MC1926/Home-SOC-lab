# Home-SOC-lab
A self-built home lab (Windows Server 2022 DC + Windows 11 client, homelab.local domain) for practicing SOC analyst skills: log analysis, incident investigation and Sysmon telemetry. Built while studying for Network+ and Security+ to gain hands-on, documented experience.
## IT Support

Help desk tickets and runbooks from my home lab: user accounts, permissions, Group Policy, networking and Windows troubleshooting. See [help-desk/](help-desk/).

## Security Labs

| Lab | What I did | Key evidence |
|---|---|---|
| [02 Sysmon setup](02-sysmon-setup/) | Deployed Sysmon with the SwiftOnSecurity configuration | Sysmon Operational log |
| [03 Privilege escalation](03-privilege-escalation/) | Added a user to Domain Admins and found it in the log | Event 4728 |
| [04 PowerShell execution](04-powershell-execution/) | Matched a suspicious command across two logs | Sysmon Event 1 and Event 4104, same ProcessId |
| [05 Account lockout](05-account-lockout/) | Investigated a lockout and unlocked the account | Event 4740 |
