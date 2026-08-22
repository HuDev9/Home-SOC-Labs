# Investigations

Case-file style writeups worked from the analyst's side of the SIEM, one per attack simulation. Each follows the same structure: summary, timeline, SPL used, findings, and disposition — the way I'd want to hand a case off to a teammate or explain it in an interview.

| Case | Title | Related Simulation | Related Detection(s) |
|---|---|---|---|
| [001](./case-001-process-creation.md) | Sysmon Process Creation | [Attack 1](../attack-simulations/attack-01-process-creation-validation.md) | [Detection 1](../detections/detection-01-process-creation.md) |
| [002](./case-002-powershell-activity.md) | PowerShell Activity | [Attack 1](../attack-simulations/attack-01-process-creation-validation.md) | [Detection 2](../detections/detection-02-powershell-monitoring.md) |
| [003](./case-003-brute-force.md) | Brute Force Detection | [Attack 2](../attack-simulations/attack-02-rdp-brute-force.md) | [Detection 3](../detections/detection-03-brute-force.md), [Detection 4](../detections/detection-04-successful-auth.md) |
| [004](./case-004-reverse-shell.md) | Reverse Shell Detection | [Attack 3](../attack-simulations/attack-03-weaponization.md) | Gap analysis — see case notes |
