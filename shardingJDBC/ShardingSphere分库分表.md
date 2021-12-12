版权声明：本文为博主原创文章，遵循[CC 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/)版权协议,转载请附上原文出处链接和本声明，KuangStudy,以学为伴，一生相伴！

原文链接：https://www.kuangstudy.com/bbs/1356097953512632321

# ShardingSphere分库分表

1.什么是垂直/水平拆分模式
2.基于客户端与服务器端实现分表分库区别
3.单表达到多大量开始分表分库
4.数据库分表分|库策略有那些
5.sharding-jdbc实战分库分表
6.MYSQL主从复制
7.Shardingjdbc读写分离
8.Sharding-Proxy
9.Sharding-Sidecar
10.为什么选ShardingSphere作为分库分表中间件而不选Mycat
11.分表分库后如何实现排序查询shardingshere如何实现迁移数据以及扩容的。

## 1.数据库分表分库规则:

**分库分表**有哪一些**规则**

**垂直/水平拆分模式**

**1.垂直模式**指的是从数据结构方面的拆分。一般来说。在系统设计阶段就应该从业务耦合性松紧来确定垂直分库。垂直分表方案。在数据量及访问量不是特别大的情况下。首先考虑缓存。读写分离。索引技术等方案。

**垂直分库**实际上就是根据公司不同的业务来拆分成不同的数据库
微服务项目架构:分成团队开发模式 也就是我们熟知的分库
会员团队、支付团队、订单团队每个团队中都有自己独立的数据库。会员数据库
订单数据库支付数据库

**缺点**：1.分布式事务问题。 2.维护成本 3.增大跨库join 4.分布式主键id

**垂直分表** 可以把一个宽表的字段按照访问频次、是否是大字段的原则拆分为多个表。这样技能业务清晰。还能提升性能。拆分后。尽量从业务的角度避免联查，否则性能方面得不偿失。

**2.水平拆分:**分为**水平分库**和**水平分表**。就是**同一个库**或者**同一个表**的数据拆分到多个库或者多个表中存放。
单张表数据量超过**500万行**的时候，查询也有可能会遇到瓶颈，分页/排序应该减轻单张表数据量
将一张表数据放入到多个不同的数据库中或者是将一张表数据拆分**n多个子表**存放。

db_01
db_02

**缺点**：查询问题。 分页/排序 内存溢出问题。（shardingjdbc归并数据）

**3.水平拆分和垂直拆分共同缺点**
*分布式事务处理困难* 夸节点join困难
*扩数据源管理复杂

**4.切分总则**
*能不切分的尽量不切分* 如果要切分，选择合适的切分规则，提前规划好
*数据库切分尽量通过数据冗余或表分组来降低跨库join* 业务尽量使用少的多表join

## 2.基于客户端与服务器端实现分表分库区别

服务器中间件： **mycat**和**sharding-jdbc**和**sharding-proxy**

**mycat**基于**服务器**端实现数据库代理：

**官方网站**：http://www.mycat.org.cn/donate.html

![1](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy39f3af65-de0d-4ff3-84af-7ab42844c721.png)

底层实际基于服务器实现数据库中间件(mycat)**Mycat**类似于**nginx**
**优点**:保证数据库的安全性归并数据结果完全**解耦**

**缺点：**相对**shardingsphere**来说,软件**更新**速度**慢**。耗损略高

**shardingjdbc**基于**客户端**实现数据库代理

![1612257327278](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyf633a2a8-7257-4570-aea5-9c26bf8906c1.png).png)

基于**客户端**方式实现数据库中间件**(ShardingJDBC)**
**优点**:效率比较高
**缺点**:归并数据结果是没有实现**解耦**，有可能会影响到我们业务逻辑代码。（当归并结果过大时会出现**内存溢出**问题)

原理：

Select *from user where id=1;
原理:基于aop 代理的方式拦截改写sql语句Select* from user_1 where id=1 ;

**sharding-proxy基于服务器端**实现数据库代理(相当于**mycat**)

![![1612256468289](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy8ee55acc-0a9c-4b88-b052-beda023e5bc0.png)]

**sharding分库分表原理:** SQL解析 -> 查询优化-> SQL路由->SQL改写->SQL执行->结果合并

## 3.单表达到多大量开始分表分库

单表行数超过 **500 万**行或者单表容量超过 **2GB**，才推荐进行分库分表。

说明：如果预计三年后的数据量根本达不到这个级别，请不要在创建表时就分库分表。

为啥单表行数超过 **500 万**行需要分库分表？

目前找到了300万数据的表 system_user加了主键**b+tree**索引

```
select count(0) from system_user  0.507s     --数据量3323001SELECT  *  FROM system_user LIMIT 3000000,10   --7.497s   SELECT id FROM system_user LIMIT 3000000,10   --0.412sselect s.* from system_user sjoin  (select id from system_user limit 3200000,10) t on t.id = s.id --0.452s--当数据量达到500万的时候 就算是加了索引 也需要将近1s的时间
```

为啥单表容量超过 **2GB**，但行数没达到500万也需要分库分表？

**B+tree**按照**页单位**来读取的，每一页读16kb 如果一行的数据很大的情况下。扫描范围是比较小的， 需要多次读取。所以就算数据库没达到500w行。但是单表数据容量达到2G的时候也需要分表分库。

## 4.数据库分表分|库策略有那些

**取余/取模** 优点：均匀存放数据 缺点：不能扩容

**按照范围分片** 1-500万 501-1000万 优点：容易扩容，实现简单。

**按照日期进行分片** 日志 订单信息。

**按照月份进行分片** 保险业务。

**按照枚举值分片** 常量分片 比如。省份分表

**二进制取模范围分片**

**一致性hash分片** 类似hashmap 缺点： 数据存放不均匀。

**按照目标字段前缀指定的进行分区**

**按照前缀ASCII码和值进行取模范围分片**

预定义：适合小型 反推数据 几年下不会太多数据。—>一般是创业型公司。

**主流分片算法**： 取余/取模 日期 一致性hash分片

根据user id

user id=1%2=1

user id=0%2=0

user id=3%2=3

user id=4%2=4

不能实现扩容。

## 5.Sharding-jdbc实战分库分表

**官方网站**：http://shardingsphere.apache.org/index_zh.html

**文档**：https://shardingsphere.apache.org/document/current/cn/overview/

概述：定位为轻量级Java框架，在Java的JDBC层提供的额外服务。 它使用客户端直连数据库，以jar包形式提供服务，无需额外部署和依赖，可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架。

- 适用于任何基于JDBC的ORM框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template或直接使用JDBC。
- 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, Druid, HikariCP等。
- 支持任意实现JDBC规范的数据库。目前支持MySQL，Oracle，SQLServer，PostgreSQL以及任何遵循SQL92标准的数据库。

![1612257304024](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy5b1f915e-1f83-4620-8ce5-fb4dacbee743.png)

![1612257304024](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy76a6c025-00a9-497c-9aba-7092e1c1d79b.png)

------

目前我实战采用的是单实例 多表的情况 也就是水平分表的情况。

**0.创建数据库**

```
SET NAMES utf8mb4;SET FOREIGN_KEY_CHECKS = 0;DROP database IF EXISTS `shardingsphere_demo`;CREATE DATABASE `shardingsphere_demo` CHARACTER SET = utf8 ;use `shardingsphere_demo`;-- ------------------------------ Table structure for t_user_0-- ----------------------------DROP TABLE IF EXISTS `t_user_0`;CREATE TABLE `t_user_0`  (  `id` bigint(0) NOT NULL COMMENT '主键',  `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '姓名',  `age` int(0) NULL DEFAULT NULL COMMENT '年龄',  `sex` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '性别',  PRIMARY KEY (`id`) USING BTREE) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;-- ------------------------------ Records of t_user_0-- ----------------------------INSERT INTO `t_user_0` VALUES (1, '张三', 18, '1');INSERT INTO `t_user_0` VALUES (3, '王五', 21, '0');-- ------------------------------ Table structure for t_user_1-- ----------------------------DROP TABLE IF EXISTS `t_user_1`;CREATE TABLE `t_user_1`  (  `id` bigint(0) NOT NULL COMMENT '主键',  `name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '姓名',  `age` int(0) NULL DEFAULT NULL COMMENT '年龄',  `sex` char(1) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '性别',  PRIMARY KEY (`id`) USING BTREE) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;-- ------------------------------ Records of t_user_1-- ----------------------------INSERT INTO `t_user_1` VALUES (2, '李四', 20, '0');INSERT INTO `t_user_1` VALUES (4, '赵六', 17, '1');SET FOREIGN_KEY_CHECKS = 1;
```

**1.创建项目**

![1612147645376](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy1c47083e-7668-4404-abce-d05a6fef46bf.png)

**2.pom.xml**依赖

```
<?xml version="1.0" encoding="UTF-8"?><project xmlns="http://maven.apache.org/POM/4.0.0"         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">    <modelVersion>4.0.0</modelVersion>    <parent>        <groupId>org.springframework.boot</groupId>        <artifactId>spring-boot-starter-parent</artifactId>        <version>2.3.1.RELEASE</version>        <relativePath/>    </parent>    <description> 学习shardingjdbc分库分表</description>    <groupId>com.thh</groupId>    <artifactId>shardingsphere-demo</artifactId>    <version>1.0.0-SNAPSHOT</version>    <properties>        <java.version>1.8</java.version>        <shardingsphere.version>4.0.0-RC1</shardingsphere.version>    </properties>    <dependencies>        <dependency>            <groupId>org.springframework.boot</groupId>            <artifactId>spring-boot-starter-test</artifactId>            <scope>test</scope>            <exclusions>                <exclusion>                    <groupId>org.junit.vintage</groupId>                    <artifactId>junit-vintage-engine</artifactId>                </exclusion>            </exclusions>        </dependency>        <!-- mysql5.7驱动 -->        <dependency>            <groupId>mysql</groupId>            <artifactId>mysql-connector-java</artifactId>            <version>5.1.4</version>            <scope>runtime</scope>        </dependency>        <!-- jsonson处理，服务于jsonutil工具和springmvc的json返回 -->        <dependency>            <groupId>com.fasterxml.jackson.dataformat</groupId>            <artifactId>jackson-dataformat-avro</artifactId>            <version>2.10.0</version>        </dependency>        <!-- md5密码加密的时依赖的包 -->        <dependency>            <groupId>commons-codec</groupId>            <artifactId>commons-codec</artifactId>            <version>1.11</version>        </dependency>        <dependency>            <groupId>com.alibaba</groupId>            <artifactId>druid-spring-boot-starter</artifactId>            <version>1.1.10</version>        </dependency>        <dependency>            <groupId>org.springframework.boot</groupId>            <artifactId>spring-boot-starter-aop</artifactId>        </dependency>        <dependency>            <groupId>org.springframework.boot</groupId>            <artifactId>spring-boot-starter-web</artifactId>        </dependency>        <dependency>            <groupId>org.springframework.boot</groupId>            <artifactId>spring-boot-devtools</artifactId>            <scope>runtime</scope>            <optional>true</optional>        </dependency>        <dependency>            <groupId>org.apache.shardingsphere</groupId>            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>            <version>${shardingsphere.version}</version>        </dependency>        <!--mybatis-plus整合mybatis-->        <dependency>            <groupId>com.baomidou</groupId>            <artifactId>mybatis-plus-boot-starter</artifactId>            <version>3.4.2</version>        </dependency>        <dependency>            <groupId>com.alibaba</groupId>            <artifactId>fastjson</artifactId>            <version>1.2.62</version>        </dependency>        <dependency>            <groupId>org.projectlombok</groupId>            <artifactId>lombok</artifactId>        </dependency>        <dependency>            <groupId>org.apache.commons</groupId>            <artifactId>commons-lang3</artifactId>            <version>3.6</version>        </dependency>    </dependencies>    <build>        <finalName>${project.artifactId}</finalName>        <plugins>            <plugin>                <groupId>org.springframework.boot</groupId>                <artifactId>spring-boot-maven-plugin</artifactId>            </plugin>        </plugins>    </build></project>
```

**3.aplication.yml**文件

```
server:  port: 8999logging:  level:    root: infospring:  profiles:    active: dev  main:   #允许相同bean名称覆盖    allow-bean-definition-overriding: true  application:    name: shardingsphere-demo  jackson:    # 日期进行格式化处理    date-format: yyyy/MM/dd HH:mm:ss    # 解决json转换少8个小时的问题    time-zone: GMT+8    # 解决json返回过程中long的精度丢失问题    generator:      write-numbers-as-strings: true      write-bigdecimal-as-plain: true# mybatis-plus配置mybatis-plus:  mapper-locations: classpath*:/mappers/*.xml  configuration:    call-setters-on-nulls: true  type-aliases-package: com.thh.entity
```

**application-dev.yml**

```
spring:  shardingsphere:    datasource:      names: ds0      ds0:        driver-class-name: com.mysql.jdbc.Driver        url: jdbc:mysql://localhost:3306/shardingsphere_demo?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT&useSSL=false        password: 123456        type: com.alibaba.druid.pool.DruidDataSource        username: root    sharding:      binding-tables: t_user      default-database-strategy:        inline:          algorithm-expression: ds0          sharding-column: user_id      tables:        t_user:          actual-data-nodes: ds0.t_user_$->{0..1}          key-generator:            column: id            type: SNOWFLAKE          table-strategy:            inline:              algorithm-expression: t_user_$->{id % 2 +1}              sharding-column: id    props:      sql:        show:  true
```

**4**.创建**实体类** **controller** 以及**mapper**

**启动类**

```
/** * @ClassName ShardingsphereDemoApplication * @Description 启动类 * @Author thh * @Date 0:03 2021/02/01 **/@SpringBootApplication@MapperScan("com.thh.mapper")public class ShardingsphereDemoApplication {    public static void main(String[] args) {        SpringApplication.run(ShardingsphereDemoApplication.class,args);    }}
```

**entity实体类**

```
@Data@ToString@NoArgsConstructor@AllArgsConstructor@TableName("t_user")public class User {    // 主键    @TableId    private Long id;    // 用户名    private String name;    // 年龄    private Integer age;    /**     * 性别 0代表男 1代表女     */    private String sex;}
```

**mapper**

```
@Repositorypublic interface UserMapper extends BaseMapper<User> {    /**     * 查询所有用户     * @return     */    @Select("select * from t_user")   List<User> selectUserList();}
```

**controller**

```
/** * @ClassName UserController * @Description web * @Author thh * @Date 0:09 2021/02/01 **/@RestController@RequestMapping("/user")public class UserController {    @Autowired    private UserMapper userMapper;    /**     * 查询所有用户     * @return     */    @RequestMapping("/list")    public List<User> getUserList(){        return userMapper.selectUserList();    }}
```

实际上我们的表 是 t_user_0 t_user_1 shardingshere帮我们通过查询 t_user虚拟表 然后帮我们映射成实际的sql，把查出来的数据进行了归并。

![1612147884181](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyd9e1bfd8-d279-494b-a471-216879f4047b.png)

**公共表**的配置

**公共表**属于系统中数据量较小,变动少,而且属于**高频联合查询**的**依赖表**。**参数表**、**数据字典表**等属于此类型。可以将这类表在每个数据库都保存一份,所有更新操作都同时发送到所有分库执行。接下来看一下如何使用Sharding-IDBC实现公共表。|

如果是实现的是多实例。**公共表**的情况的话 比如**字典表**。每个实例都需要用的情况，我们就会配置公共表.

sharding-jdbc帮我们做的就是路由到各个数据源。当我们进行插入操作的时候。把数据插入到不同数据源的公共表当中。

```
spring:  shardingsphere:    datasource:      names: ds0      ds0:        driver-class-name: com.mysql.jdbc.Driver        url: jdbc:mysql://47.115.49.189:3306/shardingsphere_demo?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT&useSSL=false        password: mogu2018        type: com.alibaba.druid.pool.DruidDataSource        username: root    sharding:      binding-tables: t_user  #绑定的表      default-database-strategy:        inline:          algorithm-expression: ds0          sharding-column: user_id      tables:        t_user:          actual-data-nodes: ds0.t_user_$->{0..1}          key-generator:            column: id            type: SNOWFLAKE          table-strategy:            inline:              algorithm-expression: t_user_$->{id % 2 +1}              sharding-column: id      broadcast-tables: t_dist   #公共表配置     props: #开启sql输出日志      sql:        show:  true
```

## 6.MYSQL主从复制(读写分离前提)

**理解读写分离**

面对日益增加的系统访问量,数据库的吞吐量面临着巨大瓶颈。对于同一 时刻有大量并发读操作和较少写操作类型的应用系统来说,将数据库拆分为主库和从库,主库负责处理事务性的增删改操作,从库负责处理查询操作,能够有效的避免由数据更新导致的行锁,使得整个系统的查询性能得到极大的改善。

通过一主多从的配置方式,可以将查询请求均匀的分散到多个数据副本,能够进一步的提升 系统的处理能力。使用多主多从的方式 ,不但能够提升系统的吞吐量,还能够提升系统的可用性,可以达到在任何-一个数据库宕机,甚至磁盘物理损坏的情况下仍然不影响系统的正常运行。

主从同步的N种实现方式?**主从同步**其实与**读写分离**是两码事
**Sharding-JDBC**读写分离则是根据SQL语义的分析,将读操作和写操作分别路由至主库与从库。 它提供透明化读写分离,让使用方尽量像使用一个数据库-样使用主从数据库集群。

![1612181228861](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy7adc4c42-2168-416d-afbc-4f6ee0f0f45c.png)

**Sharding-IDBC**提供- -主多从的读写分离配置,可独立使用,也可配合分库分表使用,同一线程且同- -数据库连接内,如有写入操作,以后的读操作均从主库读取,用于保证数据一致性。 **Sharding-JDBC**不提供主从数据库的数据同步功能 ,需要采用其他机制支持。

**MYSQL**自带**主从同步**机制。

**主从复制工作原理剖析**

> 1.Master 数据库只要发生变化，立马记录到Binary log 日志文件中(binlog 手动开启)
>
> 2.Slave数据库启动一个I/O thread连接Master数据库，请求Master变化的二进制日志
> 3.Slave I/O获取到的二进制日志，保存到自己的Relay log 日志文件中。(Relay log 中继日志 持久化存储)
> 4.Slave 有一个 SQL thread定时检查Realy log是否变化，变化那么就更新数据

**存在的问题：1.2.3部**操作都是顺序操作。执行很快。**第4部重放**会特别慢。重放的时候 解析出来的DML语句。mysql执行DML语句的时候。比如 update table set name =‘111’ where id =‘1’ update table set name =‘123’ where id =‘200’.

实际上更新的块是不同的。执行速度慢，这会造成relay log 堆积。出现**主从复制的延时问题**。

**理解：**说白了就是日志监测。只要有日志。说明数据有变化 主就往从更新数据。

![1612182222659](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy89aa05e7-ad7b-47d8-8b2d-f2eef70ad525.png)

主从配置需要**注意**的点

(1)主从服务器操作系统版本和位数一致；

(2) Master和Slave数据库的版本要一致；

(3) Master和Slave数据库中的数据要一致；

(4) Master开启二进制日志，Master和Slave的server_id在局域网内必须唯一；

**01.环境准备**：首先搭建**两个mysql服务**。可以是linux方式 docker方式或者是windows方式。两个服务器可以ping通。

docker方式创建

```
#主mysqldocker run  -di --name=mysql_master --restart always -v /docker/mysql_master/data:/var/lib/mysql -v /docker/mysql_master/config:/etc/mysql   -p  3307:3306 -e MYSQL_ROOT_PASSWORD=sharding123 mysql:5.7#需要在config下复制mysql.cnf 为my.cnf 不然配置不会生效cd /docker/mysql_master/configcp mysql.cnf my.cnf#从mysqldocker run  -di --name=mysql_slave --restart always -v /docker/mysql_slave/data:/var/lib/mysql -v /docker/mysql_slave/config:/etc/mysql   -p  3308:3306 -e MYSQL_ROOT_PASSWORD=sharding123 mysql:5.7cd /docker/mysql_slave/configcp mysql.cnf my.cnf
```

![1612189573177](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy8713ee58-f089-47b5-bc1b-e2c24a728b18.png)

![1612191701217](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyf2782874-0e99-43df-bb84-e922e3e6a0ea.png)

**02.开启mysql的binlog日志功能**

**1.**修改**主和从**的**my.cnf**文件

**主配置**

```
#mysql master1 config [mysqld]server-id = 1        # 节点ID，确保唯一 一般设置为IPbinlog-do-db=shardingsphere_demo  # 复制过滤：需要备份的数据库，输出binlog# log configlog-bin = mysql-bin        #开启mysql的binlog日志功能 可以随便取，最好有含义sync_binlog = 1            #控制数据库的binlog刷到磁盘上去 , 0 不控制，性能最好，1每次事物提交都会刷到日志文件中，性能最差，最安全binlog_format = mixed      #binlog日志格式，mysql默认采用statement，建议使用mixedexpire_logs_days = 7       #binlog过期清理时间 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除max_binlog_size = 100m     #binlog每个日志文件大小binlog_cache_size = 4m     #binlog缓存大小  为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存max_binlog_cache_size= 512m   #最大binlog缓存大binlog-ignore-db=mysql     #不需要备份的数据库不生成日志文件的数据库，多个忽略数据库可以用逗号拼接，或者 复制这句话，写多行## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。#slave-skip-errors = all #跳过从库错误slave-skip-errors = all auto-increment-offset = 1     # 自增值的偏移量auto-increment-increment = 1  # 自增值的自增量
```

**从配置**

```
#mysql slave config [mysqld]server-id = 2log-bin=mysql-binrelay-log = mysql-relay-binreplicate-wild-ignore-table=mysql.%replicate-wild-ignore-table=test.%replicate-wild-ignore-table=information_schema.%
```

**MySQL**对于二进制日志 (**binlog**)的复制类型

(1) 基于语句的复制：在Master上执行的SQL语句，在Slave上执行同样的语句。MySQL默认采用基于语句的复制，效率比较高。一旦发现没法精确复制时，会自动选着基于行的复制。

(2) 基于行的复制：把改变的内容复制到Slave，而不是把命令在Slave上执行一遍。从MySQL5.0开始支持。

(3) 混合类型的复制：默认采用基于语句的复制，一旦发现基于语句的无法精确的复制时，就会采用基于行的复制。

修改完了 别忘记重启

docker restart mysql_master

docker restart mysql_slave

**2.**可以创建**另外的用户**进行测试。或者直接拿root用户进行测试。

```
-- 创建用户CREATE USER sharding IDENTIFIED BY 'sharding123';-- 赋予权限grant replication slave on *.* to 'sharding'@'%'  identified by 'sharding123';-- 生效FLUSH PRIVILEGES;
```

**3**.查看**master**的状态

```
--先进入容器 docker exec -it mysql_master /bin/bash--连接mysqlmysql -uroot -psharding123;mysql> show master status;+------------------+----------+--------------+------------------+-------------------+| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |+------------------+----------+--------------+------------------+-------------------+| mysql-bin.000004      154   |              | mysql            |                   |+------------------+----------+--------------+------------------+-------------------+1 row in set (0.00 sec)
```

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy04e37761-9d89-400d-a1ae-c6211ddceed1.png)

**4.从节点配置master配置**。（小弟认大哥的过程）

```
-- master的配置 --master_log_pos对应的是 show master status 查出的 position--master_log_file对应的是 show master status 查出的 file    这两个都需要要对上了 才会同步成成功change master to master_host='172.18.7.9',master_port=3307,master_user='sharding',master_password='sharding123',master_log_file='mysql-bin.000004',master_log_pos=154;-- 开启SLAVE线程：start slave;
```

![1612236922306](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudydcb265de-2dab-4952-8ff5-72d058925197.png)

![1612237276983](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyb092c75f-f135-4a14-877d-432b5c5102c5.png)

**5.效验是否进行了同步**

![1612237659849](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyb8419bae-c4b9-4a00-8b4d-29c4d32c50b2.png)

测试成功。到master数据库的shadingshere_demo库进行增删改操作的时候。从节点的mysql也会对shadingshere_demo库进行操作。

## 7.Sharding-jdbc读写分离

**官方文档地址**：https://shardingsphere.apache.org/document/legacy/4.x/document/cn/manual/sharding-jdbc/configuration/config-spring-boot/

![1612254786106](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyff77f5b4-673c-4eb7-9255-975e529520b3.png)

```
spring:        #实现读写分离 配置主从库  shardingsphere:    datasource:      names: master,slave0      master:        driver-class-name: com.mysql.jdbc.Driver        url: jdbc:mysql://localhost:3307/shardingsphere_demo?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT&useSSL=false        username: sharding        password: sharding123        type: com.alibaba.druid.pool.DruidDataSource      slave0:        driver-class-name: com.mysql.jdbc.Driver        url: jdbc:mysql://localhost:3308/shardingsphere_demo?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT&useSSL=false        username: root        password: sharding123        type: com.alibaba.druid.pool.DruidDataSource    sharding:      master-slave-rules:        ds0:          master-data-source-name:  master          slave-data-source-names:  slave0      binding-tables: t_user  #绑定的表      default-database-strategy:        inline:          algorithm-expression: ds0          sharding-column: user_id      tables:        t_user:          actual-data-nodes: ds0.t_user_$->{0..1}          key-generator:            column: id            type: SNOWFLAKE          table-strategy:            inline:              algorithm-expression: t_user_$->{id % 2}              sharding-column: id      #broadcast-tables: t_dist   #公共表配置    props: #开启sql输出日志      sql:        show:  true
```

写操作路由的是主**master**数据库

![1612256011081](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy2b179e1a-83fa-438d-87a7-44428529d641.png)

读操作路由到的是**slave0**数据库

![1612256169991](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyc2ceea53-1f45-44e9-9cb9-01895e37a593.png)

## 8.Sharding-Proxy

**1.概述**：定位为透明化的数据库代理端，提供封装了数据库二进制协议的服务端版本，用于完成对异构语言的支持。 目前先提供MySQL/PostgreSQL版本，它可以使用任何兼容MySQL/PostgreSQL协议的访问客户端(如：MySQL Command Client, MySQL Workbench, Navicat等)操作数据，对DBA更加友好。

基于**服务器端**的**数据库代理**中间件 类似**mycat**

![1612256570203](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyc3394e4a-79de-45dd-9527-abb2b2ddc3ab.png)

**2.下载**

![1612256844202](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy16025653-920e-47b8-8bca-00a6c366098f.png)

也可以用官方docker镜像 https://shardingsphere.apache.org/document/legacy/4.x/document/cn/manual/sharding-proxy/docker/

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy552ab9e2-f5ad-4707-83f3-83bad289f28e.png)

**手动下载压缩包。**

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy7a84a2b2-43a9-4e44-8d1a-88737855a74a.png)

**3.安装。下载完之后进行解压。启动bin目录文件就可以了。**

**解压**的时候不要有**中文**目录和**空格**。

![1612258678268](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyafeba108-af6a-4ad6-900a-b488f0f0e3a1.png)

解压下来后lib下的**jar包**修改一下后缀名。全部改成**jar** 有**部分名称**缺了。**前缀**为**sharding**的**后缀**统一为**-4.0.1.jar**

![1612259051595](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyf897c508-bba7-4fdb-a59c-d2b93acabdc2.png)

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy3631fc28-039c-4e89-8c7b-e1463b90c4cc.png)

**4.Sharding-Proxy配置**

1. 进入conf目录修改server.yaml文件 把**两段注释**去掉

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudye5b56724-047d-481f-bddb-bbfa782eb2ad.png)

1. 进入conf目录。修改config-sharding.yaml

还是去掉注释。修改成我们对应的库就可以了。和sharding-jdbc差不多。

放mysql驱动到lib下

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy1f96977a-a274-45ff-be7f-c9488aed5466.png)

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy67548ffa-3259-4684-a07b-52418de7b6fd.png)

```
schemaName: sharding_dbdataSources:  ds_0:    url: jdbc:mysql://localhost:3306/shardingsphere_demo?serverTimezone=UTC&useSSL=false    username: root    password: 123456    connectionTimeoutMilliseconds: 30000    idleTimeoutMilliseconds: 60000    maxLifetimeMilliseconds: 1800000    maxPoolSize: 50shardingRule:  tables:    t_user:      actualDataNodes: ds_${0}.t_user_${0..1}      tableStrategy:        inline:          shardingColumn: order_id          algorithmExpression: t_user_${id % 2}      keyGenerator:        type: SNOWFLAKE        column: id  bindingTables:    - t_user  defaultDatabaseStrategy:    inline:      shardingColumn: user_id      algorithmExpression: ds_${0}  defaultTableStrategy:    none:
```

**5.启动**

bin目录下执行 默认端口3307 如果想**指定端口** 就直接 cmd打开黑框 start.bat 端口号

![1612260789305](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy66640619-d707-461b-bf5f-9fb41d5155d3.png)

由于路径上有空格 出现了以下报错。找不到文件路径

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyfae05c3e-2759-4de4-8bc6-74a494a00ddd.png)

换成**没有空格**的目录后执行。虽然执行成功了 还是报了另外一个错误。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy800e345b-a39f-4eaa-89f6-2b020db485cd.png)

后面看了之后找到原因。是因为mysql驱动过低的问题

替换**mysql8.0**的驱动后解决问题。或者换成官方网站要求的5.1.47以上的。

![1612262116385](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy6fc38884-2a7e-4aa3-8633-4205d679a128.png)

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyf8110ede-aa16-4de6-b7bc-0f9e4417c5f0.png)

果然和**mycat**没啥区别。不过好的就是文档写的还是特别到位。目前捐给apache管理。

**6.sharding-proxy配置读写分离**

老规矩。打开**config-master_slave.yaml** 去掉注释。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy510f3cb6-0d31-4b8a-8166-5189de148d9e.png)

**读写分离**配置

```
schemaName: master_slave_dbdataSources:  master_ds:    url: jdbc:mysql://localhost:3307/shardingsphere_demo?serverTimezone=UTC&useSSL=false    username: sharding    password: sharding123    connectionTimeoutMilliseconds: 30000    idleTimeoutMilliseconds: 60000    maxLifetimeMilliseconds: 1800000    maxPoolSize: 50  slave_ds_0:    url: jdbc:mysql://localhost:3308/shardingsphere_demo?serverTimezone=UTC&useSSL=false    username: sharding    password: sharding123    connectionTimeoutMilliseconds: 30000    idleTimeoutMilliseconds: 60000    maxLifetimeMilliseconds: 1800000    maxPoolSize: 50masterSlaveRule:  name: ms_ds  masterDataSourceName: master_ds  slaveDataSourceNames:    - slave_ds_0
```

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudyf92b30aa-ef79-4315-a8bb-73d9f44a5978.png)

配置完之后,用客户端**navicat12**又出现了另外一个报错 ,好家伙。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy4e89c9d9-46de-4d6f-bd00-0bcb7f7fb469.png)

然后我通过**黑框**客户端去连接后，我发现是可以进行操作的。所以实际上是没问题的。是我的那**navicat12**不兼容sharding-proxy。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudydef98f72-0911-4fbf-b657-413ea7adc6c0.png)

这样 我做一下**查询更新**的操作 看看打印出的sql 会不会做到**自动路由**达到**读写分离**

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy99fb4772-bdb3-45c0-aba0-b5703202693f.png)

**读写分离+数据分片配置**

```
schemaName: sharding_master_slave_dbdataSources:  ds0:    url: jdbc:mysql://localhost:3307/shardingsphere_demo?serverTimezone=UTC&useSSL=false    username: sharding    password: sharding123    connectionTimeoutMilliseconds: 30000    idleTimeoutMilliseconds: 60000    maxLifetimeMilliseconds: 1800000    maxPoolSize: 50  ds0_slave0:    url: jdbc:mysql://localhost:3308/shardingsphere_demo?serverTimezone=UTC&useSSL=false    username: sharding    password: sharding123    connectionTimeoutMilliseconds: 30000    idleTimeoutMilliseconds: 60000    maxLifetimeMilliseconds: 1800000    maxPoolSize: 50shardingRule:    tables:    t_user:       actualDataNodes: ms_ds${0}.t_user_${0..1}      databaseStrategy:        inline:          shardingColumn: user_id          algorithmExpression: ms_ds${0}      tableStrategy:         inline:          shardingColumn: id          algorithmExpression: t_user_${id % 2}      keyGenerator:        type: SNOWFLAKE        column: id  bindingTables:    - t_user  broadcastTables:    - t_config  defaultDataSourceName: ds0  defaultTableStrategy:    none:  masterSlaveRules:    ms_ds0:      masterDataSourceName: ds0      slaveDataSourceNames:        - ds0_slave0
```

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy3e07e898-f35f-4c29-9611-105862404767.png)

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy95806ade-5781-4cfc-91a4-e3e772d7db38.png)

## **9.Sharding-Sidecar（TODO）**

**简介：**定位为Kubernetes的云原生数据库代理，以Sidecar的形式代理所有对数据库的访问。 通过无中心、零侵入的方案提供与数据库交互的的啮合层，即Database Mesh，又可称数据网格。

Database Mesh的关注重点在于如何将分布式的数据访问应用与数据库有机串联起来，它更加关注的是交互，是将杂乱无章的应用与数据库之间的交互有效的梳理。使用Database Mesh，访问数据库的应用和数据库终将形成一个巨大的网格体系，应用和数据库只需在网格体系中对号入座即可，它们都是被啮合层所治理的对象。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy35aa649b-ba05-4726-a9bc-9d05c85b6370.png)

| *Sharding-JDBC* | *Sharding-Proxy* | *Sharding-Sidecar* |        |
| :-------------- | :--------------- | :----------------- | ------ |
| 数据库          | 任意             | MySQL              | MySQL  |
| 连接消耗数      | 高               | 低                 | 高     |
| 异构语言        | 仅Java           | 任意               | 任意   |
| 性能            | 损耗低           | 损耗略高           | 损耗低 |
| 无中心化        | 是               | 否                 | 是     |
| 静态入口        | 无               | 有                 | 无     |

**混合架构**

Sharding-JDBC采用无中心化架构，适用于Java开发的高性能的轻量级OLTP应用；Sharding-Proxy提供静态入口以及异构语言的支持，适用于OLAP应用以及对分片数据库进行管理和运维的场景。

ShardingSphere是多接入端共同组成的生态圈。 通过混合使用Sharding-JDBC和Sharding-Proxy，并采用同一注册中心统一配置分片策略，能够灵活的搭建适用于各种场景的应用系统，架构师可以更加自由的调整适合于当前业务的最佳系统架构。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy0589c788-686d-4643-b0c8-07db4f12f1e2.png)

## 10.为什么选**ShardingSphere**作为分库分表中间件而不选**Mycat**

**相同点：shadingsphere**和**mycat**都是为了解决**分库分表的中间件**。都是优秀的开源的项目。mycat开源较早。mycat在。ShardingSphere前身是**当当网**的。ShardingSphere 已于2020年4月16日成为 Apache 软件基金会的顶级项目。让ShardingSphere更上一层楼。**mycat**是通过配置 将数据库逻辑表与实体表在配置文件中对应，由mycat处理逻辑表与实体表的实际操作，应用程序层连接mycat，跟连接mysql数据库一模一样，应用程序操作数据库的逻辑表，逻辑表通过配置文件对应到实体的表。 应用程序层不做处理，mycat是安装在服务器端，在服务器端再进行的实际sql的执行。实际sql的执行也同**shardingspere**一样，**解析、改写、路由、结果归集**等操作。

**MYCAT**

**2013**年阿里的Cobar在社区使用过程中发现存在一些比较严重的问题，及其使用限制，经过Mycat发起人第一次改良，第一代改良版——Mycat诞生。 Mycat开源以后，一些Cobar的用户参与了Mycat的开发，最终Mycat发展成为一个由众多软件公司的实力派架构师和资深开发人员维护的社区型开源软件。**2014**年Mycat首次在上海的《中华架构师》大会上对外宣讲，更多的人参与进来，随后越来越多的项目采用了Mycat。**2015年5月**，由核心参与者们一起编写的第一本官方权威指南《Mycat权威指南》电子版发布，累计超过500本，成为开源项目中的首创。**2015年10月**为止，Mycat项目总共有16个Committer。截至**2015年11月**，超过300个项目采用Mycat，涵盖银行、电信、电子商务、物流、移动应用、O2O的众多领域和公司。

Mycat的官网。我在网上搜的时候实际上很难搜到这个网站。我进去后我怀疑我搞错了。因为MYCAT在我看来是非常强大的中间件。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy34f361c3-d017-4576-8be9-53d7b08e0771.png)

查看**代码更新**的时间 实际上还在**持续**的更新当中。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudydb1a8d65-faf7-4dfa-b40e-57f177e7a58d.png)

**ShardingShpere**

**官方网站**：https://shardingsphere.apache.org/index_zh.html

不管是从文档还是官方文档方方面面都做的特别好。还称为了apache基金会的顶级项目。对分库分表做了一系列的解决方案。版本还在持续更新中。并且更新的速度很快。从去年的2020年4月份3.0版本 到目前的5.X版本。

![img](https://kuangstudy.oss-cn-beijing.aliyuncs.com/bbs/2021/02/02/kuangstudy68f8b8eb-4d23-46c1-bcde-cf7a1bfc6a2a.png)

实际上两个中间件。不过我更偏向ShardingShpere。