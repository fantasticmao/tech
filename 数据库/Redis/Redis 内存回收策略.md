# Redis 内存回收策略

## 作为 LRU 缓存来使用

通过 redis.conf 配置文件或者 `CONFIG SET` 运行时命令，可以配置 Redis 的最大内存限制。Redis 提供了以下几种内存回收策略：

1. noeviction 超出内存限制之后，客户端再次添加数据时，Redis 会返回错误；
2. allkeys-lru 所有 key 参与 LRU 回收；
3. volatile-lru 存在过期时间的 key 参与 LRU 回收；
4. allkeys-random 所有 key 参与随机回收；
5. volatile-random 存在过期时间的 key 参与随机回收；
6. volatile-ttl 优先回收存在过期时间并且 TTL（time to live）较短的 key；

需要注意的是，volatile-lru、volatile-random、volatile-ttl 在没有 key 回收的时候，Redis 会像 noeviction 一样返回错误。

Reids 回收进程的工作机制：

1. 客户端发出一个新的命令，添加了新的数据；
2. Redis 检查内存使用情况，如果大于最大内存限制，则会根据具体的相关策略回收对应的 key；
3. 执行新的命令；

在实际情况中，Redis 会在占用内存超出最大内存限制之后，再回收相关 key。Redis 使用的是近似 LRU 的回收算法，因为真实的 LRU 算法会占用很多内存。

## 参考资料

- [Key eviction](https://redis.io/docs/manual/eviction)
