title: docker命令
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

3. ps -ef | grep mysql

4. cat /var/lib/jenkins/secrets/initialAdminPassword

5. service jenkins start

6. vim /etc/sysconfig/jenkins

7. vim /etc/init.d/jenkins

8. docker cp jenkins:/var/jenkins_home/jenkinsPro.jar /var/lib/jenkins/

9. 忘记root密码：找到MySQL的my.cnf配置文件，在/etc/my.cnf (有些版本是/etc/mysql/my.cnf）在里面增加下面一段信息：

        [mysqld]
        skip-grant-tables
        进入数据库
        use mysql; update mysql.user set 	authentication_string=password('123') where user='root';

10. systemctl start mysqld.service

11.	部署问题：在伟哥更新了gateway后终于可以成功部署，但是前端访问时没有访问，首先配置了防火墙，确认打开了端口后，还是无页面信息，访问了192.168.0.99：5555的gateway端口后也没有任何显示

	最后居然使用https://192.168.0.99：5555访问后就可以了，是访问一边后生成了证书吗？
     
