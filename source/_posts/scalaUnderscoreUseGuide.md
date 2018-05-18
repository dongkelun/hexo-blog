---
title: scala 下划线使用指南
date: 2018-03-29
tags:
  - 转载
  - scala
---

原文地址：[https://my.oschina.net/joymufeng/blog/863823](https://my.oschina.net/joymufeng/blog/863823)  &emsp;&emsp;作者：[joymufeng](https://my.oschina.net/joymufeng/home) 
下划线这个符号几乎贯穿了任何一本Scala编程书籍，并且在不同的场景下具有不同的含义，绕晕了不少初学者。正因如此，下划线这个特殊符号无形中增加Scala的入门难度。本文希望帮助初学者踏平这个小山坡。
## 1、用于替换Java的等价语法
由于大部分的Java关键字在Scala中拥有了新的含义，所以一些基本的语法在Scala中稍有变化。
### 1.1 导入通配符
*在Scala中是合法的方法名，所以导入包时要使用_代替。
```scala
//Java
import java.util.*;

//Scala
import java.util._
```
<!-- more -->
### 1.2 类成员默认值
Java中类成员可以不赋初始值，编译器会自动帮你设置一个合适的初始值：
```scala
class Foo{
     //String类型的默认值为null
     String s;
}
```
而在Scala中必须要显式指定，如果你比较懒，可以用_让编译器自动帮你设置初始值：
```scala
class Foo{
    //String类型的默认值为null
    var s: String = _
}
```
该语法只适用于类成员，而不适用于局部变量。
### 1.3 可变参数
Java声明可变参数如下：
```java
public static void printArgs(String ... args){
    for(Object elem: args){
        System.out.println(elem + " ");
    }
}
```
调用方法如下：
```java
 //传入两个参数
printArgs("a", "b");
//也可以传入一个数组
printArgs(new String[]{"a", "b"});
```
在Java中可以直接将数组传给printArgs方法，但是在Scala中，你必须要明确的告诉编译器，你是想将集合作为一个独立的参数传进去，还是想将集合的元素传进去。如果是后者则要借助下划线：
```scala
printArgs(List("a", "b"): _*)
```
### 1.4 类型通配符
Java的泛型系统有一个通配符类型，例如List<?>，任意的List<T>类型都是List<?>的子类型，如果我们想编写一个可以打印所有List类型元素的方法，可以如下声明：
```java
public static void printList(List<?> list){
    for(Object elem: list){
        System.out.println(elem + " ");
    }
}
```
对应的Scala版本为：
```scala
def printList(list: List[_]): Unit ={
   list.foreach(elem => println(elem + " "))
}
```
## 2、模式匹配
### 2.1 默认匹配
```scala
str match{
    case "1" => println("match 1")
    case _   => println("match default")
}
```
### 2.2 匹配集合元素
```scala
//匹配以0开头，长度为三的列表
expr match {
  case List(0, _, _) => println("found it")
  case _ =>
}

//匹配以0开头，长度任意的列表
expr match {
  case List(0, _*) => println("found it")
  case _ =>
}

//匹配元组元素
expr match {
  case (0, _) => println("found it")
  case _ =>
}

//将首元素赋值给head变量
val List(head, _*) = List("a")
```
## 3、 Scala特有语法
### 类型通配符
```scala
public static void printList(List<?> list){
    for(Object elem: list){
        System.out.println(elem + " ");
    }
}
```
### 3.1 访问Tuple元素
```scala
val t = (1, 2, 3)
println(t._1, t._2, t._3)
```
### 3.2 简写函数字面量（function literal）
如果函数的参数在函数体内只出现一次，则可以使用下划线代替：
```scala
val f1 = (_: Int) + (_: Int)
//等价于
val f2 = (x: Int, y: Int) => x + y

list.foreach(println(_))
//等价于
list.foreach(e => println(e))

list.filter(_ > 0)
//等价于
list.filter(x => x > 0)
```
### 3.3 定义一元操作符
在Scala中，操作符其实就是方法，例如1 + 1等价于1.+(1)，利用下划线我们可以定义自己的左置操作符，例如Scala中的负数就是用左置操作符实现的
```scala
-2
//等价于
2.unary_-
```
### 3.4 定义赋值操作符
我们通过下划线实现赋值操作符，从而可以精确地控制赋值过程：
```scala
class Foo {
  def name = { "foo" }
  def name_=(str: String) {
    println("set name " + str)
  }
}
val m = new Foo()
m.name = "Foo" //等价于: m.name_=("Foo")
```
### 3.5 定义部分应用函数（partially applied function）
我们可以为某个函数只提供部分参数进行调用，返回的结果是一个新的函数，即部分应用函数。因为只提供了部分参数，所以部分应用函数也因此而得名。
```scala
def sum(a: Int, b: Int, c: Int) = a + b + c
val b = sum(1, _: Int, 3)
b: Int => Int = <function1>
b(2) //6
```
### 3.6 将方法转换成函数
Scala中方法和函数是两个不同的概念，方法无法作为参数进行传递，也无法赋值给变量，但是函数是可以的。在Scala中，利用下划线可以将方法转换成函数：
```scala
//将println方法转换成函数，并赋值给p
val p = println _  
//p: (Any) => Unit
```

## 4、小结
下划线在大部分的应用场景中是以语法糖的形式出现的，可以减少击键次数，并且代码显得更加简洁。但是对于不熟悉下划线的同学阅读起来稍显困难，希望通过本文能够帮你解决这个的困惑。
