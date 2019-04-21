title: centOS常用命令
author: tr
tags:
  - centOS
  - firewall
categories:
  - centOS
date: 2019-04-03 10:13:00
---
### 常用命令

<!--more-->

#### 防火墙命令

1. 防火墙命令：
 
 	firewall-cmd --zone=public --add-port=6379/tcp --permanent
 	
  	firewall-cmd --reload
    
  	firewall-cmd --list-all