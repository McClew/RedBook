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

# hashid

hashID checks supplied hashes against a built-in list to suggest potential formats and can provide specific information for password cracking tools.

Basic usage:

```shell
clue@machine[/]$ hashid 193069ceb0461e1d40d216e32c79c704
```

| Tool          | Flag | Description                                     |
| ------------- | ---- | ----------------------------------------------- |
| HashCat       | -m   | Will list the corresponding module for HashCat. |
| JohnTheRipper | -j   | Will list the corresponding John format.        |
