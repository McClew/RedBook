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
---

# Caido



{% hint style="info" %}
#### Download & Install

_Recent versions of Kali Linux may have Caido in the repository._

Download via the official site:

[https://caido.io/](https://caido.io/)
{% endhint %}

***

## Tools

### Replay

The equivelant to Repeater in [burp-suite.md](burp-suite.md "mention").

* **Function:** Allows you to modify a single request and resend it multiple times to analyze the response.
* **Usage:** You can send requests here from the Proxy history to manually test for vulnerabilities.

### Automate

Equivelant to Intruder in [burp-suite.md](burp-suite.md "mention").

* **Function:** Fuzzing tool for sending many requests with varying payloads.
* **Key Advantage:** **No Rate Limiting.** Unlike Burp Community, Caido’s free version allows you to run "Automate" attacks at full speed.

### Workflows

An automation engine to chain actions (e.g., "If request matches X, convert body to Base64"). Includes Passive, Active, and Convert workflows.

### Assistant

A built-in AI chatbot (GPT-based) to help explain errors or suggest attacks.

### Environment

Similar to Postman, allows you to save variables (URL, tokens) for API testing.
