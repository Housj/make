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

