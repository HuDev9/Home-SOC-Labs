# Attack Simulations

Every detection in [`detections/`](../detections) was validated against activity actually generated in the lab — not sample data. This folder documents that activity from the attacker's side; [`investigations/`](../investigations) documents the same events from the analyst's side.

| # | Simulation | Generates | File |
|---|---|---|---|
| 1 | Process Creation Validation | Sysmon Event ID 1 | [`attack-01-process-creation-validation.md`](./attack-01-process-creation-validation.md) |
| 2 | RDP Brute Force Detection Lab | Windows Security 4625 / 4624 | [`attack-02-rdp-brute-force.md`](./attack-02-rdp-brute-force.md) |
| 3 | Weaponization (msfvenom) | Payload for defender-perspective analysis | [`attack-03-weaponization.md`](./attack-03-weaponization.md) |

All activity was performed by me against my own lab victim machine (`192.168.1.188`), on an isolated home lab segment, for the sole purpose of generating telemetry to validate detections.
