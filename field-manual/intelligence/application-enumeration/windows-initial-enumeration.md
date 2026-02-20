# Windows Initial Enumeration

## **Enumerating Installed Programs**

Let's start by enumerating installed applications to get a lay of the land.

```cmd-session
C:\htb> wmic product get name

Name
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29910
Update for Windows 10 for x64-based Systems (KB4023057)
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.24.28127
VMware Tools
Druva inSync 6.6.3
Microsoft Update Health Tools
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29910
Update for Windows 10 for x64-based Systems (KB4480730)
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.24.28127
```

The output looks mostly standard for a Windows 10 workstation. However, the `Druva inSync` application stands out.

A quick Google search shows that version `6.6.3` is vulnerable to a command injection attack via an exposed RPC service. We may be able to use [this](https://www.exploit-db.com/exploits/49211) exploit PoC to escalate our privileges. The client application runs a service in the context of the powerful `NT AUTHORITY\SYSTEM` account. Escalation is possible by interacting with a service running locally on port 6064.

***

## **Enumerating Local Ports**

Let's do some further enumeration to confirm that the service is running as expected. A quick look with `netstat` shows a service running locally on port `6064`.

```cmd-session
C:\htb> netstat -ano | findstr 6064

  TCP    127.0.0.1:6064         0.0.0.0:0              LISTENING       3324
  TCP    127.0.0.1:6064         127.0.0.1:50274        ESTABLISHED     3324
  TCP    127.0.0.1:6064         127.0.0.1:50510        TIME_WAIT       0
  TCP    127.0.0.1:6064         127.0.0.1:50511        TIME_WAIT       0
  TCP    127.0.0.1:50274        127.0.0.1:6064         ESTABLISHED     3860
```

***

## **Enumerating Process ID**

Next, let's map the process ID (PID) `3324` back to the running process.

```powershell-session
PS C:\htb> get-process -Id 3324

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    149      10     1512       6748              3324   0 inSyncCPHwnet64
```

***

## **Enumerating Running Service**

At this point, we have enough information to determine that the Druva inSync application is indeed installed and running, but we can do one last check using the `Get-Service` cmdlet.

```powershell-session
PS C:\htb> get-service | ? {$_.DisplayName -like 'Druva*'}

Status   Name               DisplayName
------   ----               -----------
Running  inSyncCPHService   Druva inSync Client Service
```
