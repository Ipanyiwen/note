1. Redis 有哪些数据结构，分别有什么使用场景？

   A: Str, List, Hash, Set, Zset, Bitmap, Hyperloglog, Geo, Stream

2. 使用的索引结构 - map

3. Redis 是否支持事务？

   A: 有限支持（不支持回滚）。Redis事务总是满足原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation），持久化模式下有限支持持久性（Durability）。

4. Redis 是如何保证高可用的？

   A: 主从（提高读）/哨兵（提供故障转移）/集群（提高读写，提供故障转移）。

5. Redis 单线程模型如何保障高性能

6. Redis持久化方式， 详细讲讲AOF和RDB

7. 缓存淘汰策略

8. Redis的使用场景

9. 讲讲缓存雪崩，缓存击穿，缓存穿透

   

   

