# Prometheus 概览

## 简介

Prometheus 是一个开源的系统监控和告警工具。

Prometheus 收集和存储指标信息为时序数据，Prometheus 在存储指标信息时候会记录当前时间戳，以及被称为标签的可选的 key-value 键值对信息。

## 特性

Prometheus 的主要特性如下：

- 通过指标名称和键值对来标识的多维时序数据模型
- 充分利用多维度数据的、灵活的查询语言 PromQL
- 不依赖于分布式存储；每个服务节点都具有自治能力
- 通过 HTTP pull 模式收集时序数据
- 通过中介网关来实现 push 时序数据
- 通过服务发现或者静态配置来发现监控对象
- 支持多种图标和仪表盘

## 指标

用外行的话来说，指标就是测量的数据，时序则意味着随时间的推移而记录的变化。用户对不同的应用程序期望测量的指标大不相同，对于 Web 应用来说可能是请求次数，对于数据库来说可能是活跃连接数、活跃查询数等等。

指标对于理解应用程序特定的方式运行来说，扮演着重要的角色。假如你有一个正在运行的 Web 应用程序，并且发现它运行得很慢。那么你将会需要一些信息来了解你的应用程序究竟发生了什么。例如是由于请求数变高，导致应用程序运行变慢。如果你有请求次数的指标的话，你便可以方便得找出问题原因，然后通过增加服务器数量来承受更高的负载。

## 组件

Prometheus 的生态系统由多个组件构成，大多数是可选的：

- 最主要的 [Prometheus Server](https://github.com/prometheus/prometheus)，用于抓取和存储时序数据
- [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/)，用于检测应用程序
- [push gateway](https://github.com/prometheus/pushgateway)，用于支持临时的任务
- 定制化的 [exporters](https://prometheus.io/docs/instrumenting/exporters/)，用于检测各种服务，例如 HAProxy、StatsD、Graphite 等等
- [alertmanager](https://github.com/prometheus/alertmanager)，用于处理告警
- 各种各样的工具

Prometheus 的大部分组件使用 Go 编写的，这使它们非常易于构建和部署。

## 架构

下图展示了 Prometheus 的架构及其一些系统的组件：

![architecture.png](architecture.png)

Prometheus 可以使用直接抓取或者通过中介网关来间接抓取的方式，从检测任务中抓取指标信息。Prometheus 会将所有抓取的样本数据存储在本地，并基于这些数据来运行规则，用于聚合和记录新的时序数据或者是生成告警。Grafana 和其它使用 Prometheus API 的客户端负责可视化这些数据。

## 适用场景

Prometheus 适用于记录纯数字的时序数据。它即适合以机器为中心的监控，也适合高度动态的微服务架构的应用监控。

## 不适用场景

Prometheus 不适用于对数据精确度要求很高的场景，例如「按请求次数计费」，因为它收集的数据不够详细和完整。

## 参考资料

- [Prometheus Overview](https://prometheus.io/docs/introduction/overview/)
