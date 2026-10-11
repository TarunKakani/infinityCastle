---
title: "Sample Challenge (template)"
date: 2026-10-11 12:00:00 +0530
categories: [CTF Writeups]
tags: [pwn, template]
description: Sample CTF 2026 · pwn · Easy. Copy this file for each new writeup.
---

| Event | Category | Difficulty | Points |
| :--- | :--- | :--- | :--- |
| Sample CTF 2026 | pwn | Easy | 100 |

## Challenge

Paste the challenge description here, plus any files or connection details you were given.

## Recon

What you checked first: file type, protections (`checksec`), strings, a first look in Ghidra.

```console
$ checksec ./challenge
```

## Finding the bug

Explain the vulnerability and how you found it.

## Exploit

```python
from pwn import *

p = process('./challenge')
# exploit goes here
p.interactive()
```

## Flag

```text
flag{placeholder}
```

## Takeaways

What you learned from this one.
