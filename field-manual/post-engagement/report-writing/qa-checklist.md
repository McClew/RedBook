---
icon: medal
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

# QA Checklist

Before a report goes out the door, run it through this checklist. A single sloppy screenshot, an inconsistent severity rating, or a leftover reference to a different client can undermine confidence in an otherwise excellent assessment - this is the last line of defence before the client sees it.

***

## Content & Accuracy

* [ ] Every finding maps to specific in-scope hosts, URLs, or systems - nothing out of scope has slipped in
* [ ] Severity ratings are consistent finding-to-finding and follow the criteria set out in Vulnerability Classification & Risk Quantification
* [ ] Every finding includes: description, impact, affected systems, recommendation, reference links, and reproduction steps/evidence
* [ ] Every item in the Recommendations Summary maps back to an actual finding later in the report
* [ ] No finding references a tool, host, or credential that doesn't appear anywhere else in the report or supporting notes

## Findings Consistency

* [ ] Finding titles are consistent in tense and format throughout the document
* [ ] Duplicate or near-duplicate findings have been merged, or clearly differentiated if kept separate
* [ ] The attack chain in the Assessment Narrative lines up with the individual findings it references
* [ ] Findings that were remediated or retested during the engagement are flagged as such

## Evidence & Redaction

* [ ] All cleartext credentials, hashes, and PII are redacted per Evidence Handling
* [ ] Screenshots use solid-block redaction, never blur or pixelation
* [ ] Every screenshot includes enough context (address bar, hostname, prompt) to prove it's the client's environment and not a lab
* [ ] Terminal output is presented as text/code blocks rather than screenshots wherever possible
* [ ] Any PoC scripts have been genericised and contain no leftover real credentials, IPs, or hostnames

## Language & Tone

* [ ] The Executive Summary contains no acronyms, protocol names, or unexplained jargon
* [ ] Claims are appropriately qualified where you're inferring rather than certain ("showed no indication of detecting" rather than "did not detect")
* [ ] No specific commercial vendor names are recommended, per the Executive Summary's "Do Not" guidance
* [ ] Tone is constructive throughout - call out gaps in process, not people

## Formatting & Delivery

* [ ] Table of contents matches the actual section headings and page numbers
* [ ] Heading styles, fonts, and numbering are consistent throughout
* [ ] Every appendix referenced in-line actually exists and contains the right data
* [ ] Client name and logo are correct everywhere - a leftover reference to a previous client is one of the fastest ways to lose credibility
* [ ] The report has been spell-checked and read aloud at least once by someone other than the author
* [ ] The final exported PDF opens cleanly and images haven't been compressed into illegibility

## Final Sign-off

* [ ] A second reviewer, not the primary author, has read the full report end-to-end
* [ ] Reviewer comments have been resolved, not just acknowledged
* [ ] The report file follows the firm's naming convention and includes the correct version and date
