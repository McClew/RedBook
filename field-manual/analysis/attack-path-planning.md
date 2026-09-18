---
icon: route
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

# Attack Path Planning

Attack path planning is the discipline of turning raw reconnaissance and enumeration output into an ordered, justified sequence of steps towards a defined objective - a foothold, a specific host, domain compromise, etc. - rather than attacking targets ad hoc as they're discovered.

{% hint style="info" %}
#### Hint

In other words, it's the synthesis step that follows enumeration and precedes exploitation: the point where scattered findings become a coherent plan.
{% endhint %}

***

## Core Principles

* **Consolidate Findings:** Pull everything gathered during enumeratiuon into a single view of the attack surface, rather than working from scattered notes.
* **Chain Weaknesses Together:** Individual low/medium-severity issues often only become significant once linked into a chain that ends in code execution or privilege escalation.
* **Prioritise by Likelihood & Impact:** A well documented misconfiguration with a working public PoC is usuallly a better first move than a theoretical high-severity bug with no reliable exploit.
* **Map Trust & Access Relationships:** Particularly in AD environments where the route from an initial low-privileged goothold to Domain Admin is rarely a single exploit, but a path through group memberships, ACLs, delegation settings and cache credentials.
* **Document the Plan:** Document as we work through the path, so the eventual report can present a coherent attack narrative rather than a disconnected list of findings.

***

## Analysis Before Action

Vulnerability assessment is fundamentally analytical - it's where the results of information gathering are examined and interpreted, not where new data is collected. An analysis, in this context, is a detailed examination of an event or process - describing its origin and impact - so that precautions or actions can be taken to support or prevent future occurrences.

Four types of analysis are relevant here, moving from backward-looking to forward-looking:

<table><thead><tr><th width="147.66665649414062">Analysis type</th><th>Purpose</th></tr></thead><tbody><tr><td>Descriptive</td><td>Characterises a data set based on individual features; helps surface collection errors or outliers</td></tr><tr><td>Diagnostic</td><td>Establishes the causes, effects, and interactions behind a condition - asking <em>why</em> something happened</td></tr><tr><td>Predictive</td><td>Uses historical and current data to build a model of future probabilities, spotting trends and deviations early</td></tr><tr><td>Prescriptive</td><td>Narrows down which actions to take to prevent a future problem or trigger a needed process</td></tr></tbody></table>

### Example: Reasoning Scan Results

A useful habit is to distinguish between what a scan _shows_ and what is actually _there_. Take an open TCP port 2121 found during information gathering, with no service banner returned:

1. **Observe:** it's a TCP port - connection-oriented, non-standard (outside the 0–1023 well-known range).
2. **Hypothesise:** the digits echo the well-known FTP port, 21. Administrators often disguise standard ports with "easy to remember" alternatives.
3. **Test:** connect directly with Netcat or an FTP client to confirm or disprove the guess, rather than accepting the scan output at face value.
4. **Refine:** if the connection is unusually slow (e.g. \~15 seconds), consider that the service may have a configurable response delay - confirm by re-running the scan with an adjusted Nmap `--min-rtt-timeout`.

### Vulnerability Research to Attack Vectors

Once a service or application version has been identified, check it against known-vulnerability sources - CVE Details, Exploit-DB, Vulners, Packet Storm Security, NIST - to establish whether a disclosed vulnerability is likely still present. Where covert operation is required, mirror the target environment locally (matching versions as closely as possible) and test there rather than against the live system.

If analysis doesn't surface a workable vulnerability, loop back to information gathering - these two stages routinely cycle back and forth, and that's normal.

**A real penetration test is not a CTF.** Speed matters less than thoroughness here: in a CTF the goal is the fastest route to the flag, but on an engagement, an overlooked simple vector that a client is later breached through is the worst possible outcome.

{% hint style="warning" %}
#### When there's no reliable PoC

Reconstruct the exploit locally against a VM that mirrors the target as closely as possible (matching service/application version numbers), refine it there, and only then apply it to the live target.
{% endhint %}

{% hint style="danger" %}
#### When in doubt, communicate

If there's any doubt about the risk of an attack, put the decision to the client rather than proceeding unilaterally - give them what they need to decide whether exploitation should go ahead, or whether the finding should simply be reported as an unconfirmed likely issue. Clients will always prefer extra communication over having to recover a system after a failed exploitation attempt.
{% endhint %}
