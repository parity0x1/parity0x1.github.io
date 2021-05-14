---
layout: post
title: "gobuster"
keywords: "scanning, gobuster"
---
#### Default scan
dir: Uses directory/file enumeration mode  
-u: The target URL  
-w: Path to the wordlist  
-x: File extension(s) to search for

```
gobuster dir -u $URL -w /usr/share/wordlists/dirb/common.txt -x .php
```