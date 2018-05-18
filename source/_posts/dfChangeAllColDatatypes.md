---
title: spark 将DataFrame所有的列类型改为double
date: 2018-04-27
tags:
  - spark
  - DataFrame
copyright: true
---

## 前言
由于spark机器学习要求输入的DataFrame类型为数值类型，所以如果原始数据读进来的列为string类型，需要一一转化，而如果列很多的情况下一个转化很麻烦，所以能不能一个循环或者一个函数去解决呢。
## 1、单列转化方法
```scala
import org.apache.spark.sql.types._
val data = Array(("1", "2", "3", "4", "5"), ("6", "7", "8", "9", "10"))
val df = spark.createDataFrame(data).toDF("col1", "col2", "col3", "col4", "col5")

import org.apache.spark.sql.functions._
df.select(col("col1").cast(DoubleType)).show()
```
```
+----+
|col1|
+----+
| 1.0|
| 6.0|
+----+
```
<!-- more -->
## 2、循环转变
然后就想能不能用这个方法循环把每一列转成double，但没想到怎么实现，可以用withColumn循环实现。
```scala
val colNames = df.columns

var df1 = df
for (colName <- colNames) {
  df1 = df1.withColumn(colName, col(colName).cast(DoubleType))
}
df1.show()
```
```
+----+----+----+----+----+
|col1|col2|col3|col4|col5|
+----+----+----+----+----+
| 1.0| 2.0| 3.0| 4.0| 5.0|
| 6.0| 7.0| 8.0| 9.0|10.0|
+----+----+----+----+----+
```
## 3、通过:_*
但是上面这个方法效率比较低，然后问了一下别人，发现scala 有array:_*这样传参这种语法，而df的select方法也支持这样传，于是最终可以按下面的这样写

```scala
val cols = colNames.map(f => col(f).cast(DoubleType))
df.select(cols: _*).show()
```
```
+----+----+----+----+----+
|col1|col2|col3|col4|col5|
+----+----+----+----+----+
| 1.0| 2.0| 3.0| 4.0| 5.0|
| 6.0| 7.0| 8.0| 9.0|10.0|
+----+----+----+----+----+
```
这样就可以很方便的查询指定多列和转变指定列的类型了：
```scala
val name = "col1,col3,col5"
df.select(name.split(",").map(name => col(name)): _*).show()
df.select(name.split(",").map(name => col(name).cast(DoubleType)): _*).show()
```
```
+----+----+----+
|col1|col3|col5|
+----+----+----+
|   1|   3|   5|
|   6|   8|  10|
+----+----+----+

+----+----+----+
|col1|col3|col5|
+----+----+----+
| 1.0| 3.0| 5.0|
| 6.0| 8.0|10.0|
+----+----+----+
```
附完整代码：
```scala
package com.dkl.leanring.spark.test

import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.types._
import org.apache.spark.sql.DataFrame
object DfDemo {

  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("DfDemo").master("local").getOrCreate()
    import org.apache.spark.sql.types._
    val data = Array(("1", "2", "3", "4", "5"), ("6", "7", "8", "9", "10"))
    val df = spark.createDataFrame(data).toDF("col1", "col2", "col3", "col4", "col5")

    import org.apache.spark.sql.functions._
    df.select(col("col1").cast(DoubleType)).show()

    val colNames = df.columns

    var df1 = df
    for (colName <- colNames) {
      df1 = df1.withColumn(colName, col(colName).cast(DoubleType))
    }
    df1.show()

    val cols = colNames.map(f => col(f).cast(DoubleType))
    df.select(cols: _*).show()
    val name = "col1,col3,col5"
    df.select(name.split(",").map(name => col(name)): _*).show()
    df.select(name.split(",").map(name => col(name).cast(DoubleType)): _*).show()

  }
```
