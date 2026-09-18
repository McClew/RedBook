---
icon: chart-pie-simple
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

# Vulnerability Classification & Risk Quantification

Within the Penetration Testing Execution Standard (PTES) framework, this activity is a critical component of the **Analysis** and **Post-Engagement** (**Reporting**) phases.

It is the technical mechanism used for **Risk Prioritisation**, where raw data from enumeration and exploitation is transformed into standardised metrics that allow an organisation to assess its security posture.

This process involves three distinct technical actions:

* **Vulnerability Categorisation:** Identifying the specific Common Weakness Enumeration (CWE) ID associated with the root cause of a flaw (e.g., using CWE-284 for improper access control in an anonymous LDAP bind) to provide a universal reference for the underlying weakness.
* **Quantitative Risk Assessment:** Generating a CVSS score to measure the technical severity of the issue based on standardised variables such as the Attack Vector, Complexity, and the impact on Confidentiality, Integrity, and Availability.
* **Severity Rating Assignment:** Translating these scores into a qualitative Severity Rating (High, Medium, or Low) based on the potential harm and likelihood of exploitation, which helps technical teams prioritise remediation efforts.

***

## Useful Resources

To perform vulnerability classification and risk quantification, we can utilise the following websites:

* **NIST National Vulnerability Database (NVD):** Use this site to find pre-calculated CVSS scores, CWE categorisations, and detailed technical descriptions for established CVEs (Common Vulnerabilities and Exposures).\
  Site: [https://nvd.nist.gov/](https://nvd.nist.gov/)\
  CVSS v3.1 Calculator: [https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator](https://nvd.nist.gov/vuln-metrics/cvss/v3-calculator)
* **MITRE ATT\&CK Framework:** This resource is essential for mapping findings to specific adversary techniques and understanding the broader security impact within an environment.\
  Site: [https://attack.mitre.org/](https://attack.mitre.org/)
* **CVE Program (CVE.org):** Access this site for the official records and identification of vulnerabilities in various versions of core software.\
  Site: [https://www.cve.org/](https://www.cve.org/)
* **Official PTES Technical Guidelines:** Refer to this site for standard methodologies regarding the Analysis and Post-Engagement phases of a penetration test.\
  Site: [http://www.pentest-standard.org/index.php/PTES\_Technical\_Guidelines](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines)

***

## Calculating CVSS Scores

The Common Vulnerability Scoring System (CVSS) provides a numerical representation of severity. For most modern engagements, we utilise version 3.1 or 4.0.

To calculate these values accurately, we break the assessment down into three metric groups: `Base`, `Temporal`, and `Environmental`.

### Determining the Base Score

The Base score represents the intrinsic qualities of a vulnerability.

#### Exploitability Metrics

* **Attack Vector (AV):** We define how the vulnerability is reached.
  * _Network:_ Exploitable from the internet or a different subnet.
  * _Adjacent:_ Requires being on the same shared physical or logical network.
  * _Local/Physical:_ Requires shell access or physical hardware interaction.
* **Attack Complexity (AC):** We assess if specific conditions (like a race condition or a specific non-default configuration) must exist for the exploit to work.
* **Privileges Required (PR):** We determine the level of access needed _before_ the attack. Does the exploit work as an unauthenticated guest (None), or does it require Administrative (High) rights?
* **User Interaction (UI):** We decide if a victim must perform an action, such as clicking a CSRF link or opening a malicious document.

#### Impact Metrics (The CIA Triad)

We evaluate the impact on Confidentiality (C), Integrity (I), and Availability (A). We must decide if the impact is:

* **None:** No impact.
* **Low:** Minor data leakage or slight performance degradation.
* **High:** Total loss of data secrecy, full system compromise, or a complete Denial of Service (DoS).

### Factoring in Scope

One of the most nuanced decisions we make is determining if a Scope Change has occurred.

* **Unchanged (U):** The exploited vulnerability and the impacted resource are the same (e.g., a buffer overflow in a service).
* **Changed (C):** The vulnerability in one component affects resources in a different security authority (e.g., a Virtual Machine escape that compromises the Host OS).

{% hint style="info" %}
#### Note

A Scope Change often significantly increases the final score.
{% endhint %}

### The CVSS Hierarchy

We calculate these scores in a specific order because each subsequent group "filters" or modifies the previous one:

**Base Metrics (The Constant):** As outlined in your the section, we focus here on qualities that never change, regardless of time or location. This includes _how_ the attack happens (Exploitability) and _what_ it breaks (Impact).

**Temporal Metrics (The Current State):** We use this group to adjust the Base score based on factors that change over time. For example, if a "Critical" Base score exists but no functional exploit code has been released yet, the Temporal score will lower the overall severity.

**Environmental Metrics (The Context):** This is where we tailor the score to our specific target. If a vulnerability has a high Impact score but the target system is isolated from the network or has mitigating controls, we use Environmental metrics to "dial down" the score to reflect the actual risk to the organisation.
