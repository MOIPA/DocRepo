title: Kafka入门
author: tr
tags:
  - Kafka
categories:
  - Kafka
date: 2021-02-25 05:32:00
---
# Kafka 消息队列入门

<!--more-->

> Message Queue : MQ 市场上有很多实现：RabbitMQ，Kafka，ActiveMQ等，本质和java里面的普通队列区别不大

> 一般消息不是永久存储的，消息在队列内是可以设置存储时间，是个平台间的中间件

## 应用场景

> 用户注册时，需要将注册信息保存到数据库，还需要发送邮件和短信，但是发送邮件和短信需要其他外部服务器配合，需要额外时间，这时就可以用消息队列异步处理。实质还是系统解耦用。

> 流量削峰，某一时刻流量太大的处理很容易压垮数据库，这时候可以将请求放入队列

> 日志处理，分析用户行为，可以将用户点击的商品等信息存入队列，然后大数据实时处理系统（Flink程序）会将用户行为计算出结果（推送类似视频或者商品）。

## 消息队列的交互模型

> http请求响应模型很简单，前端发http请求，后端响应

> 数据库请求模型，后端执行sql语句，通过MySql通信协议，传递给MySql，数据库执行将结果返回

> 消息队列的模型则是由生产者将消息生产到队列，消费者从队列中取出

### 交互模式：点对点模式

> 生产者生产完的消息发送到队列，消费者在消费后消息就被队列删除，所以消费者不可能会接收已经消费过的数据。

> 特点：
> + 每个消息只有一个接收者
> + 发送者和接收者无关
> + 接收者收到消息后需要应答队列，以便队列删除

### 交互模式：发布订阅模式

> 生产者生产数据到队列，监听队列的所有消费者都可以消费消息

> 特点：
> + 每个消息多个订阅者
> + 发布者和订阅者有时间上的依赖，针对某个主题，必须有订阅者订阅这个主题，才能发布消息
> + 订阅者需要提前订阅且保持在线

## Kafka特点

> * 开源，用scala写的
> * 可以发布订阅
> * 带容错可以持久化消息
> * 可以流处理（类似sparks stream）

## Kafka的应用场景

> 1. 用在系统间的数据管道
> 2. 构建实时流应用，用来转换或者响应数据流
> 很多系统产生的数据都可以放入`Kafka`，很多系统可以从`Kafka`集群中消费
 
## Kafka环境搭建(普通搭建)

> 1. 去官网下载java包，解压配置即可
> 2. 安装好自己的linux服务器（不推荐在windows下操作）

![upload successful](/images/pasted-130.png)

> 3. 将包导入到服务器解压缩，修改配置文件：server.properties
```shell
# 指定brokerID （broker就是kafka的服务进程，第一个节点是0，第二个是1...）
broker.id=0 
# 指定kafka的数据位置 一般被存放在了临时目录，随便给个目录
```
![upload successful](/images/pasted-132.png)

> 4. 将安装好的kafka复制到另一台服务器并且修改里面的brokder.id为1

> 5. 配置两台主机的KAFKA_HOME环境变量
```shell
vim /etc/profile
# 写入以下内容
export KAFKA_HOME=/opt/kafka_2.12-3.1.0
export PATH=:$PATH:${KAFKA_HOME}
# 执行生效命令
source /etc/profile
```
> 6. 启动服务 先启动zookeeper
```shell
# 启动zookeeper
nohup bin/zookeeper server start.sh config/zookeeper.properties & 
# 启动kafka
cd /opt/kafka_2.12-3.1.0
nohup bin/kafka-server-start.sh config/server.properties &
# 测试是否启动成功
bin/kafka-topics.sh --bootstrap-server node1.tr.cn:9092 --list 
```