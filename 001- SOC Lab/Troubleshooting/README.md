# Troubleshooting

Four real issues hit during the build of this lab. Each is documented the same way: **Cause → Symptoms → Fix**, written so a teammate could resolve the same issue from this page alone.

| # | Issue | Cause | File |
|---|---|---|---|
| 1 | Logs not appearing in Splunk | Incorrect / unconfigured index | [`issue-01-index-mismatch.md`](./issue-01-index-mismatch.md) |
| 2 | Sysmon events not appearing | Windows Event Log permission issue | [`issue-02-sysmon-permissions.md`](./issue-02-sysmon-permissions.md) |
| 3 | Splunk service startup failure | Running Splunk as root | [`issue-03-splunk-root.md`](./issue-03-splunk-root.md) |
| 4 | VM networking issues | Bridged adapter / firewall configuration | [`issue-04-vm-networking.md`](./issue-04-vm-networking.md) |
