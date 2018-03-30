---
layout: post
title: Oracle DBA 工作手册
categories: [教程, 维护, 手册]
tags: []
description: DBA的日常工作内容。
---

## 1.系统运行环境监控

**关注Oracle软件及数据文件所在卷空间使用率**
>df –kv

## 2.数据库运行状况监控

**检查Oracle实例核心后台进程是否都存在、状态是否正常**

>ps -ef|grep ora_

**查看数据库实例是否能正常连接、访问**

>SQL> select status from v$instance;

**监听是否正常**

>lsnrctl status

**是否有表空间出现故障**

>SQL> select tablespace_name,status from dba_tablespaces;

**日志文件是否正常**

>SQL> Select * from v$log;  
SQL> Select * from v$logfile;

## 3.日常性能监控

**通过视图查看当前主要影响性能SQL语句**
	SELECT * FROM

	(SELECT hash_value,address,substr(sql_text,1,40) sql,
		buffer_gets, executions, buffer_gets/executions "Gets/Exec"
		FROM V$SQLAREA
		WHERE buffer_gets > 100000 AND executions > 10
	ORDER BY buffer_gets DESC)
	WHERE rownum <= 10;
