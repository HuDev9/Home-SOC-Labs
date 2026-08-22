# Case 003 — Brute Force Detection

**Status:** Closed — simulated attack, detection validated
**Host:** Windows 11 Home victim (`192.168.1.188`)
**Source:** Kali Linux attacker (`192.168.1.122`)
**Related simulation:** [Attack Simulation 2 — RDP Brute Force Detection Lab](../attack-simulations/attack-02-rdp-brute-force.md)
**Related detections:** [Detection 3 — Brute Force](../detections/detection-03-brute-force.md), [Detection 4 — Successful Authentication](../detections/detection-04-successful-auth.md)

## Summary

Investigation of a simulated RDP brute-force attempt: a burst of failed logons (Event ID 4625) from the Kali attacker against the Windows victim, followed by a single successful logon (Event ID 4624) using correct credentials — the pattern of a brute force that eventually succeeds.

## Timeline

| Time (relative) | Event | Event ID |
|---|---|---|
| T+0s | First failed RDP logon attempt | 4625 |
| T+~2–30s | Repeated failed RDP logon attempts | 4625 (multiple) |
| T+~35s | Successful RDP logon with correct credentials | 4624 |

## Queries used

Initial triage — confirm failures are present:
```spl
index=main EventCode=4625
```

Confirm the eventual success:
```spl
index=main EventCode=4624
```

Correlated view — the query that actually tells the story:
```spl
index=main (EventCode=4625 OR EventCode=4624)
| transaction Account_Name maxspan=5m
| where eventcount > 1 AND mvcount(EventCode) > 1
```

## Findings

- Multiple 4625 events for the same account arrived in rapid succession — well outside normal single-mistyped-password behavior.
- A 4624 for the same account followed within the same short window.
- The `transaction`-based correlation search successfully grouped the failures and the eventual success into a single transaction, which is the exact signal that should drive an alert in a real environment: **not** "were there failures" (too noisy) and **not** "was there a success" (meaningless alone), but "did failures and a success happen together, for the same account, in a short window."

## Disposition

**True positive (simulated).** Confirms the detection logic correctly identifies the brute-force-then-success pattern using only SIEM data — no need to check the Windows host directly to determine that this was an attack.

## Analyst notes

This case is the strongest one in the repo for demonstrating investigative reasoning rather than just detection-writing: the interesting finding isn't the raw 4625 count, it's recognizing that failures-then-success is a fundamentally different risk than failures alone, and building a query that surfaces exactly that relationship instead of leaving the correlation to a human eyeballing two separate result tables.
