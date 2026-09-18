---
icon: user-secret
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

# HUMINT

Human Intelligence (HUMINT) covers the techniques used to extract information or access from people rather than systems - pretexting, phishing, vishing, and physical social engineering. It's usually the most effective attack path against a well-hardened target, and consistently the one clients underestimate. Where the rest of the Intelligence phase focuses on machines and infrastructure, HUMINT targets the humans operating them.

***

## OSINT-to-HUMINT Handoff

HUMINT rarely starts cold. It builds directly on what's already been gathered during [osint](osint/ "mention") and Identity Hunting - names, roles, reporting lines, and the small details that make a pretext believable.

* Build a target priority list from the org chart: helpdesk, reception, and executive assistants tend to be both reachable and high-value.
* Pull employee schedules, out-of-office notices, and internal jargon from LinkedIn, job postings, and any leaked internal documents to season a pretext with detail that survives scrutiny.
* Cross-reference technology stack findings from earlier enumeration (VPN vendor, ticketing system, VoIP provider) — knowing what a target's helpdesk actually uses makes a vishing pretext far more convincing.

***

## Pretexting Fundamentals

A pretext is only as strong as its weakest detail. The goal is a persona that survives a few follow-up questions, not a bulletproof legend.

### Building the Persona

* <mark style="color:$primary;">**Role:**</mark> pick something with a plausible reason to contact the target (IT support, a vendor, an auditor, a new starter).
* <mark style="color:$primary;">**Urgency:**</mark> a believable reason the target should act now rather than escalate or verify later.
* <mark style="color:$primary;">**Consistency:**</mark> matching name, department, and story across every channel used (email, phone, any documents referenced).

{% hint style="warning" %}
**Common Failure Points**

Inconsistent details (wrong extension format, wrong internal tool name), unfamiliarity with company jargon, and generic accents/scripts that don't match the target's actual support desk are the most common ways a pretext collapses.
{% endhint %}

### Escalation Path

Always have a fallback story and a graceful way to end the interaction if challenged. Pushing harder after being caught out damages the engagement and the client relationship - it's better to disengage cleanly and note the detection as a positive finding.

***

## Vishing & Phishing Playbooks

### Vishing

Common pretexts and rough call structures:

<table><thead><tr><th width="200">Pretext</th><th>Typical Ask</th></tr></thead><tbody><tr><td>IT Support / Password Reset</td><td>Get the target to read back or reset a credential, or install "support" software</td></tr><tr><td>Vendor Callback</td><td>Confirm account details or redirect a payment/process</td></tr><tr><td>Executive Assistant</td><td>Urgent request "on behalf of" a senior exec, often bypassing normal process</td></tr></tbody></table>

### Phishing

* Templates: fake invoice, HR policy update, IT outage notice, calendar invite with a malicious link/attachment.
* Track click rate, credentials submitted, and - just as important - the rate at which the attempt was reported, since that's a genuine positive signal for the client's awareness programme.

### Smishing

SMS pretexts (delivery notification, MFA prompt, "IT" text) with a shortened URL. Note that mobile carriers and shortened links behave differently to email - test delivery before relying on it during a time-boxed engagement.

{% hint style="info" %}
**Note**

Any credential-harvesting page or tracking pixel used here should follow the same evidence-handling and storage discipline as the rest of the assessment - see Note Taking.
{% endhint %}

***

## Physical Social Engineering

On-site techniques for gaining physical access or information.

* <mark style="color:$primary;">**Tailgating:**</mark> following an employee through a badge-controlled door; confident body language and carrying something (a coffee, a box) that occupies your hands does more than any cover story.
* <mark style="color:$primary;">**Badge Cloning:**</mark> cloning an RFID badge captured via a concealed reader; requires physical proximity to a legitimate badge holder.
* <mark style="color:$primary;">**Pretexts for On-Site Access:**</mark> delivery driver, contractor, fire warden inspection, or a "new starter" waiting for IT to sort their access.
* <mark style="color:$primary;">**Dumpster Diving:**</mark> discarded documents, hardware, or media in unsecured waste areas can reveal credentials, org structure, or network diagrams.
* <mark style="color:$primary;">**Shoulder Surfing / USB Drops:**</mark> opportunistic credential capture, or a labelled USB drive left somewhere an employee is likely to pick it up and plug it in.

{% hint style="danger" %}
**Authorisation**

Physical engagements carry real legal exposure. Nothing in this section should be attempted without explicit written authorisation covering the specific techniques used — see Legal & RoE below.
{% endhint %}

***

## Legal, Ethical & RoE Considerations

This is the section most likely to go wrong if skipped.

* <mark style="color:$primary;">**Written Authorisation:**</mark> the Rules of Engagement must explicitly cover any physical or social engineering activity planned, including specific techniques (tailgating, badge cloning, on-site pretexting).
* <mark style="color:$primary;">**Get-Out-Of-Jail Letter:**</mark> carry it during any on-site engagement, and know the process for using it if challenged or detained by staff or law enforcement.
* <mark style="color:$primary;">**Boundaries:**</mark> impersonating law enforcement, emergency services, or making personal threats is off-limits regardless of what the RoE technically permits.
* <mark style="color:$primary;">**Handling Detection Well:**</mark> if an employee correctly identifies and reports the attempt, that's a client success story, not a failed test. Disengage politely and make sure it's reflected positively in the report.
* <mark style="color:$primary;">**Local Laws:**</mark> trespass, impersonation, and recording-consent laws vary by jurisdiction; check before an on-site engagement, particularly for multi-site or multi-country scopes.

***

## Reporting & Evidence

* Capture timestamps, and recordings/transcripts where permitted by the RoE, for every attempt - not just the successful ones.
* Use a metrics table for phishing/vishing campaigns (sent, clicked, submitted credentials, reported) to give the client a clear before/after baseline for awareness training.
* Redact employee names and PII per the client's preference - confirm this during scoping, and apply the same redaction standard used elsewhere in the report (see Note Taking).
* Frame recommendations around the client's awareness programme rather than individual employees - the finding is a process gap, not a person's mistake.
