title: Spark 学习 01 入门
author: tr
tags:
  - Spark
categories:
  - Spark
date: 2020-11-16 09:56:00
---
### 1.0 Spark 学习

<!--more-->

#### 1.1 spark 是什么

快速，多用途，集群计算系统

##### hdfs

分布式存储工具，hadoop存在MR（map-reduce)即，每个计算节点计算完毕后hdfs存储在本地，效率地下

spark，中间结果存储在内存


#### 1.2 spark 优势

1. 速度快，是MR100倍

2. 易用，更加灵活的api

3. 通用，提供大部分计算工具（结构化，非结构化，图形计算）

4. 兼容性好，可以访问大部分中间队列和数据库


#### 1.3 Spark组件

1. spark-core:是整个spark的基础（RDDs）

2. spark-sql: 处理结构化数据，Data set,DateFrame 上执行sql

3. spark-streaming: 流分析，和批量分析

4. MLib: 机器学习

5. GraphX:图计算工具

#### 1.4 hadoop和spark

1. hadoop：基础平台（存储，调度），spark只是替换mr的计算工具
2. hadoop擅长处理大数据的批处理，spark也支持大数据，快，适合迭代，交互，流计算
3. hadoop延迟大，spark 小

### 2.0 Spark 集群

#### 2.1 spark集群结构

1. spark支持集群管理工具（standalone,yarn,mersos,kubernetes）

2. spark客户端（Driver：该进程调用spark的main，并且启动SparkContext）调用集群管理工具（Cluster manager),cm负责将任务分发，每个子主机（Worker）有个守护进程，负责和cm沟通，每个Worker执行分发下来的任务，在其内的JVM虚拟机中执行（Executor）

3. 流程：启动Driver，创建sparkContext（切割任务），Client提交程序给Driver，Driver向CM申请集群资源，CM在对应的Worker启动Executor执行任务

NOTE: 

1. spark不是每次都需要启动集群，可以在单机模式（standalone）中使用测试代码
2. spark可以运行在不同集群中，支持yarn,mersos,kubernetes和自己的standalone，在每个集群中启动方式不同
3. spark在每个集群的启动模型自行了解

#### 2.2 spark集群搭建

##### 2.2.0 配置hadoop

0. 配置hadoop的java环境 `/opt/hadoop/etc/hadoop/hadoop-env.sh`：

```shell
export JAVA_HOME=/usr/java/jdk1.8
```

1. `vim /opt/hadoop/etc/hadoop/core-site.html` 
```shell
<configuration>
  <property>
        <name>fs.default.name</name>
        <value>hdfs://node01:8020</value>
  </property>
  <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
  </property>
  <property>
        <name>io.file.buffer.size</name>
        <value>4096</value>
  </property>
  <property>
        <name>fs.trash.interval</name>
        <value>10080</value>
  </property>
  <!--property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
  </property-->
</configuration>
```

2. vim /opt/hadoop/etc/hadoop/hdfs-site.html
```shell
<configuration>
   <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node01:50090</value>
    </property>
   <property>
        <name>dfs.namenode.http-address</name>
        <value>node01:50070</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///opt/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///opt/hadoop/tmp/dfs/data</value>
    </property>
   <property>
        <name>dfs.namenode.edits.dir</name>
        <value>file:///opt/hadoop/tmp/edits</value>
    </property>
   <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
   <property>
        <name>dfs.permissions</name>
        <value>false</value>
    </property>
   <property>
        <name>dfs.blocksize</name>
        <value>134217728</value>
    </property>
</configuration>
```

3. `vim /opt/hadoop/etc/hadoop/mapred-site.xml`

```shell
<configuration>
        <property>
                <name>mapreduce.job.ubertask.enable</name>
                <value>true</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>node01:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>node01:19888</value>
        </property>
</configuration>
```

4. `vim /opt/hadoop/etc/hadoop/mapred-env.sh`

```shell
export HADOOP_JOB_HISTORYSERVER_HEAPSIZE=1000

export HADOOP_MAPRED_ROOT_LOGGER=INFO,RFA

#export HADOOP_JOB_HISTORYSERVER_OPTS=
#export HADOOP_MAPRED_LOG_DIR="" # Where log files are stored.  $HADOOP_MAPRED_HOME/logs by default.
#export HADOOP_JHS_LOGGER=INFO,RFA # Hadoop JobSummary logger.
#export HADOOP_MAPRED_PID_DIR= # The pid files are stored. /tmp by default.
#export HADOOP_MAPRED_IDENT_STRING= #A string representing this instance of hadoop. $USER by default
#export HADOOP_MAPRED_NICENESS= #The scheduling priority for daemons. Defaults to 0.
export JAVA_HOME=/usr/java/jdk1.8
```
5. `vim /opt/hadoop/etc/hadoop/slaves`: `node01`

6. `mkdir -p /opt/hadoop/tmp` `mkdir -p /opt/hadoop/edits` `mkdir -p /opt/hadoop/name` `mkdir -p /opt/hadoop/data`

7. 格式化和启动dfs

`/opt/hadoop/bin/hdfs namenode -format` `/opt/hadoop/sbin/start-dfs.sh ` `/opt/hadoop/sbin/start-yarn.sh `


8. 配置环境变量

```
export  HADOOP_HOME=/opt/hadoop
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

9. web查看hadoop存储空间

`w3m http://node01:50070`

NOTE: 如果jps查看缺了什么namenode或者datanode  `cd /opt/hadoop/tmp` `rm -r dfs`
(不使用停止服务 会导致这样)

10. 常用命令

`hdfs dfs -ls /`

`hdfs dfs -mkdir /spark_log`

`hdfs dfs -put xxx.txt /data`

##### 2.2.1 安装配置spark

1. 下载spark 2.2，hadoop 2.75和安装

2. 配置本地host：`127.0.0.1 localhost node01 localhost4 localhost4.localdomain4`

3. 配置conf/spark-env.sh

```shell
# 指定java
export JAVA_HOME=/usr/java/jdk1.8

# 指定spark master 地址
export SPARK_MASTER_HOST=node01
export SPARK_MASTER_PORT=7077

# 指定查找history server日志目录和web端口
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=4001 -Dspark.history.retainedApplications=3 -Dspark.history.fs.logDirectory=hdfs://node01:8020/spark_log"

export HADOOP_HOME=/opt/hadoop
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
```

4. 配置conf/slaves，master启动时可以启动所有从节点，告诉master worker在哪
```shell
node01
192.168.....
```

5. 配置history server:conf/spark-defaults.conf，运行结束后无法查看中间过程，配置这个即可看到

启用输出日志，指定输入地址，是否压缩
```shell
 spark.eventLog.enabled           true
 spark.eventLog.dir               hdfs://node01:8020/spark_log
 spark.eventLog.compress          true
```

6. 创建history-server的日志地址 hdfs目录下

`hdfs dfs mkdir -p /spark_log`

##### 2.2.2 分发

1. 同步配置到其他机器
```shell
cd /opt/
scp -r spark root@node02：$PWD
scp -r spark root@node03：$PWD
```

2. 启动服务
```
cd /opt/spark
sbin/start-all.sh
sbin/start-history-server.sh
```

#### 2.3 高可用

master有可能会挂，所有需要整一个备用master，有两种方式实现，一个是利用本地，一个是利用ZooKeeper，大部分是后者

配置高可用

1. 进入spark-env.sh
```shell
#注释原来的Spark Master地址
# export SPARK_MASTER_HOST=node01
# 指定spark运行时参数
export SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=node01:2181,node02:2181,node03:2181(#这里填写的是zookeeper的三台主机) -Dspark.deploy.zookeeper.dir=/spark"
```

2. 分发配置 `scp -r spark root@node02:$PWD`

3. 启动

jps查看启动进程，webUI: localhost:8080


#### 2.4 入门案例

1. 提交任务命令

程序名，master节点，执行内存，执行内数量，包名

`bin/spark-submit 
--class org.apache.spark.examples.SparkPi 
--master spark://node01:7077
--executor-memory 1G
--total-executor-cores 2
/opt/spark/examples/jars/spark-examples_2.11-2.2.0.jar                    
>> out.log
`


##### 2.4.1 执行方式

1. 观察数据集
2. 测试数据集
3. 固话代码
4. 提交集群

###### 2.4.1.1 spark-shell 读取本地文件和读取hdfs 统计单词数量

启动spark-shell：

spark-shell --master local[N] 使用N条worker线程本地运行

spark-shell --master spark://host:port standAlone中运行，指定master地址，默认端口7077

spark-shell --master mesos://host:port Apache Mesos中运行

spark-shell --master yarn  yarn中运行


1. 建立文件 vim /opt/data/WordCount.txt 

2. 进入shell spark-shell --master local[6]

```shell
// 读取文件 /opt/data/WordCount.txt
val rdd1 = sc.textFile("file:///opt/data/WordCount.txt")
// flatMap 平铺分割 按照空格分割
val rdd2 = rdd1.flatMap(_.split(" "))
// 每个元素组为元祖
val rdd3 = rdd2.map((_,1))
// 按照key聚合
val rdd4 = rdd3.reduceByKey(_+_)
// 收集
rdd4.collect
```

如果文件过大，本机存不下，需要将文件上传到hdfs，可以直接不加"hdfs://node01:8020"，需要在conf/spark-env.sh中配置

```
export HADOOP_HOME=/opt/hadoop
export HADOOP_CONF_DIR=/opt/hadoop/etc/hadoop
```

1. cd /opt/data

2. `hdfs dfs -mkdir /data` , `hdfs dfs -put WordCount.txt /data`

```shell
// 读取文件 /opt/data/WordCount.txt
val rdd1 = sc.textFile("hdfs://node01:8020/data/wordcount.txt")

// flatMap 平铺分割 按照空格分割
val rdd2 = rdd1.flatMap(_.split(" "))
// 每个元素组为元祖
val rdd3 = rdd2.map((_,1))
// 按照key聚合
val rdd4 = rdd3.reduceByKey(_+_)
// 收集
rdd4.collect
```
3. 只有在调用collect的时候才会计算

###### 2.4.1.1 独立应用

1. 本地运行

创建idea,maven 工程
pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>SparkLearn</groupId>
    <artifactId>com.tr.spark</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <scala.version>2.11.8</scala.version>
        <spark.version>2.2.0</spark.version>
        <slf4j.version>1.7.16</slf4j.version>
        <log4j.version>1.2.17</log4j.version>
    </properties>

    <!--    <repositories>-->
    <!--        <repository>-->
    <!--            <id>scala-tools.org</id>-->
    <!--            <name>Scala-Tools Maven2 Repository</name>-->
    <!--            <url>http://scala-tools.org/repo-releases</url>-->
    <!--        </repository>-->
    <!--    </repositories>-->

    <!--    <pluginRepositories>-->
    <!--        <pluginRepository>-->
    <!--            <id>scala-tools.org</id>-->
    <!--            <name>Scala-Tools Maven2 Repository</name>-->
    <!--            <url>http://scala-tools.org/repo-releases</url>-->
    <!--        </pluginRepository>-->
    <!--    </pluginRepositories>-->

    <dependencies>
        <dependency>
            <groupId>org.scala-lang</groupId>
            <artifactId>scala-library</artifactId>
            <version>${scala.version}</version>
        </dependency>
        <!--        <dependency>-->
        <!--            <groupId>junit</groupId>-->
        <!--            <artifactId>junit</artifactId>-->
        <!--            <version>4.4</version>-->
        <!--            <scope>test</scope>-->
        <!--        </dependency>-->
        <!--        <dependency>-->
        <!--            <groupId>org.specs</groupId>-->
        <!--            <artifactId>specs</artifactId>-->
        <!--            <version>1.2.5</version>-->
        <!--            <scope>test</scope>-->
        <!--        </dependency>-->

        <!-- spark -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>${spark.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>2.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>

        <!--        <dependency>-->
        <!--            <groupId>org.apache.spark</groupId>-->
        <!--            <artifactId>spark-streaming_${spark.artifactID.suffix}</artifactId>-->
        <!--            <version>${spark.version}</version>-->
        <!--        </dependency>-->
        <!--        <dependency>-->
        <!--            <groupId>org.apache.spark</groupId>-->
        <!--            <artifactId>spark-sql_${spark.artifactID.suffix}</artifactId>-->
        <!--            <version>${spark.version}</version>-->
        <!--        </dependency>-->
        <!--        <dependency>-->
        <!--            <groupId>org.apache.spark</groupId>-->
        <!--            <artifactId>spark-hive_${spark.artifactID.suffix}</artifactId>-->
        <!--            <version>${spark.version}</version>-->
        <!--        </dependency>-->
        <!--        <dependency>-->
        <!--            <groupId>org.apache.spark</groupId>-->
        <!--            <artifactId>spark-mllib_${spark.artifactID.suffix}</artifactId>-->
        <!--            <version>${spark.version}</version>-->
        <!--        </dependency>-->



    </dependencies>

    <build>
        <sourceDirectory>src/main/scala</sourceDirectory>
        <testSourceDirectory>src/test/scala</testSourceDirectory>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>scala-maven-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>testCompile</goal>
                        </goals>
                        <configuration>
                            <args>
                                <arg>-dependencyfile</arg>
                                <arg>${project.build.directory}/.scala_dependencies</arg>
                            </args>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>3.2.2</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals><goal>shade</goal></goals>
                        <configuration>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <mainClass></mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

WordCount.scala

```scala
package com.tr.spark

import org.apache.spark.{SparkConf, SparkContext}

/**
 * @author tr
 * @Date 11/16/20
 */
object WordCount extends App{

  // 1. 创建sparkcontext
  val conf = new SparkConf().setMaster("local[6]").setAppName("word_count")
  val sc = new SparkContext(conf)

  // 加载读取文件
  val rdd1 = sc.textFile("data/wordcount.txt")
  val rdd2 = rdd1.flatMap(_.split(" "))
  val rdd3 = rdd2.map((_,1))
  val rdd4 = rdd3.reduceByKey(_+_)
  rdd4.collect.foreach(println)

}
```
2. 提交运行

 + 修改本地文件读取路径为hdfs
 + 打成jar包 mvn package
 + 两个包：一个包大的包含了所有的依赖，一个小开头为orignal只包含了代码，由于云上有spark环境，只需要小包即可
 + `root@tr:/opt/spark# bin/spark-submit --class com.tr.spark.WordCount --master spark://node01:7077 /opt/MyExercise/original-tr.jar 
`