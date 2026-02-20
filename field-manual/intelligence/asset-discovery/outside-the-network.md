# Outside the Network

This section is about how to find IPs responding from the internet. We may have a scope of IPs (maybe even several ranges) and we just need to find which IPs are responding.

***

## ICMP

This is the easiest and fastest way to discover if a host is up, we can try sending ICMP packets and see if we recieve responses. To achieve this we can send an `echo` request using tools like `ping` or `fping` for an IP range.

```bash
ping -c 1 199.66.11.4
```

```bash
fping -g 199.66.11.0/24
```

We can also use [nmap](../../../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention") to send other types of ICMP packets, which may avoice filters to common ICMP echo request-response.

```bash
nmap -PE -PM -PP -sn -n 199.66.11.0/24
```

***

## TCP Port Discovery

It's very common to find ICMP packets being filtered, in which case, to check if a host is up we'll need to try find open ports.

We can use a fast port scanner like [masscan.md](../../../toolbox/tooling/information-gathering/network-enumeration/masscan.md "mention") to improve this process, though it's important to note that if we have a large scope this will likely take far too long.

{% code overflow="wrap" %}
```bash
masscan -p20,21-23,25,53,80,110,111,135,139,143,443,445,993,995,1723,3306,3389,5900,8080 199.66.11.0/24
```
{% endcode %}

{% hint style="success" %}
Using [masscan.md](../../../toolbox/tooling/information-gathering/network-enumeration/masscan.md "mention") to scan the top 20 ports in a /24 range should take less than 5 minutes.
{% endhint %}

We can also perform this step with [nmap](../../../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention"), but it's slower and may have issues identifying hosts.

***

## HTTP Port Discovery

This is simple TCP port discovery using [masscan.md](../../../toolbox/tooling/information-gathering/network-enumeration/masscan.md "mention"), which is useful when we want to focus on discovering HTTP services.

```bash
masscan -p80,443,8000-8100,8443 199.66.11.0/24
```

***

## UDP Port Discovery

We can also try finding open UDP ports. As UDP services usually don’t respond with any data to a regular empty UDP probe packet it is difficult to say if a port is being filtered or is open.&#x20;

The easiest way to confirm this is to send a packet related to the running service, and as we don’t know which service is running, we should try the most probable based on the port number.

```bash
nmap -sU -sV --version-intensity 0 -F -n 199.66.11.53/24
```

<table><thead><tr><th width="250">Scanning Options</th><th>Description</th></tr></thead><tbody><tr><td><code>-sV</code></td><td>Nmap will test each possible UDP service packet.</td></tr><tr><td><code>--version-intensity 0</code></td><td>Nmap will only test the most probable packets.</td></tr></tbody></table>

This [nmap](../../../toolbox/tooling/information-gathering/network-enumeration/nmap/ "mention") command will test the top 1000 UDP ports in every host inside the /24 range. This will likely be a slow scan and take over 20 minutes.

We could use a tool such as [udp-proto-scanner](https://github.com/portcullislabs/udp-proto-scanner):&#x20;

```bash
./udp-proto-scanner.pl 199.66.11.53/24
```

This will send UDP probes to their expected port, for a /24 range this will just take 1 minute.

***

## SCTP Port Discovery <a href="#sctp-port-discovery" id="sctp-port-discovery"></a>

This will likely not work, but why not try it?

```bash
nmap -T4 -sY -n --open -Pn <IP/range>
```
