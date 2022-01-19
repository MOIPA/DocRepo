title: Scala-Spark踩坑
author: tr
tags:
  - Spark
categories:
  - Spark
date: 2020-11-17 10:28:00
---
### Scala Spark 在idea下的错误记录

百度真坑爹，一群人互相抄作业，浪费时间

#### Object apache is not a member of package org...

idea下，使用默认的scala插件开发

错误描述：这个错误是缺少jar配置文件导致的，但是奇怪的是在linux系统下会出现这个问题，但是windows下使用msi安装sbt不会出现这个问题，idea真神奇

错误解决：
1. 导入spark下的jars（2.0版本后）
2. 下载 [spark-assembly-1.6.0-hadoop2.6.0-6.0.0.jar](https://talend-update.talend.com/nexus/content/repositories/libraries/org/talend/libraries/spark-assembly-1.6.0-hadoop2.6.0/6.0.0/) 然后导入即可，这个包包含了全部的集成环境

3. 不要使用maven了，利用sbt构建环境，新建scala项目，右边选择sbt，在打开的build.sbt中写入以下依赖,版本自行决定
```scala
name := "LearningSparkWithSBT"

version := "0.1"

scalaVersion := "2.11.8"
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-core" % "2.2.0",
  "org.apache.spark" %% "spark-sql" % "2.2.0"
)
```

NOTE: 版本一定要保持一直，spark和对于的scala版本