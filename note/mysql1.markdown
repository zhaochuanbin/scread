---
layout : post
title : "mysql相关问题"
category : mysql
tagline: ""
date : 2016-02-10
tags : [mysql,数据库]
---


### mysql负载过高，如何找到是由哪些SQL引起的


**方法：慢查询日志分析(MYSQLdumpslow)**


>show variables like '%slow%';

找出日志位置，在my.cnf或my.ini中添加
slow_query_log=true  
slow_query_log_file = *-slow.log
long_query_time=1  
log-queries-not-using-indexes=true

admin cmd->net stop mysql 

admin cmd->net start mysql 

下载[perl工具](http://www.activestate.com/activeperl/downloads),

然后执行以下命令

D:\mysql\bin>mysqldumpslow.pl -s t -t 5 D:\mysql\data\Anderson-PC-slow.log  > d:\slow.txt


>Count: 1076100  Time=0.09s (99065s)  Lock=0.00s (76s)  Rows=408.9 (440058825), edu_online[edu_online]@28hosts
  select * from t_online_group_records where UNIX_TIMESTAMP(gre_updatetime) > N

以第1条为例，表示这类SQL(N可以取很多值，这里MySQLdumpslow会归并起来)在8月19号的慢查询日志内出现了1076100次，总耗时99065秒，总返回440058825行记录，有28个客户端IP用到。

通过慢查询日志分析，就可以找到最耗时的SQL，然后进行具体的SQL分析了


**慢查询相关的配置参数**


log_slow_queries:是否打开慢查询日志，=ON

long_query_time：查询时间大于多少秒的SQL被当做是慢查询，一般设为1S

log_queries_not_using_indexes：是否将没有使用索引的记录写入慢查询日志

slow_query_log_file：慢查询日志存放路径



### 如何针对具体的MSQL做优化


**使用Explain分析SQL语句执行计划**


EXPLAIN select * from mto_config where key_ = 'site_name';

type：使用类别，有无使用到索引。结果值从好到坏：… > range(使用到索引) > index > ALL(全表扫描)，一般查询应达到range级别

rows：SQL执行检查的记录数

Extra：SQL执行的附加信息，如”Using index”表示查询只用到索引列，不需要去读表等


**使用Profiles分析SQL语句执行时间和消耗资源**

>MySQL> set profiling=1; (启动profiles，默认是没开启的)
MySQL> select count(1) from t_online_group_records where UNIX_TIMESTAMP(gre_updatetime) > 123456789; (执行要分析的SQL语句)
MySQL> show profiles;
+----------+------------+----------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                        |
+----------+------------+----------------------------------------------------------------------------------------------+
|        1 | 0.00043250 | select count(1) from t_online_group_records where UNIX_TIMESTAMP(gre_updatetime) > 123456789 |
+----------+------------+----------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
MySQL> show profile cpu,block io for query 1; (可看出SQL在各个环节的耗时和资源消耗)
+----------------------+----------+----------+------------+--------------+---------------+
| Status               | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+----------------------+----------+----------+------------+--------------+---------------+
...
| optimizing           | 0.000016 | 0.000000 |   0.000000 |            0 |             0 |
| statistics           | 0.000020 | 0.000000 |   0.000000 |            0 |             0 |
| preparing            | 0.000017 | 0.000000 |   0.000000 |            0 |             0 |
| executing            | 0.000011 | 0.000000 |   0.000000 |            0 |             0 |
| Sending data         | 0.000076 | 0.000000 |   0.000000 |            0 |             0 |


### SQL层面已难以优化，请求量继续增大时的应对策略


分库分表
使用集群(master-slave)，读写分离
增加业务的cache层
使用连接池

### MySQL如何做主从数据同步


**复制机制（Replication）**


master通过复制机制，将master的写操作通过binlog传到slave生成中继日志(relaylog)，slave再将中继日志redo，使得主库和从库的数据保持同步


**复制相关的3个MySQL线程**


slave上的I/O线程：向master请求数据

master上的Binlog Dump线程：读取binlog事件并把数据发送给slave的I/O线程

slave上的SQL线程：读取中继日志并执行，更新数据库

属于slave主动请求拉取的模式


**实际使用可能遇到的问题**


数据非强一致：CDB默认为异步复制，master和slave的数据会有一定延迟(称为主从同步距离，一般 主从同步距离变大：可能是DB写入压力大，也可能是slave机器负载高，网络波动等原因，具体问题具体分析


**相关监控命令**


show processlist：查看MySQL进程信息，包括3个同步线程的当前状态

show master status ：查看master配置及当前复制信息

show slave status：查看slave配置及当前复制信息

### 如何防止DB误操作和做好容灾

业务侧应做到的几点：

重要DB数据的手工修改操作，操作前需做到2点：1 先在测试环境操作 2 备份数据

根据业务重要性做定时备份，考虑系统可承受的恢复时间


**MySQL备份和恢复操作**


1.备份：使用MySQLdump导出数据

>MySQLdump -u 用户名 -p 数据库名 [表名] > 导出的文件名

>MySQLdump -uxxx -p xxx mytable > mytable.20140921.bak.sql

2.恢复：导入备份数据

MySQL -uxxx -p xxxx

3.恢复：导入备份数据之后发送的写操作。先使用MySQLbinlog导出这部分写操作SQL(基于时间点或位置)

如导出2014-09-21 09:59:59之后的binlog：

>MySQLbinlog --database="test" --start-date="2014-09-21 09:59:59" /var/lib/MySQL/mybinlog.000001 > binlog.data.sql

如导出起始id为123456之后的binlog：

>MySQLbinlog --database="test" --start-position="123456" /var/lib/MySQL/mybinlog.000001 > binlog.data.sql

最后把要恢复的binlog导入db

MySQL -uxxxx -p xxxx


### 该选择MySQL哪种存储引擎，Innodb具有什么特性


存储引擎解决的问题：如何组织MySQL数据在介质中高效地读取，需考虑存储机制、索引设计、并发读写的锁机制等

MySQL5.0支持的存储引擎有MyISAM、InnoDB、Memory、Merge等

1.InnoDB

MySQL5.5之后及CDB的默认引擎。

支持行锁：并发性能好

支持事务：故InnoDB称为事务性存储引擎，支持ACID，提供了具有提交、回滚和崩溃恢复能力的事务安全

支持外键：当前唯一支持外键的引擎

2.MyISAM

MySQL5.5之前默认引擎

支持表锁：插入+查询速度快，更新+删除速度慢

不支持事务

**使用show engines可查看当前MySQL支持的存储引擎详情**


### MySQL内部结构有哪些层次


Connectors：连接器。接收不同语言的Client交互

Management Serveices & Utilities：系统管理和控制工具

Connection Pool: 连接池。管理用户连接

SQL Interface: SQL接口。接受用户的SQL命令，并且返回用户需要查询的结果

Parser: 解析器。验证和解析SQL语句成内部数据结构

Optimizer: 查询优化器。为查询语句选择合适的执行路径

Cache和Buffer：查询缓存。缓存查询的结果，有命中即可直接返回

Engine：存储引擎。MySQL数据最后组织并存储成具体文件
