title: 数据库常用命令
author: tr
tags: []
categories: []
date: 2021-01-31 07:00:00
---
# 数据库常用命令

```
登录数据库：

sudo mysql -u root -p778899      
注意哦，不输入sudo会出现一下问题：

Access denied for user 'root'@'localhost'

显示数据库：

show databases；
创建数据库：

create database school；
选择当前操作数据库：

use school;
创建新用户：

create user shaowen@'%' identified by '密码'；
授权：

grant all on *.* to shaowen@'%';
//刷新
flush privileges;
收回权利：

revoke all ON *.*  FROM shaowen@'%';
flush privileges;
查询所有用户：

SELECT User, Host FROM mysql.user;
修改密码：

alter user 'shaowen'@'%' identified by '778899';
//%和localhost具体填哪个，根据上面的查询用户选项看
flush privileges;
之后就可以登录啦

sudo mysql -ushaowen -p778899
```