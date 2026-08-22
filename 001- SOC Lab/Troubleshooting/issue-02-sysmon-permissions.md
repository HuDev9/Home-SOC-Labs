# Issue 2 — Sysmon Events Not Appearing

## Symptoms

Sysmon was confirmed running and generating events locally (visible in Windows Event Viewer under `Microsoft-Windows-Sysmon/Operational`), but those events were not showing up in Splunk, even though other channels (Application, System) from the same forwarder were arriving fine.

## Cause

A Windows Event Log permission issue — the account the Splunk Universal Forwarder service runs as did not have rights to read the Sysmon operational log channel specifically. The forwarder service's own OS-level permissions were fine (it could run, it could read other channels); Windows Event Log channel access is its own separate access-control layer on top of that, and the Sysmon channel wasn't included in what the forwarder's service account was allowed to subscribe to.

## Symptom detail

The Splunk Universal Forwarder's own internal logs showed:

```text
Could not subscribe to Windows Event Log channel
errorCode=5
```

`errorCode=5` is Windows for **Access Denied** — which pointed directly at permissions rather than configuration syntax, service state, or connectivity.

## Diagnosis steps

1. Confirmed Sysmon itself was healthy and generating events locally — ruled out Sysmon as the problem.
2. Confirmed other Event Log channels (Application, System) from the same forwarder *were* arriving in Splunk — ruled out the forwarder-to-indexer pipeline and `inputs.conf` syntax as the problem.
3. Checked the Universal Forwarder's own log files, which surfaced the `errorCode=5` access-denied message specific to the Sysmon channel.

## Fix

Added the forwarder's service account to the **Event Log Readers** local group:

```text
NT SERVICE\SplunkForwarder  →  Event Log Readers
```

Restarted the Splunk Universal Forwarder service. Sysmon events began appearing in Splunk immediately after restart.

## Takeaway

Windows Event Log channels enforce their own read permissions independent of general service account rights — a service can run fine and read some channels successfully while being silently denied on others. When one channel works and another doesn't under the exact same forwarder configuration, that's a strong signal to check channel-specific permissions rather than re-checking the configuration file.
