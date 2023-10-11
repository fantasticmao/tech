# Prometheus 数据模型和指标类型

## 数据模型

从本质上来说，Prometheus 把所有数据存储为时序类型：按照指标和多维标签来划分的、带有时间戳的数据流。Prometheus 除了存储时序数据之外，可能还会生成临时的时序数据作为查询结果。

### 指标名称和标签

Prometheus 中的每个时序数据都由它的指标名称和被称为标签的可选的 key-value 键值对来被唯一标识。

指标名称指定了被测量系统的一般功能（例如 `http_requests_total` 代表了收到 HTTP 请求的总数）。指标名称可以包含 ASCII 字符、数字、下划线和冒号，并且必须满足如下的正则表达式 `[a-zA-Z_:][a-zA-Z0-9_:]*`。

标签可以使 Prometheus 支持多维度的数据模型：对于相同指标名称的不同标签组合，都可以标识该指标的特定维度的实例（例如所有使用 `POST` 方法的发送给 `/api/tracks` 处理器的 HTTP 请求）。Prometheus 的查询语言支持基于这些标签维度来过滤和聚合数据，更改标签值，包括添加和删除标签，都会创建一组新的时序数据。

标签名称可以包含 ASCII 字符、数字和下划线，并且必须满足如下的正则表达式 `[a-zA-Z_][a-zA-Z0-9_]*`，标签值可以包含任意的 Unicode 字符。

标签值为空的标签等同于该标签不存在。

### 数据样本

数据样本形成了真实的时序数据，每个数据样本包含了：

- 一个 `float64` 类型的值
- 一个毫秒精度的时间戳

### 表达式

给定一个指标名称和一组标签，时序数据通常使用如下表达式来标识：

```promql
<metric name>{<label name>=<label value>, ...}
```

例如，指标名称为 `api_http_requests_total`、带有标签 `method="POST"` 和 `handler="/messages"` 的时序数据可以写作为：

```promql
api_http_requests_total{method="POST", handler="/messages"}
```

这与 [OpenTSDB](https://opentsdb.net/) 使用的表达式相同。

## 指标类型

Prometheus 客户端库中提供了四种核心的指标类型，这些指标类型目前仅在客户端库和查询 API 中有所区别。Prometheus 服务端目前尚未使用这些类型信息，并会将所有数据扁平化为无类型的时序数据。这一点可能会在未来的版本中改变。

### Counter

counter 是一个累计的指标，代表了一个单调递增的计数器，它的值只能增加或者是在系统重启时重置为零。例如，可以使用 counter 来表示处理过的 HTTP 数量、完成的任务数量、错误数量等。

不要使用 counter 来表示可以减少的值。例如，不要使用 counter 来表示正在运行的进程数，而是使用 gauge。

### Gauge

gauge 是一个指标，代表可以任意上下浮动的单个数值。

gauge 通常用于表示测量值，例如温度或者当前内存的使用情况，也能用于表示可以上下浮动的计数值，例如当前并发请求的数量。

### Histogram

histogram 会对观测数据进行采样（通常是请求的持续时间和响应的内容大小），并将它们计入可配置的槽中。histogram 也可以提供所有被观测的数值总和。

指标名称为 `basename` 的 histogram 指标类型，在一次数据采集过程中可以暴露多个时序数据：

- 观测槽中的累计数值，会被暴露为 `<basename>_bucket{le="<upper inclusive bound>"}`
- 所有被观测的数值总和，会被暴露为 `<basename>_sum`
- 已经观测到的事件总数，会被暴露为 `<basename>_count`（与上文中的 `<basename>_bucket{le="+Inf"}` 写法相同）

使用 `histogram_quantile()` 函数可以根据 histogram 甚至是 histogram 的聚合结果中计算百分位数值。histogram 也可以用来计算 [Apdex](https://en.wikipedia.org/wiki/Apdex) 分数。在观测槽中计算时，需要注意 histogram 的数值是累计的。

### Summary

summary 与 histogram 类似，也会对观测数据进行采样。summary 不仅提供了观测的事件总数，以及观测的数值总和，还计算了滑动时间窗口内的可配置的百分位数值。

指标名称为 `basename` 的 summary 指标类型，在一次数据采集过程中可以暴露多个时序数据：

- 被观测事件的流式 φ 分位数值，会被暴露为 `<basename>{quantile="<φ>"}`
- 所有被观测的数值总和，会被暴露为 `<basename>_sum`
- 已经观测到的事件总数，会被暴露为 `<basename>_count`

## 参考资料

- [Data Model](https://prometheus.io/docs/concepts/data_model/)
- [Metric Types](https://prometheus.io/docs/concepts/metric_types/)
