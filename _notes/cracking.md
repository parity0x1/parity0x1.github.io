---
layout: post
title: "cracking"
keywords: "john, hashcat, passwords, hashes"
---

### Identifying Hashes
https://hashes.com/en/tools/hash_identifier
```
hash-identifier
```

### John The Ripper
```
john --list=formats | grep -iF "md5"
john --format=[format] --wordlist=[path to wordlist] [path to file]
```