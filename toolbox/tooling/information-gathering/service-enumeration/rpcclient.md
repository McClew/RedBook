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

# rpcclient

## **Starting a Session**

```bash
rpcclient -U <username>[%<password>] <target_server_IP_or_hostname>
```

#### Example:

```bash
rpcclient -U Administrator%MySecretPassw0rd! 192.168.1.100
```

## **Obtaining Password Policy**

```bash
querydominfo
```

#### Expected Output:

```bash
rpcclient $> querydominfo

Domain:		INLANEFREIGHT
Server:		
Comment:	
Total Users:	3650
Total Groups:	0
Total Aliases:	37
Sequence No:	1
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
rpcclient $> getdompwinfo
min_password_length: 8
password_properties: 0x00000001
	DOMAIN_PASSWORD_COMPLEX
```

## Enumerating Users & Groups

| Command                  | Use                                                               |
| ------------------------ | ----------------------------------------------------------------- |
| `enumdomusers`           | Enumerate all users.                                              |
| `enumalsgroups domain`   | Enumerate domain groups.                                          |
| `enumalsgroups builtin`  | Enumerate local system groups.                                    |
| `enumdomains`            | Enumerate domain information.                                     |
| `enumprivs`              | Enumerate user system privileges.                                 |
| `lookupnames [USERNAME]` | Identify the SID for the username.                                |
| `queryuser [RID]`        | Identify user information for the given users relative ID number. |

## Create a New User

Create a new user on the remote Windows system using rpcclient with the `createdomuser` command.

```bash
rpcclient $> createdomuser [USERNAME]
rpcclient $> setuserinfo2 [USERNAME] 24 '[NEW_PASSWORD]'
```

In this example, the `24` value represents necessary _**Windows information class constant**_ to set a user\
password. The value will _always_ be 24 when setting a password.

## Create a New Share

Create a new share on the remote Windows system using rpcclient with the `netshareadd` command.

```bash
rpcclient $> netshareadd "C:\Windows" "Windows" 10 "Windows Share"
```

## Change a Users Password

Change a user's password on the Windows system using rpcclient with the `chgpasswd3` command.

```bash
rpcclient $> chgpasswd3 [USERNAME] [OLD_PASSWORD] [NEW_PASSWORD]
```

## Resources

Official Samba docs:

[https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html)

SANS Institute rpcclient cheat sheet:

[https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf](https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf)
