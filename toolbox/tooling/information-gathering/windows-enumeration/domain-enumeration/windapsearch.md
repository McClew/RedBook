# windapsearch

`windapsearch` is a Python script to help enumerate users, groups and computers from a Windows domain through LDAP queries. By default, Windows Domain Controllers support basic LDAP operations through port 389/tcp. With any valid domain account (regardless of privileges), it is possible to perform LDAP queries against a domain controller for any AD related information.

{% hint style="info" %}
#### Download & Install

[https://github.com/ropnop/windapsearch](https://github.com/ropnop/windapsearch)
{% endhint %}

{% hint style="warning" %}
#### Superseded

This tool has been superseded by [go-windapsearch.md](go-windapsearch.md "mention").
{% endhint %}

***

## Cheatsheet



***

## Basic Commands

### Query Users in a Domain

```bash
./windapsearch.py -d <DOMAIN> --dc-ip <IP> -U
```

The `-U` flag is used to enumerate all users, i.e., objects with `objectCategory` set to `user`.

### Query all Domain Objects

```bash
./windapsearch.py -d <DOMAIN> --dc-ip <IP> --custom "objectClass=*"
```
