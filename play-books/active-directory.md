---
icon: server
---

# Active Directory

## Uncredentialed Enumeration

\-

### Host Indenticiation

{% stepper %}
{% step %}
#### Listening with Wireshark

Use Wireshark and listen for Layer 2 ( ARP, NDNS ) traffic to discover IP addresses and hostnames.
{% endstep %}

{% step %}
#### Analysing with Responder

Use Responder in 'Analyze' mode to discover IP addresses and hostnames
{% endstep %}

{% step %}
#### ICMP Sweep

Perform an Fping ICMP sweep to final all hosts on your subnet that respond to an ICMP echo request.
{% endstep %}

{% step %}
### Nmap Network Scan

Perform an NMAP identification scan to validate findings (it may find something previously missed).
{% endstep %}

{% step %}
### Nmap Service Scan

Use all disocvered hosts to perform an NMAP scan to determin services running on each host.

1. Look for quick wins for initial foothold, like outdated software, service or OS.
{% endstep %}
{% endstepper %}

### User Identification

{% stepper %}
{% step %}
###

Attempt to abuse an SMB NULL session against the domain controller.

1. rcpclient or enum4linux to grab users.
{% endstep %}

{% step %}
###

Attempt to abuse an anonymous LDAP search against the domain controller.

1. ldapsearch or windapsearch to grab users.
{% endstep %}

{% step %}
###

Check for users that do not require Kerberos pre-auth (ASREP-Roasting) to get password hash.
{% endstep %}

{% step %}
###

Use Kerbrute and wordlists to brute force usernames against DC.
{% endstep %}

{% step %}
###

Use Impackets lookupsid.py to discover users (works better with creds)
{% endstep %}

{% step %}
###

Look for systems that can be exploited to gain SYSTEM level access.
{% endstep %}
{% endstepper %}

### User Foothold

1. Use Responder/Inveigh on network interface to listen for NTLM users and hashes.
   1. Attempt to crack hashes.
2. attempt password spray on users identified during user identification.
   1. attempt to gather password policy for organisation.
   2. password spray using common passwords.

***

## Credentialed Enumeration &  Exploitation

### Host Identification

1. Run Bloodhound with discovered credentials.
2. Use `ldapdomaindump` to identify all domain joined computers.
3. Enumerate accessible shares on servers with CrackMapExec, SMBMap, PowerView or Snaffler.

### User Identificaiton

1. Run Bloodhound with discovered credentials
   1. Bloodhound.py
   2. Bloodhound from windows
2. Gather the domain password policy using the discovered credentails
   1. Gather a list of Domain Admins or Privileged users using tools:
      1. Windapsearch
      2. powerview
      3. bloodhound
      4. ad powershell module

### Foothold Enumeration

1. Enumerate security controls
2. Look for other logged-in users using CME
3. Look for Kerberoastable accounts.
4. Look at owned users for abusable ACL entries
5. Check bloodhound for CanRDP, CanPSRemote, or SQLAdmin abilities for lateral movement.

### Pivoting

1. Run chisel for windows on the windows pivot host
   1. connect with the linux chisel client on the attack host.
   2. modify the proxychain.conf file to match the proxy that is established.
   3. run command with proxychains
2. Check bloodhound for CanRDP, CanPSRemote or SQLAdmin abilities forlateral movement.

## Exploitation

1. look for kerberoastable accounts suing bloodhound, powerview, getuserspns.py
2. grab all TGS tickets with GetUserSPNs.py, save to file and attempt to crack.
3. Abuse any over permissive acl entries to gain control of more users and move laterally throughout the network.
   1. ForceChangePassword
   2. AddMember
   3. GeneralAll/GenericWrite
   4. Ds-Replpciation-GetChanges-All
   5. ACL Abuse
4. check for common vulnerabilities:
   1. NoPac
   2. PrintNightmate
   3. PetitPotam
5. Check for common misconfigurations to escalate privileges:
   1. exchange group permissions
   2. ms-rprn printer bug
   3. MS14-068
   4. sniff for LDAP credentials
   5. enumerate DNS records for interesting servers
   6. look for user passwords and other notes in AD user descriptions
   7. check for PASSWD\_NOTRREQD field on users and test for weak/no passwoords.
   8. Look for credentials and other interesting files on SMB shares.
   9. check for DONT\_REQ\_PREAUTH field and ASREPRoasting any discovered users.
   10. Check for GPOs that we have write access over to gain administrator rights or more latterally.
   11. Resource based constrained delegation, constrained delegation, unconstrained delegation
   12. active directory certificate services attacks
6. check for group policy regerences GPP passwords

### Additional Auditing

1. create a snapshot of the AD database with AD explorer for offline analysis
2. use PingCastle to discover additional AD misconfigurations and vulnerabilities
3. run group3r to uncover vulnerabilities in AD Group Policy.
4. Run ADRecon.ps1 to disocver additional AD misconfigurations and vulnerabilties that may have been missed.

### Attacking AD Trusts (Parent Domain)

1. Discover any current domain trusts with other domains using Get-ADTrust, Get-DomainTrust (PowerView) or Bloodhound.
2. From a machine with Domain Admin privileges, attempt an ExtraSIDs attack to create an Enterprise admin user in the parent domain.
3. Domain trusts overview
4. child > parent attacks Windows
5. child > parent attacks Linux

### Attacking AD Trusts (Cross Forest)

1. Discover any current domain trusts with other domains using Get-ADTrust, Get-DomainTrust (powerview) or bloodhound
2. Attempt cross-forsest kerberoasting
3. if admin accounts sharee names across domains, and on is compromsied, try credential reuse.
4. Check for SIDHistory abuse.
5. cross-forest trust abuse - windows
6. crossforst trust abuse - linux.

## Additional Methods

1. Access a host via RDP or WinRM as a local user or a local admin.
2. Authenticate to a remote host as an admin using tools such as PsExec.
3. Gain access to a sensitive file share.
4. Gain MSSQL access to a host as a DBA user, which can then be leveraged to escalate permissions.
