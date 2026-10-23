---
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: false
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
---

# Nmap

## Manual

### Scanning Options

<table><thead><tr><th width="204.33331298828125">Nmap Options</th><th>Description</th></tr></thead><tbody><tr><td>10.10.10.0/24</td><td>Target network range.</td></tr><tr><td>-sn</td><td>Disables port scanning.</td></tr><tr><td>-Pn</td><td>Disables ICMP Echo Requests</td></tr><tr><td>-n</td><td>Disables DNS Resolution.</td></tr><tr><td>-PE</td><td>Performs the ping scan by using ICMP Echo Requests against the target.</td></tr><tr><td>--packet-trace</td><td>Shows all packets sent and received.</td></tr><tr><td>--reason</td><td>Displays the reason for a specific result.</td></tr><tr><td>--disable-arp-ping</td><td>Disables ARP Ping Requests.</td></tr><tr><td>--top-ports=</td><td>Scans the specified top ports that have been defined as most frequent.</td></tr><tr><td>-p-</td><td>Scan all ports.</td></tr><tr><td>-110</td><td>Scan all ports between 22 and 110.</td></tr><tr><td>-p22,25</td><td>Scans only the specified ports 22 and 25.</td></tr><tr><td>-F</td><td>Scans top 100 ports.</td></tr><tr><td>-sS</td><td>Performs an TCP SYN-Scan.</td></tr><tr><td>-sA</td><td>Performs an TCP ACK-Scan.</td></tr><tr><td>-sU</td><td>Performs an UDP Scan.</td></tr><tr><td>-sV</td><td>Scans the discovered services for their versions.</td></tr><tr><td>-sC</td><td>Perform a Script Scan with scripts that are categorized as default.</td></tr><tr><td>--script</td><td>Performs a Script Scan by using the specified scripts.</td></tr><tr><td>-O</td><td>Performs an OS Detection Scan to determine the OS of the target.</td></tr><tr><td>-A</td><td>Performs OS Detection, Service Detection, and traceroute scans.</td></tr><tr><td>-D RND:5</td><td>Sets the number of random Decoys that will be used to scan the target.</td></tr><tr><td>-e</td><td>Specifies the network interface that is used for the scan.</td></tr><tr><td>-S 10.10.10.200</td><td>Specifies the source IP address for the scan.</td></tr><tr><td>-g</td><td>Specifies the source port for the scan.</td></tr><tr><td>--dns-server</td><td>DNS resolution is performed by using a specified name server.</td></tr></tbody></table>

### Output Options

<table><thead><tr><th width="162.3333740234375">Nmap Option</th><th>Description</th></tr></thead><tbody><tr><td>-oA filename</td><td>Stores the results in all available formats starting with the name of "filename".</td></tr><tr><td>-oN filename</td><td>Stores the results in normal format with the name "filename".</td></tr><tr><td>-oG filename</td><td>Stores the results in "grepable" format with the name of "filename".</td></tr><tr><td>-oX filename</td><td>Stores the results in XML format with the name of "filename".</td></tr></tbody></table>

### Other Options

<table><thead><tr><th width="228.3333740234375">Nmap Option</th><th>Description</th></tr></thead><tbody><tr><td>--max-retries</td><td>Sets the number of retries for scans of specific ports.</td></tr><tr><td>--stats-every=5s</td><td>Displays scan's status every 5 seconds.</td></tr><tr><td>-v/-vv</td><td>Displays verbose output during the scan.</td></tr><tr><td>--initial-rtt-timeout 50ms</td><td>Sets the specified time value as initial RTT timeout.</td></tr><tr><td>--max-rtt-timeout 100ms</td><td>Sets the specified time value as maximum RTT timeout.</td></tr><tr><td>--min-rate 300</td><td>Sets the number of packets that will be sent simultaneously.</td></tr><tr><td>-T &#x3C;0-5></td><td>Specifies the specific timing template.</td></tr></tbody></table>
