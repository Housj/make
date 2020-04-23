# [JDBC](https://www.cnblogs.com/ysocean/p/7271600.html)



**目录**

- [1、什么是MyBatis?](https://www.cnblogs.com/ysocean/p/7271600.html#_label0)
- [2、为什么会有 MyBatis?](https://www.cnblogs.com/ysocean/p/7271600.html#_label1)
- [3、分析](https://www.cnblogs.com/ysocean/p/7271600.html#_label2)

 

------

[回到顶部](https://www.cnblogs.com/ysocean/p/7271600.html#_labelTop)

### 1、什么是MyBatis?

　　MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。

　　iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。

　　MyBatis 是支持普通 SQL查询，存储过程和高级映射的优秀持久层框架。MyBatis 消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Ordinary Java Objects，普通的 Java对象）映射成数据库中的记录。

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7271600.html#_labelTop)

### 2、为什么会有 MyBatis?

　　通过上面的介绍，我们知道 MyBatis 是来和数据库打交道。那么在这之前，我们是使用 JDBC 来对数据库进行增删改查等一系列操作的，而我们之所以会放弃使用 JDBC，转而使用 MyBatis 框架，这是为什么呢？或者说使用 MyBatis 对比 JDBC 有什么好处？

　　下面我们通过一段 JDBC 对 Person 表的操作来具体看看。

　　 person 表为：

```java
public class Person {
    private Long pid;
    private String pname;
    public Long getPid() {
        return pid;
    }
    public void setPid(Long pid) {
        this.pid = pid;
    }
    public String getPname() {
        return pname;
    }
    public void setPname(String pname) {
        this.pname = pname;
    }
}
```

　　JDBC 查询操作：

```java
package com.ys.dao;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
 
import javax.swing.DebugGraphics;
 
import com.ys.bean.Person;
 
public class CRUDDao {
    //MySQL数据库驱动
    public static String driverClass = "com.mysql.jdbc.Driver";
    //MySQL用户名
    public static String userName = "root";
    //MySQL密码
    public static String passWord = "root";
    //MySQL URL
    public static String url = "jdbc:mysql://localhost:3306/test";
    //定义数据库连接
    public static Connection conn = null;
    //定义声明数据库语句,使用 预编译声明 PreparedStatement提高数据库执行性能
    public static PreparedStatement ps = null;
    //定义返回结果集
    public static ResultSet rs = null;
    /**
     * 查询 person 表信息
     * @return：返回 person 的 list 集合
     */
    public static List<Person> readPerson(){
        List<Person> list = new ArrayList<>();
        try {
            //加载数据库驱动
            Class.forName(driverClass);
            //获取数据库连接
            conn = DriverManager.getConnection(url, userName, passWord);
            //定义 sql 语句,?表示占位符
            String sql = "select * from person where pname=?";
            //获取预编译处理的statement
            ps = conn.prepareStatement(sql);
            //设置sql语句中的参数，第一个为sql语句中的参数的?(从1开始)，第二个为设置的参数值
            ps.setString(1, "qzy");
            //向数据库发出 sql 语句查询，并返回结果集
            rs = ps.executeQuery();
            while (rs.next()) {
                Person p = new Person();
                p.setPid(rs.getLong(1));
                p.setPname(rs.getString(2));
                list.add(p);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            //关闭数据库连接
            if(rs!=null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(ps!=null){
                try {
                    ps.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(conn!=null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
         
        return list;
    }
    public static void main(String[] args) {
        System.out.println(CRUDDao.readPerson());
    }
}
```

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7271600.html#_labelTop)

### 3、分析

　　通过上面的例子我们可以分析如下几点：

　　①、问题一：数据库连接，使用时就创建，使用完毕就关闭，这样会对数据库进行频繁的获取连接和关闭连接，造成数据库资源浪费，影响数据库性能。

　　　　设想解决：使用数据库连接池管理数据库连接

　　②、问题二：将 sql 语句硬编码到程序中，如果sql语句修改了，那么需要重新编译 Java 代码，不利于系统维护

　　　　设想解决：将 sql 语句配置到 xml 文件中，即使 sql 语句变化了，我们也不需要对 Java 代码进行修改，重新编译

　　③、问题三：在 PreparedStatement 中设置参数，对占位符设置值都是硬编码在Java代码中，不利于系统维护

　　　　设想解决：将 sql 语句以及占位符和参数都配置到 xml 文件中

　　④、问题四：从 resultset 中遍历结果集时，对表的字段存在硬编码，不利于系统维护

　　　　设想解决：将查询的结果集自动映射为 Java 对象

　　⑤、问题五：重复性代码特别多，频繁的 try-catch

　　　　设想解决：将其整合到一个 try-catch 代码块中

　　⑥、问题六：缓存做的很差，如果存在数据量很大的情况下，这种方式性能特别低

　　　　设想解决：集成缓存框架去操作数据库

　　⑦、问题七：sql 的移植性不好，如果换个数据库，那么sql 语句可能要重写

　　　　设想解决：在 JDBC 和 数据库之间插入第三方框架，用第三方去生成 sql 语句，屏蔽数据库的差异

 

　　既然直接使用 JDBC 操作数据库有那么多的缺点，那么我们如何去解决呢？请看下面 mybatis 框架的入门实例介绍。



# [入门实例（基于XML）](https://www.cnblogs.com/ysocean/p/7277545.html)



**目录**

- [1、创建MySQL数据库：mybatisDemo和表：user](https://www.cnblogs.com/ysocean/p/7277545.html#_label0)
- [2、建立一个Java工程，并导入相应的jar包，具体目录如下](https://www.cnblogs.com/ysocean/p/7277545.html#_label1)
- [3、在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml](https://www.cnblogs.com/ysocean/p/7277545.html#_label2)
- [4、定义表所对应的实体类](https://www.cnblogs.com/ysocean/p/7277545.html#_label3)
- [5、定义操作 user 表的sql映射文件userMapper.xml　　](https://www.cnblogs.com/ysocean/p/7277545.html#_label4)
- [6、向 mybatis-configuration.xml 配置文件中注册 userMapper.xml 文件](https://www.cnblogs.com/ysocean/p/7277545.html#_label5)
- [ 7、创建测试类](https://www.cnblogs.com/ysocean/p/7277545.html#_label6)
- [补充：如何得到插入数据之后的主键值？](https://www.cnblogs.com/ysocean/p/7277545.html#_label7)
- [总结：](https://www.cnblogs.com/ysocean/p/7277545.html#_label8)

 

------

　　通过上一小节，mybatis 和 jdbc 的区别：http://www.cnblogs.com/ysocean/p/7271600.html，我们对 mybatis有了一个大致的了解，下面我们通过一个入门实例来对mybatis有更近一步的了解。

　　我们用 mybatis 来对 user 表进行增删改查操作。

 　ps:本篇博客源代码链接：[http://pan.baidu.com/s/1eSEfc8i ](http://pan.baidu.com/s/1eSEfc8i )密码：j480

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 1、创建MySQL数据库：mybatisDemo和表：user

　　这里我们就不写脚本创建了，创建完成后，再向其中插入几条数据即可。

　　user 表字段如下：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803001845334-1050272765.png)

 

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803082356912-1932891072.png)

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 2、建立一个Java工程，并导入相应的jar包，具体目录如下

　　注意：log4j和Junit不是必须的，但是我们为了查看日志以及便于测试，加入了这两个jar包

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002013365-1628654827.png)

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 3、在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
<!-- 注意：environments标签，当mybatis和spring整合之后，这个标签是不用配置的 -->
 
<!-- 可以配置多个运行环境，但是每个 SqlSessionFactory 实例只能选择一个运行环境  
  一、development:开发模式
   二、work：工作模式-->
 <environments default="development">
 <!--id属性必须和上面的default一样  -->
    <environment id="development">
    <!--事务管理器
        一、JDBC：这个配置直接简单使用了 JDBC 的提交和回滚设置。它依赖于从数据源得到的连接来管理事务范围
        二、MANAGED：这个配置几乎没做什么。它从来不提交或回滚一个连接。而它会让容器来管理事务的整个生命周期
            比如 spring 或 JEE 应用服务器的上下文，默认情况下，它会关闭连接。然而一些容器并不希望这样，
            因此如果你需要从连接中停止它，就可以将 closeConnection 属性设置为 false，比如：
            <transactionManager type="MANAGED">
                <property name="closeConnection" value="false"/>
            </transactionManager>
      -->
      <transactionManager type="JDBC"/>
      <!--dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象源  -->
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatisdemo"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
      </dataSource>
    </environment>
  </environments>
   
</configuration>
```

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 4、定义表所对应的实体类

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002247834-447237894.png)

```
package com.ys.po;
 
import java.util.Date;
 
public class User {
    private int id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    public Date getBirthday() {
        return birthday;
    }
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
    public String getAddress() {
        return address;
    }
    public void setAddress(String address) {
        this.address = address;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + ", birthday=" + birthday + ", address=" + address + "]";
    }
}
```

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 5、定义操作 user 表的sql映射文件userMapper.xml　　

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002356303-1648947603.png)

 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.po.userMapper">
 
    <!-- 根据 id 查询 user 表中的数据
       id:唯一标识符，此文件中的id值不能重复
       resultType:返回值类型，一条数据库记录也就对应实体类的一个对象
       parameterType:参数类型，也就是查询条件的类型
    -->
    <select id="selectUserById"
            resultType="com.ys.po.User" parameterType="int">
        <!-- 这里和普通的sql 查询语句差不多，对于只有一个参数，后面的 #{id}表示占位符，里面不一定要写id,写啥都可以，但是不要空着，如果有多个参数则必须写pojo类里面的属性 -->
        select * from user where id = #{id}
    </select>
   
     
    <!-- 查询 user 表的所有数据
        注意：因为是查询所有数据，所以返回的应该是一个集合,这个集合里面每个元素都是User类型
     -->
    <select id="selectUserAll" resultType="com.ys.po.User">
        select * from user
    </select>
     
    <!-- 模糊查询：根据 user 表的username字段
            下面两种写法都可以，但是要注意
            1、${value}里面必须要写value，不然会报错
            2、${}表示拼接 sql 字符串，将接收到的参数不加任何修饰拼接在sql语句中
            3、使用${}会造成 sql 注入
     -->
    <select id="selectLikeUserName" resultType="com.ys.po.User" parameterType="String">
        select * from user where username like '%${value}%'
        <!-- select * from user where username like #{username} -->
    </select>
     
    <!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        insert into user(id,username,sex,birthday,address)
            value(#{id},#{username},#{sex},#{birthday},#{address})
    </insert>
     
    <!-- 根据 id 更新 user 表的数据 -->
    <update id="updateUserById" parameterType="com.ys.po.User">
        update user set username=#{username} where id=#{id}
    </update>
     
    <!-- 根据 id 删除 user 表的数据 -->
    <delete id="deleteUserById" parameterType="int">
        delete from user where id=#{id}
    </delete>
</mapper>
```

　　

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 6、向 mybatis-configuration.xml 配置文件中注册 userMapper.xml 文件

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002534725-1062751486.png)

```
<mappers>
       <!-- 注册userMapper.xml文件，
 userMapper.xml位于com.ys.mapper这个包下，所以resource写成com/ys/mapper/userMapper.xml-->
       <mapper resource="com/ys/mapper/userMapper.xml"/>
</mappers>
```

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

###  **7、创建测试类**

```java
package com.ys.test;
 
import java.io.InputStream;
import java.util.List;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
 
import com.ys.po.User;
 
public class CRUDTest {
    //定义 SqlSession
    SqlSession session =null;
     
    @Before
    public void init(){
        //定义mybatis全局配置文件
        String resource = "mybatis-configuration.xml";
        //加载 mybatis 全局配置文件
        InputStream inputStream = CRUDTest.class.getClassLoader()
                                    .getResourceAsStream(resource);
        //构建sqlSession的工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //根据 sqlSessionFactory 产生 session
        session = sessionFactory.openSession();
    }
     
    //根据id查询user表数据
    @Test
    public void testSelectUserById(){
        /*这个字符串由 userMapper.xml 文件中 两个部分构成
            <mapper namespace="com.ys.po.userMapper"> 的 namespace 的值
            <select id="selectUserById" > id 值*/
        String statement = "com.ys.po.userMapper.selectUserById";
        User user = session.selectOne(statement, 1);
        System.out.println(user);
        session.close();
    }
     
    //查询所有user表所有数据
    @Test
    public void testSelectUserAll(){
        String statement = "com.ys.po.userMapper.selectUserAll";
        List<User> listUser = session.selectList(statement);
        for(User user : listUser){
            System.out.println(user);
        }
        session.close();
    }
     
    //模糊查询：根据 user 表的username字段
    @Test
    public void testSelectLikeUserName(){
        String statement = "com.ys.po.userMapper.selectLikeUserName";
        List<User> listUser = session.selectList(statement, "%t%");
        for(User user : listUser){
            System.out.println(user);
        }
        session.close();
         
    }
    //向 user 表中插入一条数据
    @Test
    public void testInsertUser(){
        String statement = "com.ys.po.userMapper.insertUser";
        User user = new User();
        user.setUsername("Bob");
        user.setSex("女");
        session.insert(statement, user);
        //提交插入的数据
        session.commit();
        session.close();
    }
     
    //根据 id 更新 user 表的数据
    @Test
    public void testUpdateUserById(){
        String statement = "com.ys.po.userMapper.updateUserById";
        //如果设置的 id不存在，那么数据库没有数据更改
        User user = new User();
        user.setId(4);
        user.setUsername("jim");
        session.update(statement, user);
        session.commit();
        session.close();
    }
     
 
    //根据 id 删除 user 表的数据
    @Test
    public void testDeleteUserById(){
        String statement = "com.ys.po.userMapper.deleteUserById";
        session.delete(statement,4);
        session.commit();
        session.close();
    }
}
```

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 补充：如何得到插入数据之后的主键值？

第一种：数据库设置主键自增机制

　　　　userMapper.xml 文件中定义：

```xml
<!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        <!-- 将插入的数据主键返回到 user 对象中
             keyProperty:将查询到的主键设置到parameterType 指定到对象的那个属性
            select LAST_INSERT_ID()：查询上一次执行insert 操作返回的主键id值，只适用于自增主键
             resultType:指定 select LAST_INSERT_ID() 的结果类型
             order:AFTER，相对于 select LAST_INSERT_ID()操作的顺序
         -->
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into user(username,sex,birthday,address)
            value(#{username},#{sex},#{birthday},#{address})
    </insert>
```

　也可以在select标签下写以下的属性：

```xml
<select useGeneratedKeys="true"keyProperty="id" keyColumn="id"/>
```

　　

第二种：非自增主键机制

```xml
<!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        <!-- 将插入的数据主键返回到 user 对象中
        流程是：首先通过 select UUID()得到主键值，然后设置到 user 对象的id中，在进行 insert 操作
             keyProperty:将查询到的主键设置到parameterType 指定到对象的那个属性
             select UUID()：得到主键的id值，注意这里是字符串
             resultType:指定 select UUID() 的结果类型
             order:BEFORE，相对于 select UUID()操作的顺序
         -->
        <selectKey keyProperty="id" resultType="String" order="BEFORE">
            select UUID()
        </selectKey>
        insert into user(id,username,sex,birthday,address)
            value(#{id},#{username},#{sex},#{birthday},#{address})
    </insert>
```

　　

　　　　

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7277545.html#_labelTop)

### 总结：

　　①、parameterType:指定输入参数的类型

　　②、resultType:指定输出结果的类型，在select中如果查询结果是集合，那么也表示集合中每个元素的类型

　　③、#{}:表示占位符，用来接收输入参数，类型可以是简单类型，pojo,HashMap等等

　　　　如果接收简单类型，#{}可以写成 value 或者其他名称

　　　　如果接收 pojo 对象值，通过 OGNL 读取对象中的属性值，即属性.属性.属性...的方式获取属性值

　　④、${}:表示一个拼接符，会引起 sql 注入，不建议使用　　

　　　　用来接收输入参数，类型可以是简单类型，pojo,HashMap等等

　　　　如果接收简单类型，${}里面只能是 value

　　　　如果接收 pojo 对象值，通过 OGNL 读取对象中的属性值，即属性.属性.属性...的方式获取属性值



# [入门实例（基于注解）](https://www.cnblogs.com/ysocean/p/7282639.html)



**目录**

- [1、创建MySQL数据库：mybatisDemo和表：user](https://www.cnblogs.com/ysocean/p/7282639.html#_label0)
- [2、建立一个Java工程，并导入相应的jar包，具体目录如下](https://www.cnblogs.com/ysocean/p/7282639.html#_label1)
- [3、在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml](https://www.cnblogs.com/ysocean/p/7282639.html#_label2)
- [4、定义表所对应的实体类](https://www.cnblogs.com/ysocean/p/7282639.html#_label3)
- [5、定义操作 user 表的注解接口 UserMapper.java](https://www.cnblogs.com/ysocean/p/7282639.html#_label4)
- [6、向 mybatis-configuration.xml 配置文件中注册 UserMapper.java 文件](https://www.cnblogs.com/ysocean/p/7282639.html#_label5)
- [ 7、创建测试类](https://www.cnblogs.com/ysocean/p/7282639.html#_label6)

 

------

[回到顶部](https://www.cnblogs.com/ysocean/p/7282639.html#_labelTop)

### 1、创建MySQL数据库：mybatisDemo和表：user

　　详情参考：[mybatis 详解（二）------入门实例（基于XML）](http://www.cnblogs.com/ysocean/p/7277545.html) 一致

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7282639.html#_labelTop)

### 2、建立一个Java工程，并导入相应的jar包，具体目录如下

 　详情参考：[mybatis 详解（二）------入门实例（基于XML）](http://www.cnblogs.com/ysocean/p/7277545.html) 一致

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7282639.html#_labelTop)

### 3、在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml

 　详情参考：[mybatis 详解（二）------入门实例（基于XML）](http://www.cnblogs.com/ysocean/p/7277545.html) 一致

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7282639.html#_labelTop)

### 4、定义表所对应的实体类

 　详情参考：[mybatis 详解（二）------入门实例（基于XML）](http://www.cnblogs.com/ysocean/p/7277545.html) 一致

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7282639.html#_labelTop)

### 5、定义操作 user 表的注解接口 UserMapper.java

```java
package com.ys.annocation;
 
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
 
import com.ys.po.User;
 
public interface UserMapper {
    //根据 id 查询 user 表数据
    @Select("select * from user where id = #{id}")
    public User selectUserById(int id) throws Exception;
 
    //向 user 表插入一条数据
    @Insert("insert into user(username,sex,birthday,address) value(#{username},#{sex},#{birthday},#{address})")
    public void insertUser(User user) throws Exception;
     
    //根据 id 修改 user 表数据
    @Update("update user set username=#{username},sex=#{sex} where id=#{id}")
    public void updateUserById(User user) throws Exception;
     
    //根据 id 删除 user 表数据
    @Delete("delete from user where id=#{id}")
    public void deleteUserById(int id) throws Exception;
     
}
```

　　

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7282639.html#_labelTop)

### 6、向 mybatis-configuration.xml 配置文件中注册 UserMapper.java 文件

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803231541162-1969485958.png)

```xml
<mappers>
       <mapper class="com.ys.annocation.UserMapper"/>
</mappers>
```

　　

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7282639.html#_labelTop)

###  **7、创建测试类**

```java
package com.ys.test;
 
import java.io.InputStream;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
 
import com.ys.annocation.UserMapper;
import com.ys.po.User;
 
public class UserAnnocationTest {
    //定义 SqlSession
    SqlSession session =null;
     
    @Before
    public void init(){
        //定义mybatis全局配置文件
        String resource = "mybatis-configuration.xml";
        //加载 mybatis 全局配置文件
        InputStream inputStream = CRUDTest.class.getClassLoader()
                                    .getResourceAsStream(resource);
        //构建sqlSession的工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //根据 sqlSessionFactory 产生 session
        session = sessionFactory.openSession();
    }
     
    //注解的增删改查方法测试
    @Test
    public void testAnncationCRUD() throws Exception{
        //根据session获取 UserMapper接口
        UserMapper userMapper = session.getMapper(UserMapper.class);
        //调用selectUserById()方法
        User user = userMapper.selectUserById(1);
        System.out.println(user);
         
        //调用  insertUser() 方法
        User user1 = new User();
        user1.setUsername("aliks");
        user1.setSex("不详");
        userMapper.insertUser(user1);
         
        //调用 updateUserById() 方法
        User user2 = new User();
        user2.setId(6);
        user2.setUsername("lbj");
        userMapper.updateUserById(user2);
         
        //调用 () 方法
        userMapper.deleteUserById(6);
         
        session.commit();
        session.close();
    }
}
```

 

注意：注解配置我们不需要 userMapper.xml 文件了　



# [properties以及别名定义](https://www.cnblogs.com/ysocean/p/7287972.html)



**目录**

- [1、我们将 数据库的配置语句写在 db.properties 文件中](https://www.cnblogs.com/ysocean/p/7287972.html#_label0)
- [2、在  mybatis-configuration.xml 中加载db.properties文件并读取](https://www.cnblogs.com/ysocean/p/7287972.html#_label1)
- [通过源码我们可以分析读取优先级：](https://www.cnblogs.com/ysocean/p/7287972.html#_label2)
- [1、mybatis 默认支持的别名](https://www.cnblogs.com/ysocean/p/7287972.html#_label3)
- [2、自定义别名　　](https://www.cnblogs.com/ysocean/p/7287972.html#_label4)

 

------

　　上一篇博客我们介绍了mybatis的增删改查入门实例，我们发现在 mybatis-configuration.xml 的配置文件中，对数据库的配置都是硬编码在这个xml文件中，如下图，那么我们如何改进这个写法呢？

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804220404022-1871062647.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/7287972.html#_labelTop)

### 1、我们将 数据库的配置语句写在 db.properties 文件中

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssm
jdbc.username=root
jdbc.password=root
```

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7287972.html#_labelTop)

### 2、在  mybatis-configuration.xml 中加载db.properties文件并读取

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!-- 加载数据库属性文件 -->
<properties resource="db.properties">
</properties>
 <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!--dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象源  -->
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```

　　

　　如果数据库有变化，我们就可以通过修改 db.properties 文件来修改，而不用去修改 mybatis-configuration.xml 文件

注意：我们也可以在<properties></properties>中手动增加属性

```
<!-- 加载数据库属性文件 -->
<properties resource="db.properties">
    <property name="username" value="aaa"/>
</properties>
```

　　那么这个时候是读取的username 是以 db.properties 文件中的 root 为准，还是以自己配置的 aaa 为准呢?

我们先看一段 properties 文件加载的源码

```java
private void propertiesElement(XNode context) throws Exception {
  if (context != null) {
    /**
     *  解析properties 属性中指定的属性。
     */
    Properties defaults = context.getChildrenAsProperties();
    String resource = context.getStringAttribute("resource"); //resource 制定的属性路径
    String url = context.getStringAttribute("url"); //url制定的属性路径
    if (resource != null && url != null) {
      throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
    }
    /**
     * 根据 properties 元素中的 resource 属性读取类路径下属性文件，并覆盖properties 属性中指定的同名属性。
     */
    if (resource != null) {
      defaults.putAll(Resources.getResourceAsProperties(resource));
    } else if (url != null) {
      /**
       * 根据properties元素中的url属性指定的路径读取属性文件，并覆盖properties 属性中指定的同名属性。
       */
      defaults.putAll(Resources.getUrlAsProperties(url));
    }
    /**
     *  获取方法参数传递的properties
     *  创建XMLConfigBuilder实例时，this.configuration.setVariables(props);
     */
    Properties vars = configuration.getVariables();
    if (vars != null) {
      defaults.putAll(vars);
    }
    parser.setVariables(defaults);
    configuration.setVariables(defaults);
  }
}
```

[回到顶部](https://www.cnblogs.com/ysocean/p/7287972.html#_labelTop)

### 通过源码我们可以分析读取优先级：

　　　　1、在 properties 内部自定义的属性值第一个被读取

　　　　2、然后读取 resource 路径表示文件中的属性，如果有它会覆盖已经读取的属性；如果 resource 路径不存在，那么读取 url 表示路径文件中的属性，如果有它会覆盖第一步读取的属性值

　　　　3、最后读取 parameterType 传递的属性值，它会覆盖已读取的同名的属性

 

　　前面两步好理解，第三步我们可以举个例子来看：

　　　　我们在 userMapper.xml 文件中进行模糊查询

``"selectLikeUserName"` `resultType=``"com.ys.po.User"` `parameterType=``"String"``>``  ``select * from user where username like ``'%${jdbc.username}%'``  `````

　　　　这个时候你会发现无论你后台传给这个查询语句什么参数，都是 select * from user where username like '%root%'

　　　　

 

 

## mybatis 的别名配置　　

　　在 userMapper.xml 文件中，我们可以看到resultType 和 parameterType 需要指定，这这个值往往都是全路径，不方便开发，那么我们就可以对这些属性进行一些别名设置

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230103850-1701803236.png)

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7287972.html#_labelTop)

### 1、mybatis 默认支持的别名

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230414350-900097506.png)

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230557694-737797674.png)

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7287972.html#_labelTop)

### 2、自定义别名　　

　　一、定义单个别名

　　　　首先在全局配置文件 mybatis-configuration.xml 文件中添加如下配置：是在<configuration>标签下

```
<!-- 定义别名 -->
<typeAliases>
    <typeAlias type="com.ys.po.User" alias="user"/>
</typeAliases>
```

　　　　第二步通过 user 引用

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804231055990-1789427240.png)

　　

　　二、批量定义别名

　　　　在全局配置文件 mybatis-configuration.xml 文件中添加如下配置：是在<configuration>标签下

```
<!-- 定义别名 -->
<typeAliases>
    <!-- mybatis自动扫描包中的po类，自动定义别名，别名是类名(首字母大写或小写都可以,一般用小写) -->
    <package name="com.ys.po"/>
</typeAliases>
```

　　　　引用的时候类名的首字母大小写都可以

# [动态SQL](https://www.cnblogs.com/ysocean/p/7289529.html)



**目录**

- [1、动态SQL:if 语句](https://www.cnblogs.com/ysocean/p/7289529.html#_label0)
- [2、动态SQL:if+where 语句](https://www.cnblogs.com/ysocean/p/7289529.html#_label1)
- [3、动态SQL:if+set 语句](https://www.cnblogs.com/ysocean/p/7289529.html#_label2)
- [4、动态SQL:choose(when,otherwise) 语句](https://www.cnblogs.com/ysocean/p/7289529.html#_label3)
- [5、动态SQL:trim 语句](https://www.cnblogs.com/ysocean/p/7289529.html#_label4)
- [6、动态SQL: SQL 片段](https://www.cnblogs.com/ysocean/p/7289529.html#_label5)
- [7、动态SQL: foreach 语句](https://www.cnblogs.com/ysocean/p/7289529.html#_label6)
- [8、总结](https://www.cnblogs.com/ysocean/p/7289529.html#_label7)

 

------

　　前面几篇博客我们通过实例讲解了用mybatis对一张表进行的CRUD操作，但是我们发现写的 SQL 语句都比较简单，如果有比较复杂的业务，我们需要写复杂的 SQL 语句，往往需要拼接，而拼接 SQL ，稍微不注意，由于引号，空格等缺失可能都会导致错误。

　　那么怎么去解决这个问题呢？这就是本篇所讲的使用 mybatis 动态SQL，通过 if, choose, when, otherwise, trim, where, set, foreach等标签，可组合成非常灵活的SQL语句，从而在提高 SQL 语句的准确性的同时，也大大提高了开发人员的效率。

 

　　我们以 User 表为例来说明：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170805120304397-3670858.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 1、动态SQL:if 语句

　　根据 username 和 sex 来查询数据。如果username为空，那么将只根据sex来查询；反之只根据username来查询

　　首先不使用 动态SQL 来书写

```xml
<select id="selectUserByUsernameAndSex"
        resultType="user" parameterType="com.ys.po.User">
    <!-- 这里和普通的sql 查询语句差不多，对于只有一个参数，后面的 #{id}表示占位符，里面不一定要写id,
            写啥都可以，但是不要空着，如果有多个参数则必须写pojo类里面的属性 -->
    select * from user where username=#{username} and sex=#{sex}
</select>
```

　　

　　上面的查询语句，我们可以发现，如果 #{username} 为空，那么查询结果也是空，如何解决这个问题呢？使用 if 来判断

```xml
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user where
        <if test="username != null">
           username=#{username}
        </if>
        <if test="username != null">
           and sex=#{sex}
        </if>
</select>
```

　　这样写我们可以看到，如果 sex 等于 null，那么查询语句为 select * from user where username=#{username},但是如果usename 为空呢？那么查询语句为 select * from user where and sex=#{sex}，这是错误的 SQL 语句，如何解决呢？请看下面的 where 语句

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 2、动态SQL:if+where 语句

```xml
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <where>
        <if test="username != null">
           username=#{username}
        </if>
        <if test="username != null">
           and sex=#{sex}
        </if>
    </where>
</select>
```

　　这个“where”标签会知道如果它包含的标签中有返回值的话，它就插入一个‘where’。此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 3、动态SQL:if+set 语句

　　同理，上面的对于查询 SQL 语句包含 where 关键字，如果在进行更新操作的时候，含有 set 关键词，我们怎么处理呢？

```xml
<!-- 根据 id 更新 user 表的数据 -->
<update id="updateUserById" parameterType="com.ys.po.User">
    update user u
        <set>
            <if test="username != null and username != ''">
                u.username = #{username},
            </if>
            <if test="sex != null and sex != ''">
                u.sex = #{sex}
            </if>
        </set>
     where id=#{id}
</update>
```

　　这样写，如果第一个条件 username 为空，那么 sql 语句为：update user u set u.sex=? where id=?

　　　　　　如果第一个条件不为空，那么 sql 语句为：update user u set u.username = ? ,u.sex = ? where id=?

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 4、动态SQL:choose(when,otherwise) 语句

　　有时候，我们不想用到所有的查询条件，只想选择其中的一个，查询条件有一个满足即可，使用 choose 标签可以解决此类问题，类似于 Java 的 switch 语句

```xml
<select id="selectUserByChoose" resultType="com.ys.po.User" parameterType="com.ys.po.User">
      select * from user
      <where>
          <choose>
              <when test="id !='' and id != null">
                  id=#{id}
              </when>
              <when test="username !='' and username != null">
                  and username=#{username}
              </when>
              <otherwise>
                  and sex=#{sex}
              </otherwise>
          </choose>
      </where>
  </select>
```

　　也就是说，这里我们有三个条件，id,username,sex，只能选择一个作为查询条件

　　　　如果 id 不为空，那么查询语句为：select * from user where  id=?

　　　　如果 id 为空，那么看username 是否为空，如果不为空，那么语句为 select * from user where username=?;

　　　　如果 username 为空，那么查询语句为 select * from user where sex=?

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 5、动态SQL:trim 语句

　　trim标记是一个格式化的标记，可以完成set或者是where标记的功能

　　①、用 trim 改写上面第二点的 if+where 语句

```xml
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
        select * from user
        <!-- <where>
            <if test="username != null">
               username=#{username}
            </if>
             
            <if test="username != null">
               and sex=#{sex}
            </if>
        </where>  -->
        <trim prefix="where" prefixOverrides="and | or">
            <if test="username != null">
               and username=#{username}
            </if>
            <if test="sex != null">
               and sex=#{sex}
            </if>
        </trim>
    </select>-
        <foreach collection="ids" item="id" open="and (" close=")" separator="or">
            id=#{id}
        </foreach>
    </where>
</select>
```

　　测试：

```java
//根据id集合查询user表数据
@Test
public void testSelectUserByListId(){
    String statement = "com.ys.po.userMapper.selectUserByListId";
    UserVo uv = new UserVo();
    List<Integer> ids = new ArrayList<>();
    ids.add(1);
    ids.add(2);
    ids.add(3);
    uv.setIds(ids);
    List<User> listUser = session.selectList(statement, uv);
    for(User u : listUser){
        System.out.println(u);
    }
    session.close();
}
```

　　

③、我们用 foreach 来改写 select * from user where id in (1,2,3)

```xml
<select id="selectUserByListId" parameterType="com.ys.vo.UserVo" resultType="com.ys.po.User">
        select * from user
        <where>
            <!--
                collection:指定输入对象中的集合属性
                item:每次遍历生成的对象
                open:开始遍历时的拼接字符串
                close:结束时拼接的字符串
                separator:遍历对象之间需要拼接的字符串
                select * from user where 1=1 and id in (1,2,3)
              -->
            <foreach collection="ids" item="id" open="and id in (" close=") " separator=",">
                #{id}
            </foreach>
        </where>
    </select>
```

　　

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 8、总结

　　其实动态 sql 语句的编写往往就是一个拼接的问题，为了保证拼接准确，我们最好首先要写原生的 sql 语句出来，然后在通过 mybatis 动态sql 对照着改，防止出错。

# [通过mapper接口加载映射文件](https://www.cnblogs.com/ysocean/p/7301548.html)



**目录**

- [1、定义 userMapper 接口](https://www.cnblogs.com/ysocean/p/7301548.html#_label0)
- [2、在全局配置文件 mybatis-configuration.xml 文件中加载 UserMapper 接口（单个加载映射文件）](https://www.cnblogs.com/ysocean/p/7301548.html#_label1)
- [3、编写UserMapper.xml 文件](https://www.cnblogs.com/ysocean/p/7301548.html#_label2)
- [4、测试](https://www.cnblogs.com/ysocean/p/7301548.html#_label3)
- [5、批量加载映射文件](https://www.cnblogs.com/ysocean/p/7301548.html#_label4)
- [6、注意　](https://www.cnblogs.com/ysocean/p/7301548.html#_label5)

 

------

　　通过 mapper 接口加载映射文件，这对于后面 ssm三大框架 的整合是非常重要的。那么什么是通过 mapper 接口加载映射文件呢？

　　我们首先看以前的做法，在全局配置文件 mybatis-configuration.xml 通过 <mappers> 标签来加载映射文件，那么如果我们项目足够大，有很多映射文件呢，难道我们每一个映射文件都这样加载吗，这样肯定是不行的，那么我们就需要使用 mapper 接口来加载映射文件

　　以前的做法：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170807212857174-537878858.png)

　　改进做法：使用 mapper 接口来加载映射文件

[回到顶部](https://www.cnblogs.com/ysocean/p/7301548.html#_labelTop)

### 1、定义 userMapper 接口

```
package com.ys.mapper;
 
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
 
import com.ys.po.User;
 
public interface UserMapper {
    //根据 id 查询 user 表数据
    public User selectUserById(int id) throws Exception;
 
    //向 user 表插入一条数据
    public void insertUser(User user) throws Exception;
     
    //根据 id 修改 user 表数据
    public void updateUserById(User user) throws Exception;
     
    //根据 id 删除 user 表数据
    public void deleteUserById(int id) throws Exception;
}
```

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7301548.html#_labelTop)

### 2、在全局配置文件 mybatis-configuration.xml 文件中加载 UserMapper 接口（单个加载映射文件）

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170807213755002-286929257.png)

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7301548.html#_labelTop)

### 3、编写UserMapper.xml 文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.mapper.UserMapper">
 
     
    <!-- 根据 id 查询 user 表中的数据
       id:唯一标识符，此文件中的id值不能重复
       resultType:返回值类型，一条数据库记录也就对应实体类的一个对象
       parameterType:参数类型，也就是查询条件的类型
    -->
    <select id="selectUserById"
            resultType="com.ys.po.User" parameterType="int">
        <!-- 这里和普通的sql 查询语句差不多，后面的 #{id}表示占位符，里面不一定要写id,写啥都可以，但是不要空着 -->
        select * from user where id = #{id1}
    </select>
     
     
     
    <!-- 根据 id 更新 user 表的数据 -->
    <update id="updateUserById" parameterType="com.ys.po.User">
        update user u
            <trim prefix="set" suffixOverrides=",">
                <if test="username != null and username != ''">
                    u.username = #{username},
                </if>
                <if test="sex != null and sex != ''">
                    u.sex = #{sex},
                </if>
            </trim>
         where id=#{id}
    </update>
     
     
    <!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        <!-- 将插入的数据主键返回到 user 对象中
             keyProperty:将查询到的主键设置到parameterType 指定到对象的那个属性
             select LAST_INSERT_ID()：查询上一次执行insert 操作返回的主键id值，只适用于自增主键
             resultType:指定 select LAST_INSERT_ID() 的结果类型
             order:AFTER，相对于 select LAST_INSERT_ID()操作的顺序
         -->
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into user(username,sex,birthday,address)
            value(#{username},#{sex},#{birthday},#{address})
    </insert>
     
    <!-- 根据 id 删除 user 表的数据 -->
    <delete id="deleteUserById" parameterType="int">
        delete from user where id=#{id}
    </delete>
     
</mapper>
```

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7301548.html#_labelTop)

### 4、测试

```java
//根据id查询user表数据
    @Test
    public void testSelectUserById() throws Exception{
        //获取mapper接口
        UserMapper userMapper = session.getMapper(UserMapper.class);
        User user = userMapper.selectUserById(1);
        System.out.println(user);
        session.close();
    }
```

　　

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7301548.html#_labelTop)

### 5、批量加载映射文件

```xml
<mappers>
  <!--批量加载mapper 指定 mapper 接口的包名，mybatis自动扫描包下的mapper接口进行加载 -->
       <package name="com.ys.mapper"/>
</mappers>
```

　　

　

[回到顶部](https://www.cnblogs.com/ysocean/p/7301548.html#_labelTop)

### 6、注意　

　　1、UserMapper 接口必须要和 UserMapper.xml 文件同名且在同一个包下，也就是说 UserMapper.xml 文件中的namespace是UserMapper接口的全类名

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170807214006877-162394845.png)

 

　　2、UserMapper接口中的方法名和 UserMapper.xml 文件中定义的 id 一致

 

　　3、UserMapper接口输入参数类型要和 UserMapper.xml 中定义的 parameterType 一致

 

　　4、UserMapper接口返回数据类型要和 UserMapper.xml 中定义的 resultType 一致



# [一对一、一对多、多对多](https://www.cnblogs.com/ysocean/p/7309308.html)

**目录**

- [1、一对一](https://www.cnblogs.com/ysocean/p/7309308.html#_label0)
- [2、一对多](https://www.cnblogs.com/ysocean/p/7309308.html#_label1)
- [3、多对多](https://www.cnblogs.com/ysocean/p/7309308.html#_label2)

 

------

　　前面几篇博客我们用mybatis能对单表进行增删改查操作了，也能用动态SQL书写比较复杂的sql语句。但是在实际开发中，我们做项目不可能只是单表操作，往往会涉及到多张表之间的关联操作。那么我们如何用 mybatis 处理多表之间的关联操作呢？请看本篇博客详解。

　　本篇详细代码：http://pan.baidu.com/s/1eSzmst8 密码：3n3o

[回到顶部](https://www.cnblogs.com/ysocean/p/7309308.html#_labelTop)

### 1、一对一

　　我们以用户表 user 和订单表 orders 为例。设定一个订单只能由一个 用户创建，那么由订单到用户就是一对一的关系。

**①、创建用户表 user 和订单表 orders**

　　用户表 user

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170808213704558-288865314.png)

　　订单表 orders

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170808213742652-922604563.png)

**②、创建项目工程，导入相应的 jar 包**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170808220337667-409911824.png)

 

 **③、创建实体类**

　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170808220424792-670352773.png)**

　　User.java

```
package com.ys.po;
 
public class User {
    //用户ID
    private int id;
    //用户姓名
    private String username;
    //用户性别
    private String sex;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + "]";
    }
}
```

　　Orders.java

```
package com.ys.po;
 
public class Orders {
    //订单ID
    private int id;
    //用户ID
    private int userId;
    //订单数量
    private String number;
    //和用户表构成一对一的关系，即一个订单只能由一个用户创建
    private User user;
     
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    public int getUserId() {
        return userId;
    }
 
    public void setUserId(int userId) {
        this.userId = userId;
    }
 
    public String getNumber() {
        return number;
    }
 
    public void setNumber(String number) {
        this.number = number;
    }
 
    public User getUser() {
        return user;
    }
 
    public void setUser(User user) {
        this.user = user;
    }
 
    @Override
    public String toString() {
        return "Orders [id=" + id + ", userId=" + userId + ", number=" + number
                + ", user=" + user + "]";
    }
 
}
```

　　

**④、创建 OrderMapper 接口和 OrderMapper.xml 文件**

　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170808221022792-150096842.png)**

　　由于我们采用 Mapper 代理加载 xxxMapper.xml 文件，这里我们重复一下 Mapper 代理所需的条件，接口和xml文件必须满足以下几个条件：

　　1、接口必须要和 xml 文件同名且在同一个包下，也就是说 xml 文件中的namespace是接口的全类名　　

　　2、接口中的方法名和xml 文件中定义的 id 一致

　　3、接口输入参数类型要和xml 中定义的 parameterType 一致

　　4、接口返回数据类型要和xml 中定义的 resultType 一致

　　详细介绍参考上一篇博客：http://www.cnblogs.com/ysocean/p/7301548.html

　　OrderMapper 接口

```
package one.to.one.mapper;
 
import com.ys.po.Orders;
import com.ys.po.User;
 
public interface OrdersMapper {
    /**
     * 方式一：嵌套结果
     * select * from orders o,user u where o.user_id=u.id and o.id=#{id}
     * @param orderId
     * @return
     */
    //根据订单ID查询订单和用户信息
    public Orders selectOrderAndUserByOrderID(int orderId);
     
    /**
     * 方式二：嵌套查询
     * select * from order WHERE id=1;//得到user_id
     * select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
     * @param userID
     * @return
     */
    //根据订单ID得到订单信息(包含user_id)
    public Orders getOrderByOrderId(int orderId);
    //根据用户ID查询用户信息
    public User getUserByUserId(int userID);
 
}
```

　　

　　OrderMapper .xml文件

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="one.to.one.mapper.OrdersMapper">
    <!--
    嵌套结果：使用嵌套结果映射来处理重复的联合结果的子集
                               封装联表查询的数据(去除重复的数据)
     select * from orders o,user u where o.user_id=u.id and o.id=#{id}
     -->
    <select id="selectOrderAndUserByOrderID" resultMap="getOrderAndUser">
        select * from orders o,user u where o.user_id=u.id and o.id=#{id}
    </select>
    <resultMap type="com.ys.po.Orders" id="getOrderAndUser">
        <!--
            id:指定查询列表唯一标识，如果有多个唯一标识，则配置多个id
            column:数据库对应的列
            property:实体类对应的属性名
          -->
        <id column="id" property="id"/>
        <result column="user_id" property="userId"/>
        <result column="number" property="number"/>
        <!--association:用于映射关联查询单个对象的信息
            property:实体类对应的属性名
            javaType:实体类对应的全类名
          -->
        <association property="user" javaType="com.ys.po.User">
            <!--
                id:指定查询列表唯一标识，如果有多个唯一标识，则配置多个id
                column:数据库对应的列
                property:实体类对应的属性名
              -->
            <id column="id" property="id"/>
            <result column="username" property="username"/>
            <result column="sex" property="sex"/>
        </association>
    </resultMap>
     
     
    <!--
         方式二：嵌套查询：通过执行另外一个SQL映射语句来返回预期的复杂类型
         select user_id from order WHERE id=1;//得到user_id
         select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
         property:别名(属性名)    column：列名 -->
    <select id="getOrderByOrderId" resultMap="getOrderMap">
        select * from order where id=#{id}
    </select>
    <resultMap type="com.ys.po.Orders" id="getOrderMap">
        <id column="id" property="id"/>
        <result column="number" property="number"/>
        <association property="userId"  column="id" select="getUserByUserId">
         
        </association>
    </resultMap>
    <select id="getUserByUserId" resultType="com.ys.po.User">
        select * from user where id=#{id}
    </select>
     
</mapper>
```

　　

⑤、**向 mybatis-configuration.xml 配置文件中注册 OrderMapper.xml 文件**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!-- 加载数据库属性文件 -->
<properties resource="db.properties"></properties>
<!-- 定义别名 -->
<typeAliases>
    <!-- mybatis自动扫描包中的po类，自动定义别名，别名是类名(首字母大写或小写都可以,一般用小写) -->
    <package name="com.ys.po"/>
</typeAliases>
 <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!--dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象源  -->
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
   
  <mappers>
         <!-- 通过OrdersMapper接口注册OrdersMapper.xml文件，
            必须保证：接口和xml在同一个包下，而且名字一样
            OrdersMapper接口的方法名和OrdersMapper.xml文件的id一样
            OrdersMapper接口的输出输出参数和OrdersMapper.xml文件resultType，parameterType类型一样
         -->
         <mapper class="one.to.one.mapper.OrdersMapper"/>
  </mappers>
</configuration>
```

　　

**⑥、测试**

```
package one.to.one.mapper;
 
import java.io.InputStream;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
 
import com.ys.po.Orders;
 
public class OneToOneTest {
    //定义 SqlSession
    SqlSession session =null;
     
    @Before
    public void init(){
        //定义mybatis全局配置文件
        String resource = "mybatis-configuration.xml";
        //加载 mybatis 全局配置文件
        InputStream inputStream = OneToOneTest.class.getClassLoader()
                                    .getResourceAsStream(resource);
        //构建sqlSession的工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //根据 sqlSessionFactory 产生 session
        session = sessionFactory.openSession();
    }
     
    /**
     * 方式一：嵌套结果
     * select * from orders o,user u where o.user_id=u.id and o.id=#{id}
     */
    @Test
    public void testSelectOrderAndUserByOrderId(){
        String statement = "one.to.one.mapper.OrdersMapper.selectOrderAndUserByOrderID";
        //创建OrdersMapper对象，mybatis自动生成mapepr代理对象
        OrdersMapper orderMapper = session.getMapper(OrdersMapper.class);
        Orders order = orderMapper.selectOrderAndUserByOrderID(1);
        System.out.println(order);
        session.close();
    }
     
    /**
     * 方式二：嵌套查询
     * select * from order WHERE id=1;//得到user_id
     * select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
     */
    @Test
    public void testgetOrderByOrderId(){
        String statement = "one.to.one.mapper.OrdersMapper.getOrderByOrderId";
        //创建OrdersMapper对象，mybatis自动生成mapepr代理对象
        OrdersMapper orderMapper = session.getMapper(OrdersMapper.class);
        Orders order = orderMapper.selectOrderAndUserByOrderID(1);
        System.out.println(order);
        session.close();
    }
}
```

　　

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7309308.html#_labelTop)

### 2、一对多

　　还是以用户表 user 和 订单表 orders 为例，一个用户能创建多个订单。故用户和订单构成一对多的关联。

　　我们在 user.java 中添加一个属性 public List<Orders> orders;

**①、创建实体类**

　　user.java如下，orders.java保持不变

```
package com.ys.po;
 
import java.util.List;
 
public class User {
    //用户ID
    private int id;
    //用户姓名
    private String username;
    //用户性别
    private String sex;
    //一个用户能创建多个订单，用户和订单构成一对多的关系
    public List<Orders> orders;
     
    public List<Orders> getOrders() {
        return orders;
    }
    public void setOrders(List<Orders> orders) {
        this.orders = orders;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + "]";
    }
}
```

　　

**②、创建 UserMapper 接口和 \**UserMapper\**.xml 文件**

　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170808234802433-668316848.png)**

 

　　UserMapper接口

```
package one.to.many.mapper;
 
import com.ys.po.User;
 
public interface UserMapper {
    //根据用户id查询用户信息，以及用户下面的所有订单信息
    public User selectUserAndOrdersByUserId(int UserId);
 
}
```

　　UserMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="one.to.many.mapper.UserMapper">
    <!--
    方式一：嵌套结果：使用嵌套结果映射来处理重复的联合结果的子集
                               封装联表查询的数据(去除重复的数据)
     select * from user u,orders o where u.id=o.user_id and u.id=#{id}
     -->
    <select id="selectUserAndOrdersByUserId" resultMap="getUserAndOrders">
        select u.*,o.id oid,o.number number from user u,orders o where u.id=o.user_id and u.id=#{id}
    </select>
    <resultMap type="com.ys.po.User" id="getUserAndOrders">
        <!--id:指定查询列表唯一标识，如果有多个唯一标识，则配置多个id
            column:数据库对应的列
            property:实体类对应的属性名 -->
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
        <!--
            property:实体类中定义的属性名
            ofType：指定映射到集合中的全类名
          -->
        <collection property="orders" ofType="com.ys.po.Orders">
            <id column="oid" property="id"/>
            <result column="number" property="number"/>
        </collection>
    </resultMap>
</mapper>
```

　　

 ③、**向 mybatis-configuration.xml 配置文件中注册 UserMapper.xml 文件**

　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170808234449074-1413067667.png)**

 

④**、测试**

```
@Test
public void testSelectOrderAndUserByOrderId(){
    String statement = "one.to.many.mapper.UserMapper.selectUserAndOrdersByUserId";
    //创建OrdersMapper对象，mybatis自动生成mapepr代理对象
    UserMapper userMapper = session.getMapper(UserMapper.class);
    User user = userMapper.selectUserAndOrdersByUserId(1);
    System.out.println(user.getOrders().size());
    session.close();
}
```

　　

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7309308.html#_labelTop)

### 3、多对多

 　这里我们以用户 user 表和 角色role 表为例，假定一个用户能被分配成多重角色，而一种角色也能分给多个用户，故用户和角色构成多对多的关系。

　　需求：给定角色id，查询这个角色所属的所有用户信息

**①、在数据库中建立相应的表**

　　user 表和上面的保持不变

　　role 表

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809002047574-1619820845.png)

　　两者之间的关联表user_role 

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809002115792-909726617.png)

 

**②、建立对应的实体类**

　　**User.java**

　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809010259042-1014118612.png)**

　　Role.java

```
package com.ys.po;
 
import java.util.List;
 
public class Role {
    private int id;
    private String name;
     
    private List<User> users;
 
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public List<User> getUsers() {
        return users;
    }
 
    public void setUsers(List<User> users) {
        this.users = users;
    }
 
}
```

　　User_Role.java

```
package com.ys.po;
 
public class User_Role {
    private User user;
    private Role role;
    public User getUser() {
        return user;
    }
    public void setUser(User user) {
        this.user = user;
    }
    public Role getRole() {
        return role;
    }
    public void setRole(Role role) {
        this.role = role;
    }
}
```

　　

 ③**、创建 UserMapper 接口和 \**UserMapper\**.xml 文件**

 　UserMapper 接口

```
package many.to.many.mapper;
 
import java.util.List;
 
import com.ys.po.User;
 
public interface UserMapper {
     
    //给定一个角色id，要得到具有这个角色的所有用户信息
    public List<User> getUserByRoleId(int roleId);
 
}
```

　　UserMapper.xml 

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="many.to.many.mapper.UserMapper">
     
    <select id="getUserByRoleId" resultMap="getUserMap">
        select * from user_role ur,user u where ur.user_id=u.id and ur.role_id=#{id}
    </select>
     
    <resultMap type="com.ys.po.User" id="getUserMap">
        <id column="id" property="id"/>
        <result column="username" property="username"/>
        <result column="sex" property="sex"/>
    </resultMap>
</mapper>
```

　　

 ④、**向 mybatis-configuration.xml 配置文件中注册 UserMapper.xml 文件**

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809010622449-343032382.png)

**⑤、测试**

```
@Test
public void testGetUserByRoleId(){
    String statement = "many.to.many.mapper.UserMapper.getUserByRoleId";
    //创建OrdersMapper对象，mybatis自动生成mapepr代理对象
    UserMapper userMapper = session.getMapper(UserMapper.class);
    List<User> users = userMapper.getUserByRoleId(1);
    session.close();
}
```

　　多对多主要是关联关系要找好，然后根据关联去查询。

# [懒加载](https://www.cnblogs.com/ysocean/p/7336945.html)

目录**

- [1、需求：查询订单信息，有时候需要关联查出用户信息。](https://www.cnblogs.com/ysocean/p/7336945.html#_label0)
- [2、什么是懒加载？](https://www.cnblogs.com/ysocean/p/7336945.html#_label1)
- [3、具体实例](https://www.cnblogs.com/ysocean/p/7336945.html#_label2)
- [4、总结](https://www.cnblogs.com/ysocean/p/7336945.html#_label3)

 

------

　　本章我们讲如何通过懒加载来提高mybatis的查询效率。

　　本章所有代码:https://pan.baidu.com/s/1i6eDOwP 密码: qnbu

[回到顶部](https://www.cnblogs.com/ysocean/p/7336945.html#_labelTop)

### 1、需求：查询订单信息，有时候需要关联查出用户信息。

　　第一种方法：我们直接关联查询出所有订单和用户的信息

```
select * from orders o ,user u where o.user_id = u.id;
```

　　分析：

　　①、这里我们一次查询出所有的信息，需要什么信息的时候直接从查询的结果中筛选。但是如果订单和用户表都比较大的时候，这种关联查询肯定比较耗时。

　　②、我们的需求是有时候需要关联查询用户信息，这里不是一定需要用户信息的。即有时候不需要查询用户信息，我们也查了，程序进行了多余的耗时操作。

 

　　第二种方法：分步查询，首先查询出所有的订单信息，然后如果需要用户的信息，我们在根据查询的订单信息去关联用户信息

```
//查询所有的订单信息，包括用户id``select * from orders;``//如果需要用户信息，我们在根据上一步查询的用户id查询用户信息``select * from user where id=user_id
```

　　分析：

　　①、这里两步都是单表查询，执行效率比关联查询要高很多

　　②、分为两步，如果我们不需要关联用户信息，那么我们就不必执行第二步，程序没有进行多余的操作。

 

　　那么我们说，这第二种方法就是mybatis的懒加载。

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7336945.html#_labelTop)

### 2、什么是懒加载？

　　通俗的讲就是按需加载，我们需要什么的时候再去进行什么操作。而且先从单表查询，需要时再从关联表去关联查询，能大大提高数据库性能，因为查询单表要比关联查询多张表速度要快。

　　在mybatis中，resultMap可以实现高级映射(使用association、collection实现一对一及一对多映射)，association、collection具备延迟加载功能。

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7336945.html#_labelTop)

### 3、具体实例

　　**①、创建实体类**

　　　　User.java

```
package com.ys.lazyload;
 
import java.util.List;
 
public class User {
    //用户ID
    private int id;
    //用户姓名
    private String username;
    //用户性别
    private String sex;
    //一个用户能创建多个订单，用户和订单构成一对多的关系
    public List<Orders> orders;
     
    public List<Orders> getOrders() {
        return orders;
    }
    public void setOrders(List<Orders> orders) {
        this.orders = orders;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + "]";
    }
}
```

　　　　Orders.java

```
package com.ys.lazyload;
 
public class Orders {
    //订单ID
    private int id;
    //用户ID
    private int userId;
    //订单数量
    private String number;
    //和用户表构成一对一的关系，即一个订单只能由一个用户创建
    private User user;
     
    public int getId() {
        return id;
    }
 
    public void setId(int id) {
        this.id = id;
    }
 
    public int getUserId() {
        return userId;
    }
 
    public void setUserId(int userId) {
        this.userId = userId;
    }
 
    public String getNumber() {
        return number;
    }
 
    public void setNumber(String number) {
        this.number = number;
    }
 
    public User getUser() {
        return user;
    }
 
    public void setUser(User user) {
        this.user = user;
    }
 
    @Override
    public String toString() {
        return "Orders [id=" + id + ", userId=" + userId + ", number=" + number
                + ", user=" + user + "]";
    }
 
}
```

　　**②、创建 OrderMapper 接口和 OrderMapper.xml 文件**

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809230815136-193517654.png)

由于我们采用 Mapper 代理加载 xxxMapper.xml 文件，这里我们重复一下 Mapper 代理所需的条件，接口和xml文件必须满足以下几个条件：

　　1、接口必须要和 xml 文件同名且在同一个包下，也就是说 xml 文件中的namespace是接口的全类名　　

　　2、接口中的方法名和xml 文件中定义的 id 一致

　　3、接口输入参数类型要和xml 中定义的 parameterType 一致

　　4、接口返回数据类型要和xml 中定义的 resultType 一致

　　详细介绍参考上一篇博客：http://www.cnblogs.com/ysocean/p/7301548.html

 　OrderMapper 接口

```
package com.ys.lazyload;
 
import java.util.List;
 
import com.ys.lazyload.Orders;
import com.ys.lazyload.User;
 
public interface OrdersMapper {
    /**
     * select * from order //得到user_id
     * select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
     * @param userID
     * @return
     */
    //得到订单信息(包含user_id)
    public List<Orders> getOrderByOrderId();
    //根据用户ID查询用户信息
    public User getUserByUserId(int userID);
 
}
```

　　OrderMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.lazyload.OrdersMapper">
    <!--
         延迟加载：
         select user_id from order WHERE id=1;//得到user_id
         select * from user WHERE id=1   //1 是上一个查询得到的user_id的值
         property:别名(属性名)    column：列名 -->
    <select id="getOrderByOrderId" resultMap="getOrderMap">
        select * from orders
    </select>
    <resultMap type="com.ys.lazyload.Orders" id="getOrderMap">
        <id column="id" property="id"/>
        <result column="number" property="number"/>
        <!-- select:指定延迟加载需要执行的statement的id（根据user_id查询的statement）
                    如果不在本文件中，需要加上namespace
             property:resultMap中type指定类中的属性名
             column:和select查询关联的字段user_id
         -->
        <association property="user" javaType="com.ys.lazyload.User"  column="user_id" select="getUserByUserId">
         
        </association>
    </resultMap>
    <select id="getUserByUserId" resultType="com.ys.lazyload.User">
        select * from user where id=#{id}
    </select>
     
</mapper>
```

　　

　　③、**向 mybatis-configuration.xml 配置文件中注册 OrderMapper.xml 文件**

　　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809231708324-1791713117.png)**

　　④、**开启懒加载配置**

```
<!-- 开启懒加载配置 -->
<settings>
    <!-- 全局性设置懒加载。如果设为‘false'，则所有相关联的都会被初始化加载。 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 当设置为‘true'的时候，懒加载的对象可能被任何懒属性全部加载。否则，每个属性都按需加载。 -->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

　　**⑤、测试**

```
@Test
public void testLazy(){
    String statement = "com.ys.lazyload.OrdersMapper.getOrderByOrderId";
    //创建OrdersMapper对象，mybatis自动生成mapepr代理对象
    OrdersMapper orderMapper = session.getMapper(OrdersMapper.class);
    List<Orders> orders = orderMapper.getOrderByOrderId();//第一步
    for(Orders order : orders){
        System.out.println(order.getUser());//第二步
    }
    session.close();
}
```

　　我们在上面标识的第一步打一个断点，然后进行断点调试。

　　第一次进入断点：注意看控制台Console现在是没有任何sql语句发出的

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809232017949-1524703597.png)

 

 　下一步：这一步发出了第一次查询所有订单信息sql语句：select * from orders。注意只是查询订单信息，还没有进行关联查询

　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809232131839-1386055803.png)**

　　下一步：下图已经执行了一次for循环，因为我们需要用户信息，故发出了根据用户id查询用户信息的sql语句

　　　　　　注意：如果用户信息有多条，这里并不会发出多条sql语句，这是由于mybatis的一级缓存的原因，下一章会讲到。

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809232308964-1486649613.png)

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7336945.html#_labelTop)

### 4、总结

　　①、启动懒加载，mybatis初始化返回类型的时候，会返回一个cglib代理对象，该对象的关联对象（例如一对多，多对一）相关信息就会在loadpair里边，并且添加到loadmap中，cglib对象会过滤get，set ,is,"equals", "clone", "hashCode", "toString"触发方法，然后才会调用loadpair来加载关联对象的值。所以我们必须在进行懒加载的时候必须要导入相应的jar包，不然会报错。

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170809232917511-825564840.png)

 

 　②、其实通过上面的例子，我们很好理解懒加载的原理，就是按需加载。我们需要什么信息的时候再去查。而不是一次性查询所有的。将复杂的关联查询分解成单表查询，然后通过单表查询的结果去关联查询。

　　　　那么不用mybatis的懒加载我们也可是实现上面的例子：

　　　　一、定义两个mapper方法

　　　　　　1、查询订单列表

　　　　　　2、根据用户 id 查询用户信息

　　　　二、先去查询第一个mapper方法，获取订单信息列表，然后放入到一个集合中

　　　　三、如果需要用户信息，那么在程序中，我们可以遍历订单信息，得到用户id，然后通过id去查询用户信息。

　　　　这与mybatis懒加载的区别就是，mybatis是在mapper.xml文件中配置好关联关系了，我们直接调用就好了。而自己实现的原理就是手动去建立关联关系。

# [一级缓存、二级缓存](https://www.cnblogs.com/ysocean/p/7342498.html)

**目录**

- [1、一级缓存](https://www.cnblogs.com/ysocean/p/7342498.html#_label0)
- [2、二级缓存](https://www.cnblogs.com/ysocean/p/7342498.html#_label1)
- [3、二级缓存整合ehcache](https://www.cnblogs.com/ysocean/p/7342498.html#_label2)
- [4、二级缓存的应用场景](https://www.cnblogs.com/ysocean/p/7342498.html#_label3)

 

------

　　上一章节，我们讲解了通过mybatis的懒加载来提高查询效率，那么除了懒加载，还有什么方法能提高查询效率呢？这就是我们本章讲的缓存。

　　本篇源码下载链接：http://pan.baidu.com/s/1eRHTsIm 密码：a5wn

　　mybatis 为我们提供了一级缓存和二级缓存，可以通过下图来理解：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810222807042-122444009.png)

　　①、一级缓存是SqlSession级别的缓存。在操作数据库时需要构造sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

　　②、二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7342498.html#_labelTop)

### 1、一级缓存

　　**①、我们在一个 sqlSession 中，对 User 表根据id进行两次查询，查看他们发出sql语句的情况。**

```
@Test
public void testSelectOrderAndUserByOrderId(){
    //根据 sqlSessionFactory 产生 session
    SqlSession sqlSession = sessionFactory.openSession();
    String statement = "one.to.one.mapper.OrdersMapper.selectOrderAndUserByOrderID";
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    //第一次查询，发出sql语句，并将查询的结果放入缓存中
    User u1 = userMapper.selectUserByUserId(1);
    System.out.println(u1);
     
    //第二次查询，由于是同一个sqlSession,会在缓存中查找查询结果
    //如果有，则直接从缓存中取出来，不和数据库进行交互
    User u2 = userMapper.selectUserByUserId(1);
    System.out.println(u2);
     
    sqlSession.close();
}
```

　　查看控制台打印情况：

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810225609324-879680103.png)

 

　　**②、 同样是对user表进行两次查询，只不过两次查询之间进行了一次update操作。**

```
@Test``public` `void` `testSelectOrderAndUserByOrderId(){``  ``//根据 sqlSessionFactory 产生 session``  ``SqlSession sqlSession = sessionFactory.openSession();``  ``String stateme@Test
public void testSelectOrderAndUserByOrderId(){
    //根据 sqlSessionFactory 产生 session
    SqlSession sqlSession = sessionFactory.openSession();
    String statement = "one.to.one.mapper.OrdersMapper.selectOrderAndUserByOrderID";
    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
    //第一次查询，发出sql语句，并将查询的结果放入缓存中
    User u1 = userMapper.selectUserByUserId(1);
    System.out.println(u1);
     
    //第二步进行了一次更新操作，sqlSession.commit()
    u1.setSex("女");
    userMapper.updateUserByUserId(u1);
    sqlSession.commit();
     
    //第二次查询，由于是同一个sqlSession.commit(),会清空缓存信息
    //则此次查询也会发出 sql 语句
    User u2 = userMapper.selectUserByUserId(1);
    System.out.println(u2);
     
    sqlSession.close();
}
```

　　控制台打印情况：

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810230325792-1154192032.png)

　　

　　**③、总结**

　　1、第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。 

　　2、如果中间sqlSession去执行commit操作（执行插入、更新、删除），则会清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。

　　3、第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直接从缓存中获取用户信息。

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810230553636-190248238.png)

 

 

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7342498.html#_labelTop)

### 2、二级缓存

　　二级缓存的原理和一级缓存原理一样，第一次查询，会将数据放入缓存中，然后第二次查询则会直接去缓存中取。但是一级缓存是基于 sqlSession 的，而 二级缓存是基于 mapper文件的namespace的，也就是说多个sqlSession可以共享一个mapper中的二级缓存区域，并且如果两个mapper的namespace相同，即使是两个mapper，那么这两个mapper中执行sql查询到的数据也将存在相同的二级缓存区域中。

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810231159511-916493575.png)

　　那么二级缓存是如何使用的呢？

　　**①、开启二级缓存**

　　和一级缓存默认开启不一样，二级缓存需要我们手动开启

　　首先在全局配置文件 mybatis-configuration.xml 文件中加入如下代码：

```
<!--开启二级缓存  -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

　　其次在 UserMapper.xml 文件中开启缓存

```
<!-- 开启二级缓存 -->
<cache></cache>
```

　　我们可以看到 mapper.xml 文件中就这么一个空标签<cache/>，其实这里可以配置<cache type="org.apache.ibatis.cache.impl.PerpetualCache"/>,PerpetualCache这个类是mybatis默认实现缓存功能的类。我们不写type就使用mybatis默认的缓存，也可以去实现 Cache 接口来自定义缓存。

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810233033699-844313564.png)

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810233143089-99539433.png)

　　我们可以看到 二级缓存 底层还是 HashMap 架构。

 

 

 　**②、po 类实现** **Serializable** **序列化接口**

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810233400839-1089354206.png)

 

 　开启了二级缓存后，还需要将要缓存的pojo实现Serializable接口，为了将缓存数据取出执行反序列化操作，因为二级缓存数据存储介质多种多样，不一定只存在内存中，有可能存在硬盘中，如果我们要再取这个缓存的话，就需要反序列化了。所以mybatis中的pojo都去实现Serializable接口。

 　

 　**③、测试**

 　一、测试二级缓存和sqlSession 无关

```
@Test
public void testTwoCache(){
    //根据 sqlSessionFactory 产生 session
    SqlSession sqlSession1 = sessionFactory.openSession();
    SqlSession sqlSession2 = sessionFactory.openSession();
     
    String statement = "com.ys.twocache.UserMapper.selectUserByUserId";
    UserMapper userMapper1 = sqlSession1.getMapper(UserMapper.class);
    UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
    //第一次查询，发出sql语句，并将查询的结果放入缓存中
    User u1 = userMapper1.selectUserByUserId(1);
    System.out.println(u1);
    sqlSession1.close();//第一次查询完后关闭sqlSession
     
    //第二次查询，即使sqlSession1已经关闭了，这次查询依然不发出sql语句
    User u2 = userMapper2.selectUserByUserId(1);
    System.out.println(u2);
    sqlSession2.close();
}
```

　　可以看出上面两个不同的sqlSession,第一个关闭了，第二次查询依然不发出sql查询语句。

 

　　二、测试执行 commit() 操作，二级缓存数据清空

```
@Test
public void testTwoCache(){
    //根据 sqlSessionFactory 产生 session
    SqlSession sqlSession1 = sessionFactory.openSession();
    SqlSession sqlSession2 = sessionFactory.openSession();
    SqlSession sqlSession3 = sessionFactory.openSession();
     
    String statement = "com.ys.twocache.UserMapper.selectUserByUserId";
    UserMapper userMapper1 = sqlSession1.getMapper(UserMapper.class);
    UserMapper userMapper2 = sqlSession2.getMapper(UserMapper.class);
    UserMapper userMapper3 = sqlSession2.getMapper(UserMapper.class);
    //第一次查询，发出sql语句，并将查询的结果放入缓存中
    User u1 = userMapper1.selectUserByUserId(1);
    System.out.println(u1);
    sqlSession1.close();//第一次查询完后关闭sqlSession
     
    //执行更新操作，commit()
    u1.setUsername("aaa");
    userMapper3.updateUserByUserId(u1);
    sqlSession3.commit();
     
    //第二次查询，由于上次更新操作，缓存数据已经清空（防止数据脏读），这里必须再次发出sql语句
    User u2 = userMapper2.selectUserByUserId(1);
    System.out.println(u2);
    sqlSession2.close();
}
```

　　查看控制台情况：

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810235434292-1038719579.png)

 

 　**④、useCache和flushCache**

　　mybatis中还可以配置userCache和flushCache等配置项，userCache是用来设置是否禁用二级缓存的，在statement中设置useCache=false可以禁用当前select语句的二级缓存，即每次查询都会发出sql去查询，默认情况是true，即该sql使用二级缓存。

```
``"selectUserByUserId"` `useCache=``"false"` `resultType=``"com.ys.twocache.User"` `parameterType=``"int"``>``  ``select * from user where id=#{id}```
```

　　这种情况是针对每次查询都需要最新的数据sql，要设置成useCache=false，禁用二级缓存，直接从数据库中获取。 

　　在mapper的同一个namespace中，如果有其它insert、update、delete操作数据后需要刷新缓存，如果不执行刷新缓存会出现脏读。

   设置statement配置中的flushCache=”true” 属性，默认情况下为true，即刷新缓存，如果改成false则不会刷新。使用缓存时如果手动修改数据库表中的查询数据会出现脏读。

```
"selectUserByUserId"` `flushCache=``"true"` `useCache=``"false"` `resultType=``"com.ys.twocache.User"` `parameterType=``"int"``>``  ``select * from user where id=#{id}```


```

　　一般下执行完commit操作都需要刷新缓存，flushCache=true表示刷新缓存，这样可以避免数据库脏读。所以我们不用设置，默认即可。

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7342498.html#_labelTop)

### 3、二级缓存整合ehcache

　　上面我们介绍了mybatis自带的二级缓存，但是这个缓存是单服务器工作，无法实现分布式缓存。那么什么是分布式缓存呢？假设现在有两个服务器1和2，用户访问的时候访问了1服务器，查询后的缓存就会放在1服务器上，假设现在有个用户访问的是2服务器，那么他在2服务器上就无法获取刚刚那个缓存，如下图所示：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170814203706678-1713872147.png)

　　为了解决这个问题，就得找一个分布式的缓存，专门用来存储缓存数据的，这样不同的服务器要缓存数据都往它那里存，取缓存数据也从它那里取，如下图所示：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170814203739053-432896069.png)

 

 　如上图所示，在几个不同的服务器之间，我们使用第三方缓存框架，将缓存都放在这个第三方框架中，然后无论有多少台服务器，我们都能从缓存中获取数据。

　　这里我们介绍mybatis与第三方框架ehcache的整合。

　　上文一开始提到过，mybatis提供了一个cache接口，如果要实现自己的缓存逻辑，实现cache接口开发即可。mybatis本身默认实现了一个，但是这个缓存的实现无法实现分布式缓存，所以我们要自己来实现。ehcache分布式缓存就可以，mybatis提供了一个针对cache接口的ehcache实现类，这个类在mybatis和ehcache的整合包中。

　　**①、导入 mybatis-ehcache 整合包（最上面的源代码中包含有）**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170814205927896-1579894274.png)

 

 　**②、在全局配置文件 mybatis-configuration.xml 开启缓存**

```
<!--开启二级缓存  -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

　　

 　**③、在 xxxMapper.xml 文件中整合 ehcache 缓存**

　　将如下的类的全类名写入<cache type="" ></cache>的type属性中

```
<!-- 开启本mapper的namespace下的二级缓存
    type:指定cache接口的实现类的类型，不写type属性，mybatis默认使用PerpetualCache
    要和ehcache整合，需要配置type为ehcache实现cache接口的类型
-->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"></cache>
```

　　

　　**④、配置缓存参数**

　　在 classpath 目录下新建一个 ehcache.xml 文件，并增加如下配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
 
    <diskStore path="F:\develop\ehcache"/>
 
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU">
        <persistence strategy="localTempSwap"/>
    </defaultCache>
</ehcache>
```

　　

 diskStore：指定数据在磁盘中的存储位置。
 defaultCache：当借助CacheManager.add("demoCache")创建Cache时，EhCache便会采用<defalutCache/>指定的的管理策略
以下属性是必须的：
 maxElementsInMemory - 在内存中缓存的element的最大数目
 maxElementsOnDisk - 在磁盘上缓存的element的最大数目，若是0表示无穷大
 eternal - 设定缓存的elements是否永远不过期。如果为true，则缓存的数据始终有效，如果为false那么还要根据timeToIdleSeconds，timeToLiveSeconds判断
 overflowToDisk - 设定当内存缓存溢出的时候是否将过期的element缓存到磁盘上
以下属性是可选的：
 timeToIdleSeconds - 当缓存在EhCache中的数据前后两次访问的时间超过timeToIdleSeconds的属性取值时，这些数据便会删除，默认值是0,也就是可闲置时间无穷大
 timeToLiveSeconds - 缓存element的有效生命期，默认是0.,也就是element存活时间无穷大
 diskSpoolBufferSizeMB 这个参数设置DiskStore(磁盘缓存)的缓存区大小.默认是30MB.每个Cache都应该有自己的一个缓冲区.
 diskPersistent - 在VM重启的时候是否启用磁盘保存EhCache中的数据，默认是false。
 diskExpiryThreadIntervalSeconds - 磁盘缓存的清理线程运行间隔，默认是120秒。每个120s，相应的线程会进行一次EhCache中数据的清理工作
 memoryStoreEvictionPolicy - 当内存缓存达到最大，有新的element加入的时候， 移除缓存中element的策略。默认是LRU（最近最少使用），可选的有LFU（最不常使用）和FIFO（先进先出）

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7342498.html#_labelTop)

### 4、二级缓存的应用场景

　　对于访问多的查询请求且用户对查询结果实时性要求不高，此时可采用mybatis二级缓存技术降低数据库访问量，提高访问速度，业务场景比如：耗时较高的统计分析sql、电话账单查询sql等。实现方法如下：通过设置刷新间隔时间，由mybatis每隔一段时间自动清空缓存，根据数据变化频率设置缓存刷新间隔flushInterval，比如设置为30分钟、60分钟、24小时等，根据需求而定。 

　　mybatis二级缓存对细粒度的数据级别的缓存实现不好，比如如下需求：对商品信息进行缓存，由于商品信息查询访问量大，但是要求用户每次都能查询最新的商品信息，此时如果使用mybatis的二级缓存就无法实现当一个商品变化时只刷新该商品的缓存信息而不刷新其它商品的信息，因为mybaits的二级缓存区域以mapper为单位划分的，当一个商品信息变化会将所有商品信息的缓存数据全部清空。解决此类问题可能需要在业务层根据需求对数据有针对性缓存。

# [逆向工程](https://www.cnblogs.com/ysocean/p/7360409.html)

　通过前面的学习，在实际开发中，我们基本上能对mybatis应用自如了，但是我们发现了一个问题，所有操作都是围绕着po类，xxxMapper.xml文件，xxxMapper接口等文件来进行的。如果实际开发中数据库的表特别多，那么我们需要手动去写每一张表的po类，xxxMapper.xml，xxxMapper.java文件，这显然需要花费巨大的精力，而且可能由于表字段太多，写错了而不知道也是可能的。

　　所以我们在实际开发中，一般使用逆向工程方式来自动生成所需的文件。

　　本篇博客源码下载链接：链接: https://pan.baidu.com/s/1qIGNN4RmGIrZ

D-tpvk32_A 提取码: vwcc 

**①、新建一个工程，并导入相应的jar包（详情见上面源码）**

　　**![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170814213718428-791385599.png)**

 

 

　　注意：使用逆向工程时，最好新建一个工程，如果你在原来的工程中使用，那也可以，但是有一定的风险，因为mybatis是根据配置文件中配置的路径来生成的文件的，如果你工程中有相同名字的文件，那么就会被新生成的文件所覆盖。所以实际开发中，我们一般新建一个工程，将生成的文件复制到自己的所需的工程中。

 

**②、创建配置文件 generatorConfig.xml 文件**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
 
<generatorConfiguration>
    <context id="testTables" targetRuntime="MyBatis3">
        <commentGenerator>
            <!-- 是否去除自动生成的注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true" />
        </commentGenerator>
        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
            connectionURL="jdbc:mysql://localhost:3306/mybatisrelation" userId="root"
            password="root">
        </jdbcConnection>
        <!-- <jdbcConnection driverClass="oracle.jdbc.OracleDriver"
            connectionURL="jdbc:oracle:thin:@127.0.0.1:1521:yycg"
            userId="yycg"
            password="yycg">
        </jdbcConnection> -->
 
        <!-- 默认false，把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer，为 true时把JDBC DECIMAL和NUMERIC类型解析为java.math.BigDecimal -->
        <javaTypeResolver>
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
 
        <!-- targetProject:生成PO类的位置，重要！！ -->
        <javaModelGenerator targetPackage="com.ys.po"
            targetProject=".\src">
            <!-- enableSubPackages:是否让schema作为包的后缀 -->
            <property name="enableSubPackages" value="false" />
            <!-- 从数据库返回的值被清理前后的空格 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- targetProject:mapper映射文件生成的位置，重要！！ -->
        <sqlMapGenerator targetPackage="com.ys.mapper"
            targetProject=".\src">
            <property name="enableSubPackages" value="false" />
        </sqlMapGenerator>
        <!-- targetPackage：mapper接口生成的位置，重要！！ -->
        <javaClientGenerator type="XMLMAPPER"
            targetPackage="com.ys.mapper"
            targetProject=".\src">
            <property name="enableSubPackages" value="false" />
        </javaClientGenerator>
        <!-- 指定数据库表，要生成哪些表，就写哪些表，要和数据库中对应，不能写错！ -->
        <table tableName="items"></table>
        <table tableName="orders"></table>
        <table tableName="orderdetail"></table>
        <table tableName="user"></table>       
    </context>
</generatorConfiguration>
```

　　注意：

　　1、连接数据库的配置，包括数据名称，数据库用户名密码等配置

　　2、指定要生成代码的包名，包括实体类po的包名，mapper的包名等

　　3、指定数据库中哪些表需要生成文件

 

**③、运行主程序生成代码**

```
package com.ys.test;
 
import java.io.File;
import java.util.ArrayList;
import java.util.List;
 
import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;
 
public class GeneratorTest {
 
    public void testGenerator() throws Exception{
 
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        //指向逆向工程配置文件
        File configFile = new File(GeneratorTest.class.getResource("/generatorConfig.xml").getFile());
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
                callback, warnings);
        myBatisGenerator.generate(null);
 
    }
    public static void main(String[] args) throws Exception {
        try {
            GeneratorTest generator = new GeneratorTest();
            generator.testGenerator();
        } catch (Exception e) {
            e.printStackTrace();
        }
 
    }
}
```

　　直接运行上面的程序，控制台会打印如下代码，说明生成代码成功

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170814214512115-1911100441.png)

 

　　然后刷新generatorConfig.xml 文件中指定的包，会发现生成了如下文件

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170814214621115-1277166596.png)



# [mybatis和spring整合](https://www.cnblogs.com/ysocean/p/7368499.html)

**目录**

- [1、创建mybatis-spring 工程，并导入相应的 jar 包（详情见上面源码）](https://www.cnblogs.com/ysocean/p/7368499.html#_label0)
- [2、在 spring 全局配置文件中 applicationContext.xml 中配置 SqlSessionFactory，以及数据源](https://www.cnblogs.com/ysocean/p/7368499.html#_label1)
- [3、mapper 接口开发配置](https://www.cnblogs.com/ysocean/p/7368499.html#_label2)
- [ 4、在 spring全局配置文件applicationContext.xml 中配置 mapper](https://www.cnblogs.com/ysocean/p/7368499.html#_label3)

 

------

　　想要整合mybatis和spring,那么我们首先要知道这两个框架是干嘛的，对于mybatis我们前面几篇博客已经有了很详细的介绍，我们通过加载mybatis-configuration.xml 文件来产生SqlSessionFactory，然后通过SqlSessionFactory去产生sqlSession,最后用 sqlSession对数据库表所映射的实体类进行增删改查操作。而spring是干嘛的呢，简单来说，通过spring的DI和IOC，能帮助我们产生对象并管理对象的声明周期，而sprig的AOP也能帮助我们管理对象的事务。那么整合思路就很清晰了。

　　1、需要spring通过单例的方式管理 SqlSessionFactory，并用 SqlSessionFactory 去创建 sqlSession

　　2、持久层的 mapper 需要spring 管理

 　本篇所有源码链接：https://pan.baidu.com/s/1dG00Ksp 密码：d1ev

[回到顶部](https://www.cnblogs.com/ysocean/p/7368499.html#_labelTop)

### 1、创建mybatis-spring 工程，并导入相应的 jar 包（详情见上面源码）

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170816001726115-2093323132.png)

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7368499.html#_labelTop)

### **2、在 spring 全局配置文件中 applicationContext.xml 中配置 SqlSessionFactory，以及数据源**

　　**①、我们将数据库配置信息写入classpath 目录的 db.properties 文件中**

```
#db.properties
dataSource=org.apache.commons.dbcp.BasicDataSource
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatisrelation
jdbc.username=root
jdbc.password=root
```

　　

　　**②、在mybatis全局配置文件mybatis-configuration.xml 开启二级缓存，以及别名定义**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
<!--开启二级缓存  -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
 
<!-- 包的别名定义 -->
<typeAliases>
    <package name="com.ys.po"/>
</typeAliases>
 
<!-- 注意：下面的以前有mybatis全局配置文件管理mapper，现在转移到spring容器管理 -->
<!-- <mappers>
    <mapper class="com.ys.po.UserMapper"/>
 </mappers> -->
   
</configuration>
```

　　

　　**③、在 spring 全局配置文件中 applicationContext.xml 中配置 SqlSessionFactory，以及数据源**

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.2.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
 
    <!-- 加载classpath下的db.properties文件，里面配了数据库连接的一些信息 -->
    <context:property-placeholder location="classpath:db.properties"/>
 
    <!-- 配置数据源 -->
    <bean id="dataSource" class="${dataSource}" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driver}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="maxActive" value="10" />
        <property name="maxIdle" value="5" />
    </bean>
 
    <!-- 配置sqlSessionFactory，SqlSessionFactoryBean是用来产生sqlSessionFactory的 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!-- 加载mybatis的全局配置文件，放在classpath下的mybatis文件夹中 -->
        <property name="configLocation" value="mybatis/mybatis-configuration.xml" />
        <!-- 加载数据源，使用上面配置好的数据源 -->
        <property name="dataSource" ref="dataSource" />
    </bean>
 
</beans>
```

　　

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7368499.html#_labelTop)

### 3、mapper 接口开发配置

　　**①、创建 po 类**

　　　　在com.ys.po包下创建 User.java

```
package com.ys.po;
 
public class User {
    private int id;
    private String username;
    private String sex;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getSex() {
        return sex;
    }
    public void setSex(String sex) {
        this.sex = sex;
    }
    @Override
    public String toString() {
        return "User [id=" + id + ", username=" + username + ", sex=" + sex
                + "]";
    }
}
```

　　**②、创建接口 UserMapper.java，以及UserMapper.xml 文件**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170815234702850-992154534.png)

　　必须满足以下四点：

```
1、UserMapper 接口必须要和 UserMapper.xml 文件同名且在同一个包下，也就是说 UserMapper.xml 文件中的namespace是UserMapper接口的全类名
2、UserMapper接口中的方法名和 UserMapper.xml 文件中定义的 id 一致
3、UserMapper接口输入参数类型要和 UserMapper.xml 中定义的 parameterType 一致
4、UserMapper接口返回数据类型要和 UserMapper.xml 中定义的 resultType 一致
```

　　UserMapper.java

```
package` `com.ys.mapper;` `import` `com.ys.po.User;` `public` `interface` `UserMapper {``  ``//根据 id 查询 user 表数据``  ``public` `User selectUserById(``int` `id) ``throws` `Exception;``}
```

　　UserMapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.mapper.UserMapper">
     
    <!-- 根据 id 查询 user 表中的数据
       id:唯一标识符，此文件中的id值不能重复
       resultType:返回值类型，一条数据库记录也就对应实体类的一个对象
       parameterType:参数类型，也就是查询条件的类型
    -->
    <select id="selectUserById"
            resultType="com.ys.po.User" parameterType="int">
        select * from user where id = #{id}
    </select>
     
</mapper>
```

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7368499.html#_labelTop)

###  4、在 spring全局配置文件applicationContext.xml 中配置 mapper

```
<!-- MapperFactoryBean：根据mapper接口生成的代理对象 -->
<bean id="userMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="mapperInterface" value="com.ys.mapper.UserMapper"/>
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

　　测试：

```
package com.ys.test;
 
import org.junit.Before;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
import com.ys.mapper.UserMapper;
import com.ys.po.User;
 
public class UserMapperTest {
     
    private ApplicationContext applicationContext;
 
    @Before
    public void setUp() throws Exception {
        applicationContext = new ClassPathXmlApplicationContext("classpath:spring/applicationContext.xml");//得到spring容器
    }
 
    @Test
    public void testSelectUserById() throws Exception {
        UserMapper userMapper = (UserMapper) applicationContext.getBean("userMapper");
        User user = userMapper.selectUserById(1);
        System.out.println(user);
     
    }
 
}
```

　　从配置中可以看出，使用MapperFactoryBean来产生mapper的代理对象，首先要配置一个mapperInterface，即你要spring产生哪个mapper接口对应的代理对象，将mapper接口的全类名给传进去，spring就知道要创建对应的代理对象了，然后配置sqlSessionFactory，用来产生sqlSession。

　　但是我们发现，id="userMapper" 写死了，如果我们有很多 mapper 接口，那么我们每一个都需要配置吗？答案是不用的，我们可以不用配置 id 属性，然后加上包扫描配置，如下：

```
<!--第二种方式：mapper批量扫描
        规范：mapper.java和mapper.xml必须在同一个包下，并没名称一致
        bean的id为mapper类名（首字母小写）
      -->
    <bean  class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!-- 指定扫描的包名
            如果扫描多个包，那么每个包中间使用半角逗号分隔 -->
        <property name="basePackage" value="com.ys.mapper"></property>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory" />
    </bean>
```

　　测试程序还是和上面一样，那么spring和mybatis整合就完美了。





# 问题

### 什么是MyBatis

MyBatis 是一个可以自定义 SQL、存储过程和高级映射的半自动ORM持久层框架。

### Mybatis的好处

1）MyBatis 把 sql 语句从 Java 源程序中独立出来，放在单独的 XML 文件中编写，给程序的

维护带来了很大便利。

2）MyBatis 封装了底层 JDBC API 的调用细节，并能自动将结果集转换成 Java Bean 对象，

大大简化了 Java 数据库编程的重复工作。

3）因为 MyBatis 需要程序员自己去编写 sql 语句，程序员可以结合数据库自身的特点灵活

控制 sql 语句，因此能够实现比 Hibernate 等全自动 orm 框架更高的查询效率，能够完成复

杂查询。

### Hibernate和Mybatis

​	Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象

时，可以根据对象关系模型直接获取，所以它是全自动的。而 Mybatis 在查询关联对象或

关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。

**Mybatis**

​	1） MyBatis 把 sql 语句从 Java 源程序中独立出来，放在单独的 XML 文件中编写，给程序的

维护带来了很大便利。

2） MyBatis 封装了底层 JDBC API 的调用细节，并能自动将结果集转换成 Java Bean 对象，

大大简化了 Java 数据库编程的重复工作。

3）因为 MyBatis 需要程序员自己去编写 sql 语句，程序员可以结合数据库自身的特点灵活

控制 sql 语句，因此能够实现比 Hibernate 等全自动 orm 框架更高的查询效率，能够完成复

杂查询。

区别

1） Mybatis 和 hibernate 不同，它不完全是一个 ORM 框架，因为 MyBatis 需要程序员自己

编写 Sql 语句，不过 mybatis 可以通过 XML 或注解方式灵活配置要运行的 sql 语句，并将

java 对象和 sql 语句映射生成最终执行的 sql，最后将 sql 执行的结果再映射生成 java 对

象。

2） Mybatis 学习门槛低，简单易学，程序员直接编写原生态 sql，可严格控制 sql 执行性

能，灵活度高，非常适合对关系数据模型要求不高的软件开发，例如互联网软件、企业运

营类软件等，因为这类软件需求变化频繁，一但需求变化要求成果输出迅速。但是灵活的

前提是 mybatis 无法做到数据库无关性，如果需要实现支持多种数据库的软件则需要自定

义多套 sql 映射文件，工作量大。

3） Hibernate 对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件（例如

需求固定的定制化软件）如果用 hibernate 开发可以节省很多代码，提高效率。但是

Hibernate 的缺点是学习门槛高，要精通门槛更高，而且怎么设计 O/R 映射，在性能和对象

模型之间如何权衡，以及怎样用好 Hibernate 需要具有很强的经验和能力才行。

总之，按照用户的需求在有限的资源环境下只要能做出维护性、扩展性良好的软件架构都

是好架构，所以框架只有适合才是最好。

### 在Mybatis中，有#{}和${}两种占位符

在Mybatis中，有两种占位符

- `#{}`解析传递进来的参数数据

- ${}对传递进来的参数**原样**拼接在SQL中

- **`#{}`是预编译处理，${}是字符串替换**。

- 使用#{}可以有效的防止SQL注入，提高系统安全性。

- Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement 的 set 方法

  来赋值；

### Mybatis如何将sql执行结果封装为目标对象

第一种是使用<resultMap>标签，逐一定义列名和对象属性名之间的映射关系。

第二种是使用 sql 列的别名功能，将列别名书写为对象属性名，比如 T_NAME AS NAME，对象属性名一般是 name，小写，但是列名不区分大小写，Mybatis 会忽略列名大小写，智能找到与之对应对象属性名，你甚至可以写成 T_NAME AS NaMe，Mybatis 一样可以正常工作。

有了列名与属性名的映射关系后，Mybatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

### resultMap和resultType区别

resultType ：指定输出结果的类型（pojo、简单类型、hashmap..），将sql查询结果映射为java对象 。

- 使用resultType注意：**sql查询的列名要和resultType指定pojo的属性名相同，指定相同 属性方可映射成功**，如果sql查询的列名要和resultType指定pojo的属性名全部不相同，list中无法创建pojo对象的。

resultMap：将sql查询结果映射为java对象。

- 如果sql查询列名和最终要映射的pojo的属性名不一致**，使用resultMap将列名和pojo的属性名做一个对应关系 （列名和属性名映射配置）**

### resultType和resultMap用法总结

resultType：

- 作用：将查询结果按照**sql列名pojo属性名一致**性映射到pojo中。
- 场合：常见一些明细记录的展示，将关联查询信息全部展示在页面时，此时可直接使用resultType将每一条记录映射到pojo中，在前端页面遍历list（list中是pojo）即可。

resultMap：

- 使用**association和collection完成一对一和一对多高级映射。**

------

association：

- 作用：**将关联查询信息映射到一个pojo类中。**
- 场合：为了方便获取关联信息可以使用association将关联订单映射为pojo，比如：查询订单及关联用户信息。

collection：

- 作用：**将关联查询信息映射到一个list集合中。**
- 场合：为了方便获取关联信息可以使用collection将关联信息映射到list集合中，比如：查询用户权限范围模块和功能，可使用collection将模块和功能列表映射到list中。





### Mybatis映射文件处理特殊字符

第一种方法：

- 用了转义字符把>和<替换掉就OK了

第二种方法：

- ``

### Mybatis的XML映射文件和Mybatis内部数据映射

在Mybatis将所有的XML配置信息都封装到ALL-IN-One重量级Configuration内部，在XML映射文件中，

<parameterMap>标签会被解析为ParameterMapping对象，<resultMap>标签会被解析为ResultMap对象，

其每个子元素会被解析为ResultMapping对象。每一个select，insert，update，delete，标签会被解析为MappedStatement对象，

标签内的sql会被解析为BoundSql对象。



### mapper中传递多个参数

第⼀种：使⽤占位符的思想

1.在映射⽂件中使⽤**#{0},#{1}**代表传递进来的第⼏个参数

//对应的xml,#{0}代表接收的是dao层中的第⼀个参数，#{1}代表dao层中第⼆参数，更多参数⼀致往后

加即可。

```
<select id="selectUser"resultMap="BaseResultMap">
select * fromuser_user_t whereuser_name = #{0} anduser_area=#{1}
</select>
```

2.使⽤**@param**注解**:**来命名参数

```
public interface usermapper {

user selectuser(@param(“username”) string username,@param(“hashedpassword”) string hashedpassword);}

<select id="selectuser" resulttype="user">

select id, username, hashedpassword

from some_table

where username = #{username}

and hashedpassword = #{hashedpassword}

</select>
```

第⼆种：使⽤**Map**集合作为参数来装载

```
<!--分⻚查询-->

<select id="pagination" parameterType="map" resultMap="studentMap">

/*根据key⾃动找到对应Map集合的value*/

select * from students limit #{start},#{end};

</select>
```

### Mybatis实现一对一的方式

联合查询

​	 几个表联合查询，只查询一次，通过在resultMap里配置association节点配置一对一的类就可以完成

嵌套查询

​	先查一个表，根据这个表里面的结果的外建id，再去另外一个表里查询数据，也是通过association配置，但是另外一个表的查询通过select属性配置

   多对多和这个一样



### Mybatis的xml映射文件标签

```
<resultMap>、<parameterMap>、<sql>、<include>、<selectKey>，加上动态 sql 的 9 个标签，
trim|where|set|foreach|if|choose|when|otherwise|bind 等，
其中<sql>为 sql 片段标签，通过<include>标签引入 sql 片段，
<selectKey>为不支持自增的主键生成策略标签。
```



### 延迟加载

```
 <!-- 全局配置参数 -->
  <settings>
    <!-- true开启延迟加载	总开关 -->
    <setting name="lazyLoadingEnabled" value="true" />  
    <!-- false设置按需加载 -->
    <setting name="aggressiveLazyLoading" value="false" />
  </settings>
```



### 如何获取自动生成的(主)键值?

如果我们一般插入数据的话，**如果我们想要知道刚刚插入的数据的主键是多少，我们可以通过以下的方式来获取**

 

需求：user对象插入到数据库后，新记录的主键要通过user对象返回，通过user获取主键值。

解决思路：通过LAST_INSERT_ID()获取刚插入记录的自增主键值，**在insert语句执行后，执行select LAST_INSERT_ID()就可以获取自增主键。**

mysql:

```
  <insert id="insertUser" parameterType="cn.itcast.mybatis.po.User">
    <selectKey keyProperty="id" order="AFTER" resultType="int">
      select LAST_INSERT_ID()
    </selectKey>
    INSERT INTO USER(username,birthday,sex,address) VALUES(#{username},#{birthday},#{sex},#{address})
  </insert>
```

oracle:实现思路：**先查询序列得到主键，将主键设置到user对象中，将user对象插入数据库。**

```
  <!-- oracle
  在执行insert之前执行select 序列.nextval() from dual取出序列最大值，将值设置到user对象 的id属性
   -->
  <insert id="insertUser" parameterType="cn.itcast.mybatis.po.User">
    <selectKey keyProperty="id" order="BEFORE" resultType="int">
      select 序列.nextval() from dual
    </selectKey>
    
    INSERT INTO USER(id,username,birthday,sex,address) VALUES( 序列.nextval(),#{username},#{birthday},#{sex},#{address})
  </insert> 
```

也可以在`select`标签下写以下的属性：

```
< select useGeneratedKeys="true" keyProperty="id"  keyColumn="id" />
```

### Mybatis 映射文件中，如果 A 标签通过 include 引用了 B 标签的内容，请问，B 标签能否定义在 A 标签的后面，还是说必须定义在 A 标签的前面？

虽然 Mybatis 解析 Xml 映射文件是按照顺序解析的，但是，被引用的 B 标签依然可以

定义在任何地方，Mybatis 都可以正确识别。原理是，Mybatis 解析 A 标签，发现 A 标签引

用了 B 标签，但是 B 标签尚未解析到，尚不存在，此时，Mybatis 会将 A 标签标记为未解

析状态，然后继续解析余下的标签，包含 B 标签，待所有标签解析完毕，Mybatis 会重新

解析那些被标记为未解析的标签，此时再解析 A 标签时，B 标签已经存在，A 标签也就可

以正常解析完成了。



## Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？

> Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？

- Mybatis动态sql可以让我们在Xml映射文件内，**以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能**。
- Mybatis提供了9种动态sql标签：trim|where|set|foreach|if|choose|when|otherwise|bind。
- 其执行原理为，使用OGNL从sql参数对象中计算表达式的值，**根据表达式的值动态拼接sql，以此来完成动态sql的功能**。
- MyBatis 里面的动态 Sql 一般是通过 if 节点来实现,通过 OGNL 语法来实现,但是如果要写的完整,必须配合 where,trim 节点,where 节点是判断包含节点有内容就插入 where,否则不插入,trim 节点是用来判断如果动态语句是以 and 或 or 开始,那么会自动把这个 and 或者 or取掉。



## Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？

> Mybatis的Xml映射文件中，不同的Xml映射文件，id是否可以重复？

**如果配置了namespace那么当然是可以重复的，因为我们的Statement实际上就是namespace+id**

如果没有配置namespace的话，那么相同的id就会导致覆盖了。



## 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？

> 为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？

- Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。

- 而Mybatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，所以，称之为半自动ORM映射工具。

  



## 通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

> 通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

- Dao接口，就是人们常说的Mapper接口，接口的全限名，就是映射文件中的namespace的值，接口的方法名，就是映射文件中MappedStatement的id值，接口方法内的参数，就是传递给sql的参数。
- Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement





Dao接口里的方法，**是不能重载的，因为是全限名+方法名的保存和寻找策略**。

**Dao接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Dao接口生成代理proxy对象，代理对象proxy会拦截接口方法，转而执行MappedStatement所代表的sql，然后将sql执行结果返回。**

详情可参考：

- https://www.cnblogs.com/soundcode/p/6497291.html

## Mybatis比IBatis比较大的几个改进是什么

> Mybatis比IBatis比较大的几个改进是什么

- a.**有接口绑定,包括注解绑定sql和xml绑定Sql** ,
- b.动态sql由原来的节点配置变成OGNL表达式,
- c. 在一对一,一对多的时候引进了association,在一对多的时候引入了collection节点,不过都是在resultMap里面配置



## 接口绑定有几种实现方式,分别是怎么实现的?

> 接口绑定有几种实现方式,分别是怎么实现的?

接口绑定有两种实现方式：

- 一种是通过注解绑定,就是在接口的方法上面加上@Select@Update等注解里面包含Sql语句来绑定
- 另外一种就是通过xml里面写SQL来绑定,在这种情况下,要指定xml映射文件里面的namespace必须为接口的全路径名.

## Mybatis是如何进行分页的？分页插件的原理是什么？

> Mybatis是如何进行分页的？分页插件的原理是什么？

Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

**分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。**

举例：`select * from student，拦截sql后重写为：select t.* from （select * from student）t limit 0，10`

分页插件使用参考资料：

- https://www.cnblogs.com/kangoroo/p/7998433.html

- http://blog.csdn.net/yuchao2015/article/details/55001182

- https://www.cnblogs.com/ljdblog/p/6725094.html

  



## 简述Mybatis的插件运行原理，以及如何编写一个插件

> 简述Mybatis的插件运行原理，以及如何编写一个插件

Mybatis仅可以**编写针对ParameterHandler、ResultSetHandler、StatementHandler、Executor这4种接口的插件，Mybatis使用JDK的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能**，每当执行这4种接口对象的方法时，就会进入拦截方法，具体就是InvocationHandler的invoke()方法，当然，只会拦截那些你指定需要拦截的方法。

实现Mybatis的Interceptor接口并复写intercept()方法，**然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。**





## Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？

> Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？

Mybatis仅支持association关联对象和collection关联集合对象的延迟加载，association指的就是一对一，collection指的就是一对多查询。在Mybatis配置文件中，**可以配置是否启用延迟加载lazyLoadingEnabled=true|false。**

它的原理是，**使用CGLIB创建目标对象的代理对象**，当调用目标方法时，**进入拦截器方法**，比如调用a.getB().getName()，拦截器invoke()方法发现a.getB()是null值，那么就会单独发送事先保存好的查询关联B对象的sql，把B查询上来，然后调用a.setB(b)，于是a的对象b属性就有值了，接着完成a.getB().getName()方法的调用。这就是延迟加载的基本原理。

当然了，不光是Mybatis，几乎所有的包括Hibernate，支持延迟加载的原理都是一样的。

## Mybatis都有哪些Executor执行器？它们之间的区别是什么？

> Mybatis都有哪些Executor执行器？它们之间的区别是什么？

Mybatis有三种基本的Executor执行器，**SimpleExecutor、ReuseExecutor、BatchExecutor**。

- SimpleExecutor：每执行一次update或select，就开启一个Statement对象，**用完立刻关闭Statement对象**。
- ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用完后，不关闭Statement对象，而是放置于Map<String, Statement>内，供下一次使用。简言之，**就是重复使用Statement对象**。
- BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中（addBatch()），等待统一执行（executeBatch()），**它缓存了多个Statement对象，每个Statement对象都是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同**。

作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。

在 Mybatis 配置文件中，可以指定默认的 ExecutorType 执行器类型，也可以手动给DefaultSqlSessionFactory 的创建 SqlSession 的方法传递 ExecutorType 类型参数。

### Mybatis 是否可以映射 Enum 枚举类

Mybatis 可以映射枚举类，不单可以映射枚举类，Mybatis 可以映射任何对象到表的一列上。映射方式为自定义一个 TypeHandler，实现 TypeHandler 的 setParameter()和getResult()接口方法。TypeHandler 有两个作用，一是完成从 javaType 至 jdbcType 的转换，

二是完成 jdbcType 至 javaType 的转换，体现为 setParameter()和 getResult()两个方法，分别代表设置 sql 问号占位符参数和获取列查询结果。

## Hibernate和Mybatis常见问题

现在对hibernate和mybatis做一下对比，便于大家更好的理解和学习，使自己在做项目中更加得心应手。

第一方面：开发速度的对比

就开发速度而言，Hibernate的真正掌握要比Mybatis来得难些。Mybatis框架相对简单很容易上手，但也相对简陋些。个人觉得要用好Mybatis还是首先要先理解好Hibernate。

比起两者的开发速度，不仅仅要考虑到两者的特性及性能，更要根据项目需求去考虑究竟哪一个更适合项目开发，比如：一个项目中用到的复杂查询基本没有，就是简单的增删改查，这样选择hibernate效率就很快了，因为基本的sql语句已经被封装好了，根本不需要你去写sql语句，这就节省了大量的时间，但是对于一个大型项目，复杂语句较多，这样再去选择hibernate就不是一个太好的选择，选择mybatis就会加快许多，而且语句的管理也比较方便。

第二方面：开发工作量的对比

Hibernate和MyBatis都有相应的代码生成工具。可以生成简单基本的DAO层方法。针对高级查询，Mybatis需要手动编写SQL语句，以及ResultMap。而Hibernate有良好的映射机制，开发者无需关心SQL的生成与结果映射，可以更专注于业务流程。

第三方面：sql优化方面

Hibernate的查询会将表中的所有字段查询出来，这一点会有性能消耗。Hibernate也可以自己写SQL来指定需要查询的字段，但这样就破坏了Hibernate开发的简洁性。而Mybatis的SQL是手动编写的，所以可以按需求指定查询的字段。

Hibernate HQL语句的调优需要将SQL打印出来，而Hibernate的SQL被很多人嫌弃因为太丑了。MyBatis的SQL是自己手动写的所以调整方便。但Hibernate具有自己的日志统计。Mybatis本身不带日志统计，使用Log4j进行日志记录。

第四方面：对象管理的对比

Hibernate 是完整的对象/关系映射解决方案，它提供了对象状态管理（state management）的功能，使开发者不再需要理会底层数据库系统的细节。也就是说，相对于常见的 JDBC/SQL 持久层方案中需要管理 SQL 语句，Hibernate采用了更自然的面向对象的视角来持久化 Java 应用中的数据。

换句话说，使用 Hibernate 的开发者应该总是关注对象的状态（state），不必考虑 SQL 语句的执行。这部分细节已经由 Hibernate 掌管妥当，只有开发者在进行系统性能调优的时候才需要进行了解。而MyBatis在这一块没有文档说明，用户需要对对象自己进行详细的管理。

第五方面：缓存机制

**Hibernate缓存**

Hibernate一级缓存是Session缓存，利用好一级缓存就需要对Session的生命周期进行管理好。建议在一个Action操作中使用一个Session。一级缓存需要对Session进行严格管理。

Hibernate二级缓存是SessionFactory级的缓存。 SessionFactory的缓存分为内置缓存和外置缓存。内置缓存中存放的是SessionFactory对象的一些集合属性包含的数据(映射元素据及预定SQL语句等),对于应用程序来说,它是只读的。外置缓存中存放的是数据库数据的副本,其作用和一级缓存类似.二级缓存除了以内存作为存储介质外,还可以选用硬盘等外部存储设备。二级缓存称为进程级缓存或SessionFactory级缓存，它可以被所有session共享，它的生命周期伴随着SessionFactory的生命周期存在和消亡。

**MyBatis缓存**

MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制。MyBatis 3 中的缓存实现的很多改进都已经实现了,使得它更加强大而且易于配置。

默认情况下是没有开启缓存的,除了局部的 session 缓存,可以增强变现而且处理循环 依赖也是必须的。要开启二级缓存,你需要在你的 SQL 映射文件中添加一行:  <cache/>

字面上看就是这样。这个简单语句的效果如下:

- 映射语句文件中的所有 select 语句将会被缓存。
- 映射语句文件中的所有 insert,update 和 delete 语句会刷新缓存。
- 缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
- 根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序 来刷新。
- 缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用。
- 缓存会被视为是 read/write(可读/可写)的缓存,意味着对象检索不是共享的,而 且可以安全地被调用者修改,而不干扰其他调用者或线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改。

比如: <cache  eviction=”FIFO”  flushInterval=”60000″  size=”512″  readOnly=”true”/>

这个更高级的配置创建了一个 FIFO 缓存,并每隔 60 秒刷新,存数结果对象或列表的 512 个引用,而且返回的对象被认为是只读的,因此在不同线程中的调用者之间修改它们会 导致冲突。可用的收回策略有, 默认的是 LRU:

- LRU – 最近最少使用的:移除最长时间不被使用的对象。
- FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
- SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
- WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

flushInterval(刷新间隔)可以被设置为任意的正整数,而且它们代表一个合理的毫秒 形式的时间段。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。

size(引用数目)可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的 可用内存资源数目。默认值是1024。

readOnly(只读)属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓 存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存 会返回缓存对象的拷贝(通过序列化) 。这会慢一些,但是安全,因此默认是 false。

**相同点：**Hibernate和Mybatis的二级缓存除了采用系统默认的缓存机制外，都可以通过实现你自己的缓存或为其他第三方缓存方案，创建适配器来完全覆盖缓存行为。

**不同点：**Hibernate的二级缓存配置在SessionFactory生成的配置文件中进行详细配置，然后再在具体的表-对象映射中配置是那种缓存。

MyBatis的二级缓存配置都是在每个具体的表-对象映射中进行详细配置，这样针对不同的表可以自定义不同的缓存机制。并且Mybatis可以在命名空间中共享相同的缓存配置和实例，通过Cache-ref来实现。

**两者比较：**因为Hibernate对查询对象有着良好的管理机制，用户无需关心SQL。所以在使用二级缓存时如果出现脏数据，系统会报出错误并提示。

而MyBatis在这一方面，使用二级缓存时需要特别小心。如果不能完全确定数据更新操作的波及范围，避免Cache的盲目使用。否则，脏数据的出现会给系统的正常运行带来很大的隐患。

**他人总结**

- Hibernate功能强大，数据库无关性好，O/R映射能力强，如果你对Hibernate相当精通，而且对Hibernate进行了适当的封装，那么你的项目整个持久层代码会相当简单，需要写的代码很少，开发速度很快，非常爽。
- Hibernate的缺点就是学习门槛不低，要精通门槛更高，而且怎么设计O/R映射，在性能和对象模型之间如何权衡取得平衡，以及怎样用好Hibernate方面需要你的经验和能力都很强才行。
- iBATIS入门简单，即学即用，提供了数据库查询的自动对象绑定功能，而且延续了很好的SQL使用经验，对于没有那么高的对象模型要求的项目来说，相当完美。
- iBATIS的缺点就是框架还是比较简陋，功能尚有缺失，虽然简化了数据绑定代码，但是整个底层数据库查询实际还是要自己写的，工作量也比较大，而且不太容易适应快速数据库修改。



# Mybatis接口执行xml的sql原理

![image-20200423103313327](/Users/houshaojie/Library/Application Support/typora-user-images/image-20200423103313327.png)

说起mybatis，大伙应该都用过，有些人甚至底层源码都看过了。在mybatis中，mapper接口是没有实现类的，取而代之的是一个xml文件。也就是说我们调用mapper接口，其实是使用了mapper.xml中定义sql完成数据操作。

大家有没想过，为什么mapper没有实现类，它是如何和xml关联起来的？

先来搞个简单的查询：

UserMapper一个接口：

1. User findById(Long id);

userMapper.xml的sql语句

1. <mapper namespace="com.lfq.UserMapper">
2. ​    <select id="findById" resultType="com.lfq.User">
3. ​        select * *  from user where id = #{id}
4. ​    </select>
5. </mapper>

猜想

我们知道，接口是不直接被初始化的，但是可以被实现，所以new对象的时候是初始化实现类，然后接口再引用该对象。那么调用接口的方法实际上就是调用被引用对象的方法，也就是实现类的方法。

那么，UserMapper.findById被调用时候，不禁有这两个疑问？

- 被引用的对象是谁呢？
- 接口被调用时候发生了什么？

我们先来回答第二个问题，既然找不到实现类，UserMapper有没可能被代理起来呢，findById方法调用时候，我们找到代理对象来执行就行了。

代理有两种方式：

- 静态代理
- 动态代理

而静态代理基本是不可能的了，静态代理需要对UserMapper所有的方法进行重写。那么只能是动态代理，动态代理接口的所有方法，每次接口被调用，就会进入动态代理对象的invoke方法，然后加载xml中的sql完成操作数据库，再返回结果。

再然后说到动态代理，常见的方式有以下2种方式：

- JDK动态代理：
  - 利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。
- CGlib动态代理：
  - 利用ASM（开源的Java字节码编辑库，操作字节码）开源包，将代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

所以，动态代理代理还是对象类，那么我们只有接口，不能new，哪来的对象呢？别忘了，我们还有反射机制，我们是不是可以通过反射给接口生成对象，还记得**Class.***forName*吗。

综合上面的猜想：

第一步：通过反射机制给接口生成对象

第二步：动态代理反射对象，这样接口被调用，就会触发动态代理

嗯，好像有点道理，我果然是个天才！

知识点：动态代理

动态代理有几种实现方式，这里我们就先讲JDK动态代理，使用步骤如下：

- 新建一个接口
- 创建代理类，实现java.lang.reflect.InvocationHandler接口
- 接口测试

接口我们就用UserMapper，我们来写个代理对象。

1. public class Test {
2.   // 接口
3.   static interface Subject{
4. ​    void sayHi();
5. ​    void sayHello();
6.   }
7. 
8.   // 默认实现类（我们可以反射生成）
9.   static class SubjectImpl implements Subject{
10. ​    @Override
11. ​    public void sayHi() {
12. ​      System.out.println("hi");
13. ​    }
14. ​    @Override
15. ​    public void sayHello() {
16. ​      System.out.println("hello");
17. ​    }
18.   }
19. 
20.   // jkd动态代理
21.   // 原创：公众号：java思维导图
22.   static class ProxyInvocationHandler implements InvocationHandler{
23. ​    private Subject target;
24. ​    public ProxyInvocationHandler(Subject target) {
25. ​      this.target=target;
26. ​    }
27. 
28.    @Override
29.    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
30. ​      System.out.print("say:");
31. ​      return method.invoke(target, args);
32. ​    }
33.   }
34. 
35.   public static void main(String[] args) {
36. ​    Subject subject=new SubjectImpl();
37. 
38. ​    Subject subjectProxy=(Subject)             Proxy.newProxyInstance(subject.getClass().getClassLoader(), subject.getClass().getInterfaces(), new ProxyInvocationHandler(subject));
39. 
40. ​    subjectProxy.sayHi();
41. ​    subjectProxy.sayHello();
42. ​    }
43. }

ok，一个简单的动态代理例子送给你们，上面代码中关键生成动态代理对象的关键代码是：

1. Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h);

- loader: 用哪个类加载器去加载代理对象
- interfaces:动态代理类需要实现的接口
- h:动态代理方法在执行时，会调用h里面的invoke方法去执行

源码分析

好啦，上面该做的准备已经都准备好了，我们对mybatis的这个mapper接口大概都有些思路了，下面我们去正式验证一下，那么肯定就要去看源码了。我们只是去验证上面的mapper接口问题，所以不需要去看全部的代码，当然如果你看整个流程下来的话，会更加清晰。

论证猜想，我们可以采用结果导向的方式去看源码，从获取mapper那里开始看，也就是

1. SqlSession sqlSession = sqlSessionFactory.openSession();
2. UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

主要从sqlSession.getMapper(UserMapper.class);这里开始，先看整个UserMapper是不是被动态代理的。ok，我们进入代码中：

- org.apache.ibatis.session.SqlSessionManager#getMapper

1. @Override
2. public <T> T getMapper(Class<T> type) {
3.   return getConfiguration().getMapper(type, this);
4. }

继续走到Configuration方法里，Configuration是mybatis所有配置相关的地方，mybatis-cfg.xml、UserMapper.xml等文件都会被预先加载到Configuration里。

- org.apache.ibatis.session.Configuration#getMapper

1. public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
2.   return mapperRegistry.getMapper(type, sqlSession);
3. }

这时候，我们发现Configuration里面出现了一个mapperRegistry，翻译过来可以理解为mapper的注册器，其实在加载UserMapper.xml的时候，我们就需要在mapperRegistry里面进行注册，所有，我们可以从这里面进行获取。继续走~

- org.apache.ibatis.binding.MapperRegistry#getMapper

1. public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
2. 
3.   //获取mapper代理工厂
4.   // 原创：公众号：java思维导图
5.   final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
6. 
7.   if (mapperProxyFactory == null) {
8. ​    throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
9.   }
10.   try {
11. ​    return mapperProxyFactory.newInstance(sqlSession);
12.   } catch (Exception e) {
13. ​    throw new BindingException("Error getting mapper instance. Cause: " + e, e);
14.   }
15. }

ok，这里分为了两步：

- knownMappers.get(type);
- 获取已知的加载过的mapper中获取出mapper代理工厂
- mapperProxyFactory.newInstance(sqlSession);
- 代理工厂生成动态代理返回

我们一步步分析，别急，knownMappers其实是个map，根据userMapper.class获取MapperProxyFactory：

1. Map<Class<?>, MapperProxyFactory<?>> knownMappers

所以knownMappers必然是源码前面的步骤中set进去的。我们先找找，到底是哪里set进去的。找呀找，找到这里：

- org.apache.ibatis.builder.xml.XMLMapperBuilder#bindMapperForNamespace

1. private void bindMapperForNamespace() {
2.   String namespace = builderAssistant.getCurrentNamespace();
3.   if (namespace != null) {
4. ​    Class<?> boundType = null;
5. ​    try {
6. ​      //反射生成namespace的对象
7. ​      // 原创：公众号：java思维导图
8. ​      boundType = Resources.classForName(namespace);
9. ​    } catch (ClassNotFoundException e) {
10. ​      //ignore, bound type is not required
11. ​    }
12. ​    if (boundType != null) {
13. ​      if (!configuration.hasMapper(boundType)) {
14. ​        // Spring may not know the real resource name so we set a flag
15. ​        // to prevent loading again this resource from the mapper interface
16. ​        // look at MapperAnnotationBuilder#loadXmlResource
17. ​        configuration.addLoadedResource("namespace:" + namespace);// namespace:com.lfq.UserMapper
18. ​        configuration.addMapper(boundType);
19. ​      }
20. ​    }
21.   }
22. }

我们看这个boundType，是通过Resources.classForName(namespace);生成的class，Resources.classForName底层其实就是调用Class.forName生成的反射对象，而参数是namespace，namespacne不正是com.lfq.UserMapper嘛：

1. <mapper namespace="com.lfq.UserMapper">

完美！！Class.forName（com.lfq.UserMapper）生成反射对象。论证了我们第一点猜想。生成的boundType在被configuration.addMapper(boundType)；所以就有了：

- org.apache.ibatis.session.Configuration#addMapper

1. public <T> void addMapper(Class<T> type) {
2.   mapperRegistry.addMapper(type);
3. }

- org.apache.ibatis.binding.MapperRegistry#addMapper

1. public <T> void addMapper(Class<T> type) {
2.   if (type.isInterface()) {
3. ​    if (hasMapper(type)) {
4. ​      throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
5. ​    }
6. ​    boolean loadCompleted = false;
7. ​    try {
8. 
9. ​      /**
10. ​       \* TODO 将mapper接口包装成mapper代理
11. ​       \* 原创：公众号：java思维导图
12. ​       */
13. ​      knownMappers.put(type, new MapperProxyFactory<>(type));
14. 
15. ​      //解析接口上的注解或者加载mapper配置文件生成mappedStatement（com/lfq/UserMapper.java）
16. ​      MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
17. ​      // 开始解析
18. ​      parser.parse();
19. 
20. ​      // 加载完成标记
21. ​      loadCompleted = true;
22. ​    } finally {
23. ​      if (!loadCompleted) {
24. ​        knownMappers.remove(type);
25. ​      }
26. ​    }
27.   }
28. }

上面的代码，就给我论证了这个MapperProxyFactory是哪里来的，MapperProxyFactory里面其实就一个参数**mapperInterface**，就是反射生成的这个对象。ok，第一个猜想已经论证完毕，接着我们看刚才说到的第二点：动态代理。

回到mapperProxyFactory.newInstance(sqlSession); 这个MapperProxyFactory就是我们刚刚new出来的，我们打开newInstance方法看看：

- org.apache.ibatis.binding.MapperProxyFactory#newInstance(MapperProxy)

1. public T newInstance(SqlSession sqlSession) {
2. 
3.   //MapperProxy为InvocationHandler的实现类
4.   // 原创：公众号：java思维导图
5.   final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
6. 
7.   //真实生成代理
8.   return newInstance(mapperProxy);
9. }
10. protected T newInstance(MapperProxy<T> mapperProxy) {
11.   //采用JDK自带的Proxy代理模式生成
12.   return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
13. }

终于在里面看到Proxy.newProxyInstance了，好激动呀。又论证了第二点的动态代理猜想 上面代码中，首先把sqlSession, mapperInterface, methodCache三个参数封装到MapperProxy中，而MapperProxy是实现了InvocationHandler接口的方法，因此动态代理被调用的时候，会进入到MapperProxy的invoke方法中。

sqlSession是必须的，因为操作数据库需要用到sqlsession。具体invoke里面的内容，我们不做多分析啦，刚兴趣的同学自己去看下源码哈。可以猜想：找到对应的sql，然后执行sql操作，哈哈哈。





# SqlSession解读

在spring中，dao层大多都是用Mybatis,那么

**1，Mybatis执行sql最重要的是什么？**

在以前对Mybatis的源码解读中，我们知道，Mybatis利用了动态代理来做，最后实现的类是MapperProxy，在最后执行具体的方法时，实际上执行的是：

@Override  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {    if (Object.class.equals(method.getDeclaringClass())) {      try {        return method.invoke(this, args);      } catch (Throwable t) {        throw ExceptionUtil.unwrapThrowable(t);      }    }    final MapperMethod mapperMethod = cachedMapperMethod(method);    return mapperMethod.execute(sqlSession, args);  }

最重要的一步：

mapperMethod.execute(sqlSession, args);

这里的sqlSession 其实是在Spring的配置时设置的 sqlSessionTemplate,随便对其中的一个进行跟进：可以在sqlSessionTemplate类中发现很好这样的方法，用来执行具体的sql,如：

@Override  public <T> T selectOne(String statement, Object parameter) {    return this.sqlSessionProxy.<T> selectOne(statement, parameter);  }

这一步就是最后执行的方法，那么问题来了 **sqlSessionProxy** 到底是啥呢？ 这又得回到最开始。

**2，使用mybatis连接mysql时一般都是需要注入SqlSessionFactory，SqlSessionTemplate，PlatformTransactionManager。**

其中SqlSessionTemplate是生成sqlSession的模版，来看他的注入过程（注解形式注入）：

@Bean public SqlSessionTemplate sqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {    return new SqlSessionTemplate(sqlSessionFactory); }

在这个初始化过程中：

 /** * Constructs a Spring managed {@code SqlSession} with    the given * {@code SqlSessionFactory} and {@code ExecutorType}. * A custom {@code SQLExceptionTranslator} can be provided as an * argument so any {@code PersistenceException} thrown by MyBatis * can be custom translated to a {@code  RuntimeException} * The {@code SQLExceptionTranslator} can also be null and thus no * exception translation will be done and MyBatis exceptions will be * thrown * * @param sqlSessionFactory * @param executorType * @param exceptionTranslator */ public SqlSessionTemplate(SqlSessionFactory sqlSessionFactory, ExecutorType executorType,  PersistenceExceptionTranslator exceptionTranslator) { notNull(sqlSessionFactory, "Property 'sqlSessionFactory' is required"); notNull(executorType, "Property 'executorType' is required"); this.sqlSessionFactory = sqlSessionFactory; this.executorType = executorType; this.exceptionTranslator = exceptionTranslator; this.sqlSessionProxy = (SqlSession) newProxyInstance(    SqlSessionFactory.class.getClassLoader(),    new Class[] { SqlSession.class },    new SqlSessionInterceptor());

}

最后一步比较重要，用java动态代理生成了一个sqlSessionFactory。代理的类是：

 /** * Proxy needed to route MyBatis method calls to the proper SqlSession got * from Spring's Transaction Manager * It also unwraps exceptions thrown by {@code Method#invoke(Object, Object...)} to * pass a {@code PersistenceException} to the {@code PersistenceExceptionTranslator}. */ private class SqlSessionInterceptor implements InvocationHandler { @Override public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  SqlSession sqlSession = getSqlSession(      SqlSessionTemplate.this.sqlSessionFactory,      SqlSessionTemplate.this.executorType,      SqlSessionTemplate.this.exceptionTranslator);  try {    Object result = method.invoke(sqlSession, args);    if (!isSqlSessionTransactional(sqlSession, SqlSessionTemplate.this.sqlSessionFactory)) {      // force commit even on non-dirty sessions because some databases require      // a commit/rollback before calling close()      sqlSession.commit(true);    }    return result;  } catch (Throwable t) {    Throwable unwrapped = unwrapThrowable(t);    if (SqlSessionTemplate.this.exceptionTranslator != null && unwrapped instanceof PersistenceException) {      // release the connection to avoid a deadlock if the translator is no loaded. See issue #22      closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);      sqlSession = null;      Throwable translated = SqlSessionTemplate.this.exceptionTranslator.translateExceptionIfPossible((PersistenceException) unwrapped);      if (translated != null) {        unwrapped = translated;      }    }    throw unwrapped;  } finally {    if (sqlSession != null) {      closeSqlSession(sqlSession, SqlSessionTemplate.this.sqlSessionFactory);    }  } } }

在sqlSession执行sql的时候就会用这个代理类。isSqlSessionTransactional 这个会判断是不是有Transactional，没有则直接提交。如果有则不提交，在最外层进行提交。

其中

getSqlSession(SqlSessionTemplate.this.sqlSessionFactory,SqlSessionTemplate.this.executorType,SqlSessionTemplate.this.exceptionTranslator);

这个方法用来获取sqlSession。具体实现如下：

  /**   * Gets an SqlSession from Spring Transaction Manager or creates a new one if needed.   * Tries to get a SqlSession out of current transaction. If there is not any, it creates a new one.   * Then, it synchronizes the SqlSession with the transaction if Spring TX is active and   * <code>SpringManagedTransactionFactory</code> is configured as a transaction manager.   *   * @param sessionFactory a MyBatis {@code SqlSessionFactory} to create new sessions   * @param executorType The executor type of the SqlSession to create   * @param exceptionTranslator Optional. Translates SqlSession.commit() exceptions to Spring exceptions.   * @throws TransientDataAccessResourceException if a transaction is active and the   *             {@code SqlSessionFactory} is not using a {@code SpringManagedTransactionFactory}   * @see SpringManagedTransactionFactory   */  public static SqlSession getSqlSession(SqlSessionFactory sessionFactory, ExecutorType executorType, PersistenceExceptionTranslator exceptionTranslator) {     notNull(sessionFactory, NO_SQL_SESSION_FACTORY_SPECIFIED);    notNull(executorType, NO_EXECUTOR_TYPE_SPECIFIED);     SqlSessionHolder holder = (SqlSessionHolder) TransactionSynchronizationManager.getResource(sessionFactory);     SqlSession session = sessionHolder(executorType, holder);    if (session != null) {      return session;    }     if (LOGGER.isDebugEnabled()) {      LOGGER.debug("Creating a new SqlSession");    }     session = sessionFactory.openSession(executorType);     registerSessionHolder(sessionFactory, executorType, exceptionTranslator, session);     return session;  }

这个sqlSession的创建其实看他方法解释就够了，“从Spring事务管理器中获取一个sqlsession,如果没有，则创建一个新的”，这句话的意思其实就是如果有事务，则sqlSession用一个，如果没有，就给你个新的咯。

再通俗易懂一点：**如果在事务里，则Spring给你的sqlSession是一个，否则，每一个sql给你一个新的sqlSession。**这里生成的sqlSession其实就是DefaultSqlSession了。后续可能仍然有代理，如Mybatis分页插件等，不在此次讨论的范围内。

**3，第二步的 sqlSession 一样不一样到底有什么影响？**

在2中，我们看到如果是事务，sqlSession 一样，如果不是，则每次都不一样，且每次都会提交。这是最重要的。

sqlSession，顾名思义，就是sql的一个会话，在这个会话中发生的事不影响别的会话，如果会话提交，则生效，不提交不生效。

来看下sqlSession 这个接口的介绍。

/** * The primary Java interface for working with MyBatis. * Through this interface you can execute commands, get mappers and manage transactions. *  为Mybatis工作最重要的java接口，通过这个接口来执行命令，获取mapper以及管理事务 * @author Clinton Begin */

注释很明白了，来一一看看怎么起的这些作用。

**3.1，执行命令。**

在第一个小标题中 执行sql最重要的方法就是 this.sqlSessionProxy. selectOne(statement, parameter); 这个方法，而在第二个小标题中我们看到是通过代理来执行的，最后实际上没有事务则提交sql。这就是执行sql的基本动作了。获取sqlsession,提交执行Sql。

**3.2，获取mapper。**

在我们日常的代码中可能不会这么写，但是实际上，如果必要我们是可以这么做的，如：

XxxxxMapper xxxxxMapper = session.getMapper(xxxxxMapper.class);

一般情况下，如果要这么做，首先需要注入 sqlSessionFactory，然后利用

sqlSessionFactory.openSession()。

即可获取session。

**3.3，事务管理**

上面我一直提到一点，sqlSession 那个代理类里有个操作，判断这个是不是事务管理的sqlSession，如果是，则不提交，不是才提交，这个就是事务管理了，**那么有个问题，在哪里提交这个事务呢？？？？**

**4，事务从哪里拦截，就从哪里提交**

Spring中，如果一个方法被 @Transactional 注解标注，**在生效的情况下**(不生效的情况见我写动态代理的那篇博客)，则最终会被TransactionInterceptor 这个类所代理，执行的方法实际上是这样的：

@Override public Object invoke(final MethodInvocation invocation) throws Throwable {    // Work out the target class: may be {@code null}.    // The TransactionAttributeSource should be passed the target class    // as well as the method, which may be from an interface.    Class<?> targetClass = (invocation.getThis() != null ? AopUtils.getTargetClass(invocation.getThis()) : null);     // Adapt to TransactionAspectSupport's invokeWithinTransaction...    return invokeWithinTransaction(invocation.getMethod(), targetClass, new InvocationCallback() {        @Override        public Object proceedWithInvocation() throws Throwable {            return invocation.proceed();        }    }); }

继续看invokeWithinTransaction这个方法：

/** * General delegate for around-advice-based subclasses, delegating to several other template * methods on this class. Able to handle {@link CallbackPreferringPlatformTransactionManager} * as well as regular {@link PlatformTransactionManager} implementations. * @param method the Method being invoked * @param targetClass the target class that we're invoking the method on * @param invocation the callback to use for proceeding with the target invocation * @return the return value of the method, if any * @throws Throwable propagated from the target invocation */ protected Object invokeWithinTransaction(Method method, Class<?> targetClass, final InvocationCallback invocation)        throws Throwable {     // If the transaction attribute is null, the method is non-transactional.    final TransactionAttribute txAttr = getTransactionAttributeSource().getTransactionAttribute(method, targetClass);    final PlatformTransactionManager tm = determineTransactionManager(txAttr);    final String joinpointIdentification = methodIdentification(method, targetClass);     //基本上我们的事务管理器都不是一个CallbackPreferringPlatformTransactionManager，所以基本上都是会从这个地方进入，下面的else情况暂不讨论。    if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {        // Standard transaction demarcation with getTransaction and commit/rollback calls.        //获取具体的TransactionInfo ，如果要用编程性事务，则把这块的代码可以借鉴一下。        TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);        Object retVal = null;        try {            // This is an around advice: Invoke the next interceptor in the chain.            // This will normally result in a target object being invoked.            retVal = invocation.proceedWithInvocation(); //执行被@Transactional标注里面的具体方法。        }        catch (Throwable ex) {            // target invocation exception            //异常情况下，则直接完成了，因为在sqlsession执行完每一条指令都没有提交事务，所以表现出来的就是回滚事务。            completeTransactionAfterThrowing(txInfo, ex);            throw ex;        }        finally {            cleanupTransactionInfo(txInfo);        }        //正常执行完成的提交事务方法 跟进可以看到实际上执行的是:(编程性事务的提交)        // ==============txInfo.getTransactionManager().commit(txInfo.getTransactionStatus());===========        commitTransactionAfterReturning(txInfo);        return retVal;    }    // =======================else情况不讨论================================    else {        // It's a CallbackPreferringPlatformTransactionManager: pass a TransactionCallback in.        try {            Object result = ((CallbackPreferringPlatformTransactionManager) tm).execute(txAttr,                    new TransactionCallback<Object>() {                        @Override                        public Object doInTransaction(TransactionStatus status) {                            TransactionInfo txInfo = prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);                            try {                                return invocation.proceedWithInvocation();                            }                            catch (Throwable ex) {                                if (txAttr.rollbackOn(ex)) {                                    // A RuntimeException: will lead to a rollback.                                    if (ex instanceof RuntimeException) {                                        throw (RuntimeException) ex;                                    }                                    else {                                        throw new ThrowableHolderException(ex);                                    }                                }                                else {                                    // A normal return value: will lead to a commit.                                    return new ThrowableHolder(ex);                                }                            }                            finally {                                cleanupTransactionInfo(txInfo);                            }                        }                    });             // Check result: It might indicate a Throwable to rethrow.            if (result instanceof ThrowableHolder) {                throw ((ThrowableHolder) result).getThrowable();            }            else {                return result;            }        }        catch (ThrowableHolderException ex) {            throw ex.getCause();        }    } }

**5,小结，SqlSession 还在别的地方有用到吗？**

其实，Mybatis的一级缓存就是 SqlSession 级别的，只要SqlSession 不变，则默认缓存生效，也就是说，如下的代码，实际上只会查一次库的：

XxxxxMapper xxxxxMapper = session.getMapper(xxxxxMapper.class); //对应的sql为： select id from test_info; xxxxxMapper.selectFromDb(); xxxxxMapper.selectFromDb(); xxxxxMapper.selectFromDb();

实际上只会被执行一次，感兴趣的朋友们可以试试。

但是，在日常使用中，我们都是使用spring来管理Mapper,在执行selectFromDb 这个操作的时候，其实每次都会有一个新的SqlSession，所以，Mybatis的一级缓存是用不到的。

spring整合mybatis之后，通过动态代理的方式，使用SqlSessionTemplate持有的sqlSessionProxy属性来代理执行sql操作(比如下面SqlSessionTemplate类的insert方法)

public int insert(String statement) { return this.sqlSessionProxy.insert(statement); }

  最终，insert方法执行完操作后，会执行finally里面的代码对sqlSeesion进行关闭，因此，spring整合mybatis之后，由spring管理的sqlSeesion在sql方法(增删改查等操作)执行完毕后就自行关闭了sqlSession，不需要我们对其进行手动关闭

# Redis实现Mybatis的二级缓存

一、Mybatis的缓存

通大多数ORM层框架一样，Mybatis自然也提供了对一级缓存和二级缓存的支持。一下是一级缓存和二级缓存的作用于和定义。

   1、一级缓存是SqlSession级别的缓存。在操作数据库时需要构造 sqlSession对象，在对象中有一个(内存区域)数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession去操作数据库得到数据会存在二级缓存区域，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。 

   2、一级缓存的作用域是同一个SqlSession，在同一个sqlSession中两次执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写 到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个sqlSession结束后该sqlSession中的一级缓存 也就不存在了。Mybatis默认开启一级缓存。

   二级缓存是多个SqlSession共享的，其作用域是mapper的同一个namespace，不同的sqlSession两次执行相同 namespace下的sql语句且向sql中传递参数也相同即最终执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二 次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。Mybatis默认没有开启二级缓存需要在setting全局参数中配置开启二级缓存。

   一般的我们将Mybatis和Spring整合时，mybatis-spring包会自动分装sqlSession，而Spring通过动态代理 sqlSessionProxy使用一个模板方法封装了select()等操作，每一次select()查询都会自动先执行openSession()， 执行完close()以后调用close()方法，相当于生成了一个新的session实例，所以我们无需手动的去关闭这个session()，当然也无 法使用mybatis的一级缓存，也就是说mybatis的一级缓存在spring中是没有作用的。

   因此我们一般在项目中实现Mybatis的二级缓存，虽然Mybatis自带二级缓存功能，但是如果实在集群环境下，使用自带的二级缓存只是针对单个的节 点，所以我们采用分布式的二级缓存功能。一般的缓存NoSql数据库如redis，Mancache等，或者EhCache都可以实现，从而更好地服务 tomcat集群中ORM的查询。

Mybatis二级缓存默认采用的org.apache.ibatis.cache.impl.PerpetualCache实现的（基于内存中Map<Object, Object> cache），在项目进行分布式部署时，无法保证多实例间的分布式缓存一致性，故需要对该Cache实现进行修改以使之适应分布式部署。

Mybatis支持Ehcache二级缓存配置，默认适用于单实例部署，亦可以支持分布式部署（配置较为复杂，维护困难，支持RMI（手动、自动 - 组播 ）、JGroups、EhCache Server等部署方式）；

关于Ehcache分布式部署可参见：

https://blog.csdn.net/tang06211015/article/details/52281551

http://blog.sina.com.cn/s/blog_6151984a0101816j.html

Ehcache & Redis对比：

（1）Ehcache（分布式缓存）直接在jvm虚拟机中缓存，速度快，效率高；但是缓存共享麻烦，集群分布式应用不方便。

（2）Redis（集中式缓存）是通过socket访问到缓存服务，效率比ecache低，比数据库要快很多，处理集群和分布式缓存方便，有成熟的方案。

因此考虑使用Redis支持分布式缓存，Mybatis官方提供mybatis-redis插件（http://www.mybatis.org/redis-cache/），官方插件需要单独配置/redis.properties并且维护一个JedisConfigPool，考虑到单独配置与项目已有Redis配置重复且无法复用本地配置，因此决定参照官方org.mybatis.caches.redis.RedisCache来重写自己的MybatisRedisCache 。

MybatisRedisCache实现如下：

 import com.fasterxml.jackson.annotation.JsonAutoDetect;

import com.fasterxml.jackson.annotation.PropertyAccessor;

import com.fasterxml.jackson.databind.ObjectMapper;

import org.apache.ibatis.cache.Cache;

import org.apache.logging.log4j.LogManager;

import org.apache.logging.log4j.Logger;

import org.springframework.data.redis.core.RedisTemplate;

 

import java.util.concurrent.locks.ReadWriteLock;

import java.util.concurrent.locks.ReentrantReadWriteLock;

 

/**

 \* Mybatis - redis二级缓存

 *

 \* @author luohq

 \* @date 2019/3/15

 */

public final class MybatisRedisCache implements Cache {

​    /**

​     \* 日志工具类

​     */

​    private static final Logger logger = LogManager.getLogger(MybatisRedisCache.class);

 

​    /**

​     \* 读写锁

​     */

​    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

​    /**

​     \* ID

​     */

​    private String id;

 

​    /**

​     \* 集成redisTemplate

​     */

​    private static RedisTemplate redisTemplate;

 

​    /**

​     \* Jackson-databind对象序列化器（核心）

​     */

​    private static ObjectMapper objectMapper;

 

​    /**

​     \* 序列化器初始化

​     */

​    static {

​        objectMapper = new ObjectMapper();

​        //所有属性均可序列化

​        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);

​        /**

​         \* 序列化类型信息（使用objectMapper.readValue(hashVal, Object.class)也可以实现相应类型的序列化和反序列化

​         \* 好处：只定义一个序列化器就可以了（通用））

​         */

​        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

​    }

 

​    public MybatisRedisCache() {

​    }

 

​    public MybatisRedisCache(String id) {

​        if (id == null) {

​            throw new IllegalArgumentException("Cache instances require an ID");

​        } else {

​            logger.info("MybatisRedisCache.id={}", id);

​            this.id = id;

​        }

​    }

 

 

​    @Override

​    public String getId() {

​        return this.id;

​    }

 

​    @Override

​    public int getSize() {

​        try {

​            Long size = redisTemplate.opsForHash().size(this.id.toString());

​            logger.info("MybatisRedisCache.getSize: {}->{}", id, size);

​            return size.intValue();

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​        return 0;

​    }

 

​    @Override

​    public void putObject(final Object key, final Object value) {

​        try {

​            String hashVal = objectMapper.writeValueAsString(value);

​            logger.info("MybatisRedisCache.putObject: {}->{}->{}", id, key, hashVal);

​            redisTemplate.opsForHash().put(this.id.toString(), key.toString(), hashVal);

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​    }

 

​    @Override

​    public Object getObject(final Object key) {

​        try {

​            String hashVal = (String) redisTemplate.opsForHash().get(this.id.toString(), key.toString());

​            logger.info("MybatisRedisCache.getObject: {}->{}->{}", id, key, hashVal);

​            if (null == hashVal) {

​                return null;

​            }

​            return objectMapper.readValue(hashVal, Object.class);

​        } catch (Exception e) {

​            e.printStackTrace();

​            return null;

​        }

​    }

 

​    @Override

​    public Object removeObject(final Object key) {

​        try {

​            redisTemplate.opsForHash().delete(this.id.toString(), key.toString());

​            logger.info("MybatisRedisCache.removeObject: {}->{}->{}", id, key);

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​        return null;

​    }

 

​    @Override

​    public void clear() {

​        try {

​            redisTemplate.delete(this.id.toString());

​            logger.info("MybatisRedisCache.clear: {}", id);

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​    }

 

​    @Override

​    public ReadWriteLock getReadWriteLock() {

​        return this.readWriteLock;

​    }

 

​    @Override

​    public String toString() {

​        return "MybatisRedisCache {" + this.id + "}";

​    }

 

​    /**

​     \* 设置redisTemplate

​     *

​     \* @param redisTemplate

​     */

​    public void setRedisTemplate(RedisTemplate redisTemplate) {

​        MybatisRedisCache.redisTemplate = redisTemplate;

​    }

}

 

 MybatisRedisCache改进版：

改进如下：

（1）将序列化器直接配置进redisTemplate中，由redisTemplate来负责序列化；

代码如下：

 import com.xxx.util.JsonUtils;

import org.apache.ibatis.cache.Cache;

import org.apache.logging.log4j.LogManager;

import org.apache.logging.log4j.Logger;

import org.springframework.data.redis.core.RedisTemplate;

 

import java.util.concurrent.locks.ReadWriteLock;

import java.util.concurrent.locks.ReentrantReadWriteLock;

 

/**

 \* Mybatis - redis二级缓存

 *

 \* @author luohq

 \* @date 2019/3/15

 */

public final class MybatisRedisCache implements Cache {

​    /**

​     \* 日志工具类

​     */

​    private static final Logger logger = LogManager.getLogger(MybatisRedisCache.class);

 

​    /**

​     \* 读写锁

​     */

​    private final ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

​    /**

​     \* ID

​     */

​    private String id;

 

​    /**

​     \* 集成redisTemplate

​     */

​    private static RedisTemplate redisTemplate;

 

​    public MybatisRedisCache() {

​    }

 

​    public MybatisRedisCache(String id) {

​        if (id == null) {

​            throw new IllegalArgumentException("Cache instances require an ID");

​        } else {

​            logger.debug("MybatisRedisCache.id={}", id);

​            this.id = id;

​        }

​    }

 

 

​    @Override

​    public String getId() {

​        return this.id;

​    }

 

​    @Override

​    public int getSize() {

​        try {

​            Long size = redisTemplate.opsForHash().size(this.id.toString());

​            logger.debug("MybatisRedisCache.getSize: {}->{}", id, size);

​            return size.intValue();

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​        return 0;

​    }

 

​    @Override

​    public void putObject(final Object key, final Object value) {

​        try {

​            logger.debug("MybatisRedisCache.putObject: {}->{}->{}", id, key, JsonUtils.toJson(value));

​            redisTemplate.opsForHash().put(this.id.toString(), key.toString(), value);

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​    }

 

​    @Override

​    public Object getObject(final Object key) {

​        try {

​            Object hashVal = redisTemplate.opsForHash().get(this.id.toString(), key.toString());

​            logger.debug("MybatisRedisCache.getObject: {}->{}->{}", id, key, JsonUtils.toJson(hashVal));

​            return hashVal;

​        } catch (Exception e) {

​            e.printStackTrace();

​            return null;

​        }

​    }

 

​    @Override

​    public Object removeObject(final Object key) {

​        try {

​            redisTemplate.opsForHash().delete(this.id.toString(), key.toString());

​            logger.debug("MybatisRedisCache.removeObject: {}->{}->{}", id, key);

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​        return null;

​    }

 

​    @Override

​    public void clear() {

​        try {

​            redisTemplate.delete(this.id.toString());

​            logger.debug("MybatisRedisCache.clear: {}", id);

​        } catch (Exception e) {

​            e.printStackTrace();

​        }

​    }

 

​    @Override

​    public ReadWriteLock getReadWriteLock() {

​        return this.readWriteLock;

​    }

 

​    @Override

​    public String toString() {

​        return "MybatisRedisCache {" + this.id + "}";

​    }

 

​    /**

​     \* 设置redisTemplate

​     *

​     \* @param redisTemplate

​     */

​    public void setRedisTemplate(RedisTemplate redisTemplate) {

​        MybatisRedisCache.redisTemplate = redisTemplate;

​    }

}

 

 import com.fasterxml.jackson.annotation.JsonAutoDetect;

import com.fasterxml.jackson.annotation.PropertyAccessor;

import com.fasterxml.jackson.databind.ObjectMapper;

import org.springframework.context.annotation.Bean;

import org.springframework.context.annotation.Configuration;

import org.springframework.data.redis.connection.RedisConnectionFactory;

import org.springframework.data.redis.core.RedisTemplate;

import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

import org.springframework.data.redis.serializer.StringRedisSerializer;

/**

 \* Redis配置

 *

 \* @author luohq

 \* @date: 2019/03/13

 */

@Configuration

public class RedisConfig {

​    /**

​     \* 配置RedisTemplate

​     *

​     \* @param factory

​     \* @return

​     */

​    @Bean

​    public RedisTemplate redisTemplate(RedisConnectionFactory factory, Jackson2JsonRedisSerializer redisJsonSerializer) {

​        RedisTemplate<Object, Object> template = new RedisTemplate<>();

​        //redis连接工厂

​        template.setConnectionFactory(factory);

​        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();

​        //redis.key序列化器

​        template.setKeySerializer(stringRedisSerializer);

​        //redis.value序列化器

​        template.setValueSerializer(redisJsonSerializer);

​        //redis.hash.key序列化器

​        template.setHashKeySerializer(stringRedisSerializer);

​        //redis.hash.value序列化器

​        template.setHashValueSerializer(redisJsonSerializer);

​        //调用其他初始化逻辑

​        template.afterPropertiesSet();

​        //这里设置redis事务一致

​        template.setEnableTransactionSupport(true);

​        return template;

​    }

​    /**

​     \* 配置redis Json序列化器

​     *

​     \* @return

​     */

​    @Bean

​    public Jackson2JsonRedisSerializer redisJsonSerializer() {

​        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）

​        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);

​        ObjectMapper mapper = new ObjectMapper();

​        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);

​        mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

​        serializer.setObjectMapper(mapper);

​        return serializer;

​    }

}

使用方式：

（1）开启Mybatis二级缓存设置

方式1：mybatis-config.xml

 <configuration>    <settings>        <!-- 开启二级缓存 -->        <setting name="cacheEnabled" value="true"/>    </settings>   ... </configuration>

方式2：Springboot - application.properties

 \#使全局的映射器启用或禁用缓存。

mybatis.configuration.cache-enabled=true

 

（2）在mapper.xml使用二级缓存

 <?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd"> <mapper namespace="com.xxx.dao.UserTestMapper">   <!-- 开启二级缓存 -->   <cache type="com.xxx.config.MybatisRedisCache"/>   ...... </mapper>



# 一级缓存详解

**什么是缓存**

**
**

缓存就是内存中的一个对象，用于对数据库查询结果的保存，用于减少与数据库的交互次数从而降低数据库的压力，进而提高响应速度。



**什么是MyBatis中的缓存**

**
**

MyBatis 中的缓存就是说 MyBatis 在执行一次SQL查询或者SQL更新之后，这条SQL语句并不会消失，而是被MyBatis 缓存起来，当再次执行相同SQL语句的时候，就会直接从缓存中进行提取，而不是再次执行SQL命令。



MyBatis中的缓存分为一级缓存和二级缓存，一级缓存又被称为 SqlSession 级别的缓存，二级缓存又被称为表级缓存。



SqlSession是什么？SqlSession 是SqlSessionFactory会话工厂创建出来的一个会话的对象，这个SqlSession对象用于执行具体的SQL语句并返回给用户请求的结果。



SqlSession级别的缓存是什么意思？SqlSession级别的缓存表示的就是每当执行一条SQL语句后，默认就会把该SQL语句缓存起来，也被称为会话缓存



 **MyBatis 中的一级缓存**

**
**

一级缓存是 SqlSession级别 的缓存。在操作数据库时需要构造 sqlSession 对象，在对象中有一个(内存区域)数据结构（HashMap）用于存储缓存数据。不同的 sqlSession 之间的缓存数据区域（HashMap）是互相不影响的。用一张图来表示一下一级缓存，其中每一个 SqlSession 的内部都会有一个一级缓存对象。



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



在应用运行过程中，我们有可能在一次数据库会话中，执行多次查询条件完全相同的SQL，MyBatis 提供了一级缓存的方案优化这部分场景，如果是相同的SQL语句，会优先命中一级缓存，避免直接对数据库进行查询，提高性能。



**初探一级缓存**

**
**

我们继续使用 MyBatis基础搭建以及配置详解中的例子(https://mp.weixin.qq.com/s/Ys03zaTSaOakdGU4RlLJ1A)进行 一级缓存的探究。



在对应的 resources 根目录下加上日志的输出信息 log4j.properties



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
##define an appender named consolelog4j.appender.console=org.apache.log4j.ConsoleAppender#The Target value is System.out or System.errlog4j.appender.console.Target=System.out#set the layout type of the apperderlog4j.appender.console.layout=org.apache.log4j.PatternLayout#set the layout format patternlog4j.appender.console.layout.ConversionPattern=[%-5p] %m%n
##define a loggerlog4j.rootLogger=debug,console
```



**模拟思路**：既然每个 SqlSession 都会有自己的一个缓存，那么我们用同一个 SqlSession 是不是就能感受到一级缓存的存在呢？调用多次 getMapper 方法，生成对应的SQL语句，判断每次SQL语句是从缓存中取还是对数据库进行操作，下面的例子来证明一下



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
@Testpublic void test(){  DeptDao deptDao = sqlSession.getMapper(DeptDao.class);  Dept dept = deptDao.findByDeptNo(1);  System.out.println(dept);
  DeptDao deptDao2 = sqlSession.getMapper(DeptDao.class);  Dept dept2 = deptDao2.findByDeptNo(1);  System.out.println(dept2);  System.out.println(deptDao2.findByDeptNo(1));}
```



输出：

**
**

![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJdG25jhkaYxgbAnbP81CD5v0lWdmnr6hy2rdPKyKlfccUUvvaVOvSlGg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看到，上面代码执行了三条相同的SQL语句，但是只有一条SQL语句进行了输出，其他两条SQL语句都是从缓存中查询的，所以它们生成了相同的 Dept 对象。



**探究一级缓存是如何失效的**

**
**

上面的一级缓存初探让我们感受到了 MyBatis 中一级缓存的存在，那么现在你或许就会有疑问了，那么什么时候缓存失效呢？这个问题也就是我们接下来需要详细讨论的议题之一。



**探究更新对一级缓存失效的影响**

**
**

上面的代码执行了三次相同的查询操作，返回了相同的结果，那么，如果我在第一条和第二条SQL语句之前插入更新的SQL语句，是否会对一级缓存产生影响呢？代码如下：



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
@Testpublic void testCacheLose(){  DeptDao deptDao = sqlSession.getMapper(DeptDao.class);  Dept dept = deptDao.findByDeptNo(1);  System.out.println(dept);
  // 在两次查询之间使用 更新 操作，是否会对一级缓存产生影响  deptDao.insertDept(new Dept(7,"tengxun","shenzhen"));  //        deptDao.updateDept(new Dept(1,"zhongke","sjz"));  //        deptDao.deleteByDeptNo(7);
  DeptDao deptDao2 = sqlSession.getMapper(DeptDao.class);  Dept dept2 = deptDao2.findByDeptNo(1);  System.out.println(dept2);}
```



为了演示效果，就不贴出 insertDept 的代码了，就是一条简单的插入语句。



分别放开不同的更新语句，发现执行效果如下



输出结果：



![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJdHawXt9lZH3PWTlKToDePicdPAnPNCODbQfKTKatdr1trc2fAqDqiauAg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



如图所示，在两次查询语句中使用插入，会对一级缓存进行刷新，会导致一级缓存失效。



探究不同的 SqlSession 对一级缓存的影响

**
**

如果你看到这里了，那么你应该知道一级缓存就是 SqlSession 级别的缓存，而同一个 SqlSession 会有相同的一级缓存，那么使用不同的 SqlSession 是不是会对一级缓存产生影响呢？显而易见是的，那么下面就来演示并且证明一下



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
private SqlSessionFactory factory; // 把factory设置为全局变量
@Testpublic void testCacheLoseWithSqlSession(){  DeptDao deptDao = sqlSession.getMapper(DeptDao.class);  Dept dept = deptDao.findByDeptNo(1);  System.out.println(dept);
  SqlSession sqlSession2 = factory.openSession();  DeptDao deptDao2 = sqlSession2.getMapper(DeptDao.class);  Dept dept2 = deptDao2.findByDeptNo(1);  System.out.println(dept2);
}
```



输出：



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



上面代码使用了不同的 SqlSession 对同一个SQL语句执行了相同的查询操作，却对数据库执行了两次相同的查询操作，生成了不同的 dept 对象，由此可见，不同的 SqlSession 是肯定会对一级缓存产生影响的。



同一个 SqlSession 使用不同的查询操作

**
**

使用不同的查询条件是否会对一级缓存产生影响呢？可能在你心里已经有这个答案了，再来看一下代码吧



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
@Testpublic void testWithDifferentParam(){  DeptDao deptDao = sqlSession.getMapper(DeptDao.class);  Dept dept = deptDao.findByDeptNo(1);  System.out.println(dept);
  DeptDao deptDao2 = sqlSession.getMapper(DeptDao.class);  Dept dept2 = deptDao2.findByDeptNo(5);  System.out.println(dept2);}
```



输出结果



![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJddOFHhqwEoKNKYibMDobWbQu97mwHacfNg227y5LtKCnplNPW9mrnXhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



我们在两次查询SQL分别使用了不同的查询条件，查询出来的数据不一致，那就肯定会对一级缓存产生影响了。



手动清理缓存对一级缓存的影响



我们在两次查询的SQL语句之间使用 clearCache 是否会对一级缓存产生影响呢？下面例子证实了这一点



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
@Testpublic void testClearCache(){  DeptDao deptDao = sqlSession.getMapper(DeptDao.class);  Dept dept = deptDao.findByDeptNo(1);  System.out.println(dept);
  //在两次相同的SQL语句之间使用查询操作，对一级缓存的影响。  sqlSession.clearCache();
  DeptDao deptDao2 = sqlSession.getMapper(DeptDao.class);  Dept dept2 = deptDao2.findByDeptNo(1);  System.out.println(dept2);}
```



输出：



![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJdZHVnNCGcEj3NWcBfQtakFnhlsC7uXKDvAKQnqeN6pJVj1sjKRJyVRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



我们在两次查询操作之间，使用了 sqlSession 的 clearCache() 方法清除了一级缓存，所以使用 clearCache 也会对一级缓存产生影响。



**一级缓存原理探究**



一级缓存到底是什么？一级缓存的工作流程是怎样的？一级缓存何时消失？相信你现在应该会有这几个疑问，那么我们本节就来研究一下一级缓存的本质



嗯。。。。。。该从何处入手呢？



你可以这样想，上面我们一直提到一级缓存，那么提到一级缓存就绕不开 SqlSession，所以索性我们就直接从 SqlSession ，看看有没有创建缓存或者与缓存有关的属性或者方法



![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJdlwnibWouwxUK6Shxdic2bkVPzyf8gGtqn4u82Siaea0llZJ13jAUShJibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



调研了一圈，发现上述所有方法中，好像只有 `clearCache()` 和缓存沾点关系，那么就直接从这个方法入手吧，分析源码时,**我们要看它(此类)是谁，它的父类和子类分别又是谁**，对如上关系了解了，你才会对这个类有更深的认识，分析了一圈，你可能会得到如下这个流程图



![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJdSBakL5cVFJkCBCZjr7B9GIqJ8WOoPCvjn3cE9BGJ1rCcjnIjicbSIrA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



再深入分析，流程走到Perpetualcache 中的 clear() 方法之后，会调用其 cache.clear() 方法，那么这个cache 是什么东西呢？ 点进去发现，cache 其实就是 private Map<Object, Object> cache = new HashMap<Object, Object>(); 也就是一个Map，所以说 cache.clear() 其实就是 map.clear() ，也就是说，缓存其实就是本地存放的一个 map 对象，每一个SqlSession 都会存放一个 map 对象的引用，那么这个 cache 是何时创建的呢？



你觉得最有可能创建缓存的地方是哪里呢？ 我觉得是 Executor，为什么这么认为？ 因为 Executor 是执行器，用来执行SQL请求，而且清除缓存的方法也在 Executor 中执行，所以很可能缓存的创建也很有可能在 Executor 中，看了一圈发现 Executor 中有一个 createCacheKey 方法，这个方法很像是创建缓存的方法啊，跟进去看看，你发现 createCacheKey 方法是由 `BaseExecutor` 执行的，代码如下



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
CacheKey cacheKey = new CacheKey();//MappedStatement的id// id 就是Sql语句的所在位置 包名 + 类名 + SQL名称cacheKey.update(ms.getId());// offset 就是 0cacheKey.update(rowBounds.getOffset());// limit 就是 Integer.MAXVALUEcacheKey.update(rowBounds.getLimit());// 具体的SQL语句cacheKey.update(boundSql.getSql());//后面是update了sql中带的参数cacheKey.update(value);...if (configuration.getEnvironment() != null) {  // issue #176  cacheKey.update(configuration.getEnvironment().getId());}
```



创建缓存key会经过一系列的 update 方法，update 方法由一个 CacheKey 这个对象来执行的，这个 update 方法最终由 updateList 的 list 来把五个值存进去，对照上面的代码和下面的图示，你应该能理解这五个值都是什么了



![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJd1qyBAdGvoatAbFhPRwsEYLCmFiaSZ5w6vwxWldtajQIMV96EnzDvdCA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



这里需要注意一下最后一个值， configuration.getEnvironment().getId() 这是什么，这其实就是定义在 mybatis-config.xml 中的 <environment> 标签，见如下。



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
<environments default="development"> <environment id="development">    <transactionManager type="JDBC"/>    <dataSource type="POOLED">      <property name="driver" value="${jdbc.driver}"/>     <property name="url" value="${jdbc.url}"/>      <property name="username" value="${jdbc.username}"/>      <property name="password" value="${jdbc.password}"/>    </dataSource>  </environment>></environments>
```



那么我们回归正题，那么创建完缓存之后该用在何处呢？总不会凭空创建一个缓存不使用吧？绝对不会的，经过我们对一级缓存的探究之后，我们发现一级缓存更多是用于查询操作，毕竟一级缓存也叫做查询缓存吧，为什么叫查询缓存我们一会儿说。我们先来看一下这个缓存到底用在哪了，我们跟踪到 query 方法如下：



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
@Overridepublic <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {  BoundSql boundSql = ms.getBoundSql(parameter);  // 创建缓存  CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);  return query(ms, parameter, rowBounds, resultHandler, key, boundSql);}
@SuppressWarnings("unchecked")@Overridepublic <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {  ...  list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;  if (list != null) {      // 这个主要是处理存储过程用的。      handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);      } else {      list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);  }  ...}
// queryFromDatabase 方法private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {  List<E> list;  localCache.putObject(key, EXECUTION_PLACEHOLDER);  try {    list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);  } finally {    localCache.removeObject(key);  }  localCache.putObject(key, list);  if (ms.getStatementType() == StatementType.CALLABLE) {    localOutputParameterCache.putObject(key, parameter);  }  return list;}
```



如果查不到的话，就从数据库查，在queryFromDatabase中，会对localcache进行写入。localcache 对象的put 方法最终交给 Map 进行存放



- 
- 
- 
- 
- 
- 

```
private Map<Object, Object> cache = new HashMap<Object, Object>();
@Overridepublic void putObject(Object key, Object value) {  cache.put(key, value);}
```



那么再说一下为什么一级缓存也叫做查询缓存呢？



我们先来看一下 update 更新方法，先来看一下 update 的源码



- 
- 
- 
- 
- 
- 
- 
- 
- 

```
@Overridepublic int update(MappedStatement ms, Object parameter) throws SQLException {  ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());  if (closed) {    throw new ExecutorException("Executor was closed.");  }  clearLocalCache();  return doUpdate(ms, parameter);}
```



由 BaseExecutor 在每次执行 update 方法的时候，都会先 clearLocalCache() ，所以更新方法并不会有缓存，这也就是说为什么一级缓存也叫做查询缓存了，这也就是为什么我们没有探究多次执行更新方法对一级缓存的影响了。



**还有其他要补充的吗？**

**
**

我们上面分析了一级缓存的执行流程，为什么一级缓存要叫查询缓存以及一级缓存组成条件



那么，你可能看到这感觉这些知识还是不够连贯，那么我就帮你把 `一级缓存的探究 `小结中的原理说一下吧，为什么一级缓存会失效



\1. 探究更新对一级缓存失效的影响： 由上面的分析结论可知，我们每次执行 update 方法时，都会先刷新一级缓存，因为是同一个 SqlSession, 所以是由同一个 Map 进行存储的，所以此时一级缓存会失效

\2. 探究不同的 SqlSession 对一级缓存的影响： 这个也就比较好理解了，因为不同的 SqlSession 会有不同的Map 存储一级缓存，然而 SqlSession 之间也不会共享，所以此时也就不存在相同的一级缓存

\3. 同一个 SqlSession 使用不同的查询操作： 这个论点就需要从缓存的构成角度来讲了，我们通过 cacheKey 可知，一级缓存命中的必要条件是两个 cacheKey 相同，要使得 cacheKey 相同，就需要使 cacheKey 里面的值相同，也就是



![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJdiazjszlZ3sjp89NsZ6aOBtx73OibGtuliaTwVibtXXua7bQjsZ6xllPJlA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/laEmibHFxFw6wy99AkjbP5CpK2TQEZSJdbIudyvicvs5JicgJEiciaHiaybTwTRzIKQP722oSicOBTtQMhRUKwT2zicYUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**
**

看出差别了吗？第一个SQL 我们查询的是部门编号为1的值，而第二个SQL我们查询的是编号为5的值，两个缓存对象不相同，所以也就不存在缓存。



\4. 手动清理缓存对一级缓存的影响： 由程序员自己去调用clearCache方法，这个方法就是清除缓存的方法，所以也就不存在缓存了。



**总结**

**
**

所以此文章到底写了点什么呢？抛给你几个问题了解一下



\1. 什么是缓存？什么是 MyBatis 缓存？



\2. 认识MyBatis缓存，MyBatis 一级缓存的失效方式

\3. MyBatis 一级缓存的执行流程，MyBatis 一级缓存究竟是什么？



文章参考：



mybatis的缓存机制（一级缓存二级缓存和刷新缓存）和mybatis整合ehcache



https://blog.csdn.net/u012373815/article/details/47069223



[聊聊MyBatis缓存机制](https://tech.meituan.com/2018/01/19/mybatis-cache.html)





#[MyBatis 二级缓存全详解](https://www.cnblogs.com/cxuanBlog/p/11333021.html)

## MyBatis 二级缓存介绍

上一篇文章中我们介绍到了 MyBatis 一级缓存其实就是 SqlSession 级别的缓存，什么是 SqlSession 级别的缓存呢？一级缓存的本质是什么呢？ 以及一级缓存失效的原因？我希望你在看下文之前能够回想起来这些内容。

MyBatis 一级缓存最大的共享范围就是一个SqlSession内部，那么如果多个 SqlSession 需要共享缓存，则需要开启二级缓存，开启二级缓存后，会使用 CachingExecutor 装饰 Executor，进入一级缓存的查询流程前，先在CachingExecutor 进行二级缓存的查询，具体的工作流程如下所示

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211431621-1800471104.png)

当二级缓存开启后，同一个命名空间(namespace) 所有的操作语句，都影响着一个**共同的 cache**，也就是二级缓存被多个 SqlSession 共享，是一个**全局的变量**。当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

### 二级缓存开启条件

二级缓存默认是不开启的，需要手动开启二级缓存，实现二级缓存的时候，MyBatis要求返回的POJO必须是可序列化的。开启二级缓存的条件也是比较简单，通过直接在 MyBatis 配置文件中通过

```xml
<settings>
	<setting name = "cacheEnabled" value = "true" />
</settings>
```

来开启二级缓存，还需要在 Mapper 的xml 配置文件中加入 ``标签

**设置 cache 标签的属性**

cache 标签有多个属性，一起来看一些这些属性分别代表什么意义

- ```
  eviction
  ```

  : 缓存回收策略，有这几种回收策略

  - LRU - 最近最少回收，移除最长时间不被使用的对象
  - FIFO - 先进先出，按照缓存进入的顺序来移除它们
  - SOFT - 软引用，移除基于垃圾回收器状态和软引用规则的对象
  - WEAK - 弱引用，更积极的移除基于垃圾收集器和弱引用规则的对象

> 默认是 LRU 最近最少回收策略

- `flushinterval` 缓存刷新间隔，缓存多长时间刷新一次，默认不清空，设置一个毫秒值
- `readOnly`: 是否只读；**true 只读**，MyBatis 认为所有从缓存中获取数据的操作都是只读操作，不会修改数据。MyBatis 为了加快获取数据，直接就会将数据在缓存中的引用交给用户。不安全，速度快。**读写(默认)**：MyBatis 觉得数据可能会被修改
- `size` : 缓存存放多少个元素
- `type`: 指定自定义缓存的全类名(实现Cache 接口即可)
- `blocking`： 若缓存中找不到对应的key，是否会一直blocking，直到有对应的数据进入缓存。

### 探究二级缓存

我们继续以 MyBatis 一级缓存文章中的例子为基础，搭建一个满足二级缓存的例子，来对二级缓存进行探究，例子如下(对 一级缓存的例子部分源码进行修改)：

Dept.java

```java
//存放在共享缓存中数据进行序列化操作和反序列化操作
//因此数据对应实体类必须实现【序列化接口】
public class Dept implements Serializable {

    private Integer deptNo;
    private String  dname;
    private String  loc;

    public Dept() {}
    public Dept(Integer deptNo, String dname, String loc) {
        this.deptNo = deptNo;
        this.dname = dname;
        this.loc = loc;
    }

   get and set...
    @Override
    public String toString() {
        return "Dept{" +
                "deptNo=" + deptNo +
                ", dname='" + dname + '\'' +
                ", loc='" + loc + '\'' +
                '}';
    }
}
```

myBatis-config.xml

在myBatis-config 中添加开启二级缓存的条件

```xml
<!-- 通知 MyBatis 框架开启二级缓存 -->
<settings>
  <setting name="cacheEnabled" value="true"/>
</settings>
```

DeptDao.xml

还需要在 Mapper 对应的xml中添加 cache 标签，表示对哪个mapper 开启缓存

```xml
<!-- 表示DEPT表查询结果保存到二级缓存(共享缓存) -->
<cache/>
```

对应的二级缓存测试类如下：

```java
public class MyBatisSecondCacheTest {

    private SqlSession sqlSession;
    SqlSessionFactory factory;
    @Before
    public void start() throws IOException {
        InputStream is = Resources.getResourceAsStream("myBatis-config.xml");
        SqlSessionFactoryBuilder builderObj = new SqlSessionFactoryBuilder();
        factory = builderObj.build(is);
        sqlSession = factory.openSession();
    }
    @After
    public void destory(){
        if(sqlSession!=null){
            sqlSession.close();
        }
    }

    @Test
    public void testSecondCache(){
        //会话过程中第一次发送请求，从数据库中得到结果
        //得到结果之后，mybatis自动将这个查询结果放入到当前用户的一级缓存
        DeptDao dao =  sqlSession.getMapper(DeptDao.class);
        Dept dept = dao.findByDeptNo(1);
        System.out.println("第一次查询得到部门对象 = "+dept);
        //触发MyBatis框架从当前一级缓存中将Dept对象保存到二级缓存

        sqlSession.commit();
      	// 改成 sqlSession.close(); 效果相同

        SqlSession session2 = factory.openSession();
        DeptDao dao2 = session2.getMapper(DeptDao.class);
        Dept dept2 = dao2.findByDeptNo(1);
        System.out.println("第二次查询得到部门对象 = "+dept2);
    }
}
```

> 测试二级缓存效果，提交事务，`sqlSession`查询完数据后，`sqlSession2`相同的查询是否会从缓存中获取数据。

测试结果如下：

![image-20190720161244429](/Users/mr.l/Library/Application Support/typora-user-images/image-20190720161244429.png)

通过结果可以得知，首次执行的SQL语句是从数据库中查询得到的结果，然后第一个 SqlSession 执行提交，第二个 SqlSession 执行相同的查询后是从缓存中查取的。

用一下这幅图能够比较直观的反映两次 SqlSession 的缓存命中

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211454258-1861395227.png)

## 二级缓存失效的条件

与一级缓存一样，二级缓存也会存在失效的条件的，下面我们就来探究一下哪些情况会造成二级缓存失效

### 第一次SqlSession 未提交

SqlSession 在未提交的时候，SQL 语句产生的查询结果还没有放入二级缓存中，这个时候 SqlSession2 在查询的时候是感受不到二级缓存的存在的，修改对应的测试类，结果如下：

```java
@Test
public void testSqlSessionUnCommit(){
  //会话过程中第一次发送请求，从数据库中得到结果
  //得到结果之后，mybatis自动将这个查询结果放入到当前用户的一级缓存
  DeptDao dao =  sqlSession.getMapper(DeptDao.class);
  Dept dept = dao.findByDeptNo(1);
  System.out.println("第一次查询得到部门对象 = "+dept);
  //触发MyBatis框架从当前一级缓存中将Dept对象保存到二级缓存

  SqlSession session2 = factory.openSession();
  DeptDao dao2 = session2.getMapper(DeptDao.class);
  Dept dept2 = dao2.findByDeptNo(1);
  System.out.println("第二次查询得到部门对象 = "+dept2);
}
```

产生的输出结果：

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211506330-767047409.png)

### 更新对二级缓存影响

与一级缓存一样，更新操作很可能对二级缓存造成影响，下面用三个 SqlSession来进行模拟，第一个 SqlSession 只是单纯的提交，第二个 SqlSession 用于检验二级缓存所产生的影响，第三个 SqlSession 用于执行更新操作，测试如下：

```java
@Test
public void testSqlSessionUpdate(){
  SqlSession sqlSession = factory.openSession();
  SqlSession sqlSession2 = factory.openSession();
  SqlSession sqlSession3 = factory.openSession();

  // 第一个 SqlSession 执行更新操作
  DeptDao deptDao = sqlSession.getMapper(DeptDao.class);
  Dept dept = deptDao.findByDeptNo(1);
  System.out.println("dept = " + dept);
  sqlSession.commit();

  // 判断第二个 SqlSession 是否从缓存中读取
  DeptDao deptDao2 = sqlSession2.getMapper(DeptDao.class);
  Dept dept2 = deptDao2.findByDeptNo(1);
  System.out.println("dept2 = " + dept2);

  // 第三个 SqlSession 执行更新操作
  DeptDao deptDao3 = sqlSession3.getMapper(DeptDao.class);
  deptDao3.updateDept(new Dept(1,"ali","hz"));
  sqlSession3.commit();

  // 判断第二个 SqlSession 是否从缓存中读取
  dept2 = deptDao2.findByDeptNo(1);
  System.out.println("dept2 = " + dept2);
}
```

对应的输出结果如下

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211519078-378164910.png)

### 探究多表操作对二级缓存的影响

现有这样一个场景，有两个表，部门表dept（deptNo,dname,loc）和 部门数量表deptNum（id,name,num），其中部门表的名称和部门数量表的名称相同，通过名称能够联查两个表可以知道其坐标(loc)和数量(num)，现在我要对部门数量表的 num 进行更新，然后我再次关联dept 和 deptNum 进行查询，你认为这个 SQL 语句能够查询到的 num 的数量是多少？来看一下代码探究一下

DeptNum.java

```java
public class DeptNum {

    private int id;
    private String name;
    private int num;

    get and set...
}
```

DeptVo.java

```java
public class DeptVo {

    private Integer deptNo;
    private String  dname;
    private String  loc;
    private Integer num;

    public DeptVo(Integer deptNo, String dname, String loc, Integer num) {
        this.deptNo = deptNo;
        this.dname = dname;
        this.loc = loc;
        this.num = num;
    }

    public DeptVo(String dname, Integer num) {
        this.dname = dname;
        this.num = num;
    }

    get and set

    @Override
    public String toString() {
        return "DeptVo{" +
                "deptNo=" + deptNo +
                ", dname='" + dname + '\'' +
                ", loc='" + loc + '\'' +
                ", num=" + num +
                '}';
    }
}
```

DeptDao.java

```java
public interface DeptDao {

    ...

    DeptVo selectByDeptVo(String name);

    DeptVo selectByDeptVoName(String name);

    int updateDeptVoNum(DeptVo deptVo);
}
```

DeptDao.xml

```xml
<select id="selectByDeptVo" resultType="com.mybatis.beans.DeptVo">
  select d.deptno,d.dname,d.loc,dn.num from dept d,deptNum dn where dn.name = d.dname
  and d.dname = #{name}
</select>

<select id="selectByDeptVoName" resultType="com.mybatis.beans.DeptVo">
  select * from deptNum where name = #{name}
</select>

<update id="updateDeptVoNum" parameterType="com.mybatis.beans.DeptVo">
  update deptNum set num = #{num} where name = #{dname}
</update>
```

DeptNum 数据库初始值：

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211534753-692394679.png)

测试类对应如下：

```java
/**
     * 探究多表操作对二级缓存的影响
     */
@Test
public void testOtherMapper(){

  // 第一个mapper 先执行联查操作
  SqlSession sqlSession = factory.openSession();
  DeptDao deptDao = sqlSession.getMapper(DeptDao.class);
  DeptVo deptVo = deptDao.selectByDeptVo("ali");
  System.out.println("deptVo = " + deptVo);
  // 第二个mapper 执行更新操作 并提交
  SqlSession sqlSession2 = factory.openSession();
  DeptDao deptDao2 = sqlSession2.getMapper(DeptDao.class);
  deptDao2.updateDeptVoNum(new DeptVo("ali",1000));
  sqlSession2.commit();
  sqlSession2.close();
  // 第一个mapper 再次进行查询,观察查询结果
  deptVo = deptDao.selectByDeptVo("ali");
  System.out.println("deptVo = " + deptVo);
}
```

测试结果如下：

![img](https://img2018.cnblogs.com/blog/1515111/201908/1515111-20190810211545121-497892884.png)

> 在对DeptNum 表执行了一次更新后，再次进行联查，发现数据库中查询出的还是 num 为 1050 的值，也就是说，实际上 1050 -> 1000 ，最后一次联查实际上查询的是第一次查询结果的缓存，而不是从数据库中查询得到的值，这样就读到了脏数据。

**解决办法**

如果是两个mapper命名空间的话，可以使用 ``来把一个命名空间指向另外一个命名空间，从而消除上述的影响，再次执行，就可以查询到正确的数据

## 二级缓存源码解析

源码模块主要分为两个部分：二级缓存的创建和二级缓存的使用，首先先对二级缓存的创建进行分析：

### 二级缓存的创建

二级缓存的创建是使用 Resource 读取 XML 配置文件开始的

```java
InputStream is = Resources.getResourceAsStream("myBatis-config.xml");
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
factory = builder.build(is);
```

读取配置文件后，需要对XML创建 Configuration并初始化

```java
XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
return build(parser.parse());
```

调用 `parser.parse()` 解析根目录 /configuration 下面的标签，依次进行解析

```java
public Configuration parse() {
  if (parsed) {
    throw new BuilderException("Each XMLConfigBuilder can only be used once.");
  }
  parsed = true;
  parseConfiguration(parser.evalNode("/configuration"));
  return configuration;
}
private void parseConfiguration(XNode root) {
  try {
    //issue #117 read properties first
    propertiesElement(root.evalNode("properties"));
    Properties settings = settingsAsProperties(root.evalNode("settings"));
    loadCustomVfs(settings);
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    settingsElement(settings);
    // read it after objectFactory and objectWrapperFactory issue #631
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```

其中有一个二级缓存的解析就是

```java
mapperElement(root.evalNode("mappers"));
```

然后进去 mapperElement 方法中

```java
XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
```

继续跟 mapperParser.parse() 方法

```java
public void parse() {
  if (!configuration.isResourceLoaded(resource)) {
    configurationElement(parser.evalNode("/mapper"));
    configuration.addLoadedResource(resource);
    bindMapperForNamespace();
  }

  parsePendingResultMaps();
  parsePendingCacheRefs();
  parsePendingStatements();
}
```

这其中有一个 configurationElement 方法，它是对二级缓存进行创建，如下

```java
private void configurationElement(XNode context) {
  try {
    String namespace = context.getStringAttribute("namespace");
    if (namespace == null || namespace.equals("")) {
      throw new BuilderException("Mapper's namespace cannot be empty");
    }
    builderAssistant.setCurrentNamespace(namespace);
    cacheRefElement(context.evalNode("cache-ref"));
    cacheElement(context.evalNode("cache"));
    parameterMapElement(context.evalNodes("/mapper/parameterMap"));
    resultMapElements(context.evalNodes("/mapper/resultMap"));
    sqlElement(context.evalNodes("/mapper/sql"));
    buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing Mapper XML. Cause: " + e, e);
  }
}
```

有两个二级缓存的关键点

```java
cacheRefElement(context.evalNode("cache-ref"));
cacheElement(context.evalNode("cache"));
```

也就是说，mybatis 首先进行解析的是 `cache-ref` 标签，其次进行解析的是 `cache` 标签。

**根据上面我们的 — 多表操作对二级缓存的影响 一节中提到的解决办法，采用 cache-ref 来进行命名空间的依赖能够避免二级缓存**，但是总不能每次写一个 XML 配置都会采用这种方式吧，最有效的方式还是避免多表操作使用二级缓存

然后我们再来看一下cacheElement(context.evalNode("cache")) 这个方法

```java
private void cacheElement(XNode context) throws Exception {
  if (context != null) {
    String type = context.getStringAttribute("type", "PERPETUAL");
    Class<? extends Cache> typeClass = typeAliasRegistry.resolveAlias(type);
    String eviction = context.getStringAttribute("eviction", "LRU");
    Class<? extends Cache> evictionClass = typeAliasRegistry.resolveAlias(eviction);
    Long flushInterval = context.getLongAttribute("flushInterval");
    Integer size = context.getIntAttribute("size");
    boolean readWrite = !context.getBooleanAttribute("readOnly", false);
    boolean blocking = context.getBooleanAttribute("blocking", false);
    Properties props = context.getChildrenAsProperties();
    builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
  }
}
```

认真看一下其中的属性的解析，是不是感觉很熟悉？这不就是对 cache 标签属性的解析吗？！！！

上述最后一句代码

```java
builderAssistant.useNewCache(typeClass, evictionClass, flushInterval, size, readWrite, blocking, props);
public Cache useNewCache(Class<? extends Cache> typeClass,
      Class<? extends Cache> evictionClass,
      Long flushInterval,
      Integer size,
      boolean readWrite,
      boolean blocking,
      Properties props) {
    Cache cache = new CacheBuilder(currentNamespace)
        .implementation(valueOrDefault(typeClass, PerpetualCache.class))
        .addDecorator(valueOrDefault(evictionClass, LruCache.class))
        .clearInterval(flushInterval)
        .size(size)
        .readWrite(readWrite)
        .blocking(blocking)
        .properties(props)
        .build();
    configuration.addCache(cache);
    currentCache = cache;
    return cache;
  }
```

这段代码使用了构建器模式，一步一步构建Cache 标签的所有属性，最终把 cache 返回。

### 二级缓存的使用

在 mybatis 中，使用 Cache 的地方在 `CachingExecutor`中，来看一下 CachingExecutor 中缓存做了什么工作，我们以查询为例

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
  throws SQLException {
  // 得到缓存
  Cache cache = ms.getCache();
  if (cache != null) {
    // 如果需要的话刷新缓存
    flushCacheIfRequired(ms);
    if (ms.isUseCache() && resultHandler == null) {
      ensureNoOutParams(ms, parameterObject, boundSql);
      @SuppressWarnings("unchecked")
      List<E> list = (List<E>) tcm.getObject(cache, key);
      if (list == null) {
        list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
        tcm.putObject(cache, key, list); // issue #578 and #116
      }
      return list;
    }
  }
  // 委托模式，交给SimpleExecutor等实现类去实现方法。
  return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
```

其中，先从 MapperStatement 取出缓存。只有通过`,`或`@CacheNamespace,@CacheNamespaceRef`标记使用缓存的Mapper.xml或Mapper接口（同一个namespace，不能同时使用）才会有二级缓存。

如果缓存不为空，说明是存在缓存。如果cache存在，那么会根据sql配置(`,,,`的`flushCache`属性来确定是否清空缓存。

```java
flushCacheIfRequired(ms);
```

然后根据xml配置的属性`useCache`来判断是否使用缓存(resultHandler一般使用的默认值，很少会null)。

```java
if (ms.isUseCache() && resultHandler == null)
```

确保方法没有Out类型的参数，mybatis不支持存储过程的缓存，所以如果是存储过程，这里就会报错。

```java
private void ensureNoOutParams(MappedStatement ms, Object parameter, BoundSql boundSql) {
  if (ms.getStatementType() == StatementType.CALLABLE) {
    for (ParameterMapping parameterMapping : boundSql.getParameterMappings()) {
      if (parameterMapping.getMode() != ParameterMode.IN) {
        throw new ExecutorException("Caching stored procedures with OUT params is not supported.  Please configure useCache=false in " + ms.getId() + " statement.");
      }
    }
  }
}
```

然后根据在 `TransactionalCacheManager` 中根据 key 取出缓存，如果没有缓存，就会执行查询，并且将查询结果放到缓存中并返回取出结果，否则就执行真正的查询方法。

```java
List<E> list = (List<E>) tcm.getObject(cache, key);
if (list == null) {
  list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  tcm.putObject(cache, key, list); // issue #578 and #116
}
return list;
```

## 是否应该使用二级缓存？

那么究竟应该不应该使用二级缓存呢？先来看一下二级缓存的注意事项：

1. 缓存是以`namespace`为单位的，不同`namespace`下的操作互不影响。
2. insert,update,delete操作会清空所在`namespace`下的全部缓存。
3. 通常使用MyBatis Generator生成的代码中，都是各个表独立的，每个表都有自己的`namespace`。
4. 多表操作一定不要使用二级缓存，因为多表操作进行更新操作，一定会产生脏数据。

如果你遵守二级缓存的注意事项，那么你就可以使用二级缓存。

但是，如果不能使用多表操作，二级缓存不就可以用一级缓存来替换掉吗？而且二级缓存是表级缓存，开销大，没有一级缓存直接使用 HashMap 来存储的效率更高，所以**二级缓存并不推荐使用**。

文章参考：

[聊聊MyBatis缓存机制](https://tech.meituan.com/2018/01/19/mybatis-cache.html)

[mybatis一级缓存二级缓存](https://www.cnblogs.com/happyflyingpig/p/7739749.html)

深入了解MyBatis二级缓存 https://blog.csdn.net/isea533/article/details/44566257

