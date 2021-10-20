---
layout: post
title: "sqli"
keywords: "injection, mysql"
---

### Broken Authentication
```
Try register a new user, using a known username but starting with a whitespace.
```

### SQLite
```
ls
file example.db
sqlite3 example.db
> .tables
> PRAGMA table_info(customers)
> SELECT * from customers;
```