# ThreadLocal 工作机制

使用线程封闭是实现线程安全最简单的方式之一，维持线程封闭性的一种常规方法是使用 `ThreadLocal`，它能使线程中的某个值与保存值的对象关联起来。`ThreadLocal` 提供了 get 与 set 等方法，这些方法为每个使用该变量的线程都有一份独立的副本，因此 get 总是返回由当前执行线程在调用 set 时设置的最新值。

## 实现原理

`Thread` 在内部定义了一个 `ThreadLocal.ThreadLocalMap` 类型的 threadLocals 变量，用于在 `ThreadLocal` 中获取当前线程关联的 `ThreadLocal.ThreadLocalMap` 实例。

```java
public class Thread implements Runnable {
	// ......

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    // ......
}
```

`ThreadLocal.ThreadLocalMap` 在内部定义了一个 `ThreadLocal.ThreadLocalMap.Entry` 数组 table 变量。`ThreadLocal.ThreadLocalMap.Entry` 继承了 `WeakReference<ThreadLocal<?>>`，它使用 `ThreadLocal` 实例的引用作为 Map 的 key，使用 `ThreadLocal` set 方法设置的数据作为 Map 的 value。

```java
public class ThreadLocal<T> {
    // ......

	static class ThreadLocalMap {

		static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }

        private Entry[] table;

        private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            // ......
            tab[i] = new Entry(key, value);
            // ......
        }

        // ......
    }

    // ......
}
```

`ThreadLocal` 在对 `ThreadLocal.ThreadLocalMap` 进行封装之后，对外提供了 get 和 set 方法。

```java
public class ThreadLocal<T> {
    // ......

	public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

	public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    // ......
}
```

## 一图概览

```plain text
+------------+
|Thread      |
|------------|   +--------------------------+
|threadLocals|-->|ThreadLocal.ThreadLocalMap|
|------------|   |--------------------------|   +--------------------------------+
| ......     |   |table[]                   |-->|ThreadLocal.ThreadLocalMap.Entry|
+------------+   |--------------------------|   |-----------+-----------+--------|
                 | ......                   |   |ThreadLocal|ThreadLocal|        |
                 +--------------------------+   |  WeakRef  |  WeakRef  | ...... |
                                                |-----------+-----------+--------|
                                                |   value   |   value   | ...... |
                                                +-----------+-----------+--------+
```

## 内存泄漏

`Thread` 和 `ThreadLocal` 中都会持有 `ThreadLocal.ThreadLocalMap` 的引用，并且 `ThreadLocal` 内部与 `ThreadLocal.ThreadLocalMap` 的引用关系是 `WeakReference` 弱引用。所以在 `ThreadLocal` 已经被 GC 回收，并且当 `Thread` 还存活时，`ThreadLocal.ThreadLocalMap` 就是一个只会占用内存但不会被实际使用的内存垃圾。不过当 `Thread` 运行结束时，`ThreadLocal.ThreadLocalMap` 也会被回收，此时也就不存在内存泄漏的问题了。

## 在父子线程之间传递数据

`InheritableThreadLocal` 扩展了 `ThreadLocal`，支持在创建子线程时继承父线程中保存的 `InheritableThreadLocal` 数据。

## 向可复用的线程传递数据

`ThreadLocal` 中的数据与线程的生命周期绑定在一起，因此不适用于使用线程池等会池化复用线程的场景。但是如果应用确实使用了线程池 `ThreadPoolExecutor`，并且存在将 **任务提交时** 的数据传递给 **任务执行时** 的需求时候，可以考虑使用阿里巴巴的开源组件：[TransmittableThreadLocal](https://github.com/alibaba/transmittable-thread-local)。
