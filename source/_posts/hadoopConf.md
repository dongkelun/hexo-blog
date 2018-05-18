---
title: centos7 hadoop 单机模式安装配置
date: 2018-03-23
tags:
  - centos
  - hadoop
copyright: true
---
## 前言
由于现在要用spark,而学习spark会和hdfs和hive打交道，之前在公司服务器配的分布式集群，离开公司之后，自己就不能用了，后来用ambari搭的三台虚拟机的集群太卡了，所以就上网查了一下hadoop+hive的单机部署，以便自己能进行简单的学习，这里记录一下，本来想把hadoop和hive的放在一起写，由于太多，就分成两篇写了。
## 1、首先安装配置jdk（我安装的1.8）
## 2、下载hadoop
下载地址：[http://mirror.bit.edu.cn/apache/hadoop/common/](http://mirror.bit.edu.cn/apache/hadoop/common/)，我下载的是hadoop-2.7.5.tar.gz
（由于我之前用的2.7.1是几年前下载保存在本地的，现在发现之前在配置spark那篇写的那个hadoop下载地址较慢，所以改成这个地址）
## 3、解压到/opt目录下（目录根据自己习惯）
```bash
tar -zxvf hadoop-2.7.5.tar.gz  -C /opt/
```
<!-- more -->
## 4、配置hadoop环境变量
```bash
vim /etc/profile
```
```bash
export HADOOP_HOME=/opt/hadoop-2.7.5
export PATH=$PATH:$HADOOP_HOME/bin  
```
```bash
source /etc/profile
```
## 5、配置hadoop


### 5.1 配置hadoop-env.sh 
```bash
vim /opt/hadoop-2.7.5/etc/hadoop/hadoop-env.sh
```
找到# The java implementation to use.将其下面的一行改为：
```bash
export JAVA_HOME=/opt/jdk1.8.0_45
```
### 5.2 配置core-site.xml (5.2和5.3中配置文件里的文件路径和端口随自己习惯配置)
其中的IP:192.168.44.128为虚拟机ip,不能设置为localhost，如果用localhost,后面在windows上用saprk连接服务器（虚拟机）上的hive会报异常（win读取的配置也是localhost，这样localhost就为win本地ip了~也可以给ip加个映射，不过因为单机的我就没加）。
```
vim /opt/hadoop-2.7.5/etc/hadoop/core-site.xml
```
```
<configuration>
<property>
        <name>hadoop.tmp.dir</name>
        <value>file:///opt/hadoop-2.7.5</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://192.168.44.128:8888</value>
    </property>
</configuration>

```

### 5.3 配置hdfs-site.xml
```
vim /opt/hadoop-2.7.5/etc/hadoop/hdfs-site.xml
```
```bash
<configuration>
        <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///opt/hadoop-2.7.5/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///opt/hadoop-2.7.5/tmp/dfs/data</value>
    </property>
</configuration>

```
## 6、SSH免密码登录
参考：[linux ssh 免密登录](http://dongkelun.com/2018/04/05/sshConf/)
## 7、启动与停止
第一次启动hdfs需要格式化：
```bash
cd /opt/hadoop-2.7.5
./bin/hdfs namenode -format  
```
Re-format filesystem in Storage Directory /opt/hadoop-2.7.5/tmp/dfs/name ? (Y or N) 
输入：Y
（出现询问输入Y or N,全部输Y即可）
启动：
```bash
./sbin/start-dfs.sh
```
停止：
```bash
./sbin/stop-dfs.sh
```
![](http://wx2.sinaimg.cn/large/e44344dcly1fpnngyi59gj20p502idfw.jpg)

验证，浏览器输入：http://192.168.44.128:50070

![](http://wx3.sinaimg.cn/large/e44344dcly1fpnoeul7usj21110lz75w.jpg)

简单的验证hadoop命令：
```bash
hadoop fs -mkdir /test
```
在浏览器查看，出现如下图所示，即为成功
![](http://wx1.sinaimg.cn/large/e44344dcly1fpnoev27ouj217a0didgl.jpg)

## 8、配置yarn

### 8.1 配置mapred-site.xml
```
cd /opt/hadoop-2.7.5/etc/hadoop/
cp mapred-site.xml.template mapred-site.xml
vim mapred-site.xml
```
```bash
<configuration>
    <!-- 通知框架MR使用YARN -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
### 8.2 配置yarn-site.xml
```bash
vim yarn-site.xml
```
```bash
<configuration>
    <!-- reducer取数据的方式是mapreduce_shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
### 8.3 yarn启动与停止
启动：
```bash
cd /opt/hadoop-2.7.5
./sbin/start-yarn.sh  
```
```
./sbin/stop-yarn.sh 
```
浏览器查看：http://192.168.44.128:8088
![](http://wx4.sinaimg.cn/large/e44344dcly1fpnoewkmuaj21gw0dmwgw.jpg)
jps查看进程
![](http://wx2.sinaimg.cn/large/e44344dcly1fpnoevud3cj20d303ea9y.jpg)
到此，hadoop单机模式就配置成功了！

## 参考资料
[https://blog.csdn.net/cafebar123/article/details/73500014](https://blog.csdn.net/cafebar123/article/details/73500014)














