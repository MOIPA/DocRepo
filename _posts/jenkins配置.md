title: jenkins配置
tags: []
categories:
  - Jenkins
author: tr
date: 2019-03-22 18:06:00
---
### jenkins配置

<!--more-->

1. jenkins地址：/usr/share/jenkins/jenkins.war
2. 请在docker安装时  装最新的 ！！！！！
3. docker pull jenkins/jenkins:版本号
4. jenkins启动后需要进行配置：
   1. 配置JAVA_HOME，这需要你docker的jenkins镜像存在JAVA_HOME，在jenkins中可以设置目录
   2. 配置MAVEN_HOME，同上，使用apt命令下载后指定目录

### jenkins配置部署测试 （后端）

1. jenkins可以自动从代码库拉取分支的代码进行构建，并且进行junit测试，测试后还可以自动部署到服务器或者运行脚本

2. 创建maven项目，因为大部分项目都是maven，且可以执行maven命令：maven clean package这样的打包命令。

   示例：

   ![jenkins_工程1](/images/jenkins_工程1.png)

![jenkins_工程2](/images/jenkins_工程2.png)

![jenkins_工程3](/images/jenkins_工程3.png)

![jenkins_工程4](/images/jenkins_工程4.png)

![jenkins_工程5](/images/jenkins_工程5.png)

![jenkins_工程6](/images/jenkins_工程6.png)





### jenkins 配置部署项目（前端）

1. 我的前端项目是nvm环境下的，所以在jenkins中下载nvm插件

   1. 项目配置示例![jenkins_前端2](/images/jenkins_前端2.png)

   ![jenkins_前端1](/images/jenkins_前端1.png)