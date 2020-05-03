# SpringIOC

​	IOC和DI的概念
​		IoC(思想，设计模式)主要的实现方式有两种：依赖查找，依赖注入。
​		依赖注入是一种更可取的方式(实现的方式)
​	使用IOC的好处
​		不用自己组装，拿来就用。
​		享受单例的好处，效率高，不浪费空间
​		便于单元测试，方便切换mock组件
​		便于进行AOP操作，对于使用者是透明的
​		统一配置，便于修改
​	IOC容器
​		Spring容器(Bean工厂)可分为两种：
​			BeanFactory，这是最基础、面向Spring的
​			ApplicationContext，这是在BeanFactory基础之上，面向使用Spring框架的开发者。提供了一系列的功能！
​		Bean生命周期：
​			BeanDefinitionReader读取Resource所指向的配置文件资源，然后解析配置文件。配置文件中每一个<bean>解析成一个BeanDefinition对象，并保存到BeanDefinitionRegistry中；
​			容器扫描BeanDefinitionRegistry中的BeanDefinition；调用InstantiationStrategy进行Bean实例化的工作；使用BeanWrapper完成Bean属性的设置工作；
​		ApplicationContext和BeanFactory区别
​			ApplicationContext会利用Java反射机制自动识别出配置文件中定义的BeanPostProcessor、 InstantiationAwareBeanPostProcesso 和BeanFactoryPostProcessor后置器，并自动将它们注册到应用上下文中。而BeanFactory需要在代码中通过手工调用addBeanPostProcessor()方法进行注册
​			ApplicationContext在初始化应用上下文的时候就实例化所有单实例的Bean。而BeanFactory在初始化容器的时候并未实例化Bean，直到第一次访问某个Bean时才实例化目标Bean。
​			单例Bean缓存池：Spring 在DefaultSingletonBeanRegistry类中提供了一个用于缓存单实例 Bean 的缓存器，它是一个用HashMap实现的缓存器，单实例的Bean以beanName为键保存在这个HashMap中。
​	IOC容器装配Bean
​		装配Bean方式
​			XML配置
​			Java配置
​			注解
​			基于Groovy DSL配置(这种很少见)
​		依赖注入方式
​			属性注入-->通过setter()方法注入
​			构造函数注入
​			工厂注入
​		Bean对象之间关系
​			依赖-->挺少用的(使用depends-on就是依赖关系了-->前置依赖【依赖的Bean需要初始化之后，当前Bean才会初始化】)
​			继承-->可能会用到(指定abstract和parent来实现继承关系)
​			引用-->最常见(使用ref就是引用关系了)
​		Bean的作用域
​			单例Singleton
​			多例prototype
​			与Web应用环境相关的Bean作用域
​				reqeust
​				session
​				globalSession
​			使用与Web应用环境相关的Bean作用域需要手动设置代理
​		处理自动装配的歧义性
​			使用@Primary注解设置为首选的注入Bean
​			使用@Qualifier注解设置特定名称的Bean来限定注入！
​		引用属性文件以及Bean属性
​			引用配置文件的数据使用的是${}
​			引用Bean的属性使用的是#{}
​		短小不常见的知识点
​			组合配置文件：XML与JavaConfig互相引用
​			方法替换：使用某个Bean的方法替换成另一个Bean的方法
​			属性编辑器：Spring可以对基本类型做转换就归结于属性编辑器的功劳！
​			国际化：使用不同语言(英语、中文)的操作系统去显式不同的语言
​			profile与条件化的Bean：满足了某个条件才初始化Bean，这可以方便切换生产环境和开发环境~
​			容器事件：类似于我们的Servlet的监听器，只不过它是在Spring中实现了~

# 彻底理解SpringIOC、DI

## 前言

你可能会有如下问题：

1、想看Spring源码，但是不知道应当如何入手去看，对整个Bean的流程没有概念，碰到相关问题也没有头绪如何下手

2、看过几遍源码，没办法彻底理解，没什么感觉，没过一阵子又忘了

本文将结合实际问题，由问题引出源码，并在解释时会尽量以图表的形式让你一步一步彻底理解Spring Bean的IOC、DI、生命周期、作用域等。

## 先看一个循环依赖问题

## 现象

循环依赖其实就是循环引用，也就是两个或则两个以上的bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C又依赖于A。如下图：

![img](https://user-gold-cdn.xitu.io/2018/11/12/16707fe6df69cab7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

如何理解“依赖”呢，在Spring中有：

- 构造器循环依赖
- field属性注入循环依赖

直接上代码：

### 构造器循环依赖







```
@Service
public class A {  
    public A(B b) {  }
}
复制代码
```







```
@Service
public class B {  
    public B(C c) {  
    }
}
复制代码
```









```
@Service
public class C {  
    public C(A a) {  }
}复制代码
```



结果：项目启动失败，发现了一个cycle。

![img](https://user-gold-cdn.xitu.io/2018/11/12/16708043be99065a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

###  2.field属性注入循环依赖







```
@Service
public class A1 {  
    @Autowired  
    private B1 b1;
}复制代码
```









```
@Service
public class B1 {  
    @Autowired  
    public C1 c1;
}复制代码
```









```
@Service
public class C1 {  
    @Autowired  public A1 a1;
}复制代码
```



结果：项目启动成功

![img](https://user-gold-cdn.xitu.io/2018/11/12/1670808ac258af58?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

###  3.field属性注入循环依赖（prototype）

```
@Service
@Scope("prototype")
public class A1 {  
    @Autowired  
    private B1 b1;
}复制代码
```









```
@Service
@Scope("prototype")
public class B1 {  
    @Autowired  
    public C1 c1;
}复制代码
```









```
@Service
@Scope("prototype")
public class C1 {  
    @Autowired  public A1 a1;
}复制代码
```

结果：项目启动失败，发现了一个cycle。

![img](https://user-gold-cdn.xitu.io/2018/11/12/16708043be99065a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

现象总结：同样对于循环依赖的场景，构造器注入和prototype类型的属性注入都会初始化Bean失败。因为@Service默认是单例的，所以单例的属性注入是可以成功的。

## 分析原因

分析原因也就是在发现SpringIOC的过程，如果对源码不感兴趣可以关注每段源码分析之后的**总结和循环依赖问题的分析**即可。

### SpringBean的加载流程（源码分析）

简单一段代码作为入口

```
ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
ac.getBean(XXX.class);复制代码
```

ClassPathXmlApplicationContext是一个加载XML配置文件的类，与之相对的还有AnnotationConfigWebApplicationContext，这两个类大差不差的，只是ClassPathXmlApplicationContext的Resource是XML文件而AnnotationConfigWebApplicationContext是Scan注解获得的。

看到第二行就已经可以直接获取bean的实例了，所以第一行构造方法时，就已经完成了对所有bean的加载。

ClassPathXmlApplicationContext举例，他里面储存的东西如下：

| **对象名**                  | **类 型**                      | **作 用**                                                    | **归属类**                                  |
| --------------------------- | ------------------------------ | ------------------------------------------------------------ | ------------------------------------------- |
| configResources             | Resource[]                     | 配置文件资源对象数组                                         | ClassPathXmlApplicationContext              |
| configLocations             | String[]                       | 配置文件字符串数组，存储配置文件路径                         | AbstractRefreshableConfigApplicationContext |
| beanFactory                 | DefaultListableBeanFactory     | 上下文使用的Bean工厂                                         | AbstractRefreshableApplicationContext       |
| beanFactoryMonitor          | Object                         | Bean工厂使用的同步监视器                                     | AbstractRefreshableApplicationContext       |
| id                          | String                         | 上下文使用的唯一Id，标识此ApplicationContext                 | AbstractApplicationContext                  |
| parent                      | ApplicationContext             | 父级ApplicationContext                                       | AbstractApplicationContext                  |
| beanFactoryPostProcessors   | List<BeanFactoryPostProcessor> | 存储BeanFactoryPostProcessor接口，Spring提供的一个扩展点     | AbstractApplicationContext                  |
| startupShutdownMonitor      | Object                         | refresh方法和destory方法公用的一个监视器，避免两个方法同时执行 | AbstractApplicationContext                  |
| shutdownHook                | Thread                         | Spring提供的一个钩子，JVM停止执行时会运行Thread里面的方法    | AbstractApplicationContext                  |
| resourcePatternResolver     | ResourcePatternResolver        | 上下文使用的资源格式解析器                                   | AbstractApplicationContext                  |
| lifecycleProcessor          | LifecycleProcessor             | 用于管理Bean生命周期的生命周期处理器接口                     | AbstractApplicationContext                  |
| messageSource               | MessageSource                  | 用于实现国际化的一个接口                                     | AbstractApplicationContext                  |
| applicationEventMulticaster | ApplicationEventMulticaster    | Spring提供的事件管理机制中的事件多播器接口                   | AbstractApplicationContext                  |
| applicationListeners        | Set<ApplicationListener>       | Spring提供的事件管理机制中的应用监听器                       | AbstractApplicationContext                  |

构造方法如下：

![img](https://user-gold-cdn.xitu.io/2018/11/13/1670d0adae936cb5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

接下来大概看看refresh方法：

![img](https://user-gold-cdn.xitu.io/2018/11/13/1670d0d78678f8cb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

子方法先不看，先看看refresh方法的结构，其实就有几点值得学习：

1、方法为什么加锁？ 是为了避免多线程的场景下同时刷新Spring上下文

2、虽然整个方法是加锁的，但是却用了Synchronized关键字的对象锁startUpShutdownMonitor，这样做有两个好处：

（1）关闭资源的时候会调用close()方法，close()方法也使用了同样的对象锁，而关闭资源的close和refresh的两个冲突的方法，这样可以避免冲突

（2）此处对象锁相对于整个方法加锁的话，同步的范围更小了，锁的粒度更小，效率更高

3、这个方法refresh定义了整个Spring IOC的流程，每一个方法名字都清晰易懂，可维护性、可读性很强

**总结：看源码需要找准入口，看的时候多思考，学习Spring的巧妙的设计。ApplicationContext的构造方法中最关键是方法是refresh，其中有一些比价好的设计。**

### obtainFreshBeanFactory方法

这个方法作用是获取刷新Spring上下文的Bean工厂：

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {  
    this.refreshBeanFactory();  
    return this.getBeanFactory();
}复制代码
protected final void refreshBeanFactory() throws BeansException {  
    if (this.hasBeanFactory()) {    
        this.destroyBeans();    
        this.closeBeanFactory();  
    }  
    try {    
        DefaultListableBeanFactory beanFactory = this.createBeanFactory();    
        beanFactory.setSerializationId(this.getId());    
        this.customizeBeanFactory(beanFactory);    
        this.loadBeanDefinitions(beanFactory);    
        synchronized(this.beanFactoryMonitor) {      
            this.beanFactory = beanFactory;    }  
        } catch (IOException var5) {    
            throw new ApplicationContextException("I/O error parsing bean definition source for " + this.getDisplayName(), var5);  
        }
}复制代码
```

这断代码的核心是DefaultListableBeanFactory，核心类我们再整理一下，以图表格式：

下面有三个加粗的Map，这些个Map是解决问题的关键。。。我们之后详细分析

| **对象名**                    | **类 型**                       | **作 用**                                           | **归属类**                         |
| ----------------------------- | ------------------------------- | --------------------------------------------------- | ---------------------------------- |
| aliasMap                      | Map<String, String>             | 存储Bean名称->Bean别名映射关系                      | SimpleAliasRegistry                |
| **singletonObjects**          | **Map**                         | **存储单例Bean名称->单例Bean实现映射关系**          | **DefaultSingletonBeanRegistry**   |
| **singletonFactories**        | **Map**                         | **存储Bean名称->ObjectFactory实现映射关系**         | **DefaultSingletonBeanRegistry**   |
| **earlySingletonObjects**     | **Map**                         | **存储Bean名称->预加载Bean实现映射关系**            | **DefaultSingletonBeanRegistry**   |
| registeredSingletons          | Set<String>                     | 存储注册过的Bean名                                  | DefaultSingletonBeanRegistry       |
| singletonsCurrentlyInCreation | Set<String>                     | 存储当前正在创建的Bean名                            | DefaultSingletonBeanRegistry       |
| disposableBeans               | Map<String, Object>             | 存储Bean名称->Disposable接口实现Bean实现映射关系    | DefaultSingletonBeanRegistry       |
| factoryBeanObjectCache        | Map<String, Object>             | 存储Bean名称->FactoryBean接口Bean实现映射关系       | FactoryBeanRegistrySupport         |
| propertyEditorRegistrars      | Set<PropertyEditorRegistrar>    | 存储PropertyEditorRegistrar接口实现集合             | AbstractBeanFactory                |
| embeddedValueResolvers        | List<StringValueResolver>       | 存储StringValueResolver（字符串解析器）接口实现列表 | AbstractBeanFactory                |
| beanPostProcessors            | List<BeanPostProcessor>         | 存储 BeanPostProcessor接口实现列表                  | AbstractBeanFactory                |
| mergedBeanDefinitions         | Map<String, RootBeanDefinition> | 存储Bean名称->合并过的根Bean定义映射关系            | AbstractBeanFactory                |
| alreadyCreated                | Set<String>                     | 存储至少被创建过一次的Bean名集合                    | AbstractBeanFactory                |
| ignoredDependencyInterfaces   | Set<Class>                      | 存储不自动装配的接口Class对象集合                   | AbstractAutowireCapableBeanFactory |
| resolvableDependencies        | Map<Class, Object>              | 存储修正过的依赖映射关系                            | DefaultListableBeanFactory         |
| beanDefinitionMap             | Map<String, BeanDefinition>     | 存储Bean名称-->Bean定义映射关系                     | DefaultListableBeanFactory         |
| beanDefinitionNames           | List<String>                    | 存储Bean定义名称列表                                | DefaultListableBeanFactory         |

### BeanDefinition在IOC容器中的注册

接下来简要分析一下loadBeanDefinitions。

对于这个BeanDefinition，我是这么理解的: 它是SpringIOC过程中间的一个产物，可以看成是对Bean定义的抽象，里面封装的数据都是与Bean定义相关的，封装了一些基本的bean的Property、initi-method、destroy-method等。

这里的主要方法是loadBeanDefinitions，这里不详细展开说，它主要做了几件事：

1、初始化了BeanDefinitionReader

2、通过BeanDefinitionReader获取Resource，也就是xml配置文件的位置，并且把文件转换成一个叫Document的对象

3、接下来需要将Document对象转化成容器内部的数据结构（也就是BeanDefinition），也即是将Bean定义的List、Map、Set等各种元素进行解析，转换成Managed类（Spring对BeanDefinition数据的封装)放在BeanDefinition中；这个方法是RegisterBeanDefinition()，也就是解析的过程。

4、解析完成后，会把解析的结果放到BeanDefinition对象中并设置到一个Map中

以上这个过程就是BeanDefinition在IOC容器中的注册。

再回到Refresh方法，总结每一步如下图：



**总结：这一部分步骤主要是Spring如何加载Xml文件或者注解，并把它解析成BeanDefinition。**

### Spring创建Bean的过程

先回到之前的refresh方法(也就是在构造ApplicationContext时的方法)，我们跳过不重要的部分：

![img](https://user-gold-cdn.xitu.io/2018/11/14/1671267589e00578?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们直接看finishBeanFactoryInitialization里面的preInstantiateSingletons方法，顾名思义初始化所有的单例bean，截取部分如下：

![img](https://user-gold-cdn.xitu.io/2018/11/14/167129de5be02614?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

现在来看核心的getBean方法，对于所有获取Bean对象是实例，都是用这个getBean方法，这个方法最终调用的是doGetBean方法，这个方法就是所谓的DI（依赖注入）发生的地方。

程序=数据+算法，之前的BeanDefinition就是“数据”，依赖注入也就是在BeanDefinition准备好情况下进行进行的，这个过程不简单，因为Spring提供了很多参数配置，每一个参数都代表了IOC容器的特性，这些特性的实现需要在Bean的生命周期中完成。

代码比较多，就不贴了，大家可以自行查看AbstractBeanFactory里面的doGetBean方法,这里直接上图，这个图就是依赖注入的整个过程：

![img](https://user-gold-cdn.xitu.io/2018/11/15/16717dea23ee7ce9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**总结：Spring创建好了BeanDefinition之后呢，会开始实例化Bean，并且对Bean的依赖属性进行填充。实例化时底层使用了CGLIB或Java反射技术。上图中instantiateBean核PupulateBean方法很重要！**

### 循环依赖问题分析

我们先总结一下之前的结论：

1、构造器注入和prototype类型的field注入发生循环依赖时都无法初始化

2、field注入单例的bean时，尽管有循环依赖，但bean仍然可以被成功初始化

针对这几个结论，提出问题

1. 单例的设值注入bean是如何解决循环依赖问题呢？如果A中注入了B，那么他们初始化的顺序是什么样子的？
2. 为什么prototype类型的和构造器类型的Spring无法解决循环依赖呢？

之前在DefaultListableBeanFactory类中，列出了一个表格；现在我把关键的精华属性列出来：







```
一级缓存：
/** 保存所有的singletonBean的实例 */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(64);

二级缓存：
/** 保存所有早期创建的Bean对象，这个Bean还没有完成依赖注入 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
三级缓存：
/** singletonBean的生产工厂*/
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
 
/** 保存所有已经完成初始化的Bean的名字（name） */
private final Set<String> registeredSingletons = new LinkedHashSet<String>(64);
 
/** 标识指定name的Bean对象是否处于创建状态  这个状态非常重要 */
private final Set<String> singletonsCurrentlyInCreation =
	Collections.newSetFromMap(new ConcurrentHashMap<String, Boolean>(16));
复制代码
```



前面三个Map，我们称为单例初始化的三级缓存，理解这个问题，**我们目前只需关注“三级”，也就是****singletonFactories**

分析：

对于问题1，单例的设值注入，如果A中注入了B，B应该是A中的一个属性，那么猜想应该是A已经被instantiate（实例化）之后，在populateBean（填充A中的属性）时，对B进行初始化。

对于问题2，instantiate（实例化）其实就是理解成new一个对象的过程，而new的时候肯定要执行构造方法，所以猜想对于应该是A在instantiate（实例化）时，进行B的初始化。

有了分析和猜想之后呢，围绕关键的属性，根据从上图的doGetBean方法开始到populateBean所有的代码，我整理了如下图：

![img](https://user-gold-cdn.xitu.io/2018/11/17/1672097789fb9abe?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上图是整个过程中关键的代码路径，感兴趣的可以自己debug几回，最关键的解决循环依赖的是如上的两个标红的方法，第一个方法getSingleton会从singletonFactories里面拿Singleton，而addSingletonFactory会把Singleton放入singletonFactories。

**对于问题1：单例的设值注入bean是如何解决循环依赖问题呢？如果A中注入了B，那么他们初始化的顺序是什么样子的？**

假设循环注入是A-B-A：A依赖B(A中autowire了B)，B又依赖A（B中又autowire了A）：

![img](https://user-gold-cdn.xitu.io/2018/11/17/16720b0a8ca086c6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

本质就是三级缓存发挥作用，解决了循环。

**对于当时问题2，instantiate（实例化）其实就是理解成new一个对象的过程，而new的时候肯定要执行构造方法，所以猜想对于应该是A在instantiate（实例化）时，进行B的初始化。**

答案也很简单，因为A中构造器注入了B，那么A在关键的方法addSingletonFactory()之前就去初始化了B，导致三级缓存中根本没有A，所以会发生死循环，Spring发现之后就抛出异常了。至于Spring是如何发现异常的呢，本质上是根据Bean的状态给Bean进行mark，如果递归调用时发现bean当时正在创建中，那么久抛出循环依赖的异常即可。

**那么prototype的Bean是如何初始化的呢？**

prototypeBean有一个关键的属性：







```
/** Names of beans that are currently in creation */
private final ThreadLocal<Object> prototypesCurrentlyInCreation =
	new NamedThreadLocal<Object>("Prototype beans currently in creation");
复制代码
```



保存着正在创建的prototype的beanName，在流程上并没有暴露任何factory之类的缓存。并且在`beforePrototypeCreation(String beanName)`方法时，把每个正在创建的prototype的BeanName放入一个set中：







```
protected void beforePrototypeCreation(String beanName) {
		Object curVal = this.prototypesCurrentlyInCreation.get();
		if (curVal == null) {
			this.prototypesCurrentlyInCreation.set(beanName);
		}
		else if (curVal instanceof String) {
			Set<String> beanNameSet = new HashSet<String>(2);
			beanNameSet.add((String) curVal);
			beanNameSet.add(beanName);
			this.prototypesCurrentlyInCreation.set(beanNameSet);
		}
		else {
			Set<String> beanNameSet = (Set<String>) curVal;
			beanNameSet.add(beanName);
		}
}复制代码
```



并且会循环依赖时检查beanName是否处于创建状态，如果是就抛出异常：







```
protected boolean isPrototypeCurrentlyInCreation(String beanName) {
    Object curVal = this.prototypesCurrentlyInCreation.get();
    return (curVal != null &&
    (curVal.equals(beanName) || (curVal instanceof Set && ((Set<?>) curVal).contains(beanName))));
}复制代码
```



从流程上就可以查看，无论是构造注入还是设值注入，第二次进入同一个Bean的getBean方法是，一定会在校验部分抛出异常，因此不能完成注入，也就不能实现循环引用。

**总结：Spring在InstantiateBean时执行构造器方法，构造出实例，如果是单例的话，会将它放入一个\**singletonBeanFactory\**的缓存中，再进行populateBean方法，设置属性。通过一个singletonBeanFactory的缓存解决了循环依赖的问题。**

## 再解决一个问题

现在大家已经对Spring整个流程有点感觉了，我们再来解决一个简单的常见的问题：

考虑一下如下的singleton代码：







```
    @Service
    public class SingletonBean{

       @Autowired 
       private PrototypeBean prototypeBean;

       public void doSomething(){
         System.out.println(prototypeBean.toString());       }

    }复制代码
```









```
     @Component 
     @Scope(value="prototype")
     public class PrototypeBean{
     }复制代码
```



一个Singleton的Bean中Autowired了一个prototype的Bean，那么问题来了，每次调用SingletonBean.doSomething()时打印的对象是不是同一个呢？

有了之前的知识储备，我们简单分析一下：因为Singleton是单例的，所以在项目启动时就会初始化，prototypeBean本质上只是它的一个Property，那么ApplicationContex中只存在一个SingletonBean和一个初始化SingletonBean时创建的一个prototype类型的PrototypeBean。

那么每次调用SingletonBean.doSomething()时，Spring会从ApplicationContex中获取SingletonBean，每次获取的SingletonBean是同一个，所以即便PrototypeBean是prototype的，但PrototypeBean仍然是同一个。每次打印出来的内存地址肯定是同一个。

### 那这个问题如何解决呢？

解决办法也很简单，这种情况我们不能通过注入的方式注入一个prototypeBean，只能在程序运行时手动调用getBean("prototypeBean")方法，我写了一个简单的工具类：







```
@Service
public class SpringBeanUtils implements ApplicationContextAware {  
    private static ApplicationContext appContext;  
    @Override  
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException { 
       SpringBeanUtils.appContext=applicationContext;  
    }  
    public static ApplicationContext getAppContext() {    
        return appContext;  
    }  
    public static Object getBean(String beanName) {    
        checkApplicationContext();    
        return appContext.getBean(beanName);  
    }  
    private static void checkApplicationContext() {    
        if (null == appContext) {      
            throw new IllegalStateException("applicaitonContext未注入");   
         }  
    }  
    @SuppressWarnings("unchecked")  
    public static <T> T getBean(Class<T> clazz) {    
        checkApplicationContext();    
        Map<?, ?> map = appContext.getBeansOfType(clazz);    
        return map.isEmpty() ? null : (T) map.values().iterator().next();  
    }
 }复制代码
```



对于这个ApplicationContextAware接口：

在某些特殊的情况下，Bean需要实现某个功能，但该功能必须借助于Spring容器才能实现，此时就必须让该Bean先获取Spring容器，然后借助于Spring容器实现该功能。为了让Bean获取它所在的Spring容器，可以让该Bean实现ApplicationContextAware接口。

感兴趣的读者自己可以试试。

**总结：**

**回到循环依赖的问题，有的人可能会问 singletonBeanFactory只是一个三级缓存，那么一级缓存和二级缓存有什么用呢？**

**其实大家只要理解整个流程就可以切入了，Spring在初始化Singleton的时候大致可以分几步，初始化——设值——销毁，循环依赖的场景下只有A——B——A这样的顺序，但在并发的场景下，每一步在执行时，都有可能调用getBean方法，而单例的Bean需要保证只有一个instance，那么Spring就是通过这些个缓存外加对象锁去解决这类问题，同时也可以省去不必要的重复操作。Spring的锁的粒度选取也是很吊的，这里暂时不深入研究了。**

**解决此类问题的关键是要对SpringIOC和DI的整个流程做到心中有数，看源码一般情况下不要求每一行代码都了解透彻，但是对于整个的流程和每个流程中在做什么事需要了然，这样实际遇到问题时才可以很快的切入进行分析解决。**



# Spring AOP

​	AOP原理
​		AOP实际上就是OOP的补充，将代码横向抽取成一个独立的模块，再织入到目标方法中
​		底层是动态代理技术
​			JDK动态代理(基于接口)
​			CBLib动态代理(基于类)
​			在Spring AOP中，如果使用的是单例，推荐使用CGLib代理
​	AOP术语
​		连接点(Join point)
​			能够被拦截的地方
​		切点(Poincut)
​			具体定位的连接点
​		增强/通知(Advice)
​			表示添加到切点的一段逻辑代码，并定位连接点的方位信息
​		织入(Weaving)
​			将增强/通知添加到目标类的具体连接点上的过程。
​		引入/引介(Introduction)
​			允许我们向现有的类添加新方法或属性。是一种特殊的增强！
​		切面(Aspect)
​			切面由切点和增强/通知组成，它既包括了横切逻辑的定义、也包括了连接点的定义
​	Spring对AOP的支持
​		基于代理的经典SpringAOP：需要实现接口，手动创建代理
​		纯POJO切面：使用XML配置，aop命名空间
​		@AspectJ注解驱动的切面：使用注解的方式，这是最简洁和最方便的！
​	知识点
​		增强方式
​			前置增强
​			后置增强
​			环绕增强
​			异常抛出增强
​			引介/引入增强
​		切面类型
​			一般切面
​			切点切面
​			引介/引入切面
​		自动创建代理对象
​			BeanNameAutoProxyCreator
​			DefaultAdvisorAutoProxyCreator
​			AnnotationAwareAspectJAutoProxyCreator
​		切点函数
​			方法切点函数，其中execution()是用得最多的
​			方法入参函数切点函数
​			目标类切点函数类
​			代理类切点函数

# Spring bean的生命周期概述

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



# Spring 中的后置处理器

![img](https://pic1.zhimg.com/80/v2-9bd6efe7c86130553896c3744c338778_1440w.jpg)

上面BeanFactory、BeanDefinitionRegistryPostProcessor、BeanPostProcessor都算是后置处理器。

BeanFactoryPostProcessor是用来干预BeanFactory创建的，而BeanPostProcessor是用来干预Bean的实例化。

![img](https://pic1.zhimg.com/80/v2-b4cc010925170b3641f93c9c1ce57458_1440w.jpg)

![img](https://pic4.zhimg.com/80/v2-dc8000e96551247ddb182edc8f875e1f_1440w.jpg)

BeanPostProcessor的实现类

1 将@Spring AOP {Advisor}应用到特定bean的{BeanPostProcessor}实现的基类。

> aop.framework.AbstractAdvisingBeanPostProcessor#postProcessAfterInitialization

2 BeanPostProcessor实现使用AOP代理包装每个合格的bean，在调用bean本身之前将其委托给指定的拦截器

>   aop.framework.autoproxy.AbstractAutoProxyCreator#postProcessAfterInitialization

3 使用AdvisorAdapterRegistry（默认为{GlobalAdvisorAdapterRegistry}）在BeanFactory中注册{AdvisorAdapter} Bean的BeanPostProcessor

>   aop.framework.adapter.AdvisorAdapterRegistrationManager#postProcessAfterInitialization

4  BeanPostProcessor，用于检测实现{@code ApplicationListener} 接口的bean。这会捕获{@code getBeanNamesForType} 和仅对顶级bean有效的相关操作无法可靠检测到的beans

> org.springframework.context.support.ApplicationListenerDetector#postProcessAfterInitialization

5 当在BeanPostProcessor实例化期间创建一个Bean时，即当一个Bean不适合所有BeanPostProcessor处理时，记录一个消息消息的BeanPostProcessor

> org.springframework.context.support.PostProcessorRegistrationDelegate.BeanPostProcessorChecker#postProcessAfterInitialization

6 简单的{BeanPostProcessor}可以在Spring托管的bean中检查JSR-303约束注释，并在调用bean的init方法（如果有）之前，在违反约束的情况下抛出初始化异常。

> org.springframework.validation.beanvalidation.BeanValidationPostProcessor#postProcessAfterInitialization
>

7 BeanPostProcessor 实现调用带注释的init和destroy方法。允许使用注解替代Spring的InitializingBean 和DisposableBean回调接口

> org.springframework.beans.factory.annotation.InitDestroyAnnotationBeanPostProcessor#postProcessAfterInitialization

8 在SmartInstantiationAwareBeanPostProcessor上以无操作方式实现所有方法的适配器，该适配器不会更改容器实例化的每个bean的正常处理。子类只能覆盖它们实际感兴趣的那些方法

> org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessorAdapter#postProcessAfterInitialization
>

9 BeanPostProcessor实现,该实现将上下文的默认 LoadTimeWeaver传递给实现LoadTimeWeaverAware接口的bean

> org.springframework.context.weaving.LoadTimeWeaverAwareProcessor#postProcessAfterInitialization
>

10 Bean后处理器根据提供的“ fixedRate”，“ fixedDelay”或“ cron”表达式，注册由TaskScheduler调用的link Scheduled 注释的方法

> org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor#postProcessAfterInitialization
>



# Spring 解决循环依赖

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

**spring在默认单例的情况下是支持循环引用的**，相当于多了一个中间状态，一个对象

 new A后，发现要auto B ——>   getBean(B) 第一遍没有 —— >new B ——> 发现 auto A ——>getBean(A) 因为A当前正在创建中，所以返回一个A的对象，还不是Bean

​	getBean(A) 如果一级缓存单例池中没有，并且指定的单例bean当前正在创建中， 锁住一级缓存单例池，避免并发重复创建，

​	如果三级缓存中没有，但允许创建早期对象，二级缓存工厂返回当前单例的对象工厂，获得单例对象，加入三级缓存中，当前单例的早期引用(对象)，还未实例化为bean，从二级缓存中删除，因为单例已经创建了，不需要工厂创建该bean了，让垃圾回收)）



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

```java
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

IOC容器包括BeanDefinition，singletonObjects(单例缓存池)， beanFactory，三级缓存，后置处理器，ApplicationContext

对象工厂取出，然后放到三级缓存？然后清除二级缓存？
 三级缓存：防止对象重复创建
 二级缓存：通过工厂 取得对象(aop的代理对象)，解决循环依赖

	* singletonObjects存的是实例化好的bean对象，bean名字—bean实例
	* singletonFactories 存的是用于获取该对象的对象工厂，通过getObject方法获取，如果允许循环依赖，对象将被放入该map中，主要为了解决循环依赖
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



# Spring Bean的初始化

## AnnotationConfigApplicationContext

​	独立的应用程序上下文，接受组件类作为输入特别是{Configuration}注释的类，但也可以是普通的{ Component}类型和使用{inject}批注的符合JSR-330的类。允许使用{register（Class ...）}一注册类，以及使用{scan（String ...）}进行类路径扫描。

​	在有多个{@code @Configuration}类的情况下，在更高版本中定义的{@Bean}方法将覆盖先前版本中定义的方法。 可以通过一个额外的{@Configuration}类来故意重写某些bean的定义。

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200501203438699.png" alt="image-20200501203438699" style="zoom: 67%;" />

```java
//也可以用 ClassPathXmlApplicationContext
AnnotationConfigApplicationContext annoApplicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
```

#### 1.AnnotationConfigApplicationContext(componentClasses)

```java
public class AnnotationConfigApplicationContext extends GenericApplicationContext implements AnnotationConfigRegistry {
    
    private final AnnotatedBeanDefinitionReader reader;

    public AnnotationConfigApplicationContext() {
		this.reader = new AnnotatedBeanDefinitionReader(this);
		this.scanner = new ClassPathBeanDefinitionScanner(this);
	}
 //创建一个新的AnnotationConfigApplicationContext，从给定的组件类派生bean定义并自动刷新上下文。 @param componentClasses一个或多个组件类-例如	@Configuration类
    public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
        //1.调用自己的无参构造方法，并且调用父类GenericApplicationContext#GenericApplicationContext()的无参构造方法，里面实例化了beanFactory工厂 	this.beanFactory = new DefaultListableBeanFactory();
		this();
        //2.调用AnnotatedBeanDefinitionReader#register方法加载配置类,把配置类变成BeanDefinition并注册到 BeanDefinitionRegistry
		register(componentClasses);
		refresh();//3.调用的是父类的AbstractApplicationContext#refresh
	}
//注册一个或多个要处理的组件类。注意，必须调用refresh（）才能使上下文完全处理新类
 @Override
public void register(Class<?>... componentClasses) {
   Assert.notEmpty(componentClasses, "At least one component class must be specified");
   //通过该类 AnnotatedBeanDefinitionReader 进行注册类
   this.reader.register(componentClasses);
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
```

#### registerBeanDefinition(beanName, BeanDefinition)

```java
// BeanDefinitionRegistry接口的实现，注册BeanDefinition,beanDefinitionNames增加bean
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
```

#### 3.11.6 重点：preInstantiateSingletons()

```java
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
           // 3.11.6.6 调用AbstractBeanFactory#getBean,getBean又调用了doGetBean
            getBean(beanName); 
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







## AnnotatedBeanDefinitionReader

​	方便的适配器，用于以编程方式注册Bean类。 这是{ClassPathBeanDefinitionScanner}的替代方法，它采用相同的注释分辨率，但仅适用于显式注册的类。

#### 2. register(componentClasses)

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
```

#### 重点：2.1 doRegisterBean(beanClass)

```java
	//从给定的bean类中注册一个bean，并从类声明的批注中派生其元数据
    //@param beanClass Bean的类
    //@param name为bean的显式名称
    //@param qualifiers要考虑的特定限定符注释（如果有）除Bean类级别的限定符之外
    //@param supplier用于创建的回调Bean的实例（可能为{@code null}）
//@param customizers一个或多个用于自定义工厂的BeanDefinition的回调，例如设置lazy-init或primary标志
    private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			@Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,@Nullable BeanDefinitionCustomizer[] customizers) {	
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


## BeanDefinitionReaderUtils

对bean定义阅读器实现有用的实用程序方法。主要供内部使用

#### 2.1.1 registerBeanDefinition( BeanDefinitionHolder, BeanDefinitionRegistry)

```java
// 向给定的bean工厂注册给定的bean定义
// @param definitions保留bean定义，包括名称和别名
// @param Registry向bean工厂注册
public static void registerBeanDefinition(
      BeanDefinitionHolder definitionHolder,BeanDefinitionRegistry registry)
      throws BeanDefinitionStoreException {

   // 以beanName名称注册Bean定义
   String beanName = definitionHolder.getBeanName();
   registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

   // 注册bean名称的别名（如果有）
   String[] aliases = definitionHolder.getAliases();
   if (aliases != null) {
      for (String alias : aliases) {
         registry.registerAlias(beanName, alias);
      }
   }
}
```



## AbstractApplicationContext

​	**ApplicationContext**接口的抽象实现。不强制用于配置的存储类型；简单地实现通用上下文功能。使用模板方法设计模式，需要具体的子类来实现抽象方法。

​	与普通的BeanFactory相比，应该假定ApplicationContext来检测在其内部bean工厂中定义的特殊bean：因此，此类自动注册BeanFactoryPostProcessor、BeanPostProcessor和ApplicationListener 在上下文中被定义为bean。

#### 3. 重点：<a id="refresh">refresh()</a>

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
      implements ConfigurableApplicationContext {
      
 @Override
	public void refresh() throws BeansException, IllegalStateException {
        // 同步监视器的“刷新”和“销毁”
		synchronized (this.startupShutdownMonitor) {
			// 1.准备此上下文以进行刷新
			prepareRefresh();

			// 2.告诉子类刷新内部bean工厂
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// 3.准备 在这种情况下使用的bean工厂
			prepareBeanFactory(beanFactory);

			try {
				//4.允许在上下文子类中对bean工厂进行后处理
				postProcessBeanFactory(beanFactory);
//5.实例化并调用所有已注册的BeanFactoryPostProcessor，遵循显式顺序。必须在单例实例化之前调用
//调用在上下文中注册为bean的工厂处理器，完成了Bean的扫描，Bean装入beanDefinitionMap
				invokeBeanFactoryPostProcessors(beanFactory);

				//6.注册拦截Bean创建的Bean处理器
				registerBeanPostProcessors(beanFactory);

				//7.为此上下文初始化消息源
				initMessageSource();

				//8.为此上下文初始化事件多播器
				initApplicationEventMulticaster();

				//9.在特定上下文子类中初始化其他特殊bean
				onRefresh();

				//10.检查侦听器bean并注册它们
				registerListeners();

				//11.重点：完成该上下文的bean工厂的初始化，实例化所有剩余的（非延迟初始化）单例
				finishBeanFactoryInitialization(beanFactory);

				//12.最后一步：发布相应的事件
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
				//13. 重置Spring核心中的常见自省缓存，因为可能不再需要单例bean的元数据...
				resetCommonCaches();
			}
		}
	}
```

#### 3.5invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory)

```java
//实例化并调用所有已注册的BeanFactoryPostProcessor Bean(自己定义的和Spring自带的)，遵循显式顺序。必须在单例实例化之前调用
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
   //3.6.1 调用Bean Factory后置处理器
   //getBeanFactoryPostProcessors() 获得所有的Bean Factory后处理器
   PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
// 3.6.2 检测LoadTimeWeaver并准备进行编织（如果在此期间发现）（例如，通过ConfigurationClassPostProcessor注册的@Bean方法）
   if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
      beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
      beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
   }
}
```



#### 3.11重点：finishBeanFactoryInitialization(beanFactory)

```java
//完成该上下文的bean工厂的初始化，实例化所有剩余的（非延迟初始化）单例
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
   // 1.为此上下文初始化转换服务
   if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
         beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
      beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
   }

   //2.如果之前没有任何bean后处理器（例如PropertyPlaceholderConfigurer bean）注册过以下任何一个，则注册一个默认的嵌入式值解析器：此时，主要用于注释属性值的解析。
   if (!beanFactory.hasEmbeddedValueResolver()) {
      beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
   }

   // 3.尽早初始化LoadTimeWeaverAware Bean，以便尽早注册其转换器
   String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
   for (String weaverAwareName : weaverAwareNames) {
      getBean(weaverAwareName);
   }

   //4. 停止使用临时ClassLoader进行类型匹配
   beanFactory.setTempClassLoader(null);

   //5. 允许缓存所有bean定义元数据，而不期望进一步的更改。
   beanFactory.freezeConfiguration();

   //6. 实例化所有剩余的（非延迟初始化）单例
   beanFactory.preInstantiateSingletons();
}
```





## AbstractBeanFactory

BeanFactory实现的抽象基类，提供了ConfigurableBeanFactory SPI的全部功能。 

 假定有一个可列出的bean工厂：因此也可以用作 bean工厂实现的基类，该工厂从某些后端资源（bean定义访问是一项昂贵的操作）中获取bean定义。

此类通过其基类提供了一个单例缓存**DefaultSingletonBeanRegistry**，单例/原型确定，FactoryBean处理，别名，用于子bean定义的bean定义合并，和bean销毁DisposableBean 接口，自定义销毁方法）。此外，它还可以管理bean工厂通过实现**HierarchicalBeanFactory**接口实现层次结构（在未知bean的情况下委托给父对象）。

要由子类实现的主要模板方法是#**getBeanDefinition**和#**createBean**，分别为给定的bean名称检索一个bean定义和为给定的bean定义创建一个bean实例，这些操作的默认实现可以在**DefaultListableBeanFactory**和**AbstractAutowireCapableBeanFactory**

#### getBean(name)

​	在创建bean的时候和getBean的时候都调用getBean，先判断bean在不在单例池中，为了复用bean，不重复创建

#### 重点：doGetBean(name)

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
		//检查单例缓存是否有手动注册的单例。DefaultSingletonBeanRegistry#getSingleton(beanName)
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

				// 创建bean实例，第一次单例对象创建 
				if (mbd.isSingleton()) {
      //调用 DefaultSingletonBeanRegistry#getSingleton(beanName,ObjectFactory<?>)
					sharedInstance = getSingleton(beanName, () -> {
						try {
                            //完成了目标对象的创建，如果需要代理，还完成了代理
							//AbstractAutowireCapableBeanFactory#createBean(beanName, RootBeanDefinition, Object[] args)
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



## PostProcessorRegistrationDelegate

委托AbstractApplicationContext进行后处理器处理

#### 3.5.1 invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory , List<<BeanFactoryPostProcessor>BeanFactoryPostProcessor>)
```java
final class PostProcessorRegistrationDelegate {
   private PostProcessorRegistrationDelegate() {}

   public static void invokeBeanFactoryPostProcessors(
         ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

      //如果有的话，首先调用BeanDefinitionRegistryPostProcessors
      Set<String> processedBeans = new HashSet<>();

      if (beanFactory instanceof BeanDefinitionRegistry) {
         BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
         List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
      List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

         for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
            if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
               BeanDefinitionRegistryPostProcessor registryProcessor =
                     (BeanDefinitionRegistryPostProcessor) postProcessor;
               registryProcessor.postProcessBeanDefinitionRegistry(registry);
               registryProcessors.add(registryProcessor);
            }
            else {
               regularPostProcessors.add(postProcessor);
            }
         }

    // 不要在这里初始化FactoryBeans,我们需要保留所有未初始化的常规bean，以便让bean工厂后处理器对其应用！ 	   // 将实现PriorityOrdered，Ordered和其余的BeanDefinitionRegistryPostProcessor分开。
         List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

         // First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
         String[] postProcessorNames =
               beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
         for (String ppName : postProcessorNames) {
            if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
               currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
               processedBeans.add(ppName);
            }
         }
         sortPostProcessors(currentRegistryProcessors, beanFactory);
         registryProcessors.addAll(currentRegistryProcessors);
         invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
         currentRegistryProcessors.clear();

         // Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
         postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
         for (String ppName : postProcessorNames) {
            if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
               currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
               processedBeans.add(ppName);
            }
         }
         sortPostProcessors(currentRegistryProcessors, beanFactory);
         registryProcessors.addAll(currentRegistryProcessors);
         invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
         currentRegistryProcessors.clear();

         // Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
         boolean reiterate = true;
         while (reiterate) {
            reiterate = false;
            postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
            for (String ppName : postProcessorNames) {
               if (!processedBeans.contains(ppName)) {
                  currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                  processedBeans.add(ppName);
                  reiterate = true;
               }
            }
            sortPostProcessors(currentRegistryProcessors, beanFactory);
            registryProcessors.addAll(currentRegistryProcessors);
            invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
            currentRegistryProcessors.clear();
         }

         // Now, invoke the postProcessBeanFactory callback of all processors handled so far.
         invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
         invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
      }

      else {
         // Invoke factory processors registered with the context instance.
         invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
      }

      // Do not initialize FactoryBeans here: We need to leave all regular beans
      // uninitialized to let the bean factory post-processors apply to them!
      String[] postProcessorNames =
            beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

      // Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
      // Ordered, and the rest.
      List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
      List<String> orderedPostProcessorNames = new ArrayList<>();
      List<String> nonOrderedPostProcessorNames = new ArrayList<>();
      for (String ppName : postProcessorNames) {
         if (processedBeans.contains(ppName)) {
            // skip - already processed in first phase above
         }
         else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
         }
         else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
            orderedPostProcessorNames.add(ppName);
         }
         else {
            nonOrderedPostProcessorNames.add(ppName);
         }
      }

      // First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
      sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
      invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

      // Next, invoke the BeanFactoryPostProcessors that implement Ordered.
      List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
      for (String postProcessorName : orderedPostProcessorNames) {
         orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
      }
      sortPostProcessors(orderedPostProcessors, beanFactory);
      invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

      // Finally, invoke all other BeanFactoryPostProcessors.
      List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
      for (String postProcessorName : nonOrderedPostProcessorNames) {
         nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
      }
      invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

      // Clear cached merged bean definitions since the post-processors might have
      // modified the original metadata, e.g. replacing placeholders in values...
      beanFactory.clearMetadataCache();
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



####  getSingleton(beanName)

```java
@Override
@Nullable
public Object getSingleton(String beanName) {
   //true 代表是否允许循环引用，创建早期参考，放入第三个缓存中earlySingletonObjects
   return getSingleton(beanName, true);
}
```

####  重点：getSingleton(beanName, boolean allowEarlyReference)

```java
/**
*返回以给定名称注册的（原始）单例对象。 检查已经实例化的单例，并允许 创建当前单例的早期引用（解析循环引用）。 * @param beanName要查找的bean的名称
* @param allowEarlyReference是否应创建早期引用
* @返回注册的单例对象，如果找不到则返回{null}
*/
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
   Object singletonObject = this.singletonObjects.get(beanName);
   //如果一级缓存单例池中没有，并且指定的单例bean当前正在创建中
   if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
      // 锁住一级缓存单例池，避免并发重复创建
      synchronized (this.singletonObjects) {
         singletonObject = this.earlySingletonObjects.get(beanName);
         //如果三级缓存中没有，但允许创建早期对象
         if (singletonObject == null && allowEarlyReference) {
            //二级缓存工厂返回当前单例的对象工厂
            ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
            if (singletonFactory != null) {
               //通过二级缓存工厂，返回单例对象
               singletonObject = singletonFactory.getObject();
               //放入三级缓存中，当前单例的早期引用(对象)，还未实例化为bean
               this.earlySingletonObjects.put(beanName, singletonObject);
               //从二级缓存中删除，因为单例已经创建了，不需要工厂创建该bean了，让垃圾回收
               this.singletonFactories.remove(beanName);
            }
         }
      }
   }
   return singletonObject;
}
```



#### 重点：getSingleton(beanName, ObjectFactory<?> singletonFactory)

```java
/**
 *返回以给定名称注册的（原始）单例对象，如果尚未注册，则创建并注册一个新对象。
 * @param beanName bean的名称
 * @param singletonFactory要延迟创建单例的ObjectFactory 
 * @返回注册的单例对象
 */
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
  
   synchronized (this.singletonObjects) {
      Object singletonObject = this.singletonObjects.get(beanName);
      if (singletonObject == null) {
      
         beforeSingletonCreation(beanName);
         boolean newSingleton = false;
         boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
         if (recordSuppressedExceptions) {
            this.suppressedExceptions = new LinkedHashSet<>();
         }
         try {
            singletonObject = singletonFactory.getObject();
            newSingleton = true;
         }
         catch (IllegalStateException ex) {
            // 在此期间，是否使单例对象隐式出现-> 如果是，请继续处理该对象，因为异常指示该状态。
            singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
               throw ex;
            }
         }
         catch (BeanCreationException ex) {
            if (recordSuppressedExceptions) {
               for (Exception suppressedException : this.suppressedExceptions) {
                  ex.addRelatedCause(suppressedException);
               }
            }
            throw ex;
         }
         finally {
            if (recordSuppressedExceptions) {
               this.suppressedExceptions = null;
            }
            afterSingletonCreation(beanName);
         }
         if (newSingleton) {
            addSingleton(beanName, singletonObject);
         }
      }
      return singletonObject;
   }
}
```



## AbstractAutowireCapableBeanFactory

实现默认bean创建的抽象bean工厂超类，具有{RootBeanDefinition}类指定的全部功能。实现{AutowireCapableBeanFactory}，除了AbstractBeanFactory的{createBean}方法之外的接口。

提供bean创建（具有构造函数解析）、属性填充，布线（包括自动布线）和初始化。处理运行时bean引用、解析托管集合、调用初始化方法等。支持自动连接构造函数、按名称的属性和按类型的属性。

子类要实现的主要模板方法是{#resolveDependency（DependencyDescriptor，String，Set，TypeConverter）}，用于按类型自动布线。如果工厂能够搜索它的bean定义，匹配的bean通常通过一次搜查。对于其他工厂样式，可以实现简化的匹配算法。

```java
 * @see RootBeanDefinition
 * @see DefaultListableBeanFactory
 * @see BeanDefinitionRegistry
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
  
  /** 创建bean实例的策略 */
	private InstantiationStrategy instantiationStrategy = new CglibSubclassingInstantiationStrategy();

	/** 方法参数名称的解析器策略 */
	@Nullable
	private ParameterNameDiscoverer parameterNameDiscoverer = new DefaultParameterNameDiscoverer();

	/** 是否自动尝试解析Bean之间的循环引用 */
	private boolean allowCircularReferences = true;

	/**
	 * 在循环引用的情况下是否求助于注入原始bean实例，即使注入的bean最终被包装了
	 */
	private boolean allowRawInjectionDespiteWrapping = false;

	/**
	 * 在依赖项检查和自动装配时要忽略的依赖项类型，例如 Class对象的Set：例如String。默认为无
	 */
	private final Set<Class<?>> ignoredDependencyTypes = new HashSet<>();

	/**
	 * 依赖关系接口忽略依赖检查和自动装配，如类对象的集合。默认情况下，仅BeanFactory接口被忽略。
	 */
	private final Set<Class<?>> ignoredDependencyInterfaces = new HashSet<>();

	/**
	 *当前创建的bean的名称，用于从用户指定的Supplier回调触发的getBean等调用上的隐式依赖项注册。
	 */
	private final NamedThreadLocal<String> currentlyCreatedBean = new NamedThreadLocal<>("Currently created bean");

	/** 未完成的FactoryBean实例的高速缓存：BeanWrapper的FactoryBean名称 */
	private final ConcurrentMap<String, BeanWrapper> factoryBeanInstanceCache = new ConcurrentHashMap<>();

	/** 按工厂类别缓存候选工厂方法 */
	private final ConcurrentMap<Class<?>, Method[]> factoryMethodCandidateCache = new ConcurrentHashMap<>();

	/** 筛选后的PropertyDescriptor的缓存：Bean类到PropertyDescriptor数组 */
	private final ConcurrentMap<Class<?>, PropertyDescriptor[]> filteredPropertyDescriptorsCache =
			new ConcurrentHashMap<>();
```



#### 重点：createBean(beanName,RootBeanDefinition, Object[] args)

```java
/**
 * 此类的中央方法：创建一个bean实例，填充该bean实例，替换为aop代理类，应用后置处理器等。
 * @see #doCreateBean
 */
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
	//启用了跟踪
   if (logger.isTraceEnabled()) {
      logger.trace("Creating instance of bean '" + beanName + "'");
   }
   RootBeanDefinition mbdToUse = mbd;

   // 确保此时确实解析了bean类，并且如果动态解析的类无法存储在共享的合并bean定义中，则复制bean定义。
   Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
   if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
      mbdToUse = new RootBeanDefinition(mbd);
      mbdToUse.setBeanClass(resolvedClass);
   }

   // 准备方法替代
   // 处理lookup-method和replace-method配置，Spring将这两个配置统称为Overrides
    try {
      mbdToUse.prepareMethodOverrides();
   	}
   }

   try {
      // 给BeanPostProcessors返回一个代理而不是目标bean实例的机会
      Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
      if (bean != null) {
         return bean;
      }
   }

   try {
      // doCreateBean 方法完成了代理的织入
      Object beanInstance = doCreateBean(beanName, mbdToUse, args);
      if (logger.isTraceEnabled()) {
         logger.trace("Finished creating instance of bean '" + beanName + "'");
      }
      return beanInstance;
   }
   catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
      // A previously detected exception with proper bean creation context already,
      // or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
      throw ex;
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
   }
}
```



#### doCreateBean(beanName,RootBeanDefinition, Object[] args) 代理织入

实际创建指定的bean。预创建处理已经发生，此时检查{@code postProcessBeforeInstantiation}回调

```java
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
      throws BeanCreationException {

   // 实例化包装器bean
   BeanWrapper instanceWrapper = null;
   if (mbd.isSingleton()) {
      //factoryBeanInstanceCache未完成的FactoryBean实例的缓存：BeanWrapper的FactoryBean名称
      instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
   }
   if (instanceWrapper == null) {
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }
   final Object bean = instanceWrapper.getWrappedInstance();
   Class<?> beanType = instanceWrapper.getWrappedClass();
   if (beanType != NullBean.class) {
      mbd.resolvedTargetType = beanType;
   }

   // Allow post-processors to modify the merged bean definition.
   synchronized (mbd.postProcessingLock) {
      if (!mbd.postProcessed) {
         try {
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
         }
       
         mbd.postProcessed = true;
      }
   }

   // Eagerly cache singletons to be able to resolve circular references
   // even when triggered by lifecycle interfaces like BeanFactoryAware.
   boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
         isSingletonCurrentlyInCreation(beanName));
   if (earlySingletonExposure) {
      addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
   }

   // 初始化bean实例
   Object exposedObject = bean;
   try {
      populateBean(beanName, mbd, instanceWrapper);
      //先把原生对象拿出来，然后用把原生对象复制给新对象，改变新对象，原生对象并没有改变
      exposedObject = initializeBean(beanName, exposedObject, mbd);
   }
     
   }

   if (earlySingletonExposure) {
      Object earlySingletonReference = getSingleton(beanName, false);
      if (earlySingletonReference != null) {
         if (exposedObject == bean) {
            exposedObject = earlySingletonReference;
         }
         else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
            String[] dependentBeans = getDependentBeans(beanName);
            Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
            for (String dependentBean : dependentBeans) {
               if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                  actualDependentBeans.add(dependentBean);
               }
            }
            }
         }
      }
   }

   // Register bean as disposable.
   try {
      registerDisposableBeanIfNecessary(beanName, bean, mbd);
   }
   return exposedObject;
}
```



#### initializeBean(beanName, bean,RootBeanDefinition)

初始化给定的bean实例，应用工厂回调以及init方法和bean post处理器

对于传统定义的bean，从{ #createBean}调用，对于现有的bean实例，从{#initializeBean}调用。

```java
/** @see BeanNameAware
	 * @see BeanClassLoaderAware
	 * @see BeanFactoryAware
	 * @see #applyBeanPostProcessorsBeforeInitialization
	 * @see #invokeInitMethods
	 * @see #applyBeanPostProcessorsAfterInitialization
	 */
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
   if (System.getSecurityManager() != null) {
      AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
         invokeAwareMethods(beanName, bean);
         return null;
      }, getAccessControlContext());
   }
   else {
      invokeAwareMethods(beanName, bean);
   }

   Object wrappedBean = bean;
   if (mbd == null || !mbd.isSynthetic()) {
      //初始化前应用Bean后处理器
      wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
   }

   try {
      invokeInitMethods(beanName, wrappedBean, mbd);
   }
   catch (Throwable ex) {
      throw new BeanCreationException(
            (mbd != null ? mbd.getResourceDescription() : null),
            beanName, "Invocation of init method failed", ex);
   }
   if (mbd == null || !mbd.isSynthetic()) {
     //在初始化之后应用Bean后处理器 代理织入
      wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
   }

   return wrappedBean;
}
```



####  resolveBeforeInstantiation( beanName, RootBeanDefinition) 实例化之前的后置处理器



```java
/**
 * 应用实例化之前的后处理器，以解决指定bean实例化之前是否存在快捷方式
 * @param beanName the name of the bean
 * @param mbd the bean definition for the bean
 * @return the shortcut-determined bean instance, or {@code null} if none
 */
@Nullable
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
   Object bean = null;
   if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
      // 确保此时确实解决了bean类
      if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
         Class<?> targetType = determineTargetType(beanName, mbd);
         if (targetType != null) {
            //将InstantiationAwareBeanPostProcessors应用于指定的bean定义（按类和名称），调用其{postProcessBeforeInstantiation}方法。
            bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
            if (bean != null) {
               bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
            }
         }
      }
      mbd.beforeInstantiationResolved = (bean != null);
   }
   return bean;
}
```

#### applyBeanPostProcessorsBeforeInstantiation( beanClass, beanName) 实例化前的回调后置处理器

```java
/**
 *将InstantiationAwareBeanPostProcessors应用于指定的bean定义（按类和名称），调用其实例化之前的后期处理{postProcessBeforeInstantiation}方法。
 *任何返回的对象都将用作bean，而不是实际实例化目标bean。来自后处理器的{null}返回值将导致目标Bean被实例化。
 * @param beanClass the class of the bean to be instantiated
 * @param beanName the name of the bean
 * @return the bean object to use instead of a default instance of the target bean, or {@code null}
 * @see InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation
 */
@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
   for (BeanPostProcessor bp : getBeanPostProcessors()) {
      if (bp instanceof InstantiationAwareBeanPostProcessor) {
      InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
//调用实例化之前的后期处理InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation
         Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
         if (result != null) {
            return result;
         }
      }
   }
   return null;
}
```



#### applyBeanPostProcessorsBeforeInitialization(existingBean,beanName) 返回原始实例的包装

将{BeanPostProcessor}应用于给定的现有bean实例，调用其{ postProcessBeforeInitialization}方法。 返回的bean实例可能是原始实例的包装。

```java
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
      throws BeansException {

   Object result = existingBean;
   for (BeanPostProcessor processor : getBeanPostProcessors()) {
      Object current = processor.postProcessBeforeInitialization(result, beanName);
      if (current == null) {
         return result;
      }
      result = current;
   }
   return result;
}
```



#### applyBeanPostProcessorsAfterInitialization( existingBean,beanName)

将{BeanPostProcessors}应用于给定的现有bean 实例，调用其{@code postProcessAfterInitialization}方法。

返回的bean实例可能是原始对象的包装器

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
      throws BeansException {

   Object result = existingBean;
   for (BeanPostProcessor processor : getBeanPostProcessors()) {
       //调用所有的后置处理器，执行他们的postProcessAfterInitialization，如果Bean被子类标识为要代理的豆，则使用配置的拦截器创建代理
       org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator#postProcessAfterInitialization
      Object current = processor.postProcessAfterInitialization(result, beanName);
      if (current == null) {
         return result;
      }
      result = current;
   }
   return result;
}
```

## BeanDefinition 接口

BeanDefinition接口描述了一个bean实例，它具有属性值，构造函数参数值以及具体实现所提供的更多信息。

这只是一个最小的接口：主要目的是允许{BeanFactoryPostProcessor}内省和修改属性值和其他bean元数据

![image-20200503120038598](images\image-20200503120038598.png)

```java
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



## BeanFactoryPostProcessor 接口

工厂挂钩，允许自定义修改应用程序上下文的 bean定义，以适应上下文基础 bean工厂的bean属性值

#### postProcessBeanFactory(beanFactory)

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
	//在标准初始化之后，修改应用程序上下文的内部bean工厂。所有bean定义都将被加载，但是还没有实例化bean *。这甚至可以覆盖或添加*属性，甚至可以用于初始化bean。
	//@param beanFactory应用程序上下文使用的bean工厂
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}

```





## BeanPostProcessor初始化后的回调后置处理器接口

所有的后置处理器都实现了**BeanPostProcessor**，自动注入和AOP也是实现了它。

实例化——整个过程，初始化——对象new出来后。

默认在九个地方执行了5个后置处理器，在对象初始化后执行的。

工厂挂钩允许自定义修改新的bean实例。例如，检查标记接口或使用代理包装bean。 

通常，通过标记接口或类似对象填充bean的后处理器将实现{ postProcessBeforeInitialization}，而使用代理包装bean的后处理器通常将实现{postProcessAfterInitialization}。

#### postProcessBeforeInitialization(bean,beanName) 初始化回调之前

#### postProcessAfterInitialization(bean, beanName) 初始化回调之后

```java
//注册
//在ApplicationContext中自动检测到的 BeanPostProcessor bean将根据PriorityOrdered 和 Ordered进行排序语义。相比之下，应用通过 BeanFactory 以编程方式注册的 BeanPostProcessor bean 注册顺序；通过程序注册的后处理器将忽略通过实现 PriorityOrdered 或 Ordered 接口表示的任何排序语义。此外，对于BeanPostProcessor bean，不考虑@Order批注。
// @see InstantiationAwareBeanPostProcessor
// @see DestructionAwareBeanPostProcessor
// @see ConfigurableBeanFactory#addBeanPostProcessor
// @see BeanFactoryPostProcessor
public interface BeanPostProcessor {
/**
 * 在任何 bean 初始化回调（如InitializingBean的{afterPropertiesSet} 或自定义的init-method）之前，将此{BeanPostProcessor}应用于给定的新bean实例。该bean将已经用属性值填充。返回的bean实例可能是原始实例的包装。此时该bean未初始化
 * @param bean 新的bean实例
 * @param beanName bean的名称
 * @return 要使用的bean实例，原始实例或包装实例
 * if {@code null}, 如果{@code null}，则不会调用后续的BeanPostProcessor
 * @throws org.springframework.beans.BeansException in case of errors
 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
 */
@Nullable
default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
   return bean;
}

/**
 * 在任何bean 初始化回调（例如InitializingBean的{afterPropertiesSet} 或自定义的init-method）之后，将此{ BeanPostProcessor}应用于给定的新bean实例。该bean将已经用属性值填充。 返回的bean实例可能是原始实例的包装。 
 对于FactoryBean，将为FactoryBean 实例和由FactoryBean创建的对象（从Spring 2.0开始）调用此回调。*后处理器可以通过相应的{bean instanceof FactoryBean}检查来决定是应用到FactoryBean还是创建的对象，还是两者都应用。
 *与所有其他{ BeanPostProcessor}回调相反，此回调也会在 {InstantiationAwareBeanPostProcessor＃postProcessBeforeInstantiation}方法触发短路后被调用。
 * 默认实现按原样返回给定的{ bean}。
 * @param bean the new bean instance
 * @param beanName the name of the bean
 * @return the bean instance to use, either the original or a wrapped one;
 * if {@code null}, no subsequent BeanPostProcessors will be invoked
 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
 * @see org.springframework.beans.factory.FactoryBean
 */
@Nullable
default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
   return bean;
}
```

#### 通过BeanPostProcessor实现AOP功能

通过实现BeanPostProcessor，在初始化之后找到需要代理的对象，返回一个aop代理的对象，这个aop对象就是这个对象在spring中的bean

```java
@Component
public class AopPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("aopServiceImpl")){
            Object o = Proxy.newProxyInstance(AopPostProcessor.class.getClassLoader(),bean.getClass().getInterfaces(), new InvocationHandler(){
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    System.out.println("aop start");
                    Object s = method.invoke(bean,args);
                    System.out.println("aop end");
                    return s;
                }
            });
            return o;
        }
        return null;
    }
}
```





## InitializingBean 接口

​	由 BeanFactory 设置完所有属性后需要进行响应的bean所实现的接口：例如执行自定义初始化 或 仅检查所有必需属性是否已设置。

​	实现 InitializingBean的另一种方法是指定自定义 init方法，例如在XML bean定义中。有关所有bean生命周期方法的列表，参见{BeanFactory BeanFactory javadocs}。

#### afterPropertiesSet() 所有bean属性设置后进行最后的初始化

```java
/**
 * @see DisposableBean
 * @see BeanDefinition#getPropertyValues()
 * @see AbstractBeanDefinition#getInitMethodName()
 */
public interface InitializingBean {

   /**
    * 由包含的 BeanFactory 设置了所有bean属性并满足 BeanFactoryAware， ApplicationContextAware等之后调用。
    此方法允许bean实例对其总体配置进行验证。所有bean属性设置后进行最后的初始化。
    */
   void afterPropertiesSet() throws Exception;
```



## InstantiationAwareBeanPostProcessor 实例化之前的回调 接口 extends BeanPostProcessor

BeanPostProcessor的子接口，它添加了实例化之前的回调，*以及实例化之后但设置了显式属性或*自动装配之前的回调。

通常用于禁止特定目标bean的默认实例化，例如创建带有特殊TargetSource的代理（池目标，延迟初始化目标等），或实施其他注入策略，例如字段注入。

此接口是一个专用接口，主要供框架内部使用。建议尽可能实现简单的 BeanPostProcessor 接口，或从 InstantiationAwareBeanPostProcessorAdapter 派生，以便屏蔽该接口的扩展。

####  postProcessBeforeInstantiation(beanClass,beanName) 实例化前的后期处理

```java
// @see AbstractAutoProxyCreator#setCustomTargetSourceCreators
// @see LazyInitTargetSourceCreator
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
/**
	 * 在实例化目标bean之前，先应用BeanPostProcessor。返回的bean对象可能是代替目标bean使用的代理，*有效地抑制了目标bean的默认实例化。
	 * <p>If a non-null object is returned by this method, the bean creation process
	 * will be short-circuited. The only further processing applied is the
	 * {@link #postProcessAfterInitialization} callback from the configured
	 * {@link BeanPostProcessor BeanPostProcessors}.
	 * <p>This callback will be applied to bean definitions with their bean class,
	 * as well as to factory-method definitions in which case the returned bean type
	 * will be passed in here.
	 * <p>Post-processors may implement the extended
	 * {@link SmartInstantiationAwareBeanPostProcessor} interface in order
	 * to predict the type of the bean object that they are going to return here.
	 * <p>The default implementation returns {@code null}.
	 * @param beanClass the class of the bean to be instantiated
	 * @param beanName the name of the bean
	 * @return the bean object to expose instead of a default instance of the target bean,
	 * or {@code null} to proceed with default instantiation
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see #postProcessAfterInstantiation
	 * @see org.springframework.beans.factory.support.AbstractBeanDefinition#getBeanClass()
	 * @see org.springframework.beans.factory.support.AbstractBeanDefinition#getFactoryMethodName()
	 */
	@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}
```

#### postProcessAfterInstantiation(bean,beanName) 实例化后的后期处理

```java
/**
 * Perform operations after the bean has been instantiated, via a constructor or factory method,
 * but before Spring property population (from explicit properties or autowiring) occurs.
 * <p>This is the ideal callback for performing custom field injection on the given bean
 * instance, right before Spring's autowiring kicks in.
 * <p>The default implementation returns {@code true}.
 * @param bean the bean instance created, with properties not having been set yet
 * @param beanName the name of the bean
 * @return {@code true} if properties should be set on the bean; {@code false}
 * if property population should be skipped. Normal implementations should return {@code true}.
 * Returning {@code false} will also prevent any subsequent InstantiationAwareBeanPostProcessor
 * instances being invoked on this bean instance.
 * @throws org.springframework.beans.BeansException in case of errors
 * @see #postProcessBeforeInstantiation
 */
default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
	return true;
}
```
#### postProcessProperties(PropertyValues, bean,beanName) 后处理属性

```java
/**
 * Post-process the given property values before the factory applies them
 * to the given bean, without any need for property descriptors.
 * <p>Implementations should return {@code null} (the default) if they provide a custom
 * {@link #postProcessPropertyValues} implementation, and {@code pvs} otherwise.
 * In a future version of this interface (with {@link #postProcessPropertyValues} removed),
 * the default implementation will return the given {@code pvs} as-is directly.
 * @param pvs the property values that the factory is about to apply (never {@code null})
 * @param bean the bean instance created, but whose properties have not yet been set
 * @param beanName the name of the bean
 * @return the actual property values to apply to the given bean (can be the passed-in
 * PropertyValues instance), or {@code null} which proceeds with the existing properties
 * but specifically continues with a call to {@link #postProcessPropertyValues}
 * (requiring initialized {@code PropertyDescriptor}s for the current bean class)
 * @throws org.springframework.beans.BeansException in case of errors
 * @since 5.1
 * @see #postProcessPropertyValues
 */
@Nullable
default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
		throws BeansException {

	return null;
}
```



## AbstractAutoProxyCreator使用AOP代理合格的bean

实现BeanPostProcessor,使用AOP代理包装每个合格的bean，在调用bean本身之前将其委托给指定的拦截器。

#### postProcessAfterInitialization(bean, beanName) 使用配置的拦截器创建代理

```java
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport
		implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {
    
/**
 * 如果Bean被子类标识为要代理的话，则使用配置的拦截器创建代理。
 * @see #getAdvicesAndAdvisorsForBean
 */
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
   if (bean != null) {
      Object cacheKey = getCacheKey(bean.getClass(), beanName);
      if (this.earlyProxyReferences.remove(cacheKey) != bean) {
         //必要时包装给定的bean，即是否有资格被代理。
         return wrapIfNecessary(bean, beanName, cacheKey);
      }
   }
   return bean;
}
```

#### wrapIfNecessary(bean, beanName, cacheKey)

必要时包装给定的bean，即是否有资格被代理。

```java
/** 必要时包装给定的bean，即是否有资格被代理
* @param bean原始bean实例
* @param beanName Bean的名称
* @param cacheKey用于元数据访问的缓存键
* @返回包装该bean的代理，或原始bean实例
*/
protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
   if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
      return bean;
   }
   if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
      return bean;
   }
   if (isInfrastructureClass(bean.getClass()) || shouldSkip(bean.getClass(), beanName)) {
      this.advisedBeans.put(cacheKey, Boolean.FALSE);
      return bean;
   }

   // 如果有需要，请创建代理。
   // 返回是否要代理给定的bean，要应用哪些其他建议（例如AOP联盟拦截器）和顾问。
   Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, null);
   if (specificInterceptors != DO_NOT_PROXY) {
      this.advisedBeans.put(cacheKey, Boolean.TRUE);
      //不为空，创建代理
      Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
      this.proxyTypes.put(cacheKey, proxy.getClass());
      return proxy;
   }

   this.advisedBeans.put(cacheKey, Boolean.FALSE);
   return bean;
}
```



#### 代理：createProxy(beanClass,beanName,specificInterceptors,targetSource)为给定的bean创建一个AOP代理

```java
/** @param beanClass Bean的类
  * @param beanName Bean的名称
  * @param specificInterceptors特定于此bean的一组拦截器（可以为空，但不能为null）
  * @param targetSource代理，已预先配置为访问Bean
  * @返回该Bean的AOP代理 @请参阅#buildAdvisors
*/
protected Object createProxy(Class<?> beanClass, @Nullable String beanName,
			@Nullable Object[] specificInterceptors, TargetSource targetSource) {

		if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
			AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
		}

		ProxyFactory proxyFactory = new ProxyFactory();
		proxyFactory.copyFrom(this);

		if (!proxyFactory.isProxyTargetClass()) {
			if (shouldProxyTargetClass(beanClass, beanName)) {
				proxyFactory.setProxyTargetClass(true);
			}
			else {
				evaluateProxyInterfaces(beanClass, proxyFactory);
			}
		}

		Advisor[] advisors = buildAdvisors(beanName, specificInterceptors);
		proxyFactory.addAdvisors(advisors);
		proxyFactory.setTargetSource(targetSource);
		customizeProxyFactory(proxyFactory);

		proxyFactory.setFrozen(this.freezeProxy);
		if (advisorsPreFiltered()) {
			proxyFactory.setPreFiltered(true);
		}
		//通过ClassLoader把代理类动态加载到JVM中ProxyFactory#getProxy(ClassLoader)
    	//getProxy
		return proxyFactory.getProxy(getProxyClassLoader());
	}
```





## AnnotationAwareAspectJAutoProxyCreator 处理AspectJ注解

AspectJAwareAdvisorAutoProxyCreator子类，用于处理当前应用程序上下文以及Spring Advisor中的所有AspectJ 注释方面。

如果Spring AOP的基于代理的模型能够应用所有的AspectJ带注释的类，则它们将被自动识别。 这涵盖了方法执行连接点。

如果使用<aop：include>元素，则仅将名称与包含模式匹配的@AspectJ bean视为定义要用于Spring自动代理的方面。

Spring Advisor的处理遵循AbstractAdvisorAutoProxyCreator中建立的规则。

#### setIncludePatterns(patterns)

```java
// @see org.springframework.aop.aspectj.annotation.AspectJAdvisorFactory
public class AnnotationAwareAspectJAutoProxyCreator extends AspectJAwareAdvisorAutoProxyCreator {
   /**
	 * 设置一个正则表达式模式列表，匹配合格的@AspectJ bean名称
	 * 默认是将所有@AspectJ bean视为合格
	 */
public void setIncludePatterns(List<String> patterns) {
		this.includePatterns = new ArrayList<>(patterns.size());
		for (String patternText : patterns) {
			this.includePatterns.add(Pattern.compile(patternText));
		}
	}
```



## ProxyFactory AOP代理的工厂

用于AOP代理的工厂，以编程方式使用，而不是通过在bean工厂中进行声明式设置。此类提供一种简单的方式来获取并在自定义用户代码中配置AOP代理实例

#### getProxy(classLoader) 动态代理加载classLoader

```java
public class ProxyFactory extends ProxyCreatorSupport {
public Object getProxy(@Nullable ClassLoader classLoader) {
   //返回AopProxy接口，该接口有两个实现类CglibAopProxy和JdkDynamicAopProxy
   //确定哪个实现类加载classLoader
   //JdkDynamicAopProxy#getProxy(classLoader)
   //CglibAopProxy#getProxy(classLoader)
   return createAopProxy().getProxy(classLoader);
}
```



## ProxyCreatorSupport 代理工厂的基类

提供对可配置AopProxyFactory的便捷访问

#### createAopProxy() 返回代理实现

```java
	/**
	 *子类应调用此方法以获得新的AOP代理。使用{this}作为参数创建一个AOP代理。
	 */
	protected final synchronized AopProxy createAopProxy() {
        //this.active 设置为 true
		if (!this.active) {
			activate();
		}
		return getAopProxyFactory().createAopProxy(this);
	}
```



## AopProxyFactory AOP代理的工厂实现的接口

由能够基于{@link AdvisedSupport}配置对象创建 AOP代理的工厂实现的接口。

#### createAopProxy(config)

```java
//实现类 DefaultAopProxyFactory#createAopProxy
AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException;
```



## DefaultAopProxyFactory 创建CGLIB代理或JDK动态代理

默认的{ AopProxyFactory}实现，创建CGLIB代理或JDK动态代理

#### createAopProxy(AdvisedSupport config) 返回CGLIB代理或JDK代理

```java
public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {
	// 返回Aop代理
   @Override
   public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
      // 返回 代理是否应执行积极的优化。默认 false
      // 返回 代理目标类别 是代理目标类或者接口 默认 false
      //确定提供的{AdvisedSupport}是否仅指定了SpringProxy接口或根本没有指定代理接口 默认 false
      if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
         Class<?> targetClass = config.getTargetClass();
		 // 目标类型是接口 或者 是代理类 执行JDK动态代理
         if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
         }
         // 目标类型不是接口 或者 不是代理类 执行Cglib动态代理
         return new ObjenesisCglibAopProxy(config);
      }
      else {
         // 默认JDK代理
         return new JdkDynamicAopProxy(config);
      }
   }
```



## JdkDynamicAopProxy

基于JDK {Proxy动态代理}的Spring AOP框架的基于JDK的{ AopProxy}实现。

创建一个动态代理，实现由AopProxy公开的接口。动态代理不能用于代理在类中定义的方法，只能是接口

如果基础（目标）类是线程安全的，则使用此类创建的代理将是线程安全的

#### getProxy(classLoader)

```java
final class JdkDynamicAopProxy implements AopProxy, InvocationHandler, Serializable {

@Override
public Object getProxy(@Nullable ClassLoader classLoader) {
	Class<?>[] proxiedInterfaces = AopProxyUtils.completeProxiedInterfaces(this.advised, true);
   findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
   return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```



## CglibAopProxy

Spring AOP框架的基于CGLIB的{AopProxy}实现。

这种类型的对象应通过由{ AdvisedSupport}对象配置的代理工厂获得。此类是Spring AOP框架的内部，无需直接由客户端代码使用。

DefaultAopProxyFactory会在必要时自动创建基于CGLIB的代理，例如在代理目标类的情况下（有关详细信息，请参见{@link DefaultAopProxyFactory助理javadoc}）。

#### getProxy(classLoader)

```java
class CglibAopProxy implements AopProxy, Serializable {
	@Override
	public Object getProxy(@Nullable ClassLoader classLoader) {
		try {
			Class<?> rootClass = this.advised.getTargetClass();

			Class<?> proxySuperClass = rootClass;
			if (rootClass.getName().contains(ClassUtils.CGLIB_CLASS_SEPARATOR)) {
				proxySuperClass = rootClass.getSuperclass();
				Class<?>[] additionalInterfaces = rootClass.getInterfaces();
				for (Class<?> additionalInterface : additionalInterfaces) {
					this.advised.addInterface(additionalInterface);
				}
			}

			// 验证类，根据需要编写日志消息
			validateClassIfNecessary(proxySuperClass, classLoader);

			// 配置CGLIB Enhancer
			Enhancer enhancer = createEnhancer();
			if (classLoader != null) {
				enhancer.setClassLoader(classLoader);
				if (classLoader instanceof SmartClassLoader &&
						((SmartClassLoader) classLoader).isClassReloadable(proxySuperClass)) {
					enhancer.setUseCache(false);
				}
			}
			enhancer.setSuperclass(proxySuperClass);
			enhancer.setInterfaces(AopProxyUtils.completeProxiedInterfaces(this.advised));
			enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
			enhancer.setStrategy(new ClassLoaderAwareGeneratorStrategy(classLoader));

			Callback[] callbacks = getCallbacks(rootClass);
			Class<?>[] types = new Class<?>[callbacks.length];
			for (int x = 0; x < types.length; x++) {
				types[x] = callbacks[x].getClass();
			}
			// 仅在上述getCallbacks调用之后，此时才会填充fixedInterceptorMap
			enhancer.setCallbackFilter(new ProxyCallbackFilter(
					this.advised.getConfigurationOnlyCopy(), this.fixedInterceptorMap, this.fixedInterceptorOffset));
			enhancer.setCallbackTypes(types);

			// 生成代理类并创建代理实例
			return createProxyClassAndInstance(enhancer, callbacks);
		}

	}

```





# JDK动态代理

　代理模式是一种很常见的模式，本文主要分析jdk动态代理的过程

## 举例　

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
public class ProxyFactory implements InvocationHandler {

    private Class target;

    public <T>T getProxy(Class<T> c)  {
        this.target = c;
        return (T)Proxy.newProxyInstance(c.getClassLoader(),c.isInterface()?new Class[]{c}:c.getInterfaces(),this);
    }


    @Override
    public Object invoke(Object proxy , Method method , Object[] args) throws Throwable {
        System.out.println("代理执行执行");
        if ( !target.isInterface() ){
            method.invoke(target.newInstance(),args);
        }
        return "代理返回值";
    }

    public static void main(String[] args) {
        // 保存生成的代理类的字节码文件
        System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");
        ProxyFactory proxyFactory = new ProxyFactory();
        IProx proxyImpl = proxyFactory.getProxy(ProxImpl.class);
        String result = proxyImpl.hello("hello word");
        System.out.println(result);
        System.out.println("---------");
        IProx proxy = proxyFactory.getProxy(IProx.class);
        result = proxy.hello("hello word");
        System.out.println(result);
    }


}

interface IProx{
    String hello(String id);
}
class ProxImpl implements IProx{

    @Override
    public String hello(String id) {
        System.out.println(id);
        return null;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

执行main方法后结果如下

![img](https://img2018.cnblogs.com/blog/1201110/201912/1201110-20191205113252986-502728287.png)

 

 



可以看到定义的hello方法已经被执行，并且可以在不定义接口的实现类的时候仍然可以执行方法获取结果，这其实就很容易想到mybatis中直接调用mapper接口获取查询结果其实也是调用的mapper的动态代理类，说明动态代理对于构造框架有很重要的作用

 

## 原理解析

### 1.Proxy.newProxyInstance方法

我们可以看到构造代理类的核心方法为这句 三个参数分别为代理类的类加载器，代理类的所有接口，如果本身就是接口则直接传入本身，传入InvocationHandler接口的实现类

```
Proxy.newProxyInstance(c.getClassLoader(),c.isInterface()?new Class[]{c}:c.getInterfaces(),this);
```

直接进入方法中查看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)throws IllegalArgumentException{
        //InvocationHandler不能为null
        Objects.requireNonNull(h);
    　　 //克隆出所有传入的接口数组
        final Class<?>[] intfs = interfaces.clone();
        
        //权限校验
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * 查找或者生成代理类  核心逻辑  参数为类加载器和上面的接口数组
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /************************下面的代码逻辑先不用管 到后面会专门分析***************************/
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            //拿到其构造方法
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            //通过构造方法新建类的实例并返回
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　

可以看到生成代理类的逻辑主要为 下面，

//查找或者生成代理类 核心逻辑 参数为类加载器和上面的接口数组

Class<?> cl = getProxyClass0(loader, intfs);

### 2.getProxyClass0方法

继续进入方法getProxyClass0(loader,intfs)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```java
    private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }
        return proxyClassCache.get(loader, interfaces);
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

可以看到方法比较简单，主要对接口数量做了个判断，然后通过缓存proxyClassCache获取代理类，这儿注意proxyClassCache在初始化时已经初始化了一个KeyFactory和ProxyClassFactory

这个后面创建代理类时将会用到

```
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
```

 

我们继续查看proxyClassCache的get方法

 

### 3.WeakCache的get方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    // key为加载器  P为接口数组
    public V get(K key, P parameter) {
        //接口不能为空
        Objects.requireNonNull(parameter);
        //清理旧的缓存
        expungeStaleEntries();
        //构造缓存的ClassLoader key
        Object cacheKey = CacheKey.valueOf(key, refQueue);

        //通过缓存的map查找
        ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
        if (valuesMap == null) {
            //如果值为空  则put一个新的空值进去 
            ConcurrentMap<Object, Supplier<V>> oldValuesMap
                = map.putIfAbsent(cacheKey,
                                  valuesMap = new ConcurrentHashMap<>());
            //再次确认 应该是防止并发状况如果已经有值则将valuesMap赋值为已有的值
            if (oldValuesMap != null) {
                valuesMap = oldValuesMap;
            }
        }

        //拿到该代理类的classKey
        Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
        //通过代理类的key查找对应的缓存Supplier
        Supplier<V> supplier = valuesMap.get(subKey);
        //factory为supplier的实现类
        Factory factory = null;

        while (true) {
            //如果Supplier不为空
            if (supplier != null) {
                //直接返回值
                V value = supplier.get();
                if (value != null) {
                    return value;
                }
            }
            //如果代理类创建工厂为空
            if (factory == null) {
                //则新建一个创建工厂
                factory = new Factory(key, parameter, subKey, valuesMap);
            }
            //如果supplier为空
            if (supplier == null) {
                //将当前的代理类key   以及新建的Factory存到缓存里面去
                supplier = valuesMap.putIfAbsent(subKey, factory);
                if (supplier == null) {
                    // 这个时候将supplier设置为factory
                    supplier = factory;
                }
                //如果supplier不为空  则用这个supplier替代之前的代理类key的值
            } else {
                if (valuesMap.replace(subKey, supplier, factory)) {
                    //将supplier设置为factory实现类
                    supplier = factory;
                } else {
                    //没有替换成功则直接返回缓存里面有的
                    supplier = valuesMap.get(subKey);
                }
            }
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 4.Factory的get方法

　通过上面的逻辑可以得知 最后是新建了Factory factory = new Factory(key, parameter, subKey, valuesMap); 并将这个factory赋值给Suplier调用get方法返回构造的代理类。所以我们直接看Factory的get方法即可

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
        @Override
        public synchronized V get() { // serialize access
            // //即再次确认 对应的类代理key的supplier有没有被更改
            Supplier<V> supplier = valuesMap.get(subKey);
            //如果被更改了则返回null
            if (supplier != this) {
                return null;
            }
            
            V value = null;
            try {
                //创建value
                value = Objects.requireNonNull(valueFactory.apply(key, parameter));
            } finally {
                if (value == null) { 
                    //如果value为null  则代表当前的supplier有问题 所以直接移除
                    valuesMap.remove(subKey, this);
                }
            }
            //再次确认value即返回的代理类不能为null
            assert value != null;

            // 将返回的代理类包装为一个缓存
            CacheValue<V> cacheValue = new CacheValue<>(value);

            // 将这个包装缓存存入缓存map
            reverseMap.put(cacheValue, Boolean.TRUE);

            // 替换当前代理类key的supplier为刚包装的缓存 后面则可以直接调用缓存  必须成功
            if (!valuesMap.replace(subKey, this, cacheValue)) {
                throw new AssertionError("Should not reach here");
            }
            return value;
        }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　该方法的主要作用则是通过valueFactory创建代理类后 将代理类包装为CacheValue(注意该类实现了Supplier接口)并将valuesMap缓存中对应代理类的Supplier替换为包装后的CacheValue,这样后面就可以直接调用CacheValue的get方法来获取代理类

　　接下来我们开始分析valueFactory.apply(key,parameters),这儿注意上面在初始化weakCache时已经讲到，在构造函数中传入了两个参数，new KeyFactory(), new ProxyClassFactory(),分别对应subKeyFactory和valueFactory，所以这里的valueFactory则代表ProxyClassFactory，所以我们直接看ProxyClassFactory的apply方法逻辑

　

### 5.ProxyClassFactory的apply方法

　

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 private static final class ProxyClassFactory implements BiFunction<ClassLoader, Class<?>[], Class<?>>{
        // 代理类名前缀
        private static final String proxyClassNamePrefix = "$Proxy";

        // 原子long  用来做代理类编号
        private static final AtomicLong nextUniqueNumber = new AtomicLong();

        @Override
        public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
            //构建一个允许相同值的key的map
            Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
            //遍历代理类的接口
            for (Class<?> intf : interfaces) {
                
                Class<?> interfaceClass = null;
                try {
                    //根据接口的名字以及传入的类加载器来构建一个class
                    interfaceClass = Class.forName(intf.getName(), false, loader);
                } catch (ClassNotFoundException e) {
                }
                if (interfaceClass != intf) {
                    //如果不同 则说明传入的类加载器和指定代理方法时的类加载器不是同一个加载器
                    //，根据双亲委派机制和jvm规定，只有同样的类加载器加载出来的一个类  才确保
                    //为同一个类型   即instance of 为true
                    throw new IllegalArgumentException(
                        intf + " is not visible from class loader");
                }
                //确认传入的接口class数组没有混入奇怪的东西
                if (!interfaceClass.isInterface()) {
                    throw new IllegalArgumentException(
                        interfaceClass.getName() + " is not an interface");
                }
                //将接口存储上面的interfaceSet中
                if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                    throw new IllegalArgumentException(
                        "repeated interface: " + interfaceClass.getName());
                }
            }
            //声明代理类的包
            String proxyPkg = null;
            //设置生成代理类的修饰级别  目前是public final
            int accessFlags = Modifier.PUBLIC | Modifier.FINAL;
            
            for (Class<?> intf : interfaces) {
                //获取class修饰符魔数 
                //这儿接口正常获取到的值为1536 (INTERFACE 512  +  ABSTRACT1024)
                int flags = intf.getModifiers();
                //如果为接口则为true
                if (!Modifier.isPublic(flags)) {
                    //将accessFlags设置为final
                    accessFlags = Modifier.FINAL;
                    //获取接口名
                    String name = intf.getName();
                    
                    int n = name.lastIndexOf('.');
                    //再获取到包名
                    String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                    if (proxyPkg == null) {
                        //如果代理包名为空 则直接为这个接口的包  这儿是循环获取的  所以最后得到的包名是最后一个接口的包
                        proxyPkg = pkg;
                    } else if (!pkg.equals(proxyPkg)) {
                        throw new IllegalArgumentException(
                            "non-public interfaces from different packages");
                    }
                }
            }
            //如果包获取到的为空  则用系统提供的
            if (proxyPkg == null) {
                // com.sun.proxy.
                proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
            }

            //获得一个编号( 线程安全)
            long num = nextUniqueNumber.getAndIncrement();
            //构建代理类的名字  例如  com.sun.proxy.$Proxy0
            String proxyName = proxyPkg + proxyClassNamePrefix + num;

            //生成代理类的字节码
            byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
                proxyName, interfaces, accessFlags);
            try {
                //根据字节码生成代理类
                return defineClass0(loader, proxyName,
                                    proxyClassFile, 0, proxyClassFile.length);
            } catch (ClassFormatError e) {
               
                throw new IllegalArgumentException(e.toString());
            }
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　该方法的逻辑则主要得到代理类的包名（一般来说为最后一个接口的包名），以及产生字节码文件 ，根据字节码文件生成代理类class并返回，核心的两个方法为ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags); 以及defineClass0(loader, proxyName,proxyClassFile, 0, proxyClassFile.length);

 

### 6.generateProxyClass方法

　　我们先看字节码生成的方法，ProxyGenerator.generateProxyClass(proxyName, interfaces, accessFlags); 后面的类不开放源代码，可以使用idea自带的反编译工具查看，我将

参数名字做了些修改 （汗  。。不然var1 var2看的属实难受 ）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static byte[] generateProxyClass(final String proxyName, Class<?>[] interfaces, int accessFlags) {
        //生成一个生成器实例   并赋予对应的属性(包名，接口数组，修饰符)
        ProxyGenerator generator = new ProxyGenerator(proxyName, interfaces, accessFlags);
        //生成数组
        final byte[] resultByte = generator.generateClassFile();
        
        //如果saveGeneratedFiles为true  则生成本地文件  这儿不讲解 有兴趣的可以研究
        if (saveGeneratedFiles) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    try {
                        int interfaces = proxyName.lastIndexOf(46);
                        Path accessFlags;
                        if (interfaces > 0) {
                            Path generator = Paths.get(proxyName.substring(0, interfaces).replace('.', File.separatorChar));
                            Files.createDirectories(generator);
                            accessFlags = generator.resolve(proxyName.substring(interfaces + 1, proxyName.length()) + ".class");
                        } else {
                            accessFlags = Paths.get(proxyName + ".class");
                        }

                        Files.write(accessFlags, resultByte, new OpenOption[0]);
                        return null;
                    } catch (IOException resultBytex) {
                        throw new InternalError("I/O exception saving generated file: " + resultBytex);
                    }
                }
            });
        }

        return resultByte;
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　该方法主要声明一个生成器，然后调用generateClassFile方法生成byte[]数组，后面的可选则生成本地文件，这个就是一开始main方法中的System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true"); 当然也可以在java启动参数加上-D传入参数，我们主要看generateClassFile()方法

 

### 7.generateClassFile方法

该方法为核心生成方法 每步均有说明，代码后会附带一张java字节码文件构造，可以参照

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private byte[] generateClassFile() {
        //Map<String, List<ProxyGenerator.ProxyMethod>> proxyMethods = new HashMap(); 用来存储方法签名  及其对应的方法描述
        //addProxyMethod方法下面有详解
        
        
        //添加Object的hashcode方法
        this.addProxyMethod(hashCodeMethod, Object.class);
        //添加Object的equals方法
        this.addProxyMethod(equalsMethod, Object.class);
        //添加Object的toString方法
        this.addProxyMethod(toStringMethod, Object.class);
        Class[] interfaces = this.interfaces;
        //获取到接口数组的长度
        int interLength = interfaces.length;

        int index;
        Class in;
        for(index = 0; index < interLength; ++index) {
            in = interfaces[index];
            //迭代获取到当前接口的所有方法
            Method[] methods = in.getMethods();
            int methodLength = methods.length;
            //迭代所有方法
            for(int methodIndex = 0; methodIndex < methodLength; ++methodIndex) {
                Method m = methods[methodIndex];
                //添加方法签名与方法描述
                this.addProxyMethod(m, in);
            }
        }
        /************上面则就将所有的方法签名与方法描述都存入到了proxyMethods中********************/
        Iterator methodsIterator = this.proxyMethods.values().iterator();
        
        List meth;
        while(methodsIterator.hasNext()) {
            //迭代每个方法签名的值 （List<ProxyGenerator.ProxyMethod>）
            meth = (List)methodsIterator.next();
            //检查方法签名值一样的方法返回值是否相同
            checkReturnTypes(meth);
        }
        
        Iterator me;
        try {
            //添加构造方法  并将构造方法的二进制代码写入methodInfo中的ByteArrayOutputStream --》对应字节码文件中的有参构造
            this.methods.add(this.generateConstructor());
            //获取方法签名对应迭代器
            methodsIterator = this.proxyMethods.values().iterator();
            
            while(methodsIterator.hasNext()) {
                meth = (List)methodsIterator.next();
                me = meth.iterator();
                //获取到每个方法签名中对应描述列表的迭代器
                while(me.hasNext()) {
                    //获取到方法信息
                    ProxyGenerator.ProxyMethod proMe = (ProxyGenerator.ProxyMethod)me.next();
                    //在feilds添加方法对应参数名字 签名，以及访问修饰符的FiledInfo
                    this.fields.add(new ProxyGenerator.FieldInfo(proMe.methodFieldName, "Ljava/lang/reflect/Method;", 10));
                    //添加对应的方法  --》对应字节码文件中的代理类接口实现方法
                    this.methods.add(proMe.generateMethod());
                }
            }  
            //添加对应的静态方法  --》对应字节码文件中的static方法
            this.methods.add(this.generateStaticInitializer());
        } catch (IOException var10) {
            throw new InternalError("unexpected I/O Exception", var10);
        }
        //做一些方法和参数数量校验
        if (this.methods.size() > 65535) {
            throw new IllegalArgumentException("method limit exceeded");
        } else if (this.fields.size() > 65535) {
            throw new IllegalArgumentException("field limit exceeded");
        } else {
            //字节码常量池中的符号引用  保存类全限定名
            this.cp.getClass(dotToSlash(this.className));
            //常量池中符号引用保存继承的Proxy类的权限定名
            this.cp.getClass("java/lang/reflect/Proxy");
            //获取到所有的接口
            interfaces = this.interfaces;
            //获取到接口的长度
            interLength = interfaces.length;
            //遍历接口
            for(index = 0; index < interLength; ++index) {
                in = interfaces[index];
                //常量池中符号引用保存接口的全限定定名
                this.cp.getClass(dotToSlash(in.getName()));
            }
            //常量池符号引用中保存访问标志
            this.cp.setReadOnly();
            //创建输出流
            ByteArrayOutputStream byteOutputStream = new ByteArrayOutputStream();
            DataOutputStream outputStream = new DataOutputStream(byteOutputStream);

            try {
                //1.写入java魔数4个字节 对应16进制数据为JAVA魔数CA FE BA BE
                outputStream.writeInt(-889275714);
                //2..写入小版本号2个字节
                outputStream.writeShort(0);
                //3.写入大版本号2个字节
                outputStream.writeShort(49);
                //4.写入常量池计数2个字节 5.以及上面添加的常量池
                this.cp.write(outputStream);
                //6.设置访问标志 2个字节
                outputStream.writeShort(this.accessFlags);
                //7.设置类索引2个字节
                outputStream.writeShort(this.cp.getClass(dotToSlash(this.className)));
                //8.设置父类索引 2个字节   这儿继承Proxy类  所以只能代理接口 因为java单继承
                outputStream.writeShort(this.cp.getClass("java/lang/reflect/Proxy"));
                //9.设置接口长度 2个字节  所以前面要判断接口数量不能大于65535
                outputStream.writeShort(this.interfaces.length);
                Class[] interfacess = this.interfaces;
                int interfacelength = interfacess.length;

                for(int index = 0; index < interfacelength; ++index) {
                    Class c = interfacess[index];
                    //10.写入接口索引 2个字节
                    outputStream.writeShort(this.cp.getClass(dotToSlash(c.getName())));
                }
                //11.写入变量数量 2个字节
                outputStream.writeShort(this.fields.size());
                me = this.fields.iterator();
                
                while(me.hasNext()) {
                    ProxyGenerator.FieldInfo p = (ProxyGenerator.FieldInfo)me.next();
                    //12.写入变量
                    p.write(outputStream);
                }
                //13.写入方法数量 2个字节
                outputStream.writeShort(this.methods.size());
                me = this.methods.iterator();

                while(me.hasNext()) {
                    ProxyGenerator.MethodInfo proMe = (ProxyGenerator.MethodInfo)me.next();
                    //14.写入方法
                    proMe.write(outputStream);
                }
                //15.没有属性表  直接写0
                outputStream.writeShort(0);
                //返回一个完整class的字节码
                return byteOutputStream.toByteArray();
            } catch (IOException var9) {
                throw new InternalError("unexpected I/O Exception", var9);
            }
        }
    }
    
    
     private void addProxyMethod(Method method, Class<?> class) {
        //获取到方法名
        String methodName = method.getName();
        //获取所有方法参数类型
        Class[] methodParamTypes = method.getParameterTypes();
        //获取返回类型
        Class returnTypes = method.getReturnType();
        //获取所有的异常
        Class[] exceptions = method.getExceptionTypes();
        //方法名+方法参数类形的方法签名  用来标识唯一的方法
        String methodSign = methodName + getParameterDescriptors(methodParamTypes);
        //获取该方法的List<ProxyGenerator.ProxyMethod>
        Object me = (List)this.proxyMethods.get(methodSign);
        if (me != null) {
            //如果不为空 即出现了相同方法签名的方法
            Iterator iter = ((List)me).iterator();
            //则获得这个list的迭代器
            while(iter.hasNext()) {
                //开始迭代
                ProxyGenerator.ProxyMethod proxyMe = (ProxyGenerator.ProxyMethod)iter.next();
                //如果这个方法签名对应的方法描述中返回值也与传入的一致
                if (returnTypes == proxyMe.returnType) {
                    //则直接合并两个方法中抛出的所有异常然后直接返回
                    ArrayList methodList = new ArrayList();
                    collectCompatibleTypes(exceptions, proxyMe.exceptionTypes, methodList);
                    collectCompatibleTypes(proxyMe.exceptionTypes, exceptions, methodList);
                    proxyMe.exceptionTypes = new Class[methodList.size()];
                    proxyMe.exceptionTypes = (Class[])methodList.toArray(proxyMe.exceptionTypes);
                    return;
                }
            }
        } 
        //如果为空
        else {
            //则新建一个List
            me = new ArrayList(3);
            //将方法签名 与这个新建的List<ProxyGenerator.ProxyMethod> 方法描述 put进去
            this.proxyMethods.put(methodSign, me);
        }
        //然后再这个list中新添加一个ProxyMethod 方法描述  这个Proxymethod方法包含了传入的方法的所有特征
        ((List)me).add(new ProxyGenerator.ProxyMethod(methodName, methodParamTypes, returnTypes, exceptions, class));
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　**这个方法中涉及到两个重要概念，**一个是class字节码文件构造 参考如下 类型中u后面的数字则代表字节长度  。而上述方法中构造自字节码文件的15个步骤也是按照如下表一步一步构造的，最后没有属性表的值

![img](https://img2018.cnblogs.com/blog/1201110/201912/1201110-20191205165153176-1333623098.png)

 

 

 **另一个是常量池** ，主要放置两大类常量，一个是**字面量** 即常见的字符串，以及声明为final的变量，另一个则是语言层面的**符号引用**，包含三类常量：类和接口的全限定名，字段的名称和描述符，方法的名称和描述符

生成的字节码文件如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package cn.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

final class $Proxy0 extends Proxy implements IProx {
    
    //方法变量
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;

    //构造方法
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }
    //重写方法
    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }
    //重写方法
    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    //重写方法
    public final String hello(String var1) throws  {
        try {
            return (String)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }
    //重写方法
    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    //静态方法
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m3 = Class.forName("cn.proxy.IProx").getMethod("hello", Class.forName("java.lang.String"));
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

 

上述方法生成代理类字节码返回后则回到了 第5步中的ProxyClassFactory的apply方法 ，获取到字节码文件后 ，生成代理类class并返回，方法为

defineClass0(loader, proxyName,proxyClassFile, 0, proxyClassFile.length)  这个方法为本地方法 ，不能获取到源码，但也容易理解，就是根据类名，类加载器，字节码文件创建一个class类型

```
private static native Class<?> defineClass0(ClassLoader loader, String name,byte[] b, int off, int len);
```

defineClass0返回了代理类信息后则回到了第4步中的Factory的get()方法中，并将生成的类信息继续返回到第3步，第3步则继续返回 一直会返回到第1步

 

 

 

### 8.所以，回到第1步的代码中继续分析

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)throws IllegalArgumentException{
        //InvocationHandler不能为null
        Objects.requireNonNull(h);
    
        final Class<?>[] intfs = interfaces.clone();
        
        //权限校验
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * 查找或者生成代理类  核心逻辑  参数为类加载器和上面的接口数组
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * 获取到代理类的class类  即cl
         */
        try {
            //校验权限
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            //拿到其有参构造方法   即字节码文件中的 public $Proxy0(InvocationHandler var1)方法
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            
            //拿到我们传入的InvocationHandler
            final InvocationHandler ih = h;
            //如果类访问修饰符不是public  那么则将其设置可访问
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            //通过构造方法传入我们自己定义的InvocationHandler 新建类的实例并返回
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　获取到字节码生成的类信息后，则获取代理类的有参构造并传入我们自己一开始定义的InvocationHandler。通过构造函数创建代理类的实例并返回。所以字节码的父类文件中InvocationHandler则是我们创建的并传入的，

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
final class $Proxy0 extends Proxy implements IProx {

    ....
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }
    ....
    public final String hello(String var1) throws  {
        try {
            return (String)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
    }
}
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class Proxy implements java.io.Serializable {
    ....
    /**
     * the invocation handler for this proxy instance.
     * @serial
     */
    protected InvocationHandler h;
    ....
    protected Proxy(InvocationHandler h) {
        Objects.requireNonNull(h);
        this.h = h;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　由上述代码可以得知 代理类在执行hello方法时 实际上是通过我们传入的InvocationHandler的invoke方法进行调用。到此 jdk动态代理的分析以及全部完成

 

## 结论

至此可以得知 jdk动态代理的大致逻辑即是

​	传入代理类 类加载器，与接口数组和自定义的InvocationHandler，然后通过分析接口信息生成java文件的字节码数据，然后调用本地方法将类加载到内存中，最后返回构造参数为InvocationHandler的代理类，该类实现代理接口，并继承Proxy类（所以jdk动态代理只能代理接口，java单继承），我们调用方法实际上是调用代理类的方法，代理类则可以通过我们传入的InvocationHandler反射调用原本的方法来实现无侵入的修改原有方法逻辑

https://www.cnblogs.com/hetutu-5238/p/11988946.html

**其中真实对象接口和它的实现跟静态代理是一样的。****

  不同的是代理类的创建，静态代理是直接新增一个代理类，而动态代理则不是。动态代理是通过JDK的Proxy和一个调用处理器InvocationHandler来实现的，通过Proxy来生成代理类实例，而这个代理实例通过调用处理器InvocationHandler接收不同的参数灵活调用真实对象的方法。

 所以我们需要做的是创建调用处理器，该调用处理器必须实现JDK的InvocationHandler



# cglib动态代理

代理模式是一种很常见的模式，本文主要分析cglib动态代理的过程

## 举例

使用cglib代理需要引入两个包，maven的话包引入如下

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
        <!-- https://mvnrepository.com/artifact/cglib/cglib -->
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.3.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.ow2.asm/asm -->
        <dependency>
            <groupId>org.ow2.asm</groupId>
            <artifactId>asm</artifactId>
            <version>7.1</version>
        </dependency>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

示例代码

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
import net.sf.cglib.core.DebuggingClassWriter;
import net.sf.cglib.core.KeyFactory;
import net.sf.cglib.proxy.Callback;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @Description:
 * @author: zhoum
 * @Date: 2019-12-05
 * @Time: 9:36
 */
public class CgProxyFactory implements MethodInterceptor {



    public <T>T getProxy(Class<T> c){
        Enhancer enhancer = new Enhancer();
        enhancer.setCallbacks(new Callback[]{this});
        enhancer.setSuperclass(c);
        return (T)enhancer.create();
    }

    @Override
    public Object intercept(Object obj , Method method , Object[] args , MethodProxy proxy) throws Throwable {
        System.out.println("执行前-------->");
        Object o = proxy.invokeSuper(obj , args);
        System.out.println("执行后-------->");
        return o;
    }

    public static void main(String[] args) {
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "F:/DEBUG");
        CgProxyFactory factory = new CgProxyFactory();
        CgProxy proxy = factory.getProxy(CgProxy.class);
        proxy.say();
    }
}

class CgProxy{

    public String say(){
        System.out.println("普通方法执行");
        return "123";
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

控制台输出结果，可以看到方法已经被代理增强了

![img](https://img2018.cnblogs.com/blog/1201110/201912/1201110-20191206162013317-751212002.png)

 

 

 

## 原理解析

### 1.Enhancer.create()方法

 

　　通过上面代码，相信大家都能知道主要创建代理类的方法为Enhancer.create()方法，但是我们在执行这个方法之前设置了两个值，可以分别看下方法体

　　setCallbacks()，即设置回调，我们创建出代理类后调用方法则是使用的这个回调接口，类似于jdk动态代理中的InvocationHandler

```
public void setCallbacks(Callback[] callbacks) {
        if (callbacks != null && callbacks.length == 0) {
            throw new IllegalArgumentException("Array cannot be empty");
        }
        this.callbacks = callbacks;
    }
```

　　setSuperClass 即设置代理类，这儿可以看到做了个判断，如果为interface则设置interfaces,如果是Object则设置为null(因为所有类都自动继承Object),如果为普通class则设置class，可以看到cglib代理不光可以代理接口，也可以代理普通类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public void setSuperclass(Class superclass) {
        if (superclass != null && superclass.isInterface()) {
　　　　　　　　//设置代理接口
            setInterfaces(new Class[]{ superclass });
        } else if (superclass != null && superclass.equals(Object.class)) {
            // 未Object则用设置
            this.superclass = null;
        } else {
　　　　　　 //设置代理类
            this.superclass = superclass;
        }
    }
public void setInterfaces(Class[] interfaces) {
    this.interfaces = interfaces;
}
 
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　上面总结看来主要设置两个我们需要用到的信息

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Object create() {
        //不作代理类限制
        classOnly = false;
        //没有构造参数类型
        argumentTypes = null;
        //执行创建
        return createHelper();
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

### 2.Enhancer.createHelper()方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object createHelper() { 
    //进行验证 并确定CallBack类型 本方法是用的MethodInterceptor
    preValidate();
    
    //获取当前代理类的标识类Enhancer.EnhancerKey的代理
    Object key = KEY_FACTORY.newInstance((superclass != null) ? superclass.getName() : null,
            ReflectUtils.getNames(interfaces),
            filter == ALL_ZERO ? null : new WeakCacheKey<CallbackFilter>(filter),
            callbackTypes,
            useFactory,
            interceptDuringConstruction,
            serialVersionUID);
    //设置当前enhancer的代理类的key标识
    this.currentKey = key;
    //调用父类即 AbstractClassGenerator的创建代理类
    Object result = super.create(key);
    return result;
}

private void preValidate() {
    if (callbackTypes == null) {
        //确定传入的callback类型
        callbackTypes = CallbackInfo.determineTypes(callbacks, false);
        validateCallbackTypes = true;
    }
    if (filter == null) {
        if (callbackTypes.length > 1) {
            throw new IllegalStateException("Multiple callback types possible but no filter specified");
        }
        filter = ALL_ZERO;
    }
}

    //最后是遍历这个数组来确定  本方法是用的MethodInterceptor
    private static final CallbackInfo[] CALLBACKS = {
        new CallbackInfo(NoOp.class, NoOpGenerator.INSTANCE),
        new CallbackInfo(MethodInterceptor.class, MethodInterceptorGenerator.INSTANCE),
        new CallbackInfo(InvocationHandler.class, InvocationHandlerGenerator.INSTANCE),
        new CallbackInfo(LazyLoader.class, LazyLoaderGenerator.INSTANCE),
        new CallbackInfo(Dispatcher.class, DispatcherGenerator.INSTANCE),
        new CallbackInfo(FixedValue.class, FixedValueGenerator.INSTANCE),
        new CallbackInfo(ProxyRefDispatcher.class, DispatcherGenerator.PROXY_REF_INSTANCE),
    };
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　方法主要做了下验证并确定Callback类型，我们使用的是MethodIntercepter。然后创建当前代理类的标识代理类，用这个标识代理类调用父类(AbstractClassGenerator)的create(key方法创建)，我们主要分析下标识代理类创建的逻辑和后面父类创建我们需要的代理类逻辑。

　　标识代理类的创建类成员变量即KEY_FACTORY是创建代理类的核心，我们先分析下这个

### 3.1KEY_FACTORY

　　追踪源码可以看到，KEY_FACTORY在Enhancer的初始化即会创建一个final的静态变量

```
    private static final EnhancerKey KEY_FACTORY =
      (EnhancerKey)KeyFactory.create(EnhancerKey.class, KeyFactory.HASH_ASM_TYPE, null);
```

　

### 3.2Keyfactory_create方法

　这儿可以看到使用key工厂创建出对应class的代理类，后面的KeyFactory_HASH_ASM_TYPE即代理类中创建HashCode方法的策略。我们接着点击源码查看

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public static KeyFactory create(ClassLoader loader, Class keyInterface, KeyFactoryCustomizer customizer,
                                    List<KeyFactoryCustomizer> next) {
        //创建一个最简易的代理类生成器 即只会生成HashCode equals toString newInstance方法
        Generator gen = new Generator();
        //设置接口为enhancerKey类型
        gen.setInterface(keyInterface);

        if (customizer != null) {
            //添加定制器
            gen.addCustomizer(customizer);
        }
        if (next != null && !next.isEmpty()) {
            for (KeyFactoryCustomizer keyFactoryCustomizer : next) {
                //添加定制器
                gen.addCustomizer(keyFactoryCustomizer);
            }
        }
        //设置生成器的类加载器
        gen.setClassLoader(loader);
        //生成enhancerKey的代理类
        return gen.create();
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 3.3Generator的create方法

　　这儿创建了一个简易的代理类生成器(KeyFactory的内部类Generator ，与Enhancer一样继承自抽象类AbstractClassGenerator)来生成我们需要的标识代理类，我们接着看gen.create()方法

```
public KeyFactory create() {
　　　　　　　　//设置了该生成器生成代理类的名字前缀，即我们的接口名Enhancer.enhancerKey
            setNamePrefix(keyInterface.getName());
            return (KeyFactory)super.create(keyInterface.getName());
        }
```

　　这儿可以看到调用的是父类AbstractClassGenerator的create方法，参数名为接口名

### 3.4 AbstractClassGenerator的create(Key)方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
protected Object create(Object key) {
        try {
            //获取到当前生成器的类加载器
            ClassLoader loader = getClassLoader();
            //当前类加载器对应的缓存  缓存key为类加载器，缓存的value为ClassLoaderData  这个类后面会再讲
            Map<ClassLoader, ClassLoaderData> cache = CACHE;
            //先从缓存中获取下当前类加载器所有加载过的类
            ClassLoaderData data = cache.get(loader);
            //如果为空
            if (data == null) {
                synchronized (AbstractClassGenerator.class) {
                    cache = CACHE;
                    data = cache.get(loader);
                    //经典的防止并发修改 二次判断
                    if (data == null) {
                        //新建一个缓存Cache  并将之前的缓存Cache的数据添加进来 并将已经被gc回收的数据给清除掉
                        Map<ClassLoader, ClassLoaderData> newCache = new WeakHashMap<ClassLoader, ClassLoaderData>(cache);
                        //新建一个当前加载器对应的ClassLoaderData 并加到缓存中  但ClassLoaderData中此时还没有数据
                        data = new ClassLoaderData(loader);
                        newCache.put(loader, data);
                        //刷新全局缓存
                        CACHE = newCache;
                    }
                }
            }
            //设置一个全局key 
            this.key = key;
            
            //在刚创建的data(ClassLoaderData)中调用get方法 并将当前生成器，
            //以及是否使用缓存的标识穿进去 系统参数 System.getProperty("cglib.useCache", "true")  
            //返回的是生成好的代理类的class信息
            Object obj = data.get(this, getUseCache());
            //如果为class则实例化class并返回  就是我们需要的代理类
            if (obj instanceof Class) {
                return firstInstance((Class) obj);
            }
            //如果不是则说明是实体  则直接执行另一个方法返回实体
            return nextInstance(obj);
        } catch (RuntimeException e) {
            throw e;
        } catch (Error e) {
            throw e;
        } catch (Exception e) {
            throw new CodeGenerationException(e);
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这个方法可以看到主要为根据类加载器定义一个缓存，里面装载了缓存的类信息，然后调用这个ClassLoaderData的get方法获取到数据，如果为class信息 那么直接使用反射实例化，如果返回的是实体类，则解析实体类的信息，调用其newInstance方法重新生成一个实例(cglib的代理类都会生成newInstance方法)

　　

　　具体根据class信息或者实体信息实例化数据都比较简单，相信代码大家都能看懂 ，这儿就不讲解了。核心点还是在于如何根据生成器来返回代理类或者代理类信息，我们继续查看data.get(this,getUseCache)

 

### 3.5data.get(this,getUseCache)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public Object get(AbstractClassGenerator gen, boolean useCache) {
            //如果不用缓存  (默认使用)
            if (!useCache) {
                //则直接调用生成器的命令
              return gen.generate(ClassLoaderData.this);
            } else {
              //从缓存中获取值
              Object cachedValue = generatedClasses.get(gen);
              //解包装并返回
              return gen.unwrapCachedValue(cachedValue);
            }
        }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这儿可以看到 如果可以用缓存 则调用缓存，不能调用缓存则直接生成， 这儿我们先看调用缓存的，在看之前需要再看一个东西，就是3.4之中，我们设置了一个key为ClassLoader，值为ClassLoaderData的缓存，这儿我们new了一个ClassLoaderData 并将类加载器传了进去 ，并且设置了这个Generator的key，我们看下new的逻辑

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public ClassLoaderData(ClassLoader classLoader) {
            //判断类加载器不能为空 
            if (classLoader == null) {
                throw new IllegalArgumentException("classLoader == null is not yet supported");
            }
            //设置类加载器   弱引用 即在下次垃圾回收时就会进行回收
            this.classLoader = new WeakReference<ClassLoader>(classLoader);
            //新建一个回调函数  这个回调函数的作用在于缓存中没获取到值时  调用传入的生成的生成代理类并返回
            Function<AbstractClassGenerator, Object> load =
                    new Function<AbstractClassGenerator, Object>() {
                        public Object apply(AbstractClassGenerator gen) {
                            Class klass = gen.generate(ClassLoaderData.this);
                            return gen.wrapCachedClass(klass);
                        }
                    };
            //为这个ClassLoadData新建一个缓存类   这个loadingcache稍后会讲        
            generatedClasses = new LoadingCache<AbstractClassGenerator, Object, Object>(GET_KEY, load);
        }
        
        
 private static final Function<AbstractClassGenerator, Object> GET_KEY = new Function<AbstractClassGenerator, Object>() {
     public Object apply(AbstractClassGenerator gen) {
         return gen.key;
     }
};
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 　可以看到每个类加载器都对应着一个代理类缓存对象 ，这里面定义了类加载器，缓存调用没查询到的调用函数，以及新建了一个LoadingCache来缓存这个类加载器对应的缓存，这儿传入的两个参数，load代表缓存查询失败时的回调函数，而GET_KEY则是回调时获取调用生成器的key 即3.4中传入的key 也即是我们的代理类标识符。 然后我们接着看generatedClasses.get(gen);的方法

### 3.6 generatedClasses.get(gen);

　　这个方法主要传入代理类生成器 并根据代理类生成器获取值返回。这儿主要涉及到的类就是LoadingCache，这个类可以看做是某个CLassLoader对应的所有代理类缓存库，是真正缓存东西的地方。我们分析下这个类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
package net.sf.cglib.core.internal;

import java.util.concurrent.*;

public class LoadingCache<K, KK, V> {
    protected final ConcurrentMap<KK, Object> map;
    protected final Function<K, V> loader;
    protected final Function<K, KK> keyMapper;

    public static final Function IDENTITY = new Function() {
        public Object apply(Object key) {
            return key;
        }
    };
    //初始化类  kemapper代表获取某个代理类生成器的标识，loader即缓存查找失败后的回调函数
    public LoadingCache(Function<K, KK> keyMapper, Function<K, V> loader) {
        this.keyMapper = keyMapper;
        this.loader = loader;
        //这个map是缓存代理类的地方
        this.map = new ConcurrentHashMap<KK, Object>();
    }

    @SuppressWarnings("unchecked")
    public static <K> Function<K, K> identity() {
        return IDENTITY;
    }
    //这儿key是代理类生成器
    public V get(K key) {
        //获取到代理类生成器的标识
        final KK cacheKey = keyMapper.apply(key);
        //根据缓代理类生成器的标识获取代理类
        Object v = map.get(cacheKey);
        //如果结果不为空且不是FutureTask 即线程池中用于获取返回结果的接口
        if (v != null && !(v instanceof FutureTask)) {
            //直接返回
            return (V) v;
        }
        //否则就是没查询到  或者还未处理完
        return createEntry(key, cacheKey, v);
    }

    protected V createEntry(final K key, KK cacheKey, Object v) {
        //初始化任务task
        FutureTask<V> task;
        //初始化创建标识
        boolean creator = false;
        if (v != null) {
            // 则说明这是一个FutureTask 
            task = (FutureTask<V>) v;
        } else {
            //否则还没开始创建这个代理类  直接创建任务  
            task = new FutureTask<V>(new Callable<V>() {
                public V call() throws Exception {
                    //这儿会直接调用生成器的generate方法
                    return loader.apply(key);
                }
            });
            //将这个任务推入缓存Map  如果对应key已经有则返回已经有的task，
            Object prevTask = map.putIfAbsent(cacheKey, task);
            //如果为null则代表还没有创建  标识更新为true 且运行这个任务
            if (prevTask == null) {
                // creator does the load
                creator = true;
                task.run();
            } 
            //如果是task  说明另一个线程已经创建了task
            else if (prevTask instanceof FutureTask) {
                task = (FutureTask<V>) prevTask;
            } 
            //到这儿说明另一个线程已经执行完了  直接返回
            else {
                return (V) prevTask;
            }
            
            //上面的一堆判断主要是为了防止并发出现的问题
        }
        
        V result;
        try {
            //到这儿说明任务执行完并拿到对应的代理类了
            result = task.get();
        } catch (InterruptedException e) {
            throw new IllegalStateException("Interrupted while loading cache item", e);
        } catch (ExecutionException e) {
            Throwable cause = e.getCause();
            if (cause instanceof RuntimeException) {
                throw ((RuntimeException) cause);
            }
            throw new IllegalStateException("Unable to load cache item", cause);
        }
        //如果这次执行是新建的
        if (creator) {
            //将之前的FutureTask缓存直接覆盖为实际的代理类信息
            map.put(cacheKey, result);
        }
        //返回结果
        return result;
    }
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　通过上面的分析可以得知，这个类主要作用是传入代理类生成器，根据这个代理类生成器以及代理类生成器的key来获取缓存，如果没有获取到则构建一个FutureTask来回调我们之前初始化时传入的 回调函数，并调用其中的apply方法，而具体调用的则是我们传入的代理类生成器的generate(LoadClassData)方法，将返回值覆盖之前的FutureTask成为真正的缓存。所以这个类的主要作用还是缓存。 这样则和3.5中不使用缓存时调用了一样的方法。所以我们接着来分析生成方法 generate(ClassLoadData),这儿因为我们使用的代理类生成器是Genrator，该类没有重写generate方法，所以回到了父类AbstractClassGenerator的generate方法

 

### 3.7 AbstractClassGenerator.generate 方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    protected Class generate(ClassLoaderData data) {
        Class gen;
        Object save = CURRENT.get();
        //当前的代理类生成器存入ThreadLocal中
        CURRENT.set(this);
        try {
            //获取到ClassLoader
            ClassLoader classLoader = data.getClassLoader();
            //判断不能为空
            if (classLoader == null) {
                throw new IllegalStateException("ClassLoader is null while trying to define class " +
                        getClassName() + ". It seems that the loader has been expired from a weak reference somehow. " +
                        "Please file an issue at cglib's issue tracker.");
            }
            synchronized (classLoader) {
             //生成代理类名字
              String name = generateClassName(data.getUniqueNamePredicate()); 
             //缓存中存入这个名字
              data.reserveName(name);
              //当前代理类生成器设置类名
              this.setClassName(name);
            }
            //尝试从缓存中获取类
            if (attemptLoad) {
                try {
                    //要是能获取到就直接返回了  即可能出现并发 其他线程已经加载
                    gen = classLoader.loadClass(getClassName());
                    return gen;
                } catch (ClassNotFoundException e) {
                    // 发现异常说明没加载到 不管了
                }
            }
            //生成字节码
            byte[] b = strategy.generate(this);
            //获取到字节码代表的class的名字
            String className = ClassNameReader.getClassName(new ClassReader(b));
            //核实是否为protect
            ProtectionDomain protectionDomain = getProtectionDomain();
            synchronized (classLoader) { // just in case
                //如果不是protect
                if (protectionDomain == null) {
                    //根据字节码 类加载器 以及类名字  将class加载到内存中
                    gen = ReflectUtils.defineClass(className, b, classLoader);
                } else {
                    //根据字节码 类加载器 以及类名字 以及找到的Protect级别的实体 将class加载到内存中
                    gen = ReflectUtils.defineClass(className, b, classLoader, protectionDomain);
                }
            }
　　　　　　　 //返回生成的class信息
            return gen;
        } catch (RuntimeException e) {
            throw e;
        } catch (Error e) {
            throw e;
        } catch (Exception e) {
            throw new CodeGenerationException(e);
        } finally {
            CURRENT.set(save);
        }
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这个方法主要设置了下当前类生成器的类名，然后调用stratege的generate方法返回字节码，根据字节码 类名 类加载器将字节码所代表的类加载到内存中，这个功能看一下大概就懂，我们接下来主要分析字节码生成方法

### 3.8DefaultGeneratorStrategy.generate(ClassGenerator cg)

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public byte[] generate(ClassGenerator cg) throws Exception {
        //创建一个写入器
        DebuggingClassWriter cw = getClassVisitor();
        //加入自己的转换逻辑后执行代理类生成器的generateClass方法
        transform(cg).generateClass(cw);
        //将cw写入的东西转换为byte数组返回
        return transform(cw.toByteArray());
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这里面主要是新建一个写入器，然后执行我们代理类生成器的generateClass方法将class信息写入这个ClassWriter 最后将里面的东西转换为byte数组返回，所以又回到了我们的代理类生成器的generateClass方法，这儿进入的是Generator的generateClass方法

 

### 3.9Generator.generateClass（ClassVisitor v）

　　　　这个类是核心的class信息写入类

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 //该方法为字节码写入方法 为最后一步
 public void generateClass(ClassVisitor v) {
            //创建类写入聚合对象
            ClassEmitter ce = new ClassEmitter(v);
            //找到被代理类的newInstance方法 如果没有会报异常  由此可知 如果想用Generator代理类生成器  必须要有newInstance方法
            Method newInstance = ReflectUtils.findNewInstance(keyInterface);
            //如果被代理类的newInstance不为Object则报异常  此处我们代理的Enchaer.EnhancerKey newInstance方法返回值为Object
            if (!newInstance.getReturnType().equals(Object.class)) {
                throw new IllegalArgumentException("newInstance method must return Object");
            }
            //找到newInstance方法的所有参数类型 并当做成员变量 
            Type[] parameterTypes = TypeUtils.getTypes(newInstance.getParameterTypes());
            
            //1.创建类开始写入类头   版本号  访问权限  类名等通用信息
            ce.begin_class(Constants.V1_8,
                           Constants.ACC_PUBLIC,
                           getClassName(), 
                           KEY_FACTORY,
                           new Type[]{ Type.getType(keyInterface) },
                           Constants.SOURCE_FILE);
            //2.写入无参构造方法               
            EmitUtils.null_constructor(ce);
            //3.写入newInstance方法
            EmitUtils.factory_method(ce, ReflectUtils.getSignature(newInstance));
            
            int seed = 0;
            //4.开始构造 有参构造方法
            CodeEmitter e = ce.begin_method(Constants.ACC_PUBLIC,
                                            TypeUtils.parseConstructor(parameterTypes),
                                            null);
            e.load_this();
            //4.1有参构造中调用父类构造方法  即super.构造方法() 
            e.super_invoke_constructor();
            e.load_this();
            //4.2找到传入的定制器 例如一开始传入的hashCode方法定制器
            List<FieldTypeCustomizer> fieldTypeCustomizers = getCustomizers(FieldTypeCustomizer.class);
            //4.3遍历成员变量即newInstance方法的所有参数
            for (int i = 0; i < parameterTypes.length; i++) {
                Type parameterType = parameterTypes[i];
                Type fieldType = parameterType;
                for (FieldTypeCustomizer customizer : fieldTypeCustomizers) {
                    fieldType = customizer.getOutType(i, fieldType);
                }
                seed += fieldType.hashCode();
                //4.3将这些参数全部声明到写入类中
                ce.declare_field(Constants.ACC_PRIVATE | Constants.ACC_FINAL,
                                 getFieldName(i),
                                 fieldType,
                                 null);
                e.dup();
                e.load_arg(i);
                for (FieldTypeCustomizer customizer : fieldTypeCustomizers) {
                    customizer.customize(e, i, parameterType);
                }
                //4.4设置每个成员变量的值  即我们常见的有参构造中的this.xx = xx
                e.putfield(getFieldName(i));
            }
            //设置返回值
            e.return_value();
            //有参构造及成员变量写入完成
            e.end_method();
            
            /*************************到此已经在class中写入了成员变量  写入实现了newInstance方法  写入无参构造  写入了有参构造 *************************/
            
            // 5.写入hashcode方法
            e = ce.begin_method(Constants.ACC_PUBLIC, HASH_CODE, null);
            int hc = (constant != 0) ? constant : PRIMES[(int)(Math.abs(seed) % PRIMES.length)];
            int hm = (multiplier != 0) ? multiplier : PRIMES[(int)(Math.abs(seed * 13) % PRIMES.length)];
            e.push(hc);
            for (int i = 0; i < parameterTypes.length; i++) {
                e.load_this();
                e.getfield(getFieldName(i));
                EmitUtils.hash_code(e, parameterTypes[i], hm, customizers);
            }
            e.return_value();
            //hashcode方法结束
            e.end_method();

            // 6.写入equals方法
            e = ce.begin_method(Constants.ACC_PUBLIC, EQUALS, null);
            Label fail = e.make_label();
            e.load_arg(0);
            e.instance_of_this();
            e.if_jump(e.EQ, fail);
            for (int i = 0; i < parameterTypes.length; i++) {
                e.load_this();
                e.getfield(getFieldName(i));
                e.load_arg(0);
                e.checkcast_this();
                e.getfield(getFieldName(i));
                EmitUtils.not_equals(e, parameterTypes[i], fail, customizers);
            }
            e.push(1);
            e.return_value();
            e.mark(fail);
            e.push(0);
            e.return_value();
            //equals方法结束
            e.end_method();

            // 7.写入toString方法
            e = ce.begin_method(Constants.ACC_PUBLIC, TO_STRING, null);
            e.new_instance(Constants.TYPE_STRING_BUFFER);
            e.dup();
            e.invoke_constructor(Constants.TYPE_STRING_BUFFER);
            for (int i = 0; i < parameterTypes.length; i++) {
                if (i > 0) {
                    e.push(", ");
                    e.invoke_virtual(Constants.TYPE_STRING_BUFFER, APPEND_STRING);
                }
                e.load_this();
                e.getfield(getFieldName(i));
                EmitUtils.append_string(e, parameterTypes[i], EmitUtils.DEFAULT_DELIMITERS, customizers);
            }
            e.invoke_virtual(Constants.TYPE_STRING_BUFFER, TO_STRING);
            e.return_value();
            //toString方法结束
            e.end_method();
            //类写入结束  至此类信息收集完成 并全部写入ClassVisitor
            ce.end_class();
        }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　这个方法主要将一个完整的类信息写入ClassVisitor中，例如目前实现的Enhancer.EnhancerKey代理，即实现了newInstance方法， 重写了HashCode,toSting,equals方法，并将newInstance的所有参数作为了成员变量，这儿我们也可以看下具体实现newInstance方法的逻辑 即这个代码 EmitUtils.factory_method(ce, ReflectUtils.getSignature(newInstance)); 具体内容如下  其他的写入内容大致命令也是这样，如果有兴趣可以去研究asm字节码写入的操作

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
   public static void factory_method(ClassEmitter ce, Signature sig) 
    {
        //开始写入方法
        CodeEmitter e = ce.begin_method(Constants.ACC_PUBLIC, sig, null);
        //写入 一个创建对象命令  即new命令
        e.new_instance_this();
        e.dup();
        //加载参数命令
        e.load_args();
        //执行该类的有参构造命令
        e.invoke_constructor_this(TypeUtils.parseConstructor(sig.getArgumentTypes()));
        //将上面指令执行的值返回
        e.return_value();
        //结束写入方法
        e.end_method();
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

所以通过方法构造指令

可以得知 我们通过Generator创建的代理类大致内容应该如下，Enhancer.EhancerKey代理类字节码的class内容应该是把参数换为newInstance中的参数

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class EnhancerKeyProxy extends xxx implements xxx{

 private paramA;
 private paramB;
 private paramC;
 
 public EnhancerKeyProxy() {
         super.xxx();
 }
 public EnhancerKeyProxy(paramA, paramB,paramC) {
       super.xxx();
       this.paramA = paramA
       this.paramB = paramB
       this.paramC = paramC
    }

 public Object newInstance(paramA,paramB,paramC){
        EnhancerKeyProxy param = new EnhancerKeyProxy(o);
        return param;
 }
 
  public int hashCode(){
      ...
  }
   public String toString(){
      ...
  }
  
  public boolean equals(Object o){
      ...
  }

}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　　　最后执行传入的ClassVisitor 即我们传入的实例DebuggingClassWriter的toByteArray即可以将写入的内容转换为byte[]返回

 

　　至此 我们的成功的生成了Enhancer.EnhancerKey的代理类，也就是我们需要的代理类标识类 用来标识被代理的类，这个代理类主要用来作为被代理类的标识，在进行缓存时作为判断相等的依据。可以看到 cglib代理主要也是利用我们传入的被代理类信息来生成对应的代理类字节码，然后用类加载器加载到内存中。虽然我们的实际的代理任务才刚刚开始，但是要了解的东西已经基本上差不多了，对具体的我们案例中的ProxyFactory代理时，只是生成器Enhancer对比生成器Generator在生成过程中重写了一些操作而已

 

　　现在我们重新回看步骤2中的代码

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
private Object createHelper() {
        preValidate();
        //获取到了代理类标识类
        Object key = KEY_FACTORY.newInstance((superclass != null) ? superclass.getName() : null,
                ReflectUtils.getNames(interfaces),
                filter == ALL_ZERO ? null : new WeakCacheKey<CallbackFilter>(filter),
                callbackTypes,
                useFactory,
                interceptDuringConstruction,
                serialVersionUID);
        //设置当前enhancer正在代理生成的类信息
        this.currentKey = key;
        //调用父类的create(key方法)
        Object result = super.create(key);
        return result;
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　可以看到获取到代理类标志类后 将其设置为当前代理类生成器的正在代理的类 并同样调用父类AbstractClassGenerator中create(key)的方法

　下面开始分析Ehancer生成器的逻辑，由于部分逻辑和Generator生成器一致 所以一样的我会略过

　　

### 4.AbstractClassGenerator.create方法

　　这个逻辑和步骤3.4一致，查询当前key即代理类标志类对应的ClassLoadData缓存，如果没有则建一个空的缓存并初始化一个对应的ClassLoadData,传入相应的生成器，生成失败回调函数等

　　按照同样的逻辑一直走到3.5中的generate(ClassLoadData)方法时，由于Enhancer生成器重写了这个方法 所以我们分析Enahncer的生成逻辑

### 5.Enhancer.generate(ClassLoadData data)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 @Override
    protected Class generate(ClassLoaderData data) {
        validate();
        if (superclass != null) {
            setNamePrefix(superclass.getName());
        } else if (interfaces != null) {
            setNamePrefix(interfaces[ReflectUtils.findPackageProtected(interfaces)].getName());
        }
        return super.generate(data);
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以发现ehancer生成器只是做了个检查命名操作 在上面的Generator中也是做了个命名操作，然后继续执行父类的generate(data)方法，这个和步骤3.7一致，我们主要看其中生成字节码的方法，即最后调用的Enhancer.generatorClass(ClassVisitor c)方法，

方法代码如下

 

### 6.Enhancer.generatorClass(ClassVisitor c)

 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    public void generateClass(ClassVisitor v) throws Exception {
        //声明需代理的类 或者接口
        Class sc = (superclass == null) ? Object.class : superclass;
        //检查 final类无法被继承
        if (TypeUtils.isFinal(sc.getModifiers()))
            throw new IllegalArgumentException("Cannot subclass final class " + sc.getName());
        //找到该类所有声明了的构造函数
        List constructors = new ArrayList(Arrays.asList(sc.getDeclaredConstructors()));
        //去掉private之类的不能被继承的构造函数
        filterConstructors(sc, constructors);

        // Order is very important: must add superclass, then
        // its superclass chain, then each interface and
        // its superinterfaces.
        //这儿顺序非常重要  上面是源码的注释  直接留着  相信大家都能看懂 
        
        //声明代理类方法集合
        List actualMethods = new ArrayList();
        //声明代理接口接口方法集合
        List interfaceMethods = new ArrayList();
        //声明所有必须为public的方法集合  这儿主要是代理接口接口的方法
        final Set forcePublic = new HashSet();
        //即通过传入的代理类 代理接口，遍历所有的方法并放入对应的集合
        getMethods(sc, interfaces, actualMethods, interfaceMethods, forcePublic);
        
        //对所有代理类方法修饰符做处理 
        List methods = CollectionUtils.transform(actualMethods, new Transformer() {
            public Object transform(Object value) {
                Method method = (Method)value;
                int modifiers = Constants.ACC_FINAL
                    | (method.getModifiers()
                       & ~Constants.ACC_ABSTRACT
                       & ~Constants.ACC_NATIVE
                       & ~Constants.ACC_SYNCHRONIZED);
                if (forcePublic.contains(MethodWrapper.create(method))) {
                    modifiers = (modifiers & ~Constants.ACC_PROTECTED) | Constants.ACC_PUBLIC;
                }
                return ReflectUtils.getMethodInfo(method, modifiers);
            }
        });
        //创建类写入器
        ClassEmitter e = new ClassEmitter(v);
        
        //1.开始创建类  并写入基本信息  如java版本，类修饰符 类名等
        if (currentData == null) {
        e.begin_class(Constants.V1_8,
                      Constants.ACC_PUBLIC,
                      getClassName(),
                      Type.getType(sc),
                      (useFactory ?
                       TypeUtils.add(TypeUtils.getTypes(interfaces), FACTORY) :
                       TypeUtils.getTypes(interfaces)),
                      Constants.SOURCE_FILE);
        } else {
            e.begin_class(Constants.V1_8,
                    Constants.ACC_PUBLIC,
                    getClassName(),
                    null,
                    new Type[]{FACTORY},
                    Constants.SOURCE_FILE);
        }
        List constructorInfo = CollectionUtils.transform(constructors, MethodInfoTransformer.getInstance());
        //2. 声明一个private boolean 类型的属性：CGLIB$BOUND
        e.declare_field(Constants.ACC_PRIVATE, BOUND_FIELD, Type.BOOLEAN_TYPE, null);
        //3. 声明一个public static Object 类型的属性：CGLIB$FACTORY_DATA
        e.declare_field(Constants.ACC_PUBLIC | Constants.ACC_STATIC, FACTORY_DATA_FIELD, OBJECT_TYPE, null);
        // 这个默认为true  如果为false则会声明一个private boolean 类型的属性：CGLIB$CONSTRUCTED
        if (!interceptDuringConstruction) {
            e.declare_field(Constants.ACC_PRIVATE, CONSTRUCTED_FIELD, Type.BOOLEAN_TYPE, null);
        }
        //4. 声明一个public static final 的ThreadLocal：ThreadLocal
        e.declare_field(Constants.PRIVATE_FINAL_STATIC, THREAD_CALLBACKS_FIELD, THREAD_LOCAL, null);
        //5. 声明一个public static final 的CallBack类型的数组：CGLIB$STATIC_CALLBACKS
        e.declare_field(Constants.PRIVATE_FINAL_STATIC, STATIC_CALLBACKS_FIELD, CALLBACK_ARRAY, null);
        //如果serialVersionUID不为null  则设置一个public static final 的Long类型 serialVersionUID
        if (serialVersionUID != null) {
            e.declare_field(Constants.PRIVATE_FINAL_STATIC, Constants.SUID_FIELD_NAME, Type.LONG_TYPE, serialVersionUID);
        }
        
        //遍历CallBackTypes 即我们构建Enhancer是setCallBack的所有类的类型  本案例中是methodInterceptor 并且只传入了一个
        for (int i = 0; i < callbackTypes.length; i++) {
            //6.声明一个private 的传入的CallBack类型的属性：CGLIB$CALLBACK_0 (从0开始编号，)
            e.declare_field(Constants.ACC_PRIVATE, getCallbackField(i), callbackTypes[i], null);
        }
        //7声明一个private static 的传入的Object类型的属性：CGLIB$CALLBACK_FILTER
        e.declare_field(Constants.ACC_PRIVATE | Constants.ACC_STATIC, CALLBACK_FILTER_FIELD, OBJECT_TYPE, null);
        
        //判断currentData
        if (currentData == null) {
            //8.为null则开始声明所有的代理类方法的变量 以及其具体的重写实现方法，还有static初始化执行代码块
            emitMethods(e, methods, actualMethods);
            //9.声明构造函数
            emitConstructors(e, constructorInfo);
        } else {
            //声明默认构造函数
            emitDefaultConstructor(e);
        }
        //
        emitSetThreadCallbacks(e);
        emitSetStaticCallbacks(e);
        emitBindCallbacks(e);
        //如果currentData不为null
        if (useFactory || currentData != null) {
            //获取到所有CallBack索引数组
            int[] keys = getCallbackKeys();
            //10.声明三个newInstance方法
            //只有一个callback参数
            emitNewInstanceCallbacks(e);
            //参数为callback数组
            emitNewInstanceCallback(e);
            //参数为callback数组 以及附带的一些参数
            emitNewInstanceMultiarg(e, constructorInfo);
            //11.声明getCallBack方法
            emitGetCallback(e, keys);
            //12.声明setCallBack方法
            emitSetCallback(e, keys);
            //12.声明setCallBacks方法
            emitGetCallbacks(e);
            //12.声明setCallBacks方法
            emitSetCallbacks(e);
        }
        //类声明结束
        e.end_class();
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　可以看到这儿也是声明一个写入类 然后按照Ehancer的代理生成策略写入符合的class信息然后返回，最红依旧会执行toByteArray方法返回byte[]数组，这样则又回到了步骤3.5中 根据类加载器 字节码数组来动态将代理类加载进内存中的方法了。最后我们回到3.4中根据class获取实例的代码即可返回被代理实例。 而我们执行方法时执行的是代理类中对应的方法，然后调用我们传入的callback执行 原理和jdk动态代理类似，至此 cglib动态代理源码分析到此结束

 ## 总结

　　对比jdk动态代理和cglib动态代理我们可以得知  jdk动态代理只能代理接口，而cglib动态代理可以代理类或者接口，说明cglib的优势更加明显。但是jdk动态代理是java原生支持，所以不需要引入额外的包，cglib需要引入额外的包。二者的原理类似，都是在内存中根据jvm的字节码文件规范动态创建class并使用传入的类加载器加载到内存中，再反射调用生成代理实例，并且都根据类加载器做了相应的缓存。实际使用中应根据利弊按需使用

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

## 典型事例：jdbc的设计

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

# Spring MVC创建过程

​	先分析Spring MVC的整体结构，然后具体分析每层的创建过程

## 1.1 SpringMvc整体结构介绍

​	Spring MVC中核心Servlet的继承结构图，分为java左边部分和spring右边部分

![image-20200417122256982](E:/Git/make/总结/images/image-20200417122256982.png)

​	Servlet的继承结构中共有五个类，GenericServlet和HttpServlet在Servlet中说过，HttpServletBean、FrameWorkServlet和DispatcherServlet是Spring MVC中的。

​	这三个类直接实现三个接口：EnvironmentAware、EnvironmentCapable和ApplicationContextAware。

### EnvironmentAware接口

​	EnvironmentAware接口只有一个**setEnvironment(Environment)**方法，HttpServletBean实现了EnvironmentAware接口，spring会自动调用setEnvironment(Environment)传入Environment属性

```java
public abstract class HttpServletBean extends HttpServlet implements EnvironmentCapable, EnvironmentAware {
@Override
public void setEnvironment(Environment environment) {
   Assert.isInstanceOf(ConfigurableEnvironment.class, environment, "ConfigurableEnvironment required");
   this.environment = (ConfigurableEnvironment) environment;
}
```

> ​	XXAware在spring里表示对XX类可以感知：如果在xx类里使用spring的东西，可以通过实现xxAware接口告诉spring，spring会送过来，接收的方式通过实现接口唯一的方法set-xx

### EnvironmentCapable接口

​	EnvironmentCapable意思是具有Environment的能力，可以提供Environment，EnvironmentCapable唯一的方法是Environment getEnvironment();

​	实现EnvironmentCapable接口的类就是告诉spring它可以提供Environment，当spring需要Environment时会调用它的getEnvironment方法跟它要

ApplicatonContext和Environment

​	应用程序上下文和环境，在HttpServletBean中Environment使用的是StandardServletEnvironment(在createEnvironment方法中创建)，这里封装了**ServletContext**和**ServletConfig**、**JndiProperty**、**系统环境变量**和**系统属性**，这些都封装在**propertySources属性**下。

```java
HttpServletBean
  
protected ConfigurableEnvironment createEnvironment() {
   return new StandardServletEnvironment();
}
```

​	ServletConfigPropertySource的类型是StandardWrapperFacade，是Tomcat里定义的ServletConfig类型，所以，ServletConfigPropertySource封装的就是ServletConfig。

​	web.xml定义的contextConfigLocation可以在config下的parameters看到，config其实是Tomcat中StandardWrapper——存放Servlet的容器。

​	ServletContextProprtySource中保存的ServletContext

​	<img src="E:/Git/make/总结/images/image-20200417152435718.png" alt="image-20200417152435718" style="zoom: 50%;" />

​	JndiPropertySource存放的是Jndi	

​	MapPropertySource存放的是虚拟机的属性，如Java版本、操作系统的名字、版本等

​	SystemEnvironmentPropertySource存放的是环境变量，Java_home、path等属性

## 1.2 HttpServletBean

​	Servlet创建时可以直接调用无参数的init方法

```java
public final void init() throws ServletException {
  	//将servlet中配置的参数封装到pvs变量中，requiredProperties必须参数，否则报错
    PropertyValues pvs = new HttpServletBean.ServletConfigPropertyValues(this.getServletConfig(), this.requiredProperties);
    if (!pvs.isEmpty()) {
        try {
            BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
            ResourceLoader resourceLoader = new ServletContextResourceLoader(this.getServletContext());
            bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, this.getEnvironment()));
            //模版方法，可以在子类调用，做些初始化工作，bw代表DispatcherServlet
            this.initBeanWrapper(bw);
          	//将配置的初始化值设置到DispatcherServlet
            bw.setPropertyValues(pvs, true);
        } catch (BeansException var4) {
        }
    }
    //模版方法，子类初始化的入口方法
    this.initServletBean();
}
```

​	在HttpServletBean的init中，首先将Servlet中配置的参数使用BeanWeapper设置到DispatcherServlet的相关属性，然后调用模版方法initServletBean，子类通过该方法初始化

> ​	BeanWrapper是Spring提供的用来操作JavaBean属性的工具，可以修改一个对象的属性
>
> ​	User user = new User();
>
> BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(user);
>
> ​    bw.setPropertyValue("username","zhangsan");

## 1.3 FrameworkServlet

​	FrameworkServlet的初始化入口方法就是上面HttpServletBean中init方法里最后调用的initServletBean方法

```java
protected final void initServletBean() throws ServletException {
        this.webApplicationContext = this.initWebApplicationContext();
        this.initFrameworkServlet();
}
```

​	核心代码只有两句，初始化WebApplicationContext和初始化initFrameworkServlet(模版方法，子类可以覆盖，但子类没有使用它)。

​	所以FrameworkServlet在构建的过程主要就是初始化了initWebApplicationContext	

​	initWebApplicatoinContext方法做了三件事

		1. 获取spring的根容器rootContext

     		2. 设置webApplicationContext并根据情况调用onRefresh方法
               		3. 将webApplicationContext设置到ServletContext中

```java
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext = WebApplicationContextUtils.getWebApplicationContext(this.getServletContext());
    WebApplicationContext wac = null;
  	//如果已经通过构造方法设置了webApplicationContext
    if (this.webApplicationContext != null) {
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext)wac;
            if (!cwac.isActive()) {
                if (cwac.getParent() == null) {
                    cwac.setParent(rootContext);
                }
                this.configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        //当webApplicationContext已经存在ServletContext中时，通过配置在Servlet中的contextAttribute参数获取
        wac = this.findWebApplicationContext();
    }
    if (wac == null) {
      	//如果webApplicationContext还没创建，根据rootContext创建
        wac = this.createWebApplicationContext(rootContext);
    }
    if (!this.refreshEventReceived) {
        //当ContextRefreshedEvent事件还没有触发时调用次方法，模版方法，子类可以重写
        this.onRefresh(wac);
    }
    if (this.publishContext) {
        String attrName = this.getServletContextAttributeName();
        //将ApplicationContext保存到ServletContext中
        this.getServletContext().setAttribute(attrName, wac);
    }
    return wac;
}
```

### 获取Spring的根容器rootContext

​	默认情况spring会将自己的容器设置成ServletContext的属性，默认根容器的key为org.springframework.web.context.WebApplicationContext.ROOT，定义在WebApplicationContext中。

​	所以获取根容器只需要调用ServletContext的getArrtibute就可以

​	ServletContext # getAttribute("org.springframework.web.context.WebApplicationContext.ROOT");

### 设置webApplicationContext并根据情况调用onRefresh方法

​	设置webApplicationContext共有三种方法：

 1. 在构造方法中传递webApplicationContext参数，只需要对其设置即可。该方法主要用于Servlet3.0后的环境，Servlet3.0后可以在程序中使用ServletContext.addServlet方式注册Servlet，就可以在新建FrameworkServlet和其子类时通过构成方法传递已经准备好的webApplicationContext

 2. webApplicationContext已经在ServletContext中，这时只需要在配置Servlet时将ServletContext中的webApplicationContext的name配置到contextAttribute属性即可。

 3. 在前两种无效情况下自己创建一个。正常情况下使用这个方式。创建过程在createWebApplicationContext方法中，createWebApplicationContext内部又调用了configureAndRefreshWebApplicationContext方法。

    首先调用getContextClass方法获取创建的类型，可以通过contextClass属性设置到Servlet中，默认使用org.springframework.web.context.support.XmlWebApplicationContext。

    ​	然后检查属不属于ConfigurableWebApplicationContext类型，不属于抛异常

    ​	接下来根据BeanUtils.instantiateClass（contextClass）进行创建

    ​	将设置的contextConfigLocation参数传入，默认传入WEB-INFO/[ServletName]-Servlet.xml，然后进行配置。

    ​	configureAndRefreshWebApplicationContext方法中添加监听ContextRefreshedEvent的监听器，可以根据传入的参数进行选择，实际监听的事ContextRefreshListener所监听的事件

    ```java
    protected WebApplicationContext createWebApplicationContext(ApplicationContext parent) {
      	//获取创建类型
        Class<?> contextClass = this.getContextClass();
      	//检查创建类型
        if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        } else {
          	//具体创建
            ConfigurableWebApplicationContext wac = (ConfigurableWebApplicationContext)BeanUtils.instantiateClass(contextClass);
            wac.setEnvironment(this.getEnvironment());
            wac.setParent(parent);
            //将设置的contextConfigLocation参数传给wac，默认传入WEB-INFO/[ServletName]-Servlet.xml
            wac.setConfigLocation(this.getContextConfigLocation());
            this.configureAndRefreshWebApplicationContext(wac);
            return wac;
        }
    }
    ```

```java
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
    if (ObjectUtils.identityToString(wac).equals(wac.getId())) {
        if (this.contextId != null) {
            wac.setId(this.contextId);
        } else {
            wac.setId(ConfigurableWebApplicationContext.APPLICATION_CONTEXT_ID_PREFIX + ObjectUtils.getDisplayString(this.getServletContext().getContextPath()) + '/' + this.getServletName());
        }
    }
    wac.setServletContext(this.getServletContext());
    wac.setServletConfig(this.getServletConfig());
    wac.setNamespace(this.getNamespace());
  	//添加监听ContextRefreshedEvent的监听器
    wac.addApplicationListener(new SourceFilteringListener(wac, new FrameworkServlet.ContextRefreshListener()));
    ConfigurableEnvironment env = wac.getEnvironment();
    if (env instanceof ConfigurableWebEnvironment) {
        ((ConfigurableWebEnvironment)env).initPropertySources(this.getServletContext(), this.getServletConfig());
    }
    this.postProcessWebApplicationContext(wac);
    this.applyInitializers(wac);
    wac.refresh();
}
```

​	ContextRefreshListener是FrameworkServlet的内部类，监听ContxtRefreshedEvent事件，当接收到消息时调用FrameworkServlet的onApplicationEvent方法，在onApplicationEvent方法中会调用一次onRefresh方法，并将refreshEventReceived标志设置为true，表示已经refresh

```java
 class ContextRefreshListener implements ApplicationListener<ContextRefreshedEvent> {
        public void onApplicationEvent(ContextRefreshedEvent event) {
            FrameworkServlet.this.onApplicationEvent(event);
        }
    }
    
public void onApplicationEvent(ContextRefreshedEvent event) {
    this.refreshEventReceived = true;
    this.onRefresh(event.getApplicationContext());
}
```

​	当使用第三种方法初始化时已经refresh，不需要再调用onRefresh。第一种也调用了ConfigureAndRefreshWebApplication方法，也refresh过，所以只有使用第二种方式初始化时webApplicationContext时才会在这里调用onRefresh方法。

​	不管哪种方式，onRefresh最终肯定只会调用一次，且DispatcherServlet时通过重写这个模版方法实现初始化的。

### 将webApplicationContext设置到ServletContext中

​	根据publishContext标志判断是否将创建的webApplicationContext设置到ServletContext的属性中，publishContext标志可以在配置Servlet时通过init-param参数设置，HttpServletBean初始化时会将其设置到publishContext参数。

​	将webApplicationContext设置到ServletContext的属性中为了方便获取

> ​	配置Servlet时可以设置的一些初始化参数
>
> ​	contextAttribute：在ServletContext的属性中，用作WebApplicationContext的属性名称
>
> ​	contextClass：创建WebApplicationContext的类型
>
> ​	contextConfigLocation：Spring MVC配置文件的位置
>
> ​	publicshContext：是否将webApplicationContext设置到ServletContext的属性

## 1.4 DispatcherServlet

​	onRefresh方法是DispatcherServlet的入口方法。 onRefresh中简单调用了initStrategies，在initStrategies中调用了九个初始化方法。

​	分为两个方法为了层次清晰 刷新容器和初始化

```java
#DispatcherServlet

protected void onRefresh(ApplicationContext context) {
    this.initStrategies(context);
}

protected void initStrategies(ApplicationContext context) {
    this.initMultipartResolver(context);
    this.initLocaleResolver(context);
    this.initThemeResolver(context);
    this.initHandlerMappings(context);
    this.initHandlerAdapters(context);
    this.initHandlerExceptionResolvers(context);
    this.initRequestToViewNameTranslator(context);
    this.initViewResolvers(context);
    this.initFlashMapManager(context);
}
```

​	initLocaleResolver初始化分为两部分

​		首先通过context.getBean在容器里面按注册时的名称或类型，这里是名称localeResolver进行查找。

​		如果找不到就调用getDefaultStrategy方法按照类型LocaleResolver获取默认的组件

​		context 是FrameworkServlet中创建的WebApplicationContext，而不是ServletContext。

```java
private void initLocaleResolver(ApplicationContext context) {
    try {
        this.localeResolver = (LocaleResolver)context.getBean("localeResolver", LocaleResolver.class);
       
    } catch (NoSuchBeanDefinitionException var3) {
        this.localeResolver = (LocaleResolver)this.getDefaultStrategy(context, LocaleResolver.class);
    }
}
```

​	getDefaultStrategy方法通过调用getDefaultStrategy重载返回list的方法，不为空取第一个LocaleResolver。

​	因为HandlerMapping等组件可以有多个，所以返回list

```java
protected <T> T getDefaultStrategy(ApplicationContext context, Class<T> strategyInterface) {
    List<T> strategies = this.getDefaultStrategies(context, strategyInterface);
    if (strategies.size() != 1) {
    } else {
        return strategies.get(0);
    }
}

protected <T> List<T> getDefaultStrategies(ApplicationContext context, Class<T> strategyInterface) {
        String key = strategyInterface.getName();
  			//从DefaultStrategies获取所需策略的类型
        String value = defaultStrategies.getProperty(key);
        if (value == null) {
            return new LinkedList();
        } else {
          	//如果有多个默认值，根据都好分割为数组
            String[] classNames = StringUtils.commaDelimitedListToStringArray(value);
            List<T> strategies = new ArrayList(classNames.length);
            String[] var7 = classNames;
            int var8 = classNames.length;
						//按照取到的类型初始化策略
            for(int var9 = 0; var9 < 	、var8; ++var9) {
                String className = var7[var9];
                try {
                    Class<?> clazz = ClassUtils.forName(className, DispatcherServlet.class.getClassLoader());
                    Object strategy = this.createDefaultStrategy(context, clazz);
                    strategies.add(strategy);
                } catch (ClassNotFoundException var13) {
                } catch (LinkageError var14) {
                }
            }
            return strategies;
        }
    }
```

​	上面方法defaultStrategies.getProperty(key)是如何来的，就可以理解默认初始化的方式。

​	defaultStrategies里存放的是org.springframework.web.DispatcherServlet.properties里定义的键值

```java
    private static final Properties defaultStrategies;
static {
    try {
        ClassPathResource resource = new ClassPathResource("DispatcherServlet.properties", DispatcherServlet.class);
        defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
    } catch (IOException var1) {
        throw new IllegalStateException("Could not load 'DispatcherServlet.properties': " + var1.getMessage());
    }
}
```

​	该文件定义了不同组件的类型，上传组件MultipartResolver没有默认配置。这九个配置就是onRefresh里面初始化的九个方法。

​	HandlerMapping、HandlerAdapter、HandlerExceptionResolver都配置了多个

> ​		默认配置不是最优配置，只是在没有配置时候又个默认值。可能有些已经弃用了。
>
> ​		默认配置只有自相应类型没有配置时才会使用。
>
> ​		<mvc:annotation-driven/>不会使用默认配置，因为它配置了HandlerMapping、HandlerAdapter、HandlerExceptionResolver

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.
org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
   org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
   org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
   org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerExceptionResolver,\
   org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
   org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

​	

​	本节分析了Spring MVC自身的创建过程，Spring MVC中Servlet共有三个层次，分别是HttpServletBean、FrameworkServlet和DispatcherServlet。

​	HttpServletBean直接继承自Java的HttpServlet，作用是将Servlet中配置的参数设置到相应的属性

​	FrameworkServlet初始化了WebApplicationContext，共有三种方式

​	DispatcherServlet初始化了自身的九个组件



# Spring MVC 处理请求

​	本节分析Spring MVC如何处理请求的。分为两步：

1. 首先分析HttpServletBean、FrameworkServlet和DispatcherServlet的处理过程，可以明白从Servlet容器将请求交给Spring MVC一直到DispatcherServlet具体处理请求前都做了些什么

  2. 重点分析Spring MVC中最核心的处理方法doDispatch的结构

### 2.1 HttpServletBean

​    HttpServletBean主要参与了创建工作，在处理请求中没有涉及相应的请求处理。

### 2.2 FrameworkServlet

​    Servlet处理过程：从Servlet的service方法开始，然后在HttpServlet的service方法中根据不同类型将请求路由到doGet、doHead、doPost、doPut、doDelete、doOption、doTrace，并做了doHead、doOption、doTrace的默认实现，doHead调用了doGet，返回只有header没有body的response。

​    FrameworkServlet重写了service、doGet、doPost、doPut、doDelete、doOptions、doTrace，在service增加了对PATCH类型的处理，其他类型的请求直接交给父类进行处理；doOptions和doTrace可以设置是否交给父类处理(默认都是交给父类处理，doOptions会在父类的处理结果中增加PATCH类型)；doGet、doPost、doPut和doDelete都是自己处理。所有需要自己处理的请求都交给了processRequest方法进行统一处理。

​    以下是service和doGet的代码：

```java
protected void service(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
   HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
   if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
      processRequest(request, response);
   }
   else {
      super.service(request, response);
   }
}
protected final void doGet(HttpServletRequest request, HttpServletResponse response)
      throws ServletException, IOException {
   processRequest(request, response);
}
```

​    由上可以看出，将请求合并到processRequest中，在里面Spring MVC对不同类型的请求用不同的Handler进行处理。![image-20200417210800618](E:/Git/make/总结/images/image-20200417210800618.png)

​	ProcessRequest方法是FrameworkServlet类在处理请求中最核心的方法

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;
  	//获取localContextHolder中原来保存的localContext
    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
  	//获取当前的localContext
    LocaleContext localeContext = this.buildLocaleContext(request);
  	//获取RequestContextHolder中原来保存的RequestAttributes
    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
  	//获取当前请求的ServletRequestAttributes
    ServletRequestAttributes requestAttributes = this.buildRequestAttributes(request, response, previousAttributes);
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new FrameworkServlet.RequestBindingInterceptor());
  	//当前的localContext和当前请求的ServletRequestAttributes设置到localContextHolder和RequestContextHolder
    this.initContextHolders(request, localeContext, requestAttributes);
    try {
      	//实际处理请求入口
        this.doService(request, response);
    } catch (ServletException var17) {
        failureCause = var17;
        throw var17;
    } catch (IOException var18) {
        failureCause = var18;
        throw var18;
    } catch (Throwable var19) {
        failureCause = var19;
        throw new NestedServletException("Request processing failed", var19);
    } finally {	
      	//处理完恢复原来的localContext和ServletRequestAttributes到localContextHolder和RequestContextHolder
        this.resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
      	//发布ServletRequestHandledEvent消息
        this.publishRequestHandledEvent(request, response, startTime, (Throwable)failureCause);
    }
}
```

​	processRequest方法中的核心语句是doService(request, response),模版方法，在DispatcherServlet中具体实现。

​	processRequest自己主要做了两件事

     		1. 对localContext和ServletRequestAttributes的设置及恢复
               		2. 处理完后发布ServletRequestHandledEvent消息

​    LocaleContext:存放着Locale（本地化信息）；RequestAttributes：是spring的一个接口，通过它可以get/set/removeAttribute，根据scope参数判断操作request还是session，具体使用的是ServletRequestAttributes，ServletRequestAttributes里面封装了request、response、session，通过get就可以获得。

​	ServletRequestAttributes的setAttribute方法，设置属性时通过scope判断是对request还是session进行设置，isRequestActive()，当调用了ServletRequestAttributes的requestCompleted方法后requestActive就会变为false，之前是true。

> ​	因为request执行完了，不能再对它进行操作了，上面的finally块已调用requestAttributes.requestCompleted()的方法

```java
public void setAttribute(String name, Object value, int scope) {
    if (scope == 0) {
      	if (!this.isRequestActive()) {
                throw new IllegalStateException("Cannot set request attribute - request is not active anymore!");
            }
        this.request.setAttribute(name, value);
    } else {
        HttpSession session = this.getSession(true);
        this.sessionAttributesToUpdate.remove(name);
        session.setAttribute(name, value);
    }
}
```

​    LocaleContext可以获取Locale

​	RequestAttributes用于管理request和session的属性

​	LocaleContextHolder，是抽象类，里面的方法是static的，可以直接使用。没有父类和子类，不能对它实例化。

```java
public abstract class LocaleContextHolder {
    private static final ThreadLocal<LocaleContext> localeContextHolder = new NamedThreadLocal("Locale context");
    private static final ThreadLocal<LocaleContext> inheritableLocaleContextHolder = new NamedInheritableThreadLocal("Locale context");
```

​	   LocaleContextHolder类里面封装了两个属性localeContextHolder和inheritableLocaleContextHolder，都是LocalContext，其中第二个可以被子线程继承。localeContextHolder提供了get/set方法，可以获取和设置localeContext。另外还提供了get/setLocale方法，可以直接操作Locale，都是static的。

​	 RequestContextHolder也是一样的道理。封装了RequestAttributes，可以get/set/removeAttribute，因为实际封装的ServletRequestAttributes，还可以getRequest、getResponse、getSession，就可以在任何地方获取这些对象了。

​	最后发布 this.publishRequestHandledEvent(request, response, startTime, (Throwable)failureCause)发布ServletRequestHandledEvent消息

​	当publishEvents设置为true时，请求处理结束后就会发出这个消息，无论请求处理成功与否都会发布。publishEvents在web.xml中和Spring MVC的Servlet中可以配置，默认为true。

```java
private void publishRequestHandledEvent(HttpServletRequest request, HttpServletResponse response, long startTime, Throwable failureCause) {
    if (this.publishEvents) {
        long processingTime = System.currentTimeMillis() - startTime;
        int statusCode = responseGetStatusAvailable ? response.getStatus() : -1;
        this.webApplicationContext.publishEvent(new ServletRequestHandledEvent(this, request.getRequestURI(), request.getRemoteAddr(), request.getMethod(), this.getServletConfig().getServletName(), WebUtils.getSessionId(request), this.getUsernameForRequest(request), processingTime, failureCause, statusCode));
    }
}
```

​    FrameworkServlet工作流程：首先，在service方法中添加对PATCH的处理，并将所需要自己处理的请求集中到processRequest进行统一处理。然后，processRequest将处理逻辑交给模板方法doService，在doService的的前后使用request获取LocaleContexthe和RequestAttributes进行保存，处理完之后恢复。最后，发布ServletRequesthandledEvent事件。

### 2.3 DispatcherServlet

​    DispatcherServlet里面执行处理的入口方法是doService。doService将处理交给doDispatch进行具体处理，在doDispatch处理前doService做了一些事情：首先判断是不是include请求，如果是则对request的Attribute做个快照备份，等doDispatch处理完之后进行还原，在做完快照后又对request设置了一些属性

```java
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    if (this.logger.isDebugEnabled()) {
  String resumed = WebAsyncUtils.getAsyncManager(request).hasConcurrentResult() ? " resumed" : "";
    }
  	// 当include请求时对request的Attribute做快照备份
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap();
        Enumeration attrNames = request.getAttributeNames();
        label108:
        while(true) {
            String attrName;
            do {
                if (!attrNames.hasMoreElements()) {
                    break label108;
                }
                attrName = (String)attrNames.nextElement();
            } while(!this.cleanupAfterInclude && !attrName.startsWith("org.springframework.web.servlet"));

            attributesSnapshot.put(attrName, request.getAttribute(attrName));
        }
    }
		// 对request设置一些属性
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, this.getThemeSource());
    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
    if (inputFlashMap != null) {
        request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
    }

    request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
    request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);

    try {
        this.doDispatch(request, response);
    } finally {
      	// 还原request快照的属性
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted() && attributesSnapshot != null) {
            this.restoreAttributesAfterInclude(request, attributesSnapshot);
        }
    }
}
```

   对request设置的属性中，前面4个属性webApplicationContext、localeResolver、themeResolver和themeSource在Handler和view中使用。后面3个属性都和flashMap相关，主要用于Redirect转发是参数的传递。

​    doDispatch的核心代码：（1）根据request找到Handler  (2)根据Handler找到对应的HandlerAdapter  (3)用HandlerAdapter处理Handler  （4）调用processDispatchResult方法处理上面处理之后的返回结果(包括找到View并渲染输出给用户)。

```java
mappedHandler = this.getHandler(processedRequest);
HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
 this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
```

​    Hanlder :处理器，标注了@RequestMapping的类或者方法，直接对应MVC中的Controller

​    HandlerMapping : 根据请求查找Handler，

​    handlerAdapter ：适配器，调用具体的Handler对请求进行处理。

​	Hanlder相关与设备，HandlerMapping根据需求选择不同的设备，handlerAdapter具体操作设备的工人，不同的设备需要不同的工人。

​	View和ViewResolver的原理与Handler与HandlerMapping的原理类似。View用来展示数据的，ViewResolver用来查找View。

### 2.4 DoDispatch结构

![image-20200417223334684](E:/Git/make/总结/images/image-20200417223334684.png)

详细分析doDispatch内部的结构及处理的流程

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        try {
            ModelAndView mv = null;
            Object dispatchException = null;

            try {
              	//检查是不是上传请求
                // 是上传资源的话将request转换为multipartHttpServletRequest
                processedRequest = this.checkMultipart(request);
                multipartRequestParsed = processedRequest != request;
                //根据request找到Handler
              	mappedHandler = this.getHandler(processedRequest);
                if (mappedHandler == null || mappedHandler.getHandler() == null) {
                    this.noHandlerFound(processedRequest, response);
                    return;
                }
								//根据Handler找到HandlerAdapter
                HandlerAdapter ha = this.getHandlerAdapter(mappedHandler.getHandler());
              	//处理GET、HEAD请求的Last-Modified
                String method = request.getMethod();
                boolean isGet = "GET".equals(method);
                if (isGet || "HEAD".equals(method)) {
                    long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                    if (this.logger.isDebugEnabled()) {
                        this.logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
                    }

                    if ((new ServletWebRequest(request, response)).checkNotModified(lastModified) && isGet) {
                        return;
                    }
                }
								// 执行相应Interceptor的preHandle
                if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                    return;
                }
								//HandlerAdapter使用Handler处理请求
                mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
              	// 如果需要异步处理，直接返回
                if (asyncManager.isConcurrentHandlingStarted()) {
                    return;
                }
								// 当view为空时，根据request设置默认view
                this.applyDefaultViewName(processedRequest, mv);
                //执行相应Interceptor的postHandle
                mappedHandler.applyPostHandle(processedRequest, response, mv);
            } catch (Exception var20) {
                dispatchException = var20;
            } catch (Throwable var21) {
                dispatchException = new NestedServletException("Handler dispatch failed", var21);
            }
						// 处理返回结果。包括处理异常、渲染页面、发出完成通知触发Interceptor的afterCompletion
            this.processDispatchResult(processedRequest, response, mappedHandler, mv, (Exception)dispatchException);
        } catch (Exception var22) {
            this.triggerAfterCompletion(processedRequest, response, mappedHandler, var22);
        } catch (Throwable var23) {
            this.triggerAfterCompletion(processedRequest, response, mappedHandler, new NestedServletException("Handler processing failed", var23));
        }

    } finally {
      	// 判断是否执行异步请求
        if (asyncManager.isConcurrentHandlingStarted()) {
            if (mappedHandler != null) {
             mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
          // 删除上传请求的资源
        } else if (multipartRequestParsed) {
            this.cleanupMultipart(processedRequest);
        }
    }
}
```

doDispatch分为两部分：处理请求和渲染页面。开头定义了几个变量

> ​	HttpServletRequest processedRequest：实际处理时所用的request，如果不是上传请求则直接使用接收到的request，否则封装为上传类型的request
>
> ​	HandlerExecutionChain mappedHandler：处理请求的处理器链(包含处理器和对应的Interceptor)
>
> ​	Boolean multipartRequestParsed：是不是上传请求的标志
>
> ​	ModelAndView mv：封装Model和View的容器
>
> ​	Exception dispatchException：处理请求过程中抛出的异常。不包含渲染过程抛出的异常	

​	doDispatch中首先检查是不是上传请求，是则将request转换为multipartHttpServletRequest，并将multipartRequestParsed标志设置为true。其中使用了MultipartResolver。

​	然后通过getHandler方法获取Handler处理器链，使用HandlerMapping返回值为HandlerExecutionChain类型，其中包含与当前request相匹配的Interceptor和Handler。

```java
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
    Iterator var2 = this.handlerAdapters.iterator();
    HandlerAdapter ha;
    do {
        if (!var2.hasNext()) {
            throw new ServletException("No adapter for handler [" + handler + "]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
        }
        ha = (HandlerAdapter)var2.next();
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Testing handler adapter [" + ha + "]");
        }
    } while(!ha.supports(handler));
    return ha;
}
```

​	方法结构非常简单，HandlerExecutionChain的类型类似前面Tomcat中的Pipeline，Interceptor和Handler相当于Value和BaseValue，执行时先一次执行Interceptor的preHandle方法，最后执行Handler，返回时按相反的顺序执行Interceptor的postHandle方法。

​	接下来处理Get、HEAD请求的Last-Modified。当浏览器第一次根服务器请求资源(GET、Head请求)时，服务器在返回的请求头里包含一个Last-Modified的属性，代表本资源最后是什么时候修改的。浏览器以后发送请求时会同时发送之前接收到的Last-Modified，服务器接收到带Last-Modified的请求后会用其值和自己实际资源的最后修改事件做对比，如果资源过期了则返回新的资源(同时返回新的Last-Modified)，否则直接返回304状态吗表示资源为过期，浏览器直接使用之前缓存的结果。

​	接下来依次调用相应Interceptor的preHandle。

​	处理完Interceptor的preHandle后就到了此方法最关键的地方—让HandlerAdapter使用Handler处理请求，Controller就是在这个地方执行的。这里主要使用了HandlerAdapter。

​	Handler处理完请求后，如果需要异步处理，则直接返回，如果不需要异步处理，当view为空时(如Handler返回值为void)，设置默认view，然后执行相应Interceptor的postHandle。设置默认view的过程中使用到了ViewNameTranslator。

​	结下来使用processDispatchResult方法处理前面返回的结果，其中包括处理异常、渲染页面、触发Interceptor的afterCompletion方法三部分内容。

​	doDispatch的异常处理结构。doDispatch有两层异常捕获，内层是捕获在对请求进行处理的过程中抛出的异常，外层主要是在处理渲染页面时抛出的。

​	内层的异常就是执行请求处理时的异常会设置到dispatchException变量，然后在processDispatchResult方法中进行处理，外层则是处理processDispatchResult方法抛出的异常。processDispatchResult代码如下

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
    boolean errorView = false;
  	// 如果请求处理的过程中有异常抛出则处理异常
    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            this.logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException)exception).getModelAndView();
        } else {
            Object handler = mappedHandler != null ? mappedHandler.getHandler() : null;
            mv = this.processHandlerException(request, response, handler, exception);
            errorView = mv != null;
        }
    }	
  	// 渲染页面
    if (mv != null && !mv.wasCleared()) {
        this.render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    } else if (this.logger.isDebugEnabled()) {
    }
    if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {		 // 发出请求处理完成的通知，触发Interceptor的afterCompletion
        if (mappedHandler != null) {
            mappedHandler.triggerAfterCompletion(request, response, (Exception)null);
        }
    }
}
```

​		processDispatchResult处理异常的方式其实就是将相应的错误页面设置到View，在其中的processHandlerException方法中用到了HandlerExceptionResolver。

​	渲染页面具体在render方法中执行，render中首先对response设置了Local，过程中使用到了LocalResolver，然后判断View如果是String类型则调用resolveViewName方法使用ViewResolver得到实际的View，最后调用View的render方法对页面进行具体渲染，渲染的过程中使用到了ThemeResolver。

​	最后通过mappedHandler的triggerAfterCompletion方法触发Interceptor的afterCompletion方法，这里的Interceptor也是按反方向执行的。到这里processDispatchResult方法就执行完了。

​	再返回doDispatch方法中，在最后的finally中判断是否请求启动了异步处理，如果启动了则调用相应异步处理的拦截器，否则如果是上传请求则删除上传请求过程中产生的临时资源。	

## 小结

​	分析了Spring MVC中请求处理的过程。首先对三个Servlet进行分析，然后单独分析DispatcherServlet中的doDispatch方法。

​	三个Servlet的处理过程大致功能如下：

​		HttpServletBean：没有参与实际请求的处理

​		FrameworkServlet：将不同类型的请求合并到了processRequest方法统一处理，processRequest方法中做了三件事：

​			调用了doService模版方法具体处理请求

​			将当前请求的LocaleContext和ServletRequestAttributes在处理请求前设置到了LocaleContextHodler和RequestContextHolder，并在请求处理完成后恢复。

​			请求处理完后发布了ServletRequestHandleEvent消息

​		DispatcherServlet：doService方法给request设置了一些属性并将请求交给doDispatch方法具体处理

​	DispatcherServlet中的doDispatch方法完成Spring MVC中请求处理过程的顶层设计，它使用DispatcherServlet中的九大组件完成了具体的请求处理。另外HandlerMapping、Handler和HandlerAdapter这三个概念的含义及它们之间的关系也非常重要。

# Spring MVC 原理

​	如果在 web.xml 中设置 DispatcherServlet 的**为 /** 时,当用户发 起 请 求 , 请 求 一 个 控 制 器 , 首 先 会 执 行**DispatcherServlet**. 由DispatcherServlet 调 用 **HandlerMapping** 的DefaultAnnotationHandlerMapping **解 析 URL**, 解 析 后 调 用**HandlerAdatper** 组 件 的 AnnotationMethodHandlerAdapter **调 用Controller 中的 HandlerMethod.**当 HandlerMethod 执行完成后会返回**View**,会被 ViewResovler 进行视图解析,解析后调用 **jsp 对应的.class 文件并运行**,最终把运行.class 文件的结果响应给客户端.	

Spring MVC本质是一个Servlet，Servlet运行需要一个Servlet容器，如常用的Tocmat。Servlet容器帮我们统一做了像底层Socket连接那种通用又麻烦的工作，让开发变得轻松，只需要按照Servlet的接口做就可以。

​	Spring MVC又在此基础上提供了一套通用的解决方案，Servlet都可以不用写，只关注业务就可以。

​	下面以Tomcat为例分析Servlet容器的结构和原理。

​	Tomca可以分为两大部分：连接器和容器，连接器专门用于处理网络连接相关的事情，如Socket连接、request封装、连接线程池维护等工作，容器用来存放我们编写的网站程序，Tomcat中一共有4层容器：Engine、Host、Context和Wrapper。一个Wrapper对应一个Servlet，一个Context对应一个应用，一个Host对应一个站点，Engine是引擎，一个容器只有一个。Context和Host的区别是Host代表站点，如不同的域名，而Context表示站点下的一个应用。一套容器和多个连接器组成一个Service，一个Tomcat中可以有多个Service。

​	Servlet接口一共定义了5个方法，其中init方法和destroy用于初始化和销毁Servlet，整个生命周期中只会被调用一次；service方法实际处理请求；getServletConfig方法返回的ServletConfig，可以获取到配置Servlet时使用init-param配置的参数，还可以获取ServletContext；getServletlnfo方法可以获取到一些Servlet相关的信息，如作者、版权等，这个方法需要自己实现，默认返回空字符串。

​	Java提供了两个Servlet的实现类：GenericServlet和HttpServlet。

​	GenericServlet主要做了三件事：①实现了ServletConfig接口，让我们可以直接调用ServletConfig中的方法；②提供了无参的init方法；③提供了log方法。

​	HttpServlet主要做了两仲事：①将ServletRequest和ServletResponse转换为了HttpServletRequest和HttpServletResponse;②根据Http请求类型（如Get、Post等）将请求路由到了7个不同的处理方法，这样在编写代码时只需要将不同类型的处理代码编写到不同的方法中就可以了，如常见的doGet、doPost方法就是在这里定义的。

​	Spring MVC的本质是个Servlet，这个Servlet继承自HttpServlet。Spring MVC中提供了三个层次的Servlet：HttpServletBean、FrameworkServlet和DispatcherServlet，下层继承上层，HttpServletBean直接继承自Java的HttpServlet。

​	HttpServletBean用于将Servlet中配置的参数设置到相应的属性中，FrameworkServlet初始化了Spring MVC中所使用的WebApplicationContext，具体处理请求的9大组件是在DispatcherServlet中初始化的。

​	Spring MVC的结构就总结到这里，接下来总结Spring MVC的请求处理过程。SpringMVC中请求的处理主要在DispatcherServlet中，不过它上一层的FrameworkServlet也做了一些工作，首先它将所有类型的请求都转发到processRequest方法，然后在processRequest方法中做了三件事：①调用了doService模板方法具体处理请求，doService方法在DispatcherServlet中实现；（参将当前请求的LocaleContext和ServletRequestAttributes在处理请求前设置到了LocaleContextHolder和RequestContextHolder，并在请求处理完成后恢复；③请求处理完后发布一个ServletRequestHandledEvent类型的消息。

​	DispatcherServlet在doServic方法中将webApplicationContext、IocaleResolver、themeResolver、themeSource、FlashMap和FlashMapManager设置到request的属性中以方便使用，然后将请求交给doDispatch方法进行具体处理。

​	DispatcherServlet的doDispatch方法按执行过程大致可以分为4步：①根据request找到Handler；②根据找到的Handler找到对应的HandlerAdapter；③用HandlerAdapter凋用Handler处理请求；④调用processDispatchResult方法处理Handler处理之后的结果（主要处理异常和找到View并渲染输出给用户）。这4步中的每一步又有自己复杂的处理过程，详细内容这里就不介绍了，21.2节会再和大家一起回顾。

​	前面介绍的Handler、HandlerMapping和HandlerAdapter的关系，Handler是具体干活的工具，HandlerMapping用来找出需要的Handler，HandlerAdapter是怎么具体使用Handler干活，可以理解为使用工具的人。

​	一般做事情都是这个步骤，先找到工具，然后找到使用工具的人，最后人使用工具干活。这种思想就是Spring MVC的灵魂，它贯穿整个Spring MVC。上面说的Handler是MVC中的C层，其实Spring MVC在MVC三层使用的都是这种思想，在V层也就是View层干活的工具是View，查找View使用的是ViewResoLver和RequestToViewNameTranslator，因为View是标准的格式，使用非常简单，所以就没有"使用的人"这个角色；在M层也就是Model层，这层干活的就多了，注释了@ModelAttribute的方法、SessionAttribute、FlashMap、Model以及需要执行的方法的参数和返回值等都属于这一层，HandlerMethodArgumentResolver和HandlerMethodReturn ValueHandler、ModeIFactory和FlashMapManager是这一层中"使用的人"，HandlerMethodArgumentResolver和HandlerMethodReturn ValueHandler同时还担任着"查找共具"的角色。

​	因为Model层贯穿于Controller层和View层之中，所以很容易将其当作那两层中的内容，其实Model层才是Spring MVC最复杂的地方。

# 一个HTTP请求的处理流程

先大致分析一下启动过程，然后详细分析请求的处理过程。

​	因为在web.xml文件中给Spring MVC的Servlet配置了load-on-startup，所以程序启动时会初始化Spring MVC，在HttpServletBean中将配置的contextConfigLocation属性设置到Servlet中，然后在FrameworkServlet中创建了WebApplicationContext，DispatcherServlet根据contextConfigLocation配置的classpath下的appContext.xml文件初始化了Spring MVC中的组件。这就是Spring MVC容器创建的过程。

下面来分析具体处理请求的过程。

1)请求发送到服务器后，服务器程序就会分配一个Socket线程来跟它连接，接着创建出request和response，然后交给对应的Servlet处理，这样请求就从Servlet容器传递到了Servlet（Servlet容器）。

2)在Servlet中请求首先会被HttpServlet处理，在HttpServlet的service方法中将ServletRequest和ServletResponse转换为HttpServletRequest和HttpServletResponse，并调用转换为后Request和Response的servlce方法(Java的HttpServlet)。

3)接下来请求就到了Spring MVC，在Spring MVC中首先由FrameworkServlet的service方法进行处理，这里service方法又会将请求交给HttpServlet的service方法处理。(FrameworkServlet)

4)在HttpServlet的service方法中会根据请求类型将请求传递到对应的方法如doGet方法(Java的HttpServlet)。

5)doGet方法在Spring MVC的FrameworkServlet中，它又将传递到了processRequest方法，然后在processRequest方法中将当前请求的LocaleContext和RequestAttributes设置到LocaleContextHolder和RequestContextHolder后将请求传递到了doService方法，doService方法在DispatcherServlet里实现(FrameworkServlet)。

6)DispatcherServlet的doService方法将webApplicationContext. localeResolver. themeResolver. themeSource. outputFlashMap和flashMapManager设置到了request的属性中，然后将请求传递到了doDispatch方法中(DispatcherServlet)。

7)DispatcherServlet的doDispatch中首先调用checkMultipart方法检查是不是上传请求，然后调用getHandler方法获取到Handler( DispatcherServlet)。

8)getHandler方法获取Handler的过程会遍历容器中所有的HandlerMapping,<<mvc:annotation-driven/>>标签配置的HandlerMapping是RequestMappingHandlerMapping和BeanNameUrIHandlerMapping，在用RequestMappingHandlerMapping匹配时我们的请求会和其初始化时读取到定义的@RequestMapping(value= "hello"))所注释的内容相匹配，然后根据这个条件找到定义的处理器方法hello方法(RequestMappingHandlerMapping)。

9)找到处理器后调用RequestMappingInfoHandlerMapping里的handleMatch方法会将匹配到的Pattern( hello)设置到了request的属性中( RequestMappinglnfoHandlerMapping)。

10)找到Handler后返回DispatcherServlet的doDispatch方法中，然后调用getHandlerAdapter方法根据Handler查找HandlerAdapter．也就是根据工具找使用工具的人。查找的方式也是遍历配置的所有HandlerAdapter，然后分别调用它们的supports方法进行检查，检查的方法通常是看Handler的类型是否支持，RequestMappingHandlerAdapter. HttpRequestHandlerAdapter和SimpleControllerHandlerAdapter,最后找到RequestMappingHandlerAdapter( DispatcherServlet)。

11)DispatcherServlet的doDispatch方法中检查到是Get请求，然后检查是否可以使用缓存，因为RequestMappingHandlerAdapter的getLastModified方法直接返回-l，所以不会使用缓存，接着调用了Handlerlnterceptor的preHandle方法，这里没有配置Handlerlnterceptor，这一步就不管了，接下来用RequestMappingHandlerAdapter便用Handler处理请求（DispatcherServlet）。

12)RequestMappingHandlerAdapter首先在其父类的handle方法中直接将请求传递到了它的handlelnternal方法，handlelnternal方法首先调用getSessionAttributesHandler初始化了本处理器对应的SessionAttributesHandler，并判断出注释有@SessionAttributes，进而调用checkAndPrepare方法禁止了Response的缓存，然后将请求传递到了invokeHandleMethod方法( RequestMappingHandlerAdapter).

13)RequestMappingHandlerAdapter的invokeHandleMethod方法中首先创建了WebDataBinderFactory,ModeIFactory和ServletlnvocableHandlerMethod, ModeIFactory创建过程中会找到定义的注释了@ModeIAttribute的replaceSensitiveWords万法(RequestMappingHandlerAdapter)。

14)接着创建ModelAndViewContainer，并调用ModeIFactory的initModel方法给Model设置参数，这里会调用了定义的replaceSensitiveWords方法，调用前会使用RequestParamMethodArgumentResolver解析出"xxx"参数的值并设置给replaceSensitiveWords方法，方法处理完（去除敏感词）后，ModeIFactory将其使用@ModelAttribute注释中的"xxx"作为name，去除敏感词后的内容作为value设置到Model中(ModeIFactory)。

15)接下来调用ServletInvocableHandlerMethod的invokeAndHandle方法实际执行处理,先在父类InvocableHandlerMethod的invokeForRequest方法中调用了getMethodArgumentValues方法来解析参数，@PathVariable、RedirectAttributes、Model三个参数分别使用PathVariableMethodArgumentResolver、RedirectAttributesMethodArgumentResolver和ModelMethodProcessors三个参数解析器来解析，第一个返回在HandlerMapping中（第9步）设置到request属性中的值，第二个会新建一个RedirectAttributesModeLMap然后设置到mavContainer中并返回，第三个直接返回mavContainer中的Model，这时的Model中已经保存了前面去敏感词后的xx参数(InvocableHandlerMethod).

16)接下来调用InvocableHandlerMethod的dolnvoke方法处理请求，这里实际调用了我们编写的xxx处理方法，其中将"xx"设置到RedirectAttributes中，通过FlashMap传递，将"yy"设置到Model中，因为它在@SessionAttributes中进行了设置，所以会保存到SessionAttributes中，处理完后返回"redirect:/showArticle"，将请求返回到ServletInvocableHandlerMethod(自定义的FollowMeController).

17)ServletlnvocableHandlerMethod使用HandlerMethodReturn ValueHandler处理返回值，因为返回的是String，所以使用的是ViewNameMethodReturn ValueH andler，它首先将返回值"redirect:/showArticle"设置到mavContainer的wew里，然后将mavContainer中的redirect标志redirectModeIScenario设置为了true，这样ServletlnvocableHandlerMethod就处理完了，接着将请求返回RequestMappingHandlerAdapter( ServletInvocableHandlerMethod)。

18)RequestMappingHandlerAdapter调用getModeIAndView方法对返回值进一步处理，首先使用ModeIFactory的updateModel方法处理@SessionAttributes注释，将其中的"xx"参数从Model中取出并使用sessionAttributesHandler保存；然后使用mavContainer中的Model和View创建ModelAndView；最后检查到Model是RedirectAttributes类型（因为我们返回的是redirect视图，而且设置了RedirectAttributes属性，所以mavContamer中getModel会返回RedirectAttributes类型的Model），这时会将之前保存到RedirectAttributes中的"xxx"参数设置到outputFlashMap，这样RequestMappingHandlerAdapter的处理就完成了，请求返回DispatcherServlet中(RequestMappingHandlerAdapter).

19)DispatcherServlet在doDispatch方法中首先检查返回的View是否为空，如果为空使用RequestToViewNameTranslator查找默认View，然后执行Handlerlnterceptor的applyPostHandle方法，这里这两项都不需要处理。接下来将请求传递到processDispatchResult方法( DispatcherServlet)。

20)DispatcherServlet的processDispatchResult方法中首先判断是否有异常，这里没有则不需要处理，然后调用render方法进行页面的渲染。render方法中首先使用localeResolver解析出Locale；然后调用resolveViewName方法解析出View，解析过程使用到了ViewResolver，这里使用的是我们配置的InternalResourceViewResolver，具体处理方法在UrIBasedViewResolver的createView方法中，它检查到是redirect的返回值，所以创建了RedirectView类型的View；然后调用View的render方法渲染输出(DispatcherServlet)。

21)RedirectView的render方法在父类AbstractView中定义，其中调用了RedirectView的renderMergedOutputModel方法，renderMergedOutputModel方法中将request属性中保存的outputFlashMap和FlashMapManager取出，使用FlashMapManager将outputFlashMap保存到了Session中，然后调用sendRedirect方法使用response的sendRedirect方法将请求发出，然后请求返回DispatcherServlet昀processDispatchResult方法(RedirectView)。

22)DispatcherServlet的processDispatchResult方法接着调用Handler的triggerAfierComp-letion方法，进而调用拦截器的afterCompletion方法，然后将请求返回到FrameworkServlet( DispatcherServlet)。

23)在FrameworkServlet的processRequest方法中将原来的LocaleContext和RequestAttributes恢复到LocaleContextHolder和RequestContextHolder中，并发出ServletRequestHandledEvent消息，最后将请求返回给Servlet容器(FrameworkServlet)。

这样一个完整的请求就处理完了。 Redirect后的请求处理过程大致和前面的过程差不多，主要有以下不同：

1. 在上述第6步中DispatcherServlet的doService方法会使用flashMapManager将之前保存的FlashMap取出保存到request的"INPUT_ FLASH—MAP_ ATTRIBUTE"属性中；
2. 在上述第14步中ModeIFactory的initModel方法会将之前保存在SessionAttributes中的"xx"参数设置到Model中；
3. 在上述第1 5步中参数解析器使用的是ModeIMethodProcessor和SessionStatusMethodArgumentResolver，它们都是直接从mavContainer中获取的；
4. 在上述第1 8步中ModelFactory的updateModel方法会在判断mavContainer中sessionStatus的状态后将SessionAttributes清空；
5. 在上述第20、21步中由于本次返回值不是redirect类型所以IntemaIResourceViewResolver解析出的不是RedirectView而是jsp类型的InternaIResourceView，对应的模板是`/WEB-INF/views/xx.jsp`.

# Spring MVC Java配置源码

## SpringServletContainerInitializer

​	Servlet 3.0 {ServletContainerInitializer}设计为使用Spring的WebApplicationInitializer SPI支持基于代码的Servlet容器配置，而不是（或可能与传统的基于 web.xml 的组合）方法。

Tocmat web容器在启动的时候去调用ServletContainerInitializer的onStartup()，这是Spring实现的Servlet规范的，Servlet容器的初始化器 ，传入ServletContext web上下文对象

spring-web 下 **resources/META-INF/services/javax.servlet.ServletContainerInitializer**，通过SPI机制调用SpringServletContainerInitializer

```
org.springframework.web.SpringServletContainerInitializer
```

#### onStartup(webAppInitializerClasses, ServletContext)

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
				}
			}
		}

		//若我们的WebApplicationInitializer的实现类 实现了Orderd接口或标注@Order注解，会进行排序
		AnnotationAwareOrderComparator.sort(initializers);
		//依次循环调用我们感兴趣的实例的onStartup方法
		for (WebApplicationInitializer initializer : initializers) {
			initializer.onStartup(servletContext);
		}
	}
}
```

## WebApplicationInitializer 接口

#### onStartup(ServletContext)

​	在Servlet 3.0+环境中实现的接口，以便以编程方式配置ServletContext，这与传统的基于web.xml的方法相反（或可能与*结合）

```java
public interface WebApplicationInitializer {
	//初始化此Web应用程序所需的任何Servlet，过滤器，侦听器，上下文参数和属性来配置给定的ServletContext
	void onStartup(ServletContext servletContext) throws ServletException;
}
```

根据官网Java配置示例注册并初始化`DispatcherServlet`spring启动的时候会调用所有实现WebApplicationInitializer的类的onStartup(ServletContext servletCxt)方法，

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {
@Override
public void onStartup(ServletContext servletCxt) {

    // Load Spring web application configuration
AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
    ac.register(AppConfig.class);
    //调用的是AbstractApplicationContext#refresh
    ac.refresh();

    // Create and register the DispatcherServlet
    DispatcherServlet servlet = new DispatcherServlet(ac);
    //调用ApplicationContext#addServlet，动态注册Servlet对象
    ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
    //设置启动时加载1
    registration.setLoadOnStartup(1);
    //添加映射
    registration.addMapping("/app/*");
}
```

<a href="#refresh">ac.refresh()</a>

## Spring MVC 找Controller流程：

 	1. 扫描整个项目 定义一个Map Bean集合
 	2. 拿到所有加了@Controller注解的类
 	3. 便利类里面所有的方法对象
 	4. 判断方法是否加了@RequestMapping注解
 	5. 把@RequestMapping注解的value作为map集a合 的key给put进去，把method对象作为value放入map集合
 	6. 根据用户的请求，拿到请求的URI url:http://localhost:8080/test  uri:/test
 	7. 使用请求的 uri 作为map的key 去map里get 看是否有值
	8. 

Controller 定义方式有2种   beanName类型和@Controller类型

Controller实现有3种 HttpRequestHandler、实现Controller和@Controller

## AnnotationConfigWebApplicationContext 

注释配置Web应用程序上下文，WebApplicationContext 接受组件类作为输入的实现-特别是@Configuration}注释类，但也使用{@inject}批注来添加普通的{@Component} 类和符合JSR-330的类

#### register(componentClasses)

注册一个或多个要处理的组件类，必须调用{ #refresh（）才能使上下文完全处理新类

```java
public class AnnotationConfigWebApplicationContext extends AbstractRefreshableWebApplicationContext
      implements AnnotationConfigRegistry {
public void register(Class<?>... componentClasses) {
		Assert.notEmpty(componentClasses, "At least one component class must be specified");
		Collections.addAll(this.componentClasses, componentClasses);
	}
```

[11]: 

## DispatcherServlet 分配请求Servlet()

HTTP请求处理程序/控制器的中央调度程序，例如适用于Web UI控制器或基于HTTP的远程服务导出程序。向注册的处理程序调度以处理 Web请求，提供便利的映射和异常处理功能

**请求处理路径 doService() -》logRequest() -》doDispatch() **

#### DispatcherServlet(WebApplicationContext)


使用给定的Web应用程序上下文创建一个新的{@code DispatcherServlet}。该构造函数在Servlet 3.0+环境中非常有用，在该环境中，可以通过{ServletContext＃addServlet} API对Servlet进行基于实例的注册

```java
public class DispatcherServlet extends FrameworkServlet {
    /** 使用此构造函数表示将忽略以下属性 /init-params 
      * <li>{@link #setContextClass(Class)} / 'contextClass'</li>
	 * <li>{@link #setContextConfigLocation(String)} / 'contextConfigLocation'</li>
	 * <li>{@link #setContextAttribute(String)} / 'contextAttribute'</li>
	 * <li>{@link #setNamespace(String)} / 'namespace'</li>
	 */
    public DispatcherServlet(WebApplicationContext webApplicationContext) {
		super(webApplicationContext);
		setDispatchOptionsRequest(true);
	}
```

#### doService(HttpServletRequest, HttpServletResponse) 

公开特定于DispatcherServlet的请求属性，并将其委托给{@link #doDispatch} *，以进行实际的分派

```java
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
   logRequest(request);

   // Keep a snapshot of the request attributes in case of an include,
   // to be able to restore the original attributes after the include.
   Map<String, Object> attributesSnapshot = null;
   if (WebUtils.isIncludeRequest(request)) {
      attributesSnapshot = new HashMap<>();
      Enumeration<?> attrNames = request.getAttributeNames();
      while (attrNames.hasMoreElements()) {
         String attrName = (String) attrNames.nextElement();
         if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
            attributesSnapshot.put(attrName, request.getAttribute(attrName));
         }
      }
   }

   // Make framework objects available to handlers and view objects.
   request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
   request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
   request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
   request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

   if (this.flashMapManager != null) {
      FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
      if (inputFlashMap != null) {
         request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
      }
      request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
      request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
   }

   try {
      doDispatch(request, response);
   }
   finally {
      if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
         // Restore the original attribute snapshot, in case of an include.
         if (attributesSnapshot != null) {
            restoreAttributesAfterInclude(request, attributesSnapshot);
         }
      }
   }
}
```

#### logRequest(HttpServletRequest) 

```java
private void logRequest(HttpServletRequest request) {
   LogFormatUtils.traceDebug(logger, traceOn -> {
      String params;
      if (isEnableLoggingRequestDetails()) {
         params = request.getParameterMap().entrySet().stream()
               .map(entry -> entry.getKey() + ":" + Arrays.toString(entry.getValue()))
               .collect(Collectors.joining(", "));
      }
      else {
         params = (request.getParameterMap().isEmpty() ? "" : "masked");
      }

      String queryString = request.getQueryString();
      String queryClause = (StringUtils.hasLength(queryString) ? "?" + queryString : "");
      String dispatchType = (!request.getDispatcherType().equals(DispatcherType.REQUEST) ?
            "\"" + request.getDispatcherType().name() + "\" dispatch for " : "");
      String message = (dispatchType + request.getMethod() + " \"" + getRequestUri(request) +
            queryClause + "\", parameters={" + params + "}");

      if (traceOn) {
         List<String> values = Collections.list(request.getHeaderNames());
         String headers = values.size() > 0 ? "masked" : "";
         if (isEnableLoggingRequestDetails()) {
            headers = values.stream().map(name -> name + ":" + Collections.list(request.getHeaders(name)))
                  .collect(Collectors.joining(", "));
         }
         return message + ", headers={" + headers + "} in DispatcherServlet '" + getServletName() + "'";
      }
      else {
         return message;
      }
   });
}
```

#### doDispatch(HttpServletRequest, HttpServletResponse)

所有HTTP方法都由该方法处理。由HandlerAdapters或处理程序自己决定可接受的方法。处理实际分派给处理程序，该处理程序将通过按顺序应用servlet的HandlerMappings获得。

通过查询Servlet的已安装HandlerAdapters 来查找支持该处理程序类的第一个HandlerAdapter，即可获得HandlerAdapte

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
   HttpServletRequest processedRequest = request;
   HandlerExecutionChain mappedHandler = null;
   boolean multipartRequestParsed = false;
   // 获取异步管理器
   WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

   try {
      ModelAndView mv = null;
      Exception dispatchException = null;

      try {
         //将请求转换为多部分请求，并使多部分解析器可用,如果未设置多部分解析器，则只需使用现有请求
         processedRequest = checkMultipart(request);
         //如果processedRequest != request 说明已经将请求转换多部分请求,使用多部分请求解析
         multipartRequestParsed = (processedRequest != request);

         // 确定当前请求的处理程序
         mappedHandler = getHandler(processedRequest);
         if (mappedHandler == null) {
            //找不到处理程序->设置适当的HTTP响应状态->response 404
            noHandlerFound(processedRequest, response);
            return;
         }

         // 确定当前请求的处理程序适配器
         HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

         // 如果处理程序支持，则处理最后修改的标头
         String method = request.getMethod();
         boolean isGet = "GET".equals(method);
         if (isGet || "HEAD".equals(method)) {
     //与HttpServlet的{@code getLastModified}方法具有相同的约定。如果处理程序类不支持，则可以简单地返回-1
            long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
            if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
               return;
            }
         }

         if (!mappedHandler.applyPreHandle(processedRequest, response)) {
            return;
         }

         //  调用实际处理程序 返回 ModelAndView
         mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

         if (asyncManager.isConcurrentHandlingStarted()) {
            return;
         }
		//根据配置的参数，将传入的HttpServletRequest的请求URI转换为视图名称
         applyDefaultViewName(processedRequest, mv);
         //应用注册拦截器的postHandle方法
         mappedHandler.applyPostHandle(processedRequest, response, mv);
      }
      catch (Exception ex) {
         dispatchException = ex;
      }
      catch (Throwable err) {
         // As of 4.3, we're processing Errors thrown from handler methods as well,
         // making them available for @ExceptionHandler methods and other scenarios.
         dispatchException = new NestedServletException("Handler dispatch failed", err);
      }
      //处理程序选择和处理程序调用的结果，该结果可以是* ModelAndView或要解析为ModelAndView的Exception。
      processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
   }
   catch (Exception ex) {
      triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
   }
   catch (Throwable err) {
      triggerAfterCompletion(processedRequest, response, mappedHandler,
            new NestedServletException("Handler processing failed", err));
   }
   finally {
      if (asyncManager.isConcurrentHandlingStarted()) {
         // Instead of postHandle and afterCompletion
         if (mappedHandler != null) {
            mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
         }
      }
      else {
         // Clean up any resources used by a multipart request.
         if (multipartRequestParsed) {
            cleanupMultipart(processedRequest);
         }
      }
   }
}
```