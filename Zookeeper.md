title: Zookeeper 01
author: tr
tags:
  - Zookeeper
categories:
  - Zookeeper
date: 2021-03-16 00:47:00
---
# Zookeeper

> 应用场景

1. 分布式协调组件

	服务组件的数据一致性问题的协调，zk可以通知其他服务修改数据状态
	
2. 分布式锁

	zk可以做到强一致性

3. 无状态化的实现

	登陆组件（系统）冗余部署后，登录信息放在哪个组件都不行，因为下次自动均衡找到的不一定是上个组件，可以将登录状态放入zk
	
## 搭建zk服务器

> 安装配置

1. 官网下载zk工具

2. 打开进入 bin目录，可以看到zkServer.sh脚本，但是启动是需要zoo.cfg文件的，可以修改conf目录下的zoo_sample.cfg

3. 有了配置文件后 `./bin/zkServer.sh start ../conf/zoo.cfg `启动即可
> zoo.cfg 文件说明

```shell
# zk的时间单位（毫秒）
tickTime=2000
# 允许follower初始化连接到leader最大时长，这里是20s
initLimit=10
# 允许follower与leader数据同步最大时长
syncLimit=5
# zk数据存储目录以及日志保存目录（如果没有指明dataLogDir
dataDir=/bitnami/zookeeper/data
# 对客户端提供的端口号
clientPort=2181
# 单个客户端和zk的最大并发链接数
maxClientCnxns=60

# 自动清楚任务的时间间隔，单位为小时，0表示不自动清楚
autopurge.purgeInterval=0

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true
preAllocSize=65536
snapCount=100000
maxCnxns=0
reconfigEnabled=false
quorumListenOnAllIPs=false
4lw.commands.whitelist=srvr, mntr
maxSessionTimeout=40000
admin.serverPort=8080
admin.enableServer=true
```

## zk的基本操作

```shell
zkServer.sh start ../conf/zoo.cfg

zkServer.sh stop ../conf/zoo.cfg

zkServer.sh status 

# 客户端执行命令脚本，相当于命令行了（zk服务器启动情况下可以通过此脚本执行命令）
zkCli.sh  

# 以下为进入命令行后的命令
ls / #查看zk的数据
help # 查看帮助
create /test1 #创建znode节点
create /test2 abc #将数据放入节点
create -s /test3 #持久序号节点
create -e /test4 #临时节点
create -c /container # 容器节点
set /test4 aa #设置内容
delete /test1 # 删除
get /test2
get -s /test2 # 查看znode的stat信息
ls -R /test1 #递归查询	
deleteall /test1 #删除节点和其子节点
delete /test3 # 普通删除
delete -v 1 /test4 #删除数据版本为1的节点（每次set版本都增加了1，这里用了乐观锁，可以删不成功继续+1删除）

```

![upload successful](/images/pasted-133.png)


## zk内部数据模型

非常类似linux的内部数据结构，每次create都是创建一个node节点，可以给这个node节点内存入数据

> znode内部结构包含四个部分:
	
	1. data：数据
	2. acl：权限
		+ c：create的权限
		+ w：写
		+ r：读
		+ d：删除
		+ a：管理
	3. stat：描述当前znode的元数据
	4. child：当前节点的子节点

> zk中节点znode的类型

	+ 持久节点：会话结束后任然存在
	+ 持久序号节点：每次创建的节点自增序号（并发严重情况下，给每个事务分配顺序）
	+ 临时节点：会话结束后即可删除，通过这个实现服务注册和发现（客户端和服务器建立的链接就是一个会话，建立链接时，zk服务器会给zk客户端发送一个session id，每次客户端通信时，zk服务器就会自动续约session id，如果超时自动删除session id这个id可以通过get -s 查看到）
	+ Container节点：容器节点，如果容器内没有任何子节点，该节点会被定期删除
	+ TTL节点：自定义节点到期时间

> zk的数据持久化

 + 事务日志：每个执行的命令以日志的形式保存在dataLogDir中
 
 + 数据快照：每隔一定时间把内存的数据备份一次，存在dataDir中
 
> 之后恢复可以先回复快照文件数据到内存中，再通过日志文件的数据做增量回复，这样的恢复速度很快

## 权限设置

+ 注册当前会话的账号密码

	`addauth digest tr:0800`

+ 创建节点并且设置权限

	`create /test abcd auth:tr:0800:cdwra`
    
   其他会话使用必须执行addauth才能操作
   

## Curator客户端

> 介绍
> + 这是网飞开发的专为zk的客户端框架，是对zk支持最好的工具，支持Leader选举，分布式锁等，减少开发者使用zk的底层细节。
引入依赖

```java
<dependencies>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>2.12.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>2.12.0</version>
        </dependency>
    </dependencies>
```

application.yml配置文件

```properties
curator.retryCount=5
curator.elapsedTimesMs=5000
curator.connectString=127.0.0.1:2181
curator.sessionTimeoutMs=60000
curator.connectionTimeoutMs=5001
```

配置文件WarpperZK.class
```java
package com.zq.client.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Data
@Component
@ConfigurationProperties(prefix = "curator")
public class WrapperZK {
    private int retryCount;
    private int elapsedTimesMs;
    private String connectString;
    private int sessionTimeoutMs;
    private int connectionTimeoutMs;
}

```

CuratorConfig.class 配置文件

```java

package com.zq.client.config;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.RetryNTimes;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CuratorConfig {
    @Autowired
    WrapperZK wrapperZk;

    @Bean(initMethod = "start")
    public CuratorFramework curatorFramework() {
        return CuratorFrameworkFactory.newClient(
                wrapperZk.getConnectString(), wrapperZk.getSessionTimeoutMs(), wrapperZk.getConnectionTimeoutMs(),
                new RetryNTimes(wrapperZk.getRetryCount(), wrapperZk.getElapsedTimesMs()));
    }
}

```

测试类文件

```java
package com.zq.client;

import lombok.extern.slf4j.Slf4j;
import org.apache.curator.framework.CuratorFramework;
import org.apache.zookeeper.CreateMode;
import org.junit.Test;
import org.junit.Before;
import org.junit.After;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.nio.charset.StandardCharsets;

/**
 * com.zq.client.BootZkClientApplication Tester.
 *
 * @author <Authors name>
 * @version 1.0
 * @since <pre>3�� 16, 2022</pre>
 */
@Slf4j
@SpringBootTest
@RunWith(SpringRunner.class)
public class BootZkClientApplicationTest {

    @Autowired
    CuratorFramework curatorFramework;

    @Before
    public void before() throws Exception {
    }

    @After
    public void after() throws Exception {
    }

    /**
     * Method: main(String[] args)
     */
    @Test
    public void createNode() throws Exception {
        // 添加持久节点
        String path = curatorFramework.create().forPath("/curator-node");
        // 添加临时序号节点
//        String path1 = curatorFramework.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath("/curator-node", "some data".getBytes(StandardCharsets.UTF_8));
        System.out.println(String.format("curator create node:%s successfully.", path));
//        System.in.read();
    }

    @Test
    public void testGetData() throws Exception {
        byte[] bytes = curatorFramework.getData().forPath("/curator-node");
        System.out.println(new String(bytes));
    }

    @Test
    public void testSetData() throws Exception {
        curatorFramework.setData().forPath("/curator-node", "changed".getBytes(StandardCharsets.UTF_8));
    }

    @Test
    public void testCreateWithParent() throws Exception {
        curatorFramework.create().creatingParentsIfNeeded().forPath("/parent-node/sub", "tzq".getBytes(StandardCharsets.UTF_8));
        byte[] bytes = curatorFramework.getData().forPath("/parent-node/sub");
        System.out.println(new String(bytes));
    }
} 

```

## ZK如何实现分布式锁

> 如果两个java服务都是向数据库中写入车票数据，负载均衡每次只分配到一个java服务，当一个服务写时，另一个服务不能写，如何做到两个或多个服务的锁，通过ZK存储锁

> zk中锁的种类:
> + 读锁
> + 写锁

### ZK上读锁

1. 创建临时序号节点，节点数据为read，表示读锁
2. 获取zk中序号比自己小的所有节点，判断是不是读锁，若都是读锁，那么上锁成功，反之失败，为最小节点设置监听，阻塞等待，当小于最小节点时同时当前节点再判断

### ZK上写锁

1. 创建临时序号节点，节点数据为write，表示写锁
2. 获取所有节点并判断自己是否时最小节点，是则上锁成功，反之监听最小节点，最小节点没了才能再次检测

### 羊群效应

如果100个并发来上写锁，那么99个并发会监听写锁，当第一个节点完成写锁消失后，99个又触发监听事件，对zk压力大，调整为链式监听即可，即不再监听第一个节点，而是监听上一个节点。

这样，100个并发过来，第一个得到写锁，第二个监听第一个，第三个监听第二个，依次。如果第一个结束，第二个监听事件触发可以上写锁，但是第三位不被触发监听因为第二位节点没有删除。

## zk的watch机制

> watch机制类似触发器，当znode改变，即调用了create,delete,setData等方法的时候，会触发对应znode上注册的事件，请求watch的客户端会接收到异步通知

### cli中创建并且监听节点

```shell
craete /test9
get -w /test9 # 一次性监听节点内容
ls -w /test9 # 一次性监听节点目录（一层目录）
ls -R -w /test9 # 监听所有子目录变化
# 这时候其他会话修改数据，这里会通知
```

![upload successful](/images/pasted-134.png) 

### curator 客户端使用watch监听节点

```shell
   @Test
    public void testAddListener() throws Exception {
        NodeCache nodeCache = new NodeCache(curatorFramework, "/curator-node");
        nodeCache.getListenable().addListener(() -> {
            log.info("{} path nodeChanged:", "/curator-node");
            printNodeData();
        });
        nodeCache.start();
        System.in.read();
    }

    public void printNodeData() throws Exception {
        byte[] bytes = curatorFramework.getData().forPath("/curator-node");
        log.info("data:{}", new String(bytes));
    }
```

### Curator 上写锁和读锁

这段junit代码可以先运行读锁再运行写锁看效果

```java

    @Test
    public void testGetReadLock() throws Exception {
        // 读写锁
        InterProcessReadWriteLock interProcessReadWriteLock = new InterProcessReadWriteLock(curatorFramework, "/lock1");
        // 获取读锁对象
        InterProcessMutex interProcessMutex = interProcessReadWriteLock.readLock();
        System.out.println("等待获取读锁对象");
        // 获取锁
        interProcessMutex.acquire();
        for (int i = 0; i < 10; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        // 释放
        interProcessMutex.release();
        System.out.println("等待释放锁");

    }

    @Test
    public void testGetWriteLock() throws Exception {
        // 读写锁
        InterProcessReadWriteLock interProcessReadWriteLock = new InterProcessReadWriteLock(curatorFramework, "/lock1");
        // 获取写锁对象
        InterProcessMutex interProcessMutex = interProcessReadWriteLock.writeLock();
        System.out.println("等待获取写锁对象");
        // 获取锁
        interProcessMutex.acquire();
        for (int i = 0; i < 10; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        // 释放
        interProcessMutex.release();
        System.out.println("等待释放锁");

    }
    
```

## ZK集群

> 集群的角色
> + Leader : 处理集群的所有事务请求，只有一个，负责数据读写
> + Follower : 从，只负责数据读，还能参与Leader选举（Leader挂了的情况）
> + Observer : 观察者，只负责读，不参与选举

### 集群搭建

我这里用了docker搭建，没有用dockerCompose，而是bash脚本执行，注意！

> 在windows下使用gitbash执行shell脚本，路径是要这么写的！

```shell
#!/bin/bash

# 创建配置文件
for port in $(seq 1 5);
do
mkdir -p /D/DockerV/zkCluster/zkData-${port}/data
touch /D/DockerV/zkCluster/zkData-${port}/data/myid
cat << EOF > /D/DockerV/zkCluster/zkData-${port}/data/myid
${port}
EOF

touch /D/DockerV/zkCluster/zkData-${port}/zoo.cfg
cat << EOF > /D/DockerV/zkCluster/zkData-${port}/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/bitnami/zookeeper/data
clientPort=2181
server.1=172.38.0.11:2001:3001
server.2=172.38.0.12:2001:3001
server.3=172.38.0.13:2001:3001
server.4=172.38.0.14:2001:3001
server.5=172.38.0.15:2001:3001:observer
EOF
done

# 批量启动容器
for port in $(seq 1 5);
do
docker rm -f  zk-${port}
docker run -p 318${port}:2181 \
--name zk-${port} \
-v D:/DockerV/zkCluster/zkData-${port}/data/://bitnami/zookeeper/data \
-v D:/DockerV/zkCluster/zkData-${port}/zoo.cfg://opt/bitnami/zookeeper/conf/zoo.cfg -d \
-e  ALLOW_ANONYMOUS_LOGIN=yes \
--net zknet \
--ip 172.38.0.1${port} \
bitnami/zookeeper 
done

```

> 进入集群 `docker run -it --rm --network zknet bitnami/zookeeper zkCli.sh -server 192.168.10.28:3181,192.168.10.28:3182,192.168.10.28:3183,192.168.10.28:3184,192.168.10.28:3185`


![upload successful](/images/pasted-135.png)


![upload successful](/images/pasted-136.png)

> 集群实质配置
> + 其实本质是每台电脑配置好自己的/bitnami/zookeeper/data目录下的myid（里面是id号）
> + 配置/opt/bitnami/zookeeper/conf/zoo.cfg这个cfg文件，最重要的是里面填好通信的其他机器地址和端口

## ZAB协议

> 解决zk的崩溃恢复和主从数据同步问题

> ZAB协议定义的四种节点状态
> + Looking:选举状态
> + Following:Follower节点所处的状态
> + Leading:Leader节点的状态
> + Observing：。。。

### 集群上线时Leader选举过程：

启动第一台时会looking状态寻找，直到第二台上线，开始选举，选票（myid，zXid事务id，每次事务都自增）第一轮投票，都生成自己的一张选票，然后把选票信息给对方，每台会比较选票（选择zXid最大的），如果事务id相同，取myid最大的，然后投入投票箱。第二轮互换自己投票箱的票，比较后放入投票箱，选出leader

### 崩溃恢复时的leader选举

Leader建立完毕后，Leader和Follower是有通信端口的，Leader和Follower存在socket链接，Follower会不间断的读socket数据，Leader会不间断的发，若Leader挂了，socket断开，Follower读不到数据后进入looking状态，其他节点同理。此时Leader出来之前不提供服务！

### 主从数据同步

客户端向集群写入数据，可能链接的是从，也可能是主，若客户端链接从节点，从节点会将数据发送给主节点。

主节点将数据写入自己的数据文件，并且返回ACK，然后同步的将数据发给Follower（广播），每个从节点也写入自己的数据文件，完成后返回ack给主节点，Leader收到半数以上的Ack信号后，这时Leader给所有从节点提交commit。从节点收到后将数据写入内存。这样大部分服务器数据都同步了。

半数以上是为了提高集群写数据的性能。但是有可能会有某些服务器数据没有写入到内存。

### NIO和BIO应用

> NIO （多路复用）
> + 用于被客户端链接的2181端口
>    假设同时有4个客户端链接，有读有写，为了性能会使用NIO把所有请求放入一个队列，zk内部处理这些请求
> +客户端开启watch时也用
> 若有客户端同时监听很多znode节点，当节点变化，客户端会被通知到，NIO模式实现非阻塞状态

> BIO 传统阻塞模型
> + 用于Leader选举时的选票交换

## CAP定理

一个系统最多同时满足：一致性（每个节点数据一致），可用性（服务一直可用），分区容错性（遇到节点故障时仍然能够提供服务） 中的两项。

所以现在大部分都采用AP或者AC

## BASE理论

> + 基本可用  （双十一切掉评论，退款，注册服务，保留核心功能）
> + 软状态		（没有其他功能的中间状态）
> + 最终一致性 （最终还是能退款的）

回到之前的弊端（zk可能存在某些节点数据未同步），对于未同步数据的服务器，它的事务节点肯定落后于有数据的节点（处理完一次数据，事务id自增），因此zk最求的是顺序一致性，等网络恢复后，落后的节点总会同步到最新的数据，把事务id同步到最新。