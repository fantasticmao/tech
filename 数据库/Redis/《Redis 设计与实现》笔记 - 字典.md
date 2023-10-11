# 《Redis 设计与实现》笔记 - 字典

## 数据结构

### dictht

```c
typedef struct dictht {

    // 哈希表数组
    dictEntry **table;

    // 哈希表大小
    unsigned long size;

    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;

    // 该哈希表已有节点的数量
    unsigned long used;

} dictht;
```

```plain text
+--------+
|dictht  |
|--------|      +-------------+
|table   |----->|dictEntry*[4]|
|--------|      |-------------|
|size    |      | 0           |--> NULL
|--------|      |-------------|
|sizemask|      | 1           |--> NULL
|--------|      |-------------|
|used    |      | 2           |--> NULL
+--------+      |-------------|
                | 3           |--> NULL
                +-------------+
```

### dictEntry

```c
typedef struct dictEntry {

    // 键
    void *key;

    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;

    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;

} dictEntry;
```

```plain text
+--------+
|dictht  |
|--------|      +-------------+
|table   |----->|dictEntry*[4]|
|--------|      |-------------|
|size    |      | 0           |--> NULL
|--------|      |-------------|
|sizemask|      | 1           |--> NULL
|--------|      |-------------|           +---------+   +---------+
|used    |      | 2           |---------->|dictEntry|-->|dictEntry|--> NULL
+--------+      |-------------|           |---------|   |---------|
                | 3           |--> NULL   | k1 | v1 |   | k1 | v1 |
                +-------------+           +---------+   +---------+
```

### dict

```c
typedef struct dict {

    // 类型特定函数
    dictType *type;

    // 私有数据
    void *privdata;

    // 哈希表
    dictht ht[2];

    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */

} dict;
```

```c
typedef struct dictType {

    // 计算哈希值的函数
    unsigned int (*hashFunction)(const void *key);

    // 复制键的函数
    void *(*keyDup)(void *privdata, const void *key);

    // 复制值的函数
    void *(*valDup)(void *privdata, const void *obj);

    // 对比键的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);

    // 销毁键的函数
    void (*keyDestructor)(void *privdata, void *key);

    // 销毁值的函数
    void (*valDestructor)(void *privdata, void *obj);

} dictType;
```

```plain text
+---------+
|dict     |
|---------|           +--------+
|type     |           |dictht  |     +-------------+
|---------|     +---->|--------|     |dictEntry*[4]|
|privdata |     |     |table   |---->|-------------|
|---------|     |     |--------|     | 0           |
|ht       |-----+     |size    |     |-------------|     +---------+
|---------|     |     |--------|     | 1           |---->|dictEntry|----> NULL
|rehashidx|     |     |sizemask|     |-------------|     |---------|
+---------+     |     |--------|     | 2           |     | k1 | v1 |
                |     |used    |     |-------------|     +---------+
                |     +--------+     | 3           |
                |                    +-------------+
                |       +--------+
                +------>|dictht  |
                        |--------|
                        |table   |
                        |--------|
                        |size    |
                        |--------|
                        |sizemask|
                        |--------|
                        |used    |
                        +--------+
```

PS：和 `java.util.HashMap` 相似
