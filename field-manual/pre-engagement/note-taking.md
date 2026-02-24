---
icon: memo-circle-info
---

# Note Taking

## HackTheBox Sample Structure

* `Attack Path` - An outline of the entire path if you gain a foothold during an external penetration test or compromise one or more hosts (or the AD domain) during an internal penetration test. Outline the path as closely as possible using screenshots and command output will make it easier to paste into the report later and only need to worry about formatting.
* `Credentials` - A centralized place to keep your compromised credentials and secrets as you go along.
* `Findings` - We recommend creating a subfolder for each finding and then writing our narrative and saving it in the folder along with any evidence (screenshots, command output). It is also worth keeping a section in your notetaking tool for recording findings information to help organize them for the report.
* `Vulnerability Scan Research` - A section to take notes on things you've researched and tried with your vulnerability scans (so you don't end up redoing work you already did).
* `Service Enumeration Research` - A section to take notes on which services you've investigated, failed exploitation attempts, promising vulnerabilities/misconfigurations, etc.
* `Web Application Research` - A section to note down interesting web applications found through various methods, such as subdomain brute-forcing. It's always good to perform thorough subdomain enumeration externally, scan for common web ports on internal assessments, and run a tool such as Aquatone or EyeWitness to screenshot all applications. As you review the screenshot report, note down applications of interest, common/default credential pairs you tried, etc.
* `AD Enumeration Research` - A section for showing, step-by-step, what Active Directory enumeration you've already performed. Note down any areas of interest you need to run down later in the assessment.
* `OSINT` - A section to keep track of interesting information you've collected via OSINT, if applicable to the engagement.
* `Administrative Information` - Some people may find it helpful to have a centralized location to store contact information for other project stakeholders like Project Managers (PMs) or client Points of Contact (POCs), unique objectives/flags defined in the Rules of Engagement (RoE), and other items that you find yourself often referencing throughout the project. It can also be used as a running to-do list. As ideas pop up for testing that you need to perform or want to try but don't have time for, be diligent about writing them down here so you can come back to them later.
* `Scoping Information` - Here, we can store information about in-scope IP addresses/CIDR ranges, web application URLs, and any credentials for web applications, VPN, or AD provided by the client. It could also include anything else pertinent to the scope of the assessment so we don't have to keep re-opening scope information and ensure that we don't stray from the scope of the assessment.
* `Activity Log` - High-level tracking of everything you did during the assessment for possible event correlation.
* `Payload Log` - Similar to the activity log, tracking the payloads you're using (and a file hash for anything uploaded and the upload location) in a client environment is critical. More on this later.

## Command Logging

[Tmux logging](https://github.com/tmux-plugins/tmux-logging) is an excellent choice for terminal logging, and we should absolutely be using `Tmux` along with logging as this will save every single thing that we type into a Tmux pane to a log file. It is also essential to keep track of exploitation attempts in case the client needs to correlate events later on (or in a situation where there are very few findings and they have questions about the work performed).

For more information read [tmux-logging.md](../../toolbox/tooling/utilities/tmux-logging.md "mention").

## Artifacts

### Payloads & File Uploads

At a minimum, we should be tracking when a payload was used, which host it was used on, what file path it was placed in on the target, and whether it was cleaned up or needs to be cleaned up by the client. A file hash is also recommended for ease of searching on the client's part. It's best practice to provide this information even if we delete any web shells, payloads, or tools.

### Account Creation & System Modifications

If we create accounts or modify system settings, it should be evident that we need to track those things in case we cannot revert them once the assessment is complete. Some examples of this include:

* IP address of the host(s)/hostname(s) where the change was made
* Timestamp of the change
* Description of the change
* Location on the host(s) where the change was made
* Name of the application or service that was tampered with
* Name of the account (if you created one) and perhaps the password in case you are required to surrender it

It should go without saying, but as a professional and to prevent creating enemies out of the infrastructure team, you should get written approval from the client before making these types of system modifications or doing any sort of testing that might cause an issue with system stability or availability. This can typically be ironed out during the project kickoff call to determine the threshold beyond which the client is willing to tolerate without being notified.

## Evidence

Clients are paying for the report deliverable, which should clearly communicate the issues discovered and evidence that can be used for validation and reproduction. Without clear evidence, it can be challenging for internal security teams, sysadmins, devs, etc., to reproduce our work while working to implement a fix or even to understand the nature of the issue.

### **What to Capture**

As we know, each finding will need to have evidence. It may also be prudent to collect evidence of tests that were performed that were unsuccessful in case the client questions your thoroughness.

If you're working on the command line, Tmux logs may be sufficient evidence to paste into the report as literal terminal output, but they can be horribly formatted. For this reason, capturing your terminal output for significant steps as you go along and tracking that separately alongside your findings is a good idea. For everything else, screenshots should be taken.

### Storage

Much like with our notetaking structure, it's a good idea to come up with a framework for how we organise the data collected during an assessment. This may seem like overkill on smaller assessments, but if we're testing in a large environment and don't have a structured way to keep track of things, we're going to end up forgetting something, violating the rules of engagement, and probably doing things more than once which can be a huge time waster, especially during a time-boxed assessment.

Below is a suggested baseline folder structure, but you may need to adapt it accordingly depending on the type of assessment you're performing or unique circumstances.

* `Admin`
  * Scope of Work (SoW) that you're working off of, your notes from the project kickoff meeting, status reports, vulnerability notifications, etc
* `Deliverables`
  * Folder for keeping your deliverables as you work through them. This will often be your report but can include other items such as supplemental spreadsheets and slide decks, depending on the specific client requirements.
* `Evidence`
  * Findings
    * We suggest creating a folder for each finding you plan to include in the report to keep your evidence for each finding in a container to make piecing the walkthrough together easier when you write the report.
  * Scans
    * Vulnerability scans
      * Export files from your vulnerability scanner (if applicable for the assessment type) for archiving.
    * Service Enumeration
      * Export files from tools you use to enumerate services in the target environment like Nmap, Masscan, Rumble, etc.
    * Web
      * Export files for tools such as ZAP or Burp state files, EyeWitness, Aquatone, etc.
    * AD Enumeration
      * JSON files from BloodHound, CSV files generated from PowerView or ADRecon, Ping Castle data, Snaffler log files, CrackMapExec logs, data from Impacket tools, etc.
  * Notes
    * A folder to keep your notes in.
  * OSINT
    * Any OSINT output from tools like Intelx and Maltego that doesn't fit well in your notes document.
  * Wireless
    * Optional if wireless testing is in scope, you can use this folder for output from wireless testing tools.
  * Logging output
    * Logging output from Tmux, Metasploit, and any other log output that does not fit the `Scan` subdirectories listed above.
  * Misc Files
    * Web shells, payloads, custom scripts, and any other files generated during the assessment that are relevant to the project.
* `Retest`
  * This is an optional folder if you need to return after the original assessment and retest the previously discovered findings. You may want to replicate the folder structure you used during the initial assessment in this directory to keep your retest evidence separate from your original evidence.

### What Not to Archive

When starting a penetration test, we are being trusted by our customers to enter their network and "do no harm" wherever possible. This means not bringing down any hosts or affecting the availability of applications or resources, not changing passwords (unless explicitly permitted), making significant or difficult-to-reverse configuration changes, or viewing or removing certain types of data from the environment.

This data may include unredacted PII, potentially criminal info, anything considered legally "discoverable," etc. For example, if you gain access to a network share with sensitive data, it's probably best to just screenshot the directory with the files in it rather than opening individual files and screenshotting the file contents. If the files are as sensitive as you think, they'll get the message and know what's in them based on the file name.

Collecting actual PII and extracting it from the target environment may have significant compliance obligations for storing and processing that data like GDPR and the like and could open up a slew of issues for our company and us.

## Automation

It's a good idea to have scripts and tricks for setting up at the beginning of an assessment. We could take the following command to make our directories and subdirectories and adapt it further.

{% code overflow="wrap" %}
```bash
mkdir -p ACME-IPT/{Admin,Deliverables,Evidence/{Findings,Scans/{Vuln,Service,Web,'AD Enumeration'},Notes,OSINT,Wireless,'Logging output','Misc Files'},Retest}
```
{% endcode %}

```shell-session
ACME-IPT/
├── Admin
├── Deliverables
├── Evidence
│   ├── Findings
│   ├── Logging output
│   ├── Misc Files
│   ├── Notes
│   ├── OSINT
│   ├── Scans
│   │   ├── AD Enumeration
│   │   ├── Service
│   │   ├── Vuln
│   │   └── Web
│   └── Wireless
└── Retest
```

## Formatting and Redaction

Credentials and Personal Identifiable Information (PII) should be redacted in screenshots and anything that would be morally objectionable, like graphic material or perhaps obscene comments and language. You may also consider the following:

* Adding annotations to the image like arrows or boxes to draw attention to the important items in the screenshot, particularly if a lot is happening in the image (don't do this in MS Word).
* Adding a minimal border around the image to make it stand out against the white background of the document.
* Cropping the image to only display the relevant information (e.g., instead of a full-screen capture, just to show a basic login form).
* Include the address bar in the browser or some other information indicating what URL or host you're connected to.

### Terminal Output Formating

Wherever possible, we should try to use terminal output over screenshots of the terminal. It is easier to redact, highlight the important parts (i.e., the command we ran in blue text and the part of the output we want to call attention to in red), typically looks neater in the document, and can avoid the document from becoming a massive, unwieldy file if we have loads of findings.

We should be careful not to alter terminal output since we want to give an exact representation of the command we ran and the result. It is OK to shorten/cut out unnecessary output and mark the removed portion with `<SNIP>` but never alter output or add things that were not in the original command or output.

Using text-based figures also makes it easier for the client to copy/paste to reproduce your results. It's also important that the source material that you're pasting _from_ has all formatting stripped before going into your Word document. If you're pasting text that has embedded formatting, you may end up pasting non-UTF-8 encoded characters into your commands (usually alternate quotes or apostrophes), which may actually cause the command to not work correctly when the client tries to reproduce it.

Finally, here is a suggested way to present terminal evidence in a report document. Here we have preserved the original command and output but enhanced it to highlight both the command and the output of interest (successful authentication).

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Terminal Redactions

Typically the only thing that needs to be redacted from terminal output is credentials (whether in the command itself or the output of the command). This includes password hashes.&#x20;

For password hashes, you can usually just strip out the middle of them and leave the first and last 3 or 4 characters to show there was actually a hash there.

For cleartext credentials or any other human-readable content that needs to be obfuscated, you can just replace it with a `<REDACTED>` or `<PASSWORD REDACTED>` placeholder, or similar.

You should also consider color-coded highlighting in your terminal output to highlight the command that was run and the interesting output from running that command. This enhances the reader's ability to identify the essential parts of the evidence and what to look for if they try to reproduce it on their own.&#x20;

If you're working on a complex web payload, it can be difficult to pick out the payload in a gigantic URL-encoded request wall of text if you don't do this for a living. We should take all opportunities to make the report clearer to our readers, who will often not have as deep an understanding of the environment (especially from the perspective of a penetration tester) as we do by the end of the assessment.

### Screenshot Redactions

One common way of redacting screenshots is through pixelation or blurring using a tool such as Greenshot. [Research](https://www.bleepingcomputer.com/news/security/researcher-reverses-redaction-extracts-words-from-pixelated-image/) has shown that this method is not foolproof, and there's a high likelihood that the original data could be recovered by reversing the pixelation/blurring technique.&#x20;

This can be done with a tool such as [Unredacter](https://github.com/bishopfox/unredacter). Instead, we should avoid this technique and use black bars (or another solid shape) over the text we would like to redact. We should edit the image directly and not just apply a shape in MS Word, as someone with access to the document could easily delete this.&#x20;

As an aside, if you are writing a blog post or something published on the web with redacted sensitive data, do not rely on HTML/CSS styling to attempt to obscure the text (i.e., black text with a black background) as this can easily be viewed by highlighting the text or editing the page source temporarily. When in doubt, use console output but if you must use a terminal screenshot, then make sure you are appropriately redacting information. Below are examples of the two techniques:

#### Blurring Password Data

![](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/162/blurred.png)

#### Blanking Out Password with Solid Shape

![](https://cdn.services-k8s.prod.aws.htb.systems/content/modules/162/boxes.png)
