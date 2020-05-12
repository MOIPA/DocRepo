title: jenkins启动失败log
author: tr
tags:
  - Log
categories:
  - Log
  - Jenkins
date: 2019-04-22 10:55:00
---
### Failed to start LSB: Jenkins Automation Server.

这个错误可能是由于服务器配置了自己的jdk，导致jenkins找不到jdk启动，jdnkins没有使用系统的JDK_HOME

解决：vim /etc/init.d/jenkins

在candinate里面找到java相关的描述，把自己的java路径添加进去