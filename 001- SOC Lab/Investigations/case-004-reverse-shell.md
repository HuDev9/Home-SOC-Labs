# Case 004 — Reverse Shell Detection

**Status:** Closed — gap analysis
**Host:** Windows 11 Home victim (`192.168.x.xxx`)
**Related simulation:** [Attack Simulation 3 — Weaponization](../attack-simulations/attack-03-weaponization.md)

## Summary

Unlike Cases 001–003, this case is a **gap analysis** rather than a confirmed detection: it examines what the current Sysmon-based host telemetry pipeline would and would not catch if a reverse-shell payload executed on the victim, and what's missing to close the gap.

## What the current pipeline would catch

The lab's existing Sysmon Event ID 1 collection ([Detection 1](../detections/detection-01-process-creation.md), [Detection 2](../detections/detection-02-powershell-monitoring.md)) would capture:

- The payload process launching, with its `Image` path and file hash
- Its `ParentImage` — the delivery mechanism's process, if the payload were launched from something like a document or script rather than run directly
- Its `CommandLine`, if the payload takes arguments at launch

## What the current pipeline would miss

- **The outbound network connection itself.** This lab has no network-layer visibility (no Zeek/Suricata, no firewall log ingestion) — so the actual callback to a listener, the destination IP/port, and the connection's persistence over time are all currently invisible to Splunk. Host telemetry alone can show *that a process ran*; it can't show *what that process talked to on the network* without either network monitoring or a Sysmon configuration that also captures Event ID 3 (Network Connection), which this lab's current Sysmon config does not enable.
- **Post-exploitation behavior.** Anything the reverse shell does after the callback succeeds (further process creation, credential access, lateral movement) would show up as additional Sysmon Event ID 1 entries downstream — but only if it's process-based. File-only or memory-only actions wouldn't be visible with this lab's current instrumentation.

## Disposition

**Gap identified, not closed.** This case exists specifically to be honest about the boundary of what the lab currently detects, rather than overstating coverage.

## Remediation path

Tracked in [Future Improvements](../README.md#future-improvements):
- Enable Sysmon Event ID 3 (Network Connection) in the Sysmon config to get outbound connection visibility from the host side
- Add Zeek or Suricata at a network tap point for independent network-layer detection
- Map this gap explicitly against MITRE ATT&CK (particularly the Command and Control tactic) once ATT&CK mapping is added to the other detections

## Analyst notes

I'm including this case specifically because a portfolio that only shows detections that worked isn't a realistic picture of SOC work — knowing what you *can't* see yet, and being able to name why and what would fix it, is as much a part of the job as writing the detections that already work.
