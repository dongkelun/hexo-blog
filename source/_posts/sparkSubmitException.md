---
title: spark-submit报错:Exception in thread "main" java.sql.SQLException:No suitable driver
date: 2018-05-06
tags:
  - spark
  - spark-submit
copyright: true
---
## 前言
最近写了一个用spark连接oracle,然后将mysql所有的表保存到hive中的程序，在本地eclipse里运行没有问题，想在集群上跑一下，看看在集群上性能如何，但是用spark-submit 提交程序时抛出一个异常Exception in thread "main" java.sql.SQLException: No suitable driver，一开始以为spark-submit提交时找不到oracle 驱动jar,折腾了半天才发现是代码问题。
## 1、猜测是否是缺失oracle驱动
由于在本地没有问题，所以不会想到是代码问题，根据提示想到的是spark-submit找不到oracle驱动，因为maven或sbt仓库里没有oracle驱动，在本地跑的时候，是将oracle驱动下载到本地，然后在eclipse设置build path就可以了。

但是我在spark-submit 里已经通过--jars 加载oracle驱动了：
```bash
spark-submit --class com.dkl.leanring.spark.sql.Oracle2HiveDemo --jars ojdbc5-11.2.0.3.jar spark-scala_2.11-1.0.jar
```
开始以为自己用法不对，但是上网搜了一下，发现就是这么用的~
然后尝试用--driver-class-path、--driver-library-path等都没成功。
<!-- more -->
## 2、sbt-assembly打包
网上查的sbt-assembly打包可以将所有依赖的jar包包括你写的代码全部打包在一起，于是尝试了一下
首先在项目目录中project/plugins.sbt添加
```
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.5")
resolvers += Resolver.url("bintray-sbt-plugins", url("http://dl.bintray.com/sbt/sbt-plugin-releases"))(Resolver.ivyStylePatterns)
```
其中0.14.5为版本号，需要与自己sbt对应上，否则报错，可在http://dl.bintray.com/sbt/sbt-plugin-releases/com.eed3si9n/sbt-assembly/查看版本
然后在项目对应目录下执行sbt,然后输入plugins，即可看到sbtassembly插件了，如下：
```sbt
$ sbt
[info] Loading settings from plugins.sbt ...
[info] Loading global plugins from C:\Users\14123\.sbt\1.0\plugins
[info] Loading settings from assembly.sbt,plugins.sbt ...
[info] Loading project definition from D:\workspace\spark-scala\project
[info] Loading settings from spark-scala.sbt ...
[info] Set current project to spark-scala (in build file:/D:/workspace/spark-scala/)
[info] sbt server started at local:sbt-server-8b6c904d40b181717b3f
sbt:spark-scala> plugins
In file:/D:/workspace/spark-scala/
        sbt.plugins.IvyPlugin: enabled in spark-scala
        sbt.plugins.JvmPlugin: enabled in spark-scala
        sbt.plugins.CorePlugin: enabled in spark-scala
        sbt.plugins.JUnitXmlReportPlugin: enabled in spark-scala
        sbt.plugins.Giter8TemplatePlugin: enabled in spark-scala
        com.typesafe.sbteclipse.plugin.EclipsePlugin: enabled in spark-scala
        sbtassembly.AssemblyPlugin: enabled in spark-scala
sbt:spark-scala>

```

但是这样执行sbt-assembly打包会报错，需要解决jar包冲突(deduplicate)问题
在项目的bulid.sbt里添加如下即可（只是其中一种解决策略，可根据自己项目实际情况自己设定）
```txt
assemblyMergeStrategy in assembly := {
    case m if m.toLowerCase.endsWith("manifest.mf") => MergeStrategy.discard
    case m if m.startsWith("META-INF") => MergeStrategy.discard
    case PathList("javax", "servlet", xs @ _*) => MergeStrategy.first
    case PathList("org", "apache", xs @ _*) => MergeStrategy.first
    case PathList("org", "jboss", xs @ _*) => MergeStrategy.first
    case "about.html"  => MergeStrategy.rename
    case "reference.conf" => MergeStrategy.concat
    case _ => MergeStrategy.first
}
```
其中不同旧版本和新版本sbt写法不一样，具体可上网看一下别人的博客或者在官网查看。

这样就可以sbt-assembly进行打包了，发现这样打的jar包确实很大，用sbt package打的jar包大小1.48MB,sbt-assembly打的jar包大小194MB,将spark-scala-assembly-1.0.jar上传到服务器，然后执行submit,发现还是报同样的错，查看一下sbt-assembly日志，发现确实将oracle驱动加载上了~
## 3、真正原因
这样就猜想不是缺少oracle驱动，于是上网查了好多，偶然发现可能是代码问题，下面是我写的从oracle取数的部分代码
```scala
val allTablesDF = spark.read
  .format("jdbc")
  .option("url", "jdbc:oracle:thin:@192.168.44.128:1521:orcl")
  .option("dbtable", "(select table_name,owner from all_tables where  owner  in('BIGDATA'))a")
  .option("user", "bigdata")
  .option("password", "bigdata")
  .load()
```
写法和我之前写的spark连接mysql的博客里的写法是一样的：[Spark Sql 连接mysql](http://dongkelun.com/2018/03/21/sparkMysql/)
这样写在eclipse运行是没问题的，但是在spark-submit提交时是不行的，需要加上驱动信息
```
.option("driver", "oracle.jdbc.driver.OracleDriver")
```
重新打包，再运行，发现果然没问题
## 4、总结
### 4.1
其实在用spark提交之前写的spark连接mysql的程序也会报统一的错(如果$SPARK_HOME/jars没有mysql驱动)，和oracle驱动不在sbt仓库里没关系。
但是之前在spark-shell里测试spark连接hive时已经将mysql驱动拷贝过去了，所以mysql没有报错
### 4.2
在代码里加上driver之后再提交如果没有oracle驱动会报不同的错
```
Exception in thread "main" java.lang.ClassNotFoundException: oracle.jdbc.driver.OracleDriver
```
### 4.3 
通过--jars指定jar和sbt assembly打包都可以，看自己习惯，但通过--jars需要注意先后顺序，如果是多个jar以逗号隔开即可
正确：
```bash
spark-submit --class com.dkl.leanring.spark.sql.Oracle2HiveDemo --jars ojdbc5-11.2.0.3.jar spark-scala_2.11-1.0.jar 
spark-submit --jars ojdbc5-11.2.0.3.jar --class com.dkl.leanring.spark.sql.Oracle2HiveDemo  spark-scala_2.11-1.0.jar 
```
错误：
```bash
spark-submit --class com.dkl.leanring.spark.sql.Oracle2HiveDemo  spark-scala_2.11-1.0.jar --jars ojdbc5-11.2.0.3.jar
spark-submit --jars ojdbc5-11.2.0.3.jar spark-scala_2.11-1.0.jar --class com.dkl.leanring.spark.sql.Oracle2HiveDemo 
...
```
也就是通过sbt package和sbt assembly生成的项目jar包一定要放在最后面
### 4.4 
通过--driver-class-path也可以实现加载额外的jar
```bash
spark-submit --class com.dkl.leanring.spark.sql.Oracle2HiveDemo --driver-class-path lib/*  spark-scala_2.11-1.0.jar
```
但是这样lib下只能一个jar,两个jar就会报错，不知道什么原因~
### 4.5 
将oracle驱动拷贝到$SPARK_HOME/jars,就可以不在代码里指定driver选项了，而且也不用通过--jars添加oracle驱动，一劳永逸.
```bash
cp ojdbc5-11.2.0.3.jar $SPARK_HOME/jars
spark-submit --class com.dkl.leanring.spark.sql.Oracle2HiveDemo  spark-scala_2.11-1.0.jar
```
具体这样设置可根据实际情况和偏好习惯使用。
## 附完整代码（测试用）
比较简单就不加注释~
```scala
package com.dkl.leanring.spark.sql

import org.apache.spark.sql.SparkSession

object Oracle2HiveDemo {

  def main(args: Array[String]): Unit = {
    val spark = SparkSession
      .builder()
      .appName("Oracle2HiveDemo")
      .master("local")
      .enableHiveSupport()
      .getOrCreate()

    val allTablesDF = spark.read
      .format("jdbc")
      .option("url", "jdbc:oracle:thin:@192.168.44.128:1521:orcl")
      .option("dbtable", "(select table_name,owner from all_tables where  owner  in('BIGDATA'))a")
      .option("user", "bigdata")
      .option("password", "bigdata")
      .option("driver", "oracle.jdbc.driver.OracleDriver")
      .load()
    import spark.implicits._
    import spark.sql
    sql("use oracle_test")
    allTablesDF.rdd.collect().foreach(row => {
      val tableName: String = row(0).toString()
      val dataBase: String = row(1).toString()

      println(dataBase + "." + tableName)
      val df = spark.read
        .format("jdbc")
        .option("url", "jdbc:oracle:thin:@192.168.44.128:1521:orcl")
        .option("dbtable", dataBase + "." + tableName)
        .option("user", "bigdata")
        .option("password", "bigdata")
        .option("driver", "oracle.jdbc.driver.OracleDriver")
        .load()
      df.write.mode("overwrite").saveAsTable(tableName)

    })

    spark.stop

  }
}
```
