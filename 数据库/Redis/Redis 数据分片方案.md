# Redis 数据分片方案

分片是将数据拆分至多个 Redis 实例中的过程，因此每个 Redis 实例将会仅包含 keys 的子集。

## 分片的不同实现

实现数据分片的方式有很多种，例如 **范围分片** 和 **哈希分片**。**一致性哈希** 是一种高级形式的哈希分片，被应用于一些 Redis 的客户端和代理中。

在软件架构上，Redis 中的数据分片可以由不同的模块来实现：

- **客户端侧分片**：对于给定的 key，客户端会直接选择正确的 Redis 节点来进行读写操作。许多 Redis 客户端都实现了这种策略，例如 Jedis 中的 [ShardedJedis](https://github.com/redis/jedis/blob/jedis-3.3.0/src/main/java/redis/clients/jedis/ShardedJedis.java) 类；
- **代理侧分片**：客户端会将请求发送给 Redis 代理，由代理来选择正确的 Redis 节点。[twemproxy](https://github.com/twitter/twemproxy) 是一款 Redis 和 Memcached 的代理，它实现了这种策略；
- **查询路由**：客户端可以将请求发送给任意一台 Redis 节点，该节点会将请求重定向至正确的 Redis 节点。Redis Cluster 在和客户端的配合下，实现了这种策略。Jedis 中的 [JedisCluster](https://github.com/redis/jedis/blob/jedis-3.3.0/src/main/java/redis/clients/jedis/JedisCluster.java) 类支持 Redis Cluster。

## 分片的优点

在 Redis 中进行数据分片主要有两个目的：

- 使用多个机器的内存总和，来创建一个更大的数据库；
- 将计算能力扩展到多个内核和多个机器，将网络带宽扩展到多个机器和网络适配器。

## 分片的缺点

Redis 的一些特性在进行数据分片时，不能很好的工作：

- 通常不支持涉及多个 key 的操作；
- 不支持涉及多个 key 的 Redis 事务；
- 分片的粒度是 key，因此没有办法对单个非常大的 key 进行分片；
- 当使用分片时，数据处理会变得更加复杂；
- 在集群中扩容和缩容会变得更加复杂。

## 参考资料

- [Scaling with Redis Cluster](https://redis.io/docs/manual/scaling)
