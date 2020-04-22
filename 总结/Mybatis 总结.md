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

　　　　测试：

```
//向 user 表中插入一条数据并获取主键值``  ``@Test``  ``public` `void` `testInsertUser(){``    ``String statement = ``"com.ys.po.userMapper.insertUser"``;``    ``User user = ``new` `User();``    ``user.setUsername(``"Bob"``);``    ``user.setSex(``"女"``);``    ``session.insert(statement, user);``    ``//提交插入的数据``    ``session.commit();``    ``//打印主键值``    ``System.out.println(user.getId());``    ``session.close();``  ``}
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
    </select>
```

　　prefix：前缀　　　　　　

　　prefixoverride：去掉第一个and或者是or

 

　　②、用 trim 改写上面第三点的 if+set 语句

```xml
<!-- 根据 id 更新 user 表的数据 -->
    <update id="updateUserById" parameterType="com.ys.po.User">
        update user u
            <!-- <set>
                <if test="username != null and username != ''">
                    u.username = #{username},
                </if>
                <if test="sex != null and sex != ''">
                    u.sex = #{sex}
                </if>
            </set> -->
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
```

　　suffix：后缀　　

　　suffixoverride：去掉最后一个逗号（也可以是其他的标记，就像是上面前缀中的and一样）

 

 

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 6、动态SQL: SQL 片段

　　有时候可能某个 sql 语句我们用的特别多，为了增加代码的重用性，简化代码，我们需要将这些代码抽取出来，然后使用时直接调用。

　　比如：假如我们需要经常根据用户名和性别来进行联合查询，那么我们就把这个代码抽取出来，如下：

```xml
<!-- 定义 sql 片段 -->
<sql id="selectUserByUserNameAndSexSQL">
    <if test="username != null and username != ''">
        AND username = #{username}
    </if>
    <if test="sex != null and sex != ''">
        AND sex = #{sex}
    </if>
</sql>
```

　　引用 sql 片段

```xml
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <trim prefix="where" prefixOverrides="and | or">
        <!-- 引用 sql 片段，如果refid 指定的不在本文件中，那么需要在前面加上 namespace -->
        <include refid="selectUserByUserNameAndSexSQL"></include>
        <!-- 在这里还可以引用其他的 sql 片段 -->
    </trim>
</select>
```

　　注意：①、最好基于 单表来定义 sql 片段，提高片段的可重用性

　　　　　②、在 sql 片段中最好不要包括 where 

　　　　

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 7、动态SQL: foreach 语句

　　需求：我们需要查询 user 表中 id 分别为1,2,3的用户

　　sql语句：select * from user where id=1 or id=2 or id=3

　　　　　　 select * from user where id in (1,2,3)

 

①、建立一个 UserVo 类，里面封装一个 List<Integer> ids 的属性

```
package com.ys.vo;
 
import java.util.List;
 
public class UserVo {
    //封装多个用户的id
    private List<Integer> ids;
 
    public List<Integer> getIds() {
        return ids;
    }
 
    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }
 
}　　
```

 

②、我们用 foreach 来改写 select * from user where id=1 or id=2 or id=3

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
            select * from user where 1=1 and (id=1 or id=2 or id=3)
          -->
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

``"selectUserByUserId"` `useCache=``"false"` `resultType=``"com.ys.twocache.User"` `parameterType=``"int"``>``  ``select * from user where id=#{id}```

　　这种情况是针对每次查询都需要最新的数据sql，要设置成useCache=false，禁用二级缓存，直接从数据库中获取。 

　　在mapper的同一个namespace中，如果有其它insert、update、delete操作数据后需要刷新缓存，如果不执行刷新缓存会出现脏读。

   设置statement配置中的flushCache=”true” 属性，默认情况下为true，即刷新缓存，如果改成false则不会刷新。使用缓存时如果手动修改数据库表中的查询数据会出现脏读。

``"selectUserByUserId"` `flushCache=``"true"` `useCache=``"false"` `resultType=``"com.ys.twocache.User"` `parameterType=``"int"``>``  ``select * from user where id=#{id}```

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
