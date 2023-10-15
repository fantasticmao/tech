# RocketMQ 架构设计

```
                  +----------------------------------------+
 Broker Discovery | +----------+ +----------+ +----------+ |      Broker Discovery
      +---------->| |NameServer| |NameServer| |NameServer| |<-----------+
      |           | +----------+ +----------+ +----------+ |            |
      |           +----------------------------------------+            |
      |                   ^    ^    ^    ^    ^    ^                    |
      |                   |    | routing info |    |                    |
      |                   v    v    v    v    v    v                    |
+------------+       +----------------------------------+         +------------+
| +--------+ |       | +------------+    +------------+ |         | +--------+ |
| |Producer| |       | |BrokerServer|    |BrokerServer| |         | |Consumer| |
| +--------+ |       | |   Master   |    |   Master   | |         | +--------+ |
|            |       | +------------+    +------------+ |         |            |
| +--------+ | send  |       ^                 ^        | receive | +--------+ |
| |Producer| |------>|       |     Data Sync   |        |-------->| |Consumer| |
| +--------+ |  msg  |       v                 v        |   msg   | +--------+ |
|            |       | +------------+    +------------+ |         |            |
| +--------+ |       | |BrokerServer|    |BrokerServer| |         | +--------+ |
| |Producer| |       | |   Slave    |    |   Slave    | |         | |Consumer| |
| +--------+ |       | +------------+    +------------+ |         | +--------+ |
+------------+       +----------------------------------+         +------------+
```

## 技术架构

RocketMQ 在架构设计上由四个部分组成：Name Server、Broker Server、Producer、Consumer。

### Name Server

Name Server 是轻量化的 topic 注册中心，支持 Broker Server 的动态注册发现、路由信息的管理。Name Server 提供心跳检测机制，定时检查 Broker 是否存活。

Producer 和 Consumer 通过 Name Server 获取整个 Broker Server 集群的路由信息，从而进行消息的投递和消费。

Name Server 集群实例之间不会互相通讯，但是 Broker Server 会向所有的 Name Server 注册路由信息，所以每个 Name Server 实例上都保存了完整的路由信息。

### Broker Server

Broker Server 负责消息的存储、投递、查询，以及保证服务的高可用，支持容错机制、灾备机制、报警机制和丰富的监控指标。

Broker Server 包含以下几个重要的子模块

- Remoting Module 负责处理来自 client 的请求；
- Client Manager 负责管理 Producer/Consumer 和维护 Consumer 的 topic 订阅信息；
- Store Service 负责存储消息至磁盘、查询消息等功能；
- HA Service 负责 Master Broker 和 Slave Broker 之间的数据同步；
- Index Service 负责消息的索引服务，支持快速查询消息。

### Producer

Producer 是发布消息的角色，通过多种负载均衡的方式，选择相应的 Broker Server 集群中的 Queue 发送消息。Producer 在发送消息时，支持快速失败，并且是低延迟的。

### Consumer

Consumer 是消费消息的角色，支持 PUSH 和 PULL 两种获取消息模式，支持集群和广播两种消费消息模式。

## 部署架构

在 RocketMQ 集群部署过程中，各个部分有如下特点：

- Name Server 集群之中的各个节点之间不会互相同步数据，各个节点近乎是无状态的；
- Broker Server 集群支持以单 Master 多 Salve 模式部署，Master 与 Salve 的对应关系通过指定相同的 BrokerName 和不同的 BrokerId 来定义，BrokerId 为 0 表示 Master 节点，非 0 表示 Salve 节点。Broker Server 集群中的各个节点都会与 Name Server 集群之中的所有节点建立长链接，定时注册 topic 信息至所有 Name Server 节点中；
- Producer 会随机与 Name Server 集群中的一个节点建立长连接，定期从 Name Server 获取 topic 路由信息，并会与 topic 所在的 Broker Server 的 Master 节点建立长连接，并且会定时向 Master 发送心跳。Producer 集群完全是无状态的，可以随意集群部署；
- Consumer 会随机与 Name Server 集群中的一个节点建立长连接，定期从 Name Server 获取 topic 路由信息，并会与 topic 所在的 Broker Server 的 Master 和 Salve 节点建立长连接，并且会定时向 Master 和 Salve 发送心跳。Consumer 既可以从 Master 订阅消息，也可以从 Salve 订阅消息，Consumer 在获取消息的时候，Broker Server 的 Master 节点会根据获取消息的偏移量与最大偏移量的距离、服务器是否可读等因素建议 Consumer 下次是从 Master 或者 Salve 获取消息。

整体来看，RocketMQ 集群的工作流程如下：

1. Name Server 启动之后，等待 Broker Server、Producer、Consumer 的连接。Name Server 相当于一个路由控制中心；
2. Broker Server 启动之后，会与所有的 Name Server 建立长连接，定时发送心跳。心跳包中包含当前 Broker Server 的信息（ip + port 等），以及它所存储的所有 topic 信息。在 Broker Server 向 Name Server 注册成功之后，Name Server 中便有所有 topic 与 Broker Server 的映射关系；
3. 在发送消息之前，会需要先创建 topic。在创建 topic 时，需要指定 topic 存储在哪台 Broker Server 上。RocketMQ 也支持在发送消息时自动创建 topic；
4. Producer 启动之后会与 Name Server 集群中的一个节点建立长连接，并从 Name Server 确定待发送消息的 topic 所在的 Broker Server 节点，然后轮训从 queue 列表中选择一个 queue，并与该 queue 所在的 Broker Server 建立长连接，最后向该 Broker Server 发送消息；
5. Consumer 与 Producer 类似，会与 Name Server 集群中的一个节点建立长连接，并从 Name Server 获取已订阅的 topic 所在的 Broker Server 节点，然后与该 Broker Server 建立长连接，最后开始消费消息。

## 主流 MQ 对比

|              | RocketMQ             | Kafka                              | RabbitMQ                                          |
| ------------ | -------------------- | ---------------------------------- | ------------------------------------------------- |
| 单机 TPS     | 11.6w/s<br>吞吐量高  | 17.3w/s<br>吞吐量高                | 5.95w/s<br>实现 AMQP 协议，舍弃吞吐量，保证可靠性 |
| 持久化       | 磁盘（支持大量堆积） | 内存、磁盘、数据库（支持大量堆积） | 内存、磁盘                                        |
| 消息可靠性   | 高                   | 一般                               | 高                                                |
| 事务消息     | 支持                 | 支持                               | 支持                                              |
| 消息重试     | 支持                 | 不支持                             | 支持                                              |
| 定时消息     | 支持                 | 不支持                             | 不支持                                            |
| 消息加密     | 不支持               | SSL                                | SSL                                               |
| 查询消息轨迹 | 支持                 | 不支持                             | 不支持                                            |
| 消息回溯     | 支持指定时间点回溯   | 支持队列 offset 回溯               | 不支持                                            |
