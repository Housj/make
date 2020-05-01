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

​	BeanDefinition接口描述了一个bean实例，它具有属性值，构造函数参数值以及具体实现所提供的更多信息。
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

工厂挂钩，允许自定义修改应用程序上下文的* bean定义，以适应上下文基础* bean工厂的bean属性值

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
	//在标准初始化之后，修改应用程序上下文的内部bean工厂。所有bean定义都将被加载，但是还没有实例化bean *。这甚至可以覆盖或添加*属性，甚至可以用于初始化bean。
	//@param beanFactory应用程序上下文使用的bean工厂
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}

```



## DefaultListableBeanFactory

Spring的{@link ConfigurableListableBeanFactory} 和{@link BeanDefinitionRegistry}接口的默认实现：一个成熟的bean工厂基于bean定义元数据，可通过后处理器进行扩展。

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

​	独立的应用程序上下文，接受组件类作为输入特别是{@Configuration}注释的类，但也可以是普通的{ @Component}类型和使用{@code javax.inject}批注的符合JSR-330的类。允许使用{register（Class ...）}一注册类，以及使用{scan（String ...）}进行类路径扫描。

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

