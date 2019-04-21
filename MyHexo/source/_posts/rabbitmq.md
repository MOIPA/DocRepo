title: rabbitmq
author: tr
tags:
  - rabbitmq
categories:
  - rabbitmq
date: 2019-04-04 10:46:00
---
#### rabbitmq

消息队列服务

<!--more-->

#### 安装

很多

#### 常用命令

1. rabbitmq-plugins enable rabbitmq_management 启动网页控制，记得打开端口
2. rabbitmqctl add_user root root123
3. rabbitmqctl change_password root root321
4.  rabbitmqctl delete_user root
5. rabbitmqctl list_users -q
6. 

	
      用户的角色分为5种类型：

      none：无任何角色。新创建的用户的角色默认为none。
      management：可以访问Web管理页面。Web管理页面在5.3章节中会有详细介绍。
      policymaker：包含management的所有权限，并且可以管理策略（policy）和参数（parameter）。详细参考5.5章节。
      monitoring：包含management的所有权限，并且可以看到所有连接（connections）、信道（channels）以及节点相关的信息。
      administartor：包含monitoring的所有权限，并且可以管理用户、虚拟主机、权限、策略、参数等等。administator代表了最高的权限。
      用户的角色可以通过rabbitmqctl set_user_tags {username} {tag …}命令设置。其中username参数表示需要设置角色的用户名称；tag参数用于设置0个、1个或者多个的角色，设置之后任何之前现有的身份都会被删除。使用示例如下：
      [root@node1 ~]# rabbitmqctl set_user_tags root monitoring
      Setting tags for user "root" to [monitoring]
      [root@node1 ~]# rabbitmqctl list_users -q
      guest        [administrator]
      root         [monitoring]
      [root@node1 ~]# rabbitmqctl set_user_tags root policymaker -q
      [root@node1 ~]# rabbitmqctl list_users -q
      guest        [administrator]
      root         [policymaker]
      [root@node1 ~]# rabbitmqctl set_user_tags root
      Setting tags for user "root" to []
      [root@node1 ~]# rabbitmqctl list_users -q
      guest        [administrator]
      root         []
      [root@node1 ~]# rabbitmqctl set_user_tags root policymaker,management
      Setting tags for user "root" to ['policymaker,management']
      [root@node1 ~]# rabbitmqctl list_users -q
      guest        [administrator]
      root         [policymaker,management]
    
7. 	  rabbitmqctl set_permissions -p VHostPath User ConfP WriteP ReadP 
    // 具有/vhost1这个virtual host中所有资源的配置、写、读权限以便管理其中的资源 
    
    rabbitmqctl set_permissions -p / yuantu '.' '.' '.*'
8. 
9. 
10. 
