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
    service2:redis
    	depends_on:
        - db
        - redis

# 其他配置 网络 卷 全局规则

volumes:
network:
configs:
```
一些重要的参数的详情

```shell
# 启动的时候是有一个启动顺序的，需要保证其他服务器启动
depends_on:
- db
# db 这里指的是自己定义的服务名


# image: 这里指的是镜像import的基础镜像
services:
  db:
    image: postgres

services:
	web:
      # 指令执行 镜像构建成功后执行的命令
      command: python manage.py runserver 0.0.0.0:8000	
      # 构建 根据当前目录的dockerfile文件构建这个镜像
      build: .
      # 暴露的端口 本地8000 容器5000
      ports:
        - "8000:5000"
      # 挂载卷，启动docker的目录，映射到容器的code目录
      volumes:
        - .:/code
      # 环境变量设置
      environment:
        FLASK_ENV: development


# volume 还有其他挂载方式 具名挂载
services:
  db:
    image: mysql:5.7
    volumes: 
      - mysql_db_data:/var/lib/mysql
      
# 需要单独声明具名挂载的卷 其实是为了多个容器公用数据      
volumes:
  mysql_db_data: {}
  wordpress_data: {}
```

## springboot实例

以公司的spring项目为例，首先确保`java -jar`没有问题，但是我maven打包后执行`java -jar`自动退出，打开日志详细发现没有指定yml配置文件的问题，通过`java -jar -Dspring.config.location=./application.yml,./application-local.yml phm-admin.jar` 且拷贝两个配置文件到本地才成功。

编写这个springboot的镜像吧，基于java8

```shell
FROM java:8
COPY target/phm-admin.jar /app.jar
COPY src/main/resources/*.yml ./
CMD ["--server.port=8080"]
EXPOSE 8080
ENTRYPOINT ["java","-jar","-Dspring.config.location=./application.yml,./application-local.yml","app.jar"]
```

成功后编写对应的docker-compose文件

```
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      - redis

  redis:
    image: "redis:alpine"
```
> 注意要修改配置文件，把依赖的redis的地址改为`redis`，这里用的是redis的主机名


![upload successful](/images/pasted-158.png)


## DockerSwarm

集群的部署方式