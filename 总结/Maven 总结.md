# [Maven概述](https://www.cnblogs.com/ysocean/p/7402385.html)



**目录**

- [1、引言　　](https://www.cnblogs.com/ysocean/p/7402385.html#_label0)
- [2、常规项目开发存在的问题](https://www.cnblogs.com/ysocean/p/7402385.html#_label1)
- [3、什么是 Maven ?](https://www.cnblogs.com/ysocean/p/7402385.html#_label2)
- [4、Maven 的历史](https://www.cnblogs.com/ysocean/p/7402385.html#_label3)
- [5、Maven 的目标](https://www.cnblogs.com/ysocean/p/7402385.html#_label4)
- [6、Maven 的理念](https://www.cnblogs.com/ysocean/p/7402385.html#_label5)

 

------

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170821081110464-700792526.png)


### 1、引言　　

　　你能搜到这个教程，说明你对 Maven 感兴趣，但是又不是太理解。那么接下来这个系列的教程将会详细讲解 Maven 的用法，相信你看完之后，一定能对 Maven 的理解更进一步！


### 2、常规项目开发存在的问题

　　通常Web项目开发只会创建一个工程，然后所有的jar包都会存放到 WEB-INF/lib 目录下，如下图所示：

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170821195035089-2022006160.png)

　　通过上面的目录结构我们可以分析出如下问题：

　　①、一个项目就是一个web工程。如果项目比较庞大，那么利用包名package来划分模块，显然容易造成混淆而且不利于分工合作；

　　②、项目中需要的 jar 包必须手动 复制，粘贴 到 WEB-INF/lib 目录下。这会导致每创建一个新的工程就需要将 jar 包重复复制到 lib 目录下，从而造成工作区存在大量重复的文件；

　　③、jar需要我们手动去官网上或者其他途径下载；

　　④、一个 jar 包依赖的其他 jar 包，需要自己手动加入到项目中，而且很有可能我们漏掉了某个依赖关系，导致项目运行报错。

 　那么如何解决这些问题呢？本系列的主角 Maven 应运而生了。



### 3、什么是 Maven ?

　　Maven 读音是 [ˈmevən]，也就是“霉文”，而不是读“马文”。它是一个项目管理和综合工具，Maven使用标准的目录结构和默认构建生命周期。提供了开发人员构建一个完整的生命周期框架，开发团队可以自动完成该项目的基础设施建设。相信如果对 Maven 没有任何了解的，看了这段话等于没看，不过没关系，后面我们将会逐渐揭开 Maven 的神秘面纱。什么是 Maven，你只需要知道这玩意能简化和标准化项目建设过程。 


### 4、Maven 的历史

　　Maven的最初设计，以简化Jakarta Turbine项目的建设进程。有几个项目，每个项目包含了稍微不同的Ant构建文件。 JAR中检查到CVS。Apache组织开发的Maven可以建立多个项目，发布项目信息，项目部署。


### 5、Maven 的目标

　　Maven主要目标是提供开发人员

　　①、项目是可重复使用，易维护，更容易理解的一个综合模型。

　　②、插件或交互的工具，这种声明性的模式。

　　Maven项目的结构和内容是在一个XML文件中声明，pom.xml的项目对象模型（POM），这是整个Maven系统的基本单元。　　　

 

### 6、Maven 的理念

　　**约定优于配置！！！**

　　开发人员不需要创建构建过程本身，不必知道提到的每一个配置的详细信息。Maven提供了合理的默认行为的项目。创建一个Maven项目时，Maven创建默认的项目结构。开发者只需要把相应的文件和她需要在pom.xml中定义即可。





# [Maven的安装配置](https://www.cnblogs.com/ysocean/p/6725729.html)



**目录**

- [1、下载 Maven](https://www.cnblogs.com/ysocean/p/6725729.html#_label0)
- [2、配置 Maven 环境变量](https://www.cnblogs.com/ysocean/p/6725729.html#_label1)
- [3、查看 Maven 环境变量是否配置成功](https://www.cnblogs.com/ysocean/p/6725729.html#_label2)
- [4、在 eclipse 中集成 Maven 插件](https://www.cnblogs.com/ysocean/p/6725729.html#_label3)

 



### **1、下载 Maven**

　　①、官网下载地址：http://maven.apache.org/download.cgi

 



### **2、配置 Maven 环境变量**

　　将下载的 maven 压缩包解压到电脑的某个盘符，我这里是 D:\JavaTools\apache-maven-3.3.9

　　**①、右键---计算机属性----高级系统设置---高级---环境变量---系统变量----新建**   

　　　　变量名：MAVEN_HOME

　　　　变量值：D:\JavaTools\apache-maven-3.3.9　　　　　　PS：这个值是 maven 压缩包解压的位置

![img](https://images2015.cnblogs.com/blog/1120165/201704/1120165-20170417234626634-779498189.png)

 

 　**②、将 MAVEN_HOME 添加到 path 目录下**

　　选择 path---编辑---新建---将 %MAVEN_HOME%\bin  添加进去，然后点击确定就可以了

![img](https://images2015.cnblogs.com/blog/1120165/201704/1120165-20170417235008212-617372989.png)

 


### **3、查看 Maven 环境变量是否配置成功**

　　打开命令提示符，快捷键 windows+R.输入 mvn -v 

　　如果出现如下maven 的版本信息，那么就配置成功

![img](https://images2015.cnblogs.com/blog/1120165/201704/1120165-20170417235404337-1330383501.png)

 


### 4、在 eclipse 中集成 Maven 插件

　　**①、在 eclipse 指定 Maven 插件的位置**

```
Window->Preferences->Maven->Installations
```

　　如下图：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823203945746-1231183597.png)

 

 　**②、指定 conf/settings.xml 的位置，进而指定 Maven 本地仓库的位置**

```
Window->Preferences->Maven->User Settings
```

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823204449074-1012332898.png)

　　修改 settings.xml 如下标签的位置即可：<localRepository>自定义路径</localRepository>

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823204549777-2138278150.png)

# [Maven工程目录介绍](https://www.cnblogs.com/ysocean/p/7420373.html)



**目录**

- [1、eclipse 创建 Maven 工程](https://www.cnblogs.com/ysocean/p/7420373.html#_label0)
- [2、Maven Java工程的目录结构 ](https://www.cnblogs.com/ysocean/p/7420373.html#_label1)

 

------

　　上一章我们配置并安装好了 Maven，那么这一章我们介绍如何用eclipse创建一个 Maven 工程，然后介绍 Maven 工程的目录结构。


### 1、eclipse 创建 Maven 工程

　　**第一步：File-->New--->Maven Project**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823211800621-1801994729.png)

　　**第二步：勾上 Create a simple project ,然后点击 next**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823211948746-2012640598.png)

　　**第三步：填写 Group Id 和 Artifact Id**

　　groupid和artifactId被统称为“坐标”是为了保证项目唯一性而提出的，如果你要把你项目弄到maven本地仓库去，你想要找到你的项目就必须根据这两个id去查找。

　　groupId一般分为多个段，这里只说两段，第一段为域，第二段为公司名称。域又分为org、com、cn等等许多，其中org为非营利组织，com为商业组织。举个apache公司的tomcat项目例子：这个项目的groupId是org.apache，它的域是org（因为tomcat是非营利项目），公司名称是apache，artigactId是tomcat。

　　ArtifactID就是项目的唯一的标识符，实际对应项目的名称，就是项目根目录的名称。比如我创建一个项目，我一般会将groupId设置为com.ys，com表示域，ys是我个人姓名缩写，**Artifact Id**设置为hellomaven，表示你这个项目的名称是hellomaven，依照这个设置，你的包结构最好是com.ys.hellomaven打头的，如果有个StudentDao，它的全路径就是com.ys.hellomaven.dao.StudentDao

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823213100605-885143011.png)

 



### 2、Maven Java工程的目录结构 

 　**①、我们根据上面的步骤，创建出如下的 maven 工程：**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823213600558-595949612.png)

 

 　对每个目录结构的解析如下：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823213846668-1108166981.png)

 

　　**②、为什么 maven 工程的目录结构要这样呢？**

　　1、Maven 要负责项目的自动化构建，以编译为例，Maven 要想自动进行编译，那么它必须知道 Java 的源文件保存在哪里，这样约定之后，不用我们手动指定位置，Maven 能知道位置，从而帮我们完成自动编译。

　　2、遵循 约定>>>配置>>>编码。即能进行配置的不要去编码指定，能事先约定规则的不要去进行配置。这样既减轻了劳动力，也能防止出错。

 

　　**③、pom.xml 文件**

　　Project Object Model 项目对象模型，Maven 的核心配置文件，pom.xml，与构建过程相关的一切设置都在这个文件中进行配置。

　　这个文件非常重要，我们后面会详细讲解。



# [常用的Maven命令](https://www.cnblogs.com/ysocean/p/7416307.html)



**目录**

- [1、创建 Maven 工程](https://www.cnblogs.com/ysocean/p/7416307.html#_label0)
- [2、Maven 的常用命令](https://www.cnblogs.com/ysocean/p/7416307.html#_label1)

 

------

　　这章我们讲讲几个常用的 Maven 命令。由于执行命令是在工程的基础上来的，所以我们要先创建一个 Maven 工程，具体如何创建，在上一篇博客已经介绍了：[http://www.cnblogs.com/ysocean/p/7420373.html
](http://www.cnblogs.com/ysocean/p/7420373.html)

 

### 1、创建 Maven 工程

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823224526183-2011305908.png)

　　①、在 src/main/java 新建包 com.ys.maven,然后在这个包中创建类 HelloMaven.java

```
package com.ys.maven;
 
public class HelloMaven {
     
    //传入一个字符串并返回
    public String Hello(String name){
         
        return name;
    }
}
```

　　②、在 src/test/java 新建包 com.ys.maven,然后在这个包中创建类 HelloTest.java

```
package com.ys.maven;
 
import junit.framework.Assert;
import org.junit.Test;
 
public class HelloTest {
     
    @Test
    public void testHello(){
        HelloMaven he = new HelloMaven();
        String name = he.Hello("Tom");
        //判断 Hello 传入的参数是否是 "maven"
        Assert.assertEquals("maven", name);
    }
 
}
```

　　③、pom.xml 文件如下：

```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
     http://maven.apache.org/xsd/maven-4.0.0.xsd">
      
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.ys</groupId>
  <artifactId>hellomaven</artifactId>
  <version>0.0.1-SNAPSHOT</version>
   
  <dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.0</version>
        <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

　　为什么要这样写，我们后面会详细讲解。

　　

 

### 2、Maven 的常用命令

```
1、mvn compile 编译,将Java 源程序编译成 class 字节码文件。
2、mvn test 测试，并生成测试报告
3、mvn clean 将以前编译得到的旧的 class 字节码文件删除
4、mvn pakage 打包,动态 web工程打 war包，Java工程打 jar 包。
5、mvn install 将项目生成 jar 包放在仓库中，以便别的模块调用
```

　

　　**①、compile:将Java 源程序编译成 class 字节码文件。**

　　第一步：选择 pom.xml 文件，右键--->Run As ---->2 Maven build...

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823232128011-2096692588.png)

　　第二步：在第一步执行完后弹出来的对话框中，输入 compile，然后点击 Run 按钮

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823232252074-1370835227.png)

　　第三步：查看控制台

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823232606589-2033638078.png)

　　第四步：在 target 目录下，我们会发现编译生成的 class 文件

 

 　**②、test:测试，并生成测试报告**

　　　　第一步：选择 pom.xml 文件，右键--->Run As ---->2 Maven build...,然后在弹出框中输入 test

　　　　　　或者选择 pom.xml 文件，右键--->Run As------>6 Maven test，如下图

 

　　第二步：查看控制台

　　分析测试程序，我们传入的参数是Tom，而我们希望的是maven，很显然是不相等的，那么测试失败

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823233951089-655472389.png)

 

 　如果测试类 HelloTest.java改为如下：

 

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823234053230-224355756.png)

　　

　　重新执行 mvn test 命令，控制台如下：

 

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823234203824-1000644310.png)

　　生成的测试报告可以在如下目录查看：target/surefire-reports

 

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823234355386-292928271.png)

　　

　　**③、mvn clean 将以前编译得到的旧的 class 字节码文件删除**

　　第一步：选择 pom.xml 文件，右键--->Run As ---->2 Maven build...,然后在弹出框中输入 clean

　　　　或者选择 pom.xml 文件，右键--->Run As------>3 Maven clean，如下图

 　

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823234620808-795702161.png)

 

　　第二步：查看控制台

　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823234732621-269563223.png)

 

　　第三步：发现 mvn compile 编译好的文件这时已经清除了

 

　　

　　**④、mvn pakage 打包,动态 web工程打 war包，Java工程打 jar 包。**

　　第一步：选择 pom.xml 文件，右键--->Run As ---->2 Maven build...,然后在弹出框中输入 package

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823235055386-1195687188.png)

 

 　第二步：查看控制台

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823235443668-1045163740.png)

　　第三步：进入到 target 目录，会发现打出来的 jar 包

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170823235534589-589996042.png)

 

 　**⑤、mvn install 将项目生成 jar 包放在仓库中，以便别的模块调用**

 　这里我们就不截图了，执行命令后，进入到 settings.xml 文件中配置的仓库，你会发现生成的 jar 包

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170824074621246-139025286.png)

# [坐标及依赖管理](https://www.cnblogs.com/ysocean/p/7451054.html)



**目录**

- [1、什么是坐标？](https://www.cnblogs.com/ysocean/p/7451054.html#_label0)
- [2、什么是依赖？ ](https://www.cnblogs.com/ysocean/p/7451054.html#_label1)
- [3、依赖的详细配置](https://www.cnblogs.com/ysocean/p/7451054.html#_label2)
- [4、依赖的范围 scope ](https://www.cnblogs.com/ysocean/p/7451054.html#_label3)
- [4、依赖的传递](https://www.cnblogs.com/ysocean/p/7451054.html#_label4)
- [5、依赖的排除](https://www.cnblogs.com/ysocean/p/7451054.html#_label5)
- [6、依赖的冲突](https://www.cnblogs.com/ysocean/p/7451054.html#_label6)
- [7、可选依赖](https://www.cnblogs.com/ysocean/p/7451054.html#_label7)

 

------

　　我们知道maven能帮我们管理jar包，那么它是怎么管理的呢？这篇博客我们来详细介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/7451054.html#_labelTop)

### 1、什么是坐标？

　　**①、数学中的坐标**

　　　　在平面上，使用 X 、Y 两个向量可以唯一的定位平面中的任何一个点

　　　　在空间中，使用 X、Y、Z 三个向量可以唯一的定位空间中的任意一个点

 

　　**②、Maven 中的坐标**

　　　　俗称 gav：使用下面三个向量子仓库中唯一定位一个 Maven 工程

　　　　在项目中的 pom.xml 文件中，我们可以看到下面gav的定义：

　　　　1、groupid:公司或组织域名倒序 

　　　　　　<groupid>com.ys.maven</groupid>

　　　　2、artifactid:模块名，也是实际项目的名称

　　　　　　<artifactid>Maven_05</artifactid>

　　　　3、version:当前项目的版本

　　　　　　<version>0.0.1-SNAPSHOT</version>

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830211037062-128803329.png)

　　**③、Maven 坐标和仓库，jar 包的关系**

　　　　什么是仓库，后面我们会详细讲解，现在你只需要知道是Maven 用来存放 jar 包的地方。

　　　　那么依照上面定义的 gav，我们执行 mvn -install 命令，会出现什么情况呢？

　　　　首先进入到我们第二篇博客 [Maven 的安装配置](http://www.cnblogs.com/ysocean/p/6725729.html) ，通过 settings.xml 文件配置的仓库目录。

　　　　　　将我们上面配置的 gav 向量组合起来就是目录：

　　　　　　com/ys/maven/Maven_05/0.0.1-SNAPSHOT/Maven_05-0.0.1-SNAPSHOT.jar

 　　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830212535468-1450740064.png)

　　　　其次，我们观察打出来的 jar 包：

　　　　　　Maven_05-0.0.1-SNAPSHOT.jar

　　　　也就是 artifactid-version.jar

 


### 2、什么是依赖？ 

 　什么是 依赖？相信有过一定开发经验的人知道，每当我们需要使用某个框架时，比如 SpringMVC，那么我们需要导入相应的 jar 包，但是手动导入包的时候，往往会漏掉几个 jar 包，那么在使用该框架的时候系统就会报错。那么我们就说导入的包与未导入的包存在依赖关系。而使用 Maven,我们只需要在 pom.xml 文件中进行相应的配置，它就会帮助我们自动管理 jar 包之间的依赖关系。那么它是怎么管理的呢，接下来我们会详细讲解。

 


### 3、依赖的详细配置

　　我们以 Junit 为例，在 pom.xml 文件中进行详细而完整的配置。　

```
<project>     
    <dependencies>
        <dependency>
            <groupId>junit</groupId>     
            <artifactId>junit</artifactId>     
            <version>3.8.1</version>
            <type>...</type>
            <scope>...</scope>
            <optional>...</optional>
            <exclusions>     
                <exclusion>     
                  <groupId>...</groupId>     
                  <artifactId>...</artifactId>     
                </exclusion>
          </exclusions>     
        </dependency>        
      </dependencies>     
</project>
```

　　①、dependencies：一个 pom.xml 文件中只能存在一个这样的标签。用来管理依赖的总标签。

　　②、dependency:包含在dependencies标签中，可以有无数个，每一个表示一个依赖

　　③、groupId,artifactId和version：依赖的基本坐标，对于任何一个依赖来说，基本坐标是最重要的，Maven根据坐标才能找到需要的依赖。

　　④、type：依赖的类型，对应于项目坐标定义的packaging。大部分情况下，该元素不必声明，其默认值是jar。

　　⑤、scope：依赖的范围，默认值是 compile。后面会进行详解。

　　⑥、optional：标记依赖是否可选。

　　⑦、exclusions：用来排除传递性依赖，后面会进行详细介绍。

　　

 

### 4、依赖的范围 scope 

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830221620890-18460703.png)

　　一般情况下，我们对前面三个依赖用的比较多。下面的主程序表示maven 目录结构 src/main/java.测试程序目录结构为：src/test/java

 　**1、compile 范围依赖**

　　　　对主程序是否有效：有效

　　　　对测试程序是否有效：有效

　　　　是否参与打包：参与

　　　　是否参与部署：参与

　　　　典型例子：log4j

　　　　

　　**2、test 范围依赖**

　　　　对主程序是否有效：无效

　　　　对测试程序是否有效：有效

　　　　是否参与打包：不参与

　　　　是否参与部署：不参与

　　　　典型例子：Junit

　　

　　**3、provided 范围依赖**

　　　　对主程序是否有效：有效

　　　　对测试程序是否有效：有效

　　　　是否参与打包：不参与

　　　　是否参与部署：不参与

　　　　典型例子：servlet-api.jar，一般在发布到 服务器中，比如 tomcat，服务器会自带 servlet-api.jar 包，所以provided 范围依赖只在编译测试有效。

　　　 

 　**4、runtime 范围依赖**：在测试、运行的时候依赖，在编译的时候不依赖。例如：JDBC驱动，项目代码只需要jdk提供的jdbc接口，只有在执行测试和运行项目的时候才需要实现jdbc的功能。

 　

 　接下来我们举几个例子在工程中实际去理解：

　　　　**test 依赖和 compile 依赖的区别：**

　　　　①、首先我们在 pom.xml 文件中配置，Junit 的 test 依赖

 　　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830222649546-517820136.png)

　　　　②、我们在主程序中去导入 Junit 的包，然后进行 mvn -compile 编译，很明显，test 范围的在主程序中无效，故编译会报错。

　　　　　　我们在 src/main/java 包下新建 MavenTest.java,并导入 Junit 包

　　　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830222955749-692178369.png)

　　　　　　然后执行 mvn -compile 操作，如下图报错信息：

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830223058624-1830235216.png)

 

　　　　③、我们将 Junit 的依赖范围改为 compile，然后执行 mvn -compile

　　　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830223230562-63445240.png)

　　　　　　发现 mvn -compile 没有报错了。

　　　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830223353874-95793758.png)

 

 

### 4、依赖的传递

　　比如我们创建三个Maven 工程，maven-first,maven-second以及maven-third,而third依赖于second，second又依赖于first，那么我们说 second 是 third 的第一直接依赖，first是second的第二直接依赖。而third是first的间接依赖。

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830224603343-1515782308.png)

 

 　依赖之间的传递如下图：**第一列表示第一直接依赖，第一行表示第二直接依赖**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830232214140-802764608.png)

　　**总结：**

```
1、当第二依赖的范围是compile的时候，传递性依赖的范围与第一直接依赖的范围一致。
2、当第二直接依赖的范围是test的时候，依赖不会得以传递。
3、当第二依赖的范围是provided的时候，只传递第一直接依赖范围也为provided的依赖，且传递性依赖的范围同样为 provided；
4、当第二直接依赖的范围是runtime的时候，传递性依赖的范围与第一直接依赖的范围一致，但compile例外，此时传递的依赖范围为runtime；
```

　　我们这里举个例子来看：

　　①、第二依赖范围是 test

　　　　Maven_first 的pom.xml 文件如下：

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830233009280-1582726667.png)

　　　　然后Maven_second依赖Maven_fisrt,Maven_third依赖Maven-second,其pom.xml 文件如下：

　　　　Maven_second的 pom.xml

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830233151749-1645068897.png)

　　　　

　　　　Maven_third的 pom.xml

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830233238733-165096030.png)

　　　　我们发现在 Maven_third和 Maven_second 都没有 Maven_first 引入的 Junit 包，正好符合上面总结的第二点：当第二直接依赖的范围是test的时候，依赖不会得以传递。

 　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830233708171-340184405.png)

 　　　②、第二依赖范围是 compile

 　　　如果我们将 Maven_first 的Junit 改为 compile，那么将会符合上面总结的第一点：当第二依赖的范围是compile的时候，传递性依赖的范围与第一直接依赖的范围一致。

 　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830233829358-52202347.png)

 

 

### 5、依赖的排除

　　如果我们在当前工程中引入了一个依赖是 A，而 A 又依赖了 B，那么 Maven 会自动将 A 依赖的 B 引入当前工程，但是个别情况下 B 有可能是一个不稳定版，或对当前工程有不良影响。这时我们可以在引入 A 的时候将 B 排除。

　　比如：我们在Maven_first 中添加 spring-core,maven 会自动将 commons-logging添加进来，那么由于 Maven_second 是依赖 Maven_first  的，那么 Maven_second  中将存在 spring-core(自带了commons-logging)，这时候我们不想要 commons-logging，那该怎么办呢？我们上面第二大点提到了：

```
exclusions：用来排除传递性依赖
```

　　Maven_first 的 pom.xml 文件

![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830234858296-826245551.png)

　　由于 Maven_second 依赖 Maven_second，故Maven_second 存在 spring-core 包

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830235311890-363021832.png)

 

 　如何排除呢？我们在 Maven_second 的 pom.xml 文件中添加如下代码：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830235124733-1739762397.png)

　　再次查看工程：Maven_second 的 commons-logging 已经移除了

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170830235223780-459002277.png)

 


### 6、依赖的冲突

　　在maven中存在两种冲突方式：一种是跨pom文件的冲突，一致是同一个pom文件中的冲突。

　　**①、跨 pom 文件，路径最短者优先。**比如我们在 Maven_first 中的 Junit 是4.9版本的，Maven_second 中的 Junit 是4.8版本的，那么Maven_third 中的 Junit 将会是那个版本呢？

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831000230046-2116609529.png)

 

　　由上图我们可以看出，由于 Maven_second 是 Maven_third 的直接依赖，明显相比于 Maven_first 路径要短，所以 Maven_third 的 Junit 版本与 Maven_second 保持一致。

 

　　**②、同一个pom.xml 文件，先申明者优先。**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831000829515-1993242804.png)

　　

　　**看 Maven_second**

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831000856249-163663194.png)

 


### 7、可选依赖

　　Optional标签标示该依赖是否可选，默认是false。可以理解为，如果为true，则表示该依赖不会传递下去，如果为false，则会传递下去。

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831001250499-1078223970.png)

　　我们是在 Maven_second 的 pom 文件中设定 Junit 不可传递，那么Maven_third 工程中将不会有来自 Maven_second 的 Junit 的传递。



# [生命周期](https://www.cnblogs.com/ysocean/p/7456179.html)



**目录**

- [1、什么是 生命周期？](https://www.cnblogs.com/ysocean/p/7456179.html#_label0)
- [2、Clean Lifecycle:在进行真正的构建之前进行一些清理工作](https://www.cnblogs.com/ysocean/p/7456179.html#_label1)
- [3、Default Lifecycle：构建的核心部分，编译、测试、打包、安装、部署等等](https://www.cnblogs.com/ysocean/p/7456179.html#_label2)
- [ 4、Site Lifecycle：生成项目报告，站点，发布站点。](https://www.cnblogs.com/ysocean/p/7456179.html#_label3)

 

------


### 1、什么是 生命周期

　　Maven 强大的原因是有一个十分完善的生命周期，生命周期可以理解为项目构建步骤的集合，它定义了各个构建环节的执行顺序，有了这个顺序，Maven 就可以自动化的执行构建命令。

　　Maven 的核心程序中定义了抽象的生命周期，生命周期中各个阶段的具体任务是由插件来完成的。有三套相互独立的生命周期，各个构建环节执行顺序不能打乱，必须按照既定的正确顺序来执行。

　　　　**①、Clean Lifecycle:在进行真正的构建之前进行一些清理工作**

　　　　**②、Default Lifecycle：构建的核心部分，编译、测试、打包、安装、部署等等。**

　　　　**③、Site Lifecycle：生成项目报告，站点，发布站点。**

　　这三个都是相互独立的。你可以仅仅调用 clean 来清理工作目录，仅仅调用 site 来生成站点。当然，也可以直接运行 mvn claen install site 运行所有这三套生命周期。下面我们分别来谈谈这三个生命周期。

 


### 2、**Clean Lifecycle**:在进行真正的构建之前进行一些清理工作

```
pre-clean 执行一些需要在clean之前完成的工作 
clean 移除所有上一次构建生成的文件 
post-clean 执行一些需要在clean之后立刻完成的工作 
```

　　我们前面讲的执行命令 mvn -clean,也就等同于 Clean 生命周期中的第一个阶段 mvn pre-clean clean。注意有 Clean 声明周期，而这个声明周期中又有 clean 阶段。

　　**只要执行后面的命令，那么前面的命令都会执行，不需要再重新去输入命令。**




### 3、Default Lifecycle：构建的核心部分，编译、测试、打包、安装、部署等等

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
validate 
generate-sources 
process-sources 
generate-resources 
process-resources 复制并处理资源文件，至目标目录，准备打包。 
compile 编译项目的源代码。 
process-classes 
generate-test-sources 
process-test-sources 
generate-test-resources 
process-test-resources 复制并处理资源文件，至目标测试目录。 
test-compile 编译测试源代码。 
process-test-classes 
test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。 
prepare-package 
package 接受编译好的代码，打包成可发布的格式，如 JAR 。 
pre-integration-test 
integration-test 
post-integration-test 
verify 
install 将包安装至本地仓库，以让其它项目依赖。 
deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享。
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这里我们强调一下：**在maven中，只要在同一个生命周期，你执行后面的阶段，那么前面的阶段也会被执行，而且不需要额外去输入前面的阶段。**

　　我们举个例子：执行 mven compile 命令，根据上面的声明周期，它会默认执行前面五个个步骤也就是　　　

```
validate 
generate-sources 
process-sources 
generate-resources 
process-resources 复制并处理资源文件，至目标目录，准备打包。 
compile 编译项目的源代码。
```

 

　　我们在 eclipse 中执行 mvn compile 命令

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831083125187-1048796512.png)

　　看到红色框的两部分，第一个 maven-compiler-plugin:2.6:resource 就是用来执行前面几个步骤的插件，第二个插件 maven-compiler-plugin:3.1:compile 则是用来执行 mvn compile 的插件。这里我们提一下，mvn 的各个生命周期步骤都是依赖插件来完成的，后面我们会详细讲解 maven 插件。




###  4、**Site Lifecycle：生成项目报告，站点，发布站点。**

```
pre-site 执行一些需要在生成站点文档之前完成的工作 
site 生成项目的站点文档 
post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 
site-deploy 将生成的站点文档部署到特定的服务器上
```

　　这里经常用到的是 site 阶段和 site-deploy 阶段，用来生成和发布 maven 站点，这是 Maven 比较强大的功能，文档及统计数据自动生成。由于现在的系统会有专门工具来生成文档或报表。所以这个功能也是比较鸡肋吧，不够简洁和美观，用的不太多。



# [创建Web工程以及插件原理](https://www.cnblogs.com/ysocean/p/7460617.html)



**目录**

- [1、什么是 Maven 插件？](https://www.cnblogs.com/ysocean/p/7460617.html#_label0)
- [2、配置编译插件](https://www.cnblogs.com/ysocean/p/7460617.html#_label1)
- [3、创建 Maven Web 工程](https://www.cnblogs.com/ysocean/p/7460617.html#_label2)
- [ 4、添加 tomcat 插件](https://www.cnblogs.com/ysocean/p/7460617.html#_label3)

 

------


### 1、什么是 Maven 插件

　　上一篇博客我们讲了 Maven 的生命周期，我们知道 Maven 的核心是生命周期，生命周期指定了 Maven 命令执行的流程顺序。但是真正实现流程的工程是由插件来完成的。

　　我们也可以说 Maven 是一个执行插件的框架，每一个任务实际上都是有插件来完成。进一步说每个任务对应了一个插件目标（goal），每个插件会有一个或者多个目标，例如maven-compiler-plugin的compile目标用来编译位于src/main/java/目录下的主源码，testCompile目标用来编译位于src/test/java/目录下的测试源码。

 


### 2、配置编译插件

　　一般我们创建一个 Maven 工程，就算指定了 JDK 的版本，但是你执行 update project 操作，一般 Maven 工程会自动恢复到默认的 JDK 版本，有可能是1.4，有可能是1.5（和 Maven 版本有关）。

　　那么我们如何指定其 JDK 版本呢？在 pom.xml 中添加如下代码：

```
<build>
    <plugins>
        <!-- 编译插件，指定 JDK 的版本为1.7 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build>
```

　　下面我们来添加一个 tomcat 插件，首先我们要知道如何创建 Maven Web 工程。

 


### 3、创建 Maven Web 工程

　　第一步：New maven project，注意打包方式为 war

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831210810671-1692250452.png)

　　

　　第二步：右击项目名，选择 properties，选择Project Facets

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831210855780-316942564.png)

　　

　　第三步：将 Dynamic Web Module 取消，点击 Apply

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831210928390-1845804996.png)

　　

　　第四部：将 Dynamic Web Module 重新勾选，点击 下方生成的超链接

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831210955952-1536472047.png)

　　

　　第五步：点击超链接，修改目录结构，然后点击 OK，创建 Maven Web 工程完成

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831211017780-547663511.png)

 

 　创建的 Web 工程目录结构如下：

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831211137687-2121693110.png)

 




###  4、添加 tomcat 插件

 　我们在上面创建的 web 工程，可以输入  tomcat:run 来使用默认的 tomcat 插件去启动 web 工程，但是默认的插件版本有点低，我们可以手动添加插件。

```
<build>
    <plugins>
        <!--配置tomcat 插件  -->
    <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <configuration>
            <port>8080</port><!--端口号  -->
            <path>/</path>
        </configuration>
    </plugin>
</plugins>
```

　　执行命令是输入：tomcat7:run

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831212223452-568420852.png)

# [继承和聚合](https://www.cnblogs.com/ysocean/p/7460616.html)



**目录**

- [1、继承](https://www.cnblogs.com/ysocean/p/7460616.html#_label0)

- [ 2、聚合](https://www.cnblogs.com/ysocean/p/7460616.html#_label1)

  

------


### 1、继承

　　**需求场景：**

　　有三个 Maven 工程，每个工程都依赖某个 jar 包，比如 Junit，由于 test 范围的依赖不能传递，它必然会分散在每个工程中，而且每个工程的jar 包版本可能不一致。那么如何管理各个工程中对于某个 jar 包的版本呢？

　　**解决办法：**

　　将那个 jar 包版本统一提取到 “父" 工程中，在子工程中声明依赖时不指定版本，以父工程中统一设定的为准，同时也便于修改。

　　**操作步骤：**

　　①、创建父工程

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831220213312-30032420.png)

 

 　②、在子工程中声明对父工程的引用

```
<!--子工程中声明对父工程的引用  -->
<parent>
      <groupId>com.ys.maven</groupId>
      <artifactId>Parent</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <!-- 以当前工程文件为基准的父工程 pom.xml文件的相对路径（可以不配置） -->
      <relativePath>../Parent/pom.xml</relativePath>
</parent>
```



 　③、将子工程的坐标中与父工程坐标重复的内容删除（不删除也可以，为了简洁）

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831220907655-1243434207.png)

 

 　④、在父工程中统一那个 jar 的版本依赖

```
<dependencyManagement>
      <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.8</version>
            <scope>test</scope>
        </dependency>
      </dependencies>
  </dependencyManagement>
```

　　**dependencyManagement标签管理的依赖，其实没有真正依赖，它只是管理依赖的版本。**

 　

　　⑤、在子工程中删除 Junit 的版本号

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831221523187-91244246.png)

　　**以后要更改版本号，我们只需要更改父工程中的版本号即可！！！**

 　

　　⑥、父工程通过 properties 统一管理版本号

　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831222023765-1349206946.png)

　　我们可以通过<properties></properties>自定义标签，然后在标签里面填写常量，这种方法不仅可以用来管理版本号，还可以用来管理比如设置某种编码等等。

 


###  2、聚合

 　**需求场景：**

　　在真实项目中，一个项目有表现层、业务层、持久层等。我们在用Maven 管理项目的时候，通常为创建多个 Maven 工程，也就是一个项目的多个模块。但是这样分成多个模块了，当我们进行项目打包发布的时候，那么要每一个模块都执行打包操作吗？这种重复的操作我们怎么才能避免呢？

　　**解决办法：**

　　创建一个聚合工程，将其他的各个模块都由这个聚合工程来管理，那么我们在进行项目发布的时候，只需要打包这个聚合工程就可以了。

 

　　**第一步：创建聚合工程（注意聚合工程的打包方式也必须为 pom，通常由 上面所讲的父工程来充当聚合工程）**

 　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831223644062-1319033203.png)

 

 　**第二步：创建子工程：业务层**

　　　　①、选择 Maven Module

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831223741655-343347591.png)

 

 　　　②、填写子工程模块名，打包方式选择 jar（子工程除了 web 层我们打包方式选择 war ，其余的都选择 jar）

　　　　![img](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170831223932749-1344686600.png)

　　　　

　　　　

 　**第三步：创建子工程：表现层和持久层**

　　　　**创建步骤和前面一样，注意表现层打包方式我们要选择 war，因为要发布到 tomcat 容器运行。**

 

　　**第四步：在聚合工程中添加子工程的引用**

```
 <modules>
    <module>Service</module>
    <module>Controller</module>
    <module>Mapper</module>
  </modules>
```

　　

 　注意：

　　1、这里虽然各个模块有依赖关系，但是 <module></modelu>可以不让依赖顺序添加，maven会自动识别依赖关系进行编译打包。

　　2、这里总的聚合工程随便哪个工程都可以，但是通常用 Parent 工程来完成。