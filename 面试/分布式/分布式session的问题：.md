# 分布式session的问题：

+ 1.session粘滞   ip_hash客户端的ip和服务器的响应绑定
  + 保证一点    ip_hash策略，不能进行架构的扩容
  + 存在缺点：如果这个机器挂了，那就丢失了，没有那么随机，负载均衡空了
+ session复制
  + server.xml配置，一台服务器有新生成session时，通知发送给其他服务器，相当于复制一份。
  + 缺点：服务器资源浪费
+ session统一存储  spring-session-data-redis
  + 使用redis，在nginx后面封装一层redis,session全部存储到redis,每次服务器要获取sessionid时，全部通过redis获取，
  + 缺点：
    + 1.加大程序的网络IO消耗    服务器和redis在同一组网，局域网，减少网络IO消耗
    + 2.容灾性差
  + 单点故障，连接有限，存储有限
+ JWT ：json web token 
  + https://jwt.io/



分布式：session / 锁      并发编程



A服务调用B服务，B服务如何获取用户信息？







1. 任意用户请求落在 任意服务器：都可以保证session可用的
2. 2.session只复制一份



大公司不使用session存储数据，跨子域，跨多域

异地多活

open AM认证服务器

session