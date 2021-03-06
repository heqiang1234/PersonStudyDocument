# 1.初始Msql

javaEE：企业级java开发 Web

前端（页面：展示，数据！）

后台 (连接点：连接数据库JDBC，链接前端（控制，控制试图跳转，和给前端传输数据)

数据库 （存数据）

> 只会写代码，学好数据可，基本混饭吃
>
> 操作系统，数据结构与算法，当一个不错的程序员
>
> 离散数学，体系挤乳沟，编译原理， +实战经验

## 1.1 为什么学习数据库

1.岗位需求

2.现在的世界是大数据时代-的数据者得天下

3.==被迫需求==，存数据

==4.数据库是所有软件体系中最重要得存在== DBA



## 1.2 什么是数据库

数据库 （DB,DataBase)

概念：数据仓库，软件，安装在操作系统之上

作用：存储数据，管理数据



## 1.3 数据库分类

### 1.3.1 关系型数据库

-  Mysql, oracle,sql server, DB2, SQLlite

- 通过表和表之间，行列之间得关系进行数据的存储



### 1.3.2 非关系型数据库

- (NoSQL) Not only

- Redis, MongDB
- 非关系型数据库，对象存储，通过对象得自身得属性来决定



### 1.3.3==DBMS(数据库管理系统)==

- 数据库的管理软件，科学有效的管理我们的数据，维护和获取数据
- MySQL



## 1.4 MySql简介

MySql是一个**关系型数据库管理系统**

前世：瑞典MySql AB公司

今生: 属于Oracle公司

MySQL是最好的RDBMS(Relational Database Mangement System,关系型数据管理系统)应用软件之一。

开源的数据库软件

中小型网站、或者大型网站、集群。



5.7稳定

8.0 数据库驱动



安装建议:

1.尽量不要使用exe,会写入注册表 

2.尽可能使用压缩包安装



## 1.5安装MySQL

教程：https://cnblogs.com/hellokuangshen/p/10242958.html

1.解压

2.放置环境目录

3.配置环境变量

4.新建ini文件，配置了数据路径和安装路径

```ini
[mysqld]
# 目录为mysql本地安装目录
basedir=F:\Enviroment\mysql-5.7.19-winx64\
datadir=F:\Enviroment\mysql-5.7.19-winx64\data\
port=3306
skip-grant-tables
```



```sql
#安装mysql服务

mysql --install

#初始化数据库文件

mysqld --initialize-insecure --user=mysql

#启动mysq服务

net start mysql

#进入mysql管理界面

mysql -u root -p

#设置mysql用户密码

update mysql.user set authentication_string=password('123456') where user='root' and  Host = 'localhost'

flush privileges; #刷新权限

#关闭服务

net stop mysql

#清空服务

sc delete mysql


--分号结尾
show databases;

use school --切换数据库

show tables; --查看数据库中的所有表
describe student; --显示数据库所有表的信息
create database westor; --创建数据库
```



## 1.6 安装sqlyog

使用navicat

## 1.7 连接数据库

```sql
#设置mysql用户密码

update mysql.user set authentication_string=password('123456') where user='root' and  Host = 'localhost'

flush privileges; #刷新权限

#关闭服务

net stop mysql

#清空服务

sc delete mysql


--分号结尾
show databases;

use school --切换数据库

show tables; --查看数据库中的所有表
describe student; --显示数据库所有表的信息
create database westor; --创建数据库
```

数据库xxx语言 CRUD 增删查改

DDL 定义

DML 操作

DQL 查询

DCL 控制







# 2.操作数据库

## 2.1 数据库的列类型

> 数值

- tinyint 	十分小的数据 		  1个字节

- smallint     较小的数据 		  2个字节

- mediumint		  中等大小的数据 		  3个字节

- **int 		  标准的整数		    4个字节**

- bigint  较大的数据  		   		  8个字节

- float 		  浮点数		    4个字节

- double 浮点数		   8个字节 		  （精度问题！）

- decimal 		  字符串形式的浮点数 		  金融计算的时候，一般使用decimal

  

>字符串

- char   字符串   固定大小   0 - 255

- varchar   可变字符串  0 - 65535    常用的string里面
- tinytext   微型文本   2*8 -1
- text   文本串   2*16 -1    保存大文本



>时间日期

java.util.Date

- date  YYYY-MM-DD,日期格式
- time  HH:mm:ss 时间格式
- **datetime YYYY-MM-DD HH：mm: ss 最常用的时间格式**
- **timestamp 时间戳， 1970，1，1到现在的毫秒数！**
- year 年份表示



>null

- 没有值，未知
- ==注意，不要使用null进行运算，结果为null





## 2.2 数据库的字段属性

==unsigned==:

- 无符号的整数
- 声明了该列不能声明为负数



==zerofill==

- 0填充
- 不足的位数，使用0填充，int(3)  5   005



==自增：==

- 通常理解为自增，自动在上一条记录的基础上 +1（默认）
- 通常用来涉及唯一的主键 index,必须是整数类型
- 可以自定一设计主键自增的起始值和步长



==非空 NULL not null==

- 假设设置为 not null 如果不给它赋值，就会报错
- null,如果不填写值，默认就是null



==默认：==

- 设置默认的值
- sex,默认值为男，如果不指定该列的值，则会有默认的值





>每一个表，都必须存在以下5个字段！未来做项目，表示一个记录存在的意思
>
>id 主键
>
>`version`  乐观锁
>
>is_delete   伪删除
>
>gmt_create  创建时间
>
>gmt_update 修改时间

## 2.3 数据库的类型

```sql
show create database school --查看创建数据库的语句
show create table student --查看student数据表的定义语句
DESC student --显示表的结构
--关于数据库引擎
INNODB 默认使用
MYISAM 早些年使用
```

|              | MYISAM | INNODB        |
| ------------ | ------ | ------------- |
| 事务支持     | 不支持 | 支持          |
| 数据行锁定   | 不支持 | 支持          |
| 外键约束     | 不支持 | 支持          |
| 全文索引     | 支持   | 不支持        |
| 表空间的大小 | 较小   | 较大，约为2倍 |

常规使用操作：

- MYISAM 节约空间，速度较快
- INNODB 安全性高，事务的处理，多表多用户处理

>在物理空间存在的空间

所有的数据库文件都存在data目录下，一个文件夹对应一个数据库

本质还是文件的存储！

MySQL引擎在物理文件上的区别

- InnoDB在数据库表中只有一个*.frm文件，以及上级目录下的ibdata1文件
- MYISAM对应文件
  - *.frm  表结构的定义文件
  - *.MYD  数据文件（data）
  - *.MYI   索引文件（index）



>设置数据库表的字符集编码

```sql
charset = uft-8
```

不设置的话，会是mysql默认字符集编码（不支持中文！）

MySql的默认编码是Latin1，不支持中文

在my.ini中设置默认的编码

```sql
character-set-server=utf-8
```

## 2.4 表的删除修改

>--  修改表的字段 （重命名，修改约束！）
>
>ALTER TABLE teacher1（表名） MODIFY age（字段名）属性 VARCHAR(11) -- 修改约束
>
>ALTER TABLE teacher1 CHANGE age age1 INT(1) -- 字段重命名
>
>change用来字段的重命名，不能修改字段类型和约束
>
>modify不用来字段重命名，只能修改字段类型和约束

所有的创建和删除操作尽量加上判断，以免报错~

注意点

- ``字段名，使用这个包裹！
- 注释 .. /**/
- sql关键字大小写不敏感，建议大家写小写
- 所有的符号全部用英文

# 3.MySQL数据管理

## 3.1 外键（了解即可）

>创建表的时候  CONSTRAINT  '外键名 ' FOREIGN ('字段名') REFERENCES '字段名'

最佳实践

- 数据库就是单纯的表，只用来存数据，只有行（数据）和列（字段）
- 我们想使用多张表的数据，想使用外键（程序去实现）

## 3.2 DML语言（全部记住）

### 3.2.1数据库的意义：

数据存储，数据管理

DML语言：数据操作语言

- Insert
- update 
- delete

### 3.2.2增加insert：

```sql
#语法:
insert into 表名（字段1，字段2、、、） values(值1、值2、、、)；
```



### 3.2.3修改update

```sql
update 表名 set colnum_name  = value where 条件
```

### 3.2.4删除delete

```sql
#delete命令
#语法
delete from 表名 [where 条件]
```

>TRUNCATE命令
>
>作用：完全清空一个数据库表，表的结构和索引约束不会变！



>delete和truncate的区别

- 相同点：都能删除数据，都不会删除表结构
- 不同：
  - TRUNCATE 重新设置 自增列 计数器会归零
  - TRUNVATE 不会影响事务

了解即可： DELETE删除的问题，重启数据库，现象:

- InnoDB 自增列会吃从1开始 （存在内存当中，断电即失）--mysql 新版本已经修复
- MyISAM 继续从上一个自增量开始（存在文件中，不会丢失）

# 4.DQL查询

## 4.1 联表查询

(data Query language ： 数据库查询语言)

>查询系统版本
>
>select version

>查询自增
>
>select @auto_increment_increment



左连接，自连接，右连接

```sql
left join 
inner join 
right join

select s.studentno,s.studentname,studentresult from student s left JOIN result  r
on s.studentno = r.studentno;

inner join 可以用 on,      where 只能在内联接里面使用，
left join和 right join 指的基准表不同，  以上例子解释 left join   以 student 为基准。 

FROM a left join b
FROM a right join b
```



| 操作       | 描述                                       |
| ---------- | :----------------------------------------- |
| inner join | 如果表中至少有一个匹配，就返回行           |
| left join  | 会从左表中返回所有的值，即使右表中没有匹配 |
| right join | 会从右表中返回所有的值，即使左表中没有匹配 |

![image-20211205175325336](C:\Users\HQ\AppData\Roaming\Typora\typora-user-images\image-20211205175325336.png)

## 4.2 分页和排序

> 分页 limit   排序 order by

>排序  升序 ASC、降序DESC
>
>第N页    （n - 1）* pageSize
>
>--【pageSize：页面大小】
>
>--【（n - 1）* pageSize：起始值】
>
>--【n：当前页】
>
>--【数据总数/页面大小 = 总页数】



## 4.3 查询语句

![image-20211205200644369](C:\Users\HQ\AppData\Roaming\Typora\typora-user-images\image-20211205200644369.png)



# 5.数据库函数

>略

# 6.事务

## 6.1 什么是事务

==要么都成功，要么都失败==

>1. SQL执行  A给B转账  A 1000 -->200   B 200
>2. SQL执行 B收到A的钱 A800 --> B 400

将一组SQL放在一个批次中去执行

> 事务原则：ACID原则 原子性，一致性，隔离性，持久性    （脏读，幻读）

博客链接：https://blog.csdn.net/dengjili/article/details/82468576

知乎链接：https://zhuanlan.zhihu.com/p/98465611



## 6.2 事务管理（ACID）

谈到事务一般都是以下四点

- **原子性（Atomicity）**
  原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。
- **一致性（Consistency）**
  事务前后数据的完整性必须保持一致。
- **隔离性（Isolation）**
  事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。
- **持久性（Durability）**
  持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来即使数据库发生故障也不应该对其有任何影响

>隔离导致的问题

**脏读：**

指一个事务读取了另一个事务没有提交的数据。

**不可重复读：**

在一个事务内读取表中的某一行数据，多次读取结果不同。（这个不一定是错误，只是某些场合不对）

**虚读（幻读）：**

是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。

## 6.3 事务隔离级别

原文链接：https://developer.aliyun.com/article/743691

 ***<u>重点：</u>*** ==**原文链接：**==  https://cloud.tencent.com/developer/article/1614863

>mysql的事务该列级别有四个。分别是读未提交，读已提交，可重复读和串行化。

> 读未提交：

   事务A读取到事务B修改但是没有提交的数据，这里的数据不一定是最终数据，因为事务B可能发生回滚，这时，事务A读取到的数据就是脏数据，所以这一个隔离级别可能会发生脏读，幻读，不可重复读的问题。总体来说，几乎相当于没有事务隔离



> 读已提交

假设事务A发生数据的修改，此时事务B发生查询这条数据，那么此时的查询不能执行，只有在事务A提交修改后，事务B的查询会成功。这里可以避免脏读的问题。（脏读是由于读取到未提交的数据）



> 可重复读

事务B只有在事务A修改并且事务提交后，事务B自身也提交事务后，才能查询到事务A修改的数据。这里避免了不可重复读的问题。不可重复读（在一个事务查询的时候，另一个事务在修改，然后这事务再次查询，查询到的是另一个事务修改后的值，和之前查询到的不同）



DBTRXID 为什么是6个字节？

undo日志在事务提交后。会怎样处理？

- 存储undo日志的地方，就是**回滚段**。
- 多个事务并行操作某一行数据时，不同事务对该行数据的修改会产生多个版本，然后通过回滚指针（DBROLLPTR）连一条**Undo日志链**。



> 执行事务

```sql
#mysql默认开启事务自动提交
set autocommit = 0 /** 关闭 **/
set autocommit = 1 /** 开启（默认的） **/

#事务开启
start transaction --标记一个事务的开始，从这个之后的sql  都在同一个事务内

insert xx
insert xx

--提交：持久化 成功
COMMIT
--回滚：回到原来的样子 （失败）
ROLLBACK

--事务结束
set autocommit = 1  --开启自动提交

--了解
SAVEPOINT 保存节点 --设置一个事务的保存点
ROLLBACK TO SAVEPOINT 保存节点 --回滚到保存点
RELEASE SAVEPOINT 保存节点 --撤销保存点
```



```sql
#创建数据库
create database shop CHARACTER SET utf8 COLLATE utf8_general_ci

#新建表
USE shop

CREATE TABLE `account` (
	`id` int(3) not null AUTO_INCREMENT,
	`name` varchar(30) not null,
	`money` DECIMAL(9,2) not null,
	PRIMARY key (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8;

#插入数据
INSERT into account(`name`,`money`)
values ('A',2000.00),('B',10000.00);


#模拟转账：事务
SET autocommit = 0; #关闭自动提交
start TRANSACTION #开启一个事务

UPDATE account SET money=money-500 where `name` = 'A' #A减500
UPDATE account SET money=money+500 where `name` = 'B' #B加500

COMMIT; #提交事务，就被持久化了！
ROLLBACK; #回滚

set autocommit = 1; #恢复默认值

```



# 7、索引

>MySQL官方对索引的定义为：索引（Index）是帮助**MySQL高效获取数据的数据结构**。
>
>提取句子主干，就可以得到索引的本质：**索引是数据结构。**

## 7.1 索引分类

- 主键索引  （PRIMARY KEY）
  - 唯一的标识，主键不可重复，只能有一个列作为主键
- 唯一索引 （UNIQUE KEY）
  - 避免重复的列出现，唯一索引可以重复，多个列都可以标识位 唯一索引
- 常规索引 （KEY/INDEX）
  - 默认的，index.key关键字来设置
- 全文索引 （FullText）
  - 在特定的数据引擎下才有，MyISAM
  - 快速定位数据

```sql
#显示所有的索引信息
show index from student

#增加一个全文索引  (索引名) 列名
ALTER TABLE school.student add FULLTEXT INDEX `studentName`(`studentName`);

-- explain 分析sql执行的情况
EXPLAIN SELECT * from student;  #全文索引
```

## 7.2 测试索引

索引在小数据量的时候，区别不大，但是在大数据的时候，区别十分明显~

## 7.3、索引原则

- 索引不是越多越好
- 不要对进程变动数据加索引
- 小数据量的表不需要加索引
- 索引一般加载在常用查询的字段上！



>索引的数据结构

hash类型的索引

Btree：InnoDB 的默认数据结构

阅读：http://blog.codinglabs.org/articles/theory-of-mysql-index.html



# 8、权限管理和备份

为什么要备份：

- 保证重要的数据不丢失
- 数据转移

MYSQL数据库备份的方式

- 直接拷贝物理文件
- 可视化工具（navicat）直接导出数据和结构，落地保存
- 使用命令行导出 mysqldump 命令行使用

```sql
mysqldump -localhost -uroot -p123456 school student >D:/s.sql

source d:/a.sql
```

# 9、规范数据库设计

## 9.1、为什么需要设计

当数据库比较复杂的时候，我们就需要设计

糟糕的数据库设计：

- 数据冗余，浪费空间
- 数据库的插入和删除都会麻烦，异常【屏蔽使用物理外键】
- 程序的性能差

良好的数据库设计：

- 节省内存空间
- 保证数据库的完整性
- 方便我们开发系统

软件开发中，关于数据库的设计

- 分析需求：分析业务和需求处理的数据库的需求
- 概要设计：设计关系图 E-R图

设计数据库的步骤：（个人博客）

- 收集信息，分析需求
  - 用户表（用户登录注销，用户的个人信息，写博客，创建分类）
  - 分类表（文章分类，谁创建的）
  - 文章表（文章信息）
  - 友链接表（友链信息）
  - 自定义表（系统信息，某个关键字段，或者一些主字段）key：value
  - 说说表
- 标识实体（把需求落地到每个字段）
- 标识实体之间的关系
  - 写博客： user-->blog
  - 创建分类：user -->category
  - 关注：user -->user
  - 友链：links
  - 评论：user-user-blog

## 9.2 三大范式

>为什么需要数据规范化

- 信息重复
- 更新异常
- 插入异常
  - 无法正常显示信息
- 删除异常
  - 丢失有效信息



>三大范式

博客链接：https://www.cnblogs.com/wsg25/p/9615100.html

**第一范式（1NF）**

原子性：保证每一列都是不可再分

**第二范式（2NF）**

前提：满足第一范式

每张表只描述一件事情

**第三范式（3NF）**

前提：满足第一范式，第二范式

第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关



（规范数据库的设计）



规范性和性能的问题

阿里规范：关联查询的表不得超过三张表

- 考虑商业化的需求和目标，（成本，用户体验），数据库的性能更加重要
- 在规范性能的问题的时候，需要适当的考虑以下规范性
- 故意给某些表增加一些冗余的字段。（从多表查询中变为单表查询）
- 故意增加一些计算列（从大数据量降低为小数据量的查询：索引）



# 10、JDBC（重点）

>JDBC例子

```sql
   //1.加载驱动
   Class.forName("com.mysql.jdbc.Driver");//加载qud
   //2. 用户信息和url
   String url = "jdbc:mysql://localhost:3306/jdbcstudyuseUnicode=true&&characterEncoding=utf8&&useSSL=true";
   String userName = "root";
   String passWord = "123456";
   //3. 连接成功，数据库对象
   Connection connection = DriverManager.getConnection(url, userName, passWord);
   //4. 执行sql的对象
   Statement statement = connection.createStatement();
   //5. 执行SQL的对象 去执行SQL
   String sql = "select * from users";
   ResultSet resultSet = statement.executeQuery(sql);
   //返回的结果集
   while (resultSet.next()) {
        System.out.println("id = "+ resultSet.getObject("id"));
        System.out.println("NAME = "+ resultSet.getObject("NAME"));
        System.out.println("PASSWORD = "+ resultSet.getObject("PASSWORD"));
        System.out.println("email = "+ resultSet.getObject("email"));
        System.out.println("birth = "+ resultSet.getObject("birthday"));
    }
    //6. 释放连接
    resultSet.close();
    connection.close();
    statement.close();
```



>8.0mysql版本以上，需要加加时区：serverTimezone=UTC



>DriverManager

```java
// DriverManager.registerDriver(new com.mysql.jdbc.Driver());
        Class.forName("com.mysql.jdbc.Driver");//加载驱动
// com.mysql.jdbc.Driver 有默认方法加载注册driver
Connection connection = DriverManager.getConnection(url, userName, passWord);

// connection 代表数据库
// 数据库设置自动提交
// 事务提交
// 事务回滚
connection.rollback();
connection.commit();
connection.setAutoCommit();
```



>url

```java
 String url = "jdbc:mysql://localhost:3306/jdbcstudy?useUnicode=true&&characterEncoding=utf8&&useSSL=true";
 
 // mysql 3306
 // jdbc:mysql://主机地址：端口号/数据库名？参数1&&参数2
// 协议://主机地址：端口号/数据库名？参数1&&参数2
 
// oracle 1521
// jdbc:oracle:thin:@localhost:1521:sid
```



>statement执行SQL对象  PrepareStatement 执行SQL对象 

```java
String sql = "select * from users";

statement.executeQuery();//拆线呢操作返回 ResultSet
statement.execute(); //执行任何sql
statement.executeUpdate(); //更新、插入、删除。都是用这个，返回一个受影响的行数
```



>ResultSet查询的结果集：封装了所有的查询结果

获取指定的数据类型：

```java
 resultSet.getObject();
```



## 10.1 statement对象

==jdbc中的statement对象用于向数据库发送sql语句，想完成对数据库的增删查改，只需要通过这个对象向数据库发送增删查改语句即可==

Statement对象的executeUpdate方法，用于向数据库发送增删改的sql语句，executeUpdate执行完后，将会返回一个整数（即增删改语句导致了数据库几行数据发生了变化）

Statement.executeUpdate方法用于向数据库发送查询语句，executeUpdate方法返回代表查询结果的ResultSet对象



> CURD操作 create

使用executeUpdate(String sql)方法完成数据的添加操作，示例操作：

```java
Statement st = conn.createStatement();
String sql = "insert into user(...) values(...)";
int num = st.executeUpdate(sql);
if(num > 0) {
    System.out.println("插入成功！");
}
```



>CURD操作-delete

使用executeUpdate(String sql)方法完成数据的删除操作，示例操作：

```java
Statement st = conn.createStatement();
String sql = "delete from user where id = 1";
int num = st.executeUpdate(sql);
if(num > 0) {
    System.out.println("删除成功！");
}
```



>CURD操作-update

使用executeUpdate(String sql)方法完成数据的修改操作，示例操作：

```java
Statement st = conn.createStatement();
String sql = "update user set name = '' where name = ''";
int num = st.executeUpdate(sql);
if(num > 0) {
    System.out.println("修改成功！");
}
```



>CURD操作-select

使用executeQuery(String sql)方法完成数据的查询操作，示例操作：

```java
Statement st = conn.createStatement();
String sql = "select * from users";
Result rs = st.executeQuery(sql);
while(rs.next()){
    //根据获取列的数据类型，分别调用rs的相应方法映射到java对象中
}
```



> JdbcUtil工具类

```java
public class JdbcUtils {

    private static String driver = null;
    private static String url = null;
    private static String username = null;
    private static String password = null;

    static {

        try {
            InputStream in = JdbcUtils.class.getClassLoader().getResourceAsStream("db.properties");
            Properties properties = new Properties();
            properties.load(in);

            driver = properties.getProperty("driver");
            url = properties.getProperty("url");
            username = properties.getProperty("username");
            password = properties.getProperty("password");

            //1. 驱动只要加载一次
            Class.forName(driver);

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    //获取连接
    public static Connection getConnection() throws SQLException {
        return DriverManager.getConnection(url, username, password);
    }

    //释放连接资源
    public static void release(Connection conn, Statement st, ResultSet rs) {

        if(rs != null){
            try {
                rs.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }

        if(st != null){
            try {
                st.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }

        if(conn != null){
            try {
                conn.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
```



>测试jdbc

```java
public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            conn = JdbcUtils.getConnection();
            st = conn.createStatement();
            String sql = "INSERT INTO `users` VALUES (6, 'HQ', '123456', 'HQ@HUNDSUN.com', '1997-01-01');";
            int num = st.executeUpdate(sql);
            if(num > 0 )
            {
                System.out.println(num + "  插入成功");
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
```





>sql注入

注意拼接字符串就可以了



## 10.2 PrepareStatement对象

>prepareStatement  预编译SQL,先写sql，然后不执行
>
>PreparedStatement 防止SQL注入的本质，把传递进来的参数当作字符

```java
public static void main(String[] args) {
        Connection conn = null;
        PreparedStatement st = null;
        ResultSet rs = null;
        try {
            conn = JdbcUtils.getConnection();

            //区别
            //使用？占位符代替参数
            String sql = "INSERT INTO `users` VALUES (?, ?, ?, ?, ?);";
            // PreparedStatement 防止SQL注入的本质，把传递进来的参数当作字符
            // 假设其中存在转义字符，比如说 `  会被直接转义
            st = conn.prepareStatement(sql);//预编译SQL,先写sql，然后不执行

            //手动给参数赋值
            st.setInt(1,4);
            st.setString(2,"HQ12");
            st.setString(3,"123455");
            st.setString(4,"H123144@QQ.COM");
            //注意点：  sql.Date   数据库java.sql.Date()
            //          util.Date  Java    new Date().getTime()   获得时间戳
            st.setDate(5,new java.sql.Date(new Date().getTime()));

            int num = st.executeUpdate();
            if (num > 0) {
                System.out.println(num + "  插入成功");
            }
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        } finally {
            JdbcUtils.release(conn,st,null);
        }
    }
```



## 10.3 事务

==要么都成功，要么都失败==

> ACID原则

原子性： 要么全部完成，要么都不完成

一致性：总数不变

**隔离性：多个进程，互不干扰**

持久性：一旦提交不可逆，持久化到数据库



隔离性的问题：

脏读：一个事务读取了另一个没有提交的事务

不可重复读：在同一个事务内，重复读取表中的数据，表数据发生了改变

虚读（幻读）：在一个事务内，读取了别人插入的数据，导致前后读出来的结果不一致



## 10.4 数据库连接池

>数据库连接--执行完毕--释放--   
>
>连接--释放  十分浪费系统资源
>
>==池化技术：准备用一些预先的资源，过来就连接预先准备好的==

----开门--服务--关门

----开门--业务员：等待---关门

常用连接数 10个

最小连接数：10   **ps.根据常用连接数设置最小连接数**

最大连接数：100 业务最高承载上限 

超过连接数需要排队等待

等待超时时间：100ms



编写连接池，实现一个接口 DataSource



>开源数据源实现

DBCP

C3P0

Druid：阿里巴巴



使用这些数据库连接池之后，我们在项目开发中就不需要编写连接数据库代码



>DBCP

需要用到的jar包

commons-dbcp-1.4  commons-pool



>C3P0

需要用到的jar包

c3p0.jar

mchange-commons-java.jar



>结论

无论使用什么数据源，本质还是一样的，DataSource接口不变，方法不会变



>Druid



# x.1 重点字段

## 1、事务

尽管我们可以使用子查询（Sub-Queries）、连接（JOIN）和联合（UNION）来创建各种各样的查询，但不是所有的数据库操作都可以只用一条或少数几条SQL语句就可以完成的。更多的时候是需要用到一系列的语句来完成某种工作。但是在这种情况下，当这个语句块中的某一条语句运行出错的时候，整个语句块的操作就会变得不确定起来。设想一下，要把某个数据同时插入两个相关联的表中，可能会出现这样的情况：第一个表中成功更新后，数据库突然出现意外状况，造成第二个表中的操作没有完成，这样，就会造成数据的不完整，甚至会破坏数据库中的数据。要避免这种情况，就应该使用事务，它的作用是：要么语句块中每条语句都操作成功，要么都失败。换句话说，就是可以保持数据库中数据的一致性和完整性。事物以BEGIN关键字开始，COMMIT关键字结束。在这之间的一条SQL操作失败，那么，ROLLBACK命令就可以把数据库恢复到BEGIN开始之前的状态。

```sql
BEGIN; 
	INSERT INTO salesinfo SET CustomerID=14;
	UPDATE inventory SET Quantity=11 WHERE item='book';
COMMIT;
```

事务的另一个重要作用是当多个用户同时使用相同的数据源时，它可以利用锁定数据库的方法来为用户提供一种安全的访问方式，这样可以保证用户的操作不被其它的用户所干扰。

## 2、联合索引



## 3、索引底层

资源链接：https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzUyNjc5MzU3OA%3D%3D%26mid%3D2247484500%26idx%3D1%26sn%3D281631df6a83793b60a8c4824f17c0a0%26chksm%3Dfa082a51cd7fa347dd0c507e32e5e86ebf2e887ea8370e81fdf2983dba917ffe79fffac1f157%23rd

深入理解 Mysql 索引底层原理--知乎：https://zhuanlan.zhihu.com/p/113917726

### 3.1 索引的本质

>MySQL官方对索引的定义为：索引（Index）是帮助MySQL高效获取数据的数据结构。提取句子主干，就可以得到索引的本质：索引是数据结构。
>
>我们知道，数据库查询是数据库的最主要功能之一。我们都希望查询数据的速度能尽可能的快，因此数据库系统的设计者会从查询算法的角度进行优化。最基本的查询算法当然是顺序查找（linear search），这种复杂度为O(n)的算法在数据量很大时显然是糟糕的，好在计算机科学的发展提供了很多更优秀的查找算法，例如二分查找（binary search）、二叉树查找（binary tree search）等。如果稍微分析一下会发现，每种查找算法都只能应用于特定的数据结构之上，例如二分查找要求被检索数据有序，而二叉树查找只能应用于二叉查找树上，但是数据本身的组织结构不可能完全满足各种数据结构（例如，理论上不可能同时将两列都按顺序进行组织），所以，在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。

## 4、explain 分析sql执行的情况

文章链接：https://cloud.tencent.com/developer/article/1093229



# xx.注意事项

资源链接：https://zhuanlan.zhihu.com/p/162727511

1.【强制】数据库和表的字符集统一使用 utf8 或者 utf8mb4。

>不同字符集转化可能会产生乱码。
>
>不同字符集比较前会进行字符转换，索引失效。
>
>UTF8 每个字符占用3字节，占用空间小，但是不能存储 emoj，emoj 占用4字节。
>
>UTF8MB4 每个字符占用4字节，是真正的 UTF8，推荐使用。

2.【建议】数据库表考虑分库分表细节，推荐使用snowflake作为ID主键。

>单表设计存储数据少于 500 万条或单表容量超过 2G。
>
>不建议使用分区表，容易造成全表死锁，跨分区查询效率低。

3.【强制】表每一行中的每列数据大小相加不能大于 65535 byte。

4.【强制】不要设置预留字段，更改会锁表。https://www.jianshu.com/p/bfa00afb416c

5.【强制】不要保存文件等大的二进制数据。应放到文件服务器中。

6.【建议】InnoDB 字符集默认排序使用 _general_ci 和 _unicode_ci，推荐使用 _general_ci。

>ci不区分大小写
>
>cs区分大小写
>
>general速度更快，准确性稍低

7.【建议】表必备三个字段，id、create_time、modify_time。

>id：unsigned bigint。单表时主键单表时自增1，需要分表使用snowflake。
>
>create_time：datatime。创建时间。
>
>modify_time：datatime。修改数据更新时间。

8.【强制】不要使用触发器。

>高并发情况下不理想。
>
>可以用事务替代。

9.【强制】存储过程设计要合理，尽量少用。

>过度复制逻辑容易死锁。
>
>可以替换为在后端业务层或者脚本实现。

10.【强制】数据量大的表要使用pt工具修改表结构。

>pt-online-schema-change。
>
>原理是新建一张表并复制原表结构与数据，最终删除原表，可以有效避免行锁及表锁。
>
>字段设计规范

11.【强制】表字段表示是否概念，即is_xxx。

>1表示是，0表示否
>
>使用 unsigned tinyint

12.【强制】小数类型都使用decimal型。

>decimal精确。
>
>如果超出decimal范围建议分两个字段存储。

13.【强制】ip及手机号类固定长度字段，要用char。

14.【强制】选择合适的存储长度。

>可以减少表存储空间。
>
>可以减少索引长度，增加索引效率。

15.【建议】避免使用TEXT、BLOB数据类型。

>内存临时表不支持TEXT和BLOB。会使用磁盘临时表，降低查询速度。
>
>TEXT和BLOB需要单独成表，提高查询效率。

16.【建议】避免使用ENUM数据类型。

>枚举类型order by效率低。
>
>禁止使用数字作为枚举值。
>
>在与php使用上1和’1’差别大，PHP是弱引用很容易把’1’写为1，1为key，’1’为内容。

17.【建议】尽可能把列定义设置为 NOT NULL。

>索引NULL列，会额外增加开销，占用更多表空间。
>
>要做计算或者比较时，会对 NULL 做特别处理。
>
>在 SQL 中对 NULL 进行判断会全表扫描。

18.【强制】时间类型不要使用 varchar 或 int 等。

>使用 timestamp，占用4字节与int相同，查询计算比int快。
>
>timestamp 取值范围，1970-01-01 00:00:01 ~ 2038-01-19-03:14:07。
>
>使用 datatime，占用8字节，明确日期时间，超出timestamp用datatime。
>
>索引设计规范

19.【强制】不要使用外键和级联，应放在应用层去做。

>外键与级联更适合单机及低并发，不适合分布式和高并发集群。
>
>外键即外键约束，会影响写操作（插入速度），降低性能。
>
>级联更新是强阻塞，存在更新风暴的风险。

>降低性能的原因：往子表中插入一条数据，首先检查主表中是否有相应的主键值，锁定附表的记录，子表中插入值。多了两部操作，速度应该会慢一些。

20.【强制】一张表不要超过5个索引。

>索引过多会降低性能。
>
>合理分配索引会提高性能。

>### 索引劣势
>
>- 实际上索引也是一张表，该表保存了主键和索引字段，并指向实体表的记录,所以索引列也是要占用空间的
>- 虽然索引大大提高了查询速度，同时却会降低更新表的速度,如果对表INSERT,UPDATE和DELETE。因为更新表时，MySQL不仅要不存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息.
>- 索引只是提高效率的一个因素，如果你的MySQL有大数据量的表，就需要花时间研究建立优秀的索引，或优化查询语句

==以下未分析==

21.【强制】==联合索引==的左侧原则可以减少每个字段单独建立索引。

>避免每个字段都建立索引。
>
>联合索引区分度高的放在最左边。
>
>联合索引如果存在非等号和等号混合时，把等号的索引放在最左边。
>
>联合索引左侧原则，一定要注意顺序。

22.【强制】InnoDB 必须有主键。

>结合建表四个必要字段，id作为主键。
>
>InnoDB 属于索引组织表，逻辑顺序和索引的顺序相同。
>
>单表时主键自增1。
>
>预计三年内达到500万条，需要使用 snowflake 等分布式id生成主键。
>
>不要使用 uuid、hash、md5 等作为主键，要有顺序概念。

23.【建议】查询想走特定索引时可以用force index。

>MySQL的 optimizer 会执行它认为最优索引，但是往往不是我们需要或者最优的。
>
>使用 force index 可以强制使用索引，结合 explain 使用，确认为最优。

24.【强制】有唯一索引需求，该字段就应设置唯一索引。

>即使该字段是在联合索引内，也要单独设置唯一索引。
>
>唯一索引对insert速度影响可以忽略，但是提高查询速度和唯一性是明显的。
>
>应用层也建议做校验控制，但是根据墨菲定律，只要有可能就会出现脏数据。

25.【强制】varchar型设置索引要设置索引长度。

>不设置默认是全部长度。
>
>建议索引长度为20，区分度可以达到90%。
>
>区分度计算公式：select count(distinct left(列名, 索引长度))/count(*) FROM 表名。可以查出区分度百分比。

26.【强制】模糊查询最好用搜索引擎。

>禁止使用like %str和like %str%。因为不走索引。
>
>可以使用like str%。走索引。
>
>也可以走全文索引，但是需要看配置，还是推荐搜索引擎。

27.【强制】order by 需要注意索引的有序性。

>order by后接索引应该索引的一部分，如果是联合索引，应该是联合索引的最后，避免出现file_sort，影响查询性能。where a=? and b=? order by c那么索引是（a,b,c）。
>
>file_sort出现是没有走索引或者联合索引。出现情况：where a=? order by b索引是a。改进优化：where a=? order by b索引是（a,b）。

28.【强制】避免冗余索引。

>重复索引：primary key(id)、index(id)、unique index(id)。
>
>冗余索引：key(a,b,c)、key(a,b)、key(a)。

29.【强制】查询频率较高的sql语句，应该使用覆盖索引。

>覆盖索引不是真正的索引，是一种使用索引方式。
>
>原理是从索引中查询出想要内容，而不用回表查询，提高查询效率。
>
>表现是explain的extra为Using index。
>
>例如select user_no from user order user_age = 28索引为user_no时效率低，索引为（user_no,user_age）时为覆盖索引，查询效率高。

30.【强制】避免隐式类型转换。

>定义和使用不同数据会造成隐式转换。
>
>隐式转换会不走索引，降低查询效率。
>
>如select user_age from user where user_no='111'

31.【强制】避免在字段位置写表达式，不走索引。

>反例：`select user_no from user where user_age*2 = 36`。
>
>正例：`select user_no from user where user_age = 36/2`。
>
>查询优化

32.【强制】SQL性能优化目标，由高到低。

>const。基本是只有一行匹配。
>
>ref。基本是走普通索引。
>
>range。基本是走范围索引。
>
>index。走索引最差，和全表查询相似。
>
>NULL。不走索引，全表查询。

33.【强制】不适用索引的几种情况。

>不等式：!=、<>。
>
>null判断：is null、is not null。
>
>like模糊查询：like %a、like %a%
>
>not in。

34.【建议】避免使用IN操作，如果避免不了，需小于1000条。

>多表查询IN会影响查询效率。
>
>可以用between替代。
>
>IN(select * from)索引会失效，可以使用join（left、right、inner、full）来实现。

35.【建议】join优化。

>最好在三张表之内，最多不要超过5个，理论可以61个。
>
>on关联字段类型要相同。
>
>每关联一个表就会多分配一个关联缓存，和join_buffer_size设置相关。占用内存过大会形成溢出，影响性能和稳定性。
>
>left join的驱动表是左侧表。
>
>inner join的驱动表是数据少的表。
>
>right join的驱动表是右侧表。
>
>MySQL没有full join，可以用SQL实现。例如：`select * from A left join B on B.name = A.name where B.name is null union all select * from B`。
>
>尽量利用小表驱动大表，可以减少循环嵌套次数。
>
>straight join的使用。前提是inner join内连接。inner join优先查询小表，但有group by、order by等file_sort,Using temporary时会想改变优先查询表顺序，这时可以使用straight join。straight join强制优先查询表为左侧表。
>
>一定要是内连接才能使用straight join，否则数据可能不准确。

36.【强制】禁止select * 出现。

>select * 增加额外解析成本。
>
>增减字段对前端映射不一致。
>
>无用字段增加网络消耗。
>
>无法使用覆盖索引。

37.【强制】禁止使用不带字段的insert出现。

>正例：insert into user(user_no,user_age) values (123,18)
>
>反例：insert into user values (123,18)

38.【强制】尽量避免子查询。

>子查询一般在IN中。
>
>子查询会创建临时表，不会存在索引。
>
>结果集大的子查询，性能越差。
>
>可以使用join替代。

39.【强制】查询一条或者是否有数据时，要使用limit 1。

>索引效率最高。
>
>explain的type为const。

40.【建议】order by字段没有索引就不要排序。

>order by字段有索引会按索引排序。没有索引影响效率。
>
>可以设置索引，或者覆盖索引。

41.【建议】尽量不使用or。

>同一字段用IN、between等替代or，因为很多情况不会走索引。
>
>多字段下or两边都需要是索引且其他条件也是索引，才会走索引。
>
>最好使用union、union all来替换。

42.【建议】尽量用union all替代union。

>union会集合后进行唯一性去重，涉及到排序，加大资源开销。
>
>在没有重复数据情况强制使用union all。

43.【建议】拆分大且复杂的SQL。

>一条SQL只会使用一个CPU。
>
>拆成多个小SQL可以通过并行提高查询效率。

44.【强制】禁止使用ORDER BY RAND()

>随机排序性能差。
>
>可以用其他SQL替换。
>
>原：`select id from 'dynamic' order by rand() limit 1;，`
>
>新：`select id from 'dynamic' t1 join (select rand() * (select max(id) from 'dynamic') as nid) t2 on t1.id > t2.nid limit 1`;注意，此查询只能随机一条id，并连续查询该id的顺序条数，具体情况具体分析，适用随机取一条，不应用随机取多条。
>
>随机取多条解决方案：先查询所有id->在后端业务层做随机id->IN该id组。
>
>rand()取值范围：[ 0 , 1 )。

45.【强制】禁止对where条件字段进行函数转换。

>不走索引。
>
>正例：`select user_age from user where create_time>'20190320'`
>
>反例：`select user_age from user where date(create_time)>'20190320'`

46.【建议】in、exists、not in、not exists。

>in是子查询，优先查询驱动表为内表，所以适合内表数据小的情况。
>
>exists优先查询驱动表为外表，适合外表数据小的情况。
>
>不建议使用not in和not exists，不走索引且容易混淆。
>
>建议用其他SQL替代。
>
>反例：`select a.user_age from user a where a.user_no not in (select b.user_no from user_info b)`。
>
>正例：`select a.user_age from user a left user_info b on a.user_no = b.user_no where b.user_no is null。`

47.【建议】offset偏移量、分页。

>分页数据量大的情况会影响查询效率，因为不是跳过offset行，而是查询offset+N行，然后抛弃offset行。
>
>优化举例1：`select user_age from user where user_no > 13333 limit 20。`
>
>优化举例2：`select a.user_age from user a,(select user_no from user limit 13333,20)b where a.user_no = b.user_no。`

48.【强制】范围查询注意。

>between、>、<，查询时，如果是走联合索引，那么范围查询后的索引失效。

49.【强制】count（）相关。

>统计行数要使用count（*），不要使用count（列名）。
>
>count（*）会统计NULL数据，count（列名）不会统计NULL数据。
>
>当某一列的值全为NULL时，count（列名）返回的结果为0，但sum（列名）结果为NULL，因此使用sum（列名）需要使用IFNULL判断。
>
>例如：`select if(ifnull(sum(user_name)),0,sum(user_name))user_age from user`
