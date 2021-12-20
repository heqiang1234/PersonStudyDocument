# Watch监听机制

Zookeeper允许用户在指定的节点上注册一些Watcher，并且在一些特定事件触发的时候，Zookeeper服务端会将事件通知到感兴趣的客户端上去，该机制是Zookeeper实现分布式协调服务的重要特性。

Zookeeper中引入了Watcher机制来实现了发布/订阅功能，能够让多个订阅者同时监听某一个对象，当一个对象自身状态变化，会通知所有订阅者。

Zookeeper原生支持通过注册Watcher来进行时间监听，但是其使用并不是特别方便需要开发人员自己反复注册Watcher，比较繁琐

Curator引入了Cache来实现对Zookeeper服务端事件的监听。

Zookeeper提供了三种Watcher：

- NodeCache：只是监听某一个特定的节点
- PathChildrenCache：监控一个ZNode的子节点
- TreeCache：可以监控整个树上的所有节点，类似于NodeCache和PathChildrenCache的结合





# 分布式锁

- 在我们进行单机应用开发，涉及并发同步的时候，我们往往采用synchronize或者lock的方式来解决多线程的代码同步问题，这时多线程的运行都是在同一个JVM之下，没有任何问题。

- 但是当我们的应用是分布式集群工作的情况下，属于多JVM下的工作环境，跨JVM之间无法通过多线程的锁解决同步问题。

- 那么就需要一种更高级的锁机制，来处理**跨机器的进程之间的数据同步问题**-这就是分布式锁。

  ![image-20211220105100474](C:\Users\hspcadmin\AppData\Roaming\Typora\typora-user-images\image-20211220105100474.png)



# Zookeeper分布式锁原理

- 核心思想：当客户端要获取锁，则创建节点，使用完锁，则删除该节点。
  - 客户端获取锁时，在lock节点下创建==临时顺序==节点。
  - 然后获取lock下面所有的子节点，客户端获取到所有的子节点之后，如果发现自己创建的子节点序号最小，那么就认为客户端获取到了锁，使用完锁后，将该节点删除。
  - 如果发现自己创建的节点并非lock所有子节点最小的，说明自己还没有获取到锁，此时客户端需要找到比自己小的哪个节点，同事对其注册时+间监听器，监听删除事件。
  - 如果发现比自己小的那个节点被删除，则客户端Watcher会收到相应通知，此时再次判断自己创建的节点是否是lock子节点最小的，如果是，则获取到了锁，如果不是，则重复以上步骤继续获取到比自己小的一个节点并注册监听。
  - 临时，顺序，监听器

![image-20211220135821594](C:\Users\hspcadmin\AppData\Roaming\Typora\typora-user-images\image-20211220135821594.png)



# Curator实现分布式锁API

- 在curator中有5种锁方案：
  - InterProcessSemaphoreMutex：分布式排它锁（非可重入锁）
  - InterProcessMutex：分布式可重入排它锁
  - InterProcessReadWriteLock：分布式读写锁
  - InterProcessMultiLock：将多个锁作为单个实体管理的容器
  - InterProcessSemaphoreV2：共享信号量





# Zookeeper集群搭建

## Zookeeper集群介绍

leader选举：

- ServiceId：服务器ID
- 比如有三台服务器，编号分别是1，2，3。编号越大在选择算法中的权重越大。
- Zxid：数据ID
- 服务器中存放的最大数据ID，值越大说明数据越新，在选举算法中数据越新权重越大。
- 在leader选举中，如果某台zookeeper获得超过半数的选票
- 则此Zookeeper就可以成为leader



zookeeper集群角色

在zookeeper集群服务中三个角色：

- Leader领导者：
  - 处理事务请求
  - 集群内部各服务器的调度者
- Follow跟随者：
  - 处理客户端非事务请求，转发事务给请求给Leader服务器
  - 参与Leader选举投票
- Observer观察者：
  - 处理客户端非事务请求，转发事务请求给Leader服务器



# leader选举机制

在Zookeeper中，分为两种选举情况，一种是初始化集群分布式的时候会进行leader选举，另外一种是在运行期间leader出现故障会进行选举，所以一般Zookeeper的服务器机器数量都会选择为奇数。在分布式CAP理论中，zookeeper属于一个CP系统，即一致性、分区容错性，它保证了集群数据的一致性，但适当舍弃了一些高可用。

zookeeper节点的4种状态:
      LEADING：说明此节点已经是leader节点，处于领导者地位的状态，差不多就是一般集群中的master。但在zookeeper中，只有leader才有写权限，其他节点（FOLLOWING）是没有写权限的，可以读

      LOOKING：选举中，正在寻找leader，即将进入leader选举流程中
    
      FOLLOWING：跟随者，表示当前集群中的leader已经选举出来了，主要具备以下几个功能点             
    
                向leader发送请求（PING消息、REQUEST消息、ACK消息、REVALIDATE消息）
    
                接收leader消息并进行处理；
    
                接收client发送过来的请求，如果为写请求，会发送给Leader进行投票处理，然后返回client结果。
    
      OBSERVING：OBSERVING和FOLLOWING差不多，但不参加投票和选举，接受leader选举后的结果

选举过程:
假如有以下5台机器server1、server2、server3、server4、server5    图是网上扒的



每个server 自身都有一票，在初始化或者server崩溃数过半的时候，每个server都有一个自身的myid(zookeeper配置文件)，这里按1、2、3、4、5算

在选举过程中主要是依据zxid和myid来进行轮训server然后比较统计投票

zxid (ZooKeeper Transaction Id，每次请求对应一个唯一的zxid，如果zxid a < zxid b ，则可以保证a一定发生在b之前)。

选举分为两种情况，初始化和leader挂掉的时候，要进行leader选举，至少需要2台机器，集群机器台数基本是奇数

初始化
当启动初始化集群的时候，server1的myid为1，zxid为0   server2的myid为2，zxid同样是0，以此类推。此种情况下zxid都是为0。先比较zxid，再比较myid

服务器1启动，给自己投票，然后发投票信息，由于其它机器还没有启动所以它收不到反馈信息，服务器1的状态一直属于Looking(选举状态)。
服务器2启动，给自己投票，同时与之前启动的服务器1交换结果，由于服务器2的myid大所以服务器2胜出，但此时投票数没有大于半数，所以两个服务器的状态依然是LOOKING。
服务器3启动，给自己投票，同时与之前启动的服务器1,2交换信息，由于服务器3的myid最大所以服务器3胜出，此时投票数正好大于半数，所以服务器3成为领导者，服务器1,2成为小弟。
服务器4启动，给自己投票，同时与之前启动的服务器1,2,3交换信息，尽管服务器4的myid大，但之前服务器3已经胜出，所以服务器4只能成为小弟。
服务器5启动，后面的逻辑同服务器4成为小弟
当选举机器过半的时候，已经选举出leader后，后面的就跟随已经选出的leader，所以4和5跟随成为leader的server3

所以，在初始化的时候，一般到过半的机器数的时候谁的myid最大一般就是leader

运行期间
按照上述初始化的情况，server3成为了leader，在运行期间处于leader的server3挂了，那么非Observer服务器server1、server2、server4、server5会将自己的节点状态变为LOOKING状态

1、开始进行leader选举。现在选举同样是根据myid和zxid来进行

2、首先每个server都会给自己投一票竞选leader。假设server1的zxid为123，server2的zxid为124，server4的zxid为169，server5的zxid为188

3、同样先是比较zxid再比较，server1、server2、server4比较server4根据优先条件选举为leader。然后server5还是跟随server4，即使server5的zxid最大，但是当选举到server4的时候，机器数已经过半。不再进行选举，跟随已经选举的leader

 

zookeeper集群为保证数据的一致性所有的操作都是由leader完成，之后再由leader同步给follower。重点就在这儿，zookeeper并不会确保所有节点都同步完数据，只要有大多数节点（即n/2+1）同步成功即可。

咱们假设有一个写操作成功那么现在数据只存在于节点leader，之后leader再同步给其他follower。这时候宕掉3个机器，已经过半的机器无法进行投票选举，剩余2台不足过半，无法选举=无法提供任何服务。再启动一个机器恢复服务。所以宕掉的机器不要过半，过半就会导致无法正常服务。

在leader选举的时候会有30s-120s的过程，在这期间也是无法提供服务的。如果用zookeeper要作为服务发现是个弊端，基本无法忍受，zookeeper本身是一个CP系统，保证数据的一致性，在恢复的时候再提供服务，并没有多好高可用的方案。如果leader发生故障选举时无法提供服务发现对一个大型应用来说可能是致命的。它可以为同在一个分布式系统中的其他服务提供：统一命名服务、配置管理、分布式锁服务、集群管理等功能）是个伟大的开源项目，很成熟
————————————————
版权声明：本文为CSDN博主「ypp91zr」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/ypp91zr/article/details/89409707
