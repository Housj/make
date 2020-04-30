## apache shiro框架简介

　　Apache Shiro是一个强大而灵活的开源安全框架，它能够干净利落地处理身份认证，授权，企业会话管理和加密。现在，使用Apache Shiro的人越来越多，因为它相当简单，相比Spring Security，Shiro可能没有Spring Security那么多强大的功能，但是在实际工作时可能并不需要那么复杂的东西，所以使用简单的Shiro就足够了。

　　以下是你可以用 Apache Shiro所做的事情：

　　![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171007140443365-1110091146.png)

　　Shiro的4大核心部分——身份验证，授权，会话管理和加密

 　　　Authentication：身份验证，简称“登录”。

 　　　Authorization：授权，给用户分配角色或者权限资源

 　　　Session Management：用户session管理器，可以让CS程序也使用session来控制权限

 　　　Cryptography：把JDK中复杂的密码加密方式进行封装。

　　除了以上功能，shiro还提供很多扩展   

　　Web Support：主要针对web应用提供一些常用功能。

　　Caching：缓存可以使应用程序运行更有效率。

　　Concurrency：多线程相关功能。

　　Testing：帮助我们进行测试相关功能

　　Run As：一个允许用户假设为另一个用户身份（如果允许）的功能，有时候在管理脚本很有用。

　　Remember Me：记住用户身份，提供类似购物车功能。

 

## 　　shiro框架认证流程

![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171007141209333-929197189.png)

　　Application Code：应用程序代码，由开发人员负责开发的

　　Subject：框架提供的接口，是与程序进行交互的对象，可以是人也可以是服务或者其他，通常就理解为用户。所有Subject 实例都必须绑定到一个SecurityManager上。我们与一个 Subject 交互，运行时shiro会自动转化为与 SecurityManager交互的特定 subject的交互。

　　SecurityManager：框架提供的接口，是 Shiro的核心，代表安全管理器对象。初始化时协调各个模块运行。然而，一旦 SecurityManager协调完毕，SecurityManager 会被单独留下，且我们只需要去操作Subject即可，无需操作SecurityManager 。 但是我们得知道，当我们正与一个 Subject 进行交互时，实质上是 SecurityManager在处理 Subject 安全操作。

　　Realm：可以开发人员编写，框架也提供一些。Realms在 Shiro中作为应用程序和安全数据之间的“桥梁”或“连接器”。他获取安全数据来判断subject是否能够登录，subject拥有什么权限。他有点类似DAO。在配置realms时，需要至少一个realm。而且Shiro提供了一些常用的 Realms来连接数据源，如LDAP数据源的JndiLdapRealm，JDBC数据源的JdbcRealm，ini文件数据源的IniRealm，properties文件数据源的PropertiesRealm，等等。我们也可以插入自己的 Realm实现来代表自定义的数据源。 像其他组件一样，Realms也是由SecurityManager控制。

　　更详细的图

![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171007141753411-114145888.png)

 

 

### 　　下面就开始shiro与SSM工程的整合使用

　　下载地址：http://shiro.apache.org/download.html

　　下载下来这两个个文件，一个jar包，一个源码文件

![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171007132553161-452617462.png)

　　首先，第一步，将jar包导入到工程中

　　然后，第二步，在web.xml中配置spring框架提供的用于整合shiro框架的过滤器（一定要放到springmvc或struts框架过滤器的前面，为了保险起见，放到最上面就好了）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!--配置过滤器，用于整合Shiro-->
<filter>
    <filter-name>shiroFilter</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>shiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　第三步，在spring配置文件中配置bean，id为shiroFilter

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!-- 配置shiro框架的过滤器工厂bean -->
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
    <property name="securityManager" ref="securityManager"/>
        <property name="loginUrl" value="/index.jsp"/>
    <property name="successUrl" value=""/>
    <property name="unauthorizedUrl" value="/noPrivilegeUI.jsp"/>
    <!-- 指定URL级别拦截策略 -->
    <property name="filterChainDefinitions">
        <value>
            /script/** = anon
        　　/style/** = anon
        　　/index.jsp* = anon
        　　/noPrivilegeUI.jsp* = anon
        　　/user/login = anon
　　　　　　 /role/findAllRoleList = perms["角色管理"]
        　　/** = authc
        </value>
    </property>
</bean>    
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　*securityManager*：这个属性是必须的。

　　*loginUrl* ：没有登录的用户请求需要登录的页面时自动跳转到登录页面，不是必须的属性，不输入地址的话会自动寻找项目web项目的根目录下的”/login.jsp”页面。

　　*successUrl* ：登录成功默认跳转页面，可以不配置，不配置则跳转至”/”。如果登陆前点击的一个需要登录的页面，则在登录自动跳转到那个需要登录的页面。不跳转到此。

　　*unauthorizedUrl* ：没有权限默认跳转的页面。

![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171007200255333-305058218.png)

　　anon: 例子/admins/** = anon 没有参数，表示可以匿名使用（需要认证(登录)）。

　　authc : 例如/admins/user/** = authc 表示需要认证(登录)才能使用，没有参数

　　roles：例子/admins/user/** = roles[admin], 参数可以写多个，多个时必须加上引号，并且参数之间用逗号分割，当有多个参数时，例如admins/user/** = roles["admin,guest"], 每个参数通过才算通过，相当于hasAllRoles()方法。

　　perms：例子/admins/user/** = perms[user:add:*], 参数可以写多个，多个时必须加上引号，并且参数之间用逗号分割，例如/admins/user/** = perms["user:add:*,user:modify:*"]，当有多个参数时必须每个参数都通过才通过，想当于　　isPermitedAll()方法。

　　Rest：例子/admins/user/** = rest[user]；根据请求的方法，相当于/admins/user/** = perms[user:method] ；其中method为post，get，delete等。

　　port：例子/admins/user/** = port[8081]； 当请求的url的端口不是8081是跳转到schemal://serverName:8081?queryString,其中schmal是协议http或https等，serverName是你访问的host,8081是url配置里port的端口，queryString

是你访问的url里的？后面的参数。

　　authcBasic：例如/admins/user/** = authcBasic； 没有参数表示httpBasic认证

　　ssl:例子/admins/user/** = ssl；没有参数，表示安全的url请求，协议为https

　　user:例如/admins/user/** = user； 没有参数表示必须存在用户，当登入操作时不做检查

　　注：anon，authcBasic，auchc，user是认证过滤器，

　　　　perms，roles，ssl，rest，port是授权过滤器

 

　　第四步：配置安全管理器

```
<!-- 配置安全管理器 -->
<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager"/>
```

 

　　第五步：写一个login方法，使用shiro提供的方式进行认证操作

　　这是我之前的login方法，以这种方法，shiro是不知道登录验证通过了的，一直不通过，所以我们要以shiro提供的认证操作方式进行登录操作

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 * 登录
 */
@RequestMapping("/login")
public String login(User user, HttpServletRequest request, Model model){
　　HttpSession session = request.getSession();
　　User newUser = userService.login(user);
　　if(newUser != null){
　　　　session.setAttribute("loginUser",newUser);
　　　　return "home/home";
　　}
　　model.addAttribute("errorMessage","用户名或者密码不正确！");
　　return "forward:/index.jsp";
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　修改后的login方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
/**
 *  使用Shiro框架提供的方式进行认证登录
 */
@RequestMapping("/login")
public String login(User user, HttpServletRequest request, Model model){
　　HttpSession session = request.getSession();
   //使用Shiro框架提供的方式进行认证
   Subject subject = SecurityUtils.getSubject(); //获得当前登录用户对象，现在状态为 “未认证”
   //用户名密码令牌
   AuthenticationToken token = new UsernamePasswordToken(user.getLoginName(), MD5Utils.md5(user.getPassword()));
   try{
   　　subject.login(token); //执行你自定义的Realm
   　　User user1 = (User) subject.getPrincipal();
   　　session.setAttribute("loginUser",user1);
   　　return "home/home";
   }catch(UnknownAccountException e){
    　　e.printStackTrace();
    　　model.addAttribute("errorMessage","此用户名不存在！");
   }catch(IncorrectCredentialsException e){
    　　e.printStackTrace();
    　　model.addAttribute("errorMessage","密码不正确！");
   }catch(Exception e){
    　　e.printStackTrace();
   }
   return "forward:/index.jsp";
}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　第六步：自定义realm，并注入给安全管理器　

　　创建一个UserRealm类，继承AuthorizingRealm这个类　

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
public class UserRealm extends AuthorizingRealm{
    @Autowired
    private UserDao userDao;
    
    @Autowired
    private PrivilegeDao privilegeDao;
    
    //认证方法
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        System.out.println("realm中的认证方法执行了。。。。");
        UsernamePasswordToken myToken = (UsernamePasswordToken) token;
        String loginName = myToken.getUsername();
        User user = new User();
        //根据用户名查询数据库中的用户,这个方法是自己写的
        if(loginName.equals("admin")){   //超级管理员拥有所有权限
            user.setLoginName(loginName);
            user.setName(loginName);
            user.setPassword(MD5Utils.md5("1234"));
        }else{
            user = userDao.findUserByLoginName(loginName);
            if(user == null){
                //用户名不存在
                return null;
            }
        }
        
        //如果能查询到，再由框架比对数据库中查询到的密码和页面提交的密码是否一致
        AuthenticationInfo info = new SimpleAuthenticationInfo(user, user.getPassword(), this.getName());
        return info;
    }
    
    //授权方法
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals){
　　　　 return null;
　　}
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

　　将自定义realm注入给安全管理器

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<!-- 配置安全管理器 -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!-- 注入realm -->
        <property name="realm" ref="userRealm"/>
    </bean>
    
    <!-- 注册自定义realm -->
    <bean id="userRealm" class="com.demo.privilege.service.realm.UserRealm"/>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 　到这里程序就可以正常运行了,登录后进入首页

![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171008124715840-1224191629.png)

　　但是点击角色管理，会进入没有权限的页面

 ![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171008124811418-1220282567.png)

　　这是因为我在spring配置文件中配置了  /role/findAllRoleList = perms["角色管理"]，而我还没有给当前用户授权，所以当前用户没有权限访问此路径

　　所以要给该用户授权，在UserRealm类中，编写授权方法

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    //授权方法
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals){
        System.out.println("realm中的授权方法执行了。。。。");
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        //两种获得已认证的登录用户的方法，效果一模一样，用哪个都行
        User user = (User) principals.getPrimaryPrincipal();
//        User user = (User) SecurityUtils.getSubject().getPrincipal();
        if(user.getLoginName().equals("admin")){  //超级管理员拥有所有权限
            List<Privilege> privilegeList = privilegeDao.findAllPrivilegeList();
            for (Privilege privilege : privilegeList) {
                info.addStringPermission(privilege.getName());
            }
        }else{
            for (Role role : user.getRoles()){
                for (Privilege privilege : role.getPrivileges()){
                    info.addStringPermission(privilege.getName());
                }
            }
        }
        return info;
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

 

　　这样，就可以正常访问了，这个授权方法是在访问/role/findAllRoleList这个路径时，shiro框架自动调用的

　　

　　我们之前进行权限控制是在spring配置文件中配置了  /role/findAllRoleList = perms["角色管理"] ，现在介绍另一种方式，也是我比较喜欢的方式， 使用注解方式进行权限控制

### 　 使用shiro的方法注解方式权限控制

　　第一步：在springmvc配置文件中开启shiro注解支持（注意：springmvc框架，放到springmvc配置文件中，struts放到spring配置文件中）

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    <!-- 保证实现了Shiro内部lifecycle函数的bean执行 -->  
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

    <!-- 开启shiro框架注解支持 -->
    <!-- 自动代理 -->
    <bean id="defaultAdvisorAutoProxyCreator" class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor">
        <!-- value必须设置为true使用cglib方式为对象创建代理对象，
            默认为false，设为false，就是使用JDK方式为对象创建代理对象，程序会出错 --> 
        <property name="proxyTargetClass" value="true"/>
    </bean>
    
    <!-- 配置shiro框架提供的切面类，用于创建代理对象 -->
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor"/>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　第二步：在Controller的方法上使用shiro注解------ @RequiresPermissions("") 执行这个方法必须有相应的权限

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    /**
     * 查询岗位列表
     */
    @RequiresPermissions("角色列表") //执行这个方法必须有角色列表这个权限
    @RequestMapping("/findAllRoleList")
    public String findAllRoleList(Model model){
        List<Role> roleList = roleService.findAllRoleList();
        model.addAttribute("roleList",roleList);
        return "role/list";
    }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　@RequiresAuthentication

　　　　验证用户是否登录，等同于方法subject.isAuthenticated() 结果为true时。

　　@ RequiresUser

　　　　验证用户是否被记忆，user有两种含义：

　　　　一种是成功登录的（subject.isAuthenticated() 结果为true）；

　　　　另外一种是被记忆的（ subject.isRemembered()结果为true）。

　　@ RequiresGuest

　　　　验证是否是一个guest的请求，与@ RequiresUser完全相反。

​    　 换言之，RequiresUser == ! RequiresGuest 。

　　　  此时subject.getPrincipal() 结果为null.

　　@ RequiresRoles

　　　　例如：@RequiresRoles("aRoleName");

​        　　  void someMethod();

　　　　如果subject中有aRoleName角色才可以访问方法someMethod。如果没有这个权限则会抛出异常[AuthorizationException](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/AuthorizationException.html)。

　　@RequiresPermissions

　　　　例如： @RequiresPermissions( {"file:read", "write:aFile.txt"} )
         　　 void someMethod();

　　　　要求subject中必须同时含有file:read和write:aFile.txt的权限才能执行方法someMethod()。否则抛出异常[AuthorizationException](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/AuthorizationException.html)。

 

 　注解方式的权限控制就完成了，但这种方式没有权限时不会自动跳转到没有权限的页面，而是直接把异常抛到页面了，所以我们要配置一个全局的异常处理

　　第三步：在springmvc配置文件中，进行如下配置，配置全局异常捕获，当shiro框架抛出权限不足异常时，跳转到权限不足提示页面

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    <!-- 全局异常处理 -->    
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">  
        <property name="exceptionMappings">  
            <props>  
                <prop key="org.apache.shiro.authz.UnauthorizedException">redirect:/noPrivilegeUI.jsp</prop>  
            </props>  
        </property>  
    </bean>  
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 　　

### 　　使用shiro提供的页面标签方式权限控制

　　再来，说一说shiro提供的页面标签　　

　　第一步：在jsp页面中引入shiro的标签库

```
<%@ taglib uri="http://shiro.apache.org/tags" prefix="shiro"%>
```

　　第二步：使用shiro的标签控制页面元素展示

![img](https://images2017.cnblogs.com/blog/1227182/201710/1227182-20171008175607871-822854849.png)

 　

#### 　　最后，使用ehcache缓存权限数据

　　ehcache是专门缓存插件，可以缓存Java对象，提高系统性能。　

　　首先，导入ehcache的jar包

　　然后，在项目中引入ehcache的配置文件

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
    <!-- 缓存目录，默认为计算机系统的临时空间，也可自己设置 -->
    <diskStore path="java.io.tmpdir"/>

    <!-- 
        (1)maxElementsInMemory：缓存内容是存在内存中的，此项是设置内存中最多可以存储多少个元素（java对象）
        (2)eternal：缓存数据是否永久有效
        (3)timeToIdleSeconds：最大空闲时间，单位是秒，缓存数据不使用时间超过设置时间，即作废((2)设置为false的情况下使用)
        (4)timeToLiveSeconds：生命周期，单位是秒，不管使不使用，达到设置时间，即作废((2)设置为false的情况下使用)
        (5)overflowToDisk：溢出到磁盘，设置为true后，缓存数量超过(1)所设置的最大数量后，超出的部分缓存到上面所设置的临时空间
        (6)maxElementsOnDisk：设置磁盘中最多可以存储多少个元素（java对象）
        (7)diskPersistent：磁盘内容的持久化，意思是当服务器（例如tomcat）重启后，磁盘上缓存的内容还要不要，true是要，false是不要
        (8)diskExpiryThreadIntervalSeconds：启动线程时间，单位是秒，每隔多长时间启动一个线程，清理掉无用数据
        (9)memoryStoreEvictionPolicy：淘汰策略，就是判定数据是否有用的标准：（1）LRU:最近最少使用（2）FIFO:先进先出，主要使用这两种策略
     -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="true"
            maxElementsOnDisk="10000000"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU"
            />
</ehcache>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

　　最后，在spring配置文件中配置缓存管理器对象，并注入给安全管理器对象　　

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
    <!-- 配置安全管理器 -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!-- 注入realm -->
        <property name="realm" ref="userRealm"/>
        <!-- 注入缓存管理器 -->
        <property name="cacheManager" ref="cacheManager"/>
    </bean>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

 

```
    <!-- 注册缓存管理器 -->
    <bean id="cacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager">
        <!-- 注入ehcache配置文件 -->
        <property name="cacheManagerConfigFile" value="classpath:ehcache.xml"/>
    </bean>
```

 

　　这样，一个shiro入门程序就完成了。