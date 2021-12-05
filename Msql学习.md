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

## 4.1联表查询

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



## 4.3

![image-20211205200644369](C:\Users\HQ\AppData\Roaming\Typora\typora-user-images\image-20211205200644369.png)







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
