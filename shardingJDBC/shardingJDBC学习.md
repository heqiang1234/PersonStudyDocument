# 1、shardingJDBC概述

官网：https://shardingsphere.apache.org/document/legacy/4.x/document/cn/overview/

**ShardingSphere**是一套开源的分布式数据库中间件解决方案组成的生态圈，它由Sharding-JDBC、Sharding-Proxy和Sharding-Sidecar（计划中）这3款相互独立的产品组成。 他们均**提供标准化的数据分片、分布式事务和数据库治理功能**，可适用于如Java同构、异构语言、云原生等各种多样化的应用场景。

ShardingSphere定位为==关系型数据库中间件==，旨在充分合理地在分布式的场景下利用关系型数据库的计算和存储能力，而并非实现一个全新的关系型数据库。 它与NoSQL和NewSQL是并存而非互斥的关系。NoSQL和NewSQL作为新技术探索的前沿，放眼未来，拥抱变化，是非常值得推荐的。反之，也可以用另一种思路看待问题，放眼未来，关注不变的东西，进而抓住事物本质。 关系型数据库当今依然占有巨大市场，是各个公司核心业务的基石，未来也难于撼动，我们目前阶段更加关注在原有基础上的增量，而非颠覆。



shardingJDBC    mycat

>sharding更改名字

3.0之后更新为shardingShpere



>认识shardingJDBC

![ShardingSphere Scope](https://shardingsphere.apache.org/document/legacy/4.x/document/img/shardingsphere-scope_cn.png)

**数据量大，引发出分库分表的处理方式**

定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务。 它使用客户端直连数据库，**以 jar 包形式提供服务**，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。

适用于任何基于 JDBC 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template 或直接使用 JDBC。
支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP 等。
支持任意实现 JDBC 规范的数据库，目前支持 MySQL，Oracle，SQLServer，PostgreSQL 以及任何遵循 SQL92 标准的数据库。

>shardingJDBC和mycat的区别

mycat 独立的服务

shardingJDBC 融合在一起，jar包提供服务

> 连接池

第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP 等

频繁的开启连接，会进行TCP连接，会消耗资源，降低性能，连接池减少了TCP连接，提高性能，管理连接对象，

shardingJDBC，专门管理连接，通过shardingJDBC管理连接



## 认识 Sharding-Proxy 

![Sharding-Proxy Architecture](https://shardingsphere.apache.org/document/legacy/4.x/document/img/sharding-proxy-brief_v2.png)

定位为透明化的数据库代理端，提供封装了数据库二进制协议的服务端版本，用于完成对异构语言的支持。 目前先提供MySQL/PostgreSQL版本，它可以使用任何兼容MySQL/PostgreSQL协议的访问客户端(如：MySQL Command Client, MySQL Workbench, Navicat等)操作数据，对DBA更加友好。

- 向应用程序完全透明，可直接当做MySQL/PostgreSQL使用。
- 适用于任何兼容MySQL/PostgreSQL协议的的客户端。



## Sharding-JDBC、Sharding-Proxy、*Sharding-Sidecar*

| Sharding-JDBC | *Sharding-Proxy* | *Sharding-Sidecar* |        |
| :------------ | :--------------- | :----------------- | ------ |
| 数据库        | 任意             | MySQL              | MySQL  |
| 连接消耗数    | 高               | 低                 | 高     |
| 异构语言      | 仅Java           | 任意               | 任意   |
| 性能          | 损耗低           | 损耗略高           | 损耗低 |
| 无中心化      | 是               | 否                 | 是     |
| 静态入口      | 无               | 有                 | 无     |



## 混合架构

Sharding-JDBC采用无中心化架构，适用于Java开发的高性能的轻量级OLTP应用；Sharding-Proxy提供静态入口以及异构语言的支持，适用于OLAP应用以及对分片数据库进行管理和运维的场景。

ShardingSphere是多接入端共同组成的生态圈。 通过混合使用Sharding-JDBC和Sharding-Proxy，并采用同一注册中心统一配置分片策略，能够灵活的搭建适用于各种场景的应用系统，架构师可以更加自由的调整适合于当前业务的最佳系统架构。

[![ShardingSphere Hybrid Architecture](https://shardingsphere.apache.org/document/legacy/4.x/document/img/shardingsphere-hybrid.png)](https://shardingsphere.apache.org/document/legacy/4.x/document/img/shardingsphere-hybrid.png)



## Sharding-JDBC功能分类

**功能列表**

- 数据分片

- 分库 & 分表
- 读写分离   （读取数据的人多，修改数据的人少，使用读写分离，提高响应速度）
- 分片策略定制化
- 无中心化分布式主键

**分布式事务 ** （RabboitMQ）

- 标准化事务接口
- XA强一致事务
- 柔性事务

**数据库治理**

- 配置动态化
- 编排 & 治理
- 数据脱敏
- 可视化链路追踪
- 弹性伸缩(规划中)

**项目状态**

[![Status](https://shardingsphere.apache.org/document/legacy/4.x/document/img/shardingsphere-status_cn.png)](https://shardingsphere.apache.org/document/legacy/4.x/document/img/shardingsphere-status_cn.png)



## shardingSphere 数据分片内核剖析

shardingSphere的3个产品的数据分片主要流程是完全一致的，核心由SQL解析==> 执行器优化 ==> SQL路 ==> SQL改写 ==> SQL执行 ==>结果归并的流程组成。

> SQL解析

分为词法解析和语法解析，先通过此法解析器将SQL拆分为一个个不可再分的单词，再使用语法解析器对SQL继续理解，并最终提炼出解析上下文，解析上下文包括表，选择项，排序项，分组项，聚合函数，分页信息，查询条件以及可能需要修改的占位符的标记

>执行器优化

合并和优化分片条件，如OR等

> SQL路由

根据解析上下文匹配用户配置的分片策略，并生成路由路径。目前支持分片路由和广播路由。

>SQL改写

将SQL改写为在真实数据库中可以正确执行的语句。SQL改写分为正确性改写和优化改写。

>SQL执行

通过多线程执行器异步执行

>结果归并

将多个结果集归并便于通过统一的JDBC接口输出。结果归并包括流式归并，内存归并和使用装饰者模式的追加归并这几种方式。



# 2、Sharding-JDBC准备 - MySQL完成主从复制

3308接口的  mysql -uroot -p -h 127.0.0.1 --socket=../tmp/mysql.sock --port=3308

查看进程 ps -aux|grep mysql

![image-20211212180311800](C:\Users\HQ\AppData\Roaming\Typora\typora-user-images\image-20211212180311800.png)





# MySQL主从复制环境安装

博客链接：https://blog.csdn.net/qq_40562912/article/details/100122792

安装第一台，一般用到一个机器安装第二台的时候，一般机器上已经有一台mysql了，所以，如果你没有可以参考下面链接安装,链接没有指定mysql配置文件地址，因为一般大家都是这样子装的，所以，为了能同样流程在正式机器安装成功，所以我没有指定配置文件，保证测试机和正式一样的环境

linux安装 mysql8.0

------------------------------------------------------------------现在安装第二个mysql----------------------------------------------------------------------

一定要先先看一下当前系统版本再下载对应的包，我开始没看，然后就一堆麻烦：

cat /proc/version
Linux version 3.10.0-862.14.4.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC) ) #1 SMP Wed Sep 26 15:12:11 UTC 2018

64位就下载对应64位

**1.先下载** 

可以去官网直接下载，

mysql-8.0.17-linux-glibc2.12-x86_64.tar.xz下载链接

我是直接从官网下载的，因为我安装的服务器之前已经安装了一个mariadb了，所以我现在使用另外一个端口3308

将下载的文件移到/usr/local 目录

**切换到当前目录**

> cd /usr/local
>
> tar -xvf mysql-8.0.17-linux-glibc2.12-x86_64.tar.xz

开始用的 tar -zxvf mysql-8.0.17-linux-glibc2.12-x86_64.tar.xz,查资料说这个压缩包没有用gzip格式压缩，所以不用加z参数

ls,查看文件夹 

然后重命名一下解压后的文件：

>mv mysql-8.0.17-linux-glibc2.12-x86_64 mysql3308

![img](https://img-blog.csdnimg.cn/20190911152348868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTYyOTEy,size_16,color_FFFFFF,t_70)

新建data目录,用于存放数据

>cd mysql3308/
>
>#新建mysql用户、mysql用户组,如果之前已经建立过，就不用建立了
>
>groupadd mysql
>
>#给mysql添加用户 为 mysql
>
>useradd mysql -g mysql
>
>#将/usr/local/mysql的所有者及所属组改为mysql
>
>chown -R mysql:mysql /usr/local/mysql3308/
>
>chmod -R 755 /usr/local/mysql3308/

配置参数

在mysql3308下新增文件夹data,var,etc备用，etc是用来放配置文件的

ls 查看一下当前文件目录

将没有的文件夹添加一下

>mkdir data;
>
>mkdir var
>
>mkdir etc

 ![img](https://img-blog.csdnimg.cn/20190916135311764.png)



 创建数据库配置文件，一般为my.cnf 。其实在根目录  /etc/my.cnf 有这个文件，所以只需要复制就行了，如果根目录下没有的话，就需要使用touch命令创建新的空文件

>ls /etc
>
>cp /etc/my.cnf etc/
>
>ls etc/

编辑刚刚复制的 my.cnf

>vi etc/my.cnf 
>
>[mysqld]
>basedir = /usr/local/mysql3308
>datadir = /usr/local/mysql3308/data
>socket = /usr/local/mysql3308/tmp/mysql.sock
>port = 3308
>
>[client]
>socket = /usr/local/mysql3308/tmp/mysql.sock
>default-character-set=utf8
>
>[mysqld_safe]
>log-error=/usr/local/mysql3308/mysql-log/error.log
>pid-file=/usr/local/mysql3308/mysql.pid

#如果/usr/local/mysql3308/目录下没有tmp文件，手动创建，并且配置权限：

>mkdir tmp
>chmod 777 ./tmp
>
>mkdir mysql-log
>
>chmod 777 ./mysql-log
>
>cd mysql-log
>
>touch error.log

![img](https://img-blog.csdnimg.cn/2019091710500252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTYyOTEy,size_16,color_FFFFFF,t_70)

安装依赖包--如果已经安装过就不要执行了，虽然执行也没啥问题，会再次检查一遍

>yum -y install make gcc-c++ cmake bison-devel ncurses ncurses-devel libaio-devel

![img](https://img-blog.csdnimg.cn/20190911144336774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTYyOTEy,size_16,color_FFFFFF,t_70)

指定刚刚编写的配置文件初始化：a/NRxfzh;87p

要在mysql3308 文件夹下面执行：

>cd /usr/local/mysql3308
>
>./bin/mysqld --defaults-file=/usr/local/mysql3308/etc/my.cnf --initialize --user=mysql

#这个命令和mysql5.7之前的命令不一样了，之前命令是：bin/mysql_install_db --user=mysql，但是之后的版本已经被#mysqld --initialize替代

#mysql5.7版本之上会初始化话一个密码，在这里要记住这个初始化密码 Y_Bhf!hjo0k0，在下面初次登录会用上。 

>2019-09-17T03:01:00.156712Z 0 [System] [MY-013169] [Server] /usr/local/mysql3308/bin/mysqld (mysqld 8.0.17) initializing of server in progress as process 1554
>2019-09-17T03:01:05.651404Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: Y_Bhf!hjo0k0
>2019-09-17T03:01:07.114536Z 0 [System] [MY-013170] [Server] /usr/local/mysql3308/bin/mysqld (mysqld 8.0.17) initializing of server has completed

./bin/mysqld_safe --defaults-file=/usr/local/mysql3308/etc/my.cnf --user=mysql &

#打开当前mysql的启动文件

>一定要确保这里改正确，否则会报错：Starting MySQL....The server quit without updating PID file[FAILED]ocal/mysql3308/data/iZ2zeborh4vm8ozw6k3j07Z.pid).
>
>vi support-files/mysql.server 
>
>指定地址和配置文件的位置：
>
>改1：并删掉下面的conf=/etc/my.cnf
>
>basedir=/usr/local/mysql3308
>datadir=/usr/local/mysql3308/data
>conf=/usr/local/mysql3308/etc/my.cnf

![img](https://img-blog.csdnimg.cn/20190917132541542.png)

> 改2：加 extra_args="-c $conf"

![img](https://img-blog.csdnimg.cn/20190917132726789.png)

> 改3 --增加绿色部分

> $bindir/mysqld_safe --defaults-file="$conf" --user=root --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null &

 ![img](https://img-blog.csdnimg.cn/20190917132823425.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTYyOTEy,size_16,color_FFFFFF,t_70)

 

> support-files/mysql.server start

 ![img](https://img-blog.csdnimg.cn/20190917133020475.png)

> 查看是否已经启动起来 

netstat -ntlp

![img](https://img-blog.csdnimg.cn/20190916141153985.png)

如图，3308，已经启动起来了

#将3308 mysql加入服务

> cp /usr/local/mysql3308/support-files/mysql.server /etc/init.d/mysql3308

#开机自启

> chkconfig --add mysql3308
>
> 显示服务列表
>
> chkconfig --list
>
> #如果3,4,5都是开的就说明是自启设置成功



 ![img](https://img-blog.csdnimg.cn/20190917133245148.png)

> 重启数据库的命令：/etc/init.d/mysql3308 restart

访问mysql:

> 第二个数据库必须使用socket进入，否则默认为第一个数据库。因为配置了全局环境变量
>
> cd bin/
>
> mysql -uroot -p -h 127.0.0.1 --socket=../tmp/mysql.sock --port=3308
>
> 输入初始化的密码：a/NRxfzh;87p
>
> alter user 'root'@'localhost' identified by '111111';
>
> flush privileges;

![img](https://img-blog.csdnimg.cn/20190916144445760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTYyOTEy,size_16,color_FFFFFF,t_70)

远程连接用户设置：

> mysql -uroot -p -h 127.0.0.1 --socket=../tmp/mysql.sock --port=3308
> use mysql;
> select 'host' from user where user='root';
> update user set host = '%' where user ='root';
> flush privileges;
> select 'host' from user where user='root';

然后将阿里云安全组防火墙3308 ，打开，使用navcat远程连接

![img](https://img-blog.csdnimg.cn/20190916145041295.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTYyOTEy,size_16,color_FFFFFF,t_70)

如上图所示，即为安装cheng

 ![img](https://img-blog.csdnimg.cn/20190916145148600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTYyOTEy,size_16,color_FFFFFF,t_70)
————————————————
版权声明：本文为CSDN博主「暖花_」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_40562912/article/details/100122792



配置主从复制

博客链接：https://blog.csdn.net/dmy_521/article/details/46043467

2.1主服务器操作：

> [root@master dingmingyi]# cat /usr/local/mysql-m/etc/my.cnf
>
> 
>
> [mysqld]
>
> basedir=/usr/local/mysql-m
>
> datadir=/opt/database-m
>
> socket=/var/run/mysql-m/mysql-m.sock
>
> pid-file=/var/run/mysql-m/mysql-m.pid
>
> port=3307
>
> user=mysql
>
> server-id=1                          
>
> log-bin=mysql-bin
>
> binlog_ignore_db=mysql
>
> character_set_server=utf8
>
> [mysqld_safe]
>
> log-error=/var/log/mysql-m/mysql--error.log
>
> [mysql]
>
> socket = /var/run/mysql-m/mysql-m.sock
>
> default-character-set=utf8 

只要在my.cnf里面添加内容都要重新启动服务#     **service mysql-m restart**

```linux 
[root@master dingmingyi]#/usr/local/mysql-m/bin/mysqladmin -uroot password '321321'  -S /var/run/mysql-m/mysql-m.sock               #为M-mysql设置密码；

[root@master dingmingyi]# /usr/local/mysql-m/bin/mysql-uroot -p321321 -S /var/run/mysql-m/mysql-m.sock
```

 

> 授权给从服务器

```sql
mysql> grant replication slave on *.* tos1@localhost identified by '123123';

mysql> grant replication slave on *.* tos2@localhost identified by '123123';

mysql>flush privileges;
```

 

> 查询主数据库状态

```sql
mysql>  show master status;

+------------------+----------+--------------+------------------+-------------------+

| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |

+------------------+----------+--------------+------------------+-------------------+

| mysql-bin.000001 |      670 |              | mysql            |                   |

+------------------+----------+--------------+------------------+-------------------+

1 row in set (0.00 sec)
```

记录file和position的值，在配置从服务器时需要用到

 

**2.2从服务器操作**

修改从服务器配置文件/usr/local/mysql-s1/etc/my.cnf

将server-id=10 ，确保此id与主服务器不同                                                     #设置多个从服务器必须与M-S和S-S都不同

同样在[mysql]下增加default-character-set=utf8 ，在[mysqld]下增加character_set_server=utf8

重启服务

 

登陆从服务器

```linux
/usr/local/mysql-s1/bin/mysql -uroot -p -S/var/run/mysql-s1/mysql-s1.sock             
#由于没设密码，可以直接回车进入

```

 

执行同步语句

```sql
change master to

master_host='localhost',

 master_user='s1',

master_password='123123',

master_log_file='mysql-bin.000001',

master_log_pos=670,

master_port=3307;         #一定要指定相应的端口号，不然在查看状态时会出错

start slave;

mysql> show slave status\G
```



*************************** 1. row ***************************

                  Slave_IO_State: Waiting for master to send event
    
                       Master_Host: localhost
    
                       Master_User: s1
    
                        Master_Port: 3307
    
                   Connect_Retry: 60
    
                Master_Log_File: mysql-bin.000001
    
    Read_Master_Log_Pos: 670
    
                     Relay_Log_File: mysql-s1-relay-bin.000002
    
                     Relay_Log_Pos: 283
    
    Relay_Master_Log_File: mysql-bin.000001
    
               Slave_IO_Running: Yes            #以下两排都要为YES才表明状态正常
    
            Slave_SQL_Running: Yes
    
                 Replicate_Do_DB:          

   …………………………..省略若干…………………………………

 

另一从服务器也做类似以上操作。
————————————————
版权声明：本文为CSDN博主「dmy_521」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/dmy_521/article/details/46043467



# 3、Mysql分库分表原理



## 3.1 为什么要分库分表

>分库分表目的：解决高并发，和数据量的问题。

1. 高并发的情况，会造成IO读写频繁，自然造成读写缓慢，甚至宕机。一般单库不要超过2k并发，NB的机器除外。

2. 数据量大得问题，主要是由于底层索引实现导致，MYSQL的索引实现为B+TREE，数据量大，会导致索引树始分庞大，造成查询缓慢，第二，innodb的最大存储限制是64TB。

>要解决上述问题，最常见的做法就是分库分表。
>
>分库分表的目的，是将一个表拆成N个表，就是让每个表的数据量控制在一定的范围内，保证SQL的性能。一个表的数据建议不要超过500W。

## 3.2 分库分表

>又分为垂直拆分和水平拆分

>水平拆分：统一个表的数据拆到不同的表中，可以根据时间、地区、或某个业务维度，也可以通过hash进行拆分，最后通过路由访问到具体的数据，拆分后的每一个表结构保持一致。**数据的离散存储**

>垂直拆分：就是把一个有很多字段的表给拆分成多个表，或者是多个库上去，每个库表的结构不一致，每个库表都包含部分字段。一般来说，可以根据业务维度进行拆分，如订单表可以拆分为订单、订单支持、订单地址、订单商品、订单扩展等；也可以，根据数据冷热程度拆分，20%的热点字段拆到一个表，80%的冷字段拆到另外一个表。

![image-20211215145427323](C:\Users\hspcadmin\AppData\Roaming\Typora\typora-user-images\image-20211215145427323.png)

![image-20211215145357509](C:\Users\hspcadmin\AppData\Roaming\Typora\typora-user-images\image-20211215145357509.png)

## 3.3 不停机分库分表数据迁移

>一般数据库的拆分也是有一个过程的，一开始是单表，后面慢慢拆成多表，那么我们久看下如何平滑的从MySQL单表过度到MySQL分库分表架构。
>
>1. 利用mysql+canal做增量数据同步，利用分库分表中间件，将数据路由到对应的新表；
>2. 利用分库分表中间件，全量数据导入到对应的新表中；
>3. 通过单表数据和分库分表数据两两比较，更新不匹配的数据到新表中；
>4. 数据稳定后，将单表的配置切换到分库分表配置上。

![image-20211215150134439](C:\Users\hspcadmin\AppData\Roaming\Typora\typora-user-images\image-20211215150134439.png)

## 3.4 小结

>垂直拆分：业务模块拆分、商品库，用户库，订单库
>
>水平拆分：对表进行水平拆分（也就是我们说的：分表）
>
>表进行垂直拆分：表的字段过多，字段使用的频率不一。（可以拆分两个表建立1:1关系）



# 4、shardingjdbc的分库和分表

> ## 4.1 分库分表的方式

**水平拆分：**统一个表的数据拆到不同的表中，可以根据时间、地区、或某个业务维度，也可以通过hash进行拆分，最后通过路由访问到具体的数据，拆分后的每一个表结构保持一致。**数据的离散存储**

**垂直拆分：**就是把一个有很多字段的表给拆分成多个表，或者是多个库上去，每个库表的结构不一致，每个库表都包含部分字段。一般来说，可以根据业务维度进行拆分，如订单表可以拆分为订单、订单支持、订单地址、订单商品、订单扩展等；也可以，根据数据冷热程度拆分，20%的热点字段拆到一个表，80%的冷字段拆到另外一个表。



>## 4.2 逻辑表

逻辑表是指：水平拆分的数据库或者数据表的相同路基和数据结构表的总数。比如用户表数据用户id%2分为两个表，分别是：ksd_user和ksd_user1，他们的逻辑表名为：ksd_user。

在shardingjdbc的定义方式如下：

```java
spring:
  shardingsphere:
	sharding:
	  tables:
		#ksd_user 逻辑表名
        ksd_user:
```



>## 4.3 分库分表数据节点 actual-data-nodes

```yml
tables:
        user:
          #数据节点：多数据源$->{0...N}，逻辑表名$->{0...N} 相同表
          actual-data-nodes: ds$->{0,2},user$->{0...1}
          #数据节点：多数据源$->{0...N}，逻辑表名$->{0...N} 不同表
          actual-data-nodes: ds0.user$->{0..1},ds1.user$->{2..4}
          #指定单数据源的配置方式
          actual-data-nodes: ds0.user$->{0..4}
          #全部手动指定
          actual-data-nodes:ds0.user0,ds1.user0,ds0.user,ds0.user1,ds1.user1
```

数据分片是最小单元。由数据源名称和数据表组成，比如：ds0.user0

寻找规则如下：

![image-20211215154708263](C:\Users\hspcadmin\AppData\Roaming\Typora\typora-user-images\image-20211215154708263.png)



>## 4.4 分库分表5种分片策略

![image-20211215154743293](C:\Users\hspcadmin\AppData\Roaming\Typora\typora-user-images\image-20211215154743293.png)

> 数据源分片分为两种：

- 数据源分片
- 表分片

>这两个是不同维度的分片规则，但是他们能使用的分片策略和规则是一样的。他们由两部分构成：
>
>- 分片键
>- 分片算法

> 第一种：none

对应的NoneShardingStragey不分片策略，SQL会被发给所有节点去执行，这个规则没有子项目可以配置。

>第二种：Inline行表达时分片策略（核心、必须要掌握）

对应InlineShardingStragey，使用Grovy的表达时，提供对SQL语句中的=和in的分片操作支持，只支持单分片键，对于简单的分片算法，可以通过简单的配置使用，从而避免频繁的java代码开放，如user${分片键（数据表字段）user_id%5}表示user根据某字段（user_id）模5从而分为5张表，表名称user0到user4，库也是这样的。

