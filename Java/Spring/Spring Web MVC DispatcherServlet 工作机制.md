# Spring Web MVC DispatcherServlet 工作机制

Spring MVC 是围绕着一个名为 DispatcherServlet 的核心 Servlet 来设计的，DispatcherServlet 提供了一个用于处理请求的公共算法，并且会把这些实际工作委托给 Spring 容器当中的 Bean，同时这些 Bean 都是可配置化。Spring MVC 的这种工作模式非常灵活，因此它支持多种不同的工作流程。

## 特殊 Bean

DispatcherServlet 会感知到 Spring 容器中以下类型的特殊 Bean：

| Bean 类型                             | 说明                                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| HandlerMapping                        | 将 request 映射到 handler，同时会对关联一系列的用于 request 前置和后置处理的 interceptor。HandlerMapping 中的映射是基于某些条件的，不同的 HandlerMapping 实现类的实现细节是不一样的。<br>两个主要 HandlerMapping 实现类是 `RequestMappingHandlerMapping` 和 `SimpleUrlHandlerMapping`。前者支持将 request 映射到使用 `@RequestMapping` 注解的方法，后者支持显式注册 URI 路径，并会维护它与 handler 的映射关系。 |
| HandlerAdapter                        | 帮助 Dispatcher 调用 request 所映射的 handler，会屏蔽 handler 具体的调用方式。例如调用 `@RequestMapping` 注解的方法时需要解析注解，HandlerAdapter 的作用就是使 Dispatcher 对这些细节无感知。                                                                                                                                                                                                                    |
| HandlerExceptionResolver              | 解析 exception 的策略，可能会将 exception 解析到 `@ExceptionHandler` 注解的方法，或者是 HTML 错误页面。                                                                                                                                                                                                                                                                                                         |
| ViewResolver                          | 将从 handler 返回的 String 类型的逻辑视图解析为真实的视图，解析后的视图会被作为 response。                                                                                                                                                                                                                                                                                                                      |
| LocaleResolver、LocaleContextResolver | 解析客户端所使用的 `Locale` 和时区，为了能够提供一个国际化的视图。                                                                                                                                                                                                                                                                                                                                              |
| ThemeResolver                         | 解析 web 应用可以使用的主题。                                                                                                                                                                                                                                                                                                                                                                                   |
| MultipartResolver                     | 解析 multi-par request，例如在浏览器中上传文件。                                                                                                                                                                                                                                                                                                                                                                |
| FlashMapManager                       | 存储和检索「输入」与「输出」的 FlashMap，该 FlashMap 可用于将一个 request 中的属性传递到另一个 request 中。                                                                                                                                                                                                                                                                                                     |

## 工作流程

```plain text
                                                           +---------------------+
                                                           | HandlerMapping      |
                                        request            | +------------------+|
                                   +<--------------------->| |HandlerInterceptor||
                                   | handler, interceptors | +------------------+|
  request  +-------------------+   |                       +---------------------+
<--------->| DispatcherServlet |---+
 response  +-------------------+   |   request, handler    +----------------+
                                   +<--------------------->| HandlerAdapter |
                                   |                 view  +----------------+
                                   |
                                   |   view                +----------------+
                                   +<--------------------->| ViewResolver   |
                                                 response  +----------------+
                                                                |      ^
                                                                v      |
                                                           +----------------+
                                                           | LocaleResolver |
                                                           +----------------+
                                                                |      ^
                                                                v      |
                                                           +----------------+
                                                           | ThemeResolver  |
                                                           +----------------+
```

## 参考资料

- [Spring Web MVC - DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)
