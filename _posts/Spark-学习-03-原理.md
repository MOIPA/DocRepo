title: Spark 学习 03 原理
author: tr
tags:
  - Spark
categories:
  - Spark
date: 2020-11-17 10:25:00
---
### Spark 学习 03 spark原理

#### 1.0 总体介绍

1. 为什么说一组电脑是spark集群？ 因为这一组电脑都运行了Spark，node01 运行了masterdaemon 所以node01是master，其他电脑运行了workderDaemon，所以其他是worker，workDaemon去master上认领任务，取得任务后还要创建和关闭executor

2. executor 怎么运行？ 首先运行一个executor Backend来管理executor，一对一关系

3. driver干嘛的？  整个spark application的驱动节点，action操作实质是将结果发给driver

```scala
 val textRdd: RDD[String] = sc.textFile("data/wordcount.txt")
    val splitRdd = textRdd.flatMap(_.split(" "))
    val tupleRdd = splitRdd.map((_, 1))
    val reduceRdd = tupleRdd.reduceByKey(_ + _)
    val strRdd = reduceRdd.map(item => s"${item._1},${item._2}")
    strRdd.collect().foreach(println)
```

#### 2.0 逻辑执行图

1. 代表了数据如何计算和流转，以wordCount为例，在结果调用print(strRdd.toCollectString()),可以看到依赖关系

2. 并非执行单位，最后还是要划分到实际执行单位(机器怎么执行)

##### 2.1 边界

1. rdd从第一个rdd的创建开始，到逻辑图action执行之前结束.就是一组rdd和其依赖关系

2. RDD 5大属性：分区，依赖，计算函数，最佳位置，分区函数

##### 2.2 rdd如何生成

1. sc.textFile在源码中回去生成一个对象：HadoopRdd，这个HadoopRdd继承了父类RDD并且重写了compute方法，这个compute方法实际调用了inputFormat方法，实际就是去读取hdfs文件块，HadoopRdd的一个分区实际就对应了hadoop的一个文件块。

2. map算子: 在源码中就是new了一个MapPartitionsRdd，且传递了一个scala的map方法给他的构造函数，并且由iter调用。spark map算子中接受的函数，实际交给了里面的scala的map。 这个iter实际是一个分区的迭代器。

3. flatMap算子：和map差不多

##### 2.3 rdd之间有哪些依赖

1. rdd分区之间的关系，flatMap这些算子的分区关系是一一对应的

2. 多对一的关系：reduceByKey

为什么要对rdd划分依赖关系：想确定rdd是否能在同一流水线上执行(取决于两个是否是shuffle关系)

1. 窄依赖：没有shuffle，shuffle是必须要对被分发区的每条数据进行切分的

2. 宽依赖：有shuffle，reduceByKey:假设rddA有三个区块的数据，第一个区块的数据为：（hadoop，1），（spark，1）,假设生成到rddB，通过分区函数将每个分区的数据发送到rddB的每个分区。然后开始塞数据，假设 key为hadoop的数据给到rddB的0分区，key为spark的hadoop的数据给rddB的1分区。那么rddA的第一个区块的数据会被拆分，所以这是一个宽依赖（shuffle）

如果两个分区一对一关系，必定是窄依赖
如果多对一要看是否有数据分发，有就是宽依赖

窄依赖的类别：

1. 依赖类的关系

RDD之间的关系是由 dependency对象决定的，这个对象可以获得另一端信息

第一级别继承类：NarrowDependency,ShuffleDependency

第二级别：OneToOneDependency，RangeDependency，继承自NarrowDependency  

+ 一对一窄依赖：map算子
+ range窄依赖：只在union中使用，两个集合合起来。
+ 多对一窄依赖：和shuffle相似但是不是，coalesce求笛卡尔积为例，被发的rdd是不会对数据再切分

宽依赖只能等待前一个rdd的所有数据算好后切割分发，但是窄依赖的不同分区可以和生成的rdd的分区对应放在一个task计算。

#### 3.0 物理执行图

1. 数据如何在集群中计算

如代码所示，flatMap和map会被合并为一个计算任务在一个executor中执行完毕后，再执行，一个task表示一个flatMap和map计算，多个task组合成一个stage。
执行shuffle（reduceByKey）操作后就是另一个stage，最后将结果发给Driver。

2. RDD是被谁执行计算的？

每台电脑的executor是一个进程，使用多线程计算，和driver认领任务，运行任务线程：task。

task如何设计，如果有rddA---map--->rddB---map---rddC每个rdd都是3个分区。

如果设计每个分区和map就是一个task，那么map的结果得生成文件，给下一个分区的map这就和hadoop的mapreduce一样了

如果将rddA的分区和rddB的分区的两个map生成一个task，一共三个task，共享内存，效率高多了，但是遇到shuffle操作就有问题了。

spark采用数据流动模型设计，划分阶段：因为在遇到shuffle会出问题,所以在有shuffle的地方分段，shuffle左边的某分区的所有操作成为一个task，右边分为一个task，这样就有了两个stage。

划分stage规则：从后往前划分，知道遇到shuffle（宽依赖）断开stage，创建新的stage，继续往前走。

3. 数据流向

数据的计算发生在调用Action的RDD上，RDD一直往上请求数据，类似递归，然后不停返回数据。第一个获取数据的rdd是最左边的rdd。

#### 4.0 如何运行

1. Collect方法会去调用DAGScheduler方法==》taskScheduler方法 运行到集群中。DAGScheduler给每个job生成有向无环图，确定最佳task位置

2. 一次action生成一个job，数据从读取到生成结果就是一个job，job会被分发到集群是spark调度的颗粒，一个job有多个stage，一个stage有多个task，stage之间串行执行。

3. TaskSet：一个stage对应了一个TaskSet（多个task，数量由分区决定）

#### 5.0 spark 高级特性

##### 闭包
```scala
def closeure():Unit  = {
    val factor = 3.14
    (r:Int) = 
}

```

#### 6.0 spark shuffle原理