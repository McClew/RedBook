---
icon: pen-to-square
---

# Report Writing

## Structure

1. Statement of Confidentiality
2. Engagement Contacts (Customer and Assessor info)
3. [executive-summary](executive-summary/ "mention")
   1. Approach
   2. Scope
   3. Assessment & [recommendations-summary.md](recommendations-summary.md "mention")
4. Walkthrough Summary
   1. Summary
   2. [visuals-for-executives.md](executive-summary/visuals-for-executives.md "mention")
5. Technical Walkthrough ( [assessment-narrative.md](assessment-narrative.md "mention") )
   1. Phase 1: \<Phase name such as "Initial Access">
      1. \<Action/goal orientated section>
      2. \<Example: "Initial Port Scan">
   2. Phase 2: \<Phase name such as "Internal Disovery">
      1. \<Action/goal orientated section>
6. [findings](findings/ "mention")
7. [appendices.md](appendices.md "mention")

## Prioritisation

During an assessment, especially large ones, we'll be faced with a lot of "noise" that we need to filter out to best focus our efforts and prioritize findings. As testers, we are required to disclose everything we find, but when there is a ton of information coming at us through scans and enumeration, it is easy to get lost or focus on the wrong things and waste time and potentially miss high-impact issues.&#x20;

This is why it is essential that we understand the output that our tools produce, have repeatable steps (such as scripts or other tools) to sift through all of this data, process it, and remove false positives or informational issues that could distract us from the goal of the assessment.&#x20;

Experience and a repeatable process are key so that we can sift through all of our data and focus our efforts on high-impact findings such as remote code execution (RCE) flaws or others that may lead to sensitive data disclosure. It is worth it (and our duty) to report informational findings, but instead of spending the majority of our time validating these minor, non-exploitable issues, you may want to consider consolidating some of them into categories that show the client you were aware that the issues existed, but you were unable to exploit them in any meaningful way (e.g., 35 different variations of problems with SSL/TLS, a ton of DoS vulnerabilities in an EOL version of PHP, etc.).

When starting in penetration testing, it can be difficult to know what to prioritise, and we may fall down rabbit holes trying to exploit a flaw that doesn't exist or getting a broken PoC exploit to work. Time and experience help here, but we should also lean on senior team members and mentors to help. Something that you may waste half a day on could be something that they have seen many times and could tell you quickly whether it is a false positive or worth running down. Even if they can't give you a really quick black and white answer, they can at least point you in a direction that saves you several hours. Surround yourself with people you're comfortable with asking for help that won't make you feel like an idiot if you don't know all the answers.

## HTB Examples

{% file src="../../../.gitbook/assets/Inlanefreight_IPT_DRAFT (1).pdf" %}

{% file src="../../../.gitbook/assets/Inlanefreight_IPT_DRAFT.docx" %}

## Additional Resources

{% embed url="https://www.blackhillsinfosec.com/how-to-not-suck-at-reporting-or-how-to-write-great-pentesting-reports/" %}
