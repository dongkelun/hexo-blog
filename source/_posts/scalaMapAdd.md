---
title: scala 两个map合并，key相同时value相加
date: 2018-04-01
tags:
  - scala
copyright: true
---
## 1、先看一下map自带的合并操作的效果
```scala
val map1 = Map("key1" -> 1, "key2" -> 3, "key3" -> 5)
val map2 = Map("key2" -> 4, "key3" -> 6, "key5" -> 10)
println(map1 + ("key1" -> 3))
println(map1 ++ map2)
```
结果：
```
Map(key1 -> 3, key2 -> 3, key3 -> 5)
Map(key1 -> 1, key2 -> 4, key3 -> 6, key5 -> 10)
```
可以看到现有的方法在key相同时，没有将value相加，而是操作符右边的值把左边的值覆盖掉了。
<!-- more -->
## 2、利用map函数
### 2.1 为了便于理解先看如下代码
代码：
```scala
val map1 = Map("key1" -> 1, "key2" -> 3, "key3" -> 5)
map1.map { t => println(t._1, t._2) }
```
结果：
```
(key1,1)
(key2,3)
(key3,5)
```
可以看出map函数会遍历集合中的每个元素
可以为其指定返回结果：
```scala
println(map1.map { t => 2 })
println(map1.map { t => t._1 -> t._2 })
```
结果：
```
List(2, 2, 2)
Map(key1 -> 1, key2 -> 3, key3 -> 5)
```
可以看出，该函数返回结果类型可以不同，并且会将我们指定的值，组成一个集合，并自动判断返回类型。

### 2.2 合并两个map
```scala
val map1 = Map("key1" -> 1, "key2" -> 3, "key3" -> 5)
val map2 = Map("key2" -> 4, "key3" -> 6, "key5" -> 10)
val mapAdd1 = map1 ++ map2.map(t => t._1 -> (t._2 + map1.getOrElse(t._1, 0)))
println(mapAdd1)
```
其中map.getOrElse(key,default):如果map中有这个key,则返回对应的value,否在返回default
结果：
```
Map(key1 -> 1, key2 -> 7, key3 -> 11, key5 -> 10)
```
## 3、用foldLeft
### 3.1 语法
以下三种写法是相等的：
```scala
List(1, 2, 3, 4).foldLeft(0)((sum, i) => sum + i)
(List(1, 2, 3, 4) foldLeft 0)((sum, i) => sum + i)
(0 /: List(1, 2, 3, 4))(_ + _)  
```
为了便于理解，先看下面代码：
```scala
(0 /: List(1, 2, 3, 4))((sum, i) => {
  println(s"sum=${sum} i=${i}")
  sum
})
```
结果：
```
sum=0 i=1
sum=0 i=2
sum=0 i=3
sum=0 i=4
```
该函数的功能是从左往右遍历右边操作类型List,而sum对应的是对应的左边的0，该函数要求返回值类型和左边类型一致，上面的例子中返回值是Int。
同理，两个Map的代码如下：
```scala
val map1 = Map("key1" -> 1, "key2" -> 3, "key3" -> 5)
val map2 = Map("key2" -> 4, "key3" -> 6, "key5" -> 10)
(map1 /: map2)((map, kv) => {
  println(s"map=${map} kv=${kv}")
  map
})
```
结果：
```
map=Map(key1 -> 1, key2 -> 3, key3 -> 5) kv=(key2,4)
map=Map(key1 -> 1, key2 -> 3, key3 -> 5) kv=(key3,6)
map=Map(key1 -> 1, key2 -> 3, key3 -> 5) kv=(key5,10)
```
从结果中可以看出左边map对应的是map1整体，而不是遍历map1
### 3.2 合并两个map：
```scala
val map1 = Map("key1" -> 1, "key2" -> 3, "key3" -> 5)
val map2 = Map("key2" -> 4, "key3" -> 6, "key5" -> 10)
val mapAdd2 = (map1 /: map2)((map, kv) => {
  map + (kv._1 -> (kv._2 + map.getOrElse(kv._1, 0)))
})
println(mapAdd2)
```
其中map.getOrElse(key,default):如果map中有这个key,则返回对应的value,否在返回default
结果：
```
Map(key1 -> 1, key2 -> 7, key3 -> 11, key5 -> 10)
```
### 4、用模式匹配
在网上查的有的用的模式匹配，我感觉在这里这样用就是多余~
附上代码：
```scala
val mapAdd2 = map1 ++ map2.map { case (key, value) => key -> (value + map1.getOrElse(key, 0)) }
println(mapAdd2)
val mapAdd3 = (map1 /: map2) {
  case (map, kv) => {
    map + (kv._1 -> (kv._2 + map.getOrElse(kv._1, 0)))
  }
}
println(mapAdd3)
val mapAdd4 = (map1 /: map2) {
  case (map, (k, v)) => {
    map + (k -> (v + map.getOrElse(k, 0)))
  }
}
println(mapAdd4)
```
结果是一样的：
```
Map(key1 -> 1, key2 -> 7, key3 -> 11, key5 -> 10)
Map(key1 -> 1, key2 -> 7, key3 -> 11, key5 -> 10)
Map(key1 -> 1, key2 -> 7, key3 -> 11, key5 -> 10)
```
## 参考
[https://www.cnblogs.com/tugeler/p/5134862.html](https://www.cnblogs.com/tugeler/p/5134862.html)




