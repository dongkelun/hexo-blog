---
title: spark ML之特征处理（1）
date: 2018-05-17
tags:
  - spark
  - ml
copyright: true
---
## 前言
最近在学习总结机器学习常用算法，在看spark机器学习决策树的官方示例时，发现用到了几个特征处理的类，之前没学习过，所以查了一下，感觉spark在特征处理方面的类还是挺多的，所以准备总结记录一下相关的用法，首先总结一下决策树中用到的几种。
## 1、VectorIndexer
根据源码注释，VectorIndexer是用于在“向量”的数据集中索引分类特征列的类（Class for indexing categorical feature columns in a dataset of `Vector`），这看起来不太好理解，直接看用法，举例说明就好了。
### 1.1 数据
我们用普通的数据格式即可：
data1.txt
```
1,-1.0 1.0
0,0.0 3.0
1,-1.0 5.0
0,0.0 1.0
```
其中第一列为label,后面的为features
spark读取数据程序（供参考）：
```scala
import spark.implicits._
val data_path = "files/ml/featureprocessing/data1.txt"
val data = spark.read.text(data_path).map {
  case Row(line: String) =>
    var arr = line.split(',')
    (arr(0), Vectors.dense(arr(1).split(' ').map(_.toDouble)))
}.toDF("label", "features")
data.show(false)
```
<!-- more -->
结果：
```
+-----+----------+
|label|features  |
+-----+----------+
|1    |[-1.0,1.0]|
|0    |[0.0,3.0] |
|1    |[-1.0,5.0]|
|0    |[0.0,1.0] |
+-----+----------+
```
### 1.2 代码
用法一：通过设置MaxCategories
```scala
import org.apache.spark.ml.feature.VectorIndexer
val indexer = new VectorIndexer()
  .setInputCol("features")
  .setOutputCol("indexed")
  .setMaxCategories(2)

val indexerModel = indexer.fit(data)

val categoricalFeatures: Set[Int] = indexerModel.categoryMaps.keys.toSet
println(s"Chose ${categoricalFeatures.size} categorical features: " +
  categoricalFeatures.mkString(", "))

val indexedData = indexerModel.transform(data)
indexedData.show(false)
```
结果
```
Chose 1 categorical features: 0
+-----+----------+---------+
|label|features  |indexed  |
+-----+----------+---------+
|1    |[-1.0,1.0]|[1.0,1.0]|
|0    |[0.0,3.0] |[0.0,3.0]|
|1    |[-1.0,5.0]|[1.0,5.0]|
|0    |[0.0,1.0] |[0.0,1.0]|
+-----+----------+---------+
```
可以看出选择了一个分类特征索引0，即第一个特征，设置MaxCategories为2，因为特征1只有两个值[-1.0,0.0]小于等于2所以会被索引，索引从零开始，而第二个特征有三个值[1.0,3.0,5.0],所以被认为是连续值，不会被索引，即保留原值。
将MaxCategories设为5，重新跑一下上面的代码
```
Chose 2 categorical features: 0, 1
+-----+----------+---------+
|label|features  |indexed  |
+-----+----------+---------+
|1    |[-1.0,1.0]|[1.0,0.0]|
|0    |[0.0,3.0] |[0.0,1.0]|
|1    |[-1.0,5.0]|[1.0,2.0]|
|0    |[0.0,1.0] |[0.0,0.0]|
+-----+----------+---------+
```
这次选出两个特征都进行索引，这是因为每个特征值的数量都小于5，可以看出，索引值从0开始，依次加1，对应的原始特征值也是按大小排序对应的。
## 2、StringIndexer
StringIndexer和上面的VectorIndexer类似，是将label列进行重新编号，为了便于理解，我们重新造一个数据，只看label就好了
### 2.1 数据
data2.txt
```
10,-1.0 1.0
10,0.0 3.0
7,-1.0 5.0
5,0.0 1.0
6,0.0 1.0
6,0.0 1.0
6,0.0 1.0
```
读数程序和上面的一样，我们直接看一下StringIndexer代码和效果吧~
### 2.2 代码
```scala
import org.apache.spark.ml.feature.StringIndexer
val labelIndexer = new StringIndexer()
  .setInputCol("label")
  .setOutputCol("indexedLabel")

val indexerModel = labelIndexer.fit(data)

val indexedData = indexerModel.transform(data)
indexedData.show(false)
```
结果
```
+-----+----------+------------+
|label|features  |indexedLabel|
+-----+----------+------------+
|10   |[-1.0,1.0]|1.0         |
|10   |[0.0,3.0] |1.0         |
|7    |[-1.0,5.0]|2.0         |
|5    |[0.0,1.0] |3.0         |
|6    |[0.0,1.0] |0.0         |
|6    |[0.0,1.0] |0.0         |
|6    |[0.0,1.0] |0.0         |
+-----+----------+------------+
```
从结果一眼可以看出，该类将label重新编号了，也是从零开始，但是不是按数值（因为StringIndexer从名字上就可以看出是将string类型的转出index）大小对应,是按label出现的频次排序的，出现频次最高索引为0，依次类推，如6出现了三次则索引后为0.0。
既然提到了是将string类型的进行索引，那么简单的造个数看下效果吧，我想实际分类中字符类型的label也很多吧，比如根据特征分类胖瘦等
名字还是data2.txt吧~
```
thin,-1.0 1.0
fat,0.0 3.0
fat,-1.0 5.0
thin,0.0 1.0
fat,0.0 1.0
```
将数据替换一下，重新跑一下上面的程序
```
+-----+----------+------------+
|label|features  |indexedLabel|
+-----+----------+------------+
|thin |[-1.0,1.0]|1.0         |
|fat  |[0.0,3.0] |0.0         |
|fat  |[-1.0,5.0]|0.0         |
|thin |[0.0,1.0] |1.0         |
|fat  |[0.0,1.0] |0.0         |
+-----+----------+------------+
```
### 2.3 注意
利用训练好的模型还转化新来的数据，可能会碰到异常，为了简单起见，还是用上面的thin,fat，假如新来的数据的label里有既不是thin也不是fat的咋办，比如middle（只是举例）或者异常数据。
data2new.txt
```
thin,-1.0 1.0
fat,0.0 3.0
middle,1.0 2.0
```
spark 提供了两种解决方法
```scala
val labelIndexer = new StringIndexer()
  .setInputCol("label")
	//.setHandleInvalid("error")
  .setHandleInvalid("skip")
  .setOutputCol("indexedLabel")
```
1、默认设置，也就是.setHandleInvalid("error")：会抛出异常  
```
Caused by: org.apache.spark.SparkException: Unseen label: middle
```
2、.setHandleInvalid("skip") 忽略这些label所在行的数据，正常运行，将输出如下结果：  
```
+-----+----------+------------+
|label|features  |indexedLabel|
+-----+----------+------------+
|thin |[-1.0,1.0]|1.0         |
|fat  |[0.0,3.0] |0.0         |
+-----+----------+------------+
```
稍微有点啰嗦，附下完整代码吧~
### 2.4 完整代码（供参考理解）
```scala
package com.dkl.leanring.spark.ml

import org.apache.spark.sql.SparkSession

object StringIndexerDemo {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession
      .builder
      .master("local")
      .getOrCreate()
    import spark.implicits._

    import org.apache.spark.ml.linalg.Vectors
    import org.apache.spark.sql.Row
    val data_path = "files/ml/featureprocessing/data2.txt"
    val data = spark.read.text(data_path).map {
      case Row(line: String) =>
        var arr = line.split(',')
        (arr(0), Vectors.dense(arr(1).split(' ').map(_.toDouble)))
    }.toDF("label", "features")

    import org.apache.spark.ml.feature.StringIndexer
    val labelIndexer = new StringIndexer()
      .setInputCol("label")
      //      .setHandleInvalid("error")
      .setHandleInvalid("skip")
      .setOutputCol("indexedLabel")

    val indexerModel = labelIndexer.fit(data)

    val indexedData = indexerModel.transform(data)
    indexedData.show(false)

    val data_new_path = "files/ml/featureprocessing/data2new.txt"
    val data_new = spark.read.text(data_new_path).map {
      case Row(line: String) =>
        var arr = line.split(',')
        (arr(0), Vectors.dense(arr(1).split(' ').map(_.toDouble)))
    }.toDF("label", "features")
    val indexedData_new = indexerModel.transform(data_new)
    indexedData_new.show(false)
  }
}
```
## 3、IndexToString
IndexToString 的作用是将StringIndexer转化的结果转回去，实际应用中大概是这样，将字符型的label转换为数值代入算法模型训练预测，这时候预测的结果是StringIndexer转化的数值型，为了便于直观展示，需要变换回去
### 3.1代码
接上面2.2的代码
```scala
import org.apache.spark.ml.feature.IndexToString
val labelConverter = new IndexToString()
  .setInputCol("indexedLabel")
  .setOutputCol("predictedLabel")
val predict = labelConverter.transform(indexedData)
predict.show(false)
```
### 3.2 结果
```
+-----+----------+------------+--------------+
|label|features  |indexedLabel|predictedLabel|
+-----+----------+------------+--------------+
|thin |[-1.0,1.0]|1.0         |thin          |
|fat  |[0.0,3.0] |0.0         |fat           |
|fat  |[-1.0,5.0]|0.0         |fat           |
|thin |[0.0,1.0] |1.0         |thin          |
|fat  |[0.0,1.0] |0.0         |fat           |
+-----+----------+------------+--------------+
```
根据结果可以看到该类又把indexedLabel转换为和原来的label一样的类型了
## 4、附录
当然在实际的算法中是要将预测后的数据进行转化的，所以为了便于理解，附上一个决策树分类的代码示例，因为这篇不是讲决策树的，所以这里不展开讲解了。
### 4.1 数据
data3.txt
```
thin,1.5 50
fat,1.5 60
thin,1.6 40
fat,1.6 60
thin,1.7 60
fat,1.7 80
thin,1.8 60
fat,1.8 90
thin,1.9 70
fat,1.9 80
```
其中第一个特征为身高，第二个特征为体重，数据来源于：[用Python开始机器学习（2：决策树分类算法）](https://blog.csdn.net/lsldd/article/details/41223147),（数据是作者主观臆断，具有一定逻辑性，但请无视其合理性）
### 4.2 代码及结果
该代码主要来自官方示例。
```scala
package com.dkl.leanring.spark.ml
import org.apache.spark.ml.Pipeline
import org.apache.spark.ml.classification.DecisionTreeClassificationModel
import org.apache.spark.ml.classification.DecisionTreeClassifier
import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator
import org.apache.spark.ml.feature.{ IndexToString, StringIndexer, VectorIndexer }
import org.apache.spark.sql.SparkSession
import org.apache.spark.ml.linalg.Vectors
import org.apache.spark.sql.Row

object DecisionTreeExample {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession
      .builder
      .master("local")
      .getOrCreate()
    import spark.implicits._
    val data_path = "files/ml/featureprocessing/data3.txt"
    val data = spark.read.text(data_path).map {
      case Row(line: String) =>
        var arr = line.split(',')
        (arr(0), Vectors.dense(arr(1).split(' ').map(_.toDouble)))
    }.toDF("label", "features")

    // Index labels, adding metadata to the label column.
    // Fit on whole dataset to include all labels in index.
    val labelIndexer = new StringIndexer()
      .setInputCol("label")
      .setOutputCol("indexedLabel")
      .fit(data)
    // Automatically identify categorical features, and index them.
    val featureIndexer = new VectorIndexer()
      .setInputCol("features")
      .setOutputCol("indexedFeatures")
      .setMaxCategories(4) // features with > 4 distinct values are treated as continuous.
      .fit(data)

    // Split the data into training and test sets (30% held out for testing).
    val Array(trainingData, testData) = data.randomSplit(Array(0.7, 0.3))

    // Train a DecisionTree model.
    val dt = new DecisionTreeClassifier()
      .setLabelCol("indexedLabel")
      .setFeaturesCol("indexedFeatures")

    // Convert indexed labels back to original labels.
    val labelConverter = new IndexToString()
      .setInputCol("prediction")
      .setOutputCol("predictedLabel")
      .setLabels(labelIndexer.labels)

    // Chain indexers and tree in a Pipeline.
    val pipeline = new Pipeline()
      .setStages(Array(labelIndexer, featureIndexer, dt, labelConverter))

    // Train model. This also runs the indexers.
    val model = pipeline.fit(trainingData)

    // Make predictions.
    val predictions = model.transform(testData)

    // Select example rows to display.
    predictions.select("predictedLabel", "label", "features").show(5)

    // Select (prediction, true label) and compute test error.
    val evaluator = new MulticlassClassificationEvaluator()
      .setLabelCol("indexedLabel")
      .setPredictionCol("prediction")
      .setMetricName("accuracy")
    val accuracy = evaluator.evaluate(predictions)
    println("Test Error = " + (1.0 - accuracy))

    val treeModel = model.stages(2).asInstanceOf[DecisionTreeClassificationModel]
    println("Learned classification tree model:\n" + treeModel.toDebugString)
  }
}
```
结果：
因为训练数据和测试数据是随机分的，所以每次跑的结果不太一样。
```
+--------------+-----+----------+
|predictedLabel|label|  features|
+--------------+-----+----------+
|           fat|  fat|[1.5,60.0]|
|           fat|  fat|[1.9,80.0]|
|           fat| thin|[1.5,50.0]|
+--------------+-----+----------+

Test Error = 0.33333333333333337
Learned classification tree model:
DecisionTreeClassificationModel (uid=dtc_4f851a4dd870) of depth 3 with 7 nodes
  If (feature 1 <= 70.0)
   If (feature 0 <= 1.6)
    If (feature 1 <= 40.0)
     Predict: 0.0
    Else (feature 1 > 40.0)
     Predict: 1.0
   Else (feature 0 > 1.6)
    Predict: 0.0
  Else (feature 1 > 70.0)
   Predict: 1.0
```

## 参考资料
[https://blog.csdn.net/shenxiaoming77/article/details/63715525](https://blog.csdn.net/shenxiaoming77/article/details/63715525)

