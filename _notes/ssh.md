---
layout: post
title: "ssh"
keywords: "access, ssh, keygen"
---

#### Generating a new SSH key

```
ssh-keygen
cat id_rsa
```

#### Adding an authorized SSH key to a remote machine

```
echo "{key}" >> /root/.ssh/authorized_keys
```
***Tip: If user doesn't have write access, leverage a Unix binary that does. See https://gtfobins.github.io***


#### Connecting to a remote machine using a local SSH key
```
ssh -i id_rsa root@{remote}
```