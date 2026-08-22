# Detection 3 — Brute Force / Failed Authentication Detection

## Purpose

Detect repeated failed logon attempts against the victim host — the signature of a password-guessing or brute-force attack against an exposed authentication service (in this lab's case, RDP).

## Data source

Windows **Security Event ID 4625** (An account failed to log on), collected via the Universal Forwarder's `WinEventLog://Security` input into `index=main`.

## SPL

```spl
index=main EventCode=4625
```

A production version of this detection would aggregate rather than list raw events:

```spl
index=main EventCode=4625
| stats count by src_ip, Account_Name, _time
| where count > 5
```

The raw-event version is kept in this repo as the baseline validation query — it's what was used to first confirm 4625 events were arriving from the attack simulation before building the aggregation on top of it.

## Purpose of aggregation

A single 4625 is a typo. Dozens of 4625s against the same account or from the same source IP in a short window is a brute-force attempt. The `stats count by` version is what would actually back an alert; the flat search is what an analyst runs first to confirm the raw events look the way the aggregation assumes they will.

## Tuning / false positives

- Legitimate users mistyping passwords will generate low-volume 4625s — the threshold (`count > 5` above) needs tuning against real baseline noise for the environment.
- Locked-out accounts can generate a burst of 4625s that isn't an active attack — correlate with account lockout events if extending this further.

## Validated against

[Attack Simulation 2 — RDP Brute Force Detection Lab](../attack-simulations/attack-02-rdp-brute-force.md), investigated in [Case 003](../investigations/case-003-brute-force.md).
