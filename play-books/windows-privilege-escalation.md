---
icon: microsoft
---

# Windows Privilege Escalation

1. Run Windows privilege escalation scripts. Save the output to a file and transfer it to host for examination.
   * WinPeas
   * Searbelt
   * PowerUp / StartUp
   * JAWS - recommended by Ippsec, uses PS.
   * SessionGopher
   * Bloodhound
2. Perform basic enumeration.
   * Network enumeration.
   * System enumeration
   * Process enumeration
   * User and group enumeration.
3. Look at access rights of current user.
   * `whoami /priv`
   * Windows privileges.
   * SeImpersonate / SeAssignPrimaryToken
   * SeDebugPrivilege
   * SeTakeOwnershipPrivilege
4. Check if current user is a member of privileged groups.
   * whoami /groups
   * Backup opreators
   * event log readers
   * DmsAdmins
   * Hyper-V Admins
   * Print Operators
   * Server Operators.
5. Check for weak file / service permissions.
   * Permissive File System ACLs.
   * Weak service permissions.
   * Unquoted service path.
   * Permissive registry ACLs.
   * Modifiable registry autorun binary.
6. Check for saved credentials.
   * CmdKey saved credentials
   * `cmdkey /list`
   * If access gained, run [mimikatz.md](../toolbox/tooling/post-exploitation/mimikatz.md "mention") to attempt extracting plaintext password.
7. Look for services running on internal ports that were not accessible externally with netstat.
   * Databases?
   * `netstat -ano`
   * services
8. Check for additional NICs using commands like ipconfig.
9. Look for vulnerable applications and services.
   1. `wmic product get name`
10. Look for interesting files on the server that may have credentials or other sensitive information.
    * Credential hunting
    * further credential theft
    * dumping hashes / credentials
    * mimikatz
11. Pillage for credentials or other interesting information
    * pillaging
    * pillaging applicaitons
    * accessing instant message clients through cookies
    * pillaging the clipboard + keylogging
    * pilaging backups
12. look for scheduled tasks that can be modified
13. Look for credentials in process command line
14. Check for 'always install elevated' setting and exploit with MSI package.
15. Capture hashes with a Malicious LINK file of SCF file.
    * Capturing hashes with malicious .lnk file.
    * capture hashes with SCF on a file share ( < windows server 2019 ).
16. Look for Kernel / OS exploits.
    * Kernel exploits
    * EOL System exploits.
17. Attempt to bypass UAC controls if present.
    * UAC attacks.
18. DLL Injection
19. Common OS and program vulnerabilities.
    * Windows certificate dialog (CVE-2019-1388)
20. Attempt to capture network traffic with Inveigh or Responder, wireshark or snaffler.
    * LLMNR/NBT-NS poisoning
21. Enumerate user, computer desicpriotn fields for cleartext credentials or other information.
22. If the system has .vhd, .vhdx, and .vmdk files, mount them to potentially dump machine hashses.
    * Mount VHDX/VMDK
