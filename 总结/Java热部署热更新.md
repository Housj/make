## Java类加载器

1. `Bootstrap ClassLoader`:启动类加载器，负责加载java的核心类，它不是java.lang.ClassLoader的子类，而是由JVM自身实现；
2. `Extension ClassLoader`:扩展类加载器，扩展类加载器的加载路径是JDK目录下jre/lib/ext,扩展类的getParent()方法返回null,实际上扩展类加载器的父类加载器是根加载器，只是根加载器并不是Java实现的；
3. `System ClassLoader`:系统(应用)类加载器，它负责在JVM启动时加载来自java命令的-classpath选项、java.class.path系统属性或CLASSPATH环境变量所指定的jar包和类路径。程序可以通过getSystemClassLoader()来获取系统类加载器；
4. ![双亲委派图](https://images2018.cnblogs.com/blog/1256203/201807/1256203-20180714171531925-1737231049.png)

## 双亲委派模型的工作过程

1. 首先会先查找`当前ClassLoader`的加载范围是否加载过此类，有就返回；
2. 如果没有，查询`父ClassLoader`是否已经加载过此类，如果已经加载过,就直接返回Parent加载的类；
3. 如果整个类加载器体系上的ClassLoader都没有加载过，才由当前ClassLoader加载(调用findClass)。

## 双亲委派模型的作用

　　作用：`保证JDK核心类的优先加载`；

## 双亲委派模型的实现

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
    {
    	// 同步上锁
        synchronized (getClassLoadingLock(name)) {
            // 先查看这个类是不是已经加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                	// 递归，双亲委派的实现，先获取父类加载器，不为空则交给父类加载器
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    // 前面提到，bootstrap classloader的类加载器为null，通过find方法来获得
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                }

                if (c == null) {
                    // 如果还是没有获得该类，调用findClass找到类
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // jvm统计
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            // 连接类
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```



## Driver为什么需要破坏双亲委派？

因为在某些情况下父类加载器需要委托子类加载器去加载class文件。受到加载范围的限制，父类加载器无法加载到需要的文件，以Driver接口为例，由于Driver接口定义在jdk当中的，而其实现由各个数据库的服务商来提供，比如mysql的就写了`MySQL Connector`，那么问题就来了，DriverManager（也由jdk提供）要加载各个实现了Driver接口的实现类，然后进行管理，但是DriverManager由启动类加载器加载，只能记载JAVA_HOME的lib下文件，而其实现是由服务商提供的，由系统类加载器加载，这个时候就需要启动类加载器来委托子类来加载Driver实现，从而破坏了双亲委派，这里仅仅是举了破坏双亲委派的其中一个情况。

## Driver破坏双亲委派的实现

我们结合Driver来看一下在spi（Service Provider Inteface）中如何实现破坏双亲委派。

先从DriverManager开始看，平时我们通过DriverManager来获取数据库的Connection：

```java
String url = "jdbc:mysql://localhost:3306/testdb";
Connection conn = java.sql.DriverManager.getConnection(url, "root", "root"); 
```

在调用DriverManager的时候，会先初始化类，调用其中的静态块：

```java
static {
    loadInitialDrivers();
    println("JDBC DriverManager initialized");
}

private static void loadInitialDrivers() {
    ...
        // 加载Driver的实现类
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {

                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                }
                return null;
            }
        });
    ...
}
```

省略了一部分的代码，重点来看一下`ServiceLoader.load(Driver.class)`：

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    // 获取当前线程中的上下文类加载器
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}
```

load方法调用获取了当前线程中的上下文类加载器，那么上下文类加载器放的是什么加载器呢？

```java
public Launcher() {
	...
    try {
        this.loader = Launcher.AppClassLoader.getAppClassLoader(var1);
    } catch (IOException var9) {
        throw new InternalError("Could not create application class loader", var9);
    }
    Thread.currentThread().setContextClassLoader(this.loader);
    ...
}
```

在`sun.misc.Launcher`中，我们找到了答案，在Launcher初始化的时候，会获取AppClassLoader，然后将其设置为上下文类加载器，而这个AppClassLoader，就是之前上文提到的系统类加载器Application ClassLoader，所以**上下文类加载器默认情况下就是系统加载器**。

继续来看下`ServiceLoader.load(service, cl)`：

```java
public static <S> ServiceLoader<S> load(Class<S> service,
                                        ClassLoader loader){
    return new ServiceLoader<>(service, loader);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    // ClassLoader.getSystemClassLoader()返回的也是系统类加载器
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}

public void reload() {
    providers.clear();
    lookupIterator = new LazyIterator(service, loader);
}
```

上面这段就不解释了，比较简单，然后就是看LazyIterator迭代器：

```java
private class LazyIterator implements Iterator<S>{
    // ServiceLoader的iterator()方法最后调用的是这个迭代器里的next
    public S next() {
        if (acc == null) {
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
        String cn = nextName;
        nextName = null;
        Class<?> c = null;
        // 根据名字来加载类
        try {
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
            S p = service.cast(c.newInstance());
            providers.put(cn, p);
            return p;
        } catch (Throwable x) {
            fail(service,
                 "Provider " + cn + " could not be instantiated",
                 x);
        }
        throw new Error();          // This cannot happen
    }
    
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
    
    
    private boolean hasNextService() {
        if (nextName != null) {
            return true;
        }
        if (configs == null) {
            try {
       // 在classpath下查找META-INF/services/java.sql.Driver名字的文件夹
       // private static final String PREFIX = "META-INF/services/";
                String fullName = PREFIX + service.getName();
                if (loader == null)
   configs = ClassLoader.getSystemResources(fullName);
                else
                    configs = loader.getResources(fullName);
            } catch (IOException x) {
                fail(service, "Error locating configuration files", x);
            }
        }
        while ((pending == null) || !pending.hasNext()) {
            if (!configs.hasMoreElements()) {
                return false;
            }
            pending = parse(service, configs.nextElement());
        }
        nextName = pending.next();
        return true;
    }

}
```

好了，这里基本就差不多完成整个流程了，一起走一遍：

![spi加载过程](https://images2018.cnblogs.com/blog/1256203/201807/1256203-20180714171604349-2005119436.png)

## 双亲委派模型破坏史

1.第一次破坏

由于双亲委派模型是在JDK1.2之后才被引入的，而类加载器和抽象类java.lang.ClassLoader则在JDK1.0时代就已经存在，面对已经存在的用户自定义类加载器的实现代码，Java设计者引入双亲委派模型时不得不做出一些妥协。在此之前，用户去继承java.lang.ClassLoader的唯一目的就是为了重写loadClass()方法，因为虚拟机在进行类加载的时候会调用加载器的私有方法loadClassInternal()，而这个方法唯一逻辑就是去调用自己的loadClass()。

2.第二次破坏

双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷所导致的，双亲委派很好地解决了各个类加载器的基础类的同一问题（越基础的类由越上层的加载器进行加载），基础类之所以称为“基础”，是因为它们总是作为被用户代码调用的API，但世事往往没有绝对的完美。

如果基础类又要调用回用户的代码，那该么办？

一个典型的例子就是JNDI服务，JNDI现在已经是Java的标准服务，

它的代码由启动类加载器去加载（在JDK1.3时放进去的rt.jar），但JNDI的目的就是对资源进行集中管理和查找，它需要调用由独立厂商实现并部署在应用程序的ClassPath下的JNDI接口提供者的代码，但启动类加载器不可能“认识”这些代码。

为了解决这个问题，Java设计团队只好引入了一个不太优雅的设计：线程上下文类加载器(Thread Context ClassLoader)。这个类加载器可以通过java.lang.Thread类的setContextClassLoader()方法进行设置，如果创建线程时还未设置，他将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。

有了线程上下文加载器，JNDI服务就可以使用它去加载所需要的SPI代码，也就是父类加载器请求子类加载器去完成类加载的动作，这种行为实际上就是打通了双亲委派模型层次结构来逆向使用类加载器，实际上已经违背了双亲委派模型的一般性原则，但这也是无可奈何的事情。Java中所有涉及SPI的加载动作基本上都采用这种方式，例如JNDI、JDBC、JCE、JAXB和JBI等。

3.第三次破坏

双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求导致的，这里所说的“动态性”指的是当前一些非常“热门”的名词：代码热替换、模块热部署等，简答的说就是机器不用重启，只要部署上就能用。

OSGi实现模块化热部署的关键则是它自定义的类加载器机制的实现。每一个程序模块(Bundle)都有一个自己的类加载器，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。在OSGi幻境下，类加载器不再是双亲委派模型中的树状结构，而是进一步发展为更加复杂的网状结构，当受到类加载请求时，OSGi将按照下面的顺序进行类搜索：

1）将java.＊开头的类委派给父类加载器加载。

2）否则，将委派列表名单内的类委派给父类加载器加载。

3）否则，将Import列表中的类委派给Export这个类的Bundle的类加载器加载。

4）否则，查找当前Bundle的ClassPath，使用自己的类加载器加载。

5）否则，查找类是否在自己的Fragment Bundle中，如果在，则委派给Fragment Bundle的类加载器加载。

6）否则，查找Dynamic Import列表的Bundle，委派给对应Bundle的类加载器加载。

7）否则，类加载器失败。

## 热部署

​	热部署是在不重启 Java 虚拟机的前提下，能自动侦测到 class 文件的变化，更新运行时 class 的行为。Java 类是通过 Java 虚拟机加载的，某个类的 class 文件在被 classloader 加载后，会生成对应的 Class 对象，之后就可以创建该类的实例。默认的虚拟机行为只会在启动时加载类，如果后期有一个类需要更新的话，单纯替换编译的 class 文件，Java 虚拟机是不会更新正在运行的 class。如果要实现热部署，最根本的方式是修改虚拟机的源代码，改变 classloader 的加载行为，使虚拟机能监听 class 文件的更新，重新加载 class 文件，这样的行为破坏性	很大，为后续的 JVM 升级埋下了一个大坑。

​	另一种友好的方法是创建自己的 classloader 来加载需要监听的 class，这样就能控制类加载的时机，从而实现热部署。

## 如何打破双亲委派模型

1. 自定义类加载器，重写loadClass方法；
2. 使用线程上下文类加载器；

## 热部署步骤

1. 销毁自定义classloader(被该加载器加载的class也会自动卸载)；
2. 更新class
3. 使用新的ClassLoader去加载class

JVM中的Class只有满足以下三个条件，才能被GC回收，也就是该Class被卸载（unload）：

- 该类所有的实例都已经被GC，也就是JVM中不存在该Class的任何实例。
- 加载该类的ClassLoader已经被GC。
- 该类的java.lang.Class 对象没有在任何地方被引用，如不能在任何地方通过反射访问该类的方法

## 自定义类加载器

要创建用户自己的类加载器，只需要继承java.lang.ClassLoader类，然后覆盖它的findClass(String name)方法即可，即指明如何获取类的字节码流。

**如果要符合双亲委派规范，则重写findClass方法（用户自定义类加载逻辑）；要破坏的话，重写loadClass方法(双亲委派的具体逻辑实现)**。

Java的Instrument给运行时的动态追踪留下了希望，Attach API则给运行时动态追踪提供了“出入口”，ASM则大大方便了“人类”操作Java字节码的操作。

基于Instrument和Attach API前辈们创造出了诸如JProfiler、Jvisualvm、BTrace这样的工具。以ASM为基础发展出了cglib、动态代理，继而是应用广泛的Spring AOP。

Java是静态语言，运行时不允许改变数据结构。然而，Java 5引入Instrument，Java 6引入Attach API之后，事情开始变得不一样了。虽然存在诸多限制，然而，在前辈们的努力下，仅仅是利用预留的近似于“只读”的这一点点狭小的空间，仍然创造出了各种大放异彩的技术，极大地提高了软件开发人员定位问题的效率。

## Devtools热部署原理

​	**spring-boot-devtools** 是一个为开发者服务的一个模块，其中最重要的功能就是自动应用代码更改到最新的App上面去。原理是在发现代码有更改之后，重新启动应用，但是速度比手动停止后再启动还要更快，更快指的不是节省出来的手工操作的时间。

**其深层原理是使用了两个ClassLoader**

​	**一个Classloader加载那些不会改变的类（第三方Jar包）**
​	**另一个ClassLoader加载会更改的类，称为 restart ClassLoader**

这样在有代码更改的时候，原来的restart ClassLoader 被丢弃，重新创建一个restart ClassLoader，由于需要加载的类相比较少，所以实现了较快的重启时间。



## Arthas热更新

http://hengyunabc.github.io/arthas-online-hotswap/

### jad 反编译代码

首先运行 `jad` 命令反编译 class 文件获取源代码,运行命令如下：。

```sh
jad --source-only com.andyxh.HelloService > /tmp/HelloService.java
```

### 修改反编译之后的代码

拿到源代码之后，使用 VIM 等文本编辑工具编辑源代码，加入需要改动的逻辑。

### 查找 `ClassLoader`

然后使用 `sc` 命令查找加载修改类的 `ClassLoader`，运行命令如下:

```sh
$ sc -d  com.andyxh.HelloService | grep classLoaderHash
 classLoaderHash   4f8e5cde
```

这里运行之后将会得到 `ClassLoader` 哈希值。

### `mc` 内存编译源代码

使用 mc 命令编译上一步修改保存的源代码，生成最终 `class` 文件。

```sh
$ mc -c 4f8e5cde  /tmp/HelloService.java  -d /tmp
Memory compiler output:
/tmp/com/andyxh/HelloService.class
Affect(row-cnt:1) cost in 463 ms.
```

### redefine 热更新代码

运行 redefine 命令：

```sh
$ redefine /tmp/com/andyxh/HelloService.class
```

## Arthas 热更新原理

https://www.cnblogs.com/goodAndyxublog/p/11880314.html

[源代码地址：https://github.com/9526xu/hotswap-example](https://github.com/9526xu/hotswap-example)

Arthas 热更新功能看起来很神奇，实际上离不开 JDK 一些 API，分别为 instrument API 与 attach API。

## Instrumentation

​	Java Instrumentation 是 JDK5 之后提供接口。使用这组接口，我们可以获取到正在运行 JVM 相关信息，使用这些信息我们构建相关监控程序检测 JVM。另外， 最重要我们可以**替换**和**修改**类的，这样就实现了热更新。

​	Instrumentation 存在两种使用方式，一种为 `pre-main` 方式，这种方式需要在虚拟机参数指定 Instrumentation 程序，然后程序启动之前将会完成修改或替换类。使用方式如下:

```java
java -javaagent:jar Instrumentation_jar -jar xxx.jar
```

有没有觉得这种启动方式很熟悉，仔细观察一下 IDEA 运行输出窗口。

![image.png](https://img2018.cnblogs.com/blog/1419561/201911/1419561-20191118093540368-2141920221.png)

另外很多应用监控工具，如：zipkin、pinpoint、skywalking。

这种方式只能在应用启动之前生效，存在一定的局限性。

JDK6 针对这种情况作出了改进，增加 `agent-main` 方式。我们可以在应用启动之后，再运行 `Instrumentation` 程序。启动之后，只有连接上相应的应用，我们才能做出相应改动，这里我们就需要使用 Java 提供 attach API。

## Attach API

​	Attach API 位于 tools.jar 包，可以用来连接目标 JVM。Attach API 非常简单，内部只有两个主要的类，`VirtualMachine` 与 `VirtualMachineDescriptor`。

`VirtualMachine` 代表一个 JVM 实例， 使用它提供 `attach` 方法，我们就可以连接上目标 JVM。

```java
 VirtualMachine vm = VirtualMachine.attach(pid);
```

`VirtualMachineDescriptor` 则是一个描述虚拟机的容器类，通过该实例我们可以获取到 JVM PID(进程 ID),该实例主要通过 `VirtualMachine#list` 方法获取。

```java
        for (VirtualMachineDescriptor descriptor : VirtualMachine.list()){
            System.out.println(descriptor.id());
        }
```

## 实现热更新功能

我们使用 Instrumentation `agent-main` 方式。

### 实现 agent-main

首先需要编写一个类，包含以下两个方法：

```java
public static void agentmain (String agentArgs, Instrumentation inst);          
public static void agentmain (String agentArgs);
```

> 上面的方法只需要实现一个即可。若两个都实现， [1] 优先级大于 [2]，将会被优先执行。

接着读取外部传入 class 文件，调用 `Instrumentation#redefineClasses`，这个方法将会使用新 class 替换当前正在运行的 class，这样我们就完成了类的修改。

```java
public class AgentMain {
    /**
     *
     * @param agentArgs 外部传入的参数，类似于 main 函数 args
     * @param inst
     */
    public static void agentmain(String agentArgs, Instrumentation inst) {
        // 从 agentArgs 获取外部参数
        System.out.println("开始热更新代码");
        // 这里将会传入 class 文件路径
        String path = agentArgs;
        try {
            // 读取 class 文件字节码
            RandomAccessFile f = new RandomAccessFile(path, "r");
            final byte[] bytes = new byte[(int) f.length()];
            f.readFully(bytes);
            // 使用 asm 框架获取类名
            final String clazzName = readClassName(bytes);

            // inst.getAllLoadedClasses 方法将会获取所有已加载的 class
            for (Class clazz : inst.getAllLoadedClasses()) {
                // 匹配需要替换 class
                if (clazz.getName().equals(clazzName)) {
                    ClassDefinition definition = new ClassDefinition(clazz, bytes);
                    // 使用指定的 class 替换当前系统正在使用 class
                    inst.redefineClasses(definition);
                }
            }

        } catch (UnmodifiableClassException | IOException | ClassNotFoundException e) {
            System.out.println("热更新数据失败");
        }


    }

    /**
     * 使用 asm 读取类名
     *
     * @param bytes
     * @return
     */
    private static String readClassName(final byte[] bytes) {
     return new ClassReader(bytes).getClassName().replace("/", ".");
    }
}
```

完成代码之后，我们还需要往 jar 包 manifest 写入以下属性。

```txt
## 指定 agent-main 全名
Agent-Class: com.andyxh.AgentMain
## 设置权限，默认为 false，没有权限替换 class
Can-Redefine-Classes: true
```

我们使用 `maven-assembly-plugin`，将上面的属性写入文件中。

```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.1.0</version>
    <configuration>
        <!--指定最后产生 jar 名字-->
        <finalName>hotswap-jdk</finalName>
        <appendAssemblyId>false</appendAssemblyId>
        <descriptorRefs>
            <!--将工程依赖 jar 一块打包-->
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifestEntries>
                <!--指定 class 名字-->
                <Agent-Class>
                    com.andyxh.AgentMain
                </Agent-Class>
                <Can-Redefine-Classes>
                    true
                </Can-Redefine-Classes>
            </manifestEntries>
            <manifest>
                <!--指定 mian 类名字，下面将会使用到-->
                <mainClass>com.andyxh.JvmAttachMain</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id> <!-- this is used for inheritance merges -->
            <phase>package</phase> <!-- bind to the packaging phase -->
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

到这里我们就完成热更新主要代码，接着使用 Attach API，连接目标虚拟机，触发热更新的代码。

```java
public class JvmAttachMain {
    public static void main(String[] args) throws IOException, AttachNotSupportedException, AgentLoadException, AgentInitializationException {
        // 输入参数，第一个参数为需要 Attach jvm pid 第二参数为 class 路径
        if(args==null||args.length<2){
            System.out.println("请输入必要参数，第一个参数为 pid，第二参数为 class 绝对路径");
            return;
        }
        String pid=args[0];
        String classPath=args[1];
        System.out.println("当前需要热更新 jvm pid 为 "+pid);
        System.out.println("更换 class 绝对路径为 "+classPath);
        // 获取当前 jar 路径
        URL jarUrl=JvmAttachMain.class.getProtectionDomain().getCodeSource().getLocation();
        String jarPath=jarUrl.getPath();

        System.out.println("当前热更新工具 jar 路径为 "+jarPath);
        VirtualMachine vm = VirtualMachine.attach(pid);//7997是待绑定的jvm进程的pid号
        // 运行最终 AgentMain 中方法
        vm.loadAgent(jarPath, classPath);
    }
}
```

在这个启动类，我们最终调用 `VirtualMachine#loadAgent`，JVM 将会使用上面 AgentMain 方法使用传入 class 文件替换正在运行 class。

### 运行

这里我们继续开头使用的例子，不过这里加入一个方法获取 JVM 运行进程 ID。

```java
public class HelloService {

    public static void main(String[] args) throws InterruptedException {
        System.out.println(getPid());
        while (true){
            TimeUnit.SECONDS.sleep(1);
            hello();
        }
    }

    public static void hello(){
        System.out.println("hello world");
    }

    /**
     * 获取当前运行 JVM PID
     * @return
     */
    private static String getPid() {
        // get name representing the running Java virtual machine.
        String name = ManagementFactory.getRuntimeMXBean().getName();
        System.out.println(name);
        // get pid
        return name.split("@")[0];
    }

}
```

首先运行 `HelloService`，获取当前 PID,接着复制 `HelloService` 代码到另一个工程，修改 `hello` 方法输出 `hello agent`，重新编译生成新的 class 文件。

最后在命令行运行生成的 jar 包。

![image.png](https://img2018.cnblogs.com/blog/1419561/201911/1419561-20191118093541782-347034872.png)