---
title: vmware centos7 设置固定ip
date: 2018-01-16
tags:
  - vmware
copyright: true
---
## 1、首先设置虚拟机网络连接为NAT模式
![](http://wx2.sinaimg.cn/large/e44344dcly1fpkhd2a0sgj214f0fp44x.jpg)
<!-- more -->
## 2、修改配置文件，设置固定IP
### 2.1、执行一下命令
``` bash
cd /etc/sysconfig/network-scripts
vim ifcfg-eno16777736
```
### 2.2、修改后结果如下
```
HWADDR="00:0C:29:38:82:07"
TYPE="Ethernet"
BOOTPROTO="static"
DEFROUTE="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno16777736"
UUID="2ba00367-6814-4b96-92f3-f8d19c1b0095"
ONBOOT="yes"
IPADDR=192.168.44.128
NETMASK=255.255.255.0
```
其中：
1、BOOTPROTO 修改为static
```
BOOTPROTO="static"
```
2、 底部添加
```
IPADDR=192.168.44.128
NETMASK=255.255.255.0
```
### 2.3、重启网络
``` bash
service network restart
```
### 2.4、查看 修改后的IP
``` bash
ifconfig
```
