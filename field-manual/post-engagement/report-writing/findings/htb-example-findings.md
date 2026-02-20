# HTB Example Findings

Below are a few example findings. The first two are examples of issues that may be discovered during an Internal Penetration Test. As you can see, each finding includes all of the key elements: a detailed description to explain what is going on, the impact to the environment if the finding is left unfixed, the hosts affected by the issue (or the entire domain), remediation advice that is generic, does not recommend specific vendor tools and gives several options for remediation. Finally, the reference links are from well-known, reputable sources that will likely not be taken down anytime soon as a personal blog may.

A note on the formatting: This could potentially be a hotly contested topic. The example findings here have been laid out in a tabular format, but if you've ever worked in Word or tried to automate some of your report generation, you know that tables can be a nightmare to deal with. For this reason, others opt to separate sections of their findings with different heading levels. Either of these approaches is acceptable because what is important is whether your message gets across to the reader and how easy it is to pick out the visual cues for when one finding ends and another begins; readability is paramount. If you can accomplish this, colors, layout, order, and even section names can be adjusted.

### Weak Kerberos Authentication (“Kerberoasting”)

![Table detailing a high-risk Kerberos authentication issue, CVSS score 9.5, affecting INLANEFREIGHT.LOCAL, with remediation steps like enabling AES encryption.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/162/kbroast.png)

### Tomcat Manager Weak/Default Credentials

![Table detailing a high-risk Tomcat Manager issue with weak credentials, CVSS score 9.5, affecting host 192.168.195.205, with remediation steps like restricting access and changing default passwords.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/162/tomcat_finding.png)

### Poorly Written Finding

Below is an example of a poorly written finding which has several issues:

* Formatting is sloppy with the CWE link
* No CVSS score is filled in (not mandatory, but if your report template uses it, you should fill it in)
* The Description does not clearly explain the issue or root cause
* The security impact is vague and generic
* The Remediation section is not clear and actionable

If I am reading this report, I can see that this finding is bad (because it's red), but why do I care? What do I do about it? Each finding should present the issue in detail and educate the reader on the issue at hand (they may very likely have never heard of Kerberoasting or some other attack). Clearly articulate the security risk and `why` this needs to be remediated and some actionable remediation recommendations.

![Table detailing high-risk Kerberoasting issue, CVSS score 9.5, affecting INLANEFREIGHT.LOCAL, with remediation steps like deleting SPN accounts and using gMSA.](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/162/kbroast_weak.png)

<br>
