volatile-lru  设置了过期时间的数据，less recently used ，按照最近最少使用的规则进行淘汰数据。

volatile-ttl 设置了过期时间的数据，time to live，优先对剩下时间短的数据淘汰

volatile-random 设置了过期时间的数据，random，随机淘汰数据

allkeys-lru 优先对最近最少使用的数据进行淘汰

allkeys-lru 随机淘汰数据

noeviction 不淘汰数据，内存不足了，直接报错



volatile-lfu 设置了过期时间的数据，least frequency use 优先对使用频率最少的数据淘汰

allkeys-lfu ，优先淘汰使用频率最小的数据