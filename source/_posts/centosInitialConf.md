---
title: CentOS 初始环境配置
date: 2018-04-05
tags:
  - centos
copyright: true
---
## 前言
这个是在大三实习的时候记录在印象笔记里的，当时学长给我的，现在稍加改动一下，记录在这里。
若刚装完系统ping不通外网，如baidu.com,请参考：[http://dongkelun.com/2018/01/17/networkIsUnreachable/](http://dongkelun.com/2018/01/17/networkIsUnreachable/)
## 1、添加域名解析

在/etc/resolv.conf添加：
``` 
nameserver 114.114.114.114
nameserver 8.8.8.8
nameserver 8.8.4.4
```
然后执行
```
chattr +i /etc/resolv.conf
```
<!-- more -->
## 2、配置epel源

在root用户下执行下面命令：
```
rpm -ivh http://mirrors.yun-idc.com/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum repolist
```
## 3、安装常用工具
```
yum install  openssh wget  vim  openssh-clients openssl gcc openssh-server  mysql-connector-odbc  mysql-connector-java -y
```
## 4、配置最大文件打开数

在root用户下执行下面命令：
```
echo ulimit -n 65536 >> /etc/profile 
source /etc/profile    
ulimit -n    
vim /etc/security/limits.conf
```
在文件尾部添加如下代码：
``` 
* soft nofile 65536
* hard nofile 65536
```
重启系统，在任何用户下查看最大打开文件数：ulimit -n 结果都是65536

## 5、关闭防火墙
```
systemctl status firewalld.service
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl status firewalld.service
```
## 6、更新系统
```
yum update
```


