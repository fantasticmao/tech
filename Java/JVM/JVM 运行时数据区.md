# JVM 运行时数据区

## 程序计数器（The pc Register）

程序计数器是一块较小的内存空间，它可以看作是当前线程所执行的字节码的行号指示器。在 JVM 的概念模型中，字节码解释器工作时就是通过这个计数器值来选取下一条需要执行的字节码指令，它是程序控制流程的执行器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

## Java 虚拟机栈（JVM Stacks）

Java 虚拟机栈是线程私有的，它的生命周期与线程相同。Java 虚拟机栈所描述的是 Java 方法执行时的线程内存模型：每个方法被执行的时候，JVM 都会同步创建一个栈帧，用于存储局部变量表、操作数栈、动态连接、方法出口等信息。每一个方法被调用直至执行完毕，就对应着一个栈帧在虚拟机中从入栈到出栈的过程。

## 堆（Heap）

Java 堆是被所有线程共享的一块内存区域，此内存区域的唯一目的是存放对象实例，JVM 中几乎所有的对象实例都是在这里分配内存的。

## 方法区（Method Area）

方法区和 Java 堆一样，是各个线程共享的内存区域，它用于存储已经被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。

## 运行时常量池（Run-Time Constant Pool）

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表（Constant Pool Table），用于存放编译期生成的各种字面量与符号引用。这部分内容将在类加载后被存放到方法区的运行时常量池中。

## 本地方法栈（Native Method Stacks）

本地方法栈与 JVM 栈的作用相同，不过是被用于 native 方法的。

## 参考资料

- [Java Virtual Machine Specification - The Structure of the Java Virtual Machine - Run-Time Data Areas](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5)
