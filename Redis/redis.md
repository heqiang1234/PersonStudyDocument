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



> 启动redis服务
>
> redis-server  hqconfig/redis.conf
>
> 启动操作客户端
>
> redis-cli -p 6379







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

在redis中，我们可以吧list玩成，栈、队列、阻塞队列！

所有的list命令都是 ==l== 开头的，Redis不区分大小写命令

```bash
#############################################################
127.0.0.1:6379> LPUSH list one		#将一个值或者多个值，插入到列表头部（左），双端队列，（列表+ 栈）
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1		#获取list的值   key   start   end
1) "three"
2) "two"
3) "one"
127.0.0.1:6379> LRANGE list 0 1			#通过区间获取具体的值
1) "three"
2) "two"
127.0.0.1:6379> LRANGE list 0 0
1) "three"
127.0.0.1:6379> RPUSH list four 	#将一个值或者多个值，插入到列表头部（右），双端队列，（列表+ 栈）
(integer) 4
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "four"
127.0.0.1:6379> 


#############################################################
LPOP
RPOP

127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
3) "one"
4) "four"
127.0.0.1:6379> LPOP list	#弹出list的第一个元素
"three"
127.0.0.1:6379> RPOP list	#弹出list的最后一个元素
"four"
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> 


#############################################################
Lindex
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
2) "one"
127.0.0.1:6379> LINDEX list 1		#通过下标获取list中的某一个值
"one"
127.0.0.1:6379> LINDEX list 0
"two"


#############################################################
Llen

127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> LPUSH list two
(integer) 2
127.0.0.1:6379> LPUSH list three
(integer) 3
127.0.0.1:6379> Llen list 	#返回列表的长度
(integer) 3


#############################################################
移除指定的值！
取关 uid

LREM 【key】 【count】 【value】


127.0.0.1:6379> LREM list 1 one
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "three"
2) "two"
127.0.0.1:6379> lrem list 1 three	#移除list集合中 指定个数 的value，精确匹配
(integer) 1
127.0.0.1:6379> lrem list 41 three
(integer) 0
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
127.0.0.1:6379> LPUSH list four
(integer) 2
127.0.0.1:6379> LREM list 20 four
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "two"
127.0.0.1:6379> LPUSH list one
(integer) 2
127.0.0.1:6379> LPUSH list one
(integer) 3
127.0.0.1:6379> LPUSH list one
(integer) 4
127.0.0.1:6379> 
127.0.0.1:6379> LPUSH list one
(integer) 5
127.0.0.1:6379> LRANGE list 0 -1
1) "one"
2) "one"
3) "one"
4) "one"
5) "two"
127.0.0.1:6379> LREM list 3 one
(integer) 3
127.0.0.1:6379> LRANGE list 0 -1
1) "one"
2) "two"
127.0.0.1:6379> 



#############################################################
trim 修剪  list 截断

LTRIM 【key】	【start】 【end】

127.0.0.1:6379> RPUSH mylist "hello"
(integer) 1
127.0.0.1:6379> RPUSH mylist "hello1"
(integer) 2
127.0.0.1:6379> RPUSH mylist "hello2"
(integer) 3
127.0.0.1:6379> RPUSH mylist "hello3"
(integer) 4
127.0.0.1:6379> RPUSH mylist "hello4"
(integer) 5
127.0.0.1:6379> LTRIM mylist 1 2		#通过下标截取指定的长度，这个list已经被改变了，截断了只剩下截取的元素！
OK
127.0.0.1:6379> LRANGE  mylist 0 -1
1) "hello1"
2) "hello2"


#############################################################
rpoplpush	#弹出列表最后一个元素，并移动到新的列表中

127.0.0.1:6379> LRANGE  mylist 0 -1		#移除列表的最后一个元素，将他移动到新的列表中
1) "hello1"
2) "hello2"
127.0.0.1:6379> RPOPLPUSH mylist mymotherlist	#查看原来的列表
"hello2"
127.0.0.1:6379> LRANGE  mymotherlist 0 -1		#查看目标列表中，确实存在值
1) "hello2"
127.0.0.1:6379> LRANGE  mylist 0 -1
1) "hello1"
127.0.0.1:6379> 


#############################################################
lset 【key】	【index】 【value】	#将列表中指定下标的值替换成另外一个值，更新操作

127.0.0.1:6379> EXISTS list		#判断列表是否存在
(integer) 0
127.0.0.1:6379> LPUSH list one
(integer) 1
127.0.0.1:6379> EXISTS list
(integer) 1
127.0.0.1:6379> LSET list 0 edd		#如果存在，更新当前下标的值
OK
127.0.0.1:6379> EXISTS list
(integer) 1
127.0.0.1:6379> LRANGE list 0 -1
1) "edd"
127.0.0.1:6379> LSET list 2 hel		#如果不存在，则会报错
(error) ERR index out of range
127.0.0.1:6379> 


#############################################################
LINSERT		#将某一个具体的value，插入到列表中某个元素的前面或者后面

127.0.0.1:6379> RPUSH mylist "hello"
(integer) 1
127.0.0.1:6379> RPUSH mylist "hello1"
(integer) 2
127.0.0.1:6379> LINSERT mylist before "hello1" "other"
(integer) 3
127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello"
2) "other"
3) "hello1"
127.0.0.1:6379> LINSERT mylist before "11" "hee"
(integer) -1
127.0.0.1:6379> LINSERT mylist after hello1 "22"
(integer) 4
127.0.0.1:6379> LINSERT mylist after other "33"
(integer) 5
127.0.0.1:6379> LRANGE mylist 0 -1
1) "hello"
2) "other"
3) "33"
4) "hello1"
5) "22"


```

>小结

- 他实际上是一个链表，before Node after,,left, right 都可以插入值
- 如果key不存在，创建新链表
- 如果key存在，新增内容
- 如果移除了所有的值，空链表，也代表不存在！
- 在两边插入或者改动值，效率最高！中间元素，相对而言效率会低一些~

消息队列（ Lpush  Rpop） 栈（Lpush、Lpop）



## Set

set值是不能重复的

```bash
#############################################################
SADD	

127.0.0.1:6379> SADD myset "hello"		#set集合中添加元素
(integer) 1
127.0.0.1:6379> SADD myset "h1"
(integer) 1
127.0.0.1:6379> SADD myset "h2"
(integer) 1
127.0.0.1:6379> SMEMBERS myset			#查看set集合的所有元素的值
1) "hello"
2) "h1"
3) "h2"
127.0.0.1:6379> SISMEMBER myset hello	#判断set集合是否存在该元素
(integer) 1
127.0.0.1:6379> SISMEMBER myset ww
(integer) 0


#############################################################
SCRD	
127.0.0.1:6379> SCARD myset		#获取set集合中的内容元素
(integer) 3
127.0.0.1:6379> 


#############################################################
srem

127.0.0.1:6379> SMEMBERS myset
1) "hello"
2) "h1"
3) "h2"
127.0.0.1:6379> SISMEMBER myset hello
(integer) 1
127.0.0.1:6379> SISMEMBER myset ww
(integer) 0
127.0.0.1:6379> SCARD myset
(integer) 3
127.0.0.1:6379> SREM myset hello	#移除set集合中的指定元素
(integer) 1
127.0.0.1:6379> SCARD myset
(integer) 2
127.0.0.1:6379> 
127.0.0.1:6379> SRANDMEMBER myset 2		# 随机抽选一个元素
1) "h1"
2) "h2"
127.0.0.1:6379> 


#############################################################
spop     删除指定的key，随机删除key!

127.0.0.1:6379> SMEMBERS myset
1) "h3"
2) "h1"
3) "h2"
127.0.0.1:6379> spop myset		#随机删除一些set元素集合中的元素
"h2"
127.0.0.1:6379> SMEMBERS myset
1) "h3"
2) "h1"
127.0.0.1:6379> 



#############################################################
smove  将一个指定的值，移动到指定的位置

127.0.0.1:6379> SADD myset "hello"
(integer) 1
127.0.0.1:6379> SADD myset "world"
(integer) 1
127.0.0.1:6379> SADD myset "HQ"
(integer) 1
127.0.0.1:6379> SADD myset2 "set2"
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "HQ"
2) "hello"
3) "world"
127.0.0.1:6379> SMEMBERS myset2
1) "set2"
127.0.0.1:6379> 
127.0.0.1:6379> SMOVE myset myset2 "hq"	
(integer) 0
127.0.0.1:6379> SMOVE myset myset2 "HQ"		#将一个指定的值，移动到另外一个set集合
(integer) 1
127.0.0.1:6379> SMEMBERS myset
1) "hello"
2) "world"
127.0.0.1:6379> SMEMBERS myset2
1) "HQ"
2) "set2"
127.0.0.1:6379> 


#############################################################
微博，B站，共同关注！（并集）
数据集合类：
  - 差集：
  - 交集：
  - 并集：

127.0.0.1:6379> SADD key1 a
(integer) 1
127.0.0.1:6379> SADD key1 b
(integer) 1
127.0.0.1:6379> SADD key1 c
(integer) 1
127.0.0.1:6379> SADD key2 c
(integer) 1
127.0.0.1:6379> SADD key2 d
(integer) 1
127.0.0.1:6379> SADD key2 e
(integer) 1
127.0.0.1:6379> SDIFF key1 key2		#差集
1) "b"
2) "a"
127.0.0.1:6379> SINTER key1 key2	#交集
1) "c"
127.0.0.1:6379> SUNION key1 key2	#并集
1) "b"
2) "c"
3) "e"
4) "a"
5) "d"
127.0.0.1:6379> 

```

微博，A用户将所有关注的人放在一个set集合中！将它的粉丝也放在一个集合中！

共同关注，共同爱好，二度好友（六度分隔理论）



## Hash（哈希）

Map集合，key-map!时候这个值是一个map集合！

set  maphash field HQ

```bash
#############################################################
127.0.0.1:6379> HSET myhash field1 hq	#set一个具体 key-value
(integer) 1
127.0.0.1:6379> hget myhash field1		#获取一个字段值
"hq"
127.0.0.1:6379> hmset myhash field hello field2 world	#set多个 key-value
OK
127.0.0.1:6379> hmget myhash field1 field2	#获取多个字段值
1) "hq"
2) "world"
127.0.0.1:6379> hgetall myhash		#获取全部的数据
1) "field1"
2) "hq"
3) "field"
4) "hello"
5) "field2"
6) "world"
127.0.0.1:6379> hdel myhash field1 		#删除hash指定的kry字段！对应的value值也被删除了
(integer) 1


#############################################################
hlen

127.0.0.1:6379> hlen myhash		#获取hash表的字段数量！
(integer) 2


#############################################################
HEXISTS

127.0.0.1:6379> HEXISTS myhash field	#判断hash中指定字段是否存在
(integer) 1



#############################################################
# 只获取所有的field
# 只获得所有的field
127.0.0.1:6379> HKEYS myhash
1) "field"
2) "field2"
127.0.0.1:6379> HVALS myhash
1) "hello"
2) "world"
127.0.0.1:6379> 


#############################################################
hincr

127.0.0.1:6379> HSET myhash field5 1		#指定增量
(integer) 1
127.0.0.1:6379> HINCRBY myhash field5 1			# 如果不存在则可以设置
(integer) 2
127.0.0.1:6379> 
127.0.0.1:6379> HSETNX myhash field4 hello		# 如果不存在则可以设置
(integer) 1
127.0.0.1:6379> HSETNX myhash field4 wordls		# 如果存在则不可以设置
(integer) 0

```

hash 变更的数据 user name age,尤其是用户信息之类，经常变动的信息！hash更适合存储对象，String更加适合字符串存储。



## Zset

在set的基础上，增加了一个值，set k1 v1      zset  k1 score v1

==底层数据结构是 跳跃链表==

```bash

127.0.0.1:6379> ZADD myset 1 one	#增加一个值
(integer) 1
127.0.0.1:6379> zadd myset 2 two 3 three	#增加多个值
(integer) 2
127.0.0.1:6379> ZRANGE myset 0 1-
(error) ERR value is not an integer or out of range
127.0.0.1:6379> ZRANGE myset 0 -1
1) "one"
2) "two"
3) "three"
127.0.0.1:6379> 


#############################################################
排序

127.0.0.1:6379> ZADD sloary 2500 xiaohong
(integer) 1
127.0.0.1:6379> ZADD sloary 5000 zhangsan
(integer) 1
127.0.0.1:6379> ZADD sloary 500 hq
(integer) 1
127.0.0.1:6379> ZRANGE sloary 0 -1
1) "hq"
2) "xiaohong"
3) "zhangsan"
127.0.0.1:6379> ZRANGEBYSCORE sloary -inf inf 	#显示全部的用户 从小到大
1) "hq"
2) "xiaohong"
3) "zhangsan"
127.0.0.1:6379> ZREVRANGE sloary 0 -1	#显示全部的用户 从大到小
1) "zhangsan"
2) "hq"
127.0.0.1:6379> ZRANGEBYSCORE sloary 0 20
(empty list or set)
127.0.0.1:6379> ZRANGEBYSCORE sloary -inf +inf withscores	#显示全部的用户并且附带分数
1) "hq"
2) "500"
3) "xiaohong"
4) "2500"
5) "zhangsan"
6) "5000"
127.0.0.1:6379> ZRANGEBYSCORE sloary -inf 2500 withscores	#显示小于2500的升序排列
1) "hq"
2) "500"
3) "xiaohong"
4) "2500"
127.0.0.1:6379> 


#############################################################
移除rem中的元素

127.0.0.1:6379> ZRANGE sloary 0 -1
1) "hq"
2) "xiaohong"
3) "zhangsan"
127.0.0.1:6379> ZREM sloary xiaohong	#移除有序集合中的指定元素
(integer) 1
127.0.0.1:6379> ZREM sloary xiaohonga
(integer) 0
127.0.0.1:6379> ZRANGE sloary 0 -1
1) "hq"
2) "zhangsan"
127.0.0.1:6379> 


#############################################################
计算里面的个数

127.0.0.1:6379> ZCARD sloary	#获取有序集合中的个数
(integer) 2
127.0.0.1:6379> ZADD myset 3 hq
(integer) 1
127.0.0.1:6379> ZCOUNT myset 1 3	#获取有序集合中区间的数量
(integer) 3
127.0.0.1:6379> ZCOUNT myset 1 2
(integer) 2



```

其余的api，查看官方文档：https://redis.io/commands

案例：set的排序版  存储班级成绩，工资表排序

普通消息，1，重要消息 ，2，带权重进行判断！

排行榜应用实现，



# 三种特殊数据类型

## Geospatotal 地理位置

>GEOADD
>
>GEODIST
>
>GEOHASH
>
>GEOPOS
>
>GEORADIUS
>
>GEORADIUSBYMEMBER



>GEOADD

```bash
#geoadd 添加地理位置
# 规则：两级无法直接添加，我们一般会下载城市数据，直接通过java程序导入
# 参数  key （经度、纬度、名称）

127.0.0.1:6379> geoadd china:city 116.40 39.90 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 121.47 31.23 shanghai
(integer) 1
127.0.0.1:6379> geoadd china:city 106.50 29.53 chongqing
(integer) 1
127.0.0.1:6379> geoadd china:city 114.05 22.52 shenzhen
(integer) 1
127.0.0.1:6379> geoadd china:city 120.16 30.24 hangzhou
(integer) 1
127.0.0.1:6379> geoadd china:city 108.96 34.26 xian

```



> GEOPOS 获取当前定位：一定是一个坐标值

```bash
127.0.0.1:6379> GEOPOS china:city beijing	#h获取指定城市的经度和纬度
1) 1) "116.39999896287918091"
   2) "39.90000009167092543"
127.0.0.1:6379> GEOPOS china:city chongqing
1) 1) "106.49999767541885376"
   2) "29.52999957900659211"
127.0.0.1:6379> 

```



>GEODIST 两人之间的距离
>
>
>
>给定一个表示地理空间索引的排序集，使用[GEOADD](https://redis.io/commands/geoadd)命令填充，该命令返回指定单元中两个指定成员之间的距离。
>
>如果缺少一个或两个成员，则该命令返回 NULL。
>
>单位必须是以下之一，默认为米：
>
>- **m**为米。
>- **公里**换公里。
>- **英里**数英里。
>- **英尺**换英尺

```bash

127.0.0.1:6379> GEODIST china:city shanghai beijing
"1067378.7564"
127.0.0.1:6379> GEODIST china:city shanghai beijing km	#获取指定位置的间距，可自行设置单位
"1067.3788"
127.0.0.1:6379> GEODIST china:city shanghai hanghzhou km
(nil)
127.0.0.1:6379> GEODIST china:city shanghai hangzhou km
"166.7613"
127.0.0.1:6379> 

```



>georadius 	以给定的经纬度为中心，找出某一半径内的元素
>
>我附近的人，（获取所有附近的人的地址，定位）

```bash
7.0.0.1:6379> GEORADIUS china:city 110 30 1000 km	#以110 30为中心，寻找1000km内的城市
1) "chongqing"
2) "xian"
3) "shenzhen"
4) "hangzhou"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km
1) "chongqing"
2) "xian"
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withcoord	
1) 1) "chongqing"
   2) 1) "106.49999767541885376"
      2) "29.52999957900659211"
2) 1) "xian"
   2) 1) "108.96000176668167114"
      2) "34.25999964418929977"
#   withdist显示到中心距离的位置    withcoord 显示他人的定位   count   1 筛选出指定的结果！
127.0.0.1:6379> GEORADIUS china:city 110 30 500 km withdist withcoord count 1  
#   withdist显示到中心距离的位置
1) 1) "chongqing"
   2) "341.9374"
   3) 1) "106.49999767541885376"
      2) "29.52999957900659211"
127.0.0.1:6379> 

```



>GEORADIUSBYMEMBER

```bash
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 1000 km 	#找出位于指定元素周围的其他元素
1) "beijing"
2) "xian"
127.0.0.1:6379> GEORADIUSBYMEMBER china:city beijing 400 km
1) "beijing"
127.0.0.1:6379> 

```



>GEOHASH 返回一个或者多个位置元素的geohash表示
>
>**时间复杂度：**每个请求成员的 O(log(N))，其中 N 是排序集中的元素数。
>
>返回有效的[Geohash](https://en.wikipedia.org/wiki/Geohash)字符串，表示一个或多个元素在表示地理空间索引的排序集合值中的位置（其中元素是使用[GEOADD](https://redis.io/commands/geoadd)添加的）。

```bash
127.0.0.1:6379> GEOHASH china:city beijing chongqing	#将二维的经纬度转换为一维的字符串，如果两个字符串越像，表示越近
1) "wx4fbxxfke0"
2) "wm5xzrybty0"
127.0.0.1:6379> 
```



>GEO底层的实现原理就是zset！我们可以使用Zset命令操作GEO

```bash
127.0.0.1:6379> ZRANGE china:city 0 -1		#查看地图中全部元素
1) "chongqing"
2) "xian"
3) "shenzhen"
4) "hangzhou"
5) "shanghai"
6) "beijing"
127.0.0.1:6379> ZREM china:city beijing		#移除指定元素
(integer) 1
127.0.0.1:6379> ZRANGE china:city 0 -1	
1) "chongqing"
2) "xian"
3) "shenzhen"
4) "hangzhou"
5) "shanghai"
127.0.0.1:6379> 

```



## Hyperloglog

>什么是基数？

A {1,3,5,7,8,7}

B {1,3,5,7,8}

基数 { 不重复数量} = 5 可以接受误差

>简介

Redis 2.9版本更新二零hyperloglog数据结构

Redis Hyperloglog 基数统计的算法

优点：占用的内存是固定，2^64不同的元素的技术，只需要使用12kb的内存，如果从内存的角度来比较的话 Hyperloglog首选！

网页的UV（一个人访问一个网站多次，但是还是算作一个人）

传统方式，set保护用户的id，就会比较麻烦！我们的目的是为了计数，而不是保存用户的id

0.81的错误率，统计UV任务，可以忽略不计！

>测试使用

```bash
127.0.0.1:6379> PFADD mykey a b c d e f g h i j l	#创建第一组元素 mykey
(integer) 1
127.0.0.1:6379> PFCOUNT mykey		#统计mykey 元素的基数数量
(integer) 11
127.0.0.1:6379> PFADD mykey2 i j z x cv b n m 	#创建第一组元素 mykey2
(integer) 1
127.0.0.1:6379> PFCOUNT mykey2
(integer) 8
127.0.0.1:6379> PFMERGE mykey3 mykey mykey2		#合并两组 mykey mykey2到 mykey3	并集
OK
127.0.0.1:6379> PFCOUNT mykey3		#查看并集的数量
(integer) 16
127.0.0.1:6379> 

```

如果允许同错，一定可以使用hyperloglog，

不允许容错，就使用 set 或者自己的数据类型即可





## bitmaps
