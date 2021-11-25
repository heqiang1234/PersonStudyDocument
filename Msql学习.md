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



#### ==DBMS(数据库管理系统)==

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

#初始化数据库文件

net start mysql

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



> 



## 1.6 安装sqlyog



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



# x.1 重点字段

1、事务

尽管我们可以使用子查询（Sub-Queries）、连接（JOIN）和联合（UNION）来创建各种各样的查询，但不是所有的数据库操作都可以只用一条或少数几条SQL语句就可以完成的。更多的时候是需要用到一系列的语句来完成某种工作。但是在这种情况下，当这个语句块中的某一条语句运行出错的时候，整个语句块的操作就会变得不确定起来。设想一下，要把某个数据同时插入两个相关联的表中，可能会出现这样的情况：第一个表中成功更新后，数据库突然出现意外状况，造成第二个表中的操作没有完成，这样，就会造成数据的不完整，甚至会破坏数据库中的数据。要避免这种情况，就应该使用事务，它的作用是：要么语句块中每条语句都操作成功，要么都失败。换句话说，就是可以保持数据库中数据的一致性和完整性。事物以BEGIN关键字开始，COMMIT关键字结束。在这之间的一条SQL操作失败，那么，ROLLBACK命令就可以把数据库恢复到BEGIN开始之前的状态。

```sql
BEGIN; 
	INSERT INTO salesinfo SET CustomerID=14;
	UPDATE inventory SET Quantity=11 WHERE item='book';
COMMIT;
```

事务的另一个重要作用是当多个用户同时使用相同的数据源时，它可以利用锁定数据库的方法来为用户提供一种安全的访问方式，这样可以保证用户的操作不被其它的用户所干扰。



# xx.注意事项

链接：https://zhuanlan.zhihu.com/p/162727511

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

21.【强制】联合索引的左侧原则可以减少每个字段单独建立索引。

>避免每个字段都建立索引。
>
>联合索引区分度高的放在最左边。
>
>联合索引如果存在非等号和等号混合时，把等号的索引放在最左边。
>
>联合索引左侧原则，一定要注意顺序。
