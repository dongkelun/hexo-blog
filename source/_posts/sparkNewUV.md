---
title: spark 统计每天新增用户数
date: 2018-04-11
tags:
  - spark
  - 面试题
  - scala
copyright: true
---
## 前言
本文源自一位群友的一道美团面试题，解题思路（基于倒排索引）和代码都是这位大佬（相对于尚处于小白阶段的我）写的，我只是在基于倒排索引的基础上帮忙想出了最后一步思路，感觉这个解题思路不错，值得记录一下。
## 1、原始数据
```
2017-01-01	a
2017-01-01	b
2017-01-01	c
2017-01-02	a
2017-01-02	b
2017-01-02	d
2017-01-03	b
2017-01-03	e
2017-01-03	f
```
根据数据可以看出我们要求的结果为：
2017-01-01 新增三个用户（a,b,c）
2017-01-02 新增一个用户（d）
2017-01-03 新增两个用户（e，f）
<!-- more -->
## 2、解题思路
### 2.1 对原始数据进行倒排索引
结果如下：

用户名 | 列一 | 列二 | 列三
- | :-:  | :-:  | :-: 
a | 2017-01-01 | 2017-01-02 |
b | 2017-01-01 | 2017-01-02 | 2017-01-03
c | 2017-01-01 |            |
d | 2017-01-02 |            |  
e | 2017-01-03 |            |
f | 2017-01-03 |            |

### 2.2 统计列一中每个日期出现的次数
这样我们只看列一，统计每个日期在列一出现的次数，即为对应日期新增用户数。

## 3、代码
```scala
package com.dkl.leanring.spark.test

import org.apache.spark.sql.SparkSession

object NewUVDemo {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("NewUVDemo").master("local").getOrCreate()
    val rdd1 = spark.sparkContext.parallelize(
      Array(
        ("2017-01-01", "a"), ("2017-01-01", "b"), ("2017-01-01", "c"),
        ("2017-01-02", "a"), ("2017-01-02", "b"), ("2017-01-02", "d"),
        ("2017-01-03", "b"), ("2017-01-03", "e"), ("2017-01-03", "f")))
    //倒排
    val rdd2 = rdd1.map(kv => (kv._2, kv._1))
    //倒排后的key分组
    val rdd3 = rdd2.groupByKey()
    //取最小时间
    val rdd4 = rdd3.map(kv => (kv._2.min, 1))
    rdd4.countByKey().foreach(println)
  }
}
```

结果：
```
(2017-01-03,2)
(2017-01-02,1)
(2017-01-01,3)
```
附图：
![](http://wx3.sinaimg.cn/large/e44344dcly1fqardqntzwj21480ifdi1.jpg)

