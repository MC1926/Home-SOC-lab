<img width="512" height="384" alt="VirtualBoxVM_xL9qf8Sblx" src="https://github.com/user-attachments/assets/e6df6dfc-08b7-49a2-9726-de9e52663803" />
# Sysmon Deployment

**Status:** Complete

## What I Did

Installed Sysmon on the domain-joined client (DESKTOP-KP8DMJL) using the
SwiftOnSecurity community-standard configuration — the same config widely
referenced in real SOC environments for high-signal, low-noise event tracing.

## Steps

1. Downloaded Sysmon from Microsoft Sysinternals
2. Downloaded `sysmonconfig-export.xml` from SwiftOnSecurity's GitHub repo
3. Installed with: `Sysmon64.exe -i sysmonconfig-export.xml`
4. Applied the config: `Sysmon64.exe -c sysmonconfig-export.xml`
5. Verified via `Get-Service Sysmon64` (Running) and Event Viewer

## Result

Sysmon actively logging to Applications and Services Logs > Microsoft >
Windows > Sysmon > Operational. Over 7,600 events captured within minutes,
including process creation (Event ID 1) and registry modification
(Event ID 13) — visibility the default Windows Security log doesn't provide.

## Why This Matters

Default Windows auditing misses a lot attackers rely on: full process
command lines, registry changes, network connections tied to specific
processes. Sysmon closes that gap and is a tool real SOC analysts expect
candidates to at least understand.
