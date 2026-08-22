# Detection 4 — Successful Authentication Monitoring

## Purpose

Track successful logons on the victim host. On its own this isn't an alert — successful logons happen constantly and legitimately — but it's essential context for every other detection in this lab, especially Detection 3. A brute-force attempt (many 4625s) followed immediately by a 4624 from the same source is a materially different, higher-severity story than the failed attempts alone.

## Data source

Windows **Security Event ID 4624** (An account was successfully logged on), collected via the Universal Forwarder's `WinEventLog://Security` input into `index=main`.

## SPL

```spl
index=main EventCode=4624
```

## Correlating with brute force activity

The real value of this detection is in combination with Detection 3:

```spl
index=main (EventCode=4625 OR EventCode=4624)
| transaction Account_Name maxspan=5m
| where eventcount > 1 AND mvcount(EventCode) > 1
```

This surfaces accounts that saw both failed and successful logons within a short window — the pattern that matters for [Investigation Case 003](../investigations/case-003-brute-force.md), where the RDP brute-force simulation was deliberately followed by a correct password to see whether the eventual success was visible against the noise of the preceding failures.

## Tuning / false positives

4624 alone is far too broad to alert on directly — every normal login generates one. This detection exists specifically to be joined against other event types (4625, unusual logon times, unusual `Logon_Type` values) rather than to stand alone.

## Validated against

[Attack Simulation 2 — RDP Brute Force Detection Lab](../attack-simulations/attack-02-rdp-brute-force.md), investigated in [Case 003](../investigations/case-003-brute-force.md).
