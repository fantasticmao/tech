# Prometheus vs InfluxDB

[InfluxDB](https://www.influxdata.com/) 是一个开源的时许数据库，商业版本额外提供「弹性扩展」和「集群部署」的特性。Prometheus 和 InfluxDB 之间有着显著的差异，两者各自面向不同的使用场景。

## 数据模型 / 存储

与 Prometheus 类似，InfluxDB 数据模型也将 key-value 键值作为标签信息。另外 InfluxDB 还有二级标签，被称为字段（fields），但在使用上会受到更多限制。InfluxDB 支持的时间戳精度高达纳秒级别，还支持 `float64`、`int64`、`bool` 和 `string` 数据类型。相比之下，Prometheus 只支持毫秒级别的时间戳精度，还有 `float64` 和 `string` 数据类型（==FIXME==）。

InfluxDB 使用一种 [LSM Tree](https://en.wikipedia.org/wiki/Log-structured_merge-tree) 变种的数据结构来存储预写日志（write ahead log，WAL），并且会以时间来进行数据分片。相比 Prometheus 的 AOF（append only file）方式（==FIXME==），InfluxDB 更加适合记录事件日志的场景。

## 架构设计

Prometheus 服务彼此之间独立运行，并且仅依靠本地存储来实现 Prometheus 的核心功能：数据抓取、规则处理，以及告警。开源版本的 InfluxDB 也是类似。

商业版的 InfluxDB 提供了一个分布式的存储集群，存储和查询将由多个节点同时处理。这意味着商业版的 InfluxDB 更加易于水平扩展，但也意味着你必须从一开始就要接受和管理分布式存储系统带来的复杂性。

Prometheus 运行起来会更加简单，但在某些时候，你将需要按照产品、服务、数据中心或者类似的边界条件来显式地对 Prometheus 服务进行划分。Prometheus 服务之间独立运行（可以并行运行来冗余数据）的方式，更加有利于服务的可靠性和故障隔离。

## 总结

InfluxDB 和 Prometheus 两者之间有很多相似之处。两者都有用于高效支持多维指标的标签，都使用基本相同的数据压缩算法，都有广泛的集成（包括对彼此生态的集成），都有易于扩展的钩子（例如在统计工具中分析数据、执行自动化操作）。

InfluxDB 更加适用的场景：

- 如果需要记录日志事件
- 商业版的 InfluxDB 提供的集群模式，更加适用于数据的长期存储
- 如果需要多个数据副本之间的最终一致性

Prometheus 更加适用的场景：

- 如果主要用于记录监控指标
- 更加强大的查询语言和告警功能
- 对监控图表和告警功能需要更高的可靠性

InfluxDB 是由一家商业公司按照开放核心功能的模式来维护，并提供了闭源的高级功能，例如集群模式、服务托管和技术支持。Prometheus 是一个完全开源和独立的项目，由多家公司和个人来维护，其中还有一些公司会提供商业服务和技术支持。

## 参考资料

- [Comparison to alternatives](https://prometheus.io/docs/introduction/comparison/#prometheus-vs-influxdb)
- [InfluxDB storage engine](https://docs.influxdata.com/influxdb/v2.1/reference/internals/storage-engine/)
