# 《Java 并发编程实战》笔记 - 线程池的使用

## 线程安全性

要编写线程安全的代码，其核心在于要 **对状态访问操作进行管理**，特别是对共享（Shared）和可变（Mutable）状态的访问。「共享」意味着变量可以由许多个线程同时访问，「可变」意味着变量的值在其生命周期内可以发生变化。

从非正式的意义上来说，对象的状态是指存储在状态变量（例如实例或者静态域）中的数据。对象的状态可能包括其他依赖对象的域，例如某个 `HashMap` 的状态不仅存储在 `HashMap` 对象本身，还存储在许多 `Map.Entry` 对象中。

实现线程安全的方式有以下三种：

1. 不在线程之间共享状态变量；
2. 将状态变量设置为不可变的；
3. 在访问状态变量时，使用同步机制。

Java 中主要的同步机制是 `synchronized` 关键字，它是一种独占的加锁方式，但是「同步」还可以包括 `volatile` 类型的变量、`Lock` 显示锁、`AtomicLong` 等原子变量。

### 什么是线程安全性

简单版：当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的。

详细版：当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替执行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么就称这个类是线程安全的。

## 线程池的使用：配置 ThreadPoolExecutor

`ThreadPoolExecutor` 为 `Executor` 提供了基本的实现，这些 `Executor` 是由 `Executors` 中的 `newCachedThreadPool`、`newFixedThreadPool`、`newScheduledThreadExecutor` 等工厂方法返回的。`ThradPoolExecutor` 是一个灵活的、稳定的线程池，允许进行各种定制。

如果默认的执行策略不能满足需求，那么可以通过 `ThreadPoolExecutor` 的构造参数，根据自己的需求定制一个对象。`ThreadPoolExecutor` 的所有构造参数如下：

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler) { ... }
```

### 线程的创建与销毁

线程池的基本大小 `corePoolSize`、最大大小 `maximumPoolSize`、以及存活时间 `keepAliveTime` + `unit` 等因素共同负责线程的创建与销毁：

1. `corePoolSize` 是线程池的目标大小，即在没有任务执行时的线程池大小，**并且只有在工作队列 `workQueue` 满了的情况下，才会创建超出这个数量的线程**；
2. `maximumPoolSize` 表示可同时活动的线程数量的上限；
3. 如果某个线程的空闲时间超过了存活时间 `keepAliveTime` + `unit`，那么将被标记为可回收，并且在当线程池的当前大小超过了 `corePoolSize` 时，这个线程将被终止。

`newFixedThreadPool` 工厂方法将线程池的 `corePoolSize` 和 `maximumPoolSize` 设置为参数中指定的值，并且创建的线程池不会超时。

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

`newCachedThreadPool` 工厂方法将线程池的 `maximumPoolSize` 设置为 `Integer.MAX_VALUE`，`corePoolSize` 设置为 `0`，并且超时时间为一分钟。

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

### 管理任务队列

如果新请求的到达速率超过了线程池的处理速率，那么新到来的请求将累积起来。在线程池中，这些请求会在一个 `Runnable` 队列 `workQueue` 中等待，而不会像线程那样去竞争 CPU 资源。`ThreadPoolExecutor` 允许提供一个 `BlockingQueue` 来保存等待执行的任务，基本的任务排队方式有三种：无界队列、有界队列、同步移交。队列的选择与其它线程池配置参数有关。

`newFixedThreadPool` 和 `newSingleThreadExecutor` 工厂方法将使用一个无界的 `LinkedBlockingQueue`。如果所有工作者线程都处于忙碌状态，并且任务持续快速地到达，**那么 `workQueue` 将无限制地增加，这可能会导致程序内存溢出**。

一种更加稳妥的方式是使用有界队列，例如 `ArrayBlockingQueue`、有界的 `LinkedBlockingQueue`、`PriorityBlockingQueue`。使用有界队列有助于避免资源耗尽的情况发生，在有界队列填满之后，线程池中的饱和策略 `RejectedExecutionHandler` 将会开始发挥作用。使用有界队列时，队列的大小与线程池的大小必须一起调节，如果线程池较小，队列较大，那么将有助于降低 CPU 的使用率，同时还可以减少上下文切换，但是可能会限制程序的吞吐量。

对于非常大或者无界的线程池，可以使用 `SynchronousQueue` 来避免任务排队，以及直接将任务从生产者移交给工作者线程。

当使用像 `LinkedBlockingQueue` 或者 `ArrayBlockingQueue` 这样的 FIFO 队列时，任务的执行顺序与它们的到达顺序相同。如果想进一步控制任务的执行顺序，还可以使用 `PriorityBlockingQueue`。

### 饱和策略

当有界队列被填满后，饱和策略开始发挥作用。JDK 提供了几种不同的 `RejectedExecutionHandler` 实现，每种实现都包含不同的饱和策略：

1. `CallerRunsPolicy` 将当前任务回退给调用线程，并且会在调用线程执中行任务；
2. `AbortPolicy`（默认） 抛出 `RejectedExecutionException` 异常；
3. `DiscardPolicy` 抛弃当前任务
4. `DiscardOldestPolicy` 抛弃 `workQueue` 队首的任务，然后重新提交当前任务。

### 线程工厂

当线程池需要创建一个线程时，都是通过线程工厂 `ThreadFactory` 来完成的。默认的线程工厂方法将创建一个新的、非守护的线程，并且不包含特殊的配置。

```java
private static class DefaultThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    DefaultThreadFactory() {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                              Thread.currentThread().getThreadGroup();
        namePrefix = "pool-" +
                      poolNumber.getAndIncrement() +
                     "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                              namePrefix + threadNumber.getAndIncrement(),
                              0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

在许多情况下，都需要使用定制的线程工厂，例如希望为线程池中的线程指定一个 `UncaughtExceptionHandler`，或者实例化一个定制的 `Thread` 类用于执行调试信息的记录，或者只是希望给线程取一个更有意义的名称，用来解析线程的转储信息和错误信息。

### 一图概览

```plain text
         +------------------------------------------------------------------------+
runnbale |                                              +-----------------------+ |
-------->|                                 True         | +------+ +------+     | |
         |--> workerCount < corePoolSize? ------------->| |Worker| |Worker| ... | |
runnbale |                 |               new Worker() | +------+ +------+     | |
-------->|           False |                            +-----------------------+ |
         |                 |                               |   |   |         ^    |
runnbale |                 |                            workQueue.take       |    |
-------->|                 |                               |   |   |         |    |
         |                 |                               v   v   v         |    |
         |                 |                 +---------------------------+   |    |
         |                 v         True    | +--------+ +--------+     |   |    |
         |         workQueue.offer? -------->| |Runnable| |Runnable| ... |   |    |
         |                 |                 | +--------+ +--------+     |   |    |
         |           False |                 +---------------------------+   |    |
         |                 |                      BlockingQueue<Runnable>    |    |
         |                 |                                                 |    |
         |                 V                    True                         |    |
         |      workerCount < maximumPoolSize? ------------------------------+    |
         |                 |                    new Worker()                      |
         |           False |                                                      |
         |                 |                                                      |
         |                 v                                                      |
         |        +-------------+------------+--------------+                     |
         |        |             |            |              |                     |
         |        v             v            v              v                     |
         |  CallerRunsPolicy  AbortPolicy  DiscardPolicy  DiscardOldestPolicy     |
         +------------------------------------------------------------------------+
```
