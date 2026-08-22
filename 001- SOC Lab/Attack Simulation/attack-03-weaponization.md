# Attack Simulation 3 — Weaponization

## Goal

Understand the "weaponization" stage of the attack lifecycle from the defender's side by generating a payload with `msfvenom` in the isolated lab and reasoning through what it does, why a listener exists, and what an analyst should look for — without turning this into an offensive how-to. This section documents concepts and analyst takeaways, not a step-by-step exploitation guide.

## What happened, at a high level

A payload was generated with `msfvenom` (part of the Metasploit Framework) on the Kali attacker VM, targeting the isolated Windows 11 victim within this lab. The exercise stayed at "generate a payload, understand its structure and purpose, reason about how a defender would catch it" rather than a full intrusion chain — the value for this repo is the detection-engineering perspective, not the offensive tradecraft.

## Reverse shell concept

A reverse shell inverts the normal client-server relationship: instead of an attacker connecting inbound to a victim (which firewalls are generally built to stop), the victim process initiates an *outbound* connection back to a listener the attacker controls. Outbound connections are far less scrutinized by default network policy than inbound ones, which is precisely why this pattern is common — it's designed to blend into normal-looking egress traffic rather than trip inbound firewall rules.

## Payload generation, conceptually

`msfvenom` combines a **payload** (the code that establishes the callback and gives the attacker a shell) with an **encoder/format** (how that payload is packaged — as a standalone executable, a script, shellcode to embed elsewhere, etc.) into a single deliverable. In a real attack, that deliverable still needs a *delivery mechanism* (phishing, a dropped file, a vulnerable service) — payload generation alone doesn't compromise anything on its own, which is worth being precise about when explaining this to a non-technical audience.

## Purpose of the listener

The listener is the attacker-side process waiting for the payload's outbound connection. Its job is narrow: accept the callback and hand the attacker an interactive session once the payload connects. From a defensive standpoint, the listener itself never touches the victim — everything observable on the victim side is the payload's outbound connection attempt, which is exactly what host and network telemetry need to be positioned to catch.

## Why defenders care

- **Outbound connections are still visible.** Endpoint telemetry (like the Sysmon Event ID 1 process-creation data this lab already collects) can catch the payload process launching, its parent process, and its command line — the same fields used in [Detection 1](../detections/detection-01-process-creation.md) and [Detection 2](../detections/detection-02-powershell-monitoring.md).
- **Unusual outbound destinations and ports** are a network-layer signal this lab doesn't yet collect (see [Future Improvements](../README.md#future-improvements) — adding Zeek/Suricata is on the roadmap specifically to close this gap).
- **Process lineage matters more than the payload itself.** A payload delivered via a phishing document shows up as an abnormal parent-child chain (e.g., an Office process spawning something unexpected) before it shows up as anything network-related — reinforcing why Detection 2's focus on `ParentImage` is a defender's first and best signal, not an afterthought.

## Investigation

Worked as [Case 004 — Reverse Shell Detection](../investigations/case-004-reverse-shell.md), which focuses on what host-based telemetry *would* need to look like to catch this pattern, given what the current Sysmon-based pipeline collects today.

## Scope note

This activity was performed entirely against my own lab victim machine on an isolated home network segment, for the purpose of building detection engineering skill. No payload generated here was ever pointed at, delivered to, or run against any system outside this lab.
