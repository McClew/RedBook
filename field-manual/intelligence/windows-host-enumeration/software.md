# Software

## Installed Programs

### CMD

```cmd-session
C:\>dir "C:\Program Files"
 Volume in drive C has no label.
 Volume Serial Number is 900E-A7ED

 Directory of C:\Program Files

07/14/2022  08:31 PM    <DIR>          .
07/14/2022  08:31 PM    <DIR>          ..
05/16/2022  03:57 PM    <DIR>          Adobe
05/16/2022  12:33 PM    <DIR>          Corsair
05/16/2022  10:17 AM    <DIR>          Google
05/16/2022  11:07 AM    <DIR>          Microsoft Office 15
07/10/2022  11:30 AM    <DIR>          mRemoteNG
07/13/2022  09:14 AM    <DIR>          OpenVPN
07/19/2022  09:04 PM    <DIR>          Streamlabs OBS
07/20/2022  07:06 AM    <DIR>          TeamViewer
               0 File(s)              0 bytes
              16 Dir(s)  351,524,651,008 bytes free
```

### PowerShell & Registry Keys

```powershell
PS C:\> $INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* |  Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\> $INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\> $INSTALLED | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize

DisplayName                                         DisplayVersion    InstallLocation
-----------                                         --------------    ---------------
Adobe Acrobat DC (64-bit)                           22.001.20169      C:\Program Files\Adobe\Acrobat DC\
CORSAIR iCUE 4 Software                             4.23.137          C:\Program Files\Corsair\CORSAIR iCUE 4 Software
Google Chrome                                       103.0.5060.134    C:\Program Files\Google\Chrome\Application
Google Drive                                        60.0.2.0          C:\Program Files\Google\Drive File Stream\60.0.2.0\GoogleDriveFS.exe
Microsoft Office Profesional Plus 2016 - es-es      16.0.15330.20264  C:\Program Files (x86)\Microsoft Office
Microsoft Office Professional Plus 2016 - en-us     16.0.15330.20264  C:\Program Files (x86)\Microsoft Office
mRemoteNG                                           1.62              C:\Program Files\mRemoteNG
TeamViewer                                          15.31.5           C:\Program Files\TeamViewer
...SNIP...
```

### WMI

WMI can be used to display installed software. This information can often guide us towards hard-to-find exploits. Is FileZilla, Putty, etc. installed? Run **LaZagne** to check if stored credentials for those applications are installed. Also, some programs may be installed and running as a service that is vulnerable.

```sh
C:\> wmic product get name

Name
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.24.28127
Java 8 Update 231 (64-bit)
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.24.28127
VMware Tools
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.24.28127
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.24.28127
Java Auto Updater

<SNIP>
```

### Get-WmiObject

When using PowerShell we can use the [Get-WmiObject](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject?view=powershell-5.1) cmdlet.

```powershell
PS C:\> Get-WmiObject -Class Win32_Product |  select Name, Version

Name                                                                    Version
----                                                                    -------
SQL Server 2016 Database Engine Shared                                  13.2.5026.0
Microsoft OLE DB Driver for SQL Server                                  18.3.0.0
Microsoft Visual C++ 2010  x64 Redistributable - 10.0.40219             10.0.40219
Microsoft Help Viewer 2.3                                               2.3.28107
Microsoft Visual C++ 2010  x86 Redistributable - 10.0.40219             10.0.40219
Microsoft Visual C++ 2013 x86 Minimum Runtime - 12.0.21005              12.0.21005
Microsoft Visual C++ 2013 x86 Additional Runtime - 12.0.21005           12.0.21005
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29914          14.28.29914
Microsoft ODBC Driver 13 for SQL Server                                 13.2.5026.0
SQL Server 2016 Database Engine Shared                                  13.2.5026.0
SQL Server 2016 Database Engine Services                                13.2.5026.0
SQL Server Management Studio for Reporting Services                     15.0.18369.0
Microsoft SQL Server 2008 Setup Support Files                           10.3.5500.0
SSMS Post Install Tasks                                                 15.0.18369.0
Microsoft VSS Writer for SQL Server 2016                                13.2.5026.0
Java 8 Update 231 (64-bit)                                              8.0.2310.11
Browser for SQL Server 2016                                             13.2.5026.0
Integration Services                                                    15.0.2000.130

<SNIP>
```

***

## Running Processes

### Tasklist

Using the [tasklist](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tasklist) command to look at running processes will give us a better idea of what applications are currently running on the system.

```sh
C:\> tasklist /svc

Image Name                     PID Services
========================= ======== ============================================
System Idle Process              0 N/A
System                           4 N/A
smss.exe                       316 N/A
csrss.exe                      424 N/A
wininit.exe                    528 N/A
csrss.exe                      540 N/A
winlogon.exe                   612 N/A
services.exe                   664 N/A
lsass.exe                      672 KeyIso, SamSs, VaultSvc
svchost.exe                    776 BrokerInfrastructure, DcomLaunch, LSM,
                                   PlugPlay, Power, SystemEventsBroker
svchost.exe                    836 RpcEptMapper, RpcSs
LogonUI.exe                    952 N/A
dwm.exe                        964 N/A
svchost.exe                    972 TermService
svchost.exe                   1008 Dhcp, EventLog, lmhosts, TimeBrokerSvc
svchost.exe                    364 NcbService, PcaSvc, ScDeviceEnum, TrkWks,
                                   UALSVC, UmRdpService
<SNIP>

svchost.exe                   1468 Wcmsvc
svchost.exe                   1804 PolicyAgent
spoolsv.exe                   1884 Spooler
svchost.exe                   1988 W3SVC, WAS
svchost.exe                   1996 ftpsvc
svchost.exe                   2004 AppHostSvc
FileZilla Server.exe          1140 FileZilla Server
inetinfo.exe                  1164 IISADMIN
svchost.exe                   1736 DiagTrack
svchost.exe                   2084 StateRepository, tiledatamodelsvc
VGAuthService.exe             2100 VGAuthService
vmtoolsd.exe                  2112 VMTools
MsMpEng.exe                   2136 WinDefend

<SNIP>

FileZilla Server Interfac     5628 N/A
jusched.exe                   5796 N/A
cmd.exe                       4132 N/A
conhost.exe                   4136 N/A
TrustedInstaller.exe          1120 TrustedInstaller
TiWorker.exe                  1816 N/A
WmiApSrv.exe                  2428 wmiApSrv
tasklist.exe                  3596 N/A
```

### Netstat

The [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) command will display active TCP and UDP connections which will give us a better idea of what services are listening on which port(s) both locally and accessible to the outside. We may find a vulnerable service only accessible to the local host (when logged on to the host) that we can exploit to escalate privileges.

```powershell
PS C:\> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       1096
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       840
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:1433           0.0.0.0:0              LISTENING       3520
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       968
<SNIP>
```

***

## Pipes

Pipes are used for communication between two applications or processes using shared memory. There are two types of pipes, [named pipes](https://docs.microsoft.com/en-us/windows/win32/ipc/named-pipes) and anonymous pipes. Pipes are essentially files stored in memory that get cleared out after being read.

An example of a named pipe is `\\.\PipeName\\ExampleNamedPipeServer`. Windows systems use a client-server implementation for pipe communication. In this type of implementation, the process that creates a named pipe is the server, and the process communicating with the named pipe is the client.

Named pipes can communicate using `half-duplex`, or a one-way channel with the client only being able to write data to the server, or `duplex`, which is a two-way communication channel that allows the client to write data over the pipe, and the server to respond back with data over that pipe.

Every active connection to a named pipe server results in the creation of a new named pipe. These all share the same pipe name but communicate using a different data buffer.

### Listing Named Pipes with SysInternals

We can use the tool [PipeList](https://docs.microsoft.com/en-us/sysinternals/downloads/pipelist) from the [sysinternals-suite.md](../../../toolbox/tooling/information-gathering/windows-enumeration/privilege-escalation/sysinternals-suite.md "mention") to enumerate instances of named pipes.

```sh
C:\> pipelist.exe /accepteula

PipeList v1.02 - Lists open named pipes
Copyright (C) 2005-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

Pipe Name                                    Instances       Max Instances
---------                                    ---------       -------------
InitShutdown                                      3               -1
lsass                                             4               -1
ntsvcs                                            3               -1
scerpc                                            3               -1
Winsock2\CatalogChangeListener-340-0              1                1
Winsock2\CatalogChangeListener-414-0              1                1
epmapper                                          3               -1
Winsock2\CatalogChangeListener-3ec-0              1                1
Winsock2\CatalogChangeListener-44c-0              1                1
LSM_API_service                                   3               -1
atsvc                                             3               -1
Winsock2\CatalogChangeListener-5e0-0              1                1
eventlog                                          3               -1
Winsock2\CatalogChangeListener-6a8-0              1                1
spoolss                                           3               -1
Winsock2\CatalogChangeListener-ec0-0              1                1
wkssvc                                            4               -1
trkwks                                            3               -1
vmware-usbarbpipe                                 5               -1
srvsvc                                            4               -1
ROUTER                                            3               -1
vmware-authdpipe                                  1                1
<SNIP>
```

### Listing Named Pipes with PowerShell

We can use PowerShell to list named pipes using `gci` (`Get-ChildItem`).

```powershell
PS C:\> gci \\.\pipe\


    Directory: \\.\pipe


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              3 InitShutdown
------       12/31/1600   4:00 PM              4 lsass
------       12/31/1600   4:00 PM              3 ntsvcs
------       12/31/1600   4:00 PM              3 scerpc


    Directory: \\.\pipe\Winsock2


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              1 Winsock2\CatalogChangeListener-34c-0


    Directory: \\.\pipe


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              3 epmapper

<SNIP>
```

### Named Pipe Permissions

After obtaining a listing of named pipes, we can use [Accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) to enumerate the permissions assigned to a specific named pipe by reviewing the Discretionary Access List (DACL), which shows us who has the permissions to modify, write, read, or execute a resource. Let's take a look at the `LSASS` process. We can also review the DACLs of all named pipes using the command `.\accesschk.exe /accepteula \pipe\`.

### **Reviewing LSASS Named Pipe Permissions**

From this output, we can see that only administrators have full access to the LSASS process, as expected.

```sh
C:\> accesschk.exe /accepteula \\.\Pipe\lsass -v

Accesschk v6.12 - Reports effective permissions for securable objects
Copyright (C) 2006-2017 Mark Russinovich
Sysinternals - www.sysinternals.com

\\.\Pipe\lsass
  Untrusted Mandatory Level [No-Write-Up]
  RW Everyone
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW NT AUTHORITY\ANONYMOUS LOGON
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW APPLICATION PACKAGE AUTHORITY\Your Windows credentials
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW BUILTIN\Administrators
        FILE_ALL_ACCESS
```
