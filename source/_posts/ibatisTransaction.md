---
title: ibatis 事务 java 
date: 2018-01-22
tags:
  - ibatis
  - java
copyright: true
---

### java 代码
``` java
	SqlMapClient client = getSqlMapClient();
	try {
	      client.startTransaction();
	      client.getCurrentConnection().setAutoCommit(false);
	      client.delete("order.delete");
	      client.insert("order.insertResult");
	           
	      //测试异常
	      System.out.println(1/0);
	      client.getCurrentConnection().commit();
	      client.commitTransaction();
	  } catch (Exception e) {
	      e.printStackTrace();
	           
	  }finally{
	   	  client.endTransaction();
	  }
```