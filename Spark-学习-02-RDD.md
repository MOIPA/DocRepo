title: Spark 学习 02 RDD
author: tr
tags:
  - Spark
categories:
  - Spark
date: 2020-11-17 09:49:00
---
### RDD

<!--more-->

#### 1.0 RDD概念

1. RDD 是弹性分布式数据集，有flatMap等api，也是编程模型。RDD 之间有依赖，且可以分区

2. 分区：RDD和MR可以将自身分区到每一个block，处理每一个block的数据，如同hdfs，文件都被划分为block分布式存储在集群中。RDD和MR都是并行的。

#### 2.0 CODE

1. RDD实例化，SparkCore的入口SparkContext


Driver和ClusterManager以及Worker的分布就如同C/S架构，SparkContext是Driver（前端客户端）最核心的组件。

Spark作为大入口，可以设置参数，设置jar包等

##### 2.1 RDD创建

```scala

  // 创建RDD三种方式

  // 1. 本地集合方式创建
  @Test
  def rddCreationLocal():Unit = {
    // 由于本地集合没有分区概念，提供集合还需要提供分区数量
    val rdd1:RDD[String] = sc.parallelize(Seq("tzq","tr","tr","spark"),2)
    val rdd2:RDD[Int] = sc.parallelize(Seq(1,2,3),2)
    // 另一个方法 本质和上面一样  不常见
    val rdd3 = sc.makeRDD(Seq(1,2,3),2)
  }

  // 2. 从文件创建
  @Test
  def rddCreationFiles():Unit = {
    // 1. 路径（支持本地和hdfs。不加前缀（file or hdfs）的路径取决于启动状态是否是集群，集群默认读取hdfs）
    // 2. 分区若读取hdfs，那么分区数由文件决定
    // 3. 支持aws或者阿里云读取
    sc.textFile("hdfs://node01:8020/data....",2) // 最小分区数量参数
  }

  // 3. 从Rdd创建
  @Test
  def rddCreationFromRdd():Unit = {
    val rdd1:RDD[String] = sc.parallelize(Seq("tzq","tr","tr","spark"),2)
    // 通过rdd的算子操作 会生成新的rdd
    val rdd2 = rdd1.map((_,1))
  }
```

##### 2.2 算子

map,flatMap 同java stream
ReduceByKey 接受二元元祖（k:v)

##### 2.2.1算子分类：

1. 基础数据类型的计算

2. k:v 计算（这里特指二元元组）(reduceByKey,groupByKey.....)

3. 针对数字类型的操作（max,min....)

##### 2.2.2 转换算子学习

1. map

2. flatMap

3. reduceByKey: 传入二元元组，按照key分组，传递分组的value计算

4. mapPartitions（并行）: 和map的区别，map针对单个数据（如果在其内数据库访问，效率很低），mapPartitions（不让每一条数据执行访问数据库，按照分区访问数据库，效率高）将RDD内的所有分区数据一次传输过去，map的话得每次单条传输过去给执行器
```scala
  // 1. mapPartitions
    val rdd1 = sc.parallelize(Seq(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12), 2)
    // 一个分区肯定不止一条数据
    rdd1.mapPartitions(iter => {
      // iter 是scala的数据类型
      iter.map(_*10)
    }).collect().foreach(println)
```

5. mapPartitionsWithIndex(并行):
```scala
val rdd2 = sc.parallelize(Seq(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12), 2)
	// index 是分区号
    rdd2.mapPartitionsWithIndex((index,iter)=>{
      iter.foreach(x=>println(x+" belong index:"+index))
      iter
    }).collect()
```

6. filter
```scala
// true 就留下
val rdd3 = sc.parallelize(Seq(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12), 2)
    rdd3.filter(_>6).collect().foreach(println)
```

7. sample：如果数据太大，变为小数据集，随机抽取数据，保证速度
```scala
sc.parallelize(Seq(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12), 2)
      // 参数1:是否有放回（是否能抽到同一个东西），false就是无放回，同一数不能抽取出两次   参数2：采样比例  参数3：种子，一般不指定
      .sample(false,0.3).collect().foreach(println)
```

8. mapValues: 针对二元元组，可以用map代替，但是这个更方便
```scala
    sc.parallelize(Seq(("a", 1), ("b", 2), ("c", 3), ("d", 4)),2)
      .mapValues(_*10).collect().foreach(println)
```

9. 交集 并集 差集：interaction union subtract
```scala
  val rddx = sc.parallelize(Seq(1, 2, 3, 4, 5), 2)
    val rddy = sc.parallelize(Seq(3, 4, 5, 6, 7), 2)
    rddx.intersection(rddy).collect().foreach(println)
    rddx.union(rddy).collect().foreach(println)
    rddx.subtract(rddy).collect().foreach(println)
```

10. groupByKey:每个分区重复的k:v可以出来，但是reduceByKey每个分区只能有一个key出来（可以减少io）
```scala
    // 7. 分组 groupByKey 本质是shuffle  生成key => 数组
    sc.parallelize(Seq(("a", 1), ("a", 2), ("c", 3), ("c", 4)), 2)
      .groupByKey().foreach(println)
```

11. combineByKey: groupByKey和reduceByKey的底层
```scala
// 8. combineByKey  算平均分
    sc.parallelize(Seq(("tzq", 97.0), ("tzq", 98.0), ("tr", 88.0), ("tr", 92.0)), 2)
      // 参数说明：1.将value初步转换（分区内） 2.在每个分区把上一步结果聚合 3. 在所有分区上把每个分区结果聚合 4.可选，分区函数 5.可选，是否在map端的Combine 6.序列化器
      // 思路：将每个数据变成(分数，1) 然后聚合 （总分，几）  一个分区结果就出来了
      // 然后将不同分区的均分聚合， 然后除 （均分，1）
      // 写法说明：第一个函数作用于第一条数据后，接着将结果和第二条数据作为参数传入第二个函数。 前两个函数作用于每个分区，将每个分区的结果作为参数传递给第三个函数
      .combineByKey((_,1),(c:(Double,Int),nextValue:Double)=>(c._1+nextValue,c._2+1),(c:(Double,Int),v:(Double,Int))=>(c._1+v._1,c._2+v._2))
      .map(item=>(item._1,(item._2._1/item._2._2,1))).foreach(println)
```

12. foldByKey:比起reduceByKey有一个初始值（会加到每个元组）
```scala
 // 9. foldByKey
    sc.parallelize(Seq(("a", 1), ("b", 2), ("c", 3), ("d", 4)), 2)
      .foldByKey(10)(_+_).collect().foreach(println)
```
13. aggregateByKey: 先处理 后聚合
```scala
// 10. aggregateByKey 打八折后的总价
    // 参数说明：1. 初始值 2.seqop 作用于每个分区每条数据 传递初始值和每条数据的value 3. combOp 整体聚合生成最终结果
    sc.parallelize(Seq(("a", 10.0), ("a", 20.0), ("c", 30.0), ("d", 40.0)), 2)
      .aggregateByKey(0.8)((zeroValue,item)=>item*zeroValue,(curr,agg)=>curr+agg).foreach(println)
```

14. join
```scala
  val rdd1 = sc.parallelize(Seq(("a", 1), ("a", 2), ("b", 1)), 2)
    val rdd2 = sc.parallelize(Seq(("a", 10), ("a", 2), ("b", 12)), 2)
    rdd1.join(rdd2).foreach(println)
```

15. sortBy:作用于任何数据类型，sortByKey只用于kv 且只能按照key排序，写法简单
```scala
 // 12. sortBy
    val rdd1 = sc.parallelize(Seq(2, 1, 6, 4, 5, 3, 7, 8, 9, 10, 11, 12), 2)
    val rdd2 = sc.parallelize(Seq(("a", 2), ("b", 1), ("c", 3), ("d", 4)), 2)

    // 参数：1.用哪个进行排序
    rdd1.sortBy(item=>item).collect().foreach(println)
    rdd2.sortBy(item=>item._2).collect().foreach(println)
    rdd2.sortByKey().collect().foreach(println
```

16. repartition
```scala
 // 13. repartition
    val rdd = sc.parallelize(Seq(1, 2, 3, 4, 5), 2)
    // 重新分区  分区越多线程越多 为了节省资源 可以适当减少分区数量
    println(rdd.repartition(4).partitions.size)
    // 减少合并分区
    println(rdd.coalesce(5,shuffle = true).partitions.size)
    // repatition 重分区时 默认shuffle
    // coalesce 重分区时 默认不shuffle 所以默认不增大分区
```

<b>action 操作：一个actions生成一个job</b>


17. collect 

18. reduce  不是转换操作的reduceByKey，如果有10个不同key的多条数据，结果只有10条，但是reduce后只有1条，reduce针对所有数据类型
```scala
 // 14. reduce
    println(sc.parallelize(Seq(1, 2, 3, 4, 5), 2)
      .reduce(_ + _))
    val res = sc.parallelize(Seq(("a", 1), ("a", 2), ("b", 1)), 2)
      .reduce((curr, agg) => ("全部", curr._2 + agg._2))
    println(res._2)
```

19. foreach 不同于scala本身的foreach，spark的算子会推送到集群执行，collect会将数据拉倒driver端，所以排序后不collect直接调用foreach会并行遍历各自分区的数据

20. count
```scala
val rdd = sc.parallelize(Seq(("a", 1), ("a", 2), ("b", 1)), 2)
    println(rdd.count())
    println(rdd.countByKey())
```
21. first() 返回第一个 take(N) 返回前N个,takeSample(withReplacement,fract)乐死sample，区别在于这是个action，直接返回结果到driver

##### 2.3 Spark的一些注意点

1. 每个计算任务必须可以拆分并行

2. 计算会对应到每个文件块

3. 提高容错两种手段：保存数据集和状态到介质里 or 根据rdd依赖推算

##### 2.4 弹性分布式数据集

RDD特性：

 1. 惰性求值，只有collect，reduce才会开始计算
 
 2. 分区
 
 3. RDD是只读的
 
 4. RDD容错高，保存RDD之间的依赖，当RDD2计算错误，从RDD1计算回来，缓存
 
弹性分布式数据集：

 1. RDD可以运行在集群中，
 
 2. 高容错，RDD数据可以缓存
 
 3. RDD可以不保存具体数据，只保留必备信息（依赖和计算函数）
 
 ###### 2.5 shffle
 
 Maper1------------->reducer1 
      |---------------
                     |
 Maper2------------->reducer2
 
 Maper3
 
 Mapper1 --> reducer1 ,Mapper1 -->reducer2,Mapper2 --> reducer1 .........
 
 shuffle 分为mapper端和reduce端，mapper将数据放入partition的函数计算，求得分到哪个reducer
 
 [例子](https://www.jianshu.com/p/7f8d4484bfbd）
 
##### 2.6 RDD支持的数据类型

String,数字，KV，对象

kv：类型 省略



数字类型支持（都是action）：

1. count
2. mean 均值
3. max min sum
4. variance 方差
5. sampleVariance 采样中计算方差
6. stdev 标准差
7. sampleStdev 采样中计算标准差
8. ............很多

#### 2.0 spark core

主要内容就是RDD


#### 3.0 案例（统计北京天气）

1. 读取文件
2. 抽取需要的列
3. 按照日，时为基础，运行reduce 统计东西地区pm
4. 排序，获取结果

```scala
package com.tr.spark

import org.apache.commons.lang3.StringUtils
import org.apache.spark.{SparkConf, SparkContext}

/**
 * @author tr
 * @Date 11/19/20
 */
object StagePractice extends App {

  def pmProcess(): Unit = {
    // 1. 创建SC  读取文件
    val conf = new SparkConf().setMaster("local[6]").setAppName("stage_practice")
    val sc = new SparkContext(conf)
    val weatherSource = sc.textFile("data/beijing_all_20200101.csv")
    // 2. 算子处理
    // 2.1 抽取数据 取date,hour作为key 东西作为value ( (key,value), value )
    // 2.2 数据清洗
    // 2.3 聚合
    //    weatherSource.map(_.split(",")).foreach(item=>{
    //      item.foreach(x=>print(x+" || "))
    //      println("\n*********")
    //    })
    val resultRdd = weatherSource
      .map(_.split(",")).filter(x=>  x.size>2 && x(2).equalsIgnoreCase("PM2.5") )
      .map(item => ((item(0), item(1)), item(3)))
      .filter(item => StringUtils.isNotEmpty(item._2) && !item._2.equalsIgnoreCase("NA"))
      .map(item => (item._1, item._2.toInt))
      .reduceByKey(_ + _)
      // 按照第二项排序
      .sortBy(_._2,false)
    // 3. 获取结果
    resultRdd.take(10).foreach(println)

    sc.stop()
  }

  pmProcess()

}

```

#### 4.0 RDD 特性

##### 4.1 RDD分区和shuffle

分区作用：

1. RDD经常需要读取外部系统文件创建（那么外部存储系统往往是支持分片的，Rdd需要分区来和外部系统的文件分片一一对应）

2. Rdd的分区是一个并行计算的实现手段

shuffle特点：

只有 kV类型有shuffle

查看RDD分区
1. 进入控制台 `spark-shell --master local[6]`
2. 执行一个rdd `sc.parallelize(Seq(1,2,3,4,5,6,7,8,9))`
3. 进入webUI查看 `http://localhost:4040`

怎么创建分区：

1. 读取外部文件时指定分区数量	`sc.parallize(Seq(1,2,3),2)`
2. 通过本地集合创建时指定分区数量 `sc.textFile("/data/x.txt",2)`

怎么重分区：

1. coalesce（N，false）：可以将分区缩小，如果需要扩大分区，指定shuffle：true

2. repartitions(N): 相当于coalesce的默认shuffle为true

通过其他算子指定分区：一般通过shuffle的算子都可以手动指定分区数，如果没有指定，默认从父节点继承

shuffle过程简介：

`rdd2 = rdd1.reduceByKey()`  实质是rdd2的调用函数，rdd2调用这个函数从rdd1拉取数据
，那么如何确定数据流入哪个分区，通过Partitioner函数：HashPartitioner，rdd2的分区和rdd1的分区是交错联系的，rdd2的每个分区去rdd1的每个分区内拉取数据

##### 4.2 RDD缓存

##### 4.3 RDD的checkpoint



什么是checkpoint？ 斩断RDD的依赖链

方式： 本地存储，可靠的:缓存在hdfs上

Rdd之间有很多依赖关系，依赖链过长的话当某个rdd错误，需要追溯很久，斩断依赖链，就是不用计算之前的依赖链。

但是如果rdd错误，且之前的rdd已经斩断，正常情况下，可以重放，从上一个被斩断的节点开始(这个节点的结果已经被存放在外部可靠介质中,直接取出结果)

checkpoint本质还是缓存，但是和cache的区别是：

1. checkpoint 数据可以保存在hdfs这类可靠介质内，cache和persist只能放在内存和磁盘
2. checkpoint可以斩断依赖链，但是cache和persist不可以

使用checkpoint

```scala
 // 1.读取文件
    val sourceRdd = sc.textFile("data/access_log.txt")
    // 2.取出ip
    val ipRdd = sourceRdd.map(x => (x.split(" ")(0), 1))
    // 3.简单清洗 去掉空数据 去掉非法数据 根据业务再规整数据
    val cleanRdd = ipRdd.filter(x => StringUtils.isNotEmpty(x._1))
    // 4.根据ip次数聚合
    var aggRdd = cleanRdd.reduceByKey(_ + _)

    // 设置checkpoint,调用的时候前面计算会执行一遍，将结果放入目录，因为checkpoint是先等待job执行完后启动一个线程去计算要checkpoint的内容
    // 所以应该在checkpoint之前进行一次cache，第一次就将结果缓存到内存，调用checkpoint的时候拿缓存的数据写入外部介质
    aggRdd = aggRdd.cache()
    aggRdd.checkpoint()

    val lessIp = aggRdd.sortBy(_._2,true).first()
    val moreIp = aggRdd.sortBy(_._2,false).first()
```