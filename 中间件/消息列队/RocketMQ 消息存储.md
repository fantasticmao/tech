# RocketMQ 消息存储

消息存储模块是 RocketMQ 中最为复杂和重要的模块。

## 文件结构

```
/{user.home}/store/.
                   +--commitlog
                   |  +--00000000000000000000 (1GB)
                   |  +--00000000001073741824 (1GB)
                   +--consumequeue
                   |  +--topic_1 (topic name)
                   |  |  +--0 (queue id)
                   |  |  |  +--00000000000000000000 (5.72MB=300000*20/1024/1024)
                   |  |  |  +--00000000000006000000 (5.72MB)
                   |  |  +--1 (queue id)
                   |  |     +--00000000000000000000 (5.72MB)
                   |  +--topic_2 (topic name)
                   |     +--0 (queue id)
                   |        +--00000000000000000000 (5.72MB)
                   +--index
                      +--19950815000000000 (1995-08-15 00:00:00.000, 400MB)
                      +--20201112080000000 (2020-11-12 08:00:00.000, 400MB)
```

## 数据结构

```
                    +---------------------------+
         +-----+    |+-+ +-+ +-+ +-+ +-+ +-+ +-+|
         |+---+|--->|| | | | | | | | | | | | | ||<-----+
         ||   ||    |+-+ +-+ +-+ +-+ +-+ +-+ +-+|      |
         |+---+|    +---------------------------+      | offset
         |     |            ConsumeQueue               +-------- Consumer
         |+---+|    +---------------------------+      |
         ||   ||    |+-+ +-+ +-+ +-+ +-+ +-+ +-+|      |
         |+---+|--->|| | | | | | | | | | | | | ||<-----+
         |     |    |+-+ +-+ +-+ +-+ +-+ +-+ +-+|
         |+---+|    +---------------------------+
         ||   ||            ConsumeQueue
         |+---+|
         |     |
         |+---+|    +---------------------------+
         ||   ||    |+-+ +-+ +-+ +-+ +-+ +-+ +-+|  queyr by key
         |+---+|--->|| | | | | | | | | | | | | ||<-------------- User
         +-----+    |+-+ +-+ +-+ +-+ +-+ +-+ +-+|
        CommitLog   +---------------------------+
            ^               IndexFile
            |
Producer ---+
```

## 存储架构

RocketMQ 消息存储模块的设计主要与上面三类文件相关：

- CommitLog 用于存储消息的主体和元数据。Producer 端写入的消息主体的内容长度是不固定的。CommitLog 文件大小默认 1GB，文件长度为 20 位，左边补零，剩余为起始偏移量。例如 00000000000000000000 代表了第一个文件，起始偏移量为 0；00000000001073741824 代表了第二个文件，起始偏移量为 1GB=1024\*1024\*1024KB=1073741824KB。CommitLog 文件的内容是顺序写入的。
- ConsumeQueue 是用于提高 Consumer 消费消息性能的逻辑消费队列。由于 RocketMQ 是基于 topic 订阅机制来发送和消费消息的，因此如果 Consumer 直接从 CommitLog 中依据 topic 来查找待消费的消息，则需要遍历完整的 CommitLog，这将会是一个非常低效的操作。ConsumeQueue 作为 CommitLog 中消费消息的索引，保存了指定 topic 下的 MessageQueue 在 CommitLog 中的物理偏移量、消息大小、和消息 tag 的哈希值。因此 ConsumeQueue 文件的存储路径分为三层：consumequeue/{topic}/{queue}/{file}，并且 ConsumeQueue 文件同样采取定长设计，其中的每一个条目为 20 字节：物理偏移量 8 字节 + 消息大小 4 字节 + 消息 tag 哈希值 8 字节。每个 ConsumeQueue 由 30w 个条目组成，共计 5.72 MB，并且可以像访问数组一样随机访问 ConsumeQueue 内部的条目。
- IndexFile 和 ConsumeQueue 类似，是用于提高依据 key 来查询消息的效率的索引文件。IndexFile 底层的存储方式设计为在文件系统中实现的 HashMap 结构，因此 IndexFile 的索引类型是 hash 类型的。IndexFile 存储路径为 index/{fileName}，其中文件名称为 YYYYMMddHHmmsssS 格式的日期时间，并且 IndexFile 文件同样采取定长设计，单个文件大小约 400MB。

RocketMQ 存储 CommitLog 的代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/store/src/main/java/org/apache/rocketmq/store/CommitLog.java#L528-L638)，序列化 CommitLog 的代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/store/src/main/java/org/apache/rocketmq/store/CommitLog.java#L1186-L1330)。

RocketMQ 通过 `DefaultMessageStore` 的 `doDispatch` 方法，将消息的索引内容存储至 ConsumeQueue 和 IndexFile 中，具体代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/store/src/main/java/org/apache/rocketmq/store/DefaultMessageStore.java#L1770)。

## 缓存页和内存映射

缓存页（Page Cache）是操作系统对文件的缓存，用于加速对文件的读写速度。一般来说，应用程序对文件的顺序读取速度会接近于对内存的读写速度，这是因为操作系统使用了缓存页机制来对读写操作进行了优化。

- 对于数据的写入操作，操作系统会先将数据写入至缓存页，然后通过异步的方式由 pdflush 内核线程将缓存页中的数据刷新至磁盘；
- 对于数据的读取操作，如果在一次读取文件时出现未命中缓存页的情况，则操作系统在磁盘上访问文件的同时，会顺序对其他相邻的数据文件进行预读。

在 RocketMQ 中，ConsumeQueue 存储的数据比较少，并且是被操作系统顺序读取的。因此在缓存页的机制下，操作系统对 ConsumeQueue 的读取性能接近于对内存的读取，并且消息堆积也不会对 ConsumeQueue 的读取性能造成影响。但是 CommitLog 存储的数据比较大，并且会被操作系统随机读取，因此对 CommitLog 读取的性能会相对低效。此时，如果选择合适的系统 I/O 调用算法（例如在存储采用 SSD 时设置调度算法为 Deadline），随机读取的性能会有所提升。

在 RocketMQ 中，主要通过 MappedByteBuffer 内存映射机制对文件进行读写操作。利用 NIO 中的 FileChannel 将磁盘上的物理文件直接映射到用户态的内核空间中，将应用对文件的操作转换为直接对内存地址进行操作，从而极大地提高了文件的读写性能。
