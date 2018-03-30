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
>SELECT * FROM  
>(SELECT hash_value,address,substr(sql_text,1,40) sql,  
>	buffer_gets, executions, buffer_gets/executions "Gets/Exec"  
>	FROM V$SQLAREA  
>	WHERE buffer_gets > 100000 AND executions > 10  
>ORDER BY buffer_gets DESC)  
>WHERE rownum <= 10;  

## 4.日常数据库管理

<font color='red'> 12345 </font>
**保证5个日志组在运行**
>--查看日志组  
 select * from v$log;  
 show parameter compatible;  
 select group#,blocksize,archived,members,status from V$log;  

>--增加日志组  
alter database add logfile group 4 ('/orcl/app/oracle/oradata/ORADATA/ORCL/REDO04-01.LOG','/orcl/app/oracle/oradata/ORADATA/ORCL/REDO04-02.LOG')size 200m;  

>--切换日志组  
alter system switch logfile;  

**增加Process**
>--查看ORACLE最大进程数：  
 select count(*) from v$session;  --#连接数  
 Select count(*) from v$session where status='ACTIVE';　--#并发连接数  
 show parameter processes;  --#最大连接  

>--修改连接然后重启数据库  
 alter system set processes=2000 scope = spfile;  
 show parameter processes;  
 create pfile from spfile;  
 

**表空间使用率**
>SELECT a.tablespace_name, ROUND (100 - b.free / a.total * 100) used_pct,  
       ROUND (a.total / 1024 / 1024) "total(MB)",  
       ROUND (b.free / 1024 / 1024) "free_total(MB)",  
       ROUND (b.max_free / 1024 / 1024) "free_max(MB)", b.free_cnt fragment  
  FROM (SELECT   tablespace_name, SUM (BYTES) total  
            FROM dba_data_files  
        GROUP BY tablespace_name) a,  
       (SELECT   tablespace_name, SUM (BYTES) free, MAX (BYTES) max_free,  
                 COUNT (BYTES) free_cnt  
            FROM dba_free_space  
        GROUP BY tablespace_name) b  
WHERE a.tablespace_name=b.tablespace_name   

**创建永久表空间，10G自动扩展，每次扩展1G**
>CREATE SMALLFILE TABLESPACE "BBIS_T_0001"  
 DATAFILE  
 '/orcl/app/oracle/oradata/orcl/bbis_t_0001' SIZE 10G AUTOEXTEND ON NEXT 1G  
 LOGGING  
 DEFAULT NOCOMPRESS NO INMEMORY  
 ONLINE  
 EXTENT MANAGEMENT LOCAL AUTOALLOCATE  
 SEGMENT SPACE MANAGEMENT AUTO;  
 
 **创建临时表空间，2G自动扩展，每次扩展1G**
 >CREATE SMALLFILE TEMPORARY TABLESPACE "BBIS_T_TEMP"  
 TEMPFILE  
 '/orcl/app/oracle/oradata/orcl/bbis_t_temp' SIZE 2G AUTOEXTEND ON NEXT 1G  
 EXTENT MANAGEMENT LOCAL UNIFORM;  
 
 **创建索引表空间，10G自动扩展，每次扩展1G**
 >CREATE SMALLFILE TABLESPACE "BBIS_T_INDEX"  
 DATAFILE  
 '/orcl/app/oracle/oradata/orcl/bbis_t_index' SIZE 10G AUTOEXTEND ON NEXT 1G  
 LOGGING  
 DEFAULT NOCOMPRESS NO INMEMORY  
 ONLINE  
 EXTENT MANAGEMENT LOCAL AUTOALLOCATE  
 SEGMENT SPACE MANAGEMENT AUTO;  
 
 **删除表空间**
 >drop tablespace bbis_t_temp  
 >去HDFS删除表空间文件
 
 **查看用户会话并关闭**
 >select username,serial#, sid from v$session where username = 'bbis_t_0001';  
 >alter system kill session '43,8050';  
 
 **删除用户**
 >drop user bbis_t_0001 cascade;  
 
 **创建用户**
 >CREATE USER bbis_t_0001  IDENTIFIED BY root123  
 DEFAULT TABLESPACE bbis_t_0001  
 TEMPORARY TABLESPACE bbis_t_TEMP ACCOUNT UNLOCK;  
 grant dba to bbis_t_0001;  
 
 
