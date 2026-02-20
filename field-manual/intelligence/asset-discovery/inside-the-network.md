# Inside the Network

If we are inside the network one of the first things we will want to do is to discover other hosts. Depending on how much noise you can/want to make, different actions could be performed.

***

## Passive <a href="#passive" id="passive"></a>

We can use these tools to passively discover hosts inside a connected network:

```bash
netdiscover -p
p0f -i eth0 -p -o /tmp/p0f.log
# Bettercap
net.recon on/off #Read local ARP cache periodically
net.show
set net.show.meta true #more info
```

***

## Active <a href="#active" id="active"></a>

Note that the techniques commented in #Outside\_the\_network can be also applied here. But, as we are in the same network as the other hosts, we can do more things:

### ARP Discovery - Nmap

#### Basic Nmap Command

```bash
nmap -sn <NETWORK>
```

#### Advanced Nmap Command

```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

<table><thead><tr><th width="269">Scanning Options</th><th>Description</th></tr></thead><tbody><tr><td><code>10.129.2.0/24</code></td><td>Target network range.</td></tr><tr><td><code>-sn</code></td><td>Disables port scanning.</td></tr><tr><td><code>-oA tnet</code></td><td>Stores the results in all formats starting with the name 'tnet'.</td></tr></tbody></table>

{% hint style="warning" %}
This scanning method works only if the firewalls of the hosts allow it.
{% endhint %}

### ARP Discovery - Netdiscover

```bash
netdiscover -r <NETWORK>
```

### NBT Discovery - nbtscan

```bash
nbtscan -r 192.168.0.1/24
```

### Bettercap

```bash
net.probe on/off #Discover hosts on current subnet by probing with ARP, mDNS, NBNS, UPNP, and/or WSD
set net.probe.mdns true/false #Enable mDNS discovery probes (default=true)
set net.probe.nbns true/false #Enable NetBIOS name service discovery probes (default=true)
set net.probe.upnp true/false #Enable UPNP discovery probes (default=true)
set net.probe.wsd true/false #Enable WSD discovery probes (default=true)
set net.probe.throttle 10 #10ms between probes sent (default=10)
```

### IPV6

```bash
alive6 <IFACE> # Send a pingv6 to multicast.
```

***

## Active ICMP <a href="#active-icmp" id="active-icmp"></a>

Note that the techniques commented in # can be also applied here. But, as we are in the same network as the other hosts, we can do more things:

* If we `ping` a **subnet broadcast address** the ping should be arrive to each host and they should respond with: `ping -b 10.10.5.255`
* Pinging the **network broadcast address** we could even find hosts inside other subnets: `ping -b 255.255.255.255`
* Use the `-PE`, `-PP`, `-PM` flags of `nmap`to perform host discovery sending respectively ICMPv4 echo, timestamp, and subnet mask requests: `nmap -PE -PM -PP -sn -vvv -n 10.12.5.0/24`

***

## Wake On Lan <a href="#wake-on-lan" id="wake-on-lan"></a>

Wake On Lan is used to **turn on** computers through a network message. The magic packet used to turn on the computer is only a packet where a **MAC Dst** is provided and then it is repeated 16 times inside the same paket.

Then this kind of packets are usually sent in an **ethernet 0x0842** or in a **UDP packet to port 9**. If **no \[MAC]** is provided, the packet is sent to **broadcast ethernet** (and the broadcast MAC will be the one being repeated).

```bash
# Bettercap (if no [MAC] is specificed ff:ff:ff:ff:ff:ff will be used/entire broadcast domain)
wol.eth [MAC] #Send a WOL as a raw ethernet packet of type 0x0842
wol.udp [MAC] #Send a WOL as an IPv4 broadcast packet to UDP port 9
```
