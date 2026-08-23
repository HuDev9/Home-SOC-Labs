# Configs

Sanitized configuration files pulled directly from the lab. No credentials, license keys, or machine-identifying data beyond the lab's own private IP addressing (which is documented intentionally throughout this repo for traceability).

| File | Host | Purpose |
|---|---|---|
| [`inputs.conf`](./inputs.conf) | Windows 11 Victim | Defines which Event Log channels the Universal Forwarder collects and which index they land in |
| [`outputs.conf`](./outputs.conf) | Windows 11 Victim | Points the Universal Forwarder at the Splunk indexer (`192.168.1.x:xxxx`) |

Related reading: [`architecture/README.md`](../architecture/README.md) for how these configs fit into the overall data flow, and [`troubleshooting/issue-01-index-mismatch.md`](../troubleshooting/issue-01-index-mismatch.md) for what broke before `inputs.conf` looked like this.
