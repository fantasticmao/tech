# HBase 数据模型

在 HBase 中，数据被存储在表中，表中有行和列。这与关系型数据库中的术语相似，但不是一个恰当的比喻。相反，将 HBase 表视为多维度的映射（map）可能更为恰当。

HBase 数据模型术语：

- **表 Table**
    一个 HBase 表由多个行组成。

- **行 Row**
    一个 HBase 行由一个行键（row key）和一个或多个与行键关联的带有值的列组成。行在存储时，按照行键的字母顺序存储。因此，行键的设计十分重要。目的是以相关的行彼此靠近的方式存储数据。

- **列 Column**
    一个 HBase 列由一个列族（column family）和一个列限定符（column qualifier）组成，以 `:` 冒号分割。

- **列族 Column Family**
    出于性能原因，列族是被设计为在物理上并列存储的一组列和列的值。每个列族都有一组存储属性，例如列的值是否应该被缓存在内存中，数据如何被压缩，行键如何被编码等。

    表中的每行都有相同的列族，尽管某些行可能不会在某些列族中存储任何数据。

- **列限定符 Column Qualifier**
    列限定符是被添加到列族上的，为一组数据提供的索引。对于一个给定的列族 `content`，列限定符可以是 `content:html` 或者 `content:pdf`。

    列族在表创建时就固定不变，列限定符是动态可变的，并且不同行之间的列限定符可能会有很大的差异。

- **单元格 Cell**
    单元格式行、列族、列限定符和组合，并且还包含了一个值和一个时间戳，表示值的版本。

- **时间戳 Timestamp**
    每个值写入的时候都会附带一个时间戳，它是一个值的版本的标志符。

    默认情况下，时间戳表示写入数据时 RegionServer 上的时间，但也可以在将数据写入单元格时指定不同的时间戳。

## 概念视图

下面这个例子是 [BigTable](https://research.google/pubs/pub27898/) 论文第二页中的示例的略微修改版。

这是一张叫 `webtable` 的表，包含了两个行（`com.cnn.www`、`com.example.www`）和三个列族（`contents`、`anchor`、`people`）。

在这个示例中，对于第一个行 `com.cnn.www`，`anchor` 列族包含了两个列（`anchor:cnnsi.com`、`anchor:my.look.ca`），`contents` 列族包含了一个列（`contents:html`）。

这个示例包含 `com.cnn.www` 行键的 5 个版本，`com.example.www` 行键的 1 个版本。

| Row Key           | Timestamp | Column Family `contents`    | Column Family `anchor`      | Column Family `people`   |
| ----------------- | --------- | --------------------------- | --------------------------- | ------------------------ |
| "com.cnn.www"     | t9        |                             | anchor:cnnsi.com="CNN"      |                          |
| "com.cnn.www"     | t8        |                             | anchor:my.look.ca="CNN.com" |                          |
| "com.cnn.www"     | t6        | contents:html="\<html\>..." |                             |                          |
| "com.cnn.www"     | t5        | contents:html="\<html\>..." |                             |                          |
| "com.cnn.www"     | t3        | contents:html="\<html\>..." |                             |                          |
| "com.example.www" | t5        | contents:html="\<html\>..." |                             | people:author="John Doe" |

在 HBase 中，表中为空的单元格不占用存储空间，实际上也并不存在。这是 HBase 表「稀疏」的原因。

下面通过一个多维度的映射结构，表示了与 `webtable` 表相同的信息。

```json
{
  "com.cnn.www": {
    "contents": {
      "t6": {
        "contents:html": "<html>..."
      },
      "t5": {
        "contents:html": "<html>..."
      },
      "t3": {
        "contents:html": "<html>..."
      }
    },
    "anchor": {
      "t9": {
        "anchor:cnnsi.com": "CNN"
      },
      "t8": {
        "anchor:my.look.ca": "CNN.com"
      }
    },
    "people": {
    }
  },
  "com.example.www": {
    "contents": {
      "t5": {
        "contents:html": "<html>.."
      }
    },
    "anchor": {
    },
    "people": {
      "t5": {
        "people:author": "John Dor"
      }
    }
  }
}
```

## 物理视图

虽然在概念层面上的表可能被视为一组稀疏的行，但在物理层面上的表是由列族来存储的。

一个新的列限定符（column_family:column_qualifier）可以随时被添加到已经存在的列族中。

| Row Key       | Timestamp | Column Family `anchor`      |
| ------------- | --------- | --------------------------- |
| "com.cnn.www" | t9        | anchor:cnnsi.com="CNN"      |
| "com.cnn.www" | t8        | anchor:my.look.ca="CNN.com" |

| Row Key       | Timestamp | Column Family `contents`    |
| ------------- | --------- | --------------------------- |
| "com.cnn.www" | t6        | contents:html="\<html\>..." |
| "com.cnn.www" | t5        | contents:html="\<html\>..." |
| "com.cnn.www" | t3        | contents:html="\<html\>..." |

在概念视图中的空单元格不会被存储，因此在 `t8` 时间戳上查询 `contents:html` 列的值，将不会返回任何内容。

然而，如果没有提供时间戳字段，则会返回给定列的最新值。如果提供了多个时间戳字段，则会返回第一个找到的值，也是最新的值，因为时间戳是按倒序存储的。因为在 `com.cnn.www` 行键上查询所有列的数据，如果没有指定时间戳的话，将会返回 `t6` 时间戳的 `contents:html` 列的值、`t9` 时间戳的 `anchor:cnnsi.com` 列的值、`t8` 时间戳的 `anchor:my.look.ca` 列的值。
