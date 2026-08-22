# Issue 1 — Logs Not Appearing in Splunk

## Symptoms

Windows Event Log data was being collected by the Universal Forwarder (confirmed via forwarder logs) but was not appearing in Splunk search results. The indexer's internal logs showed:

```text
Received event for unconfigured index=wineventlog
```

## Cause

The forwarder's `inputs.conf` stanzas were not explicitly targeting an index that actually existed on the receiving Splunk Enterprise indexer. Some Windows Event Log inputs default to (or were implicitly assumed to use) an index name — `wineventlog` — that had never been created on the indexer side, so Splunk correctly rejected the events rather than silently dropping them into `main`.

## Diagnosis steps

1. Confirmed the forwarder was actually sending data by checking `splunk list forward-server` on the Windows host — connectivity itself was fine.
2. Checked Splunk Enterprise's own internal logs (`index=_internal`) on the indexer, which surfaced the `unconfigured index` message directly — this was the key clue that data was arriving but being rejected, not that it wasn't arriving at all.
3. Compared the index name referenced in the internal log message against the indexes actually configured on the Splunk Enterprise instance.

## Fix

Edited every stanza in `inputs.conf` on the Windows victim to explicitly set:

```ini
index = main
```

`main` already existed on the indexer by default, so this was the fastest correct fix. Restarted the Universal Forwarder service and confirmed events began appearing in `index=main` searches immediately.

## Takeaway

An index mismatch fails *silently from the analyst's point of view* — search results just look empty, with no obvious error unless you specifically go check the indexer's internal logs. This was the single most valuable lesson from the whole build: **`index=main` is not a default you can assume, it's a decision every input has to explicitly agree with the indexer on.** See [Lessons Learned](../lessons-learned/README.md) for more.
