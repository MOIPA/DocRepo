title: InstantClient+PLSQL安装配置
author: tr
tags:
  - Oracle
categories:
  - Oracle
date: 2020-04-07 16:15:00
---
### oracle的连接客户端安装配置


<!--more-->

#### 下载

到官网或者找资源下载instantClient+PLSQL,注意版本

#### 配置

1. instantclient下载后解压，在instantclient_12_2文件下创建NETWORK文件夹,在NETWORK下创建ADMIN文件夹

2. 修改环境变量

     添加三个变量
  + ORACLE_HOME  D:\instantclient_11_2

  + TNS_ADMIN  变量值：D:\instantclient_12_2\NETWORK\ADMIN

  + 变量名:  NLS_LANG   变量值:SIMPLIFIED CHINESE_CHINA.ZHS16GBK

  + 修改Path变量，添加 %ORACLE_HOME%
  
  
3. 安装plsql

	下一步即可，安装完毕后在 工具--》首选项 或者 Configure--》Preferences中配置以下
    
![upload successful](/images/pasted-17.png)

   最后重启连接即可
    