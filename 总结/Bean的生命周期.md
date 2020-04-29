### Spring框架中bean的生命周期

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





​	现在你已经了解了如何创建和加载一个Spring容器。但是一个空的容器并没有太大的价值，在你把东西放进去之前，它里面什么都没有。为了从Spring的DI(依赖注入)中受益，我们必须将应用对象装配进Spring容器中。

