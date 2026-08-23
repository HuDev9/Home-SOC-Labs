# Attack Simulation 1 — Process Creation Validation

## Goal

Confirm, with real activity, that the Sysmon → Universal Forwarder → Splunk pipeline captures process creation end to end — before relying on it for anything more complex.

## Method

On the Windows 11 victim (`192.168.x.x`), manually launched three processes chosen to represent distinct categories:

```text
notepad.exe      — benign GUI application
hostname.exe     — benign command-line utility
powershell.exe   — scripting engine, the process most later detections care about
```

## Expected telemetry

Each launch should produce one **Sysmon Event ID 1 (Process Creation)** event locally, which the Universal Forwarder should then ship to the indexer within seconds.

## Observed result

All three processes appeared in Splunk as expected, queried with:

```spl
index=main EventCode=1
| table _time User Image ParentImage CommandLine
| sort -_time
```

Each event showed the correct `Image` path, the interactive user session as `ParentImage`'s origin, and the correct `User` context — confirming the full pipeline (Sysmon capture → forwarder collection → TCP 9997 delivery → `index=main` landing → SPL retrieval) was working correctly before any detection logic was trusted.

## Why this simulation exists

This is the "hello world" of the lab: before writing detections that depend on process-creation telemetry being reliable, I needed direct proof that ordinary activity shows up correctly and completely. Everything in [Detection 1](../detections/detection-01-process-creation.md) and [Detection 2](../detections/detection-02-powershell-monitoring.md) depends on this baseline holding true.

## Investigation

Worked as [Case 001 — Sysmon Process Creation](../investigations/case-001-process-creation.md) and [Case 002 — PowerShell Activity](../investigations/case-002-powershell-activity.md).
