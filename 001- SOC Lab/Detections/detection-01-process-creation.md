# Detection 1 — Process Creation Monitoring

## Purpose

Baseline visibility into every process launched on the victim endpoint. This is the foundation detection everything else in this lab builds on — PowerShell monitoring (Detection 2) is a filtered view of this same data source.

## Data source

Sysmon **Event ID 1 (Process Creation)** on the Windows 11 victim, forwarded via the Splunk Universal Forwarder into `index=main`.

## SPL

```spl
index=main EventCode=1
| table _time User Image ParentImage CommandLine
| sort -_time
```

## Field notes

| Field | Meaning |
|---|---|
| `_time` | When the process was created |
| `User` | Account context the process ran under |
| `Image` | Full path of the executable that launched |
| `ParentImage` | Full path of the process that launched it — critical for spotting abnormal parent-child relationships (e.g. `winword.exe` spawning `powershell.exe`) |
| `CommandLine` | Full command-line arguments — where obfuscation, encoded payloads, and suspicious flags show up |

## Tuning / false positives

At this stage the search is intentionally broad — it's a hunting/triage view, not a fire-an-alert detection. In a larger environment this would need:
- Exclusion of known-noisy, known-good parent-child pairs (e.g. `explorer.exe → *`)
- A narrower alerting version scoped to specific `ParentImage`/`Image` combinations rather than all Event ID 1 traffic

## Validated against

[Attack Simulation 1 — Process Creation Validation](../attack-simulations/attack-01-process-creation-validation.md), investigated in [Case 001](../investigations/case-001-process-creation.md).
