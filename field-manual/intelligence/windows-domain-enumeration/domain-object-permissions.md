---
icon: draw-square
---

# Domain Object Permissions

Object Permissions Enumeration is a critical step in identifying privilege escalation opportunities within Active Directory environments.

Identifying where a user has local administrator access helps attackers or red teamers pinpoint systems that can be targeted for privilege escalation or credential harvesting. This information is especially useful when combined with other access or trust relationships.

Querying Active Directory object ACLs allows you to discover which users or groups have delegated control or specific permissions over domain objects such as users, groups, OUs, or GPOs. These permissions can often be abused to escalate privileges or move laterally within the environment.

Enumerating domain shares reveals where valuable files may be stored and which resources are accessible. Filtering for readable shares helps narrow down viable targets, especially in environments where sensitive data or scripts may be left exposed.

Together, these techniques form a foundation for effective AD enumeration and privilege escalation strategies during internal assessments.

***

## Cheatsheet

| Command                                                                                                                                                                              | Description                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Find-LocalAdminAccess`                                                                                                                                                              | <p>Searches for computers where a specified user has local administrator rights within the domain.<br><br>Depending on the size of the environment, it may take a few minutes for this command to finish.</p> |
| <p><code>$sid = Convert-NameToSid &#x3C;user></code><br><code>Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}</code></p>                          | Finds AD objects where the specified user (converted to SID) has explicit permissions set in their ACLs.                                                                                                      |
| <p><code>Find-DomainShare</code><br><br>Add <code>-CheckShareAccess</code> for only readable shares<br><br>You can then <code>dir \\&#x3C;dns-hostname>\&#x3C;share-name></code></p> | Enumerates domain shares, optionally filtering for shares accessible with read permissions.                                                                                                                   |

{% hint style="info" %}
#### Note

Most of these commands are part of [powerview.md](../../../toolbox/tooling/post-exploitation/powersploit/powerview.md "mention").
{% endhint %}
