---
layout: post
title: "fuzzing"
keywords: "scanning, wfuzz, fuzzing"
---
#### WFUZZ
-c: Output with colors
-u: Specify a URL for the request  
-w: Specify a wordlist file  
--hc: Hide responses with the specified code  
-t: Specify the number of concurrent connections

```
wfuzz -c -u $URL -w /usr/share/wordlists/wfuzz/general/common.txt --hc 404 -t 100

wfuzz -c -z file,/usr/share/wordlists/dirb/small.txt $URL/FUZZ/flag.txt --hc 404 -t 100
wfuzz -c -z file,/usr/share/wordlists/dirb/small.txt $URL/api.php?FUZZ=id
```