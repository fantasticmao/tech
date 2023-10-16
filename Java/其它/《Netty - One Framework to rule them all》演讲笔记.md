# 《Netty - One Framework to rule them all》演讲笔记

## 作者介绍

这次演讲的讲师是 Netty 项目领导者，同时也是经典书籍 [《Netty in Action》](https://book.douban.com/subject/24700704/) 的作者 [Norman Maurer](https://github.com/normanmaurer)。

## 演讲视频

<https://www.bilibili.com/video/BV14t411G7Lp>

## Netty 4 中的优化

Netty 3 存在诸多明显的设计缺陷：

- 很多内存垃圾（在高并发的场景下，GC 的负担会变得更重）；
- 很多内存拷贝（Java Heap -> OS 内存 -> Socket）；
- 缺少一个好的内存池（频繁地创建和销毁直接内存将会非常损耗性能）；
- 缺少对于 Linux 操作系统的优化（不能使用 Linux 操作系统下的特性）；
- 不太合理的线程模型（Inbound 和 Outbound 的操作是在不同线程中的）。

Netty 4 对以上问题做了优化：

1. 产生更少的内存垃圾，也会更少地触发 GC；
2. 使用 JNI 来支持某些 Linux 操作系统下仅有的特性；
3. 提供了一个高性能的 Buffer Pool；
4. 定义了一个易于使用的线程模型。

## ChannelPipeline

Netty 中的 Channel 是对 Socket 的抽象，可以用于网络数据的读取和写入。每个 Channel 会对应一个 ChannelPipeline，ChannelPipeline 是一个包含了不同 ChannelHandler 的双向链表，ChannelHandler 可以用于处理 Inbound 和 Outbound 事件。

```plain text
+------------------+
|    Channel       |
|                  |
|          +-------+----------------------------------+
|          |       |           ChannelPipeline        |
+----------+-------+                                  |
  ^   |    |                                          |
  |   |    |     +-------+   +-------+   +-------+    |
  |   v    |     |Channel|   |Channel|   |Channel|    |
  |  IN -->|---->|Inbound|-->|Inbound|-->|Inbound|    |
  |        |     |Handler|   |Handler|   |Handler|    |
  |        |     +-------+   +-------+   +-------+    |
  |        |                                          |
  |        |        +--------+      +--------+        |
  |        |        |Channel |      |Channel |        |
  OUT <----|<-------|Outbound|<-----|Outbound|        |
           |        |Handler |      |Handler |        |
           |        +--------+      +--------+        |
           |                                          |
           +------------------------------------------+
```

## 减少内存垃圾

Netty 中引入了一个非常轻量级的对象池。在 Netty 中存在着许多可以被复用的对象，并且这些对象会被限制在同一个线程中，例如在把 Byte 转换为 List\<Object\> 的编解码器中，Netty 不需要每次都创建一个新的 List，而是会把这个 List 缓存起来并且会重用它。

## 使用 JNI

JDK 中的 NIO 是对 Socket 的抽象，可以运行在不同操作系统上，但是 NIO 并没有针对具体的操作系统来做优化。

Netty 使用 JNI 来提供了 NIO 中所有没有提供的、与操作系统相关的高级特性，例如 epoll。

```java
// NIO Transport
Bootstrap bootstrap = new Bootstrap()
            .group(new NioEventLoopGroup())
            .channel(NioSocketChannel.class);

// Native Transport
Bootstrap bootstrap = new Bootstrap()
            .group(new KQueueEventLoopGroup())
            .channel(EpollSocketChannel.class);
```

## 更友好的 Buffers

JDK 中提供的 ByteBuffer 接口并不友好，Netty 中实现了一套更加友好的 Buffer 实现：ByteBuf。

Netty 中的 ByteBuf 是基于引用计数算法来管理内存的，并且 Netty 提供了一个内存溢出的检测器：`ResourceLeakDetector`，可以用于检测潜在的资源泄漏问题。

## 线程模型

Netty 中的 `EventLoop` 是对 I/O 线程的抽象，用于处理 I/O 操作和触发事件。`EventLoop` 继承了 `EventExecutor`，`EventExecutor` 继承了 `ScheduledExecutorService`，这意味着开发者可以在一个处理 Channel 的 I/O 线程里编排事件。

Netty 提供了 `EventExecutor`，可以用于处理 Handler 中的一些可能会导致阻塞的事件。

## 与 JVM

当在 Java 中做很多底层性能相关的事情，例如网络编程，便会经常接触到 JVM。

在 JVM 中，对直接内存的管理仅是通过 finalizer 或 cleaner 机制来实现的，并且 JVM 只有在堆空间耗尽的时候才会进行 GC。然而，Netty 在被用于网络编程的时候，可以确切地知道某个对象在何时应该被回收，因此 JVM 对直接内存的管理方式并不适用于 Netty。

在 JVM 中，JIT 编译器会重新布局 class 文件，为了减少内存间隙和避免浪费内存。但有时，这会产生内存伪共享（False-Sharing）问题。

在 JDK 中的 NIO 也存在诸多问题：

- NIO2 并没有真正地提供 NIO 的可用性；
- NIO 会产生很多内存垃圾，例如 Selector、Keyset 等等；
- ByteBuffer API 并不友好；
