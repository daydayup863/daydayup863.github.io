---
title: python3 ldap3 NTLM验证用户
date: 2020-12-18 17:40:54
tags: 
- python3
- ldap
categories: 
- python
top: 1
description: 
password: 

---
```
from ldap3 import Server, Connection, NTLM, ALL

server = Server("xxx.xxx.xxx.xxx", 389, use_ssl=False, get_info=ALL)
print(server)
conn = Connection(server, user="Domain\\username", password="password", authentication=NTLM)
print(conn.bind())
print(conn.extend.standard.who_am_i())

```

