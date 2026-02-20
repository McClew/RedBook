---
icon: link
---

# Assessment Narrative

## Attack Chain

The attack chain is our chance to show off the cool exploitation chain we took to gain a foothold, move laterally, and compromise the domain. It can be a helpful mechanism to help the reader connect the dots when multiple findings are used in conjunction with each other and gain a better understanding of why certain findings are given the severity rating that they are assigned.

For example, a particular finding on its own may be `medium-risk` but, combined with one or two other issues, could elevate it to `high-risk`, and this section is our chance to demonstrate that. A common example is using `Responder` to intercept `NBT-NS/LLMNR` traffic and relaying it to hosts where SMB signing is not present. It can get really interesting if some findings can be incorporated that might otherwise seem inconsequential, like using an information disclosure of some sort to help guide you through an LFI to read an interesting configuration file, log in to an external-facing application, and leverage functionality to gain remote code execution and a foothold inside the internal network.

There are multiple ways to present this, and your style may differ but let's walk through an example. We will start with a summary of the attack chain and then walk through each step along with supporting command output and screenshots to show the attack chain as clearly as possible.

***

## Professional Requirements

### The Two-Part Structure

For complex attack chains, implement a tiered approach. Start with a Walkthrough Summary, providing a high-level numbered list of the attack path for a quick overview, followed by the Detailed Walkthrough where each step is explained in depth.

### Third-Person Narrative

To maintain professional distance and match commercial standards, always write in the third person. Refer to yourself as "the tester" or use the placeholder report.candidate.name. Avoid first-person speech like "I did," which is a common reason for report failure.

### Segment by Objectives/Domains

If the narrative becomes complex (e.g., rooting multiple domains), segment the walkthrough explicitly. Cover all steps for the first domain or phase entirely before starting the narrative for the next. This reflects the shifting nature of intelligence gathering as you move from external reconnaissance to post-exploitation reconnaissance.

### The "Why" Behind the "How"

Beyond listing steps, document the exact sequence of thoughts. Explain what specific clue led you to a tool or technique, as this helps the client understand the "logic gap" an attacker would exploit.

### Reproducibility Requirement

The gold standard for a technical walkthrough is that a defender should be able to replicate the entire attack based solely on your documentation. This requires capturing tool configurations (e.g., Metasploit options) before execution and showing the resulting output in a separate figure.

***

## Tell a Story

Each vulnerability that we investigate should have a story attached to it. We should attempt to answer the following questions for each vulnerability investigated.

* What drew you to this element (vulnerability scanning results, an unidentified listening service, an uncommon open port)?
* What was the vulnerability?
* What impact does it have on the organisation?
* Did you attempt to exploit it?
* How difficult was it to construct an exploit?
* What was the result of the exploit attempt?
* Can further exploitation be attained?
* What can the organisation do to remediate the issue?
* What can the organisation do to mitigate the issue?

Reporting usually has the competing priorities of full coverage and brevity. However, your report should include details on failed attempts at exploitation as much as successful ones. This shows the target organisation where defenses are working and helps them to understand your approach to executing the test.
