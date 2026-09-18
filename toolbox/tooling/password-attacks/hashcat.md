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

# Hashcat

**Hashcat** is a well-known password cracking tool, featuring fantastic GPU support, it can be used to crack a large variety of hashes. Similar to JtR, hashcat supports multiple attack (cracking) modes which can be used to efficiently attack password hashes.

The general syntax used to run hashcat is as follows:

```shell
clue@machine[/]$ hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]
```

In the command above:

* `-a` is used to specify the attack mode.
* `-m` is used to specify the hash type.
* `<hashes>` is a either a hash string, or a file containing one or more password hashes of the same type.
* `[wordlist, rule, mask, ...]` is a placeholder for additional arguments that depend on the attack mode.

## Attack Modes

Hashcat has many different attack modes, including **dictionary**, **mask**, **combinator**, and **association**.

**Dictionary Attack** Dictionary attack `-a 0` is, as the name suggests, a dictionary attack. The user provides password hashes and a wordlist as input, and Hashcat tests each word in the list as a potential password until the correct one is found or the list is exhausted.

```shell
clue@machine[/]$ hashcat -a 0 -m 0 e3e3ec5831ad5e7288241960e5d4fdb8 /usr/share/wordlists/rockyou.txt
```

**Mask Attack** Mask attack `-a 3` is a type of brute-force attack in which the keyspace is explicitly defined by the user. For example, if we know that a password is eight characters long, rather than attempting every possible combination, we might define a mask that tests combinations of six letters followed by two numbers.

A mask is defined by combining a sequence of symbols, each representing a built-in or custom character set. Hashcat includes several built-in character sets:

| Symbol | Charset                                 |
| ------ | --------------------------------------- |
| ?l     | abcdefghijklmnopqrstuvwxyz              |
| ?u     | ABCDEFGHIJKLMNOPQRSTUVWXYZ              |
| ?d     | 0123456789                              |
| ?h     | 0123456789abcdef                        |
| ?H     | 0123456789ABCDEF                        |
| ?s     | «space»!"#$%&'()\*+,-./:;<=>?@\[]^\_\`{ |
| ?a     | ?l?u?d?s                                |
| ?b     | 0x00 - 0xff                             |

Custom charsets can be defined with the `-1`, `-2`, `-3`, and `-4` arguments, then referred to with `?1`, `?2`, `?3`, and `?4`.

Example usage:

```shell
clue@machine[/]$ hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

**Combinator Attack**

**Association Attack**

## Rules

Rules can be used to perform specific modifications to passwords to generate even more guesses. The rule files that come with hashcat are typically found under `/usr/share/hashcat/rules`.

To perform this kind of attack, append the `-r <ruleset>` option to the command.

## Wordlist Creation

Hashcat can be used to combine lists of potential names and labels with specific mutation rules to create custom wordlists. Hashcat uses a specific syntax to define characters, words, and their transformations.

| Function | Description                                      |
| -------- | ------------------------------------------------ |
| :        | Do nothing                                       |
| l        | Lowercase all letters                            |
| u        | Uppercase all letters                            |
| c        | Capitalize the first letter and lowercase others |
| sXY      | Replace all instances of X with Y                |
| $!       | Add the exclamation character at the end         |

Each rule is written on a new line and determines how a given word should be transformed. If we write the functions shown above into a file, it may look like this:

```shell
clue@machine[/]$ cat custom.rule

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

Use the following command to apply the rules in `custom.rule` to each word in `password.list` and store the mutated results in `mut_password.list`.

```shell
clue@machine[/]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

In this case, the single input word will produce fifteen mutated variants:

```shell
clue@machine[/]$ cat mut_password.list

password
Password
passw0rd
Passw0rd
p@ssword
P@ssword
P@ssw0rd
password!
Password!
passw0rd!
p@ssword!
Passw0rd!
P@ssword!
p@ssw0rd!
P@ssw0rd!
```
