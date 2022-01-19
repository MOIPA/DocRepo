title: centOS防火墙
author: tr
tags:
  - centOS
categories:
  - centOS
date: 2019-04-04 13:58:00
---
#### 防火墙操作

<!--more-->

1. firewall-cmd --zone=public --add-port=3000/tcp --permanent

2. firewall-cmd --reload

3. firewall-cmd --list-all

4. systemctl restart firewalld
