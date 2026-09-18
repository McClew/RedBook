---
icon: ethernet
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

# 88 - Kerberos

**Kerberos** is a network authentication protocol: it establishes identities and session keys, while each application service makes its own authorisation decision about the authenticated principal.

In environments like **Active Directory**, **Kerberos** is instrumental in establishing the identity of users by validating their secret passwords. This process ensures that each user’s identity is confirmed before they interact with network resources.

However, **Kerberos** does not extend its functionality to evaluate or enforce the permissions a user has over specific resources or services. Instead, it provides a secure way of authenticating users, which is a critical first step in the security process.

After Kerberos authentication, the target service evaluates the user’s rights and permissions using its local policy and any authorisation data carried in the ticket. This separates authentication from application-specific access control.

```
PORT   STATE SERVICE
88/tcp open  kerberos-sec
```
