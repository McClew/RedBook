# CPTS Exam Tactics & Survival Guide

This article provides essential tips and tactical advice for navigating the Hack The Box Certified Penetration Testing Specialist (CPTS) examination, focusing on in-exam execution, overcoming roadblocks, and securing flags.

***

## Flag Strategy & Roadblocks

The CPTS exam contains specific flags that test not only technical skill but also psychological endurance and methodical thinking.

### Specific Flag Challenges

<table><thead><tr><th width="128">Flag</th><th width="161">Challenge Type</th><th>Tactical Advice</th></tr></thead><tbody><tr><td>Flag 1</td><td>The Cold Start Filter</td><td>This flag often stops candidates cold, as there are many weak signals but nothing immediately works. Prioritise weak signals and be willing to go deeper rather than broader. Avoid spiraling by revisiting and challenging initial assumptions.</td></tr><tr><td>Flag 3</td><td>The Puzzle</td><td>This flag challenges you to transform an initial foothold into lateral movement. The solution usually involves a subtle privilege escalation vector, abusing a hidden share, or reusing credentials on an unexpected protocol.</td></tr><tr><td>Flag 9</td><td>The Mental Breaker</td><td>Considered the exam's "boss fight," this flag is often solved under immense fatigue and tunnel vision. If you reach this point, you are technically capable, but your brain may be fogged, requiring mental reset routines to regain clarity.</td></tr><tr><td>Flags 13 &#x26; 14</td><td>The Final Stretch</td><td>Once you capture 12 flags, pause and secure your report before continuing. There is no extra credit for getting all flags if the final report is weak. Only consider Flags 13 and 14 once the documentation for the first 12 is rock-solid.</td></tr></tbody></table>

***

## Methodology When Stuck

Getting stuck is an expected part of the CPTS exam, which operates as a psychological endurance test. Use the following strategies to reset your thinking and workflow.

### The "Think Dumber" Principle

Resist the urge to over-engineer complex chains or assume everything is hardened. Often, the solution is simple (or "dumb"), such as a plaintext password in a configuration file, an overlooked share, or a forgotten virtual host.

Before escalating complexity, test the obvious first:

* Re-use credentials on every exposed service, including RDP or WinRM.
* Check for anonymous SMB access.
* Look for credentials in files like `unattend.xml`, `.git`, `.env`, or configuration files like `settings.php`.
* Ask: What is the low-hanging fruit a junior admin might mess up?

### Tactical Reorientation Steps

When hitting a cognitive bottleneck, utilise this "Break Glass in Case of Panic" strategy bank.

1. **Re-enumerate Everything:** If you gained a new pivot or access, immediately redo your entire reconnaissance process from that new vantage point.
2. **Verbal Processing:** Read your notes out loud to yourself; this can activate your verbal processing and trigger "wait a second..." moments.
3. **Visualise the Terrain:** Sketch the attack surface on paper, including VMs, users, IP ranges, and trust paths, to spot missing links.
4. **Search the Academy:** If you cannot recall a solution, search the HTB Academy path using keywords. The answer is often contained within the path.
5. **Mental Reset:** Turn off the monitor, lie down, or walk through the flow mentally without distractions. Taking a shower is also a recommended way to reset stuck thinking.

### Using AI Assistance Tactically

While tools like ChatGPT can be used, treating them like a well-meaning but unreliable junior analyst is essential.

| Use AI Effectively (Life-Saver)                                          | Avoid Blind Trust (Sabotage)                                                                                          |
| ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| Use it as a syntax helper (e.g., correct PowerView cmdlet).              | The model can (and will) hallucinate fictional command options or wrong outputs for tools like `NetExec` or `Rubeus`. |
| Use it as a snippet generator (e.g., Python socket reverse shell).       | Do not use it to generate critical attack steps.                                                                      |
| Use it as a workflow explainer (e.g., standard way to use `ldapsearch`). | If it dismisses a technique, verify it with a second source before abandoning your idea.                              |

***

## Tactical Technical & Workflow Advice

### Information Gathering Discipline

Information gathering is an iterative process that must evolve as you gain new credentials or network visibility.

* **Services Enumeration:** Start with fast sweeps (`nmap -T4 --top-ports 1000`), then expand. Do not blindly run comprehensive scans like `-A` or `-p-` on everything, as this can be noisy and wasteful. Remember that TCP and UDP are both in play; do not neglect UDP services like SNMP or TFTP.
* **Web Enumeration:** Begin with Passive Web Recon before employing noisy techniques like directory and page fuzzing.
* **Active Directory Enumeration:** Start enumeration by understanding the domain structure (name, FQDN). Create AD Enumeration Trees for each compromised box to track commands run, edges found, and validation. Always confirm edges found by BloodHound with manual checks using tools like PowerView or NetExec, as BloodHound graphs can sometimes be misleading.

### Post-Exploitation and Pillaging

* Stabilise the Shell: Before attempting to escalate privileges or move laterally, always stabilise your shell using commands like `python -c 'import pty; pty.spawn("/bin/bash")'`. Broken shells lead to missed information and dead-end escalation paths.
* Manual Recon: Do not run automated tools like WinPEAS or linPEAS first. Conduct manual exploration and targeted searches first, as this often yields more findings and enhances understanding.
* Pillaging: Every host should be fully scraped. Log and track every secret found, as credentials equal keys to unlock the next door.
* File Transfer: Master file transfer techniques early, as efficiency depends on reliably moving tools and data. For Windows, low-OPSEC methods include PowerShell + Base64, `bitsadmin`, `certutil`, and SMB fetch via `impacket-smbserver`.

### Pivoting and Lateral Movement

* **Prioritise WinRM:** Get extremely comfortable with WinRM (Windows Remote Management), as CPTS frequently uses it for silent, scriptable lateral movement once credentials are known.
* **Tooling:** Ligolo-ng is highly recommended as the default tunneling stack for the exam, as it is stable, fast, encrypted, and less prone to breaking when pivot boxes reboot, unlike alternatives like Chisel,. Learn how to automate its setup.
* **Recon After Pivot:** Every time you pivot to a new segment, you must redo your entire recon process. This includes running `nmap` scans from the new vantage point, fuzzing internal web applications, and checking routing tables to confirm reachability.

***

## Reporting and Documentation Discipline

The report is the real deliverable and is manually reviewed; a poor report, even with all flags captured, can result in failure. The report is approximately 50% of the grade.

### The "Write as You Go" Workflow

This is non-negotiable; waiting until Day 10 to start writing will lead to panic, misplaced screenshots, and forgotten steps.

Use a trigger-based workflow: Every time you:

* Gain a foothold
* Pivot to a new host
* Capture a flag
* Find a misconfiguration
* Compromise a user...you must write the corresponding report section immediately.

### Evidence Hygiene

* **Screenshots:** Save them into folders named by IP/hostname (e.g., `DC01`) and name them numerically with short labels (e.g., `01-login.png`),. Ensure that every command or exploit has a corresponding screenshot.
* **Command Logging:** Log every step in a plain text file for each host, including the exact command, the context (user/privilege level), and notes on what the output meant. Add commands to the report for reviewers to easily copy/paste.
* **Tooling:** Using a dedicated report builder like SysReptor can save time and sanity by providing CPTS exam templates, handling formatting, and automating the risk matrix,.Writing Style• Third Person: Write strictly in the third person (e.g., "the tester did this").
* **Clarity and Detail:** Write for the least technical stakeholder and use plain English. You must explain every step, as the reader does not have terminal access.
* **Avoid Vague Language:** Do not assume the reader knows the tools. Instead of writing: "I ran linPEAS and saw something interesting," write: "A misconfigured GPO allowed low-priv users to write to the startup folder of a domain controller. By dropping a scheduled task script, the tester achieved SYSTEM access and pivoted...".
* **Remediation:** Provide practical, client-viable solutions, covering short, medium, and long-term remediation.
