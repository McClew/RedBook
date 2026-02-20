# Alive or Not

```bash
clue@machine[~]$ sudo nmap 10.129.2.18 -sn -oA host 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-14 23:59 CEST
Nmap scan report for 10.129.2.18
Host is up (0.087s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

| **Scanning Options** | **Description**                                                  |
| -------------------- | ---------------------------------------------------------------- |
| `10.129.2.18`        | Performs defined scans against the target.                       |
| `-sn`                | Disables port scanning.                                          |
| `-oA host`           | Stores the results in all formats starting with the name 'host'. |

If we disable port scan (`-sn`), Nmap automatically ping scans with `ICMP Echo Requests` (`-PE`). Once such a request is sent, we usually expect an `ICMP reply` if the pinging host is alive. The more interesting fact is that our previous scans did not do that because before Nmap could send an ICMP echo request, it would send an `ARP ping` resulting in an `ARP reply`. We can confirm this with the "`--packet-trace`" option. To ensure that ICMP echo requests are sent, we also define the option (`-PE`) for this.

```bash
clue@machine[~]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:08 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up (0.023s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

| **Scanning Options** | **Description**                                                          |
| -------------------- | ------------------------------------------------------------------------ |
| `10.129.2.18`        | Performs defined scans against the target.                               |
| `-sn`                | Disables port scanning.                                                  |
| `-oA host`           | Stores the results in all formats starting with the name 'host'.         |
| `-PE`                | Performs the ping scan by using 'ICMP Echo requests' against the target. |
| `--packet-trace`     | Shows all packets sent and received                                      |

***

Another way to determine why Nmap has our target marked as "alive" is with the "`--reason`" option.

```bash
clue@machine[~]$ sudo nmap 10.129.2.18 -sn -oA host -PE --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:10 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up, received arp-response (0.028s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds
```

| **Scanning Options** | **Description**                                                          |
| -------------------- | ------------------------------------------------------------------------ |
| `10.129.2.18`        | Performs defined scans against the target.                               |
| `-sn`                | Disables port scanning.                                                  |
| `-oA host`           | Stores the results in all formats starting with the name 'host'.         |
| `-PE`                | Performs the ping scan by using 'ICMP Echo requests' against the target. |
| `--reason`           | Displays the reason for specific result.                                 |

***

We see here that `Nmap` does indeed detect whether the host is alive or not through the `ARP request` and `ARP reply` alone. To disable ARP requests and scan our target with the desired `ICMP echo requests`, we can disable ARP pings by setting the "`--disable-arp-ping`" option. Then we can scan our target again and look at the packets sent and received.

```bash
clue@machine[~]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:12 CEST
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
Nmap scan report for 10.129.2.18
Host is up (0.086s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```
