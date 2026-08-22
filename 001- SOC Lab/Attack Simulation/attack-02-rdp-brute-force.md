# Attack Simulation 2 — RDP Brute Force Detection Lab

## Goal

Generate real failed and successful RDP authentication events against the Windows victim so that the 4625/4624-based detections ([Detection 3](../detections/detection-03-brute-force.md), [Detection 4](../detections/detection-04-successful-auth.md)) could be validated against actual traffic instead of assumed behavior.

## Attack methodology

From Kali Linux (`192.168.1.122`), targeted the Windows victim's RDP service (`192.168.1.188`):

1. Attempted a series of RDP logons using incorrect credentials to generate a burst of failed authentication events.
2. Followed the failed attempts with a correct-credential logon to generate a successful authentication event, deliberately positioned immediately after the failures — mirroring the real-world pattern of a brute force that eventually succeeds.

## Relevant Event IDs

| Event ID | Meaning |
|---|---|
| 4625 | An account failed to log on |
| 4624 | An account was successfully logged on |

## Detection methodology

Two SPL searches, run independently and then correlated:

```spl
index=main EventCode=4625
```
```spl
index=main EventCode=4624
```

And the correlated view used to actually catch the "failures followed by a success" pattern:

```spl
index=main (EventCode=4625 OR EventCode=4624)
| transaction Account_Name maxspan=5m
| where eventcount > 1 AND mvcount(EventCode) > 1
```

## Investigation workflow

1. Confirmed the burst of 4625 events landed in `index=main` with the expected `Account_Name` and source information.
2. Confirmed the subsequent 4624 for the same account arrived shortly after.
3. Ran the correlated `transaction` search to confirm the two event types tied together automatically within the 5-minute window, rather than having to manually eyeball two separate result sets.
4. Documented the full timeline as [Case 003 — Brute Force Detection](../investigations/case-003-brute-force.md).

## Why this simulation exists

A detection that only fires on failed logons misses the moment that actually matters most — the point where a brute force stops failing. This simulation exists specifically to validate that the lab's telemetry and SPL can surface *both* halves of that story, not just the noisy half.
