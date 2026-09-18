---
icon: receipt
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Proof-of-Concept

Some findings - particularly custom-developed exploits, unwieldy web application payloads, or multi-step exploit chains — are better supported by a standalone Proof-of-Concept (PoC) artefact delivered alongside the report, rather than being fully inlined into the Findings narrative. This page covers when that's worth doing and how to package it safely.

This is a companion to Findings' "Reproduction Steps" section, not a replacement for it - the walkthrough narrative still belongs there. A PoC artefact is the reusable, runnable version of that walkthrough that a client's technical team can execute themselves once a fix is in place.

***

## When to Provide a Standalone PoC

* Custom exploit code developed during the engagement - not just a public tool run with standard flags
* Web application payloads that are unwieldy to render inline (long URL-encoded requests, multi-request chains, anything involving CSRF tokens or timing)
* Anything the client's blue team will want to re-run themselves against a patched environment to confirm remediation

{% hint style="info" %}
#### Note

If the exploit simply relies on a well-known public tool run with standard options, don't repackage the tool itself - reference it and supply only the specific configuration or flags used.
{% endhint %}

***

## Structure of a PoC Script

* A header comment block stating the finding it corresponds to, the target scope it was tested against, the author, the date, and a one-line usage example
* Inline comments explaining each stage - not just _what_ the code does, but _why_ that step is needed
* Configurable variables declared at the top (target, port, output path) rather than hardcoded values scattered through the script
* Sane defaults, but nothing destructive by default - no automatic clean-up or overwriting of data without an explicit flag

***

## Safety & Liability

{% hint style="danger" %}
#### Before Handoff

Never hand over code containing hardcoded credentials, real target IPs from the engagement, or anything that could be run unmodified against a system outside the agreed scope without the client realising it. Genericise every PoC before it leaves your hands.
{% endhint %}

* Include a short disclaimer at the top of the script stating its intended purpose and that it should only be run against systems and within the time window covered by the Rules of Engagement.
* Test the PoC one final time in isolation immediately before handoff. A script that only worked because of leftover state on your attack host is worse than useless to the client - it erodes confidence in everything else you've delivered.

***

## Packaging & Delivery

* Keep PoCs in a dedicated `Findings/<Finding Name>/PoC/` folder, matching the evidence structure described in Note Taking.
* Provide a short `README.md` alongside the PoC if it has any dependencies, prerequisites, or requires specific tool/library versions.
* Reference the PoC's location from within the finding itself - e.g. "See attached PoC script `finding-05-poc.py`" - so a reader isn't left hunting for it.

***

## Versioning & Attribution

If a PoC is adapted from an existing public script (for example, a GitHub repository for a specific CVE), retain the original author's licence and attribution, and note what you changed rather than presenting it as original work.

If it genuinely is original work, it may be worth cleaning up and publishing separately - see Findings' guidance on writing your own source material for clients and the wider community.
