# Redis概述

> Redis 是什么 

Remote dictionary Service 远程字典服务，是开源的使用ANSI C语言编写，支持网络，可基于内存亦可持久化的日志型，key-value数据库，并提供多种语言的API.

redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。



>redis能干嘛？

1、内存存储、持久化，（内存断电即失，所持久化很重要（RDB\AOF））

2、效率高，可以用于高速缓存

3、发布订阅系统

4、地图信息分析

5、计时器、计数器（浏览量！）



>特性

1、 持久化

2、多样的数据类型

3、集群

4、事务



>学习中需要用到的东西

1、官网 https:///www.redis.io

2、中文网 https:///www.redis.cn

# Redis入门

## 安装

>基本环境安装

```bash
yum intsall gcc-c++
make
make installl
```



yum intsall gcc-c++

6.0以上的 yum intsall gcc-c++ tc



make installl 生成redis-server



将redis.conf复制到 usr/local/bin



## 性能测试

**redis-benchmark** 自带的性能测试工具

![image-20220108172344748](C:\Users\HQ\AppData\Roaming\Typora\typora-user-images\image-20220108172344748.png)

```bash
# 测试性能
redis-benchmark  -h localhost -p 6379 -c 100 -n 100000
```





## 基本知识

右16个数据库

redis.conf

```bash
# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
databases 16

```



默认使用的是第0个

```bash
127.0.0.1:6379> select 3  #切换数据库
OK
127.0.0.1:6379[3]> DBSIZE  #查看大小
(integer) 0
127.0.0.1:6379[3]> 


```

查看当前库大小 ==DBSIZE==

清除当前数据库 ==flushdb==

```bash
127.0.0.1:6379[3]> KEYS * #获取所有的key
1) "name1"
2) "name"
127.0.0.1:6379[3]> FLUSHDB #清楚当前库
OK
```

清除当前数据库 ==flushall==

```bash
127.0.0.1:6379[4]> FLUSHALL #清除所有库数据
OK

```

 

> Redis 是单线程的！（==6.0版本后是多线程，io多线程，工作单线程，执行命令是单线程，网络的IO读写使用了多线程==）

明白Redis是很快的，官方表示，Redis是基于内存操作的，CPU不是Redis性能瓶颈，Redis的瓶颈是根据机器的内存和网络带宽，既然可以使用单线程来实现，就使用单线程了！所有就使用了单线程。

==Redis多线程部分只是为了处理网络数据的读写和协议解析，执行命令仍然是单线程==

Redis是C语言写的，官方提供数据为100000+ 的QPS，完全不比同样使用key-value的Memecache差！

**Redis为什么是单线程的还这么快？****

1、误区1：高性能的服务器一定是多线程的？



2、误区2：多线程（==CPU上下文会切换==！）一定比单线效率高？

 速度：CPU>内存>磁盘

核心：redis是将所有的数据全部放在内存中的，所有说使用单线程去操作的效率就是最高的，多线程（CPU上下文会切换：耗时的操作！！！），对于内存系统来说，如果没有上下文切换效率就是最高的！多次读写都是在一个CPU上的，在内存情况下，这个就是最佳方案



# 五大基本类型

## Redis-Key

>官网文档

Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache, and message broker. Redis provides data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs, geospatial indexes, and streams. Redis has built-in replication, Lua scripting, LRU eviction, transactions, and different levels of on-disk persistence, and provides high availability via Redis Sentinel and automatic partitioning with Redis Cluster.

Redis 是一个开源（BSD 许可）的内存数据结构存储，用作数据库、缓存和消息代理。 Redis 提供数据结构，例如字符串、散列、列表、集合、具有范围查询的排序集合、位图、超日志、地理空间索引和流。 Redis 具有内置复制、Lua 脚本、LRU 驱逐、事务和不同级别的磁盘持久性，并通过 Redis Sentinel 和 Redis Cluster 自动分区提供高可用性。

```bash
127.0.0.1:6379> set name heqiang
OK
127.0.0.1:6379> KEYS *
1) "name"
127.0.0.1:6379> set age  1
OK
127.0.0.1:6379> KEYS *
1) "name"
2) "age"
127.0.0.1:6379> EXISTS name 
(integer) 1
127.0.0.1:6379> EXISTS name1 #判断是否存在该key
(integer) 0
127.0.0.1:6379> MOVE name 1
(integer) 1
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> SELECT 1
OK
127.0.0.1:6379[1]> get name
"heqiang"
127.0.0.1:6379[1]> move name 0  #移动key到指定的db库
(integer) 1
127.0.0.1:6379[1]> get name
(nil)
127.0.0.1:6379[1]> SELECT 0 #切换库db
OK
127.0.0.1:6379> get name
"heqiang"
127.0.0.1:6379> set name heqiang
OK
127.0.0.1:6379> KEYS *
1) "name"
2) "age"
127.0.0.1:6379> set name heqiang222
OK
127.0.0.1:6379> get name
"heqiang222"
127.0.0.1:6379> EXPIRE name 10 # 设置 key过期时间
(integer) 1
127.0.0.1:6379> ttl name  #查看key的过期时间
(integer) 1
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> KEYS *
1) "age"
127.0.0.1:6379> ttl age


127.0.0.1:6379> TYPE name  #查看当前key类型
string



```



## String（字符串）

  ```bash
  127.0.0.1:6379> set key1 v1
  OK
  127.0.0.1:6379> get key1
  "v1"
  127.0.0.1:6379> KEYS *
  1) "name"
  2) "age"
  3) "key1"
  127.0.0.1:6379> EXISTS key1
  (integer) 1
  127.0.0.1:6379> APPEND key1 "hello"  #key 后面拼接 字符串
  (integer) 7
  127.0.0.1:6379> get key1
  "v1hello"
  127.0.0.1:6379> STRLEN
  (error) ERR wrong number of arguments for 'strlen' command
  127.0.0.1:6379> STRLEN key1 #获取key长度
  (integer) 7
  127.0.0.1:6379> 
  ############################################################
  
  127.0.0.1:6379> get view
  "0"
  127.0.0.1:6379> INCR view	#key自增1
  (integer) 1
  127.0.0.1:6379> INCR view
  (integer) 2
  127.0.0.1:6379> get view
  "2"
  127.0.0.1:6379> DECR view	#key自减1
  (integer) 1
  127.0.0.1:6379> get view
  "1"
  
  
  ############################################################
  127.0.0.1:6379> get view
  "1"
  127.0.0.1:6379> INCRBY view 5   #自增指定步长，指定增量
  (integer) 6
  127.0.0.1:6379> DECRBY view 3	#自减指定步长，指定减量
  (integer) 3
  127.0.0.1:6379> 
  
  
  #############################################################
  127.0.0.1:6379> get key1
  "hello,heqiang"
  127.0.0.1:6379> GETRANGE key1 0 3	#截取字符串	【0，3】
  "hell"
  127.0.0.1:6379> get key1
  "hello,heqiang"
  127.0.0.1:6379> GETRANGE key1 0 -1	#截取全部的字符串 和 get key 是一样的
  "hello,heqiang"
  127.0.0.1:6379> 
  
  #替换
  127.0.0.1:6379> get key2
  "abcdefg"
  127.0.0.1:6379> SETRANGE key2 1 xx	#替换指定位置开始的字符串
  (integer) 7
  127.0.0.1:6379> get key2
  "axxdefg"
  127.0.0.1:6379> 
  
  #############################################################
  #setex(set with expire)		# 设置过期时间
  #setnx(set if not exist)	# 不存在，再设置（==在分布式锁中会常常使用==）
  
  127.0.0.1:6379> setex key3 30 hello		#设置key3 的值为hello,30秒后过期
  OK
  127.0.0.1:6379> ttl key3
  (integer) 21
  127.0.0.1:6379> get key3
  "hello"
  127.0.0.1:6379> setnx mykey "redis"		#如果mykey不存在，创建mykey
  (integer) 1
  127.0.0.1:6379> keys *
  1) "mykey"
  2) "key2"
  3) "key1"
  127.0.0.1:6379> ttl key3
  (integer) -2
  127.0.0.1:6379> setnx mykey "mongodb"	#如果mykey存在，创建失败，返回值 0
  (integer) 0
  127.0.0.1:6379> get mykey
  "redis"
  127.0.0.1:6379> 
  
  
  #############################################################
  mset
  mget
  
  127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3 k4 v4	#	同时设置多个值
  OK
  127.0.0.1:6379> keys *
  1) "k4"
  2) "k1"
  3) "k3"
  4) "k2"
  127.0.0.1:6379> mget k1 k2 k3 k4	#	同时获取多个值
  1) "v1"
  2) "v2"
  3) "v3"
  4) "v4"
  127.0.0.1:6379> msetnx k1 v1
  (integer) 0
  127.0.0.1:6379> msetnx k1 v1 k5 v5	#msetnx 是一个原子性操作，要么一起成功，要么一起失败
  (integer) 0
  
  #对象
  set user:1 {name:zhangsan,age:3} #设置一个user:1对象 值为json字符来保存一个对象
  
  #这里的key是一个巧妙的设计：user:{id}:{field},如此设置，在redis中完全OK.
  
  127.0.0.1:6379> mset user:1:name zhangsan user:1:age 2
  OK
  127.0.0.1:6379> mget user:1
  1) (nil)
  127.0.0.1:6379> mget user:1:name user:1:age
  1) "zhangsan"
  2) "2"
  127.0.0.1:6379> 
  
  
  #############################################################
  getset #	先get 后set
  
  127.0.0.1:6379> getset db redis	#如果不存在值，则返回nil
  (nil)
  127.0.0.1:6379> get db
  "redis"
  127.0.0.1:6379> getset db mongodb	#	如果存在值，获取原来的值，并设置新的值。
  "redis"
  127.0.0.1:6379> get db
  "mongodb"
  127.0.0.1:6379> 
  #############################################################
  ```

数据结构是相同的！

String类型的使用场景：value除了是我们的字符串还可以是我们的数字！

- 计数器
- 统计多单位的数量 uid:1234follow 0
- 粉丝数
- 对象缓存存储



## List



## Set



## Hash



## Zset

