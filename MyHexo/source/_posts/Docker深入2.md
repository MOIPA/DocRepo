title: Docker深入2
author: tr
tags:
  - Docker
categories:
  - Docker
date: 2020-01-23 10:52:00
---
# Docker深入2

<!--more-->

## portainer可视化界面安装

```shell
docker run -d 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer 
```

访问测试

![upload successful](/images/pasted-96.png)

如果是在windos中 挂载数据卷需要换

![upload successful](/images/pasted-97.png)

## Docker镜像探究

镜像类似一个独立软件包，但这个软件包包含了代码，运行时的库，环境变量和配置文件。

得到镜像：从远程仓库，或者别人复制，或者自己做（DockerFile）

### Docker镜像加载原理

下载镜像的时候可以看到分了很多层，其实就是`UnionFs文件系统`，它可以对文件系统的修改作为一次次的提交，类似于`git`（比如安装一个jenkins，先安装centos内核，安装库，安装jdk，最后安装jenkins本身，多个应用可能共有安装内核和库的行为），下载时候看到的一层一层的就是这个提交记录。假设两个镜像都用到了同样的几个记录，这几个记录就不用下载了。

Docker镜像需要bootfs:最底层的启动器包含bootloader和kernel（精简版内核），前者用于引导加载bootfs文件系统，加载内核。当boot完成后内核就存在在内存中了，此时内存使用权转移给内核，系统卸载bootfs

还有rootfs：rootfs就是在bootfs之上的各个系统的发行版（ubuntu，centos等）

### Docker镜像分层原理

docker下载镜像的时候回下载多次记录，下载过的记录不再下载，可以通过`docker inspect`查看每层的记录

具体原理：和git一样，每层（ly:layer)认为一个文件记录，下一层可能新增，或者修改，或者删除了某些文件，这一层的操作为一个记录。

### commit镜像

```shell
docker commit 提交容器成为新副本

docker commit -m "" -a "作者" <镜像名>:[Tag]

```

tomcat为例，官方的镜像内webapps是没有东西的，可以手动将webapps.list内容拷贝过去，重新打包成一个镜像


![upload successful](/images/pasted-98.png)

这个镜像现在可以发布给任何人了

## Docker的容器数据卷

一个闭合的容器没有太大意义，比如配置多的nginx，每次修改配置需要取容器内部vim修改，可以通过容器内和容器外共享文件吗。答案是肯定的，将内部容器的一个目录挂载到主机上的一个目录，实现自动同步目录内容虽然本质是硬链接。

使用挂载卷，每次创建容器时输入`参数：-v`即可。

以centos为例，从镜像启动一个容器时：
```shell
docker run -it -v /d/DockerV:/home/ centos /bin/bash #启动时挂载卷

docker inspect #可以查看挂载的卷
```

![upload successful](/images/pasted-99.png)

查看挂载好的卷

![upload successful](/images/pasted-100.png)

### 具名挂载和匿名挂载（不指定主机目录的情况）

```shell
-v 容器内路径 
#匿名挂载
#例子 docker run -d -P --name nginx01 -v /etc/nginx nginx  # -P 随机选择主机 端口

#查看所有数据卷
docker volume ls #会看到大部分卷是哈希数，因为当时挂载的时候没有起名

#具名挂载
```

![upload successful](/images/pasted-102.png)

如果是具名挂载那么数据卷默认挂载到仓库

![upload successful](/images/pasted-103.png)

### 挂载数据卷的读写权限配置

```shell
#加上参数 :rw 即可
docker run -d -v myvolume:/etc/nginx:ro nginx
# rw:可读可写
# ro:只可读
```

### Dockfile 挂载数据卷

DockerFile只是用来构建docker镜像的脚本，每个命令都是一层修改(commit)
```shell
# dockerfile里面的内容
# dockerfile里面的内容
#以什么系统作为基础
FROM centos
#匿名挂载数据卷 这里的配置在运行这个镜像的时候会自动将两个卷挂载到宿主机
VOLUME ["volume01","volume02"] 

CMD echo "---build finished---"
```
将这个文件用作为我们的构建脚本
```shell
docker build -f .\dockerfile -t tr/centos ./
```

![upload successful](/images/pasted-104.png)

### 数据卷容器

就是创建一个卷，这个数据卷也运行在一个容器里。

假设有一个需求，需要几个容器之间做数据的同步，需要参数`--volumes-from #同步挂载点操作` 子容器挂载父容器（数据卷容器）

例子：启动三个容器（最好是自己构建的镜像）

![upload successful](/images/pasted-105.png)

```shell
PS D:\DockerV\docker-volume1> docker run -itd --name trcentos01 tr/centos /bin/bash
e14d5b0f7558eff1028427bd47ded7d85dd1087010a6dab8ed777bca41e59b2b
PS D:\DockerV\docker-volume1> docker run -itd --name trcentos02 --volumes-from trcentos01 tr/centos /bin/bash
9d89c8cffb07f14b51853a7d79f9b125ab339e00ed97638ce35a77fe42f516b8
PS D:\DockerV\docker-volume1>

--itd ：后台运行centos但是不进入命令行
```
这样两个容器内的两个卷数据完全同步了。且删除随便一个容器，卷内的数据也不会丢失。

## Dockfile

DockeFile是用来构建docker镜像的文件，一般开发完就得写，简单的很。

需要注意的是，dockerfile编写的时候不能在命令后面写注释。


![upload successful](/images/pasted-106.png)

要注意的指令：
```shell
onbuild: 这是一个特殊指令，它后面跟的是其他指令如run，copy等，而这些指令当前构建不会执行，只有以当前镜像为基础镜像来构建下一级镜像的时候才会执行
cmd: 容器启动执行的一些命令,如果run的时候指定了参数，会覆盖原来的参数，和run不同，run是构建镜像过程中执行的命令，通常用来安装一些软件
entrypoint: 容器启动执行的一些命令,如果run的时候指定了参数，会追加原来的参数
``` 

例子：构建自己的centos
```shell
FROM centos
MAINTAINER tangrui<1540525748@qq.com>
#配置环境变量
ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools
RUN yum -y install cowsay

EXPOSE 80

CMD echo $MYPATH
CMD echo "--build finished--"
CMD /bin/bash
```

构建镜像：`docker build -f dockerfile -t tr/centos:2.0 .`

查看构建历史：`docker history`

### cmd和entrypoint用法

```shell
在dockerfile中cmd和entrypoin使用方式有三种：
CMD ["executable","param1","param2"] (exec form, this is the preferred form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
CMD command param1 param2 (shell form)

#举例
CMD ls -a
CMD echo "hello"

CMD ["ls","-a"]
CMD ["echo","hello"]

ENTRYPOINT ["ls","-a"]
```
使用的区别：在如果dockerfile内使用了cmd，那么`docker run ... ls -a`最后一个命令会覆盖之前dockerfile内的命令

如果使用了entrypoint，`docker run ... -al`那么会在里面的参数追加`al`参数

### Tomcat镜像

1. tomcat需要jdk和tomcat的压缩包
2. 编写dockerfile文件(如果dockerfile名字就是`Dockerfile`则不需要指定文件了

下载好文件后执行build即可

![upload successful](/images/pasted-107.png)
```shell
FROM centos
MAINTAINER tangrui<1540525748@qq.com>

COPY readme.txt /usr/local/readme.txt

ADD jdk-8u311-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.56.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local/
WORKDIR $MYPATH

# 配置javahome
ENV JAVA_HOME /usr/local/jdk1.8.0_311
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# 配置tomcat
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.56
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.56

# 写入系统环境变量PATH
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

# 暴露端口
EXPOSE 8080

# 启动tomcat

CMD /usr/local/apache-tomcat-9.0.56/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.56/logs/catalina.out
```
```shell
构建：docker build -t tr/tomcat .
启动：docker run -itd -P -v /d/DockerV/myTomcat/TomcatHome/webapps:/usr/local/apache-tomcat-9.0.56/webapps -v /d/DockerV/myTomcat/TomcatHome/logs:/usr/local/apache-tomcat-9.0.56/logs --name trtomcat tr/tomcat
```

至此tomcat已经完毕，可以放入一个项目到webapps内测试如图：

![upload successful](/images/pasted-109.png)

访问测试：

![upload successful](/images/pasted-110.png)

## Docker发布镜像

命令行里使用 `docker push`

建议先去官网点击用户下的安全设置，生成一个安全口令作为密码。

![upload successful](/images/pasted-111.png)
拒绝是因为发送的docker镜像必须符合`用户名/...`的格式，可以通过`docker  tag`重命名镜像

![upload successful](/images/pasted-112.png)

也推荐使用docker desktop

![upload successful](/images/pasted-113.png)

## 本地保存镜像

```shell
docker save spring-boot-docker  -o  /home/tr/docker/spring-boot-docker.tar

docker load -i spring-boot-docker.tar  
```