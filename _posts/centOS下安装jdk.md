title: centOS下安装jdk
author: tr
tags:
  - centOS
  - jdk
categories:
  - centOS
date: 2019-04-04 13:47:00
---
#### 卸载系统自带的OpenJDK以及相关的java文件

<!--more-->

1. java -version
2. rpm -qa | grep java
  命令说明：

  rpm 　　管理套件    

  -qa 　　使用询问模式，查询所有套件

  grep　　查找文件里符合条件的字符串

  java 　　查找包含java字符串的文件
  
3. 在命令窗口键入：

  rpm -e --nodeps java-1.7.0-openjdk-1.7.0.111-2.6.7.8.el7.x86_64
 
4. 如果还没有删除，则用yum -y remove去删除他们

#### 安装官方jdk

1. wget 安装官方最新，可以安装到/usr/java/jdk1.8

2. tar -zxvf jdk-8u144-linux-x64.tar.gz 解压

#### 配置

1. 配置JDK环境变量
    ①编辑全局变量

    在命令行键入：
    vim /etc/profile

    在文本的最后一行粘贴如下：

    （注意JAVA_HOME=/usr/java/jdk1.8.0_144  就是你自己的目录）

    复制代码
    #java environment
    export JAVA_HOME=/usr/java/jdk1.8.0_144
    export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
    export PATH=$PATH:${JAVA_HOME}/bin

    复制代码
    【注】：CentOS6上面的是JAVAHOME，CentOS7是{JAVA_HOME}
    
    最后：source /etc/profile 生效