---
title: docker配置jenkins
date: 2019-03-22 17:46:36
tags: docker
descriptions: docker配置 启动jenkins 常见docker命令
---

### docker是啥

<!--more-->

1. docker就是小型虚拟机，当然也有傻子那它当虚拟机
2. docker需要将自己的项目挂载在宿主机目录下，如果挂载了/etc这样的敏感目录，会导致风险

### docker配置

1. pacman -S docker 安装
2. docker info 查看信息

### docker 命令

1. systemctl start docker 启动docker

2. docker pull  jenkins     下载jenkins

3. docker run jenkins      也会自动下载 但是不推荐这么粗暴的使用

4. docker pull是下载镜像，镜像运行需要容器，使用docker image ls -a 可以查看所有镜像

   ​	使用docker container ls -a 查看所有的容器 包括不运行的容器，可以看到他们的ID

5. 删除image或者container：docker image rmi 镜像id

   ​						docker container rm 容器id

6. 运行一个镜像产生容器：docker run --name jenkins --user root -p 38080:8080 -p 50000:50000 -v /root/jenkins/data/:/var/jenkins_home jenkins -d

   说明：run-> 运行

   ​		--name -> 指定名字

   ​		-p -> 指定端口映射    宿主机访问端口 : docker虚拟镜像访问端口

   ​    		-v -> 挂载目录

   ​		-d -> 后台运行

7. docker镜像后台运行时想查看镜像：docker logs <跑起来的容器>

8. docker里面跑的容器其实就是虚拟机，如何进入这个虚拟机？：docker exec --user root -it 2645da9f9fc0 /bin/bash

   说明： --user root -> 以root用户进入虚拟机

9. 访问宿主机的38080端口
