---
title: 旧版spark（1.6版本） 将rdd动态转为dataframe
date: 2018-05-11
tags:
  - spark
  - DataFrame
  - Rdd
copyright: true
---
## 前言
旧版本spark不能直接读取csv转为df,没有spark.read.option("header", "true").csv这么简单的方法直接将第一行作为df的列名，只能现将数据读取为rdd,然后通过map和todf方法转为df,如果csv的列数很多的话用如Array((1,2..))即Arrar(元组)创建的话很麻烦，本文解决如何用旧版spark读取多列txt文件转为df

## 1、新版
为了直观明白本文的目的，先看一下新版spark如何实现
<!-- more -->
### 1.1 数据
data.csv，如图：
![](http://wx1.sinaimg.cn/large/e44344dcly1fr7skek00uj20v209yt9j.jpg)

### 1.2 代码
新版代码较简单，直接通过spark.read.option("header", "true").csv(data_path)即可实现！

```scala
package com.dkl.leanring.spark.sql

import org.apache.spark.sql.SparkSession

object Txt2Df {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("Txt2Df").master("local").getOrCreate()
    val data_path = "files/data.csv"
    val df = spark.read.option("header", "true").csv(data_path)
    df.show()
  }
}
```
### 1.3 结果
```
+----+----+----+----+----+
|col1|col2|col3|col4|col5|
+----+----+----+----+----+
|  11|  12|  13|  14|  15|
|  21|  22|  23|  24|  25|
|  31|  32|  33|  34|  35|
|  41|  42|  43|  44|  45|
+----+----+----+----+----+
```
## 2、旧版
### 2.1 数据
data.txt
```
col1,col2,col3,col4,col5
11,12,13,14,15
21,22,23,24,25
31,32,33,34,35
41,42,43,44,45
```
其中列数可任意指定
### 2.2 代码
```scala
package com.dkl.leanring.spark.sql

import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.sql.SQLContext
import org.apache.spark.sql.types._
import org.apache.spark.sql.Row
object Rdd2Df {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("Rdd2Df").setMaster("local")
    val sc = new SparkContext(conf)
    val sqlContext = new SQLContext(sc)
    import sqlContext.implicits._
    val data_path = "files/data.txt"
    val data = sc.textFile(data_path)
    val arr = data.collect()
    //arr1为除去第一行即列名的数据
    val arr1 = arr.slice(1, arr.length)
    val rdd = sc.parallelize(arr1)
    //列名
    val schema = StructType(arr(0).split(",").map(fieldName => StructField(fieldName, StringType, true)))
    val rowRDD = rdd.map(_.split(",")).map(p => Row(p: _*))
    sqlContext.createDataFrame(rowRDD, schema).show()

  }
}
```
### 2.3 结果
```
+----+----+----+----+----+
|col1|col2|col3|col4|col5|
+----+----+----+----+----+
|  11|  12|  13|  14|  15|
|  21|  22|  23|  24|  25|
|  31|  32|  33|  34|  35|
|  41|  42|  43|  44|  45|
+----+----+----+----+----+
```
根据结果看，符合逾期的效果！
