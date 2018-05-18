---
title: Spark Sql 连接mysql
date: 2018-03-21
tags:
  - spark
  - scala
copyright: true
---

## 1、基本概念和用法（摘自spark官方文档中文版）

Spark SQL 还有一个能够使用 JDBC 从其他数据库读取数据的数据源。当使用 JDBC 访问其它数据库时，应该首选 JdbcRDD。这是因为结果是以数据框（DataFrame）返回的，且这样 Spark SQL操作轻松或便于连接其它数据源。因为这种 JDBC 数据源不需要用户提供 ClassTag，所以它也更适合使用 Java 或 Python 操作。（注意，这与允许其它应用使用 Spark SQL 执行查询操作的 Spark SQL JDBC 服务器是不同的）。

使用 JDBC 访问特定数据库时，需要在 spark classpath 上添加对应的 JDBC 驱动配置。例如，为了从 Spark Shell 连接 postgres，你需要运行如下命令 : 
```bash
bin/spark-shell --driver-class-path postgresql-9.4.1207.jar --jars postgresql-9.4.1207.jar
```
通过调用数据源API，远程数据库的表可以被加载为DataFrame 或Spark SQL临时表。支持的参数有 : 

属性名 | 含义
- | :-: 
url | 要连接的 JDBC URL。
dbtable | 要读取的 JDBC 表。 注意，一个 SQL 查询的 From 分语句中的任何有效表都能被使用。例如，既可以是完整表名，也可以是括号括起来的子查询语句。 
driver | 用于连接 URL 的 JDBC 驱动的类名。
partitionColumn, lowerBound, upperBound, numPartitions | 这几个选项，若有一个被配置，则必须全部配置。它们描述了当从多个 worker 中并行的读取表时，如何对它分区。partitionColumn 必须时所查询表的一个数值字段。注意，lowerBound 和 upperBound 都只是用于决定分区跨度的，而不是过滤表中的行。因此，表中的所有行将被分区并返回。
fetchSize | JDBC fetch size，决定每次读取多少行数据。 默认将它设为较小值（如，Oracle上设为 10）有助于 JDBC 驱动上的性能优化。
<!-- more -->
## 2、scala代码实现连接mysql
### 2.1 添加mysql 依赖
在sbt 配置文件里添加：
```
"mysql" % "mysql-connector-java" % "6.0.6"
```
然后执行：
```bash
sbt eclipse
```
### 2.2 建表并初始化数据

```sql
DROP TABLE IF EXISTS `USER_T`;  
CREATE TABLE `USER_T` (  
  `ID` INT(11) NOT NULL,  
  `USER_NAME` VARCHAR(40) NOT NULL,  
  PRIMARY KEY (`ID`)  
) ENGINE=INNODB  DEFAULT CHARSET=UTF8;  
```
```sql
INSERT  INTO `USER_T`(`ID`,`USER_NAME`) VALUES (1,'测试1');
INSERT  INTO `USER_T`(`ID`,`USER_NAME`) VALUES (2,'测试2');
```
![](http://wx4.sinaimg.cn/large/e44344dcly1fpqmb7zd4gj208e04n3yh.jpg)
### 2.3 代码
#### 2.3.1 查询
``` scala
package com.dkl.leanring.spark.sql

import org.apache.spark.sql.SparkSession

/**
 * spark查询mysql测试
 */
object MysqlQueryDemo {

  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("MysqlQueryDemo").master("local").getOrCreate()
    val jdbcDF = spark.read
      .format("jdbc")
      .option("url", "jdbc:mysql://192.168.44.128:3306/hive?useUnicode=true&characterEncoding=utf-8")
      .option("dbtable", "USER_T")
      .option("user", "root")
      .option("password", "Root-123456")
      .load()
    jdbcDF.show()
  }
}
```
![](http://wx3.sinaimg.cn/large/e44344dcly1fprgqfbd99j213y0kf0v5.jpg)
#### 2.3.2 插入数据

新建USER_T.csv,造几条数据如图：
（需将csv的编码格式转为utf-8,否则spark读取中文乱码，转码方法见：[https://jingyan.baidu.com/article/fea4511a092e53f7bb912528.html](https://jingyan.baidu.com/article/fea4511a092e53f7bb912528.html)）
![](http://wx4.sinaimg.cn/large/e44344dcly1fpqmb942xfj20vz09z0tj.jpg)

```scala
package com.dkl.leanring.spark.sql

import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.SaveMode
import java.util.Properties

/**
 * 从USER_T.csv读取数据并插入的mysql表中
 */
object MysqlInsertDemo {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("MysqlInsertDemo").master("local").getOrCreate()
    val df = spark.read.option("header", "true").csv("src/main/resources/scala/USER_T.csv")
    df.show()
    val url = "jdbc:mysql://192.168.44.128:3306/hive?useUnicode=true&characterEncoding=utf-8"
    val prop = new Properties()
    prop.put("user", "root")
    prop.put("password", "Root-123456")
    df.write.mode(SaveMode.Append).jdbc(url, "USER_T", prop)
  }
}

```
![](http://wx4.sinaimg.cn/large/e44344dcly1fpqmb9pmnmj213v0o1tbo.jpg)

再查询一次，就会发现表里多了几条数据

![](http://wx4.sinaimg.cn/large/e44344dcly1fpqmba88iyj215n07r0t8.jpg)