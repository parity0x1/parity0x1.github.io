---
layout: post
title: "smtp"
keywords: "simple mail transfer protocol"
---

SMTP stands for "Simple Mail Transfer Protocol". It is utilized to handle the sending of emails. In order to support email services, a protocol pair is required, comprising of SMTP and POP/IMAP. Together they allow the user to send outgoing mail and retrieve incoming mail, respectively. 

### Enumeration
```
25/tcp open smtp Postfix smtpd
```

### MSF smtp_version
```
msfconsole
> use auxiliary/scanner/smtp/smtp_version
> options
> set rhosts <IP>
> run
```

### MSF smtp_enum
```
msfconsole
> use auxiliary/scanner/smtp/smtp_enum
> options
> set rhosts <IP>
> set user_file /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
> run
```

