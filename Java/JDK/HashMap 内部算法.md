# HashMap 内部算法

## 定位 key 位于 table 中的位置

从 `public V get(Object key)` 方法来看在 HashMap 内部是如何分布 key 的。

第一步是计算 key 在 HashMap 内部的哈希值：如果 key 值为 null 则直接返回 0，否则使用 `hashCode()` 方法获取 key 的原始哈希值 h，然后计算 key 的内部哈希值，计算公式为 `h ^ (h >>> 16)`。一次运算的过程如下：

```plain text
       h = hashCode(): 0000 1111 1111 0000 0000 0000 0101 0101
             h >>> 16: 0000 0000 0000 0000 0000 1111 1111 0000
hash = h ^ (h >>> 16): 0000 1111 1111 0000 0000 1111 1010 0101
```

在计算 key 的内部哈希值时，会保留 key 的原始哈希值高 16 位值不变，低 16 位值会与高 16 位值进行 **异或** 运算，这一步中的算法是与计算 key 在 HashMap 的 table 中位置相关的。

第二步是计算 key 在 HashMap 的 table 中的位置：根据 key 的内部哈希值来决定 key 在 table 中的位置，计算公式为 `(table.length - 1) & hash`。一次运算的过程如下：

```plain text
    hash = h ^ (h >>> 16): 0000 1111 1111 0000 0000 1111 1010 0101
         table.length - 1: 0000 0000 0000 0000 0000 0000 1111 1111
(table.length - 1) & hash: 0000 0000 0000 0000 0000 0000 1010 0101
```

此处需要注意 HashMap 中的一个限制：**table 的长度总会是 2 的幂次**。在计算 key 位于 table 中的具体位置时，计算结果只会与 key 的内部哈希值的低位有关，为了让 key 的原始哈希值的高位也参与到计算过程中来，所以在第一步计算 key 内部哈希值时采用了将原始哈希值高 16 位与低 16 位进行异或的运算过程。

## 扩容 table

当元素个数达到 `threshold` 个时，HashMap 便会对 table 进行扩容，newTable 的容量是 oldTable 容量的两倍。

### 在 OpenJDK 中的实现

OpenJDK 7 中 HashMap 的扩容算法是从 oldTable 中各个队列的队头读取元素，然后逐个按序插入到 newTable 新位置中队列的队尾，该过程中会 **反转哈希槽上的链表元素顺序**，具体源码请见 [GitHub](https://github.com/openjdk/jdk/blob/jdk7-b147/jdk/src/share/classes/java/util/HashMap.java#L483-L502)。一次扩容示例过程如下：

```plain text
// 扩容之前，哈希表槽数为 2
+-+
| |
+-+  +-+  +-+
|3|->|7|->|5|->NULL
+-+  +-+  +-+

// 扩容之后，哈希表槽数为 4
+-+
| |
+-+
|5|->NULL
+-+
| |
+-+  +-+
|7|->|3|->NULL
+-+  +-+
```

该算法在并发场景下可能会产生死循环问题。例如上例中的数据，扩容之前为 3->7，扩容之后为 7->3，当两个线程交替执行扩容算法时，就可能会产生 3<->7 的数据，从而产生了死循环。具体分析请见文章 [《疫苗：JAVA HASHMAP的死循环》](https://coolshell.cn/articles/9606.html#并发下的Rehash)。

### 在 OracleJDK 中的实现

OracleJDK 8 中 HashMap 扩容算法是将 oldTable 中哈希槽上的链表整体迁移到 newTable 中，该过程中会 **保留哈希槽上的链表元素顺序**，因此不会产生如 OpenJDK 8 中 HashMap 的死循环问题，一次扩容示例过程如下：

```plain text
// 扩容之前
+-+
| |
+-+  +-+  +-+
|3|->|7|->|5|->NULL
+-+  +-+  +-+

// 扩容之后
+-+
| |
+-+
|5|->NULL
+-+
| |
+-+  +-+
|3|->|7|->NULL
+-+  +-+
```
