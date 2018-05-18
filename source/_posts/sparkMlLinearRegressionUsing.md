---
title: spark ML算法之线性回归使用
date: 2018-04-09
tags:
  - spark
  - ml
  - 算法
copyright: true
---
## 前言
本文是讲如何使用spark ml进行线性回归，不涉及线性回归的原理。
## 1、数据格式
### 1.1 普通标签格式
#### 1.1.1 格式为：
```bash
标签,特征值1 特征值2 特征值3...
```
```
1,1.9
2,3.1
3,4
3.5,4.45
4,5.02
9,9.97
-2,-0.98
```
#### 1.1.2 spark 读取
1、Rdd 
旧版（mllib）的线性回归要求传入的参数类型为RDD[LabeledPoint]
<!-- more -->
```scala
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LabeledPoint

val data_path = "files/ml/linear_regression_data1.txt"
val data = sc.textFile(data_path)
val training = data.map { line =>
  val arr = line.split(',')
  LabeledPoint(arr(0).toDouble, Vectors.dense(arr(1).split(' ').map(_.toDouble)))
}.cache()
training.foreach(println)
```
结果：
```
(1.0,[1.9])
(2.0,[3.1])
(3.0,[4.0])
(3.5,[4.45])
(4.0,[5.02])
(9.0,[9.97])
(-2.0,[-0.98])
```
一共有两列，第一列可以通过.label获得（类型为Double），第二列可以通过.features获得（类型为Vector[Double]）
2、 DataFrame
新版（ml）的线性回归要求传入的参数类型为Dataset[_]
```scala
import org.apache.spark.ml.linalg.Vectors
import org.apache.spark.sql.Row
import spark.implicits._
val data_path = "files/ml/linear_regression_data1.txt"
val data = spark.read.text(data_path)
val training = data.map {
  case Row(line: String) =>
    var arr = line.split(',')
    (arr(0).toDouble, Vectors.dense(arr(1).split(' ').map(_.toDouble)))
}.toDF("label", "features")
training.show()
```
结果：
```
+-----+--------+
|label|features|
+-----+--------+
|  1.0|   [1.9]|
|  2.0|   [3.1]|
|  3.0|   [4.0]|
|  3.5|  [4.45]|
|  4.0|  [5.02]|
|  9.0|  [9.97]|
| -2.0| [-0.98]|
+-----+--------+
```
其中列名"label", "features"固定，不能改为其他列名。
### 1.2 LIBSVM格式
#### 1.2.1 格式为：
```bash
label index1:value1 index2:value2 ...
```
其中每一行的index必须为升序
为了便于理解，造几条多维数据：
```
1 1:1.9 2:2 4:2 100:3 101:6
2 1:3.1 2:2 4:2 100:3 101:6
3 1:4 2:2 4:2 100:3 101:6
3.5 1:4.45 2:2 4:2 100:3 101:6
4 1:5.02 2:2 4:2 100:3 101:6
9 1:9.97 4:2 100:3 101:6
-2 1:-0.98 2:2 4:2 100:3 201:6
```
#### 1.2.2 spark 读取
1、Rdd 
```scala
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.mllib.util.MLUtils
val data_path = "files/ml/linear_regression_data2.txt"
val training = MLUtils.loadLibSVMFile(sc, data_path)
training.foreach(println)

```
结果：
```
(1.0,(201,[0,1,3,99,100],[1.9,2.0,2.0,3.0,6.0]))
(2.0,(201,[0,1,3,99,100],[3.1,2.0,2.0,3.0,6.0]))
(3.0,(201,[0,1,3,99,100],[4.0,2.0,2.0,3.0,6.0]))
(3.5,(201,[0,1,3,99,100],[4.45,2.0,2.0,3.0,6.0]))
(4.0,(201,[0,1,3,99,100],[5.02,2.0,2.0,3.0,6.0]))
(9.0,(201,[0,3,99,100],[9.97,2.0,3.0,6.0]))
(-2.0,(201,[0,1,3,99,200],[-0.98,2.0,2.0,3.0,6.0]))
```
返回类型为RDD[LabeledPoint]，其中第一列为label,第二列vector的第一个值为max(index),第二个index-1组成的数组，第三个为value组成的数组。
2、DataFrame
```scala
val data_path = "files/ml/linear_regression_data2.txt"
val data = spark.read.text(data_path)
val training = spark.read.format("libsvm").load(data_path)
training.show(false)
```
结果：
```
+-----+--------------------------------------------+
|label|features                                    |
+-----+--------------------------------------------+
|1.0  |(201,[0,1,3,99,100],[1.9,2.0,2.0,3.0,6.0])  |
|2.0  |(201,[0,1,3,99,100],[3.1,2.0,2.0,3.0,6.0])  |
|3.0  |(201,[0,1,3,99,100],[4.0,2.0,2.0,3.0,6.0])  |
|3.5  |(201,[0,1,3,99,100],[4.45,2.0,2.0,3.0,6.0]) |
|4.0  |(201,[0,1,3,99,100],[5.02,2.0,2.0,3.0,6.0]) |
|9.0  |(201,[0,3,99,100],[9.97,2.0,3.0,6.0])       |
|-2.0 |(201,[0,1,3,99,200],[-0.98,2.0,2.0,3.0,6.0])|
+-----+--------------------------------------------+
```
## 2、线性回归代码
### 2.1 数据
用libsvm格式的数据：
```
1 1:1.9
2 1:3.1
3 1:4
3.5 1:4.45
4 1:5.02
9 1:9.97
-2 1:-0.98
```
### 2.2 旧版代码
```scala
package com.dkl.leanring.spark.ml

import org.apache.log4j.{ Level, Logger }
import org.apache.spark.{ SparkConf, SparkContext }
import org.apache.spark.mllib.regression.LinearRegressionWithSGD
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LinearRegressionModel

object OldLinearRegression {

  def main(args: Array[String]) {
    // 构建Spark对象
    val conf = new SparkConf().setAppName("OldLinearRegression").setMaster("local")
    val sc = new SparkContext(conf)
    Logger.getRootLogger.setLevel(Level.WARN)

    //读取样本数据
    val data_path = "files/ml/linear_regression_data3.txt"
    val training = MLUtils.loadLibSVMFile(sc, data_path)
    val numTraing = training.count()

    // 新建线性回归模型，并设置训练参数
    val numIterations = 10000
    val stepSize = 0.5
    val miniBatchFraction = 1.0

    //书上的代码 intercept 永远为0
    //val model = LinearRegressionWithSGD.train(examples, numIterations, stepSize, miniBatchFraction)
    var lr = new LinearRegressionWithSGD().setIntercept(true)
    lr.optimizer.setNumIterations(numIterations).setStepSize(stepSize).setMiniBatchFraction(miniBatchFraction)
    val model = lr.run(training)
    println(model.weights)
    println(model.intercept)

    // 对样本进行测试
    val prediction = model.predict(training.map(_.features))
    val predictionAndLabel = prediction.zip(training.map(_.label))
    val print_predict = predictionAndLabel.take(20)
    println("prediction" + "\t" + "label")
    for (i <- 0 to print_predict.length - 1) {
      println(print_predict(i)._1 + "\t" + print_predict(i)._2)
    }
    // 计算测试误差
    val loss = predictionAndLabel.map {
      case (p, l) =>
        val err = p - l
        err * err
    }.reduce(_ + _)
    val rmse = math.sqrt(loss / numTraing)
    println(s"Test RMSE = $rmse.")

  }

}
```
其中注释的第30行代码为书上的写法，但这样写intercept一直为0，也就是只适用于y=a*x的形式，不适用于y=ax+b,改为31、32替代即可。

结果：
```
[0.992894785953067]
-0.9446037936869749
prediction	label
0.9418962996238525	1.0
2.133370042767533	2.0
3.0269753501252934	3.0
3.473778003804174	3.5
4.039728031797421	4.0
8.954557222265104	9.0
-1.9176406839209805	-2.0
Test RMSE = 0.06866615969192089.
```
即a=0.992894785953067,b=-0.9446037936869749,y=0.992894785953067*x-0.9446037936869749
### 2.2 新版代码
``` scala
package com.dkl.leanring.spark.ml

import org.apache.spark.ml.regression.LinearRegression
import org.apache.spark.sql.SparkSession

object NewLinearRegression {

  def main(args: Array[String]): Unit = {
    val spark = SparkSession
      .builder
      .appName("NewLinearRegression")
      .master("local")
      .getOrCreate()
    val data_path = "files/ml/linear_regression_data3.txt"

    import spark.implicits._
    import org.apache.spark.ml.linalg.Vectors
    import org.apache.spark.sql.Row
    val training = spark.read.format("libsvm").load(data_path)

    val lr = new LinearRegression()
      .setMaxIter(10000)
      .setRegParam(0.3)
      .setElasticNetParam(0.8)

    val lrModel = lr.fit(training)

    println(s"Coefficients: ${lrModel.coefficients} Intercept: ${lrModel.intercept}")

    val trainingSummary = lrModel.summary
    println(s"numIterations: ${trainingSummary.totalIterations}")
    println(s"objectiveHistory: [${trainingSummary.objectiveHistory.mkString(",")}]")
    trainingSummary.residuals.show()
    println(s"RMSE: ${trainingSummary.rootMeanSquaredError}")
    println(s"r2: ${trainingSummary.r2}")
    trainingSummary.predictions.show()

    spark.stop()
  }
}
```
结果：
```
Coefficients: [0.9072296333951224] Intercept: -0.630360819004294
numIterations: 3
objectiveHistory: [0.5,0.41543560544030766,0.08269406021049913]
+--------------------+
|           residuals|
+--------------------+
| -0.0933754844464385|
|-0.18205104452058585|
|0.001442285423804...|
| 0.09318895039599973|
| 0.07606805936077965|
|  0.5852813740549223|
| -0.4805541402684861|
+--------------------+

RMSE: 0.2999573166705823
r2: 0.9906296595124621
+-----+---------------+------------------+
|label|       features|        prediction|
+-----+---------------+------------------+
|  1.0|  (1,[0],[1.9])|1.0933754844464385|
|  2.0|  (1,[0],[3.1])| 2.182051044520586|
|  3.0|  (1,[0],[4.0])|2.9985577145761955|
|  3.5| (1,[0],[4.45])|3.4068110496040003|
|  4.0| (1,[0],[5.02])|3.9239319406392204|
|  9.0| (1,[0],[9.97])| 8.414718625945078|
| -2.0|(1,[0],[-0.98])|-1.519445859731514|
+-----+---------------+------------------+
```



