# 《Redis 设计与实现》笔记 - 跳跃表

## Java 中的跳跃表 - ConcurrentSkipListMap

### 数据结构

```java
/**
 * Nodes hold keys and values, and are singly linked in sorted
 * order, possibly with some intervening marker nodes. The list is
 * headed by a header node accessible as head.node. Headers and
 * marker nodes have null keys. The val field (but currently not
 * the key field) is nulled out upon deletion.
 */
static final class Index<K,V> {
    final Node<K,V> node;  // currently, never detached
    final Index<K,V> down;
    Index<K,V> right;
    Index(Node<K,V> node, Index<K,V> down, Index<K,V> right) {
        this.node = node;
        this.down = down;
        this.right = right;
    }
}

/**
 * Index nodes represent the levels of the skip list.
 */
static final class Node<K,V> {
    final K key; // currently, never detached
    V val;
    Node<K,V> next;
    Node(K key, V value, Node<K,V> next) {
        this.key = key;
        this.val = value;
        this.next = next;
    }
}
```

```plain text
+-+                 +-+
|1|---------------->|5|--> NULL
+-+                 +-+
 |                   |
 v                   v
+-+       +-+       +-+       +-+
|1|------>|3|-------|5|------>|7|--> NULL
+-+       +-+       +-+       +-+
 |         |         |         |
 v         v         v         v
+-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+
|1|->|2|->|3|->|4|->|5|->|6|->|7|->|8|->|9|-> NULL
+-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+  +-+
 |    |    |    |    |    |    |    |    |
 v    v    v    v    v    v    v    v    v
NULL NULL NULL NULL NULL NULL NULL NULL NULL
```

## Redis 中的跳跃表

### 数据结构

#### zskiplistNoe

```c
typedef struct zskiplistNode {

    // 后退指针
    struct zskiplistNode *backward;

    // 分值
    double score;

    // 成员对象
    robj *obj;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 跨度
        unsigned int span;

    } level[];

} zskiplistNode;
```

```plain text
                                  +-------------+
                                  |zskiplistNode|
                                  |-------------|
                                  | level[4]    |
                 +-------------+  |-------------|
                 |zskiplistNode|  | level[3]    |
                 |-------------|  |-------------|
                 | level[2]    |  | level[2]    |
+-------------+  |-------------|  |-------------|
|zskiplistNode|  | level[1]    |  | level[1]    |
|-------------|  |-------------|  |-------------|
| level[0]    |  | level[0]    |  | level[0]    |
|-------------|  |-------------|  |-------------|
| backward    |  | backward    |  | backward    |
|-------------|  |-------------|  |-------------|
| score       |  | score       |  | score       |
|-------------|  |-------------|  |-------------|
| obj         |  | obj         |  | obj         |
+-------------+  +-------------+  +-------------+
```

```plain text
+--------+                                          +--------+
|level[2]|----------------------------------------->|level[2]|--> NULL
|--------|                +--------+                |--------|                +--------+
|level[1]|--------------->|level[1]|--------------->|level[1]|--------------->|level[1]| --> NULL
|--------|   +--------+   |--------|   +--------+   |--------|   +--------+   |--------|   +--------+   +--------+
|level[0]|-->|level[0]|-->|level[0]|-->|level[0]|-->|level[0]|-->|level[0]|-->|level[0]|-->|level[0]|-->|level[0]|--> NULL
|--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|
|backward|<--|backward|<--|backward|<--|backward|<--|backward|<--|backward|<--|backward|<--|backward|<--|backward|
|--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|
|score=1 |   |score=2 |   |score=3 |   |score=4 |   |score=5 |   |score=6 |   |score=7 |   |score=8 |   |score=9 |
|--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|   |--------|
|obj1    |   |obj2    |   |obj3    |   |obj4    |   |obj5    |   |obj6    |   |obj7    |   |obj8    |   |obj9    |
+--------+   +--------+   +--------+   +--------+   +--------+   +--------+   +--------+   +--------+   +--------+
```

字段描述：

1. zskiplistNode 的 level 数组可以包含多个元素，每个元素都包含一个指向其它节点的指针；
    1. zskiplistLevel 每层都有一个指向表尾方向的指针（level[i].forward），用于从表头向表尾访问节点；
    2. zskiplistLevel 每层都有一个跨度（level[i].span），用于记录两个节点之间的距离；
2. zskiplistNode 的 backward 属性用于从表尾向表头方向访问节点，但每次只能后退一个节点；
3. zskiplistNode 的 score 属性是一个 double 类型的浮点数，跳跃表中的所有节点都按分值的从小到大来排序；
4. zskiplistNode 的 obj 属性是一个指向字符串对象的指针，字符串对象保存着一个 SDS 值。

#### zskiplistlist

```c
typedef struct zskiplist {

    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;

    // 表中节点的数量
    unsigned long length;

    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```

```plain text
+---------+
|zskiplist|       +--------+   +--------+                +--------+
|---------|       |level[1]|-->|level[1]|--------------->|level[1]|--> NULL
| head    |------>|--------|   |--------|   +--------+   |--------|
|---------|       |level[0]|-->|level[0]|-->|level[0]|-->|level[0]|--> NULL
| tail    |---+   +--------+   |--------|   |--------|   |--------|
|---------|   |        NULL <--|backward|<--|backward|<--|backward|
| level   |   |                |--------|   |--------|   |--------|
|---------|   |                |score=1 |   |score=2 |   |score=3 |
| length  |   |                |--------|   |--------|   |--------|
+---------+   |                |obj1    |   |obj2    |   |obj3    |
              |                +--------+   +--------+   +--------+
              |                                               ^
              +-----------------------------------------------+
```
