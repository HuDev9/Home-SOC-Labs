# Lessons Learned

## 1. Index configuration is a decision, not a default

The single most instructive failure of the whole build ([Issue 1](../troubleshooting/issue-01-index-mismatch.md)) was assuming that forwarded Windows Event Log data would land somewhere sensible by default. It doesn't — every input has to explicitly agree with the indexer on which index it's writing to, and a mismatch fails silently from the analyst's seat: search results are just empty, with no obvious error unless you go check the indexer's internal logs (`index=_internal`). I now treat "check the internal logs first" as a standing first step whenever expected data isn't showing up, rather than re-checking configuration syntax first.

## 2. Windows Event Log permissions are their own access-control layer

[Issue 2](../troubleshooting/issue-02-sysmon-permissions.md) taught me that a service account having general OS-level rights to run doesn't imply it can read every Event Log channel. Channel-level read access (`Event Log Readers`) is separate and has to be granted explicitly per channel in some configurations. The diagnostic pattern that found this — noticing that *some* channels worked and *one specific channel* didn't, under identical forwarder config — is a technique I'd reach for again: when a config works for 90% of a case and fails for one specific piece, look for permissions/access scoped more narrowly than the config itself, not a config bug.

## 3. Service warnings usually mean something real

[Issue 3](../troubleshooting/issue-03-splunk-root.md) was resolved fast specifically because I treated the "deprecated" warning as something to fix immediately rather than something to work around. Splunk telling me running as root is deprecated wasn't cosmetic — it reflected a real expectation about how the service is meant to run in production, and ignoring it would have just deferred a problem rather than avoided one.

## 4. A failed ping can mean three different things

[Issue 4](../troubleshooting/issue-04-vm-networking.md) reinforced that "no connectivity" is a symptom with multiple unrelated possible causes (adapter mode, IP conflict, firewall) that all look identical from one failed ping. Isolating variables one at a time — adapter setting, then IP, then firewall — found the actual cause faster than assuming the most likely explanation was correct on the first try.

## 5. Bridged networking made troubleshooting *more* representative of real work, not harder

I chose Bridged Adapter over the simpler default NAT specifically because I wanted network troubleshooting in this lab to resemble troubleshooting real infrastructure — routable addresses, real firewall interaction, real adapter configuration — rather than debugging VirtualBox-specific NAT quirks that wouldn't transfer to a real environment. That decision is directly responsible for Issue 4 existing at all, and I'd make the same trade again.

## 6. Validate the pipeline before trusting the detections

Attack Simulation 1 and Cases 001–002 exist because I wanted proof that ordinary, known-good activity showed up completely and correctly *before* I started trusting any detection logic against ambiguous activity. Separating "does the pipeline work" from "does the detection work" made every subsequent troubleshooting step faster, because I could rule the pipeline in or out immediately instead of re-deriving that confidence every time something looked wrong.

## 7. Detections should say what they don't catch, not just what they do

[Case 004](../investigations/case-004-reverse-shell.md) was deliberately written as a gap analysis rather than a confirmed detection. A portfolio (or a real detection library) that only shows wins isn't an honest picture of coverage — being able to say precisely what current telemetry does and doesn't see, and what would close the gap, is as much a demonstration of SOC maturity as the working detections themselves.
