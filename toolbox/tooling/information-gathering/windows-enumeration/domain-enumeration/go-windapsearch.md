# go-windapsearch

`windapsearch` is a tool to assist in Active Directory Domain enumeration through LDAP queries. It contains several modules to enumerate users, groups, computers, as well as perform searching and unauthenticated information gathering.

{% hint style="info" %}
#### Download & Install

[https://github.com/ropnop/go-windapsearch](https://github.com/ropnop/go-windapsearch)
{% endhint %}

***

## Cheatsheet



***

## Basic Commands

### Query Users in a Domain

```bash
./windapsearch -d <DOMAIN> --dc-ip <IP> -U
```

The `-U` flag is used to enumerate all users, i.e., objects with `objectCategory` set to `user`.

### Query all Domain Objects

```bash
./windapsearch -d <DOMAIN> --dc-ip <IP> --custom "objectClass=*"
```
