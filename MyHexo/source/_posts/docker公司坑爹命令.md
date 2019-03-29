title: docker公司坑爹命令
author: tr
tags:
  - docker
categories:
  - docker
date: 2019-03-29 14:43:00
---
### 记录在公司开发使用的docker命令

1. docker exec -it mysql-server mysql -uroot -p 进入容器mysql

2. docker inspect containerID |grep Mounts -A 20 查看容器挂载目录