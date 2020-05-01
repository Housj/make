# 深入理解java SPI机制



# 1 SPI是什么

SPI全称Service Provider Interface，是Java提供的一套用来被第三方实现或者扩展的API，它可以用来启用框架扩展和替换组件。

整体机制图如下：



![img](https:////upload-images.jianshu.io/upload_images/5618238-5d8948367cb9b18e.png?imageMogr2/auto-orient/strip|imageView2/2/w/848/format/webp)

Java SPI 实际上是“**基于接口的编程＋策略模式＋配置文件**”组合实现的动态加载机制。

系统设计的各个抽象，往往有很多不同的实现方案，在面向的对象的设计里，一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制。
 Java SPI就是提供这样的一个机制：为某个接口寻找服务实现的机制。有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。所以SPI的核心思想就是**解耦**。

# 2 使用场景

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

# 3 使用介绍

要使用Java SPI，需要遵循如下约定：

- 1、当服务提供者提供了接口的一种具体实现后，在jar包的META-INF/services目录下创建一个以“接口全限定名”为命名的文件，内容为实现类的全限定名；
- 2、接口实现类所在的jar包放在主程序的classpath中；
- 3、主程序通过java.util.ServiceLoder动态装载实现模块，它通过扫描META-INF/services目录下的配置文件找到实现类的全限定名，把类加载到JVM；
- 4、SPI的实现类必须携带一个不带参数的构造方法；

## 示例代码

**步骤1**、定义一组接口 (假设是org.foo.demo.IShout)，并写出接口的一个或多个实现，(假设是org.foo.demo.animal.Dog、org.foo.demo.animal.Cat)。



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

# 4 原理解析

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
- (3) 把实例化后的类缓存到providers对象中，(LinkedHashMap<String,S>类型）
   然后返回实例对象。

# 5 总结

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

# 参考

[Java核心技术36讲](https://links.jianshu.com/go?to=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Fintro%2F82%3Fcode%3Dw8EZ6RGOQApZJ5tpAzP8dRzeVHxZ4q%2FfOdSbSZzbkhc%3D)
 [The Java™ Tutorials](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2Ftutorial%2Fext%2Fbasics%2Fspi.html)
 [Java Doc](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F8%2Fdocs%2Fapi%2Fjava%2Futil%2FServiceLoader.html)
 [Service Provider Interface: Creating Extensible Java Applications](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.developer.com%2Fjava%2Farticle.php%2F3848881%2FService-Provider-Interface-Creating-Extensible-Java-Applications.htm)
 [Service provider interface](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FService_provider_interface)
 [Java ServiceLoader使用和解析](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Flovesqcc%2Fp%2F5229353.html)
 [Java基础之SPI机制](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fyangguosb%2Farticle%2Fdetails%2F78772730)
 [Java中SPI机制深入及源码解析](https://links.jianshu.com/go?to=https%3A%2F%2Fcxis.me%2F2017%2F04%2F17%2FJava%E4%B8%ADSPI%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E5%8F%8A%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90%2F)
 [SPI机制简介](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.spring4all.com%2Farticle%2F260)



作者：分布式系统架构
链接：https://www.jianshu.com/p/46b42f7f593c
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

# 典型实例：jdbc的设计

jdbc连接源码分析
使用实例
What?
SPI机制（Service Provider Interface)其实源自服务提供者框架（Service Provider Framework，参考【EffectiveJava】page6)，是一种将服务接口与服务实现分离以达到解耦、大大提升了程序可扩展性的机制。引入服务提供者就是引入了spi接口的实现者，通过本地的注册发现获取到具体的实现类，轻松可插拔

典型实例：jdbc的设计
通常各大厂商（如Mysql、Oracle）会根据一个统一的规范(java.sql.Driver)开发各自的驱动实现逻辑。客户端使用jdbc时不需要去改变代码，直接引入不同的spi接口服务即可。
Mysql的则是com.mysql.jdbc.Drive,Oracle则是oracle.jdbc.driver.OracleDriver。



伪代码如下:

	//注:从jdbc4.0之后无需这个操作,spi机制会自动找到相关的驱动实现
	//Class.forName(driver);
	
	//1.getConnection()方法，连接MySQL数据库。有可能注册了多个Driver，这里通过遍历成功连接后返回。
	con = DriverManager.getConnection(mysqlUrl,user,password);
	//2.创建statement类对象，用来执行SQL语句！！
	Statement statement = con.createStatement();
	//3.ResultSet类，用来存放获取的结果集！！
	ResultSet rs = statement.executeQuery(sql);

jdbc连接源码分析

1. java.sql.DriverManager静态块初始执行，其中使用spi机制加载jdbc具体实现

```
 //java.sql.DriverManager.java   
 //当调用DriverManager.getConnection(..)时，static会在getConnection(..)执行之前被触发执行
    /**
     * Load the initial JDBC drivers by checking the System property
     * jdbc.properties and then use the {@code ServiceLoader} mechanism
     */
    static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
```


2.loadInitialDrivers()中完成了引入的数据库驱动的查找以及载入，本示例只引入了oracle厂商的mysql，我们具体看看。

2.loadInitialDrivers()中完成了引入的数据库驱动的查找以及载入，本示例只引入了oracle厂商的mysql，我们具体看看。

     //java.util.serviceLoader.java
    
       private static void loadInitialDrivers() {
            String drivers;
            try {
                drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                    public String run() {
                    //使用系统变量方式加载
                        return System.getProperty("jdbc.drivers");
                    }
                });
            } catch (Exception ex) {
                drivers = null;
            }
            //如果spi 存在将使用spi方式完成提供的Driver的加载
            // If the driver is packaged as a Service Provider, load it.
            // Get all the drivers through the classloader
            // exposed as a java.sql.Driver.class service.
            // ServiceLoader.load() replaces the sun.misc.Providers()
    
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
    
    //查找具体的provider,就是在META-INF/services/***.Driver文件中查找具体的实现。
                    ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                    Iterator<Driver> driversIterator = loadedDrivers.iterator();
     /* Load these drivers, so that they can be instantiated.
                 * It may be the case that the driver class may not be there
                 * i.e. there may be a packaged driver with the service class
                 * as implementation of java.sql.Driver but the actual class
                 * may be missing. In that case a java.util.ServiceConfigurationError
                 * will be thrown at runtime by the VM trying to locate
                 * and load the service.
                 *
                 * Adding a try catch block to catch those runtime errors
                 * if driver not available in classpath but it's
                 * packaged as service and that service is there in classpath.
                 */
                 //查找具体的实现类的全限定名称
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();//加载并初始化实现类
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });
    
        println("DriverManager.initialize: jdbc.drivers = " + drivers);
    
        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
       ....
            }
        }


3.java.util.ServiceLoader 加载spi实现类.

上一步的核心代码如下，我们接着分析：

```
//java.util.serviceLoader.java

ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
Iterator<Driver> driversIterator = loadedDrivers.iterator();
try{
  //查找具体的实现类的全限定名称
     while(driversIterator.hasNext()) {
     //加载并初始化实现
         driversIterator.next();
     }
 } catch(Throwable t) {
 // Do nothing
 }
```

主要是通过ServiceLoader来完成的,我们按照执行顺序来看看ServiceLoader实现：

```
//初始化一个ServiceLoader,load参数分别是需要加载的接口class对象,当前类加载器
    public static <S> ServiceLoader<S> load(Class<S> service) {
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
    public static <S> ServiceLoader<S> load(Class<S> service,
                                            ClassLoader loader)
    {
        return new ServiceLoader<>(service, loader);
    }

```


遍历所有存在的service实现

        public boolean hasNext() {
            if (acc == null) {
                return hasNextService();
            } else {
                PrivilegedAction<Boolean> action = new PrivilegedAction<Boolean>() {
                    public Boolean run() { return hasNextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }


     //写死的一个目录
           private static final String PREFIX = "META-INF/services/";
     private boolean hasNextService() {
            if (nextName != null) {
                return true;
            }
            if (configs == null) {
                try {
                    String fullName = PREFIX + service.getName();
                    //通过相对路径读取classpath中META-INF目录的文件，也就是读取服务提供者的实现类全限定名
                    if (loader == null)
                        configs = ClassLoader.getSystemResources(fullName);
                    else
                        configs = loader.getResources(fullName);
                } catch (IOException x) {
                    fail(service, "Error locating configuration files", x);
                }
            }
            //判断是否读取到实现类全限定名,比如mysql的“com.mysql.jdbc.Driver
                while ((pending == null) || !pending.hasNext()) {
                    if (!configs.hasMoreElements()) {
                        return false;
                    }
                    pending = parse(service, configs.nextElement());
                }
                nextName = pending.next();//nextName保存,后续初始化实现类使用
                return true;//查到了 返回true，接着调用next()
            }



```
     public S next() {
            if (acc == null) {//用来判断serviceLoader对象是否完成初始化
                return nextService();
            } else {
                PrivilegedAction<S> action = new PrivilegedAction<S>() {
                    public S run() { return nextService(); }
                };
                return AccessController.doPrivileged(action, acc);
            }
        }
      private S nextService() {
            if (!hasNextService())
                throw new NoSuchElementException();
            String cn = nextName;//上一步找到的服务实现者全限定名
            nextName = null;
            Class<?> c = null;
            try {
            //加载字节码返回class对象.但并不去初始化（换句话就是说不去执行这个类中的static块与static变量初始化）
            //
                c = Class.forName(cn, false, loader);
            } catch (ClassNotFoundException x) {
                fail(service,
                     "Provider " + cn + " not found");
            }
            if (!service.isAssignableFrom(c)) {
                fail(service,
                     "Provider " + cn  + " not a subtype");
            }
            try {
	            //初始化这个实现类.将会通过static块的方式触发实现类注册到DriverManager(其中组合了一个CopyOnWriteArrayList的registeredDrivers成员变量)中
                S p = service.cast(c.newInstance());
                providers.put(cn, p);//本地缓存 （全限定名，实现类对象）
                return p;
            } catch (Throwable x) {
                fail(service,
                     "Provider " + cn + " could not be instantiated",
                     x);
            }
            throw new Error();          // This cannot happen
        }

```

上一步中，Sp = service.cast(c.newInstance()) 将会导致具体实现者的初始化，比如mysqlJDBC，会触发如下代码：

上一步中，Sp = service.cast(c.newInstance()) 将会导致具体实现者的初始化，比如mysqlJDBC，会触发如下代码：

    //com.mysql.jdbc.Driver.java
    ......
        private final static CopyOnWriteArrayList<DriverInfo> registeredDrivers = new CopyOnWriteArrayList<>();
    ......
    static {
        try {
    	     //并发安全的想一个copyOnWriteList中方
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }

4.最终Driver全部注册并初始化完毕，开始执行DriverManager.getConnection(url, “root”, “root”)方法并返回。

使用实例
四个项目:spiInterface、spiA、spiB、spiDemo

spiInterface中定义了一个com.zs.IOperation接口。

spiA、spiB均是这个接口的实现类，服务提供者。

spiDemo作为客户端,引入spiA或者spiB依赖，面向接口编程，通过spi的方式获取具体实现者并执行接口方法。

```
├─spiA
│  └─src
│      ├─main
│      │  ├─java
│      │  │  └─com
│      │  │      └─zs
│      │  ├─resources
│      │  │  └─META-INF
│      │  │      └─services
│      │  └─webapp
│      │      └─WEB-INF
│      └─test
│          └─java
├─spiB
│  └─src
│      ├─main
│      │  ├─java
│      │  │  └─com
│      │  │      └─zs
│      │  ├─resources
│      │  │  └─META-INF
│      │  │      └─services
│      │  └─webapp
│      │      └─WEB-INF
│      └─test
│          └─java
├─spiDemo
│  └─src
│      ├─main
│      │  ├─java
│      │  │  └─com
│      │  │      └─zs
│      │  ├─resources
│      │  └─webapp
│      │      └─WEB-INF
│      └─test
│          └─java
└─spiInterface
    └─src
        ├─main
        │  ├─java
        │  │  └─com
        │  │      └─zs
        │  ├─resources
        │  └─webapp
        │      └─WEB-INF
        └─test
            └─java
                └─spiInterface
```



spiDemo		

	package com.zs;
	
	import java.sql.Connection;
	import java.sql.DriverManager;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.sql.Statement;
	import java.util.Iterator;
	import java.util.ServiceLoader;
	
	public class Launcher {
	
		public static void main(String[] args) throws Exception {
	
	//		jdbcTest();
			showSpiPlugins();
	}
	private static void jdbcTest() throws SQLException {
		String url = "jdbc:mysql://localhost:3306/test";
		Connection conn = DriverManager.getConnection(url, "root", "root");
		Statement statement = conn.createStatement();
		ResultSet set = statement.executeQuery("select * from test.user");
		while (set.next()) {
			System.out.println(set.getLong("id"));
			System.out.println(set.getString("userName"));
			System.out.println(set.getInt("age"));
		}
	}
	private static void showSpiPlugins() {
		ServiceLoader<IOperation> operations = ServiceLoader.load(IOperation.class);
		Iterator<IOperation> operationIterator = operations.iterator();
		
		while (operationIterator.hasNext()) {
			IOperation operation = operationIterator.next();
			System.out.println(operation.operation(6, 3));
		}
	}
	}
https://github.com/lemon-simple/SPIDEMO