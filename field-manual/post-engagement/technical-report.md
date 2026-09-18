---
icon: square-terminal
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

# Technical Report

The Technical Report is the primary, comprehensive deliverable of an engagement — everything the client's technical teams need to understand, reproduce, and remediate every issue found. Every other page in Documentation & Reporting exists to feed into this document.

***

## Audience

IT and security teams, sysadmins, and developers — the people who will actually implement fixes and need to reproduce and validate issues themselves. Unlike the Executive Report, acronyms and protocol-level detail are expected here, not avoided.

***

## Structure

A typical Technical Report is assembled, in order, from:

1. Executive Summary — retained for context even though the primary audience here is technical
2. Recommendations Summary
3. Assessment Narrative — the attack chain walkthrough
4. Findings — the bulk of the document
5. Proof-of-Concept artefacts, referenced from the relevant findings where applicable
6. Appendices — scope, methodology, severity ratings, and any dynamic appendices the engagement calls for

***

## What Makes This Different from the Executive Report

* Full technical detail throughout: reproduction steps, payloads, tool output, and CVSS breakdowns
* Every finding fully fleshed out per Findings' "Breakdown of a Finding" structure
* No requirement to avoid jargon — this is the one deliverable where it's expected and appropriate

***

## Quality Bar

Run the QA Checklist before this goes out. This is the document most likely to be picked apart by a technical reviewer on the client side, so reproducibility is the standard to hold yourself to: could a defender rebuild the entire attack path from this document alone, with no other input from you?

***

## Delivery Considerations

* Usually delivered as a PDF, sometimes alongside an editable working copy for the client to track remediation progress against
* If a retest is in scope, keep this document's structure consistent with the original so retest findings can be mapped 1:1 against it
