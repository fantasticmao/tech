# Spring IoC 容器扩展点

在 Spring 中，可以通过实现特定的 IoC 容器扩展接口来实现更高级的功能。

## BeanPostProcessor

使用 `BeanPostProcessor` 接口可以在 IoC 容器中自定义 Bean 实例。`BeanPostProcessor` 接口定义了创建 Bean 实例 **之前** 和 **之后** 的钩子方法，开发者可以通过这两个钩子方法来实现自定义 Bean 的实例化逻辑（甚至可以覆盖 IoC 容器中创建的实例）、依赖解析逻辑等等。

### 案例一

Spring 内置提供的 `EnvironmentAware`、`ResourceLoaderAware` 和 `ApplicationContextAware` 等接口可以为 Bean 提供感知 IoC 容器的能力，这些接口都是通过 `ApplicationContextAwareProcessor` 处理器来实现的。

### 案例二

Spring 对 JSR-250 中定义的 `@Resource`、`@PostConstruct` 和 `@PreDestroy` 等注解的支持是通过 `CommonAnnotationBeanPostProcessor` 处理器来实现的。

## BeanFactoryPostProcessor

使用 `BeanFactoryPostProcessor` 接口可以在 IoC 容器中自定义 Bean 的配置元数据。在 IoC 容器实例化任何 Bean（除了 `BeanFactoryPostProcessor` 本身）之前，`BeanFactoryPostProcessor` 接口可以读取 Bean 的配置元数据，并且可以修改它。

### 案例一

Spring 内置提供的 `PropertySourcesPlaceholderConfigurer` 支持从外部的 .properties 文件中读取 Bean 的配置元数据的属性值。

### 案例二

在基于 Annotation 的配置模式下，Spring 会通过 `ConfigurationClassPostProcessor` 处理器来解析 `@Configuration` 注解标记的配置类。

## FactoryBean

使用 `FactoryBean` 接口可以在 IoC 容器中直接创建 Bean 实例。Spring 支持使用 `FactoryBean#getObject()` 来直接创建对象，在 IoC 容器内部会对 `FactoryBean` 创建的实例名称前面加上 `&` 前缀。

### 案例一

Spring 内置提供的 `ThreadPoolExecutorFactoryBean` 支持创建一个线程池 Bean 实例。

## 参考资料

- [The IoC Container - Container Extension Points](https://docs.spring.io/spring-framework/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-extension)
