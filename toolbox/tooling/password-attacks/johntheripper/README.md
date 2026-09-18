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

# JohnTheRipper

**John the Ripper** (aka. **JtR** aka. **john**) is a well-known penetration testing tool used for cracking passwords through various attacks including brute-force and dictionary.

## Cracking Modes

**Single Crack Mode** Single crack mode is a rule-based cracking technique that is most useful when targeting Linux credentials. It generates password candidates based on the victim's username, home directory name, and GECOS values (full name, room number, phone number, etc.). These strings are run against a large set of rules that apply common string modifications seen in passwords (e.g. a user whose real name is Bob Smith might use Smith1 as their password).

```shell
clue@machine[/]$ john --single passwd
```

**Wordlist Mode** Wordlist mode is used to crack passwords with a dictionary attack, meaning it attempts all passwords in a supplied wordlist against the password hash. The basic syntax for the command is as follows:

```shell
clue@machine[/]$ john --wordlist=<wordlist_file> <hash_file>
```

**Incremental Mode** Incremental mode is a powerful, brute-force-style password cracking mode that generates candidate passwords based on a statistical model (Markov chains). It is designed to test all character combinations defined by a specific character set, prioritizing more likely passwords based on training data.

This mode is the most exhaustive, but also the most time-consuming. It generates password guesses dynamically and does not rely on a predefined wordlist, in contrast to wordlist mode. Unlike purely random brute-force attacks, Incremental mode uses a statistical model to make educated guesses, resulting in a significantly more efficient approach than naïve brute-force attacks.

```shell
clue@machine[/]$ john --incremental <hash_file>
```

## Display Cracked Hash

```shell
clue@machine[/]$ john ssh.hash --show

SSH.private:1234

1 password hash cracked, 0 left
```
