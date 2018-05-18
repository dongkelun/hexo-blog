---
title: Redis Cluster 安装配置
date: 2018-01-09
tags:
  - redis
copyright: true
---


### 服务器
CentOS
Centos 服务器初始环境配置最好先配置好，服务器时间最好配置为一致
我用的是6个服务器，一个服务器一个端口，便于配置文件的修改

### 首先下载redis3 到本地(需要3以后的版本，我下载的最新的版本:3.0.4)
``` 
wget http://download.redis.io/releases/redis-3.0.4.tar.gz
```
<!-- more -->
### 分别上传到每个服务器
``` 
scp -r redis-3.0.4.tar.gz redis@redis1:~/
```

### 分别在每个服务器上安装 gcc tcl ruby rubygems gem_redis
``` 
sudo yum -y install gcc
sudo yum -y install tcl
sudo yum -y install ruby rubygems
sudo gem install redis --version 3.0.4
```
(gem_redis如果装不上，可多试几次，若是ubuntu，换源，换成taobao,建议服务器用centOS)

### 解压安装redis
``` 
tar -zxvf redis-3.0.4.tar.gz
cd redis-3.0.4/
sudo make test
sudo make install
```
### 修改配置文件redis.conf 以端口9000为例
``` 
sudo vim redis.conf

port 9000
pidfile /var/run/redis-9000.pid
dbfilename dump-9000.rdb
appendfilename "appendonly-9000.aof"
cluster-config-file nodes-9000.conf
cluster-enabled yes
cluster-node-timeout 5000
appendonly yes
```


### 然后分别在每个服务器上启动redis
``` 
src/redis-server redis.conf
```
### 查看每个端口是否成功开启
```  
ps -aux | grep redis
```
结果如下：
```  
redis 26674 1 0 Sep15 ? 00:01:21 src/redis-server *:9000 [cluster]
```

如果后面有[cluster] 证明开启成功

### 然后建立集群
``` 
src/redis-trib.rb create --replicas 1 192.168.32.195:9000 192.168.32.196:9000
192.168.32.197:9000 192.168.32.198:9000 192.168.32.199:9000 192.168.6.214:9000
```
(master 至少有三个 1 代表 一个master 对应一个slave)

### 如果下面这样就代表成功(端口 9000 )
``` 
>>> Creating cluster
Connecting to node 192.168.32.195:9000: OK
Connecting to node 192.168.32.196:9000: OK
Connecting to node 192.168.32.197:9000: OK
Connecting to node 192.168.32.198:9000: OK
Connecting to node 192.168.32.199:9000: OK
Connecting to node 192.168.6.214:9000: OK
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.32.199:9000
192.168.32.198:9000
192.168.32.197:9000
Adding replica 192.168.32.196:9000 to 192.168.32.199:9000
Adding replica 192.168.32.195:9000 to 192.168.32.198:9000
Adding replica 192.168.6.214:9000 to 192.168.32.197:9000
S: f4c625f49b6c1b073a379c41073b9ff9786f45f6 192.168.32.195:9000
replicates 20ecac926dba10a056f4d36a72bed35bbb7fcd57
S: 77e68bf6eb86d8bf6cb993b32d726e2cd4d05a54 192.168.32.196:9000
replicates 3276d09525fff0195dd366c8d05f80773d7d9af3
M: c6f4504b2ceb8bcffb682483168b8016e6f5973e 192.168.32.197:9000
slots:10923-16383 (5461 slots) master
M: 20ecac926dba10a056f4d36a72bed35bbb7fcd57 192.168.32.198:9000
slots:5461-10922 (5462 slots) master
M: 3276d09525fff0195dd366c8d05f80773d7d9af3 192.168.32.199:9000
slots:0-5460 (5461 slots) master
S: 627c0ae61d75570dc15a03cbdb03b1a8abd56f58 192.168.6.214:9000
replicates c6f4504b2ceb8bcffb682483168b8016e6f5973e
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join...
>>> Performing Cluster Check (using node 192.168.32.195:9000)
M: f4c625f49b6c1b073a379c41073b9ff9786f45f6 192.168.32.195:9000
slots: (0 slots) master
replicates 20ecac926dba10a056f4d36a72bed35bbb7fcd57
M: 77e68bf6eb86d8bf6cb993b32d726e2cd4d05a54 192.168.32.196:9000
slots: (0 slots) master
replicates 3276d09525fff0195dd366c8d05f80773d7d9af3
M: c6f4504b2ceb8bcffb682483168b8016e6f5973e 192.168.32.197:9000
slots:10923-16383 (5461 slots) master
M: 20ecac926dba10a056f4d36a72bed35bbb7fcd57 192.168.32.198:9000
slots:5461-10922 (5462 slots) master
M: 3276d09525fff0195dd366c8d05f80773d7d9af3 192.168.32.199:9000
slots:0-5460 (5461 slots) master
M: 627c0ae61d75570dc15a03cbdb03b1a8abd56f58 192.168.6.214:9000
slots: (0 slots) master
replicates c6f4504b2ceb8bcffb682483168b8016e6f5973e
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
### 以集群的方式登陆(9000为例）
```  
redis-cli -c -p 9000
```
### 查看 主从关系
``` 
$ 127.0.0.1:9000> cluster nodes
baa1779a73a8d329dfc96104ac3116a24b16a681 192.168.32.199:9000 myself,master - 0 0
5 connected 0-5460
140a8c76dfd44a6e43045d43a4d853fa330dfd42 192.168.32.198:9000 master - 0
1442403482170 12 connected 5461-10922
c99974ae9a92ed18166d580e4d7bb438a26ad3a2 192.168.32.195:9000 slave
140a8c76dfd44a6e43045d43a4d853fa330dfd42 0 1442403481669 12 connected
e07467f74f81f31c024bb5626c37e550396eebe5 192.168.6.214:9000 master - 0
1442403481168 9 connected 10923-16383
04f3bcd569fb325e77a109a530fd376eb84512ce 192.168.32.196:9000 slave
baa1779a73a8d329dfc96104ac3116a24b16a681 0 1442403480667 5 connected
fd383105b93797d7502a7d6bcaa77308cccf2e71 192.168.32.197:9000 slave
e07467f74f81f31c024bb5626c37e550396eebe5 0 1442403482671 9 connected
$127.0.0.1:9000>
```
