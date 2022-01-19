title: SonarQube配置集成到Jenkins
author: tr
tags:
  - SonarQube
categories:
  - Jenkins
date: 2019-04-22 09:45:00
---
### SonarQube

onarQube简介
Sonar 是一个用于代码质量管理的开放平台。通过插件机制，Sonar 可以集成不同的测试工具，代码分析工具，以及持续集成工具。比如pmd-cpd、checkstyle、findbugs、Jenkins。通过不同的插件对这些结果进行再加工处理，通过量化的方式度量代码质量的变化，从而可以方便地对不同规模和种类的工程进行代码质量管理。同时 Sonar 还对大量的持续集成工具提供了接口支持，可以很方便地在持续集成中使用 Sonar。
此外，Sonar 的插件还可以对 Java 以外的其他编程语言（支持的语言包括：Java、PHP、C#、C、Cobol、PL/SQL、Flex等）提供支持，对国际化以及报告文档化也有良好的支持。可以说Sonar是目前最强大的代码质量管理工具之一。

<!--more-->

#### 下载

sonarqube下载地址：http://www.sonarqube.org/downloads/

下载配置JDK，参见博客：配置jdk

配置MySql：

```
mysql -u root -p
mysql> CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci;
mysql> CREATE USER 'sonar' IDENTIFIED BY 'sonar';
mysql> GRANT ALL ON sonar. TO 'sonar'@'%' IDENTIFIED BY 'sonar';
mysql> GRANT ALL ON sonar. TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
mysql> FLUSH PRIVILEGES;

这段的权限赋予可能有点问题，改为sonar.*，其他错误自行尝试，感觉是mysql版本的问题
```
#### 安装启动

官网下载后，解压后移动到/usr/local目录下

vim /etc/profile
```
export SONAR_HOME=/usr/local/sonarqube-6.7
export SONAR_RUNNER_HOME=/usr/local/sonar-runner-2.4
PATH=$PATH:$SONAR_HOME/bin:$SONAR_RUNNER_HOME/bin
```

source /etc/profile

配置/usr/local/sonarqube-6.7/conf/sonar.properties：

![upload successful](/images/pasted-1.png)


![upload successful](/images/pasted-2.png)

启动命令：

用root无法启动lSonarQube，需要另外新建普通用户来启动

useradd esadmin

chown -R esadmin.esadmin sonarqube-6.7

vim elasticsearch/config/elasticsearch.yml

```
 55 network.host: 0.0.0.0
 56 #
 57 # Set a custom port for HTTP:
 58 #
 59 http.port: 9200

```

su esadmin

./bin/linux-x86-64/sonar.sh start


启动成功

日后可以使用脚本启动，脚本中这么写：
```
  1 # start up sonarqube
  2 sudo su - esadmin -c '/usr/local/sonarqube-7.7/bin/linux-x86-64/sonar.sh restart &'

```
#### 报错

启动的时候会报错

1.错误：max file descriptors [4096] for elasticsearch process is too low, increase to at least

解决方法：
[root@localhost ]#vim /etc/security/limits.conf

esadmin hard nofile 65536

esadmin soft nofile 65536

在文件最后添加以上两行

查看文件限制命令

[esadmin@localhost sonarqube-6.7]$ ulimit -Hn
Centos 7环境安装SonarQube和SonarQube Runner

2.错误，max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

解决方法：
切换到root用户修改配置sysctl.conf

[root@localhost]#vim /etc/sysctl.conf

添加下面配置：
vm.max_map_count=655360

并执行命令：
[root@localhost java]# sysctl -p

解决好以上错误后就可以重启SonarQube

[esadmin@localhost sonarqube-6.7]$ ./bin/linux-x86-64/sonar.sh start

[root@localhost sonarqube-6.7]# ps -aux |grep sonar

#### 使用

1. 默认密码和用户名都是：admin

2. 生产token：在我的账户下面点击security