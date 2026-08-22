# Sysmon Setup Notes

## Why Sysmon on top of native Windows auditing

Native Windows Security auditing can tell you a logon happened. It can't reliably tell you, in one event, what process launched what child process with what command line and what parent — which is exactly the data an analyst needs to tell "someone opened Notepad" apart from "something spawned `powershell.exe -enc <base64>` from an Office document." Sysmon fills that gap, which is why it sits on the victim machine alongside — not instead of — native Event Log collection.

## Installation

Installed Sysmon (Sysinternals) on the Windows 11 Home victim (`192.168.1.188`).

## Verification — local, before touching Splunk

Before wiring anything into Splunk, Sysmon was validated locally in Windows Event Viewer under `Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`:

- Confirmed **Event ID 1 (Process Creation)** was generating on ordinary activity.
- Launched `notepad.exe`, `powershell.exe`, and `hostname.exe` and confirmed a matching Event ID 1 entry appeared for each.

Isolating this step mattered: it separates "is Sysmon capturing the right data" from "is the forwarder shipping it correctly," which made later troubleshooting (see [Issue 2](../troubleshooting/issue-02-sysmon-permissions.md)) much faster to diagnose — the local Event Viewer check ruled Sysmon itself out as the problem.

## What Sysmon Event ID 1 captures

For each process creation event, Sysmon records:

- **Process Creation** — the new process and its image path
- **Parent Process** — what launched it
- **Command Line** — full command-line arguments, including obfuscated/encoded PowerShell
- **User Context** — which account the process ran as
- **Hashes** — file hashes of the executed image (useful for correlating against known-bad hashes later)

## Confirming end-to-end delivery into Splunk

Once the Universal Forwarder was configured (see [`splunk-setup.md`](./splunk-setup.md)), the same three test processes (`notepad.exe`, `hostname.exe`, `powershell.exe`) were run again and confirmed visible in Splunk within seconds via:

```spl
index=main EventCode=1
| table _time User Image ParentImage CommandLine
| sort -_time
```

This search is the basis for [Detection 1](../detections/detection-01-process-creation.md).
