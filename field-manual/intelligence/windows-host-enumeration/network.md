# Network

## **I**nterfaces, IP Addresses & DNS Information

### ipconfig

Using the `ipconfig /all` command can provide us with crucial information such as:

* the IP address, subnet mask and Default Gateway of the machine;
* the MAC address of the machine, which could be used for later attacks;
* DNS server IP addresses which are commonly Domain Controllers in a Windows environment.

```sh
C:\> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : WINLPE-SRV01
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : .htb

Ethernet adapter Ethernet1:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
   Physical Address. . . . . . . . . : 00-50-56-B9-C5-4B
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::f055:fefd:b1b:9919%9(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.20.56(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.20.1
   DHCPv6 IAID . . . . . . . . . . . : 151015510
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-27-ED-DB-68-00-50-56-B9-90-94
   DNS Servers . . . . . . . . . . . : 8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : .com
   Description . . . . . . . . . . . : Intel(R) 82574L Gigabit Network Connection
   Physical Address. . . . . . . . . : 00-50-56-B9-90-94
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : dead:beef::e4db:5ea3:2775:8d4d(Preferred)
   Link-local IPv6 Address . . . . . : fe80::e4db:5ea3:2775:8d4d%4(Preferred)
   IPv4 Address. . . . . . . . . . . : 10.129.43.8(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Lease Obtained. . . . . . . . . . : Thursday, March 25, 2021 9:24:45 AM
   Lease Expires . . . . . . . . . . : Monday, March 29, 2021 1:28:44 PM
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:4ddf%4
                                       10.129.0.1
   DHCP Server . . . . . . . . . . . : 10.129.0.1
   DHCPv6 IAID . . . . . . . . . . . : 50352214
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-27-ED-DB-68-00-50-56-B9-90-94
   DNS Servers . . . . . . . . . . . : 1.1.1.1
                                       8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

Tunnel adapter isatap..htb:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : .htb
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Teredo Tunneling Pseudo-Interface
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes

Tunnel adapter isatap.{02D6F04C-A625-49D1-A85D-4FB454FBB3DB}:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter #2
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
```

### ARP Table

It is important to use the `arp` command to view the ARP cache for each interface and view other hosts the machine has recently communicated with. This could help us with lateral movement after obtaining credentials and could be a good indication of which hosts administrators are connecting to via RDP or WinRM from this host.

```sh
C:\> arp -a

Interface: 10.129.43.8 --- 0x4
  Internet Address      Physical Address      Type
  10.129.0.1            00-50-56-b9-4d-df     dynamic
  10.129.43.12          00-50-56-b9-da-ad     dynamic
  10.129.43.13          00-50-56-b9-5b-9f     dynamic
  10.129.255.255        ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.252           01-00-5e-00-00-fc     static
  224.0.0.253           01-00-5e-00-00-fd     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static

Interface: 192.168.20.56 --- 0x9
  Internet Address      Physical Address      Type
  192.168.20.255        ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static
```

### Routing Table

We should always look at [routing tables](https://en.wikipedia.org/wiki/Routing_table) to view information about the local network and networks around it. We can also gather information about the local domain (if the host is part of an Active Directory environment), including the IP addresses of domain controllers.

```sh
C:\> route print

===========================================================================
Interface List
  9...00 50 56 b9 c5 4b ......vmxnet3 Ethernet Adapter
  4...00 50 56 b9 90 94 ......Intel(R) 82574L Gigabit Network Connection
  1...........................Software Loopback Interface 1
  3...00 00 00 00 00 00 00 e0 Microsoft ISATAP Adapter
  5...00 00 00 00 00 00 00 e0 Teredo Tunneling Pseudo-Interface
 13...00 00 00 00 00 00 00 e0 Microsoft ISATAP Adapter #2
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.129.0.1      10.129.43.8     25
          0.0.0.0          0.0.0.0     192.168.20.1    192.168.20.56    271
       10.129.0.0      255.255.0.0         On-link       10.129.43.8    281
      10.129.43.8  255.255.255.255         On-link       10.129.43.8    281
   10.129.255.255  255.255.255.255         On-link       10.129.43.8    281
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
     192.168.20.0    255.255.255.0         On-link     192.168.20.56    271
    192.168.20.56  255.255.255.255         On-link     192.168.20.56    271
   192.168.20.255  255.255.255.255         On-link     192.168.20.56    271
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link       10.129.43.8    281
        224.0.0.0        240.0.0.0         On-link     192.168.20.56    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link       10.129.43.8    281
  255.255.255.255  255.255.255.255         On-link     192.168.20.56    271
===========================================================================
Persistent Routes:
  Network Address          Netmask  Gateway Address  Metric
          0.0.0.0          0.0.0.0     192.168.20.1  Default
===========================================================================

IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
  4    281 ::/0                     fe80::250:56ff:feb9:4ddf
  1    331 ::1/128                  On-link
  4    281 dead:beef::/64           On-link
  4    281 dead:beef::e4db:5ea3:2775:8d4d/128
                                    On-link
  4    281 fe80::/64                On-link
  9    271 fe80::/64                On-link
  4    281 fe80::e4db:5ea3:2775:8d4d/128
                                    On-link
  9    271 fe80::f055:fefd:b1b:9919/128
                                    On-link
  1    331 ff00::/8                 On-link
  4    281 ff00::/8                 On-link
  9    271 ff00::/8                 On-link
===========================================================================
Persistent Routes:
  None
```

***

## Network Services

The most common way people interact with processes is through a network socket (DNS, HTTP, SMB, etc.). The [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) command will display active TCP and UDP connections which will give us a better idea of what services are listening on which port(s) both locally and accessible to the outside. We may find a vulnerable service only accessible to the localhost (when logged on to the host) that we can exploit to escalate privileges.

### Display Active Connections

The main thing to look for with Active Network Connections are entries listening on loopback addresses (`127.0.0.1` and `::1`) that are not listening on the IP Address (`10.129.43.8`) or broadcast (`0.0.0.0`, `::/0`). The reason for this is network sockets on localhost are often insecure due to the thought that "they aren't accessible to the network."

```sh
C:\> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       3812
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       836
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       936
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       5044
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       528
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       996
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1260
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       2008
  TCP    0.0.0.0:49669          0.0.0.0:0              LISTENING       600
  TCP    0.0.0.0:49670          0.0.0.0:0              LISTENING       1888
  TCP    0.0.0.0:49674          0.0.0.0:0              LISTENING       616
  TCP    10.129.43.8:139        0.0.0.0:0              LISTENING       4
  TCP    10.129.43.8:3389       10.10.14.3:63191       ESTABLISHED     936
  TCP    10.129.43.8:49671      40.67.251.132:443      ESTABLISHED     1260
  TCP    10.129.43.8:49773      52.37.190.150:443      ESTABLISHED     2608
  TCP    10.129.43.8:51580      40.67.251.132:443      ESTABLISHED     3808
  TCP    10.129.43.8:54267      40.67.254.36:443       ESTABLISHED     3808
  TCP    10.129.43.8:54268      40.67.254.36:443       ESTABLISHED     1260
  TCP    10.129.43.8:54269      64.233.184.189:443     ESTABLISHED     2608
  TCP    10.129.43.8:54273      216.58.210.195:443     ESTABLISHED     2608
  TCP    127.0.0.1:14147        0.0.0.0:0              LISTENING       3812
<SNIP>
```

The one that sticks out immediately will be port `14147`, which is used for FileZilla's administrative interface. By connecting to this port, it may be possible to extract FTP passwords in addition to creating an FTP Share at c:\ as the FileZilla Server user (potentially Administrator).
