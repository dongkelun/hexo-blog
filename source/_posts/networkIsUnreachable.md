---
title: network is unreachable centos无法连接外网（或unknown host baidu.com）
date: 2018-01-17
tags:
  - centos
copyright: true
---
## 前言
在虚拟机上新装的系统设置固定ip、重启系统之后，可能ping不通外网，出现如标题所示错误。
## 1、执行以下命令即可（临时添加网关）
``` bash
sudo route add default gw 192.168.44.2
```
<!-- more -->
## 2、永久性修改网关
### 2.1 修改配置文件
``` bash
vim /etc/sysconfig/network
```
在最下面添加如下内容：
```
GATEWAY=192.168.44.2
```
### 2.2 重启网卡
``` bash
service network restart
```