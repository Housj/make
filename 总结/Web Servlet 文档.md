# Web on Servlet堆栈

版本5.2.6.RELEASE

[返回索引](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/index.html)

文档的此部分涵盖对基于Servlet API构建并部署到Servlet容器的Servlet堆栈Web应用程序的支持。各个章节包括[Spring MVC](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc)， [View Technologies](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-view)，[CORS支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-cors)和[WebSocket支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket)。对于无功栈的Web应用程序，请参见[网站上无堆栈](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#spring-web-reactive)。

## 1. Spring Web MVC

Spring Web MVC是基于Servlet API构建的原始Web框架，并且从一开始就已包含在Spring框架中。正式名称“ Spring Web MVC”来自其源模块（[`spring-webmvc`](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc)）的名称，但它通常被称为“ Spring MVC”。

与Spring Web MVC并行，Spring Framework 5.0引入了一个反应式堆栈Web框架，其名称“ Spring WebFlux”也基于其源模块（[`spring-webflux`](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux)）。本节介绍Spring Web MVC。在[下一节](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#spring-web-reactive) 介绍春季WebFlux。

有关基线信息以及与Servlet容器和Java EE版本范围的兼容性，请参见Spring Framework [Wiki](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Versions)。

### 1.1。分派器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-dispatcher-handler)

与其他许多Web框架一样，Spring MVC围绕前端控制器模式进行设计`Servlet`，其中`DispatcherServlet`，Central提供了用于请求处理的共享算法，而实际工作是由可配置的委托组件执行的。该模型非常灵活，并支持多种工作流程。

的`DispatcherServlet`，因为任何`Servlet`，需要根据通过使用Java配置或在Servlet说明书中声明和映射`web.xml`。反过来，`DispatcherServlet`使用Spring配置文件来发现它需要请求映射，视图解析，异常处理，委托组件[和更多](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet-special-bean-types)。

以下Java配置示例注册并初始化`DispatcherServlet`，该Servlet容器自动检测到（请参阅[Servlet Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-container-config)）：



 

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletCxt) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

|      | 除了直接使用ServletContext API之外，您还可以扩展 `AbstractAnnotationConfigDispatcherServletInitializer`和覆盖特定的方法（请参见[Context Hierarchy](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet-context-hierarchy)下的示例）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下`web.xml`配置示例注册并初始化`DispatcherServlet`：

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```

|      | Spring Boot遵循不同的初始化顺序。Spring Boot并没有陷入Servlet容器的生命周期，而是使用Spring配置来引导自身和嵌入式Servlet容器。`Filter`和`Servlet`声明在Spring配置中检测到并注册到Servlet容器中。有关更多详细信息，请参见 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-embedded-container)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.1.1。上下文层次

`DispatcherServlet`期望对它自己的配置有一个`WebApplicationContext`（plain的扩展 `ApplicationContext`）。`WebApplicationContext`有一个链接 `ServletContext`和`Servlet`与它相关联。它也与`ServletContext` 应用程序绑定在一起，以便应用程序可以使用静态方法`RequestContextUtils`查找 `WebApplicationContext`是否需要访问它。

对于许多应用程序来说，拥有一个`WebApplicationContext`简单且足够。也可能有一个上下文层次结构，其中一个根`WebApplicationContext` 在多个`DispatcherServlet`（或其他`Servlet`）实例之间共享，每个实例都有其自己的子`WebApplicationContext`配置。有关 上下文层次结构功能的更多信息，请参见的[其他`ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#context-introduction)功能。

根`WebApplicationContext`通常包含基础结构Bean，例如需要在多个`Servlet`实例之间共享的数据存储库和业务服务。这些Bean被有效地继承，并且可以在Servlet特定的子级中重写（即重新声明），该子级`WebApplicationContext`通常包含给定本地的Bean `Servlet`。下图显示了这种关系：

![mvc上下文层次结构](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/mvc-context-hierarchy.png)

以下示例配置`WebApplicationContext`层次结构：



 

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```

|      | 如果不需要应用程序上下文层次结构，则应用程序可以通过`getRootConfigClasses()`和`null`从返回所有配置`getServletConfigClasses()`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了`web.xml`等效项：

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```

|      | 如果不需要应用程序上下文层次结构，则应用程序可以仅配置“根”上下文，并将`contextConfigLocation`Servlet参数保留为空。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.1.2。特殊豆类

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-special-bean-types)

`DispatcherServlet`特殊bean 的委托处理请求并提供适当的响应。所谓“特殊bean”，是`Object`指实现框架合同的Spring托管实例。这些通常带有内置合同，但是您可以自定义其属性并扩展或替换它们。

下表列出了通过检测到的特殊bean `DispatcherServlet`：

| 豆类                                                         | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `HandlerMapping`                                             | 将请求与[拦截器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-handlermapping-interceptor)列表一起映射到处理程序，以 进行预处理和后期处理。映射基于某些标准，具体标准因`HandlerMapping` 实现而异。两个主要`HandlerMapping`实现是`RequestMappingHandlerMapping` （支持带`@RequestMapping`注释的方法）和`SimpleUrlHandlerMapping` （将URI路径模式显式注册到处理程序）。 |
| `HandlerAdapter`                                             | 帮助`DispatcherServlet`调用映射到请求的处理程序，而不管实际如何调用该处理程序。例如，调用带注释的控制器需要解析注释。a的主要目的`HandlerAdapter`是保护`DispatcherServlet`这些细节。 |
| [`HandlerExceptionResolver`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-exceptionhandlers) | 解决异常的策略，可能将它们映射到处理程序，HTML错误视图或其他目标。请参阅[例外](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-exceptionhandlers)。 |
| [`ViewResolver`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-viewresolver) | `String`将从处理程序返回的基于逻辑的视图名称解析为实际的名称`View` ，以将其呈现给响应。请参阅[查看分辨率](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-viewresolver)和[查看技术](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-view)。 |
| [`LocaleResolver`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-localeresolver)，[LocaleContextResolver](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-timezone) | 解决`Locale`一个客户正在使用的并且可能是其时区的问题，以便能够提供国际化的视图。请参阅[语言环境](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-localeresolver)。 |
| [`ThemeResolver`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-themeresolver) | 解决Web应用程序可以使用的主题，例如，提供个性化的布局。请参阅[主题](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-themeresolver)。 |
| [`MultipartResolver`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multipart) | 借助一些多部分解析库来解析多部分请求的抽象（例如，浏览器表单文件上传）。请参见[Multipart Resolver](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multipart)。 |
| [`FlashMapManager`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-flash-attributes) | 存储和检索“输入”和“输出” `FlashMap`，它们通常用于通过重定向将属性从一个请求传递到另一个请求。请参见[Flash属性](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-flash-attributes)。 |

#### 1.1.3。Web MVC配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-framework-config)

应用程序可以声明 处理请求所需的[特殊Bean类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet-special-bean-types)中列出的基础结构[Bean](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet-special-bean-types)。该`DispatcherServlet`检查 `WebApplicationContext`对每个特殊的豆。如果没有匹配的bean类型，它将使用中列出的默认类型 [`DispatcherServlet.properties`](https://github.com/spring-projects/spring-framework/blob/master/spring-webmvc/src/main/resources/org/springframework/web/servlet/DispatcherServlet.properties)。

在大多数情况下，[MVC Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)是最佳起点。它使用Java或XML声明所需的bean，并提供更高级别的配置回调API对其进行自定义。

|      | Spring Boot依靠MVC Java配置来配置Spring MVC，并提供许多额外的方便选项。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.1.4。Servlet配置

在Servlet 3.0+环境中，您可以选择以编程方式配置Servlet容器，以替代方式或与`web.xml`文件结合使用。以下示例注册一个`DispatcherServlet`：



 

```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```

`WebApplicationInitializer`是Spring MVC提供的接口，可确保检测到您的实现并将其自动用于初始化任何Servlet 3容器。通过`WebApplicationInitializer`命名方法 的抽象基类实现，`AbstractDispatcherServletInitializer`可以`DispatcherServlet`通过重写方法来指定servlet映射和`DispatcherServlet`配置位置，从而更加轻松地进行注册 。

对于使用基于Java的Spring配置的应用程序，建议这样做，如以下示例所示：



 

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

如果使用基于XML的Spring配置，则应直接从扩展 `AbstractDispatcherServletInitializer`，如以下示例所示：



 

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

`AbstractDispatcherServletInitializer`还提供了一种添加`Filter` 实例并将其自动映射到的便捷方法`DispatcherServlet`，如以下示例所示：



 

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```

每个过滤器都会根据其具体类型添加默认名称，并自动映射到`DispatcherServlet`。

的`isAsyncSupported`受保护方法`AbstractDispatcherServletInitializer` 提供了一个位置，以在`DispatcherServlet`和映射到它的所有过滤器上启用异步支持。默认情况下，此标志设置为`true`。

最后，如果您需要进一步自定义`DispatcherServlet`自身，则可以覆盖该`createDispatcherServlet`方法。

#### 1.1.5。处理中

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-dispatcher-handler-sequence)

该`DispatcherServlet`过程要求如下：

- 在`WebApplicationContext`被搜索并在请求的一个属性，在过程控制器和其它元件可以使用的约束。默认情况下，它是在`DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE`键下绑定的。
- 语言环境解析器绑定到请求，以使流程中的元素解析在处理请求（呈现视图，准备数据等）时要使用的语言环境。如果不需要语言环境解析，则不需要语言环境解析器。
- 主题解析器绑定到请求，以使诸如视图之类的元素确定要使用的主题。如果不使用主题，则可以将其忽略。
- 如果指定多部分文件解析器，则将检查请求中是否有多部分。如果找到多部分，则将请求包装在中，以`MultipartHttpServletRequest`供流程中的其他元素进一步处理。有关[多](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multipart)部分处理的更多信息，请参见[Multipart Resolver](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multipart)。
- 搜索适当的处理程序。如果找到处理程序，则执行与处理程序（预处理器，后处理器和控制器）关联的执行链，以准备模型或渲染。或者，对于带注释的控制器，可以呈现响应（内`HandlerAdapter`），而不返回视图。
- 如果返回模型，则呈现视图。如果没有返回任何模型（可能是由于预处理器或后处理器拦截了该请求，可能出于安全原因），则不会呈现任何视图，因为该请求可能已经被满足。

中`HandlerExceptionResolver`声明的Bean `WebApplicationContext`用于解决在请求处理期间引发的异常。这些异常解析器允许定制逻辑以解决异常。有关更多详细信息，请参见[异常](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-exceptionhandlers)。

Spring `DispatcherServlet`还支持`last-modification-date`Servlet API指定的的返回 。确定特定请求的最后修改日期的过程很简单： `DispatcherServlet`查找适当的处理程序映射并测试找到的处理程序是否实现了`LastModified`接口。如果是这样，则将接口`long getLastModified(request)`方法的值 `LastModified`返回给客户端。

您可以`DispatcherServlet`通过将Servlet初始化参数（`init-param`元素）添加到`web.xml`文件中的Servlet声明中 来定制各个实例。下表列出了受支持的参数：

| 参数                             | 说明                                                         |
| :------------------------------- | :----------------------------------------------------------- |
| `contextClass`                   | 实现的类，`ConfigurableWebApplicationContext`由该Servlet实例化并在本地配置。默认情况下，`XmlWebApplicationContext`使用。 |
| `contextConfigLocation`          | 传递给上下文实例的字符串（由指定`contextClass`），以指示可以在哪里找到上下文。该字符串可能包含多个字符串（使用逗号作为分隔符）以支持多个上下文。对于具有两次定义的bean的多个上下文位置，以最新位置为准。 |
| `namespace`                      | 的命名空间`WebApplicationContext`。默认为`[servlet-name]-servlet`。 |
| `throwExceptionIfNoHandlerFound` | `NoHandlerFoundException`在找不到请求的处理程序时是否抛出。然后可以使用`HandlerExceptionResolver`（例如，使用 `@ExceptionHandler`控制器方法）捕获该异常并将其作为其他任何异常进行处理。默认情况下，它设置为`false`，在这种情况下，`DispatcherServlet`将响应状态设置为404（NOT_FOUND），而不会引发异常。请注意，如果还配置了[默认servlet处理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-default-servlet-handler)，则始终将未解决的请求转发到默认servlet，并且永远不会引发404。 |

#### 1.1.6。拦截

所有`HandlerMapping`实现都支持处理程序拦截器，当您要将特定功能应用于某些请求（例如，检查主体）时，这些拦截器将非常有用。拦截器必须使用三种方法`HandlerInterceptor`从`org.springframework.web.servlet`程序包中实现，这 三种方法应提供足够的灵活性以执行各种预处理和后处理：

- `preHandle(..)`：在执行实际的处理程序之前
- `postHandle(..)`：执行处理程序后
- `afterCompletion(..)`：完成完整的请求后

该`preHandle(..)`方法返回一个布尔值。您可以使用此方法来中断或继续执行链的处理。当此方法返回时`true`，处理程序执行链继续。当它返回false时，`DispatcherServlet` 假定拦截器本身已经处理了请求（例如，渲染了适当的视图），并且不会继续执行执行链中的其他拦截器和实际处理程序。

见[拦截](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-interceptors)在MVC配置一节来配置拦截器如何例子。您还可以通过使用各个`HandlerMapping`实现上的设置器直接注册它们 。

请注意，`postHandle`使用`@ResponseBody`和`ResponseEntity`方法在`HandlerAdapter`和之前 编写和提交响应的方法用处不大`postHandle`。这意味着对响应进行任何更改为时已晚，例如添加额外的标头。对于此类情况，您可以实现`ResponseBodyAdvice`并将其声明为[Controller Advice](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-controller-advice) Bean，或直接在上进行配置 `RequestMappingHandlerAdapter`。

#### 1.1.7。例外情况

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-dispatcher-exceptions)

如果在请求映射期间发生异常或从请求处理程序（例如`@Controller`）抛出异常，则将`DispatcherServlet`委托委托给`HandlerExceptionResolver` Bean 链以解决异常并提供替代处理，通常是错误响应。

下表列出了可用的`HandlerExceptionResolver`实现：

| `HandlerExceptionResolver`                                   | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `SimpleMappingExceptionResolver`                             | 异常类名称和错误视图名称之间的映射。对于在浏览器应用程序中呈现错误页面很有用。 |
| [`DefaultHandlerExceptionResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/mvc/support/DefaultHandlerExceptionResolver.html) | 解决Spring MVC引发的异常，并将其映射到HTTP状态代码。另请参见替代`ResponseEntityExceptionHandler`和[REST API异常](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-rest-exceptions)。 |
| `ResponseStatusExceptionResolver`                            | 使用`@ResponseStatus`注释解决异常，并根据注释中的值将其映射到HTTP状态代码。 |
| `ExceptionHandlerExceptionResolver`                          | 通过调用`@ExceptionHandler`a `@Controller`或 `@ControllerAdvice`class中的方法来解决异常。请参见[@ExceptionHandler方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-exceptionhandler)。 |

##### 解析器链

您可以通过`HandlerExceptionResolver` 在Spring配置中声明多个bean并`order`根据需要设置它们的属性来形成异常解析器链。order属性越高，异常解析器的定位就越晚。

的合同`HandlerExceptionResolver`指定可以返回：

- 一个`ModelAndView`它指向一个误差图。
- `ModelAndView`如果在解析程序中处理了异常，则为空。
- `null` 如果异常未解决，则供以后的解析器尝试；如果异常仍在末尾，则允许其冒泡到Servlet容器。

在[MVC配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)自动声明内置解析器默认Spring MVC的例外，`@ResponseStatus`注释例外，并支持的 `@ExceptionHandler`方法。您可以自定义该列表或替换它。

##### 容器错误页面

如果任何人仍然无法解决异常`HandlerExceptionResolver`，因此该异常可以传播，或者如果响应状态设置为错误状态（即4xx，5xx），则Servlet容器可以在HTML中呈现默认错误页面。要自定义容器的默认错误页面，可以在中声明错误页面映射`web.xml`。以下示例显示了如何执行此操作：

```xml
<error-page>
    <location>/error</location>
</error-page>
```

给定前面的示例，当异常冒出气泡或响应具有错误状态时，Servlet容器在容器内向配置的URL（例如`/error`）进行ERROR调度。然后由进行处理`DispatcherServlet`，可能将其映射到`@Controller`，可以实现该模型以使用模型返回错误视图名称或呈现JSON响应，如以下示例所示：



 

```java
@RestController
public class ErrorController {

    @RequestMapping(path = "/error")
    public Map<String, Object> handle(HttpServletRequest request) {
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("status", request.getAttribute("javax.servlet.error.status_code"));
        map.put("reason", request.getAttribute("javax.servlet.error.message"));
        return map;
    }
}
```

|      | Servlet API没有提供在Java中创建错误页面映射的方法。但是，您可以同时使用`WebApplicationInitializer`和`web.xml`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.1.8。查看分辨率

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-viewresolution)

Spring MVC定义了`ViewResolver`和`View`接口，可让您在浏览器中呈现模型，而无需将您与特定的视图技术联系在一起。`ViewResolver` 提供视图名称和实际视图之间的映射。`View`在移交给特定的视图技术之前，先解决数据准备问题。

下表提供了有关`ViewResolver`层次结构的更多详细信息：

| ViewResolver                     | 描述                                                         |
| :------------------------------- | :----------------------------------------------------------- |
| `AbstractCachingViewResolver`    | `AbstractCachingViewResolver`它们解析的缓存视图实例的子类。缓存可以提高某些视图技术的性能。您可以通过将`cache`属性设置为来关闭缓存`false`。此外，如果必须在运行时刷新某个视图（例如，当修改FreeMarker模板时），则可以使用该`removeFromCache(String viewName, Locale loc)`方法。 |
| `XmlViewResolver`                | 的实现`ViewResolver`接受一个用XML编写的配置文件，该配置文件的DTD与Spring的XML bean工厂相同。默认配置文件是 `/WEB-INF/views.xml`。 |
| `ResourceBundleViewResolver`     | 该实现`ViewResolver`使用`ResourceBundle`由包基本名称指定的中的bean定义。对于应该解析的每个视图，它将属性的值`[viewname].(class)`用作视图类，并将属性的值`[viewname].url`用作视图URL。您可以在[View Technologies](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-view)一章中找到示例 。 |
| `UrlBasedViewResolver`           | `ViewResolver`接口的简单实现会影响将逻辑视图名称直接解析为URL而没有显式映射定义。如果您的逻辑名称以直接的方式与视图资源的名称匹配，而无需任意映射，则这是适当的。 |
| `InternalResourceViewResolver`   | 的方便子类`UrlBasedViewResolver`支持`InternalResourceView`（实际上，Servlet和JSP）和子类，如`JstlView`和`TilesView`。您可以使用来为此解析器生成的所有视图指定视图类`setViewClass(..)`。有关[`UrlBasedViewResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/reactive/result/view/UrlBasedViewResolver.html) 详细信息，请参见javadoc。 |
| `FreeMarkerViewResolver`         | 方便的子类的`UrlBasedViewResolver`支持`FreeMarkerView`和他们的自定义子类。 |
| `ContentNegotiatingViewResolver` | `ViewResolver`基于请求文件名或`Accept`头解析视图的接口的实现。请参阅[内容协商](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multiple-representations)。 |

##### 处理方式

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-viewresolution-handling)

您可以通过声明多个解析器bean，并在必要时通过设置`order`属性以指定顺序来链接视图解析器。请记住，order属性越高，视图解析器在链中的定位就越晚。

a的协定`ViewResolver`指定它可以返回null来指示找不到该视图。但是，对于JSP和`InternalResourceViewResolver`，要弄清JSP是否存在，唯一的方法是通过进行调度 `RequestDispatcher`。因此，您必须始终将`InternalResourceViewResolver` View解析器的总体顺序配置为最后。

配置视图分辨率就像将`ViewResolver`bean 添加到Spring配置中一样简单。所述[MVC配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)提供了一种专用配置API [视图解析器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-view-resolvers)和用于将逻辑较少 [视图控制器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-view-controller)，其可用于HTML模板呈现有用而不控制器逻辑。

##### 重新导向

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-redirecting-redirect-prefix)

`redirect:`视图名称中的特殊前缀使您可以执行重定向。的 `UrlBasedViewResolver`（和它的子类）识别为是需要重定向的指令。视图名称的其余部分是重定向URL。

最终效果与控制器返回a相同`RedirectView`，但是现在控制器本身可以根据逻辑视图名称进行操作。逻辑视图名称（例如`redirect:/myapp/some/resource`）相对于当前Servlet上下文`redirect:https://myhost.com/some/arbitrary/path` 重定向，而名称（例如）重定向到绝对URL。

请注意，如果使用注释控制器方法`@ResponseStatus`，则注释值优先于设置的响应状态`RedirectView`。

##### 转寄

您还可以`forward:`对视图名称使用特殊的前缀，这些视图名称最终由`UrlBasedViewResolver`和子类解析。这将创建一个 `InternalResourceView`，并执行一个`RequestDispatcher.forward()`。因此，此前缀在`InternalResourceViewResolver`和中 `InternalResourceView`（对于JSP）没有用，但是如果您使用另一种视图技术，但仍然希望强制转发由Servlet / JSP引擎处理的资源，则该前缀很有用。请注意，您也可以链接多个视图解析器。

##### 内容协商

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-multiple-representations)

[`ContentNegotiatingViewResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/view/ContentNegotiatingViewResolver.html) 不会解析视图本身，而是委托其他视图解析器，并选择类似于客户端请求的表示形式的视图。可以从`Accept`标题或查询参数（例如`"/path?format=pdf"`）中确定表示形式。

在`ContentNegotiatingViewResolver`选择一个合适的`View`，以通过与所述媒体类型（也称为比较请求的媒体类型处理该请求 `Content-Type`由所述支持）`View`与每个其相关联`ViewResolvers`。`View`列表中的第一个具有兼容`Content-Type`属性的列表将表示形式返回给客户端。如果`ViewResolver`链不能提供兼容的视图，那么将查询通过该`DefaultViews`属性指定的视图列表。后一个选项适用于单例`Views`，无论逻辑视图名称如何，该单例都可以呈现当前资源的适当表示形式。的`Accept` 报头可以包括通配符（例如`text/*`），在这种情况下`View`，其 `Content-Type`是`text/xml`为相容的匹配。

有关配置详细信息，请参见[MVC Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)下的[查看解析器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-view-resolvers)。

#### 1.1.9。区域设置

正如Spring Web MVC框架所做的那样，Spring体系结构的大多数部分都支持国际化。`DispatcherServlet`使您可以使用客户端的语言环境自动解析消息。这是通过`LocaleResolver`对象完成的。

收到请求时，将`DispatcherServlet`查找一个语言环境解析器，如果找到一个，它将尝试使用它来设置语言环境。通过使用该`RequestContext.getLocale()` 方法，您始终可以检索由语言环境解析器解析的语言环境。

除了自动的语言环境解析之外，您还可以在处理程序映射上附加一个拦截器（有关处理程序映射拦截[器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-handlermapping-interceptor)的更多信息，请参见拦截），以在特定情况下（例如，基于请求中的参数）更改语言环境。

语言环境解析器和拦截器在`org.springframework.web.servlet.i18n`程序包中定义， 并以常规方式在应用程序上下文中配置。Spring包含以下选择的语言环境解析器。

- [时区](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-timezone)
- [标头解析器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-localeresolver-acceptheader)
- [Cookie解析器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-localeresolver-cookie)
- [会话解析器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-localeresolver-session)
- [区域拦截器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-localeresolver-interceptor)

##### 时区

除了获取客户的语言环境外，了解其时区通常也很有用。该`LocaleContextResolver`界面提供了扩展`LocaleResolver`，使解析程序可以提供更丰富的内容`LocaleContext`，其中可能包含时区信息。

如果可用，则`TimeZone`可以使用`RequestContext.getTimeZone()`方法获得 用户的信息。Spring的任何日期/时间`Converter`和`Formatter`对象 都会自动使用时区信息`ConversionService`。

##### 标头解析器

此语言环境解析器检查`accept-language`客户端（例如，Web浏览器）发送的请求中的标头。通常，此头字段包含客户端操作系统的语言环境。请注意，此解析器不支持时区信息。

##### Cookie解析器

该语言环境解析器检查`Cookie`客户端上可能存在的，以查看是否 指定`Locale`或`TimeZone`。如果是这样，它将使用指定的详细信息。通过使用此语言环境解析器的属性，可以指定Cookie的名称以及最长期限。以下示例定义了一个`CookieLocaleResolver`：

```xml
<bean id="localeResolver" class="org.springframework.web.servlet.i18n.CookieLocaleResolver">

    <property name="cookieName" value="clientlanguage"/>

    <!-- in seconds. If set to -1, the cookie is not persisted (deleted when browser shuts down) -->
    <property name="cookieMaxAge" value="100000"/>

</bean>
```

下表描述了这些属性`CookieLocaleResolver`：

| 属性           | 默认            | 描述                                                         |
| :------------- | :-------------- | :----------------------------------------------------------- |
| `cookieName`   | 类名+ LOCALE    | Cookie的名称                                                 |
| `cookieMaxAge` | Servlet容器默认 | Cookie在客户端上保留的最长时间。如果`-1`指定，则cookie将不会保留。它仅在客户端关闭浏览器之前可用。 |
| `cookiePath`   | /               | 将Cookie的可见性限制在您网站的特定部分。当`cookiePath`被指定，cookie是仅对于该路径和它下面的路径。 |

##### 会话解析器

将`SessionLocaleResolver`允许您检索`Locale`和`TimeZone`从可能与用户的请求相关的会话。相反 `CookieLocaleResolver`，此策略将本地选择的语言环境设置存储在Servlet容器的中`HttpSession`。结果，这些设置对于每个会话都是临时的，因此在每个会话终止时会丢失。

请注意，与外部会话管理机制（例如Spring Session项目）没有直接关系。这将针对current `SessionLocaleResolver`评估和修改相应的`HttpSession`属性`HttpServletRequest`。

##### 区域拦截器

您可以通过将定义添加`LocaleChangeInterceptor`到其中之一 来启用语言环境更改`HandlerMapping`。它检测到请求中的参数并相应地更改语言环境，`setLocale`从而`LocaleResolver`在调度程序的应用程序上下文中调用方法。下一个示例显示，调用`*.view`包含`siteLanguage`现在名为参数的参数的所有资源都会更改语言环境。因此，例如，对URL的请求`https://www.sf.net/home.view?siteLanguage=nl`将站点语言更改为荷兰语。以下示例显示如何拦截语言环境：

```xml
<bean id="localeChangeInterceptor"
        class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor">
    <property name="paramName" value="siteLanguage"/>
</bean>

<bean id="localeResolver"
        class="org.springframework.web.servlet.i18n.CookieLocaleResolver"/>

<bean id="urlMapping"
        class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="interceptors">
        <list>
            <ref bean="localeChangeInterceptor"/>
        </list>
    </property>
    <property name="mappings">
        <value>/**/*.view=someController</value>
    </property>
</bean>
```

#### 1.1.10。主题

您可以应用Spring Web MVC框架主题来设置应用程序的整体外观，从而增强用户体验。主题是静态资源（通常是样式表和图像）的集合，这些资源会影响应用程序的视觉样式。

##### 定义主题

要在Web应用程序中使用主题，必须设置`org.springframework.ui.context.ThemeSource`接口的实现 。该`WebApplicationContext` 接口可以扩展，`ThemeSource`但将其职责委托给专用的实现。默认情况下，委托是`org.springframework.ui.context.support.ResourceBundleThemeSource`从类路径的根加载属性文件的 实现。要使用自定义`ThemeSource` 实现或配置的基本名称前缀`ResourceBundleThemeSource`，可以在应用程序上下文中使用保留名称注册Bean `themeSource`。Web应用程序上下文会自动检测到具有该名称的bean并使用它。

使用时`ResourceBundleThemeSource`，将在一个简单的属性文件中定义一个主题。属性文件列出了组成主题的资源，如以下示例所示：

```
styleSheet = / themes / cool / style.css
背景= / themes / cool / img / coolBg.jpg
```

属性的键是从视图代码引用主题元素的名称。对于JSP，通常使用`spring:theme`自定义标签执行此操作，该标签与该`spring:message`标签非常相似。以下JSP片段使用上一个示例中定义的主题来自定义外观：

```xml
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<html>
    <head>
        <link rel="stylesheet" href="<spring:theme code='styleSheet'/>" type="text/css"/>
    </head>
    <body style="background=<spring:theme code='background'/>">
        ...
    </body>
</html>
```

默认情况下，`ResourceBundleThemeSource`使用空的基本名称前缀。结果，从类路径的根加载属性文件。因此，您可以将 `cool.properties`主题定义放在类路径根目录下的目录中（例如in中`/WEB-INF/classes`）。在`ResourceBundleThemeSource`使用标准的Java资源包加载机制，允许主题的国际化。例如，我们可以使用`/WEB-INF/classes/cool_nl.properties`引用带有荷兰文字的特殊背景图片。

##### 解决主题

定义主题后，如上[一节所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-themeresolver-defining)，您可以决定使用哪个主题。在`DispatcherServlet`查找名为豆`themeResolver` 找出哪个`ThemeResolver`使用实施。主题解析器的工作方式与主题解析器的大致相同`LocaleResolver`。它可以检测用于特定请求的主题，还可以更改请求的主题。下表描述了Spring提供的主题解析器：

| 类                     | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| `FixedThemeResolver`   | 选择通过使用`defaultThemeName`属性设置的固定主题。           |
| `SessionThemeResolver` | 主题在用户的HTTP会话中维护。每个会话只需设置一次，但在会话之间不会保留。 |
| `CookieThemeResolver`  | 所选主题存储在客户端的cookie中。                             |

Spring还提供了一个`ThemeChangeInterceptor`允许使用简单的请求参数对每个请求进行主题更改的功能。

#### 1.1.11。多部分解析器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-multipart)

`MultipartResolver`从`org.springframework.web.multipart`包中提取是一种分析多部分请求（包括文件上传）的策略。有一种基于[Commons FileUpload的实现](https://jakarta.apache.org/commons/fileupload)，另一种基于Servlet 3.0多部分请求解析。

要启用多部分处理，您需要`MultipartResolver`在`DispatcherServlet`Spring配置中声明一个名称为的Bean `multipartResolver`。在`DispatcherServlet`检测到它，并将其应用于传入请求。当与内容类型的POST `multipart/form-data`被接收时，分解器解析该内容和包装了当前`HttpServletRequest`作为`MultipartHttpServletRequest`在揭除它们作为请求参数提供到解析部件的访问。

##### Apache Commons `FileUpload`

要使用Apache Commons `FileUpload`，可以配置`CommonsMultipartResolver`名称为的类型的Bean `multipartResolver`。您还需要`commons-fileupload`依赖于类路径。

##### Servlet 3.0

需要通过Servlet容器配置启用Servlet 3.0多部分解析。为此：

- 在Java中，`MultipartConfigElement`在Servlet上设置a 。
- 在中`web.xml`，`""`向Servlet声明中添加一个部分。

以下示例显示了如何`MultipartConfigElement`在Servlet注册上设置A ：



 

```java
public class AppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    // ...

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        // Optionally also set maxFileSize, maxRequestSize, fileSizeThreshold
        registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
    }

}
```

Servlet 3.0配置到位后，您可以添加`StandardServletMultipartResolver`名称为的类型的Bean `multipartResolver`。

#### 1.1.12。记录中

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-logging)

Spring MVC中的DEBUG级别的日志被设计为紧凑，最少且人性化的。它侧重于一遍又一遍有用的高价值信息，而其他信息仅在调试特定问题时才有用。

TRACE级别的日志记录通常遵循与DEBUG相同的原理（例如，也不应是消防水带），但可用于调试任何问题。此外，某些日志消息在TRACE和DEBUG上可能显示不同级别的详细信息。

良好的日志记录来自使用日志的经验。如果您发现任何不符合既定目标的东西，请告诉我们。

##### 敏感数据

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-logging-sensitive-data)

调试和跟踪日志记录可能会记录敏感信息。这就是默认情况下屏蔽请求参数和标头，并且必须通过`enableLoggingRequestDetails`on属性显式启用其完整登录的原因`DispatcherServlet`。

以下示例显示了如何通过使用Java配置来执行此操作：



 

```java
public class MyInitializer
        extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return ... ;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return ... ;
    }

    @Override
    protected String[] getServletMappings() {
        return ... ;
    }

    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {
        registration.setInitParameter("enableLoggingRequestDetails", "true");
    }

}
```

### 1.2。筛选器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-filters)

该`spring-web`模块提供了一些有用的过滤器：

- [表格数据](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#filters-http-put)
- [转发的标题](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#filters-forwarded-headers)
- [浅ETag](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#filters-shallow-etag)
- [CORS](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#filters-cors)

#### 1.2.1。表格数据

浏览器只能通过HTTP GET或HTTP POST提交表单数据，但非浏览器客户端也可以使用HTTP PUT，PATCH和DELETE。Servlet API需要一些`ServletRequest.getParameter*()` 方法来仅支持HTTP POST的表单字段访问。

该`spring-web`模块提供`FormContentFilter`了拦截内容类型为HTTP PUT，PATCH和DELETE的请求，`application/x-www-form-urlencoded`从请求主体读取表单数据，并包装`ServletRequest`使其通过`ServletRequest.getParameter*()`一系列方法可用的表单数据。

#### 1.2.2。转发的标题

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-forwarded-headers)

当请求通过代理（例如负载平衡器）进行处理时，主机，端口和方案可能会更改，这使得从客户端角度创建指向正确的主机，端口和方案的链接带来了挑战。

[RFC 7239](https://tools.ietf.org/html/rfc7239)定义了`Forwarded`代理可以用来提供有关原始请求的信息的HTTP标头。还有其他一些非标头，也包括`X-Forwarded-Host`，`X-Forwarded-Port`， `X-Forwarded-Proto`，`X-Forwarded-Ssl`，和`X-Forwarded-Prefix`。

`ForwardedHeaderFilter`是一个Servlet过滤器，用于修改请求，以便a）基于`Forwarded`标头更改主机，端口和方案，b）删除那些标头以消除进一步的影响。过滤器依赖于包装请求，因此必须先于其他过滤器（例如）订购，该过滤器`RequestContextFilter`应与修改后的请求而不是原始请求一起使用。

对于转发的标头，出于安全方面的考虑，因为应用程序无法知道标头是由代理添加的，还是由恶意客户端添加的。这就是为什么应配置位于信任边界的代理以删除`Forwarded` 来自外部的不受信任的标头的原因。您也可以配置`ForwardedHeaderFilter` with `removeOnly=true`，在这种情况下，它会删除但不使用标题。

为了支持[异步请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)和错误调度，此过滤器应与`DispatcherType.ASYNC`和也映射`DispatcherType.ERROR`。如果使用Spring Framework的`AbstractAnnotationConfigDispatcherServletInitializer` （请参阅[Servlet Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-container-config)），则会为所有调度类型自动注册所有过滤器。但是，如果通过注册该过滤器`web.xml`通过或在春季启动 `FilterRegistrationBean`时一定要包括`DispatcherType.ASYNC`和 `DispatcherType.ERROR`除`DispatcherType.REQUEST`。

#### 1.2.3。浅ETag

所述`ShallowEtagHeaderFilter`过滤器通过缓存写入响应的内容，并从它计算MD5哈希创建一个“浅”的ETag。客户端下一次发送时，将执行相同的操作，但还会将计算出的值与`If-None-Match` 请求标头进行比较，如果两者相等，则返回304（NOT_MODIFIED）。

此策略可节省网络带宽，但不会节省CPU，因为必须为每个请求计算完整响应。如前所述，控制器级别的其他策略可以避免计算。请参阅[HTTP缓存](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-caching)。

该过滤器具有一个`writeWeakETag`参数，该参数将过滤器配置为写入弱ETag，类似于以下内容：（`W/"02a2d595e6ed9a0b24f027f2b63b134d6"`如[RFC 7232第2.3节中](https://tools.ietf.org/html/rfc7232#section-2.3)所定义 ）。

为了支持[异步请求，](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)必须对此过滤器进行映射，`DispatcherType.ASYNC`以便过滤器可以延迟并成功生成ETag到最后一个异步调度的末尾。如果使用Spring Framework的 `AbstractAnnotationConfigDispatcherServletInitializer`（请参阅[Servlet Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-container-config)），则会为所有调度类型自动注册所有过滤器。但是，如果通过`web.xml`或在Spring Boot中通过注册过滤器，请`FilterRegistrationBean`确保包含 `DispatcherType.ASYNC`。

#### 1.2.4。CORS

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-filters-cors)

Spring MVC通过控制器上的注释为CORS配置提供了细粒度的支持。但是，当与Spring Security一起使用时，我们建议依赖于 `CorsFilter`必须在Spring Security的过滤器链之前订购的内置组件。

有关更多详细信息，请参见有关[CORS](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-cors)和[CORS过滤器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-cors-filter)的部分。

### 1.3。带注释的控制器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-controller)

Spring MVC提供了一个基于注释的编程模型，其中`@Controller`和 `@RestController`组件使用注释来表达请求映射，请求输入，异常处理等。带注释的控制器具有灵活的方法签名，无需扩展基类或实现特定的接口。以下示例显示了由注释定义的控制器：



 

```java
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String handle(Model model) {
        model.addAttribute("message", "Hello World!");
        return "index";
    }
}
```

在前面的示例中，该方法接受a `Model`并以a的形式返回视图名称`String`，但是还存在许多其他选项，本章稍后将对此进行说明。

|      | [spring.io](https://spring.io/guides) 上的指南和教程使用本节中描述的基于注释的编程模型。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.3.1。宣言

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-controller)

您可以通过使用Servlet的标准Spring bean定义来定义控制器bean `WebApplicationContext`。该`@Controller`原型允许自动检测，使用Spring普遍支持检测对准`@Component`在类路径中类和自动注册bean定义他们。它还充当带注释类的构造型，表明其作为Web组件的作用。

要启用对此类`@Controller`bean的自动检测，可以将组件扫描添加到Java配置中，如以下示例所示：



 

```java
@Configuration
@ComponentScan("org.example.web")
public class WebConfig {

    // ...
}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example.web"/>

    <!-- ... -->

</beans>
```

`@RestController`是一个[组合式注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-meta-annotations)，它本身带有元注释，`@Controller`并`@ResponseBody`指示一个控制器，该控制器的每个方法都继承了类型级别的`@ResponseBody`注释，因此，与视图分辨率和HTML模板渲染相比，它直接写入响应主体。

##### AOP代理

在某些情况下，您可能需要在运行时用AOP代理装饰控制器。一个示例是，如果您选择`@Transactional`直接在控制器上具有批注。在这种情况下，特别是对于控制器，我们建议使用基于类的代理。这通常是控制器的默认选择。但是，如果一个控制器必须实现一个接口，这不是一个Spring上下文回调（例如`InitializingBean`，`*Aware`和其他人），您可能需要显式配置基于类的代理。例如，使用``可以更改为``，并且 `@EnableTransactionManagement`可以使用更改为 `@EnableTransactionManagement(proxyTargetClass = true)`。

#### 1.3.2。请求映射

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping)

您可以使用`@RequestMapping`批注将请求映射到控制器方法。它具有各种属性，可以通过URL，HTTP方法，请求参数，标头和媒体类型进行匹配。您可以在类级别使用它来表示共享的映射，也可以在方法级别使用它来缩小到特定的端点映射。

还有HTTP方法特定的快捷方式变体`@RequestMapping`：

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`

快捷方式是提供的“ [自定义注释”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-composed)，因为可以说，大多数控制器方法应映射到特定的HTTP方法，而不是使用using `@RequestMapping`（默认情况下，它与所有HTTP方法匹配）。同时，`@RequestMapping`在类级别仍需要a 来表示共享映射。

以下示例具有类型和方法级别的映射：



 

```java
@RestController
@RequestMapping("/persons")
class PersonController {

    @GetMapping("/{id}")
    public Person getPerson(@PathVariable Long id) {
        // ...
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public void add(@RequestBody Person person) {
        // ...
    }
}
```

##### URI模式

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-uri-templates)

您可以使用全局模式和通配符来映射请求：

| 图案            | 描述                                             | 例                                                           |
| :-------------- | :----------------------------------------------- | :----------------------------------------------------------- |
| `?`             | 匹配一个字符                                     | `"/pages/t?st.html"`比赛`"/pages/test.html"` 和`"/pages/t3st.html"` |
| `*`             | 匹配路径段中的零个或多个字符                     | `"/resources/*.png"` 火柴 `"/resources/file.png"``"/projects/*/versions"`匹配`"/projects/spring/versions"`但不匹配`"/projects/spring/boot/versions"` |
| `**`            | 匹配零个或多个路径段，直到路径结束               | `"/resources/**"`比赛`"/resources/file.png"`和`"/resources/images/file.png"` |
| `{name}`        | 匹配路径段并将其捕获为名为“ name”的变量          | `"/projects/{project}/versions"`比赛`"/projects/spring/versions"`和捕获`project=spring` |
| `{name:[a-z]+}` | 将正则表达式匹配`"[a-z]+"`为名为“名称”的路径变量 | `"/projects/{project:[a-z]+}/versions"`匹配`"/projects/spring/versions"`但不匹配`"/projects/spring1/versions"` |

捕获的URI变量可以通过访问`@PathVariable`，如以下示例所示：



 

```java
@GetMapping("/owners/{ownerId}/pets/{petId}")
public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
    // ...
}
```

您可以在类和方法级别声明URI变量，如以下示例所示：



 

```java
@Controller
@RequestMapping("/owners/{ownerId}")
public class OwnerController {

    @GetMapping("/pets/{petId}")
    public Pet findPet(@PathVariable Long ownerId, @PathVariable Long petId) {
        // ...
    }
}
```

URI变量会自动转换为适当的类型，或者`TypeMismatchException` 引发。简单类型（`int`，`long`，`Date`，等）默认支持，你可以注册任何其它数据类型的支持。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-typeconversion)和[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-initbinder)。

您可以显式命名URI变量（例如`@PathVariable("customId")`），但是如果名称相同并且您的代码是通过调试信息或`-parameters`Java 8上的编译器标志进行编译的，则可以省去该细节。

语法`{varName:regex}`使用正则表达式声明语法为的URI变量`{varName:regex}`。例如，给定URL `"/spring-web-3.0.5 .jar"`，以下方法提取名称，版本和文件扩展名：



 

```java
@GetMapping("/{name:[a-z-]+}-{version:\\d\\.\\d\\.\\d}{ext:\\.[a-z]+}")
public void handle(@PathVariable String version, @PathVariable String ext) {
    // ...
}
```

URI路径模式还可以具有嵌入式`${…}`占位符，这些占位符在启动时可以通过`PropertyPlaceHolderConfigurer`针对本地，系统，环境和其他属性源进行解析。例如，您可以使用它来基于某些外部配置参数化基本URL。

|      | Spring MVC使用`PathMatcher`合同和`AntPathMatcher`from 的实现 `spring-core`进行URI路径匹配。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 模式比较

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-pattern-comparison)

当多个模式与URL匹配时，必须将它们进行比较以找到最佳匹配。这是通过使用来完成的`AntPathMatcher.getPatternComparator(String path)`，该工具查找更具体的模式。

如果模式的URI变量（计数为1），单通配符（计数为1）和双通配符（计数为2）的数量较少，则模式的含义不太明确。给定相等的分数，则选择更长的模式。给定相同的分数和长度，将选择URI变量比通配符更多的模式。

默认映射模式（`/**`）从评分中排除，并且始终排在最后。另外，前缀模式（例如`/public/**`）被认为比没有双通配符的其他模式更具体。

有关完整的详细信息，请参见[`AntPatternComparator`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/util/AntPathMatcher.AntPatternComparator.html) 中[`AntPathMatcher`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/util/AntPathMatcher.html)，也请记住，你可以自定义[`PathMatcher`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/util/PathMatcher.html)的实现。请参阅配置部分中的[路径匹配](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-path-matching)。

##### 后缀匹配

默认情况下，Spring MVC执行`.*`后缀模式匹配，因此映射到的控制器`/person`也隐式映射到`/person.*`。然后，将文件的扩展名是用于解释所请求的内容类型来使用该响应（即，代替`Accept`标题） -例如，`/person.pdf`， `/person.xml`，和其他。

当浏览器用来发送`Accept`难以一致解释的标头时，以这种方式使用文件扩展名是必要的。目前，这已不再是必须的，使用`Accept`标头应该是首选。

随着时间的流逝，文件扩展名的使用已经以各种方式证明是有问题的。当使用URI变量，路径参数和URI编码进行覆盖时，可能会引起歧义。关于基于URL的授权和安全性的推理（请参阅下一部分以了解更多详细信息）也变得更加困难。

若要完全禁用文件扩展名，必须设置以下两项：

- `useSuffixPatternMatching(false)`，请参阅[PathMatchConfigurer](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-path-matching)
- `favorPathExtension(false)`，请参阅[ContentNegotiationConfigurer](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-content-negotiation)

基于URL的内容协商仍然有用（例如，在浏览器中键入URL时）。为此，我们建议使用基于查询参数的策略，以避免文件扩展名附带的大多数问题。或者，如果必须使用文件扩展名，请考虑通过[ContentNegotiationConfigurer](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-content-negotiation)的`mediaTypes`属性将它们限制为显式注册的扩展名列表 。

|      | 从5.2.4开始， 已弃用与路径扩展相关的选项，这些选项用于[RequestMappingHandlerMapping中的](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/org/springframework/web/servlet/mvc/method/annotation/RequestMappingHandlerMapping.java)请求映射 和[ContentNegotiationManagerFactoryBean](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/org.springframework.web.accept/ContentNegotiationManagerFactoryBean.java)中的内容协商 。有关更多计划，请参见Spring Framework [第24179期](https://github.com/spring-projects/spring-framework/issues/24179)和相关问题。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 后缀匹配和RFD

反射文件下载（RFD）攻击与XSS相似，因为它依赖反映在响应中的请求输入（例如，查询参数和URI变量）。但是，RFD攻击不是将JavaScript插入HTML，而是依靠浏览器切换来执行下载，并在以后双击时将响应视为可执行脚本。

在Spring MVC中，`@ResponseBody`和`ResponseEntity`方法是有风险的，因为它们可以呈现不同的内容类型，客户端可以通过URL路径扩展要求。禁用后缀模式匹配并使用路径扩展进行内容协商可以降低风险，但不足以防止RFD攻击。

为了防止RFD攻击，Spring MVC在呈现响应主体之前添加了 `Content-Disposition:inline;filename=f.txt`标头，以建议固定和安全的下载文件。仅当URL路径包含既未列入白名单也未明确注册用于内容协商的文件扩展名时，才执行此操作。但是，当直接在浏览器中键入URL时，它可能会产生副作用。

默认情况下，许多常见路径扩展名都列入了白名单。具有自定义`HttpMessageConverter`实现的应用程序 可以显式注册文件扩展名以进行内容协商，以避免`Content-Disposition`为这些扩展名添加头。请参阅[内容类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-content-negotiation)。

有关RFD的其他建议，请参见[CVE-2015-5211](https://pivotal.io/security/cve-2015-5211)。

##### 消耗媒体类型

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-consumes)

您可以根据请求的来缩小请求映射`Content-Type`，如以下示例所示：



 

```java
@PostMapping(path = "/pets", consumes = "application/json") 
public void addPet(@RequestBody Pet pet) {
    // ...
}
```

|      | 使用`consumes`属性按内容类型缩小映射。 |
| ---- | -------------------------------------- |
|      |                                        |

该`consumes`属性还支持否定表达式-例如，`!text/plain`表示以外的任何内容类型`text/plain`。

您可以`consumes`在类级别声明共享属性。但是，与大多数其他请求映射属性不同，在类级别使用时，方法级别的`consumes`属性将覆盖而不是扩展类级别的声明。

|      | `MediaType`提供常用媒体类型（例如`APPLICATION_JSON_VALUE`和）的常量 `APPLICATION_XML_VALUE`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 可生产的媒体类型

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-produces)

您可以根据`Accept`请求标头和控制器方法生成的内容类型列表来缩小请求映射，如以下示例所示：



 

```java
@GetMapping(path = "/pets/{petId}", produces = "application/json") 
@ResponseBody
public Pet getPet(@PathVariable String petId) {
    // ...
}
```

|      | 使用`produces`属性按内容类型缩小映射。 |
| ---- | -------------------------------------- |
|      |                                        |

媒体类型可以指定字符集。支持否定表达式-例如， `!text/plain`表示除“文本/纯文本”之外的任何内容类型。

您可以`produces`在类级别声明共享属性。但是，与大多数其他请求映射属性不同，在类级别使用时，方法级别的`produces`属性将覆盖而不是扩展类级别的声明。

|      | `MediaType`提供常用媒体类型（例如`APPLICATION_JSON_VALUE`和）的常量 `APPLICATION_XML_VALUE`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 参数，标题

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-params-and-headers)

您可以根据请求参数条件来缩小请求映射。您可以测试是否存在请求参数（`myParam`），是否存在请求参数（）`!myParam`或特定值（`myParam=myValue`）。以下示例显示如何测试特定值：



 

```java
@GetMapping(path = "/pets/{petId}", params = "myParam=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

|      | 测试是否`myParam`相等`myValue`。 |
| ---- | -------------------------------- |
|      |                                  |

您还可以将其与请求标头条件一起使用，如以下示例所示：



 

```java
@GetMapping(path = "/pets", headers = "myHeader=myValue") 
public void findPet(@PathVariable String petId) {
    // ...
}
```

|      | 测试是否`myHeader`相等`myValue`。 |
| ---- | --------------------------------- |
|      |                                   |

|      | 您可以匹配`Content-Type`并`Accept`与标头条件匹配，但最好使用 [消耗](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-consumes)和[生产](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-produces) 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### HTTP HEAD，选项

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-head-options)

`@GetMapping`（和`@RequestMapping(method=HttpMethod.GET)`）透明支持HTTP HEAD以进行请求映射。控制器方法不需要更改。应用于的响应包装器`javax.servlet.http.HttpServlet`确保将`Content-Length` 标头设置为写入的字节数（实际上未写入响应）。

`@GetMapping`（和`@RequestMapping(method=HttpMethod.GET)`）被隐式映射到并支持HTTP HEAD。像处理HTTP GET一样处理HTTP HEAD请求，不同的是，不是写入正文，而是计算字节数并设置`Content-Length` 标头。

默认情况下，通过将`Allow`响应标头设置为所有`@RequestMapping`具有匹配URL模式的方法中列出的HTTP方法列表来处理HTTP OPTIONS 。

对于`@RequestMapping`不使用HTTP方法声明的情况，`Allow`标头设置为 `GET,HEAD,POST,PUT,PATCH,DELETE,OPTIONS`。控制器方法应该总是声明支持HTTP方法（例如，通过使用HTTP方法具体变体： `@GetMapping`，`@PostMapping`，及其他）。

您可以将`@RequestMapping`方法显式映射到HTTP HEAD和HTTP OPTIONS，但这在通常情况下不是必需的。

##### 自定义注释

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#mvc-ann-requestmapping-head-options)

Spring MVC支持将[组合注释](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-meta-annotations) 用于请求映射。这些是注释本身，它们本身具有元注释， `@RequestMapping`并被组成以`@RequestMapping` 具有更窄，更具体的目的重新声明属性的子集（或全部）。

`@GetMapping`，`@PostMapping`，`@PutMapping`，`@DeleteMapping`，和`@PatchMapping`由注解的例子。之所以提供它们，是因为大多数控制器方法应该映射到特定的HTTP方法，而不是使用using `@RequestMapping`（默认情况下与所有HTTP方法匹配）。如果需要组合注释的示例，请查看如何声明它们。

Spring MVC还支持带有自定义请求匹配逻辑的自定义请求映射属性。这是一个更高级的选项，它需要`RequestMappingHandlerMapping`对`getCustomMethodCondition`方法进行子类化 和覆盖，您可以在其中检查custom属性并返回您自己的方法`RequestCondition`。

##### 明确注册

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestmapping-registration)

您可以以编程方式注册处理程序方法，这些方法可用于动态注册或高级案例，例如同一处理程序在不同URL下的不同实例。下面的示例注册一个处理程序方法：



 

```java
@Configuration
public class MyConfig {

    @Autowired
    public void setHandlerMapping(RequestMappingHandlerMapping mapping, UserHandler handler) 
            throws NoSuchMethodException {

        RequestMappingInfo info = RequestMappingInfo
                .paths("/user/{id}").methods(RequestMethod.GET).build(); 

        Method method = UserHandler.class.getMethod("getUser", Long.class); 

        mapping.registerMapping(info, handler, method); 
    }
}
```

|      | 注入目标处理程序和控制器的处理程序映射。 |
| ---- | ---------------------------------------- |
|      | 准备请求映射元数据。                     |
|      | 获取处理程序方法。                       |
|      | 添加注册。                               |

#### 1.3.3。处理程序方法

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-methods)

`@RequestMapping` 处理程序方法具有灵活的签名，可以从支持的控制器方法参数和返回值的范围中进行选择。

##### 方法参数

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-arguments)

下表描述了受支持的控制器方法参数。任何参数均不支持反应性类型。

JDK 8次的`java.util.Optional`被支撑作为组合的方法的参数与具有注解`required`的属性（例如，`@RequestParam`，`@RequestHeader`，和其它物质）和相当于`required=false`。

| 控制器方法参数                                               | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `WebRequest`， `NativeWebRequest`                            | 通用访问请求参数以及请求和会话属性，而无需直接使用Servlet API。 |
| `javax.servlet.ServletRequest`， `javax.servlet.ServletResponse` | 选择任何特定的请求或响应类型-例如`ServletRequest`，`HttpServletRequest`或春天的`MultipartRequest`，`MultipartHttpServletRequest`。 |
| `javax.servlet.http.HttpSession`                             | 强制会话的存在。结果，这种论据永远不会`null`。请注意，会话访问不是线程安全的。考虑将`RequestMappingHandlerAdapter`实例的`synchronizeOnSession`标志设置 为`true`是否允许多个请求同时访问会话。 |
| `javax.servlet.http.PushBuilder`                             | 用于程序化HTTP / 2资源推送的Servlet 4.0推送构建器API。请注意，根据Servlet规范，`PushBuilder`如果客户端不支持HTTP / 2功能，则注入的实例可以为null。 |
| `java.security.Principal`                                    | 当前经过身份验证的用户-可能是特定的`Principal`实现类（如果已知）。 |
| `HttpMethod`                                                 | 请求的HTTP方法。                                             |
| `java.util.Locale`                                           | 当前请求的语言环境，由最具体的`LocaleResolver`可用语言（实际上是配置的`LocaleResolver`或`LocaleContextResolver`）确定。 |
| `java.util.TimeZone` + `java.time.ZoneId`                    | 与当前请求关联的时区，由决定`LocaleContextResolver`。        |
| `java.io.InputStream`， `java.io.Reader`                     | 用于访问Servlet API公开的原始请求正文。                      |
| `java.io.OutputStream`， `java.io.Writer`                    | 用于访问Servlet API公开的原始响应正文。                      |
| `@PathVariable`                                              | 用于访问URI模板变量。请参阅[URI模式](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-uri-templates)。 |
| `@MatrixVariable`                                            | 用于访问URI路径段中的名称/值对。请参阅[矩阵变量](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-matrix-variables)。 |
| `@RequestParam`                                              | 用于访问Servlet请求参数，包括多部分文件。参数值将转换为声明的方法参数类型。参见[`@RequestParam`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestparam)以及[Multipart](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multipart-forms)。请注意，`@RequestParam`对于简单参数值，使用是可选的。请参阅此表末尾的“其他任何参数”。 |
| `@RequestHeader`                                             | 用于访问请求标头。标头值将转换为声明的方法参数类型。请参阅[`@RequestHeader`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestheader)。 |
| `@CookieValue`                                               | 用于访问cookie。Cookies值将转换为声明的方法参数类型。请参阅[`@CookieValue`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-cookievalue)。 |
| `@RequestBody`                                               | 用于访问HTTP请求正文。正文内容通过使用`HttpMessageConverter`实现转换为声明的方法参数类型。请参阅[`@RequestBody`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestbody)。 |
| `HttpEntity`                                                 | 用于访问请求标头和正文。身体用转换`HttpMessageConverter`。参见[HttpEntity](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-httpentity)。 |
| `@RequestPart`                                               | 要访问`multipart/form-data`请求中的零件，请使用来转换零件的主体`HttpMessageConverter`。参见[多部分](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multipart-forms)。 |
| `java.util.Map`，`org.springframework.ui.Model`，`org.springframework.ui.ModelMap` | 用于访问HTML控制器中使用的模型，并作为视图渲染的一部分公开给模板。 |
| `RedirectAttributes`                                         | 指定在重定向的情况下使用的属性（即追加到查询字符串中），并指定要临时存储的属性，直到重定向后的请求为止。请参阅[重定向属性](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-redirecting-passing-data)和[Flash属性](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-flash-attributes)。 |
| `@ModelAttribute`                                            | 用于访问应用了数据绑定和验证的模型中的现有属性（如果不存在，则进行实例化）。参见[`@ModelAttribute`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-modelattrib-method-args)以及 [模型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-modelattrib-methods)和[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-initbinder)。请注意，使用of `@ModelAttribute`是可选的（例如，设置其属性）。请参阅此表末尾的“其他任何参数”。 |
| `Errors`， `BindingResult`                                   | 用于访问从验证错误和数据的命令对象绑定（即，`@ModelAttribute`从一个验证参数）或错误`@RequestBody`或 `@RequestPart`参数。您必须在经过验证的方法参数之后立即声明`Errors`或`BindingResult`参数。 |
| `SessionStatus` +班级 `@SessionAttributes`                   | 为了标记表单处理完成，这将触发清除通过类级`@SessionAttributes`注释声明的会话属性。请参阅 [`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-sessionattributes)以获取更多详细信息。 |
| `UriComponentsBuilder`                                       | 用于准备相对于当前请求的主机，端口，方案，上下文路径以及servlet映射的文字部分的URL。请参阅[URI链接](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-uri-building)。 |
| `@SessionAttribute`                                          | 与访问由于类级`@SessionAttributes`声明而存储在会话中的模型属性相反，用于访问任何会话属性。请参阅 [`@SessionAttribute`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-sessionattribute)以获取更多详细信息。 |
| `@RequestAttribute`                                          | 用于访问请求属性。请参阅[`@RequestAttribute`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestattrib)以获取更多详细信息。 |
| 任何其他论点                                                 | 如果方法参数不与此表中的任何较早值匹配，并且是简单类型（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ，则将其解析为`@RequestParam`。否则，将其解析为`@ModelAttribute`。 |

##### 返回值

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-return-types)

下表描述了受支持的控制器方法返回值。所有返回值都支持反应性类型。

| 控制器方法返回值                                             | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `@ResponseBody`                                              | 返回值通过`HttpMessageConverter`实现进行转换并写入响应。请参阅[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-responsebody)。 |
| `HttpEntity`， `ResponseEntity`                              | 指定完整响应（包括HTTP标头和正文）的返回值将通过`HttpMessageConverter`实现进行转换，并写入响应中。参见[ResponseEntity](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-responseentity)。 |
| `HttpHeaders`                                                | 用于返回没有标题的响应。                                     |
| `String`                                                     | 一个视图名称，将通过`ViewResolver`实现来解析，并与隐式模型一起使用-通过命令对象和`@ModelAttribute`方法确定。处理程序方法还可以通过声明`Model`参数来以编程方式丰富模型（请参见[Explicit Registrations](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-registration)）。 |
| `View`                                                       | 甲`View`实例以使用用于与所述隐式模型一起渲染-通过命令对象和确定`@ModelAttribute`方法。处理程序方法还可以通过声明`Model`参数来以编程方式丰富模型（请参见[Explicit Registrations](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-registration)）。 |
| `java.util.Map`， `org.springframework.ui.Model`             | 要添加到隐式模型的属性，视图名称通过隐式确定`RequestToViewNameTranslator`。 |
| `@ModelAttribute`                                            | 要添加到模型的属性，视图名称通过隐式确定`RequestToViewNameTranslator`。请注意，这`@ModelAttribute`是可选的。请参阅此表末尾的“其他任何返回值”。 |
| `ModelAndView` 宾语                                          | 要使用的视图和模型属性，以及响应状态（可选）。               |
| `void`                                                       | 如果具有`void`返回类型（或`null`返回值）的方法还具有`ServletResponse`，`OutputStream`参数或`@ResponseStatus`注释，则认为该方法已完全处理了响应。如果控制器进行了肯定检查`ETag`或`lastModified`时间戳检查，则情况也是如此 （有关详细信息，请参阅[控制器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-caching-etag-lastmodified)）。如果以上条件都不成立，则`void`返回类型还可以为REST控制器指示“无响应正文”，或者为HTML控制器指示默认视图名称选择。 |
| `DeferredResult`                                             | 从任何线程异步生成任何上述返回值-例如，由于某些事件或回调的结果。请参见[异步请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)和[`DeferredResult`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-deferredresult)。 |
| `Callable`                                                   | 在Spring MVC管理的线程中异步产生上述任何返回值。请参见[异步请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)和[`Callable`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-callable)。 |
| `ListenableFuture`， `java.util.concurrent.CompletionStage`， `java.util.concurrent.CompletableFuture` | `DeferredResult`为方便起见，替代，（例如，当基础服务返回其中之一时）。 |
| `ResponseBodyEmitter`， `SseEmitter`                         | 异步发出对象流，以将其写入 `HttpMessageConverter`实现中。也支持作为的主体`ResponseEntity`。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)和[HTTP流](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-http-streaming)。 |
| `StreamingResponseBody`                                      | `OutputStream`异步写入响应。也支持作为的主体 `ResponseEntity`。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)和[HTTP流](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-http-streaming)。 |
| 反应类型-Reactor，RxJava或其他类型 `ReactiveAdapterRegistry` | 替代收集到的`DeferredResult`多值流（例如`Flux`，`Observable`）`List`。用于流式传输的场景（例如，`text/event-stream`，`application/json+stream`）， `SseEmitter`和`ResponseBodyEmitter`被用来代替，其中`ServletOutputStream` 阻塞I / O是在一个Spring MVC管理线程执行和被施加针对每个写入的完成背压。请参阅[异步请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)和[响应类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types)。 |
| 任何其他返回值                                               | 如果返回值不是由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定的简单类型，则该返回值与该表中的任何较早值都不匹配且为a `String`或被`void`视为视图名称（通过`RequestToViewNameTranslator`应用默认视图名称选择 ），前提是该返回值不是简单类型 。简单类型的值仍然无法解析。 |

##### 类型转换

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-typeconversion)

表示一些注解的控制器方法的参数`String`为基础的请求输入（如 `@RequestParam`，`@RequestHeader`，`@PathVariable`，`@MatrixVariable`，和`@CookieValue`）可以要求类型转换如果参数被声明为比其它的东西`String`。

在这种情况下，将根据配置的转换器自动应用类型转换。默认情况下，简单的类型（`int`，`long`，`Date`，和其他人）的支持。您可以通过自定义类型转换`WebDataBinder`（见[`DataBinder`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-initbinder)），或者通过注册 `Formatters`与`FormattingConversionService`。参见[Spring字段格式化](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#format)。

##### 矩阵变量

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-matrix-variables)

[RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.3)讨论了路径段中的名称/值对。在Spring MVC中，我们根据Tim Berners-Lee 的[“旧帖子”](https://www.w3.org/DesignIssues/MatrixURIs.html)将其称为“矩阵变量” ，但它们也可以称为URI路径参数。

矩阵变量可以出现在任何路径段中，每个变量用分号分隔，多个值用逗号分隔（例如`/cars;color=red,green;year=2012`）。也可以通过重复的变量名称（例如`color=red;color=green;color=blue`）来指定多个值 。

如果期望URL包含矩阵变量，则控制器方法的请求映射必须使用URI变量来屏蔽该变量内容，并确保可以成功地匹配请求，而与矩阵变量的顺序和状态无关。以下示例使用矩阵变量：



 

```java
// GET /pets/42;q=11;r=22

@GetMapping("/pets/{petId}")
public void findPet(@PathVariable String petId, @MatrixVariable int q) {

    // petId == 42
    // q == 11
}
```

鉴于所有路径段都可能包含矩阵变量，因此有时您可能需要消除矩阵变量应位于哪个路径变量的歧义。下面的示例演示了如何做到这一点：



 

```java
// GET /owners/42;q=11/pets/21;q=22

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable(name="q", pathVar="ownerId") int q1,
        @MatrixVariable(name="q", pathVar="petId") int q2) {

    // q1 == 11
    // q2 == 22
}
```

可以将矩阵变量定义为可选变量，并指定默认值，如以下示例所示：



 

```java
// GET /pets/42

@GetMapping("/pets/{petId}")
public void findPet(@MatrixVariable(required=false, defaultValue="1") int q) {

    // q == 1
}
```

要获取所有矩阵变量，可以使用`MultiValueMap`，如以下示例所示：



 

```java
// GET /owners/42;q=11;r=12/pets/21;q=22;s=23

@GetMapping("/owners/{ownerId}/pets/{petId}")
public void findPet(
        @MatrixVariable MultiValueMap<String, String> matrixVars,
        @MatrixVariable(pathVar="petId") MultiValueMap<String, String> petMatrixVars) {

    // matrixVars: ["q" : [11,22], "r" : 12, "s" : 23]
    // petMatrixVars: ["q" : 22, "s" : 23]
}
```

请注意，您需要启用矩阵变量的使用。在MVC Java配置中，您需要通过 [路径匹配](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-path-matching)来设置`UrlPathHelper`with 。在MVC XML名称空间中，可以设置 。`removeSemicolonContent=false```

##### `@RequestParam`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestparam)

您可以使用`@RequestParam`批注将Servlet请求参数（即查询参数或表单数据）绑定到控制器中的方法参数。

以下示例显示了如何执行此操作：



 

```java
@Controller
@RequestMapping("/pets")
public class EditPetForm {

    // ...

    @GetMapping
    public String setupForm(@RequestParam("petId") int petId, Model model) { 
        Pet pet = this.clinic.loadPet(petId);
        model.addAttribute("pet", pet);
        return "petForm";
    }

    // ...

}
```

|      | 使用`@RequestParam`绑定`petId`。 |
| ---- | -------------------------------- |
|      |                                  |

默认情况下，使用此批注的方法参数是必需的，但是您可以通过将`@RequestParam`批注的`required`标志设置为 `false`或通过使用`java.util.Optional`包装器声明参数来指定方法参数是可选的。

如果目标方法参数类型不是，则类型转换将自动应用 `String`。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-typeconversion)。

将参数类型声明为数组或列表可允许解析同一参数名称的多个参数值。

如果将`@RequestParam`注释声明为`Map`或 `MultiValueMap`，而未在注释中指定参数名称，则将使用每个给定参数名称的请求参数值填充映射。

请注意，使用of `@RequestParam`是可选的（例如，设置其属性）。默认情况下，任何简单值类型参数（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ）且未由任何其他参数解析器解析，就如同使用注释一样`@RequestParam`。

##### `@RequestHeader`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestheader)

您可以使用`@RequestHeader`批注将请求标头绑定到控制器中的方法参数。

考虑以下带有标头的请求：

```
主机本地主机：8080
接受text / html，application / xhtml + xml，application / xml; q = 0.9
接受语言fr，en-gb; q = 0.7，en; q = 0.3
接受编码gzip，放气
接受字符集ISO-8859-1，utf-8; q = 0.7，*; q = 0.7
保持生命300
```

以下示例获取`Accept-Encoding`和`Keep-Alive`标头的值：



 

```java
@GetMapping("/demo")
public void handle(
        @RequestHeader("Accept-Encoding") String encoding, 
        @RequestHeader("Keep-Alive") long keepAlive) { 
    //...
}
```

|      | 获取`Accept-Encoding`标头的值。 |
| ---- | ------------------------------- |
|      | 获取`Keep-Alive`标头的值。      |

如果目标方法的参数类型不是`String`，则将 自动应用类型转换。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-typeconversion)。

当`@RequestHeader`注解上的使用`Map`， `MultiValueMap`或`HttpHeaders`参数，则地图被填充有所有标头值。

|      | 内置支持可用于将逗号分隔的字符串转换为数组或字符串集合或类型转换系统已知的其他类型。例如，用注释的方法参数`@RequestHeader("Accept")`可以是type `String`，也可以是 `String[]`or `List`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### `@CookieValue`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-cookievalue)

您可以使用`@CookieValue`注释将HTTP cookie的值绑定到控制器中的方法参数。

考虑带有以下cookie的请求：

```
JSESSIONID = 415A4AC178C59DACE0B2C9CA727CDD84
```

以下示例显示如何获取cookie值：



 

```java
@GetMapping("/demo")
public void handle(@CookieValue("JSESSIONID") String cookie) { 
    //...
}
```

|      | 获取`JSESSIONID`cookie 的值。 |
| ---- | ----------------------------- |
|      |                               |

如果目标方法的参数类型不是`String`，则类型转换将自动应用。请参阅[类型转换](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-typeconversion)。

##### `@ModelAttribute`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-modelattrib-method-args)

您可以`@ModelAttribute`在方法参数上使用注释，以从模型访问属性，或将其实例化（如果不存在）。model属性还覆盖了名称与字段名称匹配的HTTP Servlet请求参数中的值。这被称为数据绑定，它使您不必处理解析和转换单个查询参数和表单字段的工作。以下示例显示了如何执行此操作：



 

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute Pet pet) { } 
```

|      | 绑定的实例`Pet`。 |
| ---- | ----------------- |
|      |                   |

`Pet`上面的实例解析如下：

- 从模型（如果已使用[Model](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-modelattrib-methods)添加）。
- 通过使用HTTP会话[`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-sessionattributes)。
- 从URI路径变量通过传递`Converter`（请参见下一个示例）。
- 从默认构造函数的调用开始。
- 通过调用具有与Servlet请求参数匹配的参数的“主要构造函数”。参数名称是通过JavaBeans `@ConstructorProperties`或字节码中运行时保留的参数名称确定的。

虽然通常使用[模型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-modelattrib-methods)来用属性填充模型，但是另一种替代方法是依赖于`Converter`URI路径变量约定的组合。在以下示例中，模型属性名称 `account`匹配URI路径变量`account`，并`Account`通过将`String`帐号传递给注册的来加载`Converter`：



 

```java
@PutMapping("/accounts/{account}")
public String save(@ModelAttribute("account") Account account) {
    // ...
}
```

获取模型属性实例后，将应用数据绑定。所述 `WebDataBinder`类servlet请求参数名（查询参数和表单字段）以在目标字段名称匹配`Object`。必要时在应用类型转换后填充匹配字段。有关数据绑定（和验证）的更多信息，请参见 [验证](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation)。有关自定义数据绑定的更多信息，请参见 [`DataBinder`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-initbinder)。

数据绑定可能会导致错误。默认情况下，`BindException`引发a。但是，要检查controller方法中的此类错误，可以在`BindingResult`旁边紧紧添加一个参数`@ModelAttribute`，如以下示例所示：



 

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

|      | 在`BindingResult`旁边添加一个`@ModelAttribute`。 |
| ---- | ------------------------------------------------ |
|      |                                                  |

在某些情况下，您可能希望访问没有数据绑定的模型属性。对于这种情况，您可以将注入`Model`到控制器中并直接访问它，或者设置`@ModelAttribute(binding=false)`，如以下示例所示：



 

```java
@ModelAttribute
public AccountForm setUpForm() {
    return new AccountForm();
}

@ModelAttribute
public Account findAccount(@PathVariable String accountId) {
    return accountRepository.findOne(accountId);
}

@PostMapping("update")
public String update(@Valid AccountForm form, BindingResult result,
        @ModelAttribute(binding=false) Account account) { 
    // ...
}
```

|      | 设置`@ModelAttribute(binding=false)`。 |
| ---- | -------------------------------------- |
|      |                                        |

您可以通过添加`javax.validation.Valid`注释或Spring的`@Validated`注释（ [Bean验证](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation-beanvalidation)和 [Spring验证](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation)）在数据绑定之后自动应用验证 。以下示例显示了如何执行此操作：



 

```java
@PostMapping("/owners/{ownerId}/pets/{petId}/edit")
public String processSubmit(@Valid @ModelAttribute("pet") Pet pet, BindingResult result) { 
    if (result.hasErrors()) {
        return "petForm";
    }
    // ...
}
```

|      | 验证`Pet`实例。 |
| ---- | --------------- |
|      |                 |

请注意，using `@ModelAttribute`是可选的（例如，设置其属性）。默认情况下，任何不是简单值类型（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ）且未被其他任何参数解析器解析的参数都将被视为带有`@ModelAttribute`。

##### `@SessionAttributes`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-sessionattributes)

`@SessionAttributes`用于在请求之间的HTTP Servlet会话中存储模型属性。它是类型级别的注释，用于声明特定控制器使用的会话属性。这通常列出应透明地存储在会话中以供后续访问请求的模型属性的名称或模型属性的类型。

以下示例使用`@SessionAttributes`注释：



 

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {
    // ...
}
```

|      | 使用`@SessionAttributes`注释。 |
| ---- | ------------------------------ |
|      |                                |

在第一个请求上，将名称为的模型属性`pet`添加到模型时，该属性会自动升级到HTTP Servlet会话并保存在该会话中。它会一直保留在那里，直到另一个控制器方法使用`SessionStatus`方法参数来清除存储，如以下示例所示：



 

```java
@Controller
@SessionAttributes("pet") 
public class EditPetForm {

    // ...

    @PostMapping("/pets/{id}")
    public String handle(Pet pet, BindingResult errors, SessionStatus status) {
        if (errors.hasErrors) {
            // ...
        }
            status.setComplete(); 
            // ...
        }
    }
}
```

|      | 将`Pet`值存储在Servlet会话中。 |
| ---- | ------------------------------ |
|      | `Pet`从Servlet会话中清除值。   |

##### `@SessionAttribute`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-sessionattribute)

如果您需要访问全局存在（即在控制器外部（例如，通过过滤器）管理）并且可能存在或可能不存在的预先存在的会话属性，则可以`@SessionAttribute`在方法参数上使用注释，如下所示示例显示：



 

```java
@RequestMapping("/")
public String handle(@SessionAttribute User user) { 
    // ...
}
```

|      | 使用`@SessionAttribute`注释。 |
| ---- | ----------------------------- |
|      |                               |

对于需要添加或删除会话属性的用例，请考虑将其注入 `org.springframework.web.context.request.WebRequest`或 `javax.servlet.http.HttpSession`注入到控制器方法中。

要将模型属性临时存储在会话中作为控制器工作流程的一部分，请考虑使用`@SessionAttributes`中所述 [`@SessionAttributes`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-sessionattributes)。

##### `@RequestAttribute`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestattrib)

与相似`@SessionAttribute`，您可以使用`@RequestAttribute`批注访问之前创建的预先存在的请求属性（例如，通过Servlet `Filter` 或`HandlerInterceptor`）：



 

```java
@GetMapping("/")
public String handle(@RequestAttribute Client client) { 
    // ...
}
```

|      | 使用`@RequestAttribute`注释。 |
| ---- | ----------------------------- |
|      |                               |

##### 重定向属性

默认情况下，所有模型属性都被视为在重定向URL中作为URI模板变量公开。在其余属性中，那些属于原始类型或原始类型的集合或数组的属性会自动附加为查询参数。

如果专门为重定向准备了模型实例，则将原始类型属性作为查询参数附加可能是理想的结果。但是，在带注释的控制器中，模型可以包含为渲染目的添加的其他属性（例如，下拉字段值）。为避免此类属性出现在URL中的可能性，`@RequestMapping`方法可以声明类型的参数，`RedirectAttributes`并使用它来指定要提供给的确切属性`RedirectView`。如果该方法确实重定向，`RedirectAttributes`则使用的内容。否则，将使用模型的内容。

在`RequestMappingHandlerAdapter`提供了一个名为标志 `ignoreDefaultModelOnRedirect`，你可以用它来表示默认的内容 `Model`不应该，如果一个控制器方法重定向使用。相反，控制器方法应该声明一个类型的属性，`RedirectAttributes`或者如果没有声明，则不应将任何属性传递给`RedirectView`。MVC命名空间和MVC Java配置都将此标志设置为`false`，以保持向后兼容性。但是，对于新应用程序，我们建议将其设置为`true`。

请注意，展开重定向URL时，本请求中的URI模板变量将自动变为可用，并且您无需通过`Model`或显式添加它们`RedirectAttributes`。以下示例显示了如何定义重定向：



 

```java
@PostMapping("/files/{path}")
public String upload(...) {
    // ...
    return "redirect:files/{path}";
}
```

将数据传递到重定向目标的另一种方法是使用闪存属性。与其他重定向属性不同，闪存属性保存在HTTP会话中（因此不会出现在URL中）。有关更多信息，请参见[Flash属性](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-flash-attributes)。

##### Flash属性

Flash属性为一个请求提供了一种存储要在另一个请求中使用的属性的方式。重定向时最常需要此方法，例如Post-Redirect-Get模式。Flash属性在重定向之前（通常在会话中）被临时保存，以便在重定向之后可供请求使用，并立即被删除。

Spring MVC有两个主要的抽象来支持Flash属性。`FlashMap`用于保存Flash属性，而`FlashMapManager`用于存储，检索和管理 `FlashMap`实例。

Flash属性支持始终处于“打开”状态，无需显式启用。但是，如果不使用它，则永远不会导致HTTP会话创建。在每个请求上，都有一个“输入” `FlashMap`，该属性具有从前一个请求（如果有）传递过来的属性，而“输出”则`FlashMap`具有为后续请求保存的属性。`FlashMap` 可以通过Spring中的静态方法从Spring MVC中的任何位置访问这两个实例 `RequestContextUtils`。

带注释的控制器通常不需要`FlashMap`直接使用。而是， `@RequestMapping`方法可以接受类型的参数，`RedirectAttributes`并使用它为重定向方案添加Flash属性。通过添加的Flash属性将 `RedirectAttributes`自动传播到“输出” FlashMap。同样，重定向后，来自“输入”的属性`FlashMap`会自动添加到 `Model`服务于目标URL的控制器的。

将请求与Flash属性匹配

Flash属性的概念存在于许多其他Web框架中，并已证明有时会遇到并发问题。这是因为根据定义，闪存属性将存储到下一个请求。但是，“下一个”请求可能不是预期的接收者，而是另一个异步请求（例如，轮询或资源请求），在这种情况下，过早删除了闪存属性。

为了减少此类问题的可能性，请使用目标重定向URL的路径和查询参数`RedirectView`自动“标记” `FlashMap`实例。反过来，默认值`FlashMapManager`在查找“输入”时会将信息与传入请求匹配`FlashMap`。

这不能完全消除并发问题的可能性，但是可以通过重定向URL中已经可用的信息大大减少并发问题。因此，我们建议您主要将Flash属性用于重定向方案。

##### 多部分

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-multipart-forms)

一个后`MultipartResolver`已经[启用](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-multipart)，POST的内容与要求`multipart/form-data`进行解析，并定期请求参数进行访问。以下示例访问一个常规表单字段和一个上载文件：



 

```java
@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(@RequestParam("name") String name,
            @RequestParam("file") MultipartFile file) {

        if (!file.isEmpty()) {
            byte[] bytes = file.getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

将参数类型声明为a `List`可以解析相同参数名称的多个文件。

如果将`@RequestParam`注释声明为`Map`或 `MultiValueMap`，而未在注释中指定参数名称，则将使用每个给定参数名称的多部分文件来填充映射。

|      | 使用Servlet 3.0多部分解析时，您还可以声明`javax.servlet.http.Part` Spring而不是Spring `MultipartFile`作为方法参数或集合值类型。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您也可以将多部分内容用作绑定到[命令对象](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-modelattrib-method-args)的数据的一部分 。例如，前面示例中的表单字段和文件可以是表单对象上的字段，如以下示例所示：



 

```java
class MyForm {

    private String name;

    private MultipartFile file;

    // ...
}

@Controller
public class FileUploadController {

    @PostMapping("/form")
    public String handleFormUpload(MyForm form, BindingResult errors) {
        if (!form.getFile().isEmpty()) {
            byte[] bytes = form.getFile().getBytes();
            // store the bytes somewhere
            return "redirect:uploadSuccess";
        }
        return "redirect:uploadFailure";
    }
}
```

在RESTful服务方案中，也可以从非浏览器客户端提交多部分请求。以下示例显示了带有JSON的文件：

```
POST / someUrl
内容类型：多部分/混合

--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
内容处置：表格数据；name =“元数据”
内容类型：application / json; 字符集= UTF-8
内容传输编码：8bit

{
    “ name”：“值”
}
--edt7Tfrdusa7r3lNQc79vXuhIIMlatb7PQg7Vp
内容处置：表格数据；name =“文件数据”; filename =“ file.properties”
内容类型：text / xml
内容传输编码：8bit
...文件数据...
```

您可以通过访问“元数据”部分`@RequestParam`的`String`，但你可能会想从JSON反序列化（类似`@RequestBody`）。在使用[HttpMessageConverter](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#rest-message-conversion)`@RequestPart`将注释转换为多部分后，可使用 注释来访问它 ：



 

```java
@PostMapping("/")
public String handle(@RequestPart("meta-data") MetaData metadata,
        @RequestPart("file-data") MultipartFile file) {
    // ...
}
```

您可以将其`@RequestPart`与`javax.validation.Valid`Spring的`@Validated`注释结合使用或一起使用 ，这两种注释都会导致应用标准Bean验证。默认情况下，验证错误会导致`MethodArgumentNotValidException`，并变成400（BAD_REQUEST）响应。或者，您可以通过`Errors`或`BindingResult`参数在控制器内本地处理验证错误，如以下示例所示：



 

```java
@PostMapping("/")
public String handle(@Valid @RequestPart("meta-data") MetaData metadata,
        BindingResult result) {
    // ...
}
```

##### `@RequestBody`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-requestbody)

您可以使用`@RequestBody`注释有请求体读取和反序列化到一个 `Object`通过[`HttpMessageConverter`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#rest-message-conversion)。下面的示例使用一个`@RequestBody`参数：



 

```java
@PostMapping("/accounts")
public void handle(@RequestBody Account account) {
    // ...
}
```

您可以使用[MVC Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)的“ [消息转换器”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-message-converters)选项来配置或自定义消息转换。

可以`@RequestBody`与`javax.validation.Valid`或Spring的 `@Validated`注释结合使用，这两种注释都会导致应用标准Bean验证。默认情况下，验证错误会导致`MethodArgumentNotValidException`，并变成400（BAD_REQUEST）响应。或者，您可以通过`Errors`或`BindingResult`参数在控制器内本地处理验证错误，如以下示例所示：



 

```java
@PostMapping("/accounts")
public void handle(@Valid @RequestBody Account account, BindingResult result) {
    // ...
}
```

##### HttpEntity

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-httpentity)

`HttpEntity`与使用大致相同，[`@RequestBody`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestbody)但基于公开请求标头和正文的容器对象。以下清单显示了一个示例：



 

```java
@PostMapping("/accounts")
public void handle(HttpEntity<Account> entity) {
    // ...
}
```

##### `@ResponseBody`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-responsebody)

您可以`@ResponseBody`在方法上使用批注，以通过[HttpMessageConverter](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#rest-message-conversion)将返回序列化为响应主体 。以下清单显示了一个示例：



 

```java
@GetMapping("/accounts/{id}")
@ResponseBody
public Account handle() {
    // ...
}
```

`@ResponseBody`在类级别也受支持，在这种情况下，它由所有控制器方法继承。这就是的效果`@RestController`，无非就是带有`@Controller`和标记的元注释`@ResponseBody`。

您可以使用`@ResponseBody`反应类型。有关更多详细信息，[请](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)参见[异步请求](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async)和[响应类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types)。

您可以使用[MVC Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)的“ [消息转换器”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-message-converters)选项来配置或自定义消息转换。

您可以将`@ResponseBody`方法与JSON序列化视图结合使用。有关详细信息，请参见[Jackson JSON](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-jackson)。

##### 响应实体

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-responseentity)

`ResponseEntity`就像[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-responsebody)但带有状态和标题。例如：



 

```java
@GetMapping("/something")
public ResponseEntity<String> handle() {
    String body = ... ;
    String etag = ... ;
    return ResponseEntity.ok().eTag(etag).build(body);
}
```

Spring MVC支持使用单值[反应类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types) 来`ResponseEntity`为主体生成异步和/或单值和多值反应类型。

##### 杰克逊JSON

Spring提供了对Jackson JSON库的支持。

###### JSON视图

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-jsonview)

Spring MVC为[Jackson的序列化视图](https://www.baeldung.com/jackson-json-view-annotation)提供了内置支持 ，该[视图](https://www.baeldung.com/jackson-json-view-annotation)仅可呈现。中所有字段的子集`Object`。要将其与 `@ResponseBody`或`ResponseEntity`控制器方法一起使用，可以使用Jackson的 `@JsonView`注释来激活序列化视图类，如以下示例所示：



 

```java
@RestController
public class UserController {

    @GetMapping("/user")
    @JsonView(User.WithoutPasswordView.class)
    public User getUser() {
        return new User("eric", "7!jd#h23");
    }
}

public class User {

    public interface WithoutPasswordView {};
    public interface WithPasswordView extends WithoutPasswordView {};

    private String username;
    private String password;

    public User() {
    }

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @JsonView(WithoutPasswordView.class)
    public String getUsername() {
        return this.username;
    }

    @JsonView(WithPasswordView.class)
    public String getPassword() {
        return this.password;
    }
}
```

|      | `@JsonView`允许一组视图类，但是每个控制器方法只能指定一个。如果需要激活多个视图，则可以使用复合界面。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

对于依赖视图分辨率的控制器，可以将序列化视图类添加到模型中，如以下示例所示：



 

```java
@Controller
public class UserController extends AbstractController {

    @GetMapping("/user")
    public String getUser(Model model) {
        model.addAttribute("user", new User("eric", "7!jd#h23"));
        model.addAttribute(JsonView.class.getName(), User.WithoutPasswordView.class);
        return "userView";
    }
}
```

#### 1.3.4。模型

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-modelattrib-methods)

您可以使用`@ModelAttribute`注释：

- 在[方法](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-modelattrib-method-args)中的[方法参数](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-modelattrib-method-args)`@RequestMapping`上创建或访问`Object`模型的，并通过将该绑定绑定到请求 `WebDataBinder`。
- 作为`@Controller`或方法`@ControllerAdvice`类中的方法级注释，可在任何`@RequestMapping`方法调用之前帮助初始化模型。
- `@RequestMapping`标记其返回值的方法是模型属性。

本节讨论`@ModelAttribute`方法-前面列表中的第二项。控制器可以有多种`@ModelAttribute`方法。`@RequestMapping`在同一个控制器中，所有此类方法均在方法之前被调用。一个`@ModelAttribute` 也可以通过跨控制器共享方法`@ControllerAdvice`。有关更多详细信息，请参见“ [控制器建议](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-controller-advice) ”部分 。

`@ModelAttribute`方法具有灵活的方法签名。它们支持许多与`@RequestMapping`方法相同的参数，除了`@ModelAttribute`自身或与请求主体相关的任何东西。

以下示例显示了一种`@ModelAttribute`方法：



 

```java
@ModelAttribute
public void populateModel(@RequestParam String number, Model model) {
    model.addAttribute(accountRepository.findAccount(number));
    // add more ...
}
```

以下示例仅添加一个属性：



 

```java
@ModelAttribute
public Account addAccount(@RequestParam String number) {
    return accountRepository.findAccount(number);
}
```

|      | 如果未明确指定名称，则根据`Object` 类型选择默认名称，如javadoc中针对的解释[`Conventions`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/Conventions.html)。您始终可以使用重载`addAttribute`方法或通过`name`on 的属性`@ModelAttribute`（用于返回值）来分配显式名称。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以`@ModelAttribute`在方法上用作方法级别的注释`@RequestMapping`，在这种情况下，方法的返回值将`@RequestMapping`解释为模型属性。通常不需要这样做，因为这是HTML控制器的默认行为，除非返回值是a `String`，否则它将被解释为视图名称。 `@ModelAttribute`还可以自定义模型属性名称，如以下示例所示：



 

```java
@GetMapping("/accounts/{id}")
@ModelAttribute("myAccount")
public Account handle() {
    // ...
    return account;
}
```

#### 1.3.5。 `DataBinder`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-initbinder)

`@Controller`或`@ControllerAdvice`类可以具有`@InitBinder`初始化的实例的方法`WebDataBinder`，而这些实例又可以：

- 将请求参数（即表单或查询数据）绑定到模型对象。
- 将基于字符串的请求值（例如请求参数，路径变量，标头，Cookie等）转换为控制器方法参数的目标类型。
- `String`呈现HTML表单时，将模型对象值格式化为值。

`@InitBinder`方法可以注册控制器特异性`java.bean.PropertyEditor`或Spring `Converter`和`Formatter`组件。此外，您可以使用 [MVC配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-conversion) 在全局共享中注册`Converter`和`Formatter`键入`FormattingConversionService`。

`@InitBinder``@RequestMapping`除了`@ModelAttribute`（命令对象）参数外，方法还支持许多与方法相同的参数。通常，它们使用一个`WebDataBinder`参数（用于注册）和一个`void`返回值进行声明。以下清单显示了一个示例：



 

```java
@Controller
public class FormController {

    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }

    // ...
}
```

|      | 定义`@InitBinder`方法。 |
| ---- | ----------------------- |
|      |                         |

或者，当您`Formatter`通过shared 使用基于设置时 `FormattingConversionService`，可以重新使用相同的方法并注册特定于控制器的`Formatter`实现，如以下示例所示：



 

```java
@Controller
public class FormController {

    @InitBinder 
    protected void initBinder(WebDataBinder binder) {
        binder.addCustomFormatter(new DateFormatter("yyyy-MM-dd"));
    }

    // ...
}
```

|      | `@InitBinder`在自定义格式化程序上定义方法。 |
| ---- | ------------------------------------------- |
|      |                                             |

#### 1.3.6。例外情况

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-controller-exceptions)

`@Controller`和[@ControllerAdvice](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-controller-advice)类可以具有 `@ExceptionHandler`处理控制器方法异常的方法，如以下示例所示：



 

```java
@Controller
public class SimpleController {

    // ...

    @ExceptionHandler
    public ResponseEntity<String> handle(IOException ex) {
        // ...
    }
}
```

该异常可以与正在传播的顶级异常（即，直接`IOException`引发的异常）匹配，也可以与顶级包装器异常（例如，`IOException`包装在内`IllegalStateException`）的直接原因匹配 。

对于匹配的异常类型，最好将目标异常声明为方法参数，如前面的示例所示。当多个异常方法匹配时，根源异常匹配通常比原因异常匹配更可取。更具体地，`ExceptionDepthComparator`用于根据异常从引发的异常类型的深度对异常进行排序。

另外，注释声明可以缩小异常类型以使其匹配，如以下示例所示：



 

```java
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(IOException ex) {
    // ...
}
```

您甚至可以使用带有非常通用的参数签名的特定异常类型的列表，如以下示例所示：



 

```java
@ExceptionHandler({FileSystemException.class, RemoteException.class})
public ResponseEntity<String> handle(Exception ex) {
    // ...
}
```

|      | 根匹配和原因异常匹配之间的区别可能令人惊讶。在`IOException`前面显示的变体中，通常以实际`FileSystemException`或`RemoteException`实例作为参数来调用该方法，因为这两个方法都从扩展`IOException`。但是，如果任何此类匹配异常都在本身为的包装异常中传播`IOException`，则传入的异常实例就是该包装异常。在`handle(Exception)`变体中，行为甚至更简单。在包装方案中，总是使用包装程序异常来调用此方法，`ex.getCause()`在这种情况下，将找到实际匹配的异常。仅当将它们作为顶级异常抛出时，传入的异常才是实际的`FileSystemException`或 `RemoteException`实例。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

通常，我们建议您在参数签名中尽可能具体，以减少根类型和原因异常类型之间不匹配的可能性。考虑将多重匹配方法分解为单个`@ExceptionHandler` 方法，每个方法都通过其签名匹配单个特定的异常类型。

在多种`@ControllerAdvice`安排中，我们建议声明`@ControllerAdvice`优先级并以相应顺序声明您的主根异常映射。尽管根异常匹配优先于原因，但这是在给定控制器或`@ControllerAdvice`类的方法之间定义的。这意味着优先级较高的`@ControllerAdvice`Bean 上的原因匹配优于优先级 较低的`@ControllerAdvice`Bean 上的任何匹配（例如，根） 。

最后但并非最不重要的一点是，`@ExceptionHandler`方法实现可以选择以不处理给定异常实例的方式将其以原始形式重新抛出。在仅对根级别匹配或无法静态确定的特定上下文中的匹配感兴趣的情况下，这很有用。重新抛出的异常会在其余的解决方案链中传播，就像给定的`@ExceptionHandler`方法最初不会匹配一样。

`@ExceptionHandler`Spring MVC中对方法的支持建立在[HandlerExceptionResolver](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-exceptionhandlers)机制`DispatcherServlet` 级别上。

##### 方法参数

`@ExceptionHandler` 方法支持以下参数：

| 方法参数                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 异常类型                                                     | 用于访问引发的异常。                                         |
| `HandlerMethod`                                              | 用于访问引发异常的控制器方法。                               |
| `WebRequest`， `NativeWebRequest`                            | 对请求参数以及请求和会话属性的常规访问，而无需直接使用Servlet API。 |
| `javax.servlet.ServletRequest`， `javax.servlet.ServletResponse` | 选择任何特定的请求或响应类型（例如`ServletRequest`或 `HttpServletRequest`或Spring的`MultipartRequest`或`MultipartHttpServletRequest`）。 |
| `javax.servlet.http.HttpSession`                             | 强制会话的存在。结果，这种论据永远不会`null`。 请注意，会话访问不是线程安全的。考虑将`RequestMappingHandlerAdapter`实例的`synchronizeOnSession`标志设置 为`true`是否允许多个请求同时访问会话。 |
| `java.security.Principal`                                    | 当前经过身份验证的用户-可能是特定的`Principal`实现类（如果已知）。 |
| `HttpMethod`                                                 | 请求的HTTP方法。                                             |
| `java.util.Locale`                                           | 当前请求的语言环境，由最具体的`LocaleResolver`可用语言决定，实际上是配置的`LocaleResolver`或`LocaleContextResolver`。 |
| `java.util.TimeZone`， `java.time.ZoneId`                    | 与当前请求关联的时区，由决定`LocaleContextResolver`。        |
| `java.io.OutputStream`， `java.io.Writer`                    | 用于访问原始响应主体（由Servlet API公开）。                  |
| `java.util.Map`，`org.springframework.ui.Model`，`org.springframework.ui.ModelMap` | 用于访问模型以进行错误响应。永远是空的。                     |
| `RedirectAttributes`                                         | 指定在重定向的情况下要使用的属性（要附加到查询字符串中），并指定要临时存储的属性，直到重定向后的请求为止。请参阅[重定向属性](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-redirecting-passing-data)和[Flash属性](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-flash-attributes)。 |
| `@SessionAttribute`                                          | 与访问由于类级`@SessionAttributes`声明而存储在会话中的模型属性相反，用于访问任何会话属性。请参阅[`@SessionAttribute`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-sessionattribute)以获取更多详细信息。 |
| `@RequestAttribute`                                          | 用于访问请求属性。请参阅[`@RequestAttribute`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestattrib)以获取更多详细信息。 |

##### 返回值

`@ExceptionHandler` 方法支持以下返回值：

| 返回值                                           | 描述                                                         |
| :----------------------------------------------- | :----------------------------------------------------------- |
| `@ResponseBody`                                  | 返回值通过`HttpMessageConverter`实例转换并写入响应。请参阅[`@ResponseBody`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-responsebody)。 |
| `HttpEntity`， `ResponseEntity`                  | 返回值指定完整的响应（包括HTTP标头和正文）将通过`HttpMessageConverter`实例转换并写入响应。参见[ResponseEntity](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-responseentity)。 |
| `String`                                         | 一个视图名称，将通过`ViewResolver`实现来解析，并与隐式模型一起使用-通过命令对象和`@ModelAttribute`方法确定。该处理程序方法还可以通过声明一个自`Model` 变量（如前所述）以编程方式丰富模型。 |
| `View`                                           | 甲`View`实例以使用用于与所述隐式模型一起渲染-通过命令对象和确定`@ModelAttribute`方法。该处理程序方法还可以通过声明一个自`Model`变量（如前所述）以编程方式丰富模型。 |
| `java.util.Map`， `org.springframework.ui.Model` | 要添加到隐式模型的属性，其视图名称是通过隐式确定的`RequestToViewNameTranslator`。 |
| `@ModelAttribute`                                | 要添加到模型的属性，其视图名称通过隐式确定`RequestToViewNameTranslator`。请注意，这`@ModelAttribute`是可选的。请参见表末尾的“其他任何返回值”。 |
| `ModelAndView` 宾语                              | 要使用的视图和模型属性，以及响应状态（可选）。               |
| `void`                                           | 用的方法`void`返回类型（或`null`返回值）被认为已经完全处理的响应，如果它也具有`ServletResponse`一个`OutputStream`参数，或`@ResponseStatus`注释。如果控制器进行了肯定检查`ETag`或`lastModified`时间戳检查，则情况也是如此 （有关详细信息，请参阅[控制器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-caching-etag-lastmodified)）。如果以上条件都不成立，则`void`返回类型还可以为REST控制器指示“无响应正文”，或者为HTML控制器指示默认视图名称选择。 |
| 任何其他返回值                                   | 如果返回值与上述任何一个都不匹配并且不是简单类型（由[BeanUtils＃isSimpleProperty](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/beans/BeanUtils.html#isSimpleProperty-java.lang.Class-)确定 ），则默认情况下会将其视为要添加到模型的模型属性。如果它是简单类型，则仍无法解决。 |

##### REST API例外

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-rest-exceptions)

REST服务的常见要求是在响应正文中包含错误详细信息。Spring框架不会自动执行此操作，因为响应主体中错误详细信息的表示是特定于应用程序的。但是，a `@RestController`可以使用`@ExceptionHandler`带有`ResponseEntity`返回值的方法来设置响应的状态和主体。也可以在`@ControllerAdvice`类中声明此类方法以将其全局应用。

在响应主体中实现具有错误详细信息的全局异常处理的应用程序应考虑extend [`ResponseEntityExceptionHandler`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/ResponseEntityExceptionHandler.html)，它提供对Spring MVC引发的异常的处理，并提供用于自定义响应主体的钩子。要使用此功能，请创建的子类 `ResponseEntityExceptionHandler`，用进行注释`@ControllerAdvice`，覆盖必要的方法，然后将其声明为Spring bean。

#### 1.3.7。控制器建议

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-ann-controller-advice)

通常`@ExceptionHandler`，，`@InitBinder`和`@ModelAttribute`方法适用于`@Controller`声明它们的类（或类层次结构）。如果要使此类方法更全局地应用（跨控制器），则可以在带有`@ControllerAdvice`或注释的类中声明它们`@RestControllerAdvice`。

`@ControllerAdvice`带有注释`@Component`，这意味着可以通过[组件扫描](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#beans-java-instantiating-container-scan)将此类注册为Spring Bean 。`@RestControllerAdvice`是用`@ControllerAdvice`和注释的组合注释`@ResponseBody`，从本质上讲意味着 `@ExceptionHandler`方法是通过消息转换（与视图分辨率或模板渲染相比）呈现给响应主体的。

在启动时，用于`@RequestMapping`和`@ExceptionHandler` 方法的基础结构类检测使用注释的Spring bean `@ControllerAdvice`，然后在运行时应用其方法。全局`@ExceptionHandler`方法（来自`@ControllerAdvice`）适用*于*局部方法（来自`@Controller`）。相反，全局`@ModelAttribute` 和`@InitBinder`方法*在*局部方法*之前*应用。

默认情况下，`@ControllerAdvice`方法适用于每个请求（即所有控制器），但是您可以通过使用批注上的属性将其范围缩小到控制器的子集，如以下示例所示：



 

```java
// Target all Controllers annotated with @RestController
@ControllerAdvice(annotations = RestController.class)
public class ExampleAdvice1 {}

// Target all Controllers within specific packages
@ControllerAdvice("org.example.controllers")
public class ExampleAdvice2 {}

// Target all Controllers assignable to specific classes
@ControllerAdvice(assignableTypes = {ControllerInterface.class, AbstractController.class})
public class ExampleAdvice3 {}
```

前面示例中的选择器在运行时进行评估，如果广泛使用，可能会对性能产生负面影响。有关[`@ControllerAdvice`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html) 更多详细信息，请参见 javadoc。

### 1.4。功能端点

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-fn)

Spring Web MVC包括WebMvc.fn，这是一个轻量级的函数编程模型，其中的函数用于路由和处理请求，而契约则是为不变性而设计的。它是基于注释的编程模型的替代方案，但可以在同一[DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet)上运行。

#### 1.4.1。总览

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-fn-overview)

在WebMvc.fn中，HTTP请求使用来处理`HandlerFunction`：一个接受`ServerRequest`并返回的函数 `ServerResponse`。请求和响应对象都具有不可变的协定，这些协定为JDK 8提供了对HTTP请求和响应的友好访问。 与基于注释的编程模型中方法`HandlerFunction`的主体等效`@RequestMapping`。

传入的请求使用：路由到处理函数，`RouterFunction`该函数采用`ServerRequest`并返回可选的`HandlerFunction`（即`Optional`）。当路由器功能匹配时，返回处理程序功能。否则为空的Optional。 `RouterFunction`与`@RequestMapping`注解等效，但主要区别在于路由器功能不仅提供数据，还提供行为。

`RouterFunctions.route()` 提供了一个有助于构建路由器的路由器构建器，如以下示例所示：



 

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.servlet.function.RequestPredicates.*;
import static org.springframework.web.servlet.function.RouterFunctions.route;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> route = route()
    .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson)
    .GET("/person", accept(APPLICATION_JSON), handler::listPeople)
    .POST("/person", handler::createPerson)
    .build();


public class PersonHandler {

    // ...

    public ServerResponse listPeople(ServerRequest request) {
        // ...
    }

    public ServerResponse createPerson(ServerRequest request) {
        // ...
    }

    public ServerResponse getPerson(ServerRequest request) {
        // ...
    }
}
```

如果您将其注册`RouterFunction`为Bean（例如，通过将其暴露在@Configuration类中），则servlet将自动检测到它，如[运行服务器中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#webmvc-fn-running)。

#### 1.4.2。处理函数

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-fn-handler-functions)

`ServerRequest`并且`ServerResponse`是不可变的接口，可提供JDK 8友好的HTTP请求和响应访问，包括标头，正文，方法和状态代码。

##### `ServerRequest`

`ServerRequest`提供对HTTP方法，URI，标头和查询参数的访问，而通过这些`body`方法提供对正文的访问。

以下示例将请求正文提取到`String`：



 

```java
String string = request.body(String.class);
```

以下示例将主体提取到`List`，其中`Person`对象从序列化形式（例如JSON或XML）解码：



 

```java
List<Person> people = request.body(new ParameterizedTypeReference<List<Person>>() {});
```

以下示例显示如何访问参数：



 

```java
MultiValueMap<String, String> params = request.params();
```

##### `ServerResponse`

`ServerResponse`提供对HTTP响应的访问，并且由于它是不可变的，因此可以使用一种`build`方法来创建它。您可以使用构建器来设置响应状态，添加响应标题或提供正文。以下示例使用JSON内容创建200（确定）响应：



 

```java
Person person = ...
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON).body(person);
```

下面的示例演示如何构建`Location`不带标头的201（已创建）响应：



 

```java
URI location = ...
ServerResponse.created(location).build();
```

##### 处理程序类

我们可以将处理程序函数编写为lambda，如以下示例所示：



 

```java
HandlerFunction<ServerResponse> helloWorld =
  request -> ServerResponse.ok().body("Hello World");
```

这很方便，但是在应用程序中我们需要多个功能，并且多个内联lambda可能会变得凌乱。因此，将相关的处理程序功能分组到一个处理程序类中很有用，该类具有与`@Controller`基于注释的应用程序相似的作用。例如，以下类公开了反应式`Person`存储库：



 

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.reactive.function.server.ServerResponse.ok;

public class PersonHandler {

    private final PersonRepository repository;

    public PersonHandler(PersonRepository repository) {
        this.repository = repository;
    }

    public ServerResponse listPeople(ServerRequest request) { 
        List<Person> people = repository.allPeople();
        return ok().contentType(APPLICATION_JSON).body(people);
    }

    public ServerResponse createPerson(ServerRequest request) throws Exception { 
        Person person = request.body(Person.class);
        repository.savePerson(person);
        return ok().build();
    }

    public ServerResponse getPerson(ServerRequest request) { 
        int personId = Integer.parseInt(request.pathVariable("id"));
        Person person = repository.getPerson(personId);
        if (person != null) {
            return ok().contentType(APPLICATION_JSON).body(person))
        }
        else {
            return ServerResponse.notFound().build();
        }
    }

}
```

|      | `listPeople`是一个处理程序函数，`Person`以JSON格式返回存储库中找到的所有对象。 |
| ---- | ------------------------------------------------------------ |
|      | `createPerson`是一个处理程序函数，用于存储`Person`请求正文中包含的新内容。 |
|      | `getPerson`是一个处理程序函数，该函数返回由`id`path变量标识的一个人。我们`Person`从存储库中检索到该内容，并创建一个JSON响应（如果找到）。如果找不到，我们将返回404 Not Found响应。 |

##### 验证方式

功能端点可以使用Spring的[验证工具](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation)将验证应用于请求主体。例如，给定一个定制的Spring [Validator](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation)实现`Person`：



 

```java
public class PersonHandler {

    private final Validator validator = new PersonValidator(); 

    // ...

    public ServerResponse createPerson(ServerRequest request) {
        Person person = request.body(Person.class);
        validate(person); 
        repository.savePerson(person);
        return ok().build();
    }

    private void validate(Person person) {
        Errors errors = new BeanPropertyBindingResult(person, "person");
        validator.validate(person, errors);
        if (errors.hasErrors()) {
            throw new ServerWebInputException(errors.toString()); 
        }
    }
}
```

|      | 创建`Validator`实例。 |
| ---- | --------------------- |
|      | 应用验证。            |
|      | 引发400响应的异常。   |

处理程序还可以通过创建和注入`Validator`基于的全局实例来使用标准bean验证API（JSR-303）`LocalValidatorFactoryBean`。请参阅[春季验证](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation-beanvalidation)。

#### 1.4.3。 `RouterFunction`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-fn-router-functions)

路由器功能用于将请求路由到相应的`HandlerFunction`。通常，您不是自己编写路由器功能，而是在`RouterFunctions`实用工具类上使用一种方法 来创建一个。 `RouterFunctions.route()`（无参数）为您提供了流畅的生成器来创建路由器功能，而`RouterFunctions.route(RequestPredicate, HandlerFunction)`提供了直接的方式来创建路由器。

通常，建议使用该`route()`构建器，因为它为典型的映射方案提供了便捷的捷径，而无需发现静态导入。例如，路由器功能构建器提供了`GET(String, HandlerFunction)`为GET请求创建映射的方法。和`POST(String, HandlerFunction)`POST。

除了基于HTTP方法的映射外，路由构建器还提供了一种在映射到请求时引入其他谓词的方法。对于每个HTTP方法，都有一个以a `RequestPredicate`作为参数的重载变体，不过可以表达其他约束。

##### 谓词

您可以编写自己的`RequestPredicate`，但是`RequestPredicates`实用程序类根据请求路径，HTTP方法，内容类型等提供常用的实现。以下示例使用请求谓词基于`Accept` 标头创建约束：



 

```java
RouterFunction<ServerResponse> route = RouterFunctions.route()
    .GET("/hello-world", accept(MediaType.TEXT_PLAIN),
        request -> ServerResponse.ok().body("Hello World")).build();
```

您可以使用以下命令组合多个请求谓词：

- `RequestPredicate.and(RequestPredicate)` -两者必须匹配。
- `RequestPredicate.or(RequestPredicate)` -两者都可以匹配。

的许多谓词`RequestPredicates`组成。例如，`RequestPredicates.GET(String)`由`RequestPredicates.method(HttpMethod)` 和组成`RequestPredicates.path(String)`。上面显示的示例还使用了两个请求谓词，因为构建器在`RequestPredicates.GET`内部使用 并将其与`accept`谓词组合在一起。

##### 路线

路由器功能按顺序评估：如果第一个路由不匹配，则评估第二个路由，依此类推。因此，在通用路由之前声明更具体的路由是有意义的。请注意，此行为不同于基于注释的编程模型，在该模型中，将自动选择“最特定”的控制器方法。

使用路由器功能构建器时，所有定义的路由都组成一个`RouterFunction`从中返回的路由 `build()`。还有其他方法可以将多个路由器功能组合在一起：

- `add(RouterFunction)`在`RouterFunctions.route()`建造者上
- `RouterFunction.and(RouterFunction)`
- `RouterFunction.andRoute(RequestPredicate, HandlerFunction)` — `RouterFunction.and()`嵌套的快捷方式 `RouterFunctions.route()`。

以下示例显示了四种路线的组成：



 

```java
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.web.servlet.function.RequestPredicates.*;

PersonRepository repository = ...
PersonHandler handler = new PersonHandler(repository);

RouterFunction<ServerResponse> otherRoute = ...

RouterFunction<ServerResponse> route = route()
    .GET("/person/{id}", accept(APPLICATION_JSON), handler::getPerson) 
    .GET("/person", accept(APPLICATION_JSON), handler::listPeople) 
    .POST("/person", handler::createPerson) 
    .add(otherRoute) 
    .build();
```

|      | `GET /person/{id}`具有`Accept`与JSON匹配的标头的路由 `PersonHandler.getPerson` |
| ---- | ------------------------------------------------------------ |
|      | `GET /person`具有`Accept`与JSON匹配的标头的路由 `PersonHandler.listPeople` |
|      | `POST /person`没有其他谓词的映射到 `PersonHandler.createPerson`，并且 |
|      | `otherRoute` 是在其他地方创建并添加到所建立路由的路由器功能。 |

##### 嵌套路线

一组路由器功能通常具有共享谓词，例如共享路径。在上面的示例中，共享谓词将是与其中`/person`三个路由使用的match匹配的路径谓词。使用注释时，您可以使用`@RequestMapping` 映射到的类型级别的注释来删除此重复项`/person`。在WebMvc.fn中，可以通过`path`路由器功能构建器上的方法共享路径谓词。例如，可以通过以下方式使用嵌套路由来改进上面示例的最后几行：



 

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", builder -> builder 
        .GET("/{id}", accept(APPLICATION_JSON), handler::getPerson)
        .GET("", accept(APPLICATION_JSON), handler::listPeople)
        .POST("/person", handler::createPerson))
    .build();
```

|      | 请注意，的第二个参数`path`是使用路由器构建器的使用者。 |
| ---- | ------------------------------------------------------ |
|      |                                                        |

尽管基于路径的嵌套是最常见的，但是您可以使用`nest`构建器上的方法来嵌套在任何种类的谓词上。上面的内容仍然包含一些以共享标头`Accept`谓词形式的重复。通过结合使用该`nest`方法，我们可以进一步改进`accept`：



 

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET("", handler::listPeople))
        .POST("/person", handler::createPerson))
    .build();
```

#### 1.4.4。运行服务器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-fn-running)

通常，您可以[`DispatcherHandler`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet)通过[MVC Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)在基于设置的路由器中运行路由器功能，该 [配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config)使用Spring配置来声明处理请求所需的组件。MVC Java配置声明以下基础结构组件以支持功能端点：

- `RouterFunctionMapping`：`RouterFunction`在Spring配置中检测一个或多个bean，将它们组合在一起`RouterFunction.andOther`，并将请求路由到生成的composition `RouterFunction`。
- `HandlerFunctionAdapter`：简单的适配器，可以`DispatcherHandler`调用`HandlerFunction`映射到请求的。

前面的组件使功能端点适合于`DispatcherServlet`请求处理生命周期，并且（如果有）声明的控制器也可以（可能）与带注释的控制器并排运行。这也是Spring Boot Web启动程序如何启用功能端点的方式。

以下示例显示了WebFlux Java配置：



 

```java
@Configuration
@EnableMvc
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public RouterFunction<?> routerFunctionA() {
        // ...
    }

    @Bean
    public RouterFunction<?> routerFunctionB() {
        // ...
    }

    // ...

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // configure message conversion...
    }

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // configure CORS...
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        // configure view resolution for HTML rendering...
    }
}
```

#### 1.4.5。过滤处理程序功能

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-fn-handler-filter-function)

您可以通过使用过滤处理器功能`before`，`after`或`filter`在路由功能生成器方法。使用注释，您可以通过使用实现类似的功能`@ControllerAdvice`，一个`ServletFilter`，或两者兼而有之。该过滤器将应用于构建器构建的所有路由。这意味着在嵌套路由中定义的过滤器不适用于“顶级”路由。例如，考虑以下示例：



 

```java
RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET("", handler::listPeople)
            .before(request -> ServerRequest.from(request) 
                .header("X-RequestHeader", "Value")
                .build()))
        .POST("/person", handler::createPerson))
    .after((request, response) -> logResponse(response)) 
    .build();
```

|      | `before`添加自定义请求标头的过滤器仅应用于两个GET路由。 |
| ---- | ------------------------------------------------------- |
|      | `after`记录响应的过滤器将应用于所有路由，包括嵌套路由。 |

在`filter`路由器上构建器方法需要`HandlerFilterFunction`：一个函数，接受`ServerRequest`和`HandlerFunction`并返回`ServerResponse`。handler函数参数代表链中的下一个元素。这通常是路由到的处理程序，但是如果应用了多个，它也可以是另一个过滤器。

现在，我们可以为路由添加一个简单的安全过滤器，假设我们有一个`SecurityManager`可以确定是否允许特定路径的。以下示例显示了如何执行此操作：



 

```java
SecurityManager securityManager = ...

RouterFunction<ServerResponse> route = route()
    .path("/person", b1 -> b1
        .nest(accept(APPLICATION_JSON), b2 -> b2
            .GET("/{id}", handler::getPerson)
            .GET("", handler::listPeople))
        .POST("/person", handler::createPerson))
    .filter((request, next) -> {
        if (securityManager.allowAccessTo(request.path())) {
            return next.handle(request);
        }
        else {
            return ServerResponse.status(UNAUTHORIZED).build();
        }
    })
    .build();
```

前面的示例演示了调用`next.handle(ServerRequest)`是可选的。当允许访问时，我们只允许执行处理函数。

除了使用`filter`路由器功能构建器上的方法之外，还可以通过将过滤器应用于现有路由器功能`RouterFunction.filter(HandlerFilterFunction)`。

|      | 通过专用CORS支持功能端点 [`CorsFilter`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/webmvc-cors.html#mvc-cors-filter)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.5。URI链接

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-uri-building)

本节描述了Spring框架中可用于URI的各种选项。

#### 1.5.1。UriComponents

Spring MVC和Spring WebFlux

`UriComponentsBuilder` 有助于从带有变量的URI模板构建URI，如以下示例所示：



 

```java
UriComponents uriComponents = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")  
        .queryParam("q", "{q}")  
        .encode() 
        .build(); 

URI uri = uriComponents.expand("Westin", "123").toUri();  
```

|      | 带有URI模板的静态工厂方法。      |
| ---- | -------------------------------- |
|      | 添加或替换URI组件。              |
|      | 请求对URI模板和URI变量进行编码。 |
|      | 建立一个`UriComponents`。        |
|      | 展开变量并获得`URI`。            |

可以将前面的示例合并为一个链，并通过进行缩短`buildAndExpand`，如以下示例所示：



 

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("Westin", "123")
        .toUri();
```

您可以通过直接转到URI（这意味着编码）来进一步缩短它，如以下示例所示：



 

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

您可以使用完整的URI模板进一步缩短它，如以下示例所示：



 

```java
URI uri = UriComponentsBuilder
        .fromUriString("https://example.com/hotels/{hotel}?q={q}")
        .build("Westin", "123");
```

#### 1.5.2。UriBuilder

Spring MVC和Spring WebFlux

[`UriComponentsBuilder`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#web-uricomponents)实施`UriBuilder`。您可以`UriBuilder`依次使用创建 一个`UriBuilderFactory`。一起使用，`UriBuilderFactory`并 `UriBuilder`提供一种可插入的机制，以基于共享配置（例如基本URL，编码首选项和其他详细信息）从URI模板构建URI。

您可以配置`RestTemplate`和`WebClient`使用`UriBuilderFactory` 自定义的URI的准备。`DefaultUriBuilderFactory`是内部`UriBuilderFactory`使用`UriComponentsBuilder`并公开共享配置选项的默认实现。

以下示例显示了如何配置`RestTemplate`：



 

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);
```

以下示例配置了`WebClient`：



 

```java
// import org.springframework.web.util.DefaultUriBuilderFactory.EncodingMode;

String baseUrl = "https://example.org";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl);
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

另外，您也可以`DefaultUriBuilderFactory`直接使用。它类似于使用， `UriComponentsBuilder`但不是静态工厂方法，它是一个包含配置和首选项的实际实例，如以下示例所示：



 

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory uriBuilderFactory = new DefaultUriBuilderFactory(baseUrl);

URI uri = uriBuilderFactory.uriString("/hotels/{hotel}")
        .queryParam("q", "{q}")
        .build("Westin", "123");
```

#### 1.5.3。URI编码

Spring MVC和Spring WebFlux

`UriComponentsBuilder` 在两个级别公开编码选项：

- [UriComponentsBuilder＃encode（）](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/util/UriComponentsBuilder.html#encode--)：首先对URI模板进行预编码，然后在扩展时严格对URI变量进行编码。
- [UriComponents＃encode（）](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/util/UriComponents.html#encode--)：扩展URI变量*后，*[对](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/util/UriComponents.html#encode--) URI组件*进行*编码。

这两个选项都使用转义的八位字节替换非ASCII和非法字符。但是，第一个选项还会替换出现在URI变量中的具有保留含义的字符。

|      | 考虑“;”，这在路径上是合法的，但具有保留的含义。第一个选项代替“;” URI变量中带有“％3B”，但URI模板中没有。相比之下，第二个选项永远不会替换“;”，因为它是路径中的合法字符。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在大多数情况下，第一个选项可能会产生预期的结果，因为它将URI变量视为要完全编码的不透明数据，而选项2仅在URI变量有意包含保留字符的情况下才有用。

以下示例使用第一个选项：



 

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .encode()
        .buildAndExpand("New York", "foo+bar")
        .toUri();

// Result is "/hotel%20list/New%20York?q=foo%2Bbar"
```

您可以通过直接转到URI（这意味着编码）来缩短前面的示例，如以下示例所示：



 

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}")
        .queryParam("q", "{q}")
        .build("New York", "foo+bar")
```

您可以使用完整的URI模板进一步缩短它，如以下示例所示：



 

```java
URI uri = UriComponentsBuilder.fromPath("/hotel list/{city}?q={q}")
        .build("New York", "foo+bar")
```

在`WebClient`与`RestTemplate`扩大和编码URI通过内部模板`UriBuilderFactory`策略。两者都可以使用自定义策略进行配置。如以下示例所示：



 

```java
String baseUrl = "https://example.com";
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory(baseUrl)
factory.setEncodingMode(EncodingMode.TEMPLATE_AND_VALUES);

// Customize the RestTemplate..
RestTemplate restTemplate = new RestTemplate();
restTemplate.setUriTemplateHandler(factory);

// Customize the WebClient..
WebClient client = WebClient.builder().uriBuilderFactory(factory).build();
```

该`DefaultUriBuilderFactory`实现在`UriComponentsBuilder`内部使用以扩展和编码URI模板。作为工厂，它提供了一个位置，可以根据以下一种编码模式来配置编码方法：

- `TEMPLATE_AND_VALUES`：使用`UriComponentsBuilder#encode()`，对应于先前列表中的第一个选项，对URI模板进行预编码，并在扩展时严格编码URI变量。
- `VALUES_ONLY`：不对URI模板进行编码，而是在将URI变量`UriUtils#encodeUriUriVariables`扩展到模板之前对其应用严格的编码。
- `URI_COMPONENT`：使用`UriComponents#encode()`，对应于先前列表中的第二个选项，在扩展URI变量*后使用* URI 组件值进行编码。
- `NONE`：未应用编码。

出于历史原因和向后兼容性，`RestTemplate`将设置`EncodingMode.URI_COMPONENT`为。在`WebClient`依赖于缺省值`DefaultUriBuilderFactory`，将其从变化`EncodingMode.URI_COMPONENT`在5.0.x的到`EncodingMode.TEMPLATE_AND_VALUES`5.1。

#### 1.5.4。相对Servlet请求

您可以`ServletUriComponentsBuilder`用来创建相对于当前请求的URI，如以下示例所示：



 

```java
HttpServletRequest request = ...

// Re-uses host, scheme, port, path and query string...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromRequest(request)
        .replaceQueryParam("accountId", "{id}").build()
        .expand("123")
        .encode();
```

您可以创建相对于上下文路径的URI，如以下示例所示：



 

```java
// Re-uses host, port and context path...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromContextPath(request)
        .path("/accounts").build()
```

您可以创建与Servlet相关的URI（例如`/main/*`），如以下示例所示：



 

```java
// Re-uses host, port, context path, and Servlet prefix...

ServletUriComponentsBuilder ucb = ServletUriComponentsBuilder.fromServletMapping(request)
        .path("/accounts").build()
```

|      | 从5.1开始，`ServletUriComponentsBuilder`忽略来自`Forwarded`和 `X-Forwarded-*`标头的信息，该标头指定了客户端起源的地址。考虑使用 [`ForwardedHeaderFilter`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#filters-forwarded-headers)来提取和使用或丢弃此类标头。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.5.5。链接到控制器

Spring MVC提供了一种准备到控制器方法的链接的机制。例如，以下MVC控制器允许创建链接：



 

```java
@Controller
@RequestMapping("/hotels/{hotel}")
public class BookingController {

    @GetMapping("/bookings/{booking}")
    public ModelAndView getBooking(@PathVariable Long booking) {
        // ...
    }
}
```

您可以通过按名称引用方法来准备链接，如以下示例所示：



 

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(BookingController.class, "getBooking", 21).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

在前面的示例中，我们提供了实际的方法参数值（在本例中为long value：），`21`用作路径变量并插入到URL中。此外，我们提供了值，`42`以填充所有剩余的URI变量，例如`hotel`从类型级请求映射继承的变量。如果该方法具有更多参数，则可以为URL不需要的参数提供null。通常，只有`@PathVariable`和`@RequestParam`参数与构造URL有关。

还有其他使用方法`MvcUriComponentsBuilder`。例如，您可以使用类似于代理的模拟测试技术来避免按名称引用控制器方法，如以下示例所示（该示例假定静态导入`MvcUriComponentsBuilder.on`）：



 

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

|      | 当控制器方法签名可用于使用进行链接创建时，其设计受到限制`fromMethodCall`。除了需要适当的参数签名外，返回类型还存在技术限制（即，为链接构建器调用生成运行时代理），因此返回类型一定不能为`final`。特别是，`String`视图名称的通用返回类型在这里不起作用。您应该改用`ModelAndView` 甚至是普通的`Object`（带有`String`返回值）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

前面的示例在中使用静态方法`MvcUriComponentsBuilder`。在内部，它们依靠`ServletUriComponentsBuilder`当前请求的方案，主机，端口，上下文路径和Servlet路径准备基本URL。在大多数情况下，此方法效果很好。但是，有时可能不足。例如，您可能不在请求的上下文之内（例如，准备链接的批处理过程），或者您可能需要插入路径前缀（例如，从请求路径中删除且需要重新设置的语言环境前缀）。插入链接）。

在这种情况下，您可以使用`fromXxx`接受a 的静态重载方法 `UriComponentsBuilder`来使用基本URL。另外，您可以`MvcUriComponentsBuilder` 使用基本URL 创建的实例，然后使用基于实例的`withXxx`方法。例如，以下清单使用`withMethodCall`：



 

```java
UriComponentsBuilder base = ServletUriComponentsBuilder.fromCurrentContextPath().path("/en");
MvcUriComponentsBuilder builder = MvcUriComponentsBuilder.relativeTo(base);
builder.withMethodCall(on(BookingController.class).getBooking(21)).buildAndExpand(42);

URI uri = uriComponents.encode().toUri();
```

|      | 从5.1开始，`MvcUriComponentsBuilder`忽略来自`Forwarded`和 `X-Forwarded-*`标头的信息，该标头指定了客户端起源的地址。考虑使用 [ForwardedHeaderFilter](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#filters-forwarded-headers)提取和使用或丢弃此类标头。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.5.6。视图中的链接

在Thymeleaf，FreeMarker或JSP之类的视图中，您可以通过引用每个请求映射的隐式或显式分配的名称来构建到带注释的控制器的链接。

考虑以下示例：



 

```java
@RequestMapping("/people/{id}/addresses")
public class PersonAddressController {

    @RequestMapping("/{country}")
    public HttpEntity<PersonAddress> getAddress(@PathVariable String country) { ... }
}
```

给定前面的控制器，您可以按照以下步骤准备来自JSP的链接：

```jsp
<%@ taglib uri="http://www.springframework.org/tags" prefix="s" %>
...
<a href="${s:mvcUrl('PAC#getAddress').arg(0,'US').buildAndExpand('123')}">Get Address</a>
```

上面的示例依赖于`mvcUrl`Spring标记库中声明的函数（即META-INF / spring.tld），但是很容易定义您自己的函数或为其他模板技术准备类似的函数。

这是这样的。在启动时，每个人`@RequestMapping`都通过分配了一个默认名称`HandlerMethodMappingNamingStrategy`，其默认实现使用类的大写字母和方法名称（例如，中的`getThing`方法 `ThingController`变为“ TC＃getThing”）。如果名称冲突，则可以使用 `@RequestMapping(name="..")`来分配一个明确的名称或实现自己的名称 `HandlerMethodMappingNamingStrategy`。

### 1.6。异步请求

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-vs-webflux)

Spring MVC与Servlet 3.0异步请求[处理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-processing)具有广泛的集成 ：

- [`DeferredResult`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-deferredresult)和[`Callable`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-callable) 控制器方法中的返回值，并为单个异步返回值提供基本支持。
- 控制器可以[流式传输](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-http-streaming)多个值，包括 [SSE](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-sse)和[原始数据](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-output-stream)。
- 控制器可以使用反应式客户端并返回 [反应式类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types)以进行响应处理。

#### 1.6.1。 `DeferredResult`

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-vs-webflux)

一旦 在Servlet容器中[启用](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-configuration)了异步请求处理功能，控制器方法就可以使用来包装任何受支持的控制器方法返回值`DeferredResult`，如以下示例所示：



 

```java
@GetMapping("/quotes")
@ResponseBody
public DeferredResult<String> quotes() {
    DeferredResult<String> deferredResult = new DeferredResult<String>();
    // Save the deferredResult somewhere..
    return deferredResult;
}

// From some other thread...
deferredResult.setResult(result);
```

控制器可以从另一个线程异步生成返回值，例如，响应外部事件（JMS消息），计划任务或其他事件。

#### 1.6.2。 `Callable`

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-vs-webflux)

控制器可以使用来包装任何受支持的返回值`java.util.concurrent.Callable`，如以下示例所示：



 

```java
@PostMapping
public Callable<String> processUpload(final MultipartFile file) {

    return new Callable<String>() {
        public String call() throws Exception {
            // ...
            return "someView";
        }
    };
}
```

然后可以通过[配置的](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-configuration-spring-mvc) 运行给定任务来获得返回值 `TaskExecutor`。

#### 1.6.3。处理中

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-vs-webflux)

这是Servlet异步请求处理的非常简洁的概述：

- `ServletRequest`可以通过调用将A 置于异步模式`request.startAsync()`。这样做的主要效果是Servlet（以及所有过滤器）可以退出，但是响应保持打开状态，以便以后完成处理。
- 要调用`request.startAsync()`的回报`AsyncContext`，您可以使用超过异步处理的进一步控制。例如，它提供了`dispatch`与Servlet API中的转发类似的方法，不同之处在于，它允许应用程序恢复Servlet容器线程上的请求处理。
- 在`ServletRequest`提供对电流`DispatcherType`，它可以使用处理该初始请求，异步调度之间进行区分，前向，以及其他的调度类型。

`DeferredResult` 处理工作如下：

- 控制器返回a `DeferredResult`并将其保存在一些可访问它的内存队列或列表中。
- Spring MVC调用`request.startAsync()`。
- 同时，`DispatcherServlet`和所有配置的过滤器退出请求处理线程，但响应保持打开状态。
- 应用程序`DeferredResult`从某个线程设置，Spring MVC将请求分派回Servlet容器。
- 将`DispatcherServlet`被再次调用，并且处理与异步生产返回值恢复。

`Callable` 处理工作如下：

- 控制器返回`Callable`。
- Spring MVC调用`request.startAsync()`并将其提交`Callable`到`TaskExecutor`一个单独的线程中进行处理。
- 同时，`DispatcherServlet`和所有过滤器退出Servlet容器线程，但响应保持打开状态。
- 最终`Callable`产生一个结果，Spring MVC将请求分派回Servlet容器以完成处理。
- 将`DispatcherServlet`被再次调用，并且处理从所述异步生产返回值恢复`Callable`。

有关更多背景知识，您还可以阅读 在Spring MVC 3.2中引入了异步请求处理支持[的博客文章](https://spring.io/blog/2012/05/07/spring-mvc-3-2-preview-introducing-servlet-3-async-support)。

##### 异常处理

使用时`DeferredResult`，您可以选择呼叫`setResult`还是 `setErrorResult`例外。在这两种情况下，Spring MVC都将请求分派回Servlet容器以完成处理。然后将其视为是控制器方法返回了给定值，还是好像它产生了给定的异常。然后，异常将通过常规的异常处理机制（例如，调用 `@ExceptionHandler`方法）进行处理。

当您使用时`Callable`，会发生类似的处理逻辑，主要区别是从中返回了结果，`Callable`或者引发了异常。

##### 拦截

`HandlerInterceptor`实例的类型可以为`AsyncHandlerInterceptor`，以接收`afterConcurrentHandlingStarted`启动异步处理的初始请求（而不是`postHandle`和`afterCompletion`）上的 回调。

`HandlerInterceptor`实现也可以注册`CallableProcessingInterceptor` 或`DeferredResultProcessingInterceptor`，以与异步请求的生命周期更深入地集成（例如，处理超时事件）。请参阅 [`AsyncHandlerInterceptor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/AsyncHandlerInterceptor.html) 以获取更多详细信息。

`DeferredResult`提供`onTimeout(Runnable)`和`onCompletion(Runnable)`回调。有关 更多详细信息，请参见的[Javadoc`DeferredResult`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/context/request/async/DeferredResult.html)。`Callable`可以替换为`WebAsyncTask`暴露超时和完成回调的其他方法。

##### 与WebFlux相比

Servlet API最初是为通过Filter-Servlet链进行一次传递而构建的。Servlet 3.0中添加了异步请求处理，使应用程序可以退出Filter-Servlet链，但保留响应以进行进一步处理。Spring MVC异步支持围绕该机制构建。当控制器返回a时`DeferredResult`，退出Filter-Servlet链，并释放Servlet容器线程。稍后，当`DeferredResult`设置时，将进行一次`ASYNC`调度（到相同的URL），在此期间再次映射控制器，但是`DeferredResult`使用该值（就像控制器返回了它一样）而不是调用它来恢复处理。

相比之下，Spring WebFlux既不是基于Servlet API构建的，也不需要这种异步请求处理功能，因为它在设计上是异步的。异步处理已内置在所有框架协定中，并在请求处理的所有阶段得到内在支持。

从编程模型的角度来看，Spring MVC和Spring WebFlux都支持异步和响应[类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types)作为控制器方法中的返回值。Spring MVC甚至支持流式传输，包括反应性背压。但是，与WebFlux不同，WebFlux依赖于非阻塞I / O，并且每次写入都不需要额外的线程，因此对响应的单个写入仍然处于阻塞状态（并在单独的线程上执行）。

另一个根本区别在于Spring MVC的不支持在控制器方法参数异步或反应性类型（例如，`@RequestBody`，`@RequestPart`，和其它物质），也不会具有用于异步和反应类型作为模型属性的任何显式支持。Spring WebFlux确实支持所有这些。

#### 1.6.4。HTTP流

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-codecs-streaming)

您可以将`DeferredResult`和`Callable`用于单个异步返回值。如果要产生多个异步值并将那些值写入响应中怎么办？本节介绍如何执行此操作。

##### 对象

您可以使用`ResponseBodyEmitter`返回值生成一个对象流，其中每个对象都使用进行序列化 [`HttpMessageConverter`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#rest-message-conversion)并写入响应，如以下示例所示：



 

```java
@GetMapping("/events")
public ResponseBodyEmitter handle() {
    ResponseBodyEmitter emitter = new ResponseBodyEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

您还可以`ResponseBodyEmitter`在中将其用作主体，以`ResponseEntity`自定义响应的状态和标题。

当`emitter`引发异常时`IOException`（例如，如果远程客户端消失了），应用程序不负责清理连接，因此不应调用`emitter.complete` 或`emitter.completeWithError`。取而代之的是，Servlet容器自动启动 `AsyncListener`错误通知，Spring MVC在其中进行`completeWithError`调用。反过来，此调用`ASYNC`对应用程序执行最后一次调度，在此期间，Spring MVC调用已配置的异常解析器并完成请求。

##### 上证所

`SseEmitter`（的子类`ResponseBodyEmitter`）提供对[服务器发送事件的](https://www.w3.org/TR/eventsource/)支持 ，其中从服务器[发送的事件](https://www.w3.org/TR/eventsource/)根据W3C SSE规范进行格式化。要从控制器生成SSE流，请返回`SseEmitter`，如以下示例所示：



 

```java
@GetMapping(path="/events", produces=MediaType.TEXT_EVENT_STREAM_VALUE)
public SseEmitter handle() {
    SseEmitter emitter = new SseEmitter();
    // Save the emitter somewhere..
    return emitter;
}

// In some other thread
emitter.send("Hello once");

// and again later on
emitter.send("Hello again");

// and done at some point
emitter.complete();
```

虽然SSE是流式传输到浏览器的主要选项，但请注意Internet Explorer不支持服务器发送事件。考虑将Spring的 [WebSocket消息](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket)与 针对各种浏览器的[SockJS后备](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback)传输（包括SSE）一起使用。

另请参阅[上一节](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-objects)以获取有关异常处理的注释。

##### 原始数据

有时，绕过消息转换并直接流式传输到响应`OutputStream`（例如，用于文件下载）很有用 。您可以使用`StreamingResponseBody` 返回值类型来执行此操作，如以下示例所示：



 

```java
@GetMapping("/download")
public StreamingResponseBody handle() {
    return new StreamingResponseBody() {
        @Override
        public void writeTo(OutputStream outputStream) throws IOException {
            // write...
        }
    };
}
```

您可以`StreamingResponseBody`在中用作正文来自`ResponseEntity`定义响应的状态和标题。

#### 1.6.5。反应类型

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-codecs-streaming)

Spring MVC支持在控制器中使用反应式客户端库（另请参阅 WebFlux部分中的[反应式库](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-reactive-libraries)）。这包括`WebClient`from `spring-webflux`和其他，例如Spring Data反应数据存储库。在这种情况下，能够从控制器方法返回反应类型是很方便的。

反应性返回值的处理方式如下：

- 与使用相似，单值承诺也适用`DeferredResult`。示例包括`Mono`（Reactor）或`Single`（RxJava）。
- 与使用或 类似，适用于具有流媒体类型（例如`application/stream+json` 或`text/event-stream`）的多值流。示例包括（Reactor）或（RxJava）。应用程序也可以返回或。`ResponseBodyEmitter``SseEmitter``Flux``Observable``Flux``Observable`
- `application/json`与使用相似，适用于具有其他任何媒体类型（例如）的多值流`DeferredResult>`。

|      | Spring MVC通过[`ReactiveAdapterRegistry`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/ReactiveAdapterRegistry.html)from来 支持Reactor和RxJava `spring-core`，这使其可以适应多个反应式库。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

为了流式传输到响应，支持反应性背压，但响应的写仍处于阻塞状态，并且通过[configure](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-configuration-spring-mvc) `TaskExecutor`，在单独的线程上执行该写操作 ，以避免阻塞上游源（例如`Flux`从返回的源`WebClient`）。默认情况下，`SimpleAsyncTaskExecutor`用于阻止写操作，但是在负载下不适合使用。如果计划使用响应类型进行流传输，则应使用 [MVC配置](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-configuration-spring-mvc)来配置任务执行程序。

#### 1.6.6。断开连接

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-codecs-streaming)

当远程客户端离开时，Servlet API不提供任何通知。因此，在通过[SseEmitter](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-sse) 或[反应性类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types)流式传输到响应时，定期发送数据非常重要，因为如果客户端断开连接，写入将失败。发送可以采取空（仅评论）SSE事件或另一端必须将其解释为心跳和忽略的任何其他数据的形式。

或者，考虑使用具有内置心跳机制的Web消息传递解决方案（例如，基于 [WebSocket的STOMP](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp)或具有[SockJS的](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback) WebSocket ）。

#### 1.6.7。组态

[与WebFlux相比](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-vs-webflux)

必须在Servlet容器级别启用异步请求处理功能。MVC配置还为异步请求提供了多个选项。

##### Servlet容器

Filter和Servlet声明具有一个`asyncSupported`标志，需要将其设置`true` 为启用异步请求处理。此外，应声明过滤器映射以处理`ASYNC` `javax.servlet.DispatchType`。

在Java配置中，当您用于`AbstractAnnotationConfigDispatcherServletInitializer` 初始化Servlet容器时，这是自动完成的。

在`web.xml`配置中，您可以添加`true`到 `DispatcherServlet`和`Filter`声明以及添加 `ASYNC`到过滤器映射。

##### 春季MVC

MVC配置公开了以下与异步请求处理相关的选项：

- Java配置：在`configureAsyncSupport`上使用回调`WebMvcConfigurer`。
- XML名称空间：使用下的``元素``。

您可以配置以下内容：

- 异步请求的默认超时值（如果未设置）取决于基础Servlet容器。
- `AsyncTaskExecutor`用于在使用[响应类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-async-reactive-types)进行流式传输时阻止写操作 以及用于执行`Callable`从控制器方法返回的实例。我们强烈建议您配置此属性，如果您使用反应性类型进行流式处理或具有返回的控制器方法，则`Callable`默认情况下为a `SimpleAsyncTaskExecutor`。
- `DeferredResultProcessingInterceptor`实施和`CallableProcessingInterceptor`实施。

请注意，您还可以在a `DeferredResult`，a `ResponseBodyEmitter`和an 上设置默认超时值`SseEmitter`。对于`Callable`，您可以使用 `WebAsyncTask`提供超时值。

### 1.7。CORS

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-cors)

Spring MVC使您可以处理CORS（跨域资源共享）。本节介绍如何执行此操作。

#### 1.7.1。介绍

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-cors-intro)

出于安全原因，浏览器禁止AJAX调用当前来源以外的资源。例如，您可以将您的银行帐户放在一个标签中，将evil.com放在另一个标签中。来自evil.com的脚本不应使用您的凭据向您的银行API发出AJAX请求，例如，从您的帐户中提取资金！

跨域资源共享（CORS）是 由[大多数浏览器](https://caniuse.com/#feat=cors)实现的[W3C规范](https://www.w3.org/TR/cors/)，可让您指定授权哪种类型的跨域请求，而不是使用基于IFRAME或JSONP的安全性较低且功能较弱的变通办法。

#### 1.7.2。处理中

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-cors-processing)

CORS规范区分飞行前，简单和实际请求。要了解CORS的工作原理，您可以阅读 [本文](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)以及其他内容，或者参阅规范以获取更多详细信息。

Spring MVC `HandlerMapping`实现为CORS提供内置支持。成功将请求映射到处理程序后，`HandlerMapping`实现会检查给定请求和处理程序的CORS配置，并采取进一步的措施。飞行前请求直接处理，而简单和实际的CORS请求被拦截，验证并设置了必需的CORS响应标头。

为了启用跨域请求（即，`Origin`标头存在并且与请求的主机不同），您需要具有一些显式声明的CORS配置。如果找不到匹配的CORS配置，则飞行前请求将被拒绝。没有将CORS标头添加到简单和实际CORS请求的响应中，因此，浏览器拒绝了它们。

每个`HandlerMapping`都可以 使用基于URL模式的映射进行 单独[配置](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/handler/AbstractHandlerMapping.html#setCorsConfigurations-java.util.Map-)`CorsConfiguration`。在大多数情况下，应用程序使用MVC Java配置或XML名称空间声明此类映射，从而导致将单个全局映射传递给所有`HandlerMappping`实例。

您可以将全局CORS配置`HandlerMapping`与更细粒度的处理程序级CORS配置结合使用。例如，带注释的控制器可以使用类或方法级别的`@CrossOrigin`注释（其他处理程序可以实现 `CorsConfigurationSource`）。

组合全局和本地配置的规则通常是相加的，例如，所有全局和所有本地来源。对于只能接受单个值的那些属性（例如`allowCredentials`和`maxAge`），局部属性将覆盖全局值。请参阅 [`CorsConfiguration#combine(CorsConfiguration)`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/cors/CorsConfiguration.html#combine-org.springframework.web.cors.CorsConfiguration-) 以获取更多详细信息。

|      | 要从源中了解更多信息或进行高级自定义，请查看后面的代码：`CorsConfiguration``CorsProcessor`， `DefaultCorsProcessor``AbstractHandlerMapping` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.7.3。 `@CrossOrigin`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-cors-controller)

该[`@CrossOrigin`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html) 注释能够对带注释的控制器方法跨域请求，如下面的示例所示：



 

```java
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

默认情况下，`@CrossOrigin`允许：

- 所有起源。
- 所有标题。
- 控制器方法映射到的所有HTTP方法。

`allowedCredentials` 默认情况下不会启用，因为这会建立一个信任级别，该级别公开敏感的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。

`maxAge` 设置为30分钟。

`@CrossOrigin` 在类级别也受支持，并且被所有方法继承，如以下示例所示：



 

```java
@CrossOrigin(origins = "https://domain2.com", maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

您可以`@CrossOrigin`在类级别和方法级别上使用，如以下示例所示：



 

```java
@CrossOrigin(maxAge = 3600)
@RestController
@RequestMapping("/account")
public class AccountController {

    @CrossOrigin("https://domain2.com")
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }

    @DeleteMapping("/{id}")
    public void remove(@PathVariable Long id) {
        // ...
    }
}
```

#### 1.7.4。全局配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-cors-global)

除了细粒度的控制器方法级别配置之外，您可能还想定义一些全局CORS配置。您可以`CorsConfiguration` 在任何上分别设置基于URL的映射`HandlerMapping`。但是，大多数应用程序都使用MVC Java配置或MVC XML名称空间来执行此操作。

默认情况下，全局配置启用以下功能：

- 所有起源。
- 所有标题。
- `GET`，`HEAD`和`POST`方法。

`allowedCredentials` 默认情况下不会启用，因为这会建立一个信任级别，该级别公开敏感的用户特定信息（例如cookie和CSRF令牌），并且仅在适当的地方使用。

`maxAge` 设置为30分钟。

##### Java配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-cors-global)

要在MVC Java配置中启用CORS，可以使用`CorsRegistry`回调，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("https://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

##### XML配置

要在XML名称空间中启用CORS，可以使用``元素，如以下示例所示：

```xml
<mvc:cors>

    <mvc:mapping path="/api/**"
        allowed-origins="https://domain1.com, https://domain2.com"
        allowed-methods="GET, PUT"
        allowed-headers="header1, header2, header3"
        exposed-headers="header1, header2" allow-credentials="true"
        max-age="123" />

    <mvc:mapping path="/resources/**"
        allowed-origins="https://domain1.com" />

</mvc:cors>
```

#### 1.7.5。CORS过滤器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/webflux-cors.html#webflux-cors-webfilter)

您可以通过内置来应用CORS支持 [`CorsFilter`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/filter/CorsFilter.html)。

|      | 如果您尝试在`CorsFilter`Spring Security中使用，请记住Spring Security [内置了](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors) 对CORS的[支持](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#cors)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要配置过滤器，请将a传递 `CorsConfigurationSource`给其构造函数，如以下示例所示：



 

```java
CorsConfiguration config = new CorsConfiguration();

// Possibly...
// config.applyPermitDefaultValues()

config.setAllowCredentials(true);
config.addAllowedOrigin("https://domain1.com");
config.addAllowedHeader("*");
config.addAllowedMethod("*");

UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
source.registerCorsConfiguration("/**", config);

CorsFilter filter = new CorsFilter(source);
```

### 1.8。网络安全

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-web-security)

在[Spring Security的](https://projects.spring.io/spring-security/)项目提供了保护Web应用程序免受恶意攻击的支持。请参阅Spring Security参考文档，包括：

- [Spring MVC安全性](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#mvc)
- [Spring MVC测试支持](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#test-mockmvc)
- [CSRF保护](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#csrf)
- [安全响应标头](https://docs.spring.io/spring-security/site/docs/current/reference/html5/#headers)

[HDIV](https://hdiv.org/)是另一个与Spring MVC集成的Web安全框架。

### 1.9。HTTP缓存

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-caching)

HTTP缓存可以显着提高Web应用程序的性能。HTTP缓存围绕`Cache-Control`响应标头和随后的条件请求标头（例如`Last-Modified`和`ETag`）展开。`Cache-Control`为私有（例如浏览器）和公共（例如代理）缓存提供有关如何缓存和重用响应的建议。一个`ETag`头用于使没有身体可能导致一个304（NOT_MODIFIED）一个条件请求，如果内容没有改变。`ETag`可以看作是`Last-Modified`标头的更复杂的后继。

本节描述了Spring Web MVC中与HTTP缓存相关的选项。

#### 1.9.1。 `CacheControl`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-caching-cachecontrol)

[`CacheControl`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/http/CacheControl.html)提供对配置与`Cache-Control`标头相关的设置的支持，并在许多地方作为参数被接受：

- [`WebContentInterceptor`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/mvc/WebContentInterceptor.html)
- [`WebContentGenerator`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/support/WebContentGenerator.html)
- [控制器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-caching-etag-lastmodified)
- [静态资源](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-caching-static-resources)

尽管[RFC 7234](https://tools.ietf.org/html/rfc7234#section-5.2.2)描述了`Cache-Control`响应标头的所有可能的指令，但该`CacheControl`类型采用面向用例的方法，重点关注常见方案：



 

```java
// Cache for an hour - "Cache-Control: max-age=3600"
CacheControl ccCacheOneHour = CacheControl.maxAge(1, TimeUnit.HOURS);

// Prevent caching - "Cache-Control: no-store"
CacheControl ccNoStore = CacheControl.noStore();

// Cache for ten days in public and private caches,
// public caches should not transform the response
// "Cache-Control: max-age=864000, public, no-transform"
CacheControl ccCustom = CacheControl.maxAge(10, TimeUnit.DAYS).noTransform().cachePublic();
```

`WebContentGenerator`还接受如下更简单的`cachePeriod`属性（以秒为单位定义）：

- 甲`-1`值不产生`Cache-Control`响应头。
- 甲`0`值可以防止通过使用缓存`'Cache-Control: no-store'`指令。
- 一个`n > 0`值缓存用于给定响应`n`通过使用秒 `'Cache-Control: max-age=n'`指令。

#### 1.9.2。控制器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-caching-etag-lastmodified)

控制器可以添加对HTTP缓存的显式支持。我们建议您这样做，因为需要先计算资源的 `lastModified`or `ETag`值，然后才能将其与条件请求标头进行比较。控制器可以将`ETag`标头和`Cache-Control` 设置添加到中`ResponseEntity`，如以下示例所示：



 

```java
@GetMapping("/book/{id}")
public ResponseEntity<Book> showBook(@PathVariable Long id) {

    Book book = findBook(id);
    String version = book.getVersion();

    return ResponseEntity
            .ok()
            .cacheControl(CacheControl.maxAge(30, TimeUnit.DAYS))
            .eTag(version) // lastModified is also available
            .body(book);
}
```

如果与条件请求标头的比较表明内容未更改，则前面的示例发送带有空主体的304（NOT_MODIFIED）响应。否则， `ETag`和`Cache-Control`标头将添加到响应中。

您还可以在控制器中针对条件请求标头进行检查，如以下示例所示：



 

```java
@RequestMapping
public String myHandleMethod(WebRequest webRequest, Model model) {

    long eTag = ... 

    if (request.checkNotModified(eTag)) {
        return null; 
    }

    model.addAttribute(...); 
    return "myViewName";
}
```

|      | 特定于应用程序的计算。                           |
| ---- | ------------------------------------------------ |
|      | 响应已设置为304（NOT_MODIFIED）-无需进一步处理。 |
|      | 继续进行请求处理。                               |

可以使用三种变体来根据`eTag`值，`lastModified` 值或同时检查条件请求。对于条件`GET`和`HEAD`请求，您可以将响应设置为304（NOT_MODIFIED）。对于有条件的`POST`，`PUT`和`DELETE`，可以改为将响应设置为412（PRECONDITION_FAILED），以防止并发修改。

#### 1.9.3。静态资源

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-caching-static-resources)

您应使用`Cache-Control`和条件响应标头来提供静态资源，以实现最佳性能。请参阅“配置[静态资源](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-static-resources) ”部分。

#### 1.9.4。`ETag`过滤

您可以使用`ShallowEtagHeaderFilter`来添加`eTag`根据响应内容计算出的“浅” 值，从而节省带宽，但不节省CPU时间。请参阅[浅ETag](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#filters-shallow-etag)。

### 1.10。查看技术

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view)

Spring MVC中视图技术的使用是可插入的，无论您决定使用Thymeleaf，Groovy标记模板，JSP还是其他技术，主要取决于配置更改。本章介绍与Spring MVC集成的视图技术。我们假设您已经熟悉[View Resolution](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-viewresolver)。

|      | Spring MVC应用程序的视图位于该应用程序的内部信任范围内。视图可以访问应用程序上下文中的所有bean。因此，不建议在外部源可编辑模板的应用程序中使用Spring MVC的模板支持，因为这可能会带来安全隐患。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.10.1。胸腺

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-thymeleaf)

Thymeleaf是一种现代的服务器端Java模板引擎，它强调可以通过双击在浏览器中预览的自然HTML模板，这对于独立处理UI模板（例如，由设计师）非常有用，而无需使用正在运行的服务器。如果要替换JSP，Thymeleaf提供了最广泛的功能集之一，以使这种过渡更加容易。Thymeleaf是积极开发和维护的。有关更完整的介绍，请参见 [Thymeleaf](https://www.thymeleaf.org/)项目主页。

Thymeleaf与Spring MVC的集成由Thymeleaf项目管理。配置涉及几个bean声明，如 `ServletContextTemplateResolver`，`SpringTemplateEngine`和`ThymeleafViewResolver`。有关更多详细信息，请参见[Thymeleaf + Spring](https://www.thymeleaf.org/documentation.html)。

#### 1.10.2。FreeMarker

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-freemarker)

[Apache FreeMarker](https://freemarker.apache.org/)是一个模板引擎，用于生成从HTML到电子邮件等的任何类型的文本输出。Spring框架具有内置的集成，可以将Spring MVC与FreeMarker模板一起使用。

##### 查看配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-freemarker-contextconfig)

以下示例显示了如何将FreeMarker配置为视图技术：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.freeMarker();
    }

    // Configure FreeMarker...

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/WEB-INF/freemarker");
        return configurer;
    }
}
```

以下示例显示了如何在XML中进行配置：

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:freemarker/>
</mvc:view-resolvers>

<!-- Configure FreeMarker... -->
<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/WEB-INF/freemarker"/>
</mvc:freemarker-configurer>
```

另外，您也可以声明`FreeMarkerConfigurer`Bean以完全控制所有属性，如以下示例所示：

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
</bean>
```

您的模板需要存储在`FreeMarkerConfigurer` 上例所示的目录中。给定上述配置，如果您的控制器返回的视图名称为`welcome`，则解析器将查找 `/WEB-INF/freemarker/welcome.ftl`模板。

##### FreeMarker配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-views-freemarker)

您可以通过`Configuration`在bean上设置适当的bean属性，将FreeMarker的“设置”和“ SharedVariables”直接传递给FreeMarker 对象（由Spring管理）`FreeMarkerConfigurer`。该`freemarkerSettings`属性需要一个`java.util.Properties`对象，而该`freemarkerVariables`属性需要一个 `java.util.Map`。以下示例显示了如何使用`FreeMarkerConfigurer`：

```xml
<bean id="freemarkerConfig" class="org.springframework.web.servlet.view.freemarker.FreeMarkerConfigurer">
    <property name="templateLoaderPath" value="/WEB-INF/freemarker/"/>
    <property name="freemarkerVariables">
        <map>
            <entry key="xml_escape" value-ref="fmXmlEscape"/>
        </map>
    </property>
</bean>

<bean id="fmXmlEscape" class="freemarker.template.utility.XmlEscape"/>
```

有关设置和变量应用于`Configuration`对象的详细信息，请参见FreeMarker文档。

##### 表格处理

Spring提供了一个供JSP使用的标签库，其中包含一个 ``元素。该元素主要允许表单显示来自表单支持对象的值，并显示来自`Validator`Web或业务层中验证失败的结果。Spring还支持FreeMarker中的相同功能，并带有用于生成表单输入元素本身的附加便利宏。

###### 绑定宏

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-bind-macros)

`spring-webmvc.jar`FreeMarker 的文件中保留了一组标准的宏，因此它们始终可用于经过适当配置的应用程序。

Spring模板库中定义的某些宏被视为内部（私有）宏，但是在宏定义中不存在这种范围，使所有宏对调用代码和用户模板可见。以下各节仅关注您需要在模板中直接调用的宏。如果您希望直接查看宏代码，则将调用该文件`spring.ftl`并将其包含在 `org.springframework.web.servlet.view.freemarker`包中。

###### 简单绑定

在基于FreeMarker模板的HTML表单中，这些模板充当Spring MVC控制器的表单视图，您可以使用类似于下一个示例的代码绑定到字段值，并以类似于JSP的方式显示每个输入字段的错误消息。以下示例显示一个`personForm`视图：

```xml
<!-- FreeMarker macros have to be imported into a namespace.
    We strongly recommend sticking to 'spring'. -->
<#import "/spring.ftl" as spring/>
<html>
    ...
    <form action="" method="POST">
        Name:
        <@spring.bind "personForm.name"/>
        <input type="text"
            name="${spring.status.expression}"
            value="${spring.status.value?html}"/><br />
        <#list spring.status.errorMessages as error> <b>${error}</b> <br /> </#list>
        <br />
        ...
        <input type="submit" value="submit"/>
    </form>
    ...
</html>
```

`<@spring.bind>`需要一个'path'参数，该参数由命令对象的名称（除非您在控制器配置中对其进行了更改，否则为'command'）组成，后跟一个句点以及所需的命令对象上的字段名称绑定。您还可以使用嵌套字段，例如`command.address.street`。该`bind`宏假定默认的HTML转由指定的行为`ServletContext`参数 `defaultHtmlEscape`在`web.xml`。

宏的另一种形式称为`<@spring.bindEscaped>`第二个参数，该参数明确指定在状态错误消息或值中应使用HTML转义。您可以根据需要将其设置为`true`或`false`。附加的表单处理宏可简化HTML转义的使用，并且您应尽可能使用这些宏。下一节将对它们进行说明。

###### 输入宏

FreeMarker的其他便利宏可简化绑定和表单生成（包括验证错误显示）。从来没有必要使用这些宏来生成表单输入字段，并且您可以将它们与简单的HTML混合使用或匹配，或者直接调用我们前面强调的Spring绑定宏。

下表列出了可用的宏，它们显示了FreeMarker模板（FTL）定义和每个参数采用的参数列表：

| 巨集                                                         | FTL定义                                            |
| :----------------------------------------------------------- | :------------------------------------------------- |
| `message` （根据code参数从资源包中输出字符串）               | <@ spring.message代码/>                            |
| `messageText` （根据code参数从资源包中输出一个字符串，回退到默认参数的值） | <@ spring.message文本代码，文本/>                  |
| `url` （使用应用程序的上下文根作为相对URL的前缀）            | <@ spring.url relativeUrl />                       |
| `formInput` （用于收集用户输入的标准输入字段）               | <@ spring.formInput路径，属性，fieldType />        |
| `formHiddenInput` （用于输入非用户输入的隐藏输入字段）       | <@ spring.formHiddenInput路径，属性/>              |
| `formPasswordInput` （用于收集密码的标准输入字段。请注意，此类型的字段中不会填充任何值。） | <@ spring.formPasswordInput路径，属性/>            |
| `formTextarea` （大文本字段，用于收集长而自由格式的文本输入） | <@ spring.formTextarea路径，属性/>                 |
| `formSingleSelect` （选项的下拉框允许选择一个必需的值）      | <@ spring.formSingleSelect路径，选项，属性/>       |
| `formMultiSelect` （选项列表框，允许用户选择0个或多个值）    | <@ spring.formMultiSelect路径，选项，属性/>        |
| `formRadioButtons` （一组单选按钮，可从可用选项中进行单个选择） | <@ spring.formRadioButtons路径，选项分隔符，属性/> |
| `formCheckboxes` （一组允许选择0个或多个值的复选框）         | <@ spring.formCheckboxes路径，选项，分隔符，属性/> |
| `formCheckbox` （一个复选框）                                | <@ spring.formCheckbox路径，属性/>                 |
| `showErrors` （简化了绑定字段验证错误的显示）                | <@ spring.showErrors分隔符，classOrStyle />        |

|      | 在FreeMarker的模板，`formHiddenInput`而`formPasswordInput`实际上并不是必需的，因为你可以使用正常的`formInput`宏，指定`hidden`或`password` 作为值`fieldType`参数。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以上任何宏的参数都具有一致的含义：

- `path`：要绑定的字段名称（即“ command.name”）
- `options`：`Map`可在输入字段中选择的所有可用值中的A。映射的键表示从表单回发并绑定到命令对象的值。针对键存储的地图对象是在表单上显示给用户的标签，并且可能与表单回发的相应值不同。通常，这种地图由控制器作为参考数据提供。您可以使用任何`Map`实现，具体取决于所需的行为。对于严格排序的地图，您可以使用带有适当符号的`SortedMap`（例如`TreeMap`）`Comparator`，对于应按插入顺序返回值的任意地图，请使用a `LinkedHashMap`或`LinkedMap`from `commons-collections`。
- `separator`：在多个选项可用作离散元素（单选按钮或复选框）的情况下，用于分隔列表中的每个字符的字符序列（例如` `）。
- `attributes`：HTML标记本身包含的任意标记或文本的附加字符串。该字符串实际上是由宏回显的。例如，在 `textarea`字段中，您可以提供属性（例如'rows =“ 5” cols =“ 60”'），也可以传递样式信息，例如'style =“ border：1px solid silver”。
- `classOrStyle`：对于`showErrors`宏，`span` 包装每个错误的元素使用的CSS类的名称。如果未提供任何信息（或该值为空），则将错误包装在``标签中。

以下各节概述了宏的示例。

输入栏位

该`formInput`宏采用`path`参数（`command.name`）和一个附加`attributes` 参数（在后面的示例中为空）。该宏与所有其他表单生成宏一起，对path参数执行隐式Spring绑定。绑定将保持有效，直到发生新的绑定为止，因此该`showErrors`宏无需再次传递path参数-它在上一次为其创建绑定的字段上运行。

的`showErrors`宏需要一个分离器参数（其用于在给定的场中分离多个错误的字符），还接受第二个参数-此时，类名或风格属性。注意，FreeMarker可以为attributes参数指定默认值。以下示例显示了如何使用`formInput` 和`showWErrors`宏：

```xml
<@spring.formInput "command.name"/>
<@spring.showErrors "<br>"/>
```

下一个示例显示表单片段的输出，生成名称字段，并在提交表单后在字段中没有值的情况下显示验证错误。验证通过Spring的Validation框架进行。

生成的HTML类似于以下示例：

```jsp
Name:
<input type="text" name="name" value="">
<br>
    <b>required</b>
<br>
<br>
```

该`formTextarea`宏的工作方式相同的`formInput`宏观和接受相同的参数列表。通常，第二个参数（`attributes`）用于传递样式信息或`rows`和的`cols`属性`textarea`。

选择字段

您可以使用四个选择字段宏在HTML表单中生成常见的UI值选择输入：

- `formSingleSelect`
- `formMultiSelect`
- `formRadioButtons`
- `formCheckboxes`

四个宏中的每个宏都接受一个`Map`选项，这些选项包含表单字段的值以及与该值相对应的标签。值和标签可以相同。

下一个示例是FTL中的单选按钮。支持表单的对象为此字段指定默认值“伦敦”，因此无需验证。呈现表单时，将在模型中以“ cityMap”为名称提供可供选择的整个城市列表作为参考数据。以下清单显示了示例：

```jsp
...
Town:
<@spring.formRadioButtons "command.address.town", cityMap, ""/><br><br>
```

前面的清单呈现了一行单选按钮，其中一个对应于中的每个值`cityMap`，并使用的分隔符`""`。没有提供其他属性（缺少该宏的最后一个参数）。在`cityMap`使用相同的`String`在图中的每个键-值对。映射键是表单实际作为`POST`请求参数提交的键。映射值是用户看到的标签。在前面的示例中，给定三个知名城市的列表以及表单支持对象中的默认值，HTML类似于以下内容：

```jsp
Town:
<input type="radio" name="address.town" value="London">London</input>
<input type="radio" name="address.town" value="Paris" checked="checked">Paris</input>
<input type="radio" name="address.town" value="New York">New York</input>
```

如果您的应用程序希望通过内部代码处理城市（例如），则可以使用适当的键创建代码地图，如以下示例所示：



 

```java
protected Map<String, ?> referenceData(HttpServletRequest request) throws Exception {
    Map<String, String> cityMap = new LinkedHashMap<>();
    cityMap.put("LDN", "London");
    cityMap.put("PRS", "Paris");
    cityMap.put("NYC", "New York");

    Map<String, Object> model = new HashMap<>();
    model.put("cityMap", cityMap);
    return model;
}
```

现在，该代码会生成输出，其中无线电值是相关代码，但是用户仍然可以看到更加用户友好的城市名称，如下所示：

```jsp
Town:
<input type="radio" name="address.town" value="LDN">London</input>
<input type="radio" name="address.town" value="PRS" checked="checked">Paris</input>
<input type="radio" name="address.town" value="NYC">New York</input>
```

###### HTML转义

前面介绍的表单宏的默认用法导致HTML元素符合HTML 4.01，并且使用`web.xml`文件中定义的HTML转义的默认值（如Spring的绑定支持所使用）。要使元素符合XHTML或覆盖默认的HTML转义值，可以在模板（或模型中对模板可见的两个变量）中指定两个变量。在模板中指定它们的优点是，可以在稍后的模板处理中将它们更改为不同的值，以为表单中的不同字段提供不同的行为。

要为您的标记切换到XHTML兼容性，请`true`为名为的模型或上下文变量指定的值`xhtmlCompliant`，如以下示例所示：

```jsp
<#-- for FreeMarker -->
<#assign xhtmlCompliant = true>
```

处理完此指令后，Spring宏生成的任何元素现在都符合XHTML。

以类似的方式，您可以指定每个字段的HTML转义，如以下示例所示：

```jsp
<#-- until this point, default HTML escaping is used -->

<#assign htmlEscape = true>
<#-- next field will use HTML escaping -->
<@spring.formInput "command.name"/>

<#assign htmlEscape = false in spring>
<#-- all future fields will be bound with HTML escaping off -->
```

#### 1.10.3。Groovy标记

在[Groovy的标记模板引擎](http://groovy-lang.org/templating.html#_the_markuptemplateengine) 的主要目的是生成XML类标记（XML，XHTML，HTML5等），但你可以用它来生成任何基于文本的内容。Spring框架具有内置的集成，可以将Spring MVC与Groovy标记一起使用。

|      | Groovy标记模板引擎需要Groovy 2.3.1+。 |
| ---- | ------------------------------------- |
|      |                                       |

##### 组态

以下示例显示了如何配置Groovy标记模板引擎：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.groovy();
    }

    // Configure the Groovy Markup Template Engine...

    @Bean
    public GroovyMarkupConfigurer groovyMarkupConfigurer() {
        GroovyMarkupConfigurer configurer = new GroovyMarkupConfigurer();
        configurer.setResourceLoaderPath("/WEB-INF/");
        return configurer;
    }
}
```

以下示例显示了如何在XML中进行配置：

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:groovy/>
</mvc:view-resolvers>

<!-- Configure the Groovy Markup Template Engine... -->
<mvc:groovy-configurer resource-loader-path="/WEB-INF/"/>
```

##### 例

与传统的模板引擎不同，Groovy标记依赖于使用构建器语法的DSL。以下示例显示了HTML页面的示例模板：

```groovy
yieldUnescaped '<!DOCTYPE html>'
html(lang:'en') {
    head {
        meta('http-equiv':'"Content-Type" content="text/html; charset=utf-8"')
        title('My page')
    }
    body {
        p('This is an example of HTML contents')
    }
}
```

#### 1.10.4。脚本视图

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-script)

Spring框架具有一个内置的集成，可以将Spring MVC与可以在[JSR-223](https://www.jcp.org/en/jsr/detail?id=223) Java脚本引擎之上运行的任何模板库一起使用 。我们已经在不同的脚本引擎上测试了以下模板库：

| 脚本库                                                       | 脚本引擎                                               |
| :----------------------------------------------------------- | :----------------------------------------------------- |
| [车把](https://handlebarsjs.com/)                            | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [胡子](https://mustache.github.io/)                          | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [反应](https://facebook.github.io/react/)                    | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [EJS](https://www.embeddedjs.com/)                           | [纳斯霍恩](https://openjdk.java.net/projects/nashorn/) |
| [ERB](https://www.stuartellis.name/articles/erb/)            | [红宝石](https://www.jruby.org/)                       |
| [字符串模板](https://docs.python.org/2/library/string.html#template-strings) | [吉顿](https://www.jython.org/)                        |
| [Kotlin脚本模板](https://github.com/sdeleuze/kotlin-script-templating) | [ ](https://kotlinlang.org/)                           |

|      | 集成任何其他脚本引擎的基本规则是，它必须实现 `ScriptEngine`和`Invocable`接口。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 要求

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-script-dependencies)

您需要在类路径上具有脚本引擎，其细节因脚本引擎而异：

- 在[犀牛](https://openjdk.java.net/projects/nashorn/)的JavaScript引擎提供的Java 8+。强烈建议使用可用的最新更新版本。
- 应该将[JRuby](https://www.jruby.org/)添加为对Ruby支持的依赖。
- 应该将[Jython](https://www.jython.org/)添加为对Python支持的依赖项。
- `org.jetbrains.kotlin:kotlin-script-util`依赖关系和`META-INF/services/javax.script.ScriptEngineFactory` 包含`org.jetbrains.kotlin.script.jsr223.KotlinJsr223JvmLocalScriptEngineFactory` 一行的文件应添加以支持Kotlin脚本。有关更多详细信息，请参 [见此示例](https://github.com/sdeleuze/kotlin-script-templating)。

您需要具有脚本模板库。针对Javascript的一种方法是通过[WebJars](https://www.webjars.org/)。

##### 脚本模板

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-script-integrate)

您可以声明一个`ScriptTemplateConfigurer`bean，以指定要使用的脚本引擎，要加载的脚本文件，调用呈现模板的函数等等。以下示例使用Mustache模板和Nashorn JavaScript引擎：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("mustache.js");
        configurer.setRenderObject("Mustache");
        configurer.setRenderFunction("render");
        return configurer;
    }
}
```

以下示例显示了XML中的相同排列：

```xml
<mvc:annotation-driven/>

<mvc:view-resolvers>
    <mvc:script-template/>
</mvc:view-resolvers>

<mvc:script-template-configurer engine-name="nashorn" render-object="Mustache" render-function="render">
    <mvc:script location="mustache.js"/>
</mvc:script-template-configurer>
```

对于Java和XML配置，该控制器看起来没有什么不同，如以下示例所示：



 

```java
@Controller
public class SampleController {

    @GetMapping("/sample")
    public String test(Model model) {
        model.addAttribute("title", "Sample title");
        model.addAttribute("body", "Sample body");
        return "template";
    }
}
```

以下示例显示了Mustache模板：

```html
<html>
    <head>
        <title>{{title}}</title>
    </head>
    <body>
        <p>{{body}}</p>
    </body>
</html>
```

使用以下参数调用render函数：

- `String template`：模板内容
- `Map model`：视图模型
- `RenderingContext renderingContext`： [`RenderingContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/view/script/RenderingContext.html) 可以访问应用程序上下文，语言环境，模板加载器和URL（自5.0起）

`Mustache.render()` 与该签名本地兼容，因此您可以直接调用它。

如果您的模板技术需要一些自定义，则可以提供一个实现自定义渲染功能的脚本。例如，[Handlerbars](https://handlebarsjs.com/) 需要使用它们之前编译模板和需要 [填充工具](https://en.wikipedia.org/wiki/Polyfill)来模拟浏览器的一些设施，不可用在服务器端脚本引擎。

以下示例显示了如何执行此操作：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.scriptTemplate();
    }

    @Bean
    public ScriptTemplateConfigurer configurer() {
        ScriptTemplateConfigurer configurer = new ScriptTemplateConfigurer();
        configurer.setEngineName("nashorn");
        configurer.setScripts("polyfill.js", "handlebars.js", "render.js");
        configurer.setRenderFunction("render");
        configurer.setSharedEngine(false);
        return configurer;
    }
}
```

|      | 当使用非线程安全脚本引擎和非并发设计的模板库（例如在Nashorn上运行的Handlebars或React）时`sharedEngine`，`false`需要 将该属性设置为。在这种情况下，由于[此bug](https://bugs.openjdk.java.net/browse/JDK-8076099)，需要Java SE 8 update 60 ，但通常建议在任何情况下都使用最新的Java SE修补程序版本。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`polyfill.js`定义仅`window`车把正常运行所需的对象，如下所示：

```javascript
var window = {};
```

此基本`render.js`实现在使用模板之前先对其进行编译。生产就绪的实现还应该存储任何重用的缓存模板或预编译的模板。您可以在脚本方面进行操作（并处理所需的任何自定义，例如，管理模板引擎配置）。以下示例显示了如何执行此操作：

```javascript
function render(template, model) {
    var compiledTemplate = Handlebars.compile(template);
    return compiledTemplate(model);
}
```

查看Spring Framework单元测试， [Java](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc/src/test/java/org/springframework/web/servlet/view/script)和 [资源](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc/src/test/resources/org/springframework/web/servlet/view/script)，以获取更多配置示例。

#### 1.10.5。JSP和JSTL

Spring框架具有内置的集成，可以将Spring MVC与JSP和JSTL一起使用。

##### 查看解析器

使用JSP开发时，可以声明a `InternalResourceViewResolver`或 `ResourceBundleViewResolver`bean。

`ResourceBundleViewResolver`依靠属性文件来定义映射到类和URL的视图名称。使用`ResourceBundleViewResolver`，您可以仅使用一个解析器来混合不同类型的视图，如以下示例所示：

```xml
<!-- the ResourceBundleViewResolver -->
<bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="basename" value="views"/>
</bean>

# And a sample properties file is used (views.properties in WEB-INF/classes):
welcome.(class)=org.springframework.web.servlet.view.JstlView
welcome.url=/WEB-INF/jsp/welcome.jsp

productList.(class)=org.springframework.web.servlet.view.JstlView
productList.url=/WEB-INF/jsp/productlist.jsp
```

`InternalResourceViewResolver`也可以用于JSP。作为最佳实践，我们强烈建议您将JSP文件放在该目录下的`'WEB-INF'`目录中，以便客户端无法直接访问。

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

##### JSP与JSTL

使用JSP标准标记库（JSTL）时，必须使用特殊的视图类 `JstlView`，因为JSTL需要一些准备工作才能使用I18N功能。

##### Spring的JSP标签库

如前几章所述，Spring提供了将请求参数与命令对象的数据绑定。为了促进结合这些数据绑定功能的JSP页面的开发，Spring提供了一些使事情变得更加容易的标记。所有Spring标记都具有HTML转义功能，以启用或禁用字符转义。

该`spring.tld`标签库描述符（TLD）包含在`spring-webmvc.jar`。有关单个标签的全面参考，请浏览 [API参考](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/tags/package-summary.html#package.description) 或查看标签库说明。

##### Spring的表单标签库

从2.0版开始，Spring使用JSP和Spring Web MVC时，提供了一组全面的数据绑定感知标记，用于处理表单元素。每个标签都支持与其对应的HTML标签对应物的属性集，从而使标签变得熟悉且易于使用。标记生成的HTML符合HTML 4.01 / XHTML 1.0。

与其他表单/输入标签库不同，Spring的表单标签库与Spring Web MVC集成在一起，使标签可以访问命令对象和控制器处理的参考数据。正如我们在以下示例中所示，表单标签使JSP易于开发，读取和维护。

我们浏览一下表单标签，并查看一个如何使用每个标签的示例。我们包含了生成的HTML代码段，其中某些标记需要进一步的注释。

###### 组态

表单标签库捆绑在中`spring-webmvc.jar`。库描述符称为`spring-form.tld`。

要使用该库中的标记，请在JSP页面顶部添加以下指令：

```xml
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```

`form`您要用于此库中标签的标签名称前缀在哪里。

###### 表单标签

该标签呈现HTML'form'元素，并向内部标签公开绑定路径以进行绑定。它将命令对象放入中，`PageContext`以便内部标签可以访问该命令对象。该库中的所有其他标签都是该`form`标签的嵌套标签 。

假设我们有一个名为的域对象`User`。这是一个JavaBean，具有诸如`firstName`和的属性`lastName`。我们可以将其用作表单控制器的表单支持对象，该对象返回`form.jsp`。以下示例显示了`form.jsp`可能的样子：

```xml
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

该`firstName`和`lastName`值由放置在命令对象中检索`PageContext`由页控制器。继续阅读以查看内部标签如何与标签一起使用的更复杂的示例`form`。

下面的清单显示了生成的HTML，它看起来像标准格式：

```xml
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value="Harry"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value="Potter"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

前面的JSP假定表单支持对象的变量名称为 `command`。如果已将表单支持对象以另一个名称（肯定是最佳实践）放入模型中，则可以将表单绑定到命名变量，如以下示例所示：

```xml
<form:form modelAttribute="user">
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="2">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

###### 该`input`标签

默认情况下，此标记呈现具有`input`绑定值的HTML 元素`type='text'`。有关此标签的示例，请参见[The Form标签](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-view-jsp-formtaglib-formtag)。您还可以使用特定的HTML5类型，如`email`，`tel`，`date`，等。

###### 该`checkbox`标签

此标记呈现一个HTML `input`标记，其`type`设置为`checkbox`。

假设我们`User`有喜好，例如时事通讯订阅和兴趣爱好列表。以下示例显示了`Preferences`该类：



 

```java
public class Preferences {

    private boolean receiveNewsletter;
    private String[] interests;
    private String favouriteWord;

    public boolean isReceiveNewsletter() {
        return receiveNewsletter;
    }

    public void setReceiveNewsletter(boolean receiveNewsletter) {
        this.receiveNewsletter = receiveNewsletter;
    }

    public String[] getInterests() {
        return interests;
    }

    public void setInterests(String[] interests) {
        this.interests = interests;
    }

    public String getFavouriteWord() {
        return favouriteWord;
    }

    public void setFavouriteWord(String favouriteWord) {
        this.favouriteWord = favouriteWord;
    }
}
```

相应的内容`form.jsp`可能类似于以下内容：

```xml
<form:form>
    <table>
        <tr>
            <td>Subscribe to newsletter?:</td>
            <%-- Approach 1: Property is of type java.lang.Boolean --%>
            <td><form:checkbox path="preferences.receiveNewsletter"/></td>
        </tr>

        <tr>
            <td>Interests:</td>
            <%-- Approach 2: Property is of an array or of type java.util.Collection --%>
            <td>
                Quidditch: <form:checkbox path="preferences.interests" value="Quidditch"/>
                Herbology: <form:checkbox path="preferences.interests" value="Herbology"/>
                Defence Against the Dark Arts: <form:checkbox path="preferences.interests" value="Defence Against the Dark Arts"/>
            </td>
        </tr>

        <tr>
            <td>Favourite Word:</td>
            <%-- Approach 3: Property is of type java.lang.Object --%>
            <td>
                Magic: <form:checkbox path="preferences.favouriteWord" value="Magic"/>
            </td>
        </tr>
    </table>
</form:form>
```

`checkbox`标记有三种方法，应该可以满足您所有复选框的需求。

- 方法一：当界限值是type时`java.lang.Boolean`，将 `input(checkbox)`标记为`checked`界限值是`true`。该`value` 属性对应于`setValue(Object)`value属性的解析值。
- 方法二：当界限值的类型为`array`或时`java.util.Collection`，将 `input(checkbox)`标记为，`checked`好像`setValue(Object)`界限中存在配置的值`Collection`。
- 方法三：对于任何其他绑定值类型，将`input(checkbox)`标记为 `checked`配置的`setValue(Object)`值等于绑定值。

请注意，无论采用哪种方法，都会生成相同的HTML结构。以下HTML代码段定义了一些复选框：

```xml
<tr>
    <td>Interests:</td>
    <td>
        Quidditch: <input name="preferences.interests" type="checkbox" value="Quidditch"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Herbology: <input name="preferences.interests" type="checkbox" value="Herbology"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
        Defence Against the Dark Arts: <input name="preferences.interests" type="checkbox" value="Defence Against the Dark Arts"/>
        <input type="hidden" value="1" name="_preferences.interests"/>
    </td>
</tr>
```

您可能不希望在每个复选框之后看到其他隐藏字段。如果未选中HTML页面中的复选框，则提交表单后，其值就不会作为HTTP请求参数的一部分发送到服务器，因此我们需要一种解决方法来使HTML中的这个问题生效，以使Spring表单数据绑定生效。该 `checkbox`标记遵循现有的Spring约定，其中包括`_`每个复选框都带有下划线（）前缀的隐藏参数。通过这样做，您可以有效地告诉Spring“复选框在表单中可见，并且我希望表单数据绑定到我的对象以反映复选框的状态，无论如何。”

###### 该`checkboxes`标签

此标记呈现设置为的多个HTML `input`标记。`type``checkbox`

本节以上一个`checkbox`标记节的示例为基础。有时，您宁愿不必在JSP页面中列出所有可能的爱好。您宁愿在运行时提供可用选项的列表，然后将其传递给标记。这就是`checkboxes`标签的目的。您可以传递一个`Array`，一个`List`，或者`Map`包含在可用的选项`items`属性。通常，bound属性是一个集合，因此它可以容纳用户选择的多个值。以下示例显示了使用此标记的JSP：

```xml
<form:form>
    <table>
        <tr>
            <td>Interests:</td>
            <td>
                <%-- Property is of an array or of type java.util.Collection --%>
                <form:checkboxes path="preferences.interests" items="${interestList}"/>
            </td>
        </tr>
    </table>
</form:form>
```

这个例子假设`interestList`是`List`可用的为包含的值的字符串从被选择的模型的属性。如果使用`Map`，则将地图条目键用作值，并将地图条目的值用作要显示的标签。您还可以使用自定义对象，在其中您可以通过使用提供值的属性名称，并通过使用提供`itemValue`标签`itemLabel`。

###### 该`radiobutton`标签

此标记呈现一个HTML `input`元素，其`type`设置为`radio`。

典型的用法模式涉及绑定到相同属性但值不同的多个标记实例，如以下示例所示：

```xml
<tr>
    <td>Sex:</td>
    <td>
        Male: <form:radiobutton path="sex" value="M"/> <br/>
        Female: <form:radiobutton path="sex" value="F"/>
    </td>
</tr>
```

###### 该`radiobuttons`标签

此标记呈现多个HTML `input`元素，其`type`设置为`radio`。

与[`checkboxes`标记一样](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-view-jsp-formtaglib-checkboxestag)，您可能希望将可用选项作为运行时变量传递。对于这种用法，您可以使用 `radiobuttons`标签。你传递一个`Array`，一个`List`，或者`Map`包含在可用的选项`items`属性。如果使用`Map`，则将地图条目键用作值，并将地图条目的值用作要显示的标签。您还可以使用一个自定义对象，在其中可以使用来提供值的属性名称，并使用来提供`itemValue`标签`itemLabel`，如以下示例所示：

```xml
<tr>
    <td>Sex:</td>
    <td><form:radiobuttons path="sex" items="${sexOptions}"/></td>
</tr>
```

###### 该`password`标签

此标记呈现`input`类型为`password`具有绑定值的HTML 标记。

```xml
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password"/>
    </td>
</tr>
```

请注意，默认情况下，不显示密码值。如果确实希望显示密码值，则可以将`showPassword`属性的值设置为 `true`，如以下示例所示：

```xml
<tr>
    <td>Password:</td>
    <td>
        <form:password path="password" value="^76525bvHGq" showPassword="true"/>
    </td>
</tr>
```

###### 该`select`标签

此标记呈现HTML“ select”元素。它支持将数据绑定到所选选项以及使用嵌套`option`和`options`标记。

假设a `User`具有一系列技能。相应的HTML可能如下所示：

```xml
<tr>
    <td>Skills:</td>
    <td><form:select path="skills" items="${skills}"/></td>
</tr>
```

如果该`User’s`技能是草药学，则“技能”行的HTML源可能如下：

```xml
<tr>
    <td>Skills:</td>
    <td>
        <select name="skills" multiple="true">
            <option value="Potions">Potions</option>
            <option value="Herbology" selected="selected">Herbology</option>
            <option value="Quidditch">Quidditch</option>
        </select>
    </td>
</tr>
```

###### 该`option`标签

此标记呈现HTML `option`元素。它集`selected`的基础上限值。以下HTML显示了其典型输出：

```xml
<tr>
    <td>House:</td>
    <td>
        <form:select path="house">
            <form:option value="Gryffindor"/>
            <form:option value="Hufflepuff"/>
            <form:option value="Ravenclaw"/>
            <form:option value="Slytherin"/>
        </form:select>
    </td>
</tr>
```

如果`User’s`房屋位于格兰芬多，则“房屋”行的HTML源代码如下：

```xml
<tr>
    <td>House:</td>
    <td>
        <select name="house">
            <option value="Gryffindor" selected="selected">Gryffindor</option> 
            <option value="Hufflepuff">Hufflepuff</option>
            <option value="Ravenclaw">Ravenclaw</option>
            <option value="Slytherin">Slytherin</option>
        </select>
    </td>
</tr>
```

|      | 注意`selected`属性的添加。 |
| ---- | -------------------------- |
|      |                            |

###### 该`options`标签

此标记呈现HTML `option`元素列表。它`selected`根据绑定值设置属性。以下HTML显示了其典型输出：

```xml
<tr>
    <td>Country:</td>
    <td>
        <form:select path="country">
            <form:option value="-" label="--Please Select"/>
            <form:options items="${countryList}" itemValue="code" itemLabel="name"/>
        </form:select>
    </td>
</tr>
```

如果`User`居住在英国，则“国家/地区”行的HTML来源如下：

```xml
<tr>
    <td>Country:</td>
    <td>
        <select name="country">
            <option value="-">--Please Select</option>
            <option value="AT">Austria</option>
            <option value="UK" selected="selected">United Kingdom</option> 
            <option value="US">United States</option>
        </select>
    </td>
</tr>
```

|      | 注意`selected`属性的添加。 |
| ---- | -------------------------- |
|      |                            |

如前面的示例所示，`option`标记与`options`标记的组合用法会生成相同的标准HTML，但允许您在JSP中显式指定一个仅用于显示（该标记所属的位置）的值，例如示例中的默认字符串： “ - 请选择”。

`items`通常，该属性填充有项目对象的集合或数组。 `itemValue`并`itemLabel`引用这些项目对象的bean属性（如果已指定）。否则，项目对象本身将变成字符串。或者，您可以指定一个`Map`项，在这种情况下，映射键将解释为选项值，并且映射值对应于选项标签。如果碰巧也指定了`itemValue`或`itemLabel`（或两者），则item value属性适用于地图键，item label属性适用于地图值。

###### 该`textarea`标签

此标记呈现HTML `textarea`元素。以下HTML显示了其典型输出：

```xml
<tr>
    <td>Notes:</td>
    <td><form:textarea path="notes" rows="3" cols="20"/></td>
    <td><form:errors path="notes"/></td>
</tr>
```

###### 该`hidden`标签

此标记`input`使用`type`设置`hidden`的绑定值呈现HTML 标记。要提交未绑定的隐藏值，请使用设置为的HTML `input`标签。以下HTML显示了其典型输出：`type``hidden`

```xml
<form:hidden path="house"/>
```

如果我们选择将`house`值作为隐藏值提交，则HTML如下所示：

```xml
<input name="house" type="hidden" value="Gryffindor"/>
```

###### 该`errors`标签

此标记在HTML `span`元素中呈现字段错误。它提供对在控制器中创建的错误或由与控制器关联的任何验证程序创建的错误的访问。

假设 一旦提交表单，我们想显示`firstName`和`lastName`字段的所有错误消息。我们有一个`User`名为的类实例的验证器`UserValidator`，如以下示例所示：



 

```java
public class UserValidator implements Validator {

    public boolean supports(Class candidate) {
        return User.class.isAssignableFrom(candidate);
    }

    public void validate(Object obj, Errors errors) {
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "firstName", "required", "Field is required.");
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "lastName", "required", "Field is required.");
    }
}
```

在`form.jsp`可能如下：

```xml
<form:form>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <%-- Show errors for firstName field --%>
            <td><form:errors path="firstName"/></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <%-- Show errors for lastName field --%>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

如果我们在`firstName`和`lastName`字段中提交具有空值的表单，则HTML如下所示：

```xml
<form method="POST">
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <%-- Associated errors to firstName field displayed --%>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <%-- Associated errors to lastName field displayed --%>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

如果我们要显示给定页面的整个错误列表怎么办？下一个示例显示该`errors`标记还支持一些基本的通配符功能。

- `path="*"`：显示所有错误。
- `path="lastName"`：显示与该`lastName`字段相关的所有错误。
- 如果`path`省略，则仅显示对象错误。

以下示例在页面顶部显示错误列表，然后在字段旁边显示特定于字段的错误：

```xml
<form:form>
    <form:errors path="*" cssClass="errorBox"/>
    <table>
        <tr>
            <td>First Name:</td>
            <td><form:input path="firstName"/></td>
            <td><form:errors path="firstName"/></td>
        </tr>
        <tr>
            <td>Last Name:</td>
            <td><form:input path="lastName"/></td>
            <td><form:errors path="lastName"/></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form:form>
```

HTML将如下所示：

```xml
<form method="POST">
    <span name="*.errors" class="errorBox">Field is required.<br/>Field is required.</span>
    <table>
        <tr>
            <td>First Name:</td>
            <td><input name="firstName" type="text" value=""/></td>
            <td><span name="firstName.errors">Field is required.</span></td>
        </tr>

        <tr>
            <td>Last Name:</td>
            <td><input name="lastName" type="text" value=""/></td>
            <td><span name="lastName.errors">Field is required.</span></td>
        </tr>
        <tr>
            <td colspan="3">
                <input type="submit" value="Save Changes"/>
            </td>
        </tr>
    </table>
</form>
```

该`spring-form.tld`标签库描述符（TLD）包含在`spring-webmvc.jar`。有关单个标签的全面参考，请浏览 [API参考](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/tags/form/package-summary.html#package.description) 或查看标签库说明。

###### HTTP方法转换

REST的一个关键原则是使用“统一接口”。这意味着可以使用相同的四种HTTP方法（GET，PUT，POST和DELETE）来操纵所有资源（URL）。对于每种方法，HTTP规范都定义了确切的语义。例如，GET应该始终是安全的操作，这意味着它没有副作用，而PUT或DELETE应该是幂等的，这意味着您可以一遍又一遍地重复这些操作，但是最终结果应该相同。虽然HTTP定义了这四种方法，但是HTML仅支持两种：GET和POST。幸运的是，有两种可能的解决方法：您可以使用JavaScript进行PUT或DELETE，或者可以使用“ real”方法作为附加参数（在HTML表单中建模为隐藏的输入字段）进行POST。春天的`HiddenHttpMethodFilter`使用后一种技巧。该过滤器是一个普通的Servlet过滤器，因此，它可以与任何Web框架（不仅仅是Spring MVC）结合使用。将此过滤器添加到您的web.xml，然后将带有隐藏`method`参数的POST 转换为相应的HTTP方法请求。

为了支持HTTP方法转换，Spring MVC表单标签已更新为支持设置HTTP方法。例如，以下代码片段来自“宠物诊所”样本：

```xml
<form:form method="delete">
    <p class="submit"><input type="submit" value="Delete Pet"/></p>
</form:form>
```

前面的示例执行HTTP POST，并将“真实” DELETE方法隐藏在请求参数后面。它由`HiddenHttpMethodFilter`web.xml中定义的拾取，如以下示例所示：

```xml
<filter>
    <filter-name>httpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>httpMethodFilter</filter-name>
    <servlet-name>petclinic</servlet-name>
</filter-mapping>
```

以下示例显示了相应的`@Controller`方法：



 

```java
@RequestMapping(method = RequestMethod.DELETE)
public String deletePet(@PathVariable int ownerId, @PathVariable int petId) {
    this.clinic.deletePet(petId);
    return "redirect:/owners/" + ownerId;
}
```

###### HTML5标签

Spring表单标签库允许输入动态属性，这意味着您可以输入任何HTML5特定的属性。

表单`input`标签支持输入以外的类型属性`text`。这是为了让渲染新的HTML5特定的输入类型，如`email`，`date`， `range`，等。注意`type='text'`，由于`text` 是默认类型，因此不需要输入。

#### 1.10.6。瓷砖

您可以像使用其他视图技术一样，将Tiles集成到使用Spring的Web应用程序中。本节将广泛介绍如何执行此操作。

|      | 本节重点介绍Spring对`org.springframework.web.servlet.view.tiles3`软件包中Tiles版本3的支持 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 依存关系

为了能够使用Tiles，您必须在Tiles 3.0.1或更高版本上添加一个依赖项，并将[其传递性依赖项添加](https://tiles.apache.org/framework/dependency-management.html) 到您的项目中。

##### 组态

为了能够使用Tiles，必须使用包含定义的文件对其进行配置（有关定义和其他Tiles概念的基本信息，请参见 [https://tiles.apache.org](https://tiles.apache.org/)）。在Spring中，这是通过使用来完成的`TilesConfigurer`。以下示例`ApplicationContext`配置显示了如何执行此操作：

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>
</bean>
```

前面的示例定义了五个包含定义的文件。这些文件都位于`WEB-INF/defs`目录中。在初始化时`WebApplicationContext`，将加载文件，并初始化定义工厂。完成之后，定义文件中包含的Tiles可以用作Spring Web应用程序中的视图。为了能够使用视图，您必须具有`ViewResolver` 与Spring一起使用的任何其他视图技术一样的功能。您可以使用两种实现方式，在任`UrlBasedViewResolver`和`ResourceBundleViewResolver`。

您可以通过添加下划线然后添加语言环境来指定特定于语言环境的Tiles定义，如以下示例所示：

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/tiles.xml</value>
            <value>/WEB-INF/defs/tiles_fr_FR.xml</value>
        </list>
    </property>
</bean>
```

通过上述配置，`tiles_fr_FR.xml`用于具有`fr_FR`语言环境的请求，并且`tiles.xml`默认情况下使用。

|      | 由于下划线用于指示语言环境，因此我们建议不要在图块定义的文件名中使用下划线。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

###### `UrlBasedViewResolver`

该`UrlBasedViewResolver`实例化给出`viewClass`了它解决每个视图。以下bean定义了一个`UrlBasedViewResolver`：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.UrlBasedViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.tiles3.TilesView"/>
</bean>
```

###### `ResourceBundleViewResolver`

在`ResourceBundleViewResolver`必须提供与包含视图名称与视图类，解析器可以使用属性文件。以下示例显示了a的bean定义`ResourceBundleViewResolver`以及相应的视图名称和视图类（摘自Pet Clinic示例）：

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
    <property name="basename" value="views"/>
</bean>
    ...
    welcomeView。（类）= org.springframework.web.servlet.view.tiles3.TilesView
    welcomeView.url = welcome（这是Tiles定义的名称）

    vetsView。（class）= org.springframework.web.servlet.view.tiles3.TilesView
    vetsView.url = vetsView（同样，这是Tiles定义的名称）

    findOwnersForm。（class）= org.springframework.web.servlet.view.JstlView
    findOwnersForm.url = / WEB-INF / jsp / findOwners.jsp
    ...
```

使用时`ResourceBundleViewResolver`，您可以轻松地混合使用不同的视图技术。

请注意，`TilesView`该类支持JSTL（JSP标准标记库）。

###### `SimpleSpringPreparerFactory` 和 `SpringBeanPreparerFactory`

作为一项高级功能，Spring还支持两种特殊的Tiles `PreparerFactory` 实现。有关如何`ViewPreparer`在Tiles定义文件中使用引用的详细信息，请参见Tiles文档 。

您可以指定基于指定的准备器类`SimpleSpringPreparerFactory`自动装配`ViewPreparer`实例，应用Spring的容器回调以及应用配置的Spring BeanPostProcessors。如果已激活Spring的上下文范围的批注配置，则将`ViewPreparer`自动检测和应用类中的批注。请注意，这与默认值`PreparerFactory`一样，期望Tiles定义文件中的准备程序类。

您可以指定`SpringBeanPreparerFactory`对指定的准备器名称（而不是类）进行操作，从而从DispatcherServlet的应用程序上下文中获取相应的Spring bean。在这种情况下，完整的bean创建过程由Spring应用程序上下文控制，从而允许使用显式依赖项注入配置，作用域bean等。注意，您需要为每个准备器名称定义一个Spring bean定义（在Tiles定义中使用）。以下示例显示如何`SpringBeanPreparerFactory`在`TilesConfigurer`bean 上定义属性：

```xml
<bean id="tilesConfigurer" class="org.springframework.web.servlet.view.tiles3.TilesConfigurer">
    <property name="definitions">
        <list>
            <value>/WEB-INF/defs/general.xml</value>
            <value>/WEB-INF/defs/widgets.xml</value>
            <value>/WEB-INF/defs/administrator.xml</value>
            <value>/WEB-INF/defs/customer.xml</value>
            <value>/WEB-INF/defs/templates.xml</value>
        </list>
    </property>

    <!-- resolving preparer names as Spring bean definition names -->
    <property name="preparerFactoryClass"
            value="org.springframework.web.servlet.view.tiles3.SpringBeanPreparerFactory"/>

</bean>
```

#### 1.10.7。RSS和Atom

二者`AbstractAtomFeedView`并`AbstractRssFeedView`从继承 `AbstractFeedView`基类和用于提供Atom和RSS提要视图，分别。它们基于[ROME](https://rometools.github.io/rome/)项目，位于软件包中`org.springframework.web.servlet.view.feed`。

`AbstractAtomFeedView`需要您实现该`buildFeedEntries()`方法并可以选择覆盖该`buildFeedMetadata()`方法（默认实现为空）。以下示例显示了如何执行此操作：



 

```java
public class SampleContentAtomView extends AbstractAtomFeedView {

    @Override
    protected void buildFeedMetadata(Map<String, Object> model,
            Feed feed, HttpServletRequest request) {
        // implementation omitted
    }

    @Override
    protected List<Entry> buildFeedEntries(Map<String, Object> model,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        // implementation omitted
    }
}
```

类似的要求也适用于实施`AbstractRssFeedView`，如以下示例所示：



 

```java
public class SampleContentRssView extends AbstractRssFeedView {

    @Override
    protected void buildFeedMetadata(Map<String, Object> model,
            Channel feed, HttpServletRequest request) {
        // implementation omitted
    }

    @Override
    protected List<Item> buildFeedItems(Map<String, Object> model,
            HttpServletRequest request, HttpServletResponse response) throws Exception {
        // implementation omitted
    }
}
```

该`buildFeedItems()`和`buildFeedEntries()`在HTTP请求方法传递，如果你需要访问的语言环境。仅针对Cookie或其他HTTP标头的设置传入HTTP响应。方法返回后，提要将自动写入响应对象。

有关创建Atom视图的示例，请参见Alef Arendsen的Spring Team Blog [条目](https://spring.io/blog/2009/03/16/adding-an-atom-view-to-an-application-using-spring-s-rest-support)。

#### 1.10.8。PDF和Excel

Spring提供了返回HTML以外的输出的方法，包括PDF和Excel电子表格。本节介绍如何使用这些功能。

##### 文档视图简介

HTML页面并非始终是用户查看模型输出的最佳方式，而Spring使从模型数据动态生成PDF文档或Excel电子表格变得简单。该文档是视图，并从服务器以正确的内容类型进行流传输，以（希望）使客户端PC能够运行其电子表格或PDF查看器应用程序作为响应。

为了使用Excel视图，您需要将Apache POI库添加到类路径中。为了生成PDF，您需要添加（最好是）OpenPDF库。

|      | 如果可能，您应该使用基础文档生成库的最新版本。特别是，我们强烈建议您使用OpenPDF（例如，OpenPDF 1.2.12）而不是过时的原始iText 2.1.7，因为OpenPDF会得到积极维护并修复了不可信任PDF内容的重要漏洞。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### PDF检视

单词列表的简单PDF视图可以扩展 `org.springframework.web.servlet.view.document.AbstractPdfView`和实现该 `buildPdfDocument()`方法，如以下示例所示：



 

```java
public class PdfWordList extends AbstractPdfView {

    protected void buildPdfDocument(Map<String, Object> model, Document doc, PdfWriter writer,
            HttpServletRequest request, HttpServletResponse response) throws Exception {

        List<String> words = (List<String>) model.get("wordList");
        for (String word : words) {
            doc.add(new Paragraph(word));
        }
    }
}
```

控制器可以从外部视图定义（按名称引用）或作为`View`处理程序方法的实例返回这种视图。

##### Excel视图

从Spring Framework 4.2开始， `org.springframework.web.servlet.view.document.AbstractXlsView`它作为Excel视图的基类提供。它基于Apache POI，具有取代过时类的专用子类（`AbstractXlsxView` 和`AbstractXlsxStreamingView`）`AbstractExcelView`。

编程模型类似于`AbstractPdfView`，`buildExcelDocument()` 作为中央模板方法，控制器能够从外部定义（按名称）或`View`从处理程序方法作为实例返回这种视图。

#### 1.10.9。杰克逊

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-httpmessagewriter)

Spring提供了对Jackson JSON库的支持。

##### 基于Jackson的JSON MVC视图

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-httpmessagewriter)

该`MappingJackson2JsonView`用途杰克逊库的`ObjectMapper`渲染响应内容为JSON。默认情况下，模型映射的所有内容（特定于框架的类除外）均编码为JSON。对于需要过滤地图内容的情况，可以使用属性指定一组特定的模型属性进行编码`modelKeys`。您还可以使用该`extractValueFromSingleKeyModel` 属性直接提取和序列化单键模型中的值，而不是将其作为模型属性的映射。

您可以根据需要使用Jackson提供的注释来自定义JSON映射。当需要进一步控制时，可以为需要为特定类型提供自定义JSON序列化器和反序列化器的情况，`ObjectMapper` 通过`ObjectMapper`属性注入自定义。

##### 基于Jackson的XML视图

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-view-httpmessagewriter)

`MappingJackson2XmlView`使用 [Jackson XML扩展](https://github.com/FasterXML/jackson-dataformat-xml) `XmlMapper` 名将响应内容呈现为XML。如果模型包含多个条目，则应使用`modelKey`bean属性显式设置要序列化的对象。如果模型包含单个条目，那么它将自动序列化。

您可以根据需要使用JAXB或Jackson提供的注释自定义XML映射。当需要进一步控制时，可以`XmlMapper` 通过`ObjectMapper`属性注入自定义，对于自定义XML的情况，您需要为特定类型提供序列化器和反序列化器。

#### 1.10.10。XML编组

在`MarshallingView`使用XML `Marshaller`（在定义的`org.springframework.oxm` 包）来呈现响应内容为XML。您可以使用`MarshallingView`实例的`modelKey`bean属性来显式设置要编组的对象。或者，视图迭代所有模型属性，并封送所支持的第一个类型`Marshaller`。有关`org.springframework.oxm`包中功能的更多信息 ，请参见[使用O / X映射器编组XML](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/data-access.html#oxm)。

#### 1.10.11。XSLT视图

XSLT是XML的一种转换语言，在Web应用程序中作为一种视图技术而流行。如果您的应用程序自然处理XML，或者您的模型可以轻松转换为XML，那么XSLT可以作为视图技术的不错选择。下一节说明如何将XML文档生成为模型数据，以及如何在Spring Web MVC应用程序中使用XSLT对其进行转换。

这个示例是一个简单的Spring应用程序，它在中创建一个单词列表 `Controller`并将其添加到模型图。返回该地图以及XSLT视图的视图名称。有关Spring Web MVC 界面的详细信息， 请参见带[注释的控制器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-controller)`Controller`。XSLT控制器将单词列表转换为准备转换的简单XML文档。

##### 豆子

配置是简单Spring Web应用程序的标准配置：MVC配置必须定义`XsltViewResolver`Bean和常规MVC注释配置。以下示例显示了如何执行此操作：



 

```java
@EnableWebMvc
@ComponentScan
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public XsltViewResolver xsltViewResolver() {
        XsltViewResolver viewResolver = new XsltViewResolver();
        viewResolver.setPrefix("/WEB-INF/xsl/");
        viewResolver.setSuffix(".xslt");
        return viewResolver;
    }
}
```

##### 控制者

我们还需要一个控制器来封装词生成逻辑。

控制器逻辑封装在一个`@Controller`类中，其中handler方法的定义如下：



 

```java
@Controller
public class XsltController {

    @RequestMapping("/")
    public String home(Model model) throws Exception {
        Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().newDocument();
        Element root = document.createElement("wordList");

        List<String> words = Arrays.asList("Hello", "Spring", "Framework");
        for (String word : words) {
            Element wordNode = document.createElement("word");
            Text textNode = document.createTextNode(word);
            wordNode.appendChild(textNode);
            root.appendChild(wordNode);
        }

        model.addAttribute("wordList", root);
        return "home";
    }
}
```

到目前为止，我们仅创建了一个DOM文档并将其添加到Model映射中。请注意，您还可以将XML文件作为加载`Resource`并使用它代替自定义DOM文档。

有可用的软件包自动“对象化”对象图，但是在Spring中，您可以完全灵活地以任何选择的方式从模型中创建DOM。这样可以防止XML转换在模型数据的结构中扮演过重要的角色，这在使用工具管理DOMification流程时是一种危险。

##### 转型

最后，`XsltViewResolver`解析“原始” XSLT模板文件并将DOM文档合并到其中以生成我们的视图。如`XsltViewResolver` 配置中所示，XSLT模板`war`位于`WEB-INF/xsl`目录中的文件中，并以`xslt`文件扩展名结尾。

以下示例显示了XSLT转换：

```xml
<?xml version="1.0" encoding="utf-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

    <xsl:output method="html" omit-xml-declaration="yes"/>

    <xsl:template match="/">
        <html>
            <head><title>Hello!</title></head>
            <body>
                <h1>My First Words</h1>
                <ul>
                    <xsl:apply-templates/>
                </ul>
            </body>
        </html>
    </xsl:template>

    <xsl:template match="word">
        <li><xsl:value-of select="."/></li>
    </xsl:template>

</xsl:stylesheet>
```

前面的转换呈现为以下HTML：

```html
<html>
    <head>
        <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Hello!</title>
    </head>
    <body>
        <h1>My First Words</h1>
        <ul>
            <li>Hello</li>
            <li>Spring</li>
            <li>Framework</li>
        </ul>
    </body>
</html>
```

### 1.11。MVC配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config)

MVC Java配置和MVC XML名称空间提供适用于大多数应用程序的默认配置以及用于自定义它的配置API。

有关配置API中不可用的更多高级定制，请参阅[Advanced Java Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-advanced-java)和[Advanced XML Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-config-advanced-xml)。

您不需要了解由MVC Java配置和MVC名称空间创建的基础bean。如果要了解更多信息，请参见[特殊Bean类型](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet-special-bean-types) 和[Web MVC Config](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet-config)。

#### 1.11.1。启用MVC配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-enable)

在Java配置中，可以使用`@EnableWebMvc`注释启用MVC配置，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig {
}
```

在XML配置中，可以使用``元素来启用MVC配置，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>

</beans>
```

前面的示例注册了许多Spring MVC [基础结构Bean，](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet-special-bean-types)并适应了类路径上可用的依赖项（例如，JSON，XML等的有效负载转换器）。

#### 1.11.2。MVC Config API

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-customize)

在Java配置中，您可以实现该`WebMvcConfigurer`接口，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    // Implement configuration methods...
}
```

在XML中，您可以检查的属性和子元素``。您可以查看[Spring MVC XML模式](https://schema.spring.io/mvc/spring-mvc.xsd)或使用IDE的代码完成功能来发现可用的属性和子元素。

#### 1.11.3。类型转换

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-conversion)

默认情况下，将安装各种数字和日期类型的格式化程序，并支持通过`@NumberFormat`和`@DateTimeFormat`在字段上进行自定义。

要在Java配置中注册自定义格式器和转换器，请使用以下命令：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        // ...
    }
}
```

要在XML配置中执行相同的操作，请使用以下命令：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven conversion-service="conversionService"/>

    <bean id="conversionService"
            class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
        <property name="converters">
            <set>
                <bean class="org.example.MyConverter"/>
            </set>
        </property>
        <property name="formatters">
            <set>
                <bean class="org.example.MyFormatter"/>
                <bean class="org.example.MyAnnotationFormatterFactory"/>
            </set>
        </property>
        <property name="formatterRegistrars">
            <set>
                <bean class="org.example.MyFormatterRegistrar"/>
            </set>
        </property>
    </bean>

</beans>
```

默认情况下，Spring MVC在解析和格式化日期值时会考虑请求区域设置。这适用于使用“输入”表单字段将日期表示为字符串的表单。但是，对于“日期”和“时间”表单字段，浏览器使用HTML规范中定义的固定格式。在这种情况下，日期和时间格式可以按以下方式自定义：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        DateTimeFormatterRegistrar registrar = new DateTimeFormatterRegistrar();
        registrar.setUseIsoFormat(true);
        registrar.registerFormatters(registry);
    }
}
```

|      | 见[的`FormatterRegistrar`SPI](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#format-FormatterRegistrar-SPI) 和`FormattingConversionServiceFactoryBean`何时使用FormatterRegistrar实现以获取更多信息。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.11.4。验证方式

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-validation)

默认情况下，如果[Bean验证](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validation-beanvalidation-overview)存在于类路径中（例如，Hibernate Validator），则将`LocalValidatorFactoryBean`其注册为全局[验证器](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#validator)，以`@Valid`与 `Validated`控制器方法参数一起使用。

在Java配置中，您可以自定义全局`Validator`实例，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        // ...
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven validator="globalValidator"/>

</beans>
```

请注意，您还可以`Validator`在本地注册实现，如以下示例所示：



 

```java
@Controller
public class MyController {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.addValidators(new FooValidator());
    }
}
```

|      | 如果需要在`LocalValidatorFactoryBean`某个位置进行注入，请创建一个bean并标记为Bean，`@Primary`以避免与MVC配置中声明的bean 发生冲突。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.11.5。拦截器

在Java配置中，您可以注册拦截器以应用于传入的请求，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LocaleChangeInterceptor());
        registry.addInterceptor(new ThemeChangeInterceptor()).addPathPatterns("/**").excludePathPatterns("/admin/**");
        registry.addInterceptor(new SecurityInterceptor()).addPathPatterns("/secure/*");
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor"/>
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <mvc:exclude-mapping path="/admin/**"/>
        <bean class="org.springframework.web.servlet.theme.ThemeChangeInterceptor"/>
    </mvc:interceptor>
    <mvc:interceptor>
        <mvc:mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 1.11.6。内容类型

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-content-negotiation)

您可以配置Spring MVC如何根据请求确定请求的媒体类型（例如，`Accept`标头，URL路径扩展，查询参数等）。

默认情况下，URL路径扩展首先检查-有`json`，`xml`，`rss`，并`atom` 注册为已知扩展名（视路径依赖）。的`Accept`报头检查第二。

考虑将这些默认值更改为`Accept`仅标头，并且，如果必须使用基于URL的内容类型解析，请考虑对路径扩展使用查询参数策略。有关更多详细信息，请参见 [后缀匹配](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)和[后缀匹配以及RFD](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-requestmapping-rfd)。

在Java配置中，您可以自定义请求的内容类型解析，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
        configurer.mediaType("json", MediaType.APPLICATION_JSON);
        configurer.mediaType("xml", MediaType.APPLICATION_XML);
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager"/>

<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="mediaTypes">
        <value>
            json=application/json
            xml=application/xml
        </value>
    </property>
</bean>
```

#### 1.11.7。讯息转换器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-message-codecs)

您可以`HttpMessageConverter`通过覆盖[`configureMessageConverters()`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html#configureMessageConverters-java.util.List-) （替换由Spring MVC创建的默认转换器）或覆盖[`extendMessageConverters()`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/config/annotation/WebMvcConfigurer.html#extendMessageConverters-java.util.List-) （定制默认转换器或向默认转换器添加其他转换器） 在Java配置中 进行自定义。

以下示例使用自定义的`ObjectMapper`而不是默认的添加了XML和Jackson JSON转换器 ：



 

```java
@Configuration
@EnableWebMvc
public class WebConfiguration implements WebMvcConfigurer {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        Jackson2ObjectMapperBuilder builder = new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .dateFormat(new SimpleDateFormat("yyyy-MM-dd"))
                .modulesToInstall(new ParameterNamesModule());
        converters.add(new MappingJackson2HttpMessageConverter(builder.build()));
        converters.add(new MappingJackson2XmlHttpMessageConverter(builder.createXmlMapper(true).build()));
    }
}
```

在前面的例子中， [`Jackson2ObjectMapperBuilder`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/http/converter/json/Jackson2ObjectMapperBuilder.html) 用于创建两种共同的构成`MappingJackson2HttpMessageConverter`和 `MappingJackson2XmlHttpMessageConverter`与缩进启用，定制的日期格式，和登记 [`jackson-module-parameter-names`](https://github.com/FasterXML/jackson-module-parameter-names)，这增加了用于访问参数名称（在Java中8增加了一个功能）的支持。

此构建器自定义Jackson的默认属性，如下所示：

- [`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/DeserializationFeature.html#FAIL_ON_UNKNOWN_PROPERTIES) 被禁用。
- [`MapperFeature.DEFAULT_VIEW_INCLUSION`](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html#DEFAULT_VIEW_INCLUSION) 被禁用。

如果在类路径中检测到以下知名模块，它还将自动注册以下知名模块：

- [jackson-datatype-joda](https://github.com/FasterXML/jackson-datatype-joda)：支持Joda-Time类型。
- [jackson-datatype-jsr310](https://github.com/FasterXML/jackson-datatype-jsr310)：支持Java 8日期和时间API类型。
- [jackson-datatype-jdk8](https://github.com/FasterXML/jackson-datatype-jdk8)：支持其他Java 8类型，例如`Optional`。
- [`jackson-module-kotlin`](https://github.com/FasterXML/jackson-module-kotlin)：支持Kotlin类和数据类。

|      | 使用Jackson的XML支持启用缩进[`woodstox-core-asl`](https://search.maven.org/#search|gav|1|g%3A"org.codehaus.woodstox" AND a%3A"woodstox-core-asl") 除了需要依赖之外，还需要 依赖[`jackson-dataformat-xml`](https://search.maven.org/#search\|ga\|1\|a%3A"jackson-dataformat-xml")。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

其他有趣的Jackson模块也可用：

- [jackson-datatype-money](https://github.com/zalando/jackson-datatype-money)：支持`javax.money`类型（非官方模块）。
- [jackson-datatype-hibernate](https://github.com/FasterXML/jackson-datatype-hibernate)：支持特定于Hibernate的类型和属性（包括延迟加载方面）。

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
            <property name="objectMapper" ref="objectMapper"/>
        </bean>
        <bean class="org.springframework.http.converter.xml.MappingJackson2XmlHttpMessageConverter">
            <property name="objectMapper" ref="xmlMapper"/>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>

<bean id="objectMapper" class="org.springframework.http.converter.json.Jackson2ObjectMapperFactoryBean"
      p:indentOutput="true"
      p:simpleDateFormat="yyyy-MM-dd"
      p:modulesToInstall="com.fasterxml.jackson.module.paramnames.ParameterNamesModule"/>

<bean id="xmlMapper" parent="objectMapper" p:createXmlMapper="true"/>
```

#### 1.11.8。查看控制器

这是用于定义的快捷方式，该快捷方式`ParameterizableViewController`在调用时立即转发到视图。在视图生成响应之前没有Java控制器逻辑要执行的静态情况下，可以使用它。

以下Java配置示例将请求转发`/`到名为的视图`home`：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

以下示例通过使用``元素，实现了与上一示例相同的操作，但使用XML ：

```xml
<mvc:view-controller path="/" view-name="home"/>
```

如果将`@RequestMapping`方法映射到任何HTTP方法的URL，则视图控制器不能用于处理相同的URL。这是因为通过URL与带注释的控制器的匹配被视为端点所有权的足够有力的指示，因此可以将405（METHOD_NOT_ALLOWED），415（UNSUPPORTED_MEDIA_TYPE）或类似的响应发送给客户端，以帮助进行调试。因此，建议避免在带注释的控制器和视图控制器之间拆分URL处理。

#### 1.11.9。查看解析器

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-view-resolvers)

MVC配置简化了视图解析器的注册。

以下Java配置示例通过使用JSP和Jackson作为`View`JSON呈现的默认配置来配置内容协商视图解析：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.jsp();
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:jsp/>
</mvc:view-resolvers>
```

但是请注意，FreeMarker，Tiles，Groovy标记和脚本模板也需要配置基础视图技术。

MVC名称空间提供专用元素。以下示例适用于FreeMarker：

```xml
<mvc:view-resolvers>
    <mvc:content-negotiation>
        <mvc:default-views>
            <bean class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>
        </mvc:default-views>
    </mvc:content-negotiation>
    <mvc:freemarker cache="false"/>
</mvc:view-resolvers>

<mvc:freemarker-configurer>
    <mvc:template-loader-path location="/freemarker"/>
</mvc:freemarker-configurer>
```

在Java配置中，您可以添加相应的`Configurer`bean，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.enableContentNegotiation(new MappingJackson2JsonView());
        registry.freeMarker().cache(false);
    }

    @Bean
    public FreeMarkerConfigurer freeMarkerConfigurer() {
        FreeMarkerConfigurer configurer = new FreeMarkerConfigurer();
        configurer.setTemplateLoaderPath("/freemarker");
        return configurer;
    }
}
```

#### 1.11.10。静态资源

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-static-resources)

此选项提供了一种方便的方法来从[`Resource`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/core/io/Resource.html)基于位置的列表中提供静态资源 。

在下一个示例中，给定一个以开头的请求，`/resources`相对路径用于相对于`/public`Web应用程序根目录下或之下的类路径查找和提供静态资源`/static`。这些资源的使用期限为一年，以确保最大程度地利用浏览器缓存并减少浏览器发出的HTTP请求。`Last-Modified`标头也将被评估，如果存在，`304` 则返回状态码。

以下清单显示了如何使用Java配置进行操作：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
            .addResourceLocations("/public", "classpath:/static/")
            .setCachePeriod(31556926);
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:resources mapping="/resources/**"
    location="/public, classpath:/static/"
    cache-period="31556926" />
```

另请参见 [HTTP缓存对静态资源的支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-caching-static-resources)。

资源处理程序还支持一系列 [`ResourceResolver`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/resource/ResourceResolver.html)实现和 [`ResourceTransformer`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/resource/ResourceTransformer.html)实现，您可以使用它们来创建用于处理优化资源的工具链。

您可以`VersionResourceResolver`根据从内容，固定应用程序版本或其他内容计算得出的MD5哈希值，使用for版本资源URL。一 `ContentVersionStrategy`（MD5哈希）是一个不错的选择-有一些明显的例外，如与模块加载程序使用的JavaScript资源。

以下示例显示了如何`VersionResourceResolver`在Java配置中使用：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**")
                .addResourceLocations("/public/")
                .resourceChain(true)
                .addResolver(new VersionResourceResolver().addContentVersionStrategy("/**"));
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:resources mapping="/resources/**" location="/public/">
    <mvc:resource-chain resource-cache="true">
        <mvc:resolvers>
            <mvc:version-resolver>
                <mvc:content-version-strategy patterns="/**"/>
            </mvc:version-resolver>
        </mvc:resolvers>
    </mvc:resource-chain>
</mvc:resources>
```

然后，您可以`ResourceUrlProvider`用来重写URL并应用完整的解析器和转换器链-例如，插入版本。MVC配置提供了一个`ResourceUrlProvider` bean，以便可以将其注入其他对象。您也可以使用`ResourceUrlEncodingFilter`Thymeleaf，JSP，FreeMarker以及其他依赖于URL标签的URL 使重写透明 `HttpServletResponse#encodeURL`。

请注意，在同时使用和`EncodedResourceResolver`（例如，用于提供压缩或brotli编码的资源）和时`VersionResourceResolver`，必须按此顺序注册它们。这样可以确保始终基于未编码文件可靠地计算基于内容的版本。

`WebJarsResourceResolver`当`org.webjars:webjars-locator-core`类库中存在库时，也会通过 自动注册 [WebJars](https://www.webjars.org/documentation)。解析程序可以重写URL以包括jar的版本，还可以与没有版本的传入URL进行匹配，例如from `/jquery/jquery.min.js`到 `/jquery/1.2.0/jquery.min.js`。

#### 1.11.11。默认Servlet

Spring MVC允许映射`DispatcherServlet`到`/`（从而覆盖了容器默认Servlet的映射），同时仍然允许容器默认Servlet处理静态资源请求。它配置的 `DefaultServletHttpRequestHandler`URL映射为`/**`，相对于其他URL映射具有最低的优先级。

该处理程序将所有请求转发到默认Servlet。因此，它必须按所有其他URL的顺序保留在最后`HandlerMappings`。如果使用，就是这种情况``。另外，如果你设置你自己定制的`HandlerMapping`情况下，一定要设置其`order`属性的值比的降低`DefaultServletHttpRequestHandler`，这是`Integer.MAX_VALUE`。

以下示例显示如何使用默认设置启用功能：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:default-servlet-handler/>
```

覆盖`/`Servlet映射的警告是，`RequestDispatcher`必须通过名称而不是通过路径来检索默认Servlet的。在 `DefaultServletHttpRequestHandler`尝试自动检测默认的Servlet在启动时的容器，使用大多数主要的Servlet容器（包括软件Tomcat，Jetty的GlassFish，JBoss和树脂中，WebLogic和WebSphere）已知名称的列表。如果已使用其他名称自定义配置了默认Servlet，或者在默认Servlet名称未知的情况下使用了不同的Servlet容器，则必须显式提供默认Servlet的名称，如以下示例所示：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable("myCustomDefaultServlet");
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:default-servlet-handler default-servlet-name="myCustomDefaultServlet"/>
```

#### 1.11.12。路径匹配

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-path-matching)

您可以自定义与URL的路径匹配和处理有关的选项。有关各个选项的详细信息，请参见 [`PathMatchConfigurer`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/servlet/config/annotation/PathMatchConfigurer.html)javadoc。

以下示例显示了如何在Java配置中自定义路径匹配：



 

```java
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        configurer
            .setUseTrailingSlashMatch(false)
            .setUseRegisteredSuffixPatternMatch(true)
            .setPathMatcher(antPathMatcher())
            .setUrlPathHelper(urlPathHelper())
            .addPathPrefix("/api", HandlerTypePredicate.forAnnotation(RestController.class));
    }

    @Bean
    public UrlPathHelper urlPathHelper() {
        //...
    }

    @Bean
    public PathMatcher antPathMatcher() {
        //...
    }
}
```

以下示例显示了如何在XML中实现相同的配置：

```xml
<mvc:annotation-driven>
    <mvc:path-matching
        trailing-slash="false"
        registered-suffixes-only="true"
        path-helper="pathHelper"
        path-matcher="pathMatcher"/>
</mvc:annotation-driven>

<bean id="pathHelper" class="org.example.app.MyPathHelper"/>
<bean id="pathMatcher" class="org.example.app.MyPathMatcher"/>
```

#### 1.11.13。高级Java配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-config-advanced-java)

`@EnableWebMvc`进口`DelegatingWebMvcConfiguration`，其中：

- 为Spring MVC应用程序提供默认的Spring配置
- 检测并委托给`WebMvcConfigurer`实现以自定义该配置。

对于高级模式，您可以`@EnableWebMvc`直接从中删除和扩展 `DelegatingWebMvcConfiguration`而不是实现`WebMvcConfigurer`，如以下示例所示：



 

```java
@Configuration
public class WebConfig extends DelegatingWebMvcConfiguration {

    // ...
}
```

您可以将现有方法保留在中`WebConfig`，但是现在您还可以覆盖基类中的bean声明，并且`WebMvcConfigurer`在类路径上仍然可以具有许多其他实现。

#### 1.11.14。高级XML配置

MVC命名空间没有高级模式。如果您需要在bean上自定义一个不能更改的属性，则可以使用`BeanPostProcessor`Spring 的生命周期挂钩`ApplicationContext`，如以下示例所示：



 

```java
@Component
public class MyPostProcessor implements BeanPostProcessor {

    public Object postProcessBeforeInitialization(Object bean, String name) throws BeansException {
        // ...
    }
}
```

请注意，您需要以`MyPostProcessor`XML形式显式声明或通过``声明使其被检测为bean 。

### 1.12。HTTP / 2

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-http2)

需要Servlet 4容器支持HTTP / 2，并且Spring Framework 5与Servlet API 4兼容。从编程模型的角度来看，应用程序不需要做任何特定的事情。但是，有一些与服务器配置有关的注意事项。有关更多详细信息，请参见 [HTTP / 2 Wiki页面](https://github.com/spring-projects/spring-framework/wiki/HTTP-2-support)。

Servlet API确实公开了一种与HTTP / 2相关的构造。您可以使用 `javax.servlet.http.PushBuilder`主动向客户端推送资源的方式，并且它作为[方法的方法参数](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-arguments)而受支持`@RequestMapping`。

## 2. REST客户端

本节描述了客户端对REST端点的访问选项。

### 2.1。 `RestTemplate`

`RestTemplate`是执行HTTP请求的同步客户端。它是原始的Spring REST客户端，并在基础HTTP客户端库上公开了简单的模板方法API。

|      | 从5.0版本开始，`RestTemplate`它处于维护模式，以后只有很小的更改和错误请求被接受。请考虑使用提供更现代API并支持同步，异步和流方案的 [WebClient](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-client)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

有关详细信息，请参见[REST端点](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/integration.html#rest-client-access)。

### 2.2。 `WebClient`

`WebClient`是执行HTTP请求的非阻塞，反应式客户端。它是在5.0中引入的，提供了的现代替代方案`RestTemplate`，并有效支持同步和异步以及流方案。

与相比`RestTemplate`，`WebClient`支持以下内容：

- 非阻塞I / O。
- 反应性产生背压。
- 高并发，硬件资源更少。
- 利用Java 8 lambda的功能风格，流畅的API。
- 同步和异步交互。
- 从服务器向上流或向下流。

有关更多详细信息，请参见[WebClient](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-client)。

## 3.测试

[与Spring WebFlux相同](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-test)

本节总结了`spring-test`Spring MVC应用程序中可用的选项。

- Servlet API模拟：用于单元测试控制器，过滤器和其他Web组件的Servlet API合约的模拟实现。有关 更多详细信息，请参见[Servlet API](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html#mock-objects-servlet)模拟对象。
- TestContext Framework：支持在JUnit和TestNG测试中加载Spring配置，包括跨测试方法高效地缓存已加载的配置，并支持通过加载`WebApplicationContext`a `MockServletContext`。有关更多详细信息，请参见[TestContext Framework](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html#testcontext-framework)。
- Spring MVC Test：一种框架，也称为`MockMvc`，用于通过`DispatcherServlet`（即支持注释）测试带注释的控制器，该框架具有Spring MVC基础结构，但没有HTTP服务器。有关更多详细信息，请参见[Spring MVC Test](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html#spring-mvc-test-framework)。
- 客户端REST：`spring-test`提供一个`MockRestServiceServer`，您可以用作模拟服务器来测试内部使用的客户端代码`RestTemplate`。有关更多详细信息，请参见[客户端REST测试](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html#spring-mvc-test-client)。
- `WebTestClient`：专门用于测试WebFlux应用程序，但也可以用于通过HTTP连接到任何服务器的端到端集成测试。它是一个无阻塞的反应式客户端，非常适合测试异步和流传输方案。

## 4. WebSockets

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-websocket)

参考文档的此部分涵盖对Servlet堆栈的支持，包括原始WebSocket交互的WebSocket消息传递，通过SockJS进行WebSocket仿真以及通过STOMP作为WebSocket的子协议进行发布-订阅消息传递。

### 4.1。WebSocket介绍

WebSocket协议[RFC 6455](https://tools.ietf.org/html/rfc6455)提供了一种标准化方法，可通过单个TCP连接在客户端和服务器之间建立全双工双向通信通道。它是与HTTP不同的TCP协议，但旨在通过端口80和443通过HTTP工作，并允许重复使用现有的防火墙规则。

WebSocket交互始于一个HTTP请求，该请求使用HTTP `Upgrade`标头进行升级，或在这种情况下切换到WebSocket协议。以下示例显示了这种交互：

```yaml
GET /spring-websocket-portfolio/portfolio HTTP/1.1
Host: localhost:8080
Upgrade: websocket 
Connection: Upgrade 
Sec-WebSocket-Key: Uc9l9TMkWGbHFD2qnFHltg==
Sec-WebSocket-Protocol: v10.stomp, v11.stomp
Sec-WebSocket-Version: 13
Origin: http://localhost:8080
```

|      | 该`Upgrade`头。     |
| ---- | ------------------- |
|      | 使用`Upgrade`连接。 |

具有WebSocket支持的服务器代替通常的200状态代码，返回类似于以下内容的输出：

```yaml
HTTP/1.1 101 Switching Protocols 
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: 1qVdfYHU9hPOl4JYYNXF623Gzn0=
Sec-WebSocket-Protocol: v10.stomp
```

|      | 协议切换 |
| ---- | -------- |
|      |          |

成功握手后，HTTP升级请求的基础TCP套接字将保持打开状态，客户端和服务器均可继续发送和接收消息。

WebSockets的工作原理的完整介绍超出了本文档的范围。请参阅RFC 6455，HTML5的WebSocket章节或Web上的许多简介和教程中的任何一篇。

请注意，如果WebSocket服务器在Web服务器（例如nginx）后面运行，则可能需要对其进行配置，以将WebSocket升级请求传递到WebSocket服务器。同样，如果应用程序在云环境中运行，请检查与WebSocket支持相关的云提供商的说明。

#### 4.1.1。HTTP与WebSocket

尽管WebSocket被设计为与HTTP兼容并以HTTP请求开头，但重要的是要了解这两个协议会导致非常不同的体系结构和应用程序编程模型。

在HTTP和REST中，应用程序被建模为许多URL。为了与应用程序交互，客户端访问那些URL，即请求-响应样式。服务器根据HTTP URL，方法和标头将请求路由到适当的处理程序。

相比之下，在WebSockets中，通常只有一个URL用于初始连接。随后，所有应用程序消息都在同一TCP连接上流动。这指向了一个完全不同的异步，事件驱动的消息传递体系结构。

WebSocket也是一种低级传输协议，与HTTP不同，它不对消息的内容规定任何语义。这意味着除非客户端和服务器就消息语义达成一致，否则就无法路由或处理消息。

WebSocket客户端和服务器可以通过`Sec-WebSocket-Protocol`HTTP握手请求上的标头协商使用更高级别的消息传递协议（例如STOMP）。在这种情况下，他们需要提出自己的约定。

#### 4.1.2。何时使用WebSockets

WebSockets可以使网页具有动态性和交互性。但是，在许多情况下，结合使用Ajax和HTTP流或长时间轮询可以提供一种简单有效的解决方案。

例如，新闻，邮件和社交订阅源需要动态更新，但是每几分钟进行一次更新可能是完全可以的。另一方面，协作，游戏和金融应用程序需要更接近实时。

仅延迟并不是决定因素。如果消息量相对较少（例如，监视网络故障），则HTTP流或轮询可以提供有效的解决方案。低延迟，高频率和高音量的结合才是使用WebSocket的最佳案例。

还请记住，在Internet上，控件之外的限制性代理可能会阻止WebSocket交互，这可能是因为未将它们配置为传递 `Upgrade`标头，或者是因为它们关闭了长期处于空闲状态的连接。这意味着与面向公众的应用程序相比，将WebSocket用于防火墙内部的应用程序是一个更直接的决定。

### 4.2。WebSocket API

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-websocket-server)

Spring框架提供了一个WebSocket API，可用于编写处理WebSocket消息的客户端和服务器端应用程序。

#### 4.2.1。 `WebSocketHandler`

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-websocket-server-handler)

创建WebSocket服务器就像实现一样简单，`WebSocketHandler`或者很可能扩展`TextWebSocketHandler`或`BinaryWebSocketHandler`。以下示例使用`TextWebSocketHandler`：

```java
import org.springframework.web.socket.WebSocketHandler;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.TextMessage;

public class MyHandler extends TextWebSocketHandler {

    @Override
    public void handleTextMessage(WebSocketSession session, TextMessage message) {
        // ...
    }

}
```

有专用的WebSocket Java配置和XML名称空间支持，用于将前面的WebSocket处理程序映射到特定的URL，如以下示例所示：

```java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

前面的示例用于Spring MVC应用程序，并且应包含在的配置中[`DispatcherServlet`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet)。但是，Spring的WebSocket支持不依赖于Spring MVC。`WebSocketHandler`在的帮助下，将a集成到其他HTTP服务环境中 相对简单[`WebSocketHttpRequestHandler`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/socket/server/support/WebSocketHttpRequestHandler.html)。

当`WebSocketHandler`直接使用API或间接使用API（例如通过 [STOMP](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp)消息传递）时，由于基础标准WebSocket会话（JSR-356）不允许并发发送，因此应用程序必须同步消息的发送。一种选择是将包裹`WebSocketSession`起来 [`ConcurrentWebSocketSessionDecorator`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/socket/handler/ConcurrentWebSocketSessionDecorator.html)。

#### 4.2.2。WebSocket握手

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-websocket-server-handshake)

定制初始HTTP WebSocket握手请求的最简单方法是通过`HandshakeInterceptor`，它公开了“握手之前”和“握手之后”的方法。您可以使用此类拦截器来阻止握手或使的任何属性可用`WebSocketSession`。以下示例使用内置的拦截器将HTTP会话属性传递到WebSocket会话：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(new MyHandler(), "/myHandler")
            .addInterceptors(new HttpSessionHandshakeInterceptor());
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:handshake-interceptors>
            <bean class="org.springframework.web.socket.server.support.HttpSessionHandshakeInterceptor"/>
        </websocket:handshake-interceptors>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

一个更高级的选项是扩展，`DefaultHandshakeHandler`它执行WebSocket握手的步骤，包括验证客户端起源，协商子协议以及其他详细信息。如果应用程序需要配置自定义项`RequestUpgradeStrategy`以适应尚不支持的WebSocket服务器引擎和版本，则它可能还需要使用此选项（有关此主题的更多信息，请参阅[部署](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-server-deployment)）。Java配置和XML名称空间都可以配置自定义 `HandshakeHandler`。

|      | Spring提供了一个`WebSocketHandlerDecorator`基类，您可以使用它来装饰`WebSocketHandler`额外的行为。使用WebSocket Java配置或XML名称空间时，默认情况下会提供并添加日志记录和异常处理实现。在`ExceptionWebSocketHandlerDecorator`捕获了由任何出现的所有捕获的异常`WebSocketHandler`的方法，并关闭与状态WebSocket的会议`1011`，这表明服务器错误。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.2.3。部署方式

Spring WebSocket API易于集成到Spring MVC应用程序中，在该应用程序中`DispatcherServlet`可以同时服务HTTP WebSocket握手和其他HTTP请求。通过调用也很容易集成到其他HTTP处理方案中`WebSocketHttpRequestHandler`。这是方便且易于理解的。但是，对于JSR-356运行时，需要特别注意。

Java WebSocket API（JSR-356）提供了两种部署机制。第一个涉及启动时的Servlet容器类路径扫描（Servlet 3功能）。另一个是在Servlet容器初始化时使用的注册API。这两种机制都无法将单个“前端控制器”用于所有HTTP处理（包括WebSocket握手和所有其他HTTP请求）（例如Spring MVC）`DispatcherServlet`。

这是JSR-356的一个重大限制，`RequestUpgradeStrategy`即使在JSR-356运行时中运行，Spring的WebSocket支持也可以通过服务器特定的实现来解决。Tomcat，Jetty，GlassFish，WebLogic，WebSphere和Undertow（和WildFly）目前存在此类策略。

|      | 已经创建了克服Java WebSocket API中的上述限制的请求，可以在[eclipse-ee4j / websocket-api＃211中](https://github.com/eclipse-ee4j/websocket-api/issues/211)进行跟踪 。Tomcat，Undertow和WebSphere提供了自己的API替代方案，使之可以做到这一点，而Jetty也可以实现。我们希望更多的服务器可以做到这一点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

另一个要考虑的因素是，期望具有JSR-356支持的Servlet容器执行`ServletContainerInitializer`（SCI）扫描，这可能会减慢应用程序的启动速度-在某些情况下会大大降低。如果在升级到支持JSR-356的Servlet容器版本后观察到重大影响，则应该可以通过使用中的``元素来选择性地启用或禁用Web片段（和SCI扫描）`web.xml`，如以下示例所示：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        https://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering/>

</web-app>
```

然后，您可以按名称有选择地启用Web片段，例如Spring自己的片段， `SpringServletContainerInitializer`它提供对Servlet 3 Java初始化API的支持。以下示例显示了如何执行此操作：

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://java.sun.com/xml/ns/javaee
        https://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">

    <absolute-ordering>
        <name>spring_web</name>
    </absolute-ordering>

</web-app>
```

#### 4.2.4。服务器配置

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-websocket-server-config)

每个基础WebSocket引擎都公开配置属性，这些属性控制运行时特征，例如消息缓冲区大小的大小，空闲超时等。

对于Tomcat，WildFly和GlassFish，可以将A添加`ServletServerContainerFactoryBean`到WebSocket Java配置中，如以下示例所示：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        return container;
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <bean class="org.springframework...ServletServerContainerFactoryBean">
        <property name="maxTextMessageBufferSize" value="8192"/>
        <property name="maxBinaryMessageBufferSize" value="8192"/>
    </bean>

</beans>
```

|      | 对于客户端WebSocket配置，应使用`WebSocketContainerFactoryBean` （XML）或`ContainerProvider.getWebSocketContainer()`（Java配置）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

对于Jetty，您需要提供一个预先配置的Jetty，`WebSocketServerFactory`然后`DefaultHandshakeHandler`通过WebSocket Java配置将其插入Spring 。以下示例显示了如何执行此操作：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(echoWebSocketHandler(),
            "/echo").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(
                new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/echo" handler="echoHandler"/>
        <websocket:handshake-handler ref="handshakeHandler"/>
    </websocket:handlers>

    <bean id="handshakeHandler" class="org.springframework...DefaultHandshakeHandler">
        <constructor-arg ref="upgradeStrategy"/>
    </bean>

    <bean id="upgradeStrategy" class="org.springframework...JettyRequestUpgradeStrategy">
        <constructor-arg ref="serverFactory"/>
    </bean>

    <bean id="serverFactory" class="org.eclipse.jetty...WebSocketServerFactory">
        <constructor-arg>
            <bean class="org.eclipse.jetty...WebSocketPolicy">
                <constructor-arg value="SERVER"/>
                <property name="inputBufferSize" value="8092"/>
                <property name="idleTimeout" value="600000"/>
            </bean>
        </constructor-arg>
    </bean>

</beans>
```

#### 4.2.5。允许的来源

[Web助焊剂](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web-reactive.html#webflux-websocket-server-cors)

从Spring Framework 4.1.5开始，WebSocket和SockJS的默认行为是仅接受同源请求。也可以允许所有或指定的来源列表。此检查主要用于浏览器客户端。没有任何措施可以阻止其他类型的客户端修改`Origin`标头值（有关更多详细信息，请参阅 [RFC 6454：Web Origin概念](https://tools.ietf.org/html/rfc6454)）。

三种可能的行为是：

- 仅允许相同来源的请求（默认）：在此模式下，启用SockJS时，Iframe HTTP响应标头`X-Frame-Options`设置为`SAMEORIGIN`，并且JSONP传输被禁用，因为它不允许检查请求的来源。因此，启用此模式时，不支持IE6和IE7。
- 允许指定来源列表：每个允许的来源必须以`http://` 或开头`https://`。在此模式下，启用SockJS时，将禁用IFrame传输。因此，启用此模式时，不支持IE6到IE9。
- 允许所有原点：要启用此模式，应提供`*`作为允许的原点值。在这种模式下，所有传输都可用。

您可以配置WebSocket和SockJS允许的来源，如以下示例所示：

```java
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").setAllowedOrigins("https://mydomain.com");
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers allowed-origins="https://mydomain.com">
        <websocket:mapping path="/myHandler" handler="myHandler" />
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

### 4.3。SockJS后备

在公共Internet上，控件之外的限制性代理可能会阻止WebSocket交互，这可能是因为未将它们配置为传递`Upgrade`标头，或者是因为它们关闭了长期处于空闲状态的连接。

解决此问题的方法是WebSocket仿真—也就是说，先尝试使用WebSocket，然后再尝试使用基于HTTP的技术来模拟WebSocket交互并公开相同的应用程序级API。

在Servlet堆栈上，Spring框架为SockJS协议提供服务器（以及客户端）支持。

#### 4.3.1。总览

SockJS的目标是让应用程序使用WebSocket API，但在运行时必要时使用非WebSocket替代方案，而无需更改应用程序代码。

SockJS包含：

- 所述[SockJS协议](https://github.com/sockjs/sockjs-protocol) 以可执行的形式定义的 [解说的测试](https://sockjs.github.io/sockjs-protocol/sockjs-protocol-0.3.3.html)。
- 该[SockJS JavaScript客户端](https://github.com/sockjs/sockjs-client/) -在浏览器中使用客户端库。
- SockJS服务器实现，包括Spring Framework `spring-websocket`模块中的一个。
- `spring-websocket`模块中的SockJS Java客户端（从4.1版开始）。

SockJS设计用于浏览器。它使用多种技术来支持各种浏览器版本。有关SockJS传输类型和浏览器的完整列表，请参见 [SockJS客户端](https://github.com/sockjs/sockjs-client/)页面。传输分为三大类：WebSocket，HTTP流和HTTP长轮询。有关这些类别的概述，请参阅 [此博客文章](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)。

SockJS客户端从发送开始以`GET /info`从服务器获取基本信息。在那之后，它必须决定使用哪种交通工具。如果可能，请使用WebSocket。如果不是，在大多数浏览器中，至少有一个HTTP流选项。如果不是，则使用HTTP（长）轮询。

所有传输请求都具有以下URL结构：

```
https：// host：port / myApp / myEndpoint / {server-id} / {session-id} / {transport}
```

哪里：

- `{server-id}` 对于在集群中路由请求很有用，但否则不使用。
- `{session-id}` 关联属于SockJS会话的HTTP请求。
- `{transport}`指示传输类型（例如，`websocket`，`xhr-streaming`，和其它物质）。

WebSocket传输仅需要单个HTTP请求即可进行WebSocket握手。此后所有消息在该套接字上交换。

HTTP传输需要更多请求。例如，Ajax / XHR流依赖于对服务器到客户端消息的一个长期运行的请求，以及对客户端到服务器消息的其他HTTP POST请求。长轮询类似于长轮询，只是它在每次服务器到客户端发送之后结束当前请求。

SockJS添加了最少的消息框架。例如，服务器`o` 最初发送字母（“打开”帧），消息以`a["message1","message2"]` （JSON编码数组）发送，`h`如果在25秒内没有消息流（默认情况下），则以字母（“心跳”帧）发送消息，以及字母`c`（“关闭”框架）以关闭会话。

要了解更多信息，请在浏览器中运行示例并查看HTTP请求。SockJS客户端允许固定传输列表，因此可以一次查看每个传输。SockJS客户端还提供了一个调试标志，该标志可在浏览器控制台中启用有用的消息。在服务器端，您可以启用的 `TRACE`日志记录`org.springframework.web.socket`。有关更多详细信息，请参见SockJS协议 [旁白测试](https://sockjs.github.io/sockjs-protocol/sockjs-protocol-0.3.3.html)。

#### 4.3.2。启用SockJS

您可以通过Java配置启用SockJS，如以下示例所示：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myHandler(), "/myHandler").withSockJS();
    }

    @Bean
    public WebSocketHandler myHandler() {
        return new MyHandler();
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:handlers>
        <websocket:mapping path="/myHandler" handler="myHandler"/>
        <websocket:sockjs/>
    </websocket:handlers>

    <bean id="myHandler" class="org.springframework.samples.MyHandler"/>

</beans>
```

前面的示例用于Spring MVC应用程序，并且应包含在的配置中[`DispatcherServlet`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-servlet)。但是，Spring的WebSocket和SockJS支持不依赖于Spring MVC。在的帮助下，将其集成到其他HTTP服务环境中相对简单 [`SockJsHttpRequestHandler`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/socket/sockjs/support/SockJsHttpRequestHandler.html)。

在浏览器端，应用程序可以使用 [`sockjs-client`](https://github.com/sockjs/sockjs-client/)（版本1.0.x）。它模拟W3C WebSocket API，并与服务器通信以选择最佳的传输选项，具体取决于运行它的浏览器。请参阅 [sockjs-client](https://github.com/sockjs/sockjs-client/)页面和浏览器支持的传输类型列表。客户端还提供了几个配置选项-例如，指定要包括的传输。

#### 4.3.3。IE 8和9

Internet Explorer 8和9仍在使用。它们是拥有SockJS的关键原因。本节涵盖有关在这些浏览器中运行的重要注意事项。

SockJS客户端通过使用Microsoft的，在IE 8和9中支持Ajax / XHR流 [`XDomainRequest`](https://blogs.msdn.com/b/ieinternals/archive/2010/05/13/xdomainrequest-restrictions-limitations-and-workarounds.aspx)。跨域有效，但不支持发送Cookie。Cookies对于Java应用程序通常是必不可少的。但是，由于SockJS客户端可用于多种服务器类型（不仅是Java服务器类型），因此需要知道cookie是否重要。如果是这样，则SockJS客户端更喜欢Ajax / XHR进行流传输。否则，它依赖于基于iframe的技术。

首先`/info`从SockJS客户端请求是针对可能影响客户的传输选择信息的请求。这些详细信息之一是服务器应用程序是否依赖Cookie（例如，出于身份验证目的或使用粘性会话进行群集）。Spring的SockJS支持包括名为的属性`sessionCookieNeeded`。由于大多数Java应用程序都依赖`JSESSIONID` cookie ，因此默认情况下启用此功能。如果您的应用程序不需要它，则可以关闭此选项，然后SockJS客户端应`xdr-streaming`在IE 8和9中进行选择。

如果你使用基于iframe的运输，记住，浏览器可以通过指示HTTP响应头设置为阻止特定网页上的使用iframe `X-Frame-Options`来`DENY`， `SAMEORIGIN`或`ALLOW-FROM `。这用于防止 [点击劫持](https://www.owasp.org/index.php/Clickjacking)。

|      | Spring Security 3.2+提供了`X-Frame-Options`对每个响应进行设置的支持。默认情况下，Spring Security Java配置将其设置为`DENY`。在3.2中，Spring Security XML名称空间默认情况下不设置该标头，但可以配置为这样做。将来，它可能会默认设置。有关 如何配置标头设置的详细信息，请参见Spring Security文档的[Default Security Headers](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#headers)`X-Frame-Options`。您也可以查看 [SEC-2501](https://jira.spring.io/browse/SEC-2501)以获取更多背景信息。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您的应用程序添加了`X-Frame-Options`响应标头（应如此！）并依赖于基于iframe的传输，则需要将标头值设置为 `SAMEORIGIN`或`ALLOW-FROM `。Spring SockJS支持还需要知道SockJS客户端的位置，因为它是从iframe加载的。默认情况下，iframe设置为从CDN位置下载SockJS客户端。最好将此选项配置为使用与应用程序源相同的URL。

以下示例显示了如何在Java配置中执行此操作：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS()
                .setClientLibraryUrl("http://localhost:8080/myapp/js/sockjs-client.js");
    }

    // ...

}
```

XML名称空间通过``元素提供了类似的选项。

|      | 在初始开发期间，请启用SockJS客户端`devel`模式，以防止浏览器缓存本应缓存的SockJS请求（如iframe）。有关如何启用它的详细信息，请参见 [SockJS客户端](https://github.com/sockjs/sockjs-client/)页面。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.3.4。心跳

SockJS协议要求服务器发送心跳消息，以防止代理断定连接已挂起。Spring SockJS配置具有一个名为的属性`heartbeatTime`，您可以使用该属性来自定义频率。默认情况下，假设没有通过该连接发送任何其他消息，则在25秒后发送心跳。这个25秒的值符合以下 [IETF](https://tools.ietf.org/html/rfc6202)对公共Internet应用程序的[建议](https://tools.ietf.org/html/rfc6202)。

|      | 在WebSocket和SockJS上使用STOMP时，如果STOMP客户端和服务器协商要交换的心跳，则会禁用SockJS心跳。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring SockJS支持还使您可以配置`TaskScheduler`来安排心跳任务。任务调度程序由线程池支持，其默认设置基于可用处理器的数量。您应该考虑根据您的特定需求自定义设置。

#### 4.3.5。客户端断开连接

HTTP流和HTTP长轮询SockJS传输要求连接保持打开状态的时间比平常长。有关这些技术的概述，请参见此 [博客文章](https://spring.io/blog/2012/05/08/spring-mvc-3-2-preview-techniques-for-real-time-updates/)。

在Servlet容器中，这是通过Servlet 3异步支持完成的，该支持允许退出Servlet容器线程，处理请求并继续写入另一个线程的响应。

一个特定的问题是，Servlet API不会为已离开的客户端提供通知。参见[eclipse-ee4j / servlet-api＃44](https://github.com/eclipse-ee4j/servlet-api/issues/44)。但是，Servlet容器在随后尝试写入响应时会引发异常。由于Spring的SockJS服务支持服务器发送的心跳信号（默认情况下每25秒发送一次），这意味着通常会在该时间段内（或更早，如果消息发送频率更高）检测到客户端断开连接。

|      | 结果，由于客户端已断开连接，可能会发生网络I / O故障，这可能会在日志中填充不必要的堆栈跟踪。Spring会尽最大努力找出代表客户端断开连接的网络故障（特定于每个服务器），并使用专用的日志类别`DISCONNECTED_CLIENT_LOG_CATEGORY` （在中定义`AbstractSockJsSession`）来记录最少的消息。如果需要查看堆栈跟踪，可以将该日志类别设置为TRACE。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.3.6。SockJS和CORS

如果您允许跨域请求（请参阅[Allowed Origins](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-server-allowed-origins)），则SockJS协议将CORS用于XHR流和轮询传输中的跨域支持。因此，除非检测到响应中存在CORS头，否则将自动添加CORS头。因此，如果已经将应用程序配置为提供CORS支持（例如，通过Servlet过滤器），则Spring会`SockJsService`跳过这一部分。

也可以通过`suppressCors`在Spring的SockJsService中设置属性来禁用这些CORS标头 的添加。

SockJS需要以下标头和值：

- `Access-Control-Allow-Origin`：从`Origin`请求标头的值初始化。
- `Access-Control-Allow-Credentials`：始终设置为`true`。
- `Access-Control-Request-Headers`：从等效请求标头中的值初始化。
- `Access-Control-Allow-Methods`：传输支持的HTTP方法（请参阅`TransportType`枚举）。
- `Access-Control-Max-Age`：设置为31536000（1年）。

有关确切的实现，请参阅`addCorsHeaders`中的`AbstractSockJsService`和`TransportType`源代码中的枚举。

另外，如果CORS配置允许，请考虑排除带有SockJS端点前缀的URL，从而让Spring来`SockJsService`处理。

#### 4.3.7。 `SockJsClient`

Spring提供了一个SockJS Java客户端，无需使用浏览器即可连接到远程SockJS端点。当需要通过公共网络在两个服务器之间进行双向通信时（这是网络代理可以阻止使用WebSocket协议的地方），这特别有用。SockJS Java客户端对于测试也非常有用（例如，模拟大量并发用户）。

该SockJS Java客户端支持`websocket`，`xhr-streaming`以及`xhr-polling` 运输。其余的仅在浏览器中使用才有意义。

您可以使用配置`WebSocketTransport`：

- `StandardWebSocketClient` 在JSR-356运行时中。
- `JettyWebSocketClient` 通过使用Jetty 9+本机WebSocket API。
- Spring的任何实现`WebSocketClient`。

根据`XhrTransport`定义，An 同时支持`xhr-streaming`和`xhr-polling`，因为从客户端的角度来看，除了用于连接服务器的URL之外没有其他区别。当前有两种实现：

- `RestTemplateXhrTransport`使用Spring的`RestTemplate`HTTP请求。
- `JettyXhrTransport``HttpClient`对HTTP请求使用Jetty 。

以下示例显示了如何创建SockJS客户端并连接到SockJS端点：

```java
List<Transport> transports = new ArrayList<>(2);
transports.add(new WebSocketTransport(new StandardWebSocketClient()));
transports.add(new RestTemplateXhrTransport());

SockJsClient sockJsClient = new SockJsClient(transports);
sockJsClient.doHandshake(new MyWebSocketHandler(), "ws://example.com:8080/sockjs");
```

|      | SockJS对消息使用JSON格式的数组。默认情况下，使用Jackson 2，并且必须在类路径上。或者，您可以配置的自定义实现 `SockJsMessageCodec`并在上进行配置`SockJsClient`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要用于`SockJsClient`模拟大量并发用户，您需要配置基础HTTP客户端（用于XHR传输）以允许足够数量的连接和线程。以下示例显示了如何使用Jetty进行操作：

```java
HttpClient jettyHttpClient = new HttpClient();
jettyHttpClient.setMaxConnectionsPerDestination(1000);
jettyHttpClient.setExecutor(new QueuedThreadPool(1000));
```

下面的示例显示了与服务器端SockJS相关的属性（有关详细信息，请参见javadoc），您还应该考虑自定义：

```java
@Configuration
public class WebSocketConfig extends WebSocketMessageBrokerConfigurationSupport {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/sockjs").withSockJS()
            .setStreamBytesLimit(512 * 1024) 
            .setHttpMessageCacheSize(1000) 
            .setDisconnectDelay(30 * 1000); 
    }

    // ...
}
```

|      | 将该`streamBytesLimit`属性设置为512KB（默认值为128KB —  `128 * 1024`）。 |
| ---- | ------------------------------------------------------------ |
|      | 将`httpMessageCacheSize`属性设置为1,000（默认值为`100`）。   |
|      | 将`disconnectDelay`属性设置为30属性秒（默认为5秒—  `5 * 1000`）。 |

### 4.4。脚踩

WebSocket协议定义了两种消息类型（文本消息和二进制消息），但是其内容未定义。该协议定义了一种机制，供客户端和服务器协商用于在WebSocket之上使用的子协议（即更高级别的消息协议），以定义每种协议可以发送的消息类型，格式，内容。每个消息，依此类推。子协议的使用是可选的，但无论哪种方式，客户端和服务器都需要就定义消息内容的某种协议达成共识。

#### 4.4.1。总览

[STOMP](https://stomp.github.io/stomp-specification-1.2.html#Abstract)（面向简单文本的消息协议）最初是为脚本语言（例如Ruby，Python和Perl）创建的，用于连接企业消息代理。它旨在解决常用消息传递模式的最小子集。STOMP可以在任何可靠的双向流网络协议上使用，例如TCP和WebSocket。尽管STOMP是面向文本的协议，但是消息有效负载可以是文本或二进制。

STOMP是基于帧的协议，其帧以HTTP为模型。以下清单显示了STOMP帧的结构：

```
命令
标头1：值1
标头2：值2

身体^ @
```

客户端可以使用`SEND`或`SUBSCRIBE`命令来发送或订阅消息，以及`destination`描述消息内容以及应由谁接收的标头。这启用了一种简单的发布-订阅机制，您可以使用该机制通过代理将消息发送到其他连接的客户端，或者将消息发送到服务器以请求执行某些工作。

当您使用Spring的STOMP支持时，Spring WebSocket应用程序将充当客户端的STOMP代理。消息被路由到`@Controller`消息处理方法或简单的内存中代理，该代理跟踪订阅并向订阅的用户广播消息。您还可以将Spring配置为与专用的STOMP代理（例如RabbitMQ，ActiveMQ和其他代理）一起使用，以实际广播消息。在那种情况下，Spring维护与代理的TCP连接，将消息中继到该代理，并将消息从该代理向下传递到已连接的WebSocket客户端。因此，Spring Web应用程序可以依靠基于HTTP的统一安全性，通用验证以及用于消息处理的熟悉的编程模型。

下面的示例显示了一个订阅以接收股票报价的客户端，服务器可能会定期发出该股票报价（例如，通过将消息通过a发送`SimpMessagingTemplate`给经纪人的计划任务）：

```
订阅
id：sub-1
目的地：/topic/price.stock.*

^ @
```

以下示例显示了一个客户端发送的交易请求，服务器可以通过一种`@MessageMapping`方法来处理该交易请求：

```
发送
目的地：/队列/贸易
内容类型：application / json
内容长度：44

{“ action”：“买”，“ ticker”：“ MMM”，“股份”，44} ^ @
```

执行后，服务器可以向客户广播交易确认消息和详细信息。

在STOMP规范中，目的地的含义是故意不透明的。它可以是任何字符串，并且完全由STOMP服务器定义它们支持的目的地的语义和语法。但是，目的地通常是类似路径的字符串，其中`/topic/..`包含发布-订阅（一对多）并`/queue/`暗示点对点（一对一）消息交换。

STOMP服务器可以使用该`MESSAGE`命令向所有订户广播消息。以下示例显示了服务器向订阅的客户端发送股票报价的服务器：

```
信息
讯息编号：nxahklf6-1
订阅：sub-1
目的地：/topic/price.stock.MMM

{“ ticker”：“ MMM”，“ price”：129.45} ^ @
```

服务器无法发送未经请求的消息。来自服务器的所有消息都必须响应特定的客户端订阅，并且 `subscription-id`服务器消息的`id`标题必须与客户端订阅的标题匹配。

前面的概述旨在提供对STOMP协议的最基本的了解。我们建议您全面阅读协议 [规范](https://stomp.github.io/stomp-specification-1.2.html)。

#### 4.4.2。好处

与使用原始WebSocket相比，使用STOMP作为子协议可以使Spring Framework和Spring Security提供更丰富的编程模型。关于HTTP与原始TCP以及它如何使Spring MVC和其他Web框架提供丰富的功能，可以得出相同的观点。以下是好处列表：

- 无需发明自定义消息协议和消息格式。
- 可以使用STOMP客户端，包括 Spring框架中的[Java客户端](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-client)。
- 您可以（可选）使用消息代理（例如RabbitMQ，ActiveMQ和其他代理）来管理订阅和广播消息。
- 可以在任意数量的`@Controller`实例中组织应用程序逻辑，并且可以根据STOMP目标标头将消息路由到它们，而`WebSocketHandler`对于给定的连接，可以使用单个消息处理原始WebSocket消息。
- 您可以使用Spring Security基于STOMP目的地和消息类型来保护消息。

#### 4.4.3。启用STOMP

`spring-messaging`和 `spring-websocket`模块中提供了STOMP over WebSocket支持。一旦有了这些依赖关系，就可以使用[SockJS Fallback](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback)通过WebSocket公开STOMP端点，如以下示例所示：

```java
import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
import org.springframework.web.socket.config.annotation.StompEndpointRegistry;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();  
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.setApplicationDestinationPrefixes("/app"); 
        config.enableSimpleBroker("/topic", "/queue"); 
    }
}
```

|      | `/portfolio` 是WebSocket（或SockJS）客户端需要连接到WebSocket握手的终结点的HTTP URL。 |
| ---- | ------------------------------------------------------------ |
|      | 目的标头以STOMP开头的STOMP消息`/app`将路由到 类中的`@MessageMapping`方法`@Controller`。 |
|      | 使用内置的消息代理进行订阅和广播，并将目标标头以其开头的消息路由`/topic `or `/queue`到代理。 |

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker application-destination-prefix="/app">
        <websocket:stomp-endpoint path="/portfolio">
            <websocket:sockjs/>
        </websocket:stomp-endpoint>
        <websocket:simple-broker prefix="/topic, /queue"/>
    </websocket:message-broker>

</beans>
```

|      | 对于内置的简单代理，`/topic`and `/queue`前缀没有任何特殊含义。它们仅是区分发布订阅与点对点消息传递（即许多订户与一个消费者）的约定。使用外部代理时，请检查代理的STOMP页面以了解其支持哪种STOMP目标和前缀。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要从浏览器进行连接，对于SockJS，您可以使用 [`sockjs-client`](https://github.com/sockjs/sockjs-client)。对于STOMP，许多应用程序都使用了[jmesnil / stomp-websocket](https://github.com/jmesnil/stomp-websocket)库（也称为stomp.js），该库功能齐全，已在生产中使用多年，但不再维护。目前， [JSteunou / webstomp-client](https://github.com/JSteunou/webstomp-client)是该库中最活跃且发展最快的后继程序。以下示例代码基于此：

```javascript
var socket = new SockJS("/spring-websocket-portfolio/portfolio");
var stompClient = webstomp.over(socket);

stompClient.connect({}, function(frame) {
}
```

另外，如果您通过WebSocket连接（没有SockJS），则可以使用以下代码：

```javascript
var socket = new WebSocket("/spring-websocket-portfolio/portfolio");
var stompClient = Stomp.over(socket);

stompClient.connect({}, function(frame) {
}
```

请注意，`stompClient`在前面的示例中不需要指定`login` 和`passcode`标头。即使这样做，它们也将在服务器端被忽略（或更确切地说，被覆盖）。有关[身份验证](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-authentication)的更多信息，请参见[连接到代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-handle-broker-relay-configure) 和[身份](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-authentication)验证。

有关更多示例代码，请参见：

- [使用WebSocket构建交互式Web应用程序](https://spring.io/guides/gs/messaging-stomp-websocket/) -入门指南。
- [股票投资组合](https://github.com/rstoyanchev/spring-websocket-portfolio) —一个示例应用程序。

#### 4.4.4。WebSocket服务器

要配置基础的WebSocket服务器，请应用“ [服务器配置”中](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-server-runtime-configuration)的信息 。对于Jetty，您需要通过`HandshakeHandler`和设置和：`WebSocketPolicy``StompEndpointRegistry`

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").setHandshakeHandler(handshakeHandler());
    }

    @Bean
    public DefaultHandshakeHandler handshakeHandler() {

        WebSocketPolicy policy = new WebSocketPolicy(WebSocketBehavior.SERVER);
        policy.setInputBufferSize(8192);
        policy.setIdleTimeout(600000);

        return new DefaultHandshakeHandler(
                new JettyRequestUpgradeStrategy(new WebSocketServerFactory(policy)));
    }
}
```

#### 4.4.5。消息流

公开STOMP端点后，Spring应用程序将成为已连接客户端的STOMP代理。本节描述服务器端的消息流。

该`spring-messaging`模块包含对起源于[Spring Integration的](https://spring.io/spring-integration)消息传递应用程序的基础支持，后来被提取并合并到Spring Framework中，以在许多[Spring项目](https://spring.io/projects)和应用程序场景中更广泛地使用 。下面的列表简要描述了一些可用的消息传递抽象：

- [消息](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/messaging/Message.html)：[消息的](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/messaging/Message.html)简单表示，包括标题和有效负载。
- [MessageHandler](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/messaging/MessageHandler.html)：处理消息的合同。
- [MessageChannel](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/messaging/MessageChannel.html)：用于发送使生产者和消费者之间松散耦合的消息的合同。
- [SubscribableChannel](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/messaging/SubscribableChannel.html)： `MessageChannel`具有`MessageHandler`订阅者。
- [ExecutorSubscribableChannel](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/messaging/support/ExecutorSubscribableChannel.html)： `SubscribableChannel`使用一个`Executor`邮件传递。

Java配置（即`@EnableWebSocketMessageBroker`）和XML名称空间配置（即``）都使用前面的组件来组装消息工作流。下图显示了启用简单内置消息代理时使用的组件：

![消息流简单代理](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/message-flow-simple-broker.png)

上图显示了三个消息通道：

- `clientInboundChannel`：用于传递从WebSocket客户端收到的消息。
- `clientOutboundChannel`：用于向WebSocket客户端发送服务器消息。
- `brokerChannel`：用于从服务器端应用程序代码内将消息发送到消息代理。

下图显示了将外部代理（例如RabbitMQ）配置为用于管理订阅和广播消息时使用的组件：

![消息流代理中继](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/images/message-flow-broker-relay.png)

前面两个图之间的主要区别是使用“代理中继”将消息通过TCP传递到外部STOMP代理，以及将消息从代理传递到订阅的客户端。

从WebSocket连接接收到消息后，消息将被解码为STOMP帧，转换为Spring `Message`表示形式，然后发送给 `clientInboundChannel`进行进一步处理。例如，目标标头以开头的STOMP消息`/app`可以路由到带`@MessageMapping`注释的控制器中的方法，而`/topic`和`/queue`消息可以直接路由到消息代理。

`@Controller`处理来自客户端的STOMP消息的注释可以通过将消息发送到消息代理`brokerChannel`，并且代理通过将该消息广播给匹配的订户`clientOutboundChannel`。相同的控制器还可以响应HTTP请求执行相同的操作，因此客户端可以执行HTTP POST，然后一种`@PostMapping`方法可以将消息发送到消息代理，以广播到订阅的客户端。

我们可以通过一个简单的示例跟踪流程。考虑以下示例，该示例设置了服务器：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio");
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        registry.enableSimpleBroker("/topic");
    }
}

@Controller
public class GreetingController {

    @MessageMapping("/greeting") {
    public String handle(String greeting) {
        return "[" + getTimestamp() + ": " + greeting;
    }
}
```

前面的示例支持以下流程：

1. 客户端连接到`http://localhost:8080/portfolio`WebSocket连接，并且一旦建立了WebSocket连接，STOMP帧就开始在其上流动。
2. 客户端发送SUBSCRIBE帧，目标标头为`/topic/greeting`。收到并解码后，该消息将发送到`clientInboundChannel`，然后路由到消息代理，该代理存储客户端订阅。
3. 客户端向发送一个aSEND帧`/app/greeting`。该`/app`前缀有助于它的路线注解控制器。`/app`除去前缀后，目标的其余`/greeting` 部分将映射到中的`@MessageMapping`方法`GreetingController`。
4. 从返回的值`GreetingController`转换为`Message`带有有效负载的Spring ，该有效负载基于返回值和默认的目标标头 `/topic/greeting`（从输入目标派生为`/app`替换为 `/topic`）。结果消息将发送到`brokerChannel`，并由消息代理处理。
5. 消息代理查找所有匹配的订户，并通过通过`clientOutboundChannel`，向每个发送一个MESSAGE帧，消息从中被编码为STOMP帧并通过WebSocket连接发送。

下一节将提供有关带注释的方法的更多详细信息，包括支持的参数类型和返回值。

#### 4.4.6。带注释的控制器

应用程序可以使用带注释的`@Controller`类来处理来自客户端的消息。此类可以声明`@MessageMapping`，`@SubscribeMapping`和`@ExceptionHandler` 方法，如以下主题所述：

- [`@MessageMapping`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-mapping)
- [`@SubscribeMapping`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-subscribe-mapping)
- [`@MessageExceptionHandler`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-exception-handler)

##### `@MessageMapping`

您可以`@MessageMapping`用来注释根据消息的目的地路由消息的方法。在方法级别和类型级别都支持它。在类型级别，`@MessageMapping`用于表示控制器中所有方法之间的共享映射。

默认情况下，映射值是Ant样式的路径模式（例如`/thing*`，`/thing/**`），包括对模板变量的支持（例如，`/thing/{id}`）。可以通过`@DestinationVariable`方法参数引用这些值。应用程序还可以切换到以点分隔的映射的目标约定，如“ [作为分隔符的点”中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-destination-separator)。

###### 支持的方法参数

下表描述了方法参数：

| 方法参数                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `Message`                                                    | 用于访问完整的消息。                                         |
| `MessageHeaders`                                             | 用于访问中的标头`Message`。                                  |
| `MessageHeaderAccessor`，`SimpMessageHeaderAccessor`和`StompHeaderAccessor` | 用于通过类型化访问器方法访问标头。                           |
| `@Payload`                                                   | 为了访问消息的有效负载，由configure转换（例如，从JSON转换） `MessageConverter`。不需要此注释，因为默认情况下会假定没有其他自变量匹配。您可以使用`@javax.validation.Valid`或Spring的注释有效负载参数`@Validated`，以使有效负载参数被自动验证。 |
| `@Header`                                                    | 用于访问特定的标头值- `org.springframework.core.convert.converter.Converter`如有必要，还可以使用进行类型转换 。 |
| `@Headers`                                                   | 用于访问消息中的所有标题。此参数必须可分配给 `java.util.Map`。 |
| `@DestinationVariable`                                       | 用于访问从消息目标中提取的模板变量。根据需要将值转换为声明的方法参数类型。 |
| `java.security.Principal`                                    | 反映在WebSocket HTTP握手时登录的用户。                       |

###### 返回值

默认情况下，方法的返回值`@MessageMapping`通过匹配序列化为有效负载，`MessageConverter`并以形式发送`Message`给`brokerChannel`，从中将其广播给订户。出站消息的目的地与入站消息的目的地相同，但前缀为`/topic`。

您可以使用`@SendTo`和`@SendToUser`注释来自定义输出消息的目的地。`@SendTo`用于自定义目标目的地或指定多个目的地。`@SendToUser`用于将输出消息定向到仅与输入消息关联的用户。请参阅[用户目的地](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-user-destination)。

您可以同时使用`@SendTo`，并`@SendToUser`在同一时间在同一个方法，无一不是在一流水平，在这种情况下，他们充当类中的方法默认支持。但是，请记住，任何方法级别`@SendTo`或`@SendToUser`注释都将在类级别覆盖所有此类注释。

消息可以异步处理和`@MessageMapping`方法可以返回 `ListenableFuture`，`CompletableFuture`或`CompletionStage`。

请注意，`@SendTo`并`@SendToUser`仅仅是相当于使用便利 `SimpMessagingTemplate`发送消息。如有必要，对于更高级的方案， `@MessageMapping`可以`SimpMessagingTemplate`直接使用直接方法。可以代替返回值，也可以附加返回值。请参阅[发送消息](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-handle-send)。

##### `@SubscribeMapping`

`@SubscribeMapping`与相似，`@MessageMapping`但将映射范围缩小到仅订阅消息。它支持相同的 [方法参数](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-mapping)的`@MessageMapping`。但是，对于返回值，默认情况下，消息直接发送给客户端（通过 `clientOutboundChannel`，以响应订阅），而不发送给代理（通过 `brokerChannel`，作为对匹配订阅的广播）。添加`@SendTo`或 `@SendToUser`覆盖此行为，然后发送给代理。

什么时候有用？假定代理映射到`/topic`和`/queue`，而应用程序控制器映射到`/app`。在此设置中，代理将存储所有订阅，`/topic`并`/queue`打算将其用于重复广播，因此无需参与该应用程序。客户端也可以预订某个`/app`目的地，并且控制器可以响应该预订而返回一个值，而无需经纪人参与，而无需再次存储或使用该预订（实际上是一次请求-答复交换）。一个用例是在启动时用初始数据填充UI。

什么时候没有用？不要尝试将代理和控制器映射到相同的目标前缀，除非您出于某种原因希望两者都独立处理消息（包括订阅）。入站消息是并行处理的。无法保证经纪人还是控制者首先处理给定的消息。如果目标是在存储预订并准备好广播时得到通知，则客户端应请求服务器是否支持收据（简单代理不支持）。例如，对于Java [STOMP客户端](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-client)，您可以执行以下操作添加收据：

```java
@Autowired
private TaskScheduler messageBrokerTaskScheduler;

// During initialization..
stompClient.setTaskScheduler(this.messageBrokerTaskScheduler);

// When subscribing..
StompHeaders headers = new StompHeaders();
headers.setDestination("/topic/...");
headers.setReceipt("r1");
FrameHandler handler = ...;
stompSession.subscribe(headers, handler).addReceiptTask(() -> {
    // Subscription ready...
});
```

服务器侧选择是[注册](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-interceptors)一个 `ExecutorChannelInterceptor`上`brokerChannel`并实施`afterMessageHandled` 被消息之后被调用，包括订阅，已处理方法。

##### `@MessageExceptionHandler`

应用程序可以使用`@MessageExceptionHandler`方法来处理方法中的异常 `@MessageMapping`。如果要访问异常实例，则可以在批注本身中声明异常，也可以通过方法参数声明异常。下面的示例通过方法参数声明异常：

```java
@Controller
public class MyController {

    // ...

    @MessageExceptionHandler
    public ApplicationError handleException(MyException exception) {
        // ...
        return appError;
    }
}
```

`@MessageExceptionHandler`方法支持灵活的方法签名，并支持与方法相同的方法参数类型和返回值 [`@MessageMapping`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-mapping)。

通常，`@MessageExceptionHandler`方法适用于`@Controller`声明它们的类（或类层次结构）。如果您希望此类方法在全局范围内（跨控制器）应用，则可以在标有的类中声明它们 `@ControllerAdvice`。这与Spring MVC中可用的[类似支持](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc-ann-controller-advice)相当 。

#### 4.4.7。传送讯息

如果要从应用程序的任何部分向连接的客户端发送消息怎么办？任何应用程序组件都可以将消息发送到`brokerChannel`。最简单的方法是注入a `SimpMessagingTemplate`并使用它发送消息。通常，您将按类型注入它，如以下示例所示：

```java
@Controller
public class GreetingController {

    private SimpMessagingTemplate template;

    @Autowired
    public GreetingController(SimpMessagingTemplate template) {
        this.template = template;
    }

    @RequestMapping(path="/greetings", method=POST)
    public void greet(String greeting) {
        String text = "[" + getTimestamp() + "]:" + greeting;
        this.template.convertAndSend("/topic/greetings", text);
    }

}
```

但是，`brokerMessagingTemplate`如果存在另一个相同类型的bean，也可以通过其名称（）对其进行限定。

#### 4.4.8。简单经纪人

内置的简单消息代理处理来自客户端的订阅请求，将其存储在内存中，并将消息广播到具有匹配目标的已连接客户端。该代理支持类似路径的目标，包括对Ant样式目标模式的订阅。

|      | 应用程序还可以使用点分隔（而不是斜杠分隔）目标。请参见[点作为分隔符](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-destination-separator)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果配置了任务调度程序，则简单代理支持 [STOMP心跳](https://stomp.github.io/stomp-specification-1.2.html#Heart-beating)。为此，您可以声明自己的调度程序，也可以使用内部自动声明和使用的调度程序。以下示例显示如何声明自己的调度程序：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    private TaskScheduler messageBrokerTaskScheduler;

    @Autowired
    public void setMessageBrokerTaskScheduler(TaskScheduler taskScheduler) {
        this.messageBrokerTaskScheduler = taskScheduler;
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {

        registry.enableSimpleBroker("/queue/", "/topic/")
                .setHeartbeatValue(new long[] {10000, 20000})
                .setTaskScheduler(this.messageBrokerTaskScheduler);

        // ...
    }
}
```

#### 4.4.9。外部经纪人

简单代理非常适合入门，但仅支持STOMP命令的子集（它不支持ack，回执和某些其他功能），依赖于简单的消息发送循环，并且不适合于集群。或者，您可以升级应用程序以使用功能全面的消息代理。

请参阅STOMP文档以了解您选择的消息代理（例如 [RabbitMQ](https://www.rabbitmq.com/stomp.html)， [ActiveMQ](https://activemq.apache.org/stomp.html)等），安装代理，并在启用STOMP支持的情况下运行它。然后，您可以在Spring配置中启用STOMP代理中继（而不是简单代理）。

以下示例配置启用了功能齐全的代理：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/portfolio").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableStompBrokerRelay("/topic", "/queue");
        registry.setApplicationDestinationPrefixes("/app");
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker application-destination-prefix="/app">
        <websocket:stomp-endpoint path="/portfolio" />
            <websocket:sockjs/>
        </websocket:stomp-endpoint>
        <websocket:stomp-broker-relay prefix="/topic,/queue" />
    </websocket:message-broker>

</beans>
```

先前配置中的STOMP代理中继是一个Spring [`MessageHandler`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/messaging/MessageHandler.html) ，它通过将消息转发到外部消息代理来处理消息。为此，它建立到代理的TCP连接，将所有消息转发给它，然后通过它们的WebSocket会话将从代理收到的所有消息转发给客户端。本质上，它充当“转发器”，可以双向转发消息。

|      | 将`io.projectreactor.netty:reactor-netty`和`io.netty:netty-all` 依赖项添加到您的项目中以进行TCP连接管理。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

此外，应用程序组件（例如HTTP请求处理方法，业务服务等）也可以将消息发送到代理中继，如“ [发送消息”中所述](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-handle-send)，以将消息广播到订阅的WebSocket客户端。

实际上，代理中继可实现健壮且可伸缩的消息广播。

#### 4.4.10。连接到经纪人

STOMP代理中继器维护与代理的单个“系统” TCP连接。此连接仅用于源自服务器端应用程序的消息，而不用于接收消息。您可以为此连接配置STOMP凭据（即STOMP帧`login`和`passcode`标头）。这在XML名称空间和Java配置中都以`systemLogin`和 `systemPasscode`属性（默认值为`guest`和）公开`guest`。

STOMP代理中继还为每个连接的WebSocket客户端创建一个单独的TCP连接。您可以配置用于代表客户端创建的所有TCP连接的STOMP凭据。这在XML名称空间和Java配置中都以`clientLogin`和`clientPasscode`属性（默认值为`guest`和）公开`guest`。

|      | STOMP代理中继始终 在代表客户端转发给代理的每个帧上设置`login`和`passcode`标头`CONNECT`。因此，WebSocket客户端无需设置这些标头。他们被忽略。如“ [身份验证”](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-authentication) 部分所述，WebSocket客户端应改为依靠HTTP身份验证来保护WebSocket端点并建立客户端身份。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

STOMP代理中继还通过“系统” TCP连接向消息代理发送和从消息代理接收心跳。您可以配置发送和接收心跳的间隔（默认情况下，每个间隔为10秒）。如果与代理的连接断开，则代理中继每5秒继续尝试重新连接，直到成功。

`ApplicationListener` 当与代理的“系统”连接丢失并重新建立时，任何Spring bean都可以实现以接收通知。例如，当没有活动的“系统”连接时，广播股票报价的股票报价服务可以停止尝试发送消息。

默认情况下，STOMP代理中继始终连接到同一主机和端口，如果连接断开，则根据需要重新连接。如果希望提供多个地址，则在每次尝试连接时，都可以配置地址供应商，而不是固定的主机和端口。以下示例显示了如何执行此操作：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig extends AbstractWebSocketMessageBrokerConfigurer {

    // ...

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableStompBrokerRelay("/queue/", "/topic/").setTcpClient(createTcpClient());
        registry.setApplicationDestinationPrefixes("/app");
    }

    private ReactorNettyTcpClient<byte[]> createTcpClient() {
        return new ReactorNettyTcpClient<>(
                client -> client.addressSupplier(() -> ... ),
                new StompReactorNettyCodec());
    }
}
```

您还可以使用`virtualHost`属性配置STOMP代理中继。此属性的值设置为`host`每个`CONNECT`帧的标头，并且很有用（例如，在建立TCP连接的实际主机与提供基于云的STOMP服务的主机不同的云环境中）。

#### 4.4.11。点作为分隔符

将消息路由到`@MessageMapping`方法时，它们与匹配 `AntPathMatcher`。默认情况下，模式应使用斜杠（`/`）作为分隔符。这是Web应用程序中的良好约定，类似于HTTP URL。但是，如果您更习惯于消息传递约定，则可以切换为使用点（`.`）作为分隔符。

以下示例显示了如何在Java配置中执行此操作：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    // ...

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setPathMatcher(new AntPathMatcher("."));
        registry.enableStompBrokerRelay("/queue", "/topic");
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:websocket="http://www.springframework.org/schema/websocket"
        xsi:schemaLocation="
                http://www.springframework.org/schema/beans
                https://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/websocket
                https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker application-destination-prefix="/app" path-matcher="pathMatcher">
        <websocket:stomp-endpoint path="/stomp"/>
        <websocket:stomp-broker-relay prefix="/topic,/queue" />
    </websocket:message-broker>

    <bean id="pathMatcher" class="org.springframework.util.AntPathMatcher">
        <constructor-arg index="0" value="."/>
    </bean>

</beans>
```

之后，控制器可以使用点（`.`）作为`@MessageMapping`方法中的分隔符，如以下示例所示：

```java
@Controller
@MessageMapping("red")
public class RedController {

    @MessageMapping("blue.{green}")
    public void handleGreen(@DestinationVariable String green) {
        // ...
    }
}
```

客户端现在可以向发送消息`/app/red.blue.green123`。

在前面的示例中，我们没有更改“代理中继”上的前缀，因为这些前缀完全取决于外部消息代理。有关您使用的代理的信息，请参见STOMP文档页面，以查看其对目标标头支持的约定。

另一方面，“简单代理”确实依赖于configure `PathMatcher`，因此，如果切换分隔符，该更改也将应用于代理以及代理将目标从消息匹配到订阅中的模式的方式。

#### 4.4.12。认证方式

每个通过WebSocket进行的STOMP消息传递会话均以HTTP请求开头。这可以是升级到WebSockets的请求（即WebSocket握手），或者在SockJS后备情况下，是一系列SockJS HTTP传输请求。

许多Web应用程序已经具有身份验证和授权来保护HTTP请求。通常，使用某种机制（例如登录页面，HTTP基本认证或其他方式）通过Spring Security对用户进行认证。经过身份验证的用户的安全上下文保存在HTTP会话中，并与同一基于cookie的会话中的后续请求关联。

因此，对于WebSocket握手或SockJS HTTP传输请求，通常已经存在可通过访问的经过身份验证的用户 `HttpServletRequest#getUserPrincipal()`。Spring会自动将该用户与为其创建的WebSocket或SockJS会话相关联，并随后与该会话中通过用户标头传输的所有STOMP消息相关联。

简而言之，典型的Web应用程序除了已经为安全起见，就不需要做任何事情。通过基于cookie的HTTP会话（然后与为该用户创建的WebSocket或SockJS会话相关联）维护的安全上下文在HTTP请求级别对用户进行身份验证，并导致在每个`Message`流经的用户头上加盖用户头应用程序。

请注意，STOMP协议在帧上确实有`login`和`passcode`标头`CONNECT`。这些最初是设计用于并且仍然需要的，例如，基于TCP的STOMP。但是，对于默认情况下，对于基于WebSocket的STOMP，Spring会在STOMP协议级别忽略授权标头，并假定该用户已经在HTTP传输级别进行了身份验证，并期望WebSocket或SockJS会话包含经过身份验证的用户。

|      | Spring Security提供了 [WebSocket子协议授权](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#websocket) ，该[授权](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#websocket)使用`ChannelInterceptor`来根据消息中的用户标头对消息进行授权。而且，Spring Session提供了 [WebSocket集成](https://docs.spring.io/spring-session/docs/current/reference/html5/#websocket) ，以确保当WebSocket会话仍处于活动状态时，用户HTTP会话不会过期。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.4.13。令牌认证

[Spring Security OAuth](https://github.com/spring-projects/spring-security-oauth) 支持基于令牌的安全性，包括JSON Web令牌（JWT）。您可以将其用作Web应用程序中的身份验证机制，包括上一节中所述的WebSocket交互中的STOMP（即，通过基于cookie的会话来维护身份）。

同时，基于cookie的会话并不总是最合适（例如，在不维护服务器端会话的应用程序中或在通常使用标头进行身份验证的移动应用程序中）。

该[WebSocket协议，RFC 6455](https://tools.ietf.org/html/rfc6455#section-10.5) “并没有规定任何具体的方式，服务器可以在网页套接字握手期间验证客户端。” 但是，实际上，浏览器客户端只能使用标准身份验证标头（即基本HTTP身份验证）或cookie，而不能（例如）提供自定义标头。同样，SockJS JavaScript客户端也不提供通过SockJS传输请求发送HTTP标头的方法。请参阅 [sockjs-client问题196](https://github.com/sockjs/sockjs-client/issues/196)。相反，它确实允许发送可用于发送令牌的查询参数，但这有其自身的缺点（例如，令牌可能会无意中与服务器日志中的URL一起记录）。

|      | 前述限制适用于基于浏览器的客户端，不适用于基于Spring Java的STOMP客户端，该客户端确实支持通过WebSocket和SockJS请求发送标头。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

因此，希望避免使用Cookie的应用程序可能没有在HTTP协议级别进行身份验证的任何好选择。他们可能更喜欢在STOMP消息传递协议级别使用标头进行身份验证，而不是使用cookie，这样做需要两个简单的步骤：

1. 使用STOMP客户端在连接时传递身份验证头。
2. 使用来处理身份验证标头`ChannelInterceptor`。

下一个示例使用服务器端配置来注册自定义身份验证拦截器。请注意，拦截器仅需要认证并在CONNECT上设置用户标头`Message`。Spring记录并保存经过身份验证的用户，并将其与同一会话上的后续STOMP消息相关联。以下示例显示了如何注册自定义身份验证拦截器：

```java
@Configuration
@EnableWebSocketMessageBroker
public class MyConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new ChannelInterceptor() {
            @Override
            public Message<?> preSend(Message<?> message, MessageChannel channel) {
                StompHeaderAccessor accessor =
                        MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
                if (StompCommand.CONNECT.equals(accessor.getCommand())) {
                    Authentication user = ... ; // access authentication header(s)
                    accessor.setUser(user);
                }
                return message;
            }
        });
    }
}
```

另外，请注意，当前，当使用Spring Security的消息授权时，需要确保在`ChannelInterceptor`Spring Security之前订购身份验证配置。最好通过在`WebSocketMessageBrokerConfigurer`标 有的自己的实现中声明自定义拦截器来完成此操作`@Order(Ordered.HIGHEST_PRECEDENCE + 99)`。

#### 4.4.14。用户目的地

应用程序可以发送针对特定用户的消息，并且Spring的STOMP支持可以识别`/user/`为此目的加上前缀的目标。例如，客户端可能订阅了`/user/queue/position-updates`目的地。该目标由处理，`UserDestinationMessageHandler`并转换为用户会话唯一的目标（例如`/queue/position-updates-user123`）。这提供了订阅通用命名目的地的便利，同时确保与预订同一目的地的其他用户不发生冲突，以便每个用户都可以接收唯一的库存头寸更新。

在发送方，可以将消息发送到目的地（例如） `/user/{username}/queue/position-updates`，然后将其转换`UserDestinationMessageHandler`为一个或多个目的地，每个与用户相关联的会话都将其翻译成一个或多个目的地。这样，应用程序中的任何组件都可以发送针对特定用户的消息，而不必知道他们的姓名和通用目的地。注释和消息传递模板也支持此功能。

消息处理方法可以通过`@SendToUser`注释将消息发送给与正在处理的消息相关联的用户（在类级别上也支持共享公共目标），该消息如以下示例所示：

```java
@Controller
public class PortfolioController {

    @MessageMapping("/trade")
    @SendToUser("/queue/position-updates")
    public TradeResult executeTrade(Trade trade, Principal principal) {
        // ...
        return tradeResult;
    }
}
```

如果用户有多个会话，则默认情况下，所有订阅给定目标的会话都是目标。但是，有时可能仅需要将发送正在处理的消息的会话作为目标。可以通过将`broadcast`属性设置为false来实现，如以下示例所示：

```java
@Controller
public class MyController {

    @MessageMapping("/action")
    public void handleAction() throws Exception{
        // raise MyBusinessException here
    }

    @MessageExceptionHandler
    @SendToUser(destinations="/queue/errors", broadcast=false)
    public ApplicationError handleException(MyBusinessException exception) {
        // ...
        return appError;
    }
}
```

|      | 尽管用户目的地通常暗指经过身份验证的用户，但这并不是严格要求的。不与已认证用户关联的WebSocket会话可以订阅用户目的地。在这种情况下，`@SendToUser`注释的行为与完全相同`broadcast=false`（也就是说，仅针对发送正在处理的消息的会话）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您可以从任何应用程序组件向用户目标发送消息，例如，注入`SimpMessagingTemplate`由Java配置或XML名称空间创建的消息。（`brokerMessagingTemplate`如果需要使用，则必须使用Bean名称`@Qualifier`。）下面的示例演示了如何实现：

```java
@Service
public class TradeServiceImpl implements TradeService {

    private final SimpMessagingTemplate messagingTemplate;

    @Autowired
    public TradeServiceImpl(SimpMessagingTemplate messagingTemplate) {
        this.messagingTemplate = messagingTemplate;
    }

    // ...

    public void afterTradeExecuted(Trade trade) {
        this.messagingTemplate.convertAndSendToUser(
                trade.getUserName(), "/queue/position-updates", trade.getResult());
    }
}
```

|      | 当将用户目标与外部消息代理一起使用时，应查看代理文档以了解如何管理非活动队列，以便在用户会话结束时，所有唯一的用户队列都将被删除。例如，当您使用目的地时，RabbitMQ会创建自动删除队列`/exchange/amq.direct/position-updates`。因此，在这种情况下，客户端可以订阅`/user/exchange/amq.direct/position-updates`。同样，ActiveMQ具有 用于清除非活动目标的[配置选项](https://activemq.apache.org/delete-inactive-destinations.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在多应用程序服务器方案中，由于用户连接到其他服务器，因此用户目标可能无法解析。在这种情况下，可以将目标配置为广播未解析的消息，以便其他服务器可以尝试。这可以通过做`userDestinationBroadcast`财产 `MessageBrokerRegistry`在Java配置和`user-destination-broadcast`该属性`message-broker`的XML元素。

#### 4.4.15。消息顺序

来自代理的消息被发布到`clientOutboundChannel`，从那里被写入WebSocket会话。由于该通道由a支持`ThreadPoolExecutor`，因此消息将在不同的线程中处理，并且客户端接收到的结果序列可能与发布的确切顺序不匹配。

如果这是一个问题，请启用该`setPreservePublishOrder`标志，如以下示例所示：

```java
@Configuration
@EnableWebSocketMessageBroker
public class MyConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    protected void configureMessageBroker(MessageBrokerRegistry registry) {
        // ...
        registry.setPreservePublishOrder(true);
    }

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker preserve-publish-order="true">
        <!-- ... -->
    </websocket:message-broker>

</beans>
```

设置该标志后，同一客户端会话中的消息将一次发布到`clientOutboundChannel`一个客户端会话中 ，这样可以保证发布顺序。请注意，这会产生很小的性能开销，因此，只有在需要时才应启用它。

#### 4.4.16。大事记

`ApplicationContext`通过实现Spring的`ApplicationListener`接口，可以发布几个事件，并且可以接收这些事件：

- `BrokerAvailabilityEvent`：指示代理何时可用或不可用。当“简单”代理在启动时立即可用并保持运行状态时，STOMP“代理中继”可能会失去与全功能代理的连接（例如，如果代理重新启动）。代理中继具有重新连接逻辑，并在代理返回时重新建立与代理的“系统”连接。因此，只要状态从已连接变为断开，反之亦然，就会发布此事件。使用的组件`SimpMessagingTemplate`应订阅此事件，并避免在代理不可用时避免发送消息。无论如何，他们应该准备`MessageDeliveryException` 在发送消息时进行处理。
- `SessionConnectEvent`：在收到新的STOMP CONNECT以指示新的客户端会话开始时发布。该事件包含代表连接的消息，包括会话ID，用户信息（如果有）以及客户端发送的所有自定义标头。这对于跟踪客户端会话很有用。预订此事件的组件可以使用`SimpMessageHeaderAccessor`或 包装包含的消息`StompMessageHeaderAccessor`。
- `SessionConnectedEvent`：`SessionConnectEvent`在代理发送一个STOMP CONNECTED帧以响应CONNECT 之后不久发布。此时，可以认为STOMP会话已完全建立。
- `SessionSubscribeEvent`：在收到新的STOMP SUBSCRIBE时发布。
- `SessionUnsubscribeEvent`：在收到新的STOMP UNSUBSCRIBE时发布。
- `SessionDisconnectEvent`：在STOMP会话结束时发布。DISCONNECT可能是从客户端发送的，也可能是在WebSocket会话关闭时自动生成的。在某些情况下，每个会话多次发布此事件。关于多个断开事件，组件应该是幂等的。

|      | 当您使用功能齐全的代理时，如果代理暂时不可用，则STOMP“代理中继”会自动重新连接“系统”连接。但是，客户端连接不会自动重新连接。假设启用了心跳，客户端通常会注意到代理在10秒内没有响应。客户需要实现自己的重新连接逻辑。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.4.17。拦截

[事件](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-appplication-context-events)提供有关STOMP连接生命周期的通知，但不提供每条客户端消息的通知。应用程序还可以注册一个， `ChannelInterceptor`以拦截处理链中任何部分的任何消息。以下示例显示了如何拦截来自客户端的入站消息：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new MyChannelInterceptor());
    }
}
```

定制`ChannelInterceptor`可以使用`StompHeaderAccessor`或`SimpMessageHeaderAccessor` 访问有关消息的信息，如以下示例所示：

```java
public class MyChannelInterceptor implements ChannelInterceptor {

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor accessor = StompHeaderAccessor.wrap(message);
        StompCommand command = accessor.getStompCommand();
        // ...
        return message;
    }
}
```

应用程序还可以实现`ExecutorChannelInterceptor`，这是`ChannelInterceptor`在其中处理消息的线程中带有回调的子接口。虽然`ChannelInterceptor`对发送到通道的每个消息都调用一次， 但a `ExecutorChannelInterceptor`在每个`MessageHandler` 订阅了该通道消息的线程中都提供了挂钩。

请注意，`SesionDisconnectEvent`如前所述，DISCONNECT消息可以来自客户端，也可以在WebSocket会话关闭时自动生成。在某些情况下，对于每个会话，拦截器可能会多次拦截此消息。关于多个断开事件，组件应该是幂等的。

#### 4.4.18。STOMP客户端

Spring提供了一个基于WebSocket客户端的STOMP和一个基于TCP客户端的STOMP。

首先，您可以创建和配置`WebSocketStompClient`，如以下示例所示：

```java
WebSocketClient webSocketClient = new StandardWebSocketClient();
WebSocketStompClient stompClient = new WebSocketStompClient(webSocketClient);
stompClient.setMessageConverter(new StringMessageConverter());
stompClient.setTaskScheduler(taskScheduler); // for heartbeats
```

在前面的示例中，您可以替换`StandardWebSocketClient`为`SockJsClient`，因为这也是的实现`WebSocketClient`。在`SockJsClient`可以使用基于HTTP的WebSocket或运输作为后备。有关更多详细信息，请参见 [`SockJsClient`](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-fallback-sockjs-client)。

接下来，您可以建立连接并为STOMP会话提供处理程序，如以下示例所示：

```java
String url = "ws://127.0.0.1:8080/endpoint";
StompSessionHandler sessionHandler = new MyStompSessionHandler();
stompClient.connect(url, sessionHandler);
```

会话准备就绪时，将通知处理程序，如以下示例所示：

```java
public class MyStompSessionHandler extends StompSessionHandlerAdapter {

    @Override
    public void afterConnected(StompSession session, StompHeaders connectedHeaders) {
        // ...
    }
}
```

建立会话后，就可以发送任何有效负载，并使用configure来对其进行序列化`MessageConverter`，如以下示例所示：

```java
session.send("/topic/something", "payload");
```

您还可以订阅目的地。这些`subscribe`方法需要用于订阅消息的处理程序，并返回`Subscription`可用于取消订阅的句柄。对于每个收到的消息，处理程序可以指定`Object`有效负载应反序列化的目标 类型，如以下示例所示：

```java
session.subscribe("/topic/something", new StompFrameHandler() {

    @Override
    public Type getPayloadType(StompHeaders headers) {
        return String.class;
    }

    @Override
    public void handleFrame(StompHeaders headers, Object payload) {
        // ...
    }

});
```

要启用STOMP心跳，您可以配置`WebSocketStompClient`a `TaskScheduler` 并有选择地自定义心跳间隔（写非活动状态为10秒，导致发送心跳，读非活动状态为10秒，关闭连接）。

|      | 当`WebSocketStompClient`用于性能测试以模拟同一台计算机上的数千个客户端时，请考虑关闭心跳，因为每个连接都计划自己的心跳任务，并且并未针对在同一台计算机上运行的大量客户端进行优化。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

STOMP协议还支持回执，在`receipt` 发送或订阅处理完毕后，客户端必须在其中添加一个标头，服务器以RECEIPT帧响应该标头。为了支持这一点，`StompSession`报价 `setAutoReceipt(boolean)`，导致一`receipt`对，以后每发送添加或订阅事件标题。另外，您也可以手动将收据标头添加到中`StompHeaders`。发送和订阅都返回一个实例`Receiptable` ，您可以使用该实例注册收款成功和失败回调。要使用此功能，您必须为客户端配置`TaskScheduler` 和以及收据过期之前的时间（默认为15秒）。

请注意，`StompSessionHandler`它本身是a `StompFrameHandler`，除了`handleException`回调以外，它还可以处理ERROR帧，以处理消息中的异常以及`handleTransportError`包括在内的传输级错误`ConnectionLostException`。

#### 4.4.19。WebSocket范围

每个WebSocket会话都有一个属性映射。该映射作为标头附加到入站客户端消息，可以通过控制器方法进行访问，如以下示例所示：

```java
@Controller
public class MyController {

    @MessageMapping("/action")
    public void handle(SimpMessageHeaderAccessor headerAccessor) {
        Map<String, Object> attrs = headerAccessor.getSessionAttributes();
        // ...
    }
}
```

您可以在`websocket`范围中声明一个Spring托管的bean 。您可以将WebSocket范围的bean注入控制器和在上注册的所有通道拦截器中`clientInboundChannel`。这些通常是单例，并且比任何单独的WebSocket会话寿命更长。因此，您需要对作用域WebSocket的bean使用作用域代理模式，如以下示例所示：

```java
@Component
@Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyBean {

    @PostConstruct
    public void init() {
        // Invoked after dependencies injected
    }

    // ...

    @PreDestroy
    public void destroy() {
        // Invoked when the WebSocket session ends
    }
}

@Controller
public class MyController {

    private final MyBean myBean;

    @Autowired
    public MyController(MyBean myBean) {
        this.myBean = myBean;
    }

    @MessageMapping("/action")
    public void handle() {
        // this.myBean from the current WebSocket session
    }
}
```

与任何自定义范围一样，Spring `MyBean`首次在控制器中对其进行访问时会初始化一个新实例，并将该实例存储在WebSocket会话属性中。随后将返回相同的实例，直到会话结束。WebSocket范围的bean调用了所有Spring生命周期方法，如前面的示例所示。

#### 4.4.20。性能

关于性能，没有万灵药。影响它的因素很多，包括消息的大小和数量，应用程序方法是否执行需要阻止的工作以及外部因素（例如网络速度和其他问题）。本部分的目的是提供可用配置选项的概述，以及有关如何进行扩展的一些想法。

在消息传递应用程序中，消息通过通道传递以进行异步执行，并由线程池支持。配置这样的应用程序需要对通道和消息流有充分的了解。因此，建议查看[消息流](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#websocket-stomp-message-flow)。

最明显的地方，开始是配置线程池是背 `clientInboundChannel`和`clientOutboundChannel`。默认情况下，两者都配置为可用处理器数量的两倍。

如果带注释的方法中的消息处理主要是受CPU限制的，则的线程数`clientInboundChannel`应保持接近处理器数。如果他们所做的工作更多地受IO限制，并且需要阻塞或等待数据库或其他外部系统，则可能需要增加线程池大小。

|      | `ThreadPoolExecutor` 具有三个重要属性：核心线程池大小，最大线程池大小，以及队列存储没有可用线程的任务的容量。常见的混淆点是，配置核心池大小（例如10）和最大池大小（例如20）会导致线程池具有10到20个线程。实际上，如果将容量保留为其默认值Integer.MAX_VALUE，则线程池将永远不会增加超出核心池的大小，因为所有其他任务都已排队。请参阅的javadoc，`ThreadPoolExecutor`以了解这些属性如何工作并了解各种排队策略。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在`clientOutboundChannel`一边，它是所有关于发送消息的WebSocket客户端。如果客户端位于快速网络上，则线程数应保持接近可用处理器数。如果它们很慢或带宽很低，它们将花费更长的时间来消耗消息并给线程池增加负担。因此，必须增加线程池的大小。

尽管`clientInboundChannel`可以预测应用程序的工作量（毕竟，它是基于应用程序的工作），但是如何配置“ clientOutboundChannel”却更加困难，因为它基于应用程序无法控制的因素。因此，有两个附加属性与消息发送有关：`sendTimeLimit` 和`sendBufferSizeLimit`。您可以使用这些方法来配置发送消息到客户端时允许发送多长时间以及可以缓冲多少数据。

通常的想法是，在任何给定时间，只能使用单个线程将其发送给客户端。同时，所有其他消息都会被缓冲，您可以使用这些属性来决定允许发送消息花费多长时间以及在此期间可以缓冲多少数据。有关其他重要信息，请参见XML模式的javadoc和文档。

以下示例显示了可能的配置：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        registration.setSendTimeLimit(15 * 1000).setSendBufferSizeLimit(512 * 1024);
    }

    // ...

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker>
        <websocket:transport send-timeout="15000" send-buffer-size="524288" />
        <!-- ... -->
    </websocket:message-broker>

</beans>
```

您还可以使用前面显示的WebSocket传输配置来配置传入STOMP消息的最大允许大小。从理论上讲，WebSocket消息的大小几乎是无限的。实际上，WebSocket服务器施加了限制-例如，Tomcat 8K和Jetty 64K。因此，STOMP客户端（例如JavaScript [webstomp-client](https://github.com/JSteunou/webstomp-client) 和其他[客户端](https://github.com/JSteunou/webstomp-client)）在16K边界处拆分较大的STOMP消息，并将其作为多个WebSocket消息发送，这需要服务器进行缓冲和重新组装。

Spring的STOMP-over-WebSocket支持可以做到这一点，因此应用程序可以为STOMP消息配置最大大小，而与WebSocket服务器特定的消息大小无关。请记住，如有必要，将自动调整WebSocket消息的大小，以确保它们最多可以承载16K WebSocket消息。

以下示例显示了一种可能的配置：

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureWebSocketTransport(WebSocketTransportRegistration registration) {
        registration.setMessageSizeLimit(128 * 1024);
    }

    // ...

}
```

下面的示例显示与前面的示例等效的XML配置：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:websocket="http://www.springframework.org/schema/websocket"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/websocket
        https://www.springframework.org/schema/websocket/spring-websocket.xsd">

    <websocket:message-broker>
        <websocket:transport message-size="131072" />
        <!-- ... -->
    </websocket:message-broker>

</beans>
```

关于扩展的重要一点涉及使用多个应用程序实例。当前，您不能使用简单的代理来做到这一点。但是，当您使用功能齐全的代理（例如RabbitMQ）时，每个应用程序实例都连接到代理，并且从一个应用程序实例广播的消息可以通过代理广播到通过任何其他应用程序实例连接的WebSocket客户端。

#### 4.4.21。监控方式

当您使用`@EnableWebSocketMessageBroker`或时``，关键基础架构组件会自动收集统计信息和计数器，以提供对应用程序内部状态的重要了解。该配置还声明了一个类型为Bean的bean，该bean `WebSocketMessageBrokerStats`会将所有可用信息收集到一个位置，并且默认情况下`INFO`每30分钟将其记录在该级别一次。可以通过Spring将该bean导出到JMX `MBeanExporter`，以便在运行时进行查看（例如，通过JDK `jconsole`）。以下列表总结了可用信息：

- 客户端WebSocket会话

  当前指示当前有多少个客户端会话，并且通过WebSocket与HTTP流和轮询SockJS会话进一步细分该计数。总指示已建立的会话总数。异常关闭连接失败已建立但在60秒内未收到任何消息后关闭的会话。这通常表示代理或网络问题。超过发送限制超过配置的发送超时或发送缓冲区限制后，会话将关闭，缓慢的客户端可能会发生这些会话（请参阅上一节）。运输错误传输错误（例如无法读取或写入WebSocket连接或HTTP请求或响应）后，会话关闭。STOMP框架已处理的CONNECT，CONNECTED和DISCONNECT帧的总数，指示在STOMP级别上连接了多少个客户端。请注意，当会话异常关闭或客户端未发送DISCONNECT帧而关闭时，DISCONNECT计数可能会降低。

- STOMP经纪人接力

  TCP连接指示与代理建立了代表客户端WebSocket会话的TCP连接数。这应该等于客户端WebSocket会话的数量+ 1个用于从应用程序内部发送消息的附加共享“系统”连接。STOMP框架代表客户转发到代理或从代理接收的CONNECT，CONNECTED和DISCONNECT帧的总数。请注意，无论客户端WebSocket会话如何关闭，DISCONNECT帧都会发送到代理。因此，较低的DISCONNECT帧计数表示代理正在主动关闭连接（可能是由于未及时到达的心跳，无效的输入帧或其他问题）。

- 客户入站通道

  来自支持该线程池的线程池的统计信息`clientInboundChannel` ，可深入了解传入消息处理的运行状况。此处排队的任务表明该应用程序可能太慢而无法处理消息。如果存在I / O绑定的任务（例如，慢速的数据库查询，对第三方REST API的HTTP请求等），请考虑增加线程池的大小。

- 客户出站通道

  来自支持该线程池的线程池的统计信息`clientOutboundChannel` ，可深入了解向客户端广播消息的运行状况。此处排队的任务表明客户端太慢而无法使用消息。解决此问题的一种方法是增加线程池大小，以容纳预期数量的并发慢速客户端。另一个选择是减少发送超时和发送缓冲区大小限制（请参阅上一节）。

- SockJS任务计划程序

  来自SockJS任务调度程序的线程池的统计信息，用于发送心跳。请注意，在STOMP级别协商心跳时，将禁用SockJS心跳。

#### 4.4.22。测试中

当您使用Spring的STOMP-over-WebSocket支持时，有两种主要的方法来测试应用程序。首先是编写服务器端测试以验证控制器的功能及其带注释的消息处理方法。第二个是编写涉及运行客户端和服务器的完整的端到端测试。

两种方法不是互斥的。相反，每个人在整体测试策略中都有自己的位置。服务器端测试更加集中，更易于编写和维护。另一方面，端到端集成测试更完整，测试更多，但是编写和维护也更加复杂。

服务器端测试的最简单形式是编写控制器单元测试。但是，这还不够有用，因为控制器所做的很多事情都取决于其注释。纯单元测试根本无法测试。

理想情况下，应像在运行时一样调用被测控制器，就像使用Spring MVC Test框架测试处理HTTP请求的控制器的方法一样，即不运行Servlet容器而是依靠Spring框架来调用被测控制器。带注释的控制器。与Spring MVC Test一样，您有两种可能的选择，要么使用“基于上下文的”设置，要么使用“独立”设置：

- 在Spring TestContext框架的帮助下加载实际的Spring配置，`clientInboundChannel`作为测试字段注入，并使用它发送消息以由控制器方法处理。
- 手动设置最低的Spring框架基础结构，以调用控制器（即`SimpAnnotationMethodMessageHandler`）并将控制器消息直接传递给它。

[在股票投资组合](https://github.com/rstoyanchev/spring-websocket-portfolio/tree/master/src/test/java/org/springframework/samples/portfolio/web) 样本应用程序的[测试](https://github.com/rstoyanchev/spring-websocket-portfolio/tree/master/src/test/java/org/springframework/samples/portfolio/web)中演示了这两种设置方案 。

第二种方法是创建端到端集成测试。为此，您需要以嵌入式模式运行WebSocket服务器，并将其作为发送包含STOMP帧的WebSocket消息的WebSocket客户端连接到该服务器。 通过使用Tomcat作为嵌入式WebSocket服务器和用于测试目的的简单STOMP客户端，针对[股票投资组合](https://github.com/rstoyanchev/spring-websocket-portfolio/tree/master/src/test/java/org/springframework/samples/portfolio/web)示例应用程序的[测试](https://github.com/rstoyanchev/spring-websocket-portfolio/tree/master/src/test/java/org/springframework/samples/portfolio/web)也演示了这种方法。

## 5.其他Web框架

本章详细介绍了Spring与第三方Web框架的集成。

Spring框架的核心价值主张之一就是支持 *选择*。从一般意义上讲，Spring不会强迫您使用或购买任何特定的体系结构，技术或方法（尽管它肯定比其他建议更重要）。这种自由选择和选择与开发人员及其开发团队最相关的架构，技术或方法的自由在网络领域可以说是最明显的，在Web领域，Spring提供了自己的Web框架（[Spring MVC](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc)和[Spring WebFlux](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/webflux.html#webflux)）。同时，支持与许多流行的第三方Web框架集成。

### 5.1。通用配置

在深入研究每个受支持的Web框架的集成细节之前，让我们首先看一下并非特定于任何Web框架的通用Spring配置。（本节同样适用于Spring自己的Web框架变体。）

Spring的轻量级应用程序模型拥护的一个概念（需要一个更好的词）是分层体系结构的概念。请记住，在“经典”分层体系结构中，Web层只是许多层之一。它充当服务器端应用程序的入口点之一，并且委派服务层中定义的服务对象（外观），以满足特定于业务（与表示技术无关）的用例。在Spring中，这些服务对象，任何其他特定于业务的对象，数据访问对象和其他对象都存在于不同的“业务上下文”中，其中不包含Web或表示层对象（表示对象，例如Spring MVC控制器，通常是在不同的“展示环境”中进行配置）。本节详细介绍如何配置Spring容器（ `WebApplicationContext`），其中包含应用程序中的所有“ Business Bean”。

继续讲细节，您要做的就是[`ContextLoaderListener`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/context/ContextLoaderListener.html) 在`web.xml`Web应用程序的标准Java EE servlet 文件中声明a 并添加 `contextConfigLocation`<context-param />部分（在同一文件中），该部分定义了哪组Spring XML配置文件。装载。

考虑以下``配置：

```xml
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

进一步考虑以下``配置：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext*.xml</param-value>
</context-param>
```

如果未指定`contextConfigLocation`context参数，则 `ContextLoaderListener`查找一个名为`/WEB-INF/applicationContext.xml`load 的文件。加载上下文文件后，Spring将[`WebApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/context/WebApplicationContext.html) 根据Bean定义创建一个 对象，并将其存储在`ServletContext`Web应用程序的中。

所有Java Web框架都建立在Servlet API的基础上，因此您可以使用以下代码片段来访问`ApplicationContext` 由创建的“业务上下文” `ContextLoaderListener`。

以下示例显示了如何获取`WebApplicationContext`：

```java
WebApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(servletContext);
```

该 [`WebApplicationContextUtils`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/context/support/WebApplicationContextUtils.html) 班是为了方便，所以你不必记住的名字`ServletContext` 属性。如果 键下不存在对象，则`getWebApplicationContext()`返回其方法。与其冒险进入您的应用程序，不如使用该方法。缺少时，此方法将引发异常。`null``WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE``NullPointerExceptions``getRequiredWebApplicationContext()``ApplicationContext`

一旦有了对的引用`WebApplicationContext`，就可以按其名称或类型检索bean。大多数开发人员都按名称检索bean，然后将其转换为已实现的接口之一。

幸运的是，本节中的大多数框架都具有更简单的查找bean的方法。它们不仅使从Spring容器中获取bean变得容易，而且还使您可以在其控制器上使用依赖项注入。每个Web框架部分都有其特定集成策略的更多详细信息。

### 5.2。JSF

JavaServer Faces（JSF）是JCP的基于组件的标准，事件驱动的Web用户界面框架。它是Java EE伞的正式组成部分，但也可以单独使用，例如通过将Mojarra或MyFaces嵌入Tomcat中。

请注意，JSF的最新版本已与应用程序服务器中的CDI基础结构紧密联系在一起，其中一些新的JSF功能仅在这种环境下有效。Spring的JSF支持不再积极地发展，主要是在现代化较旧的基于JSF的应用程序时出于迁移目的而存在。

Spring的JSF集成中的关键元素是JSF `ELResolver`机制。

#### 5.2.1。Spring Bean解析器

`SpringBeanFacesELResolver`是符合JSF的`ELResolver`实现，与JSF和JSP使用的标准Unified EL集成。它首先委托给Spring的“业务上下文” `WebApplicationContext`，然后委托给底层JSF实现的默认解析器。

在配置方面，您可以`SpringBeanFacesELResolver`在JSF `faces-context.xml`文件中定义，如以下示例所示：

```xml
<faces-config>
    <application>
        <el-resolver>org.springframework.web.jsf.el.SpringBeanFacesELResolver</el-resolver>
        ...
    </application>
</faces-config>
```

#### 5.2.2。使用`FacesContextUtils`

`ELResolver`在将属性映射到中的Bean时`faces-config.xml`，自定义效果很好 ，但是有时您可能需要显式地获取Bean。该[`FacesContextUtils`](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/javadoc-api/org/springframework/web/jsf/FacesContextUtils.html) 班让一切变得简单。与相似`WebApplicationContextUtils`，除了它采用`FacesContext`参数而不是`ServletContext`参数。

以下示例显示如何使用`FacesContextUtils`：

```java
ApplicationContext ctx = FacesContextUtils.getWebApplicationContext(FacesContext.getCurrentInstance());
```

### 5.3。Apache Struts 2.x

[Struts](https://struts.apache.org/)由Craig McClanahan发明，是由Apache Software Foundation托管的开源项目。当时，它极大地简化了JSP / Servlet编程范例，并赢得了许多使用专有框架的开发人员的青睐。它简化了编程模型，它是开源的（因此像啤酒一样免费），并且拥有庞大的社区，这使该项目得以发展并在Java Web开发人员中广受欢迎。

作为原始Struts 1.x的后继产品，请查看Struts 2.x和Struts提供的 [Spring插件](https://struts.apache.org/release/2.3.x/docs/spring-plugin.html)以进行内置的Spring集成。

### 5.4。Apache Tapestry 5.x

[Tapestry](https://tapestry.apache.org/)是一个“面向组件的框架，用于在Java中创建动态，健壮，高度可伸缩的Web应用程序。”

尽管Spring具有自己[强大的Web层](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/web.html#mvc)，但通过将Tapestry用于Web用户界面并将Spring容器用于较低层，构建企业Java应用程序具有许多独特的优势。

有关更多信息，请参见Tapestry的[Spring](https://tapestry.apache.org/integrating-with-spring-framework.html)专用 [集成模块](https://tapestry.apache.org/integrating-with-spring-framework.html)。

### 5.5。更多资源

以下链接提供了有关本章中描述的各种Web框架的更多资源。

- 在[JSF](https://www.oracle.com/technetwork/java/javaee/javaserverfaces-139869.html)主页
- 在[Struts的](https://struts.apache.org/)主页
- 该[挂毯](https://tapestry.apache.org/)主页

版本5.2.6.RELEASE
最后更新时间2020-04-28 08:14:00 UTC