# ElasticSearch 概览

## 和关系型数据库（MySQL）术语的区别

| ElasticSearch | RDBMS（MySQL） |
| ------------- | -------------- |
| 索引 Index    | 表 Table       |
| 文档 Document | 行 Row         |
| 字段 Field    | 列 Column      |
| 映射 Mapping  | 表结构 Schema  |
| DSL           | SQL            |

## 正排索引和倒排索引的区别

正排索引（forward index）：document -> content

倒排索引（inverted index）：content -> document

倒排索引的级别：

1. record-level 包含「单词 -> 文档」的映射关系；
2. word-level 包含「单词 -> 文档和单词在文档中的位置」的映射关系。

例如存在以下数据：

| ID  | Content                 |
| --- | ----------------------- |
| 1   | Mastering Elasticsearch |
| 2   | Mastering Go            |
| 3   | Elasticsearch Reference |

那么 ID 列的正排索引为：

```plain text
1 -> (1, "Mastering Elasticsearch")
2 -> (2, "Mastering Go")
3 -> (3, "Elasticsearch Reference")
```

Content 列 record-level 级别的倒排索引为：

```plain text
"Mastering" -> [1, 2]
"Elasticsearch" -> [1, 3]
"Go" -> [2]
"Reference" -> [3]
```

Content 列 word-level 级别的倒排索引为：

```plain text
"Mastering" -> [1:0, 2:0]
"Elasticsearch" -> [1:1, 3:0]
"Go" -> [2:1]
"Reference" -> [3:1]
```
