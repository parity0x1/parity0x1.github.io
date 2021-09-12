---
layout: post
title: "smb"
keywords: "smb, samba"
---

### Resources
[Enumerate SMB with Enum4linux & Smbclient](https://null-byte.wonderhowto.com/how-to/enumerate-smb-with-enum4linux-smbclient-0198049/)

### Nmap Scan
```
139/tcp open netbios-ssn Samba smbd 3.X - 4.X
445/tcp open netbios-ssn Samba smbd 4.7.6-Ubuntu

sudo nmap -p 139,445 -sC $IP -v
```

### Enum4Linux
[Enum4linux](https://github.com/portcullislabs/enum4linux) is a tool used to enumerate SMB shares on both Windows and Linux systems.
```
enum4linux -a $IP | tee enum4linux
```

### SMBClient
Because we're trying to access an SMB share, we need a client to access resources on servers.
```
smbclient //$IP/[SHARE] -U [NAME] -p [PORT]
```

### Anonymous Access
Once connected to the share, we can use the `get` / `mget` command to download files to our local machine.
```
smbclient //$IP/[SHARE] -U Anonymous

smbclient //$IP/[SHARE] -N -c 'prompt OFF;recurse ON;cd 'path\to\directory\';lcd '~/path/to/download/to/';mget *'`
```