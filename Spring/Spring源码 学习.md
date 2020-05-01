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



# Spring WEB启动源码

spring-web 下 **resources/META-INF/services/javax.servlet.ServletContainerInitializer**

spring 

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

## **WebApplicationInitializer**

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

​	BeanDefinition描述了一个bean实例，它具有属性值，构造函数参数值以及具体实现所提供的更多信息。
这只是一个最小的接口：主要目的是允许{@link BeanFactoryPostProcessor}内省和修改属性值*和其他bean元数据















