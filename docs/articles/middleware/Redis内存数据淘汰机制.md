# Redis内存数据淘汰机制

## 什么是Redis内存数据淘汰机制？

​	redis持续使用，当内存空间不足继续使用的时候。redis会根据**某些策略**淘汰一部分key，从而释放出可用的空间。

1. 默认`redis.conf`配置文件信息地址:  [redis.conf](http://download.redis.io/redis-stable/redis.conf )

2. 通过配置一下命令选择对应的淘汰策略

   >```
   ># 当到达最大内存时，redis选择那种策略进行key删除
   ># MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
   ># is reached. You can select among five behaviors:
   >#
   ># volatile-lru: 从所有设置了过期时间的keys中逐除最近最少使用的key
   ># volatile-lru -> Evict using approximated LRU among the keys with an expire set.
   ># allkeys-lru: 从所有keys中逐除最近最少使用的key
   ># allkeys-lru -> Evict any key using approximated LRU.
   ># volatile-lfu: 从所有设置了过期时间的keys中逐除最少使用的key
   ># volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
   ># allkeys-lfu: 从所有keys中逐除使用频率最少的key
   ># allkeys-lfu -> Evict any key using approximated LFU.
   ># volatile-random: 从所有设置了过期时间的keys中随机逐除key
   ># volatile-random -> Remove a random key among the ones with an expire set.
   ># allkeys-random: 从所有keys中随机逐除key
   ># allkeys-random -> Remove a random key, any key.
   ># volatile-ttl: 从所有设置了过期时间的keys中逐除马上要过期的key
   ># volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
   ># noeviction: 不逐除任何key，只返回写入操作错误
   ># noeviction -> Don't evict anything, just return an error on write operations.
   >#
   ># LRU：最近最少使用的
   ># LRU means Least Recently Used
   ># LFU: 最少使用
   ># LFU means Least Frequently Used
   ># 
   ># LRU\LFU\volatile-ttl 都使用了近似的随机算法
   ># Both LRU, LFU and volatile-ttl are implemented using approximated
   ># randomized algorithms.
   >#
   ># Note: with any of the above policies, Redis will return an error on write
   >#       operations, when there are no suitable keys for eviction.
   >#	    以下相关命令进行操作会进行拒绝策略的检查
   >#       At the date of writing these commands are: set setnx setex append
   >#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
   >#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
   >#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
   >#       getset mset msetnx exec sort
   >#
   ># The default is:
   >#
   ># maxmemory-policy noeviction
   >```

3. `volatile-lru`基本上是使用最多的策略

4. Java如何实现**LRU**算法?

   ```java
   
   ```

   



