# Build Log

A chronological record of how the lab was built, in the order components actually came online.

## 1. Host and virtualization layer

- Installed VirtualBox on the Ubuntu 24.04 host.
- Allocated 16 CPU cores / 14 GB RAM across the three planned guest VMs.
- Decided on **Bridged Networking** for every VM so the lab would sit on the real home LAN (`192.168.1.0/24`) rather than behind VirtualBox NAT — see [`architecture/README.md`](../architecture/README.md) for the reasoning.

## 2. Ubuntu Server 24.04 — SIEM

- Provisioned the Ubuntu Server 24.04 guest, assigned `192.168.1.125`.
- Installed Splunk Enterprise.
- Configured Splunk Web, the receiving port (9997), and the `main` index.
- Verified Search & Reporting worked with test data before connecting any forwarder.

## 3. Windows 11 Home — Victim

- Provisioned the Windows 11 Home guest, assigned `192.168.1.188`.
- Installed Sysmon; verified Event ID 1 (Process Creation) was generating locally in Event Viewer before touching Splunk at all — isolating "is Sysmon working" from "is forwarding working" as two separate questions.
- Installed the Splunk Universal Forwarder.
- Wrote `inputs.conf` to collect Application, Security, System, and Sysmon/Operational logs into `index=main`.
- Wrote `outputs.conf` to point the forwarder at `192.168.1.125:9997`.

## 4. Kali Linux — Attacker

- Provisioned the Kali Linux guest, assigned `192.168.1.122`.
- Used purely as the source of activity for detection validation — RDP brute-force generation and msfvenom payload creation.

## 5. Validation

- Confirmed forwarder connectivity with `splunk list forward-server` on the Windows host (expected `Active forwards: 192.168.1.125:9997`).
- Ran `notepad.exe`, `hostname.exe`, and `powershell.exe` on the victim and confirmed matching Sysmon Event ID 1 entries appeared in Splunk within seconds.
- Wrote and validated the four core detections against real generated activity (see [`detections/`](../detections)).

## 6. Detection engineering and investigation practice

- Wrote SPL for process creation, PowerShell execution, and authentication (4624/4625) monitoring.
- Ran the three attack simulations and worked each one as a full investigation case (see [`investigations/`](../investigations)).

## 7. Troubleshooting encountered along the way

Four distinct issues came up during the build and are documented in full in [`troubleshooting/`](../troubleshooting):

1. Logs not appearing in Splunk (index mismatch)
2. Sysmon events not appearing (Event Log permissions)
3. Splunk service startup failure (running as root)
4. VM networking issues (bridged adapter / firewall)
