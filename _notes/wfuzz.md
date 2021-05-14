---
layout: post
title: "wfuzz"
keywords: "scanning, wfuzz"
---
#### Default scan
-u: Specify a URL for the request  
-w: Specify a wordlist file  
--hc: Hide responses with the specified code  
-t: Specify the number of concurrent connections

```
wfuzz -u $URL -w /usr/share/wordlists/wfuzz/general/common.txt --hc 404 -t 100
```