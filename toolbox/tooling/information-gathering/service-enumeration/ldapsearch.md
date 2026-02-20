# ldapsearch

{% hint style="info" %}
#### Download & Install

Via apt package manager:\
`sudo apt install ldap-utils`
{% endhint %}

{% hint style="info" %}
#### Official Documentation

[https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html](https://docs.ldap.com/ldap-sdk/docs/tool-usages/ldapsearch.html)
{% endhint %}

***

## Cheatsheet



***

## Distinguished Names

Say we find a domain with a [distinguished name](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ldap/distinguished-names) of `"DC=acme,DC=com"`. This means this Domain Controller has a top-level domain of `acme.com`.

If the user email is `admin@acme.com`, then the DistinguishedName is `CN=admin,DC=acme,DC=com`.

***

## Enumeration

### Query the Domain Context

```bash
ldapsearch -x -H ldap://<IP> -s base namingcontexts
```

### Query Domain Properties

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://<IP> -D 'CN=admin,DC=acme,DC=com' -W -b 'DC=acme,DC=com'
```
{% endcode %}

### Query all Objects in the Directory

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://<IP> -D 'admin@acme.com' -W -b 'DC=acme,DC=com' 'objectClass=*'
```
{% endcode %}

### Query all Users in the Domain

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://<IP> -D 'admin@acme.com' -W -b 'DC=AD,DC=LAB' '(objectClass=user)'
```
{% endcode %}

### Query all Groups in the Domain

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://<IP> -D 'admin@acme.com' -W -b 'DC=AD,DC=LAB' '(objectClass=group)'
```
{% endcode %}

### Query all Computer Accounts in the Domain

{% code overflow="wrap" %}
```bash
ldapsearch -x -H ldap://<IP> -D 'admin@acme.com' -W -b 'DC=AD,DC=LAB' '(objectClass=computer)'
```
{% endcode %}

### Query Members of a Group

{% code overflow="wrap" %}
```bash
group="Domain Users"
ldapsearch -x -H ldap://<IP> -D 'admin@acme.com' -W -b 'DC=AD,DC=LAB' "(sAMAccountName=${group})"
```
{% endcode %}
