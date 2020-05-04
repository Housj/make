https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/

# 入门

如果您是从Spring Boot或“ Spring”开始的，请先阅读本节。它回答了基本的“什么？”，“如何？” 和“为什么？” 问题。它包括对Spring Boot的介绍以及安装说明。然后，我们将引导您构建第一个Spring Boot应用程序，并讨论一些核心原理。

## 1.介绍Spring Boot

Spring Boot使创建可运行的独立，生产级基于Spring的应用程序变得容易。我们对Spring平台和第三方库持固执己见的观点，这样您就可以以最小的麻烦开始使用。大多数Spring Boot应用程序只需要很少的Spring配置。

您可以使用Spring Boot创建Java应用程序，该应用程序可以通过使用`java -jar`或更传统的战争部署来启动。我们还提供了一个运行“ spring脚本”的命令行工具。

我们的主要目标是：

- 为所有Spring开发提供根本上更快且可广泛访问的入门经验。
- 开箱即用，但随着需求开始偏离默认值，您会很快摆脱困境。
- 提供一系列大型项目通用的非功能性功能（例如嵌入式服务器，安全性，指标，运行状况检查和外部化配置）。
- 完全没有代码生成，也不需要XML配置。

## 2.系统要求

Spring Boot 2.2.6.RELEASE需要[Java 8，](https://www.java.com/)并且与Java 13（包括）兼容。 还需要[Spring Framework 5.2.5.RELEASE](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/)或更高版本。

为以下构建工具提供了明确的构建支持：

| 制作工具 | 版                               |
| :------- | :------------------------------- |
| 马文     | 3.3+                             |
| 摇篮     | 5.x和6.x（也支持4.10，但已弃用） |

### 2.1。Servlet容器

Spring Boot支持以下嵌入式servlet容器：

| 名称         | Servlet版本 |
| :----------- | :---------- |
| Tomcat 9.0   | 4.0         |
| 码头9.4      | 3.1         |
| Undertow 2.0 | 4.0         |

您还可以将Spring Boot应用程序部署到任何Servlet 3.1+兼容容器中。

## 3.安装Spring Boot

Spring Boot可以与“经典” Java开发工具一起使用，也可以作为命令行工具安装。无论哪种方式，都需要[Java SDK v1.8](https://www.java.com/)或更高版本。在开始之前，您应该使用以下命令检查当前的Java安装：

```
$ java-版本
```

如果您不熟悉Java开发，或者想尝试使用Spring Boot，则可能要先尝试使用[Spring Boot CLI](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started-installing-the-cli)（命令行界面）。否则，请继续阅读“经典”安装说明。

### 3.1。Java开发人员的安装说明

您可以像使用任何标准Java库一样使用Spring Boot。为此，请`spring-boot-*.jar`在类路径中包含适当的文件。Spring Boot不需要任何特殊的工具集成，因此您可以使用任何IDE或文本编辑器。另外，Spring Boot应用程序没有什么特别之处，因此您可以像运行其他Java程序一样运行和调试Spring Boot应用程序。

尽管您*可以*复制Spring Boot jar，但是我们通常建议您使用支持依赖关系管理的构建工具（例如Maven或Gradle）。

#### 3.1.1。Maven安装

Spring Boot与Apache Maven 3.3或更高版本兼容。如果尚未安装Maven，则可以按照[maven.apache.org上](https://maven.apache.org/)的说明进行操作。

|      | 在许多操作系统上，Maven可以与程序包管理器一起安装。如果您使用OSX Homebrew，请尝试`brew install maven`。Ubuntu用户可以运行`sudo apt-get install maven`。具有[Chocolatey的](https://chocolatey.org/) Windows用户可以`choco install maven`从提升的（管理员）提示符下运行。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring Boot依赖项使用`org.springframework.boot` `groupId`。通常，您的Maven POM文件从`spring-boot-starter-parent`项目继承，并声明对一个或多个[“启动器”的](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-starter)依赖关系。Spring Boot还提供了一个可选的[Maven插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-plugin)来创建可执行jar。

以下清单显示了一个典型的`pom.xml`文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
    </parent>

    <!-- Override inherited settings -->
    <description/>
    <developers>
        <developer/>
    </developers>
    <licenses>
        <license/>
    </licenses>
    <scm>
        <url/>
    </scm>
    <url/>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

|      | 这`spring-boot-starter-parent`是使用Spring Boot的一种好方法，但可能并不总是适合。有时您可能需要从其他父POM继承，或者您可能不喜欢我们的默认设置。在这种情况下，请参阅[using-spring-boot.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-maven-without-a-parent)以获取使用`import`范围的替代解决方案。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.1.2。摇篮安装

Spring Boot与5.x和6.x兼容。还支持4.10，但不支持该支持，在将来的版本中将删除该支持。如果尚未安装Gradle，则可以按照[gradle.org上](https://gradle.org/)的说明进行[操作](https://gradle.org/)。

可以使用来声明Spring Boot依赖项`org.springframework.boot` `group`。通常，您的项目声明对一个或多个[“启动器”的](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-starter)依赖。Spring Boot提供了一个有用的[Gradle插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-gradle-plugin)，可用于简化依赖项声明和创建可执行jar。

摇篮包装

当您需要构建项目时，Gradle包装器提供了一种“获取” Gradle的好方法。它是一个小的脚本和库，您随代码一起提交以引导构建过程。有关详细信息，请参见[docs.gradle.org/current/userguide/gradle_wrapper.html](https://docs.gradle.org/current/userguide/gradle_wrapper.html)。

有关Spring Boot和Gradle [入门的](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html//#getting-started)更多详细信息，可以在Gradle插件参考指南的“ [入门”部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html//#getting-started)中找到。

### 3.2。安装Spring Boot CLI

Spring Boot CLI（命令行界面）是一个命令行工具，可用于快速使用Spring进行原型设计。它使您可以运行[Groovy](http://groovy-lang.org/)脚本，这意味着您具有类似Java的熟悉语法，而没有太多样板代码。

您无需使用CLI即可与Spring Boot一起使用，但这绝对是使Spring应用程序启动的最快方法。

#### 3.2.1。手动安装

您可以从Spring软件存储库下载Spring CLI发行版：

- [spring-boot-cli-2.2.6.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.6.RELEASE/spring-boot-cli-2.2.6.RELEASE-bin.zip)
- [弹簧启动cli-2.2.6.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.6.RELEASE/spring-boot-cli-2.2.6.RELEASE-bin.tar.gz)

也提供最先进的 [快照分发](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)。

下载完成后，请按照解压缩后的存档中的[INSTALL.txt](https://raw.githubusercontent.com/spring-projects/spring-boot/v2.2.6.RELEASE/spring-boot-project/spring-boot-cli/src/main/content/INSTALL.txt)说明进行操作。总之，文件目录中有一个`spring`脚本（`spring.bat`对于Windows）。或者，您可以使用该文件（脚本可帮助您确保正确设置类路径）。`bin/``.zip``java -jar``.jar`

#### 3.2.2。使用SDKMAN安装！

SDKMAN！（软件开发套件管理器）可用于管理各种二进制SDK的多个版本，包括Groovy和Spring Boot CLI。获取SDKMAN！从[sdkman.io](https://sdkman.io/)并使用以下命令安装Spring Boot：

```
$ sdk安装springboot
$ spring --version
Spring Boot v2.2.6.RELEASE
```

如果您为CLI开发功能并希望轻松访问所构建的版本，请使用以下命令：

```
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-2.2.6.RELEASE-bin/spring-2.2.6.RELEASE/
$ sdk默认的springboot dev
$ spring --version
Spring CLI v2.2.6.RELEASE
```

前面的说明安装`spring`称为`dev`实例的本地实例。它指向您的目标构建位置，因此每次重建Spring Boot `spring`都是最新的。

您可以通过运行以下命令来查看它：

```
$ sdk ls springboot

================================================== ==============================
可用的Springboot版本
================================================== ==============================
> +开发
* 2.2.6。发布

================================================== ==============================
+-本地版本
*-已安装
>-当前正在使用
================================================== ==============================
```

#### 3.2.3。OSX Homebrew安装

如果您使用的是Mac，并且使用[Homebrew](https://brew.sh/)，则可以使用以下命令安装Spring Boot CLI：

```
$酿造水龙头枢纽/水龙头
$ brew安装springboot
```

自制软件安装`spring`到`/usr/local/bin`。

|      | 如果看不到该公式，则说明brew的安装可能已过期。在这种情况下，请运行`brew update`并重试。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.2.4。MacPorts安装

如果您使用的是Mac并使用[MacPorts](https://www.macports.org/)，则可以使用以下命令安装Spring Boot CLI：

```
$ sudo port install spring-boot-cli
```

#### 3.2.5。命令行补全

Spring Boot CLI包括为[BASH](https://en.wikipedia.org/wiki/Bash_(Unix_shell))和[zsh](https://en.wikipedia.org/wiki/Z_shell) Shell 提供命令完成的脚本。您可以`source`将脚本（也称为`spring`）放在任何shell中，或将其放在个人或系统范围内的bash完成初始化中。在Debian系统上，系统范围内的脚本位于其中，`/shell-completion/bash`并且在启动新Shell时将执行该目录中的所有脚本。例如，如果您是使用SDKMAN！安装的，则要手动运行脚本，请使用以下命令：

```
$。〜/ .sdkman / candidates / springboot / current / shell-completion / bash / spring
$ spring <这里点击标签>
  抢救jar运行测试版
```

|      | 如果使用Homebrew或MacPorts安装Spring Boot CLI，则命令行完成脚本会自动注册到您的Shell中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.2.6。Windows Scoop安装

如果您使用的是Windows并使用[Scoop](https://scoop.sh/)，则可以使用以下命令安装Spring Boot CLI：

```
>铲斗添加附加功能
>独家安装springboot
```

Scoop安装`spring`到`~/scoop/apps/springboot/current/bin`。

|      | 如果您没有看到应用清单，则可能是因为瓢的安装已过期。在这种情况下，请运行`scoop update`并重试。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.2.7。快速入门Spring CLI示例

您可以使用以下Web应用程序来测试安装。首先，创建一个名为的文件`app.groovy`，如下所示：

```groovy
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```

然后从外壳运行它，如下所示：

```
$ spring run app.groovy
```

|      | 由于依赖项已下载，因此您的应用程序的第一次运行很慢。随后的运行要快得多。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`localhost:8080`在您喜欢的网络浏览器中打开。您应该看到以下输出：

```
你好，世界！
```

### 3.3。从较早版本的Spring Boot升级

如果要从`1.x`Spring Boot发行版进行升级，请查看[项目Wiki上的“迁移指南”，](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)其中提供了详细的升级说明。还请检查[“发行说明”](https://github.com/spring-projects/spring-boot/wiki)以获取每个发行版的“新功能和值得注意的功能”列表。

升级到新功能版本时，某些属性可能已被重命名或删除。Spring Boot提供了一种在启动时分析应用程序环境并打印诊断的方法，而且还可以在运行时为您临时迁移属性。要启用该功能，请将以下依赖项添加到您的项目中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-properties-migrator</artifactId>
    <scope>runtime</scope>
</dependency>
```

|      | 较晚添加到环境的属性（例如使用时`@PropertySource`）将不被考虑。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 迁移完成后，请确保从项目的依赖项中删除此模块。 |
| ---- | ---------------------------------------------- |
|      |                                                |

要升级现有的CLI安装，请使用适当的程序包管理器命令（例如，`brew upgrade`）。如果您手动安装了CLI，请遵循[标准说明](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started-manual-cli-installation)，记住要更新`PATH`环境变量以删除所有较旧的引用。

## 4.开发您的第一个Spring Boot应用程序

本节介绍如何开发一个简单的“ Hello World！”。Web应用程序，重点介绍了Spring Boot的一些关键功能。我们使用Maven来构建该项目，因为大多数IDE都支持它。

|      | 该[spring.io](https://spring.io/)网站包含了许多“入门” [指南](https://spring.io/guides)使用Spring的引导。如果您需要解决特定问题，请首先检查。通过转到[start.spring.io](https://start.spring.io/)并从依赖项搜索器中选择“ Web”启动器，可以简化以下步骤。这样做会生成一个新的项目结构，以便您可以[立即开始编码](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started-first-application-code)。查看[Spring Initializr文档](https://docs.spring.io/initializr/docs/current/reference/html//#user-guide)以获取更多详细信息。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在开始之前，请打开终端并运行以下命令，以确保安装了有效的Java和Maven版本：

```
$ java-版本
Java版本“ 1.8.0_102”
Java（TM）SE运行时环境（内部版本1.8.0_102-b14）
Java HotSpot（TM）64位服务器VM（内部版本25.102-b14，混合模式）
$ mvn -v
Apache Maven 3.5.4（1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14：33：14-04：00）
Maven主页：/usr/local/Cellar/maven/3.3.9/libexec
Java版本：1.8.0_102，供应商：Oracle Corporation
```

|      | 该示例需要在其自己的文件夹中创建。随后的说明假定您已经创建了一个合适的文件夹，并且它是当前目录。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.1。创建POM

我们需要先创建一个Maven `pom.xml`文件。本`pom.xml`是用来构建项目的配方。打开您喜欢的文本编辑器并添加以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
    </parent>

    <description/>
    <developers>
        <developer/>
    </developers>
    <licenses>
        <license/>
    </licenses>
    <scm>
        <url/>
    </scm>
    <url/>

    <!-- Additional lines to be added here... -->

</project>
```

上面的清单应为您提供有效的构建。您可以通过运行`mvn package`进行测试（目前，您可以忽略“ jar将为空-没有内容标记为包含！”警告）。

|      | 此时，您可以将项目导入到IDE中（大多数现代Java IDE都包含对Maven的内置支持）。为简单起见，在此示例中，我们继续使用纯文本编辑器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.2。添加类路径依赖

Spring Boot提供了许多“启动器”，使您可以将jar添加到类路径中。我们的烟雾测试应用程序使用`spring-boot-starter-parent`的`parent`聚甲醛的部分。本`spring-boot-starter-parent`是一个特殊的启动提供有用的Maven的默认值。它还提供了一个[`dependency-management`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)部分，以便您可以省略`version`“祝福”依赖项的标签。

其他“入门”提供了开发特定类型的应用程序时可能需要的依赖项。由于我们正在开发Web应用程序，因此我们添加了`spring-boot-starter-web`依赖项。在此之前，我们可以通过运行以下命令来查看当前的状态：

```
$ mvn依赖关系：树

[INFO] com.example：myproject：jar：0.0.1-SNAPSHOT
```

该`mvn dependency:tree`命令显示项目依赖关系的树形表示。您可以看到它`spring-boot-starter-parent`本身不提供任何依赖关系。要添加必要的依赖项，请编辑您的依赖项，`pom.xml`并`spring-boot-starter-web`在该`parent`部分的正下方添加依赖项：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

如果`mvn dependency:tree`再次运行，您会发现现在还有许多其他依赖项，包括Tomcat Web服务器和Spring Boot本身。

### 4.3。编写代码

要完成我们的应用程序，我们需要创建一个Java文件。默认情况下，Maven会从编译源代码`src/main/java`，因此您需要创建该文件夹结构，然后添加一个名为的文件以`src/main/java/Example.java`包含以下代码：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Example.class, args);
    }

}
```

尽管这里没有太多代码，但正在进行很多工作。我们将在接下来的几节中逐步介绍重要部分。

#### 4.3.1。@RestController和@RequestMapping注释

`@RestController`称为*构造型*注释。它为阅读代码的人和Spring提供了提示，提示该类起特定的作用。在这种情况下，我们的类是web `@Controller`，因此Spring在处理传入的Web请求时会考虑使用它。

`@RequestMapping`注释提供“路由”的信息。它告诉Spring任何具有`/`路径的HTTP请求都应映射到该`home`方法。该`@RestController`注解告诉Spring使得到的字符串直接返回给调用者。

|      | 在`@RestController`与`@RequestMapping`注解是Spring MVC的注解（他们并不是专门针对春季启动）。有关更多详细信息，请参见Spring参考文档中的[MVC部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 4.3.2。@EnableAutoConfiguration批注

第二个类级别的注释是`@EnableAutoConfiguration`。这个注释告诉Spring Boot根据所添加的jar依赖关系“猜测”您如何配置Spring。由于`spring-boot-starter-web`添加了Tomcat和Spring MVC，因此自动配置假定您正在开发Web应用程序并相应地设置Spring。

启动器和自动配置

自动配置旨在与“启动器”配合使用，但是这两个概念并没有直接联系在一起。您可以自由选择启动器之外的jar依赖项。Spring Boot仍然尽其最大努力来自动配置您的应用程序。

#### 4.3.3。“主要”方法

我们应用程序的最后一部分是`main`方法。这只是遵循Java约定的应用程序入口点的标准方法。我们的main方法`SpringApplication`通过调用委托给Spring Boot的类`run`。 `SpringApplication`引导我们的应用程序，启动Spring，这反过来又启动了自动配置的Tomcat Web服务器。我们需要将`Example.class`一个参数传递给该`run`方法，以判断`SpringApplication`哪个是主要的Spring组件。该`args`数组也将通过以公开任何命令行参数。

### 4.4。运行示例

此时，您的应用程序应该可以工作了。由于使用了`spring-boot-starter-parent`POM，因此有一个有用的`run`目标，可以用来启动应用程序。`mvn spring-boot:run`从根项目目录中键入以启动应用程序。您应该看到类似于以下内容的输出：

```
$ mvn spring-boot：运行

  。____ _ __ _ _
 / \\ / ___'_ __ _ _（_）_ __ __ _ _ \ \ \ \
（（）\ ___ |'_ |'_ | |'_ \ / _` | \ \ \ \ \
 \\ / ___）| | _）| | | | | || （_ | |））））
  '| ____ | .__ | _ | | _ | _ | | _ \ __，| / / / /
 ======== | _ | ============= | ___ / = / _ / _ / _ /
 :: Spring Boot ::（v2.2.6.RELEASE）
....... 。。
....... 。。（日志输出在这里）
....... 。。
........在2.222秒内启动示例（JVM运行6.514）
```

如果您打开Web浏览器到`localhost:8080`，则应该看到以下输出：

```
你好，世界！
```

要正常退出该应用程序，请按`ctrl-c`。

### 4.5。创建一个可执行的Jar

通过创建一个可以在生产环境中运行的完全独立的可执行jar文件来结束示例。可执行jar（有时称为“胖jar”）是包含您的已编译类以及代码需要运行的所有jar依赖项的归档文件。

可执行jar和Java

Java没有提供加载嵌套jar文件（jar中本身包含的jar文件）的标准方法。如果您要分发独立的应用程序，则可能会出现问题。

为了解决这个问题，许多开发人员使用“超级”罐子。uber jar将来自应用程序所有依赖项的所有类打包到单个存档中。这种方法的问题在于，很难查看应用程序中包含哪些库。如果在多个jar中使用相同的文件名（但具有不同的内容），也可能会产生问题。

Spring Boot采用了[另一种方法](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-executable-jar-format.html#executable-jar)，实际上允许您直接嵌套jar。

要创建可执行jar，我们需要将添加`spring-boot-maven-plugin`到`pom.xml`。为此，请在该`dependencies`部分下方插入以下行：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

|      | 所述`spring-boot-starter-parent`POM包括``配置以结合`repackage`目标。如果不使用父POM，则需要自己声明此配置。有关详细信息，请参见[插件文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin//usage.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

保存您的代码，`pom.xml`然后从命令行运行`mvn package`，如下所示：

```
$ mvn软件包

[INFO]正在扫描项目...
[信息]
[INFO] ----------------------------------------------- -------------------------
[INFO]构建myproject 0.0.1-SNAPSHOT
[INFO] ----------------------------------------------- -------------------------
[INFO] ......
[INFO] --- maven-jar-plugin：2.4：jar（默认-jar）@ myproject-
[INFO]构建jar：/Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[信息]
[INFO] --- spring-boot-maven-plugin：2.2.6.RELEASE：repackage（默认）@ myproject-
[INFO] ----------------------------------------------- -------------------------
[INFO]建立成功
[INFO] ----------------------------------------------- -------------------------
```

如果查看`target`目录，应该看到`myproject-0.0.1-SNAPSHOT.jar`。该文件的大小应为10 MB左右。如果您想窥视内部，可以使用`jar tvf`，如下所示：

```
$ jar tvf target / myproject-0.0.1-SNAPSHOT.jar
```

您还应该`myproject-0.0.1-SNAPSHOT.jar.original`在`target`目录中看到一个更小的文件。这是Maven在Spring Boot重新打包之前创建的原始jar文件。

要运行该应用程序，请使用以下`java -jar`命令：

```
$ java -jar target / myproject-0.0.1-SNAPSHOT.jar

  。____ _ __ _ _
 / \\ / ___'_ __ _ _（_）_ __ __ _ _ \ \ \ \
（（）\ ___ |'_ |'_ | |'_ \ / _` | \ \ \ \ \
 \\ / ___）| | _）| | | | | || （_ | |））））
  '| ____ | .__ | _ | | _ | _ | | _ \ __，| / / / /
 ======== | _ | ============= | ___ / = / _ / _ / _ /
 :: Spring Boot ::（v2.2.6.RELEASE）
....... 。。
....... 。。（日志输出在这里）
....... 。。
........在2.536秒内启动示例（JVM运行2.864）
```

和以前一样，要退出该应用程序，请按`ctrl-c`。

## 5.接下来阅读什么

希望本节提供了一些Spring Boot基础知识，并带您开始编写自己的应用程序。如果您是面向任务的开发人员，则可能要跳到[spring.io](https://spring.io/)并查看一些解决特定“我如何使用Spring进行操作”的[入门](https://spring.io/guides/)指南。问题。我们也有特定[于](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto) Spring Boot的“ [操作方法](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto) ”参考文档。

否则，下一个逻辑步骤是阅读*[using-spring-boot.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot)*。如果您真的不耐烦，还可以继续阅读有关*[Spring Boot功能的信息](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features)*。

最后更新时间2020-03-26 11:43:46 UTC



# 使用Spring Boot

本节将详细介绍如何使用Spring Boot。它涵盖了诸如构建系统，自动配置以及如何运行应用程序之类的主题。我们还将介绍一些Spring Boot最佳实践。尽管Spring Boot没什么特别的（它只是您可以使用的另一个库），但是有一些建议可以使您的开发过程更轻松一些。

如果您是从Spring Boot开始的，那么在进入本节之前，您应该阅读*[入门指南](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started)*。

## 1.构建系统

强烈建议您选择一个支持[*依赖关系管理*](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)并且可以使用发布到“ Maven Central”存储库的工件的构建系统。我们建议您选择Maven或Gradle。可以使Spring Boot与其他构建系统（例如，Ant）一起使用，但是它们并没有得到很好的支持。

### 1.1。依赖管理

每个Spring Boot版本都提供了它所支持的依赖关系的精选列表。实际上，您不需要为构建配置中的所有这些依赖项提供版本，因为Spring Boot会为您管理该版本。当您升级Spring Boot本身时，这些依赖项也会以一致的方式升级。

|      | 您仍然可以指定版本，并在需要时覆盖Spring Boot的建议。 |
| ---- | ----------------------------------------------------- |
|      |                                                       |

精选列表包含可与Spring Boot一起使用的所有spring模块以及完善的第三方库列表。该列表作为可与[Maven](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-maven-parent-pom)和[Gradle](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-gradle)一起使用的标准[材料清单（`spring-boot-dependencies`）](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-maven-without-a-parent)提供。

|      | 每个Spring Boot版本都与Spring Framework的基本版本相关联。我们**强烈**建议您不要指定其版本。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.2 Maven

Maven用户可以从`spring-boot-starter-parent`项目继承以获得合理的默认值。父项目提供以下功能：

- Java 1.8是默认的编译器级别。
- UTF-8源编码。
- 一个[依赖管理部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)，从春天启动依赖性POM，管理公共依赖的版本继承。当在您自己的pom中使用时，这种依赖关系管理使您可以为这些依赖项省略<version>标记。
- 具有执行ID 的[`repackage`目标](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin//repackage-mojo.html)的`repackage`执行。
- 明智的[资源过滤](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)。
- 合理的插件配置（[exec插件](https://www.mojohaus.org/exec-maven-plugin/)，[Git提交ID](https://github.com/ktoso/maven-git-commit-id-plugin)和[shade](https://maven.apache.org/plugins/maven-shade-plugin/)）。
- 针对`application.properties`和`application.yml`包括特定于配置文件的文件（例如`application-dev.properties`和`application-dev.yml`）进行明智的资源过滤

请注意，由于`application.properties`和`application.yml`文件都接受Spring样式的占位符（`${…}`），因此Maven过滤已更改为使用`@..@`占位符。（您可以通过设置名为的Maven属性来覆盖它`resource.delimiter`。）

#### 1.2.1。继承入门级父级

要将您的项目配置为继承自`spring-boot-starter-parent`，请设置`parent`如下：

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
</parent>
```

|      | 您只需要为此依赖项指定Spring Boot版本号。如果导入其他启动器，则可以安全地省略版本号。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

使用该设置，您还可以通过覆盖自己项目中的属性来覆盖各个依赖项。例如，要升级到另一个Spring Data发布火车，您可以将以下内容添加到您的`pom.xml`：

```xml
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

|      | 检查[`spring-boot-dependencies`pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)以获取受支持属性的列表。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.2.2。在没有父POM的情况下使用Spring Boot

并非每个人都喜欢从`spring-boot-starter-parent`POM 继承。您可能需要使用自己的公司标准父级，或者可能希望显式声明所有Maven配置。

如果您不想使用`spring-boot-starter-parent`，仍然可以通过使用`scope=import`依赖项来保留依赖项管理（而不是插件管理）的好处，如下所示：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

如上所述，前面的示例设置不允许您使用属性来覆盖各个依赖项。为了达到同样的效果，你需要添加的条目`dependencyManagement`项目的**之前**的`spring-boot-dependencies`条目。例如，要升级到另一个Spring Data发布系列，可以将以下元素添加到`pom.xml`：

```xml
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.6.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

|      | 在前面的示例中，我们指定了*BOM*，但是可以以相同的方式覆盖任何依赖项类型。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 1.2.3。使用Spring Boot Maven插件

Spring Boot包含一个[Maven插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-plugin)，可以将项目打包为可执行jar。``如果要使用插件，请将该插件添加到您的部分，如以下示例所示：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

|      | 如果您使用Spring Boot启动器的父pom，则只需添加插件。除非您要更改父级中定义的设置，否则无需对其进行配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.3。摇篮

要了解有关将Spring Boot与Gradle结合使用的信息，请参阅Spring Boot的Gradle插件的文档：

- 参考（[HTML](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html/)和[PDF](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf)）
- [API](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/api/)

### 1.4。蚂蚁

可以使用Apache Ant + Ivy构建Spring Boot项目。该`spring-boot-antlib`“的antlib”模块还可以帮助蚂蚁创建可执行的JAR文件。

为了声明依赖关系，典型`ivy.xml`文件看起来类似于以下示例：

```xml
<ivy-module version="2.0">
    <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
    <configurations>
        <conf name="compile" description="everything needed to compile this module" />
        <conf name="runtime" extends="compile" description="everything needed to run this module" />
    </configurations>
    <dependencies>
        <dependency org="org.springframework.boot" name="spring-boot-starter"
            rev="${spring-boot.version}" conf="compile" />
    </dependencies>
</ivy-module>
```

典型`build.xml`的示例如下所示：

```xml
<project
    xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">

    <property name="spring-boot.version" value="2.2.6.RELEASE" />

    <target name="resolve" description="--> retrieve dependencies with ivy">
        <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
    </target>

    <target name="classpaths" depends="resolve">
        <path id="compile.classpath">
            <fileset dir="lib/compile" includes="*.jar" />
        </path>
    </target>

    <target name="init" depends="classpaths">
        <mkdir dir="build/classes" />
    </target>

    <target name="compile" depends="init" description="compile">
        <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
    </target>

    <target name="build" depends="compile">
        <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
            <spring-boot:lib>
                <fileset dir="lib/runtime" />
            </spring-boot:lib>
        </spring-boot:exejar>
    </target>
</project>
```

|      | 如果您不想使用该`spring-boot-antlib`模块，请参见*[howto.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-build-an-executable-archive-with-ant)* “操作方法”。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.5。初学者

入门程序是一组方便的依赖项描述符，您可以在应用程序中包括它们。您可以一站式购买所需的所有Spring和相关技术，而不必搜寻示例代码和依赖描述符的复制粘贴负载。例如，如果要开始使用Spring和JPA进行数据库访问，请`spring-boot-starter-data-jpa`在项目中包括依赖项。

入门程序包含许多启动项目并快速运行所需的依赖项，并且具有一组受支持的受管传递性依赖项。

名字叫什么

所有**官方**入门者都遵循类似的命名方式。`spring-boot-starter-*`，其中`*`是特定类型的应用程序。这种命名结构旨在在您需要寻找入门者时提供帮助。许多IDE中的Maven集成使您可以按名称搜索依赖项。例如，在安装了适当的Eclipse或STS插件的情况下，您可以按`ctrl-space`POM编辑器，然后键入“ spring-boot-starter”以获取完整列表。

如“ [创建您自己的启动程序](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-starter) ”部分中所述，第三方启动程序不应以开头`spring-boot`，因为它保留用于正式的Spring Boot构件。而是，第三方启动程序通常以项目名称开头。例如，`thirdpartyproject`通常将名为的第三方启动程序项目命名为`thirdpartyproject-spring-boot-starter`。

Spring Boot在该`org.springframework.boot`组下提供了以下应用程序启动器：

| 名称                                          | 描述                                                         | Pom                                                          |
| :-------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter`                         | 核心入门工具，包括自动配置支持，日志记录和YAML               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-activemq`                | 使用Apache ActiveMQ的JMS消息传递入门                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| `spring-boot-starter-amqp`                    | 使用Spring AMQP和Rabbit MQ的入门                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| `spring-boot-starter-aop`                     | 使用Spring AOP和AspectJ进行面向方面编程的入门                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |
| `spring-boot-starter-artemis`                 | 使用Apache Artemis的JMS消息传递入门                          | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml) |
| `spring-boot-starter-batch`                   | 使用Spring Batch的入门                                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml) |
| `spring-boot-starter-cache`                   | 开始使用Spring Framework的缓存支持                           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
| `spring-boot-starter-cloud-connectors`        | 使用Spring Cloud Connectors的入门程序，可简化与Cloud Foundry和Heroku等云平台中服务的连接。不推荐使用Java CFEnv | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml) |
| `spring-boot-starter-data-cassandra`          | 使用Cassandra分布式数据库和Spring Data Cassandra的入门       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml) |
| `spring-boot-starter-data-cassandra-reactive` | 使用Cassandra分布式数据库和Spring Data Cassandra Reactive的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
| `spring-boot-starter-data-couchbase`          | 使用Couchbase面向文档的数据库和Spring Data Couchbase的入门   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml) |
| `spring-boot-starter-data-couchbase-reactive` | 使用Couchbase面向文档的数据库和Spring Data Couchbase Reactive的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml) |
| `spring-boot-starter-data-elasticsearch`      | 使用Elasticsearch搜索和分析引擎以及Spring Data Elasticsearch的入门者 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
| `spring-boot-starter-data-jdbc`               | 使用Spring Data JDBC的入门                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jdbc/pom.xml) |
| `spring-boot-starter-data-jpa`                | 将Spring Data JPA与Hibernate结合使用的入门                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| `spring-boot-starter-data-ldap`               | 使用Spring Data LDAP的入门                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml) |
| `spring-boot-starter-data-mongodb`            | 使用MongoDB面向文档的数据库和Spring Data MongoDB的入门       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
| `spring-boot-starter-data-mongodb-reactive`   | 使用MongoDB面向文档的数据库和Spring Data MongoDB Reactive的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml) |
| `spring-boot-starter-data-neo4j`              | 使用Neo4j图形数据库和Spring Data Neo4j的入门                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml) |
| `spring-boot-starter-data-redis`              | 使用Redis键值数据存储与Spring Data Redis和Lettuce客户端的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
| `spring-boot-starter-data-redis-reactive`     | 将Redis键值数据存储与Spring Data Redis Reacting和Lettuce客户端一起使用的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml) |
| `spring-boot-starter-data-rest`               | 使用Spring Data REST通过REST公开Spring数据存储库的入门       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml) |
| `spring-boot-starter-data-solr`               | 将Apache Solr搜索平台与Spring Data Solr结合使用的入门        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml) |
| `spring-boot-starter-freemarker`              | 使用FreeMarker视图构建MVC Web应用程序的入门                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml) |
| `spring-boot-starter-groovy-templates`        | 使用Groovy模板视图构建MVC Web应用程序的入门                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml) |
| `spring-boot-starter-hateoas`                 | 使用Spring MVC和Spring HATEOAS构建基于超媒体的RESTful Web应用程序的入门者 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml) |
| `spring-boot-starter-integration`             | 使用Spring Integration的入门                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml) |
| `spring-boot-starter-jdbc`                    | 结合使用JDBC和HikariCP连接池的入门                           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml) |
| `spring-boot-starter-jersey`                  | 使用JAX-RS和Jersey构建RESTful Web应用程序的入门。的替代品[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-web) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml) |
| `spring-boot-starter-jooq`                    | 使用jOOQ访问SQL数据库的入门。替代[`spring-boot-starter-data-jpa`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-data-jpa)或[`spring-boot-starter-jdbc`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-jdbc) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml) |
| `spring-boot-starter-json`                    | 读写JSON入门                                                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml) |
| `spring-boot-starter-jta-atomikos`            | 使用Atomikos的JTA交易入门                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |
| `spring-boot-starter-jta-bitronix`            | 使用Bitronix的JTA交易入门                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml) |
| `spring-boot-starter-mail`                    | 使用Java Mail和Spring Framework的电子邮件发送支持的入门      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
| `spring-boot-starter-mustache`                | 使用Mustache视图构建Web应用程序的入门                        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml) |
| `spring-boot-starter-oauth2-client`           | 使用Spring Security的OAuth2 / OpenID Connect客户端功能的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-client/pom.xml) |
| `spring-boot-starter-oauth2-resource-server`  | 使用Spring Security的OAuth2资源服务器功能的入门              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-resource-server/pom.xml) |
| `spring-boot-starter-quartz`                  | 入门使用Quartz Scheduler                                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
| `spring-boot-starter-rsocket`                 | 用于构建RSocket客户端和服务器的入门。                        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-rsocket/pom.xml) |
| `spring-boot-starter-security`                | 使用Spring Security的入门                                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| `spring-boot-starter-test`                    | 用于使用包括JUnit，Hamcrest和Mockito在内的库测试Spring Boot应用程序的入门程序 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| `spring-boot-starter-thymeleaf`               | 使用Thymeleaf视图构建MVC Web应用程序的入门                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| `spring-boot-starter-validation`              | 通过Hibernate Validator使用Java Bean验证的入门               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| `spring-boot-starter-web`                     | 使用Spring MVC构建Web（包括RESTful）应用程序的入门者。使用Tomcat作为默认的嵌入式容器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| `spring-boot-starter-web-services`            | 使用Spring Web Services的入门                                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml) |
| `spring-boot-starter-webflux`                 | 使用Spring Framework的Reactive Web支持构建WebFlux应用程序的入门者 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml) |
| `spring-boot-starter-websocket`               | 使用Spring Framework的WebSocket支持构建WebSocket应用程序的入门 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |

除了应用程序启动程序，以下启动程序可用于添加*[生产就绪](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready)*功能：

| 名称                           | 描述                                                         | Pom                                                          |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-actuator` | 使用Spring Boot的Actuator的入门程序，它提供了生产就绪功能，可帮助您监视和管理应用程序 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) |

最后，Spring Boot还包括以下启动程序，如果您想排除或交换特定的技术方面，可以使用这些启动程序：

| 名称                                | 描述                                                         | Pom                                                          |
| :---------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-jetty`         | 使用Jetty作为嵌入式servlet容器的入门者。的替代品[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml) |
| `spring-boot-starter-log4j2`        | 使用Log4j2进行日志记录的启动器。的替代品[`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-logging) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml) |
| `spring-boot-starter-logging`       | 使用Logback进行日志记录的启动器。默认记录启动器              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml) |
| `spring-boot-starter-reactor-netty` | 使用Reactor Netty作为嵌入式反应式HTTP服务器的入门者。        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
| `spring-boot-starter-tomcat`        | 入门，用于将Tomcat用作嵌入式Servlet容器。默认使用的servlet容器启动器[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-web) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml) |
| `spring-boot-starter-undertow`      | 使用Undertow作为嵌入式servlet容器的入门。的替代品[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml) |

|      | 有关社区贡献的其他入门者的列表，请参阅GitHub 中模块中的[README文件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)`spring-boot-starters`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 2.构建代码

Spring Boot不需要任何特定的代码布局即可工作。但是，有一些最佳实践可以帮助您。

### 2.1。使用“默认”包

当一个类不包含`package`声明时，它被认为是在“默认包”中。通常不建议使用“默认程序包”，应避免使用。这可能会导致使用了Spring启动应用程序的特殊问题`@ComponentScan`，`@ConfigurationPropertiesScan`，`@EntityScan`，或`@SpringBootApplication`注解，因为从每一个罐子每一个类被读取。

|      | 我们建议您遵循Java建议的程序包命名约定，并使用反向域名（例如`com.example.project`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2。找到主要的应用程序类别

我们通常建议您将主应用程序类放在其他类之上的根包中。该[`@SpringBootApplication`注解](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-using-springbootapplication-annotation)往往放在你的主类，它隐含地定义为某些项目一基地“搜索包”。例如，如果您正在编写JPA应用程序，`@SpringBootApplication`则使用带注释的类的包搜索`@Entity`项目。使用根软件包还允许组件扫描仅应用于您的项目。

|      | 如果您不想使用`@SpringBootApplication`，则导入的`@EnableAutoConfiguration`和`@ComponentScan`注释会定义该行为，因此您也可以使用它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下清单显示了典型的布局：

```
com 
 +-示例
     +-myapplication 
         + -Application.java 
         | 
         +-客户
         | +-Customer.java 
         | +-CustomerController.java 
         | +-CustomerService.java 
         | +-CustomerRepository.java 
         | 
         +-订单
             +-Order.java 
             +-OrderController.java 
             + -OrderService.java + 
             -OrderRepository.java
```

该`Application.java`文件将声明`main`方法以及basic方法，`@SpringBootApplication`如下所示：

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

## 3.配置类

Spring Boot支持基于Java的配置。尽管可以`SpringApplication`与XML源一起使用，但是我们通常建议您的主要源为单个`@Configuration`类。通常，定义`main`方法的类是主要的候选者`@Configuration`。

|      | Internet上已经发布了许多使用XML配置的Spring配置示例。如果可能，请始终尝试使用等效的基于Java的配置。搜索`Enable*`注释可以是一个很好的起点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.1。导入其他配置类

您无需将所有内容都`@Configuration`放在一个类中。所述`@Import`注释可以用于导入额外的配置类。另外，您可以`@ComponentScan`用来自动拾取所有Spring组件，包括`@Configuration`类。

### 3.2。导入XML配置

如果绝对必须使用基于XML的配置，我们建议您仍然从一个`@Configuration`类开始。然后，您可以使用`@ImportResource`批注来加载XML配置文件。

## 4.自动配置

Spring Boot自动配置会尝试根据添加的jar依赖关系自动配置Spring应用程序。例如，如果`HSQLDB`位于类路径上，并且尚未手动配置任何数据库连接bean，那么Spring Boot会自动配置内存数据库。

您需要通过将`@EnableAutoConfiguration`或`@SpringBootApplication`注释添加到一个`@Configuration`类中来选择自动配置。

|      | 您只能添加一个`@SpringBootApplication`或`@EnableAutoConfiguration`注释。我们通常建议您仅将一个或另一个添加到您的主要`@Configuration`班级。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.1。逐渐取代自动配置

自动配置是非侵入性的。在任何时候，您都可以开始定义自己的配置，以替换自动配置的特定部分。例如，如果您添加自己的`DataSource`bean，则默认的嵌入式数据库支持将退出。

如果您需要找出当前正在应用的自动配置以及原因，请使用`--debug`开关启动应用程序。这样做可以启用调试日志，以选择核心记录器，并将条件报告记录到控制台。

### 4.2。禁用特定的自动配置类

如果发现正在应用不需要的特定自动配置类，则可以使用exclude属性`@SpringBootApplication`来禁用它们，如以下示例所示：

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;

@SpringBootApplication(exclude={DataSourceAutoConfiguration.class})
public class MyApplication {
}
```

如果该类不在类路径中，则可以使用`excludeName`注释的属性，并指定完全限定的名称。如果您喜欢使用`@EnableAutoConfiguration`而不是`@SpringBootApplication`，`exclude`并且`excludeName`也可以使用。最后，您还可以使用`spring.autoconfigure.exclude`属性来控制要排除的自动配置类的列表。

|      | 您可以在注释级别和使用属性来定义排除项。 |
| ---- | ---------------------------------------- |
|      |                                          |

|      | 即使有自动配置类`public`，该类的唯一方面也被认为是公共API，该类的名称就是可用于禁用自动配置的类的名称。这些类的实际内容（例如嵌套配置类或Bean方法）仅供内部使用，我们不建议直接使用它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 5.Spring Beans和依赖注入

您可以自由使用任何标准的Spring Framework技术来定义bean及其注入的依赖项。为简单起见，我们经常发现使用`@ComponentScan`（查找bean）和使用`@Autowired`（进行构造函数注入）效果很好。

如果按照上面的建议构造代码（将应用程序类放在根包中），则可以添加`@ComponentScan`而无需任何参数。您的所有应用程序组件（的`@Component`，`@Service`，`@Repository`，`@Controller`等）自动注册为春豆。

以下示例显示了一个`@Service`使用构造函数注入来获取所需`RiskAssessor`Bean的Bean：

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

如果bean具有一个构造函数，则可以省略`@Autowired`，如以下示例所示：

```java
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

|      | 请注意，使用构造函数注入如何将`riskAssessor`字段标记为`final`，指示该字段随后不能更改。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 6.使用@SpringBootApplication注释

许多Spring Boot开发人员喜欢他们的应用程序使用自动配置，组件扫描，并能够在其“应用程序类”上定义额外的配置。单个`@SpringBootApplication`注释可用于启用这三个功能，即：

- `@EnableAutoConfiguration`：启用[Spring Boot的自动配置机制](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-auto-configuration)
- `@ComponentScan`：启用`@Component`对应用程序所在的软件包的扫描（请参阅[最佳实践](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code)）
- `@Configuration`：允许在上下文中注册额外的bean或导入其他配置类

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

|      | `@SpringBootApplication`还提供了别名定制的属性`@EnableAutoConfiguration`和`@ComponentScan`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 这些功能都不是强制性的，您可以选择用它启用的任何功能替换此单个注释。例如，您可能不想在应用程序中使用组件扫描或配置属性扫描：`package com.example.myapplication; import org.springframework.boot.SpringApplication; import org.springframework.context.annotation.ComponentScan import org.springframework.context.annotation.Configuration; import org.springframework.context.annotation.Import; @Configuration(proxyBeanMethods = false) @EnableAutoConfiguration @Import({ MyConfig.class, MyAnotherConfig.class }) public class Application {     public static void main(String[] args) {            SpringApplication.run(Application.class, args);    } }`在此示例中，`Application`与其他任何Spring Boot应用程序一样，除了不会自动检测`@Component`-annotated类和`@ConfigurationProperties`-annotated类，并且显式导入用户定义的Bean（请参阅参考资料`@Import`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 7.运行您的应用程序

将应用程序打包为jar并使用嵌入式HTTP服务器的最大优势之一是，您可以像运行其他应用程序一样运行应用程序。调试Spring Boot应用程序也很容易。您不需要任何特殊的IDE插件或扩展。

|      | 本节仅介绍基于罐子的包装。如果选择将应用程序打包为war文件，则应参考服务器和IDE文档。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 7.1。从IDE运行

您可以将IDE中的Spring Boot应用程序作为简单的Java应用程序运行。但是，您首先需要导入您的项目。导入步骤因您的IDE和构建系统而异。大多数IDE可以直接导入Maven项目。例如，Eclipse用户可以从菜单中选择`Import…`→ 。`Existing Maven Projects``File`

如果您不能直接将项目导入IDE，则可以使用构建插件生成IDE元数据。Maven包括[Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/)和[IDEA的](https://maven.apache.org/plugins/maven-idea-plugin/)插件。Gradle提供了用于[各种IDE的](https://docs.gradle.org/current/userguide/userguide.html)插件。

|      | 如果不小心两次运行Web应用程序，则会看到“端口已在使用中”错误。STS用户可以使用`Relaunch`按钮而不是`Run`按钮来确保关闭任何现有实例。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 7.2。作为打包的应用程序运行

如果您使用Spring Boot Maven或Gradle插件来创建可执行jar，则可以使用来运行您的应用程序`java -jar`，如以下示例所示：

```
$ java -jar target / myapplication-0.0.1-SNAPSHOT.jar
```

也可以在启用了远程调试支持的情况下运行打包的应用程序。这样可以将调试器附加到打包的应用程序，如以下示例所示：

```
$ java -Xdebug -Xrunjdwp：server = y，transport = dt_socket，address = 8000，suspend = n \ 
       -jar target / myapplication-0.0.1-SNAPSHOT.jar
```

### 7.3。使用Maven插件

Spring Boot Maven插件包含一个`run`目标，可用于快速编译和运行您的应用程序。应用程序以爆炸形式运行，就像在IDE中一样。以下示例显示了运行Spring Boot应用程序的典型Maven命令：

```
$ mvn spring-boot：运行
```

您可能还想使用`MAVEN_OPTS`操作系统环境变量，如以下示例所示：

```
$ export MAVEN_OPTS = -Xmx1024m
```

### 7.4。使用Gradle插件

Spring Boot Gradle插件还包含一个`bootRun`任务，可用于以爆炸形式运行您的应用程序。`bootRun`每当您应用`org.springframework.boot`和`java`插件时，都会添加该任务，并在以下示例中显示：

```
$ gradle bootRun
```

您可能还想使用`JAVA_OPTS`操作系统环境变量，如以下示例所示：

```
$ export JAVA_OPTS = -Xmx1024m
```

### 7.5。热交换

由于Spring Boot应用程序只是普通的Java应用程序，因此JVM热交换应该可以立即使用。JVM热插拔在一定程度上受到它可以替换的字节码的限制。对于更完整的解决方案，可以使用[JRebel](https://www.jrebel.com/products/jrebel)。

该`spring-boot-devtools`模块还包括对应用程序快速重启的支持。有关详细信息，请参见本章稍后的“ [开发人员工具”](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools)部分和[热插拔“操作方法”](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-hotswapping)。

## 8.开发人员工具

Spring Boot包括一组额外的工具，这些工具可以使应用程序开发体验更加愉快。该`spring-boot-devtools`模块可以包含在任何项目中，以提供其他开发时功能。要包括devtools支持，请将模块依赖项添加到您的构建中，如以下Maven和Gradle清单所示：

马文

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

摇篮

```groovy
configurations {
    developmentOnly
    runtimeClasspath {
        extendsFrom developmentOnly
    }
}
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

|      | 运行完全打包的应用程序时，将自动禁用开发人员工具。如果您的应用程序是`java -jar`从某个特殊的类加载器启动的，或者从特殊的类加载器启动的，则将其视为“生产应用程序”。如果那对您不适用（即，如果您从容器中运行应用程序），请考虑排除devtools或设置`-Dspring.devtools.restart.enabled=false`系统属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 在Maven中将依赖项标记为可选，或`developmentOnly`在Gradle中使用自定义配置（如上所示）是一种最佳做法，可防止将devtools过渡应用到使用您项目的其他模块。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 重新打包的存档默认情况下不包含devtools。如果要使用[某个远程devtools功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-remote)，则需要禁用`excludeDevtools`build属性以包含它。Maven和Gradle插件均支持该属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.1。属性默认值

Spring Boot支持的一些库使用缓存来提高性能。例如，[模板引擎](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-template-engines)缓存已编译的模板，以避免重复解析模板文件。而且，Spring MVC可以在提供静态资源时向响应添加HTTP缓存标头。

尽管缓存在生产中非常有益，但在开发过程中可能适得其反，从而使您无法看到自己刚刚在应用程序中所做的更改。因此，默认情况下，spring-boot-devtools禁用缓存选项。

缓存选项通常由`application.properties`文件中的设置配置。例如，Thymeleaf提供了该`spring.thymeleaf.cache`属性。该`spring-boot-devtools`模块无需自动设置这些属性，而是自动应用合理的开发时配置。

因为在开发Spring MVC和Spring WebFlux应用程序时需要有关Web请求的更多信息，所以开发人员工具将启用`DEBUG`日志`web`记录组的日志记录。这将为您提供有关传入请求，正在处理的处理程序，响应结果等的信息。如果您希望记录所有请求的详细信息（包括潜在的敏感信息），则可以打开`spring.http.log-request-details`配置属性。

|      | 如果你不想被应用属性默认可以设置`spring.devtools.add-properties`到`false`你`application.properties`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 有关devtools应用的属性的完整列表，请参见[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.2。自动重启

`spring-boot-devtools`只要类路径上的文件发生更改，使用的应用程序就会自动重新启动。在IDE中工作时，这可能是一个有用的功能，因为它为代码更改提供了非常快速的反馈循环。默认情况下，将监视类路径上指向文件夹的任何条目的更改。请注意，某些资源（例如静态资产和视图模板）[不需要重新启动应用程序](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart-exclude)。

触发重启

当DevTools监视类路径资源时，触发重启的唯一方法是更新类路径。导致类路径更新的方式取决于您使用的IDE。在Eclipse中，保存修改后的文件将导致类路径被更新并触发重新启动。在IntelliJ IDEA中，构建项目（`Build +→+ Build Project`）具有相同的效果。

|      | 只要启用了分叉，您还可以使用受支持的构建插件（Maven和Gradle）来启动应用程序，因为DevTools需要隔离的应用程序类加载器才能正常运行。默认情况下，Gradle和Maven插件会分叉应用程序进程。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 与LiveReload一起使用时，自动重启非常有效。 [有关](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)详细信息，[请参见LiveReload部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)。如果使用JRebel，则禁用自动重新启动，而支持动态类重新加载。其他devtools功能（例如LiveReload和属性替代）仍可以使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | DevTools依赖于应用程序上下文的关闭挂钩在重新启动期间将其关闭。如果您禁用了关机挂钩（`SpringApplication.setRegisterShutdownHook(false)`），它将无法正常工作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 当决定是否在类路径中的条目应该触发重新启动时，它的变化，DevTools自动忽略命名的项目`spring-boot`，`spring-boot-devtools`，`spring-boot-autoconfigure`，`spring-boot-actuator`，和`spring-boot-starter`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | DevTools需求来定制`ResourceLoader`使用的`ApplicationContext`。如果您的应用程序已经提供了，它将被包装。不支持直接覆盖`getResource`方法`ApplicationContext`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

重新启动与重新加载

Spring Boot提供的重启技术通过使用两个类加载器来工作。不变的类（例如，来自第三方jar的类）将被加载到*基*类加载器中。您正在积极开发的类将加载到*重新启动*类加载器中。重新启动应用程序时，将丢弃*重新启动*类加载器，并创建一个新的类加载器。这种方法意味着应用程序的重启通常比“冷启动”要快得多，因为*基本*类加载器已经可用并已填充。

如果发现重新启动对于您的应用程序来说不够快，或者遇到类加载问题，则可以考虑从ZeroTurnaround 重新加载技术，例如[JRebel](https://jrebel.com/software/jrebel/)。这些方法通过在加载类时重写类来使它们更适合于重新加载。

#### 8.2.1。记录条件评估中的更改

默认情况下，每次应用程序重新启动时，都会记录一个报告，其中显示了条件评估增量。该报告显示了在进行诸如添加或删除bean以及设置配置属性之类的更改时对应用程序自动配置的更改。

要禁用报告的日志记录，请设置以下属性：

```
spring.devtools.restart.log-condition-evaluation-delta = false
```

#### 8.2.2。排除资源

某些资源在更改时不一定需要触发重新启动。例如，Thymeleaf模板可以就地编辑。默认情况下，改变资源`/META-INF/maven`，`/META-INF/resources`，`/resources`，`/static`，`/public`，或`/templates`不触发重新启动，但确会触发[现场重装](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)。如果要自定义这些排除项，则可以使用该`spring.devtools.restart.exclude`属性。例如，仅排除`/static`，`/public`您将设置以下属性：

```
spring.devtools.restart.exclude =静态/ **，公共/ **
```

|      | 如果要保留这些默认值并*添加*其他排除项，请改用该`spring.devtools.restart.additional-exclude`属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.2.3。观看其他路径

当您对不在类路径上的文件进行更改时，您可能希望重新启动或重新加载应用程序。为此，请使用该`spring.devtools.restart.additional-paths`属性来配置其他路径以监视更改。您可以使用[前面描述](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart-exclude)的`spring.devtools.restart.exclude`属性来控制其他路径下的更改是触发完全重新启动还是[实时重新加载](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-livereload)。

#### 8.2.4。禁用重启

如果您不想使用重新启动功能，则可以使用该`spring.devtools.restart.enabled`属性将其禁用。在大多数情况下，您可以在自己的设备中设置此属性`application.properties`（这样做仍会初始化重新启动类加载器，但它不会监视文件更改）。

如果您需要*完全*禁用重启支持（例如，因为它不适用于特定的库），则需要在调用之前将`spring.devtools.restart.enabled` `System`属性设置为，如以下示例所示：`false``SpringApplication.run(…)`

```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

#### 8.2.5。使用触发文件

如果使用持续编译更改文件的IDE，则可能更喜欢仅在特定时间触发重新启动。为此，您可以使用“触发文件”，这是一个特殊文件，当您要实际触发重新启动检查时必须对其进行修改。

|      | 对文件的任何更新都会触发检查，但是只有在Devtools检测到有事情要做的情况下，重启才会真正发生。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要使用触发文件，请将`spring.devtools.restart.trigger-file`属性设置为触发文件的名称（不包括任何路径）。触发器文件必须出现在类路径上的某个位置。

例如，如果您的项目具有以下结构：

```
src 
+-主
   +-资源
      +-.reloadtrigger
```

那么您的`trigger-file`财产将是：

```properties
spring.devtools.restart.trigger-file=.reloadtrigger
```

现在，仅在`src/main/resources/.reloadtrigger`更新时才会重新启动。

|      | 您可能希望将其设置`spring.devtools.restart.trigger-file`为[全局设置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-globalsettings)，以便所有项目都以相同的方式运行。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

某些IDE具有使您不必手动更新触发器文件的功能。 [Eclipse的Spring工具](https://spring.io/tools)和[IntelliJ IDEA（最终版）](https://www.jetbrains.com/idea/)都具有这种支持。使用Spring Tools，您可以从控制台视图中使用“重新加载”按钮（只要您`trigger-file`名为`.reloadtrigger`）。对于IntelliJ，您可以按照[其文档中](https://www.jetbrains.com/help/idea/spring-boot.html#configure-application-update-policies-with-devtools)的[说明进行操作](https://www.jetbrains.com/help/idea/spring-boot.html#configure-application-update-policies-with-devtools)。

#### 8.2.6。自定义重启类加载器

如前面的“ [重新启动与重新加载”](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-spring-boot-restart-vs-reload)部分所述，重新启动功能是通过使用两个类加载器实现的。对于大多数应用程序，此方法效果很好。但是，有时可能会导致类加载问题。

默认情况下，IDE中任何打开的项目都使用“重新启动”类加载器加载，而任何常规`.jar`文件都使用“基本”类加载器加载。如果您在多模块项目上工作，并且并非每个模块都导入到IDE中，则可能需要自定义内容。为此，您可以创建一个`META-INF/spring-devtools.properties`文件。

该`spring-devtools.properties`文件可以包含以`restart.exclude`和开头的属性`restart.include`。该`include`元素是应该被拉高到“重启”的类加载器的项目，以及`exclude`要素是应该向下推入“基地”类加载器的项目。该属性的值是一个应用于类路径的正则表达式模式，如以下示例所示：

```properties
restart.exclude.companycommonlibs=/mycorp-common-[\\w\\d-\.]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w\\d-\.]+\.jar
```

|      | 所有属性键都必须是唯一的。只要一个属性以`restart.include.`或开头`restart.exclude.`就可以考虑。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 所有`META-INF/spring-devtools.properties`从类路径加载。您可以将文件打包到项目内部或项目使用的库中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.2.7。已知局限性

重新启动功能不适用于通过使用标准反序列化的对象`ObjectInputStream`。如果你需要反序列化的数据，你可能需要使用Spring的`ConfigurableObjectInputStream`结合`Thread.currentThread().getContextClassLoader()`。

不幸的是，一些第三方库在不考虑上下文类加载器的情况下反序列化。如果发现这样的问题，则需要向原始作者请求修复。

### 8.3。LiveReload

该`spring-boot-devtools`模块包括一个嵌入式LiveReload服务器，该服务器可用于在更改资源时触发浏览器刷新。可从[livereload.com](http://livereload.com/extensions/)免费获得适用于Chrome，Firefox和Safari的LiveReload浏览器扩展。

如果您不想在应用程序运行时启动LiveReload服务器，则可以将`spring.devtools.livereload.enabled`属性设置为`false`。

|      | 一次只能运行一台LiveReload服务器。在启动应用程序之前，请确保没有其他LiveReload服务器正在运行。如果从IDE启动多个应用程序，则只有第一个具有LiveReload支持。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.4。全局设置

您可以通过将以下任何文件添加到`$HOME/.config/spring-boot`文件夹来配置全局devtools设置：

1. `spring-boot-devtools.properties`
2. `spring-boot-devtools.yaml`
3. `spring-boot-devtools.yml`

添加到这些文件的任何属性都将应用于使用devtools的计算机上的*所有* Spring Boot应用程序。例如，要将重启配置为始终使用[触发文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart-triggerfile)，您可以添加以下属性：

〜/ .config / spring-boot / spring-boot-devtools.properties

```properties
spring.devtools.restart.trigger-file=.reloadtrigger
```

|      | 如果在中找不到devtools配置文件`$HOME/.config/spring-boot`，则在`$HOME`文件夹的根目录中搜索是否存在`.spring-boot-devtools.properties`文件。这使您可以与不支持该`$HOME/.config/spring-boot`位置的旧版Spring Boot上的应用程序共享devtools全局配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 在上述文件中激活的配置文件不会影响[特定于配置文件的配置文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-profile-specific-properties)的加载。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 8.5。远程应用

Spring Boot开发人员工具不仅限于本地开发。远程运行应用程序时，您还可以使用多种功能。选择启用远程支持，因为启用它可能会带来安全风险。仅当在受信任的网络上运行或使用SSL保护时，才应启用它。如果这两个选项都不可用，则不应使用DevTools的远程支持。您永远不要在生产部署上启用支持。

要启用它，您需要确保已将`devtools`其包含在重新打包的存档中，如以下清单所示：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <excludeDevtools>false</excludeDevtools>
            </configuration>
        </plugin>
    </plugins>
</build>
```

然后，您需要设置`spring.devtools.remote.secret`属性。像任何重要的密码或机密一样，该值应唯一且强壮，以免被猜测或强行使用。

远程devtools支持分为两个部分：接受连接的服务器端端点和在IDE中运行的客户端应用程序。`spring.devtools.remote.secret`设置属性后，将自动启用服务器组件。客户端组件必须手动启动。

#### 8.5.1。运行远程客户端应用程序

远程客户端应用程序旨在在您的IDE中运行。您需要`org.springframework.boot.devtools.RemoteSpringApplication`使用与所连接的远程项目相同的类路径来运行。应用程序的唯一必需参数是它连接到的远程URL。

例如，如果您使用的是Eclipse或STS，并且有一个`my-app`已部署到Cloud Foundry的项目，则应执行以下操作：

- 选择`Run Configurations…`从`Run`菜单。
- 创建一个新的`Java Application`“启动配置”。
- 浏览该`my-app`项目。
- 使用`org.springframework.boot.devtools.RemoteSpringApplication`作为主类。
- 添加`https://myapp.cfapps.io`到`Program arguments`（或任何您的远程URL）。

正在运行的远程客户端可能类似于以下清单：

```
  。____ _ __ _ _ 
 / \\ / ___'_ __ _ _（_）_ __ __ _ ___ _ \ \ \ \ 
（（）\ ___ |'_ |'_ | |'_ \ / _` | | _ \ ___ _ __ ___ | | _ ___ \ \ \ \ 
 \\ / ___）| | _）| | | | | || （_ | [] :::::: [] / -_）'\ / _ \ _ / -_））））））
  '| ____ | .__ | _ | | _ | _ | | _ \ __，| | _ | _ \ ___ | _ | _ | _____ / \ __ \ ___ | / / / = 
 ========= __ | ============= | ___ / ================================= /////// 
 :: Spring Boot Remote :: 2.2.6。发布

2015-06-10 18：25：06.632信息14938 --- [main] osbdevtools.RemoteSpringApplication：使用PID 14938在pwmbp上启动RemoteSpringApplication（/ Users / pwebb / projects / spring-boot / code / spring-boot-project / spring -boot-devtools / target /由pwebb在/ Users / pwebb / projects / spring-boot / code中启动的类）
2015-06-10 18：25：06.671 INFO 14938 --- [main] scaAnnotationConfigApplicationContext：刷新org.spring 框架.context.annotation.AnnotationConfigApplicationContext @ 2a17b7b6：启动日期[PDT 2015年6月10日星期三18:25:06]；上下文层次结构的根
2015-06-10 18：25：07.043 WARN 14938 --- [main] osbdrcRemoteClientConfiguration：与http：// localhost：8080的连接不安全。您应使用以“ https：//”开头的URL。
2015-06-10 18：25：07.074 INFO 14938 --- [main] osbdaOptionalLiveReloadServer：LiveReload服务器在端口35729上运行
2015-06-10 18：25：07.130 INFO 14938 --- [main] osbdevtools.RemoteSpringApplication：已启动RemoteSpringApplication在0.74秒内运行（JVM运行1.105）
```

|      | 因为远程客户端使用与真实应用程序相同的类路径，所以它可以直接读取应用程序属性。这就是`spring.devtools.remote.secret`读取属性并将其传递给服务器进行身份验证的方式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 始终建议将其`https://`用作连接协议，以便对通信进行加密并且不能截获密码。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您需要使用代理来访问远程应用程序，请配置`spring.devtools.remote.proxy.host`和`spring.devtools.remote.proxy.port`属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.5.2。远程更新

远程客户端以与[本地重新启动](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart)相同的方式监视应用程序类路径中的更改。任何更新的资源都会推送到远程应用程序，并且（*如果需要*）会触发重新启动。如果您迭代使用本地没有的云服务的功能，这将很有帮助。通常，远程更新和重新启动比完整的重建和部署周期要快得多。

|      | 仅在远程客户端正在运行时监视文件。如果在启动远程客户端之前更改文件，则不会将其推送到远程服务器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.5.3。配置文件系统观察器

[FileSystemWatcher的](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/filewatch/FileSystemWatcher.java)工作方式是按一定时间间隔轮询类更改，然后等待预定义的静默期以确保没有更多更改。然后将更改上传到远程应用程序。在较慢的开发环境中，可能会发生静默期不够的情况，并且类中的更改可能会分为几批。第一批类更改上传后，服务器将重新启动。由于服务器正在重新启动，因此下一批不能发送到应用程序。

这通常通过`RemoteSpringApplication`日志中关于无法上载某些类的警告以及随后的重试来表明。但是，这也可能导致应用程序代码不一致，并且在上传第一批更改后无法重新启动。

如果您经常观察到此类问题，请尝试将`spring.devtools.restart.poll-interval`和`spring.devtools.restart.quiet-period`参数增加到适合您的开发环境的值：

```properties
spring.devtools.restart.poll-interval=2s
spring.devtools.restart.quiet-period=1s
```

现在每2秒轮询一次受监视的classpath文件夹是否有更改，并保持1秒钟的静默时间以确保没有其他类更改。

## 9.打包您的生产申请

可执行jar可以用于生产部署。由于它们是独立的，因此它们也非常适合基于云的部署。

对于其他“生产准备就绪”功能，例如运行状况，审核和度量REST或JMX端点，请考虑添加`spring-boot-actuator`。有关详细信息，请参见*[production-ready-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready)*。

## 10.下一步阅读

现在，您应该了解如何使用Spring Boot以及应遵循的一些最佳实践。现在，您可以继续深入了解特定的*[Spring Boot功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features)*，或者可以跳过并阅读有关Spring Boot 的“ [生产就绪](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready) ”方面的信息。

最后更新时间2020-03-26 11:43:46 UTC



# Spring Boot功能

[返回索引](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/index.html)

1. \1. SpringApplication
   1. 
   2. 
   3. 
   4. 
   5. 
   6. 
   7. 
   8. 
   9. 
   10. 
   11. 
2. 2.外部化配置
   1. 
   2. 
   3. 
   4. 
   5. 
   6. 
   7. 
      1. 
      2. 
      3. 
      4. 
   8. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
      7. 
      8. 
         1. 
         2. 
      9. 
      10. 
3. 3.个人资料
   1. 
   2. 
   3. 
4. 4.记录
   1. 
   2. 
      1. 
   3. 
   4. 
   5. 
   6. 
   7. 
      1. 
      2. 
5. [5.国际化](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-internationalization)
6. \6. JSON
   1. 
   2. 
   3. 
7. 7.开发Web应用程序
   1. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
      7. 
      8. 
      9. 
      10. 
      11. 
          1. 
          2. 
      12. 
      13. 
   2. 
      1. 
      2. 
      3. 
      4. 
      5. 
         1. 
      6. 
   3. 
   4. 
      1. 
         1. 
      2. 
         1. 
      3. 
      4. 
         1. 
         2. 
      5. 
   5. 
   6. 
8. \8. RSocket
   1. 
   2. 
   3. 
   4. 
9. 9.安全性
   1. 
   2. 
   3. 
      1. 
         1. 
      2. 
      3. 
   4. 
      1. 
   5. 
      1. 
10. 10.使用SQL数据库
    1. 
       1. 
       2. 
       3. 
    2. 
    3. 
       1. 
       2. 
       3. 
       4. 
    4. 
    5. 
       1. 
    6. 
       1. 
       2. 
       3. 
       4. 
11. 11.使用NoSQL技术
    1. 
       1. 
    2. 
       1. 
       2. 
       3. 
       4. 
    3. 
       1. 
       2. 
       3. 
       4. 
       5. 
    4. 
       1. 
       2. 
    5. 
       1. 
       2. 
       3. 
       4. 
       5. 
    6. 
       1. 
       2. 
    7. 
       1. 
       2. 
    8. 
       1. 
       2. 
       3. 
    9. 
       1. 
12. 12.缓存
    1. 
       1. 
       2. 
       3. 
       4. 
       5. 
       6. 
       7. 
       8. 
       9. 
       10. 
13. 13.消息传递
    1. 
       1. 
       2. 
       3. 
       4. 
       5. 
    2. 
       1. 
       2. 
       3. 
    3. 
       1. 
       2. 
       3. 
       4. 
       5. 
14. 14.使用RestTemplate调用REST服务
    1. 
15. 15.使用WebClient调用REST服务
    1. 
    2. 
16. [16.验证](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-validation)
17. [17.发送电子邮件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-email)
18. \18. JTA的分布式事务
    1. 
    2. 
    3. 
    4. 
    5. 
19. [19. Hazelcast](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-hazelcast)
20. [20. Quartz Scheduler](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-quartz)
21. [21.任务执行和计划](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-task-execution-scheduling)
22. [22. Spring集成](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-integration)
23. [23.春季会议](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-session)
24. [24.通过JMX进行监视和管理](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-jmx)
25. 25.测试
    1. 
    2. 
    3. 
       1. 
       2. 
       3. 
       4. 
       5. 
       6. 
       7. 
       8. 
       9. 
       10. 
       11. 
       12. 
       13. 
       14. 
       15. 
       16. 
       17. 
       18. 
       19. 
       20. 
       21. 
       22. 
       23. 
           1. 
           2. 
           3. 
       24. 
       25. 
       26. 
    4. 
       1. 
       2. 
       3. 
       4. 
26. [26. WebSockets](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-websockets)
27. 27.网络服务
    1. 
28. 28.创建自己的自动配置
    1. 
    2. 
    3. 
       1. 
       2. 
       3. 
       4. 
       5. 
       6. 
    4. 
       1. 
       2. 
    5. 
       1. 
       2. 
       3. 
       4. 
29. \29. Kotlin支持
    1. [29.1。要求](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kotlin-requirements)
    2. [29.2。零安全](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kotlin-null-safety)
    3. 29.3。Kotlin API
       1. 
       2. 
    4. [29.4。依赖管理](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kotlin-dependency-management)
    5. [29.5。@ConfigurationProperties](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kotlin-configuration-properties)
    6. [29.6。测试中](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kotlin-testing)
    7. 29.7。资源资源
       1. [29.7.1。进一步阅读](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kotlin-resources-further-reading)
       2. [29.7.2。例子](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kotlin-resources-examples)
30. [30.下一步阅读](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-whats-next)

本节将深入介绍Spring Boot。在这里，您可以了解可能要使用和定制的关键功能。如果您尚未这样做，则可能需要阅读“ [Getting ](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot)[-started.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started) ”和“ [using-spring-boot.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot) ”部分，以便您有足够的基础知识。

## 1. SpringApplication

本`SpringApplication`类提供了一个方便的方式来引导该从开始Spring应用程序`main()`的方法。在许多情况下，您可以委托给静态`SpringApplication.run`方法，如以下示例所示：

```java
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

当您的应用程序启动时，您应该看到类似于以下输出的内容：

```
  。____ _ __ _ _
 / \\ / ___'_ __ _ _（_）_ __ __ _ _ \ \ \ \
（（）\ ___ |'_ |'_ | |'_ \ / _` | \ \ \ \ \
 \\ / ___）| | _）| | | | | || （_ | |））））
  '| ____ | .__ | _ | | _ | _ | | _ \ __，| / / / /
 ======== | _ | ============= | ___ / = / _ / _ / _ /
 :: Spring Boot :: v2.2.6.RELEASE

2019-04-31 13：09：54.117信息56603 --- [main] osbsapp.SampleApplication：在我的计算机上使用PID 56603（/apps/myapp.jar由pwebb启动）启动SampleApplication v0.1.0
2019-04-31 13：09：54.166 INFO 56603 --- [main] ationConfigServletWebServerApplicationContext：刷新org.springframework.boot.web.ser vlet.context.AnnotationConfigServletWebServerApplicationContext @ 6e5a8246：启动日期[WDT Jul 31 00:08:16 PDT 2013]；上下文层次结构的根
2019-04-01 13：09：56.912信息41370 --- [main] .t.TomcatServletWebServerFactory：服务器初始化为端口：8080
2019-04-01 13：09：57.501信息41370 --- [main] osbsapp.SampleApplication：在2.992秒内启动SampleApplication（JVM运行3.658）
```

默认情况下，显示`INFO`日志消息，包括一些相关的启动详细信息，例如启动应用程序的用户。如果您需要除以外的其他日志级别`INFO`，则可以按照[日志级别中](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-log-levels)所述进行设置。使用主应用程序类包中的实现版本来确定应用程序版本。设置`spring.main.log-startup-info`为可以关闭启动信息记录`false`。这还将关闭应用程序活动配置文件的日志记录。

|      | 要在启动期间添加其他日志记录，可以`logStartupInfo(boolean)`在的子类中进行覆盖`SpringApplication`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.1。启动失败

如果您的应用程序无法启动，则注册`FailureAnalyzers`后将有机会提供专门的错误消息和解决该问题的具体措施。例如，如果您在port上启动Web应用程序`8080`并且该端口已在使用中，则应该看到类似于以下消息的内容：

```
***************************
申请启动失败
***************************

描述：

嵌入式Servlet容器无法启动。端口8080已被使用。

行动：

确定并停止正在端口8080上侦听的进程，或将此应用程序配置为在另一个端口上侦听。
```

|      | Spring Boot提供了许多`FailureAnalyzer`实现，您可以[添加自己的实现](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-failure-analyzer)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果没有故障分析器能够处理该异常，您仍然可以显示完整情况报告，以更好地了解出了什么问题。要做到这一点，你需要[使`debug`财产](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config)或[启用`DEBUG`日志记录](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-log-levels)的`org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener`。

例如，如果使用来运行应用程序`java -jar`，则可以`debug`按如下所示启用属性：

```
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```

### 1.2。延迟初始化

`SpringApplication`允许延迟初始化应用程序。启用惰性初始化后，将根据需要创建bean，而不是在应用程序启动期间创建bean。因此，启用延迟初始化可以减少应用程序启动所花费的时间。在Web应用程序中，启用延迟初始化将导致许多与Web相关的Bean直到收到HTTP请求后才被初始化。

延迟初始化的缺点是，它可能会延迟发现应用程序问题的时间。如果延迟配置了错误配置的Bean，则在启动期间将不再发生故障，并且只有在初始化Bean时问题才会变得明显。还必须注意确保JVM具有足够的内存来容纳所有应用程序的bean，而不仅仅是启动期间初始化的bean。由于这些原因，默认情况下不会启用延迟初始化，因此建议在启用延迟初始化之前先对JVM的堆大小进行微调。

可以使用`lazyInitialization`on方法`SpringApplicationBuilder`或`setLazyInitialization`on 方法以编程方式启用延迟初始化`SpringApplication`。或者，可以使用`spring.main.lazy-initialization`以下示例中所示的属性来启用它：

```properties
spring.main.lazy-initialization=true
```

|      | 如果要在对应用程序的其余部分使用延迟初始化时禁用某些bean的延迟初始化，则可以使用`@Lazy(false)`批注将它们的延迟属性显式设置为false 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.3。自定义横幅

可以通过将`banner.txt`文件添加到类路径或将`spring.banner.location`属性设置为此类文件的位置来更改启动时打印的横幅。如果文件的编码不是UTF-8，则可以设置`spring.banner.charset`。除了一个文本文件，你还可以添加一个`banner.gif`，`banner.jpg`或`banner.png`图像文件到类路径或设置`spring.banner.image.location`属性。图像将转换为ASCII艺术作品并打印在任何文本横幅上方。

在`banner.txt`文件内部，可以使用以下任意占位符：

| 变量                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| `${application.version}`                                     | 您的应用程序的版本号，如中所述`MANIFEST.MF`。例如，`Implementation-Version: 1.0`打印为`1.0`。 |
| `${application.formatted-version}`                           | 您的应用程序的版本号，已声明`MANIFEST.MF`并进行了格式显示（用括号括起来，并带有前缀`v`）。例如`(v1.0)`。 |
| `${spring-boot.version}`                                     | 您正在使用的Spring Boot版本。例如`2.2.6.RELEASE`。           |
| `${spring-boot.formatted-version}`                           | 您正在使用的Spring Boot版本，其格式用于显示（用括号括起来，并带有前缀`v`）。例如`(v2.2.6.RELEASE)`。 |
| `${Ansi.NAME}`（或`${AnsiColor.NAME}`，`${AnsiBackground.NAME}`，`${AnsiStyle.NAME}`） | `NAME`ANSI转义代码的名称在哪里。有关[`AnsiPropertySource`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)详细信息，请参见。 |
| `${application.title}`                                       | 您的应用程序的标题，如中所述`MANIFEST.MF`。例如`Implementation-Title: MyApp`打印为`MyApp`。 |

|      | `SpringApplication.setBanner(…)`如果要以编程方式生成横幅，则可以使用 该方法。使用该`org.springframework.boot.Banner`接口并实现您自己的`printBanner()`方法。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以使用该`spring.main.banner-mode`属性来确定横幅是否必须在`System.out`（`console`）上打印，发送到配置的记录器（`log`）或根本不制作（`off`）。

打印的横幅注册为下面的这名一个singleton bean： `springBootBanner`。

### 1.4。自定义SpringApplication

如果`SpringApplication`默认设置不符合您的喜好，则可以创建一个本地实例并对其进行自定义。例如，要关闭横幅，您可以编写：

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

|      | 传递给构造函数的参数`SpringApplication`是Spring bean的配置源。在大多数情况下，这些是对`@Configuration`类的引用，但是它们也可以是对XML配置或应扫描的程序包的引用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

也可以`SpringApplication`通过使用`application.properties`文件来配置。有关详细信息，请参见*[外部化配置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config)*。

有关配置选项的完整列表，请参见[`SpringApplication`Javadoc](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/SpringApplication.html)。

### 1.5。Fluent Builder API

如果您需要构建`ApplicationContext`层次结构（具有父/子关系的多个上下文），或者您更喜欢使用“流利的”构建器API，则可以使用`SpringApplicationBuilder`。

在`SpringApplicationBuilder`让要链接的多个方法调用，并且包括`parent`和`child`其让你创建层次结构，以显示在下面的示例性方法：

```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

|      | 创建`ApplicationContext`层次结构时有一些限制。例如，Web组件**必须**包含在子上下文中，并且`Environment`父和子上下文都使用相同的组件。有关完整的详细信息，请参见[`SpringApplicationBuilder`Javadoc](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/builder/SpringApplicationBuilder.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.6。应用程序事件和监听器

除了通常的Spring Framework事件（例如）外[`ContextRefreshedEvent`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)，`SpringApplication`还会发送一些其他应用程序事件。

|      | 实际上在`ApplicationContext`创建之前会触发一些事件，因此您不能将监听器注册为`@Bean`。您可以使用`SpringApplication.addListeners(…)`方法或`SpringApplicationBuilder.listeners(…)`方法注册它们。如果希望这些侦听器自动注册，而不管创建应用程序的方式如何，都可以将`META-INF/spring.factories`文件添加到项目中，并使用`org.springframework.context.ApplicationListener`键引用您的侦听器，如以下示例所示：`org.springframework.context.ApplicationListener = com.example.project.MyListener` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

应用程序事件在您的应用程序运行时按以下顺序发送：

1. `ApplicationStartingEvent`在运行开始时发送 ，在进行任何处理之前（侦听器和初始化程序的注册除外）发送。
2. `ApplicationEnvironmentPreparedEvent`当被发送`Environment`到中已知的上下文中使用，是在创建上下文之前。
3. `ApplicationContextInitializedEvent`在`ApplicationContext`准备好且已调用ApplicationContextInitializers之后但未加载任何bean定义之前，发送。
4. `ApplicationPreparedEvent`在刷新开始之前但在加载bean定义之后发送。
5. `ApplicationStartedEvent`上下文已被刷新后发送，在任何应用程序和命令行被调用前。
6. `ApplicationReadyEvent`任何应用程序和命令行调用后发送。它指示应用程序已准备就绪，可以处理请求。
7. 一个`ApplicationFailedEvent`在启动时异常发送。

上面的列表仅包含`SpringApplicationEvent`与绑定的`SpringApplication`。除这些以外，以下事件也在之后`ApplicationPreparedEvent`和之前发布`ApplicationStartedEvent`：

1. 刷新后`ContextRefreshedEvent`发送`ApplicationContext`
2. 准备就绪 `WebServerInitializedEvent`后发送。和分别是servlet和反应式变量。`WebServer``ServletWebServerInitializedEvent``ReactiveWebServerInitializedEvent`

|      | 您通常不需要使用应用程序事件，但是很容易知道它们的存在。在内部，Spring Boot使用事件来处理各种任务。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

应用程序事件是通过使用Spring Framework的事件发布机制发送的。此机制的一部分确保在子级上下文中发布给侦听器的事件也可以在任何祖先上下文中发布给侦听器。结果，如果您的应用程序使用`SpringApplication`实例的层次结构，则侦听器可能会收到同一类型的应用程序事件的多个实例。

为了使您的侦听器能够区分其上下文的事件和后代上下文的事件，它应请求注入其应用程序上下文，然后将注入的上下文与事件的上下文进行比较。可以通过实现来注入上下文，`ApplicationContextAware`或者，如果侦听器是bean，则可以使用注入上下文`@Autowired`。

### 1.7。网络环境

一个`SpringApplication`试图创建正确类型的`ApplicationContext`代表您。确定a的算法`WebApplicationType`非常简单：

- 如果存在Spring MVC，`AnnotationConfigServletWebServerApplicationContext`则使用
- 如果不存在Spring MVC且存在Spring WebFlux，`AnnotationConfigReactiveWebServerApplicationContext`则使用
- 否则，`AnnotationConfigApplicationContext`使用

这意味着，如果您`WebClient`在同一应用程序中使用Spring MVC和Spring WebFlux 的新版本，则默认情况下将使用Spring MVC。您可以通过调用轻松地覆盖它`setWebApplicationType(WebApplicationType)`。

也可以完全控制`ApplicationContext`调用所使用的类型`setApplicationContextClass(…)`。

|      | 在JUnit测试中`setWebApplicationType(WebApplicationType.NONE)`使用时 通常需要调用`SpringApplication`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.8。访问应用程序参数

如果您需要访问传递给的应用程序参数，则`SpringApplication.run(…)`可以注入`org.springframework.boot.ApplicationArguments`Bean。该`ApplicationArguments`接口提供对原始`String[]`参数以及已解析`option`和`non-option`参数的访问，如以下示例所示：

```java
import org.springframework.boot.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.stereotype.*;

@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }

}
```

|      | Spring Boot也`CommandLinePropertySource`向Spring 注册了一个`Environment`。这样，您还可以使用`@Value`注释注入单个应用程序参数。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.9。使用ApplicationRunner或CommandLineRunner

如果启动后需要运行一些特定的代码`SpringApplication`，则可以实现`ApplicationRunner`或`CommandLineRunner`接口。这两个接口以相同的方式工作，并提供一个单一的`run`方法，该方法在`SpringApplication.run(…)`完成之前就被调用。

所述`CommandLineRunner`接口提供访问的应用程序的参数作为一个简单的字符串数组，而`ApplicationRunner`用途`ApplicationArguments`接口前面讨论。以下示例显示了一个`CommandLineRunner`with `run`方法：

```java
import org.springframework.boot.*;
import org.springframework.stereotype.*;

@Component
public class MyBean implements CommandLineRunner {

    public void run(String... args) {
        // Do something...
    }

}
```

如果定义了多个`CommandLineRunner`或`ApplicationRunner`必须按特定顺序调用的bean，则可以额外实现`org.springframework.core.Ordered`接口或使用`org.springframework.core.annotation.Order`批注。

### 1.10。申请退出

每个都`SpringApplication`向JVM注册一个关闭钩子，以确保`ApplicationContext`退出时正常关闭。可以使用所有标准的Spring生命周期回调（例如`DisposableBean`接口或`@PreDestroy`批注）。

另外，`org.springframework.boot.ExitCodeGenerator`如果bean 希望在`SpringApplication.exit()`调用时返回特定的退出代码，则可以实现该接口。然后可以将此退出代码传递`System.exit()`为状态代码，以将其返回，如以下示例所示：

```java
@SpringBootApplication
public class ExitCodeApplication {

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> 42;
    }

    public static void main(String[] args) {
        System.exit(SpringApplication.exit(SpringApplication.run(ExitCodeApplication.class, args)));
    }

}
```

而且，该`ExitCodeGenerator`接口可以通过异常来实现。当遇到这样的异常时，Spring Boot返回由实现的`getExitCode()`方法提供的退出代码。

### 1.11。管理员功能

通过指定`spring.application.admin.enabled`属性，可以为应用程序启用与管理员相关的功能。这将[`SpringApplicationAdminMXBean`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java)在平台上公开`MBeanServer`。您可以使用此功能来远程管理Spring Boot应用程序。对于任何服务包装器实现，此功能也可能很有用。

|      | 如果您想知道应用程序在哪个HTTP端口上运行，请通过键获取属性`local.server.port`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 2.外部化配置

Spring Boot使您可以外部化配置，以便可以在不同环境中使用相同的应用程序代码。您可以使用属性文件，YAML文件，环境变量和命令行参数来外部化配置。属性值可以通过直接注射到你的bean `@Value`注释，通过Spring的访问`Environment`抽象，或者被[绑定到结构化对象](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-typesafe-configuration-properties)通过`@ConfigurationProperties`。

Spring Boot使用一个非常特殊的`PropertySource`顺序，该顺序旨在允许合理地覆盖值。按以下顺序考虑属性：

1. `$HOME/.config/spring-boot`当devtools处于活动状态时，文件夹中的[Devtools全局设置属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-globalsettings)。
2. [`@TestPropertySource`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/test/context/TestPropertySource.html) 测试中的注释。
3. `properties`测试中的属性。可[用于测试应用程序的特定部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)[`@SpringBootTest`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/test/context/SpringBootTest.html)的[测试注释](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)和[注释](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)。
4. 命令行参数。
5. 来自的属性`SPRING_APPLICATION_JSON`（嵌入在环境变量或系统属性中的嵌入式JSON）。
6. `ServletConfig` 初始化参数。
7. `ServletContext` 初始化参数。
8. 的JNDI属性`java:comp/env`。
9. Java系统属性（`System.getProperties()`）。
10. 操作系统环境变量。
11. `RandomValuePropertySource`，只有在拥有`random.*`。
12. 打包的jar（`application-{profile}.properties`和YAML变体）之外的[特定于配置文件的应用程序属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-profile-specific-properties)。
13. 打包在jar中[的特定于配置文件的应用程序属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-profile-specific-properties)（`application-{profile}.properties`和YAML变体）。
14. 打包的jar（`application.properties`和YAML变体）之外的应用程序属性。
15. 打包在jar中的应用程序属性（`application.properties`和YAML变体）。
16. [`@PropertySource`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/context/annotation/PropertySource.html)`@Configuration`类上的注释。请注意，`Environment`在刷新应用程序上下文之前，不会将此类属性源添加到中。现在配置某些属性（如`logging.*`和`spring.main.*`在刷新开始前已读取）为时已晚。
17. 默认属性（通过设置指定`SpringApplication.setDefaultProperties`）。

为了提供一个具体的示例，假设您开发了一个`@Component`使用`name`属性的，如以下示例所示：

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

在您的应用程序类路径上（例如，在jar内），您可以使用一个`application.properties`文件，该文件为提供合理的默认属性值`name`。在新环境中运行时，`application.properties`可以在jar外部提供一个文件，该文件将覆盖`name`。对于一次性测试，您可以使用特定的命令行开关（例如，`java -jar app.jar --name="Spring"`）启动。

|      | 这些`SPRING_APPLICATION_JSON`属性可以在命令行中提供环境变量。例如，您可以在UN * X shell中使用以下行：`$ SPRING_APPLICATION_JSON ='{“ acme”：{“ name”：“ test”}}}'java -jar myapp.jar`在前面的示例中，您最终`acme.name=test`在Spring中`Environment`。您还可以像`spring.application.json`在System属性中一样提供JSON ，如以下示例所示：`$ java -Dspring.application.json ='{“ name”：“ test”}'-jar myapp.jar`您还可以使用命令行参数来提供JSON，如以下示例所示：`$ java -jar myapp.jar --spring.application.json ='{“ name”：“ test”}'`您还可以将JSON作为JNDI变量提供，如下所示：`java:comp/env/spring.application.json`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.1。配置随机值

`RandomValuePropertySource`是用于注射的随机值（例如，进入机密或试验例）是有用的。它可以产生整数，longs，uuid或字符串，如以下示例所示：

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

该`random.int*`语法是`OPEN value (,max) CLOSE`其中的`OPEN,CLOSE`任何字符和`value,max`是整数。如果`max`提供，`value`则为最小值，`max`为最大值（不包括）。

### 2.2。访问命令行属性

默认情况下，`SpringApplication`将所有命令行选项参数（即以开头的参数`--`，例如`--server.port=9000`）转换为a `property`并将其添加到Spring `Environment`。如前所述，命令行属性始终优先于其他属性源。

如果不希望将命令行属性添加到中`Environment`，则可以使用禁用它们`SpringApplication.setAddCommandLineProperties(false)`。

### 2.3。应用程序属性文件

`SpringApplication`从`application.properties`以下位置的文件加载属性并将其添加到Spring中`Environment`：

1. 一个`/config`当前目录的子目录
2. 当前目录
3. 类路径`/config`包
4. 类路径根

该列表按优先级排序（在列表较高位置定义的属性会覆盖在较低位置定义的属性）。

|      | 您还可以[使用YAML（.yml）文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-yaml)来替代.properties。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您不喜欢`application.properties`配置文件名，则可以通过指定`spring.config.name`环境属性来切换到另一个文件名。您还可以使用`spring.config.location`环境属性（目录位置或文件路径的逗号分隔列表）来引用显式位置。以下示例显示如何指定其他文件名：

```
$ java -jar myproject.jar --spring.config.name = myproject
```

下面的示例演示如何指定两个位置：

```
$ java -jar myproject.jar --spring.config.location = classpath：/default.properties,classpath：/override.properties
```

|      | `spring.config.name`并且`spring.config.location`很早就用于确定必须加载哪些文件。必须将它们定义为环境属性（通常是OS环境变量，系统属性或命令行参数）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果`spring.config.location`包含目录（而不是文件），则应以目录结尾`/`（在运行时，在目录后附加从其生成`spring.config.name`之前生成的名称，包括特定于配置文件的文件名）。指定的文件`spring.config.location`按原样使用，不支持特定于配置文件的变体，并且被任何特定于配置文件的属性覆盖。

配置位置以相反的顺序搜索。默认情况下，配置的位置是`classpath:/,classpath:/config/,file:./,file:./config/`。结果搜索顺序如下：

1. `file:./config/`
2. `file:./`
3. `classpath:/config/`
4. `classpath:/`

当使用来配置自定义配置位置时`spring.config.location`，它们将替换默认位置。例如，如果`spring.config.location`使用值配置`classpath:/custom-config/,file:./custom-config/`，则搜索顺序将变为以下内容：

1. `file:./custom-config/`
2. `classpath:custom-config/`

或者，当使用来配置自定义配置位置时`spring.config.additional-location`，除默认位置外，还会使用它们。在默认位置之前搜索其他位置。例如，如果`classpath:/custom-config/,file:./custom-config/`配置了的其他位置，则搜索顺序变为以下内容：

1. `file:./custom-config/`
2. `classpath:custom-config/`
3. `file:./config/`
4. `file:./`
5. `classpath:/config/`
6. `classpath:/`

通过此搜索顺序，您可以在一个配置文件中指定默认值，然后在另一个配置文件中有选择地覆盖这些值。您可以在以下默认位置之一中为您的应用程序提供默认值`application.properties`（或您选择的其他任何基本名称`spring.config.name`）。然后，可以在运行时使用自定义位置之一中的其他文件覆盖这些默认值。

|      | 如果您使用环境变量而不是系统属性，则大多数操作系统都不允许使用句点分隔的键名，但是您可以使用下划线代替（例如，`SPRING_CONFIG_NAME`代替`spring.config.name`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您的应用程序在容器中运行，则可以使用JNDI属性（中的`java:comp/env`）或Servlet上下文初始化参数来代替环境变量或系统属性，也可以使用这些参数。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.4。特定于配置文件的属性

除`application.properties`文件外，还可以使用以下命名约定来定义特定于配置文件的属性：`application-{profile}.properties`。在`Environment`具有一组默认的配置文件（默认`[default]`）如果没有活动的简档设置中使用。换句话说，如果未显式激活任何概要文件，那么将从`application-default.properties`中加载属性。

特定于配置文件的属性是从与standard相同的位置加载的`application.properties`，特定于配置文件的文件始终会覆盖非特定文件，无论特定于配置文件的文件位于打包jar的内部还是外部。

如果指定了多个配置文件，则采用后赢策略。例如，由`spring.profiles.active`属性指定的配置文件会在通过`SpringApplication`API 配置的配置文件之后添加，因此具有优先权。

|      | 如果您在中指定了任何文件`spring.config.location`，则不会考虑这些文件的特定于配置文件的变体。`spring.config.location`如果您还想使用特定于配置文件的属性，请使用中的目录。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.5。属性中的占位符

使用中的值时，它们会`application.properties`通过现有的值进行过滤`Environment`，因此您可以参考以前定义的值（例如，从“系统”属性中）。

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

|      | 您还可以使用此技术来创建现有Spring Boot属性的“简短”变体。有关详细信息，请参见*[howto.html操作](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-use-short-command-line-arguments)*方法。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.6。加密属性

Spring Boot不提供对加密属性值的任何内置支持，但是，它确实提供了修改Spring包含的值所必需的挂钩点`Environment`。该`EnvironmentPostProcessor`界面允许您`Environment`在应用程序启动之前进行操作。有关详细信息，请参见[howto.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-customize-the-environment-or-application-context)。

如果您正在寻找一种安全的方式来存储凭据和密码，[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)项目提供了对在[HashiCorp Vault中](https://www.vaultproject.io/)存储外部化配置的支持。

### 2.7。使用YAML代替属性

[YAML](https://yaml.org/)是JSON的超集，因此是一种用于指定层次结构配置数据的便捷格式。该`SpringApplication`级自动支持YAML来替代，只要你有属性[SnakeYAML](https://bitbucket.org/asomov/snakeyaml)在classpath库。

|      | 如果您使用“入门”，则SnakeYAML将由自动提供`spring-boot-starter`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.7.1。加载YAML

Spring Framework提供了两个方便的类，可用于加载YAML文档。该`YamlPropertiesFactoryBean`负载YAML作为`Properties`和`YamlMapFactoryBean`负载YAML作为`Map`。

例如，考虑以下YAML文档：

```yaml
environments:
    dev:
        url: https://dev.example.com
        name: Developer Setup
    prod:
        url: https://another.example.com
        name: My Cool App
```

前面的示例将转换为以下属性：

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

YAML列表表示为带有`[index]`解引用器的属性键。例如，考虑以下YAML：

```yaml
my:
   servers:
       - dev.example.com
       - another.example.com
```

前面的示例将转换为以下属性：

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

要通过使用Spring Boot的`Binder`实用程序（`@ConfigurationProperties`确实如此）绑定到类似的属性，您需要在类型为`java.util.List`（或`Set`）的目标bean中具有一个属性，并且您需要提供setter或使用可变值对其进行初始化。例如，以下示例绑定到前面显示的属性：

```java
@ConfigurationProperties(prefix="my")
public class Config {

    private List<String> servers = new ArrayList<String>();

    public List<String> getServers() {
        return this.servers;
    }
}
```

#### 2.7.2。在Spring环境中将YAML公开为属性

本`YamlPropertySourceLoader`类可用于暴露YAML作为`PropertySource`在春节`Environment`。这样做使您可以将`@Value`注释和占位符语法一起使用以访问YAML属性。

#### 2.7.3。多配置文件YAML文档

您可以使用一个`spring.profiles`键来指示文档何时适用，从而在一个文件中指定多个特定于配置文件的YAML文档，如以下示例所示：

```yaml
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production & eu-central
server:
    address: 192.168.1.120
```

在前面的示例中，如果`development`配置文件处于活动状态，则`server.address`属性为`127.0.0.1`。同样，如果`production` **和** `eu-central`配置文件处于活动状态，则`server.address`属性为`192.168.1.120`。如果`development`，`production`并`eu-central`在配置文件**没有**启用，那么该属性的值`192.168.1.100`。

|      | `spring.profiles`因此可以包含一个简单的配置文件名称（例如`production`）或一个配置文件表达式。简档表达式允许例如表达更复杂的简档逻辑`production & (eu-central | eu-west)`。有关更多详细信息，请参阅[参考指南](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果在启动应用程序上下文时未显式激活任何活动，则会激活默认配置文件。因此，在以下YAML中，我们设置了一个**仅**在“默认”配置文件中`spring.security.user.password`可用的值：

```yaml
server:
  port: 8000
---
spring:
  profiles: default
  security:
    user:
      password: weak
```

而在以下示例中，始终设置密码是因为该密码未附加到任何配置文件，并且必须根据需要在所有其他配置文件中将其显式重置：

```yaml
server:
  port: 8000
spring:
  security:
    user:
      password: weak
```

使用`spring.profiles`元素指定的弹簧轮廓可以选择使用`!`字符来否定。如果为单个文档指定了否定配置文件和非否定配置文件，则至少一个非否定配置文件必须匹配，并且否定配置文件不能匹配。

#### 2.7.4。YAML的缺点

无法使用`@PropertySource`注释加载YAML文件。因此，在需要以这种方式加载值的情况下，需要使用属性文件。

在特定于配置文件的YAML文件中使用多YAML文档语法可能会导致意外行为。例如，考虑文件中的以下配置：

application-dev.yml

```yaml
server:
  port: 8000
---
spring:
  profiles: "!test"
  security:
    user:
      password: "secret"
```

如果使用参数运行应用程序，则`--spring.profiles.active=dev`可能希望`security.user.password`将其设置为“秘密”，但实际情况并非如此。

嵌套文档将被过滤，因为主文件名为`application-dev.yml`。它已经被认为是特定于配置文件的，并且嵌套文档将被忽略。

|      | 我们建议您不要混合使用特定于配置文件的YAML文件和多个YAML文档。坚持只使用其中之一。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.8。类型安全的配置属性

使用`@Value("${property}")`注释注入配置属性有时会很麻烦，尤其是当您使用多个属性或数据本质上是分层的时。Spring Boot提供了一种使用属性的替代方法，该方法使强类型的Bean可以管理和验证应用程序的配置。

|      | 另请参见[和类型安全配置属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-vs-value)[之间`@Value`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-vs-value)的[区别](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-vs-value)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.1。JavaBean属性绑定

可以绑定一个声明标准JavaBean属性的bean，如以下示例所示：

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() { ... }

    public void setEnabled(boolean enabled) { ... }

    public InetAddress getRemoteAddress() { ... }

    public void setRemoteAddress(InetAddress remoteAddress) { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() { ... }

        public void setUsername(String username) { ... }

        public String getPassword() { ... }

        public void setPassword(String password) { ... }

        public List<String> getRoles() { ... }

        public void setRoles(List<String> roles) { ... }

    }
}
```

前面的POJO定义了以下属性：

- `acme.enabled`，`false`默认值为。
- `acme.remote-address`，其类型可以从强制转换`String`。
- `acme.security.username`，其嵌套的“安全”对象的名称由属性的名称确定。特别是，返回类型根本不使用，可能已经使用过`SecurityProperties`。
- `acme.security.password`。
- `acme.security.roles`，`String`其默认值为`USER`。

|      | 映射到`@ConfigurationProperties`Spring Boot中可用类的属性是公共API，通过属性文件，YAML文件，环境变量等进行配置，这些属性是公共API，但类本身的访问器（获取器/设置器）不能直接使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 这种安排依赖于默认的空构造函数，并且getter和setter通常是强制性的，因为绑定是通过标准Java Beans属性描述符进行的，就像在Spring MVC中一样。在以下情况下，可以忽略二传手：只要将地图初始化，它们就需要使用吸气剂，但不一定需要使用setter，因为它们可以被活页夹改变。可以通过索引（通常使用YAML）或使用单个逗号分隔的值（属性）来访问集合和数组。在后一种情况下，必须使用二传手。我们建议始终为此类类型添加设置器。如果初始化集合，请确保它不是不可变的（如上例所示）。如果嵌套的POJO属性已初始化（如`Security`前面示例中的字段），则不需要setter。如果希望活页夹通过使用其默认构造函数动态创建实例，则需要一个setter。有些人使用Lombok项目自动添加获取器和设置器。确保Lombok不会为这种类型生成任何特定的构造函数，因为容器会自动使用它来实例化该对象。最后，仅考虑标准Java Bean属性，并且不支持对静态属性的绑定。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.2。构造函数绑定

上一节中的示例可以以不变的方式重写，如下例所示：

```java
package com.example;

import java.net.InetAddress;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.bind.DefaultValue;

@ConstructorBinding
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;

    public AcmeProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public boolean isEnabled() { ... }

    public InetAddress getRemoteAddress() { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private final String username;

        private final String password;

        private final List<String> roles;

        public Security(String username, String password,
                @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        public String getUsername() { ... }

        public String getPassword() { ... }

        public List<String> getRoles() { ... }

    }

}
```

在此设置中，`@ConstructorBinding`注释用于指示应使用构造函数绑定。这意味着绑定器将期望找到带有您希望绑定的参数的构造函数。

类的嵌套成员`@ConstructorBinding`（`Security`例如上面的示例）也将通过其构造函数进行绑定。

可以使用指定默认值，并且将使用`@DefaultValue`相同的转换服务将`String`值强制为缺少属性的目标类型。

|      | 要使用构造函数绑定，必须使用`@EnableConfigurationProperties`或配置属性扫描来启用该类。您无法将构造函数绑定与常规Spring机制`@Component`创建的`@Bean`bean一起使用（例如，bean，通过方法创建的bean或使用加载的bean `@Import`） |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您的类具有多个构造函数，则还可以`@ConstructorBinding`直接在应绑定的构造函数上使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.3。启用`@ConfigurationProperties`注释的类型

Spring Boot提供了绑定`@ConfigurationProperties`类型并将其注册为Bean的基础架构。您可以逐类启用配置属性，也可以启用与组件扫描类似的方式进行配置属性扫描。

有时，带注释的类`@ConfigurationProperties`可能不适合扫描，例如，如果您正在开发自己的自动配置，或者想要有条件地启用它们。在这些情况下，请使用`@EnableConfigurationProperties`注释指定要处理的类型列表。可以在任何`@Configuration`类上完成此操作，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

要使用配置属性扫描，请将`@ConfigurationPropertiesScan`注释添加到您的应用程序。通常，它将添加到带有注释的主应用程序类中，`@SpringBootApplication`但可以将其添加到任何`@Configuration`类中。默认情况下，将从声明注释的类的包中进行扫描。如果要定义要扫描的特定程序包，可以按照以下示例所示进行操作：

```java
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "org.acme.another" })
public class MyApplication {
}
```

|      | 当`@ConfigurationProperties`bean使用配置属性扫描或通过注册`@EnableConfigurationProperties`，豆具有常规名称：`-`，其中，``是在指定的环境键前缀`@ConfigurationProperties`注释和``是bean的全限定名。如果注释不提供任何前缀，则仅使用Bean的完全限定名称。上例中的Bean名称为`acme-com.example.AcmeProperties`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

我们建议`@ConfigurationProperties`仅处理环境，尤其不要从上下文中注入其他bean。对于特殊情况，可以使用setter注入或`*Aware`框架提供的任何接口（例如，`EnvironmentAware`如果需要访问`Environment`）。如果您仍然想使用构造函数注入其他bean，则必须使用注释配置属性bean `@Component`并使用基于JavaBean的属性绑定。

#### 2.8.4。使用带`@ConfigurationProperties`注释的类型

这种配置样式特别适用于`SpringApplication`外部YAML配置，如以下示例所示：

```yaml
# application.yml

acme:
    remote-address: 192.168.1.1
    security:
        username: admin
        roles:
          - USER
          - ADMIN

# additional configuration as required
```

要使用`@ConfigurationProperties`bean，可以像使用其他任何bean一样注入它们，如以下示例所示：

```java
@Service
public class MyService {

    private final AcmeProperties properties;

    @Autowired
    public MyService(AcmeProperties properties) {
        this.properties = properties;
    }

    //...

    @PostConstruct
    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        // ...
    }

}
```

|      | 使用using `@ConfigurationProperties`还可以生成可由IDE使用的元数据文件，以为您自己的键提供自动补全功能。有关详细信息，请参见[附录](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.5。第三方配置

除了`@ConfigurationProperties`用于注释类之外，您还可以在公共`@Bean`方法上使用它。当您要将属性绑定到控件之外的第三方组件时，这样做特别有用。

要从`Environment`属性中配置bean ，请添加`@ConfigurationProperties`到其bean注册中，如以下示例所示：

```java
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
    ...
}
```

用`another`前缀定义的任何JavaBean属性都`AnotherComponent`以类似于前面`AcmeProperties`示例的方式映射到该bean 。

#### 2.8.6。轻松绑定

Spring Boot使用一些宽松的规则将`Environment`属性绑定到`@ConfigurationProperties`Bean，因此`Environment`属性名称和Bean属性名称之间不需要完全匹配。有用的常见示例包括破折号分隔的环境属性（例如，`context-path`绑定到`contextPath`）和大写的环境属性（例如，`PORT`绑定到`port`）。

例如，考虑以下`@ConfigurationProperties`类：

```java
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```

使用前面的代码，可以全部使用以下属性名称：

| 属性                                | 注意                                                         |
| :---------------------------------- | :----------------------------------------------------------- |
| `acme.my-project.person.first-name` | 烤肉串盒，建议在`.properties`和`.yml`文件中使用。            |
| `acme.myProject.person.firstName`   | 标准驼峰式语法。                                             |
| `acme.my_project.person.first_name` | 下划线表示法，是在`.properties`和`.yml`文件中使用的另一种格式。 |
| `ACME_MYPROJECT_PERSON_FIRSTNAME`   | 大写格式，使用系统环境变量时建议使用。                       |

|      | `prefix`注释 的值*必须*为kebab大小写（小写并用分隔`-`，例如`acme.my-project.person`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

| 财产来源 | 简单                                                   | 清单                                                         |
| :------- | :----------------------------------------------------- | :----------------------------------------------------------- |
| 属性文件 | 骆驼案，烤肉串案或下划线                               | 使用`[ ]`或以逗号分隔的值的标准列表语法                      |
| YAML文件 | 骆驼案，烤肉串案或下划线                               | 标准YAML列表语法或逗号分隔值                                 |
| 环境变量 | 以下划线作为定界符的大写格式。 `_`不应在属性名称中使用 | 下划线括起来的数值，例如 `MY_ACME_1_OTHER = my.acme[1].other` |
| 系统属性 | 骆驼案，烤肉串案或下划线                               | 使用`[ ]`或以逗号分隔的值的标准列表语法                      |

|      | 我们建议，如果可能的话，属性以小写kebab格式存储，例如`my.property-name=acme`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

绑定到`Map`属性时，如果`key`包含除小写字母数字字符或以外的任何内容，则`-`需要使用方括号表示法，以便保留原始值。如果键没有被包围`[]`，则所有非字母数字或`-`已删除的字符。例如，考虑将以下属性绑定到`Map`：

```yaml
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```

上面的性质将结合一个`Map`具有`/key1`，`/key2`并`key3`作为映射中的键。

|      | 对于YAML文件，方括号必须用引号引起来，以便正确解析密钥。 |
| ---- | -------------------------------------------------------- |
|      |                                                          |

#### 2.8.7。合并复杂类型

如果在多个位置配置了列表，则通过替换整个列表来进行覆盖。

例如，假定默认情况下`MyPojo`具有`name`和`description`属性的对象`null`。以下示例公开了`MyPojo`来自的对象列表`AcmeProperties`：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```

考虑以下配置：

```yaml
acme:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

如果`dev`配置文件未激活，则`AcmeProperties.list`包含一个`MyPojo`条目，如先前所定义。`dev`但是，如果启用了配置文件，则`list` *仍然*仅包含一个条目（名称为`my another name`，说明为`null`）。此配置*不会*将第二个`MyPojo`实例添加到列表中，并且不会合并项目。

`List`在多个配置文件中指定a时，将使用优先级最高的一个（并且只有该优先级）。考虑以下示例：

```yaml
acme:
  list:
    - name: my name
      description: my description
    - name: another name
      description: another description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

在前面的示例中，如果`dev`配置文件处于活动状态，则`AcmeProperties.list`包含*一个* `MyPojo`条目（名称为`my another name`，描述为`null`）。对于YAML，可以使用逗号分隔的列表和YAML列表来完全覆盖列表的内容。

对于`Map`属性，可以绑定从多个来源绘制的属性值。但是，对于多个源中的同一属性，将使用优先级最高的属性。以下示例公开了一个`Map`from `AcmeProperties`：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final Map<String, MyPojo> map = new HashMap<>();

    public Map<String, MyPojo> getMap() {
        return this.map;
    }

}
```

考虑以下配置：

```yaml
acme:
  map:
    key1:
      name: my name 1
      description: my description 1
---
spring:
  profiles: dev
acme:
  map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2
```

如果`dev`概要文件未处于活动状态，则`AcmeProperties.map`包含一个带键的条目`key1`（名称为`my name 1`，描述为`my description 1`）。`dev`但是，如果启用了配置文件，则`map`包含两个带有键的项`key1`（名称为`dev name 1`和描述为`my description 1`和）和`key2`（名称为`dev name 2`和描述为`dev description 2`）。

|      | 前述合并规则不仅适用于YAML文件，而且适用于所有属性源中的属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.8。属性转换

当Spring Boot绑定到`@ConfigurationProperties`bean 时，它试图将外部应用程序属性强制为正确的类型。如果您需要自定义类型转换，则可以提供一个`ConversionService`Bean（具有一个名为的Bean `conversionService`）或一个定制属性编辑器（通过一个`CustomEditorConfigurer`Bean）或自定义`Converters`（带有定义为的Bean `@ConfigurationPropertiesBinding`）。

|      | 由于在应用程序生命周期中非常早就请求了此bean，因此请确保限制您`ConversionService`正在使用的依赖项。通常，您需要的任何依赖项在创建时可能都没有完全初始化。`ConversionService`如果配置键强制不需要自定义，则可能需要重命名自定义，而仅依赖具有限定符的自定义转换器`@ConfigurationPropertiesBinding`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 转换时间

Spring Boot为表达持续时间提供了专门的支持。如果公开`java.time.Duration`属性，则应用程序属性中的以下格式可用：

- 常规`long`表示形式（使用毫秒作为默认单位，除非`@DurationUnit`已指定a）
- 标准的ISO-8601格式[使用`java.time.Duration`](https://docs.oracle.com/javase/8/docs/api//java/time/Duration.html#parse-java.lang.CharSequence-)
- 值和单位耦合的更易读的格式（例如，`10s`表示10秒）

考虑以下示例：

```java
@ConfigurationProperties("app.system")
public class AppSystemProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public void setSessionTimeout(Duration sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }

}
```

要指定30秒的会话超时`30`，`PT30S`和`30s`都等效。500毫秒的读取超时可以以任何形式如下指定：`500`，`PT0.5S`和`500ms`。

您也可以使用任何受支持的单位。这些是：

- `ns` 纳秒
- `us` 微秒
- `ms` 毫秒
- `s` 几秒钟
- `m` 几分钟
- `h` 用了几个小时
- `d` 持续数天

默认单位是毫秒，可以使用`@DurationUnit`上面的示例中所示的方法进行覆盖。

|      | 如果您要从仅`Long`用于表示持续时间的先前版本进行升级，请确保`@DurationUnit`在切换到的时间不是毫秒时（使用）定义单位（使用）`Duration`。这样做可以提供透明的升级路径，同时支持更丰富的格式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 转换数据大小

Spring Framework的`DataSize`值类型表示字节大小。如果公开`DataSize`属性，则应用程序属性中的以下格式可用：

- 常规`long`表示形式（除非`@DataSizeUnit`已指定，否则使用字节作为默认单位）
- 值和单位耦合的更具可读性的格式（例如，`10MB`意味着10兆字节）

考虑以下示例：

```java
@ConfigurationProperties("app.io")
public class AppIoProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public void setBufferSize(DataSize bufferSize) {
        this.bufferSize = bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

    public void setSizeThreshold(DataSize sizeThreshold) {
        this.sizeThreshold = sizeThreshold;
    }

}
```

指定10 MB的缓冲区大小，`10`并且`10MB`等效。可以将256个字节的大小阈值指定为`256`或`256B`。

您也可以使用任何受支持的单位。这些是：

- `B` 对于字节
- `KB` 千字节
- `MB` 兆字节
- `GB` 千兆字节
- `TB` 太字节

默认单位是字节，可以使用`@DataSizeUnit`上面的示例中所示的方法覆盖它。

|      | 如果您要从仅`Long`用于表示大小的先前版本进行升级，请确保`@DataSizeUnit`在切换到的旁边没有字节的情况下定义单位（使用）`DataSize`。这样做可以提供透明的升级路径，同时支持更丰富的格式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.9。@ConfigurationProperties验证

`@ConfigurationProperties`每当使用Spring的`@Validated`注释对其进行注释时，Spring Boot就会尝试验证类。您可以`javax.validation`直接在配置类上使用JSR-303 约束注释。为此，请确保在类路径上有兼容的JSR-303实现，然后将约束注释添加到字段中，如以下示例所示：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

    @NotNull
    private InetAddress remoteAddress;

    // ... getters and setters

}
```

|      | 您还可以通过使用注释`@Bean`创建配置属性的方法来触发验证`@Validated`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

为确保始终为嵌套属性触发验证，即使未找到任何属性，相关字段也必须使用注释`@Valid`。下面的示例基于前面的`AcmeProperties`示例：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    // ... getters and setters

    public static class Security {

        @NotEmpty
        public String username;

        // ... getters and setters

    }

}
```

您还可以`Validator`通过创建名为的bean定义来添加自定义Spring `configurationPropertiesValidator`。该`@Bean`方法应声明`static`。配置属性验证器是在应用程序生命周期的早期创建的，并且将`@Bean`方法声明为static可以使Bean得以创建而不必实例化`@Configuration`该类。这样做避免了由早期实例化引起的任何问题。

|      | 该`spring-boot-actuator`模块包括一个公开所有`@ConfigurationProperties`bean 的端点。将您的Web浏览器指向`/actuator/configprops`或使用等效的JMX端点。有关详细信息，请参见“ [生产就绪功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints) ”部分。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.10。@ConfigurationProperties与@Value

的`@Value`注释是核心容器的功能，和它不提供相同的功能，类型安全配置属性。下表总结了`@ConfigurationProperties`和支持的功能`@Value`：

| 特征                                                         | `@ConfigurationProperties` | `@Value` |
| :----------------------------------------------------------- | :------------------------- | :------- |
| [宽松的绑定](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-relaxed-binding) | 是                         | 没有     |
| [元数据支持](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata) | 是                         | 没有     |
| `SpEL` 评价                                                  | 没有                       | 是       |

如果您为自己的组件定义了一组配置键，我们建议您将它们组合在以标记的POJO中`@ConfigurationProperties`。您还应该意识到，由于`@Value`不支持宽松的绑定，因此如果您需要通过使用环境变量来提供值，则不是一个很好的选择。

最后，尽管您可以在其中编写`SpEL`表达式`@Value`，但不会从[应用程序属性文件中](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-application-property-files)处理此类表达式。

## 3.个人资料

Spring Profiles提供了一种隔离应用程序配置部分并使之仅在某些环境中可用的方法。任何`@Component`，`@Configuration`或`@ConfigurationProperties`可与被标记`@Profile`，当它被加载到限制，因为显示在下面的例子：

```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

|      | 如果`@ConfigurationProperties`通过`@EnableConfigurationProperties`而不是自动扫描来注册bean ，则`@Profile`需要在`@Configuration`具有`@EnableConfigurationProperties`注释的类上指定注释。在`@ConfigurationProperties`被扫描的情况下，`@Profile`可以在`@ConfigurationProperties`类本身上指定。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您可以使用`spring.profiles.active` `Environment`属性来指定哪些配置文件处于活动状态。您可以通过本章前面介绍的任何方式指定属性。例如，您可以将其包括在您的中`application.properties`，如以下示例所示：

```properties
spring.profiles.active=dev,hsqldb
```

您也可以使用以下开关在命令行中指定它：`--spring.profiles.active=dev,hsqldb`。

### 3.1。添加活动配置文件

该`spring.profiles.active`属性与其他属性遵循相同的排序规则：最高`PropertySource`获胜。这意味着您可以在其中指定活动配置文件`application.properties`，然后使用命令行开关**替换**它们。

有时，将特定于配置文件的属性**添加**到活动配置文件而不是替换它们是有用的。该`spring.profiles.include`属性可用于无条件添加活动配置文件。该`SpringApplication`入口点还设置附加配置文件的Java API（即，是对那些被激活的顶级`spring.profiles.active`属性）。参见[SpringApplication中](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/SpringApplication.html)的`setAdditionalProfiles()`方法。

例如，当使用开关运行具有以下属性的应用程序时`--spring.profiles.active=prod`，`proddb`和`prodmq`配置文件也会被激活：

```yaml
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
  - proddb
  - prodmq
```

|      | 请记住，`spring.profiles`可以在YAML文档中定义该属性以确定该特定文档何时包括在配置中。有关更多详细信息，请参见[howto.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-change-configuration-depending-on-the-environment)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.2。以编程方式设置配置文件

您可以`SpringApplication.setAdditionalProfiles(…)`在应用程序运行之前通过调用来以编程方式设置活动配置文件。也可以通过使用Spring的`ConfigurableEnvironment`界面来激活配置文件。

### 3.3。特定于配置文件的配置文件

`application.properties`（或`application.yml`）和通过引用引用的文件的特定于档案的特定变体`@ConfigurationProperties`都被视为文件并已加载。有关详细信息，请参见“ [特定](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-profile-specific-properties)于[配置文件的属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-profile-specific-properties) ”。

## 4.记录

Spring Boot使用[Commons Logging](https://commons.apache.org/logging)进行所有内部日志记录，但是使底层日志实现保持打开状态。提供了[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)，[Log4J2](https://logging.apache.org/log4j/2.x/)和[Logback的](https://logback.qos.ch/)默认配置。在每种情况下，记录器都已预先配置为使用控制台输出，同时还提供可选文件输出。

默认情况下，如果使用“启动器”，则使用Logback进行日志记录。还包括适当的Logback路由，以确保使用Java Util Logging，Commons Logging，Log4J或SLF4J的从属库都能正常工作。

|      | 有许多可用于Java的日志记录框架。如果上面的列表看起来令人困惑，请不要担心。通常，您不需要更改日志记录依赖项，并且Spring Boot默认值可以正常工作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 当您将应用程序部署到servlet容器或应用程序服务器时，通过Java Util Logging API执行的日志记录不会路由到应用程序的日志中。这样可以防止由容器或已部署到容器中的其他应用程序执行的日志记录出现在应用程序的日志中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.1。日志格式

Spring Boot的默认日志输出类似于以下示例：

```
2019-03-05 10：57：51.112信息45469 --- [main] org.apache.catalina.core.StandardEngine：启动Servlet引擎：Apache Tomcat / 7.0.52
2019-03-05 10：57：51.253信息45469-[ost-startStop-1] oaccC [Tomcat]。[localhost]。[/]：初始化Spring嵌入式WebApplicationContext
2019-03-05 10：57：51.253信息45469-[ost-startStop-1] osweb.context.ContextLoader：根WebApplicationContext：初始化在1358毫秒内完成
2019-03-05 10：57：51.698信息45469-[ost-startStop-1] osbceServletRegistrationBean：映射servlet：'dispatcherServlet'到[/]
2019-03-05 10：57：51.702信息45469-[ost-startStop-1] osbcembedded.FilterRegistrationBean：映射过滤器：'hiddenHttpMethodFilter'到：[/ *]
```

输出以下项目：

- 日期和时间：毫秒精度，易于排序。
- 日志级别：`ERROR`，`WARN`，`INFO`，`DEBUG`，或`TRACE`。
- 进程ID。
- 一个`---`分离器来区分实际日志消息的开始。
- 线程名称：用方括号括起来（对于控制台输出可能会被截断）。
- 记录器名称：这通常是源类名称（通常缩写）。
- 日志消息。

|      | Logback没有`FATAL`级别。它映射到`ERROR`。 |
| ---- | ----------------------------------------- |
|      |                                           |

### 4.2。控制台输出

默认日志配置在写入消息时将消息回显到控制台。默认情况下，将记录`ERROR`-level，`WARN`-level和`INFO`-level消息。您还可以通过使用`--debug`标志启动应用程序来启用“调试”模式。

```
$ java -jar myapp.jar-调试
```

|      | 您也可以`debug=true`在中指定`application.properties`。 |
| ---- | ------------------------------------------------------ |
|      |                                                        |

启用调试模式后，将配置一些核心记录器（嵌入式容器，Hibernate和Spring Boot）以输出更多信息。启用调试模式并*没有*配置您的应用程序记录所有消息`DEBUG`的水平。

另外，您可以通过使用`--trace`标志（或`trace=true`在中`application.properties`）启动应用程序来启用“跟踪”模式。这样做可以为某些核心记录器（嵌入式容器，Hibernate模式生成以及整个Spring产品组合）启用跟踪记录。

#### 4.2.1。颜色编码输出

如果您的终端支持ANSI，则使用彩色输出来提高可读性。您可以设置`spring.output.ansi.enabled`为[支持的值](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/ansi/AnsiOutput.Enabled.html)以覆盖自动检测。

通过使用`%clr`转换字来配置颜色编码。转换器以最简单的形式根据对数级别为输出着色，如以下示例所示：

```
%clr(%5p)
```

下表描述了日志级别到颜色的映射：

| 水平    | 颜色 |
| :------ | :--- |
| `FATAL` | 红色 |
| `ERROR` | 红色 |
| `WARN`  | 黄色 |
| `INFO`  | 绿色 |
| `DEBUG` | 绿色 |
| `TRACE` | 绿色 |

另外，您可以通过将其提供为转换的选项来指定应使用的颜色或样式。例如，要使文本变黄，请使用以下设置：

```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持以下颜色和样式：

- `blue`
- `cyan`
- `faint`
- `green`
- `magenta`
- `red`
- `yellow`

### 4.3。文件输出

默认情况下，Spring Boot只记录到控制台，不写日志文件。如果除了控制台输出外还想写日志文件，则需要设置一个`logging.file.name`或`logging.file.path`属性（例如，在中`application.properties`）。

下表显示了如何`logging.*`一起使用这些属性：

| `logging.file.name` | `logging.file.path` | 例         | 描述                                                         |
| :------------------ | :------------------ | :--------- | :----------------------------------------------------------- |
| *（没有）*          | *（没有）*          |            | 仅控制台记录。                                               |
| 具体档案            | *（没有）*          | `my.log`   | 写入指定的日志文件。名称可以是确切的位置，也可以相对于当前目录。 |
| *（没有）*          | 具体目录            | `/var/log` | 写入`spring.log`指定的目录。名称可以是确切的位置，也可以相对于当前目录。 |

日志文件达到10 MB时会旋转，并且与控制台输出一样，默认情况下会记录`ERROR`-level，`WARN`-level和`INFO`-level消息。可以使用该`logging.file.max-size`属性更改大小限制。除非`logging.file.max-history`已设置该属性，否则默认情况下将保留最近7天的轮转日志文件。日志档案的总大小可以使用设置上限`logging.file.total-size-cap`。当日志归档的总大小超过该阈值时，将删除备份。要在应用程序启动时强制清除日志存档，请使用该`logging.file.clean-history-on-start`属性。

|      | 日志记录属性独立于实际的日志记录基础结构。结果，`logback.configurationFile`Spring Boot不会管理特定的配置密钥（例如Logback）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.4。日志级别

所有支持的日志系统可以在弹簧设置的记录器级别`Environment`（例如，`application.properties`通过使用）`logging.level.=`，其中`level`为TRACE，DEBUG，INFO，WARN，ERROR，FATAL或OFF之一。该`root`记录器可以通过使用被配置`logging.level.root`。

以下示例显示了中的潜在日志记录设置`application.properties`：

```properties
logging.level.root=warn
logging.level.org.springframework.web=debug
logging.level.org.hibernate=error
```

也可以使用环境变量来设置日志记录级别。例如，`LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG`将设置`org.springframework.web`为`DEBUG`。

|      | 以上方法仅适用于程序包级别的日志记录。由于轻松绑定总是将环境变量转换为小写，因此无法以这种方式为单个类配置日志记录。如果需要为类配置日志记录，则可以使用[该`SPRING_APPLICATION_JSON`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-application-json)变量。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.5。日志组

能够将相关记录器分组在一起通常很有用，以便可以同时配置它们。例如，您可能通常会更改*所有*与Tomcat相关的记录器的记录级别，但是您不容易记住顶层软件包。

为了解决这个问题，Spring Boot允许您在Spring中定义日志记录组`Environment`。例如，这是通过将“ tomcat”组添加到您的方式来定义它的方法`application.properties`：

```properties
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

定义后，您可以使用一行更改该组中所有记录器的级别：

```properties
logging.level.tomcat=TRACE
```

Spring Boot包含以下预定义的日志记录组，它们可以直接使用：

| 名称 | 记录仪                                                       |
| :--- | :----------------------------------------------------------- |
| 网络 | `org.springframework.core.codec`，`org.springframework.http`，`org.springframework.web`，`org.springframework.boot.actuate.endpoint.web`，`org.springframework.boot.web.servlet.ServletContextInitializerBeans` |
| sql  | `org.springframework.jdbc.core`，`org.hibernate.SQL`，`org.jooq.tools.LoggerListener` |

### 4.6。自定义日志配置

可以通过在类路径中包含适当的库来激活各种日志记录系统，并可以通过在类路径的根目录或以下Spring `Environment`属性指定的位置中提供适当的配置文件来进一步自定义各种日志记录系统`logging.config`。

您可以通过使用`org.springframework.boot.logging.LoggingSystem`system属性来强制Spring Boot使用特定的日志系统。该值应该是实现的完全限定的类名`LoggingSystem`。您还可以通过使用值完全禁用Spring Boot的日志记录配置`none`。

|      | 由于日志记录是**在**`ApplicationContext`创建**之前**初始化的，因此无法从`@PropertySources`Spring `@Configuration`文件中控制日志记录。更改日志记录系统或完全禁用它的唯一方法是通过系统属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

根据您的日志记录系统，将加载以下文件：

| 测井系统                    | 客制化                                                       |
| :-------------------------- | :----------------------------------------------------------- |
| 退回                        | `logback-spring.xml`，`logback-spring.groovy`，`logback.xml`，或者`logback.groovy` |
| Log4j2                      | `log4j2-spring.xml` 要么 `log4j2.xml`                        |
| JDK（Java实用程序日志记录） | `logging.properties`                                         |

|      | 如果可能，我们建议您将`-spring`变体用于日志记录配置（例如，`logback-spring.xml`而不是`logback.xml`）。如果使用标准配置位置，Spring将无法完全控制日志初始化。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 从“可执行jar”运行时，Java Util Logging存在一些已知的类加载问题，这些问题会引起问题。我们建议您尽可能从“可执行jar”运行时避免使用它。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

为了帮助进行自定义，`Environment`如下表所述，一些其他属性从Spring转移到System属性：

| 春季环境                              | 系统属性                          | 注释                                                         |
| :------------------------------------ | :-------------------------------- | :----------------------------------------------------------- |
| `logging.exception-conversion-word`   | `LOG_EXCEPTION_CONVERSION_WORD`   | 记录异常时使用的转换字。                                     |
| `logging.file.clean-history-on-start` | `LOG_FILE_CLEAN_HISTORY_ON_START` | 是否在启动时清除存档日志文件（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.file.name`                   | `LOG_FILE`                        | 如果定义，它将在默认日志配置中使用。                         |
| `logging.file.max-size`               | `LOG_FILE_MAX_SIZE`               | 日志文件的最大大小（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.file.max-history`            | `LOG_FILE_MAX_HISTORY`            | 要保留的最大归档日志文件数（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.file.path`                   | `LOG_PATH`                        | 如果定义，它将在默认日志配置中使用。                         |
| `logging.file.total-size-cap`         | `LOG_FILE_TOTAL_SIZE_CAP`         | 要保留的日志备份的总大小（如果启用了LOG_FILE）。（仅默认登录设置支持。） |
| `logging.pattern.console`             | `CONSOLE_LOG_PATTERN`             | 控制台上要使用的日志模式（stdout）。（仅默认登录设置支持。） |
| `logging.pattern.dateformat`          | `LOG_DATEFORMAT_PATTERN`          | 记录日期格式的附加模式。（仅默认登录设置支持。）             |
| `logging.pattern.file`                | `FILE_LOG_PATTERN`                | 文件中使用的日志模式（如果`LOG_FILE`已启用）。（仅默认登录设置支持。） |
| `logging.pattern.level`               | `LOG_LEVEL_PATTERN`               | 呈现日志级别时使用的格式（默认`%5p`）。（仅默认登录设置支持。） |
| `logging.pattern.rolling-file-name`   | `ROLLING_FILE_NAME_PATTERN`       | 过渡日志文件名的模式（默认`${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`）。（仅默认登录设置支持。） |
| `PID`                                 | `PID`                             | 当前进程ID（如果可能，并且尚未将其定义为OS环境变量，则发现）。 |

所有受支持的日志记录系统在解析其配置文件时都可以查阅系统属性。有关`spring-boot.jar`示例，请参见中的默认配置：

- [退回](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
- [Java Util日志记录](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

|      | 如果要在日志记录属性中使用占位符，则应使用[Spring Boot的语法](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-placeholders-in-properties)而不是基础框架的语法。值得注意的是，如果使用Logback，则应将其`:`用作属性名称与其默认值之间的分隔符，而不应使用`:-`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 您可以通过仅覆盖`LOG_LEVEL_PATTERN`（或`logging.pattern.level`使用Logback）将MDC和其他临时内容添加到日志行。例如，如果使用`logging.pattern.level=user:%X{user} %5p`，则默认日志格式包含“ user”的MDC条目（如果存在），如以下示例所示。`2019-08-30 12：30：04.031用户：某人INFO 22174-[[nio-8080-exec-0] demo.Controller 处理已认证的请求` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.7。登录扩展

Spring Boot包含许多Logback扩展，可以帮助进行高级配置。您可以在`logback-spring.xml`配置文件中使用这些扩展名。

|      | 由于标准`logback.xml`配置文件的加载时间过早，因此无法在其中使用扩展名。您需要使用`logback-spring.xml`或定义一个`logging.config`属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 这些扩展不能与Logback的[配置扫描一起使用](https://logback.qos.ch/manual/configuration.html#autoScan)。如果尝试这样做，则对配置文件进行更改将导致类似于以下记录之一的错误： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```
ch.qos.logback.core.joran.spi.Interpreter@4中的错误：71-[springProperty]没有适用的操作，当前ElementPath是[[configuration] [springProperty]]ch.qos.logback.core.joran.spi.Interpreter@4中的
错误：71-[springProfile]没有适用的操作，当前ElementPath是[[configuration] [springProfile]]
```

#### 4.7.1。特定于配置文件的配置

使用``标签，您可以根据活动的Spring配置文件选择包括或排除配置部分。概要文件部分在``元素内的任何位置都受支持。使用`name`属性指定哪个配置文件接受配置。所述``标记可包含一个简单的配置文件的名称（例如`staging`）或轮廓表达。简档表达式允许例如表达更复杂的简档逻辑`production & (eu-central | eu-west)`。有关更多详细信息，请参阅[参考指南](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。以下清单显示了三个样本概要文件：

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

#### 4.7.2。环境特性

该``标签可以让你从Spring公开属性`Environment`的范围内的logback使用。如果要从`application.properties`Logback配置中访问文件中的值，则这样做很有用。该标签的工作方式类似于Logback的标准``标签。但是，`value`您无需指定direct，而是可以指定`source`属性的（来自`Environment`）。如果需要将属性存储在`local`范围之外的其他位置，则可以使用该`scope`属性。如果需要后备值（如果未在中设置属性`Environment`），则可以使用`defaultValue`属性。以下示例显示如何公开在Logback中使用的属性：

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

|      | 在`source`必须在串的情况下（如指定`my.property-name`）。但是，可以`Environment`使用宽松规则将属性添加到中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 5.国际化

Spring Boot支持本地化消息，因此您的应用程序可以迎合不同语言首选项的用户。默认情况下，Spring Boot `messages`在类路径的根目录下查找资源包的存在。

|      | 当配置的资源束的默认属性文件可用时（即`messages.properties`默认情况下），将应用自动配置。如果您的资源包仅包含特定于语言的属性文件，则需要添加默认文件。如果找不到与任何配置的基本名称匹配的属性文件，则不会自动配置`MessageSource`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

可以使用`spring.messages`名称空间配置资源包的基本名称以及其他几个属性，如以下示例所示：

```properties
spring.messages.basename=messages,config.i18n.messages
spring.messages.fallback-to-system-locale=false
```

|      | `spring.messages.basename` 支持以逗号分隔的位置列表，即包限定符或从类路径根目录解析的资源。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

请参阅[`MessageSourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/context/MessageSourceProperties.java)以获取更多受支持的选项。

## 6. JSON

Spring Boot提供了与三个JSON映射库的集成：

- 格森
- 杰克逊
- JSON-B

Jackson是首选的默认库。

### 6.1。杰克逊

提供了Jackson的自动配置，并且Jackson是的一部分`spring-boot-starter-json`。当Jackson放在类路径上时，将`ObjectMapper`自动配置Bean。提供了几个配置属性，用于[自定义的配置`ObjectMapper`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-customize-the-jackson-objectmapper)。

### 6.2。格森

提供了Gson的自动配置。当Gson在类路径上时，将`Gson`自动配置一个bean。`spring.gson.*`提供了几个配置属性用于自定义配置。为了获得更多控制权，`GsonBuilderCustomizer`可以使用一个或多个bean。

### 6.3。JSON-B

提供了JSON-B的自动配置。当JSON-B API和实现位于类路径上时，`Jsonb`将自动配置Bean。首选的JSON-B实现是提供依赖管理的Apache Johnzon。

## 7.开发Web应用程序

Spring Boot非常适合Web应用程序开发。您可以使用嵌入式Tomcat，Jetty，Undertow或Netty创建独立的HTTP服务器。大多数Web应用程序都使用该`spring-boot-starter-web`模块来快速启动和运行。您还可以选择使用该`spring-boot-starter-webflux`模块来构建反应式Web应用程序。

如果尚未开发Spring Boot Web应用程序，则可以遵循“ Hello World！”。*[入门](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started-first-application)*部分中的示例。

### 7.1。“ Spring Web MVC框架”

在[春天Web MVC框架](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc)（通常简称为“春MVC”）是一个丰富的“模型视图控制器” Web框架。Spring MVC允许您创建特殊的`@Controller`或`@RestController`bean来处理传入的HTTP请求。控制器中的方法通过使用`@RequestMapping`注释映射到HTTP 。

以下代码显示了`@RestController`提供JSON数据的典型代码：

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
        // ...
    }

}
```

Spring MVC是核心Spring Framework的一部分，有关详细信息，请参阅[参考文档](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc)。在[spring.io/guides](https://spring.io/guides)上还有一些涵盖Spring MVC的指南。

#### 7.1.1。Spring MVC自动配置

Spring Boot为Spring MVC提供了自动配置，可与大多数应用程序完美配合。

自动配置在Spring的默认值之上添加了以下功能：

- 包含`ContentNegotiatingViewResolver`和`BeanNameViewResolver`。
- 支持服务静态资源，包括对WebJars的支持（[在本文档的后面部分中有介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content)）。
- 自动注册`Converter`，`GenericConverter`和`Formatter`豆类。
- 支持`HttpMessageConverters`（[在本文档后面介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)）。
- 自动注册`MessageCodesResolver`（[在本文档后面介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-message-codes)）。
- 静态`index.html`支持。
- 定制`Favicon`支持（[在本文档后面部分中介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)）。
- 自动使用`ConfigurableWebBindingInitializer`bean（[在本文档后面部分中介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)）。

如果要保留这些Spring Boot MVC定制并进行更多的[MVC定制](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc)（拦截器，格式化程序，视图控制器和其他功能），则可以添加自己`@Configuration`的type类，`WebMvcConfigurer`但**不添加** `@EnableWebMvc`。

如果你想提供的定制情况`RequestMappingHandlerMapping`，`RequestMappingHandlerAdapter`或者`ExceptionHandlerExceptionResolver`，仍然保持弹簧引导MVC自定义，你可以声明类型的豆`WebMvcRegistrations`，并用它来提供这些组件的定制实例。

如果你想利用Spring MVC中的完全控制，你可以添加自己的`@Configuration`注解为`@EnableWebMvc`，或者添加自己的`@Configuration`-annotated `DelegatingWebMvcConfiguration`中的Javadoc中所述`@EnableWebMvc`。

#### 7.1.2。HttpMessageConverters

Spring MVC使用该`HttpMessageConverter`接口来转换HTTP请求和响应。开箱即用中包含明智的默认设置。例如，可以将对象自动转换为JSON（通过使用Jackson库）或XML（通过使用Jackson XML扩展（如果可用），或者通过使用JAXB（如果Jackson XML扩展不可用））。默认情况下，字符串以编码`UTF-8`。

如果需要添加或自定义转换器，则可以使用Spring Boot的`HttpMessageConverters`类，如以下清单所示：

```java
import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }

}
```

`HttpMessageConverter`上下文中存在的任何Bean都将添加到转换器列表中。您也可以用相同的方法覆盖默认转换器。

#### 7.1.3。自定义JSON序列化器和反序列化器

如果使用Jackson来序列化和反序列化JSON数据，则可能需要编写自己的`JsonSerializer`和`JsonDeserializer`类。自定义序列化程序通常是[通过模块向Jackson进行注册的](https://github.com/FasterXML/jackson-docs/wiki/JacksonHowToCustomSerializers)，但是Spring Boot提供了一种可选的`@JsonComponent`批注，使直接注册Spring Bean更加容易。

您可以使用`@JsonComponent`直接的注解`JsonSerializer`，`JsonDeserializer`或`KeyDeserializer`实现。您还可以在包含序列化器/反序列化器作为内部类的类上使用它，如以下示例所示：

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

    public static class Serializer extends JsonSerializer<SomeObject> {
        // ...
    }

    public static class Deserializer extends JsonDeserializer<SomeObject> {
        // ...
    }

}
```

中的所有`@JsonComponent`bean都会`ApplicationContext`自动向Jackson进行注册。因为`@JsonComponent`已使用meta注解`@Component`，所以通常的组件扫描规则适用。

Spring Boot还提供了[`JsonObjectSerializer`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectSerializer.java)和[`JsonObjectDeserializer`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectDeserializer.java)基类，它们在序列化对象时为标准Jackson版本提供了有用的替代方法。见[`JsonObjectSerializer`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/jackson/JsonObjectSerializer.html)和[`JsonObjectDeserializer`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/jackson/JsonObjectDeserializer.html)在Javadoc了解详情。

#### 7.1.4。MessageCodesResolver

Spring MVC的具有产生错误代码从绑定错误的渲染错误消息的策略：`MessageCodesResolver`。如果您设置`spring.mvc.message-codes-resolver-format`属性`PREFIX_ERROR_CODE`或`POSTFIX_ERROR_CODE`，Spring Boot会为您创建一个（请参阅中的枚举[`DefaultMessageCodesResolver.Format`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.Format.html)）。

#### 7.1.5。静态内容

默认情况下，Spring Boot从类路径中名为`/static`（`/public`或`/resources`或`/META-INF/resources`）的目录或的根目录中提供静态内容`ServletContext`。它使用`ResourceHttpRequestHandler`Spring MVC中的from，因此您可以通过添加自己`WebMvcConfigurer`的`addResourceHandlers`方法并覆盖该方法来修改该行为。

在独立的Web应用程序中，还启用了容器中的默认servlet，并将其用作后备，从`ServletContext`Spring决定不处理它的根开始提供内容。在大多数情况下，这不会发生（除非您修改默认的MVC配置），因为Spring始终可以通过处理请求`DispatcherServlet`。

默认情况下，资源映射到`/**`，但是您可以使用`spring.mvc.static-path-pattern`属性对其进行调整。例如，将所有资源重新分配到`/resources/**`以下位置即可：

```properties
spring.mvc.static-path-pattern=/resources/**
```

您也可以使用`spring.resources.static-locations`属性来自定义静态资源位置（用目录位置列表替换默认值）。根Servlet上下文路径`"/"`也会自动添加为位置。

除了前面提到的“标准”静态资源位置，[Webjars内容也](https://www.webjars.org/)有特殊情况。`/webjars/**`如果jar文件以Webjars格式打包，则从jar文件提供带有路径的所有资源。

|      | `src/main/webapp`如果您的应用程序打包为jar，则 不要使用该目录。尽管此目录是一个通用标准，但它**仅**与war打包一起使用，并且如果生成jar，大多数构建工具都将其忽略。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring Boot还支持Spring MVC提供的高级资源处理功能，允许使用案例，例如缓存清除静态资源或对Webjars使用版本无关的URL。

要对Webjars使用版本无关的URL，请添加`webjars-locator-core`依赖项。然后声明您的Webjar。使用jQuery作为一个例子，将`"/webjars/jquery/jquery.min.js"`在结果中`"/webjars/jquery/x.y.z/jquery.min.js"`，其中`x.y.z`是Webjar版本。

|      | 如果使用JBoss，则需要声明`webjars-locator-jboss-vfs`依赖关系而不是`webjars-locator-core`。否则，所有Webjar都将解析为`404`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要使用缓存清除，以下配置为所有静态资源配置了缓存清除解决方案，从而有效地``在URL中添加了内容哈希，例如，：

```properties
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

|      | 由于`ResourceUrlEncodingFilter`为Thymeleaf和FreeMarker自动配置了，因此在运行时可以在模板中重写资源链接。使用JSP时，您应该手动声明此过滤器。目前尚不自动支持其他模板引擎，但可以与自定义模板宏/帮助程序一起使用，也可以使用[`ResourceUrlProvider`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/web/servlet/resource/ResourceUrlProvider.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

例如，当使用JavaScript模块加载器动态加载资源时，不能重命名文件。这就是为什么其他策略也受支持并且可以组合的原因。“固定”策略在URL中添加静态版本字符串，而不更改文件名，如以下示例所示：

```properties
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

通过这种配置，位于下面的JavaScript模块`"/js/lib/"`使用固定的版本控制策略（`"/v12/js/lib/mymodule.js"`），而其他资源仍使用内容版本（``）。

请参阅[`ResourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java)以获取更多受支持的选项。

|      | 专门的[博客文章](https://spring.io/blog/2014/07/24/spring-framework-4-1-handling-static-web-resources)和Spring Framework的[参考文档中](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-config-static-resources)已经对该功能进行了全面的描述。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 7.1.6。欢迎页面

Spring Boot支持静态和模板欢迎页面。它首先`index.html`在配置的静态内容位置中查找文件。如果未找到，则寻找`index`模板。如果找到任何一个，它将自动用作应用程序的欢迎页面。

#### 7.1.7。自定义图标

与其他静态资源一样，Spring Boot `favicon.ico`在配置的静态内容位置中查找a 。如果存在这样的文件，它将自动用作应用程序的收藏夹图标。

#### 7.1.8。路径匹配和内容协商

Spring MVC可以通过查看请求路径并将其匹配到应用程序中定义的映射（例如，`@GetMapping`Controller方法的注释）来将传入的HTTP请求映射到处理程序。

Spring Boot默认选择禁用后缀模式匹配，这意味着`"GET /projects/spring-boot.json"`类似的请求将不会与`@GetMapping("/projects/spring-boot")`映射匹配。这被认为是[Spring MVC应用程序](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)的[最佳实践](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)。过去，此功能主要用于未发送正确的“ Accept”请求标头的HTTP客户端。我们需要确保将正确的内容类型发送给客户端。如今，内容协商已变得更加可靠。

还有其他处理HTTP客户端的方法，这些方法不能始终发送正确的“ Accept”请求标头。除了使用后缀匹配，我们还可以使用查询参数来确保将诸如这样的请求`"GET /projects/spring-boot?format=json"`映射到`@GetMapping("/projects/spring-boot")`：

```properties
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

后缀模式匹配已被弃用，并将在以后的版本中删除。如果您了解了注意事项，但仍希望您的应用程序使用后缀模式匹配，则需要以下配置：

```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

另外，与其打开所有后缀模式，不如只支持已注册的后缀模式，这是更安全的：

```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```

#### 7.1.9。ConfigurableWebBindingInitializer

Spring MVC使用a `WebBindingInitializer`来初始化`WebDataBinder`特定请求。如果创建自己的`ConfigurableWebBindingInitializer` `@Bean`，Spring Boot会自动将Spring MVC配置为使用它。

#### 7.1.10。模板引擎

除了REST Web服务之外，您还可以使用Spring MVC来提供动态HTML内容。Spring MVC支持各种模板技术，包括Thymeleaf，FreeMarker和JSP。同样，许多其他模板引擎包括它们自己的Spring MVC集成。

Spring Boot包括对以下模板引擎的自动配置支持：

- [FreeMarker](https://freemarker.apache.org/docs/)
- [Groovy](http://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)
- [胸腺](https://www.thymeleaf.org/)
- [胡子](https://mustache.github.io/)

|      | 如果可能，应避免使用JSP。将它们与嵌入式servlet容器一起使用时，存在几个[已知的限制](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-jsp-limitations)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

在默认配置下使用这些模板引擎之一时，将从中自动提取模板`src/main/resources/templates`。

|      | 根据您运行应用程序的方式，IntelliJ IDEA对类路径的排序方式不同。与使用Maven或Gradle或从打包的jar运行应用程序时，从IDE的主要方法运行应用程序的顺序会有所不同。这可能会导致Spring Boot无法在类路径上找到模板。如果遇到此问题，可以在IDE中重新排序类路径，以首先放置模块的类和资源。或者，您可以配置模板前缀来搜索`templates`类路径上的每个目录，如下所示：`classpath*:/templates/`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 7.1.11。错误处理

默认情况下，Spring Boot提供了`/error`一种可以明智地处理所有错误的映射，并且已在servlet容器中注册为“全局”错误页面。对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于浏览器客户端，存在一个“ whitelabel”错误视图，该视图以HTML格式呈现相同的数据（要对其进行自定义，请添加`View`解析为的`error`）。要完全替换默认行为，可以实现`ErrorController`并注册该类型的Bean定义，或者添加类型的Bean `ErrorAttributes`以使用现有机制但替换其内容。

|      | 在`BasicErrorController`可以用作自定义基类`ErrorController`。如果您要为新的内容类型添加处理程序（默认是`text/html`专门处理并为其他所有内容提供后备功能），则此功能特别有用。为此，请扩展`BasicErrorController`，添加`@RequestMapping`具有`produces`属性的带有的公共方法，然后创建新类型的bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以定义带有注释的类，`@ControllerAdvice`以自定义JSON文档以针对特定的控制器和/或异常类型返回，如以下示例所示：

```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

    @ExceptionHandler(YourException.class)
    @ResponseBody
    ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        return HttpStatus.valueOf(statusCode);
    }

}
```

在前面的示例中，如果if `YourException`由与相同的程序包中定义的控制器抛出，则使用POJO `AcmeController`的JSON表示`CustomErrorType`代替该`ErrorAttributes`表示。

##### 自定义错误页面

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到文件`/error`夹。错误页面可以是静态HTML（即添加到任何静态资源文件夹下），也可以使用模板来构建。文件名应为确切的状态代码或系列掩码。

例如，要映射`404`到静态HTML文件，您的文件夹结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

要`5xx`使用FreeMarker模板映射所有错误，您的文件夹结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftlh
             +- <other templates>
```

对于更复杂的映射，还可以添加实现该`ErrorViewResolver`接口的bean ，如以下示例所示：

```java
public class MyErrorViewResolver implements ErrorViewResolver {

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request,
            HttpStatus status, Map<String, Object> model) {
        // Use the request or status to optionally return a ModelAndView
        return ...
    }

}
```

您还可以使用常规的Spring MVC功能，例如[`@ExceptionHandler`方法](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers)和[`@ControllerAdvice`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-ann-controller-advice)。在`ErrorController`随后拿起任何未处理的异常。

##### 在Spring MVC外部映射错误页面

对于不使用Spring MVC的应用程序，您可以使用`ErrorPageRegistrar`接口直接注册`ErrorPages`。此抽象直接与基础嵌入式servlet容器一起使用，即使您没有Spring MVC也可以使用`DispatcherServlet`。

```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
    return new MyErrorPageRegistrar();
}

// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }

}
```

|      | 如果您`ErrorPage`使用最终由a处理的路径注册`Filter`（这在某些非Spring Web框架，例如Jersey和Wicket中很常见），则`Filter`必须将其显式注册为`ERROR`调度程序，如以下示例所示： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```java
@Bean
public FilterRegistrationBean myFilter() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new MyFilter());
    ...
    registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
    return registration;
}
```

请注意，默认值`FilterRegistrationBean`不包括`ERROR`调度程序类型。

注意：当Spring Boot部署到servlet容器时，将使用其错误页面过滤器将具有错误状态的请求转发到相应的错误页面。如果尚未提交响应，则只能将请求转发到正确的错误页面。缺省情况下，WebSphere Application Server 8.0和更高版本在成功完成servlet的服务方法后提交响应。您应该通过设置`com.ibm.ws.webcontainer.invokeFlushAfterService`为来禁用此行为`false`。

#### 7.1.12。春季HATEOAS

如果您开发使用超媒体的RESTful API，Spring Boot会为Spring HATEOAS提供自动配置，该配置可与大多数应用程序很好地兼容。自动配置取代了使用`@EnableHypermediaSupport`和注册大量Bean的需求，以简化构建基于超媒体的应用程序，包括`LinkDiscoverers`（用于客户端支持）和`ObjectMapper`配置为将响应正确地编组为所需表示形式的Bean 。的`ObjectMapper`是通过设置各种定制的`spring.jackson.*`属性，或者，如果存在的话，通过一个`Jackson2ObjectMapperBuilder`豆。

您可以使用来控制Spring HATEOAS的配置`@EnableHypermediaSupport`。请注意，这样做会禁用`ObjectMapper`前面所述的自定义。

#### 7.1.13。CORS支持

[跨域资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是由[大多数浏览器](https://caniuse.com/#feat=cors)实施的[W3C规范](https://www.w3.org/TR/cors/)，使您可以灵活地指定对哪种类型的跨域请求进行授权，而不是使用诸如IFRAME或JSONP之类的安全性较低，功能较弱的方法。 。

从4.2版本开始，Spring MVC [支持CORS](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-cors)。在Spring Boot应用程序中将[控制器方法CORS配置](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-cors-controller)与[`@CrossOrigin`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html)注释一起使用不需要任何特定的配置。 可以通过使用自定义方法注册bean 来定义[全局CORS配置](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#mvc-cors-global)，如以下示例所示：`WebMvcConfigurer``addCorsMappings(CorsRegistry)`

```java
@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
        };
    }
}
```

### 7.2。“ Spring WebFlux框架”

Spring WebFlux是Spring Framework 5.0中引入的新的响应式Web框架。与Spring MVC不同，它不需要Servlet API，是完全异步且无阻塞的，并通过[Reactor项目](https://projectreactor.io/)实现[Reactive Streams](https://www.reactive-streams.org/)规范。

Spring WebFlux有两种形式：功能性的和基于注释的。基于注释的模型非常类似于Spring MVC模型，如以下示例所示：

```java
@RestController
@RequestMapping("/users")
public class MyRestController {

    @GetMapping("/{user}")
    public Mono<User> getUser(@PathVariable Long user) {
        // ...
    }

    @GetMapping("/{user}/customers")
    public Flux<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @DeleteMapping("/{user}")
    public Mono<User> deleteUser(@PathVariable Long user) {
        // ...
    }

}
```

功能变体“ WebFlux.fn”将路由配置与请求的实际处理分开，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
public class RoutingConfiguration {

    @Bean
    public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
        return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
                .andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
                .andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
    }

}

@Component
public class UserHandler {

    public Mono<ServerResponse> getUser(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> deleteUser(ServerRequest request) {
        // ...
    }
}
```

WebFlux是Spring Framework的一部分，其[参考文档中](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn)提供了详细信息。

|      | 您可以根据需要定义任意数量的`RouterFunction`bean，以模块化路由器的定义。如果需要应用优先级，可以订购Bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

首先，将`spring-boot-starter-webflux`模块添加到您的应用程序。

|      | 在应用程序中 同时添加`spring-boot-starter-web`和`spring-boot-starter-webflux`模块会导致Spring Boot自动配置Spring MVC，而不是WebFlux。之所以选择这种行为，是因为许多Spring开发人员将`spring-boot-starter-webflux`其添加到他们的Spring MVC应用程序中以使用反应式`WebClient`。您仍然可以通过将选定的应用程序类型设置为来强制执行选择`SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE)`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 7.2.1。Spring WebFlux自动配置

Spring Boot为Spring WebFlux提供了自动配置，可与大多数应用程序完美配合。

自动配置在Spring的默认值之上添加了以下功能：

- 为`HttpMessageReader`和`HttpMessageWriter`实例配置编解码器（[在本文档的后面部分中介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-webflux-httpcodecs)）。
- 支持服务静态资源，包括对WebJars的支持（[在本文档的后面部分中介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content)）。

如果您想保留Spring Boot WebFlux功能并想要添加其他[WebFlux配置](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-config)，则可以添加自己`@Configuration`的类型类，`WebFluxConfigurer`但**不添加** `@EnableWebFlux`。

如果要完全控制Spring WebFlux，可以使用添加自己的`@Configuration`注释`@EnableWebFlux`。

#### 7.2.2。带有HttpMessageReaders和HttpMessageWriters的HTTP编解码器

Spring WebFlux使用`HttpMessageReader`和`HttpMessageWriter`接口转换HTTP请求和响应。`CodecConfigurer`通过查看类路径中可用的库，将它们配置为具有合理的默认值。

Spring Boot为编解码器提供了专用的配置属性`spring.codec.*`。它还通过使用`CodecCustomizer`实例来应用进一步的自定义。例如，将`spring.jackson.*`配置密钥应用于Jackson编解码器。

如果需要添加或自定义编解码器，则可以创建一个自定义`CodecCustomizer`组件，如以下示例所示：

```java
import org.springframework.boot.web.codec.CodecCustomizer;

@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public CodecCustomizer myCodecCustomizer() {
        return codecConfigurer -> {
            // ...
        };
    }

}
```

您还可以利用[Boot的自定义JSON序列化器和反序列化器](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-json-components)。

#### 7.2.3。静态内容

默认情况下，Spring Boot从类路径中名为`/static`（`/public`或`/resources`或`/META-INF/resources`）的目录中提供静态内容。它使用`ResourceWebHandler`Spring WebFlux中的，因此您可以通过添加自己`WebFluxConfigurer`的`addResourceHandlers`方法并覆盖该方法来修改该行为。

默认情况下，资源映射在上`/**`，但是您可以通过设置`spring.webflux.static-path-pattern`属性来对其进行调整。例如，将所有资源重新分配到`/resources/**`以下位置即可：

```properties
spring.webflux.static-path-pattern=/resources/**
```

您还可以使用来自定义静态资源位置`spring.resources.static-locations`。这样做会将默认值替换为目录位置列表。如果这样做，默认的欢迎页面检测将切换到您的自定义位置。因此，如果`index.html`启动时您的任何位置都有，则为应用程序的主页。

除了前面列出的“标准”静态资源位置之外，[Webjars内容也](https://www.webjars.org/)有特殊情况。`/webjars/**`如果jar文件以Webjars格式打包，则从jar文件提供带有路径的所有资源。

|      | Spring WebFlux应用程序不严格依赖Servlet API，因此不能将它们部署为war文件，也不使用`src/main/webapp`目录。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 7.2.4。模板引擎

除了REST Web服务之外，您还可以使用Spring WebFlux来提供动态HTML内容。Spring WebFlux支持多种模板技术，包括Thymeleaf，FreeMarker和Mustache。

Spring Boot包括对以下模板引擎的自动配置支持：

- [FreeMarker](https://freemarker.apache.org/docs/)
- [胸腺](https://www.thymeleaf.org/)
- [胡子](https://mustache.github.io/)

在默认配置下使用这些模板引擎之一时，将从中自动提取模板`src/main/resources/templates`。

#### 7.2.5。错误处理

Spring Boot提供了`WebExceptionHandler`一个以合理的方式处理所有错误的工具。它在处理顺序中的位置紧靠WebFlux提供的处理程序之前，该处理程序被认为是最后一个。对于机器客户端，它将生成JSON响应，其中包含错误，HTTP状态和异常消息的详细信息。对于浏览器客户端，有一个“ whitelabel”错误处理程序，以HTML格式呈现相同的数据。您还可以提供自己的HTML模板来显示错误（请参阅[下一节](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-webflux-error-handling-custom-error-pages)）。

定制此功能的第一步通常涉及使用现有机制，但替换或增加错误内容。为此，您可以添加类型的Bean `ErrorAttributes`。

要更改错误处理行为，可以实现`ErrorWebExceptionHandler`并注册该类型的bean定义。由于a `WebExceptionHandler`级别很低，因此Spring Boot还提供了一种方便的方式`AbstractErrorWebExceptionHandler`，使您可以通过WebFlux功能方式处理错误，如以下示例所示：

```java
public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {

    // Define constructor here

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {

        return RouterFunctions
                .route(aPredicate, aHandler)
                .andRoute(anotherPredicate, anotherHandler);
    }

}
```

为了获得更完整的图像，您还可以`DefaultErrorWebExceptionHandler`直接继承子类并覆盖特定的方法。

##### 自定义错误页面

如果要显示给定状态代码的自定义HTML错误页面，可以将文件添加到文件`/error`夹。错误页面可以是静态HTML（即添加到任何静态资源文件夹下），也可以使用模板构建。文件名应为确切的状态代码或系列掩码。

例如，要映射`404`到静态HTML文件，您的文件夹结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

要`5xx`使用Mustache模板映射所有错误，您的文件夹结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.mustache
             +- <other templates>
```

#### 7.2.6。网页过滤器

Spring WebFlux提供了一个`WebFilter`可用于过滤HTTP请求-响应交换的接口。 `WebFilter`在应用程序上下文中找到的bean将自动用于过滤每个交换。

如果过滤器的顺序很重要，则可以使用`Ordered`或对其进行注释`@Order`。Spring Boot自动配置可能会为您配置Web过滤器。这样做时，将使用下表中显示的顺序：

| 网页过滤器                         | 订购                             |
| :--------------------------------- | :------------------------------- |
| `MetricsWebFilter`                 | `Ordered.HIGHEST_PRECEDENCE + 1` |
| `WebFilterChainProxy` （春季安全） | `-100`                           |
| `HttpTraceWebFilter`               | `Ordered.LOWEST_PRECEDENCE - 10` |

### 7.3。JAX-RS和泽西岛

如果您更喜欢REST端点使用JAX-RS编程模型，则可以使用可用的实现之一来代替Spring MVC。 [Jersey](https://jersey.github.io/)和[Apache CXF](https://cxf.apache.org/)开箱即用。CXF需要您注册其`Servlet`或`Filter`为`@Bean`您的应用程序上下文。泽西岛（Jersey）有一些本机Spring支持，因此我们在Spring Boot中还与启动器一起为其提供了自动配置支持。

要开始使用Jersey，请将包含`spring-boot-starter-jersey`为依赖项，然后需要使用一种`@Bean`类型`ResourceConfig`注册所有端点，如以下示例所示：

```java
@Component
public class JerseyConfig extends ResourceConfig {

    public JerseyConfig() {
        register(Endpoint.class);
    }

}
```

|      | 泽西岛对扫描可执行归档文件的支持非常有限。例如，它无法扫描[完全可执行的jar文件中](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment-install)或`WEB-INF/classes`运行可执行war文件时发现的包中的端点。为避免此限制，`packages`不应该使用该方法，并且应该使用该`register`方法分别注册端点，如前面的示例所示。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

对于更高级的自定义，您还可以注册任意数量的实现的Bean `ResourceConfigCustomizer`。

所有注册的端点都应`@Components`带有HTTP资源注释（`@GET`及其他注释），如以下示例所示：

```java
@Component
@Path("/hello")
public class Endpoint {

    @GET
    public String message() {
        return "Hello";
    }

}
```

由于`Endpoint`是Spring `@Component`，因此其生命周期由Spring管理，您可以使用`@Autowired`批注注入依赖项，并使用`@Value`批注注入外部配置。默认情况下，Jersey servlet已注册并映射到`/*`。您可以通过添加改变映射`@ApplicationPath`到你`ResourceConfig`。

默认情况下，Jersey 以名为`@Bean`的类型设置为Servlet 。默认情况下，该servlet延迟初始化，但是您可以通过设置来自定义该行为。您可以通过使用相同的名称创建自己的一个来禁用或覆盖该bean。您还可以通过设置使用过滤器而不是servlet （在这种情况下，要替换或覆盖的是）。过滤器具有，您可以使用设置。通过使用指定属性映射，可以为servlet和过滤器注册都赋予init参数。`ServletRegistrationBean``jerseyServletRegistration``spring.jersey.servlet.load-on-startup``spring.jersey.type=filter``@Bean``jerseyFilterRegistration``@Order``spring.jersey.filter.order``spring.jersey.init.*`

### 7.4。嵌入式Servlet容器支持

Spring Boot包括对嵌入式[Tomcat](https://tomcat.apache.org/)，[Jetty](https://www.eclipse.org/jetty/)和[Undertow](https://github.com/undertow-io/undertow)服务器的支持。大多数开发人员使用适当的“启动器”来获取完全配置的实例。默认情况下，嵌入式服务器在port上侦听HTTP请求`8080`。

#### 7.4.1。Servlet，过滤器和侦听器

使用嵌入式Servlet容器时，您可以`HttpSessionListener`通过使用Spring Bean或扫描Servlet组件来注册Servlet规范中的Servlet，过滤器和所有侦听器（例如）。

##### 将Servlet，过滤器和侦听器注册为Spring Bean

作为Spring bean的任何`Servlet`，`Filter`或servlet `*Listener`实例都向嵌入式容器注册。如果要`application.properties`在配置过程中引用一个值，这将特别方便。

默认情况下，如果上下文仅包含单个Servlet，则将其映射到`/`。如果有多个servlet bean，则将bean名称用作路径前缀。过滤器映射到`/*`。

如果以公约为基础测绘不够灵活，你可以使用`ServletRegistrationBean`，`FilterRegistrationBean`以及`ServletListenerRegistrationBean`类的完全控制。

通常可以使无序滤豆处于无序状态。如果需要一个特定的顺序，你应该注释`Filter`用`@Order`或使其实现`Ordered`。您无法`Filter`通过使用注释其bean方法来配置a的顺序`@Order`。如果您无法更改`Filter`要添加`@Order`或实现的类`Ordered`，则必须为定义一个`FilterRegistrationBean`，`Filter`并使用`setOrder(int)`方法设置注册bean的顺序。避免配置一个在处读取请求正文的过滤器`Ordered.HIGHEST_PRECEDENCE`，因为它可能与应用程序的字符编码配置不符。如果Servlet过滤器包装了请求，则应以小于或等于的顺序对其进行配置`OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER`。

|      | 要查看`Filter`应用程序中每个组件的顺序，请为`web` [日志记录组](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-log-groups)（`logging.level.web=debug`）启用调试级别的日志[记录](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-log-groups)。然后，将在启动时记录已注册过滤器的详细信息，包括其顺序和URL模式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 注册`Filter`bean 时要小心，因为它们是在应用程序生命周期中很早就初始化的。如果需要注册一个`Filter`与其他bean交互的对象，请考虑使用a [`DelegatingFilterProxyRegistrationBean`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/web/servlet/DelegatingFilterProxyRegistrationBean.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 7.4.2。Servlet上下文初始化

嵌入式Servlet容器不会直接执行Servlet 3.0+ `javax.servlet.ServletContainerInitializer`接口或Spring的`org.springframework.web.WebApplicationInitializer`接口。这是一个有意的设计决定，旨在降低旨在在战争中运行的第三方库可能破坏Spring Boot应用程序的风险。

如果您需要在Spring Boot应用程序中执行servlet上下文初始化，则应该注册一个实现该`org.springframework.boot.web.servlet.ServletContextInitializer`接口的bean 。单一`onStartup`方法提供对的访问，`ServletContext`并且在必要时可以轻松用作现有的适配器`WebApplicationInitializer`。

##### 扫描Servlet，过滤器和侦听器

当使用嵌入式容器中，类自动登记注释有`@WebServlet`，`@WebFilter`和`@WebListener`可以通过使用被使能`@ServletComponentScan`。

|      | `@ServletComponentScan` 在独立容器中无效，而是使用容器的内置发现机制。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 7.4.3。ServletWebServerApplicationContext

在幕后，Spring Boot `ApplicationContext`对嵌入式servlet容器使用了不同类型的支持。该`ServletWebServerApplicationContext`是一种特殊类型的`WebApplicationContext`通过搜索单说引导自身`ServletWebServerFactory`豆。通常是`TomcatServletWebServerFactory`，`JettyServletWebServerFactory`或`UndertowServletWebServerFactory`已被自动配置。

|      | 通常，您不需要了解这些实现类。大多数应用程序都自动配置，并适当的`ApplicationContext`和`ServletWebServerFactory`以您的名义创建。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 7.4.4。自定义嵌入式Servlet容器

可以使用Spring `Environment`属性来配置常见的servlet容器设置。通常，您将在`application.properties`文件中定义属性。

常用服务器设置包括：

- 网络设置：侦听传入HTTP请求的端口（`server.port`），绑定到的接口地址`server.address`，等等。
- 会话设置：会话是否为持久（`server.servlet.session.persistent`），会话超时（`server.servlet.session.timeout`），会话数据位置（`server.servlet.session.store-dir`）和会话cookie配置（`server.servlet.session.cookie.*`）。
- 错误管理：错误页面的位置（`server.error.path`），依此类推。
- [SSL协议](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-configure-ssl)
- [HTTP压缩](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#how-to-enable-http-response-compression)

Spring Boot尝试尽可能多地公开通用设置，但这并不总是可能的。在这种情况下，专用名称空间提供服务器特定的自定义项（请参阅`server.tomcat`和`server.undertow`）。例如，可以使用嵌入式servlet容器的特定功能配置[访问日志](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-configure-accesslogs)。

|      | 有关[`ServerProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)完整列表，请参见课程。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 程序化定制

如果需要以编程方式配置嵌入式servlet容器，则可以注册一个实现该`WebServerFactoryCustomizer`接口的Spring bean 。 `WebServerFactoryCustomizer`提供对的访问`ConfigurableServletWebServerFactory`，其中包括许多自定义设置方法。以下示例显示以编程方式设置端口：

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(9000);
    }

}
```

|      | `TomcatServletWebServerFactory`，`JettyServletWebServerFactory`和`UndertowServletWebServerFactory`是的专用变体`ConfigurableServletWebServerFactory`具有用于分别的Tomcat，码头和暗流额外定制setter方法。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 直接自定义ConfigurableServletWebServerFactory

如果前面的定制技术太有限，你可以注册`TomcatServletWebServerFactory`，`JettyServletWebServerFactory`或`UndertowServletWebServerFactory`豆你自己。

```java
@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
    TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
    factory.setPort(9000);
    factory.setSessionTimeout(10, TimeUnit.MINUTES);
    factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
    return factory;
}
```

提供了许多配置选项的设置器。如果您需要做一些更奇特的操作，还提供了几种受保护的方法“挂钩”。有关详细信息，请参见[源代码文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/web/servlet/server/ConfigurableServletWebServerFactory.html)。

#### 7.4.5。JSP局限性

当运行使用嵌入式servlet容器（并打包为可执行档案）的Spring Boot应用程序时，JSP支持存在一些限制。

- 对于Jetty和Tomcat，如果使用战争包装，它应该可以工作。与一起启动时`java -jar`，可执行的War将起作用，并且还将可部署到任何标准容器中。使用可执行jar时，不支持JSP。
- Undertow不支持JSP。
- 创建自定义`error.jsp`页面不会覆盖默认视图以进行[错误处理](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-error-handling)。 应改用[自定义错误页面](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-error-handling-custom-error-pages)。

### 7.5。嵌入式反应式服务器支持

Spring Boot包含对以下嵌入式反应式Web服务器的支持：Reactor Netty，Tomcat，Jetty和Undertow。大多数开发人员使用适当的“启动器”来获取完全配置的实例。默认情况下，嵌入式服务器在端口8080上侦听HTTP请求。

### 7.6。反应性服务器资源配置

自动配置Reactor Netty或Jetty服务器时，Spring Boot将创建特定的bean，这些bean将向服务器实例提供HTTP资源：`ReactorResourceFactory`或`JettyResourceFactory`。

默认情况下，这些资源还将与Reactor Netty和Jetty客户端共享，以实现最佳性能，具体如下：

- 服务器和客户端使用相同的技术
- 客户端实例是使用`WebClient.Builder`Spring Boot自动配置的bean 构建的

开发人员可以通过提供自定义`ReactorResourceFactory`或`JettyResourceFactory`bean 来覆盖Jetty和Reactor Netty的资源配置-这将同时应用于客户端和服务器。

您可以在[WebClient Runtime部分中](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-webclient-runtime)了解有关客户端资源配置的更多信息。

## 8. RSocket

[RSocket](https://rsocket.io/)是用于字节流传输的二进制协议。它通过通过单个连接传递的异步消息来启用对称交互模型。

`spring-messaging`Spring框架的模块在客户端和服务器端都支持RSocket请求者和响应者。有关更多详细信息，请参见Spring Framework参考中的[RSocket部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#rsocket-spring)，其中包括RSocket协议的概述。

### 8.1。RSocket策略自动配置

Spring Boot自动配置一个`RSocketStrategies`bean，该bean提供了用于编码和解码RSocket有效负载的所有必需基础结构。默认情况下，自动配置将尝试按顺序配置以下内容：

1. Jackson的[CBOR](https://cbor.io/)编解码器
2. 杰克逊的JSON编解码器

在`spring-boot-starter-rsocket`启动同时提供的依赖。查阅[Jackson支持部分，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-json-jackson)以了解有关定制可能性的更多信息。

开发人员可以`RSocketStrategies`通过创建实现该`RSocketStrategiesCustomizer`接口的bean来自定义组件。请注意，它们`@Order`很重要，因为它确定编解码器的顺序。

### 8.2。RSocket服务器自动配置

Spring Boot提供了RSocket服务器自动配置。所需的依赖项由提供`spring-boot-starter-rsocket`。

Spring Boot允许从WebFlux服务器通过WebSocket公开RSocket，或支持独立的RSocket服务器。这取决于应用程序的类型及其配置。

对于WebFlux应用程序（即类型`WebApplicationType.REACTIVE`），仅当以下属性匹配时，RSocket服务器才会插入Web服务器：

```properties
spring.rsocket.server.mapping-path=/rsocket # a mapping path is defined
spring.rsocket.server.transport=websocket # websocket is chosen as a transport
#spring.rsocket.server.port= # no port is defined
```

|      | 由于RSocket本身是使用该库构建的，因此只有Reactor Netty支持将RSocket插入Web服务器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

另外，RSocket TCP或Websocket服务器也可以作为独立的嵌入式服务器启动。除了依赖性要求之外，唯一需要的配置是为该服务器定义端口：

```properties
spring.rsocket.server.port=9898 # the only required configuration
spring.rsocket.server.transport=tcp # you're free to configure other properties
```

### 8.3。Spring Messaging RSocket支持

Spring Boot将为RSocket自动配置Spring Messaging基础结构。

这意味着Spring Boot将创建一个`RSocketMessageHandler`bean，该bean将处理对您的应用程序的RSocket请求。

### 8.4。使用以下命令调用RSocket服务`RSocketRequester`

一旦`RSocket`在服务器和客户端之间建立了通道，任何一方都可以向另一方发送或接收请求。

作为服务器，您可以`RSocketRequester`在RSocket的任何处理程序方法上注入实例`@Controller`。作为客户端，您需要首先配置和建立RSocket连接。Spring Boot会`RSocketRequester.Builder`使用预期的编解码器自动配置。

该`RSocketRequester.Builder`实例是一个原型bean，这意味着每个注入点将为您提供一个新实例。这是有意为之的，因为此构建器是有状态的，您不应使用同一实例创建具有不同设置的请求者。

以下代码显示了一个典型示例：

```java
@Service
public class MyService {

    private final Mono<RSocketRequester> rsocketRequester;

    public MyService(RSocketRequester.Builder rsocketRequesterBuilder) {
        this.rsocketRequester = rsocketRequesterBuilder
                .connectTcp("example.org", 9898).cache();
    }

    public Mono<User> someRSocketCall(String name) {
        return this.rsocketRequester.flatMap(req ->
                    req.route("user").data(name).retrieveMono(User.class));
    }

}
```

## 9.安全性

如果[Spring Security](https://spring.io/projects/spring-security)在类路径上，则默认情况下Web应用程序是安全的。Spring Boot依靠Spring Security的内容协商策略来确定使用`httpBasic`还是`formLogin`。要将方法级安全性添加到Web应用程序，您还可以添加`@EnableGlobalMethodSecurity`所需的设置。可以在《[Spring Security参考指南》中](https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/htmlsingle/#jc-method)找到更多信息。

默认值`UserDetailsService`只有一个用户。用户名为`user`，密码为随机密码，并在应用程序启动时以INFO级别显示，如以下示例所示：

```
使用生成的安全密码：78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

|      | 如果您微调日志记录配置，请确保将`org.springframework.boot.autoconfigure.security`类别设置为日志`INFO`级别的消息。否则，不会打印默认密码。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您可以通过提供`spring.security.user.name`和来更改用户名和密码`spring.security.user.password`。

默认情况下，您在Web应用程序中获得的基本功能是：

- 一个具有内存存储的bean `UserDetailsService`（或`ReactiveUserDetailsService`WebFlux应用程序）和一个具有生成的密码的单个用户（请参阅参考资料[`SecurityProperties.User`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/autoconfigure/security/SecurityProperties.User.html)，获取用户的属性）。
- `Accept`整个应用程序的基于表单的登录或HTTP基本安全性（取决于请求中的标头）（如果执行器位于类路径上，则包括执行器端点）。
- 一个`DefaultAuthenticationEventPublisher`用于发布身份验证事件。

您可以`AuthenticationEventPublisher`通过添加一个bean 来提供不同的东西。

### 9.1。MVC安全

默认的安全配置在`SecurityAutoConfiguration`和中实现`UserDetailsServiceAutoConfiguration`。 `SecurityAutoConfiguration`导入`SpringBootWebSecurityConfiguration`Web安全性并`UserDetailsServiceAutoConfiguration`配置身份验证，这在非Web应用程序中也很重要。要完全关闭默认的Web应用程序安全性配置或合并多个Spring Security组件（例如OAuth 2 Client和Resource Server），请添加一个类型的bean `WebSecurityConfigurerAdapter`（这样做不会禁用`UserDetailsService`配置或Actuator的安全性）。

为了还关闭`UserDetailsService`的配置，您可以添加类型的豆`UserDetailsService`，`AuthenticationProvider`或`AuthenticationManager`。

可以通过添加自定义来覆盖访问规则`WebSecurityConfigurerAdapter`。Spring Boot提供了便利的方法，可用于覆盖执行器端点和静态资源的访问规则。 `EndpointRequest`可用于创建`RequestMatcher`基于`management.endpoints.web.base-path`属性的。 `PathRequest`可用于`RequestMatcher`在常用位置创建for资源。

### 9.2。WebFlux安全

与Spring MVC应用程序类似，您可以通过添加`spring-boot-starter-security`依赖项来保护WebFlux应用程序。默认的安全配置在`ReactiveSecurityAutoConfiguration`和中实现`UserDetailsServiceAutoConfiguration`。 `ReactiveSecurityAutoConfiguration`导入`WebFluxSecurityConfiguration`Web安全性并`UserDetailsServiceAutoConfiguration`配置身份验证，这在非Web应用程序中也很重要。要完全关闭默认的Web应用程序安全性配置，可以添加一个类型的bean `WebFilterChainProxy`（这样做不会禁用`UserDetailsService`配置或执行器的安全性）。

要也关闭`UserDetailsService`配置，可以添加类型为`ReactiveUserDetailsService`或的Bean `ReactiveAuthenticationManager`。

可以通过添加自定义`SecurityWebFilterChain`bean 来配置访问规则以及使用多个Spring Security组件（例如OAuth 2 Client和Resource Server）。Spring Boot提供了便利的方法，可用于覆盖执行器端点和静态资源的访问规则。 `EndpointRequest`可用于创建`ServerWebExchangeMatcher`基于`management.endpoints.web.base-path`属性的。

`PathRequest`可用于`ServerWebExchangeMatcher`在常用位置创建for资源。

例如，您可以通过添加以下内容来自定义安全配置：

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    return http
        .authorizeExchange()
            .matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
            .pathMatchers("/foo", "/bar")
                .authenticated().and()
            .formLogin().and()
        .build();
}
```

### 9.3。OAuth2

[OAuth2](https://oauth.net/2/)是Spring支持的一种广泛使用的授权框架。

#### 9.3.1。客户

如果您`spring-security-oauth2-client`在类路径中，则可以利用一些自动配置功能来轻松设置OAuth2 / Open ID Connect客户端。此配置使用的属性`OAuth2ClientProperties`。相同的属性适用于servlet和反应式应用程序。

您可以在`spring.security.oauth2.client`前缀下注册多个OAuth2客户端和提供者，如以下示例所示：

```properties
spring.security.oauth2.client.registration.my-client-1.client-id=abcd
spring.security.oauth2.client.registration.my-client-1.client-secret=password
spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-1.scope=user
spring.security.oauth2.client.registration.my-client-1.redirect-uri=https://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code

spring.security.oauth2.client.registration.my-client-2.client-id=abcd
spring.security.oauth2.client.registration.my-client-2.client-secret=password
spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-2.scope=email
spring.security.oauth2.client.registration.my-client-2.redirect-uri=https://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code

spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=https://my-auth-server/oauth/authorize
spring.security.oauth2.client.provider.my-oauth-provider.token-uri=https://my-auth-server/oauth/token
spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=https://my-auth-server/userinfo
spring.security.oauth2.client.provider.my-oauth-provider.user-info-authentication-method=header
spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=https://my-auth-server/token_keys
spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name
```

对于支持[OpenID Connect发现的](https://openid.net/specs/openid-connect-discovery-1_0.html) OpenID Connect提供程序，可以进一步简化配置。提供者需要配置为，`issuer-uri`后者是它声明为其发布者标识符的URI。例如，如果`issuer-uri`提供的是“ https://example.com”，则将`OpenID Provider Configuration Request`对“ https://example.com/.well-known/openid-configuration”进行标记。结果预期为`OpenID Provider Configuration Response`。以下示例显示如何使用来配置OpenID Connect Provider `issuer-uri`：

```properties
spring.security.oauth2.client.provider.oidc-provider.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

默认情况下，Spring Security `OAuth2LoginAuthenticationFilter`只处理匹配的URL `/login/oauth2/code/*`。如果要定制`redirect-uri`来使用其他模式，则需要提供配置以处理该定制模式。例如，对于servlet应用程序，您可以添加`WebSecurityConfigurerAdapter`类似于以下内容的自己的应用程序：

```java
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .oauth2Login()
                .redirectionEndpoint()
                    .baseUri("/custom-callback");
    }
}
```

##### 普通提供商的OAuth2客户端注册

对于常见的OAuth2和OpenID提供商，包括谷歌，Github上，Facebook和1563，我们提供了一组供应商默认的（`google`，`github`，`facebook`，和`okta`，分别）。

如果不需要自定义这些提供程序，则可以将`provider`属性设置为需要为其推断默认值的属性。另外，如果用于客户端注册的密钥与默认支持的提供程序匹配，Spring Boot也会进行推断。

换句话说，以下示例中的两种配置都使用Google提供程序：

```properties
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```

#### 9.3.2。资源服务器

如果您`spring-security-oauth2-resource-server`在类路径中，则Spring Boot可以设置OAuth2资源服务器。对于JWT配置，需要指定JWK设置URI或OIDC颁发者URI，如以下示例所示：

```properties
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://example.com/oauth2/default/v1/keys
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

|      | 如果授权服务器不支持JWK设置URI，则可以使用用于验证JWT签名的公共密钥来配置资源服务器。可以使用`spring.security.oauth2.resourceserver.jwt.public-key-location`属性来完成此操作，该属性值需要指向包含PEM编码的x509格式的公钥的文件。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

相同的属性适用于servlet和反应式应用程序。

另外，您可以`JwtDecoder`为Servlet应用程序或`ReactiveJwtDecoder`响应式应用程序定义自己的bean 。

如果使用不透明令牌而不是JWT，则可以配置以下属性以通过自省来验证令牌：

```properties
spring.security.oauth2.resourceserver.opaquetoken.introspection-uri=https://example.com/check-token
spring.security.oauth2.resourceserver.opaquetoken.client-id=my-client-id
spring.security.oauth2.resourceserver.opaquetoken.client-secret=my-client-secret
```

同样，相同的属性适用于servlet和反应式应用程序。

另外，您可以`OpaqueTokenIntrospector`为Servlet应用程序或`ReactiveOpaqueTokenIntrospector`响应式应用程序定义自己的bean 。

#### 9.3.3。授权服务器

当前，Spring Security不提供实现OAuth 2.0授权服务器的支持。但是，[Spring Security OAuth](https://spring.io/projects/spring-security-oauth)项目提供了此功能，最终将被Spring Security完全取代。在此之前，您可以使用该`spring-security-oauth2-autoconfigure`模块轻松设置OAuth 2.0授权服务器；有关说明，请参见其[文档](https://docs.spring.io/spring-security-oauth2-boot)。

### 9.4。SAML 2.0

#### 9.4.1。依赖党

如果您`spring-security-saml2-service-provider`在类路径上，则可以利用一些自动配置功能来轻松设置SAML 2.0依赖方。此配置使用的属性`Saml2RelyingPartyProperties`。

依赖方注册表示身份提供者IDP和服务提供者SP之间的配对配置。您可以在`spring.security.saml2.relyingparty`前缀下注册多个依赖方，如以下示例所示：

```properties
spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].private-key-location=path-to-private-key
spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].certificate-location=path-to-certificate
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.verification.credentials[0].certificate-location=path-to-verification-cert
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.entity-id=remote-idp-entity-id1
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.sso-url=https://remoteidp1.sso.url

spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].private-key-location=path-to-private-key
spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].certificate-location=path-to-certificate
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.verification.credentials[0].certificate-location=path-to-other-verification-cert
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.entity-id=remote-idp-entity-id2
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.sso-url=https://remoteidp2.sso.url
```

### 9.5。执行器安全

为了安全起见，默认情况下，除`/health`和以外的所有执行器`/info`都是禁用的。该`management.endpoints.web.exposure.include`属性可用于启用执行器。

如果春季安全是在类路径上，并没有其他WebSecurityConfigurerAdapter存在，所有的执行机构以外`/health`，并`/info`通过春天开机自动配置固定。如果您定义一个custom `WebSecurityConfigurerAdapter`，Spring Boot自动配置将退出，您将完全控制执行器访问规则。

|      | 在设置之前`management.endpoints.web.exposure.include`，请确保裸露的执行器不包含敏感信息和/或通过将它们放置在防火墙后面或通过诸如Spring Security之类的东西进行保护。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 9.5.1。跨站点请求伪造保护

由于Spring Boot依赖于Spring Security的默认值，因此默认情况下CSRF保护是打开的。这意味着在使用默认安全性配置时，需要`POST`（关闭和记录器端点）的执行器端点`PUT`或`DELETE`将收到403禁止错误。

|      | 我们建议仅在创建非浏览器客户端使用的服务时完全禁用CSRF保护。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

关于CSRF保护的其他信息可以在[Spring Security Reference Guide中找到](https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/htmlsingle/#csrf)。

## 10.使用SQL数据库

在[Spring框架](https://spring.io/projects/spring-framework)提供了利用使用SQL数据库，直接JDBC访问广泛的支持`JdbcTemplate`来完成“对象关系映射”技术，比如Hibernate。 [Spring Data](https://spring.io/projects/spring-data)提供了更高级别的功能：`Repository`直接从接口创建实现，并使用约定从您的方法名称生成查询。

### 10.1。配置数据源

Java的 `javax.sql.DataSource`界面提供了使用数据库连接的标准方法。传统上，“数据源”使用`URL`以及一些凭据来建立数据库连接。

|      | 有关更多高级示例，请参见[“操作方法”部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-configure-a-datasource)，通常可以完全控制DataSource的配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 10.1.1。嵌入式数据库支持

通过使用内存嵌入式数据库来开发应用程序通常很方便。显然，内存数据库不提供持久存储。您需要在应用程序启动时填充数据库，并准备在应用程序结束时丢弃数据。

|      | “操作方法”部分包括一个 [有关如何初始化数据库的部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-database-initialization)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring Boot可以自动配置嵌入式[H2](https://www.h2database.com/)，[HSQL](http://hsqldb.org/)和[Derby](https://db.apache.org/derby/)数据库。您无需提供任何连接URL。您只需要包含要使用的嵌入式数据库的构建依赖项即可。

|      | 如果您在测试中使用此功能，则可能会注意到，无论使用多少应用程序上下文，整个测试套件都会重复使用同一数据库。如果要确保每个上下文都有一个单独的嵌入式数据库，则应设置`spring.datasource.generate-unique-name`为`true`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

例如，典型的POM依赖关系如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
```

|      | 您需要依赖`spring-jdbc`才能自动配置嵌入式数据库。在此示例中，它通过传递`spring-boot-starter-data-jpa`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果出于某种原因确实为嵌入式数据库配置了连接URL，请务必确保禁用了数据库的自动关闭功能。如果使用H2，则应使用H2 `DB_CLOSE_ON_EXIT=FALSE`。如果使用HSQLDB，则应确保`shutdown=true`未使用它。通过禁用数据库的自动关闭功能，Spring Boot可以控制何时关闭数据库，从而确保一旦不再需要访问数据库时就可以执行该操作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 10.1.2。连接到生产数据库

生产数据库连接也可以通过使用池自动配置 `DataSource`。Spring Boot使用以下算法来选择特定的实现：

1. 我们喜欢 [HikariCP](https://github.com/brettwooldridge/HikariCP)的性能和并发性。如果有HikariCP，我们总是选择它。
2. 否则，如果Tomcat池`DataSource`可用，我们将使用它。
3. 如果HikariCP和Tomcat池数据源均不可用，并且[Commons DBCP2](https://commons.apache.org/proper/commons-dbcp/)不可用，我们将使用它。

如果使用`spring-boot-starter-jdbc`或`spring-boot-starter-data-jpa`“启动器”，则会自动获得对的依赖`HikariCP`。

|      | 您可以完全绕过该算法，并通过设置`spring.datasource.type`属性来指定要使用的连接池。如果您在`tomcat-jdbc`默认情况下提供的Tomcat容器中运行应用程序，则这一点尤其重要。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 其他连接池始终可以手动配置。如果定义自己的`DataSource`bean，则不会进行自动配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

DataSource配置由中的外部配置属性控制`spring.datasource.*`。例如，您可以在中声明以下部分`application.properties`：

```properties
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

|      | 您至少应通过设置`spring.datasource.url`属性来指定URL 。否则，Spring Boot会尝试自动配置嵌入式数据库。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 您通常不需要指定`driver-class-name`，因为Spring Boot可以从推导大多数数据库`url`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 对于`DataSource`要创建的池，我们需要能够验证有效的`Driver`类是否可用，因此我们在进行任何操作之前都要进行检查。换句话说，如果您设置`spring.datasource.driver-class-name=com.mysql.jdbc.Driver`，则该类必须是可加载的。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

请参阅[`DataSourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceProperties.java)以获取更多受支持的选项。这些是不管实际实现如何都起作用的标准选项。它也有可能微调实现特定的设置，使用各自的前缀（`spring.datasource.hikari.*`，`spring.datasource.tomcat.*`，和`spring.datasource.dbcp2.*`）。有关更多详细信息，请参考所用连接池实现的文档。

例如，如果使用[Tomcat连接池](https://tomcat.apache.org/tomcat-9.0-doc/jdbc-pool.html#Common_Attributes)，则可以自定义许多其他设置，如以下示例所示：

```properties
# Number of ms to wait before throwing an exception if no connection is available.
spring.datasource.tomcat.max-wait=10000

# Maximum number of active connections that can be allocated from this pool at the same time.
spring.datasource.tomcat.max-active=50

# Validate the connection before borrowing it from the pool.
spring.datasource.tomcat.test-on-borrow=true
```

#### 10.1.3。连接到JNDI数据源

如果将Spring Boot应用程序部署到Application Server，则可能需要使用Application Server的内置功能来配置和管理DataSource，并使用JNDI对其进行访问。

该`spring.datasource.jndi-name`属性可以被用作一个替代`spring.datasource.url`，`spring.datasource.username`和`spring.datasource.password`属性来访问`DataSource`从一个特定的JNDI位置。例如，以下部分`application.properties`显示了如何访问定义的JBoss AS `DataSource`：

```properties
spring.datasource.jndi-name=java:jboss/datasources/customers
```

### 10.2。使用JdbcTemplate

Spring `JdbcTemplate`和`NamedParameterJdbcTemplate`class是自动配置的，您可以将`@Autowire`它们直接放入自己的bean中，如以下示例所示：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // ...

}
```

您可以使用属性来自定义模板的某些属性`spring.jdbc.template.*`，如以下示例所示：

```properties
spring.jdbc.template.max-rows=500
```

|      | 在`NamedParameterJdbcTemplate`重复使用相同的`JdbcTemplate`幕后情况。如果`JdbcTemplate`定义了多个，并且不存在主要候选对象，`NamedParameterJdbcTemplate`则不会自动配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 10.3。JPA和Spring Data JPA

Java Persistence API是一种标准技术，可让您将对象“映射”到关系数据库。该`spring-boot-starter-data-jpa`POM提供了上手的快捷方式。它提供以下关键依赖性：

- Hibernate：最流行的JPA实现之一。
- Spring Data JPA：使基于JPA的存储库的实现变得容易。
- Spring ORM：Spring Framework提供的核心ORM支持。

|      | 在这里，我们不会过多讨论JPA或[Spring Data](https://spring.io/projects/spring-data)。您可以按照[“访问数据与JPA”](https://spring.io/guides/gs/accessing-data-jpa/)从指导[spring.io](https://spring.io/)并宣读了[春天的数据JPA](https://spring.io/projects/spring-data-jpa)和[Hibernate的](https://hibernate.org/orm/documentation/)参考文档。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 10.3.1。实体类别

传统上，JPA“实体”类在`persistence.xml`文件中指定。在Spring Boot中，此文件不是必需的，而是使用“实体扫描”。默认情况下，将搜索主配置类（用`@EnableAutoConfiguration`或注释的一个`@SpringBootApplication`）下的所有软件包。

任何类别标注了`@Entity`，`@Embeddable`或者`@MappedSuperclass`被认为是。典型的实体类类似于以下示例：

```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String state;

    // ... additional members, often include @OneToMany mappings

    protected City() {
        // no-args constructor required by JPA spec
        // this one is protected since it shouldn't be used directly
    }

    public City(String name, String state) {
        this.name = name;
        this.state = state;
    }

    public String getName() {
        return this.name;
    }

    public String getState() {
        return this.state;
    }

    // ... etc

}
```

|      | 您可以使用`@EntityScan`注释来自定义实体扫描位置。请参阅“ [howto.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-separate-entity-definitions-from-spring-configuration) ”方法。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 10.3.2。Spring Data JPA存储库

[Spring Data JPA](https://spring.io/projects/spring-data-jpa)存储库是可以定义以访问数据的接口。JPA查询是根据您的方法名称自动创建的。例如，`CityRepository`接口可以声明一种`findAllByState(String state)`方法来查找给定州的所有城市。

对于更复杂的查询，您可以使用Spring Data的[`Query`](https://docs.spring.io/spring-data/jpa/docs/2.2.6.RELEASE/api/org/springframework/data/jpa/repository/Query.html)注释对方法进行注释。

Spring Data存储库通常从[`Repository`](https://docs.spring.io/spring-data/commons/docs/2.2.6.RELEASE/api/org/springframework/data/repository/Repository.html)或[`CrudRepository`](https://docs.spring.io/spring-data/commons/docs/2.2.6.RELEASE/api/org/springframework/data/repository/CrudRepository.html)接口扩展。如果使用自动配置，则从包含主配置类（用`@EnableAutoConfiguration`或注释的主配置类）的程序包中搜索存储库`@SpringBootApplication`。

以下示例显示了典型的Spring Data存储库接口定义：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

Spring Data JPA存储库支持三种不同的引导模式：默认，延迟和延迟。要启用延迟引导或延迟引导，请将`spring.data.jpa.repositories.bootstrap-mode`属性分别设置为`deferred`或`lazy`。当使用延迟或延迟启动时，自动配置`EntityManagerFactoryBuilder`将使用上下文的`AsyncTaskExecutor`（如果有）作为引导执行器。如果存在多个，`applicationTaskExecutor`将使用一个命名的。

|      | 我们几乎没有刮过Spring Data JPA的表面。有关完整的详细信息，请参见[Spring Data JPA参考文档](https://docs.spring.io/spring-data/jdbc/docs/1.1.6.RELEASE/reference/html/)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 10.3.3。创建和删除JPA数据库

默认情况下，**仅**当您使用嵌入式数据库（H2，HSQL或Derby）时，**才会**自动创建JPA数据库。您可以使用`spring.jpa.*`属性来显式配置JPA设置。例如，要创建和删除表，可以将以下行添加到中`application.properties`：

```
spring.jpa.hibernate.ddl-auto =创建-放置
```

|      | Hibernate自己的内部属性名称是（如果您记得更好的话）是`hibernate.hbm2ddl.auto`。您可以通过使用`spring.jpa.properties.*`（与其他Hibernate本机属性一起）进行设置（将前缀添加到实体管理器之前先删除前缀）。下面的行显示了为Hibernate设置JPA属性的示例： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```
spring.jpa.properties.hibernate.globally_quoted_identifiers = true
```

在前面的示例中的线穿过的一个值`true`的`hibernate.globally_quoted_identifiers`属性设置为休眠实体管理器。

默认情况下，DDL执行（或验证）推迟到`ApplicationContext`开始执行之前。还有一个`spring.jpa.generate-ddl`标志，但是如果激活了Hibernate自动配置，则不会使用它，因为`ddl-auto`设置更细粒度。

#### 10.3.4。在视图中打开EntityManager

如果您正在运行Web应用程序，则默认情况下，Spring Boot注册[`OpenEntityManagerInViewInterceptor`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html)以应用“在视图中打开EntityManager”模式，以允许在Web视图中进行延迟加载。如果你不希望这种行为，你应该设置`spring.jpa.open-in-view`到`false`你`application.properties`。

### 10.4。Spring Data JDBC

Spring Data包括对JDBC的存储库支持，并将自动为上的方法生成SQL `CrudRepository`。对于更高级的查询，`@Query`提供了注释。

当必要的依赖项在类路径上时，Spring Boot将自动配置Spring Data的JDBC存储库。可以将它们添加到您的项目中，而只需依赖于`spring-boot-starter-data-jdbc`。如有必要，您可以通过向应用程序中添加`@EnableJdbcRepositories`注释或`JdbcConfiguration`子类来控制Spring Data JDBC的配置。

|      | 有关Spring Data JDBC的完整详细信息，请参考[参考文档](https://docs.spring.io/spring-data/jdbc/docs/1.1.6.RELEASE/reference/html/)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 10.5。使用H2的Web控制台

该[H2数据库](https://www.h2database.com/)提供了一个[基于浏览器的控制台](https://www.h2database.com/html/quickstart.html#h2_console)是春天开机都能自动为您配置。满足以下条件时，将自动配置控制台：

- 您正在开发基于servlet的Web应用程序。
- `com.h2database:h2` 在类路径上。
- 您正在使用[Spring Boot的开发人员工具](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools)。

|      | 如果您不使用Spring Boot的开发人员工具，但仍想使用H2的控制台，则可以将`spring.h2.console.enabled`属性配置为`true`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | H2控制台仅在开发期间使用，因此您应注意确保`spring.h2.console.enabled`不在`true`生产环境中使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 10.5.1。更改H2控制台的路径

默认情况下，控制台位于`/h2-console`。您可以使用`spring.h2.console.path`属性来自定义控制台的路径。

### 10.6。使用jOOQ

jOOQ面向对象查询（[jOOQ](https://www.jooq.org/)）是[Data Geekery](https://www.datageekery.com/)的流行产品，它可以从数据库中生成Java代码，并允许您通过其流畅的API来构建类型安全的SQL查询。商业版和开源版都可以与Spring Boot一起使用。

#### 10.6.1。代码生成

为了使用jOOQ类型安全查询，您需要从数据库架构中生成Java类。您可以按照[jOOQ用户手册中](https://www.jooq.org/doc/3.12.4/manual-single-page/#jooq-in-7-steps-step3)的说明进行[操作](https://www.jooq.org/doc/3.12.4/manual-single-page/#jooq-in-7-steps-step3)。如果您使用`jooq-codegen-maven`插件并且还使用`spring-boot-starter-parent`“父POM”，则可以安全地省略插件的``标签。您还可以使用Spring Boot定义的版本变量（例如`h2.version`）来声明插件的数据库依赖关系。以下清单显示了一个示例：

```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <executions>
        ...
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency>
    </dependencies>
    <configuration>
        <jdbc>
            <driver>org.h2.Driver</driver>
            <url>jdbc:h2:~/yourdatabase</url>
        </jdbc>
        <generator>
            ...
        </generator>
    </configuration>
</plugin>
```

#### 10.6.2。使用DSLContext

jOOQ提供的流畅的API通过该`org.jooq.DSLContext`接口启动。Spring Boot将a自动配置`DSLContext`为Spring Bean并将其连接到您的应用程序`DataSource`。要使用`DSLContext`，可以使用`@Autowire`它，如以下示例所示：

```java
@Component
public class JooqExample implements CommandLineRunner {

    private final DSLContext create;

    @Autowired
    public JooqExample(DSLContext dslContext) {
        this.create = dslContext;
    }

}
```

|      | jOOQ手册倾向于使用名为的变量`create`来保存`DSLContext`。 |
| ---- | -------------------------------------------------------- |
|      |                                                          |

然后，您可以使用`DSLContext`构造查询，如以下示例所示：

```java
public List<GregorianCalendar> authorsBornAfter1980() {
    return this.create.selectFrom(AUTHOR)
        .where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
        .fetch(AUTHOR.DATE_OF_BIRTH);
}
```

#### 10.6.3。jOOQ SQL方言

除非`spring.jooq.sql-dialect`配置了该属性，否则Spring Boot会确定要用于您的数据源的SQL语言。如果Spring Boot无法检测到方言，则使用`DEFAULT`。

|      | Spring Boot只能自动配置开源版本的jOOQ支持的方言。 |
| ---- | ------------------------------------------------- |
|      |                                                   |

#### 10.6.4。定制jOOQ

通过定义自己的`@Bean`定义（可以在`Configuration`创建jOOQ时使用）可以实现更高级的自定义。您可以为以下jOOQ类型定义bean：

- `ConnectionProvider`
- `ExecutorProvider`
- `TransactionProvider`
- `RecordMapperProvider`
- `RecordUnmapperProvider`
- `Settings`
- `RecordListenerProvider`
- `ExecuteListenerProvider`
- `VisitListenerProvider`
- `TransactionListenerProvider`

`org.jooq.Configuration` `@Bean`如果要完全控制jOOQ配置，也可以创建自己的。

## 11.使用NoSQL技术

Spring Data提供了其他项目来帮助您访问各种NoSQL技术，包括：

- [MongoDB](https://spring.io/projects/spring-data-mongodb)
- [Neo4J](https://spring.io/projects/spring-data-neo4j)
- [弹性搜索](https://spring.io/projects/spring-data-elasticsearch)
- [索尔](https://spring.io/projects/spring-data-solr)
- [雷迪斯](https://spring.io/projects/spring-data-redis)
- [GemFire](https://spring.io/projects/spring-data-gemfire)或[Geode](https://spring.io/projects/spring-data-geode)
- [卡桑德拉](https://spring.io/projects/spring-data-cassandra)
- [Couchbase](https://spring.io/projects/spring-data-couchbase)
- [LDAP](https://spring.io/projects/spring-data-ldap)

Spring Boot为Redis，MongoDB，Neo4j，Elasticsearch，Solr Cassandra，Couchbase和LDAP提供自动配置。您可以使用其他项目，但是必须自己配置它们。请参阅[spring.io/projects/spring-data](https://spring.io/projects/spring-data)上的相应参考文档。

### 11.1。雷迪斯

[Redis](https://redis.io/)是一个缓存，消息代理和功能丰富的键值存储。Spring Boot为[Lettuce](https://github.com/lettuce-io/lettuce-core/)和[Jedis](https://github.com/xetorthio/jedis/)客户端库以及[Spring Data Redis](https://github.com/spring-projects/spring-data-redis)提供的最基本的抽象提供了基本的自动配置。

有一个`spring-boot-starter-data-redis`“启动器”可以方便地收集依赖关系。默认情况下，它使用[Lettuce](https://github.com/lettuce-io/lettuce-core/)。该启动程序可以处理传统应用程序和响应式应用程序。

|      | 我们还提供了一个`spring-boot-starter-data-redis-reactive`“入门级”软件，可与其他商店保持一致，并提供响应性支持。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 11.1.1。连接到Redis

您可以像插入其他任何Spring Bean一样注入自动配置的`RedisConnectionFactory`，`StringRedisTemplate`或Vanilla `RedisTemplate`实例。默认情况下，该实例尝试在处连接到Redis服务器`localhost:6379`。以下清单显示了这种Bean的示例：

```java
@Component
public class MyBean {

    private StringRedisTemplate template;

    @Autowired
    public MyBean(StringRedisTemplate template) {
        this.template = template;
    }

    // ...

}
```

|      | 您还可以注册任意数量的Bean，以实现`LettuceClientConfigurationBuilderCustomizer`更高级的自定义。如果您使用Jedis，`JedisClientConfigurationBuilderCustomizer`也可以使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果添加自己`@Bean`的任何一种自动配置类型，它将替换默认类型（除非是`RedisTemplate`，当排除基于Bean名称`redisTemplate`而不是其类型时为）。默认情况下，如果`commons-pool2`在类路径上，则会得到一个池化连接工厂。

### 11.2。MongoDB

[MongoDB](https://www.mongodb.com/)是一个开放源代码NoSQL文档数据库，它使用类似JSON的架构而不是传统的基于表的关系数据。Spring Boot为MongoDB的使用提供了许多便利，包括`spring-boot-starter-data-mongodb`和`spring-boot-starter-data-mongodb-reactive`Starters。

#### 11.2.1。连接到MongoDB数据库

要访问Mongo数据库，您可以注入自动配置的`org.springframework.data.mongodb.MongoDbFactory`。默认情况下，该实例尝试通过连接到MongoDB服务器`mongodb://localhost/test`。以下示例显示了如何连接到MongoDB数据库：

```java
import org.springframework.data.mongodb.MongoDbFactory;
import com.mongodb.DB;

@Component
public class MyBean {

    private final MongoDbFactory mongo;

    @Autowired
    public MyBean(MongoDbFactory mongo) {
        this.mongo = mongo;
    }

    // ...

    public void example() {
        DB db = mongo.getDb();
        // ...
    }

}
```

您可以设置`spring.data.mongodb.uri`属性来更改URL并配置其他设置，例如*副本集*，如以下示例所示：

```properties
spring.data.mongodb.uri=mongodb://user:secret@mongo1.example.com:12345,mongo2.example.com:23456/test
```

另外，只要您使用Mongo 2.x，就可以指定`host`/ `port`。例如，您可以在中声明以下设置`application.properties`：

```properties
spring.data.mongodb.host=mongoserver
spring.data.mongodb.port=27017
```

如果定义了自己的`MongoClient`，它将用于自动配置合适的`MongoDbFactory`。这两个`com.mongodb.MongoClient`和`com.mongodb.client.MongoClient`支持。

|      | 如果使用蒙戈3.0 Java驱动程序，`spring.data.mongodb.host`并且`spring.data.mongodb.port`不支持。在这种情况下，`spring.data.mongodb.uri`应使用它来提供所有配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果`spring.data.mongodb.port`未指定，`27017`则使用默认值。您可以从前面显示的示例中删除此行。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您不使用Spring Data Mongo，则可以注入`com.mongodb.MongoClient`bean而不是使用`MongoDbFactory`。如果要完全控制建立MongoDB连接的方式，也可以声明自己的`MongoDbFactory`或`MongoClient`bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您使用反应式驱动程序，则SSL需要Netty。如果可以使用Netty并且尚未自定义要使用的工厂，则自动配置会自动配置该工厂。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 11.2.2。MongoTemplate

[Spring Data MongoDB](https://spring.io/projects/spring-data-mongodb)提供了一个[`MongoTemplate`](https://docs.spring.io/spring-data/mongodb/docs/2.2.6.RELEASE/api/org/springframework/data/mongodb/core/MongoTemplate.html)与Spring的设计非常相似的类`JdbcTemplate`。与一样`JdbcTemplate`，Spring Boot为您自动配置一个Bean来注入模板，如下所示：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final MongoTemplate mongoTemplate;

    @Autowired
    public MyBean(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    // ...

}
```

有关完整的详细信息，请参见[`MongoOperations`Javadoc](https://docs.spring.io/spring-data/mongodb/docs/2.2.6.RELEASE/api/org/springframework/data/mongodb/core/MongoOperations.html)。

#### 11.2.3。Spring Data MongoDB存储库

Spring Data包括对MongoDB的存储库支持。与前面讨论的JPA存储库一样，基本原理是根据方法名称自动构造查询。

实际上，Spring Data JPA和Spring Data MongoDB共享相同的通用基础架构。您可以从以前的JPA示例开始，并假设它`City`现在是Mongo数据类而不是JPA `@Entity`，它的工作方式相同，如以下示例所示：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

|      | 您可以使用`@EntityScan`注释来自定义文档扫描位置。 |
| ---- | ------------------------------------------------- |
|      |                                                   |

|      | 有关Spring Data MongoDB的完整详细信息，包括其丰富的对象映射技术，请参阅其[参考文档](https://spring.io/projects/spring-data-mongodb)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 11.2.4。嵌入式Mongo

Spring Boot为[Embedded Mongo](https://github.com/flapdoodle-oss/de.flapdoodle.embed.mongo)提供自动配置。要在Spring Boot应用程序中使用它，请添加对的依赖`de.flapdoodle.embed:de.flapdoodle.embed.mongo`。

可以通过设置`spring.data.mongodb.port`属性来配置Mongo侦听的端口。要使用随机分配的空闲端口，请使用值0。created `MongoClient`by `MongoAutoConfiguration`被自动配置为使用随机分配的端口。

|      | 如果未配置自定义端口，则默认情况下，嵌入式支持会使用随机端口（而不是27017）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果类路径上有SLF4J，则Mongo产生的输出将自动路由到名为的记录器`org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongo`。

您可以声明自己的bean `IMongodConfig`和`IRuntimeConfig`bean，以控制Mongo实例的配置和日志记录路由。可以通过声明`DownloadConfigBuilderCustomizer`bean 来定制下载配置。

### 11.3。Neo4j

[Neo4j](https://neo4j.com/)是一个开放源代码的NoSQL图形数据库，它使用通过一级关系连接的节点的丰富数据模型，比传统的RDBMS方法更适合于连接大数据。Spring Boot提供了使用Neo4j的一些便利，包括`spring-boot-starter-data-neo4j`“ Starter”。

#### 11.3.1。连接到Neo4j数据库

要访问Neo4j服务器，您可以注入自动配置的`org.neo4j.ogm.session.Session`。默认情况下，实例尝试`localhost:7687`使用Bolt协议连接到Neo4j服务器。以下示例显示如何注入Neo4j `Session`：

```java
@Component
public class MyBean {

    private final Session session;

    @Autowired
    public MyBean(Session session) {
        this.session = session;
    }

    // ...

}
```

您可以通过设置`spring.data.neo4j.*`属性来配置要使用的uri和凭据，如以下示例所示：

```properties
spring.data.neo4j.uri=bolt://my-server:7687
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=secret
```

您可以通过添加`org.neo4j.ogm.config.Configuration`bean或`org.neo4j.ogm.session.SessionFactory`bean 来完全控制会话的创建。

#### 11.3.2。使用嵌入式模式

如果添加`org.neo4j:neo4j-ogm-embedded-driver`到应用程序的依赖项，Spring Boot会自动配置一个Neo4j的进程内嵌入式实例，该实例在应用程序关闭时不会保留任何数据。

|      | 由于嵌入式Neo4j OGM驱动程序本身不提供Neo4j内核，因此您必须自己声明`org.neo4j:neo4j`为依赖项。有关兼容版本的列表，请参阅[Neo4j OGM文档](https://neo4j.com/docs/ogm-manual/current/reference/#reference:getting-started)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

当类路径上有多个驱动程序时，嵌入式驱动程序优先于其他驱动程序。您可以通过设置显式禁用嵌入式模式`spring.data.neo4j.embedded.enabled=false`。

如果嵌入式驱动程序和Neo4j内核位于上述类路径上，则[数据Neo4j测试会](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-neo4j-test)自动使用嵌入式Neo4j实例。

|      | 您可以通过在配置中提供数据库文件的路径来启用嵌入式模式的持久性`spring.data.neo4j.uri=file://var/tmp/graph.db`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 11.3.3。使用本机类型

Neo4j-OGM可以将某些类型（例如中的类型）映射`java.time.*`到`String`基于属性或Neo4j提供的本机类型之一。出于向后兼容的原因，Neo4j-OGM的默认设置是使用`String`基于-的表示形式。要使用本机类型，请在`org.neo4j:neo4j-ogm-bolt-native-types`或上添加依赖项`org.neo4j:neo4j-ogm-embedded-native-types`，并配置`spring.data.neo4j.use-native-types`属性，如以下示例所示：

```properties
spring.data.neo4j.use-native-types=true
```

#### 11.3.4。Neo4jSession

默认情况下，如果您正在运行Web应用程序，则会话将绑定到线程以进行请求的整个处理（即，它使用“在视图中打开会话”模式）。如果您不希望出现这种情况，请在`application.properties`文件中添加以下行：

```properties
spring.data.neo4j.open-in-view=false
```

#### 11.3.5。Spring Data Neo4j存储库

Spring Data包括对Neo4j的存储库支持。

Spring Data Neo4j与许多其他Spring Data模块一样，与Spring Data JPA共享公共基础结构。您可以从以前的JPA示例开始，将其定义`City`为Neo4j OGM `@NodeEntity`而不是JPA `@Entity`，并且存储库抽象以相同的方式工作，如以下示例所示：

```java
package com.example.myapp.domain;

import java.util.Optional;

import org.springframework.data.neo4j.repository.*;

public interface CityRepository extends Neo4jRepository<City, Long> {

    Optional<City> findOneByNameAndState(String name, String state);

}
```

在`spring-boot-starter-data-neo4j`“入门”使仓库的支持以及事务管理。您可以通过在-bean 上分别使用`@EnableNeo4jRepositories`和来定制位置以查找存储库和实体。`@EntityScan``@Configuration`

|      | 有关Spring Data Neo4j的完整详细信息，包括其对象映射技术，请参阅[参考文档](https://docs.spring.io/spring-data/neo4j/docs/5.2.6.RELEASE/reference/html/)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 11.4。索尔

[Apache Solr](https://lucene.apache.org/solr/)是一个搜索引擎。Spring Boot为Solr 5客户端库提供了基本的自动配置，并由[Spring Data Solr](https://github.com/spring-projects/spring-data-solr)在其之上提供了抽象。有一个`spring-boot-starter-data-solr`“启动器”可以方便地收集依赖关系。

#### 11.4.1。连接到Solr

您可以像插入`SolrClient`其他任何Spring Bean一样注入自动配置的实例。默认情况下，该实例尝试连接到的服务器`localhost:8983/solr`。下面的示例显示如何注入Solr bean：

```java
@Component
public class MyBean {

    private SolrClient solr;

    @Autowired
    public MyBean(SolrClient solr) {
        this.solr = solr;
    }

    // ...

}
```

如果添加自己`@Bean`的type `SolrClient`，它将替换默认值。

#### 11.4.2。Spring Data Solr存储库

Spring Data包括对Apache Solr的存储库支持。与前面讨论的JPA存储库一样，基本原理是根据方法名称自动为您构建查询。

实际上，Spring Data JPA和Spring Data Solr共享相同的通用基础结构。您可以从以前的JPA示例开始，假设`City`现在是一个`@SolrDocument`类，而不是JPA `@Entity`，它的工作方式相同。

IP：有关Spring Data Solr的完整详细信息，请[参考参考文档](https://docs.spring.io/spring-data/solr/docs/4.1.6.RELEASE/reference/html/)。

### 11.5。弹性搜索

[Elasticsearch](https://www.elastic.co/products/elasticsearch)是一个开源，分布式，RESTful搜索和分析引擎。Spring Boot为Elasticsearch提供了基本的自动配置。

Spring Boot支持多个客户端：

- 官方Java“低级”和“高级” REST客户端
- 在`ReactiveElasticsearchClient`由Spring数据Elasticsearch提供

传输客户端仍然可用，但是[Spring Data Elasticsearch](https://github.com/spring-projects/spring-data-elasticsearch)和Elasticsearch本身已弃用了它的支持。它将在将来的版本中删除。Spring Boot提供了专用的“入门” `spring-boot-starter-data-elasticsearch`。

该[玩笑](https://github.com/searchbox-io/Jest)客户端已被弃用，以及，因为Elasticsearch和Spring数据Elasticsearch都提供REST客户端官方支持。

#### 11.5.1。使用REST客户端连接到Elasticsearch

Elasticsearch附带了[两个](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)可用于查询集群的[REST客户端](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)：“低级”客户端和“高级”客户端。

如果您`org.elasticsearch.client:elasticsearch-rest-client`对类路径有依赖性，Spring Boot将自动配置并注册一个`RestClient`默认为target 的bean `localhost:9200`。您可以进一步调整`RestClient`配置方式，如以下示例所示：

```properties
spring.elasticsearch.rest.uris=https://search.example.com:9200
spring.elasticsearch.rest.read-timeout=10s
spring.elasticsearch.rest.username=user
spring.elasticsearch.rest.password=secret
```

您还可以注册任意数量的Bean，以实现`RestClientBuilderCustomizer`更高级的自定义。要完全控制注册，请定义一个`RestClient`bean。

如果您`org.elasticsearch.client:elasticsearch-rest-high-level-client`对类路径有依赖性，Spring Boot将自动配置一个`RestHighLevelClient`，包装现有的`RestClient`bean，重新使用其HTTP配置。

#### 11.5.2。使用反应式REST客户端连接到Elasticsearch

[春季数据Elasticsearch](https://spring.io/projects/spring-data-elasticsearch)船舶`ReactiveElasticsearchClient`在反应方式查询Elasticsearch实例。它建立在WebFlux的基础上`WebClient`，因此`spring-boot-starter-elasticsearch`和`spring-boot-starter-webflux`依赖项对于启用此支持都是有用的。

默认情况下，Spring Boot将自动配置并注册一个`ReactiveElasticsearchClient` target 的bean `localhost:9200`。您可以进一步调整其配置，如以下示例所示：

```properties
spring.data.elasticsearch.client.reactive.endpoints=search.example.com:9200
spring.data.elasticsearch.client.reactive.use-ssl=true
spring.data.elasticsearch.client.reactive.socket-timeout=10s
spring.data.elasticsearch.client.reactive.username=user
spring.data.elasticsearch.client.reactive.password=secret
```

如果配置属性不够，并且您想完全控制客户端配置，则可以注册一个自定义`ClientConfiguration`bean。

#### 11.5.3。使用Jest连接到Elasticsearch

现在，Spring Boot支持官方`RestHighLevelClient`支持，不推荐使用Jest支持。

如果您`Jest`在类路径上，则可以注入自动配置`JestClient`的对象，默认情况下为target `localhost:9200`。您可以进一步调整客户端的配置方式，如以下示例所示：

```properties
spring.elasticsearch.jest.uris=https://search.example.com:9200
spring.elasticsearch.jest.read-timeout=10000
spring.elasticsearch.jest.username=user
spring.elasticsearch.jest.password=secret
```

您还可以注册任意数量的Bean，以实现`HttpClientConfigBuilderCustomizer`更高级的自定义。以下示例调整其他HTTP设置：

```java
static class HttpSettingsCustomizer implements HttpClientConfigBuilderCustomizer {

    @Override
    public void customize(HttpClientConfig.Builder builder) {
        builder.maxTotalConnection(100).defaultMaxTotalConnectionPerRoute(5);
    }

}
```

要完全控制注册，请定义一个`JestClient`bean。

#### 11.5.4。使用Spring数据连接到Elasticsearch

要连接到Elasticsearch，`RestHighLevelClient`必须定义一个bean，它由Spring Boot自动配置或由应用程序手动提供（请参阅前面的部分）。有了此配置后，`ElasticsearchRestTemplate`可以像其他任何Spring bean一样注入an ，如以下示例所示：

```java
@Component
public class MyBean {

    private final ElasticsearchRestTemplate template;

    public MyBean(ElasticsearchRestTemplate template) {
        this.template = template;
    }

    // ...

}
```

在存在`spring-data-elasticsearch`和使用`WebClient`（通常为`spring-boot-starter-webflux`）所需的依赖关系的情况下，Spring Boot还可以自动配置[ReactiveElasticsearchClient](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-connecting-to-elasticsearch-reactive-rest)和`ReactiveElasticsearchTemplate`as as bean。它们与其他REST客户端是等效的。

#### 11.5.5。Spring Data Elasticsearch存储库

Spring Data包括对Elasticsearch的存储库支持。与前面讨论的JPA存储库一样，基本原理是根据方法名称自动为您构造查询。

实际上，Spring Data JPA和Spring Data Elasticsearch共享相同的通用基础架构。您可以从以前的JPA示例开始，假设`City`现在是Elasticsearch `@Document`类而不是JPA `@Entity`，它的工作方式相同。

|      | 有关Spring Data Elasticsearch的完整详细信息，请[参考参考文档](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring Boot使用`ElasticsearchRestTemplate`或`ReactiveElasticsearchTemplate`Bean 支持经典和反应式Elasticsearch存储库。给定所需的依赖项，这些bean最有可能由Spring Boot自动配置。

如果您希望使用自己的模板来支持Elasticsearch存储库，则可以添加自己的`ElasticsearchRestTemplate`或`ElasticsearchOperations` `@Bean`（只要命名为）`"elasticsearchTemplate"`。Bean名称也适用于`ReactiveElasticsearchTemplate`和。`ReactiveElasticsearchOperations``"reactiveElasticsearchTemplate"`

您可以选择使用以下属性禁用存储库支持：

```properties
spring.data.elasticsearch.repositories.enabled=false
```

### 11.6。卡桑德拉

[Cassandra](https://cassandra.apache.org/)是一个开放源代码的分布式数据库管理系统，旨在处理许多商用服务器上的大量数据。Spring Boot为Cassandra提供自动配置，并由[Spring Data Cassandra](https://github.com/spring-projects/spring-data-cassandra)在其之上提供抽象。有一个`spring-boot-starter-data-cassandra`“启动器”可以方便地收集依赖关系。

#### 11.6.1。连接到Cassandra

您可以像使用其他任何Spring Bean一样注入自动配置`CassandraTemplate`的`Session`实例或Cassandra 实例。这些`spring.data.cassandra.*`属性可用于自定义连接。通常，您提供`keyspace-name`和`contact-points`属性，如以下示例所示：

```properties
spring.data.cassandra.keyspace-name=mykeyspace
spring.data.cassandra.contact-points=cassandrahost1,cassandrahost2
```

您还可以注册任意数量的Bean，以实现`ClusterBuilderCustomizer`更高级的自定义。

以下代码清单显示了如何注入Cassandra bean：

```java
@Component
public class MyBean {

    private CassandraTemplate template;

    public MyBean(CassandraTemplate template) {
        this.template = template;
    }

    // ...

}
```

如果添加自己`@Bean`的type `CassandraTemplate`，它将替换默认值。

#### 11.6.2。Spring Data Cassandra存储库

Spring Data包括对Cassandra的基本存储库支持。当前，此功能比前面讨论的JPA存储库更受限制，需要使用来注释finder方法`@Query`。

|      | 有关Spring Data Cassandra的完整详细信息，请[参考参考文档](https://docs.spring.io/spring-data/cassandra/docs/)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 11.7。Couchbase

[Couchbase](https://www.couchbase.com/)是一个开放源代码，分布式，多模型的NoSQL面向文档的数据库，已针对交互式应用程序进行了优化。Spring Boot为Couchbase提供自动配置，并由[Spring Data Couchbase](https://github.com/spring-projects/spring-data-couchbase)在其之上提供抽象。有`spring-boot-starter-data-couchbase`和`spring-boot-starter-data-couchbase-reactive`“入门者”可以方便地收集依赖项。

#### 11.7.1。连接到Couchbase

您可以通过添加Couchbase SDK和一些配置来获得`Bucket`和`Cluster`。这些`spring.couchbase.*`属性可用于自定义连接。通常，您提供引导主机，存储桶名称和密码，如以下示例所示：

```properties
spring.couchbase.bootstrap-hosts=my-host-1,192.168.1.123
spring.couchbase.bucket.name=my-bucket
spring.couchbase.bucket.password=secret
```

|      | 您*至少*需要提供引导主机，在这种情况下，存储桶名称为`default`，密码为空字符串。另外，您可以定义自己的名称`org.springframework.data.couchbase.config.CouchbaseConfigurer` `@Bean`来控制整个配置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

也可以自定义某些`CouchbaseEnvironment`设置。例如，以下配置更改了用于打开新`Bucket`邮件并启用SSL支持的超时：

```properties
spring.couchbase.env.timeouts.connect=3000
spring.couchbase.env.ssl.key-store=/location/of/keystore.jks
spring.couchbase.env.ssl.key-store-password=secret
```

检查`spring.couchbase.env.*`属性以获取更多详细信息。

#### 11.7.2。Spring Data Couchbase存储库

Spring Data包括对Couchbase的存储库支持。有关Spring Data Couchbase的完整详细信息，请[参考参考文档](https://docs.spring.io/spring-data/couchbase/docs/current/reference/html/)。

您可以`CouchbaseTemplate`像使用任何其他Spring Bean一样注入自动配置的实例，前提是可以使用*默认值* `CouchbaseConfigurer`（如前所述，启用Couchbase支持时会发生这种情况）。

以下示例显示了如何注入Couchbase bean：

```java
@Component
public class MyBean {

    private final CouchbaseTemplate template;

    @Autowired
    public MyBean(CouchbaseTemplate template) {
        this.template = template;
    }

    // ...

}
```

您可以在自己的配置中定义一些Bean，以覆盖自动配置提供的那些：

- 一个`CouchbaseTemplate` `@Bean`用的名称`couchbaseTemplate`。
- 一个`IndexManager` `@Bean`用的名称`couchbaseIndexManager`。
- 一个`CustomConversions` `@Bean`用的名称`couchbaseCustomConversions`。

为了避免在您自己的配置中对这些名称进行硬编码，您可以重复使用`BeanNames`Spring Data Couchbase提供的名称。例如，您可以自定义要使用的转换器，如下所示：

```java
@Configuration(proxyBeanMethods = false)
public class SomeConfiguration {

    @Bean(BeanNames.COUCHBASE_CUSTOM_CONVERSIONS)
    public CustomConversions myCustomConversions() {
        return new CustomConversions(...);
    }

    // ...

}
```

|      | 如果您想完全绕过Spring Data Couchbase的自动配置，请提供您自己的实现`org.springframework.data.couchbase.config.AbstractCouchbaseDataConfiguration`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 11.8。LDAP

[LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)（轻型目录访问协议）是一种开放的，与供应商无关的行业标准应用协议，用于通过IP网络访问和维护分布式目录信息服务。Spring Boot为任何兼容的LDAP服务器提供自动配置，并从[UnboundID](https://www.ldap.com/unboundid-ldap-sdk-for-java)支持嵌入式内存LDAP服务器。

LDAP抽象由[Spring Data LDAP提供](https://github.com/spring-projects/spring-data-ldap)。有一个`spring-boot-starter-data-ldap`“启动器”可以方便地收集依赖关系。

#### 11.8.1。连接到LDAP服务器

要连接到LDAP服务器，请确保您声明对`spring-boot-starter-data-ldap`“启动器” 的依赖，或者`spring-ldap-core`然后在application.properties中声明服务器的URL，如以下示例所示：

```properties
spring.ldap.urls=ldap://myserver:1235
spring.ldap.username=admin
spring.ldap.password=secret
```

如果需要自定义连接设置，可以使用`spring.ldap.base`和`spring.ldap.base-environment`属性。

将`LdapContextSource`根据这些设置自动配置。如果您需要对其进行自定义（例如使用）`PooledContextSource`，则仍可以注入自动配置`LdapContextSource`。确保将您的定制标记`ContextSource`为，`@Primary`以便自动配置`LdapTemplate`使用它。

#### 11.8.2。Spring Data LDAP存储库

Spring Data包括对LDAP的存储库支持。有关Spring Data LDAP的完整详细信息，请[参考参考文档](https://docs.spring.io/spring-data/ldap/docs/1.0.x/reference/html/)。

您还可以`LdapTemplate`像使用其他任何Spring Bean一样注入自动配置的实例，如以下示例所示：

```java
@Component
public class MyBean {

    private final LdapTemplate template;

    @Autowired
    public MyBean(LdapTemplate template) {
        this.template = template;
    }

    // ...

}
```

#### 11.8.3。嵌入式内存LDAP服务器

出于测试目的，Spring Boot支持从[UnboundID](https://ldap.com/unboundid-ldap-sdk-for-java/)自动配置内存中的LDAP服务器。要配置服务器，请向其添加依赖项`com.unboundid:unboundid-ldapsdk`并声明一个`spring.ldap.embedded.base-dn`属性，如下所示：

```properties
spring.ldap.embedded.base-dn=dc=spring,dc=io
```

|      | 可以定义多个base-dn值，但是，由于可分辨的名称通常包含逗号，因此必须使用正确的符号来定义它们。在yaml文件中，您可以使用yaml列表符号：`spring.ldap.embedded.base-dn:  - dc=spring,dc=io  - dc=pivotal,dc=io`在属性文件中，必须将索引包括在属性名称中：`spring.ldap.embedded.base-dn[0]=dc=spring,dc=io spring.ldap.embedded.base-dn[1]=dc=pivotal,dc=io` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认情况下，服务器在随机端口上启动并触发常规LDAP支持。无需指定`spring.ldap.urls`属性。

如果`schema.ldif`您的类路径中有一个文件，它将用于初始化服务器。如果要从其他资源加载初始化脚本，则也可以使用该`spring.ldap.embedded.ldif`属性。

默认情况下，使用标准架构来验证`LDIF`文件。您可以通过设置`spring.ldap.embedded.validation.enabled`属性来完全关闭验证。如果您具有自定义属性，则可以`spring.ldap.embedded.validation.schema`用来定义自定义属性类型或对象类。

### 11.9。InfluxDB

[InfluxDB](https://www.influxdata.com/)是一个开放源代码的时间序列数据库，已优化用于在操作监视，应用程序度量，物联网传感器数据和实时分析等领域中快速，高可用性地存储和检索时间序列数据。

#### 11.9.1。连接到InfluxDB

`InfluxDB`如果`influxdb-java`客户端位于类路径上并且设置了数据库的URL，Spring Boot会自动配置一个实例，如以下示例所示：

```properties
spring.influx.url=https://172.0.0.1:8086
```

如果与InfluxDB的连接需要用户和密码，则可以相应地设置`spring.influx.user`和`spring.influx.password`属性。

InfluxDB依赖OkHttp。如果需要在`InfluxDB`后台调整http客户端的使用，则可以注册一个`InfluxDbOkHttpClientBuilderProvider`bean。

## 12.缓存

Spring框架支持透明地向应用程序添加缓存。从本质上讲，抽象将缓存应用于方法，从而根据缓存中可用的信息减少执行次数。缓存逻辑是透明应用的，不会对调用者造成任何干扰。只要通过`@EnableCaching`注释启用了缓存支持，Spring Boot就会自动配置缓存基础结构。

|      | 检查Spring Framework参考的[相关部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/integration.html#cache)以获取更多详细信息。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

简而言之，将缓存添加到服务的操作就像将相关注释添加到其方法一样容易，如以下示例所示：

```java
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Component;

@Component
public class MathService {

    @Cacheable("piDecimals")
    public int computePiDecimal(int i) {
        // ...
    }

}
```

本示例说明了在潜在的昂贵操作上使用缓存的方法。在调用之前`computePiDecimal`，抽象将在`piDecimals`高速缓存中查找与`i`参数匹配的条目。如果找到条目，则缓存中的内容会立即返回给调用方，并且不会调用该方法。否则，将调用该方法，并在返回值之前更新缓存。

|      | 您还可以`@CacheResult`透明地使用标准JSR-107（JCache）注释（例如）。但是，我们强烈建议您不要混合使用Spring Cache和JCache批注。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您不添加任何特定的缓存库，Spring Boot会自动配置一个使用内存中并发映射的[简单提供程序](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-simple)。当需要缓存时（例如`piDecimals`前面的示例），此提供程序将为您创建它。实际上，不建议将简单的提供程序用于生产用途，但是它对于入门并确保您了解功能非常有用。确定要使用的缓存提供程序后，请确保阅读其文档，以了解如何配置应用程序使用的缓存。几乎所有提供程序都要求您显式配置在应用程序中使用的每个缓存。有些提供了一种自定义`spring.cache.cache-names`属性定义的默认缓存的方法。

|      | 还可以透明地[更新](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/integration.html#cache-annotations-put)或从缓存中[逐出](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/integration.html#cache-annotations-evict)数据。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 12.1。支持的缓存提供程序

缓存抽象不提供实际的存储，而是依赖于`org.springframework.cache.Cache`和`org.springframework.cache.CacheManager`接口实现的抽象。

如果尚未定义类型`CacheManager`或`CacheResolver`名称的bean `cacheResolver`（请参阅参考资料[`CachingConfigurer`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/cache/annotation/CachingConfigurer.html)），Spring Boot会尝试检测以下提供程序（按指示的顺序）：

1. [泛型](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-generic)
2. [JCache（JSR-107）](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-jcache)（EhCache 3，Hazelcast，Infinispan等）
3. [EhCache 2.x](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-ehcache2)
4. [淡褐色](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-hazelcast)
5. [Infinispan](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-infinispan)
6. [Couchbase](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-couchbase)
7. [雷迪斯](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-redis)
8. [咖啡因](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-caffeine)
9. [简单](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-simple)

|      | 也可以通过设置属性来*强制*特定的缓存提供程序`spring.cache.type`。如果您需要在某些环境（例如测试）中[完全禁用缓存，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-none)请使用此属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 使用`spring-boot-starter-cache`“入门”可以快速添加基本缓存依赖项。起动器带进来`spring-context-support`。如果手动添加依赖项，则必须包括`spring-context-support`才能使用JCache，EhCache 2.x或Caffeine支持。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果`CacheManager`Spring Boot自动配置了，您可以通过暴露实现该`CacheManagerCustomizer`接口的bean，在完全初始化之前进一步调整其配置。下面的示例设置一个标志，指示`null`应将值向下传递到基础映射：

```java
@Bean
public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
    return new CacheManagerCustomizer<ConcurrentMapCacheManager>() {
        @Override
        public void customize(ConcurrentMapCacheManager cacheManager) {
            cacheManager.setAllowNullValues(false);
        }
    };
}
```

|      | 在前面的示例中，需要进行自动配置`ConcurrentMapCacheManager`。如果不是这种情况（您提供了自己的配置，或者自动配置了其他缓存提供程序），则根本不会调用定制程序。您可以根据需要拥有任意数量的定制程序，也可以使用`@Order`或对其进行排序`Ordered`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 12.1.1。泛型

如果上下文定义*了至少*一个`org.springframework.cache.Cache`bean，则使用通用缓存。将`CacheManager`包装所有该类型的bean。

#### 12.1.2。JCache（JSR-107）

[JCache](https://jcp.org/en/jsr/detail?id=107)通过`javax.cache.spi.CachingProvider`类路径上的存在进行引导（即，类路径上存在符合JSR-107的缓存库），并且`JCacheCacheManager`由`spring-boot-starter-cache`“启动器”提供。提供了各种兼容的库，Spring Boot为Ehcache 3，Hazelcast和Infinispan提供了依赖管理。也可以添加任何其他兼容的库。

可能会出现多个提供者，在这种情况下，必须明确指定提供者。即使JSR-107标准没有强制采用标准化的方式来定义配置文件的位置，Spring Boot也会尽其所能以设置带有实现细节的缓存，如以下示例所示：

```properties
# Only necessary if more than one provider is present
spring.cache.jcache.provider=com.acme.MyCachingProvider
spring.cache.jcache.config=classpath:acme.xml
```

|      | 当缓存库同时提供本机实现和JSR-107支持时，Spring Boot会首选JSR-107支持，因此如果您切换到其他JSR-107实现，则可以使用相同的功能。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | Spring Boot [对Hazelcast](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-hazelcast)具有[常规支持](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-hazelcast)。如果单个`HazelcastInstance`可用，则`CacheManager`除非`spring.cache.jcache.config`指定了该属性，否则它也会自动重用于。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

有两种方法可以自定义基础`javax.cache.cacheManager`：

- 可以在启动时通过设置`spring.cache.cache-names`属性来创建缓存。如果定义了定制`javax.cache.configuration.Configuration`bean，则将其用于定制它们。
- `org.springframework.boot.autoconfigure.cache.JCacheManagerCustomizer`使用的引用调用bean `CacheManager`进行完全定制。

|      | 如果`javax.cache.CacheManager`定义了标准bean，它将自动包装在`org.springframework.cache.CacheManager`抽象期望的实现中。不再对其应用定制。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 12.1.3。EhCache 2.x

如果`ehcache.xml`可以在类路径的根目录找到名为的文件，则使用[EhCache2.x](https://www.ehcache.org/)。如果找到EhCache 2.x，则使用“启动器” `EhCacheCacheManager`提供的`spring-boot-starter-cache`启动程序来引导缓存管理器。也可以提供备用配置文件，如以下示例所示：

```properties
spring.cache.ehcache.config=classpath:config/another-config.xml
```

#### 12.1.4。淡褐色

Spring Boot [对Hazelcast](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-hazelcast)具有[常规支持](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-hazelcast)。如果`HazelcastInstance`已经自动配置了，则会自动将其包装在中`CacheManager`。

#### 12.1.5。Infinispan

[Infinispan](https://infinispan.org/)没有默认配置文件位置，因此必须明确指定它。否则，将使用默认的引导程序。

```properties
spring.cache.infinispan.config=infinispan.xml
```

可以在启动时通过设置`spring.cache.cache-names`属性来创建缓存。如果定义了定制`ConfigurationBuilder`bean，则将其用于定制高速缓存。

|      | Spring Boot对Infinispan的支持仅限于嵌入式模式，并且非常基础。如果您需要更多选择，则应该使用官方的Infinispan Spring Boot启动程序。有关更多详细信息，请参见[Infinispan的文档](https://github.com/infinispan/infinispan-spring-boot)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 12.1.6。Couchbase

如果[Couchbase](https://www.couchbase.com/) Java客户端和`couchbase-spring-cache`实现可用并且已[配置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-couchbase) Couchbase ，`CouchbaseCacheManager`则会自动配置a。通过设置该`spring.cache.cache-names`属性，还可以在启动时创建其他缓存。这些缓存在`Bucket`自动配置的上运行。您可以*还*创建另一个附加缓存中`Bucket`，通过使用定制。假设您在“ main”上需要两个缓存（`cache1`和`cache2`），在“ another”上需要`Bucket`一个（`cache3`2）秒的自定义生存时间`Bucket`。您可以通过配置创建前两个缓存，如下所示：

```properties
spring.cache.cache-names=cache1,cache2
```

然后，您可以定义一个`@Configuration`类来配置Extra `Bucket`和`cache3`缓存，如下所示：

```java
@Configuration(proxyBeanMethods = false)
public class CouchbaseCacheConfiguration {

    private final Cluster cluster;

    public CouchbaseCacheConfiguration(Cluster cluster) {
        this.cluster = cluster;
    }

    @Bean
    public Bucket anotherBucket() {
        return this.cluster.openBucket("another", "secret");
    }

    @Bean
    public CacheManagerCustomizer<CouchbaseCacheManager> cacheManagerCustomizer() {
        return c -> {
            c.prepareCache("cache3", CacheBuilder.newInstance(anotherBucket())
                    .withExpiration(2));
        };
    }

}
```

此样本配置重用了`Cluster`通过自动配置创建的。

#### 12.1.7。雷迪斯

如果[Redis](https://redis.io/)可用并已配置，`RedisCacheManager`则会自动配置a。可以通过设置`spring.cache.cache-names`属性在启动时创建其他缓存，并且可以使用`spring.cache.redis.*`属性配置缓存默认值。例如，以下配置创建`cache1`和`cache2`缓存的*生存时间为* 10分钟：

```properties
spring.cache.cache-names=cache1,cache2
spring.cache.redis.time-to-live=600000
```

|      | 默认情况下，添加密钥前缀，以便如果两个单独的缓存使用相同的密钥，则Redis不会有重叠的密钥，也不会返回无效值。如果您创建自己的，我们强烈建议将此设置保持启用状态`RedisCacheManager`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 您可以通过添加`RedisCacheConfiguration` `@Bean`自己的a来完全控制默认配置。如果您要自定义默认的序列化策略，这将很有用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果需要对配置进行更多控制，请考虑注册`RedisCacheManagerBuilderCustomizer`Bean。以下示例显示了一个定制器，该定制器为`cache1`和配置一个特定的生存时间`cache2`：

```java
@Bean
public RedisCacheManagerBuilderCustomizer myRedisCacheManagerBuilderCustomizer() {
    return (builder) -> builder
            .withCacheConfiguration("cache1",
                    RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofSeconds(10)))
            .withCacheConfiguration("cache2",
                    RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(1)));

}
```

#### 12.1.8。咖啡因

[咖啡因](https://github.com/ben-manes/caffeine)是对Guava缓存的Java 8重写，它取代了对Guava的支持。如果存在咖啡因，`CaffeineCacheManager`则将`spring-boot-starter-cache`自动配置（由“入门”提供）。缓存可以在启动时通过设置`spring.cache.cache-names`属性来创建，并且可以通过以下方式之一（按指示的顺序）进行自定义：

1. 缓存规范由 `spring.cache.caffeine.spec`
2. `com.github.benmanes.caffeine.cache.CaffeineSpec`定义了一个bean
3. `com.github.benmanes.caffeine.cache.Caffeine`定义了一个bean

例如，以下配置创建`cache1`和`cache2`缓存的最大大小为500，*生存时间为* 10分钟

```properties
spring.cache.cache-names=cache1,cache2
spring.cache.caffeine.spec=maximumSize=500,expireAfterAccess=600s
```

如果`com.github.benmanes.caffeine.cache.CacheLoader`定义了bean，它将自动与关联`CaffeineCacheManager`。由于`CacheLoader`将会与缓存管理器管理的*所有*缓存相关联，因此必须将其定义为`CacheLoader`。自动配置将忽略任何其他通用类型。

#### 12.1.9。简单

如果找不到其他任何提供程序，`ConcurrentHashMap`则配置使用作为缓存存储的简单实现。如果您的应用程序中没有缓存库，则这是默认设置。默认情况下，将根据需要创建缓存，但是您可以通过设置`cache-names`属性来限制可用缓存的列表。例如，如果只需要`cache1`和`cache2`缓存，则按`cache-names`如下所示设置属性：

```properties
spring.cache.cache-names=cache1,cache2
```

如果这样做，并且您的应用程序使用了未列出的缓存，那么当需要该缓存时，它将在运行时失败，但不会在启动时失败。这类似于使用未声明的缓存时“实际”缓存提供程序的行为。

#### 10.1.10。没有

当`@EnableCaching`配置中存在时，也将期望使用合适的缓存配置。如果需要在某些环境中完全禁用缓存，请强制缓存类型`none`使用无操作实现，如以下示例所示：

```properties
spring.cache.type=none
```

## 13.消息传递

Spring框架为与消息传递系统集成提供了广泛的支持，从简化的JMS API使用`JmsTemplate`到完整的基础结构，以异步接收消息。Spring AMQP为高级消息队列协议提供了类似的功能集。Spring Boot还为`RabbitTemplate`和RabbitMQ 提供了自动配置选项。Spring WebSocket本身就包含对STOMP消息的支持，而Spring Boot通过启动器和少量的自动配置对此提供了支持。Spring Boot还支持Apache Kafka。

### 13.1。JMS

该`javax.jms.ConnectionFactory`界面提供了创建`javax.jms.Connection`用于与JMS代理进行交互的的标准方法。尽管Spring需要a `ConnectionFactory`才能与JMS一起使用，但是您通常不需要自己直接使用它，而可以依赖于更高级别的消息抽象。（有关详细信息，请参见Spring Framework参考文档的[相关部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/integration.html#jms)。）Spring Boot还会自动配置必要的基础结构，以发送和接收消息。

#### 13.1.1。ActiveMQ支持

当[ActiveMQ](https://activemq.apache.org/)在类路径中可用时，Spring Boot也可以配置`ConnectionFactory`。如果存在代理，则将自动启动和配置嵌入式代理（前提是未通过配置指定代理URL）。

|      | 如果使用`spring-boot-starter-activemq`，则提供了连接或嵌入ActiveMQ实例的必要依赖关系，以及与JMS集成的Spring基础结构。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

ActiveMQ配置由中的外部配置属性控制`spring.activemq.*`。例如，您可以在中声明以下部分`application.properties`：

```properties
spring.activemq.broker-url=tcp://192.168.1.210:9876
spring.activemq.user=admin
spring.activemq.password=secret
```

默认情况下，a 用合理的设置`CachingConnectionFactory`包装本机`ConnectionFactory`，您可以通过以下方式中的外部配置属性来控制这些设置`spring.jms.*`：

```properties
spring.jms.cache.session-cache-size=5
```

如果您想使用本机池，则可以通过向其添加依赖项`org.messaginghub:pooled-jms`并进行相应的配置来实现`JmsPoolConnectionFactory`，如以下示例所示：

```properties
spring.activemq.pool.enabled=true
spring.activemq.pool.max-connections=50
```

|      | 请参阅[`ActiveMQProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/activemq/ActiveMQProperties.java)以获取更多受支持的选项。您还可以注册任意数量的Bean，以实现`ActiveMQConnectionFactoryCustomizer`更高级的自定义。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认情况下，ActiveMQ创建一个目的地（如果尚不存在），以便根据其提供的名称解析目的地。

#### 13.1.2。Artemis支持

当Spring Boot `ConnectionFactory`检测到[Artemis](https://activemq.apache.org/components/artemis/)在类路径中可用时，可以自动配置a 。如果存在代理，则将自动启动和配置嵌入式代理（除非已明确设置mode属性）。支持的模式是`embedded`（明确表示需要嵌入式代理，并且如果类路径上没有代理，则会发生错误）和`native`（使用`netty`传输协议连接到代理）。配置后者后，Spring Boot `ConnectionFactory`将使用默认设置配置一个连接到在本地计算机上运行的代理的代理。

|      | 如果使用`spring-boot-starter-artemis`，则提供了连接到现有Artemis实例所需的依赖关系，以及与JMS集成的Spring基础结构。添加`org.apache.activemq:artemis-jms-server`到您的应用程序可以让您使用嵌入式模式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Artemis配置由中的外部配置属性控制`spring.artemis.*`。例如，您可以在中声明以下部分`application.properties`：

```properties
spring.artemis.mode=native
spring.artemis.host=192.168.1.210
spring.artemis.port=9876
spring.artemis.user=admin
spring.artemis.password=secret
```

嵌入代理时，可以选择是否要启用持久性并列出应使其可用的目的地。可以将它们指定为以逗号分隔的列表，以使用默认选项创建它们，或者您可以分别为高级队列和主题配置定义类型为`org.apache.activemq.artemis.jms.server.config.JMSQueueConfiguration`或的bean `org.apache.activemq.artemis.jms.server.config.TopicConfiguration`。

默认情况下，a 用合理的设置`CachingConnectionFactory`包装本机`ConnectionFactory`，您可以通过以下方式中的外部配置属性来控制这些设置`spring.jms.*`：

```properties
spring.jms.cache.session-cache-size=5
```

如果您想使用本机池，则可以通过向其添加依赖项`org.messaginghub:pooled-jms`并进行相应的配置来实现`JmsPoolConnectionFactory`，如以下示例所示：

```properties
spring.artemis.pool.enabled=true
spring.artemis.pool.max-connections=50
```

请参阅[`ArtemisProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jms/artemis/ArtemisProperties.java)以获取更多受支持的选项。

不涉及JNDI查找，并且使用`name`Artemis配置中的属性或通过配置提供的名称来根据目的地名称解析目的地。

#### 13.1.3。使用JNDI ConnectionFactory

如果您正在应用程序服务器中运行应用程序，Spring Boot会尝试`ConnectionFactory`使用JNDI 来查找JMS 。默认情况下，`java:/JmsXA`和`java:/XAConnectionFactory`处于选中状态。`spring.jms.jndi-name`如果需要指定替代位置，则可以使用该属性，如以下示例所示：

```properties
spring.jms.jndi-name=java:/MyConnectionFactory
```

#### 13.1.4。发送信息

Spring `JmsTemplate`是自动配置的，您可以将其直接自动连接到您自己的bean中，如以下示例所示：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JmsTemplate jmsTemplate;

    @Autowired
    public MyBean(JmsTemplate jmsTemplate) {
        this.jmsTemplate = jmsTemplate;
    }

    // ...

}
```

|      | [`JmsMessagingTemplate`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jms/core/JmsMessagingTemplate.html)可以以类似方式注入。如果定义了a `DestinationResolver`或`MessageConverter`bean，则将其自动关联到auto-configured `JmsTemplate`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 13.1.5。接收讯息

存在JMS基础结构时，可以对任何bean进行注释`@JmsListener`以创建侦听器端点。如果`JmsListenerContainerFactory`未定义，则会自动配置一个默认值。如果定义了a `DestinationResolver`或`MessageConverter`bean，则将其自动关联到默认工厂。

默认情况下，默认工厂是事务性的。如果您在`JtaTransactionManager`存在a的基础结构中运行，则默认情况下它将与侦听器容器关联。如果不是，`sessionTransacted`则启用该标志。在后一种情况下，您可以通过添加`@Transactional`侦听器方法（或其委托）将本地数据存储事务与传入消息的处理相关联。这样可以确保本地事务完成后，传入消息得到确认。这还包括发送已在同一JMS会话上执行的响应消息。

以下组件在`someQueue`目标上创建一个侦听器端点：

```java
@Component
public class MyBean {

    @JmsListener(destination = "someQueue")
    public void processMessage(String content) {
        // ...
    }

}
```

|      | 有关更多详细信息，请参见[的Javadoc`@EnableJms`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/jms/annotation/EnableJms.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您需要创建更多`JmsListenerContainerFactory`实例，或者想要覆盖默认实例，Spring Boot提供了一个`DefaultJmsListenerContainerFactoryConfigurer`可用于初始化的实例，该实例具有`DefaultJmsListenerContainerFactory`与自动配置的实例相同的设置。

例如，以下示例公开了另一个使用特定工厂的工厂`MessageConverter`：

```java
@Configuration(proxyBeanMethods = false)
static class JmsConfiguration {

    @Bean
    public DefaultJmsListenerContainerFactory myFactory(
            DefaultJmsListenerContainerFactoryConfigurer configurer) {
        DefaultJmsListenerContainerFactory factory =
                new DefaultJmsListenerContainerFactory();
        configurer.configure(factory, connectionFactory());
        factory.setMessageConverter(myMessageConverter());
        return factory;
    }

}
```

然后，您可以`@JmsListener`按以下任何带注释的方法使用工厂：

```java
@Component
public class MyBean {

    @JmsListener(destination = "someQueue", containerFactory="myFactory")
    public void processMessage(String content) {
        // ...
    }

}
```

### 13.2。AMQP

高级消息队列协议（AMQP）是面向消息中间件的与平台无关的有线级别协议。Spring AMQP项目将Spring的核心概念应用于基于AMQP的消息传递解决方案的开发。Spring Boot为通过RabbitMQ使用AMQP提供了许多便利，包括`spring-boot-starter-amqp`“启动器”。

#### 13.2.1。RabbitMQ支持

[RabbitMQ](https://www.rabbitmq.com/)是基于AMQP协议的轻量级，可靠，可伸缩和便携式消息代理。Spring用于`RabbitMQ`通过AMQP协议进行通信。

RabbitMQ配置由中的外部配置属性控制`spring.rabbitmq.*`。例如，您可以在中声明以下部分`application.properties`：

```properties
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=admin
spring.rabbitmq.password=secret
```

另外，您可以使用`addresses`属性配置相同的连接：

```properties
spring.rabbitmq.addresses=amqp://admin:secret@localhost
```

如果`ConnectionNameStrategy`上下文中存在bean，它将自动用于命名由auto-configured创建的连接`ConnectionFactory`。请参阅[`RabbitProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/amqp/RabbitProperties.java)以获取更多受支持的选项。

|      | 有关更多详细信息[，](https://spring.io/blog/2010/06/14/understanding-amqp-the-protocol-used-by-rabbitmq/)请参阅[了解RabbitMQ使用的协议AMQP](https://spring.io/blog/2010/06/14/understanding-amqp-the-protocol-used-by-rabbitmq/)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 13.2.2。发送信息

Spring的`AmqpTemplate`和`AmqpAdmin`是自动配置的，您可以将它们直接自动连接到自己的bean中，如以下示例所示：

```java
import org.springframework.amqp.core.AmqpAdmin;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final AmqpAdmin amqpAdmin;
    private final AmqpTemplate amqpTemplate;

    @Autowired
    public MyBean(AmqpAdmin amqpAdmin, AmqpTemplate amqpTemplate) {
        this.amqpAdmin = amqpAdmin;
        this.amqpTemplate = amqpTemplate;
    }

    // ...

}
```

|      | [`RabbitMessagingTemplate`](https://docs.spring.io/spring-amqp/docs/2.2.5.RELEASE/api/org/springframework/amqp/rabbit/core/RabbitMessagingTemplate.html)可以以类似方式注入。如果`MessageConverter`定义了bean，它将自动关联到auto-configured `AmqpTemplate`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如有必要，任何`org.springframework.amqp.core.Queue`定义为bean的对象都会自动用于在RabbitMQ实例上声明相应的队列。

要重试操作，您可以启用重试`AmqpTemplate`（例如，在代理连接丢失的情况下）：

```properties
spring.rabbitmq.template.retry.enabled=true
spring.rabbitmq.template.retry.initial-interval=2s
```

默认情况下，重试是禁用的。您也可以`RetryTemplate`通过声明`RabbitRetryTemplateCustomizer`bean来以编程方式自定义。

#### 13.2.3。接收讯息

存在Rabbit基础结构时，可以对任何bean进行注释`@RabbitListener`以创建侦听器端点。如果`RabbitListenerContainerFactory`未定义，`SimpleRabbitListenerContainerFactory`则会自动配置一个默认值，您可以使用该`spring.rabbitmq.listener.type`属性切换到直接容器。如果定义了a `MessageConverter`或`MessageRecoverer`bean，它将自动与默认工厂关联。

以下示例组件在`someQueue`队列上创建一个侦听器端点：

```java
@Component
public class MyBean {

    @RabbitListener(queues = "someQueue")
    public void processMessage(String content) {
        // ...
    }

}
```

|      | 有关更多详细信息，请参见[的Javadoc`@EnableRabbit`](https://docs.spring.io/spring-amqp/docs/2.2.5.RELEASE/api/org/springframework/amqp/rabbit/annotation/EnableRabbit.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您需要创建更多`RabbitListenerContainerFactory`实例或想要覆盖默认实例，Spring Boot提供了一个`SimpleRabbitListenerContainerFactoryConfigurer`和`DirectRabbitListenerContainerFactoryConfigurer`，您可以使用`SimpleRabbitListenerContainerFactory`和`DirectRabbitListenerContainerFactory`自动配置所使用的工厂相同的设置来初始化一个和。

|      | 选择哪种容器都没有关系。这两个bean通过自动配置公开。 |
| ---- | ---------------------------------------------------- |
|      |                                                      |

例如，以下配置类公开了另一个使用特定工厂的工厂`MessageConverter`：

```java
@Configuration(proxyBeanMethods = false)
static class RabbitConfiguration {

    @Bean
    public SimpleRabbitListenerContainerFactory myFactory(
            SimpleRabbitListenerContainerFactoryConfigurer configurer) {
        SimpleRabbitListenerContainerFactory factory =
                new SimpleRabbitListenerContainerFactory();
        configurer.configure(factory, connectionFactory);
        factory.setMessageConverter(myMessageConverter());
        return factory;
    }

}
```

然后，您可以按任何带`@RabbitListener`注释的方法使用工厂，如下所示：

```java
@Component
public class MyBean {

    @RabbitListener(queues = "someQueue", containerFactory="myFactory")
    public void processMessage(String content) {
        // ...
    }

}
```

您可以启用重试以处理侦听器引发异常的情况。默认情况下`RejectAndDontRequeueRecoverer`使用，但是您可以定义`MessageRecoverer`自己的a。重试用尽后，如果将代理配置为这样做，则消息将被拒绝并被丢弃或路由到死信交换。默认情况下，重试是禁用的。您也可以`RetryTemplate`通过声明`RabbitRetryTemplateCustomizer`bean来以编程方式自定义。

|      | 默认情况下，如果禁用了重试，并且侦听器引发了异常，则会无限期地重试传递。您可以通过两种方式修改此行为：将`defaultRequeueRejected`属性设置为，`false`以便尝试进行零次重新传递，或者抛出`AmqpRejectAndDontRequeueException`来指示应该拒绝该消息。后者是启用重试并达到最大传递尝试次数时使用的机制。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 13.3。Apache Kafka支持

通过提供`spring-kafka`项目的自动配置来支持[Apache Kafka](https://kafka.apache.org/)。

Kafka配置由中的外部配置属性控制`spring.kafka.*`。例如，您可以在中声明以下部分`application.properties`：

```properties
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.consumer.group-id=myGroup
```

|      | 要在启动时创建主题，请添加类型为的Bean `NewTopic`。如果该主题已经存在，则将忽略Bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

请参阅[`KafkaProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/kafka/KafkaProperties.java)以获取更多受支持的选项。

#### 13.3.1。发送信息

Spring `KafkaTemplate`是自动配置的，您可以直接在自己的Bean中自动对其进行布线，如以下示例所示：

```java
@Component
public class MyBean {

    private final KafkaTemplate kafkaTemplate;

    @Autowired
    public MyBean(KafkaTemplate kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    // ...

}
```

|      | 如果`spring.kafka.producer.transaction-id-prefix`定义了属性，`KafkaTransactionManager`则会自动配置a。另外，如果`RecordMessageConverter`定义了bean，它将自动关联到auto-configured `KafkaTemplate`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 13.3.2。接收讯息

存在Apache Kafka基础结构时，可以对任何bean进行注释`@KafkaListener`以创建侦听器端点。如果`KafkaListenerContainerFactory`未定义，则会使用中定义的键自动配置默认值`spring.kafka.listener.*`。

以下组件在该`someTopic`主题上创建一个侦听器端点：

```java
@Component
public class MyBean {

    @KafkaListener(topics = "someTopic")
    public void processMessage(String content) {
        // ...
    }

}
```

如果`KafkaTransactionManager`定义了bean，它将自动关联到容器工厂。类似地，如果一个`ErrorHandler`，`AfterRollbackProcessor`或`ConsumerAwareRebalanceListener`豆被定义，它被自动关联为出厂默认。

根据侦听器的类型，a `RecordMessageConverter`或`BatchMessageConverter`bean与默认工厂关联。如果`RecordMessageConverter`对于批处理侦听器仅存在一个bean，则将其包装在中`BatchMessageConverter`。

|      | `ChainedKafkaTransactionManager`必须标记 一个自定义，`@Primary`因为它通常引用自动配置的`KafkaTransactionManager`bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 13.3.3。卡夫卡流

用于Apache Kafka的Spring提供了一个工厂bean来创建`StreamsBuilder`对象并管理其流的生命周期。`KafkaStreamsConfiguration`只要`kafka-streams`在类路径上是Spring Boot即可自动配置所需的bean，并且通过`@EnableKafkaStreams`注释启用了Kafka Streams 。

启用Kafka Streams意味着必须设置应用程序ID和引导服务器。可以使用来配置前者`spring.kafka.streams.application-id`，`spring.application.name`如果未设置，则默认为。后者可以全局设置，也可以仅针对流进行覆盖。

使用专用属性可以使用几个附加属性。可以使用`spring.kafka.streams.properties`名称空间设置其他任意Kafka属性。另请参阅[其他Kafka属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-kafka-extra-props)。

要使用工厂bean，只需将其连接`StreamsBuilder`到您的工厂，`@Bean`如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
@EnableKafkaStreams
public static class KafkaStreamsExampleConfiguration {

    @Bean
    public KStream<Integer, String> kStream(StreamsBuilder streamsBuilder) {
        KStream<Integer, String> stream = streamsBuilder.stream("ks1In");
        stream.map((k, v) -> new KeyValue<>(k, v.toUpperCase())).to("ks1Out",
                Produced.with(Serdes.Integer(), new JsonSerde<>()));
        return stream;
    }

}
```

默认情况下，由`StreamBuilder`它创建的对象管理的流将自动启动。您可以使用`spring.kafka.streams.auto-startup`属性来自定义此行为。

#### 13.3.4。卡夫卡的其他属性

自动配置支持的属性显示在[appendix-application-properties.html中](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html#common-application-properties)。请注意，在大多数情况下，这些属性（连字符或camelCase）直接映射到Apache Kafka点缀属性。有关详细信息，请参阅Apache Kafka文档。

这些属性的前几个属性适用于所有组件（生产者，使用者，管理员和流），但如果您希望使用不同的值，则可以在组件级别指定。Apache Kafka会指定重要性为HIGH，MEDIUM或LOW的属性。Spring Boot自动配置支持所有HIGH重要性属性，一些选定的MEDIUM和LOW属性以及任何没有默认值的属性。

`KafkaProperties`该类直接提供了Kafka支持的属性的子集。如果您希望使用不直接支持的其他属性来配置生产者或使用者，请使用以下属性：

```properties
spring.kafka.properties.prop.one=first
spring.kafka.admin.properties.prop.two=second
spring.kafka.consumer.properties.prop.three=third
spring.kafka.producer.properties.prop.four=fourth
spring.kafka.streams.properties.prop.five=fifth
```

这会将常见的`prop.one`Kafka属性设置为`first`（适用于生产者，消费者和管理员），将`prop.two`admin属性设置为`second`，将`prop.three`消费者属性设置为`third`，将`prop.four`生产者属性设置为`fourth`，并将`prop.five`stream属性设置为`fifth`。

您还可以`JsonDeserializer`如下配置Spring Kafka ：

```properties
spring.kafka.consumer.value-deserializer=org.springframework.kafka.support.serializer.JsonDeserializer
spring.kafka.consumer.properties.spring.json.value.default.type=com.example.Invoice
spring.kafka.consumer.properties.spring.json.trusted.packages=com.example,org.acme
```

同样，您可以禁用`JsonSerializer`在标头中发送类型信息的默认行为：

```properties
spring.kafka.producer.value-serializer=org.springframework.kafka.support.serializer.JsonSerializer
spring.kafka.producer.properties.spring.json.add.type.headers=false
```

|      | 以这种方式设置的属性将覆盖Spring Boot显式支持的任何配置项。 |
| ---- | ----------------------------------------------------------- |
|      |                                                             |

#### 13.3.5。使用嵌入式Kafka进行测试

Spring for Apache Kafka提供了一种使用嵌入式Apache Kafka代理测试项目的便捷方法。要使用此功能，请使用`@EmbeddedKafka`在`spring-kafka-test`模块中。有关更多信息，请参阅Spring for Apache Kafka [参考手册](https://docs.spring.io/spring-kafka/docs/current/reference/html/#embedded-kafka-annotation)。

要使Spring Boot自动配置与上述嵌入式Apache Kafka代理一起使用，您需要重新映射嵌入式代理地址的系统属性（由 `EmbeddedKafkaBroker`）到Apache Kafka的Spring Boot配置属性中。有几种方法可以做到这一点：

- 提供系统属性以将嵌入式代理地址映射到`spring.kafka.bootstrap-servers`测试类中：

```java
static {
    System.setProperty(EmbeddedKafkaBroker.BROKER_LIST_PROPERTY, "spring.kafka.bootstrap-servers");
}
```

- 在`@EmbeddedKafka`注释上配置属性名称：

```java
@EmbeddedKafka(topics = "someTopic",
        bootstrapServersProperty = "spring.kafka.bootstrap-servers")
```

- 在配置属性中使用占位符：

```properties
spring.kafka.bootstrap-servers=${spring.embedded.kafka.brokers}
```

## 14.通过以下方式调用REST服务 `RestTemplate`

如果您需要从应用程序中调用远程REST服务，则可以使用Spring Framework的[`RestTemplate`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)类。由于`RestTemplate`实例通常需要在使用前进行自定义，因此Spring Boot不提供任何单个自动配置的`RestTemplate`bean。但是，它会自动配置a `RestTemplateBuilder`，可以`RestTemplate`在需要时创建实例。自动配置`RestTemplateBuilder`可确保明智`HttpMessageConverters`地应用于`RestTemplate`实例。

以下代码显示了一个典型示例：

```java
@Service
public class MyService {

    private final RestTemplate restTemplate;

    public MyService(RestTemplateBuilder restTemplateBuilder) {
        this.restTemplate = restTemplateBuilder.build();
    }

    public Details someRestCall(String name) {
        return this.restTemplate.getForObject("/{name}/details", Details.class, name);
    }

}
```

|      | `RestTemplateBuilder`包括许多有用的方法，可用于快速配置`RestTemplate`。例如，要添加BASIC身份验证支持，可以使用`builder.basicAuthentication("user", "password").build()`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 14.1。RestTemplate自定义

有三种主要的`RestTemplate`自定义方法，具体取决于您希望自定义应用的范围。

为了使所有自定义的范围尽可能缩小，请注入自动配置的 `RestTemplateBuilder`，然后根据需要调用其方法。每个方法调用都返回一个新`RestTemplateBuilder`实例，因此自定义仅影响构建器的使用。

要进行应用程序范围的附加定制，请使用`RestTemplateCustomizer`Bean。所有此类bean都会自动注册到自动配置的`RestTemplateBuilder`并应用于生成的任何模板。

以下示例显示了一个定制器，该定制器为除以下之外的所有主机配置代理的使用`192.168.0.5`：

```java
static class ProxyCustomizer implements RestTemplateCustomizer {

    @Override
    public void customize(RestTemplate restTemplate) {
        HttpHost proxy = new HttpHost("proxy.example.com");
        HttpClient httpClient = HttpClientBuilder.create().setRoutePlanner(new DefaultProxyRoutePlanner(proxy) {

            @Override
            public HttpHost determineProxy(HttpHost target, HttpRequest request, HttpContext context)
                    throws HttpException {
                if (target.getHostName().equals("192.168.0.5")) {
                    return null;
                }
                return super.determineProxy(target, request, context);
            }

        }).build();
        restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory(httpClient));
    }

}
```

最后，最极端（很少使用）的选项是创建自己的`RestTemplateBuilder`bean。这样做会关闭a的自动配置，`RestTemplateBuilder`并防止使用任何`RestTemplateCustomizer`bean。

## 15.通过调用REST服务 `WebClient`

如果您的类路径中包含Spring WebFlux，则还可以选择`WebClient`用于调用远程REST服务。与相比`RestTemplate`，此客户具有更多的功能感，并且具有完全的反应性。您可以[在Spring Framework文档](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-client)`WebClient`的专用[部分中](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-client)了解有关的更多信息。。

Spring Boot `WebClient.Builder`为您创建并预配置了一个；强烈建议将其注入您的组件中并使用它来创建`WebClient`实例。Spring Boot正在配置该构建器以共享HTTP资源，以与服务器相同的方式反映编解码器的设置（请参阅[WebFlux HTTP编解码器自动配置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-webflux-httpcodecs)），以及更多。

以下代码显示了一个典型示例：

```java
@Service
public class MyService {

    private final WebClient webClient;

    public MyService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://example.org").build();
    }

    public Mono<Details> someRestCall(String name) {
        return this.webClient.get().uri("/{name}/details", name)
                        .retrieve().bodyToMono(Details.class);
    }

}
```

### 15.1。WebClient运行时

Spring Boot会根据应用程序类路径上可用的库自动检测`ClientHttpConnector`要使用哪个驱动器`WebClient`。目前，还支持Reactor Netty和Jetty RS客户端。

在`spring-boot-starter-webflux`启动依赖于`io.projectreactor.netty:reactor-netty`默认情况下，这使服务器和客户端的实现。如果选择使用Jetty作为反应式服务器，则应在Jetty反应式HTTP客户端库上添加依赖项`org.eclipse.jetty:jetty-reactive-httpclient`。对服务器和客户端使用相同的技术具有其优势，因为它将自动在客户端和服务器之间共享HTTP资源。

开发人员可以通过提供自定义`ReactorResourceFactory`或`JettyResourceFactory`bean 来覆盖Jetty和Reactor Netty的资源配置-这将同时应用于客户端和服务器。

如果您希望为客户端覆盖该选择，则可以定义自己的`ClientHttpConnector`bean并完全控制客户端配置。

您可以[`WebClient`在Spring Framework参考文档中](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-client-builder)了解有关[配置选项的](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-client-builder)更多信息。

### 15.2。WebClient定制

有三种主要的`WebClient`自定义方法，具体取决于您希望自定义应用的范围。

为了使所有定制的范围尽可能狭窄，请注入自动配置的对象`WebClient.Builder`，然后根据需要调用其方法。 `WebClient.Builder`实例是有状态的：构建器上的任何更改都会反映在随后使用它创建的所有客户端中。如果要使用同一构建器创建多个客户端，则还可以考虑使用克隆该构建器`WebClient.Builder other = builder.clone();`。

要对所有`WebClient.Builder`实例进行应用程序范围的附加自定义，可以声明`WebClientCustomizer`bean并`WebClient.Builder`在注入点更改本地实例。

最后，您可以使用原始API并使用`WebClient.create()`。在这种情况下，不会应用自动配置或`WebClientCustomizer`。

## 16.验证

只要JSR-303实现（例如Hibernate验证器）位于类路径上，就会自动启用Bean验证1.1支持的方法验证功能。这使Bean方法`javax.validation`的参数和/或返回值受到约束。具有此类带注释方法的目标类需要`@Validated`在类型级别用注释进行注释，以便在其方法中搜索内联约束注释。

例如，以下服务触发第一个参数的验证，确保其大小在8到10之间：

```java
@Service
@Validated
public class MyBean {

    public Archive findByCodeAndAuthor(@Size(min = 8, max = 10) String code,
            Author author) {
        ...
    }

}
```

## 17.发送电子邮件

Spring框架提供了一种使用`JavaMailSender`接口发送电子邮件的简单抽象方法，而Spring Boot为其提供了自动配置以及启动程序模块。

|      | 有关如何使用的详细说明，请参见[参考文档](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/integration.html#mail)`JavaMailSender`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果`spring.mail.host`和相关库（由定义`spring-boot-starter-mail`）可用，`JavaMailSender`则如果不存在则创建默认库。可以通过`spring.mail`名称空间中的配置项进一步自定义发送方。请参阅[`MailProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/mail/MailProperties.java)以获取更多详细信息。

特别是，某些默认超时值是无限的，您可能需要更改该值，以避免线程被无响应的邮件服务器阻止，如以下示例所示：

```properties
spring.mail.properties.mail.smtp.connectiontimeout=5000
spring.mail.properties.mail.smtp.timeout=3000
spring.mail.properties.mail.smtp.writetimeout=5000
```

也可以`JavaMailSender`使用`Session`JNDI 的现有配置来配置：

```properties
spring.mail.jndi-name=mail/Session
```

当`jndi-name`设置，它优先于所有其他会话相关的设置。

## 18. JTA的分布式事务

Spring Boot通过使用[Atomikos](https://www.atomikos.com/)或[Bitronix](https://github.com/bitronix/btm)嵌入式事务管理器，支持跨多个XA资源的分布式JTA事务。部署到合适的Java EE应用程序服务器时，还支持JTA事务。

当检测到JTA环境时，`JtaTransactionManager`将使用Spring 来管理事务。自动配置的JMS，DataSource和JPA Bean已升级为支持XA事务。您可以使用标准Spring习惯用法（例如`@Transactional`）来参与分布式事务。如果您在JTA环境中，但仍要使用本地事务，则可以将该`spring.jta.enabled`属性设置`false`为禁用JTA自动配置。

### 18.1。使用Atomikos交易管理器

[Atomikos](https://www.atomikos.com/)是一种流行的开源事务管理器，可以嵌入到您的Spring Boot应用程序中。您可以使用`spring-boot-starter-jta-atomikos`入门程序来引入适当的Atomikos库。Spring Boot自动配置Atomikos，并确保将适当的`depends-on`设置应用于您的Spring bean，以正确启动和关闭订单。

默认情况下，Atomikos事务日志将写入`transaction-logs`应用程序主目录中的目录（应用程序jar文件所在的目录）。您可以通过`spring.jta.log-dir`在`application.properties`文件中设置属性来自定义此目录的位置。以开头的属性`spring.jta.atomikos.properties`也可以用于自定义Atomikos `UserTransactionServiceImp`。有关完整的详细信息，请参见[`AtomikosProperties`Javadoc](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/jta/atomikos/AtomikosProperties.html)。

|      | 为了确保多个事务管理器可以安全地协调同一资源管理器，必须为每个Atomikos实例配置一个唯一的ID。默认情况下，此ID是运行Atomikos的计算机的IP地址。为确保生产中的唯一性，应`spring.jta.transaction-manager-id`为应用程序的每个实例配置一个具有不同值的属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 18.2。使用Bitronix交易管理器

[Bitronix](https://github.com/bitronix/btm)是流行的开源JTA事务管理器实现。您可以使用`spring-boot-starter-jta-bitronix`启动器将适当的Bitronix依赖项添加到您的项目中。与Atomikos一样，Spring Boot自动配置Bitronix并对您的bean进行后处理，以确保启动和关闭顺序正确。

默认情况下，Bitronix事务日志文件（`part1.btm`和`part2.btm`）被写入`transaction-logs`应用程序主目录中的目录。您可以通过设置`spring.jta.log-dir`属性来自定义此目录的位置。以开头的属性`spring.jta.bitronix.properties`也绑定到`bitronix.tm.Configuration`Bean，从而可以进行完全自定义。有关详细信息，请参见[Bitronix文档](https://github.com/bitronix/btm/wiki/Transaction-manager-configuration)。

|      | 为了确保多个事务管理器可以安全地协调同一资源管理器，必须为每个Bitronix实例配置唯一的ID。默认情况下，此ID是运行Bitronix的计算机的IP地址。为确保生产中的唯一性，应`spring.jta.transaction-manager-id`为应用程序的每个实例配置一个具有不同值的属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 18.3。使用Java EE托管事务管理器

如果将Spring Boot应用程序打包为`war`或`ear`文件，并将其部署到Java EE应用程序服务器，则可以使用应用程序服务器的内置事务管理器。Spring Boot尝试通过查看常见的JNDI位置（`java:comp/UserTransaction`，`java:comp/TransactionManager`等等）来自动配置事务管理器。如果您使用应用程序服务器提供的事务服务，则通常还需要确保所有资源都由服务器管理并通过JNDI公开。Spring Boot尝试通过`ConnectionFactory`在JNDI路径（`java:/JmsXA`或`java:/XAConnectionFactory`）中查找来尝试自动配置JMS ，您可以使用该[`spring.datasource.jndi-name`属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-connecting-to-a-jndi-datasource)来配置`DataSource`。

### 18.4。混合XA和非XA JMS连接

使用JTA时，主要的JMS `ConnectionFactory`Bean可识别XA，并参与分布式事务。在某些情况下，您可能想通过使用非XA处理某些JMS消息`ConnectionFactory`。例如，您的JMS处理逻辑可能需要比XA超时更长的时间。

如果要使用非XA `ConnectionFactory`，则可以注入`nonXaJmsConnectionFactory`bean而不是`@Primary` `jmsConnectionFactory`bean。为了保持一致性，`jmsConnectionFactory`还使用bean别名提供了bean `xaJmsConnectionFactory`。

以下示例显示了如何注入`ConnectionFactory`实例：

```java
// Inject the primary (XA aware) ConnectionFactory
@Autowired
private ConnectionFactory defaultConnectionFactory;

// Inject the XA aware ConnectionFactory (uses the alias and injects the same as above)
@Autowired
@Qualifier("xaJmsConnectionFactory")
private ConnectionFactory xaConnectionFactory;

// Inject the non-XA aware ConnectionFactory
@Autowired
@Qualifier("nonXaJmsConnectionFactory")
private ConnectionFactory nonXaConnectionFactory;
```

### 18.5。支持替代嵌入式事务管理器

该[`XAConnectionFactoryWrapper`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jms/XAConnectionFactoryWrapper.java)和[`XADataSourceWrapper`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/XADataSourceWrapper.java)接口可用于支持替代嵌入式事务经理。这些接口负责包装`XAConnectionFactory`和`XADataSource`bean，并将它们作为常规`ConnectionFactory`和`DataSource`bean 公开，它们透明地注册到分布式事务中。数据源和JMS自动配置使用JTA变体，前提是您拥有一个`JtaTransactionManager`Bean并在其中注册了适当的XA包装器Bean `ApplicationContext`。

该[BitronixXAConnectionFactoryWrapper](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXAConnectionFactoryWrapper.java)和[BitronixXADataSourceWrapper](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jta/bitronix/BitronixXADataSourceWrapper.java)提供了如何编写XA包装很好的例子。

## 19. Hazelcast

如果[Hazelcast](https://hazelcast.com/)位于类路径上，并且找到了合适的配置，则Spring Boot会自动配置一个`HazelcastInstance`您可以在应用程序中注入的a 。

如果定义一个`com.hazelcast.config.Config`bean，Spring Boot会使用它。如果您的配置定义了一个实例名称，Spring Boot会尝试查找现有实例，而不是创建一个新实例。

您还可以指定通过配置使用的Hazelcast配置文件，如以下示例所示：

```properties
spring.hazelcast.config=classpath:config/my-hazelcast.xml
```

否则，Spring Boot会尝试从默认位置查找Hazelcast配置：`hazelcast.xml`在工作目录中或类路径的根目录中，或`.yaml`在相同位置的对应目录中。我们还检查是否`hazelcast.config`设置了系统属性。有关更多详细信息，请参见[Hazelcast文档](https://docs.hazelcast.org/docs/latest/manual/html-single/)。

如果`hazelcast-client`类路径中存在，Spring Boot首先尝试通过检查以下配置选项来创建客户端：

- `com.hazelcast.client.config.ClientConfig`豆子的存在。
- 该`spring.hazelcast.config`属性定义的配置文件。
- `hazelcast.client.config`系统属性的存在。
- 一个`hazelcast-client.xml`在工作目录或在classpath的根目录。
- 一个`hazelcast-client.yaml`在工作目录或在classpath的根目录。

|      | Spring Boot还具有[对Hazelcast的显式缓存支持](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-caching-provider-hazelcast)。如果启用了缓存，`HazelcastInstance`则会自动将其包装在`CacheManager`实现中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 20. Quartz Scheduler

Spring Boot为使用[Quartz Scheduler](https://www.quartz-scheduler.org/)提供了许多便利，包括`spring-boot-starter-quartz`“ Starter”。如果Quartz可用，`Scheduler`则自动配置a（通过`SchedulerFactoryBean`抽象）。

以下类型的Bean会被自动拾取并与关联`Scheduler`：

- `JobDetail`：定义特定的作业。 `JobDetail`实例可以使用`JobBuilder`API 构建。
- `Calendar`。
- `Trigger`：定义何时触发特定作业。

默认情况下，使用内存`JobStore`。但是，如果`DataSource`应用程序中有可用的bean，并且`spring.quartz.job-store-type`属性已相应配置，则可以配置基于JDBC的存储，如以下示例所示：

```properties
spring.quartz.job-store-type=jdbc
```

使用JDBC存储时，可以在启动时初始化模式，如以下示例所示：

```properties
spring.quartz.jdbc.initialize-schema=always
```

|      | 默认情况下，使用Quartz库随附的标准脚本检测并初始化数据库。这些脚本删除现有表，并在每次重新启动时删除所有触发器。也可以通过设置`spring.quartz.jdbc.schema`属性来提供自定义脚本。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要让Quartz使用`DataSource`应用程序的main之外的其他代码`DataSource`，请声明一个`DataSource`bean，并用注释其`@Bean`方法`@QuartzDataSource`。这样做可以确保和`DataSource`都使用特定于Quartz的`SchedulerFactoryBean`模式进行初始化。

默认情况下，通过配置创建的作业将不会覆盖从持久性作业存储中读取的已注册作业。要启用覆盖现有作业定义的功能，请设置该`spring.quartz.overwrite-existing-jobs`属性。

可以使用`spring.quartz`属性和`SchedulerFactoryBeanCustomizer`bean 来定制Quartz Scheduler配置，从而允许以编程方式进行`SchedulerFactoryBean`定制。可以使用来定制高级Quartz配置属性`spring.quartz.properties.*`。

|      | 特别是，`Executor`bean没有与调度程序关联，因为Quartz提供了一种通过来配置调度程序的方法`spring.quartz.properties`。如果您需要自定义任务执行器，请考虑实现`SchedulerFactoryBeanCustomizer`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

作业可以定义设置器以注入数据映射属性。常规豆也可以类似的方式注入，如以下示例所示：

```java
public class SampleJob extends QuartzJobBean {

    private MyService myService;

    private String name;

    // Inject "MyService" bean
    public void setMyService(MyService myService) { ... }

    // Inject the "name" job data property
    public void setName(String name) { ... }

    @Override
    protected void executeInternal(JobExecutionContext context)
            throws JobExecutionException {
        ...
    }

}
```

## 21.任务执行和计划

在`Executor`上下文中没有bean的情况下，Spring Boot会`ThreadPoolTaskExecutor`使用合理的默认值自动配置a ，这些默认值可以自动与异步任务执行（`@EnableAsync`）和Spring MVC异步请求处理相关联。

|      | 如果您`Executor`在上下文中定义了一个自定义，则常规任务执行（即`@EnableAsync`）将透明地使用它，但由于需要`AsyncTaskExecutor`实现（名为`applicationTaskExecutor`），因此不会配置Spring MVC支持。根据你的目标的安排，你可以改变你`Executor`到一个`ThreadPoolTaskExecutor`或同时定义一个`ThreadPoolTaskExecutor`和`AsyncConfigurer`包装您的自定义`Executor`。通过自动配置`TaskExecutorBuilder`，您可以轻松创建实例，以重现默认情况下自动配置的功能。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

线程池使用8个核心线程，这些线程可以根据负载增长和收缩。可以使用`spring.task.execution`名称空间对那些默认设置进行微调，如以下示例所示：

```properties
spring.task.execution.pool.max-size=16
spring.task.execution.pool.queue-capacity=100
spring.task.execution.pool.keep-alive=10s
```

这会将线程池更改为使用有界队列，以便在队列已满（100个任务）时，线程池最多增加到16个线程。线程的空闲时间为10秒（而不是默认情况下的60秒）时，回收线程会使池的收缩更加激进。

`ThreadPoolTaskScheduler`如果需要与计划的任务执行（`@EnableScheduling`）关联，也可以自动配置A。线程池默认使用一个线程，可以使用`spring.task.scheduling`名称空间对这些设置进行微调。

如果需要创建自定义执行程序或调度程序，则可以在上下文中同时使用`TaskExecutorBuilder`bean和`TaskSchedulerBuilder`bean。

## 22. Spring集成

Spring Boot提供了许多使用[Spring Integration的](https://spring.io/projects/spring-integration)便利，包括`spring-boot-starter-integration`“ Starter”。Spring Integration在消息传递以及其他传输（例如HTTP，TCP等）上提供了抽象。如果您的类路径上有Spring Integration，则可以通过`@EnableIntegration`注释对其进行初始化。

Spring Boot还配置了一些功能，这些功能由其他Spring Integration模块的存在触发。如果`spring-integration-jmx`还在类路径上，则消息处理统计信息将通过JMX发布。如果`spring-integration-jdbc`可用，那么可以在启动时创建默认的数据库架构，如以下行所示：

```properties
spring.integration.jdbc.initialize-schema=always
```

有关更多详细信息，请参见[`IntegrationAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationAutoConfiguration.java)和[`IntegrationProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/integration/IntegrationProperties.java)类。

默认情况下，如果存在Micrometer `meterRegistry`bean，Spring Integration指标将由Micrometer管理。如果您希望使用旧版Spring Integration指标，请`DefaultMetricsFactory`在应用程序上下文中添加一个bean。

## 23.春季会议

Spring Boot为各种数据存储提供了[Spring Session](https://spring.io/projects/spring-session)自动配置。在构建Servlet Web应用程序时，可以自动配置以下存储：

- JDBC
- 雷迪斯
- 淡褐色
- MongoDB

构建反应式Web应用程序时，可以自动配置以下存储：

- 雷迪斯
- MongoDB

如果类路径上存在单个Spring Session模块，则Spring Boot会自动使用该存储实现。如果您有多个实现，则必须选择[`StoreType`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/session/StoreType.java)要用于存储会话的实现。例如，要将JDBC用作后端存储，可以按以下方式配置应用程序：

```properties
spring.session.store-type=jdbc
```

|      | 您可以通过将设置为`store-type`来禁用Spring Session `none`。 |
| ---- | ----------------------------------------------------------- |
|      |                                                             |

每个商店都有特定的其他设置。例如，可以为JDBC存储定制表的名称，如以下示例所示：

```properties
spring.session.jdbc.table-name=SESSIONS
```

要设置会话超时，您可以使用`spring.session.timeout`属性。如果未设置该属性，则自动配置将回退到的值`server.servlet.session.timeout`。

## 24.通过JMX进行监视和管理

Java管理扩展（JMX）提供了监视和管理应用程序的标准机制。Spring Boot将最适合`MBeanServer`作为ID为的bean 公开`mbeanServer`。您的任何豆被标注有春天JMX注释（`@ManagedResource`，`@ManagedAttribute`，或`@ManagedOperation`）接触到它。

如果您的平台提供了标准`MBeanServer`，则Spring Boot将使用该标准，并`MBeanServer`在必要时默认使用VM 。如果全部失败，`MBeanServer`将创建一个新的。

有关[`JmxAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.java)更多详细信息，请参见课程。

## 25.测试

Spring Boot提供了许多实用程序和注释，可以在测试应用程序时提供帮助。测试支持由两个模块提供：`spring-boot-test`包含核心项，并`spring-boot-test-autoconfigure`支持测试的自动配置。

大多数开发人员都使用`spring-boot-starter-test`“入门程序”，该程序可以导入Spring Boot测试模块以及JUnit Jupiter，AssertJ，Hamcrest和许多其他有用的库。

|      | 启动程序还带来了老式引擎，因此您可以运行JUnit 4和JUnit 5测试。如果已将测试迁移到JUnit 5，则应排除对JUnit 4的支持，如以下示例所示：`    org.springframework.boot    spring-boot-starter-test    test                        org.junit.vintage            junit-vintage-engine             ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 25.1。测试范围依赖性

在`spring-boot-starter-test`“入门”（中`test` `scope`）包含以下提供的库：

- [JUnit 5](https://junit.org/junit5/)（包括用于与JUnit 4向后兼容的老式引擎）：用于进行Java应用程序单元测试的事实上的标准。
- [Spring测试](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#integration-testing)和Spring Boot测试：对Spring Boot应用程序的实用程序和集成测试支持。
- [AssertJ](https://assertj.github.io/doc/)：流畅的断言库。
- [Hamcrest](https://github.com/hamcrest/JavaHamcrest)：匹配对象库（也称为约束或谓词）。
- [Mockito](https://site.mockito.org/)：一个Java [模拟](https://site.mockito.org/)框架。
- [JSONassert](https://github.com/skyscreamer/JSONassert)：JSON的断言库。
- [JsonPath](https://github.com/jayway/JsonPath)：JSON的XPath。

通常，我们发现这些通用库在编写测试时很有用。如果这些库不满足您的需求，则可以添加自己的其他测试依赖项。

### 25.2。测试Spring应用程序

依赖注入的主要优点之一是，它应该使您的代码更易于进行单元测试。您可以使用`new`运算符实例化对象，而无需使用Spring。您还可以使用*模拟对象*而不是实际的依赖项。

通常，您需要超越单元测试并开始集成测试（使用Spring `ApplicationContext`）。能够进行集成测试而无需部署应用程序或连接到其他基础结构是很有用的。

Spring框架包括用于此类集成测试的专用测试模块。您可以直接向其声明依赖项，`org.springframework:spring-test`也可以使用`spring-boot-starter-test`“启动器”将其引入。

如果您以前没有使用过该`spring-test`模块，则应先阅读Spring Framework参考文档的[相关部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testing)。

### 25.3。测试Spring Boot应用程序

Spring Boot应用程序是Spring `ApplicationContext`，因此除了常规的Spring上下文之外，无需进行任何特殊的测试。

|      | 默认情况下，Spring Boot的外部属性，日志记录和其他功能仅在`SpringApplication`用于创建时才安装在上下文中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring Boot提供了一个`@SpringBootTest`注释，`spring-test` `@ContextConfiguration`当您需要Spring Boot功能时，它可以用作标准注释的替代。注释通过[创建`ApplicationContext`在测试中使用过的来`SpringApplication`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-detecting-config)起作用。除了`@SpringBootTest`提供许多其他注释，还用于[测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)应用程序的[更多特定](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)部分。

|      | 如果您使用的是JUnit 4，请不要忘记也将其添加`@RunWith(SpringRunner.class)`到测试中，否则注释将被忽略。如果您使用的是JUnit 5，则无需添加等价物`@ExtendWith(SpringExtension.class)`，`@SpringBootTest`并且其他`@…Test`注释已经使用它进行了注释。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认情况下，`@SpringBootTest`不会启动服务器。您可以使用的`webEnvironment`属性`@SpringBootTest`进一步完善测试的运行方式：

- `MOCK`（默认）：加载Web `ApplicationContext`并提供模拟Web环境。使用此注释时，不会启动嵌入式服务器。如果您的类路径上没有网络环境，则此模式将透明地退回到创建常规的非网络环境`ApplicationContext`。它可以与Web应用程序结合使用[`@AutoConfigureMockMvc`或`@AutoConfigureWebTestClient`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-with-mock-environment)用于基于模拟的测试。
- `RANDOM_PORT`：加载`WebServerApplicationContext`并提供真实的网络环境。嵌入式服务器将启动并在随机端口上侦听。
- `DEFINED_PORT`：加载`WebServerApplicationContext`并提供真实的网络环境。嵌入式服务器将启动并在已定义的端口（来自`application.properties`）上或在的默认端口上进行侦听`8080`。
- `NONE`：`ApplicationContext`通过使用加载，`SpringApplication`但不提供*任何*网络环境（模拟或其他方式）。

|      | 如果您的测试是`@Transactional`，则默认情况下它将在每个测试方法的末尾回滚事务。但是，由于将这种安排与一个`RANDOM_PORT`或多个`DEFINED_PORT`隐式提供真实的servlet环境一起使用，HTTP客户端和服务器在单独的线程中运行，因此在单独的事务中运行。在这种情况下，服务器上启动的任何事务都不会回滚。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `@SpringBootTest``webEnvironment = WebEnvironment.RANDOM_PORT`如果您的应用程序对管理服务器使用其他端口，则with 还将在单独的随机端口上启动管理服务器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.1。检测Web应用程序类型

如果Spring MVC可用，则配置基于常规MVC的应用程序上下文。如果您只有Spring WebFlux，我们将检测到它并配置基于WebFlux的应用程序上下文。

如果两者都存在，则Spring MVC优先。如果要在这种情况下测试反应式Web应用程序，则必须设置`spring.main.web-application-type`属性：

```java
@SpringBootTest(properties = "spring.main.web-application-type=reactive")
class MyWebFluxTests { ... }
```

#### 25.3.2。检测测试配置

如果您熟悉Spring Test Framework，则可能习惯于使用`@ContextConfiguration(classes=…)`来指定`@Configuration`要加载的Spring 。另外，您可能经常`@Configuration`在测试中使用嵌套类。

在测试Spring Boot应用程序时，通常不需要这样做。`@*Test`当您没有明确定义时，Spring Boot的注释会自动搜索您的主要配置。

搜索算法从包含测试的程序包开始工作，直到找到带有`@SpringBootApplication`或注释的类`@SpringBootConfiguration`。只要您以合理的方式[对代码](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code)进行[结构化](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code)，通常就可以找到您的主要配置。

|      | 如果使用[测试注释来测试应用程序的更特定的部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)，则应避免在[main方法的应用程序类](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-user-configuration)上添加特定于特定区域的配置设置。define的基础组件扫描配置`@SpringBootApplication`排除了用于确保切片按预期工作的过滤器。如果`@ComponentScan`在`@SpringBootApplication`-annotated类上使用显式指令，请注意这些过滤器将被禁用。如果使用切片，则应重新定义它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果要自定义主要配置，则可以使用嵌套`@TestConfiguration`类。与`@Configuration`将使用嵌套类代替应用程序的主要配置的嵌套`@TestConfiguration`类不同，除了应用程序的主要配置之外，还使用了嵌套类。

|      | Spring的测试框架在测试之间缓存应用程序上下文。因此，只要您的测试共享相同的配置（无论如何发现），加载上下文的潜在耗时过程就只会发生一次。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.3。排除测试配置

如果您的应用程序使用组件扫描（例如，如果使用`@SpringBootApplication`或`@ComponentScan`），则可能会偶然发现到处仅为特定测试创建的顶级配置类。

如我们[先前所见](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-detecting-config)，`@TestConfiguration`可以在测试的内部类上使用来自定义主要配置。当放置在顶级类上时，`@TestConfiguration`指示`src/test/java`不应通过扫描来拾取其中的类。然后，可以在需要的位置显式导入该类，如以下示例所示：

```java
@SpringBootTest
@Import(MyTestsConfiguration.class)
class MyTests {

    @Test
    void exampleTest() {
        ...
    }

}
```

|      | 如果您直接使用`@ComponentScan`（即不是通过`@SpringBootApplication`），则需要向其注册`TypeExcludeFilter`。有关详细信息，请参见[Javadoc](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/context/TypeExcludeFilter.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.4。使用应用程序参数

如果您的应用程序需要[参数](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-application-arguments)，则可以`@SpringBootTest`使用`args`属性将其注入。

```java
@SpringBootTest(args = "--app.test=one")
class ApplicationArgumentsExampleTests {

    @Test
    void applicationArgumentsPopulated(@Autowired ApplicationArguments args) {
        assertThat(args.getOptionNames()).containsOnly("app.test");
        assertThat(args.getOptionValues("app.test")).containsOnly("one");
    }

}
```

#### 25.3.5。在模拟环境中进行测试

默认情况下，`@SpringBootTest`不启动服务器。如果您有要在此模拟环境下进行测试的Web终结点，则可以另外配置[`MockMvc`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference//testing.html#spring-mvc-test-framework)，如以下示例所示：

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
class MockMvcExampleTests {

    @Test
    void exampleTest(@Autowired MockMvc mvc) throws Exception {
        mvc.perform(get("/")).andExpect(status().isOk()).andExpect(content().string("Hello World"));
    }

}
```

|      | 如果您只想关注Web层而不希望开始完整工作`ApplicationContext`，请考虑[使用`@WebMvcTest`代替](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-mvc-tests)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

或者，您可以配置[`WebTestClient`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#webtestclient-tests)，如以下示例所示：

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.AutoConfigureWebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest
@AutoConfigureWebTestClient
class MockWebTestClientExampleTests {

    @Test
    void exampleTest(@Autowired WebTestClient webClient) {
        webClient.get().uri("/").exchange().expectStatus().isOk().expectBody(String.class).isEqualTo("Hello World");
    }

}
```

|      | 在模拟环境中进行测试通常比在完整的Servlet容器中运行更快。但是，由于模拟是在Spring MVC层进行的，因此无法使用MockMvc直接测试依赖于较低级别Servlet容器行为的代码。例如，Spring Boot的错误处理基于Servlet容器提供的“错误页面”支持。这意味着，尽管您可以按预期测试MVC层引发和处理异常，但是您无法直接测试是否呈现了特定的[自定义错误页面](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-error-handling-custom-error-pages)。如果您需要测试这些较低级别的问题，则可以按照下一节中的描述启动一个完全运行的服务器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.6。使用正在运行的服务器进行测试

如果需要启动完全运行的服务器，建议您使用随机端口。如果使用`@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`，则每次运行测试时都会随机选择一个可用端口。

该`@LocalServerPort`注释可用于[注射使用的实际端口](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-discover-the-http-port-at-runtime)到您的测试。为了方便起见，需要对启动的服务器进行REST调用的测试可以另外`@Autowire`使用[`WebTestClient`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#webtestclient-tests)，它解析到正在运行的服务器的相对链接，并带有用于验证响应的专用API，如以下示例所示：

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class RandomPortWebTestClientExampleTests {

    @Test
    void exampleTest(@Autowired WebTestClient webClient) {
        webClient.get().uri("/").exchange().expectStatus().isOk().expectBody(String.class).isEqualTo("Hello World");
    }

}
```

此设置需要`spring-webflux`在类路径上进行。如果您不能或不会添加webflux，Spring Boot还提供了以下`TestRestTemplate`功能：

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.web.client.TestRestTemplate;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class RandomPortTestRestTemplateExampleTests {

    @Test
    void exampleTest(@Autowired TestRestTemplate restTemplate) {
        String body = restTemplate.getForObject("/", String.class);
        assertThat(body).isEqualTo("Hello World");
    }

}
```

#### 25.3.7。自定义WebTestClient

要自定义`WebTestClient`Bean，请配置`WebTestClientBuilderCustomizer`Bean。`WebTestClient.Builder`会使用用于创建的任何此类bean进行调用`WebTestClient`。

#### 25.3.8。使用JMX

由于测试上下文框架缓存上下文，因此默认情况下禁用JMX，以防止相同的组件在同一域上注册。如果此类测试需要访问`MBeanServer`，请考虑将其标记为脏：

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest(properties = "spring.jmx.enabled=true")
@DirtiesContext
class SampleJmxTests {

    @Autowired
    private MBeanServer mBeanServer;

    @Test
    void exampleTest() {
        // ...
    }

}
```

#### 25.3.9。模拟豆和间谍豆

运行测试时，有时有必要在应用程序上下文中模拟某些组件。例如，您可能在开发过程中无法使用某些远程服务的外观。当您要模拟在实际环境中可能难以触发的故障时，模拟功能也很有用。

Spring Boot包含一个`@MockBean`注释，可用于为您的Bean中的bean定义Mockito模拟`ApplicationContext`。您可以使用注释添加新的bean或替换单个现有的bean定义。注释可以直接用于测试类，测试中的字段或`@Configuration`类和字段。在字段上使用时，还将注入创建的模拟的实例。每种测试方法后，模拟豆都会自动重置。

|      | 如果您的测试使用Spring Boot的测试注释之一（例如`@SpringBootTest`），则此功能会自动启用。要以其他方式使用此功能，必须明确添加侦听器，如以下示例所示：`@TestExecutionListeners(MockitoTestExecutionListener.class)` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下面的示例用`RemoteService`模拟实现替换现有的bean：

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.context.*;
import org.springframework.boot.test.mock.mockito.*;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.BDDMockito.*;

@SpringBootTest
class MyTests {

    @MockBean
    private RemoteService remoteService;

    @Autowired
    private Reverser reverser;

    @Test
    void exampleTest() {
        // RemoteService has been injected into the reverser bean
        given(this.remoteService.someCall()).willReturn("mock");
        String reverse = reverser.reverseSomeCall();
        assertThat(reverse).isEqualTo("kcom");
    }

}
```

|      | `@MockBean`不能用于模拟应用程序上下文刷新期间执行的bean的行为。到执行测试时，应用程序上下文刷新已完成，并且配置模拟行为为时已晚。我们建议`@Bean`在这种情况下使用一种方法来创建和配置模拟。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

另外，您可以使用`@SpyBean`Mockito来包装任何现有的bean `spy`。有关完整的详细信息，请参见[Javadoc](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/test/mock/mockito/SpyBean.html)。

|      | CGLib代理（例如为范围内的Bean创建的代理）将代理方法声明为`final`。这会阻止Mockito正常运行，因为它无法模拟或监视`final`其默认配置中的方法。如果要模拟或监视此类bean，请通过添加`org.mockito:mockito-inline`到应用程序的测试依赖项中，将Mockito配置为使用其内联模拟生成器。这允许Mockito模拟和监视`final`方法。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | Spring的测试框架在测试之间缓存应用程序上下文，并为共享相同配置的测试重用上下文，但是缓存密钥的使用`@MockBean`或`@SpyBean`影响缓存密钥，这很可能会增加上下文的数量。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果使用`@SpyBean`用于`@Cacheable`通过名称引用参数的方法来监视Bean，则必须使用编译应用程序`-parameters`。这样可以确保一旦侦察到bean，就可以将参数名称用于缓存基础结构。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.10。自动配置的测试

Spring Boot的自动配置系统适用于应用程序，但有时对于测试而言可能有点过多。它通常仅有助于加载测试应用程序“切片”所需的配置部分。例如，您可能想测试Spring MVC控制器是否正确映射了URL，并且不想在这些测试中涉及数据库调用，或者您想测试JPA实体，并且当您使用JPA实体时，您对Web层不感兴趣。测试运行。

该`spring-boot-test-autoconfigure`模块包括许多注释，可用于自动配置此类“切片”。它们中的每一个都以相似的方式工作，提供了一个`@…Test`加载的注释`ApplicationContext`以及一个或多个`@AutoConfigure…`可用于自定义自动配置设置的注释。

|      | 每个切片将组件扫描限制为适当的组件，并加载一组非常受限制的自动配置类。如果您需要排除其中之一，则大多数`@…Test`注释都提供一个`excludeAutoConfiguration`属性。或者，您可以使用`@ImportAutoConfiguration#exclude`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `@…Test`不支持在一个测试中 使用多个注释来包含多个“切片” 。如果需要多个“切片”，请选择其中一个`@…Test`注释，并`@AutoConfigure…`手动添加其他“切片” 的注释。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 也可以将`@AutoConfigure…`注释与标准`@SpringBootTest`注释一起使用。如果您对“切片”应用程序不感兴趣，但需要一些自动配置的测试bean，则可以使用此组合。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.11。自动配置的JSON测试

要测试对象JSON序列化和反序列化是否按预期工作，可以使用`@JsonTest`批注。 `@JsonTest`自动配置可用的受支持的JSON映射器，该映射器可以是以下库之一：

- 杰克逊`ObjectMapper`，任何`@JsonComponent`豆子和任何杰克逊`Module`的
- `Gson`
- `Jsonb`

|      | 由启用了自动配置的列表`@JsonTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果需要配置自动配置的元素，则可以使用`@AutoConfigureJsonTesters`注释。

Spring Boot包括基于AssertJ的助手，这些助手与JSONAssert和JsonPath库一起使用，以检查JSON是否按预期方式显示。的`JacksonTester`，`GsonTester`，`JsonbTester`，和`BasicJsonTester`类可以分别用于杰克逊，GSON，Jsonb，和字符串。`@Autowired`使用时，可以在测试类上使用任何帮助程序字段`@JsonTest`。以下示例显示了Jackson的测试类：

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.autoconfigure.json.*;
import org.springframework.boot.test.context.*;
import org.springframework.boot.test.json.*;

import static org.assertj.core.api.Assertions.*;

@JsonTest
class MyJsonTests {

    @Autowired
    private JacksonTester<VehicleDetails> json;

    @Test
    void testSerialize() throws Exception {
        VehicleDetails details = new VehicleDetails("Honda", "Civic");
        // Assert against a `.json` file in the same package as the test
        assertThat(this.json.write(details)).isEqualToJson("expected.json");
        // Or use JSON path based assertions
        assertThat(this.json.write(details)).hasJsonPathStringValue("@.make");
        assertThat(this.json.write(details)).extractingJsonPathStringValue("@.make")
                .isEqualTo("Honda");
    }

    @Test
    void testDeserialize() throws Exception {
        String content = "{\"make\":\"Ford\",\"model\":\"Focus\"}";
        assertThat(this.json.parse(content))
                .isEqualTo(new VehicleDetails("Ford", "Focus"));
        assertThat(this.json.parseObject(content).getMake()).isEqualTo("Ford");
    }

}
```

|      | JSON帮助程序类也可以直接在标准单元测试中使用。为此，如果不使用，请`initFields`在您的`@Before`方法中调用辅助方法`@JsonTest`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您正在使用基于Spring Boot的基于AssertJ的助手在给定JSON路径上对数字值进行断言，则可能无法使用，`isEqualTo`具体取决于类型。相反，您可以使用AssertJ `satisfies`声明该值匹配给定条件。例如，以下示例断言实际数字是一个接近于`0.15`偏移量内的浮点值`0.01`。

```java
assertThat(json.write(message))
    .extractingJsonPathNumberValue("@.test.numberValue")
    .satisfies((number) -> assertThat(number.floatValue()).isCloseTo(0.15f, within(0.01f)));
```

#### 25.3.12。自动配置的Spring MVC测试

要测试Spring MVC控制器是否按预期工作，请使用`@WebMvcTest`注释。 `@WebMvcTest`自动配置Spring MVC的基础设施和限制扫描豆`@Controller`，`@ControllerAdvice`，`@JsonComponent`，`Converter`，`GenericConverter`，`Filter`，`HandlerInterceptor`，`WebMvcConfigurer`，和`HandlerMethodArgumentResolver`。`@Component`使用此注释时，不会扫描常规bean。

|      | 启用的自动配置设置列表`@WebMvcTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果需要注册其他组件，例如Jackson `Module`，则可以`@Import`在测试中使用来导入其他配置类。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

通常，`@WebMvcTest`它仅限于单个控制器，并且与之结合使用`@MockBean`以为所需的协作者提供模拟实现。

`@WebMvcTest`也可以自动配置`MockMvc`。Mock MVC提供了一种强大的方法来快速测试MVC控制器，而无需启动完整的HTTP服务器。

|      | 您还可以通过用注释`MockMvc`非`@WebMvcTest`（例如`@SpringBootTest`）自动配置`@AutoConfigureMockMvc`。以下示例使用`MockMvc`： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```java
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.autoconfigure.web.servlet.*;
import org.springframework.boot.test.mock.mockito.*;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.BDDMockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserVehicleController.class)
class MyControllerTests {

    @Autowired
    private MockMvc mvc;

    @MockBean
    private UserVehicleService userVehicleService;

    @Test
    void testExample() throws Exception {
        given(this.userVehicleService.getVehicleDetails("sboot"))
                .willReturn(new VehicleDetails("Honda", "Civic"));
        this.mvc.perform(get("/sboot/vehicle").accept(MediaType.TEXT_PLAIN))
                .andExpect(status().isOk()).andExpect(content().string("Honda Civic"));
    }

}
```

|      | 如果您需要配置自动配置的元素（例如，当应该应用servlet过滤器时），则可以在`@AutoConfigureMockMvc`注释中使用属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果使用HtmlUnit或Selenium，则自动配置还会提供HtmlUnit `WebClient`bean和/或Selenium `WebDriver`bean。以下示例使用HtmlUnit：

```java
import com.gargoylesoftware.htmlunit.*;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.autoconfigure.web.servlet.*;
import org.springframework.boot.test.mock.mockito.*;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.BDDMockito.*;

@WebMvcTest(UserVehicleController.class)
class MyHtmlUnitTests {

    @Autowired
    private WebClient webClient;

    @MockBean
    private UserVehicleService userVehicleService;

    @Test
    void testExample() throws Exception {
        given(this.userVehicleService.getVehicleDetails("sboot"))
                .willReturn(new VehicleDetails("Honda", "Civic"));
        HtmlPage page = this.webClient.getPage("/sboot/vehicle.html");
        assertThat(page.getBody().getTextContent()).isEqualTo("Honda Civic");
    }

}
```

|      | 默认情况下，Spring Boot将`WebDriver`bean放在一个特殊的“作用域”中，以确保驱动程序在每次测试后退出并注入新实例。如果您不希望出现这种情况，可以将其添加`@Scope("singleton")`到`WebDriver` `@Bean`定义中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `webDriver`Spring Boot创建 的作用域将替换任何用户定义的同名作用域。如果您定义自己的`webDriver`范围，则可能会发现它在使用时停止工作`@WebMvcTest`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您在类路径上具有Spring Security，`@WebMvcTest`还将扫描`WebSecurityConfigurer`bean。您可以使用Spring Security的测试支持，而不是完全禁用此类测试的安全性。有关如何使用Spring Security `MockMvc`支持的更多详细信息，请参见*[howto.html操作](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-use-test-with-spring-security)*方法部分。

|      | 有时编写Spring MVC测试是不够的。Spring Boot可以帮助您在[实际服务器上](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-with-running-server)运行[完整的端到端测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-with-running-server)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.13。自动配置的Spring WebFlux测试

要测试[Spring WebFlux](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference//web-reactive.html)控制器是否按预期工作，可以使用`@WebFluxTest`注释。 `@WebFluxTest`自动配置春季WebFlux基础设施和限制扫描豆`@Controller`，`@ControllerAdvice`，`@JsonComponent`，`Converter`，`GenericConverter`，`WebFilter`，和`WebFluxConfigurer`。使用注释`@Component`时，不扫描常规Bean `@WebFluxTest`。

|      | 由启用了自动配置的列表`@WebFluxTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您需要注册其他组件（例如Jackson）`Module`，则可以`@Import`在测试中使用导入其他配置类。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

通常，`@WebFluxTest`它仅限于单个控制器，并且与`@MockBean`注释结合使用以为所需的协作者提供模拟实现。

`@WebFluxTest`还自动配置[`WebTestClient`](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#webtestclient)，它提供了一种强大的方法来快速测试WebFlux控制器，而无需启动完整的HTTP服务器。

|      | 您还可以通过用注释`WebTestClient`非`@WebFluxTest`（例如`@SpringBootTest`）自动配置`@AutoConfigureWebTestClient`。以下示例显示了同时使用`@WebFluxTest`和的类`WebTestClient`： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.WebFluxTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.reactive.server.WebTestClient;

@WebFluxTest(UserVehicleController.class)
class MyControllerTests {

    @Autowired
    private WebTestClient webClient;

    @MockBean
    private UserVehicleService userVehicleService;

    @Test
    void testExample() throws Exception {
        given(this.userVehicleService.getVehicleDetails("sboot"))
                .willReturn(new VehicleDetails("Honda", "Civic"));
        this.webClient.get().uri("/sboot/vehicle").accept(MediaType.TEXT_PLAIN)
                .exchange()
                .expectStatus().isOk()
                .expectBody(String.class).isEqualTo("Honda Civic");
    }

}
```

|      | WebFlux应用程序仅支持此设置，因为`WebTestClient`在模拟的Web应用程序中使用的设置目前仅适用于WebFlux。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `@WebFluxTest`无法检测通过功能Web框架注册的路由。要`RouterFunction`在上下文中测试Bean，请考虑`RouterFunction`通过`@Import`或使用导入自己`@SpringBootTest`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `@WebFluxTest`无法检测到通过`@Bean`类型注册的自定义安全配置`SecurityWebFilterChain`。要将其包含在测试中，您将需要通过`@Import`或使用导入注册Bean的配置`@SpringBootTest`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 有时编写Spring WebFlux测试是不够的。Spring Boot可以帮助您在[实际服务器上](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-with-running-server)运行[完整的端到端测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-with-running-server)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.14。自动配置的数据JPA测试

您可以使用`@DataJpaTest`注释来测试JPA应用程序。默认情况下，它会扫描`@Entity`类并配置Spring Data JPA存储库。如果在类路径上有嵌入式数据库，它也会配置一个。常规`@Component`Bean未装入`ApplicationContext`。

|      | 启用的自动配置设置列表`@DataJpaTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认情况下，数据JPA测试是事务性的，并在每次测试结束时回滚。有关更多详细信息，请参见Spring Framework参考文档中的[相关部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions)。如果这不是您想要的，则可以按以下方式禁用测试或整个类的事务管理：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@DataJpaTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
class ExampleNonTransactionalTests {

}
```

数据JPA测试也可以注入[`TestEntityManager`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure/orm/jpa/TestEntityManager.java)Bean，这`EntityManager`是专门为测试设计的标准JPA的替代方法。如果要`TestEntityManager`在`@DataJpaTest`实例外部使用，也可以使用`@AutoConfigureTestEntityManager`注释。`JdbcTemplate`如果需要，也可以使用A。以下示例显示了`@DataJpaTest`正在使用的注释：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.orm.jpa.*;

import static org.assertj.core.api.Assertions.*;

@DataJpaTest
class ExampleRepositoryTests {

    @Autowired
    private TestEntityManager entityManager;

    @Autowired
    private UserRepository repository;

    @Test
    void testExample() throws Exception {
        this.entityManager.persist(new User("sboot", "1234"));
        User user = this.repository.findByUsername("sboot");
        assertThat(user.getUsername()).isEqualTo("sboot");
        assertThat(user.getVin()).isEqualTo("1234");
    }

}
```

内存嵌入式数据库通常运行良好，不需要任何安装，因此通常可以很好地进行测试。但是，如果您希望对真实数据库运行测试，则可以使用`@AutoConfigureTestDatabase`注释，如以下示例所示：

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace=Replace.NONE)
class ExampleRepositoryTests {

    // ...

}
```

#### 25.3.15。自动配置的JDBC测试

`@JdbcTest`与相似，`@DataJpaTest`但适用于仅需`DataSource`使用且不使用Spring Data JDBC的测试。默认情况下，它配置一个内存嵌入式数据库和一个`JdbcTemplate`。常规`@Component`Bean未装入`ApplicationContext`。

|      | 由启用了自动配置的列表`@JdbcTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

缺省情况下，JDBC测试是事务性的，并在每次测试结束时回滚。有关更多详细信息，请参见Spring Framework参考文档中的[相关部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions)。如果这不是您想要的，则可以为测试或整个类禁用事务管理，如下所示：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.jdbc.JdbcTest;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@JdbcTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
class ExampleNonTransactionalTests {

}
```

如果您希望测试针对真实数据库运行，则可以使用`@AutoConfigureTestDatabase`与相同的注释方式`DataJpaTest`。（请参阅“ [自动配置的数据JPA测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test) ”。）

#### 25.3.16。自动配置的数据JDBC测试

`@DataJdbcTest`与`@JdbcTest`使用Spring Data JDBC存储库的测试类似，但适用于这些测试。默认情况下，它配置一个内存嵌入式数据库，一个`JdbcTemplate`和Spring Data JDBC存储库。常规`@Component`Bean未装入`ApplicationContext`。

|      | 由启用了自动配置的列表`@DataJdbcTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认情况下，Data JDBC测试是事务性的，并在每个测试结束时回滚。有关更多详细信息，请参见Spring Framework参考文档中的[相关部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions)。如果这不是您想要的，则可以禁用测试或整个测试类的事务管理，如[JDBC示例所示](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jdbc-test)。

如果您希望测试针对真实数据库运行，则可以使用`@AutoConfigureTestDatabase`与相同的注释方式`DataJpaTest`。（请参阅“ [自动配置的数据JPA测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test) ”。）

#### 25.3.17。自动配置的jOOQ测试

您可以使用`@JooqTest`与`@JdbcTest`jOOQ相关的测试类似的方式进行测试。由于jOOQ严重依赖与数据库模式相对应的基于Java的模式，因此`DataSource`将使用现有模式。如果要用内存数据库替换它，可以使用`@AutoConfigureTestDatabase`覆盖这些设置。（有关将jOOQ与Spring Boot结合使用的更多信息，请参阅本章前面的“ [使用jOOQ](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-jooq) ”。）常规`@Component`Bean未加载到中`ApplicationContext`。

|      | 由启用了自动配置的列表`@JooqTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`@JooqTest`配置一个`DSLContext`。常规`@Component`Bean未装入`ApplicationContext`。以下示例显示了`@JooqTest`正在使用的注释：

```java
import org.jooq.DSLContext;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.jooq.JooqTest;

@JooqTest
class ExampleJooqTests {

    @Autowired
    private DSLContext dslContext;
}
```

JOOQ测试是事务性的，默认情况下会在每个测试结束时回滚。如果这不是您想要的，则可以禁用测试或整个测试类的事务管理，如[JDBC示例所示](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jdbc-test)。

#### 25.3.18。自动配置的数据MongoDB测试

您可以`@DataMongoTest`用来测试MongoDB应用程序。默认情况下，它配置内存嵌入式MongoDB（如果有），配置`MongoTemplate`，扫描`@Document`类，并配置Spring Data MongoDB存储库。常规`@Component`Bean未装入`ApplicationContext`。（有关将MongoDB与Spring Boot结合使用的更多信息，请参阅本章前面的“ [MongoDB](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-mongodb) ”。）

|      | 启用的自动配置设置列表`@DataMongoTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

此类显示`@DataMongoTest`正在使用的注释：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;
import org.springframework.data.mongodb.core.MongoTemplate;

@DataMongoTest
class ExampleDataMongoTests {

    @Autowired
    private MongoTemplate mongoTemplate;

    //
}
```

内存嵌入式MongoDB通常运行良好，不需要任何开发人员安装，因此通常可以很好地用于测试。但是，如果您希望对真实的MongoDB服务器运行测试，则应排除嵌入式MongoDB自动配置，如以下示例所示：

```java
import org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;

@DataMongoTest(excludeAutoConfiguration = EmbeddedMongoAutoConfiguration.class)
class ExampleDataMongoNonEmbeddedTests {

}
```

#### 25.3.19。自动配置的数据Neo4j测试

您可以`@DataNeo4jTest`用来测试Neo4j应用程序。默认情况下，它使用内存中嵌入式Neo4j（如果有嵌入式驱动程序可用），扫描`@NodeEntity`类并配置Spring Data Neo4j存储库。常规`@Component`Bean未装入`ApplicationContext`。（有关将Neo4J与Spring Boot结合使用的更多信息，请参阅本章前面的“ [Neo4j](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-neo4j) ”。）

|      | 启用的自动配置设置列表`@DataNeo4jTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了在Spring Boot中使用Neo4J测试的典型设置：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.neo4j.DataNeo4jTest;

@DataNeo4jTest
class ExampleDataNeo4jTests {

    @Autowired
    private YourRepository repository;

    //
}
```

默认情况下，Data Neo4j测试是事务性的，并在每次测试结束时回滚。有关更多详细信息，请参见Spring Framework参考文档中的[相关部分](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions)。如果这不是您想要的，则可以为测试或整个类禁用事务管理，如下所示：

```java
import org.springframework.boot.test.autoconfigure.data.neo4j.DataNeo4jTest;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@DataNeo4jTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
class ExampleNonTransactionalTests {

}
```

#### 25.3.20。自动配置的数据Redis测试

您可以`@DataRedisTest`用来测试Redis应用程序。默认情况下，它会扫描`@RedisHash`类并配置Spring Data Redis存储库。常规`@Component`Bean未装入`ApplicationContext`。（有关在Spring Boot中使用Redis的更多信息，请参阅本章前面的“ [Redis](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-redis) ”。）

|      | 启用的自动配置设置列表`@DataRedisTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了`@DataRedisTest`正在使用的注释：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.redis.DataRedisTest;

@DataRedisTest
class ExampleDataRedisTests {

    @Autowired
    private YourRepository repository;

    //
}
```

#### 25.3.21。自动配置的数据LDAP测试

您可以`@DataLdapTest`用来测试LDAP应用程序。默认情况下，它配置内存嵌入式LDAP（如果可用），配置`LdapTemplate`，扫描`@Entry`类，并配置Spring Data LDAP存储库。常规`@Component`Bean未装入`ApplicationContext`。（有关将LDAP与Spring Boot结合使用的更多信息，请参阅本章前面的“ [LDAP](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-ldap) ”。）

|      | 启用的自动配置设置列表`@DataLdapTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下示例显示了`@DataLdapTest`正在使用的注释：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.ldap.DataLdapTest;
import org.springframework.ldap.core.LdapTemplate;

@DataLdapTest
class ExampleDataLdapTests {

    @Autowired
    private LdapTemplate ldapTemplate;

    //
}
```

内存嵌入式LDAP通常非常适合测试，因为它速度快并且不需要安装任何开发人员。但是，如果您希望针对真实的LDAP服务器运行测试，则应排除嵌入式LDAP自动配置，如以下示例所示：

```java
import org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration;
import org.springframework.boot.test.autoconfigure.data.ldap.DataLdapTest;

@DataLdapTest(excludeAutoConfiguration = EmbeddedLdapAutoConfiguration.class)
class ExampleDataLdapNonEmbeddedTests {

}
```

#### 25.3.22。自动配置的REST客户端

您可以使用`@RestClientTest`注释来测试REST客户端。默认情况下，它会自动配置Jackson，GSON和Jsonb支持，配置`RestTemplateBuilder`，并添加对的支持`MockRestServiceServer`。常规`@Component`Bean未装入`ApplicationContext`。

|      | 启用的自动配置设置列表`@RestClientTest`可以[在附录中找到](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

应该使用的`value`或`components`属性来指定要测试的特定Bean `@RestClientTest`，如以下示例所示：

```java
@RestClientTest(RemoteVehicleDetailsService.class)
class ExampleRestClientTest {

    @Autowired
    private RemoteVehicleDetailsService service;

    @Autowired
    private MockRestServiceServer server;

    @Test
    void getVehicleDetailsWhenResultIsSuccessShouldReturnDetails()
            throws Exception {
        this.server.expect(requestTo("/greet/details"))
                .andRespond(withSuccess("hello", MediaType.TEXT_PLAIN));
        String greeting = this.service.callRestService();
        assertThat(greeting).isEqualTo("hello");
    }

}
```

#### 25.3.23。自动配置的Spring REST Docs测试

您可以在Mock MVC，REST Assured或WebTestClient的测试中使用`@AutoConfigureRestDocs`批注来使用[Spring REST Docs](https://spring.io/projects/spring-restdocs)。它消除了Spring REST Docs中对JUnit扩展的需求。

`@AutoConfigureRestDocs`可以用来覆盖默认的输出目录（`target/generated-snippets`如果您使用的是Maven或`build/generated-snippets`Gradle）。它也可以用于配置出现在任何记录的URI中的主机，方案和端口。

##### 使用Mock MVC自动配置的Spring REST Docs测试

`@AutoConfigureRestDocs`自定义`MockMvc`bean以使用Spring REST Docs。您可以`@Autowired`像使用Mock MVC和Spring REST Docs一样，通过在测试中使用和使用它来注入它，如以下示例所示：

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
@AutoConfigureRestDocs
class UserDocumentationTests {

    @Autowired
    private MockMvc mvc;

    @Test
    void listUsers() throws Exception {
        this.mvc.perform(get("/users").accept(MediaType.TEXT_PLAIN))
                .andExpect(status().isOk())
                .andDo(document("list-users"));
    }

}
```

如果您需要对Spring REST Docs配置进行更多控制，而不是的属性所提供的控制`@AutoConfigureRestDocs`，则可以使用`RestDocsMockMvcConfigurationCustomizer`Bean，如以下示例所示：

```java
@TestConfiguration
static class CustomizationConfiguration
        implements RestDocsMockMvcConfigurationCustomizer {

    @Override
    public void customize(MockMvcRestDocumentationConfigurer configurer) {
        configurer.snippets().withTemplateFormat(TemplateFormats.markdown());
    }

}
```

如果要使用Spring REST Docs对参数化输出目录的支持，则可以创建一个`RestDocumentationResultHandler`bean。`alwaysDo`使用此结果处理程序的自动配置调用，从而使每个`MockMvc`调用自动生成默认片段。以下示例显示了一个`RestDocumentationResultHandler`正在定义的对象：

```java
@TestConfiguration(proxyBeanMethods = false)
static class ResultHandlerConfiguration {

    @Bean
    public RestDocumentationResultHandler restDocumentation() {
        return MockMvcRestDocumentation.document("{method-name}");
    }

}
```

##### 使用WebTestClient自动配置的Spring REST Docs测试

`@AutoConfigureRestDocs`也可以与一起使用`WebTestClient`。您可以`@Autowired`像在使用`@WebFluxTest`Spring REST Docs 时一样，通过在测试中使用和注入它来进行注入，如以下示例所示：

```java
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.autoconfigure.web.reactive.WebFluxTest;
import org.springframework.test.web.reactive.server.WebTestClient;

import static org.springframework.restdocs.webtestclient.WebTestClientRestDocumentation.document;

@WebFluxTest
@AutoConfigureRestDocs
class UsersDocumentationTests {

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void listUsers() {
        this.webTestClient.get().uri("/").exchange().expectStatus().isOk().expectBody()
                .consumeWith(document("list-users"));
    }

}
```

如果您需要对Spring REST Docs配置进行更多控制，而不是的属性所提供的控制`@AutoConfigureRestDocs`，则可以使用`RestDocsWebTestClientConfigurationCustomizer`Bean，如以下示例所示：

```java
@TestConfiguration(proxyBeanMethods = false)
public static class CustomizationConfiguration implements RestDocsWebTestClientConfigurationCustomizer {

    @Override
    public void customize(WebTestClientRestDocumentationConfigurer configurer) {
        configurer.snippets().withEncoding("UTF-8");
    }

}
```

##### 具有REST保证的自动配置的Spring REST Docs测试

`@AutoConfigureRestDocs`使`RequestSpecification`预配置为使用Spring REST Docs 的bean可用于您的测试。您可以`@Autowired`像在使用REST Assured和Spring REST Docs时一样，通过在测试中使用和使用它来注入它，如以下示例所示：

```java
import io.restassured.specification.RequestSpecification;
import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.web.server.LocalServerPort;

import static io.restassured.RestAssured.given;
import static org.hamcrest.Matchers.is;
import static org.springframework.restdocs.restassured3.RestAssuredRestDocumentation.document;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureRestDocs
class UserDocumentationTests {

    @Test
    void listUsers(@Autowired RequestSpecification documentationSpec, @LocalServerPort int port) {
        given(documentationSpec).filter(document("list-users")).when().port(port).get("/").then().assertThat()
                .statusCode(is(200));
    }

}
```

如果您需要对Spring REST Docs配置进行更多控制，而不是的属性所提供的控制`@AutoConfigureRestDocs`，`RestDocsRestAssuredConfigurationCustomizer`则可以使用Bean，如以下示例所示：

```java
@TestConfiguration(proxyBeanMethods = false)
public static class CustomizationConfiguration implements RestDocsRestAssuredConfigurationCustomizer {

    @Override
    public void customize(RestAssuredRestDocumentationConfigurer configurer) {
        configurer.snippets().withTemplateFormat(TemplateFormats.markdown());
    }

}
```

#### 25.3.24。额外的自动配置和切片

每个切片提供一个或多个`@AutoConfigure…`注释，这些注释定义了应作为切片一部分包含的自动配置。可以通过创建自定义`@AutoConfigure…`批注或简单地通过添加`@ImportAutoConfiguration`到测试中来添加其他自动配置，如以下示例所示：

```java
@JdbcTest
@ImportAutoConfiguration(IntegrationAutoConfiguration.class)
class ExampleJdbcTests {

}
```

|      | 确保不要使用常规`@Import`注释来导入自动配置，因为它们是由Spring Boot以特定方式处理的。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.25。用户配置和切片

如果您以合理的方式[构建代码](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code)，`@SpringBootApplication`则[默认情况下](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-detecting-config)将[使用](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-detecting-config)类作为测试的配置。

因此，重要的是不要用特定于其功能特定区域的配置设置来乱扔应用程序的主类。

假设您正在使用Spring Batch，并且依赖于它的自动配置。您可以定义`@SpringBootApplication`如下：

```java
@SpringBootApplication
@EnableBatchProcessing
public class SampleApplication { ... }
```

因为此类是测试的源配置，所以任何切片测试实际上都尝试启动Spring Batch，这绝对不是您想要执行的操作。建议的方法是将特定于区域的配置移到`@Configuration`与您的应用程序相同级别的单独的类，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
@EnableBatchProcessing
public class BatchConfiguration { ... }
```

|      | 根据您应用程序的复杂性，您可以`@Configuration`为您的自定义设置一个类，也可以为每个域区域指定一个类。后一种方法使您可以在其中的一个测试中启用该`@Import`注释（如有必要），并带有批注。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

测试切片将`@Configuration`类别排除在扫描范围之外。例如，对于a `@WebMvcTest`，以下配置将`WebMvcConfigurer`在测试切片加载的应用程序上下文中不包括给定的bean：

```java
@Configuration
public class WebConfiguration {
    @Bean
    public WebMvcConfigurer testConfigurer() {
        return new WebMvcConfigurer() {
            ...
        };
    }
}
```

但是，以下配置将导致自定义`WebMvcConfigurer`由测试片加载。

```java
@Component
public class TestWebMvcConfigurer implements WebMvcConfigurer {
    ...
}
```

混乱的另一个来源是类路径扫描。假定在以合理的方式构造代码时，您需要扫描其他程序包。您的应用程序可能类似于以下代码：

```java
@SpringBootApplication
@ComponentScan({ "com.example.app", "org.acme.another" })
public class SampleApplication { ... }
```

这样做有效地覆盖了默认的组件扫描指令，并且具有扫描这两个软件包的副作用，而与您选择的切片无关。例如，`@DataJpaTest`似乎突然扫描了应用程序的组件和用户配置。同样，将自定义指令移至单独的类是解决此问题的好方法。

|      | 如果这不是您的选择，则可以`@SpringBootConfiguration`在测试层次结构中的某个位置创建一个位置，以代替使用它。另外，您可以为测试指定一个源，从而禁用查找默认源的行为。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.3.26。使用Spock测试Spring Boot应用程序

如果您希望使用Spock测试Spring Boot应用程序，则应在`spock-spring`应用程序的构建中添加对Spock 模块的依赖。 `spock-spring`将Spring的测试框架集成到Spock中。建议您使用Spock 1.2或更高版本，以受益于Spock的Spring框架和Spring Boot集成的许多改进。有关更多详细信息，请参见[Spock的Spring模块的文档](http://spockframework.org/spock/docs/1.2/modules.html#_spring_module)。

### 25.4。测试工具

在测试应用程序时通常有用的一些测试实用程序类作为的一部分打包在一起`spring-boot`。

#### 25.4.1。ConfigFileApplicationContextInitializer

`ConfigFileApplicationContextInitializer`是一个`ApplicationContextInitializer`可以应用于测试以加载Spring Boot `application.properties`文件的工具。如不需要`@SpringBootTest`以下示例所示，可以在不需要由提供的全部功能时使用它：

```java
@ContextConfiguration(classes = Config.class,
    initializers = ConfigFileApplicationContextInitializer.class)
```

|      | `ConfigFileApplicationContextInitializer`单独 使用不能为`@Value("${…}")`注射提供支持。唯一的工作是确保将`application.properties`文件加载到Spring的中`Environment`。为了获得`@Value`支持，您需要额外配置a `PropertySourcesPlaceholderConfigurer`或使用`@SpringBootTest`，后者会为您自动配置一个。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 25.4.2。TestPropertyValues

`TestPropertyValues`可让您快速将属性添加到`ConfigurableEnvironment`或`ConfigurableApplicationContext`。您可以使用`key=value`字符串来调用它，如下所示：

```java
TestPropertyValues.of("org=Spring", "name=Boot").applyTo(env);
```

#### 25.4.3。OutputCapture

`OutputCapture`是一个`Extension`可用于捕获`System.out`和`System.err`输出的JUnit 。要使用add `@ExtendWith(OutputCaptureExtension.class)`和ject `CapturedOutput`作为测试类构造函数或测试方法的参数，如下所示：

```java
@ExtendWith(OutputCaptureExtension.class)
class OutputCaptureTests {

    @Test
    void testName(CapturedOutput output) {
        System.out.println("Hello World!");
        assertThat(output).contains("World");
    }

}
```

#### 25.4.4。TestRestTemplate

`TestRestTemplate`是Spring的一种方便替代方法，`RestTemplate`在集成测试中很有用。您可以使用普通模板或发送基本HTTP身份验证（带有用户名和密码）的模板。在这两种情况下，模板都通过不向服务器端错误抛出异常来以易于测试的方式运行。

|      | Spring Framework 5.0提供了一个新功能`WebTestClient`，适用于[WebFlux集成测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-webflux-tests)以及[WebFlux和MVC端到端测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-testing-with-running-server)。与相比，它为声明提供了流畅的API `TestRestTemplate`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

建议（但不是强制性的）使用Apache HTTP Client（版本4.3.2或更高版本）。如果您在类路径中具有该名称，则`TestRestTemplate`通过适当配置客户端来做出响应。如果确实使用Apache的HTTP客户端，则会启用一些其他易于测试的功能：

- 不遵循重定向（因此您可以声明响应位置）。
- Cookies被忽略（因此模板是无状态的）。

`TestRestTemplate` 可以在集成测试中直接实例化，如以下示例所示：

```java
public class MyTest {

    private TestRestTemplate template = new TestRestTemplate();

    @Test
    public void testRequest() throws Exception {
        HttpHeaders headers = this.template.getForEntity(
                "https://myhost.example.com/example", String.class).getHeaders();
        assertThat(headers.getLocation()).hasHost("other.example.com");
    }

}
```

另外，如果将`@SpringBootTest`注释与`WebEnvironment.RANDOM_PORT`或结合使用，则`WebEnvironment.DEFINED_PORT`可以注入完全配置的注释`TestRestTemplate`并开始使用它。如有必要，可以通过`RestTemplateBuilder`Bean 应用其他定制。未指定主机和端口的所有URL都会自动连接到嵌入式服务器，如以下示例所示：

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class SampleWebClientTests {

    @Autowired
    private TestRestTemplate template;

    @Test
    void testRequest() {
        HttpHeaders headers = this.template.getForEntity("/example", String.class).getHeaders();
        assertThat(headers.getLocation()).hasHost("other.example.com");
    }

    @TestConfiguration(proxyBeanMethods = false)
    static class Config {

        @Bean
        RestTemplateBuilder restTemplateBuilder() {
            return new RestTemplateBuilder().setConnectTimeout(Duration.ofSeconds(1))
                    .setReadTimeout(Duration.ofSeconds(1));
        }

    }

}
```

## 26. WebSockets

Spring Boot为嵌入式Tomcat，Jetty和Undertow提供了WebSockets自动配置。如果将war文件部署到独立容器，Spring Boot会假定该容器负责其WebSocket支持的配置。

Spring Framework 为MVC Web应用程序提供了[丰富的WebSocket支持](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#websocket)，可以通过该`spring-boot-starter-websocket`模块轻松访问。

WebSocket支持也可用于[反应式Web应用程序，](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web-reactive.html#webflux-websocket)并且需要在以下位置包括WebSocket API `spring-boot-starter-webflux`：

```xml
<dependency>
    <groupId>javax.websocket</groupId>
    <artifactId>javax.websocket-api</artifactId>
</dependency>
```

## 27.网络服务

Spring Boot提供了Web服务自动配置，因此您所需要做的就是定义您的`Endpoints`。

在[春天的Web服务功能](https://docs.spring.io/spring-ws/docs/3.0.8.RELEASE/reference/)可以与轻松访问`spring-boot-starter-webservices`模块。

`SimpleWsdl11Definition``SimpleXsdSchema`可以分别为您的WSDL和XSD自动创建and bean。为此，请配置其位置，如以下示例所示：

```properties
spring.webservices.wsdl-locations=classpath:/wsdl
```

### 27.1。使用调用Web服务`WebServiceTemplate`

如果需要从应用程序调用远程Web服务，则可以使用[`WebServiceTemplate`](https://docs.spring.io/spring-ws/docs/3.0.8.RELEASE/reference/#client-web-service-template)该类。由于`WebServiceTemplate`实例通常需要在使用前进行自定义，因此Spring Boot不提供任何单个自动配置的`WebServiceTemplate`bean。但是，它会自动配置a `WebServiceTemplateBuilder`，可以`WebServiceTemplate`在需要时创建实例。

以下代码显示了一个典型示例：

```java
@Service
public class MyService {

    private final WebServiceTemplate webServiceTemplate;

    public MyService(WebServiceTemplateBuilder webServiceTemplateBuilder) {
        this.webServiceTemplate = webServiceTemplateBuilder.build();
    }

    public DetailsResp someWsCall(DetailsReq detailsReq) {
         return (DetailsResp) this.webServiceTemplate.marshalSendAndReceive(detailsReq, new SoapActionCallback(ACTION));
    }

}
```

默认情况下，使用类路径上的可用HTTP客户端库来`WebServiceTemplateBuilder`检测合适的基于`WebServiceMessageSender`HTTP的服务器。您还可以如下自定义读取和连接超时：

```java
@Bean
public WebServiceTemplate webServiceTemplate(WebServiceTemplateBuilder builder) {
    return builder.messageSenders(new HttpWebServiceMessageSenderBuilder()
            .setConnectTimeout(5000).setReadTimeout(2000).build()).build();
}
```

## 28.创建自己的自动配置

如果您在开发共享库的公司中工作，或者在开源或商业库中工作，则可能要开发自己的自动配置。自动配置类可以捆绑在外部jar中，并且仍由Spring Boot拾取。

自动配置可以与“启动器”相关联，该“启动器”提供自动配置代码以及您将使用的典型库。我们首先介绍构建自己的自动配置所需的知识，然后继续[进行创建自定义启动器所需](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-starter)的[典型步骤](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-starter)。

|      | 可以使用一个[演示项目](https://github.com/snicoll-demos/spring-boot-master-auto-configuration)来展示如何逐步创建入门程序。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 28.1。了解自动配置的Bean

在后台，自动配置是通过标准`@Configuration`类实现的。其他`@Conditional`注释用于约束何时应应用自动配置。通常，自动配置类使用`@ConditionalOnClass`和`@ConditionalOnMissingBean`注释。这样可确保仅当找到相关的类且未声明自己的类时，自动配置才适用`@Configuration`。

您可以浏览的源代码[`spring-boot-autoconfigure`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure)以查看`@Configuration`Spring提供的类（请参见[`META-INF/spring.factories`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories)文件）。

### 28.2。查找自动配置候选人

Spring Boot检查`META-INF/spring.factories`发布的jar中是否存在文件。该文件应在`EnableAutoConfiguration`键下列出您的配置类，如以下示例所示：

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration = \
com.mycorp.libx.autoconfigure.LibXAutoConfiguration，\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

|      | 自动配置*只能*以这种方式加载。确保在特定的程序包空间中定义它们，并且决不要将它们作为组件扫描的目标。此外，自动配置类不应启用组件扫描以查找其他组件。`@Import`应该使用特定的。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果需要按特定顺序应用配置，则可以使用[`@AutoConfigureAfter`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.java)或[`@AutoConfigureBefore`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.java)注释。例如，如果您提供特定于Web的配置，则可能需要在之后应用类`WebMvcAutoConfiguration`。

如果要订购某些相互之间不具有直接了解的自动配置，则也可以使用`@AutoConfigureOrder`。该注释与常规注释具有相同的语义，`@Order`但为自动配置类提供了专用顺序。

### 28.3。条件注释

您几乎总是希望`@Conditional`在自动配置类中包含一个或多个注释。该`@ConditionalOnMissingBean`注释是用来让开发者重写自动配置，如果他们不满意自己的缺省值一个常见的例子。

Spring Boot包含许多`@Conditional`注释，您可以通过注释`@Configuration`类或单个`@Bean`方法在自己的代码中重用它们。这些注释包括：

- [上课条件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-class-conditions)
- [豆条件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-bean-conditions)
- [物业条件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-property-conditions)
- [资源条件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-resource-conditions)
- [Web应用条件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-web-application-conditions)
- [SpEL表达条件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-spel-conditions)

#### 28.3.1。上课条件

的`@ConditionalOnClass`和`@ConditionalOnMissingClass`注解让`@Configuration`类基于特定类的存在或不存在被包括在内。由于注释元数据是使用[ASM](https://asm.ow2.org/)进行解析的`value`，因此即使该类可能实际上没有出现在正在运行的应用程序类路径上，您也可以使用该属性来引用真实的类。`name`如果您更喜欢通过使用`String`值来指定类名，则也可以使用该属性。

这种机制不适用于`@Bean`通常以返回类型为条件的目标的方法：在方法的条件适用之前，JVM将加载该类和可能处理的方法引用，如果该类不是当下。

要处理这种情况，`@Configuration`可以使用一个单独的类来隔离条件，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
// Some conditions
public class MyAutoConfiguration {

    // Auto-configured beans

    @Configuration(proxyBeanMethods = false)
    @ConditionalOnClass(EmbeddedAcmeService.class)
    static class EmbeddedConfiguration {

        @Bean
        @ConditionalOnMissingBean
        public EmbeddedAcmeService embeddedAcmeService() { ... }

    }

}
```

|      | 如果使用元注释`@ConditionalOnClass`或`@ConditionalOnMissingClass`作为元注释的一部分来组成自己的组合注释，`name`则在不处理这种情况下，必须使用引用类。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 28.3.2。豆条件

的`@ConditionalOnBean`和`@ConditionalOnMissingBean`注解让豆基于特定豆的存在或不存在被包括在内。您可以使用该`value`属性按类型`name`指定bean 或按名称指定bean。该`search`属性使您可以限制`ApplicationContext`搜索bean时应考虑的层次结构。

当放置在`@Bean`方法上时，目标类型默认为方法的返回类型，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public MyService myService() { ... }

}
```

在前面的例子中，`myService`豆将被创建如果没有类型的豆`MyService`已经包含在所述`ApplicationContext`。

|      | 您需要非常注意添加bean定义的顺序，因为这些条件是根据到目前为止已处理的内容来评估的。因此，我们建议在自动配置类上仅使用`@ConditionalOnBean`和`@ConditionalOnMissingBean`批注（因为保证在添加任何用户定义的Bean定义后即可加载它们）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | `@ConditionalOnBean`并且`@ConditionalOnMissingBean`不要阻止`@Configuration`类的创建。在类级别使用这些条件`@Bean`与使用注释标记每个包含的方法之间的唯一区别是，`@Configuration`如果条件不匹配，则前者会阻止将该类注册为bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 28.3.3。物业条件

该`@ConditionalOnProperty`注解让基于Spring的环境属性配置包括在内。使用`prefix`和`name`属性来指定应检查的属性。默认情况下，`false`匹配存在且不等于的任何属性。您还可以使用`havingValue`和`matchIfMissing`属性创建更高级的检查。

#### 28.3.4。资源条件

该`@ConditionalOnResource`注解让配置被包括仅当特定资源是否存在。资源可通过使用通常的弹簧约定来指定，如显示在下面的例子：`file:/home/user/test.dat`。

#### 28.3.5。Web应用条件

在`@ConditionalOnWebApplication`和`@ConditionalOnNotWebApplication`注释，让配置包含依赖于应用程序是否是一个“Web应用程序”。基于Servlet的Web应用程序是使用Spring `WebApplicationContext`，定义`session`范围或具有的任何应用程序`ConfigurableWebEnvironment`。反应式Web应用程序是使用`ReactiveWebApplicationContext`或具有的任何应用程序`ConfigurableReactiveWebEnvironment`。

#### 28.3.6。SpEL表达条件

该`@ConditionalOnExpression`注解让基于一个的结果配置被包括[使用SpEL表达](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#expressions)。

### 28.4。测试您的自动配置

自动配置可能受许多因素影响：用户配置（`@Bean`定义和`Environment`自定义），条件评估（特定库的存在）以及其他因素。具体而言，每个测试都应创建一个良好的定义`ApplicationContext`，以代表这些定制的组合。 `ApplicationContextRunner`提供了实现此目标的好方法。

`ApplicationContextRunner`通常定义为测试类的一个字段，以收集基本的通用配置。以下示例确保`UserServiceAutoConfiguration`始终调用该示例：

```java
private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
        .withConfiguration(AutoConfigurations.of(UserServiceAutoConfiguration.class));
```

|      | 如果必须定义多个自动配置，则无需按与运行应用程序时完全相同的顺序调用它们的声明。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

每个测试都可以使用运行器来表示特定的用例。例如，下面的示例调用一个用户配置（`UserConfiguration`），并检查自动配置是否正确退出。调用`run`提供了可与一起使用的回调上下文`Assert4J`。

```java
@Test
void defaultServiceBacksOff() {
    this.contextRunner.withUserConfiguration(UserConfiguration.class).run((context) -> {
        assertThat(context).hasSingleBean(UserService.class);
        assertThat(context).getBean("myUserService").isSameAs(context.getBean(UserService.class));
    });
}

@Configuration(proxyBeanMethods = false)
static class UserConfiguration {

    @Bean
    UserService myUserService() {
        return new UserService("mine");
    }

}
```

也可以轻松自定义`Environment`，如以下示例所示：

```java
@Test
void serviceNameCanBeConfigured() {
    this.contextRunner.withPropertyValues("user.name=test123").run((context) -> {
        assertThat(context).hasSingleBean(UserService.class);
        assertThat(context.getBean(UserService.class).getName()).isEqualTo("test123");
    });
}
```

跑步者也可以用来显示`ConditionEvaluationReport`。可以按级别`INFO`或`DEBUG`水平打印报告。以下示例显示了如何`ConditionEvaluationReportLoggingListener`在自动配置测试中使用来打印报告。

```java
@Test
public void autoConfigTest {
    ConditionEvaluationReportLoggingListener initializer = new ConditionEvaluationReportLoggingListener(
            LogLevel.INFO);
    ApplicationContextRunner contextRunner = new ApplicationContextRunner()
            .withInitializer(initializer).run((context) -> {
                    // Do something...
            });
}
```

#### 28.4.1。模拟Web上下文

如果您需要测试仅在Servlet或Reactive Web应用程序上下文中运行的自动配置，请分别使用`WebApplicationContextRunner`或`ReactiveWebApplicationContextRunner`。

#### 28.4.2。覆盖类路径

也可以测试在运行时不存在特定的类和/或程序包时发生的情况。Spring Boot附带了一个`FilteredClassLoader`，跑步者可以轻松使用。在以下示例中，我们断言如果`UserService`不存在，则将自动禁用自动配置：

```java
@Test
void serviceIsIgnoredIfLibraryIsNotPresent() {
    this.contextRunner.withClassLoader(new FilteredClassLoader(UserService.class))
            .run((context) -> assertThat(context).doesNotHaveBean("userService"));
}
```

### 28.5。创建自己的入门

库的完整Spring Boot入门程序可能包含以下组件：

- `autoconfigure`包含自动配置代码的模块。
- 将`starter`其提供给一个依赖模块`autoconfigure`模块以及库和通常是有用的任何附加的依赖性。简而言之，添加启动程序应提供开始使用该库所需的一切。

|      | 如果不需要将这两个问题分开，则可以将自动配置代码和依赖性管理结合在一个模块中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 28.5.1。命名

您应确保为启动器提供适当的名称空间。`spring-boot`即使使用其他Maven，也不要以模块名称开头`groupId`。将来，我们可能会为您自动配置的内容提供官方支持。

根据经验，应在启动器后命名一个组合模块。例如，假设您要为“ acme”创建启动器，并命名自动配置模块`acme-spring-boot-autoconfigure`和启动器`acme-spring-boot-starter`。如果只有一个将两者结合的模块，请命名为`acme-spring-boot-starter`。

#### 28.5.2。配置键

如果您的入门者提供了配置密钥，请为其使用唯一的名称空间。特别是，不包括你的名字空间去春Boot使用键（如`server`，`management`，`spring`，等）。如果使用相同的名称空间，将来我们可能会以破坏模块的方式修改这些名称空间。根据经验，所有键都以您拥有的名称空间（例如`acme`）为前缀。

通过为每个属性添加字段javadoc来确保记录了配置键，如以下示例所示：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

    /**
     * Whether to check the location of acme resources.
     */
    private boolean checkLocation = true;

    /**
     * Timeout for establishing a connection to the acme server.
     */
    private Duration loginTimeout = Duration.ofSeconds(3);

    // getters & setters

}
```

|      | 您仅应将简单文本与`@ConfigurationProperties`Javadoc字段一起使用，因为在将它们添加到JSON之前不会对其进行处理。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

我们在内部遵循以下规则以确保描述一致：

- 请勿以“ The”或“ A”开头描述。
- 对于 `boolean`类型，请从“是否”或“启用”开始描述。
- 对于基于集合的类型，请以“以逗号分隔的列表”开始描述
- 如果默认单位与毫秒不同，请使用`java.time.Duration`而不是`long`并描述默认单位，例如“如果未指定持续时间后缀，将使用秒”。
- 除非必须在运行时确定默认值，否则请不要在描述中提供默认值。

确保[触发元数据生成，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata-annotation-processor)以便IDE协助也可用于您的密钥。您可能需要查看生成的元数据（`META-INF/spring-configuration-metadata.json`），以确保正确记录了您的密钥。在兼容的IDE中使用自己的启动程序也是验证元数据质量的一个好主意。

#### 28.5.3。`autoconfigure`模组

该`autoconfigure`模块包含开始使用该库所需的所有内容。它还可能包含配置键定义（例如`@ConfigurationProperties`）和可用于进一步自定义组件初始化方式的任何回调接口。

|      | 您应该将对库的依赖项标记为可选，以便可以`autoconfigure`更轻松地将模块包含在项目中。如果这样做，将不提供该库，并且默认情况下，Spring Boot将退出。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring Boot使用注释处理器在元数据文件（`META-INF/spring-autoconfigure-metadata.properties`）中收集自动配置的条件。如果存在该文件，它将用于急切过滤不匹配的自动配置，这将缩短启动时间。建议在包含自动配置的模块中添加以下依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure-processor</artifactId>
    <optional>true</optional>
</dependency>
```

对于Gradle 4.5及更早版本，应在`compileOnly`配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    compileOnly "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

对于Gradle 4.6和更高版本，应在`annotationProcessor`配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    annotationProcessor "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

#### 28.5.4。启动模块

起动器确实是一个空罐子。其唯一目的是提供必要的依赖关系以使用库。您可以将其视为对入门所需的看法。

不要对添加了启动器的项目做出假设。如果您要自动配置的库通常需要其他启动器，请也提及它们。如果可选依赖项的数量很高，则提供一组适当的*默认*依赖项可能会很困难，因为您应避免包括对于库的典型用法而言不必要的依赖项。换句话说，您不应包括可选的依赖项。

|      | 无论哪种方式，您的启动程序都必须`spring-boot-starter`直接或间接引用核心Spring Boot启动程序（）（即，如果您的启动程序依赖于另一个启动程序，则无需添加它）。如果仅使用您的自定义启动程序创建项目，则通过使用核心启动程序将尊重Spring Boot的核心功能。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 29. Kotlin支持

[Kotlin](https://kotlinlang.org/)是针对JVM（和其他平台）的静态类型的语言，它允许编写简洁明了的代码，同时提供与用Java编写的现有库的[互操作性](https://kotlinlang.org/docs/reference/java-interop.html)。

Spring Boot通过利用其他Spring项目（如Spring Framework，Spring Data和Reactor）中的支持来提供Kotlin支持。有关更多信息，请参见[Spring Framework Kotlin支持文档](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/languages.html#kotlin)。

开始使用Spring Boot和Kotlin的最简单方法是遵循[此综合教程](https://spring.io/guides/tutorials/spring-boot-kotlin/)。您可以通过[start.spring.io](https://start.spring.io/#!language=kotlin)创建新的Kotlin项目。如果需要支持，请随时加入[Kotlin Slack](https://slack.kotlinlang.org/)的#spring频道，或通过[Stack Overflow](https://stackoverflow.com/questions/tagged/spring+kotlin)上的`spring`and `kotlin`标签提问。

### 29.1。要求

Spring Boot支持Kotlin1.3.x。要使用Kotlin，`org.jetbrains.kotlin:kotlin-stdlib`并且`org.jetbrains.kotlin:kotlin-reflect`必须存在于类路径中。该`kotlin-stdlib`变种`kotlin-stdlib-jdk7`和`kotlin-stdlib-jdk8`也可以使用。

由于[默认情况下Kotlin类是final类](https://discuss.kotlinlang.org/t/classes-final-by-default/166)，因此您可能需要配置[kotlin-spring](https://kotlinlang.org/docs/reference/compiler-plugins.html#spring-support)插件，以便自动打开[带有](https://kotlinlang.org/docs/reference/compiler-plugins.html#spring-support) Spring注释的类，以便对其进行代理。

[在Kotlin](https://github.com/FasterXML/jackson-module-kotlin)中序列化/反序列化JSON数据时，需要[Jackson的Kotlin模块](https://github.com/FasterXML/jackson-module-kotlin)。在类路径上找到它会自动注册。如果存在Jackson和Kotlin但不存在Jackson Kotlin模块，则会记录一条警告消息。

|      | 如果在[start.spring.io](https://start.spring.io/#!language=kotlin)上引导Kotlin项目，[则](https://start.spring.io/#!language=kotlin)默认情况下会提供这些依赖项和插件。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 29.2。零安全

Kotlin的主要功能之一是[null安全](https://kotlinlang.org/docs/reference/null-safety.html)。它`null`在编译时处理值，而不是将问题推迟到运行时并遇到`NullPointerException`。这有助于消除常见的bug来源，而无需支付包装费用，例如`Optional`。Kotlin还允许使用具有可为空值的函数构造，如本[Kotlin中关于null安全性的全面指南中所述](https://www.baeldung.com/kotlin-null-safety)。

尽管Java不允许人们在其类型系统中表示null安全，但是Spring Framework，Spring Data和Reactor现在通过易于使用工具的批注为API提供了null安全。默认情况下，将Kotlin中使用的Java API中的[类型](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)识别为放松了空检查的[平台类型](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)。 [Kotlin对JSR 305批注的支持](https://kotlinlang.org/docs/reference/java-interop.html#jsr-305-support)与可空性[批注](https://kotlinlang.org/docs/reference/java-interop.html#jsr-305-support)相结合，为Kotlin中的相关Spring API提供了空安全性。

的JSR 305检查可以通过添加被配置`-Xjsr305`具有以下选项的编译标志：`-Xjsr305={strict|warn|ignore}`。默认行为与相同`-Xjsr305=warn`。`strict`从Spring API推断得出的Kotlin类型中，必须考虑到该值的空安全性，但应该使用该值，但要知道Spring API的空性声明即使在次要发行版之间也会发展，并且将来可能会添加更多检查。

|      | 尚不支持通用类型参数，varargs和数组元素的可空性。有关最新信息，请参见[SPR-15942](https://jira.spring.io/browse/SPR-15942)。另外请注意，Spring Boot自己的API [尚未被注释](https://github.com/spring-projects/spring-boot/issues/10712)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 29.3。Kotlin API

#### 29.3.1。runApplication

Spring Boot提供了一种惯用的方式来运行应用程序，`runApplication(*args)`如下例所示：

```kotlin
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
    runApplication<MyApplication>(*args)
}
```

这是的直接替代`SpringApplication.run(MyApplication::class.java, *args)`。它还允许自定义应用程序，如以下示例所示：

```kotlin
runApplication<MyApplication>(*args) {
    setBannerMode(OFF)
}
```

#### 29.3.2。扩展名

Kotlin [扩展](https://kotlinlang.org/docs/reference/extensions.html)提供了使用其他功能扩展现有类的功能。Spring Boot Kotlin API利用这些扩展为现有的API添加了新的Kotlin特定的便利。

`TestRestTemplate`提供了与Spring Framework中为`RestOperations`Spring Framework 提供的扩展类似的扩展。除其他事项外，这些扩展使利用Kotlin修饰类型参数成为可能。

### 29.4。依赖管理

为了避免在类路径中混合使用不同版本的Kotlin依赖项，Spring Boot会导入Kotlin BOM。

使用Maven，可以通过`kotlin.version`属性自定义Kotlin版本，并为提供插件管理`kotlin-maven-plugin`。使用Gradle，Spring Boot插件会自动将其`kotlin.version`与Kotlin插件的版本对齐。

Spring Boot还通过导入Kotlin Coroutines BOM管理Coroutines依赖项的版本。可以通过`kotlin-coroutines.version`属性自定义版本。

|      | `org.jetbrains.kotlinx:kotlinx-coroutines-reactor`如果一个引导程序对[start.spring.io](https://start.spring.io/#!language=kotlin)至少有一个反应依赖的Kotlin项目，则默认提供依赖。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 29.5。 `@ConfigurationProperties`

`@ConfigurationProperties`当与[`@ConstructorBinding`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-constructor-binding)具有不可变`val`属性的支持类结合使用时，如以下示例所示：

```kotlin
@ConstructorBinding
@ConfigurationProperties("example.kotlin")
data class KotlinExampleProperties(
        val name: String,
        val description: String,
        val myService: MyService) {

    data class MyService(
            val apiToken: String,
            val uri: URI
    )
}
```

|      | 要生成[自己的元数据](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata-annotation-processor)使用注释处理器，[`kapt`应该配置](https://kotlinlang.org/docs/reference/kapt.html)与`spring-boot-configuration-processor`依赖。请注意，由于kapt提供的模型的限制，某些功能（例如检测默认值或不推荐使用的项目）无法正常工作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 29.6。测试中

虽然可以使用JUnit 4测试Kotlin代码，但默认情况下建议使用JUnit 5。JUnit 5使一个测试类可以实例化一次，然后可用于该类的所有测试。这样就可以在非静态方法上使用`@BeforeClass`和`@AfterClass`注释，这非常适合Kotlin。

JUnit 5是默认的，并且提供了vintage引擎是为了与JUnit 4向后兼容。如果不使用它，请排除`org.junit.vintange:junit-vintage-engine`。您还需要将[测试实例生命周期切换为“ per-class”](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle-changing-default)。

要模拟Kotlin类，建议使用[MockK](https://mockk.io/)。如果您需要`Mockk`Mockito特定于[`@MockBean`和`@SpyBean`注释](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-testing-spring-boot-applications-mocking-beans)的等效项，则可以使用[SpringMockK](https://github.com/Ninja-Squad/springmockk)，它提供相似的`@MockkBean`和`@SpykBean`注释。

### 29.7。资源资源

#### 29.7.1。进一步阅读

- [Kotlin语言参考](https://kotlinlang.org/docs/reference/)
- [Kotlin Slack](https://kotlinlang.slack.com/)（带有专用的#spring频道）
- [带`spring`和`kotlin`标签的Stackoverflow](https://stackoverflow.com/questions/tagged/spring+kotlin)
- [在浏览器中尝试Kotlin](https://try.kotlinlang.org/)
- [Kotlin博客](https://blog.jetbrains.com/kotlin/)
- [很棒的科特林](https://kotlin.link/)
- [教程：使用Spring Boot和Kotlin构建Web应用程序](https://spring.io/guides/tutorials/spring-boot-kotlin/)
- [使用Kotlin开发Spring Boot应用程序](https://spring.io/blog/2016/02/15/developing-spring-boot-applications-with-kotlin)
- [带有Kotlin，Spring Boot和PostgreSQL的地理空间Messenger](https://spring.io/blog/2016/03/20/a-geospatial-messenger-with-kotlin-spring-boot-and-postgresql)
- [在Spring Framework 5.0中引入Kotlin支持](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0)
- [Spring Framework 5 Kotlin API的功能方式](https://spring.io/blog/2017/08/01/spring-framework-5-kotlin-apis-the-functional-way)

#### 29.7.2。例子

- [spring-boot-kotlin-demo](https://github.com/sdeleuze/spring-boot-kotlin-demo)：常规Spring Boot + Spring Data JPA项目
- [mixit](https://github.com/mixitconf/mixit)：Spring Boot 2 + WebFlux +响应式Spring Data MongoDB
- [spring-kotlin-fullstack](https://github.com/sdeleuze/spring-kotlin-fullstack)：WebFlux Kotlin完整示例，其中Kotlin2js用于前端，而不是JavaScript或TypeScript
- [spring-petclinic-kotlin](https://github.com/spring-petclinic/spring-petclinic-kotlin)：Spring PetClinic示例应用程序的Kotlin版本
- [spring-kotlin-deepdive](https://github.com/sdeleuze/spring-kotlin-deepdive)：从Boot 1.0 + Java到Boot 2.0 + Kotlin的逐步迁移
- [spring-boot-coroutines-demo](https://github.com/sdeleuze/spring-boot-coroutines-demo)：[协程](https://github.com/sdeleuze/spring-boot-coroutines-demo)示例项目

## 30.下一步阅读

如果您想了解有关本节中讨论的任何类的更多信息，可以查看[Spring Boot API文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api/)或[直接](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE)浏览[源代码](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE)。如果您有特定问题，请查看[操作方法](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto)部分。

如果您熟悉Spring Boot的核心功能，则可以继续阅读有关[准备就绪的功能的信息](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready)。

最后更新时间2020-03-26 11:43:46 UTC

<iframe id="blockbyte-bs-sidebar" class="notranslate" data-pos="left" style="box-sizing: border-box; opacity: 0; pointer-events: none; position: fixed; top: 0px; left: 0px; width: 350px; max-width: none; height: 0px; z-index: 2147483646; border: none; transform: translate3d(-350px, 0px, 0px); transition: width 0s ease 0.3s, height 0s ease 0.3s, opacity 0.3s ease 0s, transform 0.3s ease 0s; background-color: rgba(255, 255, 255, 0.8) !important; display: block !important; color: rgb(0, 0, 0); font-family: Karla, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;"></iframe>



# [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready)

[返回索引](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/index.html)

1. [1.启用生产就绪功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-enabling)
2. 2.端点
   1. 
   2. 
   3. 
   4. 
   5. 
   6. 
   7. 
      1. 
         1. 
      2. 
         1. 
         2. 
         3. 
         4. 
         5. 
         6. 
         7. 
         8. 
      3. 
      4. 
   8. 
      1. 
      2. 
      3. 
      4. 
      5. 
   9. 
      1. 
      2. 
      3. 
      4. 
      5. 
3. 3.通过HTTP进行监视和管理
   1. 
   2. 
   3. 
   4. 
   5. 
4. 4.通过JMX进行监视和管理
   1. 
   2. 
   3. 
      1. 
      2. 
5. 5.记录仪
   1. 
6. 6.指标
   1. 
   2. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
      7. 
      8. 
      9. 
      10. 
      11. 
      12. 
      13. 
      14. 
      15. 
      16. 
      17. 
   3. 
      1. 
      2. 
      3. 
      4. 
      5. 
      6. 
      7. 
      8. 
   4. 
   5. 
      1. 
      2. 
   6. 
7. 7.审核
   1. 
8. \8. HTTP跟踪
   1. 
9. 9.过程监控
   1. 
   2. 
10. \10. Cloud Foundry支持
    1. 
    2. 
    3. 
11. [11.接下来要读什么](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-whats-next)

Spring Boot包含许多其他功能，可在您将应用程序投入生产时帮助您监视和管理应用程序。您可以选择使用HTTP端点或JMX管理和监视应用程序。审核，运行状况和指标收集也可以自动应用于您的应用程序。

## 1.启用生产就绪功能

该[`spring-boot-actuator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator)模块提供了Spring Boot生产就绪的所有功能。启用功能的最简单方法是向`spring-boot-starter-actuator`“启动器” 添加依赖项。

执行器的定义

致动器是制造术语，是指用于移动或控制某些物体的机械设备。执行器可以通过很小的变化产生大量的运动。

要将执行器添加到基于Maven的项目中，请添加以下“ Starter”依赖项：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

对于Gradle，请使用以下声明：

```groovy
dependencies {
    compile("org.springframework.boot:spring-boot-starter-actuator")
}
```

## 2.端点

执行器端点使您可以监视应用程序并与之交互。Spring Boot包含许多内置端点，可让您添加自己的端点。例如，`health`端点提供基本的应用程序运行状况信息。

每个端点都可以[启用或禁用](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-enabling-endpoints)。这控制了是否创建了端点以及它的bean在应用程序上下文中是否存在。为了可以远程访问，端点还必须[通过JMX或HTTP公开](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints)。大多数应用程序选择HTTP，其中终结点的ID和前缀`/actuator`映射到URL。例如，默认情况下，`health`端点映射到`/actuator/health`。

可以使用以下与技术无关的端点：

| ID                 | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `auditevents`      | 公开当前应用程序的审核事件信息。需要一个`AuditEventRepository`豆。 |
| `beans`            | 显示应用程序中所有Spring Bean的完整列表。                    |
| `caches`           | 公开可用的缓存。                                             |
| `conditions`       | 显示在配置和自动配置类上评估的条件以及它们匹配或不匹配的原因。 |
| `configprops`      | 显示所有的整理列表`@ConfigurationProperties`。               |
| `env`              | 公开Spring的属性`ConfigurableEnvironment`。                  |
| `flyway`           | 显示已应用的所有Flyway数据库迁移。需要一个或多个`Flyway`豆。 |
| `health`           | 显示应用程序运行状况信息。                                   |
| `httptrace`        | 显示HTTP跟踪信息（默认情况下，最近100个HTTP请求-响应交换）。需要一个`HttpTraceRepository`豆。 |
| `info`             | 显示任意应用程序信息。                                       |
| `integrationgraph` | 显示Spring Integration图。需要对的依赖`spring-integration-core`。 |
| `loggers`          | 显示和修改应用程序中记录器的配置。                           |
| `liquibase`        | 显示已应用的所有Liquibase数据库迁移。需要一个或多个`Liquibase`豆。 |
| `metrics`          | 显示当前应用程序的“指标”信息。                               |
| `mappings`         | 显示所有`@RequestMapping`路径的整理列表。                    |
| `scheduledtasks`   | 显示应用程序中的计划任务。                                   |
| `sessions`         | 允许从Spring Session支持的会话存储中检索和删除用户会话。需要使用Spring Session的基于Servlet的Web应用程序。 |
| `shutdown`         | 使应用程序正常关闭。默认禁用。                               |
| `threaddump`       | 执行线程转储。                                               |

如果您的应用程序是Web应用程序（Spring MVC，Spring WebFlux或Jersey），则可以使用以下附加端点：

| ID           | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| `heapdump`   | 返回`hprof`堆转储文件。                                      |
| `jolokia`    | 通过HTTP公开JMX bean（当Jolokia在类路径上时，不适用于WebFlux）。需要对的依赖`jolokia-core`。 |
| `logfile`    | 返回日志文件的内容（如果已设置`logging.file.name`或`logging.file.path`属性）。支持使用HTTP `Range`标头来检索部分日志文件的内容。 |
| `prometheus` | 以Prometheus服务器可以抓取的格式公开指标。需要对的依赖`micrometer-registry-prometheus`。 |

要了解有关执行器端点及其请求和响应格式的更多信息，请参阅单独的API文档（[HTML](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/actuator-api//html/)或[PDF](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）。

### 2.1。启用端点

默认情况下，除之外的所有端点`shutdown`均处于启用状态。要配置端点的启用，请使用其`management.endpoint..enabled`属性。以下示例启用`shutdown`端点：

```properties
management.endpoint.shutdown.enabled=true
```

如果您希望启用端点启用而不是退出启用，请将该`management.endpoints.enabled-by-default`属性设置为，`false`并使用各个端点`enabled`属性重新启用。以下示例启用该`info`端点并禁用所有其他端点：

```properties
management.endpoints.enabled-by-default=false
management.endpoint.info.enabled=true
```

|      | 禁用的端点将从应用程序上下文中完全删除。如果只想更改公开端点的技术，请改用[`include`和`exclude`属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2。暴露端点

由于端点可能包含敏感信息，因此应谨慎考虑何时公开它们。下表显示了内置端点的默认暴露：

| ID                 | JMX    | 网页 |
| :----------------- | :----- | :--- |
| `auditevents`      | 是     | 没有 |
| `beans`            | 是     | 没有 |
| `caches`           | 是     | 没有 |
| `conditions`       | 是     | 没有 |
| `configprops`      | 是     | 没有 |
| `env`              | 是     | 没有 |
| `flyway`           | 是     | 没有 |
| `health`           | 是     | 是   |
| `heapdump`         | 不适用 | 没有 |
| `httptrace`        | 是     | 没有 |
| `info`             | 是     | 是   |
| `integrationgraph` | 是     | 没有 |
| `jolokia`          | 不适用 | 没有 |
| `logfile`          | 不适用 | 没有 |
| `loggers`          | 是     | 没有 |
| `liquibase`        | 是     | 没有 |
| `metrics`          | 是     | 没有 |
| `mappings`         | 是     | 没有 |
| `prometheus`       | 不适用 | 没有 |
| `scheduledtasks`   | 是     | 没有 |
| `sessions`         | 是     | 没有 |
| `shutdown`         | 是     | 没有 |
| `threaddump`       | 是     | 没有 |

到端点暴露的变化，使用下面的特定技术`include`和`exclude`特性：

| 属性                                        | 默认           |
| :------------------------------------------ | :------------- |
| `management.endpoints.jmx.exposure.exclude` |                |
| `management.endpoints.jmx.exposure.include` | `*`            |
| `management.endpoints.web.exposure.exclude` |                |
| `management.endpoints.web.exposure.include` | `info, health` |

该`include`属性列出了公开的端点的ID。该`exclude`属性列出了不应公开的端点的ID。该`exclude`属性优先于该`include`属性。无论`include`和`exclude`性能可与端点ID列表进行配置。

例如，要停止通过JMX公开所有端点，而仅公开`health`和`info`端点，请使用以下属性：

```properties
management.endpoints.jmx.exposure.include=health,info
```

`*`可用于选择所有端点。例如，要通过HTTP公开除`env`和`beans`端点之外的所有内容，请使用以下属性：

```properties
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=env,beans
```

|      | `*` 在YAML中具有特殊含义，因此，如果要包括（或排除）所有端点，请确保添加引号，如以下示例所示：`management:  endpoints:    web:      exposure:        include: "*"` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您的应用程序公开公开，我们强烈建议您也[保护端点](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-security)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果要实现暴露端点的自己的策略，则可以注册`EndpointFilter`Bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.3。保护HTTP端点

您应该像对待其他任何敏感URL一样，小心保护HTTP端点的安全。如果存在Spring Security，则默认情况下使用Spring Security的内容协商策略保护端点的安全。例如，如果您希望为HTTP端点配置自定义安全性，只允许具有特定角色的用户访问它们，则Spring Boot提供了一些方便的`RequestMatcher`对象，这些对象可以与Spring Security结合使用。

典型的Spring Security配置可能类似于以下示例：

```java
@Configuration(proxyBeanMethods = false)
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests((requests) ->
                requests.anyRequest().hasRole("ENDPOINT_ADMIN"));
        http.httpBasic();
    }

}
```

前面的示例用于`EndpointRequest.toAnyEndpoint()`将请求匹配到任何端点，然后确保所有`ENDPOINT_ADMIN`角色都具有该角色。上还有其他几种匹配器方法`EndpointRequest`。有关详细信息，请参见API文档（[HTML](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/actuator-api//html)或[PDF](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）。

如果将应用程序部署在防火墙后面，则可能希望无需身份验证即可访问所有执行器端点。您可以通过更改`management.endpoints.web.exposure.include`属性来做到这一点，如下所示：

application.properties

```properties
management.endpoints.web.exposure.include=*
```

另外，如果存在Spring Security，则需要添加自定义安全配置，该配置允许未经身份验证的端点访问，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
public class ActuatorSecurity extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.requestMatcher(EndpointRequest.toAnyEndpoint()).authorizeRequests((requests) ->
            requests.anyRequest().permitAll());
    }

}
```

### 2.4。配置端点

端点自动缓存对不带任何参数的读取操作的响应。要配置端点缓存响应的时间，请使用其`cache.time-to-live`属性。以下示例将`beans`端点的缓存的生存时间设置为10秒：

application.properties

```properties
management.endpoint.beans.cache.time-to-live=10s
```

|      | 该前缀`management.endpoint.`用于唯一标识正在配置的端点。 |
| ---- | -------------------------------------------------------- |
|      |                                                          |

|      | 发出经过身份验证的HTTP请求时，`Principal`会将视为端点的输入，因此将不缓存响应。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.5。用于执行器Web端点的超媒体

添加了一个“发现页面”，其中包含指向所有端点的链接。`/actuator`默认情况下，“发现页面”可用。

配置了自定义管理上下文路径后，“发现页面”会自动从`/actuator`管理上下文的根目录移到。例如，如果管理上下文路径为`/management`，则发现页面可从访问`/management`。将管理上下文路径设置为时`/`，将禁用发现页面，以防止与其他映射发生冲突的可能性。

### 2.6。CORS支持

[跨域资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是[W3C规范](https://www.w3.org/TR/cors/)，可让您灵活地指定授权哪种类型的跨域请求。如果您使用Spring MVC或Spring WebFlux，则可以将Actuator的Web端点配置为支持此类方案。

默认情况下，CORS支持是禁用的，并且仅`management.endpoints.web.cors.allowed-origins`在设置属性后才启用。以下配置允许`GET`和`POST`从`example.com`域调用：

```properties
management.endpoints.web.cors.allowed-origins=https://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```

|      | 有关选项的完整列表，请参见[CorsEndpointProperties](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/endpoint/web/CorsEndpointProperties.java)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.7。实施自定义端点

如果添加`@Bean`带注释`@Endpoint`，带注释的任何方法`@ReadOperation`，`@WriteOperation`或`@DeleteOperation`在JMX会自动显示，并在Web应用程序，以及通过HTTP。可以使用Jersey，Spring MVC或Spring WebFlux通过HTTP公开端点。如果Jersey和Spring MVC均可用，则将使用Spring MVC。

您也可以使用`@JmxEndpoint`或编写特定于技术的端点`@WebEndpoint`。这些端点仅限于各自的技术。例如，`@WebEndpoint`仅通过HTTP而不是JMX公开。

您可以使用`@EndpointWebExtension`和编写特定于技术的扩展`@EndpointJmxExtension`。这些注释使您可以提供特定于技术的操作以扩展现有端点。

最后，如果需要访问特定于Web框架的功能，则可以实现Servlet或Spring `@Controller`和`@RestController`终结点，但要付出代价，即它们无法通过JMX或使用其他Web框架使用。

#### 2.7.1。接收输入

端点上的操作通过其参数接收输入。通过网络公开时，这些参数的值取自URL的查询参数和JSON请求正文。通过JMX公开时，参数将映射到MBean操作的参数。默认情况下，参数是必需的。可以使用注释它们，使它们成为可选的`@org.springframework.lang.Nullable`。

JSON请求正文中的每个根属性都可以映射到端点的参数。考虑以下JSON请求正文：

```json
{
    "name": "test",
    "counter": 42
}
```

这可用于调用带有`String name`和`int counter`参数的写操作。

|      | 由于端点与技术无关，因此只能在方法签名中指定简单类型。特别是，不支持使用定义了`name`和`counter`属性的自定义类型声明单个参数。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 为了使输入映射到操作方法的参数，应使用编译实现端点的Java代码`-parameters`，并用编译实现端点的Kotlin代码`-java-parameters`。如果您使用的是Spring Boot的Gradle插件，或者您使用的是Maven和，这将自动发生`spring-boot-starter-parent`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### 输入类型转换

如有必要，传递给端点操作方法的参数会自动转换为所需的类型。在调用操作方法之前，通过JMX或HTTP请求接收到的输入将使用的实例`ApplicationConversionService`以及带有`Converter`或条件的`GenericConverter`Bean 转换为所需的类型`@EndpointConverter`。

#### 2.7.2。自定义Web端点

上的操作`@Endpoint`，`@WebEndpoint`或者`@EndpointWebExtension`使用新泽西州，Spring MVC的，或Spring WebFlux自动曝光通过HTTP。如果Jersey和Spring MVC均可用，则将使用Spring MVC。

##### Web端点请求谓词

对于在暴露于Web的终结点上的每个操作，都会自动生成一个请求谓词。

##### 路径

谓词的路径由终结点的ID和暴露于Web的终结点的基本路径确定。默认基本路径为`/actuator`。例如，具有ID的端点`sessions`将`/actuator/sessions`用作谓词中的路径。

通过使用注释操作方法的一个或多个参数，可以进一步自定义路径`@Selector`。将这样的参数作为路径变量添加到路径谓词。当端点操作被调用时，变量的值被传递到操作方法中。如果要捕获所有剩余的路径元素，则可以添加`@Selector(Match=ALL_REMAINING)`到最后一个参数，并将其设置为与转换兼容的类型`String[]`。

##### HTTP方法

谓词的HTTP方法由操作类型决定，如下表所示：

| 运作方式           | HTTP方法 |
| :----------------- | :------- |
| `@ReadOperation`   | `GET`    |
| `@WriteOperation`  | `POST`   |
| `@DeleteOperation` | `DELETE` |

##### 消耗

对于使用请求正文的`@WriteOperation`（HTTP `POST`），谓词的消耗子句为`application/vnd.spring-boot.actuator.v2+json, application/json`。对于所有其他操作，消耗子句为空。

##### 产生

的产生谓词子句可以由被确定`produces`的属性`@DeleteOperation`，`@ReadOperation`和`@WriteOperation`注解。该属性是可选的。如果未使用，则会自动确定produces子句。

如果操作方法返回`void`或`Void`生产子句为空。如果操作方法返回a `org.springframework.core.io.Resource`，则Produces子句为`application/octet-stream`。对于所有其他操作，produces子句为`application/vnd.spring-boot.actuator.v2+json, application/json`。

##### Web端点响应状态

端点操作的默认响应状态取决于操作类型（读，写或删除）以及该操作返回的内容（如果有）。

A `@ReadOperation`返回一个值，响应状态将为200（确定）。如果未返回值，则响应状态将为404（未找到）。

如果a `@WriteOperation`或`@DeleteOperation`返回值，则响应状态将为200（确定）。如果未返回值，则响应状态将为204（无内容）。

如果在没有必需参数或无法将参数转换为必需类型的参数的情况下调用操作，则不会调用该操作方法，并且响应状态将为400（错误请求）。

##### Web端点范围请求

HTTP范围请求可用于请求HTTP资源的一部分。使用Spring MVC或Spring Web Flux时，返回`org.springframework.core.io.Resource`自动支持范围请求的操作。

|      | 使用Jersey时，范围请求不受支持。 |
| ---- | -------------------------------- |
|      |                                  |

##### Web端点安全

Web终结点或特定于Web的终结点扩展上的操作可以接收当前参数`java.security.Principal`或`org.springframework.boot.actuate.endpoint.SecurityContext`方法参数。前者通常与结合使用`@Nullable`，以为经过身份验证和未经身份验证的用户提供不同的行为。后者通常用于使用其`isUserInRole(String)`方法执行授权检查。

#### 2.7.3。Servlet端点

阿`Servlet`可以公开为通过实施与注释的一个类的端点`@ServletEndpoint`也实现`Supplier`。Servlet端点提供了与Servlet容器的更深层集成，但以可移植性为代价。它们旨在用于将现有对象公开`Servlet`为端点。对于新的端点，应尽可能使用`@Endpoint`和`@WebEndpoint`注释。

#### 2.7.4。控制器端点

`@ControllerEndpoint`并且`@RestControllerEndpoint`可以用于实现仅由Spring MVC或Spring WebFlux公开的端点。使用Spring MVC和Spring WebFlux的标准注释（例如`@RequestMapping`和）映射方法`@GetMapping`，并将端点的ID用作路径的前缀。控制器端点提供了与Spring Web框架的更深层集成，但以可移植性为代价。的`@Endpoint`和`@WebEndpoint`注解应当优选只要有可能。

### 2.8。健康资讯

您可以使用运行状况信息来检查正在运行的应用程序的状态。监视软件通常使用它在生产系统出现故障时向某人发出警报。`health`端点公开的信息取决于`management.endpoint.health.show-details`和`management.endpoint.health.show-components`属性，可以使用以下值之一对其进行配置：

| 名称              | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `never`           | 详细信息永远不会显示。                                       |
| `when-authorized` | 详细信息仅显示给授权用户。可以使用配置授权角色`management.endpoint.health.roles`。 |
| `always`          | 向所有用户显示详细信息。                                     |

默认值为`never`。当用户担任一个或多个端点的角色时，该用户被视为已授权。如果端点没有配置的角色（默认值），则所有通过身份验证的用户均被视为已授权。可以使用`management.endpoint.health.roles`属性配置角色。

|      | 如果您已保护应用程序安全并希望使用`always`，则安全配置必须允许经过身份验证的用户和未经身份验证的用户都可以访问运行状况端点。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

健康信息是从中收集的[`HealthContributorRegistry`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthContributorRegistry.java)（默认情况[`HealthContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthContributor.java)下，您的中定义的所有实例`ApplicationContext`）的信息。Spring Boot包括许多自动配置的`HealthContributors`，您也可以编写自己的。

A `HealthContributor`可以是a `HealthIndicator`或a `CompositeHealthContributor`。A `HealthIndicator`提供实际的健康信息，包括`Status`。A `CompositeHealthContributor`提供其他的组合`HealthContributors`。合起来，贡献者形成一个树形结构来代表整个系统的健康状况。

默认情况下，最终的系统健康状况是由a派生的，`StatusAggregator`它根据状态`HealthIndicator`的有序列表对每个状态进行排序。排序列表中的第一个状态用作整体运行状况。如果没有`HealthIndicator`返回一个已知状态`StatusAggregator`，一种`UNKNOWN`是使用状态。

|      | 该`HealthContributorRegistry`可用于在运行时注册和注销的健康指标。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.1。自动配置的健康指标

`HealthIndicators`在适当的情况下，Spring Boot会自动配置以下内容：

| 名称                                                         | 描述                               |
| :----------------------------------------------------------- | :--------------------------------- |
| [`CassandraHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraHealthIndicator.java) | 检查Cassandra数据库是否已启动。    |
| [`CouchbaseHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseHealthIndicator.java) | 检查Couchbase群集是否已启动。      |
| [`DiskSpaceHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.java) | 检查磁盘空间不足。                 |
| [`DataSourceHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.java) | 检查是否可以建立连接`DataSource`。 |
| [`ElasticSearchRestHealthContributorAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/elasticsearch/ElasticSearchRestHealthContributorAutoConfiguration.java) | 检查Elasticsearch集群是否已启动。  |
| [`HazelcastHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/hazelcast/HazelcastHealthIndicator.java) | 检查Hazelcast服务器是否已启动。    |
| [`InfluxDbHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/influx/InfluxDbHealthIndicator.java) | 检查InfluxDB服务器是否已启动。     |
| [`JmsHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jms/JmsHealthIndicator.java) | 检查JMS代理是否启动。              |
| [`LdapHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/ldap/LdapHealthIndicator.java) | 检查LDAP服务器是否已启动。         |
| [`MailHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mail/MailHealthIndicator.java) | 检查邮件服务器是否已启动。         |
| [`MongoHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoHealthIndicator.java) | 检查Mongo数据库是否已启动。        |
| [`Neo4jHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/neo4j/Neo4jHealthIndicator.java) | 检查Neo4j数据库是否启动。          |
| [`PingHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/PingHealthIndicator.java) | 一律以回应`UP`。                   |
| [`RabbitHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/amqp/RabbitHealthIndicator.java) | 检查Rabbit服务器是否已启动。       |
| [`RedisHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisHealthIndicator.java) | 检查Redis服务器是否已启动。        |
| [`SolrHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/solr/SolrHealthIndicator.java) | 检查Solr服务器是否已启动。         |

|      | 您可以通过设置`management.health.defaults.enabled`属性来全部禁用它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.2。编写自定义健康指标

为了提供定制的健康信息，您可以注册实现该[`HealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthIndicator.java)接口的Spring bean 。您需要提供该`health()`方法的实现并返回`Health`响应。的`Health`响应应该包括一个状态，并且可以任选地包括另外的细节被显示。以下代码显示了示例`HealthIndicator`实现：

```java
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int errorCode = check(); // perform some specific health check
        if (errorCode != 0) {
            return Health.down().withDetail("Error Code", errorCode).build();
        }
        return Health.up().build();
    }

}
```

|      | 给定的标识符`HealthIndicator`是不带`HealthIndicator`后缀的Bean的名称（如果存在）。在前面的示例中，健康信息在名为的条目中可用`my`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

除了Spring Boot的预定义[`Status`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/Status.java)类型之外，还可以`Health`返回`Status`代表新系统状态的自定义。在这种情况下，[`StatusAggregator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/StatusAggregator.java)还需要提供接口的自定义实现，或者必须使用`management.endpoint.health.status.order`配置属性来配置默认实现。

例如，假设在您的一种实现中使用了`Status`带有代码的new 。要配置严重性顺序，请将以下属性添加到您的应用程序属性中：`FATAL``HealthIndicator`

```properties
management.endpoint.health.status.order=fatal,down,out-of-service,unknown,up
```

在响应中的HTTP状态代码反映总体健康状况（例如，`UP`映射到200，而`OUT_OF_SERVICE`并`DOWN`映射到503）。如果通过HTTP访问运行状况终结点，则可能还需要注册自定义状态映射。例如，以下属性映射`FATAL`到503（服务不可用）：

```properties
management.endpoint.health.status.http-mapping.fatal=503
```

|      | 如果需要更多控制，则可以定义自己的`HttpCodeStatusMapper`bean。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下表显示了内置状态的默认状态映射：

| 状态     | 制图                                  |
| :------- | :------------------------------------ |
| 下       | SERVICE_UNAVAILABLE（503）            |
| 中止服务 | SERVICE_UNAVAILABLE（503）            |
| 上       | 默认情况下没有映射，因此http状态为200 |
| 未知     | 默认情况下没有映射，因此http状态为200 |

#### 2.8.3。反应性健康指标

对于反应式应用程序，例如使用Spring WebFlux的那些应用程序，`ReactiveHealthContributor`提供了非阻塞合同来获取应用程序的运行状况。与传统方法类似`HealthContributor`，健康信息是从的内容中收集的[`ReactiveHealthContributorRegistry`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthContributorRegistry.java)（默认情况下，所有[`HealthContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/HealthContributor.java)和[`ReactiveHealthContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthContributor.java)实例中定义的实例`ApplicationContext`）。`HealthContributors`在弹性调度程序上执行不检查反应式API的常规。

|      | 在反应式应用程序中，`ReactiveHealthContributorRegistry`应在运行时使用The 来注册和注销健康指标。如果需要注册常规`HealthContributor`，则应使用进行包装`ReactiveHealthContributor#adapt`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

为了从反应式API提供自定义的健康信息，您可以注册实现该[`ReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/health/ReactiveHealthIndicator.java)接口的Spring bean 。以下代码显示了示例`ReactiveHealthIndicator`实现：

```java
@Component
public class MyReactiveHealthIndicator implements ReactiveHealthIndicator {

    @Override
    public Mono<Health> health() {
        return doHealthCheck() //perform some specific health check that returns a Mono<Health>
            .onErrorResume(ex -> Mono.just(new Health.Builder().down(ex).build()));
    }

}
```

|      | 要自动处理错误，请考虑从扩展`AbstractReactiveHealthIndicator`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.4。自动配置的ReactiveHealthIndicators

`ReactiveHealthIndicators`在适当的情况下，Spring Boot会自动配置以下内容：

| 名称                                                         | 描述                            |
| :----------------------------------------------------------- | :------------------------------ |
| [`CassandraReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/cassandra/CassandraReactiveHealthIndicator.java) | 检查Cassandra数据库是否已启动。 |
| [`CouchbaseReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/couchbase/CouchbaseReactiveHealthIndicator.java) | 检查Couchbase群集是否已启动。   |
| [`MongoReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoReactiveHealthIndicator.java) | 检查Mongo数据库是否已启动。     |
| [`RedisReactiveHealthIndicator`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisReactiveHealthIndicator.java) | 检查Redis服务器是否已启动。     |

|      | 如有必要，可用无功指示器代替常规指示器。此外，任何`HealthIndicator`未明确处理的内容都会自动包装。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.8.5。健康团体

有时将健康指标组织到可以用于不同目的的组中很有用。例如，如果将应用程序部署到Kubernetes，则可能需要一组不同的运行状况指示器来进行“活动”和“就绪”探针。

要创建运行状况指示器组，可以使用该`management.endpoint.health.group.`属性并将运行状况指示器ID的列表指定为`include`或`exclude`。例如，要创建仅包含数据库指示符的组，可以定义以下内容：

```properties
management.endpoint.health.group.custom.include=db
```

然后，您可以点击来检查结果`localhost:8080/actuator/health/custom`。

默认情况下，集团将继承相同的`StatusAggregator`，并`HttpCodeStatusMapper`设置为系统运行状况，但是，这些也可以在每个组来定义。如果需要，也可以覆盖`show-details`和`roles`属性：

```properties
management.endpoint.health.group.custom.show-details=when-authorized
management.endpoint.health.group.custom.roles=admin
management.endpoint.health.group.custom.status.order=fatal,up
management.endpoint.health.group.custom.status.http-mapping.fatal=500
```

|      | 您可以使用`@Qualifier("groupname")`，如果您需要注册自定义`StatusAggregator`或`HttpCodeStatusMapper`豆类用于与该组使用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.9。应用信息

应用程序信息公开了从[`InfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java)您的中定义的所有bean 收集的各种信息`ApplicationContext`。Spring Boot包含许多自动配置的`InfoContributor`bean，您可以编写自己的bean。

#### 2.9.1。自动配置的信息贡献者

`InfoContributor`适当时，Spring Boot会自动配置以下bean：

| 名称                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`EnvironmentInfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/EnvironmentInfoContributor.java) | 从键`Environment`下方公开任何`info`键。                      |
| [`GitInfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/GitInfoContributor.java) | 如果`git.properties`文件可用，则公开git信息。                |
| [`BuildInfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/BuildInfoContributor.java) | 如果`META-INF/build-info.properties`文件可用，则公开构建信息。 |

|      | 通过设置该`management.info.defaults.enabled`属性，可以全部禁用它们。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.9.2。定制应用信息

您可以`info`通过设置`info.*`Spring属性来自定义端点公开的数据。键`Environment`下的所有属性`info`都将自动显示。例如，您可以将以下设置添加到`application.properties`文件中：

```properties
info.app.encoding=UTF-8
info.app.java.source=1.8
info.app.java.target=1.8
```

|      | 除了对这些值进行硬编码之外，您还可以[在构建时扩展info属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-automatic-expansion)。假设您使用Maven，则可以按如下所示重写前面的示例：`info.app.encoding=@project.build.sourceEncoding@ info.app.java.source=@java.version@ info.app.java.target=@java.version@` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.9.3。Git提交信息

`info`端点的另一个有用功能是它能够`git`在构建项目时发布有关源代码存储库状态的信息。如果`GitProperties`豆可用，`git.branch`，`git.commit.id`，和`git.commit.time`属性暴露出来。

|      | 一个`GitProperties`bean是自动配置，如果一个`git.properties`文件可在classpath的根目录。有关更多详细[信息，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-git-info)请参见“ [生成git信息](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-git-info) ”。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果要显示完整的git信息（即的完整内容`git.properties`），请使用`management.info.git.mode`属性，如下所示：

```properties
management.info.git.mode=full
```

#### 2.9.4。建造信息

如果有`BuildProperties`可用的bean，则`info`端点也可以发布有关构建的信息。如果`META-INF/build-info.properties`文件在类路径中可用，则会发生这种情况。

|      | Maven和Gradle插件都可以生成该文件。有关更多详细[信息，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-build-info)请参见“ [生成构建信息](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-build-info) ”。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.9.5。编写自定义信息贡献者

为了提供定制的应用程序信息，您可以注册实现该[`InfoContributor`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java)接口的Spring bean 。

以下示例`example`使用单个值贡献一个条目：

```java
import java.util.Collections;

import org.springframework.boot.actuate.info.Info;
import org.springframework.boot.actuate.info.InfoContributor;
import org.springframework.stereotype.Component;

@Component
public class ExampleInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        builder.withDetail("example",
                Collections.singletonMap("key", "value"));
    }

}
```

如果到达`info`端点，则应该看到包含以下附加条目的响应：

```json
{
    "example": {
        "key" : "value"
    }
}
```

## 3.通过HTTP进行监视和管理

如果您正在开发Web应用程序，则Spring Boot Actuator会自动配置所有启用的端点以通过HTTP公开。默认约定是使用`id`前缀为的端点的`/actuator`作为URL路径。例如，`health`暴露为`/actuator/health`。

|      | Spring MVC，Spring WebFlux和Jersey本身支持Actuator。如果Jersey和Spring MVC均可用，则将使用Spring MVC。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 为了获得API文档（[HTML](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/actuator-api//html)或[PDF](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/actuator-api//pdf/spring-boot-actuator-web-api.pdf)）中记录的正确JSON响应，Jackson是必需的依赖项。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.1。自定义管理端点路径

有时，自定义管理端点的前缀很有用。例如，您的应用程序可能已经`/actuator`用于其他用途。您可以使用该`management.endpoints.web.base-path`属性来更改管理端点的前缀，如以下示例所示：

```properties
management.endpoints.web.base-path=/manage
```

前面的`application.properties`示例将端点从更改`/actuator/{id}`为`/manage/{id}`（例如`/manage/info`）。

|      | 除非管理端口已经被配置为[通过使用不同的HTTP端口暴露端点](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-customizing-management-server-port)，`management.endpoints.web.base-path`相对于`server.servlet.context-path`。如果`management.server.port`已配置，`management.endpoints.web.base-path`则相对于`management.server.servlet.context-path`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果要将端点映射到其他路径，则可以使用该`management.endpoints.web.path-mapping`属性。

以下示例重新映射`/actuator/health`到`/healthcheck`：

application.properties

```properties
management.endpoints.web.base-path=/
management.endpoints.web.path-mapping.health=healthcheck
```

### 3.2。自定义管理服务器端口

对于基于云的部署，通过使用默认的HTTP端口公开管理端点是明智的选择。但是，如果您的应用程序在自己的数据中心内运行，则您可能更喜欢使用其他HTTP端口公开端点。

您可以设置该`management.server.port`属性以更改HTTP端口，如以下示例所示：

```properties
management.server.port=8081
```

|      | 在Cloud Foundry上，默认情况下，应用程序仅在端口8080上接收HTTP和TCP路由请求。如果要在Cloud Foundry上使用自定义管理端口，则需要显式设置应用程序的路由，以将流量转发到自定义端口。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.3。配置特定于管理的SSL

当配置为使用自定义端口时，还可以通过使用各种`management.server.ssl.*`属性为管理服务器配置其自己的SSL 。例如，这样做可以使管理服务器在主应用程序使用HTTPS时通过HTTP可用，如以下属性设置所示：

```properties
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:store.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=false
```

或者，主服务器和管理服务器都可以使用SSL，但具有不同的密钥库，如下所示：

```properties
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:main.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=true
management.server.ssl.key-store=classpath:management.jks
management.server.ssl.key-password=secret
```

### 3.4。自定义管理服务器地址

您可以通过设置`management.server.address`属性来自定义管理端点可用的地址。如果您只想在内部或面向操作的网络上侦听或仅侦听来自的连接，则这样做很有用`localhost`。

|      | 仅当端口与主服务器端口不同时，您才能在其他地址上侦听。 |
| ---- | ------------------------------------------------------ |
|      |                                                        |

以下示例`application.properties`不允许进行远程管理连接：

```properties
management.server.port=8081
management.server.address=127.0.0.1
```

### 3.5。禁用HTTP端点

如果您不想通过HTTP公开端点，则可以将管理端口设置为`-1`，如以下示例所示：

```properties
management.server.port=-1
```

也可以使用该`management.endpoints.web.exposure.exclude`属性来实现，如以下示例所示：

```properties
management.endpoints.web.exposure.exclude=*
```

## 4.通过JMX进行监视和管理

Java管理扩展（JMX）提供了监视和管理应用程序的标准机制。默认情况下，不启用该功能，可以通过设置配置属性开启`spring.jmx.enabled`到`true`。`org.springframework.boot`默认情况下，Spring Boot将管理端点公开为域下的JMX MBean 。

### 4.1。自定义MBean名称

MBean的名称通常是从`id`端点的生成的。例如，`health`端点显示为`org.springframework.boot:type=Endpoint,name=Health`。

如果您的应用程序包含多个Spring `ApplicationContext`，则可能会发现名称冲突。要解决此问题，可以将`spring.jmx.unique-names`属性设置为，`true`以便MBean名称始终是唯一的。

您还可以自定义暴露端点的JMX域。以下设置显示了这样做的示例`application.properties`：

```properties
spring.jmx.unique-names=true
management.endpoints.jmx.domain=com.example.myapp
```

### 4.2。禁用JMX端点

如果不想通过JMX公开端点，则可以将`management.endpoints.jmx.exposure.exclude`属性设置为`*`，如以下示例所示：

```properties
management.endpoints.jmx.exposure.exclude=*
```

### 4.3。通过HTTP将Jolokia用于JMX

Jolokia是一个JMX-HTTP桥，它提供了一种访问JMX Bean的替代方法。要使用Jolokia，请包含的依赖项`org.jolokia:jolokia-core`。例如，使用Maven，您将添加以下依赖项：

```xml
<dependency>
    <groupId>org.jolokia</groupId>
    <artifactId>jolokia-core</artifactId>
</dependency>
```

然后，可以通过在属性中添加`jolokia`或`*`来显示Jolokia端点`management.endpoints.web.exposure.include`。然后，您可以通过`/actuator/jolokia`在管理HTTP服务器上使用来访问它。

#### 4.3.1。自定义Jolokia

Jolokia具有许多设置，这些设置传统上是通过设置servlet参数进行配置的。通过Spring Boot，您可以使用您的`application.properties`文件。为此，请为参数加上前缀`management.endpoint.jolokia.config.`，如以下示例所示：

```properties
management.endpoint.jolokia.config.debug=true
```

#### 4.3.2。禁用Jolokia

如果您使用Jolokia但不希望Spring Boot对其进行配置，请将该`management.endpoint.jolokia.enabled`属性设置为`false`，如下所示：

```properties
management.endpoint.jolokia.enabled=false
```

## 5.记录仪

Spring Boot Actuator可以在运行时查看和配置应用程序的日志级别。您可以查看整个列表，也可以查看单个记录器的配置，该配置由显式配置的记录级别以及由记录框架赋予它的有效记录级别组成。这些级别可以是以下之一：

- `TRACE`
- `DEBUG`
- `INFO`
- `WARN`
- `ERROR`
- `FATAL`
- `OFF`
- `null`

`null` 表示没有显式配置。

### 5.1。配置记录器

要配置给定的记录器，`POST`它是资源URI的部分实体，如以下示例所示：

```json
{
    "configuredLevel": "DEBUG"
}
```

|      | 要“重置”记录器的特定级别（并使用默认配置），可以将值传递`null`为`configuredLevel`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 6.指标

Spring Boot Actuator为[Micrometer](https://micrometer.io/)提供了依赖项管理和自动配置，[Micrometer](https://micrometer.io/)是一种支持众多监视系统的应用程序指标外观，包括：

- [AppOptics](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-appoptics)
- [阿特拉斯](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-atlas)
- [数据狗](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-datadog)
- [动态痕迹](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-dynatrace)
- [有弹性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-elastic)
- [神经节](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-ganglia)
- [石墨](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-graphite)
- [悍马](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-humio)
- [涌入](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-influx)
- [JMX](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-jmx)
- [Kairos数据库](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-kairos)
- [新遗物](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-newrelic)
- [普罗米修斯](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-prometheus)
- [信号交换](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-signalfx)
- [简单（内存中）](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-simple)
- [统计数据](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-statsd)
- [波前](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-export-wavefront)

|      | 要了解有关Micrometer功能的更多信息，请参阅其[参考文档](https://micrometer.io/docs)，特别是[概念部分](https://micrometer.io/docs/concepts)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 6.1。入门

Spring Boot自动配置组合，`MeterRegistry`并为其在类路径上找到的每个受支持的实现向组合添加注册表。在`micrometer-registry-{system}`运行时类路径中具有依赖项足以让Spring Boot配置注册表。

大多数注册表具有共同的特征。例如，即使Micrometer注册表实现位于类路径中，您也可以禁用特定的注册表。例如，禁用Datadog：

```properties
management.metrics.export.datadog.enabled=false
```

Spring Boot还会将任何自动配置的注册表添加到`Metrics`该类的全局静态复合注册表中，除非您明确告知不要：

```properties
management.metrics.use-global-registry=false
```

您可以注册任意数量的`MeterRegistryCustomizer`bean来进一步配置注册表，例如在向注册表注册任何计量器之前应用通用标签：

```java
@Bean
MeterRegistryCustomizer<MeterRegistry> metricsCommonTags() {
    return registry -> registry.config().commonTags("region", "us-east-1");
}
```

您可以通过更详细地了解通用类型，将自定义项应用于特定的注册表实现：

```java
@Bean
MeterRegistryCustomizer<GraphiteMeterRegistry> graphiteMetricsNamingConvention() {
    return registry -> registry.config().namingConvention(MY_CUSTOM_CONVENTION);
}
```

完成该设置后，您可以注入`MeterRegistry`组件并注册指标：

```java
@Component
public class SampleBean {

    private final Counter counter;

    public SampleBean(MeterRegistry registry) {
        this.counter = registry.counter("received.messages");
    }

    public void handleMessage(String message) {
        this.counter.increment();
        // handle message implementation
    }

}
```

Spring Boot还[配置了内置工具](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-meter)（即`MeterBinder`实现），您可以通过配置或专用注释标记来控制它们。

### 6.2。支持的监控系统

#### 6.2.1。AppOptics

默认情况下，AppOptics注册表会`api.appoptics.com/v1/measurements`定期将指标推送到。要将指标导出到SaaS [AppOptics](https://micrometer.io/docs/registry/appoptics)，必须提供您的API令牌：

```properties
management.metrics.export.appoptics.api-token=YOUR_TOKEN
```

#### 6.2.2。阿特拉斯

默认情况下，指标会导出到在本地计算机上运行的[Atlas](https://micrometer.io/docs/registry/atlas)。可以使用以下命令提供要使用的[Atlas服务器](https://github.com/Netflix/atlas)的位置：

```properties
management.metrics.export.atlas.uri=https://atlas.example.com:7101/api/v1/publish
```

#### 6.2.3。数据狗

Datadog注册表[会](https://www.datadoghq.com/)定期将指标推送到[datadoghq](https://www.datadoghq.com/)。要将指标导出到[Datadog](https://micrometer.io/docs/registry/datadog)，必须提供您的API密钥：

```properties
management.metrics.export.datadog.api-key=YOUR_KEY
```

您还可以更改将度量标准发送到Datadog的时间间隔：

```properties
management.metrics.export.datadog.step=30s
```

#### 6.2.4。动态痕迹

Dynatrace注册表会定期将指标推送到配置的URI。要将指标导出到[Dynatrace](https://micrometer.io/docs/registry/dynatrace)，必须提供您的API令牌，设备ID和URI：

```properties
management.metrics.export.dynatrace.api-token=YOUR_TOKEN
management.metrics.export.dynatrace.device-id=YOUR_DEVICE_ID
management.metrics.export.dynatrace.uri=YOUR_URI
```

您还可以更改将度量标准发送到Dynatrace的时间间隔：

```properties
management.metrics.export.dynatrace.step=30s
```

#### 6.2.5。有弹性

默认情况下，指标会导出到在本地计算机上运行的[Elastic](https://micrometer.io/docs/registry/elastic)。可以使用以下属性提供要使用的Elastic服务器的位置：

```properties
management.metrics.export.elastic.host=https://elastic.example.com:8086
```

#### 6.2.6。神经节

默认情况下，指标会导出到在本地计算机上运行的[Ganglia](https://micrometer.io/docs/registry/ganglia)。可以使用以下命令提供要使用的[Ganglia服务器](http://ganglia.sourceforge.net/)主机和端口：

```properties
management.metrics.export.ganglia.host=ganglia.example.com
management.metrics.export.ganglia.port=9649
```

#### 6.2.7。石墨

默认情况下，指标会导出到本地计算机上运行的[Graphite](https://micrometer.io/docs/registry/graphite)。可以使用以下方式提供要使用的[Graphite服务器](https://graphiteapp.org/)主机和端口：

```properties
management.metrics.export.graphite.host=graphite.example.com
management.metrics.export.graphite.port=9004
```

千分尺提供了一个默认值`HierarchicalNameMapper`，用于控制如何将尺寸仪表ID [映射到平面层次名称](https://micrometer.io/docs/registry/graphite#_hierarchical_name_mapping)。

|      | 要控制这种行为，请定义您自己的`GraphiteMeterRegistry`并提供您自己的`HierarchicalNameMapper`。除非您定义自己的，否则将提供自动配置的bean `GraphiteConfig`和`Clock`bean： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```java
@Bean
public GraphiteMeterRegistry graphiteMeterRegistry(GraphiteConfig config, Clock clock) {
    return new GraphiteMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

#### 6.2.8。悍马

默认情况下，Humio注册表[会](https://cloud.humio.com/)定期将指标推送到[cloud.humio.com](https://cloud.humio.com/)。要将指标导出到SaaS [Humio](https://micrometer.io/docs/registry/humio)，必须提供您的API令牌：

```properties
management.metrics.export.humio.api-token=YOUR_TOKEN
```

您还应该配置一个或多个标记以标识将度量标准推送到的数据源：

```properties
management.metrics.export.humio.tags.alpha=a
management.metrics.export.humio.tags.bravo=b
```

#### 6.2.9。涌入

默认情况下，指标会导出到本地计算机上运行的[Influx](https://micrometer.io/docs/registry/influx)。可使用以下命令提供要使用的[Influx服务器](https://www.influxdata.com/)的位置：

```properties
management.metrics.export.influx.uri=https://influx.example.com:8086
```

#### 6.2.10。JMX

千分尺提供了到[JMX](https://micrometer.io/docs/registry/jmx)的层次结构映射，主要是作为一种便宜且可移植的方式在本地查看指标。默认情况下，指标将导出到`metrics`JMX域。可以使用以下方式提供要使用的域：

```properties
management.metrics.export.jmx.domain=com.example.app.metrics
```

千分尺提供了一个默认值`HierarchicalNameMapper`，用于控制如何将尺寸仪表ID [映射到平面层次名称](https://micrometer.io/docs/registry/jmx#_hierarchical_name_mapping)。

|      | 要控制这种行为，请定义您自己的`JmxMeterRegistry`并提供您自己的`HierarchicalNameMapper`。除非您定义自己的，否则将提供自动配置的bean `JmxConfig`和`Clock`bean： |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

```java
@Bean
public JmxMeterRegistry jmxMeterRegistry(JmxConfig config, Clock clock) {
    return new JmxMeterRegistry(config, clock, MY_HIERARCHICAL_MAPPER);
}
```

#### 6.2.11。Kairos数据库

默认情况下，度量标准将导出到在本地计算机上运行的[KairosDB](https://micrometer.io/docs/registry/kairos)。可以使用以下方式提供要使用的[KairosDB服务器](https://kairosdb.github.io/)的位置：

```properties
management.metrics.export.kairos.uri=https://kairosdb.example.com:8080/api/v1/datapoints
```

#### 6.2.12。新遗物

New Relic注册表会定期将指标推送到[New Relic](https://micrometer.io/docs/registry/new-relic)。要将指标导出到[New Relic](https://newrelic.com/)，必须提供您的API密钥和帐户ID：

```properties
management.metrics.export.newrelic.api-key=YOUR_KEY
management.metrics.export.newrelic.account-id=YOUR_ACCOUNT_ID
```

您还可以更改将度量标准发送到新遗物的时间间隔：

```properties
management.metrics.export.newrelic.step=30s
```

#### 6.2.13。普罗米修斯

[Prometheus](https://micrometer.io/docs/registry/prometheus)希望抓取或轮询单个应用程序实例以获取指标。Spring Boot提供了一个执行器端点，可用于`/actuator/prometheus`以适当的格式显示[Prometheus刮擦](https://prometheus.io/)。

|      | 端点默认情况下不可用，必须公开，有关更多详细信息，请参见[暴露端点](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

这是`scrape_config`要添加到的示例`prometheus.yml`：

```yaml
scrape_configs:
  - job_name: 'spring'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['HOST:PORT']
```

对于短暂的或批处理的作业，其时间可能不够长，无法被[废弃，](https://github.com/prometheus/pushgateway)可以使用[Prometheus Pushgateway](https://github.com/prometheus/pushgateway)支持将其指标暴露给Prometheus。要启用Prometheus Pushgateway支持，请将以下依赖项添加到您的项目中：

```xml
<dependency>
    <groupId>io.prometheus</groupId>
    <artifactId>simpleclient_pushgateway</artifactId>
</dependency>
```

当类路径中存在Prometheus Pushgateway依赖项时，Spring Boot会自动配置`PrometheusPushGatewayManager`Bean。这可以管理将指标推送到Prometheus Pushgateway。在`PrometheusPushGatewayManager`可以在使用属性被调谐`management.metrics.export.prometheus.pushgateway`。对于高级配置，您还可以提供自己的`PrometheusPushGatewayManager`bean。

#### 6.2.14。信号交换

SignalFx注册表[会](https://micrometer.io/docs/registry/signalfx)定期将指标推送到[SignalFx](https://micrometer.io/docs/registry/signalfx)。要将指标导出到[SignalFx](https://www.signalfx.com/)，必须提供您的访问令牌：

```properties
management.metrics.export.signalfx.access-token=YOUR_ACCESS_TOKEN
```

您还可以更改将指标发送到SignalFx的时间间隔：

```properties
management.metrics.export.signalfx.step=30s
```

#### 6.2.15。简单

千分尺附带一个简单的内存后端，如果未配置其他注册表，该后端将自动用作后备。这使您可以查看在[度量标准端点](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-endpoint)中收集了哪些度量[标准](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-metrics-endpoint)。

使用任何其他可用后端时，内存后端都会自行禁用。您也可以显式禁用它：

```properties
management.metrics.export.simple.enabled=false
```

#### 6.2.16。统计数据

StatsD注册表急切地通过UDP将度量标准推送到StatsD代理。默认情况下，指标将导出到在本地计算机上运行的[StatsD](https://micrometer.io/docs/registry/statsd)代理。可以使用以下方式提供要使用的StatsD代理主机和端口：

```properties
management.metrics.export.statsd.host=statsd.example.com
management.metrics.export.statsd.port=9125
```

您还可以更改要使用的StatsD线路协议（默认为Datadog）：

```properties
management.metrics.export.statsd.flavor=etsy
```

#### 6.2.17。波前

Wavefront注册表会定期将指标推送到[Wavefront](https://micrometer.io/docs/registry/wavefront)。如果直接将指标导出到[Wavefront](https://www.wavefront.com/)，则必须提供API令牌：

```properties
management.metrics.export.wavefront.api-token=YOUR_API_TOKEN
```

或者，您可以使用在您的环境中设置的Wavefront辅助工具或内部代理，将指标数据转发到Wavefront API主机：

```properties
management.metrics.export.wavefront.uri=proxy://localhost:2878
```

|      | 如果将指标发布到Wavefront代理（如[文档](https://docs.wavefront.com/proxies_installing.html)中[所述](https://docs.wavefront.com/proxies_installing.html)），则主机必须采用以下`proxy://HOST:PORT`格式。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以更改将度量标准发送到Wavefront的时间间隔：

```properties
management.metrics.export.wavefront.step=30s
```

### 6.3。支持的指标

如果适用，Spring Boot将注册以下核心指标：

- JVM指标，报告以下各项的利用率：
  - 各种内存和缓冲池
  - 与垃圾收集有关的统计数据
  - 线程利用率
  - 加载/卸载的类数
- CPU指标
- 文件描述符指标
- Kafka使用者指标（应启用[JMX支持](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-jmx)）
- Log4j2指标：记录每个级别记录到Log4j2的事件数
- Logback指标：记录每个级别记录到Logback的事件数
- 正常运行时间指标：报告正常运行时间的指标和代表应用程序绝对启动时间的固定指标
- Tomcat指标（`server.tomcat.mbeanregistry.enabled`必须设置`true`为才能注册所有Tomcat指标）
- [Spring集成](https://docs.spring.io/spring-integration/docs/5.2.5.RELEASE/reference/html/system-management.html#micrometer-integration)指标

#### 6.3.1。Spring MVC指标

通过自动配置，可以检测由Spring MVC处理的请求。当`management.metrics.web.server.request.autotime.enabled`为时`true`，将对所有请求进行这种检测。或者，当设置为时，`false`可以通过添加`@Timed`到请求处理方法来启用检测：

```java
@RestController
@Timed 
public class MyController {

    @GetMapping("/api/people")
    @Timed(extraTags = { "region", "us-east-1" }) 
    @Timed(value = "all.people", longTask = true) 
    public List<Person> listPeople() { ... }

}
```

|      | 控制器类，用于对控制器中的每个请求处理程序启用计时。         |
| ---- | ------------------------------------------------------------ |
|      | 一种启用单个端点的方法。如果您将它放在类中，则不必这样做，但是可以用于进一步为此特定端点自定义计时器。 |
|      | 一种方法，`longTask = true`用于为该方法启用长任务计时器。长任务计时器需要单独的度量标准名称，并且可以与短任务计时器堆叠在一起。 |

默认情况下，度量标准的名称为`http.server.requests`。可以通过设置`management.metrics.web.server.request.metric-name`属性来自定义名称。

默认情况下，与Spring MVC相关的指标带有以下信息标记：

| 标签        | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| `exception` | 处理请求时引发的任何异常的简单类名。                         |
| `method`    | 请求的方法（例如`GET`或`POST`）                              |
| `outcome`   | 基于响应状态码的请求结果。1xx是`INFORMATIONAL`，2xx是`SUCCESS`，3xx是`REDIRECTION`，4xx `CLIENT_ERROR`和5xx是`SERVER_ERROR` |
| `status`    | 响应的HTTP状态代码（例如`200`或`500`）                       |
| `uri`       | 变量替换之前的请求URI模板（如果可能）（例如，`/api/person/{id}`） |

要自定义标签，请提供`@Bean`实现的`WebMvcTagsProvider`。

#### 6.3.2。Spring WebFlux指标

通过自动配置，可以检测由WebFlux控制器和功能处理程序处理的所有请求。

默认情况下，使用名称生成指标`http.server.requests`。您可以通过设置`management.metrics.web.server.request.metric-name`属性来自定义名称。

默认情况下，与WebFlux相关的度量标准标记有以下信息：

| 标签        | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| `exception` | 处理请求时引发的任何异常的简单类名。                         |
| `method`    | 请求的方法（例如`GET`或`POST`）                              |
| `outcome`   | 基于响应状态码的请求结果。1xx是`INFORMATIONAL`，2xx是`SUCCESS`，3xx是`REDIRECTION`，4xx `CLIENT_ERROR`和5xx是`SERVER_ERROR` |
| `status`    | 响应的HTTP状态代码（例如`200`或`500`）                       |
| `uri`       | 变量替换之前的请求URI模板（如果可能）（例如，`/api/person/{id}`） |

要自定义标签，请提供`@Bean`实现的`WebFluxTagsProvider`。

#### 6.3.3。泽西岛服务器指标

当Micrometer的`micrometer-jersey2`模块位于类路径上时，自动配置将启用对Jersey JAX-RS实现所处理的请求的检测。当`management.metrics.web.server.request.autotime.enabled`为时`true`，将对所有请求进行这种检测。或者，当设置为时，`false`可以通过添加`@Timed`到请求处理方法来启用检测：

```java
@Component
@Path("/api/people")
@Timed 
public class Endpoint {
    @GET
    @Timed(extraTags = { "region", "us-east-1" }) 
    @Timed(value = "all.people", longTask = true) 
    public List<Person> listPeople() { ... }
}
```

|      | 在资源类上，以对资源中的每个请求处理程序启用计时。           |
| ---- | ------------------------------------------------------------ |
|      | 关于启用单个端点的方法。如果您将它放在类中，则不必这样做，但是可以用于进一步为此特定端点自定义计时器。 |
|      | 具有`longTask = true`启用该方法的长任务计时器的方法。长任务计时器需要单独的度量标准名称，并且可以与短任务计时器堆叠在一起。 |

默认情况下，度量标准的名称为`http.server.requests`。可以通过设置`management.metrics.web.server.request.metric-name`属性来自定义名称。

默认情况下，Jersey服务器指标带有以下信息：

| 标签        | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| `exception` | 处理请求时引发的任何异常的简单类名。                         |
| `method`    | 请求的方法（例如`GET`或`POST`）                              |
| `outcome`   | 基于响应状态码的请求结果。1xx是`INFORMATIONAL`，2xx是`SUCCESS`，3xx是`REDIRECTION`，4xx `CLIENT_ERROR`和5xx是`SERVER_ERROR` |
| `status`    | 响应的HTTP状态代码（例如`200`或`500`）                       |
| `uri`       | 变量替换之前的请求URI模板（如果可能）（例如，`/api/person/{id}`） |

要自定义标签，请提供`@Bean`实现的`JerseyTagsProvider`。

#### 6.3.4。HTTP客户端指标

春季启动器同时管理的仪表`RestTemplate`和`WebClient`。为此，您必须注入自动配置的构建器并使用它来创建实例：

- `RestTemplateBuilder` 对于 `RestTemplate`
- `WebClient.Builder` 对于 `WebClient`

也可以手动应用负责此工具的定制程序，即`MetricsRestTemplateCustomizer`和`MetricsWebClientCustomizer`。

默认情况下，度量标准的名称为`http.client.requests`。可以通过设置`management.metrics.web.client.request.metric-name`属性来自定义名称。

默认情况下，通过检测的客户端生成的度量标准标记有以下信息：

| 标签         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| `clientName` | URI的主机部分                                                |
| `method`     | 请求的方法（例如`GET`或`POST`）                              |
| `outcome`    | 基于响应状态码的请求结果。1xx是`INFORMATIONAL`，2xx是`SUCCESS`，3xx是`REDIRECTION`，4xx `CLIENT_ERROR`和5xx是`SERVER_ERROR`，`UNKNOWN`否则 |
| `status`     | 响应的HTTP状态代码（例如`200`或`500`）（`IO_ERROR`如果存在I / O问题），`CLIENT_ERROR`否则 |
| `uri`        | 变量替换之前的请求URI模板（如果可能）（例如，`/api/person/{id}`） |

要自定义标签，并根据您选择的客户端，可以提供`@Bean`实现`RestTemplateExchangeTagsProvider`或的`WebClientExchangeTagsProvider`。有方便的静态函数中`RestTemplateExchangeTags`和`WebClientExchangeTags`。

#### 6.3.5。缓存指标

通过自动配置，可以`Cache`在启动时使用前缀为的指标来检测所有可用的`cache`。高速缓存检测针对一组基本指标进行了标准化。还提供其他特定于缓存的指标。

支持以下缓存库：

- 咖啡因
- 高速缓存2
- 淡褐色
- 任何兼容的JCache（JSR-107）实现

度量标准由高速缓存的名称以及`CacheManager`从Bean名称派生的的名称标记。

|      | 仅在启动时配置的缓存绑定到注册表。对于未在缓存配置中定义的缓存，例如在启动阶段后即时或以编程方式创建的缓存，需要显式注册。可以使用`CacheMetricsRegistrar`Bean来简化该过程。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 6.3.6。数据源指标

自动配置可`DataSource`使用前缀为的度量来检测所有可用对象`jdbc.connections`。数据源检测产生的量规表示池中当前活动，空闲，最大允许和最小允许的连接。

度量标准也由`DataSource`基于bean名称的计算名称标记。

|      | 默认情况下，Spring Boot为所有支持的数据源提供元数据。`DataSourcePoolMetadataProvider`如果不支持立即使用您喜欢的数据源，则可以添加其他bean。请参阅`DataSourcePoolMetadataProvidersConfiguration`示例。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

此外，特定于Hikari的指标带有`hikaricp`前缀。每个度量标准均以“池”的名称标记（可以通过来控制`spring.datasource.name`）。

#### 6.3.7。休眠指标

通过自动配置，可以检测所有可用`EntityManagerFactory`统计指标（名为）启用的可用Hibernate 实例`hibernate`。

指标也由`EntityManagerFactory`从Bean名称派生的的名称标记。

要启用统计信息，`hibernate.generate_statistics`必须将标准JPA属性设置为`true`。您可以在自动配置中启用该功能，`EntityManagerFactory`如以下示例所示：

```properties
spring.jpa.properties.hibernate.generate_statistics=true
```

#### 6.3.8。RabbitMQ指标

自动配置将启用名为的度量标准来检测所有可用的RabbitMQ连接工厂`rabbitmq`。

### 6.4。注册自定义指标

要注册自定义指标，请插入`MeterRegistry`到您的组件中，如以下示例所示：

```java
class Dictionary {

    private final List<String> words = new CopyOnWriteArrayList<>();

    Dictionary(MeterRegistry registry) {
        registry.gaugeCollectionSize("dictionary.size", Tags.empty(), this.words);
    }

    // …

}
```

如果发现您反复在组件或应用程序中检测一组度量标准，则可以将此套件封装在`MeterBinder`实现中。默认情况下，所有`MeterBinder`bean的度量标准都将自动绑定到Spring-managed `MeterRegistry`。

### 6.5。自定义单个指标

如果需要将自定义项应用于特定`Meter`实例，则可以使用该`io.micrometer.core.instrument.config.MeterFilter`界面。默认情况下，所有`MeterFilter`bean都会自动应用于千分尺`MeterRegistry.Config`。

例如，如果要将所有仪表ID 的`mytag.region`标签重命名`mytag.area`为`com.example`，则可以执行以下操作：

```java
@Bean
public MeterFilter renameRegionTagMeterFilter() {
    return MeterFilter.renameTag("com.example", "mytag.region", "mytag.area");
}
```

#### 6.5.1。常用标签

通用标签通常用于在操作环境（如主机，实例，区域，堆栈等）上进行维度深入分析。通用标签适用于所有仪表，可以按以下示例所示进行配置：

```properties
management.metrics.tags.region=us-east-1
management.metrics.tags.stack=prod
```

上面的示例将`region`和`stack`标记分别添加到`us-east-1`和`prod`分别具有和的值的所有仪表。

|      | 如果使用Graphite，则常用标签的顺序很重要。由于使用这种方法不能保证通用标签的顺序，因此建议石墨用户定义一个自定义`MeterFilter`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 6.5.2。每米属性

除了`MeterFilter`bean，还可以使用属性在每米的基础上应用一组有限的自定义设置。每表定制适用于以给定名称开头的所有所有表ID。例如，以下将禁用所有ID以以下开头的仪表`example.remote`

```properties
management.metrics.enable.example.remote=false
```

以下属性允许按米自定义：

| 属性                                                         | 描述                                                     |
| :----------------------------------------------------------- | :------------------------------------------------------- |
| `management.metrics.enable`                                  | 是否拒绝仪表发出任何指标。                               |
| `management.metrics.distribution.percentiles-histogram`      | 是否发布适用于计算可聚集（跨维度）百分位数逼近的直方图。 |
| `management.metrics.distribution.minimum-expected-value`， `management.metrics.distribution.maximum-expected-value` | 通过限制期望值的范围，发布较少的直方图桶。               |
| `management.metrics.distribution.percentiles`                | 发布在应用程序中计算的百分位值                           |
| `management.metrics.distribution.sla`                        | 发布包含您的SLA定义的存储桶的累积直方图。                |

有关概念的更多细节的背后`percentiles-histogram`，`percentiles`并`sla`请参阅[“直方图和百分”部分](https://micrometer.io/docs/concepts#_histograms_and_percentiles)微米文档。

### 6.6。指标终点

Spring Boot提供了一个`metrics`端点，可用于诊断检查应用程序收集的指标。端点默认情况下不可用，必须公开，有关更多详细信息，请参见[暴露端点](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints)。

导航以`/actuator/metrics`显示可用仪表名称的列表。您可以通过提供特定仪表的名称作为选择器来深入查看其信息，例如`/actuator/metrics/jvm.memory.max`。

|      | 您在此处使用的名称应与代码中使用的名称相匹配，而不是已针对其出厂的监视系统将其命名惯例标准化后的名称。换句话说，如果由于其蛇形命名约定`jvm.memory.max`而`jvm_memory_max`在Prometheus中出现，则`jvm.memory.max`在检查`metrics`端点的电表时仍应将其用作选择器。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以`tag=KEY:VALUE`在URL的末尾添加任意数量的查询参数，以在维度上进一步细分仪表，例如`/actuator/metrics/jvm.memory.max?tag=area:nonheap`。

|      | 报告的测量值是与仪表名称和已应用的所有标签相匹配的所有仪表的统计*总和*。因此，在上面的示例中，返回的“值”统计量是堆的“代码缓存”，“压缩类空间”和“元空间”区域的最大内存占用量的总和。如果您只想查看“ Metaspace”的最大大小，则可以添加一个额外的`tag=id:Metaspace`，即`/actuator/metrics/jvm.memory.max?tag=area:nonheap&tag=id:Metaspace`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 7.审核

一旦启动了Spring Security，Spring Boot Actuator将具有一个灵活的审核框架，该框架可以发布事件（默认情况下，“身份验证成功”，“失败”和“拒绝访问”异常）。此功能对于报告和基于身份验证失败实施锁定策略非常有用。

可以通过`AuditEventRepository`在应用程序的配置中提供类型的bean来启用审核。为了方便起见，Spring Boot提供了一个`InMemoryAuditEventRepository`。 `InMemoryAuditEventRepository`功能有限，我们建议仅将其用于开发环境。对于生产环境，请考虑创建自己的替代`AuditEventRepository`实现。

### 7.1。定制审核

要自定义发布的安全事件，可以提供自己的实现`AbstractAuthenticationAuditListener`和`AbstractAuthorizationAuditListener`。

您也可以将审核服务用于自己的业务事件。为此，您可以将`AuditEventRepository`Bean注入您自己的组件中，然后直接使用它，也可以将其发布`AuditApplicationEvent`到Spring中`ApplicationEventPublisher`（通过实现`ApplicationEventPublisherAware`）。

## 8. HTTP跟踪

可以通过`HttpTraceRepository`在应用程序的配置中提供类型的bean来启用HTTP跟踪。为了方便起见，Spring Boot提供了一个`InMemoryHttpTraceRepository`默认情况下存储最后100个请求-响应交换的跟踪的。 `InMemoryHttpTraceRepository`与其他跟踪解决方案相比，它受到限制，我们建议仅将其用于开发环境。对于生产环境，建议使用可用于生产的跟踪或可观察性解决方案，例如Zipkin或Spring Cloud Sleuth。或者，创建自己的`HttpTraceRepository`满足您需求的产品。

该`httptrace`终端可以被用来获得关于被存储在请求-响应交换信息`HttpTraceRepository`。

### 8.1。自定义HTTP跟踪

要自定义每个跟踪中包含的项目，请使用`management.trace.http.include`配置属性。对于高级定制，请考虑注册您自己的`HttpExchangeTracer`实现。

## 9.过程监控

在该`spring-boot`模块中，您可以找到两个类来创建文件，这些文件通常对于过程监视很有用：

- `ApplicationPidFileWriter`创建一个包含应用程序PID的文件（默认情况下，在应用程序目录中的文件名为`application.pid`）。
- `WebServerPortFileWriter`创建一个包含运行中的Web服务器端口的文件（默认情况下，在应用程序目录中，文件名为`application.port`）。

默认情况下，不会激活这些编写器，但是您可以启用：

- [通过扩展配置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-process-monitoring-configuration)
- [以编程方式](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-process-monitoring-programmatically)

### 9.1。扩展配置

在该`META-INF/spring.factories`文件中，您可以激活写入PID文件的侦听器，如以下示例所示：

```
org.springframework.context.ApplicationListener = \
org.springframework.boot.context.ApplicationPidFileWriter，\
org.springframework.boot.web.context.WebServerPortFileWriter
```

### 9.2。以编程方式

您也可以通过调用`SpringApplication.addListeners(…)`方法并传递适当的`Writer`对象来激活侦听器。此方法还允许您自定义`Writer`构造函数中的文件名和路径。

## 10. Cloud Foundry支持

Spring Boot的执行器模块包括额外的支持，当您将其部署到兼容的Cloud Foundry实例时就会激活。该`/cloudfoundryapplication`路径提供了通往所有`@Endpoint`bean 的替代安全路径。

扩展支持使Cloud Foundry管理UI（例如可用于查看已部署的应用程序的Web应用程序）增加了Spring Boot执行器信息。例如，应用程序状态页面可能包含完整的运行状况信息，而不是典型的“正在运行”或“已停止”状态。

|      | `/cloudfoundryapplication`普通用户不能直接访问 该路径。为了使用端点，必须将有效的UAA令牌与请求一起传递。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 10.1。禁用扩展的Cloud Foundry执行器支持

如果要完全禁用`/cloudfoundryapplication`端点，可以将以下设置添加到`application.properties`文件中：

application.properties

```properties
management.cloudfoundry.enabled=false
```

### 10.2。Cloud Foundry自签名证书

默认情况下，`/cloudfoundryapplication`端点的安全性验证会对各种Cloud Foundry服务进行SSL调用。如果您的Cloud Foundry UAA或Cloud Controller服务使用自签名证书，则需要设置以下属性：

application.properties

```properties
management.cloudfoundry.skip-ssl-validation=true
```

### 10.3。自定义上下文路径

如果服务器的上下文路径已配置为以外的任何其他值`/`，则Cloud Foundry端点在应用程序的根目录将不可用。例如，如果`server.servlet.context-path=/app`可以使用，Cloud Foundry端点将可用`/app/cloudfoundryapplication/*`。

如果您希望Cloud Foundry端点始终在处可用`/cloudfoundryapplication/*`，而与服务器的上下文路径无关，则需要在应用程序中进行显式配置。配置将根据所使用的Web服务器而有所不同。对于Tomcat，可以添加以下配置：

```java
@Bean
public TomcatServletWebServerFactory servletWebServerFactory() {
    return new TomcatServletWebServerFactory() {

        @Override
        protected void prepareContext(Host host, ServletContextInitializer[] initializers) {
            super.prepareContext(host, initializers);
            StandardContext child = new StandardContext();
            child.addLifecycleListener(new Tomcat.FixContextListener());
            child.setPath("/cloudfoundryapplication");
            ServletContainerInitializer initializer = getServletContextInitializer(getContextPath());
            child.addServletContainerInitializer(initializer, Collections.emptySet());
            child.setCrossContext(true);
            host.addChild(child);
        }

    };
}

private ServletContainerInitializer getServletContextInitializer(String contextPath) {
    return (c, context) -> {
        Servlet servlet = new GenericServlet() {

            @Override
            public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
                ServletContext context = req.getServletContext().getContext(contextPath);
                context.getRequestDispatcher("/cloudfoundryapplication").forward(req, res);
            }

        };
        context.addServlet("cloudfoundry", servlet).addMapping("/*");
    };
}
```

## 11.接下来要读什么

您可能需要阅读有关[Graphite](https://graphiteapp.org/)等绘图工具的信息。

否则，您可以继续阅读[“部署选项”，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment)或者继续阅读有关Spring Boot的*[构建工具插件的](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins)*一些深入信息。

最后更新时间2020-03-26 11:43:46 UTC

<iframe id="blockbyte-bs-sidebar" class="notranslate" data-pos="left" style="box-sizing: border-box; opacity: 0; pointer-events: none; position: fixed; top: 0px; left: 0px; width: 350px; max-width: none; height: 0px; z-index: 2147483646; border: none; transform: translate3d(-350px, 0px, 0px); transition: width 0s ease 0.3s, height 0s ease 0.3s, opacity 0.3s ease 0s, transform 0.3s ease 0s; background-color: rgba(255, 255, 255, 0.8) !important; display: block !important; color: rgb(0, 0, 0); font-family: Karla, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;"></iframe>

# 部署Spring Boot应用程序

[返回索引](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/index.html)

1. [1.部署到容器](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#containers-deployment)
2. 2.部署到云
   1. 
      1. 
   2. 
   3. 
   4. 
      1. 
         1. 
         2. 
      2. 
   5. 
   6. 
3. 3.安装Spring Boot应用程序
   1. 
   2. 
      1. 
         1. 
      2. 
      3. 
         1. 
         2. 
   3. 
4. [4.接下来阅读什么](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment-whats-next)

在部署应用程序时，Spring Boot的灵活打包选项提供了很多选择。您可以将Spring Boot应用程序部署到各种云平台，容器映像（例如Docker）或虚拟机/真实机上。

本节介绍一些更常见的部署方案。

## 1.部署到容器

如果从容器中运行应用程序，则可以使用可执行jar，但是将其爆炸并以其他方式运行通常也是一个优点。某些PaaS实施也可能选择在运行存档之前将其解压缩。例如，Cloud Foundry以这种方式运行。运行解压缩存档的最简单方法是启动相应的启动器，如下所示：

```
$ jar -xf myapp.jar
$ java org.springframework.boot.loader.JarLauncher
```

实际上，这在启动时（取决于jar的大小）比从未爆炸的存档中运行要快一些。在运行时，您不应期望有任何差异。

解压缩jar文件后，您还可以通过使用其“自然”主方法（而不是）运行应用程序，从而额外增加启动时间`JarLauncher`。例如：

```
$ jar -xf myapp.jar
$ java -cp BOOT-INF / classes：BOOT-INF / lib / * com.example.MyApplication
```

还可以通过将依赖项作为与应用程序类和资源（通常会更频繁地更改）分开的一层复制到映像中来创建更有效的容器映像。实现这一层分离的方法不止一种。例如，使用a `Dockerfile`可以将其表示为以下形式：

```
来自openjdk：8-jdk-alpine AS构建器
WORKDIR目标/依赖性
ARG APPJAR =目标/*.jar
COPY $ {APPJAR} app.jar
运行jar -xf ./app.jar

来自openjdk：8-jre-alpine
音量/ tmp
ARG DEPENDENCY =目标/依赖性
复制--from = builder $ {DEPENDENCY} / BOOT-INF / lib / app / lib
复制--from = builder $ {DEPENDENCY} / META-INF / app / META-INF
COPY --from = builder $ {DEPENDENCY} / BOOT-INF / classes / app
ENTRYPOINT [“ java”，“-cp”，“ app：app / lib / *”，“ com.example.MyApplication”]
```

假设以上`Dockerfile`内容位于当前目录中，则可以使用构建docker映像`docker build .`，或者可以选择指定应用程序jar的路径，如以下示例所示：

```
docker build --build-arg APPJAR =路径/到/myapp.jar。
```

## 2.部署到云

Spring Boot的可执行jar已为大多数流行的云PaaS（平台即服务）提供程序准备就绪。这些提供程序往往要求您“自带容器”。他们管理应用程序流程（不是专门用于Java应用程序），因此他们需要一个中间层，以使*您的*应用程序适应*云*中正在运行的流程*的*概念。

两家受欢迎的云提供商，Heroku和Cloud Foundry，采用了“ buildpack”方法。buildpack将部署的代码包装在*启动*应用程序所需的任何内容中。它可能是JDK，可能是对`java`，嵌入式Web服务器或成熟的应用程序服务器的调用。一个buildpack是可插入的，但是理想情况下，您应该能够通过尽可能少的自定义来获得它。这减少了您无法控制的功能的占用空间。它最大程度地减少了开发和生产环境之间的差异。

理想情况下，您的应用程序像Spring Boot可执行jar一样，具有打包运行所需的一切。

在本节中，我们研究如何启动在“入门”一节中[开发](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started-first-application)的[简单应用程序](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started-first-application)并在云中运行。

### 2.1。云铸造

如果未指定其他构建包，Cloud Foundry将提供默认的构建包。Cloud Foundry [Java buildpack](https://github.com/cloudfoundry/java-buildpack)对Spring应用程序（包括Spring Boot）提供了出色的支持。您可以部署独立的可执行jar应用程序以及传统的`.war`打包应用程序。

一旦您构建了应用程序（例如，使用`mvn clean package`）并[安装了`cf`命令行工具](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)，就可以通过使用`cf push`命令来部署您的应用程序，并替换已编译的路径`.jar`。推送应用程序之前，请确保已[使用`cf`命令行客户端登录](https://docs.cloudfoundry.org/cf-cli/getting-started.html#login)。以下行显示了使用`cf push`命令部署应用程序：

```
$ cf push acloudyspringtime -p target / demo-0.0.1-SNAPSHOT.jar
```

|      | 在前面的示例中，我们用`acloudyspringtime`您提供的任何值代替`cf`应用程序的名称。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

请参阅[`cf push`文档](https://docs.cloudfoundry.org/cf-cli/getting-started.html#push)以了解更多选项。如果[`manifest.yml`](https://docs.cloudfoundry.org/devguide/deploy-apps/manifest.html)同一目录中存在Cloud Foundry 文件，则将其考虑。

此时，`cf`开始上传您的应用程序，产生类似于以下示例的输出：

```
正在上传acloudyspringtime ... 确定 
正在准备启动acloudyspringtime ... 确定 
----->下载应用程序包（8.9M）
-----> Java Buildpack版本：v3.12（离线）| https://github.com/cloudfoundry/java-buildpack.git#6f25b7e
----->从https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_121.tar.gz下载Open Jdk JRE 1.8.0_121（在缓存中找到）
       将Open Jdk JRE扩展到.java-buildpack / open_jdk_jre（1.6s）
----->从https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz下载Open JDK，如Memory Calculator 2.0.2_RELEASE。缓存）
       内存设置：-Xss349K -Xmx681574K -XX：MaxMetaspaceSize = 104857K -Xms681574K -XX：MetaspaceSize = 104857K
----->从https://java-buildpack.cloudfoundry.org/container-certificate-trust-store/container-certificate-trust-store-1.0.0_RELEASE.jar下载容器证书信任库1.0.0_RELEASE在缓存中）
       将证书添加到.java-buildpack / container_certificate_trust_store / truststore.jks（0.6s）
----->从https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar下载Spring Auto Reconfiguration 1.10.0_RELEASE（在缓存中找到）
正在检查应用程序“ acloudyspringtime”的状态...
  1个实例中有0个正在运行（1个正在启动）
  ...
  1个实例中有0个正在运行（1个正在启动）
  ...
  1个实例中有0个正在运行（1个正在启动）
  ...
  1个实例中有1个正在运行（1个正在运行）

应用已启动
```

恭喜你！该应用程序现已上线！

应用程序上线后，可以使用`cf apps`命令验证已部署应用程序的状态，如以下示例所示：

```
$ cf个应用
在...中获取应用程序
好

名称请求的状态实例内存磁盘URL
...
acloudyspringtime已启动1/1 512M 1G acloudyspringtime.cfapps.io
...
```

一旦Cloud Foundry确认已经部署了您的应用程序，您就应该能够在给定的URI上找到该应用程序。在前面的示例中，您可以在找到它`https://acloudyspringtime.cfapps.io/`。

#### 2.1.1。绑定服务

默认情况下，有关正在运行的应用程序的元数据以及服务连接信息将作为环境变量（例如：）显示给应用程序`$VCAP_SERVICES`。此架构决定是由于Cloud Foundry的多语言（可以将任何语言和平台作为buildpack支持）性质所致。过程范围的环境变量与语言无关。

环境变量并非总是使用最简单的API，因此Spring Boot会自动提取它们并将数据展平为可以通过Spring的`Environment`抽象访问的属性，如以下示例所示：

```java
@Component
class MyBean implements EnvironmentAware {

    private String instanceId;

    @Override
    public void setEnvironment(Environment environment) {
        this.instanceId = environment.getProperty("vcap.application.instance_id");
    }

    // ...

}
```

所有Cloud Foundry属性均以开头`vcap`。您可以使用`vcap`属性来访问应用程序信息（例如，应用程序的公共URL）和服务信息（例如，数据库凭据）。有关完整的详细信息，请参见[“ CloudFoundryVcapEnvironmentPostProcessor”](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/cloud/CloudFoundryVcapEnvironmentPostProcessor.html) Javadoc。

|      | 在[Java的CFEnv](https://github.com/pivotal-cf/java-cfenv/)项目是一个更适合的任务，如配置数据源。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2。Heroku

Heroku是另一个流行的PaaS平台。要自定义Heroku构建，请提供`Procfile`，以提供部署应用程序所需的内容。Heroku `port`为Java应用程序分配了一个，然后确保可以路由到外部URI。

您必须配置您的应用程序以侦听正确的端口。以下示例显示了`Procfile`我们的入门REST应用程序的：

```
网址：java -Dserver.port = $ PORT -jar target / demo-0.0.1-SNAPSHOT.jar
```

Spring Boot使`-D`参数作为可从Spring `Environment`实例访问的属性而可用。所述`server.port`配置属性被馈送到嵌入的Tomcat，码头，或暗流实例，它然后使用端口在启动时。该`$PORT`环境变量由Heroku的PaaS的分配给我们。

这应该是您需要的一切。Heroku部署最常见的部署工作流程是`git push`将代码投入生产，如以下示例所示：

```
$ git push heroku master

初始化存储库，完成。
计数对象：95，完成。
增量压缩最多使用8个线程。
压缩对象：100％（78/78），完成。
书写对象：100％（95/95），8.66 MiB | 606.00 KiB / s，已完成。
总计95（增量31），已重用0（增量0）

----->检测到Java应用
----->安装OpenJDK 1.8 ...已完成 
----->安装Maven 3.3.1 ...已完成 
----->安装settings.xml ...已完成
----->执行中：mvn -B -DskipTests = true全新安装

       [INFO]正在扫描项目...
       下载：https：//repo.spring.io / ...
       下载地址：https：//repo.spring.io / ...（818 B，速度为1.8 KB /秒）
        ....
       下载地址：https：//s3pository.heroku.com/jvm / ...（152 KB，595.3 KB / sec）
       [INFO]正在安装/ tmp / build_0c35a5d2-a067-4abc-a232-14b1fb7a8229 / target / ...
       [INFO]正在安装/tmp/build_0c35a5d2-a067-4abc-a232-14b1fb7a8229/pom.xml ...
       [INFO] ----------------------------------------------- -------------------------
       [INFO]建立成功
       [INFO] ----------------------------------------------- -------------------------
       [INFO]总时间：59.358秒
       [INFO]完成于：UTC 2014年3月7日星期五7:28:25
       [INFO]最终内存：20M / 493M
       [INFO] ----------------------------------------------- -------------------------

----->发现过程类型
       Procfile声明类型-> Web

----->压缩... 完成，70.4MB
----->启动... 完成，第6版
       https://agile-sierra-1405.herokuapp.com/ 部署到Heroku

到git@heroku.com ：agile-sierra-1405.git
 * [新分支]主->主
```

您的应用程序现在应该已经在Heroku上启动并运行了。有关更多详细信息，请参阅将[Spring Boot应用程序部署到Heroku](https://devcenter.heroku.com/articles/deploying-spring-boot-apps-to-heroku)。

### 2.3。OpenShift

[OpenShift](https://www.openshift.com/)是Kubernetes容器编排平台的Red Hat公共（和企业）扩展。与Kubernetes相似，OpenShift具有许多用于安装基于Spring Boot的应用程序的选项。

OpenShift提供了许多资源来描述如何部署Spring Boot应用程序，包括：

- [使用S2I构建器](https://blog.openshift.com/using-openshift-enterprise-grade-spring-boot-deployments/)
- [建筑指南](https://access.redhat.com/documentation/en-us/reference_architectures/2017/html-single/spring_boot_microservices_on_red_hat_openshift_container_platform_3/)
- [在Wildfly上作为传统的Web应用程序运行](https://blog.openshift.com/using-spring-boot-on-openshift/)
- [OpenShift公共简报](https://blog.openshift.com/openshift-commons-briefing-96-cloud-native-applications-spring-rhoar/)

### 2.4。亚马逊网络服务（AWS）

Amazon Web Services提供了多种安装基于Spring Boot的应用程序的方法，这些方法既可以作为传统的Web应用程序（war），也可以作为具有嵌入式Web服务器的可执行jar文件安装。选项包括：

- AWS Elastic Beanstalk
- AWS Code Deploy
- AWS OPS作品
- AWS云形成
- AWS容器注册表

每个都有不同的功能和定价模型。在本文档中，我们仅描述最简单的选项：AWS Elastic Beanstalk。

#### 2.4.1。AWS Elastic Beanstalk

如官方的[Elastic Beanstalk Java指南中所述](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_Java.html)，部署Java应用程序有两个主要选项。您可以使用“ Tomcat平台”或“ Java SE平台”。

##### 使用Tomcat平台

该选项适用于产生war文件的Spring Boot项目。无需特殊配置。您只需要遵循官方指南即可。

##### 使用Java SE平台

该选项适用于产生jar文件并运行嵌入式Web容器的Spring Boot项目。Elastic Beanstalk环境在端口80上运行nginx实例来代理在端口5000上运行的实际应用程序。要对其进行配置，请在`application.properties`文件中添加以下行：

```
server.port = 5000
```

|      | 上传二进制文件而不是源文件默认情况下，Elastic Beanstalk上载源并将其编译到AWS中。但是，最好改为上传二进制文件。这样做，向您的`.elasticbeanstalk/config.yml`文件添加类似于以下内容的行：`deploy:    artifact: target/demo-0.0.1-SNAPSHOT.jar` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 通过设置环境类型来降低成本默认情况下，Elastic Beanstalk环境是负载平衡的。负载均衡器的成本很高。为避免该费用，请[按照Amazon文档](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-create-wizard.html#environments-create-wizard-capacity)中[的说明](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environments-create-wizard.html#environments-create-wizard-capacity)将环境类型设置为“单个实例” 。您还可以使用CLI和以下命令来创建单实例环境：`eb创建-s` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.4.2。摘要

这是使用AWS的最简单方法之一，但还有更多内容需要介绍，例如如何将Elastic Beanstalk集成到任何CI / CD工具中，如何使用Elastic Beanstalk Maven插件而不是CLI等等。有一篇[博客文章](https://exampledriven.wordpress.com/2017/01/09/spring-boot-aws-elastic-beanstalk-example/)详细介绍了这些主题。

### 2.5。Boxfuse和Amazon Web Services

[Boxfuse的](https://boxfuse.com/)工作原理是将您的Spring Boot可执行jar或war变成一个最小的VM映像，该映像可以在VirtualBox或AWS上不变地部署。Boxfuse与Spring Boot进行了深度集成，并使用Spring Boot配置文件中的信息自动配置端口和运行状况检查URL。Boxfuse在生成的图像以及它提供的所有资源（实例，安全组，弹性负载均衡器等）中均利用此信息。

创建[Boxfuse帐户](https://console.boxfuse.com/)，将其连接到您的AWS帐户，安装Boxfuse Client的最新版本并确保该应用程序已由Maven或Gradle构建（例如使用`mvn clean package`）后，就可以部署Spring使用与以下类似的命令将应用程序引导到AWS：

```
$ boxfuse运行myapp-1.0.jar -env = prod
```

请参阅[`boxfuse run`文档](https://boxfuse.com/docs/commandline/run.html)以了解更多选项。如果[`boxfuse.conf`](https://boxfuse.com/docs/commandline/#configuration)当前目录中存在文件，则将其考虑。

|      | 默认情况下，Boxfuse会`boxfuse`在启动时激活一个名为Spring的配置文件。如果您的可执行jar或war包含[`application-boxfuse.properties`](https://boxfuse.com/docs/payloads/springboot.html#configuration)文件，则Boxfuse的配置将基于其包含的属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

此时，`boxfuse`为您的应用程序创建一个映像，上传该映像，然后在AWS上配置并启动必要的资源，其输出类似于以下示例：

```
为myapp-1.0.jar融合图像...
图片融合于00：06.838s（53937 K）-> axelfontaine / myapp：1.0
创建axelfontaine / myapp ...
推送axelfontaine / myapp：1.0 ...
验证axelfontaine / myapp：1.0 ...
创建弹性IP ...
将myapp-axelfontaine.boxfuse.io映射到52.28.233.167 ...
等待AWS在eu-central-1中为axelfontaine / myapp：1.0创建AMI（这可能需要长达50秒的时间）...
在00：23.557s-> ami-d23f38cf中创建了AMI
创建安全组boxfuse-sg_axelfontaine / myapp：1.0 ...
在eu-central-1中启动axelfontaine / myapp：1.0（ami-d23f38cf）的t2.micro实例...
在00：30.306s-> i-92ef9f53启动实例
等待AWS引导实例i-92ef9f53和有效负载从https://52.28.235.61/开始...
有效负载始于00：29.266s-> https://52.28.235.61/
将Elastic IP 52.28.233.167重新映射到i-92ef9f53 ...
等待15秒以使AWS完成弹性IP零停机时间转换...
部署成功完成。axelfontaine / myapp：1.0已启动并在https://myapp-axelfontaine.boxfuse.io/运行
```

您的应用程序现在应该已启动并在AWS上运行。

请参阅有关[在EC2](https://boxfuse.com/blog/spring-boot-ec2.html)上[部署Spring Boot应用程序](https://boxfuse.com/blog/spring-boot-ec2.html)的博客文章以及[Boxfuse Spring Boot集成](https://boxfuse.com/docs/payloads/springboot.html)的[文档，](https://boxfuse.com/docs/payloads/springboot.html)以开始使用Maven构建来运行该应用程序。

### 2.6。谷歌云

Google云端有几种可用于启动Spring Boot应用程序的选项。最容易上手的可能是App Engine，但您也可以找到在Container Engine的容器中或Compute Engine的虚拟机上运行Spring Boot的方法。

要在App Engine中运行，您可以先在用户界面中创建一个项目，该项目将为您设置一个唯一的标识符，并还设置HTTP路由。将Java应用程序添加到项目中并将其保留为空，然后使用[Google Cloud SDK](https://cloud.google.com/sdk/install)从命令行或CI构建将Spring Boot应用程序推送到该插槽中。

App Engine Standard要求您使用WAR包装。请按照[以下步骤](https://github.com/GoogleCloudPlatform/getting-started-java/blob/master/appengine-standard-java8/springboot-appengine-standard/README.md)将App Engine标准应用程序部署到Google Cloud。

另外，App Engine Flex要求您创建一个`app.yaml`文件来描述您的应用程序所需的资源。通常，您将此文件放在中`src/main/appengine`，它应类似于以下文件：

```yaml
service: default

runtime: java
env: flex

runtime_config:
  jdk: openjdk8

handlers:
- url: /.*
  script: this field is required, but ignored

manual_scaling:
  instances: 1

health_check:
  enable_health_check: False

env_variables:
  ENCRYPT_KEY: your_encryption_key_here
```

您可以通过将项目ID添加到构建配置中来部署应用程序（例如，使用Maven插件），如以下示例所示：

```xml
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>appengine-maven-plugin</artifactId>
    <version>1.3.0</version>
    <configuration>
        <project>myproject</project>
    </configuration>
</plugin>
```

然后使用进行部署`mvn appengine:deploy`（如果您需要首先进行身份验证，则构建将失败）。

## 3.安装Spring Boot应用程序

除了使用来运行Spring Boot应用程序外`java -jar`，还可以为Unix系统制作完全可执行的应用程序。完全可执行的jar可以像其他任何可执行二进制文件一样执行，也可以[向`init.d`或注册`systemd`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment-service)。这使得在普通生产环境中安装和管理Spring Boot应用程序变得非常容易。

|      | 完全可执行的jar通过在文件的开头嵌入一个额外的脚本来工作。当前，某些工具不接受此格式，因此您可能无法始终使用此技术。例如，`jar -xf`可能无声地无法提取出已完全可执行的jar或war。建议仅当您打算直接执行jar或war时才使其完全可执行，而不是与jar `java -jar`容器一起运行或将其部署到servlet容器中。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 不能使zip64格式的jar文件完全可执行。尝试这样做将导致jar文件直接执行或使用时报告为损坏`java -jar`。包含一个或多个zip64格式嵌套jar的标准格式jar文件可以完全执行。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要使用Maven创建一个“完全可执行”的jar，请使用以下插件配置：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <executable>true</executable>
    </configuration>
</plugin>
```

以下示例显示了等效的Gradle配置：

```groovy
bootJar {
    launchScript()
}
```

然后，您可以通过键入`./my-application.jar`（`my-application`工件的名称在其中）来运行应用程序。包含jar的目录用作应用程序的工作目录。

### 3.1。支持的操作系统

默认脚本支持大多数Linux发行版，并已在CentOS和Ubuntu上进行了测试。其他平台，例如OS X和FreeBSD，则需要使用custom `embeddedLaunchScript`。

### 3.2。Unix / Linux服务

使用`init.d`或可轻松将Spring Boot应用程序作为Unix / Linux服务启动`systemd`。

#### 3.2.1。安装即`init.d`服务（系统V）

如果您将Spring Boot的Maven或Gradle插件配置为生成[完全可执行的jar](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment-install)，而不使用custom `embeddedLaunchScript`，则您的应用程序可以用作`init.d`服务。要做到这一点，符号链接罐子`init.d`，支持标准`start`，`stop`，`restart`，和`status`命令。

该脚本支持以下功能：

- 以拥有jar文件的用户身份启动服务
- 通过使用跟踪应用程序的PID `/var/run//.pid`
- 将控制台日志写入 `/var/log/.log`

假设您在`/var/myapp`中安装了Spring Boot应用程序，以将Spring Boot应用程序作为`init.d`服务安装，请创建一个符号链接，如下所示：

```
$ sudo ln -s /var/myapp/myapp.jar /etc/init.d/myapp
```

安装后，您可以按照通常的方式启动和停止服务。例如，在基于Debian的系统上，您可以使用以下命令启动它：

```
$服务myapp启动
```

|      | 如果您的应用程序无法启动，请检查写入的日志文件中`/var/log/.log`是否有错误。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还可以使用标准操作系统工具将应用程序标记为自动启动。例如，在Debian上，您可以使用以下命令：

```
$ update-rc.d myapp默认为<priority>
```

##### 保护`init.d`服务

|      | 以下是一组有关如何保护作为init.d服务运行的Spring Boot应用程序的准则。它并不旨在详尽列出增强应用程序及其运行环境所需的所有工作。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

当以root身份执行时（例如使用root来启动init.d服务时），默认的可执行脚本以`RUN_AS_USER`环境变量中指定的用户身份运行应用程序。如果未设置环境变量，则使用拥有jar文件的用户。您永远不要以身份运行Spring Boot应用程序`root`，因此`RUN_AS_USER`绝对不能以root 身份运行，并且应用程序的jar文件也绝对不能由root拥有。而是创建一个特定用户来运行您的应用程序并设置`RUN_AS_USER`环境变量，或使用`chown`它来使其成为jar文件的所有者，如以下示例所示：

```
$ chown bootapp：bootapp your-app.jar
```

在这种情况下，默认的可执行脚本以`bootapp`用户身份运行该应用程序。

|      | 为了减少应用程序的用户帐户遭到破坏的机会，您应该考虑阻止它使用登录外壳程序。例如，您可以将帐户的外壳设置为`/usr/sbin/nologin`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您还应该采取措施防止修改应用程序的jar文件。首先，配置其权限，使其不能被写入，只能由其所有者读取或执行，如以下示例所示：

```
$ chmod 500 your-app.jar
```

其次，如果您的应用程序或运行该应用程序的帐户受到威胁，您还应采取措施限制损害。如果攻击者确实获得了访问权限，则他们可以使jar文件可写并更改其内容。防止这种情况发生的一种方法是使用使其不可变`chattr`，如以下示例所示：

```
$ sudo chattr + i your-app.jar
```

这将阻止任何用户（包括root用户）修改jar。

如果使用root来控制应用程序的服务，并且您[使用`.conf`文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment-script-customization-conf-file)自定义其启动，`.conf`则root用户将读取并评估该文件。应该相应地对其进行保护。使用，`chmod`以便文件只能由所有者读取，并用于`chown`使root用户成为所有者，如以下示例所示：

```
$ chmod 400 your-app.conf
$ sudo chown root：root your-app.conf
```

#### 3.2.2。安装即`systemd`服务

`systemd`是System V init系统的后继产品，现在被许多现代Linux发行版使用。尽管您可以继续使用`init.d`带有脚本`systemd`，但也可以通过使用`systemd`“服务”脚本来启动Spring Boot应用程序。

假设您在`/var/myapp`中安装了Spring Boot应用程序，以将Spring Boot应用程序作为`systemd`服务安装，请创建一个名为的脚本`myapp.service`并将其放置在`/etc/systemd/system`目录中。以下脚本提供了一个示例：

```
[单元]
说明= myapp
之后= syslog.target

[服务]
用户= myapp
ExecStart = / var / myapp / myapp.jar
SuccessExitStatus = 143

[安装]
WantedBy =多用户目标
```

|      | 请记住，改变`Description`，`User`和`ExecStart`域为您的应用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 该`ExecStart`字段未声明脚本操作命令，这意味着`run`默认情况下使用该命令。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

请注意，与作为`init.d`服务运行时不同，运行应用程序的用户，PID文件和控制台日志文件由`systemd`自己管理，因此必须使用“服务”脚本中的适当字段进行配置。有关更多详细信息，请查阅[服务单元配置手册页](https://www.freedesktop.org/software/systemd/man/systemd.service.html)。

要将应用程序标记为在系统启动时自动启动，请使用以下命令：

```
$ systemctl启用myapp.service
```

`man systemctl`有关更多详细信息，请参阅。

#### 3.2.3。自定义启动脚本

由Maven或Gradle插件编写的默认嵌入式启动脚本可以通过多种方式进行自定义。对于大多数人来说，使用默认脚本以及一些自定义设置通常就足够了。如果发现无法自定义所需的内容，请使用该`embeddedLaunchScript`选项完全写入自己的文件。

##### 编写后自定义启动脚本

在将启动脚本写入jar文件时，自定义启动脚本的元素通常很有意义。例如，init.d脚本可以提供“描述”。由于您已经预先了解了描述（并且无需更改），因此在生成jar时也可以提供它。

要自定义书面元素，请使用`embeddedLaunchScriptProperties`Spring Boot Maven插件的选项或[`properties`Spring Boot Gradle插件的属性`launchScript`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html//#packaging-executable-configuring-launch-script)。

默认脚本支持以下属性替换：

| 名称                       | 描述                                                         | Gradle默认                                                   | Maven默认                                           |
| :------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------------------------------------------- |
| `mode`                     | 脚本模式。                                                   | `auto`                                                       | `auto`                                              |
| `initInfoProvides`         | 在`Provides`的“INIT INFO”节                                  | `${task.baseName}`                                           | `${project.artifactId}`                             |
| `initInfoRequiredStart`    | `Required-Start` “初始信息”部分。                            | `$remote_fs $syslog $network`                                | `$remote_fs $syslog $network`                       |
| `initInfoRequiredStop`     | `Required-Stop` “初始信息”部分。                             | `$remote_fs $syslog $network`                                | `$remote_fs $syslog $network`                       |
| `initInfoDefaultStart`     | `Default-Start` “初始信息”部分。                             | `2 3 4 5`                                                    | `2 3 4 5`                                           |
| `initInfoDefaultStop`      | `Default-Stop` “初始信息”部分。                              | `0 1 6`                                                      | `0 1 6`                                             |
| `initInfoShortDescription` | `Short-Description` “初始信息”部分。                         | 的单行版本`${project.description}`（回退到`${task.baseName}`） | `${project.name}`                                   |
| `initInfoDescription`      | `Description` “初始信息”部分。                               | `${project.description}`（回落到`${task.baseName}`）         | `${project.description}`（回落到`${project.name}`） |
| `initInfoChkconfig`        | `chkconfig` “ INIT INFO”部分                                 | `2345 99 01`                                                 | `2345 99 01`                                        |
| `confFolder`               | 的默认值 `CONF_FOLDER`                                       | 包含罐子的文件夹                                             | 包含罐子的文件夹                                    |
| `inlinedConfScript`        | 引用应在默认启动脚本中内联的文件脚本。这可用于设置环境变量，例如`JAVA_OPTS`在加载任何外部配置文件之前 |                                                              |                                                     |
| `logFolder`                | 的默认值`LOG_FOLDER`。仅对`init.d`服务有效                   |                                                              |                                                     |
| `logFilename`              | 的默认值`LOG_FILENAME`。仅对`init.d`服务有效                 |                                                              |                                                     |
| `pidFolder`                | 的默认值`PID_FOLDER`。仅对`init.d`服务有效                   |                                                              |                                                     |
| `pidFilename`              | PID文件名的默认值`PID_FOLDER`。仅对`init.d`服务有效          |                                                              |                                                     |
| `useStartStopDaemon`       | 该`start-stop-daemon`命令在可用时是否应用于控制过程          | `true`                                                       | `true`                                              |
| `stopWaitTime`             | 缺省值（`STOP_WAIT_TIME`以秒为单位）。仅对`init.d`服务有效   | 60                                                           | 60                                                  |

##### 运行时自定义脚本

对于在编写jar *之后*需要自定义脚本的项目，可以使用环境变量或[配置文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment-script-customization-conf-file)。

默认脚本支持以下环境属性：

| 变量                    | 描述                                                         |
| :---------------------- | :----------------------------------------------------------- |
| `MODE`                  | 操作的“模式”。默认值取决于jar的构建方式，但通常`auto`是这样（这意味着它会通过检查目录中是否为符号链接来尝试猜测它是否为初始化脚本`init.d`）。您可以将其显式设置为，`service`以便`stop|start|status|restart`命令`run`可以运行，也可以将其设置为在前台运行脚本。 |
| `RUN_AS_USER`           | 将用于运行应用程序的用户。未设置时，将使用拥有jar文件的用户。 |
| `USE_START_STOP_DAEMON` | 该`start-stop-daemon`命令在可用时是否应用于控制过程。默认为`true`。 |
| `PID_FOLDER`            | pid文件夹的根名称（`/var/run`默认情况下）。                  |
| `LOG_FOLDER`            | 放置日志文件的文件夹的名称（`/var/log`默认情况下）。         |
| `CONF_FOLDER`           | 从中读取.conf文件的文件夹的名称（默认情况下与jar文件相同的文件夹）。 |
| `LOG_FILENAME`          | 日志文件的名称`LOG_FOLDER`（`.log`默认情况下）。             |
| `APP_NAME`              | 应用程序的名称。如果jar是从符号链接运行的，则脚本会猜测应用程序名称。如果它不是符号链接，或者您要显式设置应用程序名称，则此功能很有用。 |
| `RUN_ARGS`              | 传递给程序的参数（Spring Boot应用程序）。                    |
| `JAVA_HOME`             | `java`可执行文件的位置`PATH`默认情况下是通过使用来发现的，但是如果在上有可执行文件，则可以显式设置它`$JAVA_HOME/bin/java`。 |
| `JAVA_OPTS`             | 启动JVM时传递给JVM的选项。                                   |
| `JARFILE`               | jar文件的显式位置，以防脚本被用于启动实际上未嵌入的jar。     |
| `DEBUG`                 | 如果不为空，则`-x`在shell进程上设置该标志，从而易于查看脚本中的逻辑。 |
| `STOP_WAIT_TIME`        | 停止应用程序之前强制关闭的等待时间（以秒为单位`60`）（默认情况下）。 |

|      | 的`PID_FOLDER`，`LOG_FOLDER`和`LOG_FILENAME`变量仅适用于一个`init.d`服务。对于`systemd`，等效的自定义是通过使用“服务”脚本进行的。有关更多详细信息，请参见[服务单元配置手册页](https://www.freedesktop.org/software/systemd/man/systemd.service.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

除`JARFILE`和之外`APP_NAME`，可以使用`.conf`文件配置上一节中列出的设置。该文件应位于jar文件的旁边，并且具有相同的名称，但后缀为`.conf`而不是`.jar`。例如，名为jar的jar `/var/myapp/myapp.jar`使用名为的配置文件`/var/myapp/myapp.conf`，如以下示例所示：

myapp.conf

```
JAVA_OPTS = -Xmx1024M
LOG_FOLDER = / custom / log / folder
```

|      | 如果您不喜欢将配置文件放在jar文件旁边，则可以设置`CONF_FOLDER`环境变量以自定义配置文件的位置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要了解有关适当保护此文件的信息，请参阅[保护init.d服务的准则](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment-initd-service-securing)。

### 3.3。Microsoft Windows服务

可以使用来将Spring Boot应用程序作为Windows服务启动[`winsw`](https://github.com/kohsuke/winsw)。

（一个[单独维护的示例](https://github.com/snicoll-scratches/spring-boot-daemon)）逐步说明了如何为Spring Boot应用程序创建Windows服务。

## 4.接下来阅读什么

请访问[Cloud Foundry](https://www.cloudfoundry.org/)，[Heroku](https://www.heroku.com/)，[OpenShift](https://www.openshift.com/)和[Boxfuse](https://boxfuse.com/)网站，以获取有关PaaS可以提供的各种功能的更多信息。这些只是最受欢迎的Java PaaS提供程序中的四个。由于Spring Boot非常适合基于云的部署，因此您也可以自由考虑其他提供商。

下一节将继续介绍*[Spring Boot CLI](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-cli.html#cli)*，或者您可以直接阅读有关*[构建工具插件的信息](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins)*。

最后更新时间2020-03-26 11:43:46 UTC

<iframe id="blockbyte-bs-sidebar" class="notranslate" data-pos="left" style="box-sizing: border-box; opacity: 0; pointer-events: none; position: fixed; top: 0px; left: 0px; width: 350px; max-width: none; height: 0px; z-index: 2147483646; border: none; transform: translate3d(-350px, 0px, 0px); transition: width 0s ease 0.3s, height 0s ease 0.3s, opacity 0.3s ease 0s, transform 0.3s ease 0s; background-color: rgba(255, 255, 255, 0.8) !important; display: block !important; color: rgb(0, 0, 0); font-family: Karla, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;"></iframe>

# 生成工具插件

[返回索引](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/index.html)

1. \1. Spring Boot Maven插件
   1. [1.1。包括插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-include-maven-plugin)
   2. [1.2。包装可执行的Jar和War文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-packaging)
2. [2. Spring Boot Gradle插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-gradle-plugin)
3. \3. Spring Boot AntLib模块
   1. 
      1. 
      2. 
   2. 
      1. 
4. 4.支持其他构建系统
   1. 
   2. 
   3. 
   4. 
5. [5.接下来阅读什么](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-whats-next)

Spring Boot为Maven和Gradle提供了构建工具插件。插件提供了多种功能，包括可执行jar的打包。本节提供有关这两个插件的更多详细信息，以及在扩展不受支持的构建系统时所需的一些帮助。如果您刚刚入门，则可能需要先阅读“ [using-spring-boot.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-build-systems) ”部分中的“ [using-spring-boot.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot) ”。

## 1. Spring Boot Maven插件

Spring Boot Maven插件在Maven中提供了Spring Boot支持，使您可以打包可执行jar或war归档文件并“就地”运行应用程序。要使用它，必须使用Maven 3.2（或更高版本）。

|      | 请参阅[Spring Boot Maven插件站点](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin/)以获取完整的插件文档。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.1。包括插件

要使用Spring Boot Maven插件，请在的`plugins`部分中包含适当的XML `pom.xml`，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!-- ... -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.2.6.RELEASE</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

前面的配置重新打包了`package`在Maven生命周期阶段构建的jar或war 。以下示例在`target`目录中显示了重新打包的jar和原始jar ：

```
$ mvn软件包
$ ls target / *。jar
target / myproject-1.0.0.jar target / myproject-1.0.0.jar.original
```

``如上例所示，如果不包括配置，则可以单独运行插件（但也必须同时使用软件包目标），如以下示例所示：

```
$ mvn软件包spring-boot：repackage
$ ls target / *。jar
target / myproject-1.0.0.jar target / myproject-1.0.0.jar.original
```

如果使用里程碑或快照版本，则还需要添加适当的`pluginRepository`元素，如以下清单所示：

```xml
<pluginRepositories>
    <pluginRepository>
        <id>spring-snapshots</id>
        <url>https://repo.spring.io/snapshot</url>
    </pluginRepository>
    <pluginRepository>
        <id>spring-milestones</id>
        <url>https://repo.spring.io/milestone</url>
    </pluginRepository>
</pluginRepositories>
```

### 1.2。包装可执行的Jar和War文件

一旦`spring-boot-maven-plugin`包含在您的中`pom.xml`，它就会自动尝试重写归档文件，以使归档文件可以通过使用`spring-boot:repackage`目标来执行。您应该使用通常的`packaging`元素将项目配置为构建jar或war（视情况而定），如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>jar</packaging>
    <!-- ... -->
</project>
```

在此`package`阶段中，Spring Boot会增强您现有的存档。可以通过使用配置选项（如下所示）或向`Main-Class`清单添加属性来指定要启动的主类。如果您未指定主类，则插件会使用`public static void main(String[] args)`方法搜索类。

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <mainClass>com.example.app.Main</mainClass>
    </configuration>
</plugin>
```

要构建和运行项目工件，可以输入以下内容：

```
$ mvn软件包
$ java -jar target / mymodule-0.0.1-SNAPSHOT.jar
```

要构建既可执行又可部署到外部容器的war文件，您需要将嵌入式容器的相关性标记为“已提供”，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- ... -->
    <packaging>war</packaging>
    <!-- ... -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <!-- ... -->
    </dependencies>
</project>
```

|      | 有关如何创建可部署的war文件的更多详细信息，请参见“ [howto.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-create-a-deployable-war-file) ”部分。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

[插件信息页面](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin/)中提供了高级配置选项和示例。

## 2. Spring Boot Gradle插件

Spring Boot Gradle插件在Gradle中提供了Spring Boot支持，可让您打包可执行jar或war归档文件，运行Spring Boot应用程序以及使用所提供的依赖项管理`spring-boot-dependencies`。它需要Gradle 5.x（也支持4.10，但不支持该支持，在将来的版本中将删除该支持）。请参考插件的文档以了解更多信息：

- 参考（[HTML](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html/)和[PDF](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf)）
- [API](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/api/)

## 3. Spring Boot AntLib模块

Spring Boot AntLib模块为Apache Ant提供了基本的Spring Boot支持。您可以使用该模块创建可执行jar。要使用该模块，您需要在中声明一个额外的`spring-boot`名称空间`build.xml`，如以下示例所示：

```xml
<project xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">
    ...
</project>
```

您需要记住使用该`-lib`选项启动Ant ，如以下示例所示：

```
$ ant -lib <包含spring-boot-antlib-2.2.6.RELEASE.jar的文件夹>
```

|      | “使用Spring Boot”部分包含[将Apache Ant与结合使用`spring-boot-antlib`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-ant)的更完整示例。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.1。Spring Boot Ant任务

一旦`spring-boot-antlib`命名空间已申报，以下附加任务：

- [`spring-boot:exejar`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#spring-boot-ant-exejar)
- [`spring-boot:findmainclass`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#spring-boot-ant-findmainclass)

#### 3.1.1。 `spring-boot:exejar`

您可以使用该`exejar`任务创建一个Spring Boot可执行jar。任务支持以下属性：

| 属性          | 描述                   | 需要                                         |
| :------------ | :--------------------- | :------------------------------------------- |
| `destfile`    | 要创建的目标jar文件    | 是                                           |
| `classes`     | Java类文件的根目录     | 是                                           |
| `start-class` | 要运行的主要应用程序类 | 否*（默认为找到的第一个声明`main`方法的类）* |

以下嵌套元素可用于任务：

| 元件        | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| `resources` | 一个或多个[资源集合，](https://ant.apache.org/manual/Types/resources.html#collection)描述应添加到创建的jar文件内容中的一组[资源](https://ant.apache.org/manual/Types/resources.html)。 |
| `lib`       | 应将一个或多个[资源集合](https://ant.apache.org/manual/Types/resources.html#collection)添加到组成应用程序运行时依赖项类路径的jar库集合中。 |

#### 3.1.2。例子

本节显示了两个Ant任务示例。

指定开始等级

```xml
<spring-boot:exejar destfile="target/my-application.jar"
        classes="target/classes" start-class="com.example.MyApplication">
    <resources>
        <fileset dir="src/main/resources" />
    </resources>
    <lib>
        <fileset dir="lib" />
    </lib>
</spring-boot:exejar>
```

检测入门班

```xml
<exejar destfile="target/my-application.jar" classes="target/classes">
    <lib>
        <fileset dir="lib" />
    </lib>
</exejar>
```

### 3.2。 `spring-boot:findmainclass`

该`findmainclass`任务在内部`exejar`用于查找声明的类`main`。如有必要，您也可以在构建中直接使用此任务。支持以下属性：

| 属性          | 描述                        | 需要                           |
| :------------ | :-------------------------- | :----------------------------- |
| `classesroot` | Java类文件的根目录          | 是*（除非`mainclass`指定）*    |
| `mainclass`   | 可用于短路`main`班级搜索    | 没有                           |
| `property`    | 应该与结果一起设置的Ant属性 | 否*（如果未指定，将记录结果）* |

#### 3.2.1。例子

本节包含使用的三个示例`findmainclass`。

查找并记录

```xml
<findmainclass classesroot="target/classes" />
```

查找并设置

```xml
<findmainclass classesroot="target/classes" property="main-class" />
```

覆盖并设置

```xml
<findmainclass mainclass="com.example.MainClass" property="main-class" />
```

## 4.支持其他构建系统

如果要使用Maven，Gradle或Ant以外的构建工具，则可能需要开发自己的插件。可执行jar需要遵循特定的格式，某些条目需要以未压缩的形式编写（有关详细信息，请参见附录中的“ [可执行jar格式](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-executable-jar-format.html#executable-jar) ”部分）。

Spring Boot Maven和Gradle插件都利用它们`spring-boot-loader-tools`来实际生成jar。如果需要，可以直接使用此库。

### 4.1。重新打包档案

要重新打包现有存档，使其成为独立的可执行存档，请使用`org.springframework.boot.loader.tools.Repackager`。该`Repackager`班采取的是指现有的罐子或战争归档单个构造函数的参数。使用两种可用`repackage()`方法之一替换原始文件或写入新目标。在重新打包程序运行之前，还可以在其上配置各种设置。

### 4.2。嵌套库

重新打包档案时，可以使用该`org.springframework.boot.loader.tools.Libraries`接口包含对依赖文件的引用。我们`Libraries`这里不提供任何具体的实现，因为它们通常是特定于构建系统的。

如果归档文件中已经包含库，则可以使用`Libraries.NONE`。

### 4.3。寻找主班

如果您不用于`Repackager.setMainClass()`指定主类，则重新包装器将使用[ASM](https://asm.ow2.io/)读取类文件，并尝试使用`public static void main(String[] args)`方法查找合适的类。如果找到多个候选者，则会引发异常。

### 4.4。重新打包实施示例

以下示例显示了典型的重新打包实现：

```java
Repackager repackager = new Repackager(sourceJarFile);
repackager.setBackupSource(false);
repackager.repackage(new Libraries() {
            @Override
            public void doWithLibraries(LibraryCallback callback) throws IOException {
                // Build system specific implementation, callback for each dependency
                // callback.library(new Library(nestedFile, LibraryScope.COMPILE));
            }
        });
```

## 5.接下来阅读什么

如果您对构建工具插件的工作方式感兴趣，可以查看[`spring-boot-tools`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-tools)GitHub 上的模块。可执行jar格式的更多技术细节在[附录中介绍](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-executable-jar-format.html#executable-jar)。

如果您有与构建相关的特定问题，可以查看“操作[方法](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto) ”指南。

最后更新时间2020-03-26 11:43:46 UTC

<iframe id="blockbyte-bs-sidebar" class="notranslate" data-pos="left" style="box-sizing: border-box; opacity: 0; pointer-events: none; position: fixed; top: 0px; left: 0px; width: 350px; max-width: none; height: 0px; z-index: 2147483646; border: none; transform: translate3d(-350px, 0px, 0px); transition: width 0s ease 0.3s, height 0s ease 0.3s, opacity 0.3s ease 0s, transform 0.3s ease 0s; background-color: rgba(255, 255, 255, 0.8) !important; display: block !important; color: rgb(0, 0, 0); font-family: Karla, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;"></iframe>

# “使用方法”指南

[返回索引](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/index.html)

1. \1. Spring Boot应用程序
   1. 
   2. 
   3. 
   4. 
   5. 
2. 2.属性和配置
   1. 
      1. 
      2. 
   2. 
   3. 
   4. 
   5. 
   6. 
   7. 
   8. 
3. 3.嵌入式Web服务器
   1. 
   2. 
   3. 
   4. 
   5. 
   6. 
   7. 
   8. 
      1. 
      2. 
      3. 
      4. 
   9. 
   10. 
       1. 
          1. 
       2. 
   11. 
   12. 
       1. 
   13. 
   14. 
   15. 
   16. 
   17. 
4. \4. Spring MVC
   1. 
   2. 
   3. 
   4. 
   5. 
   6. 
   7. 
   8. 
5. [5.使用Spring Security进行测试](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-use-test-with-spring-security)
6. 6.球衣
   1. 
   2. 
7. \7. HTTP客户端
   1. 
   2. 
8. 8.记录
   1. 
      1. 
   2. 
      1. 
9. 9.资料存取
   1. 
   2. 
   3. 
   4. 
   5. 
   6. 
   7. 
   8. 
   9. 
   10. 
   11. 
   12. 
   13. 
   14. 
   15. 
   16. 
10. 10.数据库初始化
    1. 
    2. 
    3. 
    4. 
    5. 
       1. 
       2. 
11. 11.消息传递
    1. 
12. 12.批处理应用
    1. 
    2. 
    3. 
       1. 
    4. 
13. 13.执行器
    1. 
    2. 
    3. 
14. 14.安全性
    1. 
    2. 
    3. 
15. 15.热插拔
    1. 
    2. 
       1. 
       2. 
       3. 
    3. 
    4. 
16. 16.建立
    1. 
    2. 
    3. 
    4. 
    5. 
    6. 
    7. 
    8. 
    9. 
17. 17.传统部署
    1. [17.1。创建可部署的战争文件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-create-a-deployable-war-file)
    2. [17.2。将现有应用程序转换为Spring Boot](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-convert-an-existing-application-to-spring-boot)
    3. [17.3。将WAR部署到WebLogic](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-weblogic)
    4. [17.4。使用Jedis代替生菜](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-use-jedis-instead-of-lettuce)

本节提供了一些在使用Spring Boot时经常出现的常见``我该怎么做...''问题的答案。它的覆盖范围不是很详尽，但是确实覆盖了很多。

如果您有一个我们不在此讨论的特定问题，则可能需要查看[stackoverflow.com](https://stackoverflow.com/tags/spring-boot)来查看是否有人已经提供了答案。这也是一个提出新问题的好地方（请使用`spring-boot`标签）。

我们也很乐意扩展此部分。如果您想添加“操作方法”，请向我们发送[请求请求](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE)。

## 1. Spring Boot应用程序

本部分包括与Spring Boot应用程序直接相关的主题。

### 1.1。创建自己的FailureAnalyzer

[`FailureAnalyzer`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/diagnostics/FailureAnalyzer.html)是在启动时拦截异常并将其转换为人类可读消息的好方法，并包装在内[`FailureAnalysis`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/diagnostics/FailureAnalysis.html)。Spring Boot为与应用程序上下文相关的异常，JSR-303验证等提供了此类分析器。您也可以创建自己的。

`AbstractFailureAnalyzer`是一个方便的扩展，`FailureAnalyzer`它检查要处理的异常中是否存在指定的异常类型。您可以对此进行扩展，以便您的实现只有在异常出现时才有机会处理该异常。如果由于某种原因无法处理该异常，请返回`null`以使另一个实现有机会处理该异常。

`FailureAnalyzer`实施必须在中注册`META-INF/spring.factories`。以下示例注册`ProjectConstraintViolationFailureAnalyzer`：

```properties
org.springframework.boot.diagnostics.FailureAnalyzer=\
com.example.ProjectConstraintViolationFailureAnalyzer
```

|      | 如果您需要访问`BeanFactory`或`Environment`，则`FailureAnalyzer`可以分别实现`BeanFactoryAware`或`EnvironmentAware`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.2。自动配置故障排除

Spring Boot自动配置会尽力“做正确的事”，但有时会失败，并且很难说出原因。

`ConditionEvaluationReport`任何Spring Boot中都有一个非常有用的功能`ApplicationContext`。如果启用`DEBUG`日志记录输出，则可以看到它。如果使用`spring-boot-actuator`（请参阅[“执行器”一章](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready)），则还有一个`conditions`终结点，该终结点以JSON形式呈现报告。使用该端点来调试应用程序，并在运行时查看Spring Boot添加了哪些功能（哪些尚未添加）。

通过查看源代码和Javadoc，可以回答更多问题。阅读代码时，请记住以下经验法则：

- 查找被调用的类`*AutoConfiguration`并阅读其源代码。特别注意`@Conditional*`注释，以了解它们启用了哪些功能以及何时启用。添加`--debug`到命令行或“系统”属性`-Ddebug`以在控制台上获取应用程序中做出的所有自动配置决策的日志。在启用了执行器的运行应用程序中，查看`conditions`端点（`/actuator/conditions`或等效的JMX）以获取相同信息。
- 查找类`@ConfigurationProperties`（例如[`ServerProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)），然后从中读取可用的外部配置选项。该`@ConfigurationProperties`注释具有一个`name`充当前缀外部性能属性。因此，`ServerProperties`拥有`prefix="server"`和它的配置性能`server.port`，`server.address`以及其他。在启用了执行器的运行应用程序中，查看`configprops`端点。
- 寻找对`bind`方法的使用，以一种轻松的方式`Binder`将配置值明确地拉出`Environment`。它通常与前缀一起使用。
- 查找`@Value`直接绑定到的注释`Environment`。
- 寻找可`@ConditionalOnExpression`根据SpEL表达式打开和关闭功能的注释，这些注释通常使用从中解析的占位符进行评估`Environment`。

### 1.3。启动之前自定义环境或ApplicationContext

一个`SpringApplication`具有`ApplicationListeners`与`ApplicationContextInitializers`被用于应用自定义的背景或环境。Spring Boot加载了许多此类自定义项，以供内部使用`META-INF/spring.factories`。注册其他自定义项的方法有多种：

- 通过对每个应用程序进行编程，方法是在运行它之前调用`addListeners`和`addInitializers`方法`SpringApplication`。
- 通过设置`context.initializer.classes`或`context.listener.classes`属性，以声明方式针对每个应用程序。
- 声明性地，对于所有应用程序，通过添加`META-INF/spring.factories`和打包一个jar文件，这些文件都被应用程序用作库。

该`SpringApplication`发送一些特殊`ApplicationEvents`的听众（背景下创造了一些甚至更早），然后注册了在公布的事件监听器`ApplicationContext`为好。有关完整列表，请参见“ Spring Boot功能”部分中的“ [应用程序事件和侦听器](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-application-events-and-listeners) ”。

还可以使用来自定义`Environment`刷新应用程序上下文之前的`EnvironmentPostProcessor`。每个实现都应在中注册`META-INF/spring.factories`，如以下示例所示：

```properties
org.springframework.boot.env.EnvironmentPostProcessor=com.example.YourEnvironmentPostProcessor
```

该实现可以加载任意文件并将其添加到中`Environment`。例如，以下示例从类路径加载YAML配置文件：

```java
public class EnvironmentPostProcessorExample implements EnvironmentPostProcessor {

    private final YamlPropertySourceLoader loader = new YamlPropertySourceLoader();

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        Resource path = new ClassPathResource("com/example/myapp/config.yml");
        PropertySource<?> propertySource = loadYaml(path);
        environment.getPropertySources().addLast(propertySource);
    }

    private PropertySource<?> loadYaml(Resource path) {
        if (!path.exists()) {
            throw new IllegalArgumentException("Resource " + path + " does not exist");
        }
        try {
            return this.loader.load("custom-resource", path).get(0);
        }
        catch (IOException ex) {
            throw new IllegalStateException("Failed to load yaml configuration from " + path, ex);
        }
    }

}
```

|      | 在`Environment`已经准备与所有常见的财产来源春天引导加载默认。因此，可以从环境中获取文件的位置。前面的示例将`custom-resource`属性源添加到列表的末尾，以便在其他任何常见位置定义的键具有优先权。定制实现可以定义另一个顺序。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 在使用`@PropertySource`你的`@SpringBootApplication`似乎是加载在一个自定义资源的便利方式`Environment`，我们不建议这样做。`Environment`在刷新应用程序上下文之前，不会将此类属性源添加到中。现在配置某些属性（如`logging.*`和`spring.main.*`在刷新开始前已读取）为时已晚。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 1.4。建立ApplicationContext层次结构（添加父上下文或根上下文）

您可以使用`ApplicationBuilder`该类创建父级/子级`ApplicationContext`层次结构。有关更多信息，请参见“ Spring Boot功能”部分中的“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-fluent-builder-api) ”。

### 1.5。创建一个非Web应用程序

并非所有的Spring应用程序都必须是Web应用程序（或Web服务）。如果要在`main`方法中执行一些代码，又要引导Spring应用程序以设置要使用的基础结构，则可以使用`SpringApplication`Spring Boot 的功能。A 根据`SpringApplication`其`ApplicationContext`是否认为需要Web应用程序来更改其类。您可以做的第一件事是让服务器相关的依赖项（例如Servlet API）脱离类路径。如果你不能做到这一点（例如，您从相同的代码库的两个应用程序），那么你可以显式调用`setWebApplicationType(WebApplicationType.NONE)`您的`SpringApplication`实例或设置`applicationContextClass`属性（通过Java API或与外部属性）。您可以将要作为业务逻辑运行的应用程序代码实现为`CommandLineRunner`并作为`@Bean`定义放到上下文中。

## 2.属性和配置

本节包括有关设置和读取属性和配置设置以及它们与Spring Boot应用程序的交互的主题。

### 2.1。在构建时自动扩展属性

除了可以对项目的构建配置中也指定的某些属性进行硬编码之外，您可以使用现有的构建配置自动扩展它们。在Maven和Gradle中都是可能的。

#### 2.1.1。使用Maven自动扩展属性

您可以使用资源过滤从Maven项目自动扩展属性。如果使用`spring-boot-starter-parent`，则可以使用`@..@`占位符引用Maven的“项目属性” ，如以下示例所示：

```properties
app.encoding=@project.build.sourceEncoding@
app.java.version=@java.version@
```

|      | 这样只会过滤生产配置（也就是说，不会对进行过滤`src/test/resources`）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果启用该`addResources`标志，则`spring-boot:run`目标可以`src/main/resources`直接添加到类路径中（用于热重载）。这样做避免了资源过滤和此功能。相反，您可以使用`exec:java`目标或自定义插件的配置。有关更多详细信息，请参见[插件使用情况页面](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin//usage.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您不使用入门级父级，则需要在``元素中包括以下元素`pom.xml`：

```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```

您还需要在其中包含以下元素``：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-resources-plugin</artifactId>
    <version>2.7</version>
    <configuration>
        <delimiters>
            <delimiter>@</delimiter>
        </delimiters>
        <useDefaultDelimiters>false</useDefaultDelimiters>
    </configuration>
</plugin>
```

|      | `useDefaultDelimiters`如果`${placeholder}`在配置中使用标准的Spring占位符（例如），则 该属性很重要。如果该属性未设置为`false`，则可以通过构建扩展这些属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 2.1.2。使用Gradle自动扩展属性

您可以通过配置Java插件的`processResources`任务来自动扩展Gradle项目中的属性，如以下示例所示：

```groovy
processResources {
    expand(project.properties)
}
```

然后，您可以使用占位符来引用Gradle项目的属性，如以下示例所示：

```properties
app.name=${name}
app.description=${description}
```

|      | Gradle的`expand`方法使用Groovy 的方法`SimpleTemplateEngine`来转换`${..}`令牌。该`${..}`风格的冲突与Spring自己的财产占位符机制。要将Spring属性占位符与自动扩展一起使用，请按以下步骤对Spring属性占位符进行转义：`\${..}`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.2。外化配置`SpringApplication`

A `SpringApplication`具有bean属性（主要是setter），因此在创建应用程序时可以使用其Java API修改其行为。或者，您可以通过在中设置属性来外部化配置`spring.main.*`。例如，在中`application.properties`，您可能具有以下设置：

```properties
spring.main.web-application-type=none
spring.main.banner-mode=off
```

然后，Spring Boot标语不会在启动时打印，并且应用程序也不会启动嵌入式Web服务器。

外部配置中定义的属性会覆盖用Java API指定的值，但用于创建的源的显着例外除外`ApplicationContext`。考虑以下应用程序：

```java
new SpringApplicationBuilder()
    .bannerMode(Banner.Mode.OFF)
    .sources(demo.MyApp.class)
    .run(args);
```

现在考虑以下配置：

```properties
spring.main.sources=com.acme.Config,com.acme.ExtraConfig
spring.main.banner-mode=console
```

实际应用中*，现在*示出的旗帜（如通过配置覆盖），并使用了三个源`ApplicationContext`（按以下顺序）： ，`demo.MyApp`，`com.acme.Config`和`com.acme.ExtraConfig`。

### 2.3。更改应用程序外部属性的位置

默认情况下，来自不同来源的属性`Environment`将以定义的顺序添加到Spring 中（有关确切顺序，请参见“ Spring Boot功能”部分中的“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config) ”）。

您还可以提供以下系统属性（或环境变量）来更改行为：

- `spring.config.name`（`SPRING_CONFIG_NAME`）：默认为`application`作为文件名的根。
- `spring.config.location`（`SPRING_CONFIG_LOCATION`）：要加载的文件（例如类路径资源或URL）。`Environment`为此文档设置了单独的属性源，可以通过系统属性，环境变量或命令行来覆盖它。

无论您在环境中进行什么设置，Spring Boot都将始终`application.properties`如上所述进行加载。默认情况下，如果使用YAML，则扩展名为'.yml'的文件也将添加到列表中。

Spring Boot记录在该`DEBUG`级别加载的配置文件以及在该级别找不到的候选文件`TRACE`。

请参阅参考资料[`ConfigFileApplicationListener`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java)。

### 2.4。使用“简短”命令行参数

有些人喜欢使用（例如）`--port=9000`而不是`--server.port=9000`在命令行上设置配置属性。您可以通过在中使用占位符来启用此行为`application.properties`，如以下示例所示：

```properties
server.port=${port:8080}
```

|      | 如果您从`spring-boot-starter-parent`POM 继承，则将的默认过滤器令牌`maven-resources-plugins`从更改`${*}`为`@`（即，`@maven.token@`而不是`${maven.token}`），以防止与Spring样式的占位符冲突。如果您`application.properties`直接启用了Maven过滤，则可能还需要更改默认过滤器令牌以使用[其他定界符](https://maven.apache.org/plugins/maven-resources-plugin/resources-mojo.html#delimiters)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 在这种特定情况下，端口绑定可在PaaS环境（例如Heroku或Cloud Foundry）中工作。在这两个平台中，`PORT`环境变量是自动设置的，Spring可以绑定到大写的`Environment`属性同义词。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 2.5。对外部属性使用YAML

YAML是JSON的超集，因此是一种方便的语法，用于以分层格式存储外部属性，如以下示例所示：

```yaml
spring:
    application:
        name: cruncher
    datasource:
        driverClassName: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost/test
server:
    port: 9000
```

创建一个名为的文件`application.yml`，并将其放在类路径的根目录中。然后添加`snakeyaml`到您的依赖项（Maven坐标`org.yaml:snakeyaml`，如果使用，则已经包含在内`spring-boot-starter`）。将YAML文件解析为Java `Map`（如JSON对象），然后Spring Boot展宽地图，使其深一层，并具有句点分隔的键，这是许多人习惯使用`Properties`Java中的文件的原因。

前面的示例YAML对应于以下`application.properties`文件：

```properties
spring.application.name=cruncher
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost/test
server.port=9000
```

请参阅“ Spring Boot功能”部分中的“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-yaml) ”，以获取有关YAML的更多信息。

### 2.6。设置活动弹簧轮廓

Spring `Environment`为此提供了一个API，但是您通常会设置一个System属性（`spring.profiles.active`）或OS环境变量（`SPRING_PROFILES_ACTIVE`）。另外，您可以使用`-D`参数启动应用程序（请记住将其放在主类或jar存档之前），如下所示：

```
$ java -jar -Dspring.profiles.active =生产演示-0.0.1-SNAPSHOT.jar
```

在Spring Boot中，您还可以在中设置活动配置文件`application.properties`，如以下示例所示：

```properties
spring.profiles.active=production
```

以这种方式设置的值将由“系统”属性或环境变量设置代替，而不由`SpringApplicationBuilder.profiles()`方法替代。因此，后一种Java API可用于扩充配置文件，而无需更改默认值。

有关更多信息，请参见“ Spring Boot功能”部分中的“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-profiles) ”。

### 2.7。根据环境更改配置

YAML文件实际上是由`---`行分隔的文档序列，每个文档都分别解析为展平图。

如果YAML文档包含`spring.profiles`密钥，则将配置文件值（以逗号分隔的配置文件列表）馈入Spring `Environment.acceptsProfiles()`方法。如果这些配置文件中的任何一个处于活动状态，那么该文档将包含在最终合并中（否则，它不会包含在此文档中），如以下示例所示：

```yaml
server:
    port: 9000
---

spring:
    profiles: development
server:
    port: 9001

---

spring:
    profiles: production
server:
    port: 0
```

在前面的示例中，默认端口为9000。但是，如果名为“ development”的Spring概要文件处于活动状态，则该端口为9001。如果“ production”为活动状态，则该端口为0。

|      | YAML文档按照它们遇到的顺序进行合并。以后的值将覆盖以前的值。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要对属性文件执行相同的操作，可以使用`application-${profile}.properties`指定特定于配置文件的值。

### 2.8。发现外部属性的内置选项

Spring Boot 在运行时将来自`application.properties`（或`.yml`文件和其他位置）的外部属性绑定到应用程序中。在一个位置上没有（而且从技术上来说不是）所有受支持属性的详尽列表，因为贡献可能来自类路径上的其他jar文件。

具有Actuator功能的正在运行的应用程序具有一个`configprops`终结点，该终结点显示了可通过访问的所有绑定和可绑定属性`@ConfigurationProperties`。

附录中包含一个[`application.properties`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html#common-application-properties)示例，其中列出了Spring Boot支持的最常见属性。最终的列表来自搜索源代码中的`@ConfigurationProperties`和`@Value`注释，以及偶尔使用`Binder`。有关加载属性的确切顺序的更多信息，请参见“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config) ”。

## 3.嵌入式Web服务器

每个Spring Boot Web应用程序都包含一个嵌入式Web服务器。此功能导致许多方法问题，包括如何更改嵌入式服务器以及如何配置嵌入式服务器。本节回答这些问题。

### 3.1。使用其他Web服务器

许多Spring Boot启动器都包含默认的嵌入式容器。

- 对于servlet堆栈应用程序，通过`spring-boot-starter-web`包括来包括Tomcat `spring-boot-starter-tomcat`，但是您可以使用`spring-boot-starter-jetty`或`spring-boot-starter-undertow`代替。
- 对于反应栈的应用，`spring-boot-starter-webflux`包括反应堆的Netty通过包括`spring-boot-starter-reactor-netty`，但你可以使用`spring-boot-starter-tomcat`，`spring-boot-starter-jetty`或`spring-boot-starter-undertow`代替。

切换到其他HTTP服务器时，除了包括所需的依赖关系之外，还需要排除默认的依赖关系。Spring Boot为HTTP服务器提供了单独的启动器，以帮助简化该过程。

以下Maven示例显示了如何排除Tomcat并包括Jetty for Spring MVC：

```xml
<properties>
    <servlet-api.version>3.1.0</servlet-api.version>
</properties>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- Exclude the Tomcat dependency -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- Use Jetty instead -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

|      | Servlet API的版本已被覆盖，因为与Tomcat 9和Undertow 2.0不同，Jetty 9.4不支持Servlet 4.0。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

以下Gradle示例显示了如何排除Netty并为Spring WebFlux包括Undertow：

```groovy
configurations {
    // exclude Reactor Netty
    compile.exclude module: 'spring-boot-starter-reactor-netty'
}

dependencies {
    compile 'org.springframework.boot:spring-boot-starter-webflux'
    // Use Undertow instead
    compile 'org.springframework.boot:spring-boot-starter-undertow'
    // ...
}
```

|      | `spring-boot-starter-reactor-netty`使用`WebClient`该类是必需的，因此即使您需要包括其他HTTP服务器，也可能需要保持对Netty的依赖。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.2。禁用Web服务器

如果您的类路径包含启动Web服务器所需的位，则Spring Boot将自动启动它。要禁用此行为，请`WebApplicationType`在中配置`application.properties`，如以下示例所示：

```properties
spring.main.web-application-type=none
```

### 3.3。更改HTTP端口

在独立应用程序中，主HTTP端口默认为，`8080`但可以使用`server.port`（例如，在`application.properties`System属性中或作为System属性）进行设置。由于轻松地绑定了`Environment`值，因此您也可以使用`SERVER_PORT`（例如，作为OS环境变量）。

要完全关闭HTTP端点但仍创建一个`WebApplicationContext`，请使用`server.port=-1`（这样做有时对测试很有用）。

有关更多详细信息，请参阅“ Spring Boot功能”部分中的“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-customizing-embedded-containers) ”或[`ServerProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)源代码。

### 3.4。使用随机未分配的HTTP端口

要扫描可用端口（使用操作系统本机来防止冲突），请使用`server.port=0`。

### 3.5。在运行时发现HTTP端口

您可以从日志输出或`ServletWebServerApplicationContext`通过其端口访问服务器正在运行的端口`WebServer`。最好的方法是确保它已初始化，是添加一个`@Bean`type `ApplicationListener`并将其发布时将其拉出事件。

使用的测试`@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)`还可以通过使用`@LocalServerPort`批注将实际端口注入字段，如以下示例所示：

```java
@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)
public class MyWebIntegrationTests {

    @Autowired
    ServletWebServerApplicationContext server;

    @LocalServerPort
    int port;

    // ...

}
```

|      | `@LocalServerPort`是的元注释`@Value("${local.server.port}")`。不要尝试在常规应用程序中注入端口。如我们所见，仅在容器初始化之后才设置该值。与测试相反，应早处理应用程序代码回调（在实际可用该值之前）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 3.6。启用HTTP响应压缩

Jetty，Tomcat和Undertow支持HTTP响应压缩。可以在中启用它`application.properties`，如下所示：

```properties
server.compression.enabled=true
```

默认情况下，响应的长度必须至少为2048个字节才能执行压缩。您可以通过设置`server.compression.min-response-size`属性来配置此行为。

默认情况下，仅当响应的内容类型为以下之一时，它们才被压缩：

- `text/html`
- `text/xml`
- `text/plain`
- `text/css`
- `text/javascript`
- `application/javascript`
- `application/json`
- `application/xml`

您可以通过设置`server.compression.mime-types`属性来配置此行为。

### 3.7。配置SSL

可以通过设置各种`server.ssl.*`属性来声明性地配置SSL ，通常在`application.properties`或中`application.yml`。以下示例显示了在中设置SSL属性`application.properties`：

```properties
server.port=8443
server.ssl.key-store=classpath:keystore.jks
server.ssl.key-store-password=secret
server.ssl.key-password=another-secret
```

有关[`Ssl`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/web/server/Ssl.java)所有受支持属性的详细信息，请参见。

使用前面示例中的配置意味着应用程序不再在端口8080上支持纯HTTP连接器。SpringBoot不支持通过进行HTTP连接器和HTTPS连接器的配置`application.properties`。如果要同时拥有两者，则需要以编程方式配置其中之一。我们建议您使用`application.properties`HTTPS进行配置，因为HTTP连接器是两者中以编程方式进行配置的较容易方式。

### 3.8。配置HTTP / 2

您可以使用`server.http2.enabled`配置属性在Spring Boot应用程序中启用HTTP / 2支持。这种支持取决于所选的Web服务器和应用程序环境，因为JDK8不立即支持该协议。

|      | Spring Boot不支持`h2c`HTTP / 2协议的明文版本。因此，您必须先[配置SSL](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-configure-ssl)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.8.1。带有Undertow的HTTP / 2

从Undertow 1.4.0+开始，在JDK8上没有任何其他要求就支持HTTP / 2。

#### 3.8.2。HTTP / 2与码头

从Jetty 9.4.8开始，[Conscrypt库](https://www.conscrypt.org/)还支持HTTP / 2 。要启用该支持，您的应用程序需要具有两个附加依赖项：`org.eclipse.jetty:jetty-alpn-conscrypt-server`和`org.eclipse.jetty.http2:http2-server`。

#### 3.8.3。Tomcat的HTTP / 2

默认情况下，Spring Boot随Tomcat 9.0.x一起提供，当使用JDK 9或更高版本时，Tomcat 9.0.x支持HTTP / 2。另外，如果`libtcnative`库及其依赖项已安装在主机操作系统上，则可以在JDK 8上使用HTTP / 2 。

必须将库文件夹提供给JVM库路径（如果尚未提供）。您可以使用JVM参数（例如）来执行此操作`-Djava.library.path=/usr/local/opt/tomcat-native/lib`。有关更多信息，请参见[Tomcat官方文档](https://tomcat.apache.org/tomcat-9.0-doc/apr.html)。

在没有该本机支持的情况下，在JDK 8上启动Tomcat 9.0.x会记录以下错误：

```
错误8787-[[main] oacoyote.http11.Http11NioProtocol：[h2]的升级处理程序[org.apache.coyote.http2.Http2Protocol]仅支持通过ALPN进行升级，但已为[“ https-jsse-nio”配置-8443“]不支持ALPN的连接器。
```

此错误不是致命错误，并且该应用程序仍以HTTP / 1.1 SSL支持开头。

#### 3.8.4。HTTP / 2和Reactor Netty

在`spring-boot-webflux-starter`默认情况下，反应堆的Netty作为服务器使用。使用JDK 9或更高版本的JDK支持，可以将Reactor Netty配置为HTTP / 2。对于JDK 8环境或最佳运行时性能，此服务器还支持带有本机库的HTTP / 2。为此，您的应用程序需要具有其他依赖项。

Spring Boot管理`io.netty:netty-tcnative-boringssl-static`“超级罐” 的版本，其中包含所有平台的本机库。开发人员可以选择使用分类器仅导入所需的依赖项（请参见[Netty官方文档](https://netty.io/wiki/forked-tomcat-native.html)）。

### 3.9。配置Web服务器

通常，您应该首先考虑使用许多可用的配置键之一，并通过在您的`application.properties`（或`application.yml`，或环境等）中添加新条目来自定义Web服务器。请参阅“ [发现外部属性的内置选项](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-discover-build-in-options-for-external-properties) ”。该`server.*`命名空间是非常有用的在这里，它包括命名空间一样`server.tomcat.*`，`server.jetty.*`和其他人，对服务器的特定功能。请参阅[appendix-application-properties.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html#common-application-properties)的列表。

前面的部分已经介绍了许多常见的用例，例如压缩，SSL或HTTP / 2。但是，如果您的用例不存在配置密钥，则应查看[`WebServerFactoryCustomizer`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/api//org/springframework/boot/web/server/WebServerFactoryCustomizer.html)。您可以声明一个这样的组件并访问与您选择的服务器相关的工厂：您应该为所选服务器（Tomcat，Jetty，Reactor Netty，Undertow）和所选Web堆栈（Servlet或Reactive）选择变体。

以下示例适用于具有`spring-boot-starter-web`（Servlet堆栈）的Tomcat ：

```java
@Component
public class MyTomcatWebServerCustomizer
        implements WebServerFactoryCustomizer<TomcatServletWebServerFactory> {

    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        // customize the factory here
    }
}
```

此外，Spring Boot还提供：

| 服务器 | Servlet堆栈                       | 反应堆                             |
| :----- | :-------------------------------- | :--------------------------------- |
| 雄猫   | `TomcatServletWebServerFactory`   | `TomcatReactiveWebServerFactory`   |
| 码头   | `JettyServletWebServerFactory`    | `JettyReactiveWebServerFactory`    |
| 底拖   | `UndertowServletWebServerFactory` | `UndertowReactiveWebServerFactory` |
| 反应堆 | 不适用                            | `NettyReactiveWebServerFactory`    |

一旦可以访问`WebServerFactory`，您通常可以向其添加定制程序以配置特定的部分，例如连接器，服务器资源或服务器本身-全部使用服务器特定的API。

最后，您还可以声明自己的`WebServerFactory`组件，该组件将覆盖Spring Boot提供的组件。在这种情况下，您不能再依赖`server`命名空间中的配置属性。

### 3.10。将Servlet，过滤器或侦听器添加到应用程序

在一个servlet栈的应用，即用`spring-boot-starter-web`，有两种方法可以添加`Servlet`，`Filter`，`ServletContextListener`，和由Servlet API到您的应用程序支持的其他听众：

- [使用Spring Bean添加Servlet，过滤器或侦听器](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-add-a-servlet-filter-or-listener-as-spring-bean)
- [使用类路径扫描添加Servlet，过滤器和侦听器](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-add-a-servlet-filter-or-listener-using-scanning)

#### 3.10.1。使用Spring Bean添加Servlet，过滤器或侦听器

要添加`Servlet`，`Filter`或servlet `*Listener`使用的Spring bean，你必须提供一个`@Bean`它的定义。当您要注入配置或依赖项时，这样做非常有用。但是，您必须非常小心，以免引起过多其他bean的急切初始化，因为必须在应用程序生命周期的早期就将它们安装在容器中。（例如，让它们依赖于您的`DataSource`或JPA配置不是一个好主意。）您可以通过在第一次使用bean时而不是在初始化时进行延迟初始化来解决这些限制。

对于`Filters`和`Servlets`，您还可以通过添加`FilterRegistrationBean`或`ServletRegistrationBean`代替基础组件或在基础组件之外添加映射和init参数。

|      | 如果`dispatcherType`在过滤器注册上未指定no ，`REQUEST`则使用。这符合Servlet规范的默认调度程序类型。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

像其他任何Spring bean一样，您可以定义Servlet过滤器bean的顺序。请确保检查“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-embedded-container-servlets-filters-listeners-beans) ”部分。

##### 禁用Servlet或过滤器的注册

正如[前面所述](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-add-a-servlet-filter-or-listener-as-spring-bean)，任何`Servlet`或`Filter`豆与servlet容器自动注册。要禁用特定`Filter`或`Servlet`bean的注册，请为其创建注册bean并将其标记为已禁用，如以下示例所示：

```java
@Bean
public FilterRegistrationBean registration(MyFilter filter) {
    FilterRegistrationBean registration = new FilterRegistrationBean(filter);
    registration.setEnabled(false);
    return registration;
}
```

#### 3.10.2。使用类路径扫描添加Servlet，过滤器和侦听器

`@WebServlet`，，`@WebFilter`和带`@WebListener`注释的类可以通过向注释一个`@Configuration`类`@ServletComponentScan`并指定包含要注册的组件的包来自动向嵌入式servlet容器注册。默认情况下，`@ServletComponentScan`从带注释的类的包进行扫描。

### 3.11。配置访问日志

可以通过它们各自的名称空间为Tomcat，Undertow和Jetty配置访问日志。

例如，以下设置使用[自定义模式](https://tomcat.apache.org/tomcat-9.0-doc/config/valve.html#Access_Logging)记录对Tomcat的访问。

```properties
server.tomcat.basedir=my-tomcat
server.tomcat.accesslog.enabled=true
server.tomcat.accesslog.pattern=%t %a "%r" %s (%D ms)
```

|      | 日志的默认位置是`logs`相对于Tomcat基本目录的目录。默认情况下，该`logs`目录是一个临时目录，因此您可能需要修复Tomcat的基本目录或对日志使用绝对路径。在前面的示例中，日志`my-tomcat/logs`相对于应用程序的工作目录可用。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

可以用类似的方式配置Undertow的访问日志，如以下示例所示：

```properties
server.undertow.accesslog.enabled=true
server.undertow.accesslog.pattern=%t %a "%r" %s (%D ms)
```

日志存储在`logs`相对于应用程序工作目录的目录中。您可以通过设置`server.undertow.accesslog.dir`属性来自定义此位置。

最后，Jetty的访问日志也可以配置如下：

```properties
server.jetty.accesslog.enabled=true
server.jetty.accesslog.filename=/var/log/jetty-access.log
```

默认情况下，日志重定向到`System.err`。有关更多详细信息，请参见[Jetty文档](https://www.eclipse.org/jetty/documentation/9.4.27.v20200227/configuring-jetty-request-logs.html)。

### 3.12。在前端代理服务器后面运行

如果您的应用程序是在代理，负载均衡器之后或在云中运行的，则请求信息（例如主机，端口，方案等）可能会随之变化。您的应用程序可能正在运行`10.10.10.10:8080`，但是HTTP客户端只能看到`example.org`。

[RFC7239“转发的报头”](https://tools.ietf.org/html/rfc7239)定义了`Forwarded`HTTP报头；代理可以使用此标头提供有关原始请求的信息。您可以将应用程序配置为读取这些标头，并在创建链接并将其发送到HTTP 302响应，JSON文档或HTML页面中的客户端时自动使用该信息。还有一些非标准的标题，如：`X-Forwarded-Host`，`X-Forwarded-Port`，`X-Forwarded-Proto`，`X-Forwarded-Ssl`，和`X-Forwarded-Prefix`。

如果代理添加常用`X-Forwarded-For`和`X-Forwarded-Proto`标题，设置`server.forward-headers-strategy`到`NATIVE`足以支持这些。使用此选项，Web服务器本身就本身支持此功能。您可以查看他们的特定文档以了解特定行为。

如果这还不够的话，Spring Framework将提供[ForwardedHeaderFilter](https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/web.html#filters-forwarded-headers)。通过将设置`server.forward-headers-strategy`为，可以将其注册为应用程序中的Servlet过滤器`FRAMEWORK`。

|      | 如果您的应用程序在Cloud Foundry或Heroku中运行，则该`server.forward-headers-strategy`属性默认为`NATIVE`。在所有其他情况下，默认为`NONE`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 3.12.1。自定义Tomcat的代理配置

如果使用Tomcat，则可以另外配置用于携带“转发的”信息的标头名称，如以下示例所示：

```
server.tomcat.remote-ip-header = x-您的远程IP-header
server.tomcat.protocol-header = x-您的协议头
```

Tomcat还配置有一个默认正则表达式，该正则表达式与要信任的内部代理匹配。默认情况下，IP地址`10/8`，`192.168/16`，`169.254/16`并且`127/8`是值得信赖的。您可以通过在上添加一个条目来自定义阀门的配置`application.properties`，如以下示例所示：

```
server.tomcat.internal-proxies = 192 \\。168 \\。\\ d {1,3} \\。\\ d {1,3}
```

|      | 仅当使用属性文件进行配置时，才需要双反斜杠。如果使用YAML，则单个反斜杠就足够了，并且与上一个示例中所示的值相等`192\.168\.\d{1,3}\.\d{1,3}`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 您可以通过将设置`internal-proxies`为空来信任所有代理（但在生产环境中不要这样做）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

您可以`RemoteIpValve`通过关闭自动开关（设置为set `server.forward-headers-strategy=NONE`）并在`TomcatServletWebServerFactory`bean中添加新的Valve实例来完全控制Tomcat的配置。

### 3.13。使用Tomcat启用多个连接器

您可以在中添加一个，`org.apache.catalina.connector.Connector`以`TomcatServletWebServerFactory`允许多个连接器，包括HTTP和HTTPS连接器，如以下示例所示：

```java
@Bean
public ServletWebServerFactory servletContainer() {
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
    tomcat.addAdditionalTomcatConnectors(createSslConnector());
    return tomcat;
}

private Connector createSslConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
    try {
        File keystore = new ClassPathResource("keystore").getFile();
        File truststore = new ClassPathResource("keystore").getFile();
        connector.setScheme("https");
        connector.setSecure(true);
        connector.setPort(8443);
        protocol.setSSLEnabled(true);
        protocol.setKeystoreFile(keystore.getAbsolutePath());
        protocol.setKeystorePass("changeit");
        protocol.setTruststoreFile(truststore.getAbsolutePath());
        protocol.setTruststorePass("changeit");
        protocol.setKeyAlias("apitester");
        return connector;
    }
    catch (IOException ex) {
        throw new IllegalStateException("can't access keystore: [" + keystore
                + "] or truststore: [" + truststore + "]", ex);
    }
}
```

### 3.14。使用Tomcat的LegacyCookieProcessor

默认情况下，Spring Boot使用的嵌入式Tomcat不支持Cookie格式的“版本0”，因此您可能会看到以下错误：

```
java.lang.IllegalArgumentException：Cookie值中存在无效字符[32]
```

如果可能的话，您应该考虑将代码更新为仅存储符合以后Cookie规范的值。但是，如果无法更改cookie的编写方式，则可以将Tomcat配置为使用`LegacyCookieProcessor`。要切换到`LegacyCookieProcessor`，请使用`WebServerFactoryCustomizer`添加了的Bean `TomcatContextCustomizer`，如以下示例所示：

```java
@Bean
public WebServerFactoryCustomizer<TomcatServletWebServerFactory> cookieProcessorCustomizer() {
    return (factory) -> factory
            .addContextCustomizers((context) -> context.setCookieProcessor(new LegacyCookieProcessor()));
}
```

### 3.15。启用Tomcat的MBean注册表

默认情况下，嵌入式Tomcat的MBean注册表是禁用的。这样可以最大程度地减少Tomcat的内存占用。例如，如果要使用Tomcat的MBean，以便可以将它们用于通过Micrometer公开指标，则必须使用`server.tomcat.mbeanregistry.enabled`属性来这样做，如以下示例所示：

```properties
server.tomcat.mbeanregistry.enabled=true
```

### 3.16。使用Undertow启用多个侦听器

在中添加一个`UndertowBuilderCustomizer`，`UndertowServletWebServerFactory`并在中添加一个侦听器`Builder`，如以下示例所示：

```java
@Bean
public UndertowServletWebServerFactory servletWebServerFactory() {
    UndertowServletWebServerFactory factory = new UndertowServletWebServerFactory();
    factory.addBuilderCustomizers(new UndertowBuilderCustomizer() {

        @Override
        public void customize(Builder builder) {
            builder.addHttpListener(8080, "0.0.0.0");
        }

    });
    return factory;
}
```

### 3.17。使用@ServerEndpoint创建WebSocket端点

如果要在使用`@ServerEndpoint`嵌入式容器的Spring Boot应用程序中使用，必须声明一个single `ServerEndpointExporter` `@Bean`，如以下示例所示：

```java
@Bean
public ServerEndpointExporter serverEndpointExporter() {
    return new ServerEndpointExporter();
}
```

前面示例中显示的`@ServerEndpoint`Bean在基础WebSocket容器中注册了所有带注释的Bean。当部署到独立servlet容器时，此角色由servlet容器初始化程序执行，并且`ServerEndpointExporter`不需要bean。

## 4. Spring MVC

Spring Boot有许多启动器，其中包括Spring MVC。请注意，一些入门者包括对Spring MVC的依赖，而不是直接包含它。本部分回答有关Spring MVC和Spring Boot的常见问题。

### 4.1。编写JSON REST服务

`@RestController`只要Jackson2在类路径上，Spring Boot应用程序中的任何Spring 默认情况下都应呈现JSON响应，如以下示例所示：

```java
@RestController
public class MyController {

    @RequestMapping("/thing")
    public MyThing thing() {
            return new MyThing();
    }

}
```

只要`MyThing`可以通过Jackson2进行序列化（对于普通的POJO或Groovy对象为true），则`localhost:8080/thing`默认情况下会为其提供JSON表示。请注意，在浏览器中，有时您可能会看到XML响应，因为浏览器倾向于发送更喜欢XML的接受标头。

### 4.2。编写XML REST服务

如果`jackson-dataformat-xml`类路径上具有Jackson XML扩展名（），则可以使用它来呈现XML响应。我们用于JSON的先前示例可以正常工作。要使用Jackson XML渲染器，请将以下依赖项添加到您的项目中：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

如果无法使用Jackson的XML扩展名而可以使用JAXB，则可以将XML `MyThing`注释为`@XmlRootElement`，以使其具有附加要求，如以下示例所示：

```java
@XmlRootElement
public class MyThing {
    private String name;
    // .. getters and setters
}
```

JAXB仅在Java 8中是开箱即用的。如果您使用的是较新的Java版本，请在项目中添加以下依赖项：

```xml
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
</dependency>
```

|      | 要使服务器呈现XML而不是JSON，您可能必须发送`Accept: text/xml`标头（或使用浏览器）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.3。自定义Jackson ObjectMapper

Spring MVC（客户端和服务器端）用于`HttpMessageConverters`协商HTTP交换中的内容转换。如果Jackson在类路径中，则您已经获得所提供的默认转换器，该转换器`Jackson2ObjectMapperBuilder`的实例已为您自动配置。

该`ObjectMapper`（或`XmlMapper`为杰克逊XML转换器）实例（默认创建）具有以下定义的属性：

- `MapperFeature.DEFAULT_VIEW_INCLUSION` 被禁用
- `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES` 被禁用
- `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS` 被禁用

Spring Boot还具有一些功能，可以更轻松地自定义此行为。

您可以使用环境配置`ObjectMapper`和`XmlMapper`实例。杰克逊提供了一套广泛的简单的开/关功能，可用于配置其处理的各个方面。在六个枚举（在Jackson中）中描述了这些功能，这些枚举映射到环境中的属性：

| 枚举                                                    | 属性                                        | 价值观                                                       |
| :------------------------------------------------------ | :------------------------------------------ | :----------------------------------------------------------- |
| `com.fasterxml.jackson.databind.DeserializationFeature` | `spring.jackson.deserialization.`           | `true`， `false`                                             |
| `com.fasterxml.jackson.core.JsonGenerator.Feature`      | `spring.jackson.generator.`                 | `true`， `false`                                             |
| `com.fasterxml.jackson.databind.MapperFeature`          | `spring.jackson.mapper.`                    | `true`， `false`                                             |
| `com.fasterxml.jackson.core.JsonParser.Feature`         | `spring.jackson.parser.`                    | `true`， `false`                                             |
| `com.fasterxml.jackson.databind.SerializationFeature`   | `spring.jackson.serialization.`             | `true`， `false`                                             |
| `com.fasterxml.jackson.annotation.JsonInclude.Include`  | `spring.jackson.default-property-inclusion` | `always`，`non_null`，`non_absent`，`non_default`，`non_empty` |

例如，要启用漂亮打印，请设置`spring.jackson.serialization.indent_output=true`。请注意，由于使用了[宽松的绑定](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-relaxed-binding)，的大小写`indent_output`不必与相应的枚举常量的大小写匹配`INDENT_OUTPUT`。

这种基于环境的配置将应用于自动配置的`Jackson2ObjectMapperBuilder`bean，并应用于使用构建器创建的任何映射器，包括自动配置的`ObjectMapper`bean。

上下文`Jackson2ObjectMapperBuilder`可以由一个或多个`Jackson2ObjectMapperBuilderCustomizer`bean 自定义。可以对此类定制器bean进行排序（Boot自己的定制器的顺序为0），从而可以在Boot定制之前和之后应用其他定制。

任何类型的bean都会`com.fasterxml.jackson.databind.Module`自动向自动配置中注册，`Jackson2ObjectMapperBuilder`并应用于`ObjectMapper`它创建的任何实例。当您向应用程序中添加新功能时，这提供了一种用于贡献自定义模块的全局机制。

如果要`ObjectMapper`完全替换默认值，请定义`@Bean`该类型的a并将其标记为，`@Primary`或者，如果您更喜欢基于构建器的方法，请定义一个`Jackson2ObjectMapperBuilder` `@Bean`。请注意，无论哪种情况，这样做都会禁用的所有自动配置`ObjectMapper`。

如果提供任何`@Beans`type `MappingJackson2HttpMessageConverter`，它们将替换MVC配置中的默认值。此外，`HttpMessageConverters`还提供了一种类型的便捷bean （如果使用默认的MVC配置，该bean 始终可用）。它提供了一些有用的方法来访问默认的和用户增强的消息转换器。

请参阅“ [自定义@ResponseBody渲染](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-customize-the-responsebody-rendering) ”部分和[`WebMvcAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration.java)源代码以获取更多详细信息。

### 4.4。自定义@ResponseBody渲染

Spring用于`HttpMessageConverters`呈现`@ResponseBody`（或的响应`@RestController`）。您可以通过在Spring Boot上下文中添加适当类型的bean来贡献额外的转换器。如果您添加的bean的类型无论如何都是默认包含的（例如`MappingJackson2HttpMessageConverter`JSON转换），它将替换默认值。`HttpMessageConverters`提供了类型的便捷bean，如果您使用默认的MVC配置，它将始终可用。它提供了一些有用的方法来访问默认的和用户增强的消息转换器（例如，如果您想将它们手动注入到custom中，可能会很有用`RestTemplate`）。

与正常的MVC用法一样，`WebMvcConfigurer`您提供的任何bean也可以通过重写`configureMessageConverters`方法来贡献转换器。但是，与普通的MVC不同，您只能提供所需的其他转换器（因为Spring Boot使用相同的机制来提供其默认值）。最后，如果您通过提供自己的`@EnableWebMvc`配置选择退出Spring Boot的默认MVC 配置，则可以完全控制并使用`getMessageConverters`from 手动进行所有操作`WebMvcConfigurationSupport`。

请参阅[`WebMvcAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration.java)源代码以获取更多详细信息。

### 4.5。处理分段文件上传

Spring Boot包含Servlet 3 `javax.servlet.http.Part`API以支持上传文件。默认情况下，Spring Boot用单个请求将Spring MVC配置为每个文件最大大小为1MB，最大文件数据为10MB。您可以覆盖这些值，使用中间类提供`/tmp`的属性将中间数据存储到的位置（例如，存储到目录）以及将数据刷新到磁盘的阈值之上`MultipartProperties`。例如，如果要指定文件不受限制，请将`spring.servlet.multipart.max-file-size`属性设置为`-1`。

当您想在Spring MVC控制器处理程序方法中将多部分编码的文件数据作为带`@RequestParam`注释的类型的参数来接收时，多部分支持会很有帮助`MultipartFile`。

有关[`MultipartAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/MultipartAutoConfiguration.java)更多详细信息，请参见源。

|      | 建议使用容器的内置支持进行分段上传，而不要引入其他依赖项，例如Apache Commons File Upload。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 4.6。关闭Spring MVC DispatcherServlet

默认情况下，所有内容均从应用程序的根目录（`/`）提供。如果您希望映射到其他路径，则可以如下配置：

```properties
spring.mvc.servlet.path=/acme
```

如果您有其他servlet，则可以声明一个或`@Bean`类型，然后Spring Boot会将它们透明地注册到容器。由于servlet是通过这种方式注册的，因此可以将它们映射到的子上下文而无需调用它。`Servlet``ServletRegistrationBean``DispatcherServlet`

配置`DispatcherServlet`您自己是不寻常的，但是如果您确实需要这样做，则还必须提供一个`@Bean`type `DispatcherServletPath`以提供您的custom的路径`DispatcherServlet`。

### 4.7。关闭默认的MVC配置

完全控制MVC配置的最简单方法是为您自己`@Configuration`提供`@EnableWebMvc`注释。这样做会使您掌握所有MVC配置。

### 4.8。自定义ViewResolvers

A `ViewResolver`是Spring MVC的核心组件，将视图名称转换`@Controller`为实际的`View`实现。请注意，`ViewResolvers`它主要用于UI应用程序，而不是REST风格的服务（a `View`并不用于呈现a `@ResponseBody`）。有很多实现`ViewResolver`可供选择，Spring本身对是否应使用哪个没有意见。另一方面，Spring Boot根据在类路径和应用程序上下文中找到的内容为您安装一个或两个。会`DispatcherServlet`使用它在应用程序上下文中找到的所有解析器，依次尝试每个解析器，直到获得结果。如果添加自己的解析器，则必须知道其顺序以及解析器的添加位置。

`WebMvcAutoConfiguration`将以下内容添加`ViewResolvers`到您的上下文中：

- 一个`InternalResourceViewResolver`名为“defaultViewResolver”。此代码查找可以使用渲染的物理资源`DefaultServlet`（包括静态资源和JSP页面，如果使用的话）。它将一个前缀和一个后缀应用于视图名称，然后在Servlet上下文中查找具有该路径的物理资源（默认值均为空，但可以通过`spring.mvc.view.prefix`和进行外部配置访问`spring.mvc.view.suffix`）。您可以通过提供相同类型的bean覆盖它。
- 一个`BeanNameViewResolver`名为“beanNameViewResolver”。这是视图解析器链的有用成员，它拾取与`View`被解析名称相同的所有bean 。不必重写或替换它。
- `ContentNegotiatingViewResolver`仅当实际上存在类型**为** bean的情况下，才添加一个名为“ viewResolver”的视图`View`。这是一个“主”解析器，委派给所有其他解析器，并尝试查找与客户端发送的“ Accept” HTTP标头匹配的内容。有一个有用的[博客`ContentNegotiatingViewResolver`](https://spring.io/blog/2013/06/03/content-negotiation-using-views)，您可能想学习以了解更多信息，并且您也可以查看源代码以获取详细信息。您可以`ContentNegotiatingViewResolver`通过定义一个名为'viewResolver'的bean 来关闭自动配置。
- 如果您使用Thymeleaf，则还有一个`ThymeleafViewResolver`名为“ thymeleafViewResolver”。它通过在视图名称前后加上前缀和后缀来查找资源。前缀为`spring.thymeleaf.prefix`，后缀为`spring.thymeleaf.suffix`。前缀和后缀的值分别默认为“ classpath：/ templates /”和“ .html”。您可以`ThymeleafViewResolver`通过提供相同名称的bean 来覆盖。
- 如果您使用FreeMarker，则还有一个`FreeMarkerViewResolver`名为“ freeMarkerViewResolver”的文件。它`spring.freemarker.templateLoaderPath`通过在视图名称前加上前缀和后缀来在加载程序路径（已外部化并具有默认值'classpath：/ templates /'）中查找资源。前缀被外部`spring.freemarker.prefix`化为，后缀被外部化为`spring.freemarker.suffix`。前缀和后缀的默认值分别为空和'.ftlh'。您可以`FreeMarkerViewResolver`通过提供相同名称的bean 来覆盖。
- 如果使用Groovy模板（实际上，如果`groovy-templates`在类路径中），则还具有一个`GroovyMarkupViewResolver`名为“ groovyMarkupViewResolver”的文件。它通过在视图名称周围加上前缀和后缀（`spring.groovy.template.prefix`并扩展为和`spring.groovy.template.suffix`）来查找加载程序路径中的资源。前缀和后缀分别具有默认值'classpath：/ templates /'和'.tpl'。您可以`GroovyMarkupViewResolver`通过提供相同名称的bean 来覆盖。
- 如果您使用Moustache，则还有一个`MustacheViewResolver`名为“ mustacheViewResolver”的文件。它通过在视图名称前后加上前缀和后缀来查找资源。前缀为`spring.mustache.prefix`，后缀为`spring.mustache.suffix`。前缀和后缀的值分别默认为“ classpath：/ templates /”和“ .mustache”。您可以`MustacheViewResolver`通过提供相同名称的bean 来覆盖。

有关更多详细信息，请参见以下部分：

- [`WebMvcAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration.java)
- [`ThymeleafAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)
- [`FreeMarkerAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java)
- [`GroovyTemplateAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java)

## 5.使用Spring Security进行测试

Spring Security提供了对以特定用户身份运行测试的支持。例如，下面的代码段中的测试将与具有该`ADMIN`角色的经过身份验证的用户一起运行。

```java
@Test
@WithMockUser(roles="ADMIN")
public void requestProtectedUrlWithUser() throws Exception {
    mvc
        .perform(get("/"))
        ...
}
```

Spring Security提供了与Spring MVC Test的全面集成，并且在使用`@WebMvcTest`slice和测试控制器时也可以使用它`MockMvc`。

有关Spring Security的测试支持的更多详细信息，请参阅Spring Security的[参考文档](https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/htmlsingle/#test)。

## 6.球衣

### 6.1。使用Spring Security保护Jersey端点

可以使用Spring Security来保护基于Jersey的Web应用程序，其方式与用来保护基于Spring MVC的Web应用程序的方式几乎相同。但是，如果您想在Jersey上使用Spring Security的方法级安全性，则必须将Jersey配置为使用`setStatus(int)`pretty `sendError(int)`。这可以防止Jersey在Spring Security有机会向客户端报告身份验证或授权失败之前提交响应。

该`jersey.config.server.response.setStatusOverSendError`属性必须`true`在应用程序的`ResourceConfig`Bean 上设置为，如以下示例所示：

```java
@Component
public class JerseyConfig extends ResourceConfig {

    public JerseyConfig() {
        register(Endpoint.class);
        setProperties(Collections.singletonMap("jersey.config.server.response.setStatusOverSendError", true));
    }

}
```

### 6.2。与另一个Web框架一起使用Jersey

要将Jersey与其他Web框架（例如Spring MVC）一起使用，应对其进行配置，以便它将允许其他框架处理无法处理的请求。首先，通过将`spring.jersey.type`应用程序属性配置为值，将Jersey配置为使用过滤器而不是Servlet `filter`。其次，配置您的设备`ResourceConfig`以转发可能导致404的请求，如以下示例所示。

```java
@Component
public class JerseyConfig extends ResourceConfig {

    public JerseyConfig() {
        register(Endpoint.class);
        property(ServletProperties.FILTER_FORWARD_ON_404, true);
    }

}
```

## 7. HTTP客户端

Spring Boot提供了许多可与HTTP客户端一起使用的启动器。本节回答与使用它们有关的问题。

### 7.1。配置RestTemplate使用代理

如[spring-boot-features.html中所述](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-resttemplate-customization)，您可以使用`RestTemplateCustomizer`with `RestTemplateBuilder`来构建自定义的`RestTemplate`。这是创建`RestTemplate`配置为使用代理的推荐方法。

代理配置的确切详细信息取决于所使用的基础客户端请求工厂。以下示例使用进行配置`HttpComponentsClientRequestFactory`，该`HttpClient`代理使用除`192.168.0.5`以下主机之外的所有主机的代理：

```java
static class ProxyCustomizer implements RestTemplateCustomizer {

    @Override
    public void customize(RestTemplate restTemplate) {
        HttpHost proxy = new HttpHost("proxy.example.com");
        HttpClient httpClient = HttpClientBuilder.create().setRoutePlanner(new DefaultProxyRoutePlanner(proxy) {

            @Override
            public HttpHost determineProxy(HttpHost target, HttpRequest request, HttpContext context)
                    throws HttpException {
                if (target.getHostName().equals("192.168.0.5")) {
                    return null;
                }
                return super.determineProxy(target, request, context);
            }

        }).build();
        restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory(httpClient));
    }

}
```

### 7.2。配置基于Reactor Netty的WebClient使用的TcpClient

当Reactor Netty在类路径上时，将`WebClient`自动配置基于Reactor Netty的文件。要自定义客户端对网络连接的处理，请提供一个`ClientHttpConnector`bean。以下示例配置60秒的连接超时并添加`ReadTimeoutHandler`：

```java
@Bean
ClientHttpConnector clientHttpConnector(ReactorResourceFactory resourceFactory) {
    TcpClient tcpClient = TcpClient.create(resourceFactory.getConnectionProvider())
            .runOn(resourceFactory.getLoopResources()).option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 60000)
            .doOnConnected((connection) -> connection.addHandlerLast(new ReadTimeoutHandler(60)));
    return new ReactorClientHttpConnector(HttpClient.from(tcpClient));
}
```

|      | 请注意对`ReactorResourceFactory`连接提供程序和事件循环资源的使用。这确保了用于服务器接收请求和客户端发出请求的资源的有效共享。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 8.记录

除了通常由Spring Framework的`spring-jcl`模块提供的Commons Logging API之外，Spring Boot没有强制性的日志记录依赖项。要使用[Logback](https://logback.qos.ch/)，您需要将其包括`spring-jcl`在类路径中。最简单的方法是通过启动器，这都取决于`spring-boot-starter-logging`。对于Web应用程序，您仅需要`spring-boot-starter-web`，因为它暂时依赖于日志记录启动器。如果使用Maven，则以下依赖项会为您添加日志记录：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot有一个`LoggingSystem`抽象，它尝试根据类路径的内容配置日志记录。如果可以使用Logback，则它是首选。

如果您需要对日志记录进行的唯一更改是设置各种记录器的级别，则可以`application.properties`使用“ logging.level”前缀来进行设置，如以下示例所示：

```properties
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate=ERROR
```

您还可以使用“ logging.file.name”来设置要写入日志的文件的位置（除了控制台）。

要配置日志记录系统的更细粒度的设置，您需要使用所支持的本机配置格式`LoggingSystem`。默认情况下，Spring Boot从系统的默认位置（例如`classpath:logback.xml`Logback）中拾取本地配置，但是您可以使用`logging.config`属性设置配置文件的位置。

### 8.1。配置登录以进行日志记录

如果您需要将自定义设置应用于Logback之外的自定义设置，则`application.properties`需要添加标准的Logback配置文件。您可以将`logback.xml`文件添加到类路径的根目录中，以供登录后查找。`logback-spring.xml`如果要使用[Spring Boot Logback扩展，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-logback-extensions)也可以使用。

|      | Logback文档有一个[专用部分，](https://logback.qos.ch/manual/configuration.html)其中详细[介绍了配置](https://logback.qos.ch/manual/configuration.html)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

Spring Boot提供了许多`included`来自您自己的配置的logback 配置。这些包括旨在允许重新应用某些常见的Spring Boot约定。

以下文件提供了`org/springframework/boot/logging/logback/`：

- `defaults.xml` -提供转换规则，模式属性和通用记录器配置。
- `console-appender.xml`- `ConsoleAppender`使用添加`CONSOLE_LOG_PATTERN`。
- `file-appender.xml`- `RollingFileAppender`使用`FILE_LOG_PATTERN`和并`ROLLING_FILE_NAME_PATTERN`通过适当的设置添加。

此外，`base.xml`还提供了一个旧文件以与早期版本的Spring Boot兼容。

典型的自定义`logback.xml`文件如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
    <logger name="org.springframework.web" level="DEBUG"/>
</configuration>
```

您的logback配置文件也可以利用`LoggingSystem`为您创建的System属性：

- `${PID}`：当前进程ID。
- `${LOG_FILE}`：是否`logging.file.name`在Boot的外部配置中设置。
- `${LOG_PATH}`：是否`logging.file.path`在Boot的外部配置中设置了（表示用于存放日志文件的目录）。
- `${LOG_EXCEPTION_CONVERSION_WORD}`：是否`logging.exception-conversion-word`在Boot的外部配置中设置。
- `${ROLLING_FILE_NAME_PATTERN}`：是否`logging.pattern.rolling-file-name`在Boot的外部配置中设置。

通过使用自定义的Logback转换器，Spring Boot还可以在控制台上提供一些不错的ANSI颜色终端输出（但不在日志文件中）。看到`CONSOLE_LOG_PATTERN`在`defaults.xml`一个示例配置。

如果Groovy在类路径上，那么您也应该能够配置Logback `logback.groovy`。如果存在，则优先考虑此设置。

|      | Groovy配置不支持Spring扩展。`logback-spring.groovy`不会检测到任何文件。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.1.1。配置仅文件输出的Logback

如果要禁用控制台日志记录并且仅将输出写入文件，则需要一个`logback-spring.xml`导入`file-appender.xml`但不导入的自定义`console-appender.xml`，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}/}spring.log}"/>
    <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
    <root level="INFO">
        <appender-ref ref="FILE" />
    </root>
</configuration>
```

您还需要将添加`logging.file.name`到中`application.properties`，如以下示例所示：

```properties
logging.file.name=myapplication.log
```

### 8.2。配置Log4j进行日志记录

如果Spring Boot 在类路径上，则它支持[Log4j 2](https://logging.apache.org/log4j/2.x/)进行日志记录配置。如果使用入门程序来组装依赖项，则必须排除Logback，然后改为包括log4j 2。如果您不使用启动器，则`spring-jcl`除了Log4j 2之外，还需要提供（至少）。

最简单的方法可能是通过启动程序，即使它需要对排除对象进行微调。以下示例显示了如何在Maven中设置启动器：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

以下示例显示了在Gradle中设置启动器的一种方法：

```groovy
dependencies {
    compile 'org.springframework.boot:spring-boot-starter-web'
    compile 'org.springframework.boot:spring-boot-starter-log4j2'
}

configurations {
    all {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-logging'
    }
}
```

|      | Log4j入门人员将依赖关系集中在一起，以满足常见的日志记录要求（例如使用Tomcat，`java.util.logging`但使用Log4j 2配置输出）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 为了确保将使用进行的调试日志记录`java.util.logging`路由到Log4j 2中，请将系统属性设置为，以配置其[JDK日志记录适配器](https://logging.apache.org/log4j/2.0/log4j-jul/index.html)。 `java.util.logging.manager``org.apache.logging.log4j.jul.LogManager` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

#### 8.2.1。使用YAML或JSON配置Log4j 2

除了默认的XML配置格式外，Log4j 2还支持YAML和JSON配置文件。要将Log4j 2配置为使用备用配置文件格式，请将适当的依赖项添加到类路径中，并命名您的配置文件以匹配您选择的文件格式，如以下示例所示：

| 格式     | 依存关系                                                     | 档案名称                     |
| :------- | :----------------------------------------------------------- | :--------------------------- |
| YAML     | `com.fasterxml.jackson.core:jackson-databind` + `com.fasterxml.jackson.dataformat:jackson-dataformat-yaml` | `log4j2.yaml` + `log4j2.yml` |
| JSON格式 | `com.fasterxml.jackson.core:jackson-databind`                | `log4j2.json` + `log4j2.jsn` |

## 9.资料存取

Spring Boot包含许多用于处理数据源的启动器。本节回答与这样做有关的问题。

### 9.1。配置自定义数据源

要配置自己的`DataSource`，请在配置中定义`@Bean`该类型的。Spring Boot可以`DataSource`在任何需要重用的地方重复使用，包括数据库初始化。如果需要外部化某些设置，则可以将其绑定`DataSource`到环境（请参阅“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-3rd-party-configuration) ”）。

以下示例显示了如何在Bean中定义数据源：

```java
@Bean
@ConfigurationProperties(prefix="app.datasource")
public DataSource dataSource() {
    return new FancyDataSource();
}
```

以下示例显示如何通过设置属性来定义数据源：

```properties
app.datasource.url=jdbc:h2:mem:mydb
app.datasource.username=sa
app.datasource.pool-size=30
```

假设您`FancyDataSource`具有URL，用户名和池大小的常规JavaBean属性，则在将设置`DataSource`用于其他组件之前，将自动绑定这些设置。常规的[数据库初始化](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-initialize-a-database-using-spring-jdbc)也会发生（因此的相关子集`spring.datasource.*`仍可以与您的自定义配置一起使用）。

Spring Boot还提供了一个名为的实用程序生成器类，`DataSourceBuilder`可用于创建标准数据源之一（如果它在类路径上）。构建器可以根据类路径上可用的内容来检测要使用的一个。它还基于JDBC URL自动检测驱动程序。

以下示例显示如何使用来创建数据源`DataSourceBuilder`：

```java
@Bean
@ConfigurationProperties("app.datasource")
public DataSource dataSource() {
    return DataSourceBuilder.create().build();
}
```

要使用that运行应用程序`DataSource`，您需要的只是连接信息。还可以提供特定于池的设置。有关更多详细信息，请检查将在运行时使用的实现。

以下示例显示如何通过设置属性来定义JDBC数据源：

```properties
app.datasource.url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.pool-size=30
```

但是，有一个陷阱。由于未公开连接池的实际类型，因此在自定义元数据中不会生成任何键，`DataSource`并且在IDE中也无法完成操作（因为该`DataSource`接口未公开任何属性）。另外，如果您碰巧在类路径上有Hikari，则此基本设置将不起作用，因为Hikari没有`url`属性（但确实具有`jdbcUrl`属性）。在这种情况下，您必须按照以下方式重写配置：

```properties
app.datasource.jdbc-url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.maximum-pool-size=30
```

您可以通过强制连接池使用并返回专用的实现而不是来解决此问题`DataSource`。您不能在运行时更改实现，但是选项列表将是明确的。

下面的示例演示如何创建一个`HikariDataSource`具有`DataSourceBuilder`：

```java
@Bean
@ConfigurationProperties("app.datasource")
public HikariDataSource dataSource() {
    return DataSourceBuilder.create().type(HikariDataSource.class).build();
}
```

您甚至可以利用`DataSourceProperties`为您服务的功能（即，如果没有提供URL，则为默认的嵌入式数据库提供具有合理的用户名和密码）来走得更远。您可以轻松地`DataSourceBuilder`从任何`DataSourceProperties`对象的状态初始化a ，因此您还可以注入Spring Boot自动创建的DataSource。然而，这将配置分为两个命名空间：`url`，`username`，`password`，`type`，和`driver`上`spring.datasource`和您的自定义命名空间中的其余部分（`app.datasource`）。为避免这种情况，可以`DataSourceProperties`在自定义名称空间上重新定义一个自定义，如以下示例所示：

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource")
public DataSourceProperties dataSourceProperties() {
    return new DataSourceProperties();
}

@Bean
@ConfigurationProperties("app.datasource.configuration")
public HikariDataSource dataSource(DataSourceProperties properties) {
    return properties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
}
```

默认情况下，该设置使您与Spring Boot所做的*同步*，不同的是，已选择了专用连接池（以代码形式），并且其设置在`app.datasource.configuration`子名称空间中公开。因为`DataSourceProperties`正在为您处理`url`/ `jdbcUrl`转换，所以可以按以下方式配置它：

```properties
app.datasource.url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.configuration.maximum-pool-size=30
```

|      | Spring Boot会将Hikari特定的设置公开给`spring.datasource.hikari`。本示例使用更通用的`configuration`子名称空间，因为该示例不支持多个数据源实现。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 由于您的自定义配置选择与Hikari一起使用，`app.datasource.type`因此无效。实际上，构建器会使用您可以在其中设置的任何值进行初始化，然后被调用覆盖`.type()`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

有关更多详细信息，请参见“ Spring Boot功能”部分中的“ [spring-boot-features.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-configure-datasource) ”和[`DataSourceAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration.java)该类。

### 9.2。配置两个数据源

如果需要配置多个数据源，则可以应用上一节中介绍的相同技巧。但是，您必须将其中一个`DataSource`实例标记为`@Primary`，因为将来各种自动配置都希望能够按类型获取一个。

如果您创建自己的`DataSource`，则自动配置将退出。在以下示例中，我们提供与自动配置在主数据源上提供的功能*完全相同的*功能集：

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource.first")
public DataSourceProperties firstDataSourceProperties() {
    return new DataSourceProperties();
}

@Bean
@Primary
@ConfigurationProperties("app.datasource.first.configuration")
public HikariDataSource firstDataSource() {
    return firstDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
}

@Bean
@ConfigurationProperties("app.datasource.second")
public BasicDataSource secondDataSource() {
    return DataSourceBuilder.create().type(BasicDataSource.class).build();
}
```

|      | `firstDataSourceProperties`必须标记为，`@Primary`以便数据库初始化程序功能使用您的副本（如果使用初始化程序）。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

这两个数据源也都必须进行高级定制。例如，您可以按以下方式配置它们：

```properties
app.datasource.first.url=jdbc:mysql://localhost/first
app.datasource.first.username=dbuser
app.datasource.first.password=dbpass
app.datasource.first.configuration.maximum-pool-size=30

app.datasource.second.url=jdbc:mysql://localhost/second
app.datasource.second.username=dbuser
app.datasource.second.password=dbpass
app.datasource.second.max-total=30
```

您也可以将相同的概念应用于辅助服务器`DataSource`，如以下示例所示：

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource.first")
public DataSourceProperties firstDataSourceProperties() {
    return new DataSourceProperties();
}

@Bean
@Primary
@ConfigurationProperties("app.datasource.first.configuration")
public HikariDataSource firstDataSource() {
    return firstDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
}

@Bean
@ConfigurationProperties("app.datasource.second")
public DataSourceProperties secondDataSourceProperties() {
    return new DataSourceProperties();
}

@Bean
@ConfigurationProperties("app.datasource.second.configuration")
public BasicDataSource secondDataSource() {
    return secondDataSourceProperties().initializeDataSourceBuilder().type(BasicDataSource.class).build();
}
```

前面的示例在自定义名称空间上配置两个数据源，其逻辑与Spring Boot在自动配置中使用的逻辑相同。请注意，每个`configuration`子名称空间都根据所选实现提供高级设置。

### 9.3。使用Spring数据仓库

Spring Data可以创建`@Repository`各种类型的接口的实现。只要Spring Boot `@Repositories`包含在类的同一包（或子包）中，Spring Boot就会为您处理所有这些操作`@EnableAutoConfiguration`。

对于许多应用程序，您所需要做的就是在类路径上放置正确的Spring Data依赖项。有一个`spring-boot-starter-data-jpa`JPA，`spring-boot-starter-data-mongodb`Mongodb等。开始之前，创建一些存储库接口来处理您的`@Entity`对象。

Spring Boot会`@Repository`根据`@EnableAutoConfiguration`发现的内容尝试猜测您定义的位置。要获得更多控制权，请使用`@EnableJpaRepositories`注释（来自Spring Data JPA）。

有关Spring Data的更多信息，请参见[Spring Data项目页面](https://spring.io/projects/spring-data)。

### 9.4。将@Entity定义与Spring配置分开

Spring Boot会`@Entity`根据`@EnableAutoConfiguration`发现的内容尝试猜测您定义的位置。要获得更多控制权，可以使用`@EntityScan`注释，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration
@EntityScan(basePackageClasses=City.class)
public class Application {

    //...

}
```

### 9.5。配置JPA属性

Spring Data JPA已经提供了一些独立于供应商的配置选项（例如，用于SQL日志记录的那些），并且Spring Boot将这些选项以及其他一些Hibernate作为外部配置属性公开。其中一些会根据上下文自动检测到，因此您不必进行设置。

这`spring.jpa.hibernate.ddl-auto`是一种特殊情况，因为根据运行时条件，它具有不同的默认值。如果使用嵌入式数据库并且没有模式管理器（例如Liquibase或Flyway）正在处理`DataSource`，则默认为`create-drop`。在所有其他情况下，它默认为`none`。

JPA提供程序检测到要使用的方言。如果您想自己设置方言，请设置`spring.jpa.database-platform`属性。

下例显示了最常用的设置选项：

```
spring.jpa.hibernate.naming.physical-strategy = com.example.MyPhysicalNamingStrategy
spring.jpa.show-sql = true
```

此外，创建`spring.jpa.properties.*`本地时，所有属性都作为常规JPA属性（前缀被去除）传递`EntityManagerFactory`。

|      | 您需要确保在下定义的名称`spring.jpa.properties.*`与您的JPA提供程序期望的名称完全匹配。Spring Boot不会尝试对这些条目进行任何形式的宽松绑定。例如，如果要配置Hibernate的批处理大小，则必须使用`spring.jpa.properties.hibernate.jdbc.batch_size`。如果您使用其他形式，例如`batchSize`或`batch-size`，则Hibernate将不会应用该设置。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 如果您需要对Hibernate属性应用高级自定义，请考虑`HibernatePropertiesCustomizer`在创建之前注册将被调用的bean `EntityManagerFactory`。这优先于自动配置应用的任何内容。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 9.6。配置休眠命名策略

Hibernate使用[两种不同的命名策略](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#naming)将名称从对象模型映射到相应的数据库名称。可以分别通过设置`spring.jpa.hibernate.naming.physical-strategy`和`spring.jpa.hibernate.naming.implicit-strategy`属性来配置物理和隐式策略实现的标准类名。替代地，如果`ImplicitNamingStrategy`或`PhysicalNamingStrategy`豆在应用程序上下文是可用的，Hibernate会被自动配置为使用它们。

默认情况下，Spring Boot使用配置物理命名策略`SpringPhysicalNamingStrategy`。该实现提供了与Hibernate 4相同的表结构：所有点都由下划线替换，骆驼套也由下划线替换。默认情况下，所有表名均以小写形式生成，但是如果您的架构需要它，则可以覆盖该标志。

例如，一个`TelephoneNumber`实体被映射到`telephone_number`表。

如果您更喜欢使用Hibernate 5的默认设置，请设置以下属性：

```
spring.jpa.hibernate.naming.physical-strategy = org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

另外，您可以配置以下bean：

```java
@Bean
public PhysicalNamingStrategy physicalNamingStrategy() {
    return new PhysicalNamingStrategyStandardImpl();
}
```

请参阅[`HibernateJpaAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaAutoConfiguration.java)和[`JpaBaseConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)了解更多详细信息。

### 9.7。配置Hibernate二级缓存

可以为一系列缓存提供程序配置Hibernate [二级](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#caching)缓存。与其将Hibernate配置为再次查找缓存提供程序，不如提供尽可能在上下文中可用的缓存提供程序。

如果您使用的是JCache，这非常简单。首先，确保`org.hibernate:hibernate-jcache`在类路径上可用。然后，添加一个`HibernatePropertiesCustomizer`bean，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
public class HibernateSecondLevelCacheExample {

    @Bean
    public HibernatePropertiesCustomizer hibernateSecondLevelCacheCustomizer(JCacheCacheManager cacheManager) {
        return (properties) -> properties.put(ConfigSettings.CACHE_MANAGER, cacheManager.getCacheManager());
    }

}
```

该定制程序将配置Hibernate以`CacheManager`使其与应用程序所使用的相同。也可以使用单独的`CacheManager`实例。有关详细信息，请参阅[《 Hibernate用户指南》](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#caching-provider-jcache)。

### 9.8。在休眠组件中使用依赖注入

默认情况下，Spring Boot注册使用的`BeanContainer`实现，`BeanFactory`以便转换器和实体侦听器可以使用常规依赖项注入。

您可以通过注册`HibernatePropertiesCustomizer`删除或更改`hibernate.resource.beans.container`属性的来禁用或调整此行为。

### 9.9。使用自定义EntityManagerFactory

要完全控制的配置`EntityManagerFactory`，您需要添加一个`@Bean`名为“ entityManagerFactory”的文件。如果存在这种类型的Bean，Spring Boot自动配置将关闭其实体管理器。

### 9.10。使用两个EntityManager

即使默认设置`EntityManagerFactory`工作正常，您也需要定义一个新的设置。否则，该类型的第二个bean的存在将关闭默认值。为了使操作变得容易，您可以使用`EntityManagerBuilder`Spring Boot提供的便利。另外，您也可以`LocalContainerEntityManagerFactoryBean`直接从Spring ORM进行操作，如以下示例所示：

```java
// add two data sources configured as above

@Bean
public LocalContainerEntityManagerFactoryBean customerEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(customerDataSource())
            .packages(Customer.class)
            .persistenceUnit("customers")
            .build();
}

@Bean
public LocalContainerEntityManagerFactoryBean orderEntityManagerFactory(
        EntityManagerFactoryBuilder builder) {
    return builder
            .dataSource(orderDataSource())
            .packages(Order.class)
            .persistenceUnit("orders")
            .build();
}
```

|      | 当您为`LocalContainerEntityManagerFactoryBean`自己创建一个bean 时，在创建自动配置的过程中应用的所有定制都将`LocalContainerEntityManagerFactoryBean`丢失。例如，在Hibernate的情况下，`spring.jpa.hibernate`前缀下的任何属性都不会自动应用于`LocalContainerEntityManagerFactoryBean`。如果您依靠这些属性来配置诸如命名策略或DDL模式之类的东西，那么在创建`LocalContainerEntityManagerFactoryBean`bean 时将需要显式配置。另一方面，如果您使用自动配置来构建Bean ，则将自动应用`EntityManagerFactoryBuilder`通过所指定的应用于自动配置的属性。 `spring.jpa.properties``EntityManagerFactoryBuilder``LocalContainerEntityManagerFactoryBean` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

上面的配置几乎可以独立工作。为了完成图片，您还需要`TransactionManagers`为两者进行配置`EntityManagers`。如果将其中一个标记为`@Primary`，则可以`JpaTransactionManager`在Spring Boot 中将其默认选中。另一个必须显式地注入到新实例中。另外，您也许可以使用跨两个JTA事务管理器。

如果使用Spring Data，则需要进行相应的配置`@EnableJpaRepositories`，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
@EnableJpaRepositories(basePackageClasses = Customer.class,
        entityManagerFactoryRef = "customerEntityManagerFactory")
public class CustomerConfiguration {
    ...
}

@Configuration(proxyBeanMethods = false)
@EnableJpaRepositories(basePackageClasses = Order.class,
        entityManagerFactoryRef = "orderEntityManagerFactory")
public class OrderConfiguration {
    ...
}
```

### 9.11。使用传统`persistence.xml`文件

`META-INF/persistence.xml`默认情况下，Spring Boot不会搜索或使用。如果您喜欢使用传统的`persistence.xml`，则需要定义自己`@Bean`的类型`LocalEntityManagerFactoryBean`（ID为'entityManagerFactory'）并在其中设置持久性单元名称。

请参阅[`JpaBaseConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/orm/jpa/JpaBaseConfiguration.java)以获取默认设置。

### 9.12。使用Spring Data JPA和Mongo存储库

Spring Data JPA和Spring Data Mongo都可`Repository`以为您自动创建实现。如果它们都存在于类路径中，则可能必须做一些额外的配置以告知Spring Boot要创建哪个存储库。最明确的方法是使用标准Spring数据`@EnableJpaRepositories`和`@EnableMongoRepositories`注释并提供`Repository`接口的位置。

还有一些标志（`spring.data.*.repositories.enabled`和`spring.data.*.repositories.type`），您可以使用它们在外部配置中打开和关闭自动配置的存储库。这样做很有用，例如，如果您要关闭Mongo存储库并仍使用自动配置的`MongoTemplate`。

对于其他自动配置的Spring Data存储库类型（Elasticsearch，Solr等），存在相同的障碍和相同的功能。要使用它们，请相应地更改注释和标志的名称。

### 9.13。定制Spring Data的Web支持

Spring Data提供了Web支持，从而简化了Web应用程序中Spring Data存储库的使用。Spring Boot在`spring.data.web`名称空间中提供了用于自定义其配置的属性。请注意，如果您使用的是Spring Data REST，则必须`spring.data.rest`改为使用名称空间中的属性。

### 9.14。将Spring数据存储库公开为REST端点

`Repository`如果已为应用程序启用了Spring MVC，Spring Data REST可以为您将实现公开为REST端点。

Spring Boot公开了一组有用的属性（来自`spring.data.rest`名称空间），用于自定义[`RepositoryRestConfiguration`](https://docs.spring.io/spring-data/rest/docs/3.2.6.RELEASE/api/org/springframework/data/rest/core/config/RepositoryRestConfiguration.html)。如果需要提供其他定制，则应使用[`RepositoryRestConfigurer`](https://docs.spring.io/spring-data/rest/docs/3.2.6.RELEASE/api/org/springframework/data/rest/webmvc/config/RepositoryRestConfigurer.html)Bean。

|      | 如果您没有在custom上指定任何顺序`RepositoryRestConfigurer`，它将在一个Spring Boot内部使用之后运行。如果您需要指定订单，请确保该订单大于0。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 9.15。配置JPA使用的组件

如果要配置JPA使用的组件，则需要确保在JPA之前初始化该组件。当组件被自动配置后，Spring Boot会为您处理。例如，当自动配置Flyway时，Hibernate被配置为依赖Flyway，这样Flyway就有机会在Hibernate尝试使用数据库之前初始化数据库。

如果您自己配置组件，则可以使用`EntityManagerFactoryDependsOnPostProcessor`子类作为建立必要依赖关系的便捷方法。例如，如果将Hibernate Search与Elasticsearch一起用作其索引管理器，则`EntityManagerFactory`必须将任何bean配置为依赖于该`elasticsearchClient`bean，如以下示例所示：

```java
/**
 * {@link EntityManagerFactoryDependsOnPostProcessor} that ensures that
 * {@link EntityManagerFactory} beans depend on the {@code elasticsearchClient} bean.
 */
@Component
static class ElasticsearchEntityManagerFactoryDependsOnPostProcessor
        extends EntityManagerFactoryDependsOnPostProcessor {

    ElasticsearchEntityManagerFactoryDependsOnPostProcessor() {
        super("elasticsearchClient");
    }

}
```

### 9.16。使用两个数据源配置jOOQ

如果需要将jOOQ与多个数据源一起使用，则应该`DSLContext`为每个数据源创建自己的数据源。有关更多详细信息，请参阅[JooqAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jooq/JooqAutoConfiguration.java)。

|      | 特别是，`JooqExceptionTranslator`并且`SpringTransactionProvider`可以重复使用，以提供与单个自动配置相似的功能`DataSource`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

## 10.数据库初始化

可以使用不同的方式初始化SQL数据库，具体取决于堆栈是什么。当然，如果数据库是一个单独的过程，您也可以手动执行。建议使用单一机制生成模式。

### 10.1。使用JPA初始化数据库

JPA具有用于DDL生成的功能，可以将其设置为在启动时针对数据库运行。这是通过两个外部属性控制的：

- `spring.jpa.generate-ddl` （布尔值）打开和关闭该功能，并且与供应商无关。
- `spring.jpa.hibernate.ddl-auto`（枚举）是一种Hibernate功能，可以更精细地控制行为。此功能将在本指南的后面部分详细介绍。

### 10.2。使用Hibernate初始化数据库

您可以设置`spring.jpa.hibernate.ddl-auto`明确和标准的Hibernate属性值`none`，`validate`，`update`，`create`，和`create-drop`。Spring Boot根据是否认为您的数据库已嵌入为您选择默认值。默认为`create-drop`是否未检测到任何模式管理器或`none`在所有其他情况下。通过查看`Connection`类型可以检测到嵌入式数据库。 `hsqldb`，`h2`和`derby`是嵌入式的，其他不是。从内存数据库切换到“真实”数据库时，请不要对新平台中表和数据的存在做出假设。您必须`ddl-auto`显式设置或使用其他机制之一来初始化数据库。

|      | 您可以通过启用`org.hibernate.SQL`记录器来输出架构创建。如果启用[调试模式，](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-logging-console-output)此操作将自动为您完成。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

另外，`import.sql`如果Hibernate从头开始创建模式（即，如果`ddl-auto`属性设置为`create`或`create-drop`），则在启动时将执行在类路径的根目录中命名的文件。如果您小心的话，这对于演示和测试很有用，但是可能不希望出现在生产环境的类路径中。这是一个Hibernate功能（与Spring无关）。

### 10.3。初始化数据库

Spring Boot可以自动创建您的架构（DDL脚本）`DataSource`并对其进行初始化（DML脚本）。它分别从标准根类路径位置`schema.sql`和加载SQL `data.sql`。另外，Spring Boot处理`schema-${platform}.sql`和`data-${platform}.sql`文件（如果存在），其中`platform`的值为`spring.datasource.platform`。这使您可以在必要时切换到特定于数据库的脚本。例如，您可以选择将其设置为数据库中的供应商名称（`hsqldb`，`h2`，`oracle`，`mysql`，`postgresql`，等）。

|      | Spring Boot自动创建一个嵌入式模式`DataSource`。可以使用该`spring.datasource.initialization-mode`属性来自定义此行为。例如，如果您要始终初始化，则`DataSource`无论其类型如何：`spring.datasource.initialization-mode =总是` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

默认情况下，Spring Boot启用Spring JDBC初始化程序的快速失败功能。这意味着，如果脚本导致异常，则应用程序将无法启动。您可以通过设置来调整行为`spring.datasource.continue-on-error`。

|      | 在基于JPA的应用程序中，您可以选择让Hibernate创建架构或使用`schema.sql`，但您不能两者都做。`spring.jpa.hibernate.ddl-auto`如果使用，请确保禁用`schema.sql`。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 10.4。初始化一个Spring Batch数据库

如果您使用Spring Batch，则它随大多数流行的数据库平台一起预包装了SQL初始化脚本。Spring Boot可以检测数据库类型并在启动时执行这些脚本。如果您使用嵌入式数据库，则默认情况下会发生这种情况。您还可以为任何数据库类型启用它，如以下示例所示：

```
spring.batch.initialize-schema =总是
```

您还可以通过设置明确关闭初始化`spring.batch.initialize-schema=never`。

### 10.5。使用高级数据库迁移工具

春天引导支持两种更高级别的迁移工具：[迁飞](https://flywaydb.org/)和[Liquibase](https://www.liquibase.org/)。

#### 10.5.1。在启动时执行Flyway数据库迁移

要在启动时自动运行Flyway数据库迁移，请将添加`org.flywaydb:flyway-core`到您的类路径中。

通常，迁移是采用以下形式的脚本`V__.sql`（带有``下划线分隔的版本，例如“ 1”或“ 2_1”）。默认情况下，它们位于名为的文件夹中`classpath:db/migration`，但是您可以通过设置修改该位置`spring.flyway.locations`。这是一个或多个`classpath:`或多个`filesystem:`位置的逗号分隔列表。例如，以下配置将在默认类路径位置和`/opt/migration`目录中搜索脚本：

```properties
spring.flyway.locations=classpath:db/migration,filesystem:/opt/migration
```

您也可以添加特殊的`{vendor}`占位符以使用特定于供应商的脚本。假设以下内容：

```properties
spring.flyway.locations=classpath:db/migration/{vendor}
```

`db/migration`前面的配置不是使用，而是根据数据库的类型（例如，`db/migration/mysql`对于MySQL）来设置要使用的文件夹。支持的数据库列表在中提供[`DatabaseDriver`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jdbc/DatabaseDriver.java)。

迁移也可以用Java编写。Flyway将使用任何实现的bean自动配置`JavaMigration`。

[`FlywayProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayProperties.java)提供Flyway的大多数设置以及少量的其他属性，这些属性可用于禁用迁移或关闭位置检查。如果需要对配置进行更多控制，请考虑注册`FlywayConfigurationCustomizer`Bean。

Spring Boot调用`Flyway.migrate()`以执行数据库迁移。如果您想要更多控制，请提供`@Bean`实现[`FlywayMigrationStrategy`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/flyway/FlywayMigrationStrategy.java)。

Flyway支持SQL和Java [回调](https://flywaydb.org/documentation/callbacks.html)。要使用基于SQL的回调，请将回调脚本放在`classpath:db/migration`文件夹中。要使用基于Java的回调，请创建一个或多个实现的bean `Callback`。任何此类bean都会自动向进行注册`Flyway`。可以通过使用`@Order`或实现来订购它们`Ordered`。`FlywayCallback`也可以检测到实现了不推荐使用的接口的Bean ，但是不能与`Callback`Bean 一起使用。

默认情况下，Flyway自动在您的上下文中自动连接（`@Primary`）`DataSource`并将其用于迁移。如果您想使用其他名称`DataSource`，则可以创建一个并将其标记`@Bean`为`@FlywayDataSource`。如果这样做并想要两个数据源，请记住创建另一个数据源并将其标记为`@Primary`。另外，您可以`DataSource`通过设置`spring.flyway.[url,user,password]`外部属性来使用Flyway的本机。设置`spring.flyway.url`或`spring.flyway.user`足以使Flyway使用其自己的`DataSource`。如果未设置这三个属性中的任何一个，`spring.datasource`则将使用其等效属性的值。

您还可以使用Flyway为特定情况提供数据。例如，您可以放入特定于测试的迁移，`src/test/resources`并且仅在应用程序开始进行测试时才运行它们。另外，您可以使用特定于配置文件的配置进行自定义，`spring.flyway.locations`以便仅在特定配置文件处于活动状态时才运行某些迁移。例如，在中`application-dev.properties`，您可以指定以下设置：

```properties
spring.flyway.locations=classpath:/db/migration,classpath:/dev/db/migration
```

使用该设置，`dev/db/migration`仅在`dev`配置文件处于活动状态时才运行迁移。

#### 10.5.2。在启动时执行Liquibase数据库迁移

要在启动时自动运行Liquibase数据库迁移，请将添加`org.liquibase:liquibase-core`到您的类路径中。

默认情况下，从中读取主更改日志`db/changelog/db.changelog-master.yaml`，但是您可以通过设置来更改位置`spring.liquibase.change-log`。除了YAML，Liquibase还支持JSON，XML和SQL更改日志格式。

默认情况下，Liquibase自动在您的上下文中自动连接（`@Primary`）`DataSource`并将其用于迁移。如果需要使用其他名称`DataSource`，则可以创建一个并将其标记`@Bean`为`@LiquibaseDataSource`。如果这样做，并且想要两个数据源，请记住创建另一个数据源并将其标记为`@Primary`。另外，您可以`DataSource`通过设置`spring.liquibase.[url,user,password]`外部属性来使用Liquibase的本机。设置`spring.liquibase.url`或`spring.liquibase.user`足以使Liquibase使用其自己的`DataSource`。如果未设置这三个属性中的任何一个，`spring.datasource`则将使用其等效属性的值。

请参阅[`LiquibaseProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/liquibase/LiquibaseProperties.java)以获取有关可用设置的详细信息，例如上下文，默认架构和其他。

## 11.消息传递

Spring Boot提供了许多入门，包括消息传递。本部分回答了将消息与Spring Boot一起使用所引起的问题。

### 11.1。禁用事务JMS会话

如果您的JMS代理不支持事务处理会话，则必须完全禁用事务支持。如果您创建自己的`JmsListenerContainerFactory`，则无所事事，因为默认情况下无法进行交易。如果要使用`DefaultJmsListenerContainerFactoryConfigurer`来重用Spring Boot的默认值，则可以禁用事务性会话，如下所示：

```java
@Bean
public DefaultJmsListenerContainerFactory jmsListenerContainerFactory(
        ConnectionFactory connectionFactory,
        DefaultJmsListenerContainerFactoryConfigurer configurer) {
    DefaultJmsListenerContainerFactory listenerFactory =
            new DefaultJmsListenerContainerFactory();
    configurer.configure(listenerFactory, connectionFactory);
    listenerFactory.setTransactionManager(null);
    listenerFactory.setSessionTransacted(false);
    return listenerFactory;
}
```

前面的示例将覆盖默认工厂，并且应将其应用于应用程序定义的任何其他工厂（如果有）。

## 12.批处理应用

当人们从Spring Boot应用程序中使用Spring Batch时，经常会出现许多问题。本节解决这些问题。

### 12.1。指定批处理数据源

默认情况下，批处理应用程序需要一个`DataSource`来存储作业详细信息。Spring Batch `DataSource`默认情况下需要一个。要使用它而`DataSource`不是应用程序的主程序`DataSource`，请声明一个`DataSource`bean，并用注释其`@Bean`方法`@BatchDataSource`。如果这样做并需要两个数据源，请记住标记另一个`@Primary`。要获得更大的控制权，请实施`BatchConfigurer`。有关更多详细信息，请参见[Javadoc`@EnableBatchProcessing`](https://docs.spring.io/spring-batch/docs/4.2.1.RELEASE/api/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html)。

有关Spring Batch的更多信息，请参见[Spring Batch项目页面](https://spring.io/projects/spring-batch)。

### 12.2。在启动时运行Spring Batch作业

通过添加`@EnableBatchProcessing`到您的`@Configuration`类之一来启用Spring Batch自动配置。

默认情况下，它在启动时在应用程序上下文中执行**所有操作** `Jobs`（[`JobLauncherCommandLineRunner`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/JobLauncherCommandLineRunner.java)有关详细信息，请参见）。您可以通过指定`spring.batch.job.names`（以逗号分隔的作业名称模式列表）缩小到一个或多个特定作业。

有关更多详细信息，请参见[BatchAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java)和[@EnableBatchProcessing](https://docs.spring.io/spring-batch/docs/4.2.1.RELEASE/api/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html)。

### 12.3。从命令行运行

从命令行通过Spring Boot运行Spring Batch与从命令行单独运行Spring Batch有所不同。最重要的区别是，它们对命令行参数的解析方式不同，这可能会引起麻烦，如下一节所述。

#### 12.3.1。传递命令行参数

Spring Boot将任何`--`以开头的命令行参数转换为要添加到的属性`Environment`，请参见[访问命令行属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-external-config-command-line-args)。不应将其用于将参数传递给批处理作业。要在命令行上指定批处理参数，请使用常规格式（即，不带`--`），如以下示例所示：

```
$ java -jar myapp.jar someParameter = someValue anotherParameter = anotherValue
```

如果`Environment`在命令行上指定的属性，则该属性也会影响作业使用的参数。考虑以下命令：

```
$ java -jar myapp.jar --server.port = 7070 someParameter = someValue
```

这为批处理作业提供了两个参数：`someParameter=someValue`和`-server.port=7070`。请注意，第二个参数缺少一个连字符，因为Spring Batch将删除`-`属性的第一个字符（如果存在）。

### 12.4。存储作业库

Spring Batch需要用于`Job`存储库的数据存储。如果使用Spring Boot，则必须使用实际的数据库。请注意，它可以是内存数据库，请参阅“ [配置作业存储库”](https://docs.spring.io/spring-batch/docs/4.2.1.RELEASE/reference/html/job.html#configuringJobRepository)。

## 13.执行器

Spring Boot包括Spring Boot执行器。本部分回答了经常因使用而引起的问题。

### 13.1。更改执行器端点的HTTP端口或地址

在独立应用程序中，Actuator HTTP端口默认与主HTTP端口相同。要使应用程序在其他端口上侦听，请设置外部属性：`management.server.port`。要侦听完全不同的网络地址（例如，当您有一个用于管理的内部网络而有一个用于用户应用程序的外部网络）时，还可以设置`management.server.address`为服务器可以绑定到的有效IP地址。

有关更多详细信息，请参见[`ManagementServerProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/web/server/ManagementServerProperties.java)源代码和“ [生产](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready-customizing-management-server-port)就绪功能”部分中的“ production-ready-features.html”。

### 13.2。自定义“ whitelabel”错误页面

如果遇到服务器错误，Spring Boot将安装一个“ whitelabel”错误页面，您会在浏览器客户端中看到该错误页面（使用JSON和其他媒体类型的机器客户端应该看到带有正确错误代码的明智响应）。

|      | 设置`server.error.whitelabel.enabled=false`以关闭默认错误页面。这样做将还原您正在使用的servlet容器的默认值。请注意，Spring Boot仍然尝试解决错误视图，因此您应该添加自己的错误页面，而不是完全禁用它。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

用自己的方法覆盖错误页面取决于您使用的模板技术。例如，如果您使用Thymeleaf，则可以添加`error.html`模板。如果您使用FreeMarker，则可以添加`error.ftlh`模板。通常，您需要`View`使用名称解析的`error`或`@Controller`处理`/error`路径的。除非你更换了一些默认配置，你应该找一个`BeanNameViewResolver`你`ApplicationContext`，所以`@Bean`命名`error`是这样做的一个简单的方法。请参阅[`ErrorMvcAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.java)以获取更多选项。

有关如何在servlet容器中注册处理程序的详细信息，另请参见“ [错误处理](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features-error-handling) ” 部分。

### 13.3。清理明智的价值观

`env`和`configprops`端点返回的信息可能有些敏感，因此默认情况下会清理与某个模式匹配的键（即，它们的值将替换为`******`）。

可以分别使用`management.endpoint.env.keys-to-sanitize`和来定制要使用的模式`management.endpoint.configprops.keys-to-sanitize`。

Spring Boot对此类密钥使用合理的默认值：任何以单词“ password”，“ secret”，“ key”，“ token”，“ vcap_services”，“ sun.java.command”，“ uri”，“ uris”结尾的密钥，“地址”或“地址”被清除。此外，将包含单词`credentials`作为键的一部分的任何键都进行清理（配置为正则表达式，即`*credentials.*`）。

如果要清除的任何键都是URI格式（即`://:@:/`），则仅清除密码部分。

## 14.安全性

本部分解决有关使用Spring Boot时的安全性的问题，包括因将Spring Security与Spring Boot一起使用而引起的问题。

有关Spring Security的更多信息，请参见[Spring Security项目页面](https://spring.io/projects/spring-security)。

### 14.1。关闭Spring Boot安全性配置

如果您在应用程序中`@Configuration`使用定义A `WebSecurityConfigurerAdapter`，它将关闭Spring Boot中的默认Webapp安全设置。

### 14.2。更改UserDetailsService并添加用户帐户

如果你提供了一个`@Bean`类型的`AuthenticationManager`，`AuthenticationProvider`或者`UserDetailsService`，默认`@Bean`为`InMemoryUserDetailsManager`不创建。这意味着您拥有完整的Spring Security功能集（例如[各种身份验证选项](https://docs.spring.io/spring-security/site/docs/5.2.2.RELEASE/reference/htmlsingle/#jc-authentication)）。

添加用户帐户的最简单方法是提供您自己的`UserDetailsService`bean。

### 14.3。在代理服务器上运行时启用HTTPS

对于所有应用程序而言，确保所有主要端点仅可通过HTTPS进行访问都是一项重要的工作。如果您将Tomcat用作servlet容器，则Spring Boot `RemoteIpValve`如果检测到某些环境设置，则会自动添加Tomcat自己的Tomcat ，并且您应该能够依靠Tomcat `HttpServletRequest`报告其是否安全（甚至在处理代理服务器的下游）。真正的SSL终止）。标准行为由某些请求标头（`x-forwarded-for`和`x-forwarded-proto`）的存在或不存在决定，它们的名称是常规名称，因此它应适用于大多数前端代理。您可以通过向中添加一些条目来打开阀门`application.properties`，如以下示例所示：

```properties
server.tomcat.remote-ip-header=x-forwarded-for
server.tomcat.protocol-header=x-forwarded-proto
```

（这些属性中的任何一个都会在阀上切换。或者，您可以`RemoteIpValve`通过添加`TomcatServletWebServerFactory`bean 来添加。）

要将Spring Security配置为要求所有（或某些）请求使用安全通道，请考虑添加自己的`WebSecurityConfigurerAdapter`添加以下`HttpSecurity`配置：

```java
@Configuration(proxyBeanMethods = false)
public class SslWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // Customize the application security
        http.requiresChannel().anyRequest().requiresSecure();
    }

}
```

## 15.热插拔

Spring Boot支持热插拔。本部分回答有关其工作方式的问题。

### 15.1。重新加载静态内容

有几种热重装选项。推荐的方法是使用[`spring-boot-devtools`](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools)，因为它提供了其他开发时功能，例如对快速应用程序重新启动和LiveReload的支持，以及合理的开发时配置（例如模板缓存）。Devtools通过监视类路径的更改来工作。这意味着必须“构建”静态资源更改才能使更改生效。默认情况下，当您保存更改时，这会自动在Eclipse中发生。在IntelliJ IDEA中，“生成项目”命令将触发必要的构建。由于[默认的重新启动排除项](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-restart-exclude)，对静态资源的更改不会触发应用程序的重新启动。但是，它们确实会触发实时重新加载。

另外，在IDE中运行（特别是在调试时运行）是进行开发的好方法（所有现代IDE都允许重新加载静态资源，并且通常还允许热交换Java类更改）。

最后，可以配置[Maven和Gradle插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins)（请参阅`addResources`属性）以支持从命令行运行，并直接从源代码重新加载静态文件。如果要使用高级工具编写该代码，则可以将其与外部css / js编译器过程一起使用。

### 15.2。重新加载模板，而无需重新启动容器

Spring Boot支持的大多数模板技术都包含用于禁用缓存的配置选项（在本文档的后面介绍）。如果使用`spring-boot-devtools`模块，则在开发时会[自动](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-property-defaults)为您[配置](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-property-defaults)这些属性。

#### 15.2.1。胸腺模板

如果您使用Thymeleaf，请将设置`spring.thymeleaf.cache`为`false`。有关[`ThymeleafAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)其他Thymeleaf定制选项，请参见。

#### 15.2.2。FreeMarker模板

如果使用FreeMarker，请设置`spring.freemarker.cache`为`false`。有关[`FreeMarkerAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java)其他FreeMarker定制选项，请参见。

#### 15.2.3。Groovy模板

如果使用Groovy模板，请设置`spring.groovy.template.cache`为`false`。请参阅参考资料，[`GroovyTemplateAutoConfiguration`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java)以获取其他Groovy定制选项。

### 15.3。快速应用重启

该`spring-boot-devtools`模块包括对应用程序自动重启的支持。尽管不如[JRebel](https://www.jrebel.com/products/jrebel)这样的技术快，但通常比“冷启动”要快得多。在研究本文档后面讨论的一些更复杂的重载选项之前，您可能应该先尝试一下。

有关更多详细信息，请参见[using-spring-boot.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools)部分。

### 15.4。重新加载Java类而不重新启动容器

许多现代的IDE（Eclipse，IDEA等）都支持字节码的热交换。因此，如果所做的更改不影响类或方法签名，则应干净地重新加载而没有副作用。

## 16.建立

Spring Boot包括Maven和Gradle的构建插件。本部分回答有关这些插件的常见问题。

### 16.1。生成构建信息

Maven插件和Gradle插件都允许生成包含项目的坐标，名称和版本的构建信息。也可以将插件配置为通过配置添加其他属性。当存在这样的文件时，Spring Boot会自动配置一个`BuildProperties`bean。

要使用Maven生成构建信息，请为`build-info`目标添加执行，如以下示例所示：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.2.6.RELEASE</version>
            <executions>
                <execution>
                    <goals>
                        <goal>build-info</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

|      | 有关更多详细信息，请参见[Spring Boot Maven插件文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin/)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

下面的示例对Gradle执行相同的操作：

```groovy
springBoot {
    buildInfo()
}
```

|      | 有关更多详细信息，请参见[Spring Boot Gradle插件文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html//#integrating-with-actuator-build-info)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 16.2。生成Git信息

Maven和Gradle都允许生成一个`git.properties`文件，其中包含有关`git`项目构建时源代码存储库状态的信息。

对于Maven用户，`spring-boot-starter-parent`POM包含一个预配置的插件来生成`git.properties`文件。要使用它，请将以下声明添加到您的POM中：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>pl.project13.maven</groupId>
            <artifactId>git-commit-id-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

Gradle用户可以通过使用[`gradle-git-properties`](https://plugins.gradle.org/plugin/com.gorylenko.gradle-git-properties)插件获得相同的结果，如以下示例所示：

```groovy
plugins {
    id "com.gorylenko.gradle-git-properties" version "2.2.2"
}
```

|      | 提交时间`git.properties`应与以下格式匹配：`yyyy-MM-dd’T’HH:mm:ssZ`。这是上面列出的两个插件的默认格式。使用此格式，可以将时间解析为，`Date`并将其序列化为JSON时的格式由Jackson的日期序列化配置设置控制。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

### 16.3。自定义依赖版本

如果您使用的Maven构建直接或间接继承自`spring-boot-dependencies`（例如`spring-boot-starter-parent`），但是您想覆盖特定的第三方依赖关系，则可以添加适当的``元素。浏览[`spring-boot-dependencies`](https://github.com/spring-projects/spring-boot/tree/v2.2.6.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)POM以获取属性的完整列表。例如，要选择其他`slf4j`版本，可以添加以下属性：

```xml
<properties>
    <slf4j.version>1.7.5<slf4j.version>
</properties>
```

|      | 仅当您的Maven项目从（直接或间接）继承时，此方法才有效`spring-boot-dependencies`。如果您使用添加`spring-boot-dependencies`了自己的`dependencyManagement`部分`import`，则必须自己重新定义工件，而不是覆盖属性。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

|      | 每个Spring Boot版本都是针对这组特定的第三方依赖关系进行设计和测试的。覆盖版本可能会导致兼容性问题。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

要覆盖Gradle中的依赖版本，请参阅Gradle插件文档的[这一部分](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/gradle-plugin/reference/html//#managing-dependencies-customizing)。

### 16.4。使用Maven创建可执行JAR

该`spring-boot-maven-plugin`可用于创建可执行的“肥肉” JAR。如果使用`spring-boot-starter-parent`POM，则可以声明插件，然后将jar重新包装如下：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

如果您不使用父POM，则仍然可以使用该插件。但是，您必须另外添加一个``部分，如下所示：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.2.6.RELEASE</version>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

有关完整用法的详细信息，请参见[插件文档](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin//usage.html)。

### 16.5。使用Spring Boot应用程序作为依赖项

像war文件一样，Spring Boot应用程序也不打算用作依赖项。如果您的应用程序包含要与其他项目共享的类，建议的方法是将该代码移到单独的模块中。然后，您的应用程序和其他项目可以依赖单独的模块。

如果您不能按照上面的建议重新排列代码，则必须配置Spring Boot的Maven和Gradle插件以生成一个单独的工件，该工件适合用作依赖项。可执行档案不能用作依赖项，因为[可执行jar格式将](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-executable-jar-format.html#executable-jar-jar-file-structure)打包应用程序类`BOOT-INF/classes`。这意味着当将可执行jar用作依赖项时，找不到它们。

为了产生两个工件，一个可以用作依赖项，另一个可以执行，必须指定分类器。该分类器应用于可执行档案的名称，保留默认档案作为依赖项。

要`exec`在Maven中配置分类器，可以使用以下配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 16.6。运行可执行jar时提取特定的库

可执行jar中的大多数嵌套库不需要解压即可运行。但是，某些库可能会有问题。例如，JRuby包含其自己的嵌套jar支持，它假定`jruby-complete.jar`总是可以直接以文件的形式直接使用。

要处理任何有问题的库，您可以标记在可执行jar首次运行时应自动解压缩特定的嵌套jar。这些嵌套的jar会写在`java.io.tmpdir`system属性标识的临时目录下。

|      | 应注意确保已配置您的操作系统，以便在应用程序仍在运行时，它不会删除已解压缩到临时目录中的jar。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

例如，为了指示应该使用Maven插件将JRuby标记为解包，您可以添加以下配置：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <requiresUnpack>
                    <dependency>
                        <groupId>org.jruby</groupId>
                        <artifactId>jruby-complete</artifactId>
                    </dependency>
                </requiresUnpack>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 16.7。创建带有排除项的不可执行的JAR

通常，如果您具有一个可执行文件和一个不可执行的jar作为两个单独的构建产品，则可执行版本具有库jar中不需要的其他配置文件。例如，`application.yml`配置文件可能会从不可执行的JAR中排除。

在Maven中，可执行jar必须是主要工件，您可以为库添加一个分类的jar，如下所示：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <artifactId>maven-jar-plugin</artifactId>
            <executions>
                <execution>
                    <id>lib</id>
                    <phase>package</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                    <configuration>
                        <classifier>lib</classifier>
                        <excludes>
                            <exclude>application.yml</exclude>
                        </excludes>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 16.8。远程调试以Maven开头的Spring Boot应用程序

要将远程调试器附加到使用Maven启动的Spring Boot应用程序，可以使用[maven plugin](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin/)的`jvmArguments`属性。

有关更多详细信息，请参[见此示例](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/maven-plugin//examples/run-debug.html)。

### 16.9。在不使用Ant的情况下从Ant构建可执行存档`spring-boot-antlib`

要使用Ant进行构建，您需要获取依赖项，进行编译，然后创建一个jar或war存档。要使其可执行，可以使用该`spring-boot-antlib`模块，也可以按照以下说明进行操作：

1. 如果要构建jar，请将应用程序的类和资源打包到嵌套`BOOT-INF/classes`目录中。如果要发动战争，请`WEB-INF/classes`照常将应用程序的类打包在嵌套目录中。
2. 在`BOOT-INF/lib`jar或`WEB-INF/lib`war 的嵌套目录中添加运行时依赖项。切记**不要**压缩存档中的条目。
3. `provided`在`BOOT-INF/lib`jar或`WEB-INF/lib-provided`war 的嵌套目录中添加（嵌入式容器）依赖项。切记**不要**压缩存档中的条目。
4. 将`spring-boot-loader`类添加到存档的根目录（以便`Main-Class`可用）。
5. 使用适当的启动器（例如`JarLauncher`jar文件）作为`Main-Class`清单中的属性，并指定其所需的其他属性作为清单条目（主要是通过设置`Start-Class`属性）。

以下示例显示了如何使用Ant构建可执行归档文件：

```xml
<target name="build" depends="compile">
    <jar destfile="target/${ant.project.name}-${spring-boot.version}.jar" compress="false">
        <mappedresources>
            <fileset dir="target/classes" />
            <globmapper from="*" to="BOOT-INF/classes/*"/>
        </mappedresources>
        <mappedresources>
            <fileset dir="src/main/resources" erroronmissingdir="false"/>
            <globmapper from="*" to="BOOT-INF/classes/*"/>
        </mappedresources>
        <mappedresources>
            <fileset dir="${lib.dir}/runtime" />
            <globmapper from="*" to="BOOT-INF/lib/*"/>
        </mappedresources>
        <zipfileset src="${lib.dir}/loader/spring-boot-loader-jar-${spring-boot.version}.jar" />
        <manifest>
            <attribute name="Main-Class" value="org.springframework.boot.loader.JarLauncher" />
            <attribute name="Start-Class" value="${start-class}" />
        </manifest>
    </jar>
</target>
```

## 17.传统部署

Spring Boot支持传统部署以及更现代的部署形式。本节回答有关传统部署的常见问题。

### 17.1。创建可部署的战争文件

|      | 由于Spring WebFlux不严格依赖Servlet API，并且默认情况下将应用程序部署在嵌入式Reactor Netty服务器上，因此WebFlux应用程序不支持War部署。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

生成可部署war文件的第一步是提供一个`SpringBootServletInitializer`子类并覆盖其`configure`方法。这样做利用了Spring Framework的Servlet 3.0支持，并允许您在由Servlet容器启动应用程序时对其进行配置。通常，应将应用程序的主类更新为extend `SpringBootServletInitializer`，如以下示例所示：

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

下一步是更新构建配置，以使您的项目生成war文件而不是jar文件。如果您使用Maven和`spring-boot-starter-parent`（为您配置Maven的war插件），那么您要做的就是修改`pom.xml`以将包装更改为war，如下所示：

```xml
<packaging>war</packaging>
```

如果使用Gradle，则需要进行修改`build.gradle`以将war插件应用于项目，如下所示：

```groovy
apply plugin: 'war'
```

该过程的最后一步是确保嵌入式servlet容器不干扰war文件所部署到的servlet容器。为此，您需要将嵌入式Servlet容器依赖项标记为已提供。

如果使用Maven，则以下示例将servlet容器（在本例中为Tomcat）标记为已提供：

```xml
<dependencies>
    <!-- … -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- … -->
</dependencies>
```

如果使用Gradle，则以下示例将servlet容器（在本例中为Tomcat）标记为已提供：

```groovy
dependencies {
    // …
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
    // …
}
```

|      | `providedRuntime`比Gradle的`compileOnly`配置更可取。除其他限制外，`compileOnly`依赖项不在测试类路径上，因此任何基于Web的集成测试都将失败。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

如果您使用[Spring Boot构建工具](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins)，则将提供的嵌入式servlet容器依赖项标记为提供，将生成可执行的war文件，并将提供的依赖项打包在`lib-provided`目录中。这意味着，除了可以部署到Servlet容器之外，还可以通过`java -jar`在命令行上使用来运行应用程序。

### 17.2。将现有应用程序转换为Spring Boot

对于非Web应用程序，将现有的Spring应用程序转换为Spring Boot应用程序应该很容易。为此，请舍弃创建您的代码，`ApplicationContext`并用`SpringApplication`或调用替换它`SpringApplicationBuilder`。Spring MVC Web应用程序通常适合首先创建可部署的war应用程序，然后再将其迁移到可执行的war或jar。请参阅[有关将罐子转换为战争的入门指南](https://spring.io/guides/gs/convert-jar-to-war/)。

要通过扩展`SpringBootServletInitializer`（例如，在名为的类中`Application`）并添加Spring Boot `@SpringBootApplication`注释来创建可部署的战争，请使用类似于以下示例所示的代码：

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        // Customize the application or call application.sources(...) to add sources
        // Since our example is itself a @Configuration class (via @SpringBootApplication)
        // we actually don't need to override this method.
        return application;
    }

}
```

请记住，无论您输入什么，`sources`都只是Spring `ApplicationContext`。通常，任何已经起作用的东西都应该在这里工作。可能有些bean可以稍后删除，然后让Spring Boot为它们提供自己的默认值，但是应该可以使某些东西工作，然后再执行此操作。

可以将静态资源移动到类路径根目录中`/public`（或`/static`或`/resources`或`/META-INF/resources`）。这同样适用于`messages.properties`（Spring Boot自动在类路径的根目录中检测到）。

在Spring `DispatcherServlet`和Spring Security中使用Vanilla 不需要进一步更改。如果您的应用程序具有其他功能（例如，使用其他servlet或过滤器），则可能需要`Application`通过从中替换这些元素来向上下文中添加一些配置`web.xml`，如下所示：

- 甲`@Bean`类型的`Servlet`或`ServletRegistrationBean`将安装豆在容器中，好像它是一个``与``在`web.xml`。
- 甲`@Bean`型的`Filter`或`FilterRegistrationBean`表现得类似（作为``和``）。
- 一个`ApplicationContext`XML文件可以通过添加`@ImportResource`在你的`Application`。或者，可以在几行中重新创建已经大量使用注释配置的简单情况作为`@Bean`定义。

war文件运行后，可以通过向中添加一个`main`方法使其变为可执行文件`Application`，如以下示例所示：

```java
public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
}
```

|      | 如果打算以战争或可执行应用程序的形式启动应用程序，则需要使用对`SpringBootServletInitializer`回调函数可用的方法和`main`类似于以下类的方法来共享构建器的定制：`@SpringBootApplication public class Application extends SpringBootServletInitializer {     @Override    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {        return configureApplication(builder);    }     public static void main(String[] args) {        configureApplication(new SpringApplicationBuilder()).run(args);    }     private static SpringApplicationBuilder configureApplication(SpringApplicationBuilder builder) {        return builder.sources(Application.class).bannerMode(Banner.Mode.OFF);    } }` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

应用程序可以分为多个类别：

- 没有的Servlet 3.0+应用程序`web.xml`。
- 带有的应用程序`web.xml`。
- 具有上下文层次结构的应用程序。
- 没有上下文层次结构的应用程序。

所有这些都应该适合翻译，但是每种可能都需要稍微不同的技术。

如果Servlet 3.0+应用程序已经使用了Spring Servlet 3.0+初始化程序支持类，那么它们可能会很容易转换。通常，来自现有代码的所有代码`WebApplicationInitializer`都可以移入`SpringBootServletInitializer`。如果您现有的应用程序有多个`ApplicationContext`（例如，如果使用`AbstractDispatcherServletInitializer`），那么您可以将所有上下文源组合为一个`SpringApplication`。您可能会遇到的主要并发症是，如果合并不起作用，则需要维护上下文层次结构。有关示例，请参见[有关构建层次结构](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-build-an-application-context-hierarchy)的[条目](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto-build-an-application-context-hierarchy)。通常，需要拆分包含特定于Web的功能的现有父上下文，以便所有`ServletContextAware`组件都位于子上下文中。

不是Spring应用程序的应用程序可以转换为Spring Boot应用程序，前面提到的指南可能会有所帮助。但是，您可能仍然遇到问题。在这种情况下，建议您[使用标记来在Stack Overflow上提问`spring-boot`](https://stackoverflow.com/questions/tagged/spring-boot)。

### 17.3。将WAR部署到WebLogic

要将Spring Boot应用程序部署到WebLogic，必须确保servlet初始化程序**直接**实现`WebApplicationInitializer`（即使从已经实现了它的基类扩展）。

WebLogic的典型初始化程序应类似于以下示例：

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.web.WebApplicationInitializer;

@SpringBootApplication
public class MyApplication extends SpringBootServletInitializer implements WebApplicationInitializer {

}
```

如果使用Logback，则还需要告诉WebLogic首选打包版本，而不是服务器预先安装的版本。您可以通过添加`WEB-INF/weblogic.xml`具有以下内容的文件来做到这一点：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<wls:weblogic-web-app
    xmlns:wls="http://xmlns.oracle.com/weblogic/weblogic-web-app"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
        https://java.sun.com/xml/ns/javaee/ejb-jar_3_0.xsd
        http://xmlns.oracle.com/weblogic/weblogic-web-app
        https://xmlns.oracle.com/weblogic/weblogic-web-app/1.4/weblogic-web-app.xsd">
    <wls:container-descriptor>
        <wls:prefer-application-packages>
            <wls:package-name>org.slf4j</wls:package-name>
        </wls:prefer-application-packages>
    </wls:container-descriptor>
</wls:weblogic-web-app>
```

### 17.4。使用Jedis代替生菜

默认情况下，Spring Boot启动程序（`spring-boot-starter-data-redis`）使用[Lettuce](https://github.com/lettuce-io/lettuce-core/)。您需要排除该依赖性，而改为包含[Jedis](https://github.com/xetorthio/jedis/)。Spring Boot管理这些依赖关系，以帮助简化该过程。

以下示例显示了如何在Maven中执行此操作：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

以下示例显示了如何在Gradle中执行此操作：

```groovy
configurations {
    compile.exclude module: "lettuce"
}

dependencies {
    compile("redis.clients:jedis")
    // ...
}
```

最后更新时间2020-03-26 11:43:46 UTC



# 参考文档

| [法律](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/legal.html#legal) | 法律信息。                                                   |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [文档概述](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/documentation-overview.html#boot-documentation) | 关于文档，获得帮助，第一步等。                               |
| [入门](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/getting-started.html#getting-started) | 介绍Spring Boot，系统要求，Servlet容器，安装Spring Boot，开发第一个Spring Boot应用程序 |
| [使用Spring Boot](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/using-spring-boot.html#using-boot) | 构建系统，结构化代码，配置，Spring Bean和依赖注入等。        |
| [Spring Boot功能](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-features.html#boot-features) | 配置文件，日志记录，安全性，缓存，Spring集成，测试等。       |
| [弹簧启动器](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/production-ready-features.html#production-ready) | 监控，指标，审核等。                                         |
| [部署Spring Boot应用程序](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/deployment.html#deployment) | 部署到云，作为Unix应用程序安装。                             |
| [Spring Boot CLI](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/spring-boot-cli.html#cli) | 安装CLI，使用CLI，配置CLI等。                                |
| [生成工具插件](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins) | Maven插件，Gradle插件，Antlib等。                            |
| [“使用方法”指南](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/howto.html#howto) | 应用程序开发，配置，嵌入式服务器，数据访问等等               |

# 参考文档附录

| [应用属性](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-application-properties.html#common-application-properties) | 可用于配置应用程序的通用应用程序属性。     |
| ------------------------------------------------------------ | ------------------------------------------ |
| [配置元数据](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata) | 用于描述配置属性的元数据。                 |
| [自动配置类](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-auto-configuration-classes.html#auto-configuration-classes) | Spring Boot提供的自动配置类。              |
| [测试自动配置注释](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-test-auto-configuration.html#test-auto-configuration) | 用于测试应用程序切片的测试自动配置批注。   |
| [可执行罐](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-executable-jar-format.html#executable-jar) | Spring Boot的可执行jar，其启动器及其格式。 |
| [依赖版本](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/appendix-dependency-versions.html#appendix-dependency-versions) | Spring Boot管理的依赖项的详细信息。        |