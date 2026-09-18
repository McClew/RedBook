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

# CeWL

**CeWL** can be used to scan potential words from a company's website and save them in a separate list. We can then combine this list with the desired rules to create a customized password list - one that has a higher probability of containing the correct password for an employee. We specify some parameters, like the depth to spider `-d`, the minimum length of the word `-m`, the storage of the found words in lowercase `--lowercase`, as well as the file where we want to store the results `-w`.

```shell
clue@machine[/]$ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
clue@machine[/]$ wc -l inlane.wordlist

326
```
