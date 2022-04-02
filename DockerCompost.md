title: DockerCompose
author: tr
tags:
  - Docker
categories: []
date: 2022-03-31 09:45:00
---
# Docker compose

<!--more-->
[官方学习文档](https://docs.docker.com/compose/gettingstarted/)

## 出现的原因

DockerFile有了以后，build，run只针对一个容器，效率不高，如果多个服务之前存在依赖关系很麻烦

Docker Compose 是用来定义和运行多个容器的应用，使用yaml配置文件

正常使用有三个步骤：

1. 定义应用的环境（Dockefile）
2. docker-compose.yml 用来定义服务
3. 运行 docker compose up 启动项目

这个其实也就比自己手动写脚本好用，还是很局限，本质是脚本方式启动电脑上的一些服务

> 对于docker compose的一些说明

> 如果使用的是官方示例可以发现启动的容器名字是文件夹的名字+副本数量

> 会默认创建对应的 文件夹_default的网络，yml配置文件内定义的若干个服务启动后，这些服务都在同一个网络下，之后互相访问可以直接通过容器名访问

## yml 规则

```shell
# 3层

version : '' #版本

service : #服务
	service1:web
    	images:
        build:
        volume:
        ...
    service2:redijs

# 其他配置 网络 卷 全局规则

volumes:
network:
configs:


```


## DockerSwarm

集群的部署方式