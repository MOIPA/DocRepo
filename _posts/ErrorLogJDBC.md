---
title: ErrorLogJDBC
date: 2019-03-15 09:18:53
tags: error
categories: Log
---

# Loading class `com.mysql.jdbc.Driver'.

<!--more-->

用了最新的mysql 连接驱动导致报错



```
jdbc.driverClass   = com.mysql.dbc.Driver
jdbc.url      = jdbc:mysql://127.0.0.1:3306/db?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT
jdbc.username = root
jdbc.password = root123
```



现在按照最新官方提示支持将com.mysql.jdbc.Driver  改为  com.mysql.cj.jdbc.Driver



```
jdbc.driverClass   = com.mysql.cj.jdbc.Driver
jdbc.url      = jdbc:mysql://127.0.0.1:3306/db?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT
jdbc.username = root
jdbc.password = root123
```

