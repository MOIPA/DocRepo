title: docker的build和dockerfile
author: tr
tags:
  - docker
categories:
  - docker
date: 2019-03-28 10:55:00
---
### docker 的Dockerfile

```sh
From centos
From openjdk:8-jdk
VOLUME /tmp
ADD *.jar app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""
#ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
CMD exec java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar

```

### docker build -t name .

1. 记住名字后面有个点 标识DockerFile的位置

2. 启动创建的容器用 docker start 