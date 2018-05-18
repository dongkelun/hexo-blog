---
title: centos7 hive 单机模式安装配置
date: 2018-03-24
tags:
  - centos
  - hive
copyright: true
---
### 前言：
由于只是在自己的虚拟机上进行学习，所以对hive只是进行最简单的配置，其他复杂的配置文件没有配置。
## 1、前提
### 1.1 安装配置jdk1.8
### 1.2 安装hadoop2.x
hadoop单机模式安装见：[centos7 hadoop 单机模式安装配置](http://dongkelun.com/2018/03/23/hadoopConf/)
### 1.3 安装mysql并配置myql允许远程访问，我的mysql版本5.7.18。
mysql数据库安装过程请参考：[Centos 7.2 安装 Mysql 5.7.13](https://blog.csdn.net/lochy/article/details/51721319)
## 2、下载hive
下载地址：[http://mirror.bit.edu.cn/apache/hive/](http://mirror.bit.edu.cn/apache/hive/)，我下载的是apache-hive-2.3.2-bin.tar.gz。
```
wget http://mirror.bit.edu.cn/apache/hive/hive-2.3.2/apache-hive-2.3.2-bin.tar.gz
或者下载到本地，通过工具上传到虚拟机中
```
## 3、解压到/opt目录下（目录根据自己习惯）
```bash
tar -zxvf apache-hive-2.3.2-bin.tar.gz  -C /opt/
```
<!-- more -->
## 4、配置hive环境变量
```bash
vim /etc/profile
```
```bash
export HIVE_HOME=/opt/apache-hive-2.3.2-bin
export PATH=$PATH:$HIVE_HOME/bin  
```
```bash
source /etc/profile
```
## 5、配置hive
### 5.1 配置hive-site.xml 
其中ConnectionUserName和ConnectionPassword为mysql远程访问的用户名和密码，hive_metadata为mysql数据库，随自己习惯命名。
```bash
cd /opt/apache-hive-2.3.2-bin/conf/
vim hive-site.xml 
```
```bash
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://192.168.44.128:3306/hive_metadata?&amp;createDatabaseIfNotExist=true&amp;characterEncoding=UTF-8&amp;useSSL=false</value>
 </property>
<property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>root</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>Root-123456</value>
</property>
<property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
</property>
<property>
    <name>datanucleus.schema.autoCreateAll</name>
    <value>true</value> </property>
<property>
    <name>hive.metastore.schema.verification</name>
    <value>false</value>
 </property>
</configuration>

```
### 5.2 配置hive-site.xml 
```
cp hive-env.sh.template hive-env.sh
vim hive-env.sh
```
```
HADOOP_HOME=/opt/hadoop-2.7.5
export HIVE_CONF_DIR=/opt/apache-hive-2.3.2-bin/conf
```
具体位置如图：
![](http://wx2.sinaimg.cn/large/e44344dcly1fpoejy2vccj20m3047aa6.jpg)
## 6、加载mysql驱动（要与自己安装的mysql版本一致）
下载地址：[http://dev.mysql.com/downloads/connector/j/ ](http://dev.mysql.com/downloads/connector/j/)
我下载的是：mysql-connector-java-5.1.46.tar.gz，解压并将其中的mysql-connector-java-5.1.46-bin.jar放到hive/lib下
具体路径为：/opt/apache-hive-2.3.2-bin/lib
## 7、初始化数据库
```bash
schematool -initSchema -dbType mysql
```
## 8、启动hive
启动hive之前先启动hadoop,不然会报Connection refused异常，在命令行jps看一下hadoop是否启动成功然后启动hive
```bash
hive
```
然后简单的测试:
```bash
show databases;
```
出现如下图所示即代表配置成功！
![](http://wx2.sinaimg.cn/large/e44344dcly1fpoejygnraj20z206xwf5.jpg)
## 9、简单的hive语句测试
建表：
```
CREATE TABLE IF NOT EXISTS test (id INT,name STRING)ROW FORMAT DELIMITED FIELDS TERMINATED BY " " LINES TERMINATED BY "\n";
```
插入数据

```bash
insert into test values(1,'张三');
```
查询

```bash
select * from test;
```
![](http://wx1.sinaimg.cn/large/e44344dcly1fpoemkx2yxj20yu0dkdh6.jpg)
