---
layout: post
title: "brute-forcing"
keywords: "brute, wfuzz"
---
### Hydra
```
hydra -l <user> -P /usr/share/wordlists/rockyou.txt -t 1 -f <ip> http-get <path>
```