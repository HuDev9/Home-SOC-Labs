# Splunk Setup Notes

## Splunk Enterprise (Ubuntu Server 24.04 — `192.168.1.125`)

**Installed:** Splunk Enterprise

**Configured:**
- Splunk Web (management/search UI)
- Receiving port `9997` (so it can accept forwarded data)
- Indexes — data lands in `main`
- Search & Reporting app for querying ingested data

**Verified:**
- Forwarder connectivity from the Windows victim
- Log ingestion — confirmed new events arriving in near-real-time after the forwarder connected
- Searches returning expected fields (`_time`, `User`, `Image`, `ParentImage`, `CommandLine`, `EventCode`, etc.)
- Basic dashboards built on top of the core detections

## Splunk Universal Forwarder (Windows 11 Home — `192.168.1.188`)

**Installed:** Splunk Universal Forwarder

**Configured:**
- `inputs.conf` — see [`configs/inputs.conf`](../configs/inputs.conf) — collecting Application, Security, System, and Sysmon/Operational Windows Event Log channels into `index=main`
- `outputs.conf` — see [`configs/outputs.conf`](../configs/outputs.conf) — forwarding to `192.168.1.125:9997`

**Verified:**
```text
> splunk list forward-server

Active forwards:
        192.168.1.125:9997
```

Successfully sent both Windows Events and Sysmon Events to the Splunk indexer.

## Why `index=main` explicitly, everywhere

Splunk will silently drop events into `main` by default in some configurations — but not all, and definitely not for Windows Event Log inputs if the stanza doesn't specify an index that actually exists on the receiving indexer. Every stanza in `inputs.conf` names `index = main` explicitly rather than relying on defaults, precisely because relying on the default is what caused [Issue 1](../troubleshooting/issue-01-index-mismatch.md).
