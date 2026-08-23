# Architecture

## Overview

The lab is three virtual machines under VirtualBox on a single Ubuntu 24.04 host, wired together with bridged networking so each VM behaves like a real host on the home LAN (`192.168.x.x/xx`) instead of a NATed guest.

```
                         192.168.x.x/xx  (Bridged Networking)

  ┌─────────────────────┐        ┌──────────────────────┐        ┌─────────────────────────┐
  │   Kali Linux          │        │  Windows 11 Home       │        │  Ubuntu Server 24.04      │
  │   (Attacker)           │        │  (Victim)               │        │  (SIEM)                    │
  │   192.168.x.xxx        │        │  192.168.x.xxx          │        │  192.168.x.xxx             │
  │                         │        │                         │        │                             │
  │                         │  ───▶  │  Sysmon                 │  ───▶  │  Splunk Enterprise          │
  │                         │        │  Splunk Universal        │  TCP   │  - Splunk Web               │
  │                         │        │  Forwarder                │  9997  │  - Receiving port xxxx      │
  └─────────────────────┘        └──────────────────────┘        │  - Indexes (main)           │
                                                                     │  - Search & Reporting       │
                                                                     └─────────────────────────┘
```

## Host machine

| Property | Value |
|---|---|
| OS | Ubuntu 24.04 |
| Virtualization | VirtualBox |
| Allocated resources | 16 CPU cores / 14 GB RAM |

Running all three guests on one host meant resource allocation had to be deliberate — Splunk Enterprise and the Windows victim are both memory-hungry, so CPU/RAM headroom was budgeted per VM rather than left on defaults.

## Why bridged networking

VirtualBox's default NAT mode hides each VM behind the host's network stack, which is fine for internet access but wrong for this lab's purpose: I needed the Windows victim and the Splunk indexer to reach each other directly on TCP 9997, and I needed to troubleshoot that reachability the way you would on real infrastructure — pings, firewall rules, ARP — not NAT port-forwarding rules. Bridged Adapter puts every VM directly on `192.168.1.0/24` with its own address, so connectivity issues (see [Issue 4](../troubleshooting/issue-04-vm-networking.md)) are the same class of problem an analyst would hit on physical infrastructure.

## Component roles

### Kali Linux — Attacker (`192.168.x.xxx`)
Source of all offensive activity used to validate detections: RDP brute-force attempts and msfvenom payload generation. Everything Kali does in this lab exists to generate telemetry for the victim/SIEM side to catch — it isn't the subject of the lab, it's the stimulus.

### Windows 11 Home — Victim (`192.168.x.xxx`)
The monitored endpoint. Runs:
- **Sysmon** — captures process creation and other system activity at a much higher fidelity than native Windows auditing
- **Splunk Universal Forwarder** — reads local Windows Event Log channels (Application, Security, System, Sysmon/Operational) per `inputs.conf` and ships them to the indexer over TCP 9997

See [`configs/inputs.conf`](../configs/inputs.conf) for the exact forwarder configuration.

### Ubuntu Server 24.04 — SIEM (`192.168.x.xxx`)
Runs **Splunk Enterprise**: Splunk Web, a receiving port on 9997, the `main` index that all forwarded data lands in, and Search & Reporting for detection and investigation. This is where every detection in [`detections/`](../detections) and every investigation in [`investigations/`](../investigations) is actually run.

## Data flow, end to end

1. An action happens on the Windows victim (a process launches, a logon succeeds or fails).
2. **Sysmon** (for process-level activity) or native **Windows Event Log** (for logon/auth activity) records the event.
3. The **Splunk Universal Forwarder**, per its `inputs.conf`, picks up the relevant Event Log channel.
4. The forwarder ships the event over **TCP xxxx** to the Ubuntu Splunk Enterprise indexer at `192.168.x.xxx`.
5. The event lands in `index=main`.
6. Detections (saved SPL searches) and manual investigation happen against that index in Splunk's Search & Reporting app.

This chain is the backbone of the whole repo — every detection writeup and every investigation case traces back through these exact six steps.
