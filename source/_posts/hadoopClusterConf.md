---

title: centos7 hadoop 集群安装配置
date: 2018-04-05
tags:
  - centos
  - hadoop
copyright: true
---
### 前言：
本文安装配置的hadoop为分布式的集群，单机配置见：[centos7 hadoop 单机模式安装配置](http://dongkelun.com/2018/03/23/hadoopConf/)
我用的三个centos7, 先将常用环境配置好（[CentOS 初始环境配置](http://dongkelun.com/2018/04/05/centosInitialConf/)），设置的ip分别为：192.168.44.138、192.168.44.139，192.168.44.140，分别对应别名master、slave1、slave2
## 1、首先安装配置jdk（我安装的1.8）
## 2、给每个虚拟机的ip起个别名
在每个虚拟机上执行
```
vim /etc/hosts 
```
在最下面添加：
```
192.168.44.138 master
192.168.44.139 slave1
192.168.44.140 slave2
```
在每个虚拟机上ping一下，保证都能ping通
```
ping master
ping slave1
ping slave2
```
## 3、SSH免密码登录
保证三台机器都可以免密互通，参考：[linux ssh 免密登录](http://dongkelun.com/2018/04/05/sshConf/)
## 3、下载hadoop（每台机器）

下载地址：[http://mirror.bit.edu.cn/apache/hadoop/common/](http://mirror.bit.edu.cn/apache/hadoop/common/)，我下载的是hadoop-2.7.5.tar.gz
## 4、解压到/opt目录下（每台机器、目录根据自己习惯）
```bash
tar -zxvf hadoop-2.7.5.tar.gz  -C /opt/
```
<!-- more -->
## 5、配置hadoop环境变量（每台机器）
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
## 6、配置hadoop（仅master）
配置文件里的文件路径和端口随自己习惯配置
### 6.1 配置slaves
需要现将slaves1文件中的localhost删掉，本次使用两个slave节点，让master仅作为NameNode使用，也可以让master既作为NameNode也作为 DataNode，在slaves添加master即可
```
vim /opt/hadoop-2.7.5/etc/hadoop/slaves 
```
```
slave1
slave2
```


### 6.2 配置hadoop-env.sh 
```bash
vim /opt/hadoop-2.7.5/etc/hadoop/hadoop-env.sh
```
找到# The java implementation to use.将其下面的一行改为：
```bash
export JAVA_HOME=/opt/jdk1.8.0_45
```
### 6.3 配置core-site.xml 

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
        <value>hdfs://master:8888</value>
    </property>
</configuration>

```

### 6.4 配置hdfs-site.xml
```
vim /opt/hadoop-2.7.5/etc/hadoop/hdfs-site.xml
```
dfs.replication 一般设为 3，但这次只使用两个slave，所以 dfs.replication 的值设为 2
```bash
<configuration>
	<property>
	    <name>dfs.namenode.secondary.http-address</name>
	    <value>master:50090</value>
	</property>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
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

### 6.5 配置yarn-site.xml
```bash
vim /opt/hadoop-2.7.5/etc/hadoop/yarn-site.xml 
```

```bash
<configuration>

<!-- Site specific YARN configuration properties -->
	<property>
		<name>yarn.resourcemanager.hostname</name>
		<value>master</value>
	</property>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
</configuration>
```

### 6.6 配置mapred-site.xml
```bash
cd /opt/hadoop-2.7.5/etc/hadoop/
cp mapred-site.xml.template mapred-site.xml
vim mapred-site.xml
```
```bash
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```
### 6.7 将上述配置的文件传到其他节点的/opt/hadoop-2.7.5/etc/hadoop/目录中
```bash
scp -r slaves hadoop-env.sh core-site.xml  hdfs-site.xml yarn-site.xml hdfs-site.xml root@slave1:/opt/hadoop-2.7.5/etc/hadoop/
scp -r slaves hadoop-env.sh core-site.xml  hdfs-site.xml yarn-site.xml hdfs-site.xml root@slave2:/opt/hadoop-2.7.5/etc/hadoop/
```
![](http://wx2.sinaimg.cn/large/e44344dcly1fq263swsiaj20x506m0ta.jpg)
## 7、启动与停止（仅master）
### 7.1 hdfs启动与停止
第一次启动hdfs需要先格式化：
```bash
cd /opt/hadoop-2.7.5
./bin/hdfs namenode -format  
```
启动：
```bash
./sbin/start-dfs.sh
```
停止：
```bash
./sbin/stop-dfs.sh
```
![](http://wx3.sinaimg.cn/large/e44344dcly1fq263te315j20ms034t8u.jpg)

验证，浏览器输入：http://192.168.44.138:50070

![](http://wx1.sinaimg.cn/large/e44344dcly1fq263tw8s6j21aw0s4tb1.jpg)

简单的验证hadoop命令：
```bash
hadoop fs -mkdir /test
```
在浏览器查看，出现如下图所示，即为成功
![](http://wx1.sinaimg.cn/large/e44344dcly1fpnoev27ouj217a0didgl.jpg)
### 7.2 yarn启动与停止
启动：
```bash
cd /opt/hadoop-2.7.5
./sbin/start-yarn.sh  
```
```
./sbin/stop-yarn.sh 
```
浏览器查看：http://192.168.44.138:8088
![](http://wx1.sinaimg.cn/large/e44344dcly1fq263v33xrj21gr0e540x.jpg)
jps查看进程
master:
![](http://wx3.sinaimg.cn/large/e44344dcly1fq263vg3qnj20g602a3yd.jpg)
slave1:
![](http://wx2.sinaimg.cn/large/e44344dcly1fq263wn8q9j20ld01v744.jpg)
slave2:
![](http://wx4.sinaimg.cn/large/e44344dcly1fq263w3pe3j20hr01tjr7.jpg)

若各节点的进程均如图所示，那么hadoop集群就配置成功！


## 参考资料
[http://www.powerxing.com/install-hadoop-cluster/](http://www.powerxing.com/install-hadoop-cluster/)