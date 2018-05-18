---
title: 连接mysql报错：Exception in thread "main" java.sql.SQLException:The server time zone value 'EDT' is unrecognized or represents more than one time zone
date: 2018-03-22
tags:
  - mysql
copyright: true
---

最近用spark和srping mybatis 连接mysql会报如下错误：
```
Exception in thread "main" java.sql.SQLException: The server time zone value 'EDT' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
```
网上查了一下说是数据库和系统时区差异所造成的，我想应该是我在虚拟机上装的centos系统是英文版导致的，解决办法有如下两个：
## 1、修改Mysql的时区为东8区
```
set global time_zone='+8:00'
```
## 2、在连接数据库的url后面加上?serverTimezone=UTC，即默认0时区
```
jdbc:mysql://192.168.44.128:3306/hive?serverTimezone=UTC
```
