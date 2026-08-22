# Detection Engineering

Four detections, each written against telemetry this lab actually produces, tested against real generated activity rather than sample data.

| # | Detection | Source | Event ID | File |
|---|---|---|---|---|
| 1 | Process Creation Monitoring | Sysmon | 1 | [`detection-01-process-creation.md`](./detection-01-process-creation.md) |
| 2 | PowerShell Execution Monitoring | Sysmon | 1 (filtered) | [`detection-02-powershell-monitoring.md`](./detection-02-powershell-monitoring.md) |
| 3 | Brute Force / Failed Authentication | Windows Security | 4625 | [`detection-03-brute-force.md`](./detection-03-brute-force.md) |
| 4 | Successful Authentication | Windows Security | 4624 | [`detection-04-successful-auth.md`](./detection-04-successful-auth.md) |

## Format

Every detection writeup follows the same structure so they're easy to scan and easy to extend:

- **Purpose** — what behavior this detects and why it matters
- **Data source** — where the event originates and how it gets to Splunk
- **SPL** — the actual search
- **Field notes** — what each returned field means and why it's included
- **Tuning / false positives** — what legitimate activity can trigger this and how to narrow it
- **Validated against** — which attack simulation / investigation case exercised this detection
