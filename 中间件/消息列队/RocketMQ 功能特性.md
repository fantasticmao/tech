# RocketMQ 功能特性

## 发布与订阅

消费者可以通过订阅 topic 机制，来消费生产者向该 topic 发出的消息。RocketMQ 中的消息可以再以 tag 属性区分。

## 消息顺序

顺序消息是指需要按照发送顺序被消费的一类消息。例如一个订单产生了三条消息：订单创建、订单付款、订单完成，此时在业务上，便会要求消息需要按照发送时的顺序来被消费。

RocketMQ 支持全局消息有序和分区消息有序：

- 全局有序：对于指定 topic 下的所有消息，严格按照先进先出（FIFO）的顺序进行消费。适用于对性能要求不高、对消息顺序有严格要求的场景；
- 分区有序：对于指定 topic 下的一组消息，使用 `MessageQueueSelector` 来依据 sharding key 进行分区来实现消息的有序性。适用于对性能要求高、允许同一个区块内的消息有序即可的场景。

## 消息过滤

RocketMQ 支持通过 tag 属性来对消息进行过滤，也支持自定义其它属性。

RocketMQ 的消息过滤是在 Broker Server 端实现的，这样做的好处是减少了对 Consumer 来说多余的网络开销，缺点是增加了 Broker Server 的负担和实现复杂度。

## 消息可靠性

RocketMQ 支持存储消息时的高可靠，影响消息可靠性的因素主要如下：

1. Broker Server 非正常关闭
2. Broker Server 崩溃
3. 操作系统崩溃
4. 机器断电
5. 机器无法开机
6. 磁盘损坏

在 1、2、3、4 的情况都属于硬件资源可以立即恢复的情况，RocketMQ 可以保证消息不丢失或者少量丢失（取决于数据写入磁盘的方式是同步还是异步）。5、6 属于单点故障，一旦发生时此单点上的数据无法恢复，数据将会全部丢失。在这种情况下，RocketMQ 可以通过 Master + Salve 同步数据的方式避免单点故障，保障 99% 的消息不会丢失。但是在同步数据时仍然可能会导致数据丢失，此时可以通过 RocketMQ 同步双写机制来避免数据丢失，不过同步双写机制对性能并不友好，仅适用于一些对消息可靠性要求极高的场景。

## 至少一次（消费消息时）

RoekctMQ 支持保证消息至少被消费一次。Consumer 在消费完消息之后，会向 Broker Server 返回一个 ack 来表示消息已经被消费了。

但是 RoekctMQ 无法保证消息只会被消费一次，即 Consumer 消费的消息时可能会是重复的。因此 Consumer 端需要在消费消息时进行幂等处理。

## 消息重试（消费消息时）

消息重试是指在 Consumer 消费消息失败之后，需要使 Consumer 再次消费消息的机制。

Consumer 消费消息失败主要有以下两种情况：

1. 由于消息本身的原因，例如反序列化失败，消息数据异常等。这种情况下一般的建议是跳过这条消息，先消费其它消息，经过一段时间之后再进行重试该消息。因为即使立即重试该消息，Consumer 大概率还是消费失败的；
2. 由于下游应用可不用，例如数据库不可用、调用接口超时等。这种情况下一般的建议是使应用等段一段时间之后再重试消息。因为此时 Consumer 消费其它消息也会是失败的，反而应用在等待这段时间内可以减轻 Broker Server 重试消息的压力。

### 源码分析 - 消费消息

以集群模式的消费消息为例，从 RocketMQ 源码中看它对于消息消费模块的具体实现，大致步骤如下：

1. 先加载 Consumer 待消费消息的偏移量（相关代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/impl/consumer/DefaultMQPushConsumerImpl.java#L579-L594)）；
2. 再从 MessageQueue 中 pull Consumer 所订阅的相关消息（相关代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/impl/consumer/DefaultMQPushConsumerImpl.java#L199-L436)，这其中包含了当对 Consumer 的流量控制）；
3. 在 pull 消息成功之后，RocketMQ 会在回调代码中将相关消息封装成一个消费请求，然后提交给 Consumer 本地的消费消息线程池（相关代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/impl/consumer/DefaultMQPushConsumerImpl.java#L312-L316)）；
4. 最后在相关消息的消费请求中，RocketMQ 会将消息委托给业务代码处理（相关代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/impl/consumer/ConsumeMessageConcurrentlyService.java#L385-L471)）。

## 消息重投（发送消息时）

消息重投是指在 Producer 发送消息失败之后，需要使 Producer 再次发送消息的机制。RocketMQ 对于同步消息会尝试重新发送，对于异步消息会尝试简单地重新发起网络请求，对于 oneway 消息会忽略。

RocketMQ 在消息重投的时候可能会造成消息重复，因此需要 Consumer 端需要在消费消息时进行幂等处理。

### 源码分析 - 发送消息

以发送普通的同步消息为例，从 RocketMQ 源码（入口代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/impl/producer/DefaultMQProducerImpl.java#L515-L658)）中看它对于消息发送模块的具体实现，大致步骤如下：

1. 先从本地缓存或者 Name Server 中获取 topic 相关的路由信息（相关代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/impl/producer/DefaultMQProducerImpl.java#L660-L675)）；
2. 再从 topic 路由信息中选择一个可用的 MessageQueue（相关代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/latency/MQFaultStrategy.java#L58-L93)）；
3. 最后使用 MessageQueue 来连接对应的 Broker Server，并通过底层的 sendKernelImpl 方法来发送消息（相关代码请见 [GitHub](https://github.com/apache/rocketmq/blob/rocketmq-all-4.4.0/client/src/main/java/org/apache/rocketmq/client/impl/producer/DefaultMQProducerImpl.java#L677-L852)）。

## 流量控制

对 Producer 的流量控制是发生在 Broker Server 处理能力达到瓶劲的时候，对 Consumer 的流量控制是发生在 Consumer 消费能力达到瓶劲的时候。

## 消息回溯

消息回溯是指 Consumer 在消费完消息之后，由于业务上的要求，该消息需要重新被消费。RocketMQ 支持按照单位为毫秒的时间维度来回溯消息。

## 定时/延时消息

定时（延时）消息时指 Producer 在发送消息到 Broker Server 之后，Broker Server 会在特定时间之后再投递给 Consumer。RocketMQ 通过 Broker Server 的 messageDelayLevel 属性来实现定时消息。

## 死信消息/死信队列

死信队列是用于处理异常的消息。当一条消息最初被消费失败时，RocketMQ 会进行消息重试。当该消息达到最大重试次数之后依然消费失败时，即当 Consumer 无法正常消费该消息时，RocketMQ 会将该消息发送到一个特殊的队列中。

RocketMQ 将无法被正常消费的消息成为死信消息（Dead-Letter Message），将存储死信消息的队列成为死信队列（Dead-Letter Queue）。在 RocketMQ 中可以通过控制台来查看和重新发送死信消息。

## 事务消息
