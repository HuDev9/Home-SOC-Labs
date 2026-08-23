# Case 002 — PowerShell Activity

**Status:** Closed — expected activity, detection validated
**Host:** Windows 11 Home victim (`192.168.x.xxx`)
**Related simulation:** [Attack Simulation 1](../attack-simulations/attack-01-process-creation-validation.md)
**Related detection:** [Detection 2 — PowerShell Execution Monitoring](../detections/detection-02-powershell-monitoring.md)

## Summary

Focused investigation into the `powershell.exe` launch from Attack Simulation 1, isolating it from the broader process-creation dataset to confirm the PowerShell-specific detection surfaces the right fields for triage.

## Query used

```spl
index=main EventCode=1
| search Image="*powershell.exe"
| table _time User Image ParentImage CommandLine
```

## Findings

| Field | Value observed |
|---|---|
| `Image` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| `ParentImage` | `explorer.exe` |
| `CommandLine` | Bare `powershell.exe` invocation, no arguments, no encoded command |
| `User` | Interactive logged-in account |

- `ParentImage` of `explorer.exe` is consistent with a user directly opening PowerShell (e.g., via Start menu or Run) rather than a script, document macro, or scheduled task spawning it — a materially different and lower-concern lineage than what a real living-off-the-land technique would show.
- No `-enc`/`-EncodedCommand` or network cmdlets present in `CommandLine`, so there was nothing to decode or unpack.

## Disposition

**Benign / expected.** This event is the "known-good" baseline for what a normal, interactive PowerShell launch looks like on this host — useful as a comparison point for any future event where `ParentImage` or `CommandLine` looks different from this.

## Analyst notes

This case is the reference point I'd use to explain, in an interview, what "normal" PowerShell activity looks like on this specific host — which matters because effective PowerShell detection is less about "did PowerShell run" (it runs constantly, legitimately) and more about "does this specific launch's lineage and command line deviate from the host's own baseline."
