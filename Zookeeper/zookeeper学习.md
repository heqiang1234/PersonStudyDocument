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

