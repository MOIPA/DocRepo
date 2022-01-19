title: Docker 深入1
author: tr
tags:
  - Docker
categories:
  - Docker
date: 2020-01-22 11:06:00
---
# Docker 学习

<!--more-->

## Docker出现的历史和原因

开发环境，测试环境，生产环境，要配置很多东西，nginx端口，web端口，后端，注册中心配置等等。版本问题也会导致服务不可用，运维成本比较大。

主要还是环境配置很麻烦，特别是集群的话，每个机器安装一遍，费时费力

发布项目时可能发个jar或者war包，但是需要环境才能运行这个包，最好能把环境一起打包发过去。

Docker就是服务于开发解决上述问题的解决方案。

从而达成了：java -- jar包 -- 打包项目带上环境（镜像） --- 放入docker仓库 --- 下载镜像--运行镜像


10年发布，后来docker开源才火了起来，因为大家发现docker比起vm这些软件是真的轻量。

Docker和虚拟机本质都是虚拟化技术，虚拟机需要虚拟化所有硬件，容器只关注软件本身，只安装最核心的东西（linux核心也就一些指令大概4m）

## docker本身

Docker基于Golang开发，是个开源项目

Docker推荐阅读官方文档，文档非常详细，可以看看dockerHub类似github

Docker容器是没有自己的内核的（虚拟机有完整的内核），但有自己独特的环境，容器之间是共享内核的。

image：docker镜像类似与java中的类概念
container：容器，类似与java中的对象的概念，容器是从镜像过来的，容器之间是隔离的
repository：仓库，存放image的地方

## docker初步安装和运行

根据docker官方安装文档装下来就行了，可能要改下源，安装好后`docker version`查看docker版本

执行`docker run hello-world`测试hello world

执行`docker images`查看镜像

如果是linux系统，资源一般放在/var/lib/docker内

执行`docker run ...`发生的事：docker会先去本地仓库寻找镜像，如果找不到回去docker hub找对应镜像并且下载，如果用的阿里云服务器，可以配置镜像加速地址，就不去docker hub下载了直接从阿里云下载更快点。

docker是cs架构的系统，Docker的守护进程运行在主机，客户端命令是通过socket发送通信的，DockerServer接收后执行指令

## docker常用指令

所有指令可以在官网看到

帮助命令：
```
docker version
docker info  显示当前运行状态
docker <Command> --help
```
镜像命令：
```
docker images 查看镜像 -a显示全部 -q只显示id

docker search 查询镜像 --filte=stars=100 搜索stars大于100的镜像 

docker pull 下载镜像 例：docker pull mysql:5.7(带版本) docker下载采用了分层的方式

docker rmi -f <id> 通过镜像id删除，或者 docker image rm <id>

批量删除：docker rmi -f $(docker images -aq)
```

容器命令：
运行创建容器
```shell
docker run [参数] <镜像> 
# 参数说明
--name=""    #给容器起名
-d      	#类似nohup，静默运行
-it			 #进入容器查看内容
-p			#指定容器端口 -p 9999:8080
	主机端口：容器端口
    容器端口
-P 			#随机端口
-c			#写入命令
--rm		#容器启动后删除
--net		#网络配置
#例子 启动centos 并且写入命令（一段shell脚本）
docker run -it --name="mycentos" centos #直接进入执行
 docker run -d --name="mycentos" centos /bin/bash -c "while true;do echo hello world;sleep 1;done" 
```
查看容器
```shell
docker ps
		-a #查看所有的容器
        -n=? #显示最近创建的容器
        -q #只显示id
```

退出容器
```shell
exit #容器停止并且推出
ctrl + P + Q #容器不停止退出
```

删除容器
```shell
docker rm <容器id> #不能删除正在运行的容器除非加上 -f
docker rm $(docker ps -aq) #删除全部容器
```

启动停止容器
```shell
docker start <id>
docker stop <id>
docker restart <id>
docker kill <id> #暴力停止
```

进入当前运行的容器
```shell
1. docker exec -it <容器id> /bin/bash
2. docker attach <容器id> 
# 区别： 
方式1 docker exec
进入容器后开启新的终端
方式2 docker attach
进入容器正在执行的终端
```

容器和主机文件传递
```shell
docker cp <容器id:路径> 目的主机路径 #容器到主机，反过来就是主机到容器

docker 

#例子 docker cp mycentos:/home/mycentosConfig ./


```

查看镜像源数据
```shell
docker inspect --help
#里面包含了容器id，挂载信息，网络信息等等等等
```

其他命令
```shell
#后台启动
docker run -d <镜像id>

例子：docker run -d centos 
# 会发现centos停止了，因为容器后台运行必须要有前台进程，如果没有自动停止

#查看日志
docker logs [参数] <id>
	-f 实时显示
    --tail 11 显示最后11条
    -t 显示时间
    
# 查看进程信息 类似top命令
docker top <容器id>

#查看cpu使用状态
docker stats <容器Id>
```

## docker 常用示例

### 安装配置nginx

可以去官网搜索nginx，看看有哪些版本和使用日志

 
![upload successful](/images/pasted-92.png)

启动nginx并且映射一个端口到主机，nginx默认80端口，这里映射到本地9999
`docker run -d --name mynginx -p 9999:80 nginx`

启动后可以通过浏览器或者命令访问nginx服务：`curl http://localhost:9999`

也可以再次进入容器，找到nginx的配置文件
```shell
docker exec -it mynginx /bin/bash
whereis nginx
```

![upload successful](/images/pasted-93.png)

因为要经常修改nginx配置文件，所以最好挂载数据卷。

### 安装配置tomcat

可以直接用官方的命令，里面的rm参数是用完即删除容器的，最好不要加上

![upload successful](/images/pasted-94.png)

装好的可以exec进入，但是缺很多命令，因为docker安装的是最小可运行环境

### 安装部署ES+kibana

1. es端口多还很耗内存，数据得放到安全目录

2. 还是推荐使用官方的命令


![upload successful](/images/pasted-95.png)

先不用network
`$ docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:tag
`
太耗费内存和cpu，需要增加限制
`$ docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:8.0 -e ES_JAVA_OPTS="-Xms64m -Xmx512m"
`
