---
icon: screwdriver-wrench
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

# Tooling

The "Tooling" section provides a practical compendium of the various software applications, scripts, and utilities utilised throughout a cybersecurity assessment or penetration testing engagement.

Its objective is to serve as a quick reference, enabling practitioners to efficiently identify and recall the appropriate instruments for specific tasks, thereby streamlining the execution of assessment methodologies.

{% hint style="warning" %}
## Windows Tool Transfer

Depending on how we gain access to a system we may not have many directories that are writeable by our user to upload tools. It is always a safe bet to upload tools to `C:\Windows\Temp` because the `BUILTIN\Users` group has write access.
{% endhint %}
