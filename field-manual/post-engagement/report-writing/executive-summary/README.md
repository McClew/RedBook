---
icon: user-tie
---

# Executive Summary

The Executive Summary is one of the most important parts of the report. Clients are ultimately paying for the report deliverable which has several purposes aside from showing weaknesses and reproduction steps that can be used by technical teams working on remediation. The report will likely be viewed in some part by other internal stakeholders such as Internal Audit, IT and IT Security management, C-level management, and even the Board of Directors.&#x20;

The report may be used to either validate funding from the prior year for infosec or to request additional funding for the following year. For this reason, we need to ensure that there is content in the report that can be easily understood by people without technical knowledge.

## Anatomy

Firstly, we should get a list of the findings together and try categorising the nature of the risk of each one. These categories will be the foundation for what you're going to discuss in the executive summary. In the [htb-example-report.md](htb-example-report.md "mention"), we have the following findings:

* LLMNR/NBT-NS Response Spoofing - `configuration change/system hardening`
* Weak Kerberos Authentication (“Kerberoasting”) - `configuration change/system hardening`
* Local Administrator Password Re-Use - `behavioral/system hardening`
* Weak Active Directory Passwords - `behavioral`
* Tomcat Manager Weak/Default Credentials High - `configuration change/system hardening`
* Insecure File Shares - `configuration change/system hardening/permissions`
* Directory Listing Enabled - `configuration change/system hardening`
* Enhance Security Monitoring Capabilities - `configuration change/system hardening`

{% hint style="info" %}
#### Recognise the Good Stuff

It's notable that there aren't any issues in this list linked to missing patches, indicating that the client may have spent considerable time and effort maturing that process. This is no small feat, so we want to make sure to recognise their efforts. This endears you to the sysadmin team by showing their executives that the work they've been doing has been effective, and it encourages the executives to continue to invest in people and technology that can help correct some of their issues.
{% endhint %}

Back to our findings, you can see nearly every finding has some sort of configuration change or system hardening resolution. To collapse it even further, you could start to conclude that this particular client has an immature configuration management process (i.e., they don't do a very good job of changing default configurations on anything before placing it into production).

Since there is a lot to unpack in eight findings, you probably don't want to just write a paragraph that says "configure things better ."You have some real estate to get into some individual issues and describe some of the impact (the attention-grabbing stuff) of some of the more damaging findings. Developing a configuration management process will take a lot of work, so it's important to describe what did or could happen if this issue remains unchecked.

As you read each paragraph, you'll probably be able to map the high-level description to the associated finding to give you some idea of how to describe some of the more technical terms in a way that a non-technical audience can follow without having to look things up. You'll notice that we do not use acronyms, talk about protocols, mention tickets that grant other tickets, or anything like that. In a few cases, we also describe general anecdotes about what level of effort to expect from remediation, changes that should be made cautiously, workarounds to monitor for a given threat, and the skill level required to perform exploitation. You do NOT have to have a paragraph for every finding. If you have a report with 20 findings, that would get out of control quickly. Try to focus on the most impactful ones.

A couple of nuances to mention as well:

* Certain observations you make during the assessment can indicate a more significant issue the client may not be aware of. It's obviously valuable to provide this analysis, but you must be careful how it's worded to ensure you are not speaking in absolutes because of an assumption.
* At the end, you'll notice a paragraph about how it **seems like** and **indicated that** the client did not detect our testing activity. These qualifiers are important because you aren't absolutely sure they didn't. They may have just not told you they did.
* Another example of this (in general, not in this executive summary) would be if you wrote something to the effect of "begin documenting system hardening templates and processes ." This insinuates that they have done nothing, which could be insulting if they actually tried and failed. Instead, you might say, "review configuration management processes and address the gaps that led to the issues identified in this report."

## Accessibility

The intended audience for the Executive Summary is typically the person that is going to be responsible for allocating the budget to fixing the issues we discovered. For better or worse, some of our clients have likely been trying to get funding to fix the issues presented in the report for years and fully intend to use the report as ammunition to finally get some stuff done.

This is our best chance to help them out. If we lose our audience here and there are budgetary limitations, the rest of the report can quickly become worthless. Some key things to assume (that may or may not be true) to maximize the effectiveness of the Executive Summary are:

* It should be obvious, but this should be written for someone who isn't technical at all. The typical barometer for this is "if your parents can't understand what the point is, then you need to try again" (assuming your parents aren't CISOs or sysadmins or something of the sort).
* The reader doesn't do this every day. They don't know what Rubeus does, what password spraying means, or how it's possible that tickets can grant different tickets (or likely even what a ticket is, aside from a piece of paper to enter a concert or a ballgame).
* This may be the first time they've ever been through a penetration test.
* Much like the rest of the world in the instant gratification age, their attention span is small. When we lose it, we are extraordinarily unlikely to get it back.
* Along the same lines, no one likes to read something where they have to Google what things mean. Those are called distractions.

## Key Concepts

### Do

* **`When talking about metrics, be as specific as possible.`**\
  Words like "several," "multiple," and "few" are ambiguous and could mean 6 or 500. Executives aren't going to dig through the report for this information, so if you're going to talk about this, let them know what you've got; otherwise, you're going to lose their attention. The most common reason people do not commit to a specific number is to leave it open in case the consultant missed one. You can make minor changes to the language to account for this, such as "while there may be additional instances of X, in the time allotted to the assessment, we observed 25 occurrences of X".
* **`It's a summary. Keep it that way.`**\
  If you wrote more than 1.5-2 pages, you've probably been too verbose. Examine the topics you talked about and determine whether they can be collapsed into higher-level categories that might fall into specific policies or procedures.
* **`Describe the types of things you managed to access.`**\
  Your audience may not have any idea what "Domain Admin" means, but if you mention that you gained access to an account that enabled you to get your hands on HR documents, banking systems, and other critical assets, that's universally understandable.
* **`Describe the general things that need to improve to mitigate the risks you discovered.`**\
  This should not be "install 3 patches and call me in a year". You should be thinking in terms of "what process broke down that enabled a five-year-old vulnerability to go unpatched in a quarter of the environment?". If you password spray and get 500 hits on Welcome1!, changing the passwords of those 500 accounts is only part of the solution. The other part is probably providing the Help Desk with a way to set stronger initial passwords efficiently.
* **`If you're feeling brave and have a decent amount of experience on both sides, provide a general expectation for how much effort will be necessary to fix some of this.`**\
  If you have a long past as a sysadmin or engineer and you know how much internal politics people may have to wade through to start manipulating group policies, you may want to try and set an expectation for low, moderate, and significant levels of time and effort to correct the issues, so an overzealous CEO doesn't go tell his server team they need to apply CIS hardening templates to their GPOs over the weekend without testing them first.

### **Do Not**

* **`Name or recommend specific vendors.`** \
  The deliverable is a technical document, not a sales document. It's acceptable to suggest technologies such as EDR or log aggregation but stay away from recommending specific vendors of those technologies, like CrowdStrike and Splunk. If you have experience with a particular vendor that is recent and you feel comfortable giving the client that feedback, do so out-of-band and make sure that you're clear that they should make their own decision (and probably bring the client's account executive into that discussion). If you're describing specific vulnerabilities, your reader is more likely to recognize something like "vendors like VMWare, Apache, and Adobe" instead of "vSphere, Tomcat, and Acrobat."
* **`Use Acronyms.`** \
  IP and VPN have reached a level of ubiquity that they're maybe okay, but using acronyms for protocols and types of attacks (e.g., SNMP, MitM) is tone-deaf and will render your executive summary completely ineffective for its intended audience.
* **`Spend more time talking about stuff that doesn't matter than you do about the significant findings in the report.`** \
  It is within your power to steer attention. Don't waste it on the issues you discovered that weren't that impactful.
* **`Use words that no one has ever heard of before.`** \
  Having a large vocabulary is great, but if no one can understand the point you're trying to make or they have to look up what words mean, all they are is a distraction. Show that off somewhere else.
* **`Reference a more technical section of the report.`** \
  The reason the executive is reading this might be because they don't understand the technical details, or they may decide they just don't have time for it. Also, no one likes having to scroll back and forth throughout the report to figure out what's going on.

