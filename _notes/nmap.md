---
layout: post
title: "nmap"
keywords: "scanning, nmap"
---
#### Default scan
-sC: equivalent to --script=default  
-sV: Probe open ports to determine service/version info  
-oN: Output scan in normal

```
sudo nmap -sC -sV $IP -oN nmap
```