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

