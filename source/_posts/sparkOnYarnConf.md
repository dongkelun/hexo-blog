---
title: spark on yarn 配置及异常解决
date: 2018-04-16
tags:
  - spark
  - yarn
copyright: true
---
## 前言
YARN 是在Hadoop 2.0 中引入的集群管理器，它可以让多种数据处理框架运行在一个共享的资源池上，并且通常安装在与Hadoop 文件系统（简称HDFS）相同的物理节点上。在这样配置的YARN 集群上运行Spark 是很有意义的，它可以让Spark 在存储数据的物理节点上运行，以快速访问HDFS 中的数据。
## 1、配置
### 1.1 配置HADOOP_CONF_DIR
```bash
vim /etc/profile
```
```bash
export HADOOP_CONF_DIR=/opt/hadoop-2.7.5/etc/hadoop
```
```bash
source /etc/profile
```
<!-- more -->
### 1.2 命令行启动
``` bash
spark-shell --master yarn
```
但是在spark2.x里会报一个错误
```
18/04/16 07:59:23 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
18/04/16 07:59:27 WARN Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
18/04/16 07:59:54 ERROR SparkContext: Error initializing SparkContext.
org.apache.spark.SparkException: Yarn application has already ended! It might have been killed or unable to launch application master.
	at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.waitForApplication(YarnClientSchedulerBackend.scala:85)
	at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.start(YarnClientSchedulerBackend.scala:62)
	at org.apache.spark.scheduler.TaskSchedulerImpl.start(TaskSchedulerImpl.scala:173)
	at org.apache.spark.SparkContext.<init>(SparkContext.scala:509)
	at org.apache.spark.SparkContext$.getOrCreate(SparkContext.scala:2516)
	at org.apache.spark.sql.SparkSession$Builder$$anonfun$6.apply(SparkSession.scala:918)
	at org.apache.spark.sql.SparkSession$Builder$$anonfun$6.apply(SparkSession.scala:910)
	at scala.Option.getOrElse(Option.scala:121)
	at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:910)
	at org.apache.spark.repl.Main$.createSparkSession(Main.scala:101)
	at $line3.$read$$iw$$iw.<init>(<console>:15)
	at $line3.$read$$iw.<init>(<console>:42)
	at $line3.$read.<init>(<console>:44)
	at $line3.$read$.<init>(<console>:48)
	at $line3.$read$.<clinit>(<console>)
	at $line3.$eval$.$print$lzycompute(<console>:7)
	at $line3.$eval$.$print(<console>:6)
	at $line3.$eval.$print(<console>)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at scala.tools.nsc.interpreter.IMain$ReadEvalPrint.call(IMain.scala:786)
	at scala.tools.nsc.interpreter.IMain$Request.loadAndRun(IMain.scala:1047)
	at scala.tools.nsc.interpreter.IMain$WrappedRequest$$anonfun$loadAndRunReq$1.apply(IMain.scala:638)
	at scala.tools.nsc.interpreter.IMain$WrappedRequest$$anonfun$loadAndRunReq$1.apply(IMain.scala:637)
	at scala.reflect.internal.util.ScalaClassLoader$class.asContext(ScalaClassLoader.scala:31)
	at scala.reflect.internal.util.AbstractFileClassLoader.asContext(AbstractFileClassLoader.scala:19)
	at scala.tools.nsc.interpreter.IMain$WrappedRequest.loadAndRunReq(IMain.scala:637)
	at scala.tools.nsc.interpreter.IMain.interpret(IMain.scala:569)
	at scala.tools.nsc.interpreter.IMain.interpret(IMain.scala:565)
	at scala.tools.nsc.interpreter.ILoop.interpretStartingWith(ILoop.scala:807)
	at scala.tools.nsc.interpreter.ILoop.command(ILoop.scala:681)
	at scala.tools.nsc.interpreter.ILoop.processLine(ILoop.scala:395)
	at org.apache.spark.repl.SparkILoop$$anonfun$initializeSpark$1.apply$mcV$sp(SparkILoop.scala:38)
	at org.apache.spark.repl.SparkILoop$$anonfun$initializeSpark$1.apply(SparkILoop.scala:37)
	at org.apache.spark.repl.SparkILoop$$anonfun$initializeSpark$1.apply(SparkILoop.scala:37)
	at scala.tools.nsc.interpreter.IMain.beQuietDuring(IMain.scala:214)
	at org.apache.spark.repl.SparkILoop.initializeSpark(SparkILoop.scala:37)
	at org.apache.spark.repl.SparkILoop.loadFiles(SparkILoop.scala:98)
	at scala.tools.nsc.interpreter.ILoop$$anonfun$process$1.apply$mcZ$sp(ILoop.scala:920)
	at scala.tools.nsc.interpreter.ILoop$$anonfun$process$1.apply(ILoop.scala:909)
	at scala.tools.nsc.interpreter.ILoop$$anonfun$process$1.apply(ILoop.scala:909)
	at scala.reflect.internal.util.ScalaClassLoader$.savingContextLoader(ScalaClassLoader.scala:97)
	at scala.tools.nsc.interpreter.ILoop.process(ILoop.scala:909)
	at org.apache.spark.repl.Main$.doMain(Main.scala:74)
	at org.apache.spark.repl.Main$.main(Main.scala:54)
	at org.apache.spark.repl.Main.main(Main.scala)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.apache.spark.deploy.SparkSubmit$.org$apache$spark$deploy$SparkSubmit$$runMain(SparkSubmit.scala:775)
	at org.apache.spark.deploy.SparkSubmit$.doRunMain$1(SparkSubmit.scala:180)
	at org.apache.spark.deploy.SparkSubmit$.submit(SparkSubmit.scala:205)
	at org.apache.spark.deploy.SparkSubmit$.main(SparkSubmit.scala:119)
	at org.apache.spark.deploy.SparkSubmit.main(SparkSubmit.scala)
18/04/16 07:59:54 WARN YarnSchedulerBackend$YarnSchedulerEndpoint: Attempted to request executors before the AM has registered!
18/04/16 07:59:54 WARN MetricsSystem: Stopping a MetricsSystem that is not running
org.apache.spark.SparkException: Yarn application has already ended! It might have been killed or unable to launch application master.
  at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.waitForApplication(YarnClientSchedulerBackend.scala:85)
  at org.apache.spark.scheduler.cluster.YarnClientSchedulerBackend.start(YarnClientSchedulerBackend.scala:62)
  at org.apache.spark.scheduler.TaskSchedulerImpl.start(TaskSchedulerImpl.scala:173)
  at org.apache.spark.SparkContext.<init>(SparkContext.scala:509)
  at org.apache.spark.SparkContext$.getOrCreate(SparkContext.scala:2516)
  at org.apache.spark.sql.SparkSession$Builder$$anonfun$6.apply(SparkSession.scala:918)
  at org.apache.spark.sql.SparkSession$Builder$$anonfun$6.apply(SparkSession.scala:910)
  at scala.Option.getOrElse(Option.scala:121)
  at org.apache.spark.sql.SparkSession$Builder.getOrCreate(SparkSession.scala:910)
  at org.apache.spark.repl.Main$.createSparkSession(Main.scala:101)
  ... 47 elided
<console>:14: error: not found: value spark
       import spark.implicits._
              ^
<console>:14: error: not found: value spark
       import spark.sql
              ^
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.2.1
      /_/
         
Using Scala version 2.11.8 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_45)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 

```
## 2、错误解决
### 2.1 添加spark.yarn.jars
首先看到第二条warn
```bash
Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
```
联想到是不是这条warn信息导致的，然后根据这条warn信息上网查了一下，再根据错误信息也查了一下
```
Yarn application has already ended! It might have been killed or unable to ...
```
发现，都是说要配置spark.yarn.jars，于是按照如下命令配置
``` bash
hdfs dfs -mkdir /hadoop
hdfs dfs -mkdir /hadoop/spark_jars
hdfs dfs -put /opt/spark-2.2.1-bin-hadoop2.7/jars/* /hadoop/spark_jars
cd /opt/spark-2.2.1-bin-hadoop2.7/conf/
cp spark-defaults.conf.template spark-defaults.conf
vim spark-defaults.conf
```
在最下面添加：
```
spark.yarn.jars hdfs://192.168.44.128:8888/hadoop/spark_jars/*
```
(注意后面的*不能去掉)
然后启动spark-shell,发现还是报相似错误（没了warn）
### 2.2 配置hadoop的yarn-site.xml
因为java8导致的问题
```bash
vim /opt/hadoop-2.7.5/etc/hadoop/yarn-site.xml
```
添加：
```
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>

<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property> 
```
再次启动spark-shell,成功！
![](http://wx2.sinaimg.cn/large/e44344dcly1fqerons9kej20uk07bdg6.jpg)
## 3、意外之喜
由于要写博客记录，所以需要将错误还原，第一次只将spark.yarn.jars注释掉，启动spark-shell,发现是成功的，只是会有条warn而已，也就是说，这个错误的根本原因，是java8导致没有配置2.2中的yarn-site.xml！！
![](http://wx1.sinaimg.cn/large/e44344dcly1fqeroo8ic2j20w5073t95.jpg)
## 参考资料
[https://blog.csdn.net/lxhandlbb/article/details/54410644](https://blog.csdn.net/lxhandlbb/article/details/54410644)
[https://blog.csdn.net/gg584741/article/details/72825713](https://blog.csdn.net/gg584741/article/details/72825713)



