# Issue 3 — Splunk Service Startup Failure

## Symptoms

Splunk Enterprise on the Ubuntu Server 24.04 SIEM would not start cleanly, surfacing a warning at startup:

```text
Running Splunk Enterprise as root is deprecated
```

## Cause

Splunk was being started under the root account. Recent Splunk Enterprise versions explicitly discourage — and in some startup paths block or warn against — running as root, for the same reason running any service as root is generally bad practice: it's a much larger blast radius than the service actually needs, and Splunk has supported running as a dedicated non-root user for a long time.

## Fix

Started and ran Splunk Enterprise under a dedicated, non-root service context rather than root, and verified clean startup and correct service status afterward:

```text
splunk status
```

confirmed the service was running normally with no deprecation warning on subsequent starts.

## Takeaway

Running as root doesn't just carry unnecessary risk — in this case it was actively surfaced as deprecated behavior by Splunk itself, which is a useful reminder that service warnings like this are worth resolving immediately rather than working around, since they tend to indicate a real architectural expectation the software has about how it should be run.
