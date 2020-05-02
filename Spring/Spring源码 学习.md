# Servlet3.0新规范

## 目的

Servlet3.0规范的新特性主要是为了3个目的：
	1.简化开发
	2.便于布署
	3.支持Web2.0原则
为了简化开发流程，Servlet3.0引入了注解（annotation），这使得web布署描述符web.xml不在是必须的选择

1、要求：
	jdk6.0+
	Tomcat7.0+
2、利用注解替代了web.xml配置文件
	@WebServlet：配置Servlet的
	@WebInitParam:配置Servlet、Filter的初始化参数
	@WebFilter：配置过滤器
	@WebListener：注册监听器
	@MultipartConfig//证明客户端提交的表单数据是由多部分组成的

没有web.xml文件的情况下，使用注解进行代替。
由此可见，开发中对项目起到配置功能的除了xml和properties文件之外，注解也可以进行配置。而且使用起来极为方便，唯一的缺点在于在进行修改映射的时候需要改动源代码。但是，这可以直接忽略。

## **Pluggability可插入性**
​	当使用任何第三方的框架，如Struts，JSF或Spring，我们都需要在web.xml中添加对应的Servlet的入口。这使得web描述符笨重而难以维护。Servlet3.0的新的可插入特性使得web应用程序模块化而易于维护。通过web fragment实现的可插入性减轻了开发人员的负担，不需要再在web.xml中配置很多的Servlet入口。

## **Asynchronous Processing异步处理**

​	另外一个显著的改变就是Servlet3.0支持异步处理，这对AJAX应用程序非常有用。当一个Servlet创建一个线程来创建某些请求的时候，如查询数据库或消息连接，这个线程要等待直到获得所需要的资源才能够执行其他的操作。异步处理通过运行线程执行其他的操作来避免了这种阻塞。
Apart from the features mentioned here, several other enhancements have been made to the existing API. The sections towards the end of the article will explore these features one by one in detail.
除了这些新特性之外， Servlet3.0对已有的API也做了一些改进，在本文的最后我们会做介绍。

## **Annotations in Servlet 注解**

Servlet3.0的一个主要的改变就是支持注解。使用注解来定义Servlet和filter使得我们不用在web.xml中定义相应的入口。

### **@WebServlet**

@WebServlet用来定义web应用程序中的一个Servlet。这个注解可以应用于继承了HttpServlet。这个注解有多个属性，例如name，urlPattern, initParams,我们可以使用者的属性来定义Servlet的行为。urlPattern属性是必须指定的。
例如我们可以象下面的例子这样定义：

```java
1. @WebServlet(name = "GetQuoteServlet",  urlPatterns = {"/getquote"} )
2. public class GetQuoteServlet extends HttpServlet {
3.     @Override
4.     protected void doGet(HttpServletRequest request, HttpServletResponse response)
5.             throws ServletException, IOException {
6.         PrintWriter out = response.getWriter();
7.         try {
8.             String symbol = request.getParameter("symbol");
9.             out.println("<h1>Stock Price is</h1>" + StockQuoteBean.getPrice(symbol);
10.         } finally {
11.             out.close();
12.         }
13.     }
14. }
15.
16. public class StockQuoteBean {
17. private StockQuoteServiceEntity serviceEntity = new StockQuoteServiceEntity();
18.     public double getPrice(String symbol) {
19.         if(symbol !=null )  {
20. return serviceEntity.getPrice(symbol);
21.          } else {
22.             return 0.0;
23.         }
24.     }
25. }
```

在上面的例子中，一个Servlet只对应了一个urlPattern。实际上一个Servlet可以对应多个urlPattern，我们可以这样定义：

```java
1. @WebServlet(name = "GetQuoteServlet",  urlPatterns = {"/getquote",  "/stockquote"} )

2. public class GetQuoteServlet extends HttpServlet {

3.     @Override

4.     protected void doGet(HttpServletRequest request, HttpServletResponse response)

5.             throws ServletException, IOException {

6.         PrintWriter out = response.getWriter();

7.         try {

8.             String symbol = request.getParameter("symbol");

9.             out.println("<h1>Stock Price is</h1>" + StockQuoteBean.getPrice(symbol);

10.         } finally {

11.             out.close();

12.         }

13.     }

14. }
```

### **@WebFilter**

我们可以使用@WebFilter注解来定义filter。这个注解可以被应用在实现了javax.servlet.Filter接口的类上。同样的，urlPattern属性是必须指定的。下面就是一个例子。

```java
1. @WebFilter(filterName = "AuthenticateFilter", urlPatterns = {"/stock.jsp", "/getquote"})

2. public class AuthenticateFilter implements Filter {

3.

4.     public void doFilter(ServletRequest request, ServletResponse response,

5.             FilterChain chain)     throws IOException, ServletException {

6.         String username = ((HttpServletRequest) request).getParameter("uname");

7.         String password = ((HttpServletRequest) request).getParameter("password");

8.           if (username == null || password == null) {

9.                  ((HttpServletResponse) response).sendRedirect("index.jsp");            }

10. if (username.equals("admin") && password.equals("admin")) {

11.                 chain.doFilter(request, response);      }

12. else {

13.                 ((HttpServletResponse) response).sendRedirect("index.jsp");         }

14.          }

15.

16.     public void destroy() {

17.     }

18.     public void init(FilterConfig filterConfig) {

19.     }

20. }
```

### **@WebInitParam**

可以使用@WebInitParam注解来制定Servlet或filter的初始参数。当然我们也可以使用@WebServlet或@WebFileter的initParam属性来指定初始参数。下面是使用@WebInitParam的例子：

```java
1. @WebServlet(name = "GetQuoteServlet", urlPatterns = {"/getquote"})

2. @WebInitParam(name = "default_market", value = "NASDAQ")

3. public class GetQuoteServlet extends HttpServlet {

4.     @Override

5.     protected void doGet(HttpServletRequest request, HttpServletResponse response)

6.             throws ServletException, IOException {

7.         response.setContentType("text/html;charset=UTF-8");

8.         PrintWriter out = response.getWriter();

9.         try {

10.             String market = getInitParameter("default_market");

11.             String symbol = request.getParameter("symbol");

12.             out.println("<h1>Stock Price in " + market + " is</h1>" + StockQuoteBean.getPrice(symbol, market));

13.         } finally {

14.             out.close();

15.         }

16.     }

17. }
```

下面是使用initParam属性的例子：

```java
. @WebServlet(name = "GetQuoteServlet",

2.             urlPatterns = {"/getquote"},

3.             initParams={@WebInitParam(name="default_market",  value="NASDAQ")}

4.            )

5. public class GetQuoteServlet extends HttpServlet {

6.     @Override

7.     protected void doGet(HttpServletRequest request, HttpServletResponse response)

8.             throws ServletException, IOException {

9.         response.setContentType("text/html;charset=UTF-8");

10.         PrintWriter out = response.getWriter();

11.         try {

12.             String market = getInitParameter("default_market");

13.             String symbol = request.getParameter("symbol");

14.             out.println("<h1>Stock Price in " + market + " is</h1>" + StockQuoteBean.getPrice(symbol, market));

15.         } finally {

16.             out.close();

17.         }

18.     }

19. }
```

### **@WebListener**

@WebListener注解被应用在作为listener监听web应用程序事件的类上，所以@WebListener能够被应用在实现了ServletContextListener，ServletContextAttributeListener，ServletRequestListener，ServletRequestAttributeListener，HttpSessionListener和HttpSessionAttributeListener接口的类上。在下面的例子中，该类实现了ServletContextListener接口。

```java
1. @WebListener

2. public class QuoteServletContextListener implements ServletContextListener {

3.    public void contextInitialized(ServletContextEvent sce) {

4.    ServletContext context = sce.getServletContext();

5. context.setInitParameter(“default_market”, “NASDAQ”);

6. }

7. public void contextDestroyed(ServletContextEvent sce) {

8. }

9. }
```

### **@MultipartConfig**

使用@MultipartConfig注解来指定Servlet要求的multipart MIME类型。这种类型的MIME附件将从request对象中读取。

## 其他

**The Metadata and Common Annotations 元数据与通用的注解**
除了以上的Servlet特定的注解之外，Servlet3.0还支持JSR175（Java元数据规范）和JSR250（Java平台通用注解）所规定的注解，包括：
  \*  安全相关的注解，如 @DeclareRoles 和 @RolesAllowed
  \*  使用EJB的注解，如 @EJB 和 @EJBs
  \* 资源注入相关的注解，如 @Resource 和 @Resources
  \* 使用JPA的注解，如 @PersistenceContext, @PersistenceContexts, @PersistenceUnit, 和 @PersistenceUnits
  \* 生命周期的注解，如 @PostConstruct和 @PreDestroy
  \* 提供WebService引用的注解，如 @WebServiceRef and @WebServiceRefs
**注解和web.xml哪个会生效**
注解的引入使得web.xml变成可选的了。但是，我们还是可以使用web.xml。容器会根据web.xml中的metadata-complete元素的值来决定使用web.xml还是使用注解。如果该元素的值是true，那么容器不处理注解，web.xml是所有信息的来源。如果该元素不存在或者其值不为true，容器才会处理注解。
**Web框架的可插入性**
我们前面说过了Servlet3.0的改进之一就是使得我们能够将框架和库插入到web应用程序中。这种可插入性减少了配置，并且提高了web应用程序的模块化。Servlet3.0是通过web模块布署描述片段（简称web片段）来实现插入性的。
一个web片段就是web.xml文件的一部分，被包含在框架特定的Jar包的META-INF目录中。Web片段使得该框架组件逻辑上就是web应用程序的一部分，不需要编辑web布署描述文件。
Web片段中使用的元素和布署文件中使用的元素基本相同，除了根元素不一样。Web片段的根元素是<web-fragment>,而且文件名必须叫做web-fragment.xml。容器只会在放在WEB-INF\lib目录下的Jar包中查找web-fragment.xml文件。如果这些Jar包含有web-fragment.xml文件，容器就会装载需要的类来处理他们。
在web.xml中，我们要求Servlet的name必须唯一。同样的，在web.xml和所有的web片段中，Servlet的name也必须唯一。
下面就是一个web-fragment的例子：
web-fragment.xml

```xml
 <web-fragment>

2. <servlet>

3. <servlet-name>ControllerServlet</servlet-name>

4. <servlet-class>com.app.control.ControllerServlet</servlet-class>

5. </servlet>

6. <listener>

7. <listener-class>com.listener.AppServletContextListener</listener-class>

8. </listener>

9. </web-fragment>
```

框架的Jar包是放在WEB-INF\lib目录下的，但是Servlet3.0提供两种方法指定多个web片段之间的顺序：
  1. 绝对顺序
  2. 相对顺序
我们通过web.xml文件中的<absolute-ordering>元素来指定绝对顺序。这个元素有之元素name，name的值是各个web片段的name元素的值。这样就指定了web片段的顺序。如果多个web片段有相同的名字，容器会忽略后出现的web片段。下面是一个指定绝对顺序的例子：
web.xml

```
1. <web-app>

2. <name>DemoApp</name>

3. <absolute-ordering>

4. <name>WebFragment1</name>

5. <name>WebFragment2</name>

6. </absolute-ordering>

7. ...

8. </web-app>
```



相对顺序通过web-fragment.xml中的<ordering>元素来确定。Web片段的顺序由<ordering>的子元素<before>,<after>和<others>来决定。当前的web片段会放在所有的<before>元素中的片段之前。同样的，会放在所有的<after>元素中的片段之后。<others>用来代替所有的其他片段。注意只有当web.xml中没有<absolute-ordering>时，容器才会使用web片段中定义的相对顺序。
下面是一个帮助理解相对顺序的例子：
web-fragment.xml

```xml
1. <web-fragment>

2. <name>WebFragment1</name>

3. <ordering><after>WebFragment2</after></ordering>

4. ...

5. </web-fragment>
```



web-fragment.xml

```xml
1. <web-fragment>

2. <name>WebFragment2</name>

3. ..

4. </web-fragment>
```

复制代码

web-fragment.xml

```xml
1. <web-fragment>

2. <name>WebFragment3</name>

3. <ordering><before><others/></before></ordering>
....
</web-fragment>
```



这些文件将会按照下面的顺序被处理：

  1. WebFragment3
  2. WebFragment2
  3. WebFragment1

包含WebFragment3的Jar文件被最先处理，包含WebFragment2的文件被第二个处理，包含WebFragment1的文件被最后处理。
如果既没有定义绝对顺序，也没有定义相对顺序，那么容器就认为所有的web片段间没有顺序上的依赖关系。

**Servlet中的异步处理**

​	很多时候Servlet要和其他的资源进行互动，例如访问数据库，调用web service。在和这些资源互动的时候，Servlet不得不等待数据返回，然后才能够继续执行。这使得Servlet调用这些资源的时候阻塞。Servlet3.0通过引入异步处理解决了这个问题。异步处理允许线程调用资源的时候不被阻塞，而是直接返回。AsyncContext负责管理从资源来的回应。AsyncContext决定该回应是应该被原来的线程处理还是应该分发给容器中其他的资源。AsyncContext有一些方法如start，dispatch和complete来执行异步处理。
要想使用Servlet3.0的异步处理，我们需要设置@Webservlet和@WebFilter注解的asyncSupport属性。这个属性是布尔值，缺省值是false。
Servlet3.0的异步处理可以很好的和AJAX配合。在Servlet的init方法中，我们能够访问数据库或从JMS读取消息。在doGet或doPost方法中，我们能够启动异步处理，AsyncContext会通过AsyncEvent和AsyncListener来管理线程和数据库操作/JMS操作自己的关系。

## **已有API的改进**
除了这些新特性之外，Servlet3.0还对以往已经存在的API做了一些改进。

### **HttpServletRequest**

To support the multipart/form-data MIME type, the following methods have been added to the HttpServletRequest interface:
为了支持multipart/form-data MIME类型，在HttpServletRequest接口中添加了项目的方法：
  * Iterable<Part> getParts()
  * Part getPart(String name)

### **Cookies**
为了避免一些跨站点攻击，Servlet3.0支持HttpOnly的cookie。HttpOnly cookie不想客户端暴露script代码。Servlet3.0在Cookie类中添加了如下的方法来支持HttpOnly cookie：
  * void setHttpOnly(boolean isHttpOnly)
  * boolean isHttpOnly()

### **ServletContext**
通过在ServletContext中添加下面的方法，Servlet3.0允许Servlet或filter被编程的加入到context中：

  * addServlet(String servletName, String className)
  * addServlet(String servletName, Servlet servlet)
  * addServlet(String servletName, Class<? extends Servlet> servletClass)
  * addFilter(String filterName, String className)
  * addFilter(String filterName, Filter filter)
  * addFilter(String filterName, Class<? extends Filter>filterClass)
  * setInitParameter (String name, String Value)





# Java SPI 服务动态扩展

## SPI是什么

SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的API，它可以用来启用框架扩展和替换组件。

整体机制图如下：



![img](https:////upload-images.jianshu.io/upload_images/5618238-5d8948367cb9b18e.png?imageMogr2/auto-orient/strip|imageView2/2/w/848/format/webp)

Java SPI 实际上是“**基于接口的编程＋策略模式＋配置文件**”组合实现的动态加载机制。

系统设计的各个抽象，往往有很多不同的实现方案，在面向的对象的设计里，一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。

 Java SPI就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。所以SPI的核心思想就是**解耦**。

## 使用场景

概括地说，适用于：**调用者根据实际使用需要，启用、扩展、或者替换框架的实现策略**

比较常见的例子：

- 数据库驱动加载接口实现类的加载
   JDBC加载不同类型数据库的驱动
- 日志门面接口实现类加载
   SLF4J加载不同提供商的日志实现类
- Spring
   Spring中大量使用了SPI,比如：对servlet3.0规范对ServletContainerInitializer的实现、自动类型转换Type Conversion SPI(Converter SPI、Formatter SPI)等
- Dubbo
   Dubbo中也大量使用SPI的方式实现框架的扩展, 不过它对Java提供的原生SPI做了封装，允许用户扩展实现Filter接口

## 使用介绍

要使用Java SPI，需要遵循如下约定：

- 1、当服务提供者提供了接口的一种具体实现后，在jar包的META-INF/services目录下创建一个以“接口全限定名”为命名的文件，内容为实现类的全限定名；
- 2、接口实现类所在的jar包放在主程序的classpath中；
- 3、主程序通过java.util.ServiceLoder动态装载实现模块，它通过扫描META-INF/services目录下的配置文件找到实现类的全限定名，把类加载到JVM；
- 4、SPI的实现类必须携带一个不带参数的构造方法；

## 示例代码

**步骤1**、定义一组接口 (假设是org.foo.demo.IShout)，并写出接口的一个或多个实现，(假设是org.foo.demo.animal.Dog、org.foo.demo.animal.Cat)

```java
public interface IShout {
    void shout();
}
public class Cat implements IShout {
    @Override
    public void shout() {
        System.out.println("miao miao");
    }
}
public class Dog implements IShout {
    @Override
    public void shout() {
        System.out.println("wang wang");
    }
}
```

**步骤2**、在 src/main/resources/ 下建立 /META-INF/services 目录， 新增一个以接口命名的文件 (org.foo.demo.IShout文件)，内容是要应用的实现类（这里是org.foo.demo.animal.Dog和org.foo.demo.animal.Cat，每行一个类）。

文件位置

```java
- src
    -main
        -resources
            - META-INF
                - services
                    - org.foo.demo.IShout
```

文件内容

```java
org.foo.demo.animal.Dog
org.foo.demo.animal.Cat
```

**步骤3**、使用 ServiceLoader 来加载配置文件中指定的实现。

```java
public class SPIMain {
    public static void main(String[] args) {
        ServiceLoader<IShout> shouts = ServiceLoader.load(IShout.class);
        for (IShout s : shouts) {
            s.shout();
        }
    }
}
```

代码输出：

```java
wang wang
miao miao
```

## 原理解析

首先看ServiceLoader类的签名类的成员变量：

```java
public final class ServiceLoader<S> implements Iterable<S>{
private static final String PREFIX = "META-INF/services/";

    // 代表被加载的类或者接口
    private final Class<S> service;

    // 用于定位，加载和实例化providers的类加载器
    private final ClassLoader loader;

    // 创建ServiceLoader时采用的访问控制上下文
    private final AccessControlContext acc;

    // 缓存providers，按实例化的顺序排列
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // 懒查找迭代器
    private LazyIterator lookupIterator;
  
    ......
}
```

参考具体ServiceLoader具体源码，代码量不多，加上注释一共587行，梳理了一下，实现的流程如下：

- 1 应用程序调用ServiceLoader.load方法
   ServiceLoader.load方法内先创建一个新的ServiceLoader，并实例化该类中的成员变量，包括：
  - loader(ClassLoader类型，类加载器)
  - acc(AccessControlContext类型，访问控制器)
  - providers(LinkedHashMap<String,S>类型，用于缓存加载成功的类)
  - lookupIterator(实现迭代器功能)
- 2 应用程序通过迭代器接口获取对象实例
   ServiceLoader先判断成员变量providers对象中(LinkedHashMap<String,S>类型)是否有缓存实例对象，如果有缓存，直接返回。
   如果没有缓存，执行类的装载，实现如下：
- (1) 读取META-INF/services/下的配置文件，获得所有能被实例化的类的名称，值得注意的是，ServiceLoader**可以跨越jar包获取META-INF下的配置文件**，具体加载配置的实现代码如下：

```dart
        try {
            String fullName = PREFIX + service.getName();
            if (loader == null)
                configs = ClassLoader.getSystemResources(fullName);
            else
                configs = loader.getResources(fullName);
        } catch (IOException x) {
            fail(service, "Error locating configuration files", x);
        }
```

- (2) 通过反射方法Class.forName()加载类对象，并用instance()方法将类实例化。
- (3) 把实例化后的类缓存到providers对象中，(LinkedHashMap<String,S>类型） 然后返回实例对象。

## 总结

**优点**：
 使用Java SPI机制的优势是实现解耦，使得第三方服务模块的装配控制的逻辑与调用者的业务代码分离，而不是耦合在一起。应用程序可以根据实际业务情况启用框架扩展或替换框架组件。

相比使用提供接口jar包，供第三方服务模块实现接口的方式，SPI的方式使得源框架，不必关心接口的实现类的路径，可以不用通过下面的方式获取接口实现类：

- 代码硬编码import 导入实现类
- 指定类全路径反射获取：例如在JDBC4.0之前，JDBC中获取数据库驱动类需要通过**Class.forName("com.mysql.jdbc.Driver")**，类似语句先动态加载数据库相关的驱动，然后再进行获取连接等的操作
- 第三方服务模块把接口实现类实例注册到指定地方，源框架从该处访问实例

通过SPI的方式，第三方服务模块实现接口后，在第三方的项目代码的META-INF/services目录下的配置文件指定实现类的全路径名，源码框架即可找到实现类

**缺点**：

- 虽然ServiceLoader也算是使用的延迟加载，但是基本只能通过遍历全部获取，也就是接口的实现类全部加载并实例化一遍。如果你并不想用某些实现类，它也被加载并实例化了，这就造成了浪费。获取某个实现类的方式不够灵活，只能通过Iterator形式获取，不能根据某个参数来获取对应的实现类。
- 多个并发多线程使用ServiceLoader类的实例是不安全的。

## 参考

[Java核心技术36讲](https://links.jianshu.com/go?to=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Fintro%2F82%3Fcode%3Dw8EZ6RGOQApZJ5tpAzP8dRzeVHxZ4q%2FfOdSbSZzbkhc%3D)
 [The Java™ Tutorials](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2Ftutorial%2Fext%2Fbasics%2Fspi.html)
 [Java Doc](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Fapi%2Fjava%2Futil%2FServiceLoader.html)
 [Service Provider Interface: Creating Extensible Java Applications](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.developer.com%2Fjava%2Farticle.php%2F3848881%2FService-Provider-Interface-Creating-Extensible-Java-Applications.htm)
 [Service provider interface](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FService_provider_interface)
 [Java ServiceLoader使用和解析](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Flovesqcc%2Fp%2F5229353.html)
 [Java基础之SPI机制](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fyangguosb%2Farticle%2Fdetails%2F78772730)
 [Java中SPI机制深入及源码解析](https://links.jianshu.com/go?to=https%3A%2F%2Fcxis.me%2F2017%2F04%2F17%2FJava%E4%B8%ADSPI%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90%2F)
 [SPI机制简介](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.spring4all.com%2Farticle%2F260)



# Spring中bean的生命周期

正确理解Spring bean的生命周期非常重要，bean在Spring容器中从创建到销毁经历了若干阶段，每一阶段都可以针对Spring如何管理bean进行个性化定制。在bean准备就绪之前，bean工厂执行了若干启动步骤。

![img](https://img-blog.csdnimg.cn/201911012343410.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1RoaW5rV29u,size_16,color_FFFFFF,t_70)



1. Spring 对bean进行实例化

2. Spring将值和bean的引用注入到bean对应的属性中

3. 如果bean实现了**BeanNameAware**接口，Spring将bean的ID传递给**setBeanName(String name)**方法；

   ​	让Bean获取自己在BeanFactory配置中的名字（根据情况是id或者name）

   ​	在Spring自身完成Bean配置之后，且在调用任何Bean生命周期回调（初始化或者销毁）方法之前就调用这个方法。换言之，在程序中使用BeanFactory.getBean(String beanName)之前，Bean的名字就已经设定好了。所以，程序中可以尽情的使用BeanName而不用担心它没有被初始化。

4. 如果bean实现了**BeanFactoryAware**接口，Spring将调用**setBeanFactory(BeanFactory factory)**方法，将BeanFactory容器实例传入

   ​	让Bean获取配置他们的BeanFactory的引用,可以在Bean中使用这个工厂的getBean()方法。

5. 如果bean实现了**ApplicationContextAware**接口，Spring将调用**setApplicationContext(ApplicationContext applicationContext)**方法，将bean所在的应用上下文的引用传入进来

   ​	让Bean获取它所在的ApplicationContext——Spring容器上下文，这个类就可以方便获得ApplicationContext中的所有bean。

6. 如果bean实现了**BeanPostProcessor**接口，Spring将调用它们的**postProcessBeforeInitialization(Object var1, String var2)**方法, bean初始化方法调用前被调用

   ​	bean工厂的bean属性处理容器，可以管理我们的bean工厂内所有的beandefinition（未实例化）数据，可以随心所欲的修改属性

   ​	如果我们想在Spring容器中完成bean实例化、配置以及其他初始化方法前后要添加一些自己逻辑处理。我们需要定义一个或多个BeanPostProcessor接口实现类，然后注册到Spring IoC容器中。

7. 如果bean实现了InitializingBean接口，Spring将调用它们的afterPropertiesSet()方法。类似地，如果bean使用initmethod声明了初始化方法，该方法也会被调用

   ​	InitializingBean接口为bean提供了初始化方法的方式，它只包括afterPropertiesSet方法，凡是继承该接口的类，在初始化bean的时候都会执行该方法。

   ​	1、Spring为bean提供了两种初始化bean的方式，实现InitializingBean接口，实现afterPropertiesSet方法，或者在配置文件中通过init-method指定，两种方式可以同时使用。

   ​    2、实现InitializingBean接口是直接调用afterPropertiesSet方法，比通过反射调用init-method指定的方法效率要高一点，但是init-method方式消除了对spring的依赖。

   ​	3、如果调用afterPropertiesSet方法时出错，则不调用init-method指定的方法。

8. 如果bean实现了**BeanPostProcessor**接口，Spring将调用它们的**postProcessAfterInitialization(Object var1, String var2)**方法, bean初始化方法调用后被调用

9. 此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁

10. 如果bean实现了DisposableBean接口，Spring将调用它的destroy()接口方法。同样，如果bean使用destroy-method声明了销毁方法，该方法也会被调用



​	现在你已经了解了如何创建和加载一个Spring容器，这只是一个简单的流程。但是一个空的容器并没有太大的价值，在你把东西放进去之前，它里面什么都没有。为了从Spring的DI(依赖注入)中受益，我们必须将应用对象装配进Spring容器中。



#Spring 解决循环依赖

比如在单例池中的表示bean已经完整的被创建，在单例工厂中的表示bean正在被创建，在early中的表示已经被创建但不完整

在Spring的DefaultSingletonBeanRegistry类中，你会赫然发现类上方挂着这三个Map：

- singletonObjects 它是我们最熟悉的朋友，俗称“单例池”“容器”，缓存创建完成单例Bean的地方。
- singletonFactories  映射创建Bean的原始工厂
- earlySingletonObjects 映射Bean的早期引用，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为“Bean”，只是一个Instance.

循环依赖就是N个类中循环嵌套引用，如果在日常开发中我们用new 对象的方式发生这种循环依赖的话程序会在运行时一直循环调用，直至内存溢出报错。

下面说一下Spring是如果解决循环依赖的。

 ## 第一种：构造器参数循环依赖

Spring容器会将每一个正在创建的Bean 标识符放在一个“当前创建Bean池”中，Bean标识符在创建过程中将一直保持在这个池中。

因此如果在创建Bean过程中发现自己已经在“当前创建Bean池”里时将抛出BeanCurrentlyInCreationException异常表示循环依赖；而对于创建完毕的Bean将从“当前创建Bean池”中清除掉。

首先我们先初始化三个Bean。

```
public class StudentA {

  private StudentB studentB ;

  public void setStudentB(StudentB studentB) {

   this.studentB = studentB;

  }

  public StudentA() {

  }

  public StudentA(StudentB studentB) {

    this.studentB = studentB;

  }

}

public class StudentB {

  private StudentC studentC ;

  public void setStudentC(StudentC studentC) {

    this.studentC = studentC;

  }

  public StudentB() {

  }

  public StudentB(StudentC studentC) {

​    this.studentC = studentC;

  }

}

public class StudentC {

  private StudentA studentA ;

  public void setStudentA(StudentA studentA) {

​    this.studentA = studentA;

  }

  public StudentC() {

  }

  public StudentC(StudentA studentA) {

​    this.studentA = studentA;

  }

}
```



OK，上面是很基本的3个类，StudentA有参构造是StudentB。StudentB的有参构造是StudentC，StudentC的有参构造是StudentA ，这样就产生了一个循环依赖的情况。

我们都把这三个Bean交给 [Spring](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247492151&idx=2&sn=0b8fbb5f150d4ee427c0ab7969d4abd1&chksm=eb506701dc27ee17cf3d49899ce7c7643c0866c83264b1d449e4611836adcd7bcd1669a38252&scene=21#wechat_redirect) 管理，并用有参构造实例化。

```
<bean id="a" class="com.zfx.student.StudentA">

 <constructor-arg index="0" ref="b"></constructor-arg>

</bean>

<bean id="b" class="com.zfx.student.StudentB">

 <constructor-arg index="0" ref="c"></constructor-arg>

</bean>

<bean id="c" class="com.zfx.student.StudentC">

 <constructor-arg index="0" ref="a"></constructor-arg>

</bean>
```



下面是测试类：

```
public class Test {

  public static void main(String[] args) {

​    ApplicationContext context = new ClassPathXmlApplicationContext("com/zfx/student/applicationContext.xml");

​    *//System.out.println(context.getBean("a", StudentA.class));*

  }

}
```



执行结果报错信息为：

Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException:

 Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?

如果大家理解开头那句话的话，这个报错应该不惊讶，Spring容器先创建单例StudentA，StudentA依赖StudentB，然后将A放在“当前创建Bean池”中。

此时创建StudentB,StudentB依赖StudentC ,然后将B放在“当前创建Bean池”中,此时创建StudentC，StudentC又依赖StudentA。

但是，此时Student已经在池中，所以会报错，因为在池中的Bean都是未初始化完的，所以会依赖错误 ，初始化完的Bean会从池中移除。

## 第二种：setter方式单例，默认方式

如果要说setter方式注入的话，我们最好先看一张Spring中Bean实例化的图

Spring是先将Bean对象实例化之后再设置对象属性的，[Spring 中的 bean 为什么默认单例](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247492186&idx=3&sn=cbb8fda8f97f28a24939134e711b51a9&chksm=eb50676cdc27ee7afead02ae8c3c0fb6611681c119fa0b3f6dac238676aeb2660a9d77dde392&scene=21#wechat_redirect)， 

修改配置文件为set方式注入

```
<bean id="a" class="com.zfx.student.StudentA" scope="singleton">

 <property name="studentB" ref="b"></property>

</bean>

<bean id="b" class="com.zfx.student.StudentB" scope="singleton">

 <property name="studentC" ref="c"></property>

</bean>

<bean id="c" class="com.zfx.student.StudentC" scope="singleton">

 <property name="studentA" ref="a"></property>

</bean>
```



下面是测试类：

```
public class Test {

  public static void main(String[] args) {

   ApplicationContext context = new ClassPathXmlApplicationContext("com/zfx/student/applicationContext.xml");

   System.out.println(context.getBean("a", StudentA.class));

  }

}
```



打印结果为：

com.zfx.student.StudentA@1fbfd6

**为什么用set方式就不报错了呢 ？**

我们结合上面那张图看，Spring先是用构造实例化Bean对象 ，此时 [Spring](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247492151&idx=2&sn=0b8fbb5f150d4ee427c0ab7969d4abd1&chksm=eb506701dc27ee17cf3d49899ce7c7643c0866c83264b1d449e4611836adcd7bcd1669a38252&scene=21#wechat_redirect) 会将这个实例化结束的对象放到一个Map中，并且 [Spring](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247492151&idx=2&sn=0b8fbb5f150d4ee427c0ab7969d4abd1&chksm=eb506701dc27ee17cf3d49899ce7c7643c0866c83264b1d449e4611836adcd7bcd1669a38252&scene=21#wechat_redirect) 提供了获取这个未设置属性的实例化对象引用的方法。 

结合我们的实例来看，当Spring实例化了StudentA、StudentB、StudentC后，紧接着会去设置对象的属性，此时StudentA依赖StudentB，就会去Map中取出存在里面的单例StudentB对象，以此类推，不会出来循环的问题喽

下面是Spring源码中的实现方法。以下的源码在Spring的Bean包中的**DefaultSingletonBeanRegistry**类中

```java
// Cache of singleton objects: bean name --> bean instance（缓存单例实例化对象的Map集合）
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

// Cache of singleton factories: bean name --> ObjectFactory（单例的工厂Bean缓存集合）
private final Map<String, ObjectFactory> singletonFactories = new HashMap<String, ObjectFactory>(16);

// Cache of early singleton objects: bean name --> bean instance（早期的单例对象缓存集合） 
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
// Set of registered singletons, containing the bean names in registration order（单例的实例化对象名称集合）
private final Set<String> registeredSingletons = new LinkedHashSet<String>(64);

*/***
 ** 添加单例实例*
 ** 解决循环引用的问题*
 ** Add the given singleton factory for building the specified singleton*
 ** if necessary.*
 ** To be called for eager registration of singletons, e.g. to be able to*
 ** resolve circular references.*
 ** @param beanName the name of the bean*
 ** @param singletonFactory the factory for the singleton object*
 **/*

protected void addSingletonFactory(String beanName, ObjectFactory singletonFactory) {

 Assert.notNull(singletonFactory, "Singleton factory must not be null");

 synchronized (this.singletonObjects) {

  if (!this.singletonObjects.containsKey(beanName)) {
   this.singletonFactories.put(beanName, singletonFactory);
   this.earlySingletonObjects.remove(beanName);
   this.registeredSingletons.add(beanName);
  }
 }
}


```

## 第三种：setter方式原型，prototype

修改配置文件为：

```java
<bean id="a" class="com.zfx.student.StudentA" scope="prototype">

 <property name="studentB" ref="b"></property>

</bean>

<bean id="b" class="com.zfx.student.StudentB" scope="prototype">

 <property name="studentC" ref="c"></property>

</bean>

<bean id="c" class="com.zfx.student.StudentC" scope="prototype">

 <property name="studentA" ref="a"></property>

</bean>
```

scope="prototype" 意思是 每次请求都会创建一个实例对象。

两者的区别是：有状态的bean都使用Prototype作用域，无状态的一般都使用singleton单例作用域。

测试用例：

```
public class Test {

  public static void main(String[] args) {

   ApplicationContext context = new ClassPathXmlApplicationContext("com/zfx/student/applicationContext.xml");

    *//此时必须要获取Spring管理的实例，因为现在scope="prototype" 只有请求获取的时候才会实例化对象*

    System.out.println(context.getBean("a", StudentA.class));

  }

}
```

打印结果：

Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException:

  Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference?

**为什么原型模式就报错了呢 ？**

对于“prototype”作用域Bean，Spring容器无法完成依赖注入，因为“prototype”作用域的Bean，Spring容器不进行缓存，因此无法提前暴露一个创建中的Bean。

# spring源码关于循环引用

https://www.bilibili.com/read/cv3791985

**spring在默认单例的情况下是支持循环引用的**

## spring默认是支持循环的依赖的

面试官会问你为什么spring当中默认支持循环依赖？或者问spring在哪里体现了默认支持循环依赖。从上面的结果我们可以推导出来spring是默认支持循环依赖的，但这仅仅是个结果，你不可能告诉面试官说你写了个简单的demo从而说明了他默认支持，高级面试中最好是能通过源码设计来说明这个问题。比如spring的循环依赖其实是可以关闭的，spring提供了api来关闭循环依赖的功能。当然你也可以修改spring源码来关闭这个功能。

**AbstractAutowireCapableBeanFactory**类的**allowCircularReferences**属性为true，表示允许自动尝试解析Bean的循环引用。

spring循环依赖这块的源码 其实所谓的循环依赖无非就是属性注入，或者就是大家常常说的自动注入，

故而我们需要去研究spring自动注入的源码

## spring创建一个bean的流程

通过源码结合在idea当中debug截图来说明

**1、main方法，初始化spring容器，在初始化容器之后默认的单例bean已经实例化完成了，也就是说spring的默认单例bean创建流程、和所谓自动注入的功能都在容器初始化的过程中。故而我们需要研究这个容器初始化过程、在哪里体现了自动注入；**

**进入AnnotationConfigApplicationContext类的构造方法**

![img](https://i0.hdslb.com/bfs/article/2e21dda370477cfd084c40e7135c2daa6e6289d2.png@1320w_724h.png)

**2、在构造方法中调用了refresh方法，查看refresh方法**

![img](https://i0.hdslb.com/bfs/article/4903b5f874cc987bb6186bf55399670acd146230.png@1320w_726h.png)

**3、refresh方法当中调用finishBeanFactoryInitialization方法来实例化所有扫描出来的类**

![img](https://i0.hdslb.com/bfs/article/b28bc801281f394a6b4e90bc3a7530145e656280.png@1320w_604h.png)



**4、finishBeanFactoryInitialization方法当中调用preInstantiateSingletons初始化扫描出来的类**

![img](https://i0.hdslb.com/bfs/article/bae75a4e5cc8f56d71c334009746945d96cfb019.png@1320w_814h.png)

**5、preInstantiateSingletons方法经过一系列判断之后会调用getBean方法去实例化扫描出来的类**

![img](https://i0.hdslb.com/bfs/article/a3f673775fc1ed46e32866f0eb9cfa4d639a338f.png@1320w_694h.png)

**6、getBean方法就是个空壳方法，调用了doGetBean方法**

![img](https://i0.hdslb.com/bfs/article/d091b11748a1fb8661c6233318dcfa11ae0f684f.png@1320w_574h.png)

**doGetBean方法内容有点多，这个方法非常重要，不仅仅针对循环依赖，**

**甚至整个spring bean生命周期中这个方法也有着举足轻重的地位，读者可以认真看看笔者的分析。需要说明的是我为了更好的说清楚这个方法，我把代码放到文章里面进行分析；但是删除了一些无用的代码；比如日志的记录这些无关紧要的代码。下面重点说这个doGetBean方法**

### doGetBean 方法

```
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,

      @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

    

    //这个方法非常重要，但是和笔者今天要分析的循环依赖没什么很大的关系

    //读者可以简单的认为就是对beanName做一个校验特殊字符串的功能

    //我会在下次更新博客的时候重点讨论这个方法

    //transformedBeanName(name)这里的name就是bean的名字

   final String beanName = transformedBeanName(name);

   //定义了一个对象，用来存将来返回出来的bean

   Object bean;

   // Eagerly check singleton cache for manu  ally registered singletons.

   /**

    * 注意这是第一次调用getSingleton方法，下面spring还会调用一次

    * 但是两次调用的不是同一个方法；属于方法重载

    * 第一次 getSingleton(beanName) 也是循环依赖最重要的方法

    * 关于getSingleton(beanName)具体代码分析可以参考笔者后面的分析

    * 这里给出这个方法的总结

    * 首先spring会去单例池去根据名字获取这个bean，单例池就是一个map

    * 如果对象被创建了则直接从map中拿出来并且返回

    * 但是问题来了，为什么spring在创建一个bean的时候会去获取一次呢？

    * 因为作为代码的书写者肯定知道这个bean这个时候没有创建？为什么需要get一次呢？

    * 关于这个问题其实原因比较复杂，需要读者对spring源码设计比较精通

    * 笔者不准备来针对这个原因大书特书，稍微解释一下吧

    * 我们可以分析doGetBean这个方法，顾名思义其实是用来获取bean的

    * 为什么创建bean会调用这个doGetBean方法呢？难道是因为spring作者疏忽，获取乱起名字

    * 显然不是，spring的源码以其书写优秀、命名优秀而著名，那么怎么解释这个方法呢？

    * 其实很简单doGetBean这个方法不仅仅在创建bean的时候会被调用，在getBean的时候也会调用

    * 他是创建bean和getBean通用的方法。但是这只是解释了这个方法的名字意义

    * 并么有解释这个方法为什么会在创建bean的时候调用

    * 笔者前面已经说过原因很复杂，和本文有关的就是因为循环引用

    * 由于循环引用需要在创建bean的过程中去获取被引用的那个类

    * 而被引用的这个类如果没有创建，则会调用createBean来创建这个bean

    * 在创建这个被引用的bean的过程中会判断这个bean的对象有没有实例化

    * bean的对象？什么意思呢？

    * 为了方便阅读，请读者一定记住两个概念；什么是bean，什么是对象

    * 笔者以为一个对象和bean是有区别的

    * 对象：只要类被实例化就可以称为对象

    * bean：首先得是一个对象，然后这个对象需要经历一系列的bean生命周期

    * 最后把这个对象put到单例池才能算一个bean

    * 读者千万注意，笔者下文中如果写到bean和写到对象不是随意写的

    * 是和这里的解释有关系的；重点一定要注意；一定；一定；

    * 简而言之就是spring先new一个对象，继而对这个对象进行生命周期回调

    * 接着对这个对象进行属性填充，也是大家说的自动注入

    * 然后在进行AOP判断等等；这一些操作简称----spring生命周期

    * 所以一个bean是一个经历了spring周期的对象，和一个对象有区别

    * 再回到前面说的循环引用，首先spring扫描到一个需要被实例化的类A

    * 于是spring就去创建A；A=new A-a;new A的过程会调用getBean("a"))；

    * 所谓的getBean方法--核心也就是笔者现在写注释的这个getSingleton(beanName)

    * 这个时候get出来肯定为空？为什么是空呢？我写这么多注释就是为了解释这个问题?

    * 可能有的读者会认为getBean就是去容器中获取，所以肯定为空，其实不然，接着往下看

    * 如果getA等于空；spring就会实例化A；也就是上面的new A

    * 但是在实例化A的时候会再次调用一下 

    * getSingleton(String beanName, ObjectFactory<?> singletonFactory)

    * 笔者上面说过现在写的注释给getSingleton(beanName)

    * 也即是第一次调用getSingleton(beanName)

    * 实例化一共会调用两次getSingleton方法；但是是重载

    * 第二次调用getSingleton方法的时候spring会在一个set集合当中记录一下这个类正在被创建

    * 这个一定要记住，在调用完成第一次getSingleton完成之后

    * spring判读这个类没有创建，然后调用第二次getSingleton

    * 在第二次getSingleton里面记录了一下自己已经开始实例化这个类

    * 这是循环依赖做的最牛逼的地方，两次getSingleton的调用

    * 也是回答面试时候关于循环依赖必须要回答道的地方；

    * 需要说明的spring实例化一个对象底层用的是反射；

    * spring实例化一个对象的过程非常复杂，需要推断构造方法等等；

    * 这里笔者先不讨论这个过程，以后有机会更新一下

    * 读者可以理解spring直接通过new关键字来实例化一个对象

    * 但是这个时候对象a仅仅是一个对象，还不是一个完整的bean

    * 接着让这个对象去完成spring的bean的生命周期

    * 过程中spring会判断容器是否允许循环引用，判断循环引用的代码笔者下面会分析

    * 前面说过spring默认是支持循环引用的，笔者后面解析这个判断的源码也是spring默认支持循环引用的证据

    * 如果允许循环依赖，spring会把这个对象(还不是bean)临时存起来，放到一个map当中

    * 注意这个map和单例池是两个map，在spring源码中单例池的map叫做 singletonObjects

    * 而这个存放临时对象的map叫做singletonFactories

    * 当然spring还有一个存放临时对象的map叫做earlySingletonObjects

    * 所以一共是三个map，有些博客或者书籍人也叫做三级缓存

    * 为什么需要三个map呢？先来了解这三个map到底都缓存了什么

    * 第一个map singletonObjects 存放的单例的bean

    * 第二个map singletonFactories 存放的临时对象(没有完整springBean生命周期的对象)

    * 第三个map earlySingletonObjects 存放的临时对象(没有完整springBean生命周期的对象)

    * 笔者为了让大家不懵逼这里吧第二个和第三个map功能写成了一模一样

    * 其实第二个和第三个map会有不一样的地方，但这里不方便展开讲，下文会分析

    * 读者姑且认为这两个map是一样的

    * 第一个map主要为了直接缓存创建好的bean；方便程序员去getBean；很好理解

    * 第二个和第三个主要为了循环引用；为什么为了方便循环引用，接着往下看

    * 把对象a缓存到第二个map之后，会接着完善生命周期；

    * 当然spring bean的生命周期很有很多步骤；本文先不详细讨论；

    * 后面的博客笔者再更新；

    * 当进行到对象a的属性填充的这一周期的时候，发觉a依赖了一个B类

    * 所以spring就会去判断这个B类到底有没有bean在容器当中

    * 这里的判断就是从第一个map即单例池当中去拿一个bean

    * 后面我会通过源码来证明是从第一个map中拿一个bean的

    * 假设没有，那么spring会先去调用createBean创建这个bean

    * 于是又回到和创建A一样的流程，在创建B的时候同样也会去getBean("B")；

    * getBean核心也就是笔者现在写注释的这个getSingleton(beanName)方法

    * 下面我重申一下：因为是重点

    * 这个时候get出来肯定为空？为什么是空呢？我写这么多注释就是为了解释这个问题?

    * 可能有的读者会认为getBean就是去容器中获取；

    * 所以肯定为空，其实不然，接着往下看；

    * 第一次调用完getSingleton完成之后会调用第二次getSingleton

   * 第二次调用getSingleton同样会在set集合当中去记录B正在被创建

   * 请笔者记住这个时候 set集合至少有两个记录了 A和B；

   * 如果为空就 b=new B()；创建一个b对象；

   * 再次说明一下关于实例化一个对象，spring做的很复杂，下次讨论

   * 创建完B的对象之后，接着完善B的生命周期

   * 同样也会判断是否允许循环依赖，如果允许则把对象b存到第二个map当中；

   * 提醒一下笔者这个时候第二个map当中至少有两个对象了，a和b

   * 接着继续生命周期；当进行到b对象的属性填充的时候发觉b需要依赖A

   * 于是就去容器看看A有没有创建，说白了就是从第一个map当中去找a

   * 有人会说不上A在前面创建了a嘛？注意那只是个对象，不是bean;

   * 还不在第一个map当中 对所以b判定A没有创建，于是就是去创建A；

   * 那么又再次回到了原点了，创建A的过程中；首先调用getBean("a")

   * 上文说到getBean("a")的核心就是 getSingleton(beanName)

   * 上文也说了get出来a==null；但是这次却不等于空了

   * 这次能拿出一个a对象；注意是对象不是bean

   * 为什么两次不同？原因在于getSingleton(beanName)的源码

   * getSingleton(beanName)首先从第一个map当中获取bean

   * 这里就是获取a；但是获取不到；然后判断a是不是等于空

   * 如果等于空则在判断a是不是正在创建？什么叫做正在创建？

   * 就是判断a那个set集合当中有没有记录A；

   * 如果这个集合当中包含了A则直接把a对象从map当中get出来并且返回

   * 所以这一次就不等于空了，于是B就可以自动注入这个a对象了

   * 这个时候a还只是对象，a这个对象里面依赖的B还没有注入

   * 当b对象注入完成a之后，把B的周期走完，存到容器当中

   * 存完之后继续返回，返回到a注入b哪里？

   * 因为b的创建时因为a需要注入b；于是去get b

   * 当b创建完成一个bean之后，返回b(b已经是一个bean了)

   * 需要说明的b是一个bean意味着b已经注入完成了a；这点上面已经说明了

   * 由于返回了一个b，故而a也能注入b了；

   * 接着a对象继续完成生命周期，当走完之后a也在容器中了

   * 至此循环依赖搞定

   * 需要说明一下上文提到的正在创建这种说法并没有官方支持

   * 是笔者自己的认为；各位读者可以自行给他取名字

   * 笔者是因为存放那些记录的set集合的名字叫做singletonsCurrentlyInCreation

   * 顾名思义，当前正在创建的单例对象。。。。。

   * 还有上文提到的对象和bean的概念；也没有官方支持

   * 也是笔者为了让读者更好的理解spring源码而提出的个人概念

   * 但是如果你觉得这种方法确实能让你更好的理解spring源码

   * 那么请姑且相信笔者对spring源码的理解，假设10个人相信就会有100个人相信

   * 继而会有更多人相信，就会成为官方说法，哈哈。

   * 以上是循环依赖的整个过程，其中getSingleton(beanName)

   * 这个方法的存在至关重要

   * 最后说明一下getSingleton(beanName)的源码分析，下文会分析

   **/

    Object sharedInstance = getSingleton(beanName);

    

    

  /**

   * 如果sharedInstance不等于空直接返回

   * 当然这里没有直接返回而是调用了getObjectForBeanInstance

   * 关于这方法以后解释，读者可以认为这里可以理解为

   * bean =sharedInstance; 然后方法最下面会返回bean

   * 什么时候不等于空？

   * 再容器初始化完成之后

   * 程序员直接调用getbean的时候不等于空

   * 什么时候等于空？

   * 上文已经解释过了，创建对象的时候调用就会等于空

   */  

   if (sharedInstance != null && args == null) {

      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);

   }



   else {

      /**

        * 判断这个类是不是在创建过程中

        * 上文说了，一个类是否在创建的过程中是第二次调用getSingleton中决定的

        * 这里还没有执行到，如果就在创建过程中则出异常

        * 

        **/

      //prototypesCurrentlyInCreation 需要联系 getSingleton方法

      if (isPrototypeCurrentlyInCreation(beanName)) {

         throw new BeanCurrentlyInCreationException(beanName);

      }else{

          /**

           * 需要说明的笔者删了很多和本文无用的代码

           * 意思就是源码中执行到这个if的时候有很多其他代码

           * 但是都是一些判断，很本文需要讨论的问题关联不大

           * 这个if就是判断当前需要实例化的类是不是单例的

           * spring默认都是单例的，故而一般都成立的

           * 接下来便是调用第二次 getSingleton

           * 第二次会把当前正在创建的类记录到set集合

           * 然后反射创建这个实例，并且走完生命周期

           * 第二次调用getSingleton的源码分析会在下文

           **/

         if (mbd.isSingleton()) {

            sharedInstance = getSingleton(beanName, () -> {

               try {

                  //完成了目标对象的创建

                  //如果需要代理，还完成了代理

                  return createBean(beanName, mbd, args);

               }

               catch (BeansException ex) {

                  // Explicitly remove instance from singleton cache: It might have been put there

                  // eagerly by the creation process, to allow for circular reference resolution.

                  // Also remove any beans that received a temporary reference to the bean.

                  destroySingleton(beanName);

                  throw ex;

               }

            });

            bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);

         }

   return (T) bean;

}
```

### 两次调用getSingleton方法

第一次调用getSingleton的源码分析 Object sharedInstance =
getSingleton(beanName);




//空壳方法

public Object getSingleton(String beanName) {

    //重点，一定要记住这里传的是一个true，面试会考

   return getSingleton(beanName, true);

}







/**

上面说的true对应这里的第二个参数boolean allowEarlyReference

顾名思义 叫做允许循环引用，而spring在内部调用这个方法的时候传的true

这也能说明spring默认是支持循环引用的，这也是需要讲过面试官的

但是你不能只讲这一点，后面我会总结，这里先记着这个true

这个allowEarlyReference也是支持spring默认支持循环引用的其中一个原因

**/

protected Object getSingleton(String beanName, boolean allowEarlyReference) {

    /**
    
     首先spring会去第一个map当中去获取一个bean；说白了就是从容器中获取
    
     说明我们如果在容器初始化后调用getBean其实就从map中去获取一个bean
    
     假设是初始化A的时候那么这个时候肯定等于空，前文分析过这个map的意义
    
     **/
    
    Object singletonObject = this.singletonObjects.get(beanName);
    
       /**
    
       我们这里的场景是初始化对象A第一次调用这个方法
    
       这段代码非常重要，首先从容器中拿，如果拿不到，再判断这个对象是不是在set集合
    
       这里的set集合前文已经解释过了，就是判断a是不是正在创建
    
       假设现在a不在创建过程，那么直接返回一个空，第一次getSingleton返回
    
       **/

   if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {

      synchronized (this.singletonObjects) {
    
         singletonObject = this.earlySingletonObjects.get(beanName);
    
         if (singletonObject == null && allowEarlyReference) {
    
            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
    
            if (singletonFactory != null) {
    
               singletonObject = singletonFactory.getObject();
    
               this.earlySingletonObjects.put(beanName, singletonObject);
    
               this.singletonFactories.remove(beanName);
    
            }
    
         }
    
      }

   }

   return singletonObject;

}


第二次调用getSingleton sharedInstance = getSingleton(beanName, () ->
代码我做了删减，删了一些本本文无关的代码




public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {

   synchronized (this.singletonObjects) {

​    

       //首先也是从第一个map即容器中获取
    
       //再次证明如果我们在容器初始化后调用getBean其实就是从map当中获取一个bean
    
       //我们这里的场景是初始化对象A第一次调用这个方法
    
       //那么肯定为空
    
      Object singletonObject = this.singletonObjects.get(beanName);
    
      if (singletonObject == null) {
    
          /**注意这行代码，就是A的名字添加到set集合当中
    
          也就是笔者说的标识A正在创建过程当中
    
          这个方法比较简单我就不单独分析了，直接在这里给出
    
          singletonsCurrentlyInCreation.add就是放到set集合当中
    
          protected void beforeSingletonCreation(String beanName) {
    
               if (!this.inCreationCheckExclusions.contains(beanName)
    
                && !this.singletonsCurrentlyInCreation.add(beanName)) {
    
                  throw new BeanCurrentlyInCreationException(beanName);
    
           }
    
        }
    
        **/


​        

         beforeSingletonCreation(beanName);
    
         boolean newSingleton = false;
    
         try {
    
             //这里便是创建一个bean的入口了
    
             //spring会首先实例化一个对象，然后走生命周期
    
             //走生命周期的时候前面说过会判断是否允许循环依赖
    
             //如果允许则会把创建出来的这个对象放到第二个map当中
    
             //然后接着走生命周期当他走到属性填充的时候
    
             //会去get一下B，因为需要填充B，也就是大家认为的自动注入
    
             //这些代码下文分析，如果走完了生命周期
    
            singletonObject = singletonFactory.getObject();
    
            newSingleton = true;
         }
      }
      return singletonObject;
       }
    }

  





### doCreatebean方法

**如果允许则会把创建出来的这个对象放到第二个map当中**

**AbstractAutowireCapableBeanFactory#doCreateBean()方法部分代码**

**由于这个方法内容过去多，我删减了一些无用代码 上面说的 singletonObject =**

**singletonFactory.getObject();**

**会开始创建bean调用AbstractAutowireCapableBeanFactory#doCreateBean() 在创建bean；**

**下面分析这个方法**

```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)

      throws BeanCreationException {



   // Instantiate the bean.

   BeanWrapper instanceWrapper = null;

   if (mbd.isSingleton()) {

       //如果你bean指定需要通过factoryMethod来创建则会在这里被创建

       //如果读者不知道上面factoryMethod那你就忽略这行代码

       //你可以认为你的A是一个普通类，不会再这里创建

      instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);

   }

   if (instanceWrapper == null) {

       //这里就通过反射创建一个对象，注意是对象不是bean

       //这个createBeanInstance的方法过于复杂，本文不做分析

       //以后如果有更新再来分析这个代码

       //读者可以理解这里就是new了一个A对象

      instanceWrapper = createBeanInstance(beanName, mbd, args);

   }

   //得到new出来的A，为什么需要得到呢？因为Anew出来之后存到一个对象的属性当中

   final Object bean = instanceWrapper.getWrappedInstance();

   //重点:面试会考

   //这里就是判断是不是支持循环引用和是否单例以及bean是否在创建过程中

   //判断循环引用的是&& this.allowCircularReferences

   //allowCircularReferences在spring源码当中默认就是true

   // private boolean allowCircularReferences = true; 这是spring源码中的定义

   //并且这个属性上面spring写了一行非常重要的注释

   // Whether to automatically try to resolve circular references between beans

   // 读者自行翻译，这是支持spring默认循环引用最核心的证据

   //读者一定要讲给面试官，关于怎么讲，我后面会总结

   boolean earlySingletonExposure = (mbd.isSingleton() 

       && this.allowCircularReferences &&

         isSingletonCurrentlyInCreation(beanName));

   //如果是单例，并且正在创建，并且是没有关闭循环引用则执行

   //所以spring原形是不支持循环引用的这是证据，但是其实可以解决

   //怎么解决原形的循环依赖，笔者下次更新吧      

   if (earlySingletonExposure) {

       //这里就是这个创建出来的A 对象a 放到第二个map当中

       //注意这里addSingletonFactory就是往map当中put

       //需要说明的是他的value并不是一个a对象

       //而是一段表达式，但是包含了这个对象的

       //所以上文说的第二个map和第三个map的有点不同

       //第三个map是直接放的a对象(下文会讲到第三个map的)，

       //第二个放的是一个表达式包含了a对象

       //为什么需要放一个表达式？下文分析吧

      addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));

   }



   // Initialize the bean instance.

   Object exposedObject = bean;

   try {

       //填充属性，也就是所谓的自动注入

       //这个代码我同一张图来说明

      populateBean(beanName, mbd, instanceWrapper);

      exposedObject = initializeBean(beanName, exposedObject, mbd);

   }

   return exposedObject;

   }
```

populateBean(beanName, mbd, instanceWrapper)截图图说明

![img](https://i0.hdslb.com/bfs/article/dccd959a82857d213778445549264dd2a5b9de45.png@1320w_672h.png)

**当A执行到属性填充的时候会调用AutowiredAnnotationBeanPostProcessor的postProcessProperties方法来完成填充或者叫做自动注入b
下图有很多文字注释，可以放大图上的注释**

![img](https://i0.hdslb.com/bfs/article/b8be4f919d727dae01f1e49af4357a1ef0ece393.png@1320w_678h.png)

**填充B的时候先从容器中获取B，这个时候b没有创建则等于空，然后看B是不是正在创建，这个时候B只是执行了第一次getSingleton故而不在第二个map当中，所以返回空，返回空之后会执行创建B的流程；执行第二遍调用getSingleton的时候会把b标识正在创建的过程中，也就是添加到那个set集合当中；下图做说明**

![img](https://i0.hdslb.com/bfs/article/d03cd88ab826d9a41bb2abad3b1a79eee2380c18.png@1320w_662h.png)

**创建B的流程和创建A差不多，把B放到set集合，标识B正在创建，继而实例化b对象，然后执行生命周期流程，把创建的这个b对象放到第二个map当中，这个时候map当中已经有了a，b两个对象。**

**然后执行b对象的属性填充或者叫自动注入时候发觉需要依赖a，于是重复上面的getbean步骤，调用getSingleton方法；只不过现在a对象已经可以获取了故而把获取出来的a对象、临时对象注入给b对象，然后走完b的生命周期流程后返回b所表示bean，跟着把这个b所表示的bean注入给a对象，最后走完a对象的其他生命周期流程；循环依赖流程全部走完；但是好像没有说到第三个map，第三个map到底充当了什么角色呢？**

**这个知识点非常的重要，关于这个知识不少书籍和博客都说错了，这也是写这篇文章的意义；笔者说每次读spring源码都不一样的收获，这次最大的收获便是这里了；我们先来看一下代码；**

**场景是这样的，spring创建A，记住第一次创建A，过程中发觉需要依赖B，于是创建B，创建B的时候发觉需要依赖A，于是再一次创建–第二次创建A，下面代码就是基于第二次创建的A来分析；第二次创建A的时候依然会调用getSingleton，先获取一下a**



### getSingleton方法

```
protected Object getSingleton(String beanName, boolean allowEarlyReference) {

    //先从第一个map获取a这个bean，也就是单例池获取

   Object singletonObject = this.singletonObjects.get(beanName);

   if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {

      synchronized (this.singletonObjects) {

          //然后从第三个map当中获取a这个对象

         singletonObject = this.earlySingletonObjects.get(beanName);

         //如果第三个map获取不到a对象，再看是否允许了循环引用

         //而这里的allowEarlyReference是true

         //为什么是true，上文说了这个方法是spring自己调用的，他默认传了true

         if (singletonObject == null && allowEarlyReference) {

             //然后从第二个map中获取一个表达式

             //这里要非常注意第二个map当中存的不是一个单纯的对象

             //前面说了第二个map当中存的是一个表达式，你可以理解为存了一个工厂

             //或者理解存了一个方法，方法里面有个参数就是这个对象

             //安装spring的命名来分析应该理解为一个工厂singletonFactory

             //一个能够生成a对象的工厂

             //那么他为什么需要这么一个工厂

             //这里我先大概说一下，是为了通过工厂来改变这个对象

             //至于为什么要改变对象，下文我会分析

             //当然大部分情况下是不需要改变这个对象的

             //读者先可以考虑不需要改变这个对象，

             //那么这个map里面存的工厂就生产就是这个原对象，那么和第三个map功能一样

            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);

            if (singletonFactory != null) {

                //调用表达式，说白了就是调用工厂的方法，然后改变对象

                //我们假设对象不需要改变的情况那么返回了原对象就是a

                //需要改变的情况我们下文再分享

               singletonObject = singletonFactory.getObject();

               //然后把这个对象放到第三个map当中

               this.earlySingletonObjects.put(beanName, singletonObject);

               //把这个对象、或者表达式、或者工厂从第二个map中移除

               this.singletonFactories.remove(beanName);

               //重点:面试会考---为什么要放到第三个？为什么要移除第二个？

               首先我们通过分析做一个总结:

                   spring首先从第一个map中拿a这个bean

                   拿不到，从第三个map当中拿a这个对象

                   拿不到，从第二个map拿a这个对象或者工厂

                   拿到之后放到第三个map，移除第二个map里面的表达式、或者工厂

               如果对象需要改变，当改变完成之后就把他放到第三个里面

               这里的情况是b需要a而进行的步骤，试想一下以后如果还有C需要依赖a

               就不需要重复第二个map的工作了，也就是改变对象的工作了。

               因为改变完成之后的a对象已经在第三个map中了。不知道读者能不能懂笔者的意思

               如果对象不需要改变道理是一样的，也同样在第三个map取就是了；

               至于为什么需要移除第二个map里面的工厂、或者表达式就更好理解了

               他已经对a做完了改变，改变之后的对象已经在第三个map了，为了方便gc啊

               下面对为什么需要改变对象做分析
            }
         }
      }
   }
   return singletonObject;
}
```

**为什么需要改变对象？那个表达式、或者说工厂主要干什么事呢？ 那个工厂、或者表达式主要是调用了下面这个方法**



//关于后置处理器笔者其实要说话很多很多

//现在市面上可见的资料或者书籍对后置处理器的说法笔者一般都不苟同

//我在B站上传过一个4个小时的视频，其中对spring后置处理器做了详细的分析

//也提出了一些自己的理解和主流理解不同的地方，有兴趣同学可以去看看

//其实简单说--这个方法作用主要是为了来处理aop的；

//当然还有其他功能，但是一般的读者最熟悉的就是aop

//这里我说明一下，aop的原理或者流程有很多书籍说到过

//但是笔者今天亲测了，现在市面可见的资料和书籍对aop的说法都不全

//很多资料提到aop是在spring bean的生命周期里面填充属性之后的代理周期完成的

//而这个代理周期甚至是在执行生命周期回调方法之后的一个周期

//那么问题来了？什么叫spring生命周期回调方法周期呢？

// 首先spring bean生命周期和spring生命周期回调方法周期是两个概念

//spring生命周期回调方法是spring bean生命周期的一部分、或者说一个周期

//简单理解就是spring bean的生命的某个过程会去执行spring的生命周期的回调方法

//比如你在某个bean的方法上面写一个加@PostConstruct的方法（一般称bean初始化方法）

//那么这个方法会在spring实例化一个对象之后，填充属性之后执行这个加注解的方法

//我这里叫做spring 生命周期回调方法的生命周期，不是我胡说，有官方文档可以参考的

//在执行完spring生命周期回调方法的生命周期之后才会执行代理生命周期

//在代理这个生命周期当中如果有需要会完成aop的功能

//以上是现在主流的说法，也是一般书籍或者“某些大师”的说法

//但是在循环引用的时候就不一样了，循环引用的情况下这个周期这里就完成了aop的代理

//这个周期严格意义上是在填充属性之前（填充属性也是一个生命周期阶段）

//填充属性的周期甚至在生命周期回调方法之前，更在代理这个周期之前了

//简单来说主流说法代理的生命周期比如在第8个周期或者第八步吧

//但是笔者这里得出的结论，如果一个bean是循环引用的则代理的周期可能在第3步就完成了

//那么为什么需要在第三步就完成呢？

//试想一下A、B两个类，现在对A类做aop处理，也就是需要对A代理

不考虑循环引用 spring 先实例化A，然后走生命周期确实在第8个周期完成的代理

关于这个结论可以去看b站我讲的spring aop源码分析

但是如果是循环依赖就会有问题

比如spring 实例化A 然后发现需要注入B这个时候A还没有走到8步

还没有完成代理，发觉需要注入B，便去创建B，创建B的时候

发觉需要注入A，于是创建A，创建的过程中通过getSingleton

得到了a对象，注意是对象，一个没有完成代理的对象

然后把这个a注入给B？这样做合适吗？注入的a根本没有aop功能；显然不合适

因为b中注入的a需要是一个代理对象

而这个时候a存在第二个map中；不是一个代理对象；

于是我在第二个map中就不能单纯的存一个对象，需要存一个工厂

这个工厂在特殊的时候需要对a对象做改变，比如这里说的代理（需要aop功能的情况）

这也是三个map存在的必要性，不知道读者能不能get到点

```
protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {

   Object exposedObject = bean;

   if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {

      for (BeanPostProcessor bp : getBeanPostProcessors()) {

         if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {

            SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;

            exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);

         }

      }

   }

   return exposedObject;

}
```



### 总结

**总结关于循环引用，如何回答面试:**

首先spring在单例的情况下是默认支持循环引用的

在不做任何配置的情况下，两个bean相互依赖是能初始化成功的；spring源码中在创建bean的时候先创建这个bean的对象，创建对象完成之后通过判断容器对象的**allowCircularReferences**属性决定是否允许缓存这个临时对象，如果能被缓存成功则通过缓存提前暴露这个临时对象来完成循环依赖；

而这个属性默认为true，所以说spring默认支持循环依赖的，但是这个属性spring提供了api让程序员来修改，所以spring也提供了关闭循环引用的功能；再就是spring完成这个临时对象的生命周期的过程中当执行到注入属性或者自动装配的周期时候会通过getSingleton方法去得到需要注入的b对象；而b对象这个时候肯定不存在故而会创建b对象创建b对象成功后继续b对象的生命周期，当执行到b对象的自动注入周期时候会要求注入a对象；

调用getSingleton；从map缓存中得到a的临时对象（因为这个时候a在set集合中singletonsCurrentlyInCreation；这里可以展开讲），而且获取的时候也会判断是否允许循环引用，但是判断的这个值是通过参数传进来的，也就是spring内部调用的，spring源码当中写死了为true，故而如果需要扩展spring、或者对spring二次开发的的时候程序员可以自定义这个值来实现自己的功能；不管放到缓存还是从缓存中取出这个临时都需要判断；而这两次判断spring源码当中都是默认为true；这里也能再次说明spring默认是支持循环引用的；

DefaultSingletonBeanRegistry类的三个map

	* singletonObjects存的是实例化好的bean对象，bean名字—bean实例
	* singletonFactories 存的是用于获取该对象的对象工厂，通过getObject方法获取，如果允许循环依赖，对象将被放入该map中
	* earlySingletonObjects 存的是早期单例对象，并没有实例化，只是new一个对象
	* registeredSingletons 已经注册的单例
	* singletonsCurrentlyInCreation 表示正在创建中的bean，如果不允许循环引用...

```java
 //singleton对象的缓存：bean名到bean实例
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap(64);
    //单例工厂的缓存：从bean名到ObjectFactory，存的是一个表达式，getObject()能够得到该对象
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap(16);
    //早期单例对象的缓存：bean名到bean实例
    private final Map<String, Object> earlySingletonObjects = new HashMap(16);
    //一组已注册的单例，按注册顺序包含bean名称
    private final Set<String> registeredSingletons = new LinkedHashSet(64);
    //当前正在创建的bean的名称
    private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap(16));
```

**然后面试中可以在说说两次调用getSingleton的意义，正在创建的那个set集合有什么用；最后在说说你在看spring循环引用的时候得出的aop实例化过程的新发现；就比较完美了**




# Spring WEB启动源码

spring-web 下 **resources/META-INF/services/javax.servlet.ServletContainerInitializer**

```
org.springframework.web.SpringServletContainerInitializer
```

## **SpringServletContainerInitializer **

​	Servlet 3.0 {@link ServletContainerInitializer}设计为使用Spring的{@link WebApplicationInitializer} * SPI支持基于代码的Servlet容器配置，而不是（或可能与传统的基于* {@code web.xml}的组合）方法。

它的onStartup方法找到所有的**WebApplicationInitializer**的实现类，按照Order排序，然后依次执行

```java
//感兴趣的类WebApplicationInitializer，该类的所有实现会传入onStartup方法 Set<Class<?>> webAppInitializerClasses 属性中
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {
	
    /**
 * 将{ServletContext}委托给应用程序类路径上存在的任何{@link WebApplicationInitializer} 实现。
	 *  <p>因为此类声明了@ {@ code HandlesTypes（WebApplicationInitializer.class）}，所以Servlet 3.0+容器将自动扫描类路径以查找Spring的{@code WebApplicationInitializer}接口的实现
	 *  并提供所有此类类型的集合此方法的{@code webAppInitializerClasses}参数。  <p>如果在类路径上没有找到{@code WebApplicationInitializer}实现，则*该方法实际上是无操作的。
	 *  将发出INFO级别的日志消息，通知用户*实际上已经调用了{@code ServletContainerInitializer}，但是没有找到{@code WebApplicationInitializer}实现。
	 *  * <p>假设检测到一种或多种{@code WebApplicationInitializer}类型，则将实例化它们（如果@ {@ link * org.springframework.core.annotation.Order @，则将对它们进行排序）。存在Order}注释
	 *  或*已实现{@link org.springframework.core.Ordered Ordered}接口）。然后，将在每个实例上调用{@link WebApplicationInitializer＃onStartup（ServletContext）} *方法，
	 *  委派{@code ServletContext}这样，以便每个实例可以注册和配置Servlet，例如Spring的{@code DispatcherServlet}，侦听器等作为Spring的{@code ContextLoaderListener}，
	 *  *或其他任何Servlet API组件（例如过滤器）。 * @param webAppInitializer对在应用程序类路径上发现的{{link WebApplicationInitializer}的所有实现进行分类
	 *  * @param servletContext要初始化的Servlet上下文
*/
    @Override
	public void onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
			throws ServletException {

		List<WebApplicationInitializer> initializers = new LinkedList<>();

		if (webAppInitializerClasses != null) {
			for (Class<?> waiClass : webAppInitializerClasses) {
				// Be defensive: Some servlet containers provide us with invalid classes,
				// no matter what @HandlesTypes says...
				if (!waiClass.isInterface() && !Modifier.isAbstract(waiClass.getModifiers()) &&
						WebApplicationInitializer.class.isAssignableFrom(waiClass)) {
					try {
						initializers.add((WebApplicationInitializer)
								ReflectionUtils.accessibleConstructor(waiClass).newInstance());
					}
					catch (Throwable ex) {
						throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
					}
				}
			}
		}

		if (initializers.isEmpty()) {
			servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
			return;
		}

		servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
		//若我们的WebApplicationInitializer的实现类 实现了Orderd接口或标注@Order注解，会进行排序
		AnnotationAwareOrderComparator.sort(initializers);
		//依次循环调用我们感兴趣的实例的onStartup方法
		for (WebApplicationInitializer initializer : initializers) {
			initializer.onStartup(servletContext);
		}
	}
}
```

## WebApplicationInitializer

​	在Servlet 3.0+环境中实现的接口，以便以编程方式配置ServletContext，这与传统的基于web.xml的方法相反（或可能与*结合）

```java
public interface WebApplicationInitializer {
	/**
	 *初始化此Web应用程序所需的任何Servlet，过滤器，侦听器，上下文参数和属性来配置给定的ServletContext
	 */
	void onStartup(ServletContext servletContext) throws ServletException;
}
```



## BeanDefinition

	BeanDefinition接口描述了一个bean实例，它具有属性值，构造函数参数值以及具体实现所提供的更多信息。
这只是一个最小的接口：主要目的是允许{@link BeanFactoryPostProcessor}内省和修改属性值*和其他bean元数据

```
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

setParentName
getParentName
setBeanClassName
getBeanClassName
setScope
getScope
setLazyInit
isLazyInit
setDependsOn
getDependsOn
setAutowireCandidate
isAutowireCandidate
setPrimary
isPrimary
setFactoryBeanName
getFactoryBeanName
setFactoryMethodName
getFactoryMethodName
getConstructorArgumentValues
hasConstructorArgumentValues
getPropertyValues
hasPropertyValues
setInitMethodName
getInitMethodName
setDestroyMethodName
getDestroyMethodName
setRole
getRole
setDescription
getDescription
getResolvableType
isSingleton
isPrototype
isAbstract
getResourceDescription
getOriginatingBeanDefinition
SCOPE_SINGLETON
SCOPE_PROTOTYPE
ROLE_APPLICATION
ROLE_SUPPORT
ROLE_INFRASTRUCTURE
```



## BeanFactoryPostProcessor

工厂挂钩，允许自定义修改应用程序上下文的 bean定义，以适应上下文基础 bean工厂的bean属性值

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
	//在标准初始化之后，修改应用程序上下文的内部bean工厂。所有bean定义都将被加载，但是还没有实例化bean *。这甚至可以覆盖或添加*属性，甚至可以用于初始化bean。
	//@param beanFactory应用程序上下文使用的bean工厂
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}

```



## DefaultListableBeanFactory

Spring的{ ConfigurableListableBeanFactory} 和{BeanDefinitionRegistry}接口的默认实现：一个成熟的bean工厂基于bean定义元数据，可通过后处理器进行扩展。

典型用法是在访问bean之前先注册所有bean定义（可能从bean定义文件中读取）。因此，按名称查找Bean 在本地Bean定义表中是一种廉价的操作，对预解析的Bean定义元数据对象进行操作

**getBeanDefinition**从 **beanDefinitionMap** 中取出**Bean**

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
      implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {
	/** 从依赖项类型映射到相应的自动装配值 */
	private final Map<Class<?>, Object> resolvableDependencies = new ConcurrentHashMap<>(16);
	/** Bean定义对象的映射，以Bean名称为键 */
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
	/** 单例和非单例Bean名称的映射，按依赖项类型进行键控 */
	private final Map<Class<?>, String[]> allBeanNamesByType = new ConcurrentHashMap<>(64);
	/** 仅依赖单例的bean名称的映射，按依赖项类型进行键控 */
	private final Map<Class<?>, String[]> singletonBeanNamesByType = new ConcurrentHashMap<>(64);
	/** Bean定义名称列表，按注册顺序 */
	private volatile List<String> beanDefinitionNames = new ArrayList<>(256);
    /** 手动注册单例的名称列表，按注册顺序 */
	private volatile Set<String> manualSingletonNames = new LinkedHashSet<>(16);
      ....
    @Override
	public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
		BeanDefinition bd = this.beanDefinitionMap.get(beanName);
		.......
		return bd;
	}
}

	// BeanDefinitionRegistry接口的实现， beanDefinitionNames增加bean
	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {
		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
		}
		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// 无法再修改启动时收集元素（用于稳定的迭代）
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// 仍处于启动注册阶段
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}
		if (existingDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
	}


//实例化所有剩余的（非延迟初始化）单例
public void preInstantiateSingletons() throws BeansException {
   if (logger.isTraceEnabled()) {
      logger.trace("Pre-instantiating singletons in " + this);
   }

   //遍历所有beanDefinition名字 副本以允许使用init方法，这些方法依次注册新的bean定义。
   //虽然这可能不是常规工厂引导程序的一部分，但可以正常运行。
   List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

   // 触发所有非惰性单例bean的初始化...
   // 遍历所有的beanDefinition根据名字、验证beanDefinition
   for (String beanName : beanNames) {
      RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
       // 不是抽象类  是单例的  不是懒加载的
      if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
         //是不是FactoryBean
         if (isFactoryBean(beanName)) {
            //&+beanName
            Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
            if (bean instanceof FactoryBean) {
               final FactoryBean<?> factory = (FactoryBean<?>) bean;
               boolean isEagerInit;
               if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                  isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                              ((SmartFactoryBean<?>) factory)::isEagerInit,
                        getAccessControlContext());
               }
               else {
                  isEagerInit = (factory instanceof SmartFactoryBean &&
                        ((SmartFactoryBean<?>) factory).isEagerInit());
               }
               if (isEagerInit) {
                  getBean(beanName);
               }
            }
         }
         else {
            getBean(beanName); // 调用AbstractBeanFactory#getBean
         }
      }
   }

   // Trigger post-initialization callback for all applicable beans...
   for (String beanName : beanNames) {
      Object singletonInstance = getSingleton(beanName);
      if (singletonInstance instanceof SmartInitializingSingleton) {
         final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
         if (System.getSecurityManager() != null) {
            AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
               smartSingleton.afterSingletonsInstantiated();
               return null;
            }, getAccessControlContext());
         }
         else {
            smartSingleton.afterSingletonsInstantiated();
         }
      }
   }
}
```



## AnnotationConfigApplicationContext

​	独立的应用程序上下文，接受组件类作为输入特别是{Configuration}注释的类，但也可以是普通的{ Component}类型和使用{inject}批注的符合JSR-330的类。允许使用{register（Class ...）}一注册类，以及使用{scan（String ...）}进行类路径扫描。

​	在有多个{@code @Configuration}类的情况下，在更高版本中定义的{@Bean}方法将覆盖先前版本中定义的方法。 可以通过一个额外的{@Configuration}类来故意重写某些bean的定义。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200501203438699.png" alt="image-20200501203438699" style="zoom: 67%;" />

```java
//示例
AnnotationConfigApplicationContext annoApplicationContext = new AnnotationConfigApplicationContext();
annoApplicationContext.register(AppConfig.class);
annoApplicationContext.refresh();
```



```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
    
    private final AnnotatedBeanDefinitionReader reader;

 //创建一个新的AnnotationConfigApplicationContext，从给定的组件类派生bean定义并自动刷新上下文。 @param componentClasses一个或多个组件类-例如	@Configuration类
    public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		this();
		register(componentClasses);
		refresh();//调用的是父类的AbstractApplicationContext#refresh
	}
//注册一个或多个要处理的组件类。注意，必须调用refresh（）才能使上下文完全处理新类
 @Override
public void register(Class<?>... componentClasses) {
   Assert.notEmpty(componentClasses, "At least one component class must be specified");
   //通过该类 AnnotatedBeanDefinitionReader 进行注册类
   this.reader.register(componentClasses);
}
```





## AnnotatedBeanDefinitionReader

​	方便的适配器，用于以编程方式注册Bean类。 这是{ClassPathBeanDefinitionScanner}的替代方法，它采用相同的注释分辨率，但仅适用于显式注册的类。



```java
public class AnnotatedBeanDefinitionReader {
	//注册一个或多个要处理的组件类。 对register的调用是幂等的；多次添加相同的组件类不会产生任何其他影响。
	public void register(Class<?>... componentClasses) {
		for (Class<?> componentClass : componentClasses) {
			registerBean(componentClass);
		}
	}
    //从给定的bean类中注册一个bean，并从类声明的批注中派生其元数据
    public void registerBean(Class<?> beanClass) {
		doRegisterBean(beanClass, null, null, null, null);
	}
    //从给定的bean类中注册一个bean，并从类声明的批注中派生其元数据
    //@param beanClass Bean的类
    //@param name为bean的显式名称
    //@param qualifiers要考虑的特定限定符注释（如果有）除Bean类级别的限定符之外
    //@param supplier用于创建的回调Bean的实例（可能为{@code null}）
//@param customizers一个或多个用于自定义工厂的BeanDefinition的回调，例如设置lazy-init或primary标志
    private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
			@Nullable BeanDefinitionCustomizer[] customizers) {

		AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
		if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
			return;
		}
		abd.setInstanceSupplier(supplier);
		ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
		abd.setScope(scopeMetadata.getScopeName());
		String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

		AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
		if (qualifiers != null) {
			for (Class<? extends Annotation> qualifier : qualifiers) {
				if (Primary.class == qualifier) {
					abd.setPrimary(true);
				}
				else if (Lazy.class == qualifier) {
					abd.setLazyInit(true);
				}
				else {
					abd.addQualifier(new AutowireCandidateQualifier(qualifier));
				}
			}
		}
		if (customizers != null) {
			for (BeanDefinitionCustomizer customizer : customizers) {
				customizer.customize(abd);
			}
		}

		BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
		definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
		BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
	}
```



## AbstractApplicationContext

​	**ApplicationContext**接口的抽象实现。不强制用于配置的存储类型；简单地实现通用上下文功能。使用模板方法设计模式，需要具体的子类来实现抽象方法。

​	与普通的BeanFactory相比，应该假定ApplicationContext来检测在其内部bean工厂中定义的特殊bean：因此，此类自动注册BeanFactoryPostProcessor、BeanPostProcessor和ApplicationListener 在上下文中被定义为bean。

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
      implements ConfigurableApplicationContext {
      
 @Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// 准备此上下文以进行刷新
			prepareRefresh();

			// 告诉子类刷新内部bean工厂
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// 准备 在这种情况下使用的bean工厂
			prepareBeanFactory(beanFactory);

			try {
				//允许在上下文子类中对bean工厂进行后处理
				postProcessBeanFactory(beanFactory);
//实例化并调用所有已注册的BeanFactoryPostProcessor，遵循显式顺序（如果给定）。必须在单例实例化之前调用
	//调用在上下文中注册为bean的工厂处理器，完成了Bean的扫描，Bean装入beanDefinitionMap
				invokeBeanFactoryPostProcessors(beanFactory);

				//注册拦截Bean创建的Bean处理器
				registerBeanPostProcessors(beanFactory);

				//为此上下文初始化消息源
				initMessageSource();

				//为此上下文初始化事件多播器
				initApplicationEventMulticaster();

				//在特定上下文子类中初始化其他特殊bean
				onRefresh();

				//检查侦听器bean并注册它们
				registerListeners();

				//完成该上下文的bean工厂的初始化，实例化所有剩余的（非延迟初始化）单例
				finishBeanFactoryInitialization(beanFactory);

				//最后一步：发布相应的事件
				finishRefresh();
			}

			catch (BeansException ex) {
				//销毁已创建的单例以避免资源泄露
				destroyBeans();

				//重置“活动”标志
				cancelRefresh(ex);

				//将异常传播给调用者
				throw ex;
			}

			finally {
				// 重置Spring核心中的常见自省缓存，因为可能不再需要单例bean的元数据...
				resetCommonCaches();
			}
		}
	}
```



```java
//完成该上下文的bean工厂的初始化，实例化所有剩余的（非延迟初始化）单例
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
   // 为此上下文初始化转换服务
   if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
         beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
      beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
   }

   //如果之前没有任何bean后处理器（例如PropertyPlaceholderConfigurer bean）注册过以下任何一个，则注册一个默认的嵌入式值解析器：此时，主要用于注释属性值的解析。
   if (!beanFactory.hasEmbeddedValueResolver()) {
      beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
   }

   // 尽早初始化LoadTimeWeaverAware Bean，以便尽早注册其转换器
   String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
   for (String weaverAwareName : weaverAwareNames) {
      getBean(weaverAwareName);
   }

   // 停止使用临时ClassLoader进行类型匹配
   beanFactory.setTempClassLoader(null);

   // 允许缓存所有bean定义元数据，而不期望进一步的更改。
   beanFactory.freezeConfiguration();

   // 实例化所有剩余的（非延迟初始化）单例
   beanFactory.preInstantiateSingletons();
}
```





## AbstractBeanFactory

BeanFactory实现的抽象基类，提供了ConfigurableBeanFactory SPI的全部功能。 

 假定有一个可列出的bean工厂：因此也可以用作 bean工厂实现的基类，该工厂从某些后端资源（bean定义访问是一项昂贵的操作）中获取bean定义。

此类通过其基类提供了一个单例缓存**DefaultSingletonBeanRegistry**，单例/原型确定，FactoryBean处理，别名，用于子bean定义的bean定义合并，和bean销毁DisposableBean 接口，自定义销毁方法）。此外，它还可以管理bean工厂通过实现**HierarchicalBeanFactory**接口实现层次结构（在未知bean的情况下委托给父对象）。

要由子类实现的主要模板方法是#getBeanDefinition和#createBean，分别为给定的bean名称检索一个bean定义和为给定的bean定义创建一个bean实例，这些操作的默认实现可以在**DefaultListableBeanFactory**和**AbstractAutowireCapableBeanFactory**

```java
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {
    
    @Override
    public Object getBean(String name) throws BeansException {
        return doGetBean(name, null, null, false);
    }

    /**
	 * 返回一个实例，该实例可以是指定bean的共享或独立的
	 * @param name 要检索的bean的名称
	 * @param requiredType 要检索的bean的必需类型
	 * @param args 使用显式参数创建bean实例时要使用的参数（仅在创建新实例时才应用检索现有实例）
	 * @param typeCheckOnly 是否获取实例以进行类型检查，*并非用于实际用途*
	 * @return bean的实例
	 */
	@SuppressWarnings("unchecked")
	protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
        //返回Bean名称，必要时去除工厂取消引用前缀，并将别名解析为规范名称。
		final String beanName = transformedBeanName(name);
		Object bean;
		//认真检查单例缓存是否有手动注册的单例。
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
            //如果已经注册了单例，返回该bean
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
			//如果我们已经在创建此bean实例中，则失败：我们大概在循环引用中。
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}

			//检查该工厂中是否存在bean定义
			BeanFactory parentBeanFactory = getParentBeanFactory();
			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
				// Not found -> check parent. 找不到->检查父项
				String nameToLookup = originalBeanName(name);
				if (parentBeanFactory instanceof AbstractBeanFactory) {
					return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
							nameToLookup, requiredType, args, typeCheckOnly);
				}
				else if (args != null) {
					// 使用显式参数委派给父级
					return (T) parentBeanFactory.getBean(nameToLookup, args);
				}
				else if (requiredType != null) {
					// 没有参数->委托给标准的getBean方法
					return parentBeanFactory.getBean(nameToLookup, requiredType);
				}
				else {
					return (T) parentBeanFactory.getBean(nameToLookup);
				}
			}

			if (!typeCheckOnly) {
				markBeanAsCreated(beanName);
			}

			try {
				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
				checkMergedBeanDefinition(mbd, beanName, args);

				// 确保当前bean依赖的bean的初始化
				String[] dependsOn = mbd.getDependsOn();
				if (dependsOn != null) {
					for (String dep : dependsOn) {
                        //如果bean有依赖未初始化
						if (isDependent(beanName, dep)) {
							throw new BeanCreationException(mbd.getResourceDescription(), beanName,"Circular depends-on relationship between '" + beanName + "' and '" + dep );
						}
						registerDependentBean(dep, beanName);
						try {
							getBean(dep);
						}
					}
				}

				// 创建bean实例
				if (mbd.isSingleton()) {
					sharedInstance = getSingleton(beanName, () -> {
						try {
							return createBean(beanName, mbd, args);
						}
						catch (BeansException ex) {
							//从单例缓存中显式删除实例：创建过程可能急切地将其放置在该实例中，以允许循环引用解析。还删除所有收到对该bean的临时引用的bean
							destroySingleton(beanName);
							throw ex;
						}
					});
					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
				}

				else if (mbd.isPrototype()) {
					// 这是一个原型->创建一个新实例
					Object prototypeInstance = null;
					try {
						beforePrototypeCreation(beanName);
						prototypeInstance = createBean(beanName, mbd, args);
					}
					finally {
						afterPrototypeCreation(beanName);
					}
					bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
				}

				else {
					String scopeName = mbd.getScope();
					final Scope scope = this.scopes.get(scopeName);
					if (scope == null) {
						throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
					}
					try {
						Object scopedInstance = scope.get(beanName, () -> {
							beforePrototypeCreation(beanName);
							try {
								return createBean(beanName, mbd, args);
							}
							finally {
								afterPrototypeCreation(beanName);
							}
						});
						bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
					}
					catch (IllegalStateException ex) {
					}
				}
			}
			catch (BeansException ex) {
				cleanupAfterBeanCreationFailure(beanName);
				throw ex;
			}
		}

		// 检查所需类型是否与实际bean实例的类型匹配
		if (requiredType != null && !requiredType.isInstance(bean)) {
			try {
		T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
				if (convertedBean == null) {
		throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
				}
				return convertedBean;
			}
			catch (TypeMismatchException ex) {		
	throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
			}
		}
		return (T) bean;
	}
```





## DefaultSingletonBeanRegistry

 共享bean实例的通用注册表，实现{SingletonBeanRegistry}。允许注册应该由bean名称获得的所有注册表调用者共享的单例实例。还支持{DisposableBean}实例（可能对应或可能不对应已注册的单例）的注册，在注册表关闭时销毁。可以注册 bean之间的依赖关系以强制执行适当的关闭命令。

此类主要用作 {BeanFactory}实现的基类，从而排除了单例bean实例的常见管理。

请注意，{ConfigurableBeanFactory}接口扩展了{SingletonBeanRegistry}接口。

请注意，与{AbstractBeanFactory}和{DefaultListableBeanFactory} （从其继承而来）相比，该类既不假定bean定义概念也不为bean实例指定创建过程。也可以用作委托的嵌套助手。

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
    protected static final Object NULL_OBJECT = new Object();
    protected final Log logger = LogFactory.getLog(this.getClass());
  	//singleton对象的缓存：bean名到bean实例
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap(64);
    //单例工厂的缓存：从bean名到ObjectFactory，存的是一个表达式，getObject()能够得到该对象
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap(16);
    //早期单例对象的缓存：bean名到bean实例
    private final Map<String, Object> earlySingletonObjects = new HashMap(16);
    //一组已注册的单例，按注册顺序包含bean名称
    private final Set<String> registeredSingletons = new LinkedHashSet(64);
    //当前正在创建的bean的名称
    private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap(16));
    //当前在创建检查中排除的bean的名称
    private final Set<String> inCreationCheckExclusions = Collections.newSetFromMap(new ConcurrentHashMap(16));
    private Set<Exception> suppressedExceptions;
    private boolean singletonsCurrentlyInDestruction = false;
    private final Map<String, Object> disposableBeans = new LinkedHashMap();
    private final Map<String, Set<String>> containedBeanMap = new ConcurrentHashMap(16);
    private final Map<String, Set<String>> dependentBeanMap = new ConcurrentHashMap(64);
    private final Map<String, Set<String>> dependenciesForBeanMap = new ConcurrentHashMap(64);
```



## AbstractAutowireCapableBeanFactory

实现默认bean创建的抽象bean工厂超类，具有{RootBeanDefinition}类指定的全部功能。实现{AutowireCapableBeanFactory}，除了AbstractBeanFactory的{@link#createBean}方法之外的接口。

提供bean创建（具有构造函数解析）、属性填充，布线（包括自动布线）和初始化。处理运行时bean引用、解析托管集合、调用初始化方法等。支持自动连接构造函数、按名称的属性和按类型的属性。

子类要实现的主要模板方法是{#resolveDependency（DependencyDescriptor，String，Set，TypeConverter）}，用于按类型自动布线。如果工厂能够搜索它的bean定义，匹配的bean通常通过一次搜查。对于其他工厂样式，可以实现简化的匹配算法。

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
  
  /** Strategy for creating bean instances. */
	private InstantiationStrategy instantiationStrategy = new CglibSubclassingInstantiationStrategy();

	/** Resolver strategy for method parameter names. */
	@Nullable
	private ParameterNameDiscoverer parameterNameDiscoverer = new DefaultParameterNameDiscoverer();

	/** 是否自动尝试解析Bean之间的循环引用 */
	private boolean allowCircularReferences = true;

	/**
	 * Whether to resort to injecting a raw bean instance in case of circular reference,
	 * even if the injected bean eventually got wrapped.
	 */
	private boolean allowRawInjectionDespiteWrapping = false;

	/**
	 * Dependency types to ignore on dependency check and autowire, as Set of
	 * Class objects: for example, String. Default is none.
	 */
	private final Set<Class<?>> ignoredDependencyTypes = new HashSet<>();

	/**
	 * Dependency interfaces to ignore on dependency check and autowire, as Set of
	 * Class objects. By default, only the BeanFactory interface is ignored.
	 */
	private final Set<Class<?>> ignoredDependencyInterfaces = new HashSet<>();

	/**
	 * The name of the currently created bean, for implicit dependency registration
	 * on getBean etc invocations triggered from a user-specified Supplier callback.
	 */
	private final NamedThreadLocal<String> currentlyCreatedBean = new NamedThreadLocal<>("Currently created bean");

	/** Cache of unfinished FactoryBean instances: FactoryBean name to BeanWrapper. */
	private final ConcurrentMap<String, BeanWrapper> factoryBeanInstanceCache = new ConcurrentHashMap<>();

	/** Cache of candidate factory methods per factory class. */
	private final ConcurrentMap<Class<?>, Method[]> factoryMethodCandidateCache = new ConcurrentHashMap<>();

	/** Cache of filtered PropertyDescriptors: bean Class to PropertyDescriptor array. */
	private final ConcurrentMap<Class<?>, PropertyDescriptor[]> filteredPropertyDescriptorsCache =
			new ConcurrentHashMap<>();
```