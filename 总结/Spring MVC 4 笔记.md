# 精通 Spring MVC 4

## 第一章 快速搭建Spring Web应用

### 1.1 @SpringBootApplication注解

它实际上组合了 3 个其他的注解，也就是@Configuration、@EnableAutoConfiguration 和@ComponentScan：

@Configuration	## 表明我们的这个类将会处理 Spring 的常规配置，如 bean 的声明

@ComponentScan  ## 告诉 Spring 去哪里查找 Spring 组件（服务、控制器等）。在默认情况下，这个注解将会扫描当前包以及该包下面的所有子包。

@EnableAutoConfiguration ## 开启Spring Boot 的自动配置

```java
@Target(ElementType.TYPE) 
@Retention(RetentionPolicy.RUNTIME) 
@Documented 
@Inherited 
@Configuration
@EnableAutoConfiguration 
@ComponentScan
public @interface SpringBootApplication { 
/** 
* Exclude specific auto-configuration classes such that they will
never be applied. 
*/ 
Class<?>[] exclude() default {}; 
}
```

```java
@Controller
public class HelloController { 

 @RequestMapping("/") 
 @ResponseBody 
 public String hello() { 
		return "Hello, world!"; 
 }
}
```

将控制器放到 controller 子包中，这样它就能够被@ComponentScan 注解所发现

打开浏览器并访问 http://localhost:8080 的话，就能看到我们钟爱的“Hello, world!”被输出了出来

###1.2 幕后的Spring Boot

之前的Spring MVC应用需要编写相关的XML和Java注解配置类

1．初始化 Spring MVC 的 DispatcherServlet； 

2．搭建转码过滤器，保证客户端请求进行正确地转码；

3．搭建视图解析器（view resolver），告诉 Spring 去哪里查找视图，以及它们是使用哪种方言编写的（JSP、Thymeleaf 模板等）；

4．配置静态资源的位置（CSS、JS）；

5．配置所支持的地域以及资源 bundle； 

6．配置 multipart 解析器，保证文件上传能够正常工作；

7．将 Tomcat 或 Jetty 包含进来，从而能够在 Web 服务器上运行我们的应用；

8．建立错误页面（如 404）。

Spring Boot 为我们处理了所有的事情。Spring Boot 是带有一定倾向性的 Spring 项目配置器。它基于约定，并且默认会在你的项目中使用这些约定。

#### 1.2.1 DispatcherServlet分发器和 multipart 配置

在Spring Boot的application.properties中添加debug = true 就可以在启动时看到Spring Boot的自动配置报告。

分为两部分：positive matches 列出了应用中所有的自动配置，negative matches 没有满足的Spring Boot自动配置

#####DispatcherServletAutoConfiguration 配置了dispatcherServlet和multipart

​	@AutoConfigureOrder(-2147483648）声明优先级

​	@ConditionalOnWebApplication  运行在web应用里

​	@ConditionalOnClass({DispatcherServlet.class}) 确保类路径下包含DispatcherServlet

​	@Bean(name = {"dispatcherServlet"})  public DispatcherServlet dispatcherServlet()

​    @Bean 

​	@ConditionalOnBean({MultipartResolver.class}) 

​	@ConditionalOnMissingBean(name = {"multipartResolver"}) 只有满足有MultipartResolver.class没有multipartResolver bean时才会激活该方法 MultipartResolver multipartResolver(MultipartResolver resolver)

```java
@AutoConfigureOrder(-2147483648)
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({DispatcherServlet.class})
@AutoConfigureAfter({EmbeddedServletContainerAutoConfiguration.class})
public class DispatcherServletAutoConfiguration {
    public static final String DEFAULT_DISPATCHER_SERVLET_BEAN_NAME ="dispatcherServlet";
    public static final String DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME = "dispatcherServletRegistration";
  
  
@Configuration  @Conditional({DispatcherServletAutoConfiguration.DefaultDispatcherServletCondition.class})
    @ConditionalOnClass({ServletRegistration.class})
    @EnableConfigurationProperties({WebMvcProperties.class})
    protected static class DispatcherServletConfiguration {
        private final WebMvcProperties webMvcProperties;

        public DispatcherServletConfiguration(WebMvcProperties webMvcProperties) {
            this.webMvcProperties = webMvcProperties;
        }

        @Bean(name = {"dispatcherServlet"})
        public DispatcherServlet dispatcherServlet() {
            DispatcherServlet dispatcherServlet = new DispatcherServlet();
            dispatcherServlet.setDispatchOptionsRequest(this.webMvcProperties.isDispatchOptionsRequest());
            dispatcherServlet.setDispatchTraceRequest(this.webMvcProperties.isDispatchTraceRequest());
            dispatcherServlet.setThrowExceptionIfNoHandlerFound(this.webMvcProperties.isThrowExceptionIfNoHandlerFound());
            return dispatcherServlet;
        }

        @Bean
        @ConditionalOnBean({MultipartResolver.class})
        @ConditionalOnMissingBean(
            name = {"multipartResolver"}
        )
        public MultipartResolver multipartResolver(MultipartResolver resolver) {
            return resolver;
        }
    }
  
```

####1.2.2 试图解析器、静态资源及区域配置

##### WebMvcAutoConfiguration 声明了视图解析器、地域解析器（localeresolver）及静态资源的位置

```java
@Configuration
@Import({WebMvcAutoConfiguration.EnableWebMvcConfiguration.class})
@EnableConfigurationProperties({WebMvcProperties.class, ResourceProperties.class})
public static class WebMvcAutoConfigurationAdapter extends WebMvcConfigurerAdapter {
    private static final Log logger = LogFactory.getLog(WebMvcConfigurerAdapter.class);
    private final ResourceProperties resourceProperties;
    private final WebMvcProperties mvcProperties;
    private final ListableBeanFactory beanFactory;
    private final HttpMessageConverters messageConverters;
    final WebMvcAutoConfiguration.ResourceHandlerRegistrationCustomizer resourceHandlerRegistrationCustomizer;
  @Bean
  @ConditionalOnMissingBean
  public InternalResourceViewResolver defaultViewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix(this.mvcProperties.getView().getPrefix());
    resolver.setSuffix(this.mvcProperties.getView().getSuffix());
    return resolver;
  }
```

######试图解析器的配置

view类的prefix字段 匹配**application.properties**中的	**spring.mvc.view.prefix**

view类的prefix字段 匹配**application.properties**中的	**spring.mvc.view.prefix**

view类的suffix字段 匹配**application.properties**中的	**spring.mvc.view.suffix**

```java
@ConfigurationProperties(
    prefix = "spring.mvc"
)
public class WebMvcProperties {
    private Format messageCodesResolverFormat;
    private Locale locale;
    private WebMvcProperties.LocaleResolver localeResolver;
    private String dateFormat;
    private boolean dispatchTraceRequest;
    private boolean dispatchOptionsRequest;
    private boolean ignoreDefaultModelOnRedirect;
    private boolean throwExceptionIfNoHandlerFound;
    private boolean logResolvedException;
    private Map<String, MediaType> mediaTypes;
    private String staticPathPattern;
    private final WebMvcProperties.Async async;
    private final WebMvcProperties.Servlet servlet;
    private final WebMvcProperties.View view;
  public static class View {
      private String prefix;
      private String suffix;
  }
}
```

###### 静态资源

对带有“webjar”前缀的资源访问将会在类路径中解析。这样的话，我们就能使用 Mavan 中央仓库中预先打包好的 JavaScript 依赖；

> ​	WebJars 是 JAR 包格式的客户端 JavaScript 库，可以通过 Maven中央仓库来获取。它们包含了Maven项目文件，这个文件允许定义传递性依赖，能够用于所有基于 JVM的应用之中。WebJars 是 JavaScript 包管理器的替代方案，如 bower 或 npm。对于只需要较少 JavaScript 库的应用来说，这种方案是很棒的。你可以在 www.webjars.org 站点上看到所有可用的 WebJars 列表。

我们的静态资源需要放在类路径中，并且要位于以下 4 个目录中的任意一个之中，

```java
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties implements ResourceLoaderAware {
    private static final String[] SERVLET_RESOURCE_LOCATIONS = new String[]{"/"};
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
    private static final String[] RESOURCE_LOCATIONS;
    private String[] staticLocations;
    private Integer cachePeriod;
    private boolean addMappings;
    private final ResourceProperties.Chain chain;
    private ResourceLoader resourceLoader;
 
   public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Integer cachePeriod = this.resourceProperties.getCachePeriod();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(cachePeriod));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(this.resourceProperties.getStaticLocations()).setCachePeriod(cachePeriod));
                }

            }
        }
```

###### 地域管理

默认的地域解析器只会处理一个地域，并且允许我们通过spring.mvc.locale配置属性进行定义

```java
@Bean
@ConditionalOnMissingBean
@ConditionalOnProperty(
    prefix = "spring.mvc",
    name = {"locale"}
)
public LocaleResolver localeResolver() {
    if (this.mvcProperties.getLocaleResolver() == org.springframework.boot.autoconfigure.web.WebMvcProperties.LocaleResolver.FIXED) {
        return new FixedLocaleResolver(this.mvcProperties.getLocale());
    } else {
        AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
        localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
        return localeResolver;
    }
}
```

###1.3 错误与转码配置

在没有添加控制器的时候，会看到“Whitelabel Error Page”输出。 Spring Boot 的事情**ErrorMvcAutoConfiguration **处理这些事情

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({Servlet.class, DispatcherServlet.class})
@AutoConfigureBefore({WebMvcAutoConfiguration.class})
@EnableConfigurationProperties({ResourceProperties.class})
public class ErrorMvcAutoConfiguration {
    private final ServerProperties serverProperties;
    private final List<ErrorViewResolver> errorViewResolvers;
    
    @Configuration
    @ConditionalOnProperty(prefix = "server.error.whitelabel",name = {"enabled"}, matchIfMissing = true)
    @Conditional({ErrorMvcAutoConfiguration.ErrorTemplateMissingCondition.class})
    protected static class WhitelabelErrorViewConfiguration {
        private final ErrorMvcAutoConfiguration.SpelView defaultErrorView = new ErrorMvcAutoConfiguration.SpelView("<html><body><h1>Whitelabel Error Page</h1><p>This application has no explicit mapping for /error, so you are seeing this as a fallback.</p><div id='created'>${timestamp}</div><div>There was an unexpected error (type=${error}, status=${status}).</div><div>${message}</div></body></html>");
        protected WhitelabelErrorViewConfiguration() {}
        @Bean(name = {"error"})
        @ConditionalOnMissingBean(name = {"error"})
        public View defaultErrorView() {return this.defaultErrorView; }
```

error错误页面配置路径

```java
@ConfigurationProperties(
    prefix = "server",
    ignoreUnknownFields = true
)
public class ServerProperties implements EmbeddedServletContainerCustomizer, EnvironmentAware, Ordered {
    private Integer port;
    private InetAddress address;
    private String contextPath;
    private String displayName = "application";
    @NestedConfigurationProperty
    private ErrorProperties error = new ErrorProperties();
    private String servletPath = "/";
    private final Map<String, String> contextParameters = new HashMap();
    private Boolean useForwardHeaders;
    private String serverHeader;
    private int maxHttpHeaderSize = 0;
    private int maxHttpPostSize = 0;
    private Integer connectionTimeout;
    private ServerProperties.Session session = new ServerProperties.Session();
    @NestedConfigurationProperty
    private Ssl ssl;
    @NestedConfigurationProperty
    private Compression compression = new Compression();
    @NestedConfigurationProperty
    private JspServlet jspServlet;
    private final ServerProperties.Tomcat tomcat = new ServerProperties.Tomcat();
    private final ServerProperties.Jetty jetty = new ServerProperties.Jetty();
    private final ServerProperties.Undertow undertow = new ServerProperties.Undertow();
    private Environment environment;
```

```java
public class ErrorProperties {
    @Value("${error.path:/error}")
    private String path = "/error";
    private ErrorProperties.IncludeStacktrace includeStacktrace;
    public static enum IncludeStacktrace {
          NEVER,
          ALWAYS,
          ON_TRACE_PARAM;
          private IncludeStacktrace() {
          }
      }
```

这段配置都做了些什么呢？

- 定义了一个 bean，即 DefaultErrorAttributes，它通过特定的属性暴露了有用的错误信息，这些属性包括状态、错误码和相关的栈跟踪信息。
- 定义了一个 BasicErrorController bean，这是一个 MVC 控制器，负责展现我们所看到的错误页面。
- 允许我们将 Spring Boot 的 whitelabel 错误页面设置为无效，这需要将配置文件application.properties 中的 server.error.whitelabel.enabled 设置为 false。
- 我们还可以借助模板引擎提供自己的错误页面。例如，它的名字是 error.html，ErrorTemplateMissingCondition 条件会对此进行检查。

转码配置

**HttpEncodingAutoConfiguration** 会负责处理相关的事宜，这是通过提供 Spring 的 CharacterEncodingFilter 类来实现的。通过 spring.http.encoding.charset配置，我们可以覆盖默认的编码（“UTF-8”），也可以通过 spring.http.encoding.enabled 禁用这项配置。

```java
@Configuration
@EnableConfigurationProperties({HttpEncodingProperties.class})
@ConditionalOnWebApplication
@ConditionalOnClass({CharacterEncodingFilter.class})
@ConditionalOnProperty(
    prefix = "spring.http.encoding",
    value = {"enabled"},
    matchIfMissing = true
)
public class HttpEncodingAutoConfiguration {
    private final HttpEncodingProperties properties;
    @Bean
    @ConditionalOnMissingBean({CharacterEncodingFilter.class})
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
        return filter;
    }
```

```java
@ConfigurationProperties(
    prefix = "spring.http.encoding"
)
public class HttpEncodingProperties {
    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
    private Charset charset;
    private Boolean force;
    private Boolean forceRequest;
    private Boolean forceResponse;
    private Map<Locale, Charset> mapping;
    public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
        this.properties = properties;
    }
```
### 1.4 嵌入式Servlet容器Tomcat的配置

默认情况下，Spring Boot 在打包和运行应用时，会使用 Tomcat 嵌入式 API（Tomcat embedded API）。

我们来看一下 EmbeddedServletContainerAutoConfiguration,包含了 3 个不同的配置，哪一个会处于激活状态

要取决于类路径下哪些内容是可用的。

> 可以将 Spring Boot 与 Tomcat、tc-server、Jetty 或者 Undertow 结合使用。服务器可以很容易地进行替换，只需将 spring-boot-starter-tomcat JAR 依赖移除掉，并将其替换为 Jetty或 Undertow 对应的依赖即可。
>

```java
@AutoConfigureOrder(-2147483648)
@Configuration
@ConditionalOnWebApplication
@Import({EmbeddedServletContainerAutoConfiguration.BeanPostProcessorsRegistrar.class})
public class EmbeddedServletContainerAutoConfiguration {
    public EmbeddedServletContainerAutoConfiguration() {}
 @Configuration
    @ConditionalOnClass({Servlet.class, Undertow.class, SslClientAuthMode.class})
    @ConditionalOnMissingBean(
        value = {EmbeddedServletContainerFactory.class},
        search = SearchStrategy.CURRENT
    )
    public static class EmbeddedUndertow {
        public EmbeddedUndertow() {
        }

        @Bean
        public UndertowEmbeddedServletContainerFactory undertowEmbeddedServletContainerFactory() {
            return new UndertowEmbeddedServletContainerFactory();
        }
    }

    @Configuration
    @ConditionalOnClass({Servlet.class, Server.class, Loader.class, WebAppContext.class})
    @ConditionalOnMissingBean(
        value = {EmbeddedServletContainerFactory.class},
        search = SearchStrategy.CURRENT
    )
    public static class EmbeddedJetty {
        public EmbeddedJetty() {
        }

        @Bean
        public JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory() {
            return new JettyEmbeddedServletContainerFactory();
        }
    }

    @Configuration
    @ConditionalOnClass({Servlet.class, Tomcat.class})
    @ConditionalOnMissingBean(
        value = {EmbeddedServletContainerFactory.class},
        search = SearchStrategy.CURRENT
    )
    public static class EmbeddedTomcat {
        public EmbeddedTomcat() {
        }

        @Bean
        public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
            return new TomcatEmbeddedServletContainerFactory();
        }
    }
```

对 Servlet 容器（Tomcat）的所有配置都会在 TomcatEmbeddedServletContainerFactory中进行。它为嵌入式 Tomcat 提供一个非常高级的配置（为其查找文档会非常困难）

```java
public class TomcatEmbeddedServletContainerFactory extends AbstractEmbeddedServletContainerFactory implements ResourceLoaderAware {
    private static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
    private static final Set<Class<?>> NO_CLASSES = Collections.emptySet();
    public static final String DEFAULT_PROTOCOL = "org.apache.coyote.http11.Http11NioProtocol";
    private File baseDirectory;
    private List<Valve> engineValves = new ArrayList();
    private List<Valve> contextValves = new ArrayList();
    private List<LifecycleListener> contextLifecycleListeners = new ArrayList();
    private List<TomcatContextCustomizer> tomcatContextCustomizers = new ArrayList();
    private List<TomcatConnectorCustomizer> tomcatConnectorCustomizers = new ArrayList();
    private List<Connector> additionalTomcatConnectors = new ArrayList();
    private ResourceLoader resourceLoader;
    private String protocol = "org.apache.coyote.http11.Http11NioProtocol";
    private Set<String> tldSkipPatterns;
    private Charset uriEncoding;
    private int backgroundProcessorDelay;
```

####1.4.1 HTTP端口

通过在 application.properties 文件中定义 server.port 属性或者定义名为 SERVER_PORT的环境变量，我们可以修改默认的 HTTP 端口。

通过将该变量设置为−1，可以禁用 HTTP，或者将其配置为 0，这样的话，就会在随机的端口上启动应用。对于测试，这是很便利的。

####1.4.2 SSL端口

 Spring Boot 只需一些属性就能保护服务器了，不过需要先生成keystore文件

​	**server.port = 8443**

​	**server.ssl.key-store = classpath:keystore.jks**

​	**server.ssl.key-store-password = secret**

​	**server.ssl.key-password = another-secret**

第 6 章中，深入介绍安全的可选方案。还可以添加自己的EmbeddedServletContainerFactory 来进一步自定义 TomcatEmbeddedServletContainerFactory 的功能。如果希望添加多个连接器，非常便利，可以参考 http://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-servlet-containers.html#howto-configuressl 来获取更多信息。

####1.4.3 其他配置

在配置中，我们可以通过简单地声明@Bean 元素来添加典型的 Java Web 元素，如Servlet、Filter 和 ServletContextListener。除此之外，Spring Boot 还为我们内置了 3 项内容：

 在 JacksonAutoConfiguration 中，声明使用 Jackson 进行 JSON 序列化；

 在 HttpMessageConvertersAutoConfiguration 中，声明了默认的 HttpMessageConverter；

 在 JmxAutoConfiguration 中，声明了 JMX 功能。

关于 JMX 配置，我们可以在本地通过 jconsole 连接应用之后进行尝试，

> 通过将 org.springframework.boot:spring-boot-starter-actuator 添加到类路径下，我们可以
>
> 添加更多有意思的 MBean。我们甚至可以定义自己的 MBean，并通过 Jolokia 将其暴露为
>
> HTTP。另一方面，我们也可以禁用这些端点，只需在配置中添加 spring.jmx.enabled=false
>
> 即可。http://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-jmx.html 了解更多细节。



##第二章 精通MVC架构

###2.1 MVC架构

MVC 代表的是模型（Model）、视图（View）和控制器（Controller）,将数据和展现层进行解耦，被视为构建用户界面的一种很流行的方式。

它的架构可以分为 3 层。

​	** 模型：包含了应用中所需的各种展现数据。**

​	** 视图：由数据的多种表述所组成，它将会展现给用户。**

​	** 控制器：将会处理用户的操作，它是连接模型和视图的桥梁。**

MVC 的理念是将视图与模型进行解耦，模型是自包含的与 UI 无关。就可以实现相同的数据跨多个视图重用。其实，这些视图就是以不同的方式来查看数据。

控制器会作为用户和数据的中间协调者，它的角色就是控制终端用户的可用行为，并引导他们在应用的不同视图间跳转。

###2.2 对MVC的质疑及其最佳实践

#### 2.21贫血的领域模型

Eric Evans 编的《领域驱动设计》（Domain Driven Design，DDD）书中，定义了一组架构规则，能够指导我们更好地将业务领域集成到代码之中。

​	其中有一项核心的理念就是将面向对象的范式应用到领域对象之中。如果违背这一原则的话，就会被称之为贫血的领域模型（Anemic Domain Model）。

贫血的领域模型通常来讲会具有如下的症状：

​	 模型是由简单老式的 Java 对象（plain old Java object，POJO）所构成的，只有getter 和 setter 方法；

​	 所有业务逻辑都是在服务层处理的；

​	 对模型的校验会在本模型外部进行，例如在控制器中。

​	根据业务领域的复杂性不同，这可能是一种较差的实践方式。通常来讲，DDD 实践需要付出额外的努力，将领域从应用逻辑中分离出来。

​	架构通常都是一种权衡，需要注意的是，设计 Spring 应用的典型方式往往会在这个过程中导致系统在可维护性上变得较为复杂。

避免领域贫血的途径如下：

​	 服务层适合进行应用级别的抽象（如事务处理），而不是业务逻辑；

​	 领域对象应该始终处于合法的状态。通过校验器（validator）或 JSR-303 的校验注解，让校验过程在表单对象中进行；

​	 将输入转换成有意义的领域对象；

​	 将数据层按照 Repository 的方式来实现，Repository 中会包含领域查询（例如参考Spring Data 规范）；

​	 将领域逻辑与底层的持久化框架解耦；

​	 尽可能使用实际的对象，例如操作 FirstName 类而不是操作 String。

​	DDD 所涉及的内容远不止上述的规则：实体（Entity）、值类型（value type）、通用语言（Ubiquitous Language）、限界上下文（Bounded Context）、洋葱架构（Onion Architecture）以及防腐化层（anti corruption layer）。

#### 2.2.2 spring官网的源码

 http://spring.io它全部是由 Spring 构建的，而且很棒的一点在于它是开源的。

这个项目的名称为 sagan，它包含了很多有意思的特性：

​	 基于 Gradle 的多模块项目；

​	 集成了安全；

​	 集成了 Github； 

​	 集成了 Elasticsearch； 

​	 JavaScript 前端应用。

这个项目的 Github wiki 页面非常详尽，能够帮助你非常容易地开始了解该项目。

> 如果你对这个实际应用的 Spring 架构感兴趣的话，可以访问如下的 URL：https://github.com/spring-io/sagan

### 2.3 Spring MVC 1-0-1

​	Spring MVC 中，模型是由 Spring MVC 的 Model 或 ModelAndView 封装的简单 Map。它可以来源于数据库、文件、外部服务等，这取决于你如何获取数据并将其放到模型中。

> ​	与数据层进行交互的推荐方式是使用 Spring Data 库：Spring Data JPA、Spring Data MongoDB 等。有 10 多个与 Spring Data 相关的项目，推荐你查看一下 http://projects.spring.io/spring-data。

​	Spring MVC 的控制层是通过使用@Controller 注解来进行处理的。在 Web 应用中，控制器的角色是响应 HTTP 请求。带有@Controller 注解的类将会被 Spring 检索到，并且能够有机会处理传入的请求。

​	使用@RequestMapping 注解，控制器能够声明它们会根据 HTTP 方法（如 GET 或POST 方法）和 URL 来处理特定的请求。控制器就可以确定是在 Web 响应中直接写入内容，还是将应用路由一个视图并将属性注入到该视图中。

​	纯粹的 RESTful 应用将会选择第一种方式，在 Web 响应中直接写入内容(JSON)，并且会在 HTTP 响应中直接暴露模型的JSON 或 XML 表述，这需要用到@ResponseBody 注解。在 Web 应用中，这种类型的架构通常会与前端 JavaScript 框架关联.

> ​	如 Backbone.js、AngularJS 或 React。在这种场景中，Spring 应用只需处理 MVC 中的模型层。我们将会在第 4 章中学习这种架构。

​	在第二种方式中，模型会传递到视图中，视图会由模板引擎进行渲染，并写入到响应之中。

​	视图通常会与某种模板方言关联，这种模板允许遍历模型中的内容，流行的模板方言包括 JSP、FreeMarker 或 Thymeleaf。

​	混合式的方式则会在某些方面采用模板引擎与应用进行交互，并将视图层委托给前端框架。

### 2.4 使用Thymeleaf

Thymeleaf 是一个模板引擎，成功很大程度上要归因于对用户友好的语法（它几乎就是 HTML）以及扩展的

便利性。

> Thymeleaf 与 Spring 集成入门指南 http://www.thymeleaf.org/doc/tutorials/2.1/thymeleafspring.html

| 所支持的功能                 | 依赖                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| 布局                         | nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect             |
| HTML 5 data-*属性            | com.github.mxab.thymeleaf.extras:thymeleaf-extras-data-attribute |
| Internet Explorer 的条件注释 | org.thymeleaf.extras:thymeleaf-extras-conditionalcomments    |
| 对 Spring Security 的支持    | org.thymeleaf.extras:thymeleaf-extras-springsecurity3        |

增加 **thymeleaf** 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

新建页面	**resultPage**

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8"/>
    <title>Hello thymeleaf</title>
</head>
<body>
    <span th:text="|Hello thymeleaf|">Hello html</span>
</body>
</html>
```

配置文件 **application.properties**

```properties
spring.thymeleaf.cache=false
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.suffix: .html
spring.thymeleaf.mode: HTML5
spring.thymeleaf.servlet.content-type: text/html
```

控制器

```java
@Controller
public class HelloWorldController {
    @RequestMapping("/")
    public String page() {
        return "resultPage";
    }
 }
```

### 2.5 Spring MVC架构

#### 2.5.1 DispatcherServlet

每个Spring Web应用的入口都是DispatcherServlet

![WX20200415-131840@2x](image/WX20200415-131840@2x.png)

​		这个一个典型的 HttpServlet 类，它会将 HTTP 请求分发给 HandlerMapping。 HandlerMapping 将资源（URL）与控制器关联起来。

​		控制器上对应的方法（也就是带有@RequestMapping 注解的方法）将会被调用。在这个方法中，控制器会设置模型数据并将视图名称返回给分发器。

​		然后，DispatcherServlet 将会查询 ViewResolver 接口，从而得到对应视图的实现。

​		在样例中，ThymeleafAutoConfiguration 将会为我们搭建视图解析器。通过查看 ThymeleafProperties 类，可以知道视图的默认前缀是“classpath:/templates/”，后缀是“.html”。

> ​		这就意味着，假设视图名为 resultPage，那么视图解析器将会在类路径的 templates 目录下查找名为 resultPage.html 的文件。

​	   在我们的应用中，ViewResolver 接口是静态的，但是更为高级的实现能够根据请求的头信息或用户的地域信息，返回不同的结果。

​	  视图最终将会被渲染，其结果会写入到响应之中。

#### 2.5.2 将数据传递给视图

修改控制器，将信息保存到模型中

```java
@RequestMapping("/")
public String message(Model model) {
    model.addAttribute("message","Hello from the controller");
    return "resultPage";
}
```

修改thymeleaf页面

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8"/>
    <title>Hello thymeleaf</title>
</head>
<body>
    <span th:text="${message}">Hello html</span>
</body>
</html>
```

 DispatcherServlet 会提供正确的对象到控制器的方法中。实际上，很多对象都可以注入到控制器的方法之中，例如 HttpRequest、HttpResponse、Locale、TimeZone 和 Principal，其中 Principal 代表了一个认证过的用户。

> 完整的对象列表https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-arguments

#### 2.6 Spring表达式语言

“${}”是 Spring 表达式语言（Spring Expression Language，SpEL）。关于 EL，有多个不同的变种，而 SpEL 是其中威力强大的一种。

| 特性          | 语法                          | 描述                                           |
| ------------- | ----------------------------- | ---------------------------------------------- |
| 访问列表元素  | list[0]                       |                                                |
| 访问 Map 条目 | map[key]                      |                                                |
| 三元运算符    | condition ? 'yes' :'no'       |                                                |
| Elvis 运算符  | person ?: default             | 如果 person 的值为空的话，将会返回 default     |
| 安全导航      | person?.name                  | 如果 person 为空或其 name 为空的话，返回 null  |
| 模板          | 'Your name is #{person.name}' | 将值注入到字符串中                             |
| 投影          | ${persons.![name]}            | 抽取所有 persons 的 name，并将其放到一个列表中 |
| 选择          | persons.?[name =='Bob']'      | 返回列表中 name 为 Bob 的 person               |
| 函数调用      | person.sayHello()             |                                                |

http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html

​	SpEL 的用处并不仅限于视图之中，可以将它用在 Spring 框架的各种地方，例如，在通过@Value 注解往 bean 中注入属性时，也可以使用 SpEL。

#####从请求参数中获取数据

根据 HTTP 协议，有很多方式可以从请求参数中获取数据，最简单的就是传递查询参数到 URL 之中

> ​	查询参数位于 URL的“?”字符后面，由名称和值所组成的列表，每一项会使用“&”符号进行分割，例如：page?var1=value1&var2=value2。

```java
@Controller 
public class HelloController {
 @RequestMapping("/") 
 public String hello(@RequestParam("name") String userName, Model model) { 
		model.addAttribute("message", "Hello, " + userName); 
	return "resultPage"; 
 }
}
```

​	默认，请求参数是强制要求存在的。@RequestParam 还有其他两个可用的属性：required 和 defaultValue。 可以为其指定一个默认值或者将其设置为非必填项。

```java
@Controller 
public class HelloController { 
 @RequestMapping("/") 
 public String hello(@RequestParam(defaultValue = "world") String name, Model model) { 
		model.addAttribute("message", "Hello, " + name); 
	return "resultPage"; 
 }
} 
```

#### 2.8 Java 8 的流和lambda表达式

Java 8 中，每个集合都会有一个默认的方法 stream()，它能够实现函数式风格的操作。

​	中间操作（intermediate_operation）会返回一个流，能将其连接起来，终止操作（terminal operation）会返回一个值。

```java
users.stream().filter(user -> user.getName().equals(search)).collect(Collectors.toList())
```

最著名的中间操作：

​	 map：它会为列表中的每个元素都应用某个方法，并返回结果所组成的列表；

​	 filter：它会返回匹配断言的所有元素；

​	 reduce：它会借助一个操作和累加器（accumulator）将一个列表聚合到单个值上。

Lambda 是函数表达式的便捷语法，它可以用到单个的抽象方法（Single Abstract Method）之中，也就是只包含一个函数的接口。

例如，我们可以按照如下的方式来实现 Comparator 接口：

​		Comparator<Integer> c = (e1, e2) -> e1 - e2; 

在 lambda 之中，return 关键字就是最后的表达式。

双冒号操作符是引用类方法的快捷方式：

​		Tweet::getText  =  (Tweet t) -> t.getText() 

collect 方法允许我们调用一个终止操作。Collectors 类是一组终止操作，它会将结果放到列表、集合或 Map 之中，允许进行分组（grouping）、连接（joining）等操作。

调用 collect(Collectors.toList())方法将会产生一个列表，其中包含了流中的每一个元素。

#### 2.9 使用WebJars实现质感设计

使用 Materialize（http://materializecss.com），这是一个非常漂亮的 CSS 和 JavaScript 库，与 Bootstrap 类似

引入依赖

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>materializecss</artifactId>
    <version>0.96.0</version>
</dependency>
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>2.1.1</version>
</dependency>
```

每个 WebJar 的结构都是标准的，每个库的 JS 和 CSS 文件都会位于/webjars/{lib}/{version}/*.js 中

修改控制器

```java
@RequestMapping("/")
public String message(Model model) {
    model.addAttribute("message","Hello from the controller");
    List<User> users = new ArrayList<>();
    users.add(new User(1,"tom",23));
    users.add(new User(2,"joy",25));
    model.addAttribute("users",users);
    return "resultPage";
}
```

 页面显示

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head lang="en">
    <meta charset="UTF-8"/>
    <title>Hello thymeleaf</title>
</head>
<body>
    <span th:text="${message}">Hello html</span>
        <div class="row" layout:fragment="content">
    <ul class="collection">
        <li class="collection-item avatar" th:each="user:${users}">
            <span class="title" th:text="${user.id}">id</span>
            <span class="title" th:text="${user.name}">name</span>
            <span class="title" th:text="${user.age}">age</span>
        </li>
    </ul>
        </div>
    <script src="/webjars/jquery/2.1.4/jquery.js"></script>
    <script src="/webjars/materializecss/0.96.0/js/materialize.js"/>
</body>
</html>
```

##### 2.9.1 使用布局

​	需要使用 thymeleaf-layout-dialect 依赖，它已经包含在项目的 spring-boot-starter-thymeleaf依赖里面，将 UI 中可重用的代码块放到模板之中。

创建文件名为 default.html，并将其放在 src/main/resources/templates/layout 之中

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
    <head>
        <meta http-equiv="Content-Type" content="text/html;
charset=UTF-8"/>
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1.0, user-scalable=no"/>
        <title>Default title</title>
        <link href="/webjars/materializecss/0.96.0/css/materialize.css"
              type="text/css" rel="stylesheet" media="screen,projection"/>
    </head>
    <body>
        <section layout:fragment="content">
            <p>Page content goes here</p>
        </section>
        <script src="/webjars/jquery/2.1.4/jquery.js"></script>
        <script src="/webjars/materializecss/0.96.0/js/materialize.js"/>
    </body>
</html>
```

修改显示页面

​	layout:decorator="layout/default"能够表明去哪里查找布局。这样，我们可以将内容注入到布局的不同 layout:fragment 区域中。

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorator="layout/default">
<head lang="en">
    <meta charset="UTF-8"/>
    <title>Hello thymeleaf</title>
</head>
<body>
    <span th:text="${message}">Hello html</span>
        <div class="row" layout:fragment="content">
    <ul class="collection">
        <li class="collection-item avatar" th:each="user:${users}">
            <span class="title" th:text="${user.id}">id</span>
            <span class="title" th:text="${user.name}">name</span>
            <span class="title" th:text="${user.age}">age</span>
        </li>
    </ul>
        </div>
</body>
</html>
```

#### 2.9.2 导航

增加一个搜索页面 searchPage.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.w3.org/1999/xhtml"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"
      layout:decorator="layout/default">
<head lang="en">
    <title>Search</title>
</head>
<body>
<div class="row" layout:fragment="content">
    <h4 class="indigo-text center">Please enter a search term</h4>
    <div class="col s6 offset-s3">
        <div id="errorMessage" class="card-panel red lighten-2"
             th:if="${error}">
            <span class="card-title" th:text="${error}"></span>
        </div>
        <form action="/postSearch" method="post" class="col s12">
            <div class="row center">
                <div class="input-field">
                    <i class="mdi-action-search prefix"></i>
                    <input id="search" name="search" type="text"
                           class="validate"/>
                    <label for="search">Search</label>
                </div>
            </div>
        </form>
    </div>
</div>
</body>
</html>
```

控制器

```java
List<User> users = new ArrayList<>();

@RequestMapping("/")
public String message() {
    return "searchPage";
}

@RequestMapping("/postSearch")
public String postSearch(HttpServletRequest request, RedirectAttributes redirectAttributes) {
    String search = request.getParameter("search");
    users.add(new User(1,"tom",23));
    users.add(new User(2,"joy",25));
    if(users.stream().noneMatch(user -> user.getName().equals(search))){
      redirectAttributes.addFlashAttribute("error", "Try using spring instead!");
    	return "redirect:/";
    }
    redirectAttributes.addAttribute("search", search);
        return "redirect:result";
}

@RequestMapping("/result")
public String result(@RequestParam(defaultValue = "tom")String search,Model model) {
    users.add(new User(1,"tom",23));
    users.add(new User(2,"joy",25));
    users = users.stream().filter(user -> user.getName().equals(search)).collect(Collectors.toList());
    model.addAttribute("users",users);
    model.addAttribute("search",search);
    return "resultPage";
}
```

​	 在请求映射的注解中，指定了想要处理的 HTTP 方法，也就是 POST； 

​	 在方法参数中，直接注入了两个属性，它们是 request 和 RedirectAttributes； 

​	 检索到请求中 post 提交过来的数据，并将其传递给下一个视图；

​	 现在不是直接返回视图的名称，而是重定向到另一个 URL。

如果用户的搜索不是“tom、joy”这个术语的话，我们会将其重定向到 searchPage 页面并且使用 flash 属性添加一点错误信息。

redirectAttributes.addFlashAttribute/addAttribute这个特殊的属性会在该请求的时间范围内一直存活，直到页面渲染完成才消失。当使用 POST-REDIRECT-GET 模式的时候，它是非常有用的。



#SpringMVC路由

​	写好了Controller和对应的方法，加上@RequestMapping注解，MVC是如何根据请求路径找到对应的Controller和Controller中具体的Method

**HandlerMapping**

1. SimpleUrlHandlerMapping
2. BeanNameUrlHandlerMapping
3. RequestMappingHandlerMapping

HandlerMapping就是处理请求path和具体handler映射关系的组件，我们将从最基础到SimpleUrlHandlerMapping到目前MVC中默认优先使用使用的RequestMappingHandlerMapping来进行讲解，如下实验中，我们假设已经在web.xml中配置了DispatcherServlet这个基础的servlet，且每次实验时DispatcherServlet会加载对应实验中的配置文件.

一、SimpleUrlHandlerMapping
bean 配置

```
 1<?xml version="1.0" encoding="UTF-8"?>
 2<beans xmlns="http://www.springframework.org/schema/beans"
 3       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 4       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
 5    <bean id="simpleController" class="com.seven.springmvc.controller.SimUrlController"></bean>
 6    <bean id="simpleUrlHandlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
 7        <property name="mappings">
 8            <props>
 9                <prop key="/simple">simpleController</prop>
10            </props>
11        </property>
12    </bean>
13</beans>
```

Controller 代码

```
1public class SimUrlController implements  Controller {
2    @Override
3    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
4        PrintWriter writer = response.getWriter();
5        writer.print("hi simple");
6        return null;
7    }
8}
```

运行结果

```
1请求：http://localhost:8080/simple
2输出  hi simple
```

二、BeanNameUrlHandlerMapping
bean 配置

```
1<?xml version="1.0" encoding="UTF-8"?>
2<beans xmlns="http://www.springframework.org/schema/beans"
3       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
4       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
5    <bean id="/beanNameController" class="com.seven.springmvc.controller.BeanNameController"/>
6    <bean id="beanNameUrlHandlerMapping" class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
7</beans>
```

Controller 代码

```
1public class BeanNameController implements Controller {
2    @Override
3    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
4        PrintWriter writer = response.getWriter();
5        writer.print("hi beanName");
6        return null;
7    }
8}
```

运行结果

```
1请求：http://localhost:8080/beanNameController
2输出：hi beanName
```

三、RequestMappingHandlerMapping
bean 配置

```
1<?xml version="1.0" encoding="UTF-8"?>
2<beans xmlns="http://www.springframework.org/schema/beans"
3       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
4       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
5    <bean id="controller" class="com.seven.springmvc.controller.RequestMappingController"/>
6    <bean id="beanNameUrlHandlerMapping" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
7</beans>
```

Controller 代码

```
1@Controller
2public class RequestMappingController {
3    @RequestMapping("/index")
4    @ResponseBody
5    public String index(){
6        return "hi requestMapping";
7    }
8}
```

------

运行结果

```
1请求：http://localhost:8080/requestMapping
2输出：hi requestMapping
```

**结论**

1. SimpleUrlHandlerMapping 是根据其mappings属性中配置的key-value来定义映射关系的,根据请求path【作为映射关系中的key】，找到对应的Controller【此时只要找到Controller就可以了，不用到具体方法，因为自定义的Controller必须继承Controller接口，并实现handleRequest方法，最后也是执行handleRequest方法】
2. BeanNameUrlHandlerMapping 是根据spring容器中配置的bean名称【注意bean的名称需要以'/'开头】和Controller来定义映射关系的， 根据请求路径path去查找beanName和path一致的bean【其中bean必须继承Controller并实现handleRequest方法】，然后调用handleRequest来处理请求
3. RequestMappingHandlerMapping 是依赖注解RequestMapping来实现path和Hander的映射关系的，具体是根据注解中的value值作为key，注解所在方法作为Handler设置映射关系，然后请求到达时则根据请求path去查找对应的Handler【具体Controller中的Method】

开发中大多使用注解【RequestMapping】的方式实现路由的映射，也就是用RequestMappingHandlerMapping来实现的，关于RequestMappingHandlerMapping的具体初始化和查询的功能是如何根据注解来完成路由的映射的，

##RequestMappingHandlerMapping

**关系梳理**

1. spring ioc 是spring的核心，用来管理spring bean的生命周期
2. MVC 是一种使用 MVC（Model View Controller 模型-视图-控制器）设计创建 Web 应用程序的模式
3. spring mvc 是spring的一个独立的模块，就像AOP一样

在spring mvc中把web框架和spring ioc融合在一起，是通过ContextLoaderListener监听servlet上下文的创建后来加载父容器完成的，然后通过配置一个servlet对象DispatcherServlet，在初始化DispatcherServlet时来加载具体子容器，详细的可以参考[spring ioc & web宿主](https://mp.weixin.qq.com/s?__biz=MzU3MDQyODI1OA==&mid=2247483750&idx=1&sn=9d7d6fbc8a57f8f3ebcc27ca9829ce63&chksm=fceedf6bcb99567d9528266ab153573457cba396b893fe952f106ded41f0e4d4ab08af82e6f2#rd) 这篇文章

关于我们今天要讲的RequestMappingHandlerMapping也是在DispatcherServlet的初始化过程中自动加载的，默认会自动加载所有实现HandlerMapping接口的bean，且我们可以通过serOrder来设置优先级，系统默认会加载RequestMappingHandlerMapping、BeanNameUrlHandlerMapping、SimpleUrlHandlerMapping 并且按照顺序使用



```kotlin
private void initHandlerMappings(ApplicationContext context) {
        this.handlerMappings = null;

        if (this.detectAllHandlerMappings) {
            // Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
            Map<String, HandlerMapping> matchingBeans =
                    BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.handlerMappings = new ArrayList<>(matchingBeans.values());
                // We keep HandlerMappings in sorted order.
                AnnotationAwareOrderComparator.sort(this.handlerMappings);
            }
        }
}
```

**RequestMappingHandlerMapping 加载过程**

1. RequestMappingHandlerMapping 实现了接口InitializingBean，在bean加载完成后会自动调用afterPropertiesSet方法，在此方法中调用了initHandlerMethods()来实现初始化
2. 遍历所有bean，如果bean实现带有注解@Controller或者@RequestMapping 则进一步调用detectHandlerMethods处理，处理逻辑大致就是根据@RequestMapping配置的信息，构建RequestMappingInfo，然后注册到MappingRegistry中

```kotlin
protected void initHandlerMethods() {
    
        String[] beanNames = (this.detectHandlerMethodsInAncestorContexts ?
                BeanFactoryUtils.beanNamesForTypeIncludingAncestors(obtainApplicationContext(), Object.class) :
                obtainApplicationContext().getBeanNamesForType(Object.class));

        for (String beanName : beanNames) {
            if (!beanName.startsWith(SCOPED_TARGET_NAME_PREFIX)) {
                Class<?> beanType = null;
                beanType = obtainApplicationContext().getType(beanName);
                if (beanType != null && isHandler(beanType)) {
                    detectHandlerMethods(beanName);
                }
            }
        }
        handlerMethodsInitialized(getHandlerMethods());
    }
```

```dart
protected void detectHandlerMethods(final Object handler) {
        Class<?> handlerType = (handler instanceof String ?
                obtainApplicationContext().getType((String) handler) : handler.getClass());

        if (handlerType != null) {
            final Class<?> userType = ClassUtils.getUserClass(handlerType);
            Map<Method, T> methods = MethodIntrospector.selectMethods(userType,
                    (MethodIntrospector.MetadataLookup<T>) method -> {
                        try {
                            return getMappingForMethod(method, userType);
                        }
                        catch (Throwable ex) {
                            throw new IllegalStateException("Invalid mapping on handler class [" +
                                    userType.getName() + "]: " + method, ex);
                        }
                    });
            methods.forEach((method, mapping) -> {
                Method invocableMethod = AopUtils.selectInvocableMethod(method, userType);
                registerHandlerMethod(handler, invocableMethod, mapping);
            });
        }
    }
```

**RequestMappingHandlerMapping 解析过程**

1. 在DispatcherServlet中，根据请求对象调用getHander方法获取HandlerExecutionChain对象
2. 在getHander方法中也是遍历上面默认加载的三个HandlerMapping，当然第一个就是RequestMappingHandlerMapping对象，调用其getHandler方法,根据请求path，找到一个最为匹配的HandlerMethod来处理请求

```kotlin
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
        if (this.handlerMappings != null) {
            for (HandlerMapping hm : this.handlerMappings) {
                if (logger.isTraceEnabled()) {
                    logger.trace(
                            "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
                }
                HandlerExecutionChain handler = hm.getHandler(request);
                if (handler != null) {
                    return handler;
                }
            }
        }
        return null;
    }
```

1. 根据请求路径获取HandlerInterceptor，然后和上面获得的HandlerMethod一起构成HandlerExecutionChain返回给DispatcherServlet

DispatcherServlet得到HandlerExecutionChain也就获得了处理此次请求所需的Handler【即我们熟悉的Controller和对应的Action】，后续将会选择合适HandlerAdapter来执行对应的Handler，获取返回值，再根据返回值类型，进一步觉决定用什么方式展示给用户



# HandlerAdapter

1. HandlerAdapter从字面意思就是处理适配器,也就是handler的适配器模式
2. DispatcherServlet 通过适配器来执行具体的Handler，开发者也是通过适配器来扩展Handler
3. 源码如下



```dart
public interface HandlerAdapter {

    /**
     * Given a handler instance, return whether or not this {@code HandlerAdapter}
     * can support it. Typical HandlerAdapters will base the decision on the handler
     * type. HandlerAdapters will usually only support one handler type each.
     * <p>A typical implementation:
     * <p>{@code
     * return (handler instanceof MyHandler);
     * }
     * @param handler handler object to check
     * @return whether or not this object can use the given handler
     */
    boolean supports(Object handler);

    /**
     * Use the given handler to handle this request.
     * The workflow that is required may vary widely.
     * @param request current HTTP request
     * @param response current HTTP response
     * @param handler handler to use. This object must have previously been passed
     * to the {@code supports} method of this interface, which must have
     * returned {@code true}.
     * @throws Exception in case of errors
     * @return ModelAndView object with the name of the view and the required
     * model data, or {@code null} if the request has been handled directly
     */
    @Nullable
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

    /**
     * Same contract as for HttpServlet's {@code getLastModified} method.
     * Can simply return -1 if there's no support in the handler class.
     * @param request current HTTP request
     * @param handler handler to use
     * @return the lastModified value for the given handler
     * @see javax.servlet.http.HttpServlet#getLastModified
     * @see org.springframework.web.servlet.mvc.LastModified#getLastModified
     */
    long getLastModified(HttpServletRequest request, Object handler);

}
```

1. 针对不同类型handler，需要不同的HandlerAdapter来适配，然后DispatchServlet会根据已经加载的HandlerAdapter集合，选择最适合当前Hander【Handler是根据当前请求path从HandlerMapping中获取的】的HandlerAdapter，然后调用ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)方法，获取ModelAndView对象
2. SimplController这种类型的Handler，就用SimpleControllerHandlerAdapter这种适配器来针对这种Handler来做适配，可以看出是先根据supports方法来判断是否支持某种类型的Handler，SimpleControllerHanderAdapter就仅仅支持实现了Controller接口的Handler
   然后调用Hander方法，其实就是调用Controller的handleRequest方法直接处理请求，返回ModelAndView对象

```java
public class SimpleControllerHandlerAdapter implements HandlerAdapter {

    @Override
    public boolean supports(Object handler) {
        return (handler instanceof Controller);
    }

    @Override
    @Nullable
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {

        return ((Controller) handler).handleRequest(request, response);
    }

    @Override
    public long getLastModified(HttpServletRequest request, Object handler) {
        if (handler instanceof LastModified) {
            return ((LastModified) handler).getLastModified(request);
        }
        return -1L;
    }

}
```

1. DispatchServlet 首先是初始化HandlerAdapt，在Servlet的init方法中加载，其实就是从spring上下文中收集出所有的HandlerAdapter类型的bean，然后排下序就可以了

```kotlin
   private void initHandlerAdapters(ApplicationContext context) {
        this.handlerAdapters = null;

        if (this.detectAllHandlerAdapters) {
            // Find all HandlerAdapters in the ApplicationContext, including ancestor contexts.
            Map<String, HandlerAdapter> matchingBeans =
                    BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerAdapter.class, true, false);
            if (!matchingBeans.isEmpty()) {
                this.handlerAdapters = new ArrayList<>(matchingBeans.values());
                // We keep HandlerAdapters in sorted order.
                AnnotationAwareOrderComparator.sort(this.handlerAdapters);
            }
        }
        else {
            try {
                HandlerAdapter ha = context.getBean(HANDLER_ADAPTER_BEAN_NAME, HandlerAdapter.class);
                this.handlerAdapters = Collections.singletonList(ha);
            }
            catch (NoSuchBeanDefinitionException ex) {
                // Ignore, we'll add a default HandlerAdapter later.
            }
        }
    }
```

1. 初始化完成之后，当一个请求进来之后，首先会根据HandlerMapping获取对应的HandlerExecutionChain,然后根据其中的Handler类型找到适合的HandlerAdapt，最后开始执行Handler，然后返回ModelAndView对象

```kotlin
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
        if (this.handlerAdapters != null) {
            for (HandlerAdapter ha : this.handlerAdapters) {
                if (logger.isTraceEnabled()) {
                    logger.trace("Testing handler adapter [" + ha + "]");
                }
                if (ha.supports(handler)) {
                    return ha;
                }
            }
        }
    }
```

