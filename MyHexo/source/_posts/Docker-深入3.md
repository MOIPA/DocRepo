title: Docker 深入3
author: tr
tags:
  - Docker
categories:
  - Docker
date: 2020-01-24 09:42:00
---
# Docker 深入3

<!--more-->

## Docker 网络

首先启动一个带net-tool的镜像（可以自己写个dockerfile构建下镜像或者进入容器安装` apt update && apt install -y iproute2`），查看容器的网卡信息


![upload successful](/images/pasted-114.png)

对于windows下，默认并没有docker的网卡，所以启动的容器可能ping不通，如果存在网卡，会发现网卡地址和容器内网卡地址处于一个网段下。

实验发现每启动一个容器，宿主机内会多一个虚拟网卡，正好和容器匹配，这个技术称为`veth-pair`

![upload successful](/images/pasted-115.png)

工作原理，就是个交换机（多网桥）


## 容器互联

### --link

类似dns，通过容器名访问

```shell
docker run -d -P --name tomcat03 --link tomcat02 tomcat
```
这样03即可ping通02，其实--link只是在启动的容器内添加了host配置，不过我感觉这种方式不好用，直接把host文件共享更方便配集群的话。

查看网络配置:`docker network ls` `docker network inspect`

### 自定义网络

直接启动的容器，默认会添加`--net bridge`参数

在默认的桥接模式下，宿主机会存在一个虚拟网卡用作所有容器的网关，容器之间通信都走这个网关。
```shell
# 创建网络:docker network create 
docker network create --driver bridge --subnet 192.168.0.0/24 --gateway 192.168.0.1 mynet

# 使用自己创建的网络
docker run -d -P --name trtomcat01 --net mynet tr/tomcat:1.0
# 自定义网络本质是划分网段，将容器放入网段下 查看网络
docker network inspect mynet
``` 

![upload successful](/images/pasted-116.png)

这样的最大好处是可以通过容器名访问了，且保证了只有子网内的容器是可以互相访问的
![upload successful](/images/pasted-117.png)

通过docker创建的网络，不同的网段是无法通信的，通过`docker network connect 网络 目标容器`联通

## REDIS集群

做这样一个模型，两个redis组合一个哨兵模式，共有三组

1. 组网，把redis集群放在一个局域网网段`docker network create --driver bridge --subnet 172.38.0.0/16 redisnet`

2. 启动容器

```shell
# 批量创建配置文件
#!/bin/bash
for port in $(seq 1 6);
do
mkdir -p ~/dockerPractice/redis/node-${port}/conf
touch  ~/dockerPractice/redis/node-${port}/conf/redis.conf
cat << EOF > ~/dockerPractice/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

# 批量启动redis容器
#!/bin/bash
for port in $(seq 1 6);
do
docker run -p 637${port}:6379 \
-p 1637${port}:16379 \
--name redis-${port} \
-v ~/dockerPractice/redis/node-${port}/data:/data \
-v ~/dockerPractice/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf -d \
--net redisnet \
--ip 172.38.0.1${port} \
redis \
redis-server /etc/redis/redis.conf
done
```
> 容器启动了，现在需要将这六个容器集群，这里可以随便进入一个redis容器，默认目录是/data,里面有节点配置文件，可以通过命令配置三个主机和从机:
```shell
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379
172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
```

![upload successful](/images/pasted-118.png)

> 使用集群测试，可以在集群内登陆：`redis-cli -c`,查看集群主从信息`cluster nodes`，键入一个值保存`set name tr`，这时候docker停止保存的主机，在重新进入redis-cli，获取`get name`发现可以在从机获取

## SpringBoot打包微服务

> 通过idea环境工具打包

![upload successful](/images/pasted-119.png)

> 可以配置idea的docker仓库

![upload successful](/images/pasted-120.png)

> 编写Dockerfile（和pom同一层级）


![upload successful](/images/pasted-121.png)

```shell
FROM java:8
COPY *.jar /app.jar
CMD ["--server.port=9080"]
EXPOSE 9080
ENTRYPOINT ["java","-jar","/app.jar"]
```

> 打包就行了 docker build

![upload successful](/images/pasted-127.png)

> 启动测试镜像

![upload successful](/images/pasted-128.png)