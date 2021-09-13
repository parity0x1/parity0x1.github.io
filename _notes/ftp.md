---
layout: post
title: "ftp"
keywords: "ftp, file transfer protocol"
---

### Enumeration
```
sudo nmap -sC -sV -p 21 $IP

21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
```

### Hydra
```
hydra -t 4 -l <USER> -P /usr/share/wordlists/rockyou.txt -vV $IP ftp
```