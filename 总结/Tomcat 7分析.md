# 1 Tomcat的顶层结构及启动过程

## 1.1 Tomcat的顶层结构

​	Tomcat最顶层容器叫Server，代表整个服务器，Server中至少有一个Service，用于提供服务。

​	Service主要包含两部分：

​		**Connector**：用于处理连接相关的事情，并提供Socket与request、response的转换

​		**Container**：用于封装和管理Servlet及具体处理request请求。

   一个Tomcat中只有一个Server，一个Server可以包含多个Service，一个Service只有一个Container，但可以有多个Connector。(因为一个服务可以有多个连接，如同时提供http和https连接，也可以提供相同协议不同端口的连接)

<img src="images/image-20200416130011264.png" alt="image-20200416130011264" style="zoom: 67%;" />

<img src="images/image-20200421102442832.png" alt="image-20200421102442832" style="zoom:50%;" />

​	Tomcat中的Server由Catalina管理，Catalina是整个Tomcat的管理类，里面的三个方法load、start、stop分别管理整个服务器的生命周期，load方法用于根据Tomcat的 conf/server.xml 文件创建Server并调用Server的init方法进行初始化，start方法用于启动服务器，stop方法用于停止服务器，start和sto方法内部分别调用Server的start和stop方法，load内部调用了Server的init方法。

​	这三个方法都会按容器的结构逐层调用相应的方法，如Server的start方法中会调用所有Service中的start方法，Service中的start方法会调用所有包含的Connectors和Container的start方法，这样整个服务器就启动了。init和stop方法也一样，这就是Tomcat的生命周期的管理方式。

​	Catalina的await方法直接调用了Server的await方法，作用是进入一个循环，让主线程不会退出。

​	Tomcat的入口Bootstrap中，Bootstrap的作用类似CatalinaAdaptor，具体处理过程还是使用Catalina完成。好处是把启动的入口和具体的管理类分开，方便创建多种启动方式，每种方式只需要写一个相应的CatalinaAdaptor。

## 1.2 Bootstrap的启动过程

正常启动Tomcat调用Bootstrap的main方法，主要分为两部分：

### 1.2.1 首先新建Bootstrap，并执行init方法初始化

 	init方法初始化ClassLoader，并用ClassLoader创建Catalina实例，赋给catalinaDaemon变量，后面对命令的操作都使用catalinaDaemon具体执行。

### 1.2.2 处理main方法传入的命令，如果args参数为空，默认执行start

​	start命令的处理调用了三个方法：setAwait(true)、load(args)和start()。都调用了Catalina的相应方法进行具体执行，是用反射来调用的。

```java
public static void main(String[] args) {
    if (daemon == null) { // 先新建一个Bootstrap
        Bootstrap bootstrap = new Bootstrap();
        try {
            //初始化ClassLoader，并用ClassLoader创建Catalina实例，赋给catalinaDaemon变量
            bootstrap.init();
        } catch (Throwable var3) {
            handleThrowable(var3);
            var3.printStackTrace();
            return;
        }
        daemon = bootstrap;
    } else {
        Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);
    }
    try {
        String command = "start";
        if (args.length > 0) {
            command = args[args.length - 1];
        }

        if (command.equals("startd")) {
            args[args.length - 1] = "start";
            daemon.load(args);
            daemon.start();
        } else if (command.equals("stopd")) {
            args[args.length - 1] = "stop";
            daemon.stop();
        } else if (command.equals("start")) {
            daemon.setAwait(true);
            daemon.load(args);
            daemon.start();
        } else if (command.equals("stop")) {
            daemon.stopServer(args);
        } else if (command.equals("configtest")) {
            daemon.load(args);
            if (null == daemon.getServer()) {
                System.exit(1);
            }
            System.exit(0);
        } else {
            log.warn("Bootstrap: command \"" + command + "\" does not exist.");
        }
    } catch (Throwable var4) {
        Throwable t = var4;
        if (var4 instanceof InvocationTargetException && var4.getCause() != null) {
            t = var4.getCause();
        }
        handleThrowable(t);
        t.printStackTrace();
        System.exit(1);
    }
}
```

​	start方法/setAwait方法/load方法——>根据反射调用catalina的start/setAwait/load方法

​	首先判断catalinaDaemon是否初始化，否则调用init方法进行初始化。然后使用Method进行反射调用Catalina的start方法。((Catalina)catalinaDaemon).start()。

> ​	Method是reflect包的类，代表一个具体的方法，可以使用其invoke方法执行代表的方法，第一个参数是Method方法所在的实体，第二个参数是可变参数，用于Method方法执行时需要的参数。

> ​	catalina.start()——>server.start()——>service.start()——>connector.start()/container.start()

```java
public void start() throws Exception {
    if (this.catalinaDaemon == null) {
        this.init();
    }
    Method method = this.catalinaDaemon.getClass().getMethod("start", (Class[])null);
    method.invoke(this.catalinaDaemon, (Object[])null);
}
 public void setAwait(boolean await) throws Exception {
        Class<?>[] paramTypes = new Class[]{Boolean.TYPE};
        Object[] paramValues = new Object[]{await};
        Method method = this.catalinaDaemon.getClass().getMethod("setAwait",paramTypes);
        method.invoke(this.catalinaDaemon, paramValues);
    }
private void load(String[] arguments) throws Exception {
        String methodName = "load";
        Object[] param;
        Class[] paramTypes;
        if (arguments != null && arguments.length != 0) {
            paramTypes = new Class[]{arguments.getClass()};
            param = new Object[]{arguments};
        } else {
            paramTypes = null;
            param = null;
        }
        Method method = this.catalinaDaemon.getClass().getMethod(methodName,paramTypes);
        if (log.isDebugEnabled()) {
            log.debug("Calling startup class " + method);
        }
        method.invoke(this.catalinaDaemon, param);
    }
```

## 1.3 Catalina的启动过程

Catalina的启动主要是调用setAwait、load和start方法完成的。

​	setAwait 方法用于设置Server启动完成后是否进入等待状态的标志，true则进入；

​	load 方法用于加载配置文件，创建并初始化Server；

​	start 方法用于启动服务器。

### 1.3.1 setAwait(boolean)方法

setAwait方法设置await属性的值，await属性在start方法中 服务器启动后使用它判断是否进入等待状态。

```java
public class Catalina {
    protected static final StringManager sm = StringManager.getManager("org.apache.catalina.startup");
    protected boolean await = false;
    protected String configFile = "conf/server.xml";
  
  	public void setAwait(boolean b) {
        this.await = b;
    }
```

### 1.3.2 load 方法		

​	load方法根据conf/server.xml创建Server对象，并赋值给server属性，解析操作使用Tomcat开源的Digester完成的，然后调用server的init方法

```java
public void load() {
    long t1 = System.nanoTime();
    this.initDirs();
    this.initNaming();
    Digester digester = this.createStartDigester();
    InputSource inputSource = null;
    InputStream inputStream = null;
    File file = null;
    .....
      file = this.configFile();// conf/server.xml
  		inputStream = new FileInputStream(file);
  		inputSource = new InputSource(file.toURI().toURL().toString());

  		inputSource.setByteStream((InputStream)inputStream);
  		digester.push(this);
 		  digester.parse(inputSource);
    ......
    this.getServer().setCatalina(this);
    this.getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
    this.getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());
    this.initStreams();
    try {
        this.getServer().init();
    } catch (LifecycleException var24) {
    }
    long t2 = System.nanoTime();
    if (log.isInfoEnabled()) {
        log.info("Initialization processed in " + (t2 - t1) / 1000000L + " ms");
    }

}
```

### 1.3.3 start方法

Catalina的start方法主要调用了server的start方法启动服务器，并根据await属性判断是否让程序进入等待状态

 1. 先判断Server是否存在，不存在则调用load方法初始化Server

 2. 然后调用Server的start方法启动服务器

 3. 最后注册关闭钩子并根据await属性判断是否进入等待状态，由于之前设置为true，所以需要等待

 4. 进入等待状态会调用await和stop两个方法，await方法直接调用了Server的await方法，会执行一个while循环，这样程序就停在await方法，当await方法的while循环退出时，就会执行stop方法，从而关闭服务器。

    ```
    server的await方法里，while循环根据volatile类型的stopAwait是否为true停止，默认false，一直循环
    ```

```java
public void start() {
        if (this.getServer() == null) {
            this.load();
        }
				.....    
			  try {
            this.getServer().start();
        } catch (LifecycleException var7) {
            try {
                this.getServer().destroy();
            } 
            return;
        }
        if (this.useShutdownHook) {
            if (this.shutdownHook == null) {
                this.shutdownHook = new Catalina.CatalinaShutdownHook();
            }
            Runtime.getRuntime().addShutdownHook(this.shutdownHook);
            LogManager logManager = LogManager.getLogManager();
            if (logManager instanceof ClassLoaderLogManager) {
                ((ClassLoaderLogManager)logManager).setUseShutdownHook(false);
            }
        }
        if (this.await) {
            this.await();
            this.stop();
        }
    }
}
```
## 1.4 Server的启动过程

类继承和实现的关系图

<img src="images/image-20200416143502554.png" alt="image-20200416143502554" style="zoom: 50%;" />	

Server接口的addService(Service)、removeService(Service)添加和删除Service、**Server的init方法和start方法分别循环调用每个Service的init方法和start方法启动所有Service。**

​	**Server默认的实现是StandardServer**，**StandardServer又继承自LifecycleMBeanBase**

​	**LifecycleMBeanBase又继承自LifecycleBase**

​	**LifecycleBase中定义了init和start方法**

​	**LifecycleBase里的init方法和start方法又调用initInternal方法和startInternal方法**，这两个方法都是模版方法，由子类具体实现，

```java
public final synchronized void init() throws LifecycleException {
    if (!this.state.equals(LifecycleState.NEW)) {
        this.invalidTransition("before_init");
    }
    try {
        this.setStateInternal(LifecycleState.INITIALIZING, (Object)null, false);
        this.initInternal();
        this.setStateInternal(LifecycleState.INITIALIZED, (Object)null, false);
    } catch (Throwable var2) {
        ExceptionUtils.handleThrowable(var2);
        this.setStateInternal(LifecycleState.FAILED, (Object)null, false);
    }
}

protected abstract void initInternal() throws LifecycleException;

public final synchronized void start() throws LifecycleException {
  try {
    this.setStateInternal(LifecycleState.STARTING_PREP, (Object)null,false);
    this.startInternal();
    if (this.state.equals(LifecycleState.FAILED)) {
      this.stop();
    } else if (!this.state.equals(LifecycleState.STARTING)) {
      this.invalidTransition("after_start");
    } else {
      this.setStateInternal(LifecycleState.STARTED, (Object)null, false);
    }

protected abstract void startInternal() throws LifecycleException;
```

所以**调用StandardServer的init方法和start方法**会执行**StandardServer自己的initInternal方法和startInternal方法**，里面又分别执行所有的services的init和start方法。

```java
protected void initInternal() throws LifecycleException {
    super.initInternal();
    MBeanFactory factory = new MBeanFactory();
    factory.setContainer(this);
    this.onameMBeanFactory = this.register(factory, "type=MBeanFactory");
    this.globalNamingResources.init();
    for(int i = 0; i < this.services.length; ++i) {
        this.services[i].init();
    }
}
protected void startInternal() throws LifecycleException {
  this.fireLifecycleEvent("configure_start", (Object)null);
  this.setState(LifecycleState.STARTING);
  this.globalNamingResources.start();
  synchronized(this.servicesLock) {
    for(int i = 0; i < this.services.length; ++i) {
      this.services[i].start();
    }

  }
}
```

**StandardServer的await方法**

await方法处理流程，省略了一些处理异常、关闭Socket及对接收到数据处理的代码。

首先判断端口号port，然后根据port的值分为三种处理方法：

 1. port为-2，会直接退出，不进行循环

 2. port为-1，会进入while（！stopAwait）的循环，且内部没有berak跳出的语句，stopAwait标志只有调用了stop方法才会设置为true，所以port为-1时只有在外部调用stop方法才会退出循环

 3. port为其他值，会进入一个while（！stopAwait）循环，同时会在port所在的端口启动一个ServerSocker监听关闭命令，接收到了则使用break跳出循环。

    > ​	这里的端口port和关闭命令shutdown是在conf/server.xml文件中配置Server时设置的。默认设置如下
    >
    > ```xml
    > <!-- server.xml -->
    > <Server port="8005" shutdown="SHUTDOWN">
    > ```
    >
    > ​	这时在8005端口监听"SHUTDOWN"命令，接收到了就会关闭Tomcat，如果不想使用网络命令来关闭服务器可以将端口设置为-1。
    >
    > ​	await方法中从端口接受到数据后还会进行处理，如果接收到的数据中有ASCII码小于32的(ASCII中32以下的为控制符)则从小于32的字符截断并丢弃后面的数据。

<img src="images/image-20200416144840086.png" alt="image-20200416144840086" style="zoom: 50%;" />

<img src="images/image-20200416144913097.png" alt="image-20200416144913097" style="zoom: 50%;" />



## 1.5 Service的启动过程

**Service默认的实现是StandardService**，**StandardService也继承自LifecycleMBeanBase**，

**StandardService的init方法和start方法最终会执行StandardServer自己的initInternal方法和startInternal方法**

initInternal方法主要调用engine、mapperListener、executor和connector的init方法

> ​	mapperListener是Mapper的监听器，监听container容器的变化，
>
> ​	executors是用在connectors中管理线程的线程池
>
> ​	<img src="images/image-20200416160059238.png" alt="image-20200416160059238" style="zoom: 50%;" />

```java
protected void initInternal() throws LifecycleException {
    super.initInternal();
    if (engine != null) {
        engine.init();
    }
    // Initialize any Executors
    for (Executor executor : findExecutors()) {
        if (executor instanceof JmxEnabled) {
            ((JmxEnabled) executor).setDomain(getDomain());
        }
        executor.init();
    }
    // Initialize mapper listener
    mapperListener.init();
    // Initialize our defined Connectors
    synchronized (connectorsLock) {
        for (Connector connector : connectors) {
            connector.init();
        }
    }
}
```

startInternal主要调用engine、mapperListener、executor和connector的start方法

```java
protected void startInternal() throws LifecycleException {
    setState(LifecycleState.STARTING);
    // Start our defined Container first
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }
    synchronized (executors) {
        for (Executor executor: executors) {
            executor.start();
        }
    }
    mapperListener.start();
    // Start our defined Connectors second
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            // If it has already failed, don't try and start it
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}
```

现在整个Tomcat服务器就启动了，整个启动流程

![image-20200416160211274](images/image-20200416160211274.png)

# 2 Tocmat的生命周期管理

## 2.1 Lifecycle 接口

![image-20200421111746824](images/image-20200421111746824.png)	

Tomcat通过Lifecycle接口统一管理生命周期，所有有生命周期的组件都要实现Lifecycle接口。它一共做了4件事

 1. 定义13个String类型常量，用于LifecycleEvent事件的type属性中，为了区分组件发出的LifecycleEvent事件时的状态(初始化前、启动前、启动中等)。

 2. > ```java
    > public interface Lifecycle {
    >     String BEFORE_INIT_EVENT = "before_init";
    >     String AFTER_INIT_EVENT = "after_init";
    >     String START_EVENT = "start";
    >     String BEFORE_START_EVENT = "before_start";
    >     String AFTER_START_EVENT = "after_start";
    >     String STOP_EVENT = "stop";
    >     String BEFORE_STOP_EVENT = "before_stop";
    >     String AFTER_STOP_EVENT = "after_stop";
    >     String AFTER_DESTROY_EVENT = "after_destroy";
    >     String BEFORE_DESTROY_EVENT = "before_destroy";
    >     String PERIODIC_EVENT = "periodic";
    >     String CONFIGURE_START_EVENT = "configure_start";
    >     String CONFIGURE_STOP_EVENT = "configure_stop";
    >   
    >     void addLifecycleListener(LifecycleListener var1);
    >     LifecycleListener[] findLifecycleListeners();
    >     void removeLifecycleListener(LifecycleListener var1);
    >   
    >     void init() throws LifecycleException;
    >     void start() throws LifecycleException;
    >     void stop() throws LifecycleException;
    >     void destroy() throws LifecycleException;
    > 
    >     LifecycleState getState();
    >     String getStateName();
    >     public interface SingleUse { }
    > ```

2. 定义三个管理监听器的方法addLifecycleListener、findLifecycleEvent和removeLifecycleListener，分别用来添加、查找和删除LifecycleListener类型的监听器

3. 定义4个生命周期的方法：init、start、stop和destroy，用于执行生命周期的各个阶段的操作
4. 定义了获取当前 状态的两个方法getState和getStateName，用来获取当前的状态，getState获取的事枚举类型，里面列举了生命周期的各个节点，getStateName方法返回String类型的状态的名字，用于JMX中。

## 2.2 LifecycleBase

​	Lifecycle默认实现是LifecycleBase，实现了生命周期的组件都直接或间接地继承自LifecycleBase。

* 监听管理使用了LifecycleSupport类来完成，定义了一个LifecycleListener数组类型的属性来保存所有的监听器，然后定义了添加、删除、查找和执行监听器的方法
* 生命周期方法中设置了相应的状态并调用了相应的模版方法，init、start、stop和destroy对应的模版方法分别是initInternal、startInternal、stopInternal和destroyInternal方法，这四个方法由子类具体实现
* 组件当前的状态在声明周期的四个方法中已经设置好了，直接返回

### 三个管理监听器的方法

```java
  private LifecycleSupport lifecycle = new LifecycleSupport(this);
    @Override
    public void addLifecycleListener(LifecycleListener listener) {
        lifecycle.addLifecycleListener(listener);
    }
    @Override
    public LifecycleListener[] findLifecycleListeners() {
        return lifecycle.findLifecycleListeners();
    }
    @Override
    public void removeLifecycleListener(LifecycleListener listener) {
        lifecycle.removeLifecycleListener(listener);
    }
```

​	addLifecycleListener、findLifecycleListeners和removeLifecycleListener分别调用了LifecycleSupport中同名的方法，LifecycleSupport监听器是通过一个数组属性listeners保存的

​	这三个方法就是对listeners（数组类型）属性进行操作

* 添加时新建比当前数组大一的数组，将原来的数据按顺序保存进去，将新的添加最后，并将新建的数组赋给listeners属性
* 删除时先找到要删除的监听器在数组中的序号，新建一个比当前小一的数组，将除了要删除的监听器所在的序号的元素按顺序添加进入，最后再赋值给listeners属性

```java
private LifecycleListener listeners[] = new LifecycleListener[0];
private final Object listenersLock = new Object(); // Lock object for changes to listeners
public void addLifecycleListener(LifecycleListener listener) {
  synchronized (listenersLock) {
      LifecycleListener results[] = new LifecycleListener[listeners.length + 1];
      for (int i = 0; i < listeners.length; i++)
          results[i] = listeners[i];
      results[listeners.length] = listener;
      listeners = results;
  }
}
public LifecycleListener[] findLifecycleListeners() {
    return listeners;
}
public void removeLifecycleListener(LifecycleListener listener) {
    synchronized (listenersLock) {
        int n = -1;
        for (int i = 0; i < listeners.length; i++) {
            if (listeners[i] == listener) {
                n = i;
                break;
            }
        }
        if (n < 0)
            return;
        LifecycleListener results[] = new LifecycleListener[listeners.length - 1];
        int j = 0;
        for (int i = 0; i < listeners.length; i++) {
            if (i != n)
                results[j++] = listeners[i];
        }
        listeners = results;
    }
}
```

​	LifecycleSupport中还定义了处理LifecycleEvent实践的fireLifecycleEvent方法，按事件的类型(组件的状态)创建了LifecycleEvent事件，然后遍历所有的监听器进行处理。

```java
public void fireLifecycleEvent(String type, Object data) {
    LifecycleEvent event = new LifecycleEvent(lifecycle, type, data);
    LifecycleListener interested[] = listeners;
    for (int i = 0; i < interested.length; i++)
        interested[i].lifecycleEvent(event);
}
```

### 四个生命周期方法

​	四个声明周期方法的实现首先判断当前的状态和要处理的方法是否匹配，不匹配就会执行相应方法使其匹配(在init之前调用start，start方法里检验状态等，还是会先执行init方法)，或不处理甚至抛出异常，如果匹配或者处理后匹配了，会调用相应的模版方法并设置相应的状态。

#### init方法

​	LifecycleBase中的状态是通过LifecycleState类型的state属性来保存的，最开始初始化值为NEW，如果不为NEW会调用invalidTransition方法抛出异常，其他三个方法也会这样

```java
public final synchronized void init() throws LifecycleException {
    //最开始的值必须是NEW，否则会抛出异常
    if (!state.equals(LifecycleState.NEW)) {
        invalidTransition(Lifecycle.BEFORE_INIT_EVENT);
    }
    try {
      //初始化前将状态设置为INITALIZING
        setStateInternal(LifecycleState.INITIALIZING, null, false);
        initInternal();
      //初始化后将状态设置为INITALIZED
        setStateInternal(LifecycleState.INITIALIZED, null, false);
    } catch (Throwable t) {
        ExceptionUtils.handleThrowable(t);
        setStateInternal(LifecycleState.FAILED, null, false);
        throw new LifecycleException(
                sm.getString("lifecycleBase.initFail",toString()), t);
    }
}
```

#### start方法

	1. 通过状态检查是否已经启动，已启动则打印日志并直接返回
 	2. 如果没有初始化则先进性初始化，如果启动失败则关闭，如果状态无法吹了则抛出异常
 	3. 启动前将状态设置为LifecycleState.STARTING_PREP
 	4. 调用模版方法启动组件
 	5. 启动失败将状态设置为FALLED并执行stop方法停止
 	6. 启动状态不是STARTED则抛出异常
 	7. 成功启动后将状态设置为STARTED

```java
public final synchronized void start() throws LifecycleException {
    if (LifecycleState.STARTING_PREP.equals(state) || LifecycleState.STARTING.equals(state) ||
            LifecycleState.STARTED.equals(state)) {
        if (log.isDebugEnabled()) {
            Exception e = new LifecycleException();
            log.debug(sm.getString("lifecycleBase.alreadyStarted", toString()), e);
        } else if (log.isInfoEnabled()) {
            log.info(sm.getString("lifecycleBase.alreadyStarted", toString()));
        }
        return;
    }
    if (state.equals(LifecycleState.NEW)) {
        init();
    } else if (state.equals(LifecycleState.FAILED)) {
        stop();
    } else if (!state.equals(LifecycleState.INITIALIZED) &&
            !state.equals(LifecycleState.STOPPED)) {
        invalidTransition(Lifecycle.BEFORE_START_EVENT);
    }
    try {
        setStateInternal(LifecycleState.STARTING_PREP, null, false);
        startInternal();
        if (state.equals(LifecycleState.FAILED)) {
            stop();
        } else if (!state.equals(LifecycleState.STARTING)) {
            invalidTransition(Lifecycle.AFTER_START_EVENT);
        } else {
            setStateInternal(LifecycleState.STARTED, null, false);
        }
    } catch (Throwable t) {
        setStateInternal(LifecycleState.FAILED, null, false);
    }
}
```

#### stop方法

 	1. 通过状态检查是否已经关闭，已关闭则打印日志并直接返回
 	2. 如果是NEW状态，将当前state设置为STOPPED并返回
 	3. 如果状态不是STARTED也不是FAILED则抛出异常
 	4. 如果当前状态是FAILED则将当前标志为BEFORE_STOP_EVENT事件
 	5. 调用模版方法进行stop
 	6. 停止状态不是STOPPING和FAILED则抛出异常
 	7. 成功启动后将状态设置为STOPPED
 	8. finally里面执行destory方法

```java
public final synchronized void stop() throws LifecycleException {
    if (LifecycleState.STOPPING_PREP.equals(state) || LifecycleState.STOPPING.equals(state) ||
            LifecycleState.STOPPED.equals(state)) {
        if (log.isDebugEnabled()) {
        } else if (log.isInfoEnabled()) {
        }
        return;
    }
    if (state.equals(LifecycleState.NEW)) {
        state = LifecycleState.STOPPED;
        return;
    }
    if (!state.equals(LifecycleState.STARTED) && !state.equals(LifecycleState.FAILED)) {
        invalidTransition(Lifecycle.BEFORE_STOP_EVENT);
    }
    try {
        if (state.equals(LifecycleState.FAILED)) {
            fireLifecycleEvent(BEFORE_STOP_EVENT, null);
        } else {
            setStateInternal(LifecycleState.STOPPING_PREP, null, false);
        }
        stopInternal();
        if (!state.equals(LifecycleState.STOPPING) && !state.equals(LifecycleState.FAILED)) {
            invalidTransition(Lifecycle.AFTER_STOP_EVENT);
        }
        setStateInternal(LifecycleState.STOPPED, null, false);
    } catch (Throwable t) {
        setStateInternal(LifecycleState.FAILED, null, false);
    } finally {
        if (this instanceof Lifecycle.SingleUse) {
            setStateInternal(LifecycleState.STOPPED, null, false);
            destroy();
        }
    }
}
```

#### destory方法

1. 如果LifecycleState状态是FAILED则先执行stop方法
2. 如果状态是DESTROYING或DESTROYED 记录日志并退出
3. 如果状态不是STOPPED、FAILED、NEW、INITIALIZED 抛出异常
4. 销毁前将状态设置为LifecycleState.DESTROYING
 5. 调用模版方法销毁组件
 6. 销毁后将状态设置为DESTROYED

```java
public final synchronized void destroy() throws LifecycleException {
    if (LifecycleState.FAILED.equals(state)) {
        try {
            stop();
        } catch (LifecycleException e) {
        }
   
    if (!state.equals(LifecycleState.STOPPED) &&
            !state.equals(LifecycleState.FAILED) &&
            !state.equals(LifecycleState.NEW) &&
            !state.equals(LifecycleState.INITIALIZED)) {
        invalidTransition(Lifecycle.BEFORE_DESTROY_EVENT);
    }
    try {
        setStateInternal(LifecycleState.DESTROYING, null, false);
        destroyInternal();
        setStateInternal(LifecycleState.DESTROYED, null, false);
    } catch (Throwable t) {
        setStateInternal(LifecycleState.FAILED, null, false);
    }
}
```

#### setStateInternal()

该方法除了设置状态还检查设置的状态合不合逻辑，并在最后发布相应的事件

```java
private synchronized void setStateInternal(LifecycleState state,
        Object data, boolean check) throws LifecycleException {
    if (log.isDebugEnabled()) {
        log.debug(sm.getString("lifecycleBase.setState", this, state));
    }
    if (check) {
        if (state == null) {
            invalidTransition("null");
            return;
        }
        if (!(state == LifecycleState.FAILED ||
                (this.state == LifecycleState.STARTING_PREP &&
                        state == LifecycleState.STARTING) ||
                (this.state == LifecycleState.STOPPING_PREP &&
                        state == LifecycleState.STOPPING) ||
                (this.state == LifecycleState.FAILED &&
                        state == LifecycleState.STOPPING))) {
            invalidTransition(state.name());
        }
    }
    this.state = state;
    String lifecycleEvent = state.getLifecycleEvent();
    if (lifecycleEvent != null) {
        fireLifecycleEvent(lifecycleEvent, data);
    }
}
```

#### fireLifecycleEvent()

处理事件，该方法调用了LifecycleSupport的fireLifecycleEvent方法具体处理

```java
protected void fireLifecycleEvent(String type, Object data) {
    lifecycle.fireLifecycleEvent(type, data);
}
```

#### invalidTransition()

负责抛出指定状态的异常

```java
private void invalidTransition(String type) throws LifecycleException {
    String msg = sm.getString("lifecycleBase.invalidTransition", type,toString(), state);
    throw new LifecycleException(msg);
}
```

###两个获取当前状态的方法

​	在生命周期的相应方法已经将状态设置到了state属性，直接将state返回就可以

```java
public LifecycleState getState() {
    return state;
}
@Override
public String getStateName() {
    return getState().toString();
}
```

# 3 Container分析

## 3.1 ContainerBase的结构

​	Container是Tomcat中容器的接口，通常使用的Servlet就封装在其子接口Wrapper中 。

​	Container一共有四个子接口Engine、Host、Context、Wrapper和一个默认实现类ContainerBase。每个子接口都是一个容器，四个容器都有对应的StandardXX实现类，且这些实现类都继承COntainerBase类。

​	Container还继承Lifecycle接口，且ContainerBase间接继承LifecycleBase，所以Engine、Host、Context、Wrapper这四个子容器都符合Tomcat的生命周期管理模式。

<img src="images/image-20200416182455094.png" alt="image-20200416182455094" style="zoom: 50%;" />

## 3.2 Container的四个子容器

​	Container的子容器Engine、Host、Context、Wrapper是逐层包含的关系，Engine是最顶层，每个service最多只能有一个Engine，Engine里可以有多个Host，每个Host下可以有多个Context，每个Context下可以有多个Weapper。

<img src="images/image-20200416190914329.png" alt="image-20200416190914329" style="zoom:50%;" />

* Engine：引擎，用来管理多个站点，一个Service最多只能有一个Engine
* Host：代表一个站点，也叫虚拟主机，通过配置Host就可以添加站点 /localhost
* Context：代表一个应用程序localhost/test，对应着平时开发的一个程序，或者一个WEB-INF目录及下面的web.xml文件
* Wrapper：每个Wrapper封装着一个Servlet

## 3.3 四种容器的配置方法

### Engine和Host的配置

都在conf/server.xml文件中，server.xml文件是Tomcat中最重要的配置文件，大部分功能都在这配置

* 首先定义Server，8005端口监听关闭命令"SHUTDOWN"
* Server里定义了名为Cataline的Service
* Service定义了两个Connector：HTTP协议和AJP协议(通过AJP协议和另一个web容器进行交互，主要用于集成)
* Service还定义了名为Catalina的Engine
* Engine里定义了名为localhost的Host
  * defaultHost 表示请求的域名如果在所有的Host的name和Alias中都找不到时使用的默认Host，
* Host标签
  * name 表示定义的站点可以通过localhost访问
  * appBase 指定站点的位置，默认在webapps下，
  * unpackWARs 表示是否自动解压war文件
  * autoDeploy 表示是否自动部署（运行中新加入webapp目录的应用会自动部署并启动）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8" />
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
    <Engine name="Catalina" defaultHost="localhost">
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
      </Host>
    </Engine>
  </Service>
</Server>
```
### Context配置

​	Context有三种配置方法

  1. 通过文件配置，通过文件配置有五个位置可以配置

       1. conf/server.xml文件中的Context标签   只有在Tomcat重启才会重新加载

       2. conf/[enginename]/[hostname]/目录下以应用命令的xml文件

       3. 应用自己的/META-INF/context.xml文件

       4. conf/context.xml文件   配置的内容在整个Tomcat中共享

       5. conf/[enginename]/[hostname]/context.xml.default文件  在对应的host中共享

          前三个用于配置单独的应用，后两个配置的Context是共享的

  2. 将WAR应用直接放到Host的appBase配置的目录下，Tomcat会自动查找并添加到Host中

  3. 将应用的文件夹放到HostHost的appBase配置的目录下，Tomcat也会自动查找并添加到Host中

### Wrapper配置

​	就是在web.xml中配置的Servlet，一个Servlet对应一个Wrapper。

​	也可以在conf/web.xml配置全局的Wrapper，处理Jsp的JspServlet就配置在这里，不需要自己配置Jsp就可以处理Jsp请求了

## 3.4 Container的启动

​	Container启动是通过init和start方法来完成的，这两个方法会在Tomcat启动时被Service调用。

​	Container也是按照Tomcat的生命周期来管理的，init和start方法也会调用initInternal和startInternal方法具体处理，不过Container和Tomcat整体结构启动的过程稍微有些不一样

		1. Container的四个子容器有一个共同的父类ContainerBase，定义了Container容器的initInternal和startInternal方法通用处理内容，具体容器还可以添加自己的内容
  		2. 除了最顶层容器的init被Service调用的，子容器的init方法并不是在容器中逐层循环调用的，而是在执行start方法时通过状态判断还没有初始化才会调用
        		3. start方法除了在父容器的startInternal方法中调用，还会在父容器的添加子容器的addChild方法中调用，主要是因为Context和Wrapper是动态添加的，

### ContainerBase的initInternal

​	该方法主要初始化ThreadPoolExecutor类型的startStopExecutor属性，用于管理启动和关闭的线程。这里并没有设置生命周期的相应状态，所以如果具体容器也没有设置相应生命周期状态，那么即使已经调用init方法进行初始化，在start进行启动前也会再次调用init方法。

> ​	ThreadPoolExecutor继承Executor用于管理线程，特别是Runable类型的线程。

```java
protected void initInternal() throws LifecycleException {
    BlockingQueue<Runnable> startStopQueue =
        new LinkedBlockingQueue<Runnable>();
    startStopExecutor = new ThreadPoolExecutor(
            getStartStopThreadsInternal(),
            getStartStopThreadsInternal(), 10, TimeUnit.SECONDS,
            startStopQueue,
            new StartStopThreadFactory(getName() + "-startStop-"));
    startStopExecutor.allowCoreThreadTimeOut(true);
    super.initInternal();
}
```

### ContainerBase的startInternal

​	该方法主要做了五件事

 1. **如果有cluster和Realm则调用其start方法**

    cluster用于配置集群，作用同步Session；Realm是Tomcat的安全域，同来管理资源的访问权限

 2. **调用所有子容器的start方法启动子容器**

    子容器使用stratStopExecutor调用新线程启动的，这样可以用多个线程同时启动，效率更高，具体是通过for循环对每个子容器启动一个线程，并将返回的Future保存在List中，然后遍历每个Future并调用其get方法。

    > 遍历Future有两个作用：
    >
    > 		1. 其get方法是阻塞的，只有线程处理完后才会向下走，保证了管道pipeline启动前容器已经启动完成了		
    >
    >   		2. 可以处理启动过程中遇到的异常 

 3. **调用管道中Value的start方法来启动管道**

 4. **启动完成后将生命周期状态设置为LifecycleState.STARTING状态**

 5. **启用后台线程定时处理一些事**

    ```java
    protected synchronized void startInternal() throws LifecycleException {
        Loader loader = getLoaderInternal();
        if ((loader != null) && (loader instanceof Lifecycle))
            ((Lifecycle) loader).start();
        getLogger();
        Manager manager = getManagerInternal();
        if ((manager != null) && (manager instanceof Lifecycle))
            ((Lifecycle) manager).start();
        Cluster cluster = getClusterInternal();
        if ((cluster != null) && (cluster instanceof Lifecycle))
            ((Lifecycle) cluster).start();
        Realm realm = getRealmInternal();
        if ((realm != null) && (realm instanceof Lifecycle))
            ((Lifecycle) realm).start();
        DirContext resources = getResourcesInternal();
        if ((resources != null) && (resources instanceof Lifecycle))
            ((Lifecycle) resources).start();
       //获取所有子容器
        Container children[] = findChildren();
        List<Future<Void>> results = new ArrayList<Future<Void>>();
        for (int i = 0; i < children.length; i++) {
            results.add(startStopExecutor.submit(new StartChild(children[i])));
        }
        boolean fail = false;
      //处理子容器 启动线程的Future
        for (Future<Void> result : results) {
            try {
                result.get();
            } catch (Exception e) {
                fail = true;
            }
        }
      //启动管道
        if (pipeline instanceof Lifecycle)
            ((Lifecycle) pipeline).start();
      //将生命周期状态设置为LifecycleState.STARTING
        setState(LifecycleState.STARTING);
      //启动后台线程
        threadStart();
    }
    ```

    ​	启动子容器的线程类型StartChild是实现了Callable的内部类，作用就是调用子容器的start方法

    ```java
    private static class StartChild implements Callable<Void> {
        private Container child;
        public StartChild(Container child) {
            this.child = child;
        }
        @Override
        public Void call() throws LifecycleException {
            child.start();
            return null;
        }
    }
    ```

    ​	threadStart方法启动的后台线程是一个while循环，内部定期调用backgroundProcess方法做些事，间隔时间通过ContainerBase的backgroundProcessorDelay属性来设置，单位事秒，小于0不启动后台线程，其backgroundProcess方法会在父容器的后台线程中调用。

    ​	backgroundProcess方法是Container接口中的一个方法，共有三个实现，分别在ContainerBase、StandardContext和StandardWrapper中，ContainerBase提供了所有容器共同的处理过程。

    > ​	StandardContext和StandardWrapper的backgroundProcess方法除了处理自己相关的业务，也调用ContainerBase中的处理。
    >
    > ​	ContainerBase的backgroundProcess方法调用了Cluster、Realm和管道的backgroundProcess方法
    >
    > ​	StandardContext的backgroundProcess方法中对Session过期和资源变化进行了处理
    >
    > ​	StandardWrapper的backgroundProcess方法对Jsp生成的Servlet定期进行检查

### Engine

​	Service会调用最顶层容器的init和start方法，如果使用了Engine会调用Engine的。Engine的默认实现类StandardEngine中的initInternal和startInternal方法如下

```java
protected void initInternal() throws LifecycleException {
    getRealm();
    super.initInternal();
}
protected synchronized void startInternal() throws LifecycleException {
    super.startInternal();
}
```

​	它们分别调用ContainerBase中的相应方法，initInternal方法还调用了getRealm方法，如果没有配置Realm，则使用一个默认的NullRealm		

```java
public Realm getRealm() {
    Realm configured = super.getRealm();
    if (configured == null) {
        configured = new NullRealm();
        this.setRealm(configured);
    }
    return configured;
}
```

### Host

​	Host的默认实现类StandardHost没有重写initInternal方法，初始化默认调用ContainerBase的initInternal方法。startInternal方法如下，检查Host的管道中有没有指定的value，没有则添加进去。

> ​	检查的value的类型通过getErrorReportValveClass方法获取，它返回errorReportValveClass属性，可以配置，默认是ErrorReportValve
>
> private String errorReportValveClass = "org.apache.catalina.valves.ErrorReportValve"

```java
protected synchronized void startInternal() throws LifecycleException {
    String errorValve = getErrorReportValveClass();
    if ((errorValve != null) && (!errorValve.equals(""))) {
        try {
            boolean found = false;
            Valve[] valves = getPipeline().getValves();
            for (Valve valve : valves) {
                if (errorValve.equals(valve.getClass().getName())) {
                    found = true;
                    break;
                }
            }
            if(!found) {
 								Valve valve =(Valve)Class.forName(errorValve).getDeclaredConstructor().newInstance();
              	//管道中没有ErrorReportValv则将其加入管道
                getPipeline().addValve(valve);
            }
        }
    }
    super.startInternal();
}
```

​	Host的启动除了startInternal方法还有HostConfig中相应的方法，HostConfig继承子LifecycleListener的监听器(Engine对应的EngineConfig监听器只是做了日志记录)，在接受到Lifecycle.START_EVENT事件时会调用start方法启动，HostConfig的start方法会检查配置的Host站点的位置是否存在及是不是目录，最后调用deployApps方法部署应用。

```java
  protected void deployApps() {
        File appBase = appBase();
        File configBase = configBase();
        String[] filteredAppPaths = filterAppPaths(appBase.list());
    		// 部署XML描述文件
        deployDescriptors(configBase, configBase.list());
    		// 部署WAR文件
        deployWARs(appBase, filteredAppPaths);
    		// 部署 文件夹
        deployDirectories(appBase, filteredAppPaths);
    }
```

 * XML文件指conf/[enginename]/[hostname]/*.xml文件

 * WAR文件和文件夹是Host站点目录下的，会自动找出来并部署

   部署后会将部署的Context通过StandardHost的addChild方法添加到Host里面。

   StandardHost的addChild方法会调用父类ContainerBase的addChild方法，其中会调用子类(这里指Context)的start方法来启动子容器

### Context

​	Context的默认实现类StandardContext在startInteral方法中调用了在web.xml定义的Listener，还初始化了其中的Filter和load-on-startup的Servlet

```java
protected synchronized void startInternal() throws LifecycleException {
// 触发listener
if (ok) {
    if (!listenerStart()) {
        log.error(sm.getString("standardContext.listenerFail"));
        ok = false;
    }
}
// 初始化Filter
if (ok) {
     if (!filterStart()) {
       log.error(sm.getString("standardContext.filterFail"));
       ok = false;
     }
   }
// 初始化Servlets
if (ok) {
  if (!loadOnStartup(findChildren())){
    log.error(sm.getString("standardContext.servletFail"));
    ok = false;
  }
}
```

​	listenerStart、filterStart和loadOnStartup方法分别调用配置在Listener的contextInitialized方法及Filter和配置了load-on-startup的Servlet的init方法

​	Context和Host一样也有个LifecycleListener类型的监听器ContextConfig，其中configureStart方法用来处理CONFIGURE_START_CEVENT事件，这个方法里面调用webConfig方法，webConfig方法解析了web.xml文件，相应地创建了Wrapper并使用addChild添加到了Context里面

### Wrapper	

​	Wrapper的默认实现类StandardWrapper没有重写initInternal方法，初始化时会默认调用ContainerBase的initInternal方法

```java
protected synchronized void startInternal() throws LifecycleException {
    if (this.getObjectName() != null) {
        Notification notification = new Notification("j2ee.state.starting",
                                                    this.getObjectName(),
                                                    sequenceNumber++);
        broadcaster.sendNotification(notification);
    }
    super.startInternal();
    setAvailable(0L);
    if (this.getObjectName() != null) {
        Notification notification =
            new Notification("j2ee.state.running", this.getObjectName(),
                            sequenceNumber++);
        broadcaster.sendNotification(notification);
    }
}
```

​	主要做了三件事

 1. 用broadcaster发送通知，用于JMX

 2. 调用了父类ContainerBase中的startInternal方法

 3. 调用setAvaliable方法让Servlet有效

    ​	该方法是Wrapper接口的方法，作用是设置Wrapper所包含的Servlet有效的起始时间，如果设置的是将来的事件，那么调用所对应的Servlet就会产生错误，知道过了所设置的事件后才可以正常调用。

Wrapper没有XXConfig样式的LifecycleListener监听器

# 4 Pipeline-Valve管道

​	上一节讲了Container的创建过程，Container处理请求是使用Pipeline-valve管道处理的。下面先分析它的处理模式，然后分析实现方法

## 4.1 Pipeline-Valve处理模式

​	Pipeline-Valve是责任链模式，在一个请求处理过程中有多个处理者依次对请求进行处理。

> ​		Pipeline相当于处理请求的路线，Valve相当于每个处理者处理的过程

​	Pipeline-Valve的管道模型和普通的责任链模式有些不同，区别有两点：

  1. 每个Pipeline都有特定的Valve，且在管道的最后一个执行，这个Valve叫BaseValve，是不可删除的

  2. 上层容器的管道的BaseValue会调用下层容器的管道。4个容器的BaseValue分别是StandardEngineValve、StandardHostValve、StandardContextValve和StandardWrapperValve。处理流程如下

     <img src="images/image-20200417083137246.png" alt="image-20200417083137246" style="zoom:50%;" />

     > ​	Filter中的FilterChain就是这种模式，FilterChain相当于Pipeline，每个Filter相当于一个Value，Servlet相当于对后的BaseValue

<img src="images/image-20200417082031787.png" alt="image-20200417082031787" style="zoom: 50%;" />

## 4.2 Pipeline-Valve的实现方法

​	Pipeline管道的实现分为生命周期管理和处理请求两部分

### Pipeline管道生命周期的实现方法

​	Container中的Pipeline在抽象类ContainerBase中定义，并在生命周期的startInternal、stopInternal、destoryInternal方法中调用管道的相应生命周期方法(因为管道不需要初始化所以initInternal没有调用)

```java
public abstract class ContainerBase extends LifecycleMBeanBase
        implements Container
protected synchronized void startInternal() throws LifecycleException {
		if (pipeline instanceof Lifecycle)
    		((Lifecycle) pipeline).start();
  
protected synchronized void stopInternal() throws LifecycleException {
          if (pipeline instanceof Lifecycle &&
                ((Lifecycle) pipeline).getState().isAvailable()) {
            ((Lifecycle) pipeline).stop();
        }
protected void destroyInternal() throws LifecycleException {
          if (pipeline instanceof Lifecycle) {
            ((Lifecycle) pipeline).destroy();
        }
```

​	Container的四个子容器都继承自ContainerBase，所以四个子容器在执行生命周期的方法时都会调用管道相应的生命周期方法

​	Pipeline使用的是StanardPipeline类型，里面的Value保存在first属性中，Value是链式结构，可以通过getNext方法一次获取每个Valve，BaseValue单独保存在basic属性中(basic不能为空，在调用addValve方法添加Value时basic会同时保存到dirst的最后一个，如果没调用addValve方法first可能为空)。

​	StanardPipeline继承子LifecycleBase，实际处理生命周期的方法是startInternal、stopInternal和destoryInternel	

```java
public class StandardPipeline extends LifecycleBase implements Pipeline, Contained {
  protected synchronized void startInternal() throws LifecycleException {
    // 使用临时变量current遍历Value链里的所有Value，如果first为空则使用basic
    Valve current = first;
    if (current == null) {
      current = basic;
    }
    //遍历所有Value并调用start方法
    while (current != null) {
      if (current instanceof Lifecycle)
        ((Lifecycle) current).start();
      current = current.getNext();
    }
    //设置LifecycleState.STARTING状态
    setState(LifecycleState.STARTING);
  }
    protected synchronized void stopInternal() throws LifecycleException {
        setState(LifecycleState.STOPPING);
        Valve current = first;
        if (current == null) {
            current = basic;
        }
        while (current != null) {
            if (current instanceof Lifecycle)
                ((Lifecycle) current).stop();
            current = current.getNext();
        }
    }
		//删除所有的Valve
    protected void destroyInternal() {
        Valve[] valves = getValves();
        for (Valve valve : valves) {
            removeValve(valve);
        }
    }
```

​	getValve方法得到basic在内的所有Value集合

```java
public Valve[] getValves() {
    ArrayList<Valve> valveList = new ArrayList<Valve>();
    Valve current = first;
    if (current == null) {
        current = basic;
    }
    while (current != null) {
        valveList.add(current);
        current = current.getNext();
    }
    return valveList.toArray(new Valve[0]);
}
```

### Pipeline管道处理请求的实现方法

​	pipeline调用所包含Value的invoke方法处理请求，并在BaseValue里又调用了子容器pipeline所包含Value的invoke方法，直到最后调用了Wrapper的Pipeline所包含的BaseValue——StandardWrapperValve。

* Connector接收到请求后调用最顶层容器的pipeline处理

* 顶层容器的Pipeline处理后会在其BaseValue里调用下一层容器的pipeline进行处理，这样就可以 逐层调用所有容器的pipeline进行处理了。

  Engine的BaseValue是StandardEngineValve，它的invoke方法如下

  ​	1. Host事先设置到request中，其他各层容器也会实现设置到request中

  ​	2. 然后将请求传递到Host的管道

  ​	3. Host的BaseValue也会调用Context的pipeline

  ​	4. Context的BaseValue调用Wrapper的pipeline

  ​	5. Wrapper的pipeline最后在其BaseValue(StandardWrapperValve)中创建FilterChain并调用其doFilter方法处理请求

  ​	6. FilterChain包含我们配置的和请求相匹配的Filter和Servlet。它的doFilter方法会依次调用所有Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理

  ```java
  public final void invoke(Request request, Response response)
      throws IOException, ServletException {
      Host host = request.getHost();
      if (host == null) {
          response.sendError
              (HttpServletResponse.SC_BAD_REQUEST,
               sm.getString("standardEngine.noHost", 
                            request.getServerName()));
          return;
      }
      if (request.isAsyncSupported()) {
          request.setAsyncSupported(host.getPipeline().isAsyncSupported());
      }
      host.getPipeline().getFirst().invoke(request, response);
  }
  ```
  
  

# 5 Connector分析

​		Connector用于接收请求并将请求封装成Request和Response具体处理，底层使用Socket进行连接，Request和Response是按照HTTP协议来封装的，所以Connector同时实现了TCP/IP协议和HTTP协议。	     

​		Request和Response封装完后交给Container进行处理，Container就是Servlet的容器，Container处理完返回给Connector，最后Connector使用Socket将处理结果返回给客户端，整个请求就处理结束。

## 5.1 Connector的结构

​	Connector中具体是用ProtocolHandler处理请求的，不同的ProtocolHandler代表不同的连接类型。

> ​	Http11Protocol使用普通Socket连接，Http11NioProtocol使用NioSocket连接的
>

​	  ProtocolHandler有三个重要组件：Endpoint、Processor和Adapter。

​		**Endpoint 处理底层Socket的网络连接 —— 实现TCP/IP协议**

​		**Process 将Endpoint接收到的Socket封装成Request —— 实现HTTP协议**

​		**Adapter 用于将封装好的Request交给Container进行具体处理 —— 将请求适配到Servlet容器进行处理**

  	Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口

​		Acceptor 用于监听请求

​		AsyncTimeout 检查异步request的超时

​		Handler 处理接收到的Socket，在内部调用Process进行处理

​	<img src="images/image-20200417091936647.png" alt="image-20200417091936647" style="zoom: 67%;" />    

<img src="images/image-20200417095001065.png" alt="image-20200417095001065" style="zoom: 50%;" />

## 5.2 Connector自身类

​	Connector类的作用是为了在它创建时 创建ProtocolHandler，然后在生命周期的相关方法中调用了ProtocolHandler的相关生命周期方法。

​	Connector的使用方法是通过Connector标签配置在conf/server.xml文件中，所以Connector是在Catalina的load方法中根据conf/server.xml配置文件创建Server对象时创建的。Connector的生命周期方法是在Service中调用的。

### Connector的创建

​	Connector的创建过程主要是初始化ProtocolHandler。根据server.xml配置的Connector的protocol属性，会设置到Connector构造函数的参数中，用于指定ProtocolHandler的类型。

> ​	根据protocol参数指定protocolHandlerClassName，根据反射实例化ProtocolHandler并赋值给当前的protocolHandler属性

```java
public Connector(String protocol) {
    setProtocol(protocol);
    try {
        Class<?> clazz = Class.forName(protocolHandlerClassName);
        this.protocolHandler = (ProtocolHandler) clazz.getDeclaredConstructor().newInstance();
    } catch (Exception e) {
    }
}
```

​	setProtocol方法设置了protocolHandlerClassName属性

​	APr是Apache Protable Runtime的缩写，是Apache提供的一个运行时环境，使用它需要安装，Tomcat可以自己检测出来，并使用apr里面的协议。

​	没有安装apr，会根据配置的HTTP/1.1属性将protocolHandlerClassName设置为org.apache.coyote.http11.Http11Protocol

> ```
> tomcat8 默认使用 org.apache.coyote.http11.Http11NioProtocol
> ```

```java
public void setProtocol(String protocol) {
    if (AprLifecycleListener.isAprAvailable()) {
        if ("HTTP/1.1".equals(protocol)) {
            setProtocolHandlerClassName
            ("org.apache.coyote.http11.Http11AprProtocol");
        } else if ("AJP/1.3".equals(protocol)) {
            setProtocolHandlerClassName
            ("org.apache.coyote.ajp.AjpAprProtocol");
        } else if (protocol != null) {
            setProtocolHandlerClassName(protocol);
        } else {
            setProtocolHandlerClassName
            ("org.apache.coyote.http11.Http11AprProtocol");
        }
    } else {
        if ("HTTP/1.1".equals(protocol)) {
            setProtocolHandlerClassName
            ("org.apache.coyote.http11.Http11Protocol");
        } else if ("AJP/1.3".equals(protocol)) {
            setProtocolHandlerClassName
            ("org.apache.coyote.ajp.AjpProtocol");
        } else if (protocol != null) {
            setProtocolHandlerClassName(protocol);
        }
    }
}
```

### Connector生命周期处理方法

​	主要调用了ProtocolHandler的相应生命周期方法

```java
public class Connector extends LifecycleMBeanBase
  
protected void initInternal() throws LifecycleException {
    super.initInternal();
  	//新建adapter，并设置到protocolHandler
    adapter = new CoyoteAdapter(this);
    protocolHandler.setAdapter(adapter);
    if( null == parseBodyMethodsSet ) {
        setParseBodyMethods(getParseBodyMethods());
    }
    try {
        protocolHandler.init();
    } catch (Exception e) {
    }
    // 初始化Initialize mapper listener
    mapperListener.init();
}

protected void startInternal() throws LifecycleException {
  //判断端口小于0抛出异常
   if (getPort() < 0) {
            throw new LifecycleException(sm.getString(
                    "coyoteConnector.invalidPort", Integer.valueOf(getPort())));
        }
  setState(LifecycleState.STARTING);
  	try {
    protocolHandler.start();
  } catch (Exception e) {
  }
  mapperListener.start();
}

 protected void stopInternal() throws LifecycleException {
        setState(LifecycleState.STOPPING);
        try {
            protocolHandler.stop();
        } catch (Exception e) {
        }
        mapperListener.stop();
    }

protected void destroyInternal() throws LifecycleException {
  		//调用MapperListener的destory方法
        mapperListener.destroy();
        try {
          //调用protocolHandler的destory方法
            protocolHandler.destroy();
        } catch (Exception e) {
        }
        if (getService() != null) {
           //将当前的Connector从service中删除
            getService().removeConnector(this);
        }
  		 //调用父类的destoryInternel方法
        super.destroyInternal();
    }
```

## 5.3 ProtocolHandler

​	Tomcat的ProtocolHandler的继承结构图

![image-20200417101242718](images/image-20200417101242718.png)

​	ProtocolHandler有一个抽象实现类AbstractProtocol，下面分了两类

	1. AbstractAjpProtocol	又分为AjpApr、AjpNio和Ajp协议
 	2. AbstractHTTP11Protocol 分为AbstractHttp11jsse（Http11Nio和Http11）、Http11Apr

​	Http11Nio的构造函数中创建了NioEndpoint类型的Endpoint，并新建了Http11ConnectionHandler类型的Handler然后设置到了Endpoint

```java
public class Http11NioProtocol extends AbstractHttp11JsseProtocol<NioChannel> {
    public Http11NioProtocol() {
        endpoint=new NioEndpoint();
        cHandler = new Http11ConnectionHandler(this);
        ((NioEndpoint) endpoint).setHandler(cHandler);
        setSoLinger(Constants.DEFAULT_CONNECTION_LINGER);
        setSoTimeout(Constants.DEFAULT_CONNECTION_TIMEOUT);
        setTcpNoDelay(Constants.DEFAULT_TCP_NO_DELAY);
    }
```

​	四个生命周期方法是在父类AbstractProtocol中实现的，主要调用了Endpoing的生命周期方法。

## 5.4 处理TPC/IP协议的Endpoint

​	Endpoint用于处理具体连接和传输数据，NioEndpoint继承子AbstractEndpoing，在NioEndpoint中新增了Poller和SocketProcessor内部类

<img src="images/image-20200417102448306.png" alt="image-20200417102448306" style="zoom: 67%;" />	

​	NioEndpoint的init和start方法在父类AbstractEndpoint中

```java
public abstract class AbstractEndpoint<S> {
public void init() throws Exception {
    if (this.bindOnInit) {
        this.bind();
        this.bindState = AbstractEndpoint.BindState.BOUND_ON_INIT;
    }
}
public final void start() throws Exception {
    if (this.bindState == AbstractEndpoint.BindState.UNBOUND) {
        this.bind();
        this.bindState = AbstractEndpoint.BindState.BOUND_ON_START;
    }
    this.startInternal();
}
```

​	这两个方法主要调用bind和startInternal方法，他们是模版方法，在NioEndpoint中实现

> ​		acceptor用于接收请求，然后交给poller处理

```java
public class NioEndpoint extends AbstractEndpoint<NioChannel> {

public void bind() throws Exception {
    serverSock = ServerSocketChannel.open();
    socketProperties.setProperties(serverSock.socket());
    InetSocketAddress addr = (getAddress()!=null?new InetSocketAddress(getAddress(),getPort()):new InetSocketAddress(getPort()));
    serverSock.socket().bind(addr,getBacklog());
    serverSock.configureBlocking(true); //mimic APR behavior
    serverSock.socket().setSoTimeout(getSocketProperties().getSoTimeout());
		//	初始化需要启动accrptor线程的个数，为0则改为1，否则不能工作
    // Initialize thread count defaults for acceptor, poller
    if (acceptorThreadCount == 0) {
        // FIXME: Doesn't seem to work that well with multiple accept threads
        acceptorThreadCount = 1;
    }
  	// 初始化需要启动poller的线程个数，<=0 改为1
    if (pollerThreadCount <= 0) {
        //minimum one poller thread
        pollerThreadCount = 1;
    }
    stopLatch = new CountDownLatch(pollerThreadCount);
    // Initialize SSL if needed 是否需要初始化SSL
    if (isSSLEnabled()) {
        SSLUtil sslUtil = handler.getSslImplementation().getSSLUtil(this);

        sslContext = sslUtil.createSSLContext();
        sslContext.init(wrap(sslUtil.getKeyManagers()),
                sslUtil.getTrustManagers(), null);

        SSLSessionContext sessionContext =
            sslContext.getServerSessionContext();
        if (sessionContext != null) {
            sslUtil.configureSessionContext(sessionContext);
        }
        // Determine which cipher suites and protocols to enable
        enabledCiphers = sslUtil.getEnableableCiphers(sslContext);
        enabledProtocols = sslUtil.getEnableableProtocols(sslContext);
    }

    if (oomParachute>0) reclaimParachute(true);
    selectorPool.open();
}

  public void startInternal() throws Exception {
        if (!running) {
            running = true;
            paused = false;
            // Create worker collection 创建Executor
            if ( getExecutor() == null ) {
                createExecutor();
            }
            initializeConnectionLatch();
            // Start poller threads 启动poller线程
            pollers = new Poller[getPollerThreadCount()];
            for (int i=0; i<pollers.length; i++) {
                pollers[i] = new Poller();
                Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
                pollerThread.setPriority(threadPriority);
                pollerThread.setDaemon(true);
                pollerThread.start();
            }
           //启动acceptor线程
            startAcceptorThreads();
        }
    }
```

​	startAcceptorThreads方法，getAcceptorThreadCount()是获取的init方法中处理过的acceptorThreadCount属性，然后启动相应数量的Acceptor线程来接收请求。

```java
protected final void startAcceptorThreads() {
    int count = getAcceptorThreadCount();
    acceptors = new Acceptor[count];
    for (int i = 0; i < count; i++) {
        acceptors[i] = createAcceptor();
        String threadName = getName() + "-Acceptor-" + i;
        acceptors[i].setThreadName(threadName);
        Thread t = new Thread(acceptors[i], threadName);
        t.setPriority(getAcceptorThreadPriority());
        t.setDaemon(getDaemon());
        t.start();
    }
}
```

## 5.5 处理HTTP协议的Processor

​	Processor用于处理应用层协议(如HTTP)，有两个AbstractProcessor抽象类，分别在coyote包和upgrade包。(左边和右边)，右边是Servlet3.1后新增的，用于处理HTTP的升级协议。

> ​	如http升级WebSocket协议，当正常的Processor处理后如果Socket状态是UPGRADING，那么Endpoint中的Handler将会接着创建并调用upgrade包中的processor进行处理。

​	![image-20200417104222859](images/image-20200417104222859.png)

​	coyote包中的Processor和前面的ProtocolHandler对应，具体实现层协议处理请求的是AbstractAjpProcess和AbstractHttp11Processor中的process方法，这个方法首先封装了Request和Response，然后调用Adapter将请求传递到了Container中，最后对处理的结果进行处理，如有没有启动异步处理、处理过程中有没有抛出异常。

## 5.6 适配器Adapter

​	Adapter只有一个实现类，就是connector包下的CoyoteAdapter类。

​	Processor在process方法中会调用Adapter的sevice方法处理请求

​		Adapter的service方法主要调用Container管道中的invoke方法处理请求

​		处理前对Request和Response做了处理，将原来创建的coyote包下的Request和Response封装成connector的Request和Response，并在处理完成后判断是否启动了Comet(长连接模式)和是否启动了异步模式，并作出相应处理。

​		调用Container管道的相应代码

```java
public class CoyoteAdapter implements Adapter {
public void service(org.apache.coyote.Request req,
                    org.apache.coyote.Response res)
    throws Exception {

    Request request = (Request) req.getNote(ADAPTER_NOTES);
    Response response = (Response) res.getNote(ADAPTER_NOTES);

    if (request == null) {

        // Create objects
        request = connector.createRequest();
        request.setCoyoteRequest(req);
        response = connector.createResponse();
        response.setCoyoteResponse(res);

        // Link objects
        request.setResponse(response);
        response.setRequest(request);

        // Set as notes
        req.setNote(ADAPTER_NOTES, request);
        res.setNote(ADAPTER_NOTES, response);

        // Set query string encoding
        req.getParameters().setQueryStringEncoding
            (connector.getURIEncoding());
              req.getRequestProcessor().setWorkerThreadName(Thread.currentThread().getName());
            postParseSuccess = postParseRequest(req, request, res, response);
            if (postParseSuccess) {
                //check valves if we support async
                request.setAsyncSupported(connector.getService().getContainer().getPipeline().isAsyncSupported());
                // Calling the container
                connector.getService().getContainer().getPipeline().getFirst().invoke(request, response);
```

​	这里首先从Connector中获取Service(Connector在initInternal方法中创建CoyoteAdapter时将自己设置到了CoyoteAdapter中)，

​	然后Service中获取Container，接着获取管道的第一个Value，最后调用invoke方法执行请求。

​	Service中保存的是最顶层的容器，当调用最顶层容器管道的invoke方法时，管道将逐层调用各层容器管道中的Value的invoke方法，知道最后调用Wrapper的管道中的BaseValue(StandardWrapperValve)来处理Filter和Servlet。





# EndPoint源码解析

我们知道了Tomcat分为两个大组件，一个连接器和一个容器。而我们这次要讲的`EndPoint`的组件就是属于连接器里面的。它是一个通信的端点，就是负责对外实现TCP/IP协议。`EndPoint`是个接口，它的具体实现类就是`AbstractEndpoint`，而`AbstractEndpoint`具体的实现类有`AprEndpoint`、`Nio2Endpoint`、`NioEndpoint`。

- `AprEndpoint`：对应的是APR模式，简单理解就是从操作系统级别解决异步IO的问题，大幅度提高服务器的处理和响应性能。但是启用这种模式需要安装一些其他的依赖库。
- `Nio2Endpoint`：利用代码来实现异步IO
- `NioEndpoint`：利用了JAVA的NIO实现了非阻塞IO，Tomcat默认启动是以这个来启动的，而这个也是我们的讲述重点。

## NioEndpoint中重要的组件

我们知道`NioEndpoint`的原理还是对于Linux的多路复用器的使用，而在多路复用器中简单来说就两个步骤。

1. 创建一个Selector，在它身上注册各种Channel，然后调用select方法，等待通道中有感兴趣的事件发生。
2. 如果有感兴趣的事情发生了，例如是读事件，那么就将信息从通道中读取出来。

而`NioEndpoint`为了实现上面这两步，用了五个组件来。这五个组件是`LimitLatch`、`Acceptor`、`Poller`、`SocketProcessor`、`Executor`

```java
/**
 * Threads used to accept new connections and pass them to worker threads.
 */
protected List<Acceptor<U>> acceptors;

/**
 * counter for nr of connections handled by an endpoint
 */
private volatile LimitLatch connectionLimitLatch = null;
/**
 * The socket pollers. 
 */
private Poller[] pollers = null;

内部类

SocketProcessor

/**
 * External Executor based thread pool.
 */
private Executor executor = null;

```

我们可以看到在代码中定义的这五个组件。具体这五个组件是干嘛的呢？

- `LimitLatch`：连接控制器，负责控制最大的连接数
- `Acceptor`：负责接收新的连接，然后返回一个`Channel`对象给`Poller`
- `Poller`：可以将其看成是NIO中`Selector`，负责监控`Channel`的状态
- `SocketProcessor`：可以看成是一个被封装的任务类
- `Executor`：Tomcat自己扩展的线程池，用来执行任务类

用图简单表示就是以下的关系

![image-20200421103218209](images/image-20200421103218209.png)

接下来我们就来分别的看一下每个组件里面关键的代码

## LimitLatch

我们上面说了`LimitLatch`主要是用来控制Tomcat所能接收的最大数量连接，如果超过了此连接，那么Tomcat就会将此连接线程阻塞等待，等里面有其他连接释放了再消费此连接。那么`LimitLatch`是如何做到呢？我们可以看`LimitLatch`这个类

```java
public class LimitLatch {

    private static final Log log = LogFactory.getLog(LimitLatch.class);

    private class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 1L;

        public Sync() {
        }

        @Override
        protected int tryAcquireShared(int ignored) {
            long newCount = count.incrementAndGet();
            if (!released && newCount > limit) {
                // Limit exceeded
                count.decrementAndGet();
                return -1;
            } else {
                return 1;
            }
        }

        @Override
        protected boolean tryReleaseShared(int arg) {
            count.decrementAndGet();
            return true;
        }
    }

    private final Sync sync;
    //当前连接数
    private final AtomicLong count;
    //最大连接数
    private volatile long limit;
    private volatile boolean released = false;
}
```

我们可以看到它内部实现了`AbstractQueuedSynchronizer`，AQS其实就是一个框架，实现它的类可以自定义控制线程什么时候挂起什么时候释放。`limit`参数就是控制的最大连接数。我们可以看到`AbstractEndpoint`调用`LimitLatch`的`countUpOrAwait`方法来判断是否能获取连接。

```
    public void countUpOrAwait() throws InterruptedException {
        if (log.isDebugEnabled()) {
            log.debug("Counting up["+Thread.currentThread().getName()+"] latch="+getCount());
        }
        sync.acquireSharedInterruptibly(1);
    }
```

AQS是如何知道什么时候阻塞线程呢？即不能获取连接呢？这些就靠用户自己实现`AbstractQueuedSynchronizer`自己来定义什么时候获取连接，什么时候释放连接了。可以看到Sync类重写了`tryAcquireShared`和`tryReleaseShared`方法。在`tryAcquireShared`方法中定义了一旦当前连接数大于了设置的最大连接数，那么就会返回`-1`表示将此线程放入AQS队列中等待。

## Acceptor

`Acceptor`是接收连接的，我们可以看到`Acceptor`实现了`Runnable`接口，那么在哪会新开启线程来执行`Acceptor`的run方法呢？在`AbstractEndpoint`的`startAcceptorThreads`方法中。

```java
protected void startAcceptorThreads() {
    int count = getAcceptorThreadCount();
    acceptors = new ArrayList<>(count);

    for (int i = 0; i < count; i++) {
        Acceptor<U> acceptor = new Acceptor<>(this);
        String threadName = getName() + "-Acceptor-" + i;
        acceptor.setThreadName(threadName);
        acceptors.add(acceptor);
        Thread t = new Thread(acceptor, threadName);
        t.setPriority(getAcceptorThreadPriority());
        t.setDaemon(getDaemon());
        t.start();
    }
}
```

可以看到这里可以设置开启几个`Acceptor`，默认是一个。而一个端口只能对应一个`ServerSocketChannel`，那么这个`ServerSocketChannel`在哪初始化呢？我们可以看到在`Acceptor acceptor = new Acceptor<>(this);`这句话中传入了this进去，那么应该是由`Endpoint`组件初始化的连接。在`NioEndpoint`的`initServerSocket`方法中初始化了连接。

```java
// Separated out to make it easier for folks that extend NioEndpoint to
// implement custom [server]sockets
protected void initServerSocket() throws Exception {
    if (!getUseInheritedChannel()) {
        serverSock = ServerSocketChannel.open();
        socketProperties.setProperties(serverSock.socket());
        InetSocketAddress addr = new InetSocketAddress(getAddress(), getPortWithOffset());
        serverSock.socket().bind(addr,getAcceptCount());
    } else {
        // Retrieve the channel provided by the OS
        Channel ic = System.inheritedChannel();
        if (ic instanceof ServerSocketChannel) {
            serverSock = (ServerSocketChannel) ic;
        }
        if (serverSock == null) {
            throw new IllegalArgumentException(sm.getString("endpoint.init.bind.inherited"));
        }
    }
    serverSock.configureBlocking(true); //mimic APR behavior
}
```

这里面我们能够看到两点

1. 在bind方法中的第二个参数表示操作系统的等待队列长度，即Tomcat不再接受连接时（达到了设置的最大连接数），但是在操作系统层面还是能够接受连接的，此时就将此连接信息放入等待队列，那么这个队列的大小就是此参数设置的。
2. `ServerSocketChannel`被设置成了阻塞的模式，也就是说是以阻塞方式接受连接的。或许会有疑问。在平时的NIO编程中Channel不是都要设置成非阻塞模式吗？这里解释一下，如果是设置成非阻塞模式那么就必须设置一个`Selector`不断的轮询，但是接受连接只需要阻塞一个通道即可。

<img src="images/image-20200421104447503.png" alt="image-20200421104447503" style="zoom: 50%;" />

这里需要注意一点，每个`Acceptor`在生成`PollerEvent`对象放入`Poller`队列中时都是随机取出`Poller`对象的，具体代码可以看如下，所以`Poller`中的`Queue`对象设置成了`SynchronizedQueue`，因为可能有多个`Acceptor`同时向此`Poller`的队列中放入`PollerEvent`对象。

```
public Poller getPoller0() {
    if (pollerThreadCount == 1) {
        return pollers[0];
    } else {
        int idx = Math.abs(pollerRotater.incrementAndGet()) % pollers.length;
        return pollers[idx];
    }
}
```

> 什么是操作系统级别的连接呢？在TCP的三次握手中，系统通常会每一个LISTEN状态的Socket维护两个队列，一个是半连接队列（SYN）：这些连接已经收到客户端SYN;另一个是全连接队列（ACCEPT）：这些链接已经收到客户端的ACK，完成了三次握手，等待被应用调用accept方法取走使用。

所有的`Acceptor`共用这一个连接，在`Acceptor`的`run`方法中，放一些重要的代码。

```
 @Override
    public void run() {
        // Loop until we receive a shutdown command
        while (endpoint.isRunning()) {
            try {
                //如果到了最大连接数,线程等待
                endpoint.countUpOrAwaitConnection();
                U socket = null;
                try {
                    //调用accept方法获得一个连接
                    socket = endpoint.serverSocketAccept();
                } catch (Exception ioe) {
                    // 出异常以后当前连接数减掉1
                    endpoint.countDownConnection();
                }
                // 配置Socket
                if (endpoint.isRunning() && !endpoint.isPaused()) {
                    // setSocketOptions() will hand the socket off to
                    // an appropriate processor if successful
                    if (!endpoint.setSocketOptions(socket)) {
                        endpoint.closeSocket(socket);
                    }
                } else {
                    endpoint.destroySocket(socket);
                }
    }
```

里面我们可以得到两点

1. 运行时会先判断是否到达了最大连接数，如果到达了那么就阻塞线程等待，里面调用的就是`LimitLatch`组件判断的。
2. 最重要的就是配置socket这一步了，是`endpoint.setSocketOptions(socket)`这段代码

```
 protected boolean setSocketOptions(SocketChannel socket) {
        // Process the connection
        try {
            // 设置Socket为非阻塞模式,供Poller调用
            socket.configureBlocking(false);
            Socket sock = socket.socket();
            socketProperties.setProperties(sock);

            NioChannel channel = null;
            if (nioChannels != null) {
                channel = nioChannels.pop();
            }
            if (channel == null) {
                SocketBufferHandler bufhandler = new SocketBufferHandler(
                        socketProperties.getAppReadBufSize(),
                        socketProperties.getAppWriteBufSize(),
                        socketProperties.getDirectBuffer());
                if (isSSLEnabled()) {
                    channel = new SecureNioChannel(socket, bufhandler, selectorPool, this);
                } else {
                    channel = new NioChannel(socket, bufhandler);
                }
            } else {
                channel.setIOChannel(socket);
                channel.reset();
            }
            //注册ChannelEvent,其实是将ChannelEvent放入到队列中,然后Poller从队列中取
            getPoller0().register(channel);
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            try {
                log.error(sm.getString("endpoint.socketOptionsError"), t);
            } catch (Throwable tt) {
                ExceptionUtils.handleThrowable(tt);
            }
            // Tell to close the socket
            return false;
        }
        return true;
    }
```

其实里面重要的就是将`Acceptor`与一个`Poller`绑定起来，然后两个组件通过队列通信，每个Poller都维护着一个`SynchronizedQueue`队列，`ChannelEvent`放入到队列中，然后`Poller`从队列中取出事件进行消费。

## Poller

我们可以看到`Poller`是`NioEndpoint`的内部类，而它也是实现了`Runnable`接口，可以看到在其类中维护了一个Quene和Selector，定义如下。所以本质上`Poller`就是`Selector`。

```
private Selector selector;
private final SynchronizedQueue<PollerEvent> events = new SynchronizedQueue<>();
```

重点在其run方法中，这里删减了一些代码，只展示重要的。

```
  @Override
        public void run() {
            // Loop until destroy() is called
            while (true) {
                boolean hasEvents = false;
                try {
                    if (!close) {
                        //查看是否有连接进来,如果有就将Channel注册进Selector中
                        hasEvents = events();
                    }
                    if (close) {
                        events();
                        timeout(0, false);
                        try {
                            selector.close();
                        } catch (IOException ioe) {
                            log.error(sm.getString("endpoint.nio.selectorCloseFail"), ioe);
                        }
                        break;
                    }
                } catch (Throwable x) {
                    ExceptionUtils.handleThrowable(x);
                    log.error(sm.getString("endpoint.nio.selectorLoopError"), x);
                    continue;
                }
                if (keyCount == 0) {
                    hasEvents = (hasEvents | events());
                }
                Iterator<SelectionKey> iterator =
                    keyCount > 0 ? selector.selectedKeys().iterator() : null;
                // Walk through the collection of ready keys and dispatch
                // any active event.
                while (iterator != null && iterator.hasNext()) {
                    SelectionKey sk = iterator.next();
                    NioSocketWrapper socketWrapper = (NioSocketWrapper) sk.attachment();
                    // Attachment may be null if another thread has called
                    // cancelledKey()
                    if (socketWrapper == null) {
                        iterator.remove();
                    } else {
                        iterator.remove();
                        processKey(sk, socketWrapper);
                    }
                }

                // Process timeouts
                timeout(keyCount,hasEvents);
            }

            getStopLatch().countDown();
        }
```

其中主要的就是调用了`events()`方法，就是不断的查看队列中是否有`Pollerevent`事件，如果有的话就将其取出然后把里面的`Channel`取出来注册到该`Selector`中，然后不断轮询所有注册过的`Channel`查看是否有事件发生。

## SocketProcessor

我们知道`Poller`在轮询`Channel`有事件发生时，就会调用将此事件封装起来，然后交给线程池去执行。那么这个包装类就是`SocketProcessor`。而我们打开此类，能够看到它也实现了`Runnable`接口，用来定义线程池`Executor`中线程所执行的任务。那么这里是如何将`Channel`中的字节流转换为Tomcat需要的`ServletRequest`对象呢？其实就是调用了`Http11Processor`来进行字节流与对象的转换的。

## Executor

`Executor`其实是Tomcat定制版的线程池。我们可以看它的类的定义，可以发现它其实是扩展了Java的线程池。

```
public interface Executor extends java.util.concurrent.Executor, Lifecycle
```

在线程池中最重要的两个参数就是核心线程数和最大线程数，正常的Java线程池的执行流程是这样的。

1. 如果当前线程小于核心线程数，那么来一个任务就创建一个线程。
2. 如果当前线程大于核心线程数，那么就再来任务就将任务放入到任务队列中。所有线程抢任务。
3. 如果队列满了，那么就开始创建临时线程。
4. 如果总线程数到了最大的线程数并且队列也满了，那么就抛出异常。

但是在Tomcat自定义的线程池中是不一样的，通过重写了`execute`方法实现了自己的任务处理逻辑。

1. 如果当前线程小于核心线程数，那么来一个任务就创建一个线程。
2. 如果当前线程大于核心线程数，那么就再来任务就将任务放入到任务队列中。所有线程抢任务。
3. 如果队列满了，那么就开始创建临时线程。
4. 如果总线程数到了最大的线程数，再次获得任务队列，再尝试一次将任务加入队列中。
5. 如果此时还是满的，就抛异常。

差别就在于第四步的差别，原生线程池的处理策略是只要当前线程数大于最大线程数，那么就抛异常，而Tomcat的则是如果当前线程数大于最大线程数，就再尝试一次，如果还是满的才会抛异常。下面是定制化线程池`execute`的执行逻辑。

```
public void execute(Runnable command, long timeout, TimeUnit unit) {
    submittedCount.incrementAndGet();
    try {
        super.execute(command);
    } catch (RejectedExecutionException rx) {
        if (super.getQueue() instanceof TaskQueue) {
            //获得任务队列
            final TaskQueue queue = (TaskQueue)super.getQueue();
            try {
                if (!queue.force(command, timeout, unit)) {
                    submittedCount.decrementAndGet();
                    throw new RejectedExecutionException(sm.getString("threadPoolExecutor.queueFull"));
                }
            } catch (InterruptedException x) {
                submittedCount.decrementAndGet();
                throw new RejectedExecutionException(x);
            }
        } else {
            submittedCount.decrementAndGet();
            throw rx;
        }

    }
}
```

在代码中，我们可以看到有这么一句`submittedCount.incrementAndGet();`，为什么会有这句呢？我们可以看看这个参数的定义。简单来说这个参数就是定义了任务已经提交到了线程池中，但是还没有执行的任务个数。

```
/**
 * The number of tasks submitted but not yet finished. This includes tasks
 * in the queue and tasks that have been handed to a worker thread but the
 * latter did not start executing the task yet.
 * This number is always greater or equal to {@link #getActiveCount()}.
 */
private final AtomicInteger submittedCount = new AtomicInteger(0);
```

为什么会有这么一个参数呢？我们知道定制的队列是继承了`LinkedBlockingQueue`，而`LinkedBlockingQueue`队列默认是没有边界的。于是我们就传入了一个参数，`maxQueueSize`给构造的队列。但是在Tomcat的任务队列默认情况下是无限制的，那么这样就会出一个问题，如果当前线程达到了核心线程数，则开始向队列中添加任务，那么就会一直是添加成功的。那么就不会再创建新的线程。那么在什么情况下要新建线程呢？

> 线程池中创建新线程会有两个地方，一个是小于核心线程时，来一个任务创建一个线程。另一个是超过核心线程并且任务队列已满，则会创建临时线程。

那么如何规定任务队列是否已满呢？如果设置了队列的最大长度当然好了，但是Tomcat默认情况下是没有设置，所以默认是无限的。所以Tomcat的`TaskQueue`继承了`LinkedBlockingQueue`，重写了`offer`方法，在里面定义了什么时候返回false。

```
@Override
public boolean offer(Runnable o) {
    if (parent==null) return super.offer(o);
    //如果当前线程数等于最大线程数,此时不能创建新线程,只能添加进任务队列中
    if (parent.getPoolSize() == parent.getMaximumPoolSize()) return super.offer(o);
    //如果已提交但是未完成的任务数小于等于当前线程数,说明能处理过来,就放入队列中
    if (parent.getSubmittedCount()<=(parent.getPoolSize())) return super.offer(o);
    //到这一步说明,已提交但是未完成的任务数大于当前线程数,如果当前线程数小于最大线程数,就返回false新建线程
    if (parent.getPoolSize()<parent.getMaximumPoolSize()) return false;
    return super.offer(o);
}
```

这就是`submittedCount`的意义，目的就是为了在任务队列长度无限的情况下，让线程池有机会创建新的线程。


作者：不学无数的程序员
链接：https://juejin.im/post/5d119281518825431f1622da



# Tomcat中的类加载器

​	Tomcat为什么要实现自己的类加载器打破双亲委派模型原因之前，我们首先需要知道Java中定义的类加载器是什么，双亲委派模型是什么。

## Java中的类加载器

> 类加载器负责在程序运行时将java文件动态加载到JVM中

从Java虚拟机的角度来讲的话，存在两种不同的类加载器：

- 启动类加载器(Bootstrap ClassLoader)：这个类加载器是使用C++语言实现的，是虚拟机自身的一部分。
- 其他的类加载器：这些类加载器都由Java语言实现，独立于虚拟机外部，并且全都继承自抽象类`java.lang.ClassLoader`，其中其他类加载器大概又分为
  - ExtensionClassLoader:这个类加载器由`ExtClassLoader`实现，它负责加载`JAVA_HOME/lib/ext`目录中的所有类，或者被`java.ext.dir`系统变量所指定的路径中所有的类。
  - ApplicationClassLoader：这个类加载器是由`AppClassLoader`实现的，它负责加载用户类路径（ClassPath）上所指定的所有类，如果应用中没有自定义自己的类加载器，那么一般情况就是程序中默认的类加载器。
  - 自定义加载器：根据自己需求，自定义加载特定路径的加载器。



![img](https://user-gold-cdn.xitu.io/2019/7/5/16bc10e9f53267a6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



> 对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性

### 双亲委派模型

上图中展示的层次结构，称之为类加载器的双亲委派模型。双亲委派模型要求除了顶层的启动类加载器外，其他加载器都应该有自己的父加载器。这里的父子关系不是通过继承来实现的，而是通过设置`parent`变量来实现的。

双亲委派模型工作过程是：如果收到一个类加载的请求，本身不会先加载此类，而是会先将此请求委派给父类加载器去完成，每个层次都是如此，直到启动类加载器中，只有父类都没有加载此文件，那么子类才会尝试自己去加载。

为什么要设置双亲委派模型呢？其实是为了保证Java程序的稳定运行，例如Object类，它是存放在`rt.jar`中，无论哪一个类加载器要加载Object类，最终都会委托给顶层的BootStrapClassLoader，所以所有的类中使用的Object都是同一个类，相反如果没有双亲委派模型的话，那么随意一个类加载器都可以定义一个新的Object类，那么应用程序将会变得非常混乱。其实双亲委派模型代码非常简单。实现在ClassLoader中的loadClass方法下。

```
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // 首先,检查请求类是否被加载过
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
            // 如果没被本类类加载器加载过,先委托给父类进行加载
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                // 如果没有父类,则表明在顶层,就交给BootStrap类加载器加载
                    c = findBootstrapClassOrNull(name);
                }
                // 如果最顶层的类也找不到,那么就会抛出ClassNotFoundException异常
            } catch (ClassNotFoundException e) {

            }
            // 如果父类都没有加载过此类,子类才开始加载此类
            if (c == null) {
                c = findClass(name);
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}

protected Class<?> findClass(String name) throws ClassNotFoundException {
    throw new ClassNotFoundException(name);
}

复制代码
```

我们可以看到`findClass`方法是需要子类自己去实现的逻辑。

## Tomcat中的类加载器

下面的简图是Tomcat9版本的官方文档给出的Tomcat的类加载器的图。

```
      Bootstrap
          |
       System
          |
       Common
       /     \
  Webapp1   Webapp2 ..

复制代码
```

- Bootstrap ： 是Java的最高的加载器，用C语言实现，主要用来加载JVM启动时所需要的核心类，例如`$JAVA_HOME/jre/lib/ext`路径下的类。
- System： 会加载`CLASSPATH`系统变量所定义路径的所有的类。
- Common：会加载Tomcat路径下的lib文件下的所有类。
- Webapp1、Webapp2……： 会加载webapp路径下项目中的所有的类。一个项目对应一个WebappClassLoader，这样就实现了应用之间类的隔离了。

这3个部分，在上面的Java双亲委派模型图中都有体现。不过可以看到ExtClassLoader没有画出来，可以理解为是跟bootstrap合并了，都是去`JAVA_HOME/jre/lib`下面加载类。 那么Tomcat为什么要自定义类加载器呢？

- 隔离不同应用：部署在同一个Tomcat中的不同应用A和B，例如A用了Spring2.5。B用了Spring3.5，那么这两个应用如果使用的是同一个类加载器，那么Web应用就会因为jar包覆盖而无法启动。
- 灵活性：Web应用之间的类加载器相互独立，那么就可以根据修改不同的文件重建不同的类加载器替换原来的。从而不影响其他应用。
- 性能：如果在一个Tomcat部署多个应用，多个应用中都有相同的类库依赖。那么可以把这相同的类库让Common类加载器进行加载。

Tomcat自定义了WebAppClassLoader类加载器。打破了双亲委派的机制，**即如果收到类加载的请求，会尝试自己去加载，如果找不到再交给父加载器去加载**，目的就是为了优先加载Web应用自己定义的类。我们知道ClassLoader默认的loadClass方法是以双亲委派的模型进行加载类的，那么Tomcat既然要打破这个规则，就要重写loadClass方法，我们可以看WebAppClassLoader类中重写的loadClass方法。

```java
@Override
public Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {

    synchronized (getClassLoadingLock(name)) {
        Class<?> clazz = null;
        // 1. 从本地缓存中查找是否加载过此类
        clazz = findLoadedClass0(name);
        if (clazz != null) {
            if (log.isDebugEnabled())
                log.debug("  Returning class from cache");
            if (resolve)
                resolveClass(clazz);
            return clazz;
        }

        // 2. 从AppClassLoader中查找是否加载过此类
        clazz = findLoadedClass(name);
        if (clazz != null) {
            if (log.isDebugEnabled())
                log.debug("  Returning class from cache");
            if (resolve)
                resolveClass(clazz);
            return clazz;
        }

        String resourceName = binaryNameToPath(name, false);
        // 3. 尝试用ExtClassLoader 类加载器加载类,防止Web应用覆盖JRE的核心类
        ClassLoader javaseLoader = getJavaseClassLoader();
        boolean tryLoadingFromJavaseLoader;
        try {
            URL url;
            if (securityManager != null) {
                PrivilegedAction<URL> dp = new PrivilegedJavaseGetResource(resourceName);
                url = AccessController.doPrivileged(dp);
            } else {
                url = javaseLoader.getResource(resourceName);
            }
            tryLoadingFromJavaseLoader = (url != null);
        } catch (Throwable t) {
            tryLoadingFromJavaseLoader = true;
        }

        boolean delegateLoad = delegate || filter(name, true);

        // 4. 判断是否设置了delegate属性,如果设置为true那么就按照双亲委派机制加载类
        if (delegateLoad) {
            if (log.isDebugEnabled())
                log.debug("  Delegating to parent classloader1 " + parent);
            try {
                clazz = Class.forName(name, false, parent);
                if (clazz != null) {
                    if (log.isDebugEnabled())
                        log.debug("  Loading class from parent");
                    if (resolve)
                        resolveClass(clazz);
                    return clazz;
                }
            } catch (ClassNotFoundException e) {
                // Ignore
            }
        }

        // 5. 默认是设置delegate是false的,那么就会先用WebAppClassLoader进行加载
        if (log.isDebugEnabled())
            log.debug("  Searching local repositories");
        try {
            clazz = findClass(name);
            if (clazz != null) {
                if (log.isDebugEnabled())
                    log.debug("  Loading class from local repository");
                if (resolve)
                    resolveClass(clazz);
                return clazz;
            }
        } catch (ClassNotFoundException e) {
            // Ignore
        }

        // 6. 如果此时在WebAppClassLoader没找到类,那么就委托给AppClassLoader去加载
        if (!delegateLoad) {
            if (log.isDebugEnabled())
                log.debug("  Delegating to parent classloader at end: " + parent);
            try {
                clazz = Class.forName(name, false, parent);
                if (clazz != null) {
                    if (log.isDebugEnabled())
                        log.debug("  Loading class from parent");
                    if (resolve)
                        resolveClass(clazz);
                    return clazz;
                }
            } catch (ClassNotFoundException e) {
                // Ignore
            }
        }
    }
    throw new ClassNotFoundException(name);
}

```

最后借用Tomcat官网上的话总结：

Web应用默认的类加载顺序是（打破了双亲委派规则）：

1. 先从JVM的BootStrapClassLoader中加载。
2. 加载Web应用下`/WEB-INF/classes`中的类。
3. 加载Web应用下`/WEB-INF/lib/*.jap`中的jar包中的类。
4. 加载上面定义的System路径下面的类。
5. 加载上面定义的Common路径下面的类。

如果在配置文件中配置了`<Loader delegate="true"/>`，那么就是遵循双亲委派规则，加载顺序如下：

1. 先从JVM的BootStrapClassLoader中加载。
2. 加载上面定义的System路径下面的类。
3. 加载上面定义的Common路径下面的类。
4. 加载Web应用下`/WEB-INF/classes`中的类。
5. 加载Web应用下`/WEB-INF/lib/*.jap`中的jar包中的类。



## 参考文章

- [tomcat.apache.org/tomcat-9.0-…](http://tomcat.apache.org/tomcat-9.0-doc/class-loader-howto.html)
- [深入理解Java虚拟机]()
- [深入拆解Tomcat]()
- [kyfxbl.iteye.com/blog/170723…](https://kyfxbl.iteye.com/blog/1707237)



# Tomcat如何做到一键式启停的

在没有SpringBoot内嵌有Tomcat之前，我们都是将项目打为War包放在Tomcat的webapp目录下面，然后如果是Linux系统，运行命令`start.sh`、如果是Windows系统，运行命令`start.bat`以后就能启动起来并访问到页面。如果是想要停止运行只需要运行`shutdown.sh`或者`shutdown.bat`就能将程序停止起来，那么Tomcat是如何做到只需要一个命令就将所有容器启动起来呢？

## 脚本分析

`start.sh`和`start.bat`里面的内容相同，所以这里就主要分析`start.sh`的内容了。

```sh
os400=false
case "`uname`" in
OS400*) os400=true;;
esac

# resolve links - $0 may be a softlink
# PRG是脚本路径,如果当前脚本文件为软连接,则会解析出PRG真正文件所在的路径
PRG="$0"

while [ -h "$PRG" ] ; do # 判断是否为软连接
  ls=`ls -ld "$PRG"`   # 如果是软连接,输出中含有lin -> source的字符串
  link=`expr "$ls" : '.*-> \(.*\)$'` # 模式匹配出源文件的路径
  if expr "$link" : '/.*' > /dev/null; then # 正则匹配 /.* 这里expr会输出匹配个数,如果不为0,则说明$link包含目录
    PRG="$link"
  else
    PRG=`dirname "$PRG"`/"$link" # 当不包含目录,说明软连接和源文件在同一目录
  fi
done

# 获取脚本目录路径
PRGDIR=`dirname "$PRG"`
EXECUTABLE=catalina.sh

# Check that target executable exists
if $os400; then
  # -x will Only work on the os400 if the files are:
  # 1. owned by the user
  # 2. owned by the PRIMARY group of the user
  # this will not work if the user belongs in secondary groups
  eval
else
  if [ ! -x "$PRGDIR"/"$EXECUTABLE" ]; then
    echo "Cannot find $PRGDIR/$EXECUTABLE"
    echo "The file is absent or does not have execute permission"
    echo "This file is needed to run this program"
    exit 1
  fi
fi

# 执行catalina.sh的start命令
exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```

其实上面简单来说就做了两件事

1. 拿到脚本的真正路径
2. 执行`catalina.sh`的`start`命令

而`shutdown.sh`和`start.sh`命令一样，只不过后面是执行`catalina.sh`的`stop`命令

### catalina.sh脚本

脚本中重要的步骤有以下几个

1. 设置两个重要的环境变量，`CATALINA_HOME`、`CATALINA_BASE`

   ```
   PRGDIR=`dirname "$PRG"`
   [ -z "$CATALINA_HOME" ] && CATALINA_HOME=`cd "$PRGDIR/.." >/dev/null; pwd`
   [ -z "$CATALINA_BASE" ] && CATALINA_BASE="$CATALINA_HOME"
   ```

2. 设置`CLASSPATH`变量，这里注意，默认是没有setenv.sh文件的，可以自己新建一个并添加参数

   ```
   CLASSPATH=
   if [ -r "$CATALINA_BASE/bin/setenv.sh" ]; then
     . "$CATALINA_BASE/bin/setenv.sh"
   elif [ -r "$CATALINA_HOME/bin/setenv.sh" ]; then
     . "$CATALINA_HOME/bin/setenv.sh"
   fi
   ```

3. 将`bootstrap.jar`作为`CLASSPATH`变量传进去

   ```
   if [ ! -z "$CLASSPATH" ] ; then
     CLASSPATH="$CLASSPATH":
   fi
   CLASSPATH="$CLASSPATH""$CATALINA_HOME"/bin/bootstrap.jar
   
   if [ -z "$CATALINA_OUT" ] ; then
     CATALINA_OUT="$CATALINA_BASE"/logs/catalina.out
   fi
   
   复制代码
   ```

4. 执行脚本参数，执行`bootstrap.jar`中的`Bootstrap`类中`main`方法，并传入参数`start`

   ```
       shift
       eval exec "\"$_RUNJAVA\"" "\"$LOGGING_CONFIG\"" $LOGGING_MANAGER $JAVA_OPTS $CATALINA_OPTS \
         -D$ENDORSED_PROP="\"$JAVA_ENDORSED_DIRS\"" \
         -classpath "\"$CLASSPATH\"" \
         -Djava.security.manager \
         -Djava.security.policy=="\"$CATALINA_BASE/conf/catalina.policy\"" \
         -Dcatalina.base="\"$CATALINA_BASE\"" \
         -Dcatalina.home="\"$CATALINA_HOME\"" \
         -Djava.io.tmpdir="\"$CATALINA_TMPDIR\"" \
         org.apache.catalina.startup.Bootstrap "$@" start
   
   复制代码
   ```

在上面脚本中我们可以看出最后执行的都是从`Bootstrap`的`main`方法作为入口的，所以我们打开Tomcat源码进去`Bootstrap`类中看它到底做了什么。

## 启动类分析

作为Tomcat的入口类，我们先看看`Bootstrap`中做了什么。这里只贴出main方法中重要的代码。

```
//初始化类加载器并且将Catalina文件加载进内存中
bootstrap.init();
String command = "start";
if (args.length > 0) {
    command = args[args.length - 1];
}

if (command.equals("startd")) {
    args[args.length - 1] = "start";
    //调用Catalina.java的load方法
    daemon.load(args);
    //调用Catalina.java的start
    daemon.start();
} else if (command.equals("stopd")) {
    args[args.length - 1] = "stop";
    //调用Catalina.java的stop
    daemon.stop();
} else if (command.equals("start")) {
    daemon.setAwait(true);
    daemon.load(args);
    daemon.start();
    if (null == daemon.getServer()) {
        System.exit(1);
    }
} else if (command.equals("stop")) {
    daemon.stopServer(args);
} else if (command.equals("configtest")) {
    daemon.load(args);
    if (null == daemon.getServer()) {
        System.exit(1);
    }
    System.exit(0);
} else {
    log.warn("Bootstrap: command \"" + command + "\" does not exist.");
}
复制代码
```

这里是根据脚本中传入的不同命令，调用`Catalina`不同的方法。由于我们主要分析的Tomcat如何做到一键式启停的，所以我们主要分析`Catalina`的`start`方法。

在`Catalina`的`satrt`方法中我们看到了这一句

```
getServer().start();

复制代码
```

随后经过Debug都是经过了`Lifecycle`的`start`方法，我们把`Lifecycle`的方法列出来

```
public interface Lifecycle {

    public void addLifecycleListener(LifecycleListener listener);

    public LifecycleListener[] findLifecycleListeners();

    public void removeLifecycleListener(LifecycleListener listener);

    public void init() throws LifecycleException;

    public void start() throws LifecycleException;

    public void stop() throws LifecycleException;

    public void destroy() throws LifecycleException;

    public LifecycleState getState();

    public String getStateName();

    public interface SingleUse {
    }
}

复制代码
```

然后再看它的实现类，我们发现我们前面所讲的整体架构中的组件都实现了此类。而在它的子类`LifecycleBase`实现了`start`、`init`、`stop`等方法，并且里面都相应调用了`startInternal`、`initInternal`、`stopInternal`方法，这里我们如果对于设计模式了解的话，应该会想到这里运用了[模板设计模式](https://juejin.im/post/5cc66d88e51d456e2d69a7fa)，抽象出所有子类的公有的代码，然后重新定义一个内部抽象方法，其子类实现自己的定制化的操作。

在`Server.xml`中我们发现第一个层级也是`Server`，然后`Catalina`的`satrt`方法中第一个启动的也是`Server`。

![image-20200421113221658](images/image-20200421113221658.png)



上面表示了Tomcat所有模块的层级结构，只要是带有层级的结构，我们应该能够立马想到[组合设计模式](https://juejin.im/post/5b9877c0f265da0a9624b7ff)，从这个层级结构中我们能够得到模块之间的关系，**有大有小，有内有外**。

- 有大有小：大组件管理小组件，例如Server管理Service，Service管理连接器和容器
- 有内有外：连接器控制对外的连接，而外层组件调用内层组件完成业务功能。即请求处理的过程是由外层组件驱动的。

那么根据上面的两条，我们知道，**有小才有大，有内才有外**。这也就是整个层级的加载顺序，先加载小组件再加载大组件，先加载内层组件再加载外层组件。此时我们应该就明白了Tomcat是如何做到一键式启停的了。通过层级结构，加载的优先级。层层迭代进行启动。而停止和启动差不多。也是层层迭代进行停止。


作者：不学无数的程序员
链接：https://juejin.im/post/5d1335996fb9a07ebb05437a

- [expr命令实例](http://www.wallcopper.com/linux/567.html)
- [Tomcat运行脚本及启动过程分析--catalina.sh](https://blog.csdn.net/qq_21508059/article/details/82713797)



# Tomcat在SpringBoot中是如何启动的

我们知道SpringBoot给我们带来了一个全新的开发体验，我们可以直接把web程序达成jar包，直接启动，这就得益于SpringBoot内置了容器，可以直接启动，本文将以Tomcat为例，来看看SpringBoot是如何启动Tomcat的，同时也将展开学习下Tomcat的源码，了解Tomcat的设计。

## 从 Main 方法说起

用过SpringBoot的人都知道，首先要写一个main方法来启动

```java
@SpringBootApplication
public class TomcatdebugApplication {

    public static void main(String[] args) {
        SpringApplication.run(TomcatdebugApplication.class, args);
    }

}
```

我们直接点击run方法的源码，跟踪下来，发下最终 的`run`方法是调用`ConfigurableApplicationContext`方法，源码如下：

```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		//设置系统属性『java.awt.headless』，为true则启用headless模式支持
		configureHeadlessProperty();
		//通过*SpringFactoriesLoader*检索*META-INF/spring.factories*，
       //找到声明的所有SpringApplicationRunListener的实现类并将其实例化，
       //之后逐个调用其started()方法，广播SpringBoot要开始执行了
		SpringApplicationRunListeners listeners = getRunListeners(args);
		//发布应用开始启动事件
		listeners.starting();
		try {
		//初始化参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			//创建并配置当前SpringBoot应用将要使用的Environment（包括配置要使用的PropertySource以及Profile）,
        //并遍历调用所有的SpringApplicationRunListener的environmentPrepared()方法，广播Environment准备完毕。
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			//打印banner
			Banner printedBanner = printBanner(environment);
			//创建应用上下文
			context = createApplicationContext();
			//通过*SpringFactoriesLoader*检索*META-INF/spring.factories*，获取并实例化异常分析器
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			//为ApplicationContext加载environment，之后逐个执行ApplicationContextInitializer的initialize()方法来进一步封装ApplicationContext，
        //并调用所有的SpringApplicationRunListener的contextPrepared()方法，【EventPublishingRunListener只提供了一个空的contextPrepared()方法】，
        //之后初始化IoC容器，并调用SpringApplicationRunListener的contextLoaded()方法，广播ApplicationContext的IoC加载完成，
        //这里就包括通过**@EnableAutoConfiguration**导入的各种自动配置类。
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			//刷新上下文
			refreshContext(context);
			//再一次刷新上下文,其实是空方法，可能是为了后续扩展。
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			//发布应用已经启动的事件
			listeners.started(context);
			//遍历所有注册的ApplicationRunner和CommandLineRunner，并执行其run()方法。
        //我们可以实现自己的ApplicationRunner或者CommandLineRunner，来对SpringBoot的启动过程进行扩展。
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
		//应用已经启动完成的监听事件
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

其实这个方法我们可以简单的总结下步骤为

> 1. 配置属性
> 2. 获取监听器，发布应用开始启动事件
> 3. 初始化输入参数
> 4. 配置环境，输出banner
> 5. 创建上下文
> 6. 预处理上下文
> 7. 刷新上下文
> 8. 再刷新上下文
> 9. 发布应用已经启动事件
> 10. 发布应用启动完成事件

其实上面这段代码，如果只要分析tomcat内容的话，只需要关注两个内容即可，上下文是如何创建的，上下文是如何刷新的，分别对应的方法就是`createApplicationContext()` 和`refreshContext(context)`，接下来我们来看看这两个方法做了什么。

```java
protected ConfigurableApplicationContext createApplicationContext() {
		Class<?> contextClass = this.applicationContextClass;
		if (contextClass == null) {
			try {
				switch (this.webApplicationType) {
				case SERVLET:
					contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
					break;
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, " + "please specify an ApplicationContextClass",
						ex);
			}
		}
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}
复制代码
```

这里就是根据我们的`webApplicationType` 来判断创建哪种类型的Servlet,代码中分别对应着Web类型(SERVLET),响应式Web类型（REACTIVE),非Web类型（default),我们建立的是Web类型，所以肯定实例化 `DEFAULT_SERVLET_WEB_CONTEXT_CLASS`指定的类，也就是`AnnotationConfigServletWebServerApplicationContext`类，我们来用图来说明下这个类的关系

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzEyLzE2YzgzMWIzOTcyM2EzNzE)



通过这个类图我们可以知道，这个类继承的是`ServletWebServerApplicationContext`,这就是我们真正的主角，而这个类最终是继承了`AbstractApplicationContext`，了解完创建上下文的情况后，我们再来看看刷新上下文，相关代码如下：

```java
//类：SpringApplication.java
private void refreshContext(ConfigurableApplicationContext context) {
    //直接调用刷新方法
		refresh(context);
		if (this.registerShutdownHook) {
			try {
				context.registerShutdownHook();
			}
			catch (AccessControlException ex) {
				// Not allowed in some environments.
			}
		}
	}
//类：SpringApplication.java

protected void refresh(ApplicationContext applicationContext) {
		Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
		((AbstractApplicationContext) applicationContext).refresh();
	}
```

这里还是直接传递调用本类的`refresh(context)`方法，最后是强转成父类`AbstractApplicationContext`调用其`refresh()`方法,该代码如下：

```java
// 类：AbstractApplicationContext	
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.这里的意思就是调用各个子类的onRefresh()
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

这里我们看到`onRefresh()`方法是调用其子类的实现，根据我们上文的分析，我们这里的子类是`ServletWebServerApplicationContext`。

```java
//类：ServletWebServerApplicationContext
protected void onRefresh() {
		super.onRefresh();
		try {
			createWebServer();
		}
		catch (Throwable ex) {
			throw new ApplicationContextException("Unable to start web server", ex);
		}
	}
	
private void createWebServer() {
		WebServer webServer = this.webServer;
		ServletContext servletContext = getServletContext();
		if (webServer == null && servletContext == null) {
			ServletWebServerFactory factory = getWebServerFactory();
			this.webServer = factory.getWebServer(getSelfInitializer());
		}
		else if (servletContext != null) {
			try {
				getSelfInitializer().onStartup(servletContext);
			}
			catch (ServletException ex) {
				throw new ApplicationContextException("Cannot initialize servlet context", ex);
			}
		}
		initPropertySources();
	}

```

到这里，其实庐山真面目已经出来了，`createWebServer()`就是启动web服务，但是还没有真正启动Tomcat，既然`webServer`是通过`ServletWebServerFactory`来获取的，我们就来看看这个工厂的真面目。

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzEyLzE2YzgzMWJiZjUyMjYzYTg)



## 走进Tomcat内部

根据上图我们发现，工厂类是一个接口，各个具体服务的实现是由各个子类来实现的，所以我们就去看看`TomcatServletWebServerFactory.getWebServer()`的实现。

```java
	@Override
	public WebServer getWebServer(ServletContextInitializer... initializers) {
		Tomcat tomcat = new Tomcat();
		File baseDir = (this.baseDirectory != null) ? this.baseDirectory : createTempDir("tomcat");
		tomcat.setBaseDir(baseDir.getAbsolutePath());
		Connector connector = new Connector(this.protocol);
		tomcat.getService().addConnector(connector);
		customizeConnector(connector);
		tomcat.setConnector(connector);
		tomcat.getHost().setAutoDeploy(false);
		configureEngine(tomcat.getEngine());
		for (Connector additionalConnector : this.additionalTomcatConnectors) {
			tomcat.getService().addConnector(additionalConnector);
		}
		prepareContext(tomcat.getHost(), initializers);
		return getTomcatWebServer(tomcat);
	}
```

根据上面的代码，我们发现其主要做了两件事情，第一件事就是把Connnctor(我们称之为连接器)对象添加到Tomcat中，第二件事就是`configureEngine`,这连接器我们勉强能理解（不理解后面会述说），那这个`Engine`是什么呢？我们查看`tomcat.getEngine()`的源码：

```java
    public Engine getEngine() {
        Service service = getServer().findServices()[0];
        if (service.getContainer() != null) {
            return service.getContainer();
        }
        Engine engine = new StandardEngine();
        engine.setName( "Tomcat" );
        engine.setDefaultHost(hostname);
        engine.setRealm(createDefaultRealm());
        service.setContainer(engine);
        return engine;
    }
```

根据上面的源码，我们发现，原来这个Engine是容器，我们继续跟踪源码，找到`Container`接口

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzEyLzE2YzgzMWMwNzdlZmU3MmE)

上图中，我们看到了4个子接口，分别是Engine,Host,Context,Wrapper。我们从继承关系上可以知道他们都是容器，那么他们到底有啥区别呢？我看看他们的注释是怎么说的。

```java
 /**
 If used, an Engine is always the top level Container in a Catalina
 * hierarchy. Therefore, the implementation's <code>setParent()</code> method
 * should throw <code>IllegalArgumentException</code>.
 *
 * @author Craig R. McClanahan
 */
public interface Engine extends Container {
    //省略代码
}
/**
 * <p>
 * The parent Container attached to a Host is generally an Engine, but may
 * be some other implementation, or may be omitted if it is not necessary.
 * <p>
 * The child containers attached to a Host are generally implementations
 * of Context (representing an individual servlet context).
 *
 * @author Craig R. McClanahan
 */
public interface Host extends Container {
//省略代码
    
}
/*** <p>
 * The parent Container attached to a Context is generally a Host, but may
 * be some other implementation, or may be omitted if it is not necessary.
 * <p>
 * The child containers attached to a Context are generally implementations
 * of Wrapper (representing individual servlet definitions).
 * <p>
 *
 * @author Craig R. McClanahan
 */
public interface Context extends Container, ContextBind {
    //省略代码
}
/**<p>
 * The parent Container attached to a Wrapper will generally be an
 * implementation of Context, representing the servlet context (and
 * therefore the web application) within which this servlet executes.
 * <p>
 * Child Containers are not allowed on Wrapper implementations, so the
 * <code>addChild()</code> method should throw an
 * <code>IllegalArgumentException</code>.
 *
 * @author Craig R. McClanahan
 */
public interface Wrapper extends Container {

    //省略代码
}

```

上面的注释翻译过来就是，`Engine`是最高级别的容器，其子容器是`Host`,`Host`的子容器是`Context`,`Wrapper`是`Context`的子容器，所以这4个容器的关系就是父子关系，也就是`Engine`>`Host`>`Context`>`Wrapper`。 我们再看看`Tomcat`类的源码:

```java
//部分源码，其余部分省略。
public class Tomcat {
//设置连接器
     public void setConnector(Connector connector) {
        Service service = getService();
        boolean found = false;
        for (Connector serviceConnector : service.findConnectors()) {
            if (connector == serviceConnector) {
                found = true;
            }
        }
        if (!found) {
            service.addConnector(connector);
        }
    }
    //获取service
       public Service getService() {
        return getServer().findServices()[0];
    }
    //设置Host容器
     public void setHost(Host host) {
        Engine engine = getEngine();
        boolean found = false;
        for (Container engineHost : engine.findChildren()) {
            if (engineHost == host) {
                found = true;
            }
        }
        if (!found) {
            engine.addChild(host);
        }
    }
    //获取Engine容器
     public Engine getEngine() {
        Service service = getServer().findServices()[0];
        if (service.getContainer() != null) {
            return service.getContainer();
        }
        Engine engine = new StandardEngine();
        engine.setName( "Tomcat" );
        engine.setDefaultHost(hostname);
        engine.setRealm(createDefaultRealm());
        service.setContainer(engine);
        return engine;
    }
    //获取server
       public Server getServer() {

        if (server != null) {
            return server;
        }

        System.setProperty("catalina.useNaming", "false");

        server = new StandardServer();

        initBaseDir();

        // Set configuration source
        ConfigFileLoader.setSource(new CatalinaBaseConfigurationSource(new File(basedir), null));

        server.setPort( -1 );

        Service service = new StandardService();
        service.setName("Tomcat");
        server.addService(service);
        return server;
    }
    
    //添加Context容器
      public Context addContext(Host host, String contextPath, String contextName,
            String dir) {
        silence(host, contextName);
        Context ctx = createContext(host, contextPath);
        ctx.setName(contextName);
        ctx.setPath(contextPath);
        ctx.setDocBase(dir);
        ctx.addLifecycleListener(new FixContextListener());

        if (host == null) {
            getHost().addChild(ctx);
        } else {
            host.addChild(ctx);
        }
        
    //添加Wrapper容器
         public static Wrapper addServlet(Context ctx,
                                      String servletName,
                                      Servlet servlet) {
        // will do class for name and set init params
        Wrapper sw = new ExistingStandardWrapper(servlet);
        sw.setName(servletName);
        ctx.addChild(sw);

        return sw;
    }
    
}

```

阅读`Tomcat`的`getServer()`我们可以知道，`Tomcat`的最顶层是`Server`,Server就是`Tomcat`的实例，一个`Tomcat`一个`Server`;通过`getEngine()`我们可以了解到Server下面是Service，而且是多个，一个Service代表我们部署的一个应用，而且我们还可以知道，`Engine`容器，一个`service`只有一个；根据父子关系，我们看`setHost()`源码可以知道，`host`容器有多个；同理，我们发现`addContext()`源码下，`Context`也是多个；`addServlet()`表明`Wrapper`容器也是多个，而且这段代码也暗示了，其实`Wrapper`和`Servlet`是一层意思。另外我们根据`setConnector`源码可以知道，连接器(`Connector`)是设置在`service`下的，而且是可以设置多个连接器(`Connector`)。

根据上面分析，我们可以小结下： Tomcat主要包含了2个核心组件，连接器(Connector)和容器(Container),用图表示如下：

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzEyLzE2YzgzMWM3OTQ5MjUwYzk)

一个`Tomcat`是一个`Server`,一个`Server`下有多个`service`，也就是我们部署的多个应用，一个应用下有多个连接器(`Connector`)和一个容器（`Container`）,容器下有多个子容器，关系用图表示如下：



![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOS84LzEyLzE2YzgzMWNkMTVhMzA4ZGQ)

`Engine`下有多个`Host`子容器，`Host`下有多个`Context`子容器，`Context`下有多个`Wrapper`子容器。

## 总结

SpringBoot的启动是通过`new SpringApplication()`实例来启动的，启动过程主要做如下几件事情：

> 1. 配置属性
> 2. 获取监听器，发布应用开始启动事件
> 3. 初始化输入参数
> 4. 配置环境，输出banner
> 5. 创建上下文
> 6. 预处理上下文
> 7. 刷新上下文
> 8. 再刷新上下文
> 9. 发布应用已经启动事件
> 10. 发布应用启动完成事件

而启动Tomcat就是在第7步中“刷新上下文”；Tomcat的启动主要是初始化2个核心组件，连接器(Connector)和容器（Container），一个Tomcat实例就是一个Server，一个Server包含多个Service，也就是多个应用程序，每个Service包含多个连接器（Connetor）和一个容器（Container),而容器下又有多个子容器，按照父子关系分别为：Engine,Host,Context,Wrapper，其中除了Engine外，其余的容器都是可以有多个。
