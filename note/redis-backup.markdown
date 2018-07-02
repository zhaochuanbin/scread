---
layout : post
title : "redis备份与恢复"
category : redis
tagline: ""
date : 2016-06-14
tags : [nosql,redis,恢复]
---

在redis.conf文件，两种备份策略:RDB方式(默认)和AOF方式

### RDB方式
Redis实现快照
1. Redis使用fork函数复制一份当前进程（父进程）的副本（子进程）
2. 父进程继续接收并处理客户端发来的命令，而子进程开始将内存中的数据写入硬盘中的临时文件
3. 当子进程写入完所有数据后会用该临时文件替换旧的RDB文件，至此一次快照操作完成

在执行fork的时候操作系统（类Unix操作系统）会使用写时复制（copy-on-write）策略，即fork函数发生的一刻父子进程共享同一内存数据，当父进程要更改其中某片数据时（如执行一个写命令 ），操作系统会将该片数据复制一份以保证子进程的数据不受影响，所以新的RDB文件存储的是执行fork一刻的内存数据。

>save 900 1    # 900秒内有至少1个键被更改则进行快照
save 300 10   # 300秒内有至少10个键被更改则进行快照
save 60 10000 # 60秒内有至少10000个键被更改则进行快照

RDB方式的持久化是通过快照（snapshotting）完成的，当符合一定条件时Redis会自动将内存中的所有数据进行快照并存储在硬盘上。
只要满足其中一个条件，就会进行快照。 如果想要禁用自动快照，只需要将所有的save参数删除即可。
Redis默认会将快照文件存储在当前目录(可CONFIG GET dir来查看)的dump.rdb文件中，可以通过配置dir和dbfilename两个参数分别指定快照文件的存储路径和文件名。
除了自动快照，还可以手动发送SAVE或BGSAVE命令让Redis执行快照，两个命令的区别在于，前者是由主进程进行快照操作，会阻塞住其他请求，后者会通过fork子进程进行快照操作。 Redis启动后会读取RDB快照文件，将数据从硬盘载入到内存。根据数据量大小与结构和服务器性能不同，这个时间也不同。通常将一个记录一千万个字符串类型键、大小为1GB的快照文件载入到内 存中需要花费20～30秒钟。 通过RDB方式实现持久化，一旦Redis异常退出，就会丢失最后一次快照以后更改的所有数据。这就需要开发者根据具体的应用场合，通过组合设置自动快照条件的方式来将可能发生的数据损失控制在能够接受的范围。如果数据很重要以至于无法承受任何损失，则可以考虑使用AOF方式进行持久化。

### AOF方式
在redis.conf中通过appendonly参数开启
> appendonly yes

在启动时Redis会逐个执行AOF文件中的命令来将硬盘中的数据载入到内存中，载入的速度相较RDB会慢一些

开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件。AOF文件的保存位置和RDB文件的位置相同，都是通过dir参数设置的，默认的文件名是appendonly.aof，可以通过appendfilename参数修改：
> appendfilename appendonly.aof

配置redis自动重写AOF文件的条件
>auto-aof-rewrite-percentage 100  # 当目前的AOF文件大小超过上一次重写时的AOF文件大小的百分之多少时会再次进行重写，如果之前没有重写过，则以启动时的AOF文件大小为依据
auto-aof-rewrite-min-size 64mb   # 允许重写的最小AOF文件大小

配置写入AOF文件后，要求系统刷新硬盘缓存的机制
> appendfsync always   # 每次执行写入都会执行同步，最安全也最慢
appendfsync everysec   # 每秒执行一次同步操作
appendfsync no       # 不主动进行同步操作，而是完全交由操作系统来做（即每30秒一次），最快也最不安全

Redis允许同时开启AOF和RDB，重新启动Redis后Redis会使用AOF文件来恢复数据
