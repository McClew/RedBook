# BloodHound.py

> BloodHound.py is a Python based ingestor for [BloodHound](https://github.com/BloodHoundAD/BloodHound), based on [Impacket](https://github.com/CoreSecurity/impacket/).
>
> The code in this branch is **only compatible with BloodHound CE**.

{% hint style="info" %}
#### Download & Install

Install via apt:

`sudo apt install bloodhound-ce-python`



Install via GitHub:

[https://github.com/dirkjanm/BloodHound.py/tree/bloodhound-ce](https://github.com/dirkjanm/BloodHound.py/tree/bloodhound-ce)
{% endhint %}

***

## Setup

{% stepper %}
{% step %}
### Download the Git Repo

```bash
git clone https://github.com/dirkjanm/BloodHound.py
```
{% endstep %}

{% step %}
### Check Out the CE Branch

```bash
cd BloodHound.py
```

```bash
git checkout bloodhound-ce
```
{% endstep %}

{% step %}
### PIP Install

```bash
pip install .
```
{% endstep %}
{% endstepper %}

***

## Basic Commands

{% code overflow="wrap" %}
```bash
bloodhound-ce-python -u '<USER>' -p '<PASS>' -d '<DOMAIN>' -dc '<HOST_NAME>.<DOMAIN>' -ns '<DC_IP> -c All --zip
```
{% endcode %}
