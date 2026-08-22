# Issue 4 — VM Networking Issues

## Symptoms

Intermittent and, in some cases, total ping failures between the lab VMs — most notably between the Windows victim and the Ubuntu Splunk indexer, which blocked forwarder connectivity entirely until resolved.

## Troubleshooting steps

1. **Verified Bridged Adapter configuration** on each VM in VirtualBox — confirmed all three guests were actually attached to the bridged adapter (rather than accidentally left on NAT or Host-only, which would explain isolation between guests).
2. **Verified IP addresses** on each VM matched what was expected for the `192.168.1.0/24` segment, and that no two VMs had landed on conflicting addresses.
3. **Tested connectivity directly** with `ping` between each pair of VMs to isolate which specific links were failing rather than assuming the whole network was down.
4. **Investigated Windows Firewall** on the victim machine — Windows' default firewall profile can silently block inbound ICMP (ping) and other traffic even when the underlying network path is fine, which made this a necessary check once basic adapter/IP configuration was ruled out.

## Cause

A combination of adapter misconfiguration (a VM not actually attached to the bridged network as assumed) and Windows Firewall rules blocking traffic that the network layer itself was actually delivering correctly — two different problems that looked identical from a single failed ping.

## Fix

- Corrected the network adapter setting on the affected VM(s) to Bridged Adapter.
- Adjusted the Windows Firewall profile on the victim to allow the traffic required for the lab (ICMP for troubleshooting visibility, and the ports needed for Splunk forwarding and RDP).
- Re-tested connectivity between every VM pair after each change rather than assuming one fix resolved everything.

## Takeaway

"Ping failed" can mean several unrelated things — wrong adapter mode, IP conflict, or a firewall silently dropping traffic that would otherwise arrive fine. Isolating *which* VM pairs failed and testing one variable at a time (adapter setting, then IP, then firewall) was what actually found the real cause, instead of assuming the first plausible explanation was the right one.
