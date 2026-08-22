# Detection 2 — PowerShell Execution Monitoring

## Purpose

PowerShell is one of the highest-value places to watch on a Windows endpoint — it's a living-off-the-land binary that legitimate admin tooling and malicious tradecraft both rely on. This detection filters the broader process-creation stream down to PowerShell executions specifically, so command-line content (encoded commands, download cradles, obfuscation) is front and center rather than buried in general process noise.

## Data source

Sysmon **Event ID 1 (Process Creation)**, filtered to `Image` values matching `powershell.exe`, forwarded into `index=main`.

## SPL

```spl
index=main EventCode=1
| search Image="*powershell.exe"
| table _time User Image ParentImage CommandLine
```

## Parent-child relationships

`ParentImage` is the single most useful field in this search. PowerShell launched directly by `explorer.exe` (a user opening a shortcut or typing into Run) reads very differently from PowerShell launched by `winword.exe`, `outlook.exe`, `wscript.exe`, or `cmd.exe` spawned from a script — the latter set are classic macro/dropper execution chains. This lab's own baseline PowerShell launches (from the [process creation validation simulation](../attack-simulations/attack-01-process-creation-validation.md)) were spawned directly by the interactive user session, which is what "normal" looks like on this specific host.

## Command line visibility

Because `renderXml = true` is set in `inputs.conf`, the full `CommandLine` field is preserved rather than truncated. This is what makes it possible to actually read encoded (`-enc`/`-EncodedCommand`) or obfuscated arguments instead of just knowing PowerShell ran at all.

## Threat hunting use cases

- Hunting for `-enc`/`-EncodedCommand` flags, which usually indicate base64-encoded payloads worth decoding
- Hunting for network-related cmdlets in `CommandLine` (`Invoke-WebRequest`, `IEX`, `DownloadString`) as indicators of a download cradle
- Baselining which `ParentImage` values are normal for PowerShell on this specific host, so new/unusual parents stand out

## Tuning / false positives

Legitimate scheduled tasks, software installers, and admin scripts all launch PowerShell routinely. This search is a hunting query, not a standalone alert — in a production setting it would be paired with allow-listed parent processes and known-good scheduled task names before being wired to notify anyone.

## Validated against

[Attack Simulation 1 — Process Creation Validation](../attack-simulations/attack-01-process-creation-validation.md) (the `powershell.exe` launch), investigated in [Case 002](../investigations/case-002-powershell-activity.md).
