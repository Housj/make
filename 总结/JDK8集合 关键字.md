# [Java的深拷贝和浅拷贝](https://www.cnblogs.com/ysocean/p/8482979.html)



**目录**

- [1、创建对象的5种方式](https://www.cnblogs.com/ysocean/p/8482979.html#_label0)
- [3、Clone 方法](https://www.cnblogs.com/ysocean/p/8482979.html#_label1)
- [4、基本类型和引用类型](https://www.cnblogs.com/ysocean/p/8482979.html#_label2)
- [5、浅拷贝](https://www.cnblogs.com/ysocean/p/8482979.html#_label3)
- [6、深拷贝](https://www.cnblogs.com/ysocean/p/8482979.html#_label4)
- 7、如何实现深拷贝？
  - [　　①、让每个引用类型属性内部都重写clone() 方法](https://www.cnblogs.com/ysocean/p/8482979.html#_label5_0)
  - [　　②、利用序列化](https://www.cnblogs.com/ysocean/p/8482979.html#_label5_1)

 

------

　　关于Java的深拷贝和浅拷贝，简单来说就是创建一个和已知对象一模一样的对象。可能日常编码过程中用的不多，但是这是一个面试经常会问的问题，而且了解深拷贝和浅拷贝的原理，对于Java中的所谓值传递或者引用传递将会有更深的理解。

[回到顶部](https://www.cnblogs.com/ysocean/p/8482979.html#_labelTop)

### 1、创建对象的5种方式

　　**①、通过 new 关键字**

　　这是最常用的一种方式，通过 new 关键字调用类的有参或无参构造方法来创建对象。比如 Object obj = new Object();

　　**②、通过 Class 类的 newInstance() 方法**

　　这种默认是调用类的无参构造方法创建对象。比如 Person p2 = (Person) Class.forName("com.ys.test.Person").newInstance();

　　**③、通过 Constructor 类的 newInstance 方法**

　　这和第二种方法类时，都是通过反射来实现。通过 java.lang.relect.Constructor 类的 newInstance() 方法指定某个构造器来创建对象。

　　Person p3 = (Person) Person.class.getConstructors()[0].newInstance();

　　实际上第二种方法利用 Class 的 newInstance() 方法创建对象，其内部调用还是 Constructor 的 newInstance() 方法。

　　**④、利用 Clone 方法**

　　Clone 是 Object 类中的一个方法，通过 对象A.clone() 方法会创建一个内容和对象 A 一模一样的对象 B，clone 克隆，顾名思义就是创建一个一模一样的对象出来。

　　Person p4 = (Person) p3.clone();

　　**⑤、反序列化**

　　序列化是把堆内存中的 Java 对象数据，通过某种方式把对象存储到磁盘文件中或者传递给其他网络节点（在网络上传输）。而反序列化则是把磁盘文件中的对象数据或者把网络节点上的对象数据，恢复成Java对象模型的过程。

　　具体如何实现可以参考我 [这篇博文](http://www.cnblogs.com/ysocean/p/6870069.html)。

[回到顶部](https://www.cnblogs.com/ysocean/p/8482979.html#_labelTop)

### 3、Clone 方法

　　本篇博客我们讲解的是 Java 的深拷贝和浅拷贝，其实现方式正是通过调用 Object 类的 clone() 方法来完成。在 Object.class 类中，源码为：

```
protected native Object clone() throws CloneNotSupportedException;
```

　　这是一个用 native 关键字修饰的方法，关于native关键字有一篇[博客](https://www.cnblogs.com/ysocean/p/8476933.html)专门有介绍，不理解也没关系，只需要知道用 native 修饰的方法就是告诉操作系统，这个方法我不实现了，让操作系统去实现。具体怎么实现我们不需要了解，只需要知道 clone方法的作用就是复制对象，产生一个新的对象。那么这个新的对象和原对象是什么关系呢？

[回到顶部](https://www.cnblogs.com/ysocean/p/8482979.html#_labelTop)

### 4、基本类型和引用类型

　　这里再给大家普及一个概念，在 Java 中基本类型和引用类型的区别。

　　在 Java 中数据类型可以分为两大类：基本类型和引用类型。

　　基本类型也称为值类型，分别是字符类型 char，布尔类型 boolean以及数值类型 byte、short、int、long、float、double。

　　引用类型则包括类、接口、数组、枚举等。

　　Java 将内存空间分为堆和栈。基本类型直接在栈中存储数值，而引用类型是将引用放在栈中，实际存储的值是放在堆中，通过栈中的引用指向堆中存放的数据。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180312213707709-1454308742.png)

　　上图定义的 a 和 b 都是基本类型，其值是直接存放在栈中的；而 c 和 d 是 String 声明的，这是一个引用类型，引用地址是存放在 栈中，然后指向堆的内存空间。

　　下面 d = c；这条语句表示将 c 的引用赋值给 d，那么 c 和 d 将指向同一块堆内存空间。

[回到顶部](https://www.cnblogs.com/ysocean/p/8482979.html#_labelTop)

### 5、浅拷贝

　　我们看如下这段代码：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
package com.ys.test;

public class Person implements Cloneable{
    public String pname;
    public int page;
    public Address address;
    public Person() {}
    
    public Person(String pname,int page){
        this.pname = pname;
        this.page = page;
        this.address = new Address();
    }
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
    
    public void setAddress(String provices,String city ){
        address.setAddress(provices, city);
    }
    public void display(String name){
        System.out.println(name+":"+"pname=" + pname + ", page=" + page +","+ address);
    }

    public String getPname() {
        return pname;
    }

    public void setPname(String pname) {
        this.pname = pname;
    }

    public int getPage() {
        return page;
    }

    public void setPage(int page) {
        this.page = page;
    }
    
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
package com.ys.test;

public class Address {
    private String provices;
    private String city;
    public void setAddress(String provices,String city){
        this.provices = provices;
        this.city = city;
    }
    @Override
    public String toString() {
        return "Address [provices=" + provices + ", city=" + city + "]";
    }
    
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这是一个我们要进行赋值的原始类 Person。下面我们产生一个 Person 对象，并调用其 clone 方法复制一个新的对象。

　　**注意：调用对象的 clone 方法，必须要让类实现 Cloneable 接口，并且覆写 clone 方法。**

　　测试：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
@Test
public void testShallowClone() throws Exception{
    Person p1 = new Person("zhangsan",21);
    p1.setAddress("湖北省", "武汉市");
    Person p2 = (Person) p1.clone();
    System.out.println("p1:"+p1);
    System.out.println("p1.getPname:"+p1.getPname().hashCode());
    
    System.out.println("p2:"+p2);
    System.out.println("p2.getPname:"+p2.getPname().hashCode());
    
    p1.display("p1");
    p2.display("p2");
    p2.setAddress("湖北省", "荆州市");
    System.out.println("将复制之后的对象地址修改：");
    p1.display("p1");
    p2.display("p2");
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　打印结果为：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180312214947767-1020527476.png)

　　首先看原始类 Person 实现 Cloneable 接口，并且覆写 clone 方法,它还有三个属性，一个引用类型 String定义的 pname，一个基本类型 int定义的 page，还有一个引用类型 Address ，这是一个自定义类，这个类也包含两个属性 pprovices 和 city 。

　　接着看测试内容，首先我们创建一个Person 类的对象 p1，其pname 为zhangsan,page为21，地址类 Address 两个属性为 湖北省和武汉市。接着我们调用 clone() 方法复制另一个对象 p2，接着打印这两个对象的内容。

　　从第 1 行和第 3 行打印结果:

　　p1:com.ys.test.Person@349319f9

　　p2:com.ys.test.Person@258e4566

　　可以看出这是两个不同的对象。

　　从第 5 行和第 6 行打印的对象内容看，原对象 p1 和克隆出来的对象 p2 内容完全相同。

　　代码中我们只是更改了克隆对象 p2 的属性 Address 为湖北省荆州市（原对象 p1 是湖北省武汉市） ，但是从第 7 行和第 8 行打印结果来看，原对象 p1 和克隆对象 p2 的 Address 属性都被修改了。

　　也就是说对象 Person 的属性 Address，经过 clone 之后，其实只是复制了其引用，他们指向的还是同一块堆内存空间，当修改其中一个对象的属性 Address，另一个也会跟着变化。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180312220935185-85540133.png)

 

　　**浅拷贝：创建一个新对象，然后将当前对象的非静态字段复制到该新对象，如果字段是值类型的，那么对该字段执行复制；如果该字段是引用类型的话，则复制引用但不复制引用的对象。因此，原始对象及其副本引用同一个对象。**

[回到顶部](https://www.cnblogs.com/ysocean/p/8482979.html#_labelTop)

### 6、深拷贝

　　弄清楚了浅拷贝，那么深拷贝就很容易理解了。

　　**深拷贝：创建一个新对象，然后将当前对象的非静态字段复制到该新对象，无论该字段是值类型的还是引用类型，都复制独立的一份。当你修改其中一个对象的任何内容时，都不会影响另一个对象的内容。**

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180313205221201-187008977.png)

　　那么该如何实现深拷贝呢？**Object 类提供的 clone 是只能实现 浅拷贝的。**

[回到顶部](https://www.cnblogs.com/ysocean/p/8482979.html#_labelTop)

### 7、如何实现深拷贝？

　　深拷贝的原理我们知道了，就是要让原始对象和克隆之后的对象所具有的引用类型属性不是指向同一块堆内存，这里有三种实现思路。



#### 　　①、让每个引用类型属性内部都重写clone() 方法

　　既然引用类型不能实现深拷贝，那么我们将每个引用类型都拆分为基本类型，分别进行浅拷贝。比如上面的例子，Person 类有一个引用类型 Address(其实String 也是引用类型，但是String类型有点特殊，后面会详细讲解)，我们在 Address 类内部也重写 clone 方法。如下：

　　Address.class:

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.test;
 2 
 3 public class Address implements Cloneable{
 4     private String provices;
 5     private String city;
 6     public void setAddress(String provices,String city){
 7         this.provices = provices;
 8         this.city = city;
 9     }
10     @Override
11     public String toString() {
12         return "Address [provices=" + provices + ", city=" + city + "]";
13     }
14     @Override
15     protected Object clone() throws CloneNotSupportedException {
16         return super.clone();
17     }
18     
19 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　Person.class 的 clone() 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     @Override
2     protected Object clone() throws CloneNotSupportedException {
3         Person p = (Person) super.clone();
4         p.address = (Address) address.clone();
5         return p;
6     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　测试还是和上面一样，我们会发现更改了p2对象的Address属性，p1 对象的 Address 属性并没有变化。

　　但是这种做法有个弊端，这里我们Person 类只有一个 Address 引用类型，而 Address 类没有，所以我们只用重写 Address 类的clone 方法，但是如果 Address 类也存在一个引用类型，那么我们也要重写其clone 方法，这样下去，有多少个引用类型，我们就要重写多少次，如果存在很多引用类型，那么代码量显然会很大，所以这种方法不太合适。



#### 　　②、利用序列化

　　序列化是将对象写到流中便于传输，而反序列化则是把对象从流中读取出来。这里写到流中的对象则是原始对象的一个拷贝，因为原始对象还存在 JVM 中，所以我们可以利用对象的序列化产生克隆对象，然后通过反序列化获取这个对象。

　　注意每个需要序列化的类都要实现 Serializable 接口，如果有某个属性不需要序列化，可以将其声明为 transient，即将其排除在克隆属性之外。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
//深度拷贝
public Object deepClone() throws Exception{
    // 序列化
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(bos);

    oos.writeObject(this);

    // 反序列化
    ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
    ObjectInputStream ois = new ObjectInputStream(bis);

    return ois.readObject();
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 　因为序列化产生的是两个完全独立的对象，所有无论嵌套多少个引用类型，序列化都是能实现深拷贝的。



# [java.lang.Object类](https://www.cnblogs.com/ysocean/p/8419559.html)

https://www.cnblogs.com/ysocean/category/1149707.html

**目录**

- [1、Object 类的结构图](https://www.cnblogs.com/ysocean/p/8419559.html#_label0)
- [2、 为什么java.lang包下的类不需要手动导入？](https://www.cnblogs.com/ysocean/p/8419559.html#_label1)
- [3、类构造器](https://www.cnblogs.com/ysocean/p/8419559.html#_label2)
- [4、equals 方法](https://www.cnblogs.com/ysocean/p/8419559.html#_label3)
- [5、getClass 方法 ](https://www.cnblogs.com/ysocean/p/8419559.html#_label4)
- 6、hashCode 方法
  - [　　一、hashCode 要求](https://www.cnblogs.com/ysocean/p/8419559.html#_label5_0)
  - [　　二、hashCode 编写指导：](https://www.cnblogs.com/ysocean/p/8419559.html#_label5_1)
- [7、toString 方法](https://www.cnblogs.com/ysocean/p/8419559.html#_label6)
- [8、notify()/notifyAll()/wait()](https://www.cnblogs.com/ysocean/p/8419559.html#_label7)
- [9、finalize 方法](https://www.cnblogs.com/ysocean/p/8419559.html#_label8)
- [10、registerNatives 方法 ](https://www.cnblogs.com/ysocean/p/8419559.html#_label9)

 

------

　　本系列博客将对JDK1.8版本的相关类从源码层次进行介绍，JDK8的[下载地址](http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-windows-x64.exe?AuthParam=1519472606_b1014bbce320a78aa9638b63275dfb16)。

　　首先介绍JDK中所有类的基类——java.lang.Object。

　　Object 类属于 java.lang 包，此包下的所有类在使用时无需手动导入，系统会在程序编译期间自动导入。Object 类是所有类的基类，当一个类没有直接继承某个类时，默认继承Object类，也就是说任何类都直接或间接继承此类，Object 类中能访问的方法在所有类中都可以调用，下面我们会分别介绍Object 类中的所有方法。

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 1、Object 类的结构图

 　![img](https://images2018.cnblogs.com/blog/1120165/201802/1120165-20180226212832117-1362368877.png)

　　Object.class类

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 /*
 2  * Copyright (c) 1994, 2012, Oracle and/or its affiliates. All rights reserved.
 3  * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
 4  *
 5  */
 6 
 7 package java.lang;
 8 
 9 /**
10  * Class {@code Object} is the root of the class hierarchy.
11  * Every class has {@code Object} as a superclass. All objects,
12  * including arrays, implement the methods of this class.
13  *
14  * @author  unascribed
15  * @see     java.lang.Class
16  * @since   JDK1.0
17  */
18 public class Object {
19 
20     private static native void registerNatives();
21     static {
22         registerNatives();
23     }
24 
25     public final native Class<?> getClass();
26 
27     public native int hashCode();
28 
29     public boolean equals(Object obj) {
30         return (this == obj);
31     }
32 
33     protected native Object clone() throws CloneNotSupportedException;
34     
35     public String toString() {
36         return getClass().getName() + "@" + Integer.toHexString(hashCode());
37     }
38 
39     public final native void notify();
40     
41     public final native void notifyAll();
42     
43     public final native void wait(long timeout) throws InterruptedException;
44     
45     public final void wait(long timeout, int nanos) throws InterruptedException {
46         if (timeout < 0) {
47             throw new IllegalArgumentException("timeout value is negative");
48         }
49         if (nanos < 0 || nanos > 999999) {
50             throw new IllegalArgumentException(
51                     "nanosecond timeout value out of range");
52         }
53         if (nanos > 0) {
54             timeout++;
55         }
56         wait(timeout);
57     }
58     
59     public final void wait() throws InterruptedException {
60         wait(0);
61     }
62     
63     protected void finalize() throws Throwable { }
64 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 2、 为什么java.lang包下的类不需要手动导入？

　　不知道大家注意到没，我们在使用诸如Date类时，需要手动导入import java.util.Date，再比如使用File类时，也需要手动导入import java.io.File。但是我们在使用Object类，String 类，Integer类等不需要手动导入，而能直接使用，这是为什么呢？

　　这里先告诉大家一个结论：**使用 java.lang 包下的所有类，都不需要手动导入。**

　　另外我们介绍一下Java中的两种导包形式，导包有两种方法：

　　①、单类型导入（single-type-import），例如import java.util.Date

　　②、按需类型导入(type-import-on-demand)，例如import java.util.*

　　单类型导入比较好理解，我们编程所使用的各种工具默认都是按照单类型导包的，需要什么类便导入什么类，这种方式是导入指定的public类或者接口；

　　按需类型导入，比如 import java.util.*，可能看到后面的 *，大家会以为是导入java.util包下的所有类，其实并不是这样，我们根据名字按需导入要知道他是按照需求导入，并不是导入整个包下的所有类。

　　Java编译器会从启动目录(bootstrap)，扩展目录(extension)和用户类路径下去定位需要导入的类，而这些目录进仅仅是给出了类的顶层目录，编译器的类文件定位方法大致可以理解为如下公式：　

```
顶层路径名 \ 包名 \ 文件名.``class` `= 绝对路径
```

　　单类型导入我们知道包名和文件名，所以编译器可以一次性查找定位到所要的类文件。按需类型导入则比较复杂，编译器会把包名和文件名进行排列组合，然后对所有的可能性进行类文件查找定位。例如：　　

```
package` `com;` `import` `java.io.*;` `import` `java.util.*;
```

　　如果我们文件中使用到了 File 类，那么编译器会根据如下几个步骤来进行查找 File 类：

　　①、File 　　　　　　// File类属于无名包，就是说File类没有package语句，编译器会首先搜索无名包

　　②、com.File 　　　　// File类属于当前包，就是我们当前编译类的包路径

　　③、java.lang.File 　　//由于编译器会自动导入java.lang包，所以也会从该包下查找

　　④、java.io.File

　　⑤、java.util.File

　　**......**

　　需要注意的地方就是，编译器找到java.io.File类之后并不会停止下一步的寻找，而要把所有的可能性都查找完以确定是否有类导入冲突。假设此时的顶层路径有三个，那么编译器就会进行3*5=15次查找。

　　如果在查找完成后，编译器发现了两个同名的类，那么就会报错。要删除你不用的那个类，然后再编译。

　　所以我们可以得出这样的结论：**按需类型导入是绝对不会降低Java代码的执行效率的，但会影响到Java代码的编译速度。**所以我们在编码时最好是使用单类型导入，这样不仅能提高编译速度，也能避免命名冲突。

　　讲清楚Java的两种导包类型了，我们在回到为什么可以直接使用 Object 类，看到上面查找类文件的第③步，编译器会自动导入 java.lang 包，那么当然我们能直接使用了。至于原因，因为用的多，提前加载了，省资源。

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 3、类构造器

　　我们知道类构造器是创建Java对象的途径之一，通过new 关键字调用构造器完成对象的实例化，还能通过构造器对对象进行相应的初始化。一个类必须要有一个构造器的存在，如果没有显示声明，那么系统会默认创造一个无参构造器，在JDK的Object类源码中，是看不到构造器的，系统会自动添加一个无参构造器。我们可以通过：

　　 Object obj = new Object()；构造一个Object类的对象。

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 4、equals 方法

　　通常很多面试题都会问 equals() 方法和 == 运算符的区别，== 运算符用于比较基本类型的值是否相同，或者比较两个对象的引用是否相等，而 equals 用于比较两个对象是否相等，这样说可能比较宽泛，两个对象如何才是相等的呢？这个标尺该如何定？

　　我们可以看看 Object 类中的equals 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

　　可以看到，在 Object 类中，== 运算符和 equals 方法是等价的，都是比较两个对象的引用是否相等，从另一方面来讲，如果两个对象的引用相等，那么这两个对象一定是相等的。对于我们自定义的一个对象，如果不重写 equals 方法，那么在比较对象的时候就是调用 Object 类的 equals 方法，也就是用 == 运算符比较两个对象。我们可以看看 String 类中的重写的 equals 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　String 是引用类型，比较时不能比较引用是否相等，重点是字符串的内容是否相等。所以 String 类定义两个对象相等的标准是**字符串内容都相同**。

　　在Java规范中，对 equals 方法的使用必须遵循以下几个原则：

　　①、自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。

　　②、**对称性**：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。 

　　③、传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。

　　④、一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改

　　⑤、对于任何非空引用值 x，x.equals(null) 都应返回 false。

　　下面我们自定义一个 Person 类，然后重写其equals 方法，比较两个 Person 对象：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.bean;
 2 /**
 3  * Create by vae
 4  */
 5 public class Person {
 6     private String pname;
 7     private int page;
 8 
 9     public Person(){}
10 
11     public Person(String pname,int page){
12         this.pname = pname;
13         this.page = page;
14     }
15     public int getPage() {
16         return page;
17     }
18 
19     public void setPage(int page) {
20         this.page = page;
21     }
22 
23     public String getPname() {
24         return pname;
25     }
26 
27     public void setPname(String pname) {
28         this.pname = pname;
29     }
30     @Override
31     public boolean equals(Object obj) {
32         if(this == obj){//引用相等那么两个对象当然相等
33             return true;
34         }
35         if(obj == null || !(obj instanceof  Person)){//对象为空或者不是Person类的实例
36             return false;
37         }
38         Person otherPerson = (Person)obj;
39         if(otherPerson.getPname().equals(this.getPname()) && otherPerson.getPage()==this.getPage()){
40             return true;
41         }
42         return false;
43     }
44 
45     public static void main(String[] args) {
46         Person p1 = new Person("Tom",21);
47         Person p2 = new Person("Marry",20);
48         System.out.println(p1==p2);//false
49         System.out.println(p1.equals(p2));//false
50 
51         Person p3 = new Person("Tom",21);
52         System.out.println(p1.equals(p3));//true
53     }
54 
55 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　通过重写 equals 方法，我们自定义两个对象相等的标尺为Person**对象的两个属性都相等，则对象相等，否则不相等**。如果不重写 equals 方法，那么始终是调用 Object 类的equals 方法，也就是用 == 比较两个对象在栈内存中的引用地址是否相等。

 　这时候有个Person 类的子类 Man，也重写了 equals 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.bean;
 2 /**
 3  * Create by vae
 4  */
 5 public class Man extends Person{
 6     private String sex;
 7 
 8     public Man(String pname,int page,String sex){
 9         super(pname,page);
10         this.sex = sex;
11     }
12     @Override
13     public boolean equals(Object obj) {
14         if(!super.equals(obj)){
15             return false;
16         }
17         if(obj == null || !(obj instanceof  Man)){//对象为空或者不是Person类的实例
18             return false;
19         }
20         Man man = (Man) obj;
21         return sex.equals(man.sex);
22     }
23 
24     public static void main(String[] args) {
25         Person p = new Person("Tom",22);
26         Man m = new Man("Tom",22,"男");
27 
28         System.out.println(p.equals(m));//true
29         System.out.println(m.equals(p));//false
30     }
31 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　通过打印结果我们发现 person.equals(man)得到的结果是 true，而man.equals(person)得到的结果却是false,这显然是不正确的。

　　问题出现在 instanceof 关键字上，关于 instanceof 关键字的用法，可以参考我的这篇文章：http://www.cnblogs.com/ysocean/p/8486500.html

　　Man 是 Person 的子类，person instanceof Man 结果当然是false。这违反了我们上面说的**对称性。**

　　**实际上用 instanceof 关键字是做不到对称性的要求的。这里推荐做法是用 getClass()方法取代 instanceof 运算符。getClass() 关键字也是 Object 类中的一个方法，作用是返回一个对象的运行时类，下面我们会详细讲解。**

　　那么 Person 类中的 equals 方法为：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    public boolean equals(Object obj) {
        if(this == obj){//引用相等那么两个对象当然相等
            return true;
        }
        if(obj == null || (getClass() != obj.getClass())){//对象为空或者不是Person类的实例
            return false;
        }
        Person otherPerson = (Person)obj;
        if(otherPerson.getPname().equals(this.getPname()) && otherPerson.getPage()==this.getPage()){
            return true;
        }
        return false;
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　打印结果 person.equals(man)得到的结果是 false，man.equals(person)得到的结果也是false，满足对称性。

　　注意：使用 getClass 不是绝对的，要根据情况而定，毕竟定义对象是否相等的标准是由程序员自己定义的。而且使用 getClass 不符合多态的定义，比如 AbstractSet 抽象类，它有两个子类 TreeSet 和 HashSet,他们分别使用不同的算法实现查找集合的操作，但无论集合采用哪种方式实现，都需要拥有对两个集合进行比较的功能，如果使用 getClass 实现equals方法的重写，那么就不能在两个不同子类的对象进行相等的比较。而且集合类比较特殊，其子类是不需要自定义相等的概念的。

　　所以什么时候使用 instanceof 运算符，什么时候使用 getClass() 有如下建议：

　　**①、如果子类能够拥有自己的相等概念，则对称性需求将强制采用 getClass 进行检测。**

　　**②、如果有超类决定相等的概念，那么就可以使用 instanceof 进行检测，这样可以在不同的子类的对象之间进行相等的比较。**

　　下面给出一个完美的 equals 方法的建议：

　　1、显示参数命名为 otherObject，稍后会将它转换成另一个叫做 other 的变量。

　　2、判断比较的两个对象引用是否相等，如果引用相等那么表示是同一个对象，那么当然相等

　　3、如果 otherObject 为 null，直接返回false，表示不相等

　　4、比较 this 和 otherObject 是否是同一个类：如果 equals 的语义在每个子类中有所改变，就使用 getClass 检测；如果所有的子类都有统一的定义，那么使用 instanceof 检测

　　5、将 otherObject 转换成对应的类类型变量

　　6、最后对对象的属性进行比较。使用 == 比较基本类型，使用 equals 比较对象。如果都相等则返回true，否则返回false。注意如果是在子类中定义equals，则要包含 super.equals(other)

　　下面我们给出 Person 类中完整的 equals 方法的书写：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     @Override
 2     public boolean equals(Object otherObject) {
 3         //1、判断比较的两个对象引用是否相等，如果引用相等那么表示是同一个对象，那么当然相等
 4         if(this == otherObject){
 5             return true;
 6         }
 7         //2、如果 otherObject 为 null，直接返回false，表示不相等
 8         if(otherObject == null ){//对象为空或者不是Person类的实例
 9             return false;
10         }
11         //3、比较 this 和 otherObject 是否是同一个类（注意下面两个只能使用一种）
12         //3.1：如果 equals 的语义在每个子类中所有改变，就使用 getClass 检测
13         if(this.getClass() != otherObject.getClass()){
14             return false;
15         }
16         //3.2：如果所有的子类都有统一的定义，那么使用 instanceof 检测
17         if(!(otherObject instanceof Person)){
18             return false;
19         }
20 
21         //4、将 otherObject 转换成对应的类类型变量
22         Person other = (Person) otherObject;
23 
24         //5、最后对对象的属性进行比较。使用 == 比较基本类型，使用 equals 比较对象。如果都相等则返回true，否则返回false
25         //   使用 Objects 工具类的 equals 方法防止比较的两个对象有一个为 null而报错，因为 null.equals() 是会抛异常的
26         return Objects.equals(this.pname,other.pname) && this.page == other.page;
27 
28         //6、注意如果是在子类中定义equals，则要包含 super.equals(other)
29         //return super.equals(other) && Objects.equals(this.pname,other.pname) && this.page == other.page;
30 
31     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**请注意，无论何时重写此方法，通常都必须重写hashCode方法，以维护hashCode方法的一般约定，该方法声明相等对象必须具有相同的哈希代码。hashCode 也是 Object 类中的方法，后面会详细讲解。**

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 5、getClass 方法 

　　上面我们在介绍 equals 方法时，介绍如果 equals 的语义在每个子类中有所改变，那么使用 getClass 检测，为什么这样说呢？

　　getClass()在 Object 类中如下，作用是返回对象的运行时类。

```
public` `final` `native` `Class getClass();
```

　　这是一个用 native 关键字修饰的方法，关于 native 关键字的详细介绍如下：http://www.cnblogs.com/ysocean/p/8476933.html

　　这里我们要知道用 native 修饰的方法我们不用考虑，由操作系统帮我们实现，该方法的作用是返回一个对象的运行时类，通过这个类对象我们可以获取该运行时类的相关属性和方法。也就是Java中的反射，各种通用的框架都是利用反射来实现的，这里我们不做详细的描述。

　　这里详细的介绍 getClass 方法返回的是一个对象的运行时类对象，这该怎么理解呢？Java中还有一种这样的用法，通过 类名.class 获取这个类的类对象 ，这两种用法有什么区别呢？

　　父类：Parent.class

```
public class Parent {}
```

　　子类：Son.class

```
public class Son extends Parent{}
```

　　测试：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
@Test
public void testClass(){
    Parent p = new Son();
    System.out.println(p.getClass());
    System.out.println(Parent.class);
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　打印结果：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180313103040339-2057656884.png)

　　**结论：class 是一个类的属性，能获取该类编译时的类对象，而 getClass() 是一个类的方法，它是获取该类运行时的类对象。**

　　还有一个需要大家注意的是，虽然Object类中getClass() 方法声明是：public final native Class<?> getClass();返回的是一个 Class<?>，但是如下是能通过编译的：

```
Class<? extends String> c = "".getClass();
```

　　也就是说类型为T的变量getClass方法的返回值类型其实是Class<? extends T>而非getClass方法声明中的Class<?>。

　　这在官方文档中也有说明：https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#getClass--

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 6、hashCode 方法

　　hashCode 在 Object 类中定义如下：

```
public native int hashCode();
```

　　这也是一个用 native 声明的本地方法，作用是返回对象的散列码，是 int 类型的数值。

　　那么这个方法存在的意义是什么呢?

　　我们知道在Java 中有几种集合类，比如 List,Set，还有 Map，List集合一般是存放的元素是有序可重复的，Set 存放的元素则是无序不可重复的，而 Map 集合存放的是键值对。

　　前面我们说过判断一个元素是否相等可以通过 equals 方法，没增加一个元素，那么我们就通过 equals 方法判断集合中的每一个元素是否重复，但是如果集合中有10000个元素了，但我们新加入一个元素时，那就需要进行10000次equals方法的调用，这显然效率很低。

　　于是，Java 的集合设计者就采用了 **哈希表** 来实现。关于[哈希表的数据结构](http://www.cnblogs.com/ysocean/p/8032656.html)我有过介绍。哈希算法也称为散列算法，是将数据依特定算法产生的结果直接指定到一个地址上。这个结果就是由 hashCode 方法产生。这样一来，当集合要添加新的元素时，先调用这个元素的 hashCode 方法，就一下子能定位到它应该放置的物理位置上。

　　①、如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；

　　②、如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了；

　　③、不相同的话，也就是发生了Hash key相同导致冲突的情况，那么就在这个Hash key的地方产生一个链表，将所有产生相同HashCode的对象放到这个单链表上去，串在一起（很少出现）。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180314065445441-1398731623.png)

　　这里有 A,B,C,D四个对象，分别通过 hashCode 方法产生了三个值，注意 A 和 B 对象调用 hashCode 产生的值是相同的，即 A.hashCode() = B.hashCode() = 0x001,发生了哈希冲突，这时候由于最先是插入了 A，在插入的B的时候，我们发现 B 是要插入到 A 所在的位置，而 A 已经插入了，这时候就通过调用 equals 方法判断 A 和 B 是否相同，如果相同就不插入 B，如果不同则将 B 插入到 A 后面的位置。所以对于 equals 方法和 hashCode 方法有如下要求：



#### 　　一、hashCode 要求

　　①、在程序运行时期间，只要对象的（字段的）变化不会影响equals方法的决策结果，那么，在这个期间，无论调用多少次hashCode，都必须返回同一个散列码。

　　②、通过equals调用返回true 的2个对象的hashCode一定一样。

　　③、通过equasl返回false 的2个对象的散列码不需要不同，也就是他们的hashCode方法的返回值允许出现相同的情况。

　　因此我们可以得到如下推论：

　　**两个对象相等，其 hashCode 一定相同;**

　　**两个对象不相等，其 hashCode 有可能相同;**

　　**hashCode 相同的两个对象，不一定相等;**

　　**hashCode 不相同的两个对象，一定不相等;**

 　这四个推论通过上图可以更好的理解。

　　可能会有人疑问，对于不能重复的集合，为什么不直接通过 hashCode 对于每个元素都产生唯一的值，如果重复就是相同的值，这样不就不需要调用 equals 方法来判断是否相同了吗？
　　实际上对于元素不是很多的情况下，直接通过 hashCode 产生唯一的索引值，通过这个索引值能直接找到元素，而且还能判断是否相同。比如数据库存储的数据，ID 是有序排列的，我们能通过 ID 直接找到某个元素，如果新插入的元素 ID 已经有了，那就表示是重复数据，这是很完美的办法。但现实是存储的元素很难有这样的 ID 关键字，也就很难这种实现 hashCode 的唯一算法，再者就算能实现，但是产生的 hashCode 码是非常大的，这会大的超过 Java 所能表示的范围，很占内存空间，所以也是不予考虑的。



#### 　　二、hashCode 编写指导：

　　①、不同对象的hash码应该尽量不同，避免hash冲突，也就是算法获得的元素要尽量均匀分布。

　　②、hash 值是一个 int 类型，在Java中占用 4 个字节，也就是 232 次方，要避免溢出。

　　在 JDK 的 Integer类，Float 类，String 类等都重写了 hashCode 方法，我们自定义对象也可以参考这些类来写。

　　下面是 JDK String 类的hashCode 源码：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　再次提醒大家，对于 Map 集合，我们可以选取Java中的基本类型，还有引用类型 String 作为 key，因为它们都按照规范重写了 equals 方法和 hashCode 方法。但是如果你用自定义对象作为 key，那么一定要覆写 equals 方法和 hashCode 方法，不然会有意想不到的错误产生。

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 7、toString 方法

　　该方法在 JDK 的源码如下：

```
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

　　getClass().getName()是返回对象的全类名（包含包名）,Integer.toHexString(hashCode()) 是以16进制无符号整数形式返回此哈希码的字符串表示形式。

　　打印某个对象时，默认是调用 toString 方法，比如 System.out.println(person),等价于 System.out.println(person.toString())

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 8、notify()/notifyAll()/wait()

　　这是用于多线程之间的通信方法，在后面讲解多线程会详细描述，这里就不做讲解了。

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 9、finalize 方法

```
protected void finalize() throws Throwable { }
```

　　该方法用于垃圾回收，一般由 JVM 自动调用，一般不需要程序员去手动调用该方法。后面再讲解 JVM 的时候会详细展开描述。

[回到顶部](https://www.cnblogs.com/ysocean/p/8419559.html#_labelTop)

### 10、registerNatives 方法 

　　该方法在 Object 类中定义如下：

```
private static native void registerNatives();
```

　　这是一个本地方法，在[ native 介绍](http://www.cnblogs.com/ysocean/p/8476933.html) 中我们知道一个类定义了本地方法后，想要调用操作系统的实现，必须还要装载本地库，但是我们发现在 Object.class 类中具有很多本地方法，但是却没有看到本地库的载入代码。而且这是用 private 关键字声明的，在类外面根本调用不了，我们接着往下看关于这个方法的类似源码：

```
    static {
        registerNatives();
    }
```

　　看到上面的代码，这就明白了吧。静态代码块就是一个类在初始化过程中必定会执行的内容，所以在类加载的时候是会执行该方法的，通过该方法来注册本地方法。

　　

　　参考文档：https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html



# [java.lang.Integer 类](https://www.cnblogs.com/ysocean/p/8564466.html)



**目录**

- [1、Integer 的声明](https://www.cnblogs.com/ysocean/p/8564466.html#_label0)
- [2、Integer 的主要属性](https://www.cnblogs.com/ysocean/p/8564466.html#_label1)
- [3、构造方法 Integer(int)  Integer(String)](https://www.cnblogs.com/ysocean/p/8564466.html#_label2)
- [ 4、toString() toString(int i)  toString(int i, int radix)](https://www.cnblogs.com/ysocean/p/8564466.html#_label3)
- 5、自动拆箱和装箱
  - [　　①、自动装箱](https://www.cnblogs.com/ysocean/p/8564466.html#_label4_0)
  - [　　②、自动拆箱](https://www.cnblogs.com/ysocean/p/8564466.html#_label4_1)
- [6、equals(Object obj)方法](https://www.cnblogs.com/ysocean/p/8564466.html#_label5)
- [7、hashCode() 方法](https://www.cnblogs.com/ysocean/p/8564466.html#_label6)
- [8、parseInt(String s) 和 parseInt(String s, int radix) 方法](https://www.cnblogs.com/ysocean/p/8564466.html#_label7)
- [9、compareTo(Integer anotherInteger) 和 compare(int x, int y) 方法](https://www.cnblogs.com/ysocean/p/8564466.html#_label8)

 

------

　　上一篇博客我们介绍了 java.lang 包下的 Object 类，那么本篇博客接着介绍该包下的另一个类 Integer。在前面 [浅谈 Integer 类](http://www.cnblogs.com/ysocean/p/8075676.html) 博客中我们主要介绍了 Integer 类 和 int 基本数据类型的关系，本篇博客是从源码层次详细介绍 Integer 的实现。

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 1、Integer 的声明

```
public final class Integer extends Number implements Comparable<Integer>{}
```

　　Integer 是用 final 声明的常量类，不能被任何类所继承。并且 Integer 类继承了 Number 类和实现了 Comparable 接口。 Number 类是一个抽象类，8中基本数据类型的包装类除了Character 和 Boolean 没有继承该类外，剩下的都继承了 Number 类，该类的方法用于各种数据类型的转换。Comparable 接口就一个 compareTo 方法，用于元素之间的大小比较，下面会对这些方法详细展开介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 2、Integer 的主要属性

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180314211859826-1491124122.png)

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180314213412289-2147425658.png)

　　int 类型在 Java 中是占据 4 个字节，所以其可以表示大小的范围是 -2 31——2 31 -1即 -2147483648——2147483647，我们在用 int 表示数值时一定不要超出这个范围了。

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 3、构造方法 Integer(int)  Integer(String)

　　对于第一个构造方法 Integer(int)，源码如下，这没什么好说的。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public Integer(int var1) {
2         this.value = var1;
3     }
```

　　对于第二个构造方法 Integer(String) 就是将我们输入的字符串数据转换成整型数据。

　　首先我们必须要知道能转换成整数的字符串必须分为两个部分：第一位必须是"+"或者"-"，剩下的必须是 0-9 和 a-z 字符

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public Integer(String s) throws NumberFormatException {
 2     this.value = parseInt(s, 10);//首先调用parseInt(s,10)方法，其中s表示我们需要转换的字符串，10表示以十进制输出，默认也是10进制
 3 }
 4 
 5 public static int parseInt(String s, int radix) throws NumberFormatException{
 6     //如果转换的字符串如果为null，直接抛出空指针异常
 7     if (s == null) {
 8         throw new NumberFormatException("null");
 9     }
10     //如果转换的radix(默认是10)<2 则抛出数字格式异常，因为进制最小是 2 进制
11     if (radix < Character.MIN_RADIX) {
12         throw new NumberFormatException("radix " + radix +
13                                         " less than Character.MIN_RADIX");
14     }
15     //如果转换的radix(默认是10)>36 则抛出数字格式异常，因为0到9一共10位，a到z一共26位，所以一共36位
16     //也就是最高只能有36进制数
17     if (radix > Character.MAX_RADIX) {
18         throw new NumberFormatException("radix " + radix +
19                                         " greater than Character.MAX_RADIX");
20     }
21     int result = 0;
22     boolean negative = false;
23     int i = 0, len = s.length();//len是待转换字符串的长度
24     int limit = -Integer.MAX_VALUE;//limit = -2147483647
25     int multmin;
26     int digit;
27     //如果待转换字符串长度大于 0 
28     if (len > 0) {
29         char firstChar = s.charAt(0);//获取待转换字符串的第一个字符
30         //这里主要用来判断第一个字符是"+"或者"-"，因为这两个字符的 ASCII码都小于字符'0'
31         if (firstChar < '0') {  
32             if (firstChar == '-') {//如果第一个字符是'-'
33                 negative = true;
34                 limit = Integer.MIN_VALUE;
35             } else if (firstChar != '+')//如果第一个字符是不是 '+'，直接抛出异常
36                 throw NumberFormatException.forInputString(s);
37 
38             if (len == 1) //待转换字符长度是1，不能是单独的"+"或者"-"，否则抛出异常 
39                 throw NumberFormatException.forInputString(s);
40             i++;
41         }
42         multmin = limit / radix;
43         //通过不断循环，将字符串除掉第一个字符之后，根据进制不断相乘在相加得到一个正整数
44         //比如 parseInt("2abc",16) = 2*16的3次方+10*16的2次方+11*16+12*1 
45         //parseInt("123",10) = 1*10的2次方+2*10+3*1 
46         while (i < len) {
47             digit = Character.digit(s.charAt(i++),radix);
48             if (digit < 0) {
49                 throw NumberFormatException.forInputString(s);
50             }
51             if (result < multmin) {
52                 throw NumberFormatException.forInputString(s);
53             }
54             result *= radix;
55             if (result < limit + digit) {
56                 throw NumberFormatException.forInputString(s);
57             }
58             result -= digit;
59         }
60     } else {//如果待转换字符串长度小于等于0，直接抛出异常
61         throw NumberFormatException.forInputString(s);
62     }
63     //根据第一个字符得到的正负号，在结果前面加上符号
64     return negative ? result : -result;
65 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

###  4、toString() toString(int i)  toString(int i, int radix)

　　这三个方法重载，能返回一个整型数据所表示的字符串形式，其中最后一个方法 toString(int,int) 第二个参数是表示的进制数。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
public String toString() {
    return toString(value);
}

public static String toString(int i) {
    if (i == Integer.MIN_VALUE)
        return "-2147483648";
    int size = (i < 0) ? stringSize(-i) + 1 : stringSize(i);
    char[] buf = new char[size];
    getChars(i, size, buf);
    return new String(buf, true);
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　toString(int) 方法内部调用了 stringSize() 和 getChars() 方法，stringSize() 它是用来计算参数 i 的位数也就是转成字符串之后的字符串的长度，内部结合一个已经初始化好的int类型的数组sizeTable来完成这个计算。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };

    // Requires positive x
    static int stringSize(int x) {
        for (int i=0; ; i++)
            if (x <= sizeTable[i])
                return i+1;
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　实现的形式很巧妙。注意负数包含符号位，所以对于负数的位数是 stringSize(-i) + 1。

　　再看 getChars 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1    static void getChars(int i, int index, char[] buf) {
 2         int q, r;
 3         int charPos = index;
 4         char sign = 0;
 5 
 6         if (i < 0) {
 7             sign = '-';
 8             i = -i;
 9         }
10 
11         // Generate two digits per iteration
12         while (i >= 65536) {
13             q = i / 100;
14         // really: r = i - (q * 100);
15             r = i - ((q << 6) + (q << 5) + (q << 2));
16             i = q;
17             buf [--charPos] = DigitOnes[r];
18             buf [--charPos] = DigitTens[r];
19         }
20 
21         // Fall thru to fast mode for smaller numbers
22         // assert(i <= 65536, i);
23         for (;;) {
24             q = (i * 52429) >>> (16+3);
25             r = i - ((q << 3) + (q << 1));  // r = i-(q*10) ...
26             buf [--charPos] = digits [r];
27             i = q;
28             if (i == 0) break;
29         }
30         if (sign != 0) {
31             buf [--charPos] = sign;
32         }
33     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　i:被初始化的数字，

　　index:这个数字的长度(包含了负数的符号“-”)，

　　buf:字符串的容器-一个char型数组。

　　第一个if判断，如果i<0,sign记下它的符号“-”，同时将i转成整数。下面所有的操作也就只针对整数了，最后在判断sign如果不等于零将 sign 你的值放在char数组的首位buf [--charPos] = sign;。　　

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 5、自动拆箱和装箱

　　自动拆箱和自动装箱是 JDK1.5 以后才有的功能，也就是java当中众多的语法糖之一，它的执行是在编译期，会根据代码的语法，在生成class文件的时候，决定是否进行拆箱和装箱动作。



#### 　　①、自动装箱

　　我们知道一般创建一个类的对象需要通过 new 关键字，比如：

```
Object obj = new Object();
```

　　但是实际上，对于 Integer 类，我们却可以直接这样使用：

```
Integer a = 128;
```

　　为什么可以这样，通过反编译工具，我们可以看到，生成的class文件是：

```
Integer a = Integer.valueOf(128);
```

　　我们可以看看 valueOf() 方法

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
public static Integer valueOf(int i) {
    assert IntegerCache.high >= 127;
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　其实最后返回的也是通过new Integer() 产生的对象，但是这里要注意前面的一段代码，当i的值 -128 <= i <= 127 返回的是缓存类中的对象，并没有重新创建一个新的对象，这在通过 equals 进行比较的时候我们要注意。

　　这就是基本数据类型的自动装箱，128是基本数据类型，然后被解析成Integer类。



#### 　　②、自动拆箱

　　我们将 Integer 类表示的数据赋值给基本数据类型int，就执行了自动拆箱。

```
1 Integer a = new Integer(128);
2 int m = a;
```

　　反编译生成的class文件：

```
1 Integer a = new Integer(128);
2 int m = a.intValue();
```

　　简单来讲：自动装箱就是Integer.valueOf(int i);自动拆箱就是 i.intValue();关于拆箱和装箱的详细介绍可以看我[这篇博客](http://www.cnblogs.com/ysocean/p/8075676.html)。

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 6、equals(Object obj)方法

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这个方法很简单，先通过 [instanceof](http://www.cnblogs.com/ysocean/p/8486500.html) 关键字判断两个比较对象的关系，然后将对象强转为 Integer，在通过自动拆箱，转换成两个基本数据类 int，然后通过 == 比较。

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 7、hashCode() 方法

```
1     public int hashCode() {
2         return value;
3     }
```

　　Integer 类的hashCode 方法也比较简单，直接返回其 int 类型的数据。

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 8、parseInt(String s) 和 parseInt(String s, int radix) 方法

　　前面通过 toString(int i) 可以将整型数据转换成字符串类型输出，这里通过 parseInt(String s) 能将字符串转换成整型输出。

　　这两个方法我们在介绍 构造函数 Integer(String s) 时已经详细讲解了。

[回到顶部](https://www.cnblogs.com/ysocean/p/8564466.html#_labelTop)

### 9、compareTo(Integer anotherInteger) 和 compare(int x, int y) 方法

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public int compareTo(Integer anotherInteger) {
2         return compare(this.value, anotherInteger.value);
3     }
```

　　compareTo 方法内部直接调用 compare 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public static int compare(int x, int y) {
2         return (x < y) ? -1 : ((x == y) ? 0 : 1);
3     }
```

　　如果 x < y 返回 -1

　　如果 x == y 返回 0

　　如果 x > y 返回 1

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1 System.out.println(Integer.compare(1, 2));//-1
2 System.out.println(Integer.compare(1, 1));//0
3 System.out.println(Integer.compare(1, 0));//1
```

　　那么基本上 Integer 类的主要方法就介绍这么多了，后面如果有比较重要的，会再进行更新。

 

参考文档：https://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html



# [java.lang.String 类](https://www.cnblogs.com/ysocean/p/8571426.html)



**目录**

- [1、String 类的定义](https://www.cnblogs.com/ysocean/p/8571426.html#_label0)
- [2、字段属性](https://www.cnblogs.com/ysocean/p/8571426.html#_label1)
- [3、构造方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label2)
- [4、equals(Object anObject) 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label3)
- [5、hashCode() 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label4)
- [6、charAt(int index) 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label5)
- [7、compareTo(String anotherString) 和 compareToIgnoreCase(String str) 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label6)
- [8、concat(String str) 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label7)
- [9、indexOf(int ch) 和 indexOf(int ch, int fromIndex) 方法 ](https://www.cnblogs.com/ysocean/p/8571426.html#_label8)
- [10、split(String regex) 和 split(String regex, int limit) 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label9)
- [11、replace(char oldChar, char newChar) 和 String replaceAll(String regex, String replacement) 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label10)
- [12、substring(int beginIndex) 和 substring(int beginIndex, int endIndex) 方法](https://www.cnblogs.com/ysocean/p/8571426.html#_label11)
- [13、常量池](https://www.cnblogs.com/ysocean/p/8571426.html#_label12)
- [14、intern() 方法 ](https://www.cnblogs.com/ysocean/p/8571426.html#_label13)
- [15、String 真的不可变吗? ](https://www.cnblogs.com/ysocean/p/8571426.html#_label14)

 

------

　　String 类也是java.lang 包下的一个类，算是日常编码中最常用的一个类了，那么本篇博客就来详细的介绍 String 类。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 1、String 类的定义

```
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {}
```

　　和上一篇博客所讲的 Integer 类一样，这也是一个用 final 声明的常量类，不能被任何类所继承,而且**一旦一个String对象被创建, 包含在这个对象中的字符序列是不可改变的, 包括该类后续的所有方法都是不能修改该对象的，直至该对象被销毁，这是我们需要特别注意的（该类的一些方法看似改变了字符串，其实内部都是创建一个新的字符串，下面讲解方法时会介绍）**。接着实现了 Serializable接口，这是一个序列化标志接口，还实现了 Comparable 接口，用于比较两个字符串的大小（按顺序比较单个字符的ASCII码），后面会有具体方法实现；最后实现了 CharSequence 接口，表示是一个有序字符的集合，相应的方法后面也会介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 2、字段属性

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 /**用来存储字符串  */
2 private final char value[];
3 
4 /** 缓存字符串的哈希码 */
5 private int hash; // Default to 0
6 
7 /** 实现序列化的标识 */
8 private static final long serialVersionUID = -6849794470754667710L;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　一个 String 字符串实际上是一个 char 数组。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 3、构造方法

　　String 类的构造方法很多。可以通过初始化一个字符串，或者字符数组，或者字节数组等等来创建一个 String 对象。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180319082356444-1846575793.png)

```
1 String str1 = "abc";//注意这种字面量声明的区别，文末会详细介绍
2 String str2 = new String("abc");
3 String str3 = new String(new char[]{'a','b','c'});
```

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 4、equals(Object anObject) 方法

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public boolean equals(Object anObject) {
 2         if (this == anObject) {
 3             return true;
 4         }
 5         if (anObject instanceof String) {
 6             String anotherString = (String)anObject;
 7             int n = value.length;
 8             if (n == anotherString.value.length) {
 9                 char v1[] = value;
10                 char v2[] = anotherString.value;
11                 int i = 0;
12                 while (n-- != 0) {
13                     if (v1[i] != v2[i])
14                         return false;
15                     i++;
16                 }
17                 return true;
18             }
19         }
20         return false;
21     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　String 类重写了 equals 方法，比较的是组成字符串的每一个字符是否相同，如果都相同则返回true，否则返回false。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 5、hashCode() 方法

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public int hashCode() {
 2         int h = hash;
 3         if (h == 0 && value.length > 0) {
 4             char val[] = value;
 5 
 6             for (int i = 0; i < value.length; i++) {
 7                 h = 31 * h + val[i];
 8             }
 9             hash = h;
10         }
11         return h;
12     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　String 类的 hashCode 算法很简单，主要就是中间的 for 循环，计算公式如下：

```
s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```

　　s 数组即源码中的 val 数组，也就是构成字符串的字符数组。这里有个数字 **31 ，**为什么选择31作为乘积因子，而且没有用一个常量来声明？主要原因有两个：

　　①、31是一个不大不小的质数，是作为 hashCode 乘子的优选质数之一。

　　②、31可以被 JVM 优化，`31 * i = (i << 5) - i。因为移位运算比乘法运行更快更省性能。`

　　具体解释可以参考[这篇文章](https://www.cnblogs.com/nullllun/p/8350178.html)。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 6、charAt(int index) 方法

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public char charAt(int index) {
2         //如果传入的索引大于字符串的长度或者小于0，直接抛出索引越界异常
3         if ((index < 0) || (index >= value.length)) {
4             throw new StringIndexOutOfBoundsException(index);
5         }
6         return value[index];//返回指定索引的单个字符
7     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　我们知道一个字符串是由一个字符数组组成，这个方法是通过传入的索引（数组下标），返回指定索引的单个字符。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 7、compareTo(String anotherString) 和 compareToIgnoreCase(String str) 方法

　　我们先看看 compareTo 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public int compareTo(String anotherString) {
 2         int len1 = value.length;
 3         int len2 = anotherString.value.length;
 4         int lim = Math.min(len1, len2);
 5         char v1[] = value;
 6         char v2[] = anotherString.value;
 7 
 8         int k = 0;
 9         while (k < lim) {
10             char c1 = v1[k];
11             char c2 = v2[k];
12             if (c1 != c2) {
13                 return c1 - c2;
14             }
15             k++;
16         }
17         return len1 - len2;
18     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　源码也很好理解，该方法是按字母顺序比较两个字符串，是基于字符串中每个字符的 Unicode 值。当两个字符串某个位置的字符不同时，返回的是这一位置的字符 Unicode 值之差，当两个字符串都相同时，返回两个字符串长度之差。

 　compareToIgnoreCase() 方法在 compareTo 方法的基础上忽略大小写，我们知道大写字母是比小写字母的Unicode值小32的，底层实现是先都转换成大写比较，然后都转换成小写进行比较。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 8、concat(String str) 方法

　　该方法是将指定的字符串连接到此字符串的末尾。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public String concat(String str) {
 2         int otherLen = str.length();
 3         if (otherLen == 0) {
 4             return this;
 5         }
 6         int len = value.length;
 7         char buf[] = Arrays.copyOf(value, len + otherLen);
 8         str.getChars(buf, len);
 9         return new String(buf, true);
10     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　首先判断要拼接的字符串长度是否为0，如果为0，则直接返回原字符串。如果不为0，则通过 Arrays 工具类（后面会详细介绍这个工具类）的copyOf方法创建一个新的字符数组，长度为原字符串和要拼接的字符串之和，前面填充原字符串，后面为空。接着在通过 getChars 方法将要拼接的字符串放入新字符串后面为空的位置。

　　**注意：**返回值是 new String(buf, true)，也就是重新通过 new 关键字创建了一个新的字符串，原字符串是不变的。这也是前面我们说的**一旦一个String对象被创建, 包含在这个对象中的字符序列是不可改变的。**

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 9、indexOf(int ch) 和 indexOf(int ch, int fromIndex) 方法 

　　indexOf(int ch)，参数 ch 其实是字符的 Unicode 值，这里也可以放单个字符（默认转成int），作用是返回指定字符第一次出现的此字符串中的索引。其内部是调用 indexOf(int ch, int fromIndex)，只不过这里的 fromIndex =0 ，因为是从 0 开始搜索；而 indexOf(int ch, int fromIndex) 作用也是返回首次出现的此字符串内的索引，但是从指定索引处开始搜索。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
    public int indexOf(int ch) {
        return indexOf(ch, 0);//从第一个字符开始搜索
    }
```

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public int indexOf(int ch, int fromIndex) {
 2     final int max = value.length;//max等于字符的长度
 3     if (fromIndex < 0) {//指定索引的位置如果小于0，默认从 0 开始搜索
 4         fromIndex = 0;
 5     } else if (fromIndex >= max) {
 6         //如果指定索引值大于等于字符的长度（因为是数组，下标最多只能是max-1），直接返回-1
 7         return -1;
 8     }
 9 
10     if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {//一个char占用两个字节，如果ch小于2的16次方（65536），绝大多数字符都在此范围内
11         final char[] value = this.value;
12         for (int i = fromIndex; i < max; i++) {//for循环依次判断字符串每个字符是否和指定字符相等
13             if (value[i] == ch) {
14                 return i;//存在相等的字符，返回第一次出现该字符的索引位置，并终止循环
15             }
16         }
17         return -1;//不存在相等的字符，则返回 -1
18     } else {//当字符大于 65536时，处理的少数情况，该方法会首先判断是否是有效字符，然后依次进行比较
19         return indexOfSupplementary(ch, fromIndex);
20     }
21 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 10、split(String regex) 和 split(String regex, int limit) 方法

　　split(String regex) 将该字符串拆分为给定正则表达式的匹配。split(String regex , int limit) 也是一样，不过对于 limit 的取值有三种情况：

　　①、limit > 0 ，则pattern（模式）应用n - 1 次

```
1 String str = "a,b,c";
2 String[] c1 = str.split(",", 2);
3 System.out.println(c1.length);//2
4 System.out.println(Arrays.toString(c1));//{"a","b,c"}
```

　　②、limit = 0 ，则pattern（模式）应用无限次并且省略末尾的空字串

```
1 String str2 = "a,b,c,,";
2 String[] c2 = str2.split(",", 0);
3 System.out.println(c2.length);//3
4 System.out.println(Arrays.toString(c2));//{"a","b","c"}
```

　　③、limit < 0 ，则pattern（模式）应用无限次

```
1 String str2 = "a,b,c,,";
2 String[] c2 = str2.split(",", -1);
3 System.out.println(c2.length);//5
4 System.out.println(Arrays.toString(c2));//{"a","b","c","",""}
```

　　下面我们看看底层的源码实现。对于 split(String regex) 没什么好说的，内部调用 split(regex, 0) 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public String[] split(String regex) {
2         return split(regex, 0);
3     }
```

　　重点看 split(String regex, int limit) 的方法实现：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public String[] split(String regex, int limit) {
 2     /* 1、单个字符，且不是".$|()[{^?*+\\"其中一个
 3      * 2、两个字符，第一个是"\"，第二个大小写字母或者数字
 4      */
 5     char ch = 0;
 6     if (((regex.value.length == 1 &&
 7          ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
 8          (regex.length() == 2 &&
 9           regex.charAt(0) == '\\' &&
10           (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
11           ((ch-'a')|('z'-ch)) < 0 &&
12           ((ch-'A')|('Z'-ch)) < 0)) &&
13         (ch < Character.MIN_HIGH_SURROGATE ||
14          ch > Character.MAX_LOW_SURROGATE))
15     {
16         int off = 0;
17         int next = 0;
18         boolean limited = limit > 0;//大于0，limited==true,反之limited==false
19         ArrayList<String> list = new ArrayList<>();
20         while ((next = indexOf(ch, off)) != -1) {
21             //当参数limit<=0 或者 集合list的长度小于 limit-1
22             if (!limited || list.size() < limit - 1) {
23                 list.add(substring(off, next));
24                 off = next + 1;
25             } else {//判断最后一个list.size() == limit - 1
26                 list.add(substring(off, value.length));
27                 off = value.length;
28                 break;
29             }
30         }
31         //如果没有一个能匹配的，返回一个新的字符串，内容和原来的一样
32         if (off == 0)
33             return new String[]{this};
34 
35         // 当 limit<=0 时，limited==false,或者集合的长度 小于 limit是，截取添加剩下的字符串
36         if (!limited || list.size() < limit)
37             list.add(substring(off, value.length));
38 
39         // 当 limit == 0 时，如果末尾添加的元素为空（长度为0），则集合长度不断减1，直到末尾不为空
40         int resultSize = list.size();
41         if (limit == 0) {
42             while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
43                 resultSize--;
44             }
45         }
46         String[] result = new String[resultSize];
47         return list.subList(0, resultSize).toArray(result);
48     }
49     return Pattern.compile(regex).split(this, limit);
50 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 11、replace(char oldChar, char newChar) 和 String replaceAll(String regex, String replacement) 方法

　　①、replace(char oldChar, char newChar) ：将原字符串中所有的oldChar字符都替换成newChar字符，返回一个新的字符串。

　　②、String replaceAll(String regex, String replacement)：将匹配正则表达式regex的匹配项都替换成replacement字符串，返回一个新的字符串。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 12、substring(int beginIndex) 和 substring(int beginIndex, int endIndex) 方法

　　①、substring(int beginIndex)：返回一个从索引 beginIndex 开始一直到结尾的子字符串。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
public String substring(int beginIndex) {
    if (beginIndex < 0) {//如果索引小于0，直接抛出异常
        throw new StringIndexOutOfBoundsException(beginIndex);
    }
    int subLen = value.length - beginIndex;//subLen等于字符串长度减去索引
    if (subLen < 0) {//如果subLen小于0，也是直接抛出异常
        throw new StringIndexOutOfBoundsException(subLen);
    }
    //1、如果索引值beginIdex == 0，直接返回原字符串
    //2、如果不等于0，则返回从beginIndex开始，一直到结尾
    return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　②、 substring(int beginIndex, int endIndex)：返回一个从索引 beginIndex 开始，到 endIndex 结尾的子字符串。

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 13、常量池

 　在前面讲解构造函数的时候，我们知道最常见的两种声明一个字符串对象的形式有两种：

　　①、通过“字面量”的形式直接赋值

```
String str = "hello";
```

　　②、通过 new 关键字调用构造函数创建对象

```
String str = new String("hello");
```

　　那么这两种声明方式有什么区别呢？在讲解之前，我们先介绍 **JDK1.7（不包括1.7）**以前的 JVM 的内存分布：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180321102257650-212450093.png)

　　①、程序计数器：也称为 PC 寄存器，保存的是程序当前执行的指令的地址（也可以说保存下一条指令的所在存储单元的地址），当CPU需要执行指令时，需要从程序计数器中得到当前需要执行的指令所在存储单元的地址，然后根据得到的地址获取到指令，在得到指令之后，程序计数器便自动加1或者根据转移指针得到下一条指令的地址，如此循环，直至执行完所有的指令。**线程私有**。

　　②、虚拟机栈：基本数据类型、对象的引用都存放在这。**线程私有**。

　　③、本地方法栈：虚拟机栈是为执行Java方法服务的，而本地方法栈则是为执行本地方法（Native Method）服务的。在JVM规范中，并没有对本地方法栈的具体实现方法以及数据结构作强制规定，虚拟机可以自由实现它。在HotSopt虚拟机中直接就把本地方法栈和虚拟机栈合二为一。

　　④、方法区：存储了每个类的信息（包括类的名称、方法信息、字段信息）、静态变量、常量以及编译器编译后的代码等。注意：在Class文件中除了类的字段、方法、接口等描述信息外，还有一项信息是**常量池**，用来存储编译期间生成的**字面量**和符号引用。

　　⑤、堆：用来存储对象本身的以及数组（当然，数组引用是存放在Java栈中的）。

　　在 JDK1.7 以后，**方法区的常量池被移除放到堆中了**，如下：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180321102316512-713026536.png)

　　常量池：Java运行时会维护一个String Pool（String池）， 也叫“字符串缓冲区”。String池用来存放运行时中产生的各种字符串，并且池中的字符串的内容不重复。

　　**①、字面量创建字符串或者纯字符串（常量）拼接字符串会先在字符串池中找，看是否有相等的对象，没有的话就在字符串池创建该对象；有的话则直接用池中的引用，避免重复创建对象。**

　　**②、new关键字创建时，直接在堆中创建一个新对象，变量所引用的都是这个新对象的地址，但是如果通过new关键字创建的字符串内容在常量池中存在了，那么会由堆在指向常量池的对应字符；但是反过来，如果通过new关键字创建的字符串对象在常量池中没有，那么通过new关键词创建的字符串对象是不会额外在常量池中维护的。**

　　**③、使用包含变量表达式来创建String对象，则不仅会检查维护字符串池，还会在堆区创建这个对象，最后是指向堆内存的对象。**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 String str1 = "hello";
2 String str2 = "hello";
3 String str3 = new String("hello");
4 System.out.println(str1==str2);//true
5 System.out.println(str1==str3);//fasle
6 System.out.println(str2==str3);//fasle
7 System.out.println(str1.equals(str2));//true
8 System.out.println(str1.equals(str3));//true
9 System.out.println(str2.equals(str3));//true
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　对于上面的情况，首先 String str1 = "hello"，会先到常量池中检查是否有“hello”的存在，发现是没有的，于是在常量池中创建“hello”对象，并将常量池中的引用赋值给str1；第二个字面量 String str2 = "hello"，在常量池中检测到该对象了，直接将引用赋值给str2；第三个是通过new关键字创建的对象，常量池中有了该对象了，不用在常量池中创建，然后在堆中创建该对象后，将堆中对象的引用赋值给str3，再将该对象指向常量池。如下图所示：

　　![img](https://img2018.cnblogs.com/blog/1120165/201906/1120165-20190615234817441-1307711026.png)

　　**注意**：看上图红色的箭头，通过 new 关键字创建的字符串对象，如果常量池中存在了，会将堆中创建的对象指向常量池的引用。我们可以通过文章末尾介绍的intern()方法来验证。

　　**使用包含变量表达式创建对象：**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 String str1 = "hello";
2 String str2 = "helloworld";
3 String str3 = str1+"world";//编译器不能确定为常量(会在堆区创建一个String对象)
4 String str4 = "hello"+"world";//编译器确定为常量，直接到常量池中引用
5 
6 System.out.println(str2==str3);//fasle
7 System.out.println(str2==str4);//true
8 System.out.println(str3==str4);//fasle
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　str3 由于含有变量str1，编译器不能确定是常量，会在堆区中创建一个String对象。而str4是两个常量相加，直接引用常量池中的对象即可。

![img](https://img2018.cnblogs.com/blog/1120165/201906/1120165-20190615234833062-287021429.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 14、intern() 方法 

　　这是一个本地方法：

```
public native String intern();
```

　　**当调用intern方法时，如果池中已经包含一个与该`String`确定的字符串相同`equals(Object)`的字符串，则返回该字符串。否则，将此`String`对象添加到池中，并``返回此对象的引用。**

　　这句话什么意思呢？就是说调用一个String对象的intern()方法，如果常量池中有该对象了，直接返回该字符串的引用（存在堆中就返回堆中，存在池中就返回池中），如果没有，则将该对象添加到池中，并返回池中的引用。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 String str1 = "hello";//字面量 只会在常量池中创建对象
 2 String str2 = str1.intern();
 3 System.out.println(str1==str2);//true
 4 
 5 String str3 = new String("world");//new 关键字只会在堆中创建对象
 6 String str4 = str3.intern();
 7 System.out.println(str3 == str4);//false
 8 
 9 String str5 = str1 + str2;//变量拼接的字符串，会在常量池中和堆中都创建对象
10 String str6 = str5.intern();//这里由于池中已经有对象了，直接返回的是对象本身，也就是堆中的对象
11 System.out.println(str5 == str6);//true
12 
13 String str7 = "hello1" + "world1";//常量拼接的字符串，只会在常量池中创建对象
14 String str8 = str7.intern();
15 System.out.println(str7 == str8);//true
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8571426.html#_labelTop)

### 15、String 真的不可变吗? 

　　前面我们介绍了，String 类是用 final 关键字修饰的，所以我们认为其是不可变对象。但是真的不可变吗？

　　每个字符串都是由许多单个字符组成的，我们知道其源码是由 char[] value 字符数组构成。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 public final class String
2     implements java.io.Serializable, Comparable<String>, CharSequence {
3     /** The value is used for character storage. */
4     private final char value[];
5 
6     /** Cache the hash code for the string */
7     private int hash; // Default to 0
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　value 被 final 修饰，只能保证引用不被改变，但是 value 所指向的堆中的数组，才是真实的数据，只要能够操作堆中的数组，依旧能改变数据。而且 value 是基本类型构成，那么一定是可变的，即使被声明为 private，我们也可以通过反射来改变。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 String str = "vae";
 2 //打印原字符串
 3 System.out.println(str);//vae
 4 //获取String类中的value字段 
 5 Field fieldStr = String.class.getDeclaredField("value");
 6 //因为value是private声明的，这里修改其访问权限
 7 fieldStr.setAccessible(true);
 8 //获取str对象上的value属性的值
 9 char[] value = (char[]) fieldStr.get(str);
10 //将第一个字符修改为 V(小写改大写)
11 value[0] = 'V';
12 //打印修改之后的字符串
13 System.out.println(str);//Vae
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**通过前后两次打印的结果，我们可以看到 String 被改变了，但是在代码里，几乎不会使用反射的机制去操作 String 字符串，所以，我们会认为 String 类型是不可变的。**

　　那么，String 类为什么要这样设计成不可变呢？我们可以从性能以及安全方面来考虑：

- 安全
  - 引发安全问题，譬如，数据库的用户名、密码都是以字符串的形式传入来获得数据库的连接，或者在socket编程中，主机名和端口都是以字符串的形式传入。因为字符串是不可变的，所以它的值是不可改变的，否则黑客们可以钻到空子，改变字符串指向的对象的值，造成安全漏洞。
  - 保证线程安全，在并发场景下，多个线程同时读写资源时，会引竞态条件，由于 String 是不可变的，不会引发线程的问题而保证了线程。
  - HashCode，当 String 被创建出来的时候，hashcode也会随之被缓存，hashcode的计算与value有关，若 String 可变，那么 hashcode 也会随之变化，针对于 Map、Set 等容器，他们的键值需要保证唯一性和一致性，因此，String 的不可变性使其比其他对象更适合当容器的键值。
- 性能
  - 当字符串是不可变时，字符串常量池才有意义。字符串常量池的出现，可以减少创建相同字面量的字符串，让不同的引用指向池中同一个字符串，为运行时节约很多的堆内存。若字符串可变，字符串常量池失去意义，基于常量池的String.intern()方法也失效，每次创建新的 String 将在堆内开辟出新的空间，占据更多的内存。

 

参考文档：

https://docs.oracle.com/javase/8/docs/api/java/lang/String.html

https://segmentfault.com/a/1190000009914328



# [java.util.Arrays 类](https://www.cnblogs.com/ysocean/p/8616122.html)



**目录**

- [1、asList](https://www.cnblogs.com/ysocean/p/8616122.html#_label0)
- [2、sort](https://www.cnblogs.com/ysocean/p/8616122.html#_label1)
- [3、binarySearch](https://www.cnblogs.com/ysocean/p/8616122.html#_label2)
- [4、copyOf](https://www.cnblogs.com/ysocean/p/8616122.html#_label3)
- [5、equals 和 deepEquals](https://www.cnblogs.com/ysocean/p/8616122.html#_label4)
- [6、fill](https://www.cnblogs.com/ysocean/p/8616122.html#_label5)
- [7、toString 和 deepToString](https://www.cnblogs.com/ysocean/p/8616122.html#_label6)

 

------

　　java.util.Arrays 类是 JDK 提供的一个工具类，用来处理数组的各种方法，而且每个方法基本上都是静态方法，能直接通过类名Arrays调用。

[回到顶部](https://www.cnblogs.com/ysocean/p/8616122.html#_labelTop)

### 1、asList

```
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```

　　作用是返回由指定数组支持的**固定大小列表**。

　　注意：这个方法返回的 ArrayList 不是我们常用的集合类 java.util.ArrayList。这里的 ArrayList 是 Arrays 的一个内部类 java.util.Arrays.ArrayList。这个内部类有如下属性和方法：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180321135927416-748309816.png)

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     private static class ArrayList<E> extends AbstractList<E>
 2         implements RandomAccess, java.io.Serializable
 3     {
 4         private static final long serialVersionUID = -2764017481108945198L;
 5         private final E[] a;
 6 
 7         ArrayList(E[] array) {
 8             if (array==null)
 9                 throw new NullPointerException();
10             a = array;
11         }
12 
13         public int size() {
14             return a.length;
15         }
16 
17         public Object[] toArray() {
18             return a.clone();
19         }
20 
21         public <T> T[] toArray(T[] a) {
22             int size = size();
23             if (a.length < size)
24                 return Arrays.copyOf(this.a, size,
25                                      (Class<? extends T[]>) a.getClass());
26             System.arraycopy(this.a, 0, a, 0, size);
27             if (a.length > size)
28                 a[size] = null;
29             return a;
30         }
31 
32         public E get(int index) {
33             return a[index];
34         }
35 
36         public E set(int index, E element) {
37             E oldValue = a[index];
38             a[index] = element;
39             return oldValue;
40         }
41 
42         public int indexOf(Object o) {
43             if (o==null) {
44                 for (int i=0; i<a.length; i++)
45                     if (a[i]==null)
46                         return i;
47             } else {
48                 for (int i=0; i<a.length; i++)
49                     if (o.equals(a[i]))
50                         return i;
51             }
52             return -1;
53         }
54 
55         public boolean contains(Object o) {
56             return indexOf(o) != -1;
57         }
58     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**①、返回的 ArrayList 数组是一个定长列表，我们只能对其进行查看或者修改，但是不能进行添加或者删除操作**

　　通过源码我们发现该类是没有add()或者remove() 这样的方法的，如果对其进行增加或者删除操作，都会调用其父类 AbstractList 对应的方法，而追溯父类的方法最终会抛出 UnsupportedOperationException 异常。如下：

```
1 String[] str = {"a","b","c"};
2 List<String> listStr = Arrays.asList(str);
3 listStr.set(1, "e");//可以进行修改
4 System.out.println(listStr.toString());//[a, e, c]
5 listStr.add("a");//添加元素会报错 java.lang.UnsupportedOperationException 
```

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180321142251208-1448605579.png)

　　**②、引用类型的数组和基本类型的数组区别**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 String[] str = {"a","b","c"};
2 List listStr = Arrays.asList(str);
3 System.out.println(listStr.size());//3
4 
5 int[] i = {1,2,3};
6 List listI = Arrays.asList(i);
7 System.out.println(listI.size());//1
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　上面的结果第一个listStr.size()==3，而第二个 listI.size()==1。这是为什么呢？

　　我们看源码，在 Arrays.asList 中，方法声明为 <T> List<T> asList(T... a)。该方法接收一个可变参数，并且这个可变参数类型是作为泛型的参数。我们知道基本数据类型是不能作为泛型的参数的，但是数组是引用类型，所以数组是可以泛型化的，于是 int[] 作为了整个参数类型，而不是 int 作为参数类型。

　　所以将上面的方法泛型化补全应该是：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 String[] str = {"a","b","c"};
 2 List<String> listStr = Arrays.asList(str);
 3 System.out.println(listStr.size());//3
 4 
 5 int[] i = {1,2,3};
 6 List<int[]> listI = Arrays.asList(i);//注意这里List参数为 int[] ，而不是 int
 7 System.out.println(listI.size());//1
 8 
 9 Integer[] in = {1,2,3};
10 List<Integer> listIn = Arrays.asList(in);//这里参数为int的包装类Integer，所以集合长度为3
11 System.out.println(listIn.size());//3
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**③、返回的列表ArrayList里面的元素都是引用，不是独立出来的对象**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 String[] str = {"a","b","c"};
2 List<String> listStr = Arrays.asList(str);
3 //执行更新操作前
4 System.out.println(Arrays.toString(str));//[a, b, c]
5 listStr.set(0, "d");//将第一个元素a改为d
6 //执行更新操作后
7 System.out.println(Arrays.toString(str));//[d, b, c]
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这里的Arrays.toString()方法就是打印数组的内容，后面会介绍。我们看修改集合的内容，原数组的内容也变化了，所以这里传入的是引用类型。

　　**④、已知数组数据，如何快速获取一个可进行增删改查的列表List？**

```
1 String[] str = {"a","b","c"};
2 List<String> listStr = new ArrayList<>(Arrays.asList(str));
3 listStr.add("d");
4 System.out.println(listStr.size());//4
```

　　这里的ArrayList 集合类后面我们会详细讲解，大家目前只需要知道有这种用法即可。

　　**⑤、Arrays.asList() 方法使用场景**

　　Arrays工具类提供了一个方法asList, 使用该方法可以将一个变长参数或者数组转换成List 。但是，生成的List的长度是固定的；能够进行修改操作（比如，修改某个位置的元素）；不能执行影响长度的操作（如add、remove等操作），否则会抛出UnsupportedOperationException异常。

　　所以 **Arrays.asList 比较适合那些已经有数组数据或者一些元素，而需要快速构建一个List，只用于读取操作，而不进行添加或删除操作的场景。**

[回到顶部](https://www.cnblogs.com/ysocean/p/8616122.html#_labelTop)

### 2、sort

　　该方法是用于数组排序，在 Arrays 类中有该方法的一系列重载方法，能对7种基本数据类型，包括 byte,char,double,float,int,long,short 等都能进行排序，还有 Object 类型（实现了Comparable接口），以及比较器 Comparator 。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180321150243908-2093552626.png)

　　**①、基本类型的数组**

　　这里我们以 int[ ] 为例看看：

```
1 int[] num = {1,3,8,5,2,4,6,7};
2 Arrays.sort(num);
3 System.out.println(Arrays.toString(num));//[1, 2, 3, 4, 5, 6, 7, 8]
```

　　通过调用 sort(int[] a) 方法，将原数组按照升序的顺序排列。下面我们通过源码看看是如何实现排序的：

```
    public static void sort(int[] a) {
        DualPivotQuicksort.sort(a, 0, a.length - 1, null, 0, 0);
    }
```

　　在 Arrays.sort 方法内部调用 DualPivotQuicksort.sort 方法，这个方法的源码很长，分别对于数组的长度进行了各种算法的划分，包括快速排序，插入排序，冒泡排序都有使用。详细源码可以参考[这篇博客](https://www.cnblogs.com/yuxiaofei93/p/5722714.html)。

　　**②、对象类型数组**

　　该类型的数组进行排序可以实现 Comparable 接口，重写 compareTo 方法进行排序。

```
1 String[] str = {"a","f","c","d"};
2 Arrays.sort(str);
3 System.out.println(Arrays.toString(str));//[a, c, d, f]
```

　　String 类型实现了 Comparable 接口，内部的 compareTo 方法是按照字典码进行比较的。

　　**③、没有实现Comparable接口的，可以通过Comparator实现排序**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 Person[] p = new Person[]{new Person("zhangsan",22),new Person("wangwu",11),new Person("lisi",33)};
 2 Arrays.sort(p,new Comparator<Person>() {
 3     @Override
 4     public int compare(Person o1, Person o2) {
 5         if(o1 == null || o2 == null){
 6             return 0;
 7         }
 8         return o1.getPage()-o2.getPage();
 9     }
10 });    
11 System.out.println(Arrays.toString(p));
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8616122.html#_labelTop)

### 3、binarySearch

　　用二分法查找数组中的某个元素。该方法和 sort 方法一样，适用于各种基本数据类型以及对象。

　　注意：二分法是对以及**有序的数组**进行查找（比如先用Arrays.sort()进行排序，然后调用此方法进行查找）。找到元素返回下标，没有则返回 -1

　　实例：

```
1 int[] num = {1,3,8,5,2,4,6,7};
2 Arrays.sort(num);
3 System.out.println(Arrays.toString(num));//[1, 2, 3, 4, 5, 6, 7, 8]
4 System.out.println(Arrays.binarySearch(num, 2));//返回元素的下标 1
```

　　具体源码实现：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

[回到顶部](https://www.cnblogs.com/ysocean/p/8616122.html#_labelTop)

### 4、copyOf

　　拷贝数组元素。底层采用 System.arraycopy() 实现，这是一个native方法。

```
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```

　　src:源数组

　　srcPos:源数组要复制的起始位置

　　dest:目的数组

　　destPos:目的数组放置的起始位置

　　length:复制的长度

　　注意：src 和 dest都必须是同类型或者可以进行转换类型的数组。

```
int[] num1 = {1,2,3};
int[] num2 = new int[3];
System.arraycopy(num1, 0, num2, 0, num1.length);
System.out.println(Arrays.toString(num2));//[1, 2, 3]
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    /**
     * @param original 源数组
     * @param newLength //返回新数组的长度
     * @return
     */
    public static int[] copyOf(int[] original, int newLength) {
        int[] copy = new int[newLength];
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8616122.html#_labelTop)

### 5、equals 和 deepEquals

　　**①、equals**

　　equals 用来比较两个数组中对应位置的每个元素是否相等。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180322101515906-4885028.png)

　　八种基本数据类型以及对象都能进行比较。

　　我们先看看 int类型的数组比较源码实现：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　在看对象数组的比较：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　基本上也是通过 equals 来判断。

　　**②、deepEquals**

　　也是用来比较两个数组的元素是否相等，不过 deepEquals 能够进行比较多维数组，而且是任意层次的嵌套数组。

```
         String[][] name1 = {{ "G","a","o" },{ "H","u","a","n"},{ "j","i","e"}};  
         String[][] name2 = {{ "G","a","o" },{ "H","u","a","n"},{ "j","i","e"}};
         System.out.println(Arrays.equals(name1,name2));// false  
         System.out.println(Arrays.deepEquals(name1,name2));// true
```

[回到顶部](https://www.cnblogs.com/ysocean/p/8616122.html#_labelTop)

### 6、fill

　　该系列方法用于给数组赋值，并能指定某个范围赋值。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    //给a数组所有元素赋值 val
    public static void fill(int[] a, int val) {
        for (int i = 0, len = a.length; i < len; i++)
            a[i] = val;
    }
    
    //给从 fromIndex 开始的下标，toIndex-1结尾的下标都赋值 val,左闭右开
    public static void fill(int[] a, int fromIndex, int toIndex, int val) {
        rangeCheck(a.length, fromIndex, toIndex);//判断范围是否合理
        for (int i = fromIndex; i < toIndex; i++)
            a[i] = val;
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8616122.html#_labelTop)

### 7、toString 和 deepToString

　　toString 用来打印一维数组的元素，而 deepToString 用来打印多层次嵌套的数组元素。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public static String toString(int[] a) {
 2         if (a == null)
 3             return "null";
 4         int iMax = a.length - 1;
 5         if (iMax == -1)
 6             return "[]";
 7 
 8         StringBuilder b = new StringBuilder();
 9         b.append('[');
10         for (int i = 0; ; i++) {
11             b.append(a[i]);
12             if (i == iMax)
13                 return b.append(']').toString();
14             b.append(", ");
15         }
16     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 

参考文档：https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html



# [java.util.ArrayList 类](https://www.cnblogs.com/ysocean/p/8622264.html)



**目录**

- [1、ArrayList 定义](https://www.cnblogs.com/ysocean/p/8622264.html#_label0)
- [2、字段属性](https://www.cnblogs.com/ysocean/p/8622264.html#_label1)
- [3、构造函数](https://www.cnblogs.com/ysocean/p/8622264.html#_label2)
- [4、添加元素](https://www.cnblogs.com/ysocean/p/8622264.html#_label3)
- 5、删除元素
  - [　　①、根据索引删除元素](https://www.cnblogs.com/ysocean/p/8622264.html#_label4_0)
  - [　　②、直接删除指定元素](https://www.cnblogs.com/ysocean/p/8622264.html#_label4_1)
- [6、修改元素](https://www.cnblogs.com/ysocean/p/8622264.html#_label5)
- 7、查找元素
  - [　　①、根据索引查找元素](https://www.cnblogs.com/ysocean/p/8622264.html#_label6_0)
  - [　　②、根据元素查找索引](https://www.cnblogs.com/ysocean/p/8622264.html#_label6_1)
- 8、遍历集合
  - [　　①、普通 for 循环遍历](https://www.cnblogs.com/ysocean/p/8622264.html#_label7_0)
  - [　　②、迭代器 iterator](https://www.cnblogs.com/ysocean/p/8622264.html#_label7_1)
  - [　　③、迭代器的变种 forEach](https://www.cnblogs.com/ysocean/p/8622264.html#_label7_2)
  - [　　④、迭代器 ListIterator](https://www.cnblogs.com/ysocean/p/8622264.html#_label7_3)
- [ 9、SubList](https://www.cnblogs.com/ysocean/p/8622264.html#_label8)
- [10、size()](https://www.cnblogs.com/ysocean/p/8622264.html#_label9)
- [11、isEmpty()](https://www.cnblogs.com/ysocean/p/8622264.html#_label10)
- [12、trimToSize()](https://www.cnblogs.com/ysocean/p/8622264.html#_label11)

 

------

　　关于 JDK 的集合类的整体介绍可以看[这张图](http://img.blog.csdn.net/20160124221843905)，本篇博客我们不系统的介绍整个集合的构造，重点是介绍 ArrayList 类是如何实现的。

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 1、ArrayList 定义

　　**ArrayList 是一个用数组实现的集合，支持随机访问，元素有序且可以重复。**

```
public` `class` `ArrayList ``extends` `AbstractList``    ``implements` `List, RandomAccess, Cloneable, java.io.Serializable
```

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180323122628544-789225625.png)

　　**①、实现 RandomAccess 接口**

　　这是一个标记接口，一般此标记接口用于 `List `实现，以表明它们支持快速（通常是恒定时间）的随机访问。该接口的主要目的是允许通用算法改变其行为，以便在应用于随机或顺序访问列表时提供良好的性能。

　　比如在工具类 Collections(这个工具类后面会详细讲解)中，应用二分查找方法时判断是否实现了 RandomAccess 接口：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　**②、实现 Cloneable 接口**

　　这个类是 java.lang.Cloneable，前面我们讲解[深拷贝和浅拷贝](http://www.cnblogs.com/ysocean/p/8482979.html)的原理时，我们介绍了浅拷贝可以通过调用 Object.clone() 方法来实现，但是调用该方法的对象必须要实现 Cloneable 接口，否则会抛出 CloneNoSupportException`异常。`

　　Cloneable 和 RandomAccess 接口一样也是一个标记接口，接口内无任何方法体和常量的声明，也就是说如果想克隆对象，必须要实现 Cloneable 接口，表明该类是可以被克隆的。

　　**③、实现 Serializable 接口**

　　这个没什么好说的，也是标记接口，表示能被序列化。

　　**④、实现 List 接口**

　　这个接口是 List 类集合的上层接口，定义了实现该接口的类都必须要实现的一组方法，如下所示，下面我们会对这一系列方法的实现做详细介绍。

　　**![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180326080038389-874708021.png)**

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 2、字段属性

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
        //集合的默认大小
        private static final int DEFAULT_CAPACITY = 10;
        //空的数组实例
        private static final Object[] EMPTY_ELEMENTDATA = {};
        //这也是一个空的数组实例，和EMPTY_ELEMENTDATA空数组相比是用于了解添加元素时数组膨胀多少
        private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
        //存储 ArrayList集合的元素，集合的长度即这个数组的长度
        //1、当 elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA 时将会清空 ArrayList
        //2、当添加第一个元素时，elementData 长度会扩展为 DEFAULT_CAPACITY=10
        transient Object[] elementData;
        //表示集合的长度
        private int size;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 3、构造函数

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public ArrayList() {
2         this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
3     }
```

　　此无参构造函数将创建一个 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 声明的数组，注意此时初始容量是0，而不是大家以为的 10。

　　**注意：根据默认构造函数创建的集合，ArrayList list = new ArrayList();此时集合长度是0.**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　初始化集合大小创建 ArrayList 集合。当大于0时，给定多少那就创建多大的数组；当等于0时，创建一个空数组；当小于0时，抛出异常。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public ArrayList(Collection<? extends E> c) {
 2         elementData = c.toArray();
 3         if ((size = elementData.length) != 0) {
 4             // c.toArray might (incorrectly) not return Object[] (see 6260652)
 5             if (elementData.getClass() != Object[].class)
 6                 elementData = Arrays.copyOf(elementData, size, Object[].class);
 7         } else {
 8             // replace with empty array.
 9             this.elementData = EMPTY_ELEMENTDATA;
10         }
11     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这是将已有的集合复制到 ArrayList 集合中去。

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 4、添加元素

　　通过前面的字段属性和构造函数，我们知道 ArrayList 集合是由数组构成的，那么向 ArrayList 中添加元素，也就是向数组赋值。我们知道一个数组的声明是能确定大小的，而使用 ArrayList 时，好像是能添加任意多个元素，这就涉及到数组的扩容。

　　扩容的核心方法就是调用前面我们讲过的**Arrays.copyOf** 方法，创建一个更大的数组，然后将原数组元素拷贝过去即可。下面我们看看具体实现：

```
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  //添加元素之前，首先要确定集合的大小
        elementData[size++] = e;
        return true;
    }
```

　　如上所示，在通过调用 add 方法添加元素之前，我们要首先调用 ensureCapacityInternal 方法来确定集合的大小，如果集合满了，则要进行扩容操作。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     private void ensureCapacityInternal(int minCapacity) {//这里的minCapacity 是集合当前大小+1
 2         //elementData 是实际用来存储元素的数组，注意数组的大小和集合的大小不是相等的，前面的size是指集合大小
 3         ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
 4     }
 5     private static int calculateCapacity(Object[] elementData, int minCapacity) {
 6         if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {//如果数组为空，则从size+1的值和默认值10中取最大的
 7             return Math.max(DEFAULT_CAPACITY, minCapacity);
 8         }
 9         return minCapacity;//不为空，则返回size+1
10     }
11     private void ensureExplicitCapacity(int minCapacity) {
12         modCount++;
13 
14         // overflow-conscious code
15         if (minCapacity - elementData.length > 0)
16             grow(minCapacity);
17     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　在 ensureExplicitCapacity 方法中，首先对修改次数modCount加一，这里的modCount给ArrayList的迭代器使用的，在并发操作被修改时，提供快速失败行为（保证modCount在迭代期间不变，否则抛出ConcurrentModificationException异常，可以查看源码865行），接着判断minCapacity是否大于当前ArrayList内部数组长度，大于的话调用grow方法对内部数组elementData扩容，grow方法代码如下：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     private void grow(int minCapacity) {
 2         int oldCapacity = elementData.length;//得到原始数组的长度
 3         int newCapacity = oldCapacity + (oldCapacity >> 1);//新数组的长度等于原数组长度的1.5倍
 4         if (newCapacity - minCapacity < 0)//当新数组长度仍然比minCapacity小，则为保证最小长度，新数组等于minCapacity
 5             newCapacity = minCapacity;
 6         //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8 = 2147483639
 7         if (newCapacity - MAX_ARRAY_SIZE > 0)//当得到的新数组长度比 MAX_ARRAY_SIZE 大时，调用 hugeCapacity 处理大数组
 8             newCapacity = hugeCapacity(minCapacity);
 9         //调用 Arrays.copyOf 将原数组拷贝到一个大小为newCapacity的新数组（注意是拷贝引用）
10         elementData = Arrays.copyOf(elementData, newCapacity);
11     }
12     
13     private static int hugeCapacity(int minCapacity) {
14         if (minCapacity < 0) // 
15             throw new OutOfMemoryError();
16         return (minCapacity > MAX_ARRAY_SIZE) ? //minCapacity > MAX_ARRAY_SIZE,则新数组大小为Integer.MAX_VALUE
17             Integer.MAX_VALUE :
18             MAX_ARRAY_SIZE;
19     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　对于 ArrayList 集合添加元素，我们总结一下：

　　**①、当通过 ArrayList() 构造一个空集合，初始长度是为0的，第 1 次添加元素，会创建一个长度为10的数组，并将该元素赋值到数组的第一个位置。**

　　**②、第 2 次添加元素，集合不为空，而且由于集合的长度size+1是小于数组的长度10，所以直接添加元素到数组的第二个位置，不用扩容。**

　　**③、第 11 次添加元素，此时 size+1 = 11，而数组长度是10，这时候创建一个长度为10+10\*0.5 = 15 的数组（扩容1.5倍），然后将原数组元素引用拷贝到新数组。并将第 11 次添加的元素赋值到新数组下标为10的位置。**

　　**④、第 Integer.MAX_VALUE - 8 = 2147483639，然后 2147483639%1.5=1431655759（这个数是要进行扩容） 次添加元素，为了防止溢出，此时会直接创建一个 1431655759+1 大小的数组，这样一直，每次添加一个元素，都只扩大一个范围。**

　　**⑤、第 Integer.MAX_VALUE - 7 次添加元素时，创建一个大小为 Integer.MAX_VALUE 的数组，在进行元素添加。**

　　**⑥、第 Integer.MAX_VALUE + 1 次添加元素时，抛出 OutOfMemoryError 异常。**

　　**注意：能向集合中添加 null 的，因为数组可以有 null 值存在。**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 Object[] obj = {null,1};
2 
3 ArrayList list = new ArrayList();
4 list.add(null);
5 list.add(1);
6 System.out.println(list.size());//2
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 5、删除元素



#### 　　①、根据索引删除元素

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public E remove(int index) {
 2         rangeCheck(index);//判断给定索引的范围，超过集合大小则抛出异常
 3 
 4         modCount++;
 5         E oldValue = elementData(index);//得到索引处的删除元素
 6 
 7         int numMoved = size - index - 1;
 8         if (numMoved > 0)//size-index-1 > 0 表示 0<= index < (size-1),即索引不是最后一个元素
 9             //通过 System.arraycopy()将数组elementData 的下标index+1之后长度为 numMoved的元素拷贝到从index开始的位置
10             System.arraycopy(elementData, index+1, elementData, index,
11                              numMoved);
12         elementData[--size] = null; //将数组最后一个元素置为 null，便于垃圾回收
13 
14         return oldValue;
15     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　remove(int index) 方法表示删除索引index处的元素，首先通过 rangeCheck(index) 方法判断给定索引的范围，超过集合大小则抛出异常；接着通过 System.arraycopy 方法对数组进行自身拷贝。关于这个方法的用法可以参考[这篇博客](http://www.cnblogs.com/ysocean/p/8616122.html#_label3)。



#### 　　②、直接删除指定元素

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public boolean remove(Object o) {
 2         if (o == null) {//如果删除的元素为null
 3             for (int index = 0; index < size; index++)
 4                 if (elementData[index] == null) {
 5                     fastRemove(index);
 6                     return true;
 7                 }
 8         } else {//不为null，通过equals方法判断对象是否相等
 9             for (int index = 0; index < size; index++)
10                 if (o.equals(elementData[index])) {
11                     fastRemove(index);
12                     return true;
13                 }
14         }
15         return false;
16     }
17 
18 
19     private void fastRemove(int index) {
20         modCount++;
21         int numMoved = size - index - 1;
22         if (numMoved > 0)
23             System.arraycopy(elementData, index+1, elementData, index,
24                              numMoved);
25         elementData[--size] = null; // 
26     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　remove(Object o)方法是删除第一次出现的该元素。然后通过System.arraycopy进行数组自身拷贝。

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 6、修改元素

　　通过调用 set(int index, E element) 方法在指定索引 index 处的元素替换为 element。并返回原数组的元素。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public E set(int index, E element) {
2         rangeCheck(index);//判断索引合法性
3 
4         E oldValue = elementData(index);//获得原数组指定索引的元素
5         elementData[index] = element;//将指定所引处的元素替换为 element
6         return oldValue;//返回原数组索引元素
7     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　通过调用 rangeCheck(index) 来检查索引合法性。

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　当索引为负数时，会抛出 java.lang.ArrayIndexOutOfBoundsException 异常。当索引大于集合长度时，会抛出 IndexOutOfBoundsException 异常。

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 7、查找元素



#### 　　①、根据索引查找元素

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public E get(int index) {
2         rangeCheck(index);
3 
4         return elementData(index);
5     }
```

　　同理，首先还是判断给定索引的合理性，然后直接返回处于该下标位置的数组元素。



#### 　　②、根据元素查找索引

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public int indexOf(Object o) {
 2         if (o == null) {
 3             for (int i = 0; i < size; i++)
 4                 if (elementData[i]==null)
 5                     return i;
 6         } else {
 7             for (int i = 0; i < size; i++)
 8                 if (o.equals(elementData[i]))
 9                     return i;
10         }
11         return -1;
12     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　注意：indexOf(Object o) 方法是返回第一次出现该元素的下标，如果没有则返回 -1。

　　还有 lastIndexOf(Object o) 方法是返回最后一次出现该元素的下标。

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 8、遍历集合



#### 　　①、普通 for 循环遍历

　　前面我们介绍查找元素时，知道可以通过get(int index)方法，根据索引查找元素，那么遍历同理：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 ArrayList list = new ArrayList();
2 list.add("a");
3 list.add("b");
4 list.add("c");
5 for(int i = 0 ; i < list.size() ; i++){
6     System.out.print(list.get(i)+" ");
7 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　②、迭代器 iterator

　　先看看具体用法：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 ArrayList<String> list = new ArrayList<>();
2 list.add("a");
3 list.add("b");
4 list.add("c");
5 Iterator<String> it = list.iterator();
6 while(it.hasNext()){
7     String str = it.next();
8     System.out.print(str+" ");
9 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　在介绍 ArrayList 时，我们知道该类实现了 List 接口，而 List 接口又继承了 Collection 接口，Collection 接口又继承了 Iterable 接口，该接口有个 Iterator<T> iterator() 方法，能获取 Iterator 对象，能用该对象进行集合遍历，为什么能用该对象进行集合遍历？我们再看看 ArrayList 类中的该方法实现：

```
1     public Iterator<E> iterator() {
2         return new Itr();
3     }
```

　　该方法是返回一个 Itr 对象，这个类是 ArrayList 的内部类。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     private class Itr implements Iterator<E> {
 2         int cursor;       //游标， 下一个要返回的元素的索引
 3         int lastRet = -1; // 返回最后一个元素的索引; 如果没有这样的话返回-1.
 4         int expectedModCount = modCount;
 5 
 6         //通过 cursor ！= size 判断是否还有下一个元素
 7         public boolean hasNext() {
 8             return cursor != size;
 9         }
10 
11         @SuppressWarnings("unchecked")
12         public E next() {
13             checkForComodification();//迭代器进行元素迭代时同时进行增加和删除操作，会抛出异常
14             int i = cursor;
15             if (i >= size)
16                 throw new NoSuchElementException();
17             Object[] elementData = ArrayList.this.elementData;
18             if (i >= elementData.length)
19                 throw new ConcurrentModificationException();
20             cursor = i + 1;//游标向后移动一位
21             return (E) elementData[lastRet = i];//返回索引为i处的元素，并将 lastRet赋值为i
22         }
23 
24         public void remove() {
25             if (lastRet < 0)
26                 throw new IllegalStateException();
27             checkForComodification();
28 
29             try {
30                 ArrayList.this.remove(lastRet);//调用ArrayList的remove方法删除元素
31                 cursor = lastRet;//游标指向删除元素的位置，本来是lastRet+1的，这里删除一个元素，然后游标就不变了
32                 lastRet = -1;//lastRet恢复默认值-1
33                 expectedModCount = modCount;//expectedModCount值和modCount同步，因为进行add和remove操作，modCount会加1
34             } catch (IndexOutOfBoundsException ex) {
35                 throw new ConcurrentModificationException();
36             }
37         }
38 
39         @Override
40         @SuppressWarnings("unchecked")
41         public void forEachRemaining(Consumer<? super E> consumer) {//便于进行forEach循环
42             Objects.requireNonNull(consumer);
43             final int size = ArrayList.this.size;
44             int i = cursor;
45             if (i >= size) {
46                 return;
47             }
48             final Object[] elementData = ArrayList.this.elementData;
49             if (i >= elementData.length) {
50                 throw new ConcurrentModificationException();
51             }
52             while (i != size && modCount == expectedModCount) {
53                 consumer.accept((E) elementData[i++]);
54             }
55             // update once at end of iteration to reduce heap write traffic
56             cursor = i;
57             lastRet = i - 1;
58             checkForComodification();
59         }
60 
61         //前面在新增元素add() 和 删除元素 remove() 时，我们可以看到 modCount++。修改set() 是没有的
62         //也就是说不能在迭代器进行元素迭代时进行增加和删除操作，否则抛出异常
63         final void checkForComodification() {
64             if (modCount != expectedModCount)
65                 throw new ConcurrentModificationException();
66         }
67     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　注意在进行 next() 方法调用的时候，会进行 checkForComodification() 调用，该方法表示迭代器进行元素迭代时，如果同时进行增加和删除操作，会抛出 ConcurrentModificationException 异常。比如：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 ArrayList<String> list = new ArrayList<>();
 2 list.add("a");
 3 list.add("b");
 4 list.add("c");
 5 Iterator<String> it = list.iterator();
 6 while(it.hasNext()){
 7     String str = it.next();
 8     System.out.print(str+" ");
 9     list.remove(str);//集合遍历时进行删除或者新增操作，都会抛出 ConcurrentModificationException 异常
10     //list.add(str);
11     list.set(0, str);//修改操作不会造成异常
12 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180327132712244-110170583.png)

　　解决办法是不调用 ArrayList.remove() 方法，转而调用 迭代器的 remove() 方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 Iterator<String> it = list.iterator();
2 while(it.hasNext()){
3     String str = it.next();
4     System.out.print(str+" ");
5     //list.remove(str);//集合遍历时进行删除或者新增操作，都会抛出 ConcurrentModificationException 异常
6     it.remove();
7 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**注意：迭代器只能向后遍历，不能向前遍历，能够删除元素，但是不能新增元素。**



#### 　　③、迭代器的变种 forEach

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 ArrayList<String> list = new ArrayList<>();
2 list.add("a");
3 list.add("b");
4 list.add("c");
5 for(String str : list){
6     System.out.print(str + " ");
7 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这种语法可以看成是 JDK 的一种语法糖，通过反编译 class 文件，我们可以看到生成的 java 文件，其具体实现还是通过调用 Iterator 迭代器进行遍历的。如下：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1         ArrayList list = new ArrayList();
2         list.add("a");
3         list.add("b");
4         list.add("c");
5         String str;
6         for (Iterator iterator1 = list.iterator(); iterator1.hasNext(); System.out.print((new StringBuilder(String.valueOf(str))).append(" ").toString()))
7             str = (String)iterator1.next();
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　④、迭代器 ListIterator

　　还是先看看具体用法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 ArrayList<String> list = new ArrayList<>();
 2 list.add("a");
 3 list.add("b");
 4 list.add("c");
 5 ListIterator<String> listIt = list.listIterator();
 6 
 7 //向后遍历
 8 while(listIt.hasNext()){
 9     System.out.print(listIt.next()+" ");//a b c
10 }
11 
12 //向后前遍历,此时由于上面进行了向后遍历，游标已经指向了最后一个元素，所以此处向前遍历能有值
13 while(listIt.hasPrevious()){
14     System.out.print(listIt.previous()+" ");//c b a
15 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　还能一边遍历，一边进行新增或者删除操作：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 ArrayList<String> list = new ArrayList<>();
 2 list.add("a");
 3 list.add("b");
 4 list.add("c");
 5 ListIterator<String> listIt = list.listIterator();
 6 
 7 //向后遍历
 8 while(listIt.hasNext()){
 9     System.out.print(listIt.next()+" ");//a b c
10     listIt.add("1");//在每一个元素后面增加一个元素 "1"
11 }
12 
13 //向后前遍历,此时由于上面进行了向后遍历，游标已经指向了最后一个元素，所以此处向前遍历能有值
14 while(listIt.hasPrevious()){
15     System.out.print(listIt.previous()+" ");//1 c 1 b 1 a 
16 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**也就是说相比于 Iterator 迭代器，这里的 ListIterator 多出了能向前迭代，以及能够新增元素**。下面我们看看具体实现：

　　对于 Iterator 迭代器，我们查看 JDK 源码，发现还有 ListIterator 接口继承了 Iterator:

```
public interface ListIterator<E> extends Iterator<E> 
```

　　该接口有如下方法：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180327134149223-1777742214.png)

　　我们看在 ArrayList 类中，有如下方法可以获得 ListIterator 接口：

```
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }
```

　　这里的 ListItr 也是一个内部类。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //注意 内部类 ListItr 继承了另一个内部类 Itr
 2     private class ListItr extends Itr implements ListIterator<E> {
 3         ListItr(int index) {//构造函数，进行游标初始化
 4             super();
 5             cursor = index;
 6         }
 7 
 8         public boolean hasPrevious() {//判断是否有上一个元素
 9             return cursor != 0;
10         }
11 
12         public int nextIndex() {//返回下一个元素的索引
13             return cursor;
14         }
15 
16         public int previousIndex() {//返回上一个元素的索引
17             return cursor - 1;
18         }
19 
20         //该方法获取当前索引的上一个元素
21         @SuppressWarnings("unchecked")
22         public E previous() {
23             checkForComodification();//迭代器进行元素迭代时同时进行增加和删除操作，会抛出异常
24             int i = cursor - 1;
25             if (i < 0)
26                 throw new NoSuchElementException();
27             Object[] elementData = ArrayList.this.elementData;
28             if (i >= elementData.length)
29                 throw new ConcurrentModificationException();
30             cursor = i;//游标指向上一个元素
31             return (E) elementData[lastRet = i];//返回上一个元素的值
32         }
33 
34         
35         public void set(E e) {
36             if (lastRet < 0)
37                 throw new IllegalStateException();
38             checkForComodification();
39 
40             try {
41                 ArrayList.this.set(lastRet, e);
42             } catch (IndexOutOfBoundsException ex) {
43                 throw new ConcurrentModificationException();
44             }
45         }
46 
47         //相比于迭代器 Iterator ，这里多了一个新增操作
48         public void add(E e) {
49             checkForComodification();
50 
51             try {
52                 int i = cursor;
53                 ArrayList.this.add(i, e);
54                 cursor = i + 1;
55                 lastRet = -1;
56                 expectedModCount = modCount;
57             } catch (IndexOutOfBoundsException ex) {
58                 throw new ConcurrentModificationException();
59             }
60         }
61     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

###  9、SubList

　　在 ArrayList 中有这样一个方法：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public List<E> subList(int fromIndex, int toIndex) {
2         subListRangeCheck(fromIndex, toIndex, size);
3         return new SubList(this, 0, fromIndex, toIndex);
4     }
```

　　作用是返回从 fromIndex(包括) 开始的下标，到 toIndex(不包括) 结束的下标之间的元素**视图**。如下：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 ArrayList<String> list = new ArrayList<>();
2 list.add("a");
3 list.add("b");
4 list.add("c");
5 
6 List<String> subList = list.subList(0, 1);
7 for(String str : subList){
8     System.out.print(str + " ");//a
9 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这里出现了 SubList 类，这也是 ArrayList 中的一个内部类。

　　**注意：返回的是原集合的视图，也就是说，如果对 subList 出来的集合进行修改或新增操作，那么原始集合也会发生同样的操作。**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 ArrayList<String> list = new ArrayList<>();
 2 list.add("a");
 3 list.add("b");
 4 list.add("c");
 5 
 6 List<String> subList = list.subList(0, 1);
 7 for(String str : subList){
 8     System.out.print(str + " ");//a
 9 }
10 subList.add("d");
11 System.out.println(subList.size());//2
12 System.out.println(list.size());//4,原始集合长度也增加了
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　想要独立出来一个集合，解决办法如下：

```
List<String> subList = new ArrayList<>(list.subList(0, 1));
```

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 10、size()

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
    public int size() {
        return size;
    }
```

　　注意：返回集合的长度，而不是数组的长度，这里的 size 就是定义的全局变量。

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 11、isEmpty()

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public boolean isEmpty() {
2         return size == 0;
3     }
```

　　返回 size == 0 的结果。

[回到顶部](https://www.cnblogs.com/ysocean/p/8622264.html#_labelTop)

### 12、trimToSize()

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public void trimToSize() {
2         modCount++;
3         if (size < elementData.length) {
4             elementData = Arrays.copyOf(elementData, size);
5         }
6     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　该方法用于回收多余的内存。也就是说一旦我们确定集合不在添加多余的元素之后，调用 trimToSize() 方法会将实现集合的数组大小刚好调整为集合元素的大小。

　　注意：该方法会花时间来复制数组元素，所以应该在确定不会添加元素之后在调用。

参考文档：https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html#



# [java.util.LinkedList 类](https://www.cnblogs.com/ysocean/p/8657850.html)



**目录**

- [1、LinkedList 定义](https://www.cnblogs.com/ysocean/p/8657850.html#_label0)
- [2、字段属性](https://www.cnblogs.com/ysocean/p/8657850.html#_label1)
- [3、构造函数 ](https://www.cnblogs.com/ysocean/p/8657850.html#_label2)
- 4、添加元素
  - [　　①、addFirst(E e)](https://www.cnblogs.com/ysocean/p/8657850.html#_label3_0)
  - [　　②、addLast(E e)和add(E e)](https://www.cnblogs.com/ysocean/p/8657850.html#_label3_1)
  - [　　③、add(int index, E element)](https://www.cnblogs.com/ysocean/p/8657850.html#_label3_2)
  - [　　④、addAll(Collection c)](https://www.cnblogs.com/ysocean/p/8657850.html#_label3_3)
- 5、删除元素
  - [　　①、remove()和removeFirst()](https://www.cnblogs.com/ysocean/p/8657850.html#_label4_0)
  - [　　②、removeLast()](https://www.cnblogs.com/ysocean/p/8657850.html#_label4_1)
  - [　　③、remove(int index)](https://www.cnblogs.com/ysocean/p/8657850.html#_label4_2)
  - [　　④、remove(Object o)](https://www.cnblogs.com/ysocean/p/8657850.html#_label4_3)
- [ 6、修改元素](https://www.cnblogs.com/ysocean/p/8657850.html#_label5)
- 7、查找元素
  - [　　①、getFirst()](https://www.cnblogs.com/ysocean/p/8657850.html#_label6_0)
  - [　　②、getLast()](https://www.cnblogs.com/ysocean/p/8657850.html#_label6_1)
  - [　　③、get(int index)](https://www.cnblogs.com/ysocean/p/8657850.html#_label6_2)
  - [　　④、indexOf(Object o)](https://www.cnblogs.com/ysocean/p/8657850.html#_label6_3)
- 8、遍历集合
  - [　　①、普通 for 循环](https://www.cnblogs.com/ysocean/p/8657850.html#_label7_0)
  - [　　②、迭代器](https://www.cnblogs.com/ysocean/p/8657850.html#_label7_1)
- [9、迭代器和for循环效率差异](https://www.cnblogs.com/ysocean/p/8657850.html#_label8)

 

------

　　上一篇博客我们介绍了List集合的一种典型实现 [ArrayList](https://www.cnblogs.com/ysocean/p/8622264.html)，我们知道 ArrayList 是由数组构成的，本篇博客我们介绍 List 集合的另一种典型实现 LinkedList，这是一个由链表构成的数组，关于链表的介绍，在[这篇博客中](http://www.cnblogs.com/ysocean/p/7928988.html) 我们也详细介绍过，本篇博客我们将介绍 LinkedList 是如何实现的。

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 1、LinkedList 定义

　　**LinkedList 是一个用链表实现的集合，元素有序且可以重复。**

```
1 public class LinkedList<E>
2     extends AbstractSequentialList<E>
3     implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

**![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180329133938645-733252704.png)**

　　和 ArrayList 集合一样，LinkedList 集合也实现了Cloneable接口和Serializable接口，分别用来支持克隆以及支持序列化。List 接口也不用多说，定义了一套 List 集合类型的方法规范。

　　注意，相对于 ArrayList 集合，LinkedList 集合多实现了一个 **Deque** 接口，这是一个双向队列接口，双向队列就是两端都可以进行增加和删除操作。

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 2、字段属性

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    //链表元素（节点）的个数
    transient int size = 0;

    /**
     *指向第一个节点的指针
     */
    transient Node<E> first;

    /**
     *指向最后一个节点的指针
     */
    transient Node<E> last;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　注意这里出现了一个 Node 类，这是 LinkedList 类中的一个内部类，其中每一个元素就代表一个 Node 类对象，LinkedList 集合就是由许多个 Node 对象类似于手拉着手构成。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     private static class Node<E> {
 2         E item;//实际存储的元素
 3         Node<E> next;//指向上一个节点的引用
 4         Node<E> prev;//指向下一个节点的引用
 5 
 6         //构造函数
 7         Node(Node<E> prev, E element, Node<E> next) {
 8             this.item = element;
 9             this.next = next;
10             this.prev = prev;
11         }
12     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　如下图所示：

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180402091402743-458763981.png)

　　上图的 LinkedList 是有四个元素，也就是由 4 个 Node 对象组成，size=4，head 指向第一个elementA,tail指向最后一个节点elementD。

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 3、构造函数 

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    public LinkedList() {
    }
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　LinkedList 有两个构造函数，第一个是默认的空的构造函数，第二个是将已有元素的集合Collection 的实例添加到 LinkedList 中，调用的是 addAll() 方法，这个方法下面我们会介绍。

　　注意：LinkedList 是没有初始化链表大小的构造函数，因为链表不像数组，一个定义好的数组是必须要有确定的大小，然后去分配内存空间，而链表不一样，它没有确定的大小，通过指针的移动来指向下一个内存地址的分配。

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 4、添加元素



#### 　　①、addFirst(E e)

　　**将指定元素添加到链表头**

　　**![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180330200701990-1309034444.png)**

 

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //将指定的元素附加到链表头节点
 2     public void addFirst(E e) {
 3         linkFirst(e);
 4     }
 5     private void linkFirst(E e) {
 6         final Node<E> f = first;//将头节点赋值给 f
 7         final Node<E> newNode = new Node<>(null, e, f);//将指定元素构造成一个新节点，此节点的指向下一个节点的引用为头节点
 8         first = newNode;//将新节点设为头节点，那么原先的头节点 f 变为第二个节点
 9         if (f == null)//如果第二个节点为空，也就是原先链表是空
10             last = newNode;//将这个新节点也设为尾节点（前面已经设为头节点了）
11         else
12             f.prev = newNode;//将原先的头节点的上一个节点指向新节点
13         size++;//节点数加1
14         modCount++;//和ArrayList中一样，iterator和listIterator方法返回的迭代器和列表迭代器实现使用。
15     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　②、addLast(E e)和add(E e)

　　**将指定元素添加到链表尾**

**![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180330200726125-2016092250.png)**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //将元素添加到链表末尾
 2     public void addLast(E e) {
 3         linkLast(e);
 4     }
 5     //将元素添加到链表末尾
 6     public boolean add(E e) {
 7         linkLast(e);
 8         return true;
 9     }
10     void linkLast(E e) {
11         final Node<E> l = last;//将l设为尾节点
12         final Node<E> newNode = new Node<>(l, e, null);//构造一个新节点，节点上一个节点引用指向尾节点l
13         last = newNode;//将尾节点设为创建的新节点
14         if (l == null)//如果尾节点为空，表示原先链表为空
15             first = newNode;//将头节点设为新创建的节点（尾节点也是新创建的节点）
16         else
17             l.next = newNode;//将原来尾节点下一个节点的引用指向新节点
18         size++;//节点数加1
19         modCount++;//和ArrayList中一样，iterator和listIterator方法返回的迭代器和列表迭代器实现使用。
20     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　③、add(int index, E element)

　　**将指定的元素插入此列表中的指定位置**

　**![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180330201245271-1936601304.png)**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    //将指定的元素插入此列表中的指定位置
    public void add(int index, E element) {
        //判断索引 index >= 0 && index <= size中时抛出IndexOutOfBoundsException异常
        checkPositionIndex(index);

        if (index == size)//如果索引值等于链表大小
            linkLast(element);//将节点插入到尾节点
        else
            linkBefore(element, node(index));
    }
    void linkLast(E e) {
        final Node<E> l = last;//将l设为尾节点
        final Node<E> newNode = new Node<>(l, e, null);//构造一个新节点，节点上一个节点引用指向尾节点l
        last = newNode;//将尾节点设为创建的新节点
        if (l == null)//如果尾节点为空，表示原先链表为空
            first = newNode;//将头节点设为新创建的节点（尾节点也是新创建的节点）
        else
            l.next = newNode;//将原来尾节点下一个节点的引用指向新节点
        size++;//节点数加1
        modCount++;//和ArrayList中一样，iterator和listIterator方法返回的迭代器和列表迭代器实现使用。
    }
    Node<E> node(int index) {
        if (index < (size >> 1)) {//如果插入的索引在前半部分
            Node<E> x = first;//设x为头节点
            for (int i = 0; i < index; i++)//从开始节点到插入节点索引之间的所有节点向后移动一位
                x = x.next;
            return x;
        } else {//如果插入节点位置在后半部分
            Node<E> x = last;//将x设为最后一个节点
            for (int i = size - 1; i > index; i--)//从最后节点到插入节点的索引位置之间的所有节点向前移动一位
                x = x.prev;
            return x;
        }
    }
    void linkBefore(E e, Node<E> succ) {
        final Node<E> pred = succ.prev;//将pred设为插入节点的上一个节点
        final Node<E> newNode = new Node<>(pred, e, succ);//将新节点的上引用设为pred,下引用设为succ
        succ.prev = newNode;//succ的上一个节点的引用设为新节点
        if (pred == null)//如果插入节点的上一个节点引用为空
            first = newNode;//新节点就是头节点
        else
            pred.next = newNode;//插入节点的下一个节点引用设为新节点
        size++;
        modCount++;
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　④、addAll(Collection<? extends E> c)

　　**按照指定集合的迭代器返回的顺序，将指定集合中的所有元素追加到此列表的末尾**

　　此方法还有一个 addAll(int index, Collection<? extends E> c)，将集合 c 中所有元素插入到指定索引的位置。其实 

　　　　**addAll(Collection c) == addAll(size, Collection c)**

　　源码如下：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //按照指定集合的迭代器返回的顺序，将指定集合中的所有元素追加到此列表的末尾。
 2     public boolean addAll(Collection<? extends E> c) {
 3         return addAll(size, c);
 4     }
 5     //将集合 c 中所有元素插入到指定索引的位置。
 6     public boolean addAll(int index, Collection<? extends E> c) {
 7         //判断索引 index >= 0 && index <= size中时抛出IndexOutOfBoundsException异常
 8         checkPositionIndex(index);
 9 
10         Object[] a = c.toArray();//将集合转换成一个 Object 类型的数组
11         int numNew = a.length;
12         if (numNew == 0)//如果添加的集合为空，直接返回false
13             return false;
14 
15         Node<E> pred, succ;
16         if (index == size) {//如果插入的位置等于链表的长度，就是将原集合元素附加到链表的末尾
17             succ = null;
18             pred = last;
19         } else {
20             succ = node(index);
21             pred = succ.prev;
22         }
23 
24         for (Object o : a) {//遍历要插入的元素
25             @SuppressWarnings("unchecked") E e = (E) o;
26             Node<E> newNode = new Node<>(pred, e, null);
27             if (pred == null)
28                 first = newNode;
29             else
30                 pred.next = newNode;
31             pred = newNode;
32         }
33 
34         if (succ == null) {
35             last = pred;
36         } else {
37             pred.next = succ;
38             succ.prev = pred;
39         }
40 
41         size += numNew;
42         modCount++;
43         return true;
44     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　看到上面向 LinkedList 集合中添加元素的各种方式，我们发现LinkedList 每次添加元素只是改变元素的上一个指针引用和下一个指针引用，而且没有扩容。，对比于 ArrayList ，需要扩容，而且在中间插入元素时，后面的所有元素都要移动一位，两者插入元素时的效率差异很大，下一篇博客会对这两者的效率，以及何种情况选择何种集合进行分析。

　　还有，每次进行添加操作，都有modCount++ 的操作，

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 5、删除元素

　　删除元素和添加元素一样，也是通过更改指向上一个节点和指向下一个节点的引用即可，这里就不作图形展示了。



#### 　　①、remove()和removeFirst()

　　**从此列表中移除并返回第一个元素**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //从此列表中移除并返回第一个元素
 2     public E remove() {
 3         return removeFirst();
 4     }
 5     //从此列表中移除并返回第一个元素
 6     public E removeFirst() {
 7         final Node<E> f = first;//f设为头结点
 8         if (f == null)
 9             throw new NoSuchElementException();//如果头结点为空，则抛出异常
10         return unlinkFirst(f);
11     }
12     private E unlinkFirst(Node<E> f) {
13         // assert f == first && f != null;
14         final E element = f.item;
15         final Node<E> next = f.next;//next 为头结点的下一个节点
16         f.item = null;
17         f.next = null; // 将节点的元素以及引用都设为 null，便于垃圾回收
18         first = next; //修改头结点为第二个节点
19         if (next == null)//如果第二个节点为空（当前链表只存在第一个元素）
20             last = null;//那么尾节点也置为 null
21         else
22             next.prev = null;//如果第二个节点不为空，那么将第二个节点的上一个引用置为 null
23         size--;
24         modCount++;
25         return element;
26     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　②、removeLast()

　　**从该列表中删除并返回最后一个元素**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //从该列表中删除并返回最后一个元素
 2     public E removeLast() {
 3         final Node<E> l = last;
 4         if (l == null)//如果尾节点为空，表示当前集合为空，抛出异常
 5             throw new NoSuchElementException();
 6         return unlinkLast(l);
 7     }
 8     
 9     private E unlinkLast(Node<E> l) {
10         // assert l == last && l != null;
11         final E element = l.item;
12         final Node<E> prev = l.prev;
13         l.item = null;
14         l.prev = null; //将节点的元素以及引用都设为 null，便于垃圾回收
15         last = prev;//尾节点为倒数第二个节点
16         if (prev == null)//如果倒数第二个节点为null
17             first = null;//那么将节点也置为 null
18         else
19             prev.next = null;//如果倒数第二个节点不为空，那么将倒数第二个节点的下一个引用置为 null
20         size--;
21         modCount++;
22         return element;
23     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　③、remove(int index)

　　**删除此列表中指定位置的元素**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //删除此列表中指定位置的元素
 2     public E remove(int index) {
 3         //判断索引 index >= 0 && index <= size中时抛出IndexOutOfBoundsException异常
 4         checkElementIndex(index);
 5         return unlink(node(index));
 6     }
 7     E unlink(Node<E> x) {
 8         // assert x != null;
 9         final E element = x.item;
10         final Node<E> next = x.next;
11         final Node<E> prev = x.prev;
12 
13         if (prev == null) {//如果删除节点位置的上一个节点引用为null（表示删除第一个元素）
14             first = next;//将头结点置为第一个元素的下一个节点
15         } else {//如果删除节点位置的上一个节点引用不为null
16             prev.next = next;//将删除节点的上一个节点的下一个节点引用指向删除节点的下一个节点（去掉删除节点）
17             x.prev = null;//删除节点的上一个节点引用置为null
18         }
19 
20         if (next == null) {//如果删除节点的下一个节点引用为null（表示删除最后一个节点）
21             last = prev;//将尾节点置为删除节点的上一个节点
22         } else {//不是删除尾节点
23             next.prev = prev;//将删除节点的下一个节点的上一个节点的引用指向删除节点的上一个节点
24             x.next = null;//将删除节点的下一个节点引用置为null
25         }
26 
27         x.item = null;//删除节点内容置为null，便于垃圾回收
28         size--;
29         modCount++;
30         return element;
31     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　④、remove(Object o)

　　**如果存在，则从该列表中删除指定元素的第一次出现**

　　此方法本质上和 remove(int index) 没多大区别，通过循环判断元素进行删除，需要注意的是，是删除第一次出现的元素，不是所有的。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public boolean remove(Object o) {
 2         if (o == null) {
 3             for (Node<E> x = first; x != null; x = x.next) {
 4                 if (x.item == null) {
 5                     unlink(x);
 6                     return true;
 7                 }
 8             }
 9         } else {
10             for (Node<E> x = first; x != null; x = x.next) {
11                 if (o.equals(x.item)) {
12                     unlink(x);
13                     return true;
14                 }
15             }
16         }
17         return false;
18     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

###  6、修改元素

　　通过调用 set(int index, E element) 方法，用指定的元素替换此列表中指定位置的元素。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    public E set(int index, E element) {
        //判断索引 index >= 0 && index <= size中时抛出IndexOutOfBoundsException异常
        checkElementIndex(index);
        Node<E> x = node(index);//获取指定索引处的元素
        E oldVal = x.item;
        x.item = element;//将指定位置的元素替换成要修改的元素
        return oldVal;//返回指定索引位置原来的元素
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这里主要是通过 node(index) 方法获取指定索引位置的节点，然后修改此节点位置的元素即可。

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 7、查找元素



#### 　　①、getFirst()

　　**返回此列表中的第一个元素**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public E getFirst() {
2         final Node<E> f = first;
3         if (f == null)
4             throw new NoSuchElementException();
5         return f.item;
6     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　②、getLast()

　　**返回此列表中的最后一个元素**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public E getLast() {
2         final Node<E> l = last;
3         if (l == null)
4             throw new NoSuchElementException();
5         return l.item;
6     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)



#### 　　③、get(int index)

　　**返回指定索引处的元素**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
1     public E get(int index) {
2         checkElementIndex(index);
3         return node(index).item;
4     }
```



#### 　　④、indexOf(Object o)

　　**返回此列表中指定元素第一次出现的索引，如果此列表不包含元素，则返回-1。**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //返回此列表中指定元素第一次出现的索引，如果此列表不包含元素，则返回-1。
 2     public int indexOf(Object o) {
 3         int index = 0;
 4         if (o == null) {//如果查找的元素为null(LinkedList可以允许null值)
 5             for (Node<E> x = first; x != null; x = x.next) {//从头结点开始不断向下一个节点进行遍历
 6                 if (x.item == null)
 7                     return index;
 8                 index++;
 9             }
10         } else {//如果查找的元素不为null
11             for (Node<E> x = first; x != null; x = x.next) {
12                 if (o.equals(x.item))
13                     return index;
14                 index++;
15             }
16         }
17         return -1;//找不到返回-1
18     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 8、遍历集合



#### 　　①、普通 for 循环

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A");
linkedList.add("B");
linkedList.add("C");
linkedList.add("D");
for(int i = 0 ; i < linkedList.size() ; i++){
    System.out.print(linkedList.get(i)+" ");//A B C D
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　代码很简单，我们就利用 LinkedList 的 get(int index) 方法，遍历出所有的元素。

　　但是需要注意的是， get(int index) 方法每次都要遍历该索引之前的所有元素，这句话这么理解：

　　比如上面的一个 LinkedList 集合，我放入了 A,B,C,D是个元素。总共需要四次遍历：

　　第一次遍历打印 A：只需遍历一次。

　　第二次遍历打印 B：需要先找到 A，然后再找到 B 打印。

　　第三次遍历打印 C：需要先找到 A，然后找到 B，最后找到 C 打印。

　　第四次遍历打印 D：需要先找到 A，然后找到 B，然后找到 C，最后找到 D。

　　这样如果集合元素很多，越查找到后面（当然此处的get方法进行了优化，查找前半部分从前面开始遍历，查找后半部分从后面开始遍历，但是需要的时间还是很多）花费的时间越多。那么如何改进呢？



#### 　　②、迭代器

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 LinkedList<String> linkedList = new LinkedList<>();
 2 linkedList.add("A");
 3 linkedList.add("B");
 4 linkedList.add("C");
 5 linkedList.add("D");
 6 
 7 
 8 Iterator<String> listIt = linkedList.listIterator();
 9 while(listIt.hasNext()){
10     System.out.print(listIt.next()+" ");//A B C D
11 }
12 
13 //通过适配器模式实现的接口，作用是倒叙打印链表
14 Iterator<String> it = linkedList.descendingIterator();
15 while(it.hasNext()){
16     System.out.print(it.next()+" ");//D C B A
17 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　在 LinkedList 集合中也有一个内部类 ListItr，方法实现大体上也差不多，通过移动游标指向每一次要遍历的元素，不用在遍历某个元素之前都要从头开始。其方法实现也比较简单：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
  1     public ListIterator<E> listIterator(int index) {
  2         checkPositionIndex(index);
  3         return new ListItr(index);
  4     }
  5 
  6     private class ListItr implements ListIterator<E> {
  7         private Node<E> lastReturned;
  8         private Node<E> next;
  9         private int nextIndex;
 10         private int expectedModCount = modCount;
 11 
 12         ListItr(int index) {
 13             // assert isPositionIndex(index);
 14             next = (index == size) ? null : node(index);
 15             nextIndex = index;
 16         }
 17 
 18         public boolean hasNext() {
 19             return nextIndex < size;
 20         }
 21 
 22         public E next() {
 23             checkForComodification();
 24             if (!hasNext())
 25                 throw new NoSuchElementException();
 26 
 27             lastReturned = next;
 28             next = next.next;
 29             nextIndex++;
 30             return lastReturned.item;
 31         }
 32 
 33         public boolean hasPrevious() {
 34             return nextIndex > 0;
 35         }
 36 
 37         public E previous() {
 38             checkForComodification();
 39             if (!hasPrevious())
 40                 throw new NoSuchElementException();
 41 
 42             lastReturned = next = (next == null) ? last : next.prev;
 43             nextIndex--;
 44             return lastReturned.item;
 45         }
 46 
 47         public int nextIndex() {
 48             return nextIndex;
 49         }
 50 
 51         public int previousIndex() {
 52             return nextIndex - 1;
 53         }
 54 
 55         public void remove() {
 56             checkForComodification();
 57             if (lastReturned == null)
 58                 throw new IllegalStateException();
 59 
 60             Node<E> lastNext = lastReturned.next;
 61             unlink(lastReturned);
 62             if (next == lastReturned)
 63                 next = lastNext;
 64             else
 65                 nextIndex--;
 66             lastReturned = null;
 67             expectedModCount++;
 68         }
 69 
 70         public void set(E e) {
 71             if (lastReturned == null)
 72                 throw new IllegalStateException();
 73             checkForComodification();
 74             lastReturned.item = e;
 75         }
 76 
 77         public void add(E e) {
 78             checkForComodification();
 79             lastReturned = null;
 80             if (next == null)
 81                 linkLast(e);
 82             else
 83                 linkBefore(e, next);
 84             nextIndex++;
 85             expectedModCount++;
 86         }
 87 
 88         public void forEachRemaining(Consumer<? super E> action) {
 89             Objects.requireNonNull(action);
 90             while (modCount == expectedModCount && nextIndex < size) {
 91                 action.accept(next.item);
 92                 lastReturned = next;
 93                 next = next.next;
 94                 nextIndex++;
 95             }
 96             checkForComodification();
 97         }
 98 
 99         final void checkForComodification() {
100             if (modCount != expectedModCount)
101                 throw new ConcurrentModificationException();
102         }
103     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这里需要重点注意的是 modCount 字段，前面我们在增加和删除元素的时候，都会进行自增操作 modCount，这是因为如果想一边迭代，一边用集合自带的方法进行删除或者新增操作，都会抛出异常。（使用迭代器的增删方法不会抛异常）

```
final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

　　比如：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 LinkedList<String> linkedList = new LinkedList<>();
 2 linkedList.add("A");
 3 linkedList.add("B");
 4 linkedList.add("C");
 5 linkedList.add("D");
 6 
 7 
 8 Iterator<String> listIt = linkedList.listIterator();
 9 while(listIt.hasNext()){
10     System.out.print(listIt.next()+" ");//A B C D
11     //linkedList.remove();//此处会抛出异常
12     listIt.remove();//这样可以进行删除操作
13 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**迭代器的另一种形式就是使用 foreach 循环，底层实现也是使用的迭代器，这里我们就不做介绍了。**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A");
linkedList.add("B");
linkedList.add("C");
linkedList.add("D");
for(String str : linkedList){
    System.out.print(str + "");
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8657850.html#_labelTop)

### 9、迭代器和for循环效率差异

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 LinkedList<Integer> linkedList = new LinkedList<>();
 2 for(int i = 0 ; i < 10000 ; i++){//向链表中添加一万个元素
 3     linkedList.add(i);
 4 }
 5 long beginTimeFor = System.currentTimeMillis();
 6 for(int i = 0 ; i < 10000 ; i++){
 7     System.out.print(linkedList.get(i));
 8 }
 9 long endTimeFor = System.currentTimeMillis();
10 System.out.println("使用普通for循环遍历10000个元素需要的时间："+ (endTimeFor - beginTimeFor));
11 
12 
13 long beginTimeIte = System.currentTimeMillis();
14 Iterator<Integer> it = linkedList.listIterator();
15 while(it.hasNext()){
16     System.out.print(it.next()+" ");
17 }
18 long endTimeIte = System.currentTimeMillis();
19 System.out.println("使用迭代器遍历10000个元素需要的时间："+ (endTimeIte - beginTimeIte));
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　打印结果为：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180330142506666-819006901.png)

　　一万个元素两者之间都相差一倍多的时间，如果是十万，百万个元素，那么两者之间相差的速度会越来越大。下面通过图形来解释：

　　**普通for循环：每次遍历一个索引的元素之前，都要访问之间所有的索引。**

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180330144334457-1162312538.png)

　　**迭代器：每次访问一个元素后，都会用游标记录当前访问元素的位置，遍历一个元素，记录一个位置。**

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180330150426561-157314787.png)

 

参考文档：https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html#



# [java.util.HashMap 类](https://www.cnblogs.com/ysocean/p/8711071.html)



**目录**

- [1、哈希表](https://www.cnblogs.com/ysocean/p/8711071.html#_label0)
- [2、什么是 HashMap？](https://www.cnblogs.com/ysocean/p/8711071.html#_label1)
- [3、HashMap定义](https://www.cnblogs.com/ysocean/p/8711071.html#_label2)
- [4、字段属性](https://www.cnblogs.com/ysocean/p/8711071.html#_label3)
- [5、构造函数](https://www.cnblogs.com/ysocean/p/8711071.html#_label4)
- [6、确定哈希桶数组索引位置](https://www.cnblogs.com/ysocean/p/8711071.html#_label5)
- [7、添加元素](https://www.cnblogs.com/ysocean/p/8711071.html#_label6)
- [8、扩容机制](https://www.cnblogs.com/ysocean/p/8711071.html#_label7)
- [9、删除元素](https://www.cnblogs.com/ysocean/p/8711071.html#_label8)
- [10、查找元素](https://www.cnblogs.com/ysocean/p/8711071.html#_label9)
- [11、遍历元素](https://www.cnblogs.com/ysocean/p/8711071.html#_label10)
- [12、总结](https://www.cnblogs.com/ysocean/p/8711071.html#_label11)

 

------

　　本篇博客我们来介绍在 JDK1.8 中 HashMap 的源码实现，这也是最常用的一个集合。但是在介绍 HashMap 之前，我们先介绍什么是 Hash表。

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 1、哈希表

　　Hash表也称为散列表，也有直接译作哈希表，Hash表是一种根据关键字值（key - value）而直接进行访问的数据结构。也就是说它通过把关键码值映射到表中的一个位置来访问记录，以此来加快查找的速度。在链表、数组等数据结构中，查找某个关键字，通常要遍历整个数据结构，也就是O(N)的时间级，但是对于哈希表来说，只是O(1)的时间级。

　　比如对于前面我们讲解的 [ArrayList](http://www.cnblogs.com/ysocean/p/8622264.html) 集合和 [LinkedList](http://www.cnblogs.com/ysocean/p/8657850.html) ，如果我们要查找这两个集合中的某个元素，通常是通过遍历整个集合，需要**O(N)**的时间级。

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180403234003772-1775417648.png)

　　如果是哈希表，它是通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做**散列函数**，存放记录的数组叫做**散列表，**只需要**O(1)**的时间级。

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180403235818954-222956412.png)

　　①、存放在哈希表中的数据是key-value 键值对，比如存放哈希表的数据为:

　　**{Key1-Value1,Key2-Value2,Key3-Value3,Key4-Value4,Key5-Value5,Key6-Value6}**

　　如果我们想查找是否存在键值对 Key3-Value3，首先通过 Key3 经过散列函数，得到值 k3，然后通过 k3 和散列表对应的值找到是 Value3。

　　②、当然也有可能存放哈希表的值只是 Value1,Value2,Value3这种类型：

　　**{Value1,Value2,Value3,Value4,Value5,Value6}**

　　这时候我们可以假设 Value1 是等于 Key1的，也就是{Value1-Value1,Value2-Value2,Value3-Value3,Value4-Value4,Value5-Value5,Value6-Value6}可以将 Value1经过散列函数转换成与散列表对应的值。

```
大家都用过汉语字典吧，汉语字典的优点是我们可以通过前面的拼音目录快速定位到所要查找的汉字。当给定我们某个汉字时，大脑会自动将汉字转换成拼音（如果我们认识，不认识可以通过偏旁部首），这个转换的过程我们可以看成是一个散列函数，之后在根据转换得到的拼音找到该字所在的页码，从而找到该汉字。
```

 　汉语字典是哈希表的典型实现，但是我们仔细思考，会发现这样几个问题？

　　**①、为什么要有散列函数？**

　　**②、多个 key 通过散列函数会得到相同的值，这时候怎么办？**

　　对于第一个问题，散列函数的存在能够帮助我们更快的确定key和value的映射关系，试想一下，如果没有汉字和拼音的转换规则（或者汉字和偏旁部首的），给你一个汉字，你该如何从字典中找到该汉字？我想除了遍历整部字典，你没有什么更好的办法。

　　对于第二个问题，多个 key 通过散列函数得到相同的值，这其实也是哈希表最大的问题——**冲突**。比如同音字汉字，我们得到的拼音就会是相同的，那么我们该如何在字典中存放同音字汉字呢？有两种做法：

　　第一种是**开放地址法**，当我们遇到冲突了，这时候通过另一种函数再计算一遍，得到相应的映射关系。比如对于汉语字典，一个字 “余”，拼音是“yu”，我们将其放在页码为567(假设在该位置)，这时候又来了一个汉字“于”，拼音也是“yu”，那么这时候我们要是按照转换规则，也得将其放在页码为567的位置，但是我们发现这个页码已经被占用了，这时候怎么办？我们可以在通过另一种函数，得到的值加1。那么汉字"于"就会被放在576+1=577的位置。

　　第二种是**链地址法**，我们可以将字典的每一页都看成是一个子数组或者子链表，当遇到冲突了，直接往当前页码的子数组或者子链表里面填充即可。那么我们进行同音字查找的时候，可能需要遍历其子数组或者子链表。如下图所示：

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180404082228573-2042730199.png)

　　对于开放地址法，可能会遇到二次冲突，三次冲突，所以需要良好的散列函数，分布的越均匀越好。对于链地址法，虽然不会造成二次冲突，但是如果一次冲突很多，那么会造成子数组或者子链表很长，那么我们查找所需遍历的时间也会很长。

　　关于哈希表的详细介绍，请[点击这里](http://www.cnblogs.com/ysocean/p/8032656.html)。

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 2、什么是 HashMap？

　　听名字就知道，HashMap 是一个利用哈希表原理来存储元素的集合。遇到冲突时，HashMap 是采用的链地址法来解决，在 JDK1.7 中，HashMap 是由 数组+链表构成的。但是在 JDK1.8 中，HashMap 是由 数组+链表+红黑树构成，新增了红黑树作为底层数据结构，结构变得复杂了，但是效率也变的更高效。下面我们来具体介绍在 JDK1.8 中 HashMap 是如何实现的。

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180404211600945-1711602320.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 3、HashMap定义

　　HashMap 是一个散列表，它存储的内容是键值对(key-value)映射，而且 key 和 value 都可以为 null。

```
1 public class HashMap<K,V> extends AbstractMap<K,V>
2     implements Map<K,V>, Cloneable, Serializable {
```

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180403211343832-2129553780.png)

　　首先该类实现了一个 Map 接口，该接口定义了一组键值对映射通用的操作。储存一组成对的键-值对象，提供key（键）到value（值）的映射，Map中的key不要求有序，不允许重复。value同样不要求有序，但可以重复。但是我们发现该接口方法有很多，我们设计某个键值对的集合有时候并不像实现那么多方法，那该怎么办？

　　JDK 还为我们提供了一个抽象类 AbstractMap ，该抽象类继承 Map 接口，所以如果我们不想实现所有的 Map 接口方法，就可以选择继承抽象类 AbstractMap 。

　　**但是我们发现 HashMap 类即继承了 AbstractMap 接口，也实现了 Map 接口，这样做难道不是多此一举？后面我们会讲的 LinkedHashSet 集合也有这样的写法。**

　　毕竟 JDK 经过这么多年的发展维护，博主起初也是认为这样是有具体的作用的，后来找了很多资料，发现这其实完全没有任何作用，[具体出处](https://stackoverflow.com/questions/2165204/why-does-linkedhashsete-extend-hashsete-and-implement-sete)。

```
据 java 集合框架的创始人Josh Bloch描述，这样的写法是一个失误。在java集合框架中，类似这样的写法很多，最开始写java集合框架的时候，他认为这样写，在某些地方可能是有价值的，直到他意识到错了。显然的，JDK的维护者，后来不认为这个小小的失误值得去修改，所以就这样存在下来了。
```

　　HashMap 集合还实现了 Cloneable 接口以及 Serializable 接口，分别用来进行对象克隆以及将对象进行序列化。

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 4、字段属性

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //序列化和反序列化时，通过该字段进行版本一致性验证
 2     private static final long serialVersionUID = 362498820763181265L;
 3     //默认 HashMap 集合初始容量为16（必须是 2 的倍数）
 4     static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
 5     //集合的最大容量，如果通过带参构造指定的最大容量超过此数，默认还是使用此数
 6     static final int MAXIMUM_CAPACITY = 1 << 30;
 7     //默认的填充因子
 8     static final float DEFAULT_LOAD_FACTOR = 0.75f;
 9     //当桶(bucket)上的结点数大于这个值时会转成红黑树(JDK1.8新增)
10     static final int TREEIFY_THRESHOLD = 8;
11     //当桶(bucket)上的节点数小于这个值时会转成链表(JDK1.8新增)
12     static final int UNTREEIFY_THRESHOLD = 6;
13     /**(JDK1.8新增)
14      * 当集合中的容量大于这个值时，表中的桶才能进行树形化 ，否则桶内元素太多时会扩容，
15      * 而不是树形化 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
16      */
17     static final int MIN_TREEIFY_CAPACITY = 64;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**注意：后面三个字段是 JDK1.8 新增的，主要是用来进行红黑树和链表的互相转换。**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     /**
 2      * 初始化使用，长度总是 2的幂
 3      */
 4     transient Node<K,V>[] table;
 5 
 6     /**
 7      * 保存缓存的entrySet（）
 8      */
 9     transient Set<Map.Entry<K,V>> entrySet;
10 
11     /**
12      * 此映射中包含的键值映射的数量。（集合存储键值对的数量）
13      */
14     transient int size;
15 
16     /**
17      * 跟前面ArrayList和LinkedList集合中的字段modCount一样，记录集合被修改的次数
18      * 主要用于迭代器中的快速失败
19      */
20     transient int modCount;
21 
22     /**
23      * 调整大小的下一个大小值（容量*加载因子）。capacity * load factor
24      */
25     int threshold;
26 
27     /**
28      * 散列表的加载因子。
29      */
30     final float loadFactor;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　下面我们重点介绍上面几个字段：

　　**①、Node[] table**

　　我们说 HashMap 是由数组+链表+红黑树组成，这里的数组就是 table 字段。后面对其进行初始化长度默认是 DEFAULT_INITIAL_CAPACITY= 16。而且 JDK 声明数组的长度总是 2的n次方(一定是合数)，为什么这里要求是合数，一般我们知道哈希算法为了避免冲突都要求长度是质数，这里要求是合数，下面在介绍 HashMap 的hashCode() 方法(散列函数)，我们再进行讲解。

　　②**、size**

　　集合中存放key-value 的实时对数。

　　**③、loadFactor**

　　装载因子，是用来衡量 HashMap 满的程度，计算HashMap的实时装载因子的方法为：size/capacity，而不是占用桶的数量去除以capacity。capacity 是桶的数量，也就是 table 的长度length。

　　默认的负载因子0.75 是对空间和时间效率的一个平衡选择，建议大家不要修改，除非在时间和空间比较特殊的情况下，如果内存空间很多而又对时间效率要求很高，可以降低负载因子loadFactor 的值；相反，如果内存空间紧张而对时间效率要求不高，可以增加负载因子 loadFactor 的值，这个值可以大于1。

　　**④、threshold**

　　计算公式：capacity * loadFactor。这个值是当前已占用数组长度的最大值。过这个数目就重新resize(扩容)，扩容后的 HashMap 容量是之前容量的两倍

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 5、构造函数

　　**①、默认无参构造函数**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    /**
     * 默认构造函数，初始化加载因子loadFactor = 0.75
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; 
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　无参构造器，初始化散列表的加载因子为0.75

　　**②、指定初始容量的构造函数**

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     /**
 2      * 
 3      * @param initialCapacity 指定初始化容量
 4      * @param loadFactor 加载因子 0.75
 5      */
 6     public HashMap(int initialCapacity, float loadFactor) {
 7         //初始化容量不能小于 0 ，否则抛出异常
 8         if (initialCapacity < 0)
 9             throw new IllegalArgumentException("Illegal initial capacity: " +
10                                                initialCapacity);
11         //如果初始化容量大于2的30次方，则初始化容量都为2的30次方
12         if (initialCapacity > MAXIMUM_CAPACITY)
13             initialCapacity = MAXIMUM_CAPACITY;
14         //如果加载因子小于0，或者加载因子是一个非数值，抛出异常
15         if (loadFactor <= 0 || Float.isNaN(loadFactor))
16             throw new IllegalArgumentException("Illegal load factor: " +
17                                                loadFactor);
18         this.loadFactor = loadFactor;
19         this.threshold = tableSizeFor(initialCapacity);
20     }
21     // 返回大于等于initialCapacity的最小的二次幂数值。
22     // >>> 操作符表示无符号右移，高位取0。
23     // | 按位或运算
24     static final int tableSizeFor(int cap) {
25         int n = cap - 1;
26         n |= n >>> 1;
27         n |= n >>> 2;
28         n |= n >>> 4;
29         n |= n >>> 8;
30         n |= n >>> 16;
31         return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
32     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 6、确定哈希桶数组索引位置

　　前面我们讲解哈希表的时候，我们知道是用散列函数来确定索引的位置。散列函数设计的越好，使得元素分布的越均匀。HashMap 是数组+链表+红黑树的组合，我们希望在有限个数组位置时，尽量每个位置的元素只有一个，那么当我们用散列函数求得索引位置的时候，我们能马上知道对应位置的元素是不是我们想要的，而不是要进行链表的遍历或者红黑树的遍历，这会大大优化我们的查询效率。我们看 HashMap 中的哈希算法：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    
    i = (table.length - 1) & hash;//这一步是在后面添加元素putVal()方法中进行位置的确定
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　主要分为三步：

　　①、取 hashCode 值： key.hashCode()

　　②、高位参与运算：h>>>16

　　③、取模运算：(n-1) & hash

　　这里获取 hashCode() 方法的值是变量，但是我们知道，对于任意给定的对象，只要它的 hashCode() 返回值相同，那么程序调用 hash(Object key) 所计算得到的 hash码 值总是相同的。

　　为了让数组元素分布均匀，我们首先想到的是把获得的 hash码对数组长度取模运算( hash%length)，但是计算机都是二进制进行操作，取模运算相对开销还是很大的，那该如何优化呢？

　　HashMap 使用的方法很巧妙，它通过 hash & (table.length -1)来得到该对象的保存位，前面说过 HashMap 底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当 length 总是2的n次方时，hash & (length-1)运算等价于对 length 取模，也就是 hash%length，但是&比%具有更高的效率。比如 n % 32 = n & (32 -1)

　　**这也解释了为什么要保证数组的长度总是2的n次方。**

　　再就是在 JDK1.8 中还有个高位参与运算，hashCode() 得到的是一个32位 int 类型的值，通过hashCode()的高16位 **异或** 低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

　　下面举例说明下，n为table的长度：

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180405000930403-1215437085.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 7、添加元素

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //hash(key)就是上面讲的hash方法，对其进行了第一步和第二步处理
 2     public V put(K key, V value) {
 3         return putVal(hash(key), key, value, false, true);
 4     }
 5     /**
 6      * 
 7      * @param hash 索引的位置
 8      * @param key  键
 9      * @param value  值
10      * @param onlyIfAbsent true 表示不要更改现有值
11      * @param evict false表示table处于创建模式
12      * @return
13      */
14     final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
15             boolean evict) {
16          Node<K,V>[] tab; Node<K,V> p; int n, i;
17          //如果table为null或者长度为0，则进行初始化
18          //resize()方法本来是用于扩容，由于初始化没有实际分配空间，这里用该方法进行空间分配，后面会详细讲解该方法
19          if ((tab = table) == null || (n = tab.length) == 0)
20              n = (tab = resize()).length;
21          //注意：这里用到了前面讲解获得key的hash码的第三步，取模运算，下面的if-else分别是 tab[i] 为null和不为null
22          if ((p = tab[i = (n - 1) & hash]) == null)
23              tab[i] = newNode(hash, key, value, null);//tab[i] 为null，直接将新的key-value插入到计算的索引i位置
24          else {//tab[i] 不为null，表示该位置已经有值了
25              Node<K,V> e; K k;
26              if (p.hash == hash &&
27                  ((k = p.key) == key || (key != null && key.equals(k))))
28                  e = p;//节点key已经有值了，直接用新值覆盖
29              //该链是红黑树
30              else if (p instanceof TreeNode)
31                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
32              //该链是链表
33              else {
34                  for (int binCount = 0; ; ++binCount) {
35                      if ((e = p.next) == null) {
36                          p.next = newNode(hash, key, value, null);
37                          //链表长度大于8，转换成红黑树
38                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
39                              treeifyBin(tab, hash);
40                          break;
41                      }
42                      //key已经存在直接覆盖value
43                      if (e.hash == hash &&
44                          ((k = e.key) == key || (key != null && key.equals(k))))
45                          break;
46                      p = e;
47                  }
48              }
49              if (e != null) { // existing mapping for key
50                  V oldValue = e.value;
51                  if (!onlyIfAbsent || oldValue == null)
52                      e.value = value;
53                  afterNodeAccess(e);
54                  return oldValue;
55              }
56          }
57          ++modCount;//用作修改和新增快速失败
58          if (++size > threshold)//超过最大容量，进行扩容
59              resize();
60          afterNodeInsertion(evict);
61          return null;
62     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　①、判断键值对数组 table 是否为空或为null，否则执行resize()进行扩容；

　　②、根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向⑥，如果table[i]不为空，转向③；

　　③、判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向④，这里的相同指的是hashCode以及equals；

　　④、判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向⑤；

　　⑤、遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

　　⑥、插入成功后，判断实际存在的键值对数量size是否超过了最大容量threshold，如果超过，进行扩容。

　　⑦、如果新插入的key不存在，则返回null，如果新插入的key存在，则返回原key对应的value值（注意新插入的value会覆盖原value值）

　　**注意1：看第 58,59 行代码：**

```
if (++size > threshold)//超过最大容量，进行扩容
    resize();
```

　　这里有个考点，我们知道 HashMap 是由数组+链表+红黑树（JDK1.8）组成，如果在添加元素时，发生冲突，会将冲突的数放在链表上，当链表长度超过8时，会自动转换成红黑树。

　　那么有如下问题：**数组上有5个元素，而某个链表上有3个元素，问此HashMap的 size 是多大？**

　　我们分析第58,59 行代码，很容易知道，**只要是调用put() 方法添加元素，那么就会调用 ++size(这里有个例外是插入重复key的键值对，不会调用，但是重复key元素不会影响size),所以，上面的答案是 7。**

　　**注意2：看第 53 、 60 行代码：**

```
 afterNodeAccess(e);
 afterNodeInsertion(evict);
```

　　这里调用的该方法，其实是调用了如下实现方法：

```
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```

　　这都是一个空的方法实现，我们在这里可以不用管，但是在后面介绍 LinkedHashMap 会用到，LinkedHashMap 是继承的 HashMap，并且重写了该方法，后面我们会详细介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 8、扩容机制

　　扩容（resize），我们知道集合是由数组+链表+红黑树构成，向 HashMap 中插入元素时，如果HashMap 集合的元素已经大于了最大承载容量threshold（capacity * loadFactor），这里的threshold不是数组的最大长度。那么必须扩大数组的长度，Java中数组是无法自动扩容的，我们采用的方法是用一个更大的数组代替这个小的数组，就好比以前是用小桶装水，现在小桶装不下了，我们使用一个更大的桶。

　　JDK1.8融入了红黑树的机制，比较复杂，这里我们先介绍 JDK1.7的扩容源码，便于理解，然后在介绍JDK1.8的源码。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //参数 newCapacity 为新数组的大小
 2     void resize(int newCapacity) {
 3         Entry[] oldTable = table;//引用扩容前的 Entry 数组
 4         int oldCapacity = oldTable.length;
 5         if (oldCapacity == MAXIMUM_CAPACITY) {//扩容前的数组大小如果已经达到最大(2^30)了
 6             threshold = Integer.MAX_VALUE;///修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
 7             return;
 8         }
 9 
10         Entry[] newTable = new Entry[newCapacity];//初始化一个新的Entry数组
11         transfer(newTable, initHashSeedAsNeeded(newCapacity));//将数组元素转移到新数组里面
12         table = newTable;
13         threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);//修改阈值
14     }
15     void transfer(Entry[] newTable, boolean rehash) {
16         int newCapacity = newTable.length;
17         for (Entry<K,V> e : table) {//遍历数组
18             while(null != e) {
19                 Entry<K,V> next = e.next;
20                 if (rehash) {
21                     e.hash = null == e.key ? 0 : hash(e.key);
22                 }
23                 int i = indexFor(e.hash, newCapacity);//重新计算每个元素在数组中的索引位置
24                 e.next = newTable[i];//标记下一个元素，添加是链表头添加
25                 newTable[i] = e;//将元素放在链上
26                 e = next;//访问下一个 Entry 链上的元素
27             }
28         }
29     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　通过方法我们可以看到，JDK1.7中首先是创建一个新的大容量数组，然后依次重新计算原集合所有元素的索引，然后重新赋值。如果数组某个位置发生了hash冲突，使用的是单链表的头插入方法，同一位置的新元素总是放在链表的头部，这样与原集合链表对比，扩容之后的可能就是倒序的链表了。

　　下面我们在看看JDK1.8的。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     final Node<K,V>[] resize() {
 2         Node<K,V>[] oldTab = table;
 3         int oldCap = (oldTab == null) ? 0 : oldTab.length;//原数组如果为null，则长度赋值0
 4         int oldThr = threshold;
 5         int newCap, newThr = 0;
 6         if (oldCap > 0) {//如果原数组长度大于0
 7             if (oldCap >= MAXIMUM_CAPACITY) {//数组大小如果已经大于等于最大值(2^30)
 8                 threshold = Integer.MAX_VALUE;//修改阈值为int的最大值(2^31-1)，这样以后就不会扩容了
 9                 return oldTab;
10             }
11             //原数组长度大于等于初始化长度16，并且原数组长度扩大1倍也小于2^30次方
12             else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
13                      oldCap >= DEFAULT_INITIAL_CAPACITY)
14                 newThr = oldThr << 1; // 阀值扩大1倍
15         }
16         else if (oldThr > 0) //旧阀值大于0，则将新容量直接等于就阀值 
17             newCap = oldThr;
18         else {//阀值等于0，oldCap也等于0（集合未进行初始化）
19             newCap = DEFAULT_INITIAL_CAPACITY;//数组长度初始化为16
20             newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//阀值等于16*0.75=12
21         }
22         //计算新的阀值上限
23         if (newThr == 0) {
24             float ft = (float)newCap * loadFactor;
25             newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
26                       (int)ft : Integer.MAX_VALUE);
27         }
28         threshold = newThr;
29         @SuppressWarnings({"rawtypes","unchecked"})
30             Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
31         table = newTab;
32         if (oldTab != null) {
33             //把每个bucket都移动到新的buckets中
34             for (int j = 0; j < oldCap; ++j) {
35                 Node<K,V> e;
36                 if ((e = oldTab[j]) != null) {
37                     oldTab[j] = null;//元数据j位置置为null，便于垃圾回收
38                     if (e.next == null)//数组没有下一个引用（不是链表）
39                         newTab[e.hash & (newCap - 1)] = e;
40                     else if (e instanceof TreeNode)//红黑树
41                         ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
42                     else { // preserve order
43                         Node<K,V> loHead = null, loTail = null;
44                         Node<K,V> hiHead = null, hiTail = null;
45                         Node<K,V> next;
46                         do {
47                             next = e.next;
48                             //原索引
49                             if ((e.hash & oldCap) == 0) {
50                                 if (loTail == null)
51                                     loHead = e;
52                                 else
53                                     loTail.next = e;
54                                 loTail = e;
55                             }
56                             //原索引+oldCap
57                             else {
58                                 if (hiTail == null)
59                                     hiHead = e;
60                                 else
61                                     hiTail.next = e;
62                                 hiTail = e;
63                             }
64                         } while ((e = next) != null);
65                         //原索引放到bucket里
66                         if (loTail != null) {
67                             loTail.next = null;
68                             newTab[j] = loHead;
69                         }
70                         //原索引+oldCap放到bucket里
71                         if (hiTail != null) {
72                             hiTail.next = null;
73                             newTab[j + oldCap] = hiHead;
74                         }
75                     }
76                 }
77             }
78         }
79         return newTab;
80     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　该方法分为两部分，首先是计算新桶数组的容量 newCap 和新阈值 newThr，然后将原集合的元素重新映射到新集合中。

　　![img](https://images2018.cnblogs.com/blog/1120165/201804/1120165-20180408222123209-919665629.png)

　　相比于JDK1.7，1.8使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 9、删除元素

　　HashMap 删除元素首先是要找到 桶的位置，然后如果是链表，则进行链表遍历，找到需要删除的元素后，进行删除；如果是红黑树，也是进行树的遍历，找到元素删除后，进行平衡调节，注意，当红黑树的节点数小于 6 时，会转化成链表。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public V remove(Object key) {
 2         Node<K,V> e;
 3         return (e = removeNode(hash(key), key, null, false, true)) == null ?
 4             null : e.value;
 5     }
 6     
 7     final Node<K,V> removeNode(int hash, Object key, Object value,
 8             boolean matchValue, boolean movable) {
 9         Node<K,V>[] tab; Node<K,V> p; int n, index;
10         //(n - 1) & hash找到桶的位置
11         if ((tab = table) != null && (n = tab.length) > 0 &&
12         (p = tab[index = (n - 1) & hash]) != null) {
13         Node<K,V> node = null, e; K k; V v;
14         //如果键的值与链表第一个节点相等，则将 node 指向该节点
15         if (p.hash == hash &&
16         ((k = p.key) == key || (key != null && key.equals(k))))
17         node = p;
18         //如果桶节点存在下一个节点
19         else if ((e = p.next) != null) {
20             //节点为红黑树
21         if (p instanceof TreeNode)
22          node = ((TreeNode<K,V>)p).getTreeNode(hash, key);//找到需要删除的红黑树节点
23         else {
24          do {//遍历链表，找到待删除的节点
25              if (e.hash == hash &&
26                  ((k = e.key) == key ||
27                   (key != null && key.equals(k)))) {
28                  node = e;
29                  break;
30              }
31              p = e;
32          } while ((e = e.next) != null);
33         }
34         }
35         //删除节点，并进行调节红黑树平衡
36         if (node != null && (!matchValue || (v = node.value) == value ||
37                       (value != null && value.equals(v)))) {
38         if (node instanceof TreeNode)
39          ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
40         else if (node == p)
41          tab[index] = node.next;
42         else
43          p.next = node.next;
44         ++modCount;
45         --size;
46         afterNodeRemoval(node);
47         return node;
48         }
49         }
50         return null;
51     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　注意第 46 行代码

```
afterNodeRemoval(node);
```

　　这也是为实现 LinkedHashMap 做准备的，在这里和上面一样，是一个空方法实现，可以不用管。而在 LinkedHashMap 中进行了重写，用来维护删除节点后，链表的前后关系。

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 10、查找元素

　　①、通过 key 查找 value

　　首先通过 key 找到计算索引，找到桶位置，先检查第一个节点，如果是则返回，如果不是，则遍历其后面的链表或者红黑树。其余情况全部返回 null。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public V get(Object key) {
 2         Node<K,V> e;
 3         return (e = getNode(hash(key), key)) == null ? null : e.value;
 4     }
 5     
 6     final Node<K,V> getNode(int hash, Object key) {
 7         Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
 8         if ((tab = table) != null && (n = tab.length) > 0 &&
 9             (first = tab[(n - 1) & hash]) != null) {
10             //根据key计算的索引检查第一个索引
11             if (first.hash == hash && // always check first node
12                 ((k = first.key) == key || (key != null && key.equals(k))))
13                 return first;
14             //不是第一个节点
15             if ((e = first.next) != null) {
16                 if (first instanceof TreeNode)//遍历树查找元素
17                     return ((TreeNode<K,V>)first).getTreeNode(hash, key);
18                 do {
19                     //遍历链表查找元素
20                     if (e.hash == hash &&
21                         ((k = e.key) == key || (key != null && key.equals(k))))
22                         return e;
23                 } while ((e = e.next) != null);
24             }
25         }
26         return null;
27     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　②、判断是否存在给定的 key 或者 value

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public boolean containsKey(Object key) {
 2         return getNode(hash(key), key) != null;
 3     }
 4     public boolean containsValue(Object value) {
 5         Node<K,V>[] tab; V v;
 6         if ((tab = table) != null && size > 0) {
 7             //遍历桶
 8             for (int i = 0; i < tab.length; ++i) {
 9                 //遍历桶中的每个节点元素
10                 for (Node<K,V> e = tab[i]; e != null; e = e.next) {
11                     if ((v = e.value) == value ||
12                         (value != null && value.equals(v)))
13                         return true;
14                 }
15             }
16         }
17         return false;
18     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 11、遍历元素 

　首先构造一个 HashMap 集合：

```
1 HashMap<String,Object> map = new HashMap<>();
2 map.put("A","1");
3 map.put("B","2");
4 map.put("C","3");
```

　　①、分别获取 key 集合和 value 集合。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 //1、分别获取key和value的集合
2 for(String key : map.keySet()){
3     System.out.println(key);
4 }
5 for(Object value : map.values()){
6     System.out.println(value);
7 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　②、获取 key 集合，然后遍历key集合，根据key分别得到相应value

```
1 //2、获取key集合，然后遍历key，根据key得到 value
2 Set<String> keySet = map.keySet();
3 for(String str : keySet){
4     System.out.println(str+"-"+map.get(str));
5 }
```

　　③、得到 Entry 集合，然后遍历 Entry

```
1 //3、得到 Entry 集合，然后遍历 Entry
2 Set<Map.Entry<String,Object>> entrySet = map.entrySet();
3 for(Map.Entry<String,Object> entry : entrySet){
4     System.out.println(entry.getKey()+"-"+entry.getValue());
5 }
```

　　④、迭代

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1 //4、迭代
2 Iterator<Map.Entry<String,Object>> iterator = map.entrySet().iterator();
3 while(iterator.hasNext()){
4     Map.Entry<String,Object> mapEntry = iterator.next();
5     System.out.println(mapEntry.getKey()+"-"+mapEntry.getValue());
6 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　基本上使用第三种方法是性能最好的，

　　第一种遍历方法在我们只需要 key 集合或者只需要 value 集合时使用；

　　第二种方法效率很低，不推荐使用；

　　第四种方法效率也挺好，关键是在遍历的过程中我们可以对集合中的元素进行删除。

[回到顶部](https://www.cnblogs.com/ysocean/p/8711071.html#_labelTop)

### 12、总结

　　①、基于JDK1.8的HashMap是由数组+链表+红黑树组成，当链表长度超过 8 时会自动转换成红黑树，当红黑树节点个数小于 6 时，又会转化成链表。相对于早期版本的 JDK HashMap 实现，新增了红黑树作为底层数据结构，在数据量较大且哈希碰撞较多时，能够极大的增加检索的效率。

　　②、允许 key 和 value 都为 null。key 重复会被覆盖，value 允许重复。

　　③、非线程安全

　　④、无序（遍历HashMap得到元素的顺序不是按照插入的顺序）

 

参考文档：https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html#

　　　　　http://www.importnew.com/20386.html

　　　　　https://www.cnblogs.com/nullllun/p/8327664.html





# [java.util.HashSet 类](https://www.cnblogs.com/ysocean/p/9809046.html)



**目录**

- [1、HashSet 定义](https://www.cnblogs.com/ysocean/p/9809046.html#_label0)
- [2、字段属性](https://www.cnblogs.com/ysocean/p/9809046.html#_label1)
- [3、构造函数](https://www.cnblogs.com/ysocean/p/9809046.html#_label2)
- [4、添加元素](https://www.cnblogs.com/ysocean/p/9809046.html#_label3)
- [5、删除元素](https://www.cnblogs.com/ysocean/p/9809046.html#_label4)
- [6、查找元素](https://www.cnblogs.com/ysocean/p/9809046.html#_label5)
- [7、遍历元素](https://www.cnblogs.com/ysocean/p/9809046.html#_label6)

 

------

　　在上一篇博客，我们介绍了 Map 集合的一种典型实现 [ HashMap ](https://www.cnblogs.com/ysocean/p/8711071.html) ，在 JDK1.8 中，HashMap 是由 数组+链表+红黑树构成，相对于早期版本的 JDK HashMap 实现，新增了红黑树作为底层数据结构，在数据量较大且哈希碰撞较多时，能够极大的增加检索的效率。了解 HashMap 的具体实现后，我们再来介绍由 HashMap 作为底层数据结构实现的一种数据结构——HashSet。（如果不了解 HashMap 的实现原理，建议先看看 HashMap，不然直接看 HashSet 是很难看懂的）

[回到顶部](https://www.cnblogs.com/ysocean/p/9809046.html#_labelTop)

### 1、HashSet 定义

　　**HashSet 是一个由 HashMap 实现的集合。元素无序且不能重复。**

```
1 public class HashSet<E>
2     extends AbstractSet<E>
3     implements Set<E>, Cloneable, java.io.Serializable
```

　　![img](https://img2018.cnblogs.com/blog/1120165/201810/1120165-20181018103334934-1509611642.png)

　　和前面介绍的大多数集合一样，HashSet 也实现了 Cloneable 接口和 Serializable 接口，分别用来支持克隆以及支持序列化。还实现了 Set 接口，该接口定义了 Set 集合类型的一套规范。

[回到顶部](https://www.cnblogs.com/ysocean/p/9809046.html#_labelTop)

### 2、字段属性

```
1 //HashSet集合中的内容是通过 HashMap 数据结构来存储的
2 private transient HashMap<E,Object> map;
3 //向HashSet中添加数据，数据在上面的 map 结构是作为 key 存在的，而value统一都是 PRESENT
4 private static final Object PRESENT = new Object();
```

　　第一个定义一个 HashMap，作为实现 HashSet 的数据结构；第二个 PRESENT 对象，因为前面讲过 HashMap 是作为键值对 key-value 进行存储的，而 HashSet 不是键值对，那么选择 HashMap 作为实现，其原理就是存储在 HashSet 中的数据 作为 Map 的 key，而 Map 的value 统一为 PRESENT（下面介绍具体实现时会了解）。

[回到顶部](https://www.cnblogs.com/ysocean/p/9809046.html#_labelTop)

### 3、构造函数

　　①、无参构造

```
1     public HashSet() {
2         map = new HashMap<>();
3     }
```

　　直接 new 一个 HashMap 对象出来，采用无参的 HashMap 构造函数，具有默认初始容量（16）和加载因子（0.75）。

　　②、指定初始容量

```
1     public HashSet(int initialCapacity) {
2         map = new HashMap<>(initialCapacity);
3     }
```

　　③、指定初始容量和加载因子

```
1     public HashSet(int initialCapacity, float loadFactor) {
2         map = new HashMap<>(initialCapacity, loadFactor);
3     }
```

　　④、构造包含指定集合中的元素

```
1     public HashSet(Collection<? extends E> c) {
2         map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
3         addAll(c);
4     }
```

　　集合容量很好理解，这里我介绍一下什么是加载因子。在 HashMap 中，能够存储元素的数量就是：总的容量*加载因子 ，新增一个元素时，如果HashMap集合中的元素大于前面公式计算的结果了，那么就必须要进行扩容操作，从时间和空间考虑，加载因子一般都选默认的0.75。

[回到顶部](https://www.cnblogs.com/ysocean/p/9809046.html#_labelTop)

### 4、添加元素

```
1     public boolean add(E e) {
2         return map.put(e, PRESENT)==null;
3     }
```

　　通过 map.put() 方法来添加元素，在上一篇博客介绍该方法时，说明了该方法如果新插入的key不存在，则返回null，如果新插入的key存在，则返回原key对应的value值（注意新插入的value会覆盖原value值）。

　　也就是说 HashSet 的 add(E e) 方法，会将 e 作为 key，PRESENT 作为 value 插入到 map 集合中，如果 e 不存在，则插入成功返回 true;如果存在，则返回false。

[回到顶部](https://www.cnblogs.com/ysocean/p/9809046.html#_labelTop)

### 5、删除元素

```
1     public boolean remove(Object o) {
2         return map.remove(o)==PRESENT;
3     }
```

　　调用 HashMap 的remove(Object o) 方法，该方法会首先查找 map 集合中是否存在 o ，如果存在则删除，并返回该值，如果不存在则返回 null。

　　也就是说 HashSet 的 remove(Object o) 方法，删除成功返回 true，删除的元素不存在会返回 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/9809046.html#_labelTop)

### 6、查找元素

```
1     public boolean contains(Object o) {
2         return map.containsKey(o);
3     }
```

　　调用 HashMap 的 containsKey(Object o) 方法，找到了返回 true，找不到返回 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/9809046.html#_labelTop)

### 7、遍历元素

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 HashSet<Integer> set = new HashSet<>();
 2 set.add(1);
 3 set.add(2);
 4 //增强for循环
 5 for(Integer i : set){
 6     System.out.println(i);
 7 }
 8 //普通for循环
 9 Iterator<Integer> iterator = set.iterator();
10 while (iterator.hasNext()){
11     System.out.println(iterator.next());
12 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)





# [java.util.LinkedHashMap 类](https://www.cnblogs.com/ysocean/p/9839173.html)



**目录**

- [1、LinkedHashMap 定义](https://www.cnblogs.com/ysocean/p/9839173.html#_label0)
- [2、字段属性](https://www.cnblogs.com/ysocean/p/9839173.html#_label1)
- [3、构造函数](https://www.cnblogs.com/ysocean/p/9839173.html#_label2)
- [4、添加元素](https://www.cnblogs.com/ysocean/p/9839173.html#_label3)
- [5、删除元素](https://www.cnblogs.com/ysocean/p/9839173.html#_label4)
- [6、查找元素](https://www.cnblogs.com/ysocean/p/9839173.html#_label5)
- [7、遍历元素](https://www.cnblogs.com/ysocean/p/9839173.html#_label6)
- [8、迭代器](https://www.cnblogs.com/ysocean/p/9839173.html#_label7)
- [9、总结](https://www.cnblogs.com/ysocean/p/9839173.html#_label8)

 

------

　　前面我们介绍了 Map 集合的一种典型实现 [ HashMap ](https://www.cnblogs.com/ysocean/p/8711071.html) ，关于 HashMap 的特性，我们再来复习一遍：

　　①、基于JDK1.8的HashMap是由数组+链表+红黑树组成，相对于早期版本的 JDK HashMap 实现，新增了红黑树作为底层数据结构，在数据量较大且哈希碰撞较多时，能够极大的增加检索的效率。

　　②、允许 key 和 value 都为 null。key 重复会被覆盖，value 允许重复。

　　③、非线程安全

　　④、无序（遍历HashMap得到元素的顺序不是按照插入的顺序）

　　HashMap 集合可以说是最重要的集合之一，上篇博客介绍的 HashSet 集合就是继承 HashMap 来实现的。而本篇博客我们介绍 Map 集合的另一种实现——LinkedHashMap，其实也是继承 HashMap 集合来实现的，而且我们在介绍 HashMap 集合的 put 方法时，也指出了 put 方法中调用的部分方法在 HashMap 都是空实现，而在 LinkedHashMap 中进行了重写。所以想要彻底了解 LinkedHashMap 的实现原理，HashMap 的实现原理一定不能不懂。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 1、LinkedHashMap 定义

　　LinkedHashMap 是基于 HashMap 实现的一种集合，具有 HashMap 集合上面所说的所有特点，除了 HashMap 无序的特点，LinkedHashMap 是有序的，因为 LinkedHashMap 在 HashMap 的基础上单独维护了一个具有所有数据的双向链表，该链表保证了元素迭代的顺序。

　　所以我们可以直接这样说：LinkedHashMap = HashMap + LinkedList。LinkedHashMap 就是在 HashMap 的基础上多维护了一个双向链表，用来保证元素迭代顺序。

　　更形象化的图形展示可以直接移到文章末尾。

```
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
```

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181119211450454-1436611376.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 2、字段属性

 　①、Entry<K,V>

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　LinkedHashMap 的每个元素都是一个 Entry，我们看到对于 Entry 继承自 HashMap 的 Node 结构，相对于 Node 结构，LinkedHashMap 多了 before 和 after 结构。

　　下面是Map类集合基本元素的实现演变。

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181119213656738-1642597597.png)

　　LinkedHashMap 中 Entry 相对于 HashMap 多出的 before 和 after 便是用来维护 LinkedHashMap 插入 Entry 的先后顺序的。

　　②、其它属性

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
//用来指向双向链表的头节点
transient LinkedHashMap.Entry<K,V> head;
//用来指向双向链表的尾节点
transient LinkedHashMap.Entry<K,V> tail;
//用来指定LinkedHashMap的迭代顺序
//true 表示按照访问顺序，会把访问过的元素放在链表后面，放置顺序是访问的顺序
//false 表示按照插入顺序遍历
final boolean accessOrder;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 　注意：这里有五个属性别搞混淆的，对于 Node next 属性，是用来维护整个集合中 Entry 的顺序。对于 Entry before，Entry after ，以及 Entry head，Entry tail，这四个属性都是用来维护保证集合顺序的链表，其中前两个before和after表示某个节点的上一个节点和下一个节点，这是一个双向链表。后两个属性 head 和 tail 分别表示这个链表的头节点和尾节点。

　　PS：关于双向链表的介绍，可以看[这篇博客](https://www.cnblogs.com/ysocean/p/7928988.html)。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 3、构造函数

　　①、无参构造

```
1     public LinkedHashMap() {
2         super();
3         accessOrder = false;
4     }
```

　　调用无参的 HashMap 构造函数，具有默认初始容量（16）和加载因子（0.75）。并且设定了 accessOrder = false，表示默认按照插入顺序进行遍历。

　　②、指定初始容量

```
1     public LinkedHashMap(int initialCapacity) {
2         super(initialCapacity);
3         accessOrder = false;
4     }
```

　　③、指定初始容量和加载因子

```
1     public LinkedHashMap(int initialCapacity, float loadFactor) {
2         super(initialCapacity, loadFactor);
3         accessOrder = false;
4     }
```

　　④、指定初始容量和加载因子，以及迭代规则

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public LinkedHashMap(int initialCapacity,
2                          float loadFactor,
3                          boolean accessOrder) {
4         super(initialCapacity, loadFactor);
5         this.accessOrder = accessOrder;
6     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　⑤、构造包含指定集合中的元素

```
1     public LinkedHashMap(Map<? extends K, ? extends V> m) {
2         super();
3         accessOrder = false;
4         putMapEntries(m, false);
5     }
```

　　上面所有的构造函数默认 accessOrder = false，除了第四个构造函数能够指定 accessOrder 的值。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 4、添加元素

 　LinkedHashMap 中是没有 put 方法的，直接调用父类 HashMap 的 put 方法。关于 HashMap 的put 方法，可以参看我对于[ HashMap 的介绍](https://www.cnblogs.com/ysocean/p/8711071.html#_label6)。

 　我将方法介绍复制到下面：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //hash(key)就是上面讲的hash方法，对其进行了第一步和第二步处理
 2     public V put(K key, V value) {
 3         return putVal(hash(key), key, value, false, true);
 4     }
 5     /**
 6      * 
 7      * @param hash 索引的位置
 8      * @param key  键
 9      * @param value  值
10      * @param onlyIfAbsent true 表示不要更改现有值
11      * @param evict false表示table处于创建模式
12      * @return
13      */
14     final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
15             boolean evict) {
16          Node<K,V>[] tab; Node<K,V> p; int n, i;
17          //如果table为null或者长度为0，则进行初始化
18          //resize()方法本来是用于扩容，由于初始化没有实际分配空间，这里用该方法进行空间分配，后面会详细讲解该方法
19          if ((tab = table) == null || (n = tab.length) == 0)
20              n = (tab = resize()).length;
21          //注意：这里用到了前面讲解获得key的hash码的第三步，取模运算，下面的if-else分别是 tab[i] 为null和不为null
22          if ((p = tab[i = (n - 1) & hash]) == null)
23              tab[i] = newNode(hash, key, value, null);//tab[i] 为null，直接将新的key-value插入到计算的索引i位置
24          else {//tab[i] 不为null，表示该位置已经有值了
25              Node<K,V> e; K k;
26              if (p.hash == hash &&
27                  ((k = p.key) == key || (key != null && key.equals(k))))
28                  e = p;//节点key已经有值了，直接用新值覆盖
29              //该链是红黑树
30              else if (p instanceof TreeNode)
31                  e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
32              //该链是链表
33              else {
34                  for (int binCount = 0; ; ++binCount) {
35                      if ((e = p.next) == null) {
36                          p.next = newNode(hash, key, value, null);
37                          //链表长度大于8，转换成红黑树
38                          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
39                              treeifyBin(tab, hash);
40                          break;
41                      }
42                      //key已经存在直接覆盖value
43                      if (e.hash == hash &&
44                          ((k = e.key) == key || (key != null && key.equals(k))))
45                          break;
46                      p = e;
47                  }
48              }
49              if (e != null) { // existing mapping for key
50                  V oldValue = e.value;
51                  if (!onlyIfAbsent || oldValue == null)
52                      e.value = value;
53                  afterNodeAccess(e);
54                  return oldValue;
55              }
56          }
57          ++modCount;//用作修改和新增快速失败
58          if (++size > threshold)//超过最大容量，进行扩容
59              resize();
60          afterNodeInsertion(evict);
61          return null;
62     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 　这里主要介绍上面方法中，为了保证 LinkedHashMap 的迭代顺序，在添加元素时重写了的4个方法，分别是第23行、31行以及53、60行代码：

```
1 newNode(hash, key, value, null);
2 putTreeVal(this, tab, hash, key, value)//newTreeNode(h, k, v, xpn)
3 afterNodeAccess(e);
4 afterNodeInsertion(evict);
```

　　①、对于 newNode(hash,key,value,null) 方法

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    HashMap.Node<K,V> newNode(int hash, K key, V value, HashMap.Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
                new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        //用临时变量last记录尾节点tail
        LinkedHashMap.Entry<K,V> last = tail;
        //将尾节点设为当前插入的节点p
        tail = p;
        //如果原先尾节点为null，表示当前链表为空
        if (last == null)
            //头结点也为当前插入节点
            head = p;
        else {
            //原始链表不为空，那么将当前节点的上节点指向原始尾节点
            p.before = last;
            //原始尾节点的下一个节点指向当前插入节点
            last.after = p;
        }
    }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　也就是说将当前添加的元素设为原始链表的尾节点。

　　②、对于 putTreeVal 方法

　　是在添加红黑树节点时的操作，LinkedHashMap 也重写了该方法的 newTreeNode 方法：

```
1     TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
2         TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
3         linkNodeLast(p);
4         return p;
5     }
```

　　也就是说上面两个方法都是在将新添加的元素放置到链表的尾端，并维护链表节点之间的关系。　

　　③、对于 afterNodeAccess(e) 方法，在 putVal 方法中，是当添加数据键值对的 key 存在时，会对 value 进行替换。然后调用 afterNodeAccess(e) 方法：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //把当前节点放到双向链表的尾部
 2     void afterNodeAccess(HashMap.Node<K,V> e) { // move node to last
 3         LinkedHashMap.Entry<K,V> last;
 4         //当 accessOrder = true 并且当前节点不等于尾节点tail。这里将last节点赋值为tail节点
 5         if (accessOrder && (last = tail) != e) {
 6             //记录当前节点的上一个节点b和下一个节点a
 7             LinkedHashMap.Entry<K,V> p =
 8                     (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
 9             //释放当前节点和后一个节点的关系
10             p.after = null;
11             //如果当前节点的前一个节点为null
12             if (b == null)
13                 //头节点=当前节点的下一个节点
14                 head = a;
15             else
16                 //否则b的后节点指向a
17                 b.after = a;
18             //如果a != null
19             if (a != null)
20                 //a的前一个节点指向b
21                 a.before = b;
22             else
23                 //b设为尾节点
24                 last = b;
25             //如果尾节点为null
26             if (last == null)
27                 //头节点设为p
28                 head = p;
29             else {
30                 //否则将p放到双向链表的最后
31                 p.before = last;
32                 last.after = p;
33             }
34             //将尾节点设为p
35             tail = p;
36             //LinkedHashMap对象操作次数+1，用于快速失败校验
37             ++modCount;
38         }
39     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　该方法是在 accessOrder = true 并且 插入的当前节点不等于尾节点时，该方法才会生效。并且该方法的作用是将插入的节点变为尾节点，后面在get方法中也会调用。代码实现可能有点绕，可以借助下图来理解：

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120073825056-124637381.png)

 　④、在看 afterNodeInsertion(evict) 方法

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     void afterNodeInsertion(boolean evict) { // possibly remove eldest
2         LinkedHashMap.Entry<K,V> first;
3         if (evict && (first = head) != null && removeEldestEntry(first)) {
4             K key = first.key;
5             removeNode(hash(key), key, null, false, true);
6         }
7     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　该方法用来移除最老的首节点，首先方法要能执行到if语句里面，必须 evict = true，并且 头节点不为null，并且 removeEldestEntry(first) 返回true，这三个条件必须同时满足，前面两个好理解，我们看最后这个方法条件：

```
1     protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
2         return false;
3     }
```

　　这就奇怪了，该方法直接返回的是 false，也就是说怎么都不会进入到 if 方法体内了，那这是这么回事呢？

　　这其实是用来实现 LRU（Least Recently Used，最近最少使用）Cache 时，重写的一个方法。比如在 mybatis-connector 包中，有这样一个类：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.mysql.jdbc.util;
 2 
 3 import java.util.LinkedHashMap;
 4 import java.util.Map.Entry;
 5 
 6 public class LRUCache<K, V> extends LinkedHashMap<K, V> {
 7     private static final long serialVersionUID = 1L;
 8     protected int maxElements;
 9 
10     public LRUCache(int maxSize) {
11         super(maxSize, 0.75F, true);
12         this.maxElements = maxSize;
13     }
14 
15     protected boolean removeEldestEntry(Entry<K, V> eldest) {
16         return this.size() > this.maxElements;
17     }
18 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　可以看到，它重写了 removeEldestEntry(Entry<K,V> eldest) 方法，当元素的个数大于设定的最大个数，便移除首元素。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 5、删除元素

 　同理也是调用 HashMap 的remove 方法，这里我不作过多的讲解，着重看LinkedHashMap 重写的第 46 行方法。

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　我们看第 46 行代码实现：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     void afterNodeRemoval(HashMap.Node<K,V> e) { // unlink
 2         LinkedHashMap.Entry<K,V> p =
 3                 (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
 4         p.before = p.after = null;
 5         if (b == null)
 6             head = a;
 7         else
 8             b.after = a;
 9         if (a == null)
10             tail = b;
11         else
12             a.before = b;
13     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　该方法其实很好理解，就是当我们删除某个节点时，为了保证链表还是有序的，那么必须维护其前后节点。而该方法的作用就是维护删除节点的前后节点关系。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 6、查找元素

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public V get(Object key) {
2         Node<K,V> e;
3         if ((e = getNode(hash(key), key)) == null)
4             return null;
5         if (accessOrder)
6             afterNodeAccess(e);
7         return e.value;
8     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　相比于 HashMap 的 get 方法，这里多出了第 5,6行代码，当 accessOrder = true 时，即表示按照最近访问的迭代顺序，会将访问过的元素放在链表后面。

　　对于 afterNodeAccess(e) 方法，在前面第 4 小节 添加元素已经介绍过了，这就不在介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 7、遍历元素

　　在介绍 HashMap 时，我们介绍了 4 中遍历方式，同理，对于 LinkedHashMap 也有 4 种，这里我们介绍效率较高的两种遍历方式：

　　①、得到 Entry 集合，然后遍历 Entry

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1         LinkedHashMap<String,String> map = new LinkedHashMap<>();
2         map.put("A","1");
3         map.put("B","2");
4         map.put("C","3");
5         map.get("B");
6         Set<Map.Entry<String,String>> entrySet = map.entrySet();
7         for(Map.Entry<String,String> entry : entrySet ){
8             System.out.println(entry.getKey()+"---"+entry.getValue());
9         }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　②、迭代

```
1         Iterator<Map.Entry<String,String>> iterator = map.entrySet().iterator();
2         while(iterator.hasNext()){
3             Map.Entry<String,String> entry = iterator.next();
4             System.out.println(entry.getKey()+"----"+entry.getValue());
5         }
```

　　这两种效率都还不错，通过迭代的方式可以对一边遍历一边删除元素，而第一种删除元素会报错。

　　打印结果：

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120110846197-503850128.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 8、迭代器

 　我们把上面遍历的LinkedHashMap 构造函数改成下面的：

```
LinkedHashMap<String,String> map = new LinkedHashMap<>(16,0.75F,true);
```

　　也就是说将 accessOrder = true，表示按照访问顺序来遍历，注意看上面的 第 5 行代码：map.get("B)。也就是说设置 accessOrder = true 之后，那么 B---2 应该是最后输出，我们看一下打印结果：

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120111619239-802966740.png)

　　结果跟预期一致。那么在遍历的过程中，LinkedHashMap 是如何进行的呢？

　　我们追溯源码：首先进入到 map.entrySet() 方法里面：

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120111811584-1862159615.png)

　　发现 entrySet = new LinkedEntrySet() ，接下来我们查看 LinkedEntrySet 类。

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120111937376-827200474.png)

　　这是一个内部类，我们查看其 iterator() 方法，发现又new 了一个新对象 LinkedEntryIterator，接着看这个类：

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120112048656-1526339775.png)

　　这个类继承 LinkedHashIterator。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     abstract class LinkedHashIterator {
 2         LinkedHashMap.Entry<K,V> next;
 3         LinkedHashMap.Entry<K,V> current;
 4         int expectedModCount;
 5 
 6         LinkedHashIterator() {
 7             next = head;
 8             expectedModCount = modCount;
 9             current = null;
10         }
11 
12         public final boolean hasNext() {
13             return next != null;
14         }
15 
16         final LinkedHashMap.Entry<K,V> nextNode() {
17             LinkedHashMap.Entry<K,V> e = next;
18             if (modCount != expectedModCount)
19                 throw new ConcurrentModificationException();
20             if (e == null)
21                 throw new NoSuchElementException();
22             current = e;
23             next = e.after;
24             return e;
25         }
26 
27         public final void remove() {
28             HashMap.Node<K,V> p = current;
29             if (p == null)
30                 throw new IllegalStateException();
31             if (modCount != expectedModCount)
32                 throw new ConcurrentModificationException();
33             current = null;
34             K key = p.key;
35             removeNode(hash(key), key, null, false, false);
36             expectedModCount = modCount;
37         }
38     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　看到 nextNode() 方法，很显然是通过遍历链表的方式来遍历整个 LinkedHashMap 。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839173.html#_labelTop)

### 9、总结

　　通过上面的介绍，关于 LinkedHashMap ，我想直接用下面一幅图来解释：

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120211140408-453616443.png)

 

　　去掉红色和蓝色的虚线指针，其实就是一个HashMap。



# [java.util.LinkedHashSet类](https://www.cnblogs.com/ysocean/p/9839172.html)



**目录**

- [1、LinkedHashSet 定义](https://www.cnblogs.com/ysocean/p/9839172.html#_label0)
- [2、构造函数](https://www.cnblogs.com/ysocean/p/9839172.html#_label1)
- [3、添加元素](https://www.cnblogs.com/ysocean/p/9839172.html#_label2)
- [4、删除元素](https://www.cnblogs.com/ysocean/p/9839172.html#_label3)
- [5、查找元素](https://www.cnblogs.com/ysocean/p/9839172.html#_label4)
- [6、遍历元素](https://www.cnblogs.com/ysocean/p/9839172.html#_label5)

 

------

　　同 HashSet 与 HashMap 的关系一样，本篇博客所介绍的 LinkedHashSet 和 LinkedHashMap 也是一致的。在 JDK 集合框架中，类似 Set 集合通常都是由对应的 Map 类集合来实现的（TreeSet 和 TreeMap 同理），这里很重要的一个理论就是：Set 类集合是不允许重复的，而 Map 类集合的 key 也是不允许重复的，所以通常很容易就用 Map 类集合实现了 Set 类集合。

　　本篇博客之前，我们已经详细介绍了 [HashSet](https://www.cnblogs.com/ysocean/p/9809046.html)、[HashMap](https://www.cnblogs.com/ysocean/p/8711071.html)、[LinkedHashMap](https://www.cnblogs.com/ysocean/p/9839173.html)。在此基础上，再来理解 LinkedHashSet 就很容易了。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839172.html#_labelTop)

### 1、LinkedHashSet 定义

　　LinkedHashSet 是由 LinkedHashMap 实现的集合。元素有序且不能重复。

```
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
```

　　![img](https://img2018.cnblogs.com/blog/1120165/201811/1120165-20181120212637211-509584112.png)

　　看上图类定义，LinkedHashSet 是由 HashSet 来实现的，其实底层是通过 LinkedHashMap 来实现的。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839172.html#_labelTop)

### 2、构造函数

　　在 LinkedHashSet 中，有如下几个构造方法：

　　①、指定初始容量和加载因子

```
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }
```

　　②、指定初始容量

```
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }
```

　　③、默认无参构造函数

```
    public LinkedHashSet() {
        super(16, .75f, true);
    }
```

　　④、构造包含指定集合的元素

```
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
```

　　上面所有的构造方法，都调用父类，也就是 HashSet 的 super(initialCapacity, loadFactor, true);

```
1     HashSet(int initialCapacity, float loadFactor, boolean dummy) {
2         map = new LinkedHashMap<>(initialCapacity, loadFactor);
3     }
```

　　前面两个参数分别设置HashMap 的初始容量和加载因子。dummy 可以忽略掉，这个参数只是为了区分 HashSet 别的构造方法。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839172.html#_labelTop)

### 3、添加元素

```
1     public boolean add(E e) {
2         return map.put(e, PRESENT)==null;
3     }
```

　　通过 map.put() 方法来添加元素，说明了该方法如果新插入的key不存在，则返回null，如果新插入的key存在，则返回原key对应的value值（注意新插入的value会覆盖原value值）。

　　也就是说 add(E e) 方法，会将 e 作为 key，PRESENT 作为 value 插入到 map 集合中，如果 e 不存在，则插入成功返回 true;如果存在，则返回false。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839172.html#_labelTop)

### 4、删除元素

```
1     public boolean remove(Object o) {
2         return map.remove(o)==PRESENT;
3     }
```

　　调用 HashMap 的remove(Object o) 方法，该方法会首先查找 map 集合中是否存在 o ，如果存在则删除，并返回该值，如果不存在则返回 null。

　　也就是 remove(Object o) 方法，删除成功返回 true，删除的元素不存在会返回 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839172.html#_labelTop)

### 5、查找元素

```
1     public boolean contains(Object o) {
2         return map.containsKey(o);
3     }
```

　　调用 HashMap 的 containsKey(Object o) 方法，找到了返回 true，找不到返回 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/9839172.html#_labelTop)

### 6、遍历元素

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 LinkedHashSet<String> hashSet = new LinkedHashSet<>();
 2 hashSet.add("A");
 3 hashSet.add("B");
 4 hashSet.add("C");
 5 //1、增强for循环
 6 for(String str : hashSet){
 7     System.out.println(str);
 8 }
 9 //2、迭代器
10 Iterator<String> iterator = hashSet.iterator();
11 while(iterator.hasNext()){
12     System.out.println(iterator.next());
13 }
```



# [java.util.TreeMap类](https://www.cnblogs.com/ysocean/p/10300976.html)



**目录**

- [1、TreeMap 定义](https://www.cnblogs.com/ysocean/p/10300976.html#_label0)
- [2、字段定义](https://www.cnblogs.com/ysocean/p/10300976.html#_label1)
- [3、构造函数](https://www.cnblogs.com/ysocean/p/10300976.html#_label2)
- [4、添加元素](https://www.cnblogs.com/ysocean/p/10300976.html#_label3)
- [5、删除元素](https://www.cnblogs.com/ysocean/p/10300976.html#_label4)
- [6、查找元素](https://www.cnblogs.com/ysocean/p/10300976.html#_label5)
- [7、遍历元素](https://www.cnblogs.com/ysocean/p/10300976.html#_label6)

 

------

　　在前面几篇博客分别介绍了这样几种集合，基于数组实现的[ArrayList 类](https://www.cnblogs.com/ysocean/p/8622264.html)，基于链表实现的[LinkedList 类](https://www.cnblogs.com/ysocean/p/8657850.html)，基于散列表实现的[HashMap 类](https://www.cnblogs.com/ysocean/p/8711071.html)，本篇博客我们来介绍另一种数据类型，基于树实现的**TreeSet**类。

[回到顶部](https://www.cnblogs.com/ysocean/p/10300976.html#_labelTop)

### 1、TreeMap 定义

　　听名字就知道，TreeMap 是由Tree 和 Map 集合有关的，没错，TreeMap 是由红黑树实现的有序的 key-value 集合。

　　PS:想要学懂TreeMap的实现原理，[红黑树](https://www.cnblogs.com/ysocean/p/8004211.html)的了解是必不可少的！！！

```
public` `class` `TreeMap``  ``extends` `AbstractMap``  ``implements` `NavigableMap, Cloneable, java.io.Serializable
```

　　![img](https://img2018.cnblogs.com/blog/1120165/201901/1120165-20190121213836953-1363088203.png)

　　TreeMap 首先继承了 AbstractMap 抽象类，表示它具有散列表的性质，也就是由 key-value 组成。

　　其次 TreeMap 实现了 NavigableMap 接口，该接口支持一系列获取指定集合的导航方法，比如获取小于指定key的集合。

　　最后分别实现 Serializable 接口以及 Cloneable 接口，分别表示支持对象序列化以及[对象克隆](https://www.cnblogs.com/ysocean/p/8482979.html)。

[回到顶部](https://www.cnblogs.com/ysocean/p/10300976.html#_labelTop)

### 2、字段定义

 　①、Comparator

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
    /**
     * The comparator used to maintain order in this tree map, or
     * null if it uses the natural ordering of its keys.
     *
     * @serial
     */
    private final Comparator<? super K> comparator;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　可以看上面的英文注释，Comparator 是用来维护treemap集合中的顺序，如果为null，则按照key的自然顺序。

　　Comparator 是一个接口，**排序时需要实现其 compare 方法，该方法返回正数，零，负数分别代表大于，等于，小于**。那么怎么使用呢？这里举个例子：

　　这里有一个Person类，里面有两个属性pname，page，我们将该person对象放入ArrayList集合时，需要对其按照年龄进行排序。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.test;
 2 
 3 /**
 4  * Create by YSOcean
 5  */
 6 public class Person {
 7     private String pname;
 8     private Integer page;
 9 
10     public Person() {
11     }
12 
13     public Person(String pname, Integer page) {
14         this.pname = pname;
15         this.page = page;
16     }
17 
18     public String getPname() {
19         return pname;
20     }
21 
22     public void setPname(String pname) {
23         this.pname = pname;
24     }
25 
26     public Integer getPage() {
27         return page;
28     }
29 
30     public void setPage(Integer page) {
31         this.page = page;
32     }
33 
34     @Override
35     public String toString() {
36         return "Person{" +
37                 "pname='" + pname + '\'' +
38                 ", page=" + page +
39                 '}';
40     }
41 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　排序代码：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 List<Person> personList = new ArrayList<>();
 2 personList.add(new Person("李四",20));
 3 personList.add(new Person("张三",10));
 4 personList.add(new Person("王五",30));
 5 System.out.println("原始顺序为："+personList.toString());
 6 Collections.sort(personList, new Comparator<Person>() {
 7     @Override
 8     public int compare(Person o1, Person o2) {
 9         //升序
10         //return o1.getPage()-o2.getPage();
11         //降序
12         return o2.getPage()-o1.getPage();
13         //不变
14         //return 0
15     }
16 });
17 System.out.println("排序后顺序为："+personList.toString());
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　打印结果为：

　　![img](https://img2018.cnblogs.com/blog/1120165/201902/1120165-20190220214637269-349676356.png)

　　②、Entry

```
private transient Entry<K,V> root;
```

　　对于Entry详细源码如下：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     static final class Entry<K,V> implements Map.Entry<K,V> {
 2         K key;
 3         V value;
 4         Entry<K,V> left;
 5         Entry<K,V> right;
 6         Entry<K,V> parent;
 7         boolean color = BLACK;
 8 
 9         /**
10          * Make a new cell with given key, value, and parent, and with
11          * {@code null} child links, and BLACK color.
12          */
13         Entry(K key, V value, Entry<K,V> parent) {
14             this.key = key;
15             this.value = value;
16             this.parent = parent;
17         }
18 
19         /**
20          * Returns the key.
21          *
22          * @return the key
23          */
24         public K getKey() {
25             return key;
26         }
27 
28         /**
29          * Returns the value associated with the key.
30          *
31          * @return the value associated with the key
32          */
33         public V getValue() {
34             return value;
35         }
36 
37         /**
38          * Replaces the value currently associated with the key with the given
39          * value.
40          *
41          * @return the value associated with the key before this method was
42          *         called
43          */
44         public V setValue(V value) {
45             V oldValue = this.value;
46             this.value = value;
47             return oldValue;
48         }
49 
50         public boolean equals(Object o) {
51             if (!(o instanceof Map.Entry))
52                 return false;
53             Map.Entry<?,?> e = (Map.Entry<?,?>)o;
54 
55             return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
56         }
57 
58         public int hashCode() {
59             int keyHash = (key==null ? 0 : key.hashCode());
60             int valueHash = (value==null ? 0 : value.hashCode());
61             return keyHash ^ valueHash;
62         }
63 
64         public String toString() {
65             return key + "=" + value;
66         }
67     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这里主要看 Entry 类的几个字段：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
K key;
V value;
Entry<K,V> left;
Entry<K,V> right;
Entry<K,V> parent;
boolean color = BLACK;
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　相信对[红黑树](https://www.cnblogs.com/ysocean/p/8004211.html)这种数据结构了解的人，一看这几个字段就明白了，这也印证了前面所说的TreeMap底层有红黑树这种数据结构。

　　③、size

```
    /**
     * The number of entries in the tree
     */
    private transient int size = 0;
```

　　用来表示entry的个数，也就是key-value的个数。

　　④、modCount

```
    /**
     * The number of structural modifications to the tree.
     */
    private transient int modCount = 0;
```

　　基本上前面讲的在ArrayList,LinkedList,HashMap等线程不安全的集合都有此字段，用来实现Fail-Fast 机制，如果在迭代这些集合的过程中，有其他线程修改了这些集合，就会抛出ConcurrentModificationException异常。

　　⑤、红黑树常量

```
    private static final boolean RED   = false;
    private static final boolean BLACK = true;
```

[回到顶部](https://www.cnblogs.com/ysocean/p/10300976.html#_labelTop)

### 3、构造函数

　　①、无参构造函数

```
1     public TreeMap() {
2         comparator = null;
3     }
```

　　将比较器 comparator 置为 null，表示按照key的自然顺序进行排序。

　　②、带比较器的构造函数

```
1     public TreeMap(Comparator<? super K> comparator) {
2         this.comparator = comparator;
3     }
```

　　需要自己实现Comparator。

　　③、构造包含指定map集合的元素

```
1     public TreeMap(Map<? extends K, ? extends V> m) {
2         comparator = null;
3         putAll(m);
4     }
```

　　使用该构造器创建的TreeMap,会默认插入m表示的集合元素，并且comparator表示按照自然顺序进行插入。

　　④、带 SortedMap的构造函数

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     public TreeMap(SortedMap<K, ? extends V> m) {
2         comparator = m.comparator();
3         try {
4             buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
5         } catch (java.io.IOException cannotHappen) {
6         } catch (ClassNotFoundException cannotHappen) {
7         }
8     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　和上面带Map的构造函数不一样，map是无序的，而SortedMap 是有序的，使用 buildFromSorted() 方法将SortedMap集合中的元素插入到TreeMap 中。

[回到顶部](https://www.cnblogs.com/ysocean/p/10300976.html#_labelTop)

### 4、添加元素

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     //添加元素
 2     public V put(K key, V value) {
 3         TreeMap.Entry<K,V> t = root;
 4         //如果根节点为空，即TreeMap中一个元素都没有，那么设置新添加的元素为根节点
 5         //并且设置集合大小size=1,以及modCount+1，这是用于快速失败
 6         if (t == null) {
 7             compare(key, key); // type (and possibly null) check
 8 
 9             root = new TreeMap.Entry<>(key, value, null);
10             size = 1;
11             modCount++;
12             return null;
13         }
14         int cmp;
15         TreeMap.Entry<K,V> parent;
16         // split comparator and comparable paths
17         Comparator<? super K> cpr = comparator;
18         //如果比较器不为空，即初始化TreeMap构造函数时，有传递comparator类
19         //那么插入新的元素时，按照comparator实现的类进行排序
20         if (cpr != null) {
21             //通过do-while循环不断遍历树，调用比较器对key值进行比较
22             do {
23                 parent = t;
24                 cmp = cpr.compare(key, t.key);
25                 if (cmp < 0)
26                     t = t.left;
27                 else if (cmp > 0)
28                     t = t.right;
29                 else
30                     //遇到key相等，直接将新值覆盖到原值上
31                     return t.setValue(value);
32             } while (t != null);
33         }
34         //如果比较器为空，即初始化TreeMap构造函数时，没有传递comparator类
35         //那么插入新的元素时，按照key的自然顺序
36         else {
37             //如果key==null，直接抛出异常
38             //注意，上面构造TreeMap传入了Comparator，是可以允许key==null
39             if (key == null)
40                 throw new NullPointerException();
41             @SuppressWarnings("unchecked")
42             Comparable<? super K> k = (Comparable<? super K>) key;
43             do {
44                 parent = t;
45                 cmp = k.compareTo(t.key);
46                 if (cmp < 0)
47                     t = t.left;
48                 else if (cmp > 0)
49                     t = t.right;
50                 else
51                     return t.setValue(value);
52             } while (t != null);
53         }
54         //找到父亲节点，根据父亲节点创建一个新节点
55         TreeMap.Entry<K,V> e = new TreeMap.Entry<>(key, value, parent);
56         if (cmp < 0)
57             parent.left = e;
58         else
59             parent.right = e;
60         //修正红黑树（包括节点的左旋和右旋，具体可以看我Java数据结构和算法中对红黑树的介绍）
61         fixAfterInsertion(e);
62         size++;
63         modCount++;
64         return null;
65     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　添加元素，如果初始化TreeMap构造函数时，没有传递comparator类，是不允许插入key==null的键值对的，相反，如果实现了Comparator，则可以传递key=null的键值对。

　　另外，当插入一个新的元素后（除了根节点），会对TreeMap数据结构进行修正，也就是对红黑树进行修正，使其满足红黑树的几个特点，具体修正方法包括改变节点颜色，左旋，右旋等操作，这里我不做详细介绍了，具体可以参考我的这篇博客：https://www.cnblogs.com/ysocean/p/8004211.html

[回到顶部](https://www.cnblogs.com/ysocean/p/10300976.html#_labelTop)

### 5、删除元素

　　①、根据key删除

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public V remove(Object key) {
 2         //根据key找到该节点
 3         TreeMap.Entry<K,V> p = getEntry(key);
 4         if (p == null)
 5             return null;
 6         //获取该节点的value，并返回
 7         V oldValue = p.value;
 8         //调用deleteEntry()方法删除节点
 9         deleteEntry(p);
10         return oldValue;
11     }
12 
13     private void deleteEntry(TreeMap.Entry<K,V> p) {
14         modCount++;
15         size--;
16 
17         //如果删除节点的左右节点都不为空，即有两个孩子
18         if (p.left != null && p.right != null) {
19             //得到该节点的中序后继节点
20             TreeMap.Entry<K,V> s = successor(p);
21             p.key = s.key;
22             p.value = s.value;
23             p = s;
24         } // p has 2 children
25 
26         // Start fixup at replacement node, if it exists.
27         TreeMap.Entry<K,V> replacement = (p.left != null ? p.left : p.right);
28         //待删除节点只有一个子节点，直接删除该节点，并用该节点的唯一子节点顶替该节点
29         if (replacement != null) {
30             // Link replacement to parent
31             replacement.parent = p.parent;
32             if (p.parent == null)
33                 root = replacement;
34             else if (p == p.parent.left)
35                 p.parent.left  = replacement;
36             else
37                 p.parent.right = replacement;
38 
39             // Null out links so they are OK to use by fixAfterDeletion.
40             p.left = p.right = p.parent = null;
41 
42             // Fix replacement
43             if (p.color == BLACK)
44                 fixAfterDeletion(replacement);
45 
46             //TreeMap中只有待删除节点P，也就是只有一个节点，直接返回nul即可
47         } else if (p.parent == null) { // return if we are the only node.
48             root = null;
49         } else { //  No children. Use self as phantom replacement and unlink.
50             //待删除节点没有子节点，即为叶子节点，直接删除即可
51             if (p.color == BLACK)
52                 fixAfterDeletion(p);
53 
54             if (p.parent != null) {
55                 if (p == p.parent.left)
56                     p.parent.left = null;
57                 else if (p == p.parent.right)
58                     p.parent.right = null;
59                 p.parent = null;
60             }
61         }
62     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　删除节点分为四种情况：

　　1、根据key没有找到该节点：也就是集合中不存在这一个节点，直接返回null即可。

　　2、根据key找到节点，又分为三种情况：

　　　　①、待删除节点没有子节点，即为叶子节点：直接删除该节点即可。

　　　　②、待删除节点只有一个子节点：那么首先找到待删除节点的子节点，然后删除该节点，用其唯一子节点顶替该节点。

　　　　③、待删除节点有两个子节点：首先找到该节点的中序后继节点，然后把这个后继节点的内容复制给待删除节点，然后删除该中序后继节点，删除过程又转换成前面①、②两种情况了，这里主要是找到中序后继节点，相当于待删除节点的一个替身。

[回到顶部](https://www.cnblogs.com/ysocean/p/10300976.html#_labelTop)

### 6、查找元素

　　①、根据key查找

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     public V get(Object key) {
 2         TreeMap.Entry<K,V> p = getEntry(key);
 3         return (p==null ? null : p.value);
 4     }
 5 
 6     final TreeMap.Entry<K,V> getEntry(Object key) {
 7         // Offload comparator-based version for sake of performance
 8         if (comparator != null)
 9             return getEntryUsingComparator(key);
10         if (key == null)
11             throw new NullPointerException();
12         @SuppressWarnings("unchecked")
13         Comparable<? super K> k = (Comparable<? super K>) key;
14         TreeMap.Entry<K,V> p = root;
15         while (p != null) {
16             int cmp = k.compareTo(p.key);
17             if (cmp < 0)
18                 p = p.left;
19             else if (cmp > 0)
20                 p = p.right;
21             else
22                 return p;
23         }
24         return null;
25     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

[回到顶部](https://www.cnblogs.com/ysocean/p/10300976.html#_labelTop)

### 7、遍历元素

　　通常有下面两种方法，第二种方法效率要快很多。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 TreeMap<String,Integer> map = new TreeMap<>();
 2 map.put("A",1);
 3 map.put("B",2);
 4 map.put("C",3);
 5 
 6 //第一种方法
 7 //首先利用keySet()方法得到key的集合，然后利用map.get()方法根据key得到value
 8 Iterator<String> iterator = map.keySet().iterator();
 9 while(iterator.hasNext()){
10     String key = iterator.next();
11     System.out.println(key+":"+map.get(key));
12 }
13 
14 //第二种方法
15 Iterator<Map.Entry<String,Integer>> iterator1 = map.entrySet().iterator();
16 while(iterator1.hasNext()){
17     Map.Entry<String,Integer> entry = iterator1.next();
18     System.out.println(entry.getKey()+":"+entry.getValue());
19 }
```



# [Java 集合详解](https://www.cnblogs.com/ysocean/p/6555373.html)

一、集合的由来

　　通常，我们的程序需要根据程序运行时才知道创建多少个对象。但若非程序运行，程序开发阶段，我们根本不知道到底需要多少个数量的对象，甚至不知道它的准确类型。为了满足这些常规的编程需要，我们要求能在任何时候，任何地点创建任意数量的对象，而这些对象用什么来容纳呢？我们首先想到了数组，但是数组只能放统一类型的数据，而且其长度是固定的，那怎么办呢？集合便应运而生了！

 

为了对集合有个更加深入的了解，可以看我的这一篇文章：**用 Java 数组来实现 ArrayList 集合** http://www.cnblogs.com/ysocean/p/6812674.html

 

二、集合是什么？

　　Java集合类存放于 java.util 包中，是一个用来存放对象的容器。

注意：①、集合只能存放对象。比如你存一个 int 型数据 1放入集合中，其实它是自动转换成 Integer 类后存入的，Java中每一种基本类型都有对应的引用类型。

　　　②、集合存放的是多个对象的引用，对象本身还是放在堆内存中。

　　　③、集合可以存放不同类型，不限数量的数据类型。

 

三、Java 集合框架图

![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170505143634711-1395541187.png)

**此图来源于：http://blog.csdn.net/u010887744/article/details/50575735**

**大图可以点此访问：http://img.blog.csdn.net/20160124221843905**

 

　　发现一个特点，上述所有的集合类，除了 map 系列的集合，即左边集合都实现了 **Iterator** 接口，这是一个用于遍历集合中元素的接口，主要hashNext(),next(),remove()三种方法。它的一个子接口 **ListIterator** 在它的基础上又添加了三种方法，分别是 add(),previous(),hasPrevious()。也就是说如果实现 Iterator 接口，那么在遍历集合中元素的时候，只能往后遍历，被遍历后的元素不会再被遍历到，通常无序集合实现的都是这个接口，比如HashSet；而那些元素有序的集合，实现的一般都是 LinkedIterator接口，实现这个接口的集合可以双向遍历，既可以通过next()访问下一个元素，又可以通过previous()访问前一个 元素，比如ArrayList。

　　还有一个特点就是抽象类的使用。如果要自己实现一个集合类，去实现那些抽象的接口会非常麻烦，工作量很大。这个时候就可以使用抽象类，这些抽象类中给我们提供了许多

现成的实现，我们只需要根据自己的需求重写一些方法或者添加一些方法就可以实现自己需要的集合类，工作量大大降低。

 

四、集合详解

①、**Iterator**:迭代器，它是Java集合的顶层接口（不包括 map 系列的集合，Map接口 是 map 系列集合的顶层接口）

　　Object next()：返回迭代器刚越过的元素的引用，返回值是 Object，需要强制转换成自己需要的类型

　　boolean hasNext()：判断容器内是否还有可供访问的元素

　　void remove()：删除迭代器刚越过的元素

所以除了 map 系列的集合，我们都能通过迭代器来对集合中的元素进行遍历。

注意：我们可以在源码中追溯到集合的顶层接口，比如 Collection 接口，可以看到它继承的是类 Iterable

![img](https://images2015.cnblogs.com/blog/1120165/201703/1120165-20170315161812151-117827845.png)

那这就得说明一下 Iterator 和 Iterable 的区别：

 Iterable ：存在于 java.lang 包中。

　　　　![img](https://images2015.cnblogs.com/blog/1120165/201703/1120165-20170315162024541-482452136.png)

我们可以看到，里面封装了 Iterator 接口。所以只要实现了只要实现了Iterable接口的类，就可以使用Iterator迭代器了。

 

 Iterator ：存在于 java.util 包中。核心的方法next(),hasnext(),remove()。

 这里我们引用一个Iterator 的实现类 ArrayList 来看一下迭代器的使用：暂时先不管 List 集合是什么，只需要看看迭代器的用法就行了

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1         //产生一个 List 集合，典型实现为 ArrayList。
 2         List list = new ArrayList();
 3         //添加三个元素
 4         list.add("Tom");
 5         list.add("Bob");
 6         list.add("Marry");
 7         //构造 List 的迭代器
 8         Iterator it = list.iterator();
 9         //通过迭代器遍历元素
10         while(it.hasNext()){
11             Object obj = it.next();
12             System.out.println(obj);
13         }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 

 ②、**Collection**:List 接口和 Set 接口的父接口

　　　　![img](https://images2015.cnblogs.com/blog/1120165/201703/1120165-20170315163013854-1249426924.png)

　　看一下 Collection 集合的使用例子：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1         //我们这里将 ArrayList集合作为 Collection 的实现类
 2         Collection collection = new ArrayList();
 3         
 4         //添加元素
 5         collection.add("Tom");
 6         collection.add("Bob");
 7         
 8         //删除指定元素
 9         collection.remove("Tom");
10         
11         //删除所有元素
12         Collection c = new ArrayList();
13         c.add("Bob");
14         collection.removeAll(c);
15         
16         //检测是否存在某个元素
17         collection.contains("Tom");
18         
19         //判断是否为空
20         collection.isEmpty();
21         
22         //利用增强for循环遍历集合
23         for(Object obj : collection){
24             System.out.println(obj);
25         }
26         //利用迭代器 Iterator
27         Iterator iterator = collection.iterator();
28         while(iterator.hasNext()){
29             Object obj = iterator.next();
30             System.out.println(obj);
31         }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 

 ③、List :有序，可以重复的集合。

　　![img](https://images2015.cnblogs.com/blog/1120165/201703/1120165-20170315164917916-1891467256.png)

由于 List 接口是继承于 Collection 接口，所以基本的方法如上所示。

1、List 接口的三个典型实现：

　　①、List list1 = new ArrayList();

　　　　底层数据结构是数组，查询快，增删慢;线程不安全，效率高

  ②、List list2 = new Vector();

　　　　底层数据结构是数组，查询快，增删慢;线程安全，效率低,几乎已经淘汰了这个集合

　  ③、List list3 = new LinkedList();

　　　　底层数据结构是链表，查询慢，增删快;线程不安全，效率高

 

 怎么记呢？我们可以想象：

　　数组就像身上编了号站成一排的人，要找第10个人很容易，根据人身上的编号很快就能找到。但插入、删除慢，要望某个位置插入或删除一个人时，后面的人身上的编号都要变。当然，加入或删除的人始终末尾的也快。

　　链表就像手牵着手站成一圈的人，要找第10个人不容易，必须从第一个人一个个数过去。但插入、删除快。插入时只要解开两个人的手，并重新牵上新加进来的人的手就可以。删除一样的道理。

 

 2、除此之外，List 接口遍历还可以使用普通 for 循环进行遍历，指定位置添加元素，替换元素等等。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 　　　　 //产生一个 List 集合，典型实现为 ArrayList
 2         List list = new ArrayList();
 3         //添加三个元素
 4         list.add("Tom");
 5         list.add("Bob");
 6         list.add("Marry");
 7         //构造 List 的迭代器
 8         Iterator it = list.iterator();
 9         //通过迭代器遍历元素
10         while(it.hasNext()){
11             Object obj = it.next();
12             //System.out.println(obj);
13         }
14         
15         //在指定地方添加元素
16         list.add(2, 0);
17          
18         //在指定地方替换元素
19         list.set(2, 1);
20          
21         //获得指定对象的索引
22         int i=list.indexOf(1);
23         System.out.println("索引为："+i);
24         
25         //遍历：普通for循环
26         for(int j=0;j<list.size();j++){
27              System.out.println(list.get(j));
28         }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 

 ④、Set：典型实现 HashSet()是一个无序，不可重复的集合

 **1、Set hashSet = new HashSet();**

　　①、HashSet:不能保证元素的顺序；不可重复；不是线程安全的；集合元素可以为 NULL;

　　②、其底层其实是一个数组，存在的意义是加快查询速度。我们知道在一般的数组中，元素在数组中的索引位置是随机的，元素的取值和元素的位置之间不存在确定的关系，因此，在数组中查找特定的值时，需要把查找值和一系列的元素进行比较，此时的查询效率依赖于查找过程中比较的次数。而 HashSet 集合底层数组的索引和值有一个确定的关系：index=hash(value),那么只需要调用这个公式，就能快速的找到元素或者索引。

　　③、对于 HashSet: 如果两个对象通过 equals() 方法返回 true，这两个对象的 hashCode 值也应该相同。

  　　1、当向HashSet集合中存入一个元素时，HashSet会先调用该对象的hashCode（）方法来得到该对象的hashCode值，然后根据hashCode值决定该对象在HashSet中的存储位置

　　　　　　1.1、如果 hashCode 值不同，直接把该元素存储到 hashCode() 指定的位置

　　　　　　1.2、如果 hashCode 值相同，那么会继续判断该元素和集合对象的 equals() 作比较

　　　　　　　　　　1.2.1、hashCode 相同，equals 为 true，则视为同一个对象，不保存在 hashSet（）中

　　　　　　　　　　1.2.2、hashCode 相同，equals 为 false，则存储在之前对象同槽位的链表上，这非常麻烦，我们应该约束这种情况，即保证：如果两个对象通过 equals() 方法返回 true，这两个对象的 hashCode 值也应该相同。

 

**注意：每一个存储到 哈希 表中的对象，都得提供 hashCode() 和 equals() 方法的实现，用来判断是否是同一个对象**

　　　**对于 HashSet 集合，我们要保证如果两个对象通过 equals() 方法返回 true，这两个对象的 hashCode 值也应该相同。**

 

　　

常见的 hashCode()算法：

![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170509001300847-1382384473.png)

 

 

**2、Set linkedHashSet = new LinkedHashSet();**

　　①、不可以重复，有序

　　因为底层采用 链表 和 哈希表的算法。链表保证元素的添加顺序，哈希表保证元素的唯一性

 

 

**3、Set treeSet = new TreeSet();**

　　TreeSet:有序；不可重复，底层使用 红黑树算法，擅长于范围查询。

　　*  如果使用 TreeSet() 无参数的构造器创建一个 TreeSet 对象, 则要求放入其中的元素的类必须实现 Comparable 接口所以, 在其中不能放入 null 元素

 

   \*  **必须放入同样类的对象**.(默认会进行排序) 否则可能会发生类型转换异常.我们可以使用泛型来进行限制

```
Set treeSet = ``new` `TreeSet();``    ``treeSet.add(``1``); ``//添加一个 Integer 类型的数据``    ``treeSet.add(``"a"``);  ``//添加一个 String 类型的数据``    ``System.out.println(treeSet); ``//会报类型转换异常的错误
```

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170509004119019-1767190549.png)

  *　自动排序：添加自定义对象的时候，必须要实现 Comparable 接口，并要覆盖 compareTo(Object obj) 方法来自定义比较规则

　　　　如果 this > obj,返回正数 1

　　　　如果 this < obj,返回负数 -1

　　　　如果 this = obj,返回 0 ，则认为这两个对象相等

 

   　　　　　*  两个对象通过 Comparable 接口 compareTo(Object obj) 方法的返回值来比较大小, 并进行升序排列

![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170509081622488-1398934456.png)

 

 

　　

  \*  定制排序: 创建 TreeSet 对象时, 传入 Comparator 接口的实现类. 要求: Comparator 接口的 compare 方法的返回值和 两个元素的 equals() 方法具有一致的返回值  

```
public` `class` `TreeSetTest {``  ``public` `static` `void` `main(String[] args) {``    ``Person p1 = ``new` `Person(``1``);``    ``Person p2 = ``new` `Person(``2``);``    ``Person p3 = ``new` `Person(``3``);``    ` `    ``Set set = ``new` `TreeSet<>(``new` `Person());``    ``set.add(p1);``    ``set.add(p2);``    ``set.add(p3);``    ``System.out.println(set); ``//结果为[1, 2, 3]``  ``}` `}` `class` `Person ``implements` `Comparator{``  ``public` `int` `age;``  ``public` `Person(){}``  ``public` `Person(``int` `age){``    ``this``.age = age;``  ``}``  ``@Override``  ``/***``   ``* 根据年龄大小进行排序``   ``*/``  ``public` `int` `compare(Person o1, Person o2) {``    ``// TODO Auto-generated method stub``    ``if``(o1.age > o2.age){``      ``return` `1``;``    ``}``else` `if``(o1.age < o2.age){``      ``return` `-``1``;``    ``}``else``{``      ``return` `0``;``    ``}``  ``}``  ` `  ``@Override``  ``public` `String toString() {``    ``// TODO Auto-generated method stub``    ``return` `""``+``this``.age;``  ``}``}
```

　　

 

   \*  当需要把一个对象放入 TreeSet 中，重写该对象对应的 equals() 方法时，应保证该方法与 compareTo(Object obj) 方法有一致的结果

  

  

以上三个 Set 接口的实现类比较：

　　共同点：1、都不允许元素重复

　　　　　　2、都不是线程安全的类，解决办法：Set set = Collections.synchronizedSet(set 对象)

 

 

　　不同点：

　　　　HashSet:不保证元素的添加顺序，底层采用 哈希表算法，查询效率高。判断两个元素是否相等，equals() 方法返回 true,hashCode() 值相等。即要求存入 HashSet 中的元素要覆盖 equals() 方法和 hashCode()方法

 

　　　　LinkedHashSet:HashSet 的子类，底层采用了 哈希表算法以及 链表算法，既保证了元素的添加顺序，也保证了查询效率。但是整体性能要低于 HashSet　　　　

 

　　　　TreeSet:不保证元素的添加顺序，但是会对集合中的元素进行排序。底层采用 红-黑 树算法（树结构比较适合范围查询）

 

 

 

 

　　⑤、Map：key-value 的键值对，key 不允许重复，value 可以

　　1、严格来说 Map 并不是一个集合，而是两个集合之间 的映射关系。

　　  2、这两个集合没每一条数据通过映射关系，我们可以看成是一条数据。即 Entry(key,value）。Map 可以看成是由多个 Entry 组成。

　　  3、因为 Map 集合即没有实现于 Collection 接口，也没有实现 Iterable 接口，所以不能对 Map 集合进行 for-each 遍历。

　　　　　　　　　　　　

　　　　　　　　　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170509215438113-1495103512.png)

 

```
Map hashMap = ``new` `HashMap<>();``    ``//添加元素到 Map 中``    ``hashMap.put(``"key1"``, ``"value1"``);``    ``hashMap.put(``"key2"``, ``"value2"``);``    ``hashMap.put(``"key3"``, ``"value3"``);``    ``hashMap.put(``"key4"``, ``"value4"``);``    ``hashMap.put(``"key5"``, ``"value5"``);``    ` `    ``//删除 Map 中的元素,通过 key 的值``    ``hashMap.remove(``"key1"``);``    ` `    ``//通过 get(key) 得到 Map 中的value``    ``Object str1 = hashMap.get(``"key1"``);``    ` `    ``//可以通过 添加 方法来修改 Map 中的元素``    ``hashMap.put(``"key2"``, ``"修改 key2 的 Value"``);``    ` `    ``//通过 map.values() 方法得到 Map 中的 value 集合``    ``Collection value = hashMap.values();``    ``for``(Object obj : value){``      ``//System.out.println(obj);``    ``}``    ` `    ``//通过 map.keySet() 得到 Map 的key 的集合，然后 通过 get(key) 得到 Value``    ``Set set = hashMap.keySet();``    ``for``(String str : set){``      ``Object obj = hashMap.get(str);``      ``//System.out.println(str+"="+obj);``    ``}``    ` `    ``//通过 Map.entrySet() 得到 Map 的 Entry集合，然后遍历``    ``Set> entrys = hashMap.entrySet();``    ``for``(Map.Entry entry: entrys){``      ``String key = entry.getKey();``      ``Object value2 = entry.getValue();``      ``System.out.println(key+``"="``+value2);``    ``}``    ` `    ``System.out.println(hashMap);
```

　　Map 的常用实现类：

　　　　　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170509224035660-1080038371.png)

 

 

 

 

　　⑥、Map 和 Set 集合的关系

　　　　1、都有几个类型的集合。HashMap 和 HashSet ，都采 哈希表算法；TreeMap 和 TreeSet 都采用 红-黑树算法；LinkedHashMap 和 LinkedHashSet 都采用 哈希表算法和红-黑树算法。

　　　　2、分析 Set 的底层源码，我们可以看到，Set 集合 就是 由 Map 集合的 Key 组成。

　　　　　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170509223239738-2032105541.png)

# [Java关键字(一)——instanceof](https://www.cnblogs.com/ysocean/p/8486500.html)



**目录**

- [1、obj 必须为引用类型，不能是基本类型](https://www.cnblogs.com/ysocean/p/8486500.html#_label0)
- [2、obj 为 null](https://www.cnblogs.com/ysocean/p/8486500.html#_label1)
- [3、obj 为 class 类的实例对象](https://www.cnblogs.com/ysocean/p/8486500.html#_label2)
- [4、obj 为 class 接口的实现类](https://www.cnblogs.com/ysocean/p/8486500.html#_label3)
- [5、obj 为 class 类的直接或间接子类](https://www.cnblogs.com/ysocean/p/8486500.html#_label4)
- [6、问题](https://www.cnblogs.com/ysocean/p/8486500.html#_label5)
- [7、深究原理](https://www.cnblogs.com/ysocean/p/8486500.html#_label6)
- [8、instanceof 的实现策略](https://www.cnblogs.com/ysocean/p/8486500.html#_label7)

 

------

　　instanceof 严格来说是Java中的一个双目运算符，用来测试一个对象是否为一个类的实例，用法为：

```
boolean` `result = obj ``instanceof` `Class
```

　　其中 obj 为一个对象，Class 表示一个类或者一个接口，当 obj 为 Class 的对象，或者是其直接或间接子类，或者是其接口的实现类，结果result 都返回 true，否则返回false。

　　注意：编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定类型，则通过编译，具体看运行时定。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 1、obj 必须为引用类型，不能是基本类型

```
int` `i = ``0``;``System.out.println(i ``instanceof` `Integer);``//编译不通过``System.out.println(i ``instanceof` `Object);``//编译不通过
```

　　instanceof 运算符只能用作对象的判断。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 2、obj 为 null

```
System.out.println(``null` `instanceof` `Object);``//false
```

　　关于 null 类型的描述在官方文档：https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.1 有一些介绍。一般我们知道Java分为两种数据类型，一种是基本数据类型，有八个分别是 byte short int long float double char boolean,一种是引用类型，包括类，接口，数组等等。而Java中还有一种特殊的 null 类型，该类型没有名字，所以不可能声明为 null 类型的变量或者转换为 null 类型，null 引用是 null 类型表达式唯一可能的值，null 引用也可以转换为任意引用类型。我们不需要对 null 类型有多深刻的了解，我们只需要知道 null 是可以成为任意引用类型的**特殊符号**。

　　在 [JavaSE规范](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.20.2) 中对 instanceof 运算符的规定就是：如果 obj 为 null，那么将返回 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 3、obj 为 class 类的实例对象

```
Integer integer = ``new` `Integer(``1``);``System.out.println(integer ``instanceof` `Integer);``//true
```

　　这没什么好说的，最普遍的一种用法。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 4、obj 为 class 接口的实现类

　　了解Java 集合的，我们知道集合中有个上层接口 List，其有个典型实现类 ArrayList

```
public` `class` `ArrayList ``extends` `AbstractList``    ``implements` `List, RandomAccess, Cloneable, java.io.Serializable
```

　　所以我们可以用 instanceof 运算符判断 某个对象是否是 List 接口的实现类，如果是返回 true，否则返回 false

```
ArrayList arrayList = ``new` `ArrayList();``System.out.println(arrayList ``instanceof` `List);``//true
```

　　或者反过来也是返回 true

```
List list = ``new` `ArrayList();``System.out.println(list ``instanceof` `ArrayList);``//true
```

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 5、obj 为 class 类的直接或间接子类

　　我们新建一个父类 Person.class，然后在创建它的一个子类 Man.class

```
public` `class` `Person {` `}
```

　　Man.class

```
public` `class` `Man ``extends` `Person{``  ` `}
```

　　测试：

```
Person p1 = ``new` `Person();``Person p2 = ``new` `Man();``Man m1 = ``new` `Man();``System.out.println(p1 ``instanceof` `Man);``//false``System.out.println(p2 ``instanceof` `Man);``//true``System.out.println(m1 ``instanceof` `Man);``//true
```

　　注意第一种情况， p1 instanceof Man ，Man 是 Person 的子类，Person 不是 Man 的子类，所以返回结果为 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 6、问题

　　前面我们说过**编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定类型，则通过编译，具体看运行时定。**

　　看如下几个例子：

```
Person p1 = ``new` `Person();` `System.out.println(p1 ``instanceof` `String);``//编译报错``System.out.println(p1 ``instanceof` `List);``//false``System.out.println(p1 ``instanceof` `List);``//false``System.out.println(p1 ``instanceof` `List);``//编译报错
```

　　按照我们上面的说法，这里就存在问题了，Person 的对象 p1 很明显不能转换为 String 对象，那么自然 Person 的对象 p1 instanceof String 不能通过编译，但为什么 p1 instanceof List 却能通过编译呢？而 instanceof List<Person> 又不能通过编译了？

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 7、深究原理

　　我们可以看Java语言规范Java SE 8 版：https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.20.2

![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180302000448613-26394231.png)

 　如果用伪代码描述：

```
boolean` `result;``if` `(obj == ``null``) {`` ``result = ``false``;``} ``else` `{`` ``try` `{``   ``T temp = (T) obj; ``// checkcast``   ``result = ``true``;`` ``} ``catch` `(ClassCastException e) {``   ``result = ``false``;`` ``}``}
```

　　也就是说有表达式 obj instanceof T，instanceof 运算符的 obj 操作数的类型必须是引用类型或空类型; 否则，会发生编译时错误。 

　　如果 obj 强制转换为 T 时发生编译错误，则关系表达式的 instanceof 同样会产生编译时错误。 在这种情况下，表达式实例的结果永远为false。

　　在运行时，如果 T 的值不为null，并且 obj 可以转换为 T 而不引发ClassCastException，则instanceof运算符的结果为true。 否则结果是错误的

　　简单来说就是：**如果 obj 不为 null 并且 (T) obj 不抛 ClassCastException 异常则该表达式值为 true ，否则值为 false 。**

　　所以对于上面提出的问题就很好理解了，为什么 p1 instanceof String 编译报错，因为(String)p1 是不能通过编译的，而 (List)p1 可以通过编译。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 8、instanceof 的实现策略

　　JavaSE 8 instanceof 的实现算法：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180302002919162-2045599504.png)

 

　　1、obj如果为null，则返回false；否则设S为obj的类型对象，剩下的问题就是检查S是否为T的子类型；

　　2、如果S == T，则返回true；

　　3、接下来分为3种情况，之所以要分情况是因为instanceof要做的是“子类型检查”，而Java语言的类型系统里数组类型、接口类型与普通类类型三者的子类型规定都不一样，必须分开来讨论。

　　①、S是数组类型：如果 T 是一个类类型，那么T必须是Object；如果 T 是接口类型，那么 T 必须是由数组实现的接口之一；

　　②、接口类型：对接口类型的 instanceof 就直接遍历S里记录的它所实现的接口，看有没有跟T一致的；

　　③、类类型：对类类型的 instanceof 则是遍历S的super链（继承链）一直到Object，看有没有跟T一致的。遍历类的super链意味着这个算法的性能会受类的继承深度的影响。

参考链接：https://www.zhihu.com/question/21574535　　



# [Java关键字(二)——native](https://www.cnblogs.com/ysocean/p/8476933.html)



**目录**

- [1、JNI：Java Native Interface](https://www.cnblogs.com/ysocean/p/8476933.html#_label0)
- 3、用C语言编写程序本地方法
  - [　　一、编写带有 native 声明的方法的java类](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_0)
  - [　　二、使用 javac 命令编译所编写的java类，生成.class文件](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_1)
  - [　　三、使用 javah -jni java类名 生成扩展名为 h 的头文件](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_2)
  - [　　四、使用C语言实现本地方法](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_3)
- [4、JNI调用C的流程图](https://www.cnblogs.com/ysocean/p/8476933.html#_label2)
- [5、native关键字](https://www.cnblogs.com/ysocean/p/8476933.html#_label3)

 

------

　　本篇博客我们将介绍Java中的一个关键字——native。

　　native 关键字在 JDK 源码中很多类中都有，在 Object.java类中，其 getClass() 方法、hashCode()方法、clone() 方法等等都是用 native 关键字修饰的。

```
public` `final` `native` `Class getClass();``public` `native` `int` `hashCode();``protected` `native` `Object clone() ``throws` `CloneNotSupportedException;
```

　　那么为什么要用 native 来修饰方法，这样做有什么用？

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 1、JNI：Java Native Interface

　　在介绍 native 之前，我们先了解什么是 JNI。

　　一般情况下，我们完全可以使用 Java 语言编写程序，但某些情况下，Java 可能会不满足应用程序的需求，或者是不能更好的满足需求，比如：

　　①、标准的 Java 类库不支持应用程序平台所需的平台相关功能。

　　②、我们已经用另一种语言编写了一个类库，如何用Java代码调用？

　　③、某些运行次数特别多的方法代码，为了加快性能，我们需要用更接近硬件的语言（比如汇编）编写。

　　上面这三种需求，其实说到底就是如何用 Java 代码调用不同语言编写的代码。那么 JNI 应运而生了。

　　从Java 1.1开始，Java Native Interface (JNI)标准就成为java平台的一部分，它允许Java代码和其他语言写的代码进行交互。JNI一开始是为了本地已编译语言，尤其是C和C++而设计 的，但是它并不妨碍你使用其他语言，只要调用约定受支持就可以了。使用java与本地已编译的代码交互，通常会丧失平台可移植性。但是，有些情况下这样做是可以接受的，甚至是必须的，比如，使用一些旧的库，与硬件、操作系统进行交互，或者为了提高程序的性能。JNI标准至少保证本地代码能工作在任何Java 虚拟机实现下。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180307233528546-1790949526.png)

　　通过 JNI，我们就可以通过 Java 程序（代码）调用到操作系统相关的技术实现的库函数，从而与其他技术和系统交互，使用其他技术实现的系统的功能；同时其他技术和系统也可以通过 JNI 提供的相应原生接口开调用 Java 应用系统内部实现的功能。

　　在windows系统上，一般可执行的应用程序都是基于 native 的PE结构，windows上的 JVM 也是基于native结构实现的。Java应用体系都是构建于 JVM 之上。

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180307235217786-1681048194.png)

　　可能有人会问，Java不是跨平台的吗？如果用 JNI，那么程序不就将失去跨平台的优点?确实是这样的。

　　**JNI 的缺点：**

　　①、程序不再跨平台。要想跨平台，必须在不同的系统环境下重新编译本地语言部分。

　　②、程序不再是绝对安全的，本地代码的不当使用可能导致整个程序崩溃。一个通用规则是，你应该让本地方法集中在少数几个类当中。这样就降低了JAVA和C之间的耦合性。

 　目前来讲使用 JNI 的缺点相对于优点还是可以接受的，可能后面随着 Java 的技术发展，我们不在需要 JNI，但是目前 JDK 还是一直提供对 JNI 标准的支持。

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 3、用C语言编写程序本地方法

　　上面讲解了什么是 JNI，那么我们接下来就写个例子，如何用 Java 代码调用本地的 C 程序。

　　官方文档如下：https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html

　　步骤如下：

　　①、编写带有 **native** 声明的方法的java类，生成.java文件；**(****注意这里出现了 native 声明的方法关键字）**

　　②、使用 **javac** 命令编译所编写的java类，生成.class文件；

　　③、使用 **javah -jni java类名** 生成扩展名为 h 的头文件，也即生成.h文件；

　　④、使用C/C++（或者其他编程想语言）实现本地方法，创建.h文件的实现，也就是创建.cpp文件实现.h文件中的方法；

　　⑤、将C/C++编写的文件生成动态连接库，生成dll文件；

　　下面我们通过一个 HelloWorld 程序的调用来完成这几个步骤。

　　注意：下面所有操作都是在所有操作都是在目录：D:\JNI 下进行的。



#### 　　一、编写带有 **native** 声明的方法的java类

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　用 native 声明的方法表示告知 JVM 调用，该方法在外部定义，也就是我们会用 C 语言去实现。

　　System.loadLibrary("helloJNI");加载动态库，参数 helloJNI 是动态库的名字。我们可以这样理解：程序中的方法 helloJNI() 在程序中没有实现，但是我们下面要调用这个方法，怎么办呢？我们就需要对这个方法进行初始化，所以用 static 代码块进行初始化。

　　这时候如果我们直接运行该程序，会报“A Java Exception has occurred”错误：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308002259932-1479682705.png)



#### 　　二、使用 **javac** 命令编译所编写的java类，生成.class文件

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003043572-1546432391.png)

　　执行上述命令后，生成 HelloJNI.class 文件：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003151699-1569400870.png)



#### 　　三、使用 **javah -jni java类名** 生成扩展名为 h 的头文件

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003250559-2084750367.png)

　　执行上述命令后，在 D:/JNI 目录下多出了个 HelloJNI.h 文件：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003335427-592698657.png)



#### 　　四、使用C语言实现本地方法

　　如果不想安装visual studio 的，我们需要在 windows平台安装 gcc。

　　安装教程如下：http://blog.csdn.net/altland/article/details/63252757

　　注意安装版本的选择，根据系统是32位还是64位来选择。[64位点击下载](https://sourceforge.net/projects/mingw-w64/files/latest/download?source=files)。

　　安装完成之后注意配置环境变量，在 cmd 中输入 g++ -v，如果出现如下信息，则安装配置完成：

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308013743763-382483401.png)

　　接着输入如下命令：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

```
gcc -m64  -Wl,--add-stdcall-alias -I"C:\Program Files\Java\jdk1.8.0_152\include" -I"C:\Program Files\Java\jdk1.8.0_152\include\include\win32" -shared -o helloJNI.dll helloJNI.c
```

　　-m64表示生成dll库是64位的。后面的路径表示本机安装的JDK路径。生成之后多了一个helloJNI.dll 文件

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308215629249-636146276.png)

　　最后运行 HelloJNI：输出 Hello JNI! **大功告成。**

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308014528999-1880600112.png)

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 4、JNI调用C的流程图

　　![img](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308220935516-668760521.png)

　　图片引用自：https://www.cnblogs.com/Qian123/p/5702574.html

　　

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 5、native关键字

　　通过上面介绍了那么多JNI的知识，终于到介绍本篇文章的主角——native 关键字了。相信大家看完上面的介绍，应该也是知道什么是 native 了吧。

　　native 用来修饰方法，用 native 声明的方法表示告知 JVM 调用，该方法在外部定义，我们可以用任何语言去实现它。 简单地讲，一个native Method就是一个 Java 调用非 Java 代码的接口。

　　native 语法：

　　①、修饰方法的位置必须在返回类型之前，和其余的方法控制符前后关系不受限制。

　　②、不能用 abstract 修饰，也没有方法体，也没有左右大括号。

　　③、返回值可以是任意类型

　　我们在日常编程中看到native修饰的方法，只需要知道这个方法的作用是什么，至于别的就不用管了，操作系统会给我们实现。

　　

参考文档：https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html　

　　　　　https://www.cnblogs.com/Qian123/p/5702574.html



# [Java关键字(三)——static](https://www.cnblogs.com/ysocean/p/9202016.html)



**目录**

- [1、修饰成员变量](https://www.cnblogs.com/ysocean/p/9202016.html#_label0)
- [2、修饰修饰成员方法](https://www.cnblogs.com/ysocean/p/9202016.html#_label1)
- [3、静态代码块](https://www.cnblogs.com/ysocean/p/9202016.html#_label2)
- [4、静态导包](https://www.cnblogs.com/ysocean/p/9202016.html#_label3)
- [5、静态内部类](https://www.cnblogs.com/ysocean/p/9202016.html#_label4)
- [6、常见问题](https://www.cnblogs.com/ysocean/p/9202016.html#_label5)

 

------

　　我们说Java是一种面向对象编程的语言，而对象是把数据及对数据的操作方法放在一起，作为一个相互依存的整体，对同类对象抽象出其共性，便是Java中的类，我们可以用类描述世间万物，也可以说万物皆对象。但是这里有个特殊的东西——static，它不属于对象，那么为什么呢？

　　static 是Java的一个关键字，可以用来修饰成员变量、修饰成员方法、构造静态代码块、实现静态导包以及实现静态内部类，下面我们来分别介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202016.html#_labelTop)

### 1、修饰成员变量

　　用 static 修饰成员变量可以说是该关键字最常用的一个功能，通常将用 static 修饰的成员变量称为类成员或者静态成员，那么静态成员和不用 static 修饰的非静态成员有什么区别呢？

　　我们先看看不用 static 修饰的成员变量在内存中的构造。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.bean;
 2 
 3 /**
 4  * Create by YSOcean
 5  */
 6 public class Person {
 7     private String name;
 8     private Integer age;
 9 
10     public Person(String name, Integer age) {
11         this.name = name;
12         this.age = age;
13     }
14 
15     @Override
16     public String toString() {
17         return "Person{" +
18                 "name='" + name + '\'' +
19                 ", age=" + age +
20                 '}';
21     }
22     //get和set方法省略
23 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　首先，我们创建一个实体类 Person，有两个属性 name 和 age，都是普通成员变量（没有用 static 关键字修饰），接着我们通过其构造方法创建两个对象：

```
1 Person p1 = new Person("Tom",21);
2 Person p2 = new Person("Marry",20);
3 System.out.println(p1.toString());//Person{name='Tom', age=21}
4 System.out.println(p2.toString());//Person{name='Marry', age=20}
```

　　这两个对象在内存中的存储结构如下：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180620210908837-557591297.png)

　　由上图可知，我们创建的两个对象 p1 和 p2 存储在堆中，但是其引用地址是存放在栈中的，而且这两个对象的两个变量互相独立，我们修改任何一个对象的属性值，是不改变另外一个对象的属性值的。

　　下面我们将 Person 类中的 age 属性改为由 static 关键字修饰：

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.bean;
 2 
 3 /**
 4  * Create by YSOcean
 5  */
 6 public class Person {
 7     private  String name;
 8     private static Integer age;
 9 
10     public Person(String name, Integer age) {
11         this.name = name;
12         this.age = age;
13     }
14 
15     @Override
16     public String toString() {
17         return "Person{" +
18                 "name='" + name + '\'' +
19                 ", age=" + age +
20                 '}';
21     }
22     //get和set方法省略
23 
24 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　同样我们还是向上面一样，创建 p1 和 p2 两个对象，并打印这两个对象，看看和上面打印的有啥区别呢？

```
1 Person p1 = new Person("Tom",21);
2 Person p2 = new Person("Marry",20);
3 System.out.println(p1.toString());//Person{name='Tom', age=20}
4 System.out.println(p2.toString());//Person{name='Marry', age=20}
```

　　我们发现第三行代码打印的 p1 对象 age 属性变为 20了，这是为什么呢？

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180620212435546-1183265110.png)

　　这是因为用在 jvm 的内存构造中，会在堆中开辟一块内存空间，专门用来存储用 static 修饰的成员变量，称为静态存储区，无论我们创建多少个对象，用 static 修饰的成员变量有且只有一份存储在静态存储区中，所以该静态变量的值是以最后创建对象时设置该静态变量的值为准，也就是由于 p1 先设置 age = 21，后来创建了 p2 对象，p2将 age 改为了20，那么该静态存储区的 age 属性值也被修改成了20。

　　PS：在 JDK1.8 以前，静态存储区是存放在方法区的，而方法区不属于堆，在 JDK1.8 之后，才将方法区干掉了，方法区中的静态存储区改为到堆中存储。

　　**总结：static 修饰的变量被所有对象所共享，在内存中只有一个副本。由于与对象无关，所以我们可以直接通过 类名.静态变量 的方式来直接调用静态变量。对应的非静态变量是对象所拥有的，多少个对象就有多少个非静态变量，各个对象所拥有的副本不受影响。**

[回到顶部](https://www.cnblogs.com/ysocean/p/9202016.html#_labelTop)

### 2、修饰修饰成员方法

　　用 static 关键字修饰成员方法也是一样的道理，我们可以直接通过 类名.静态方法名() 的方式来调用，而不用创建对象。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public class Person {
 2     private  String name;
 3     private static Integer age;
 4 
 5     public static void printClassName(){
 6         System.out.println("com.ys.bean.Person");
 7     }
 8     public Person(String name, Integer age) {
 9         this.name = name;
10         this.age = age;
11     }
12 
13     @Override
14     public String toString() {
15         return "Person{" +
16                 "name='" + name + '\'' +
17                 ", age=" + age +
18                 '}';
19     }
20     //get和set方法省略
21 
22 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　调用静态方法：

```
1 Person.printClassName();//com.ys.bean.Person
```

[回到顶部](https://www.cnblogs.com/ysocean/p/9202016.html#_labelTop)

### 3、静态代码块

　　用 static 修饰的代码块称为静态代码块，静态代码块可以置于类的任意一个地方（和成员变量成员方法同等地位，不可放入方法中），并且一个类可以有多个静态代码块，在类初次载入内存时加载静态代码块，并且按照声明静态代码块的顺序来加载，且仅加载一次，优先于各种代码块以及构造函数。

　　关于静态代码块、构造代码块、构造函数、普通代码块的区别可以参考我的[这篇博客](https://www.cnblogs.com/ysocean/p/8194428.html)。

```
1 public class CodeBlock {
2     static{
3         System.out.println("静态代码块");
4     }
5 }
```

　　由于静态代码块只在类载入内存时加载一次的特性，我们可以利用静态代码块来优化程序性能，比如某个比较大配置文件需要在创建对象时加载，这时候为了节省内存，我们可以将该配置文件的加载时机放入到静态代码块中，那么我们无论创建多少个对象时，该配置文件也只加载了一次。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202016.html#_labelTop)

### 4、静态导包

　　用 static 来修饰成员变量，成员方法，以及静态代码块是最常用的三个功能，静态导包是 JDK1.5以后的新特性，用 import static 包名 来代替传统的 import 包名 方式。那么有什么用呢？

　　比如我们创建一个数组，然后用 JDK 自带的 Arrays 工具类的 sort 方法来对数组进行排序：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.test;
 2 
 3 import java.util.Arrays;
 4 /**
 5  * Create by YSOcean
 6  */
 7 public class StaticTest {
 8 
 9     public static void main(String[] args) {
10         int[] arrays = {3,4,2,8,1,9};
11         Arrays.sort(arrays);
12     }
13 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　我们可以看到，调用 sort 方法时，需要进行 import java.util.Arrays 的导包操作，那么变为静态导包呢？

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.test;
 2 
 3 import static java.util.Arrays.*;
 4 /**
 5  * Create by YSOcean
 6  */
 7 public class StaticTest {
 8 
 9     public static void main(String[] args) {
10         int[] arrays = {3,4,2,8,1,9};
11         sort(arrays);
12     }
13 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　我们可以看到第三行代码的 import java.util.Arrays 变为了 import static java.util.Arrays.*，意思是导入 Arrays 类中的所有静态方法，当然你也可以将 * 变为某个方法名，也就是只导入该方法，那么我们在调用该方法时，就可以不带上类名，直接通过方法名来调用（第 11 行代码）。

　　静态导包只会减少程序员的代码编写量，对于性能是没有任何提升的（也不会降低性能，Java核心技术第10版卷1第148页4.7.1章节类的导入有介绍），反而会降低代码的可读性，所以实际如何使用需要权衡。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202016.html#_labelTop)

### 5、静态内部类

　　首先我们要知道什么是内部类，定义在一个类的内部的类叫内部类，包含内部类的类叫外部类，内部类用 static 修饰便是我们所说的静态内部类。

　　定义内部类的好处是外部类可以访问内部类的所有方法和属性，包括私有方法和私有属性。

　　访问普通内部类，我们需要先创建外部类的对象，然后通过外部类名.new 创建内部类的实例。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.bean;
 2 
 3 /**
 4  * Create by hadoop
 5  */
 6 public class OutClass {
 7 
 8     public class InnerClass{
 9 
10     }
11 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 

```
1  * OuterClass oc = new OuterClass();
2  * OuterClass.InnerClass in = oc.new InnerClass();
```

　　访问静态内部类，我们不需要创建外部类的对象，可以直接通过 外部类名.内部类名 来创建实例。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.bean;
 2 
 3 /**
 4  * Create by hadoop
 5  */
 6 public class OutClass {
 7 
 8     public static class InnerClass{
 9 
10     }
11 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 

```
1 OuterClass.StaticInnerClass sic = new OuterClass.StaticInnerClass();
```

[回到顶部](https://www.cnblogs.com/ysocean/p/9202016.html#_labelTop)

### 6、常见问题

　　**①、静态变量能存在于普通方法中吗？**

　　能。很明显，普通方法必须通过对象来调用，静态变量都可以直接通过类名来调用了，更不用说通过对象来调用，所以是可以存在于普通方法中的。

　　**②、静态方法能存在普通变量吗？**

　　不能。因为静态方法可以直接通过类名来直接调用，不用创建对象，而普通变量是必须通过对象来调用的。那么将普通变量放在静态方法中，在直接通过类来调用静态方法时就会报错。所以不能。

　　**③、静态代码块能放在方法体中吗？**

　　不能。首先我们要明确静态代码块是在类加载的时候自动运行的。

　　普通方法需要我们创建对象，然后手工去调用方法，所静态代码块不能声明在普通方法中。

　　那么对于用 static 修饰的静态方法呢？同样也是不能的。因为静态方法同样也需要我们手工通过类名来调用，而不是直接在类加载的时候就运行了。

　　也就是说静态代码块能够自动执行，而不管是普通方法还是静态方法都是需要手工执行的。

　　**④、静态导包会比普通导包消耗更多的性能？**

　　不会。静态导包实际上在编译期间都会被编译器进行处理，将其转换成普通按需导包的形式，所以在程序运行期间是不影响性能的。

　　**⑤、static 可以用来修饰局部变量吗？**

　　不能。不管是在普通方法还是在静态方法中，static 关键字都不能用来修饰局部变量，这是Java的规定。稍微想想也能明白，局部变量的声明周期是随着方法的结束而结束的，因为static 修饰的变量是全局的，不与对象有关的，如果用 static 修饰局部变量容易造成理解上的冲突，所以Java规定 static 关键字不能用来修饰局部变量。



# [Java关键字(四)——final](https://www.cnblogs.com/ysocean/p/9202015.html)



**目录**

- [1、修饰变量](https://www.cnblogs.com/ysocean/p/9202015.html#_label0)
- [2、修饰方法](https://www.cnblogs.com/ysocean/p/9202015.html#_label1)
- [3、修饰类](https://www.cnblogs.com/ysocean/p/9202015.html#_label2)

 

------

　　对于Java中的 final 关键字，我们首先可以从字面意思上去理解，百度翻译显示如下：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180621080149078-282773135.png)

　　也就是说 final 英文意思表示是最后的，不可更改的。那么对应在 Java 中也是表达这样的意思，可以用 final 关键字修饰变量、方法和类。不管是用来修饰什么，其本意都是指 “它是无法更改的”，这是我们需要牢记的，为什么要无法更改？无非就是设计所需或者能提高效率，与前面介绍 static 关键字需要记住其与对象无关的理念一样，牢记 final 的不可变的设计理念后再来了解 final 关键字的用法，便会顺其自然了。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202015.html#_labelTop)

### 1、修饰变量

　　稍微有点Java基础的都知道用final关键字修饰的变量称为常量，常量的意思是不可更改。变量为基本数据类型，不可更改很容易理解，那么对于引用类型呢？不可能改的是其引用地址，还是对象的内容？

　　我们首先构造一个实体类：Person

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.bean;
 2 
 3 /**
 4  * Create by YSOcean
 5  */
 6 public class Person {
 7     private  String name;
 8 
 9     public Person(String name) {
10         this.name = name;
11     }
12 
13     public String getName() {
14         return name;
15     }
16 
17     public void setName(String name) {
18         this.name = name;
19     }
20 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　接着根据创建一个 Person 对象：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180621203318523-67929805.png)

　　可以看到，首先通过 final 关键字修饰一个对象 p，然后接着将 p 对象指向另一个新的对象，发现报错，也就是说final修饰的引用类型是不能改变其引用地址的。

　　接着我们改动 p 对象的 name 属性：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180621203519749-361874978.png)

　　发现程序没有报错。

　　结论：被 final 修饰的变量不可更改其引用地址，但是可以更改其内部属性。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202015.html#_labelTop)

### 2、修饰方法

　　final 关键字修饰的方法不可被覆盖。

　　在《Java编程思想》第 4 版 7.8.2 章节 final 方法p176 页这样描述：使用 final 方法原因有两个：

　　①、第一个原因是把方法锁定，以防止任何继承类修改它的含义，这是出于设计的考虑：想要确保在继承中使方法的行为保持不变，并且不会被覆盖。

　　②、第二个原因是效率，在 Java 的早期实现中，如果将一个方法声明为 final，就是同意编译器将针对该方法的所有调用都转为内嵌调用，内嵌调用能够提高方法调用效率，但是如果方法很大，内嵌调用不会提高性能。而在目前的Java版本中（JDK1.5以后），虚拟机可以自动进行优化了，而不需要使用 final 方法。

　　所以**final 关键字只有明确禁止覆盖方法时，才使用其修饰方法。**

　　**PS：《Java编程思想》中指出类中所有的 private 方法都隐式指定为 final 的，所以对于 private 方法，我们显式的声明 final 并没有什么效果。但是我们创建一个父类，并在父类中声明一个 private 方法，其子类中是能够重写其父类的private 方法的，这是为什么呢？**

　　父类：Parent.class

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
package com.ys.bean;
/**
 * Create by YSOcean
 */
public class Parent {
    private void say(){
        System.out.println("parent");
    }
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　子类：Son.class

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
package com.ys.bean;
/**
 * Create by YSOcean
 */
public class Son extends Parent {

    private void say(){
        System.out.println("son");
    }

}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　其实仔细看看，这种写法是方法的覆盖吗？我们通过多态的形式并不能调用到父类的 say() 方法：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180621210117804-1988915508.png)

　　并且，如果我们在子类的 say() 方法中，添加 @Override 注解也是会报错的。

　　**![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180621210206647-794954558.png)**

　　所以这种形式并不算方法的覆盖。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202015.html#_labelTop)

### 3、修饰类

　　final 修饰类表示该类不可被继承。

　　也就是说不希望某个类有子类的时候，用final 关键字来修饰。并且由于是用 final 修饰的类，其类中所有的方法也被隐式的指为 final 方法。

　　在 JDK 中有个最明显的类 String ，就是用 final 修饰的，将 String 类用 final 修饰很重要的一个原因是常量池。关于 String 类的描述，可以[参考我的这篇博客](https://www.cnblogs.com/ysocean/p/8571426.html)。

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180621210937598-1783125709.png)



# [Java关键字(六)——super](https://www.cnblogs.com/ysocean/p/9202053.html)



**目录**

- [1、调用父类的构造方法](https://www.cnblogs.com/ysocean/p/9202053.html#_label0)
- [2、调用父类的成员属性](https://www.cnblogs.com/ysocean/p/9202053.html#_label1)
- [3、调用父类的方法](https://www.cnblogs.com/ysocean/p/9202053.html#_label2)
- [ 4、this 和 super 出现在同一个构造方法中？](https://www.cnblogs.com/ysocean/p/9202053.html#_label3)

 

------

 

　　在 [Java关键字(五)——this](https://www.cnblogs.com/ysocean/p/9202051.html) 中我们说 this 关键字是表示当前对象的引用。而 Java 中的 super 关键字则是表示 **父类对象的引用。**

　　我们分析这句话“父类对象的引用”，那说明我们使用的时候只能在子类中使用，既然是对象的引用，那么我们也可以用来调用成员属性以及成员方法，当然了，这里的 super 关键字还能够调用父类的构造方法。具体有如下几种用法：

[回到顶部](https://www.cnblogs.com/ysocean/p/9202053.html#_labelTop)

### 1、调用父类的构造方法

　　Java中的继承大家都应该了解，子类继承父类，我们是能够用子类的对象调用父类的属性和方法的，我们知道属性和方法只能够通过对象调用，那么我们可以大胆假设一下：

　　在创建子类对象的同时，也创建了父类的对象，而创建对象是通过调用构造函数实现的，那么我们在创建子类对象的时候，应该会调用父类的构造方法。

　　下面我们看这段代码：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public class Parent {
 2 
 3     public Parent(){
 4         System.out.println("父类默认无参构造方法");
 5     }
 6 }
 7 
 8 
 9 public class Son extends Parent {
10 
11     public Son(){
12         System.out.println("子类默认无参构造方法");
13     }
14 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　下面我们创建子类的对象：

```
1     public static void main(String[] args) {
2         Son son = new Son();
3     }
```

　　打印结果：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180623094013357-2050806155.png)

　　通过打印结果看到我们在创建子类对象的时候，首先调用了父类的构造方法，接着调用子类的构造方法，也就是说在创建子类对象的时候，首先创建了父类对象，与前面我们猜想的一致。

　　那么问题又来了：是在什么时候调用的父类构造方法呢？

　　可以参考Java官方文档：https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#d5e14278

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180623095640160-2086100404.png)

　　

　　红色框内的英文翻译为：**如果声明的类是原始类`Object`，那么默认的构造函数有一个空的主体。否则，默认构造函数只是简单地调用没有参数的超类构造函数。**

　　也就是说除了顶级类 Object.class 构造函数没有调用父类的构造方法，其余的所有类都默认在构造函数中调用了父类的构造函数（没有显式声明父类的子类其父类是 Object）。

　　那么是通过什么来调用的呢？我们接着看官方文档：

![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180623100141314-893008607.png)

　　上面的意思大概就是超类构造函数通过 super 关键字调用，并且是以 super 关键字开头。

　　所以上面的 Son 类的构造方法实际上应该是这样的：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180623100522313-1024015047.png)

　　①、子类默认是通过 super() 调用父类的无参构造方法，如果父类显示声明了一个有参构造方法，而没有声明无参构造方法，实例化子类是会报错的。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public class Parent {
 2 
 3     public Parent(String name){
 4         System.out.println("父类有参构造方法");
 5     }
 6 }
 7 
 8 public class Son extends Parent {
 9 
10     public Son(){
11         System.out.println("子类默认无参构造方法");
12     }
13 
14     public static void main(String[] args) {
15         Son son = new Son();
16     }
17 
18 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　上面代码是会报错的：

　　![img](https://images2018.cnblogs.com/blog/1120165/201806/1120165-20180623100916026-1455644673.png)

　　解决办法就是通过 super 关键字调用父类的有参构造方法：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public class Son extends Parent {
 2 
 3     public Son(){
 4         super("Tom");
 5         System.out.println("子类默认无参构造方法");
 6     }
 7 
 8     public static void main(String[] args) {
 9         Son son = new Son();
10     }
11 
12 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　**注意看第 4 行代码，同理，多个参数也是这种调法。**

[回到顶部](https://www.cnblogs.com/ysocean/p/9202053.html#_labelTop)

### **2、调用父类的成员属性**

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public class Parent {
 2     public String name;
 3 
 4     public Parent(){
 5         System.out.println("父类默认无参构造方法");
 6     }
 7 }
 8 
 9 public class Son extends Parent {
10 
11     public Son(){
12         System.out.println("子类默认无参构造方法");
13     }
14 
15     public void printName(){
16         System.out.println(super.name);
17     }
18 
19 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　第 16 行代码 super.父类属性 通过这种形式来调用父类的属性。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202053.html#_labelTop)

### 3、调用父类的方法

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public class Parent {
 2     public String name;
 3 
 4     public Parent(){
 5         System.out.println("父类默认无参构造方法");
 6     }
 7 
 8     public void setName(String name){
 9         this.name = name;
10     }
11 }
12 
13 public class Son extends Parent {
14 
15     public Son(){
16         super();//1、调用父类构造函数
17         System.out.println("子类默认无参构造方法");
18     }
19 
20     public void printName(){
21         super.setName("Tom");//2、调用父类方法
22         System.out.println(super.name);//3、调用父类属性
23     }
24 
25     public static void main(String[] args) {
26         Son son = new Son();
27         son.printName();//Tom
28     }
29 
30 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这个例子我们在子类中分别调用了父类的构造方法、普通方法以及成员属性。

[回到顶部](https://www.cnblogs.com/ysocean/p/9202053.html#_labelTop)

###  4、this 和 super 出现在同一个构造方法中？

　　不能！！！

　　在上一篇博客对 [this 关键字](https://www.cnblogs.com/ysocean/p/9202051.html) 的介绍中，我们知道能够通过 this 关键字调用自己的构造方法。而本篇博客介绍 super 关键字，我们知道了能够通过 super 调用父类的构造方法，那么这两个关键字能同时出现在子类的构造方法中吗？

　　**①、假设 super() 在 this() 关键字的前面**

　　首先通过 super() 调用父类构造方法，对父类进行一次实例化。接着调用 this() ，this() 方法会调用子类的构造方法，在子类的构造方法中又会对父类进行一次实例化。也就是说我们对子类进行一次实例化，对造成对父类进行两次实例化，所以显然编译器是不允许的。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public class Parent {
 2     public String name;
 3 
 4     public Parent(){
 5         System.out.println("父类默认无参构造方法");
 6     }
 7 
 8     public Parent(String name){
 9         System.out.println("父类有参构造方法");
10     }
11 
12 }
13 
14 public class Son extends Parent {
15 
16     public Son(){
17         super();//1、调用父类构造函数
18         this("Tom");//2、调用子类构造方法
19         System.out.println("子类默认无参构造方法");
20     }
21 
22     public Son(String name){
23         System.out.println("子类有参构造方法");
24     }
25 
26 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　反过来 this() 在 super() 之前也是一样。

　　而且编译器有限定 this() 和 super() 这两个关键字都只能出现在构造方法的第一行，将这两个关键字放在一起，总有一个关键字在第二行，编译是不能通过的。　　　