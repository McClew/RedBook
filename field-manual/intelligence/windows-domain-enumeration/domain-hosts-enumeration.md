---
icon: display
---

# Domain Hosts Enumeration

## Cheetsheet

{% tabs %}
{% tab title="Linux" %}
<table><thead><tr><th width="339">Command</th><th>Description</th></tr></thead><tbody><tr><td><code>nxc smb &#x3C;network-range></code></td><td>Scans the specified network range for SMB services, helping identify live Windows hosts.</td></tr><tr><td><p><code>sudo responder</code> </p><p><code>-I &#x3C;network-interface> -A</code></p></td><td><a data-mention href="../../../toolbox/tooling/sniffing-and-spoofing/responder.md">responder.md</a> captures LLMNR, NBT-NS, and MDNS traffic on the specified interface to passively identify hosts.<br><br>Analyse mode (<code>-A</code>) avoids active poisoning.</td></tr><tr><td><a data-mention href="../../post-exploitation/pivoting-tunnelling-and-port-forwarding/pivoting-reconnaisance.md">pivoting-reconnaisance.md</a></td><td>Techniques and tools used for internal reconnaissance after gaining a foothold in the network.</td></tr></tbody></table>
{% endtab %}

{% tab title="Windows" %}
| Command                                                                                                                                                       | Description                                                                                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [bloodhound.md](../../../toolbox/tooling/exploitation-tools/bloodhound.md "mention")                                                                          | [bloodhound.md](../../../toolbox/tooling/exploitation-tools/bloodhound.md "mention") can be used to enumerate the entire AD network and visualize relationships. |
| <p>Get all info:<br><code>Get-NetComputer</code><br><br>Just crucial information:<br><code>Get-NetComputer | select cn,operatingsystem,dnshostname</code></p> | [powerview.md](../../../toolbox/tooling/post-exploitation/powersploit/powerview.md "mention") command to list domain computers and filter key details.           |
| `nslookup <dnshostname>`                                                                                                                                      | Resolves a hostname to an IP address via DNS.                                                                                                                    |
{% endtab %}
{% endtabs %}
