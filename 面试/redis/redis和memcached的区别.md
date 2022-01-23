1. **存储方式上**: Memcache会把数据全部存在内存中，断电后会挂掉，数据不能超过内存大小

，Redis有部分数据存在硬盘上，这样能保证数据的持久性。

2. **数据支持类型上**：Memcached对数据的支持很简单，只有简单的key-value,redis支持5中数据类型

String（SDS (Simple Dynamic String)）,list(双向链表),hash(hashmap),set（hashtable）,zset(跳表)

3. **使用的底层模型不同：**它们底层实现方式以及客户端之间通信的应用协议不一样，Redis直接自己构建VM机制，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
4. **Value的大小**：Redis可以达到1G，而 Memcache 只有 1MB