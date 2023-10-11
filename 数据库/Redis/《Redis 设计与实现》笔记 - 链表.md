# 《Redis 设计与实现》笔记 - 链表

## 数据结构

### listNode

```c
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;
```

```plain text
       +--------+   +--------+   +--------+
... -->|listNode|-->|listNode|-->|listNode|--> ...
       |--------|   |--------|   |--------|
... <--| value  |<--| value  |<--| value  |<-- ...
       +--------+   +--------+   +--------+
```

### list

```c
typedef struct list {

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 链表所包含的节点数量
    unsigned long len;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

} list;
```

```plain text
+-----+
|list |
|-----|               +--------+   +--------+   +--------+
|head |-------------->|listNode|-->|listNode|-->|listNode|--> NULL
|-----|               |--------|   |--------|   |--------|
|tail |--+    NULL <--| value  |<--| value  |<--| value  |<---+
|-----|  |            +--------+   +--------+   +--------+    |
|len  |  +----------------------------------------------------+
|-----|
|dup  |--> ...
|-----|
|free |--> ...
|-----|
|match|--> ...
+-----+
```

PS：和 `java.util.LinkedList` 相似

### 特性

1. 双向：链表节点带有 prev 和 next 指针，获取某个节点的前置节点和后置节点的时间复杂度都是 O(1)；
2. 无环：表头节点的 prev 指针和表尾的 next 指针都指向 null，对链表的访问以 null 为终点；
3. 带表头和表尾指针：通过 list 的 head 和 tail 指针，获取链表的表头节点和表尾节点时间复杂度为 O(1)；
4. 带链表长度计数器：通过 len 属性来对链表节点进行计数，获取链表的节点数量时间复杂度为 O(1)；
5. 多态：链表节点受用 `void*` 指针来保存节点值，所以链表可以用于保存各种不同类型的数据。
