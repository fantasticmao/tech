# 《Redis 设计与实现》笔记 - 对象

Redis 中的数据结构有简单动态字符串、链表、字典、跳跃表等等。但是 Redis 并没有直接使用这些数据结构来实现键值对数据库，而是基于这些数据结构创建了一个对象系统，包括有 **字符串对象**、**列表对象**、**哈希对象**、**集合对象** 和 **有序集合对象** 这五种类型的对象。

Redis 创建对象系统的好处：

1. 可以根据对象的类型来判断是否可以执行某个命令；
2. 可以针对不同场景，为对象设置不同的数据结构，从而优化对象在不同场景下的使用效率。

Redis 的对象系统还实现了基于 **引用计数** 的内存回收机制：当某个对象不再被使用时，这个对象所占用的内存就会被自动释放。Redis 的对象系统还基于 **引用计数** 实现了对象共享机制，使得 Redis 在适当的条件下，可以共享同一个值对象。

Redis 的对象记录了访问时间的信息，在服务器开启 `maxmemory` 功能的情况下，长时间未使用的对象可能会优先被删除。

## 数据结构

Redis 使用对象来表示数据库中的键值对，每次在 Redis 中创建一个新的键值对时，Redis 至少会创建两个对象，一个对象用于键值对的键，一个对象用于键值对的值。

Redis 中的每个对象都是由一个 redisObject 结构表示

```c
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码
    unsigned encoding:4;

    // 指向底层实现数据结构的指针
    void *ptr;

    // ...

} robj;
```

对象中的字段描述：

1. type 属性记录了对象的类型，值为以下常量之一；
    1. **REDIS_STRING** 字符串对象
    2. **REDIS_LIST** 列表对象
    3. **REDIS_HASH** 哈希对象
    4. **REDIS_SET** 集合对象
    5. **REDIS_ZSET** 有序集合对象
2. encoding 属性记录了对象所使用的编码，即实现对象所使用的数据结构，值为以下常量之一；
    1. **REDIS_ENCODING_INT** long 类型的整数
    2. **REDIS_ENCODING_EMBSTR** embstr 编码的简单动态字符串
    3. **REDIS_ENCODING_RAW** 简单动态字符串
    4. **REDIS_ENCODING_HT** 字典
    5. **REDIS_ENCODING_LINKEDLIST** 链表
    6. **REDIS_ENCODING_ZIPLIST** 压缩列表
    7. **REDIS_ENCODING_INTSET** 整数集合
    8. **REDIS_ENCODING_SKIPLIST** 跳跃表和字典

每个对象都至少可以使用两种不同的数据结构来实现，具体的对象关系如下：

| type                          | encoding                                                 | 含义                                    |
| ----------------------------- | -------------------------------------------------------- | --------------------------------------- |
| **REDIS_STRING**              | **REDIS_ENCODING_INT**                                   | 使用 **整数集合** 实现 **字符串对象**   |
| **REDIS_ENCODING_EMBSTR**     | 使用 **embstr 编码的简单动态字符串** 实现 **字符串对象** |                                         |
| **REDIS_ENCODING_RAW**        | 使用 **简单动态字符串** 实现 **字符串对象**              |                                         |
| **REDIS_LIST**                | **REDIS_ENCODING_ZIPLIST**                               | 使用 **压缩列表** 实现 **列表对象**     |
| **REDIS_ENCODING_LINKEDLIST** | 使用 **链表** 实现 **列表对象**                          |                                         |
| **REDIS_HASH**                | **REDIS_ENCODING_ZIPLIST**                               | 使用 **压缩列表** 实现 **哈希对象**     |
| **REDIS_ENCODING_HT**         | 使用 **字典** 实现 **哈希对象**                          |                                         |
| **REDIS_SET**                 | **REDIS_ENCODING_INTSET**                                | 使用 **整数集合** 实现 **集合对象**     |
| **REDIS_ENCODING_HT**         | 使用 **字典** 实现 **集合对象**                          |                                         |
| **REDIS_ZSET**                | **REDIS_ENCODING_ZIPLIST**                               | 使用 **压缩列表** 实现 **有序集合对象** |
| **REDIS_ENCODING_SKIPLIST**   | 使用 **跳跃表和字典** 实现 **有序集合对象**              |                                         |

Redis 可以根据不同的使用场景来为一个对象设置不同的编码，从而优化对象在某个场景下的使用效率。例如，当列表对象包含的元素比较少时，Redis 会使用压缩列表作为对象的底层实现：

1. 因为压缩列表比链表更加节省内存，并且在元素数量比较少时，在内存中以连续块方式保存的压缩列表会比链表更快地加载到 CPU 缓存中；
2. 随着列表对象包含的元素越来越多，使用压缩列表来保存元素的优势会逐渐消失，此时 Redis 会将对象的底层实现从压缩列表转向功能更强、也更适合保存大量元素的链表。

## 相关命令

- [TYPE](https://redis.io/commands/type) 命令可以以字符串的形式，返回值对象的类型
- [OBJECT](https://redis.io/commands/object) 命令可以检查值对象的内部信息，例如 `object refcount foo` 可以查看值对象的引用计数、`object encoding foo` 可以查看值对象的编码
