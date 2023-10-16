# Java Agent

`java.lang.instrument` 包提供了这样的一种能力：允许开发者使用 Java Agent 来探测 JVM 中正在运行的应用程序。其中，探测或修改应用程序需要通过修改程序的字节码来实现。

一个 Java Agent 程序是以 Jar 包的形式部署，Java Agent 程序打包生成的 Jar 包中的 `META-INF/MANIFEST.MF` 文件中的属性（`Premain-Class` 或 `Agent-Class`）指定了引导这个 Java Agent 程序的主类。Java Agent 程序可以通过添加新的 JVM 启动参数选项，来以 Command-Line 的方式加载和使用。除此之外，Java Agent 程序也支持在 JVM 启动之后再加载的方式，不过这种使用方式的具体实现是与 JVM 平台独立无关的。

## 加载 Java Agent

### 通过 Command-Line

在 Command-Line 使用方式中，一个 Java Agent 程序是通过在 JVM 的启动参数中添加 `-javaagent:jarpath[=options]` 选项来加载的。其中，`jarpath` 是 Java Agent 对应 Jar 包的路径，`options` 是启动 Java Agent 所需的参数。

在这种使用方式中，Java Agent 对应 Jar 包中的 `META-INF/MANIFEST.MF` 文件需要含有 `Premain-Class` 属性，`Premain-Class` 属性值是引导这个 Java Agent 程序启动的主类，并且这个类需要含有 `public static void premain(String agentArgs, Instrumentation inst)` 或者 `public static void premain(String agentArgs)` 签名的方法（与一般 Java 应用程序的主类需要含有 `public static void main(String[] args)` 签名的方法类似）。

Java Agent 程序中的主类会被应用程序的 `ClassLoader#getSystemClassLoader()` 类加载器所加载，这个类加载器通常也会加载应用程序的主类。如果 Java Agent 程序无法被 JVM 加载或者加载时抛出异常，那么 JVM 的启动流程将会中止。

### 在 JVM 启动之后加载

在 JVM 启动之后再加载 Java Agent 使用方式中，Java Agent 的初始化细节是基于应用特定实现的，但通常来说，这是发生在应用已经启动和加载主类之后的事情。

在这种使用方式中，Java Agent 对应 Jar 包中的 `META-INF/MANIFEST.MF` 文件需要含有 `Agent-Class` 属性，`Agent-Class` 属性值是引导这个 Java Agent 程序启动的主类，并且这个类需要含有 `public static void agentmain(String agentArgs, Instrumentation inst)` 或者 `public static void agentmain(String agentArgs)` 签名的方法。

Java Agent 程序中的主类会被应用程序的 `ClassLoader#getSystemClassLoader()` 类加载器所加载，并且要求这个类加载器支持将 Java Agent 对应 Jar 包添加到系统类路径的机制。如果 Java Agent 程序无法被 JVM 加载或者加载时抛出异常，JVM 则会忽略异常，并且不会中止退出。

原文链接：[https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html](https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html)。

## 使用案例

Java Agent 可以被应用于很多场景，例如；

- [arthas](https://github.com/alibaba/arthas/) 使用 Java Agent  作为探测应用状态程序的启动类；
- [transmittable-thread-local](https://github.com/alibaba/transmittable-thread-local) 支持使用 Java Agent 无侵入地方式装饰 JDK 线程池实现类；

## 代码示例

Java Agent 的主类：

```java
package com.demo;
import java.lang.instrument.Instrumentation;
public class MyAgent {
    public static void premain(String agentArgs, Instrumentation inst) {
        // 此处通常会操作字节码，修改应用程序逻辑，实现监控、热部署、AOP 等功能
        System.out.println("Hello My Java Agent");
    }
}
```

Java Agent 的 Maven 打包方式：

```xml
<groupId>com.demo</groupId>
<artifactId>MyAgent</artifactId>
<version>1.0</version>
......
<build>
    <plugins>
        <plugin>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.1.2</version>
            <configuration>
                <archive>
                    <!-- see https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html -->
                    <manifestEntries>
                        <!-- The manifest of the agent JAR file must contain the attribute Premain-Class. -->
                        <Premain-Class>com.demo.MyAgent</Premain-Class>
                        <!-- Is the ability to redefine classes needed by this agent. -->
                        <Can-Redefine-Classes>false</Can-Redefine-Classes>
                        <!-- Is the ability to retransform classes needed by this agent. -->
                        <Can-Retransform-Classes>true</Can-Retransform-Classes>
                        <!-- Is the ability to set native method prefix needed by this agent. -->
                        <Can-Set-Native-Method-Prefix>false</Can-Set-Native-Method-Prefix>
                    </manifestEntries>
                </archive>
            </configuration>
        </plugin>
    </plugins>
</build>
```

Java Agent 的 JVM 启动参数：

```bash
java -javaagent:.../MyAgent-1.0.jar -jar application.jar
```
