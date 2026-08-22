# Case 001 — Sysmon Process Creation

**Status:** Closed — expected activity, pipeline validated
**Host:** Windows 11 Home victim (`192.168.1.188`)
**Related simulation:** [Attack Simulation 1](../attack-simulations/attack-01-process-creation-validation.md)
**Related detection:** [Detection 1 — Process Creation Monitoring](../detections/detection-01-process-creation.md)

## Summary

Baseline validation case confirming that process-creation telemetry flows correctly from the Windows victim, through Sysmon and the Universal Forwarder, into Splunk — using three manually launched processes of known, benign intent.

## Timeline

| Time (relative) | Event |
|---|---|
| T+0s | `notepad.exe` launched interactively on victim |
| T+~30s | `hostname.exe` launched interactively on victim |
| T+~60s | `powershell.exe` launched interactively on victim |
| T+~65s | All three events confirmed visible in Splunk |

## Query used

```spl
index=main EventCode=1
| table _time User Image ParentImage CommandLine
| sort -_time
```

## Findings

- All three processes appeared with correct `Image` paths.
- `ParentImage` for each traced back to the interactive user session (`explorer.exe`), consistent with manual, interactive launches — not automated or remote execution.
- `User` field correctly reflected the logged-in account.
- Latency from launch to visibility in Splunk was on the order of seconds, well within what's needed for real-time triage.

## Disposition

**Benign / expected.** This case exists to prove the pipeline works, not to investigate a threat. Closed once all three events were confirmed present and correctly fielded.

## Analyst notes

This is the case I'd point to first if asked "how do you know your data pipeline is actually reliable" — it's a controlled, known-input test with a known-good expected output, run before any detection logic was trusted for anything more ambiguous.
