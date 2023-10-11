# Go vs C vsJava

## 指针

Go 有指针，但不支持多级指针。

C 有指针，并且支持多级指针。

Java 没有指针。

## 垃圾回收

Go 和 Java 带有垃圾回收，C 是没有的。

## switch case 语句

Go 的 switch-case 语句默认自带 break，C 和 Java 是没有的。

## if 语句

Go 的 if 语句和 for 语句一样，可以声明一个只在 if / for 语句作用域之内的变量，C 和 Java 不支持。

## 多值返回

Go 支持函数返回多个参数，C 和 Java 不支持。

## 变量声明

Go 的变量声明语法是：名称 + 类型，C 和 Java 是：类型 + 名称。

## 包管理

Go 使用官方的 Go Modules 包管理工具，除了一些库依赖 GitHub，其它都挺好用的。

C/C++ 使用 Make、CMake ...... 包管理工具，由于历史包袱太重，C 的各种库使用各种不同的包管理工具，相比于 Go 和 Java 来说比较难用。

Java 使用 Maven 和 Gradle 包管理工具，还有个中央 Maven 仓库，使用体验很好。

## 实现接口

Go 中采用隐式实现接口的方式，一个类型只要实现了某个接口的所有方法，则就是实现了该接口，不需要使用也没有 `implements` 关键字。

C 中没有接口的概念。

Java 中需要显示声明实现接口，需要使用 `implements` 关键字。

## 数据结构

| 语言 \ 数据结构 | array | slice / list | set | map |
| --------------- | ----- | ------------ | --- | --- |
| Go              | ✅    | ✅           | ❌  | ✅  |
| C               | ✅    | ❌           | ❌  | ❌  |
| Java            | ✅    | ✅           | ✅  | ✅  |
