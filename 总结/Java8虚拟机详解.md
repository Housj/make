# [简介](https://www.cnblogs.com/ysocean/p/9345787.html)



##**目录**



- [1、Oracle Java SE 8 产品组件](https://www.cnblogs.com/ysocean/p/9345787.html#_label0)
- [2、虚拟机](https://www.cnblogs.com/ysocean/p/9345787.html#_label1)
- [3、Java虚拟机](https://www.cnblogs.com/ysocean/p/9345787.html#_label2)
- [4、Java虚拟机种类](https://www.cnblogs.com/ysocean/p/9345787.html#_label3)

 

------

　　本系列博客我们将以当前默认的主流虚拟机HotSpot 为例，详细介绍 Java虚拟机。以 JDK1.7 为主，同时介绍与 JDK1.8 的不同之处，通过Oracle官网以及各种文献进行整理，并加以验证，力求保证这块知识的正确性，完整性。

　　以下是本系列博客参考的相关文档：

　　①、JDK1.7虚拟机规范：https://docs.oracle.com/javase/specs/jvms/se7/html/index.html

　　　　JDK1.8虚拟机规范：https://docs.oracle.com/javase/specs/jvms/se8/html/index.html

　　②、Oracle Java SE 8 产品组件：https://docs.oracle.com/javase/8/docs/index.html

　　③、周志明老师：《深入理解Java虚拟机：JVM高级特性与最佳实践》

　　④、陈涛老师：《HotSpot实战》

[回到顶部](https://www.cnblogs.com/ysocean/p/9345787.html#_labelTop)

### 1、Oracle Java SE 8 产品组件

　　通过上面给定的地址，我们可以看到如下这张图：

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180721112148301-26051730.png)

　　通常来说 Java平台标准版（Java SE）包括 Java SE开发工具包（JDK）和Java SE运行时环境（JRE）。

　　JRE提供了*运行*以Java编程语言编写的applet和应用程序所必需的库，**Java虚拟机**和其他组件；JDK包括JRE以及编译器和调试器等命令行开发工具，可以用来开发Java应用程序 。

　　PS：JDK包含JRE，我们通常安装JDK的同时也会安装JRE。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345787.html#_labelTop)

### 2、虚拟机

　　上图的最下一行Java虚拟机是被 JRE 所包含，我们在介绍Java虚拟机时，先了解虚拟机的概念。

　　所谓虚拟机，其实就是一台虚拟的机器，可以用来执行一系列虚拟的命令。大体上虚拟机可以分为两种：系统虚拟机和程序虚拟机。

　　①、系统虚拟机：是完全对物理计算机的仿真，可以说和一台真实的PC操作系统没什么区别。比如常用的 Vmare 以及 Visual Box 软件，通过这些软件能够模拟出具有完整硬件系统功能的、运行在一个完全隔离环境中的完整计算机系统。

　　②、程序虚拟机：专门为执行单个计算程序而产生，最典型的就是Java虚拟机，在Java虚拟机中执行字节码文件命令。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345787.html#_labelTop)

### 3、Java虚拟机

　　了解了什么是虚拟机，我们再看什么是 Java虚拟机。

　　Java虚拟机可以看做是一台抽象的计算机，如同真实的计算机那样，它有自己的指令集以及各种运行时内存区域，它与Java语言没有必然的联系，只与特定的二进制文件——class 文件格式关联（字节码文件），可以通过Java语言或者其他语言编写的程序编译成class文件，然后在Java虚拟机上运行。Java虚拟机有以下二个特点：

　　**①、语言无关**

　　Java虚拟机只和class文件关联，所以只要你编写程序的语言能够编译成class文件，那么都能够在Java虚拟机上运行。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180721133224710-1719932704.png)

 

　　**②、平台无关**

　　Java从诞生之初就宣传的一个口号：一次编写，到处运行。

　　也就是说Java是一个跨平台的语言，那么Java是如何实现跨平台的呢？

　　其实Java之所以跨平台是因为Java虚拟机的适配，不同的系统实现不同的Java虚拟机。Java虚拟机就相当于操作系统和应用程序之间的中介，每种平台安装适应该平台的Java虚拟机，那么我们编写的程序当然能够在任意平台运行。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180721133841198-101928900.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/9345787.html#_labelTop)

### 4、Java虚拟机种类

　　**商用虚拟机：**

 　**①、Sun HotSpot**

　　该虚拟机性能优越，是 sun JDK1.3 及以后所有 sun JDK 版本默认的虚拟机，使用最为广泛，本系列博客就是以这个虚拟机为平台进行介绍。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180721134438047-1736661668.png)

　　**②、BEA JRockit**

　　JRockit 虚拟机是 BEA 公司于 2002 年从 Appeal Virtual Machines 收购获得的虚拟机。它是一款面向服务器硬件和服务端使用场景高度优化过得虚拟机，曾经号称是“世界上速度最快的虚拟机”。由于专注于服务端应用，它的内部不包含解析器的实现，全部代码都靠即时编译器编译后执行。

　　**③、IBM J9**

　　J9 虚拟机是 IBM 公司单独开发的高性能虚拟机，它并不独立出售，而是作为 IBM 公司各种产品的执行平台，IBM 把它定义为一个可以适应从嵌入式设备到大型企业级应用的、高可移植性的Java运行平台。

　　**④、Sun Classic** 

　　这个虚拟机很原始，是 JDK1.0 时代使用的Java虚拟机，是各种虚拟机的鼻祖，它的内部不存在即时编译器，只能使用纯解释的方式运行。

　　**⑤、Sun Exact** 

　　这是 Sun 公司在 HotSpot 之外的另一个虚拟机，在 JDK1.2 时代曾短暂的投入过商用，它和 HotSpot 同时开发，但最终被 HotSpot 取代。

　　**⑥、Apache Harmony** 

　　Harmony 是 Apache 软件基金会主导的、开源的、独立的、实际兼容与 JDK1.5 和 JDK1.6的虚拟机实现，它间接催生了 Google Android 平台的 Dalvik 虚拟机，Android 的影响力现在有多大不用多说，目前已经是最成功的的数码设备通用平台。但是由于它的 TCK 授权问题，直接导致 Apache 与 Oracle 的决裂，从而退出了 JCP 组成，这是近代 Java 阵营遇到的最严重的分裂危机。

　　**嵌入式虚拟机**

　　**①、Dalvik**

　　Dalvik 虚拟机是 Google 等厂商合作开发的 Android 移动设备平台的核心组成部分之一，它执行 dex(Dalvik Executable) 文件而不是 class 文件，使用寄存器架构而不是栈架构，但是它的开发体系与Java有着千丝万缕的关系，可以直接使用大部分的 Java API、dex 文件可以直接从class文件转化而来。并且在 Android 2.2 中提供了即时编译器的实现，性能大大的提高。

　　**②、KVM**

　　在 Android、IOS 等智能手机操作系统出现之前，曾广泛应用于手机平台的一种虚拟机。

　　**③、CDC/CLDC HotSpot**

　　CDC和 CLDC HotSpot 分别是 Sun 针对高端嵌入式设备和中低端嵌入式设备的虚拟机，用来代替 KVM。



# [运行时内存结构](https://www.cnblogs.com/ysocean/p/9345622.html)



**目录**

- [1、运行时数据区结构图](https://www.cnblogs.com/ysocean/p/9345622.html#_label0)
- [2、程序计数器](https://www.cnblogs.com/ysocean/p/9345622.html#_label1)
- [3、虚拟机栈](https://www.cnblogs.com/ysocean/p/9345622.html#_label2)
- [4、本地方法栈](https://www.cnblogs.com/ysocean/p/9345622.html#_label3)
- [5、Java堆](https://www.cnblogs.com/ysocean/p/9345622.html#_label4)
- [6、方法区](https://www.cnblogs.com/ysocean/p/9345622.html#_label5)
- [7、运行时常量池](https://www.cnblogs.com/ysocean/p/9345622.html#_label6)
- [8、直接内存](https://www.cnblogs.com/ysocean/p/9345622.html#_label7)

 

------

　　首先通过一张图了解 Java程序的执行流程：

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180721205440562-1136861165.png)

　　我们编写好的Java源代码程序，通过Java编译器javac编译成Java虚拟机识别的class文件（字节码文件），然后由 JVM 中的类加载器加载编译生成的字节码文件，加载完毕之后再由 JVM 执行引擎去执行。在加载完毕到执行过程中，JVM会将程序执行时用到的数据和相关信息存储在运行时数据区（Runtime Data Area），这块区域也就是我们常说的JVM内存结构，垃圾回收也是作用在该区域。

　　关于这幅图涉及到的：

　　①、class文件

　　②、类加载器

　　③、运行时数据区

　　④、执行引擎

　　⑤、垃圾回收器

　　这都是接下来将要介绍的重点。

　　本篇博客我们将首先介绍什么是运行时数据区。

　　PS：下面介绍的是根据 **Java虚拟机规范** 定义的运行时数据区，上一篇博客我们讲过根据虚拟机规范实现的虚拟机有很多个，而不同的虚拟机其运行时数据区定义也会有所不同。比如默认的 HotSpot 在实现 JDK1.7 虚拟机规范时，其常量池的定义不在方法区中，而是移到了堆中；到了 HotSpot JDK1.8 中，则彻底移除了持久代（方法区）而使用Metaspace（元数据区）来进行替代等等，关于这些区别本篇博客也会在文章末尾进行相应的说明。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 1、运行时数据区结构图

　　**①、Java虚拟机规范定义的运行时数据区**

 　![img](https://img2018.cnblogs.com/blog/1120165/201906/1120165-20190629223841564-1721538458.png)

　　**②、HotSpot JDK1.8定义的运行时数据区**

　　![img](https://img2018.cnblogs.com/blog/1120165/201906/1120165-20190629232328134-1467592616.png)

　　**注意：**HotSpot实现的运行时数据区和Java虚拟机规范定义的还是有所不同的，

　　①、将**Java虚拟机栈和本地方法栈**合二为一；

　　②、**元数据区取代了方法区**，并且元数据区不在Java虚拟机中，而是在本地内存中。

　　③、**运行时常量池**由方法区中移到了堆中

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 2、程序计数器

　　程序计数器（Program Conputer Register）这是一块较小的内存空间，可以看做是当前线程所执行的字节码的行号指示器，在虚拟机的概念模型里，字节码解释器的工作就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

　　**①、线程私有**

　　Java虚拟机支持多线程，是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任一确定的时刻，一个处理器只会执行一条线程中的指令，因此为了线程切换后能恢复到正确的执行位置，每条线程都需要一个独立的程序计数器。因此线程启动时，JVM 会为每个线程分配一个PC寄存器（Program Conter，也称程序计数器）。

　　**②、记录当前字节码指令执行地址**

　　如果当前线程执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是 Native 方法，则这个计数器值为空（Undefined）。

　　**③、不抛 OutOfMemoryError 异常**

　　程序计数器的空间大小不会随着程序执行而改变，始终只是保存一个 returnAdress 类型的数据或者一个与平台相关的本地指针的值。所以该区域是Java运行时内存区域中唯一一个Java虚拟机规范中没有规定任何 OutOfMemoryError 情况的区域。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 3、虚拟机栈

　　Java虚拟机栈（Java Virtual Machine stack），这块区域也是线程私有的，与线程同时创建，用于存储栈帧。Java 每个方法执行的时候都会同时创建一个栈帧（Stack Frame）用于存储局部变量表、操作栈、动态链接、方法出口等信息，每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180722152410736-914758638.png)

　　**①、线程私有**

　　随线程创建而创建，声明周期和线程保持一致。

　　**②、由栈帧组成**

　　线程每个方法被执行的时候都会创建一个栈帧，用于存储局部变量表、操作栈、动态链接、方法出口等信息，每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程。

　　**③、抛出 StackOverflowError 和 OutOfMemoryError 异常**

　　如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 StackOverflowError 异常；如果虚拟机栈可以动态扩展，当扩展时无法申请到足够的内存时会抛出 OutOfMemoryError 异常。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 4、本地方法栈

　　本地方法栈（Native Method Stacks）作用和虚拟机栈类型，虚拟机栈执行的是Java方法，本地方法栈执行的是 Native 方法，本地方法栈也会抛出抛出 StackOverflowError 和 OutOfMemoryError 异常。

　　注意：由于虚拟机规范并没有对本地方法栈中的方法使用语言、使用方式和数据结构强制规定，因此具体的虚拟机可以自由实现它。上图我们也给出在 HotSpot 虚拟机中，本地方法栈和虚拟机栈合为一体了。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 5、Java堆

　　Java堆是Java虚拟机所管理内存最大、被所有线程共享的一块区域，目的是用来存放对象，基本上所有的对象实例和数组都在堆上分配（不是绝对）。Java堆也是垃圾回收器管理的主要区域。

　　**①、线程共享**

　　堆存放的对象，某个线程修改了对象属性，另外一个线程从堆中获取的该对象是修改后的对象，为什么堆要设计成线程共享呢？

　　我们可以假设堆是线程私有的，很显然一个系统创建的对象会有很多，而且有些对象会比较大，如果设计成线程私有的，那么如果有很多线程同时工作，那么都必须给他们分配相应的私有内存，我相信内存很快就撑爆了，很显然将堆设计为线程共享是最好不过了，不过凡事都具有两面性，线程共享的设计这也带来了**多线程并发资源冲突问题**，关于这个问题由于不是本系列博客的主旨，这里就不做详细介绍了。

　　**②、存放对象**

　　基本上所有的对象实例和数组都要在堆上进行分配，但是随着 JIT 编译器的发展和逃逸分析技术的成熟，栈上分配、标量替换等优化技术会导致对象不一定在堆上进行分配。

　　**③、垃圾收集**

　　Java堆也被称为“GC堆”，是垃圾回收器的主要操作内存区域。当前垃圾回收器都是使用的分代收集算法，所以Java堆还可以分为：新生代和老年代，而新生代又可以分为 Eden 空间、From Survivor 空间、To Survivor空间。这是为了更好的回收内存，关于垃圾回收算法在后续博客会详细介绍。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180722113121049-1392980428.png)

　　**④、抛出 OutOfMemoryError 异常**

　　根据Java虚拟机规范，Java堆可以处于物理上不连续的内存空间中，只要逻辑上连续即可，实现时既可以实现成固定大小，也可以是扩展的。如果在堆中没有完成实例分配，并且堆也无法扩展，将抛出OutOfMemoryError 异常。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 6、方法区

 　方法区（Method Area）用来存储已被Java虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

　　方法区也称为“永久代”，这是因为垃圾回收器对方法区的垃圾回收比较少，主要是针对常量池的回收以及对类型的卸载，回收条件比较苛刻。经常会导致对此内存未完全回收而导致内存泄露，最后当方法区无法满足内存分配时，将抛出 OutOfMemoryError 异常。

　　PS：在Java虚拟机规范中把方法区描述为堆的一个逻辑部分（https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.4），在很多虚拟机中（JRockit、IBM J9等虚拟机不存在永久代的概念）。

　　　　 在JDK1.8 的 HotSpot 虚拟机中，已经去掉了方法区的概念，用 Metaspace 代替，并且将其移到了本地内存来规划了。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 7、运行时常量池

 　在Java虚拟机规范中，运行时常量池（Runtime Constant Pool）用于存放编译期生成的各种字面量和符号引用，是**方法区**的一部分。但是Java虚拟机规范对其没有做任何细节的要求，所以不同虚拟机实现商可以按照自己的需求来实现该区域，比如在 **HotSpot** 虚拟机实现中，就将运行时常量池移到了堆中。

　　**①、存放字面量、符号引用、直接引用**

　　通常来说，该区域除了保存Class文件中描述的引用外，还会把翻译出来的直接引用也存储在运行时常量池，并且Java语言并不要求常量一定只能在编译器产生，运行期间也可能将常量放入池中，比如String类的intern()方法，**当调用intern方法时，如果池中已经包含一个与该`String`确定的字符串相同`equals(Object)`的字符串，则返回该字符串。否则，将此`String`对象添加到池中，并``返回此对象的引用。**关于该方法的介绍可以看我[这篇博客](https://www.cnblogs.com/ysocean/p/8571426.html#_label12)。

　　**②、抛出 OutOfMemoryError 异常**

　　运行时常量池是方法区的一部分，会受到方法区内存的限制，当常量池无法申请到内存时，会抛出该异常。

[回到顶部](https://www.cnblogs.com/ysocean/p/9345622.html#_labelTop)

### 8、直接内存

　　直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，它也不是Java虚拟机规范定义的内存区域。我们可以看到在 HotSpot 中，就将方法区移除了，用元数据区来代替，并且将元数据区从虚拟机运行时数据区移除了，转到了本地内存中，也就是说这块区域是受本机物理内存的限制，当申请的内存超过了本机物理内存，才会抛出 OutOfMemoryError 异常。

　　直接内存也是受本机物理内存的限制，在JDK1.4中新加入的 NIO（new input/output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的 I/O 方式，它可以使用 Native 函数库直接分配堆外内存，然后通过一个存储在Java堆里面的 DirectByteBuffer 对象作为这块内存的引用操作，这样避免了在Java堆和Native堆中来回复制数据，显著提高性能。



# [垃圾回收](https://www.cnblogs.com/ysocean/p/11108933.html)



**目录**

- [1、为什么要进行垃圾回收](https://www.cnblogs.com/ysocean/p/11108933.html#_label0)
- [2、为什么要了解垃圾回收](https://www.cnblogs.com/ysocean/p/11108933.html#_label1)
- [3、回收哪部分区域内存](https://www.cnblogs.com/ysocean/p/11108933.html#_label2)
- 4、如何判断对象为垃圾对象
  - [①、引用计数算法](https://www.cnblogs.com/ysocean/p/11108933.html#_label3_0)
  - [②、根搜索算法](https://www.cnblogs.com/ysocean/p/11108933.html#_label3_1)
- 5、如何进行垃圾回收
  - [①、标记-清除算法](https://www.cnblogs.com/ysocean/p/11108933.html#_label4_0)
  - [②、复制算法](https://www.cnblogs.com/ysocean/p/11108933.html#_label4_1)
  - [③、标记-整理算法](https://www.cnblogs.com/ysocean/p/11108933.html#_label4_2)
  - [④、分代收集算法](https://www.cnblogs.com/ysocean/p/11108933.html#_label4_3)
- [6、何时进行垃圾回收](https://www.cnblogs.com/ysocean/p/11108933.html#_label5)

 

------

　　如果对C++这门语言熟悉的人，再来看Java，就会发现这两者对垃圾（内存）回收的策略有很大的不同。

　　C++：垃圾回收很重要，我们必须要自己来回收！！！

　　Java：垃圾回收很重要，我们必须交给系统来帮我们完成！！！

　　我想这也能看出这两门语言设计者的心态吧，总之，Java和C++之间有一堵由内存动态分布和垃圾回收技术所围成的高墙，墙外面的人想进去，墙里面的人想出来。

　　本篇博客我们就来详细介绍Java的垃圾回收策略。

[回到顶部](https://www.cnblogs.com/ysocean/p/11108933.html#_labelTop)

### 1、为什么要进行垃圾回收

　　我们知道Java是一门面向对象的语言，在一个系统运行中，会伴随着很多对象的创建，而这些对象一旦创建了就占据了一定的内存，在上一篇博客[Java运行时内存结构](https://www.cnblogs.com/ysocean/p/9345622.html)中，我们介绍过创建的对象是保存在堆中的，当对象使用完毕之后，不对其进行清理，那么会一直占据内存空间，很明显内存空间是有限的，如果不回收这些无用的对象占据的内存，那么新创建的对象申请不了内存空间，系统就会抛出异常而无法运行，所以必须要经常进行内存的回收，也就是垃圾收集。

[回到顶部](https://www.cnblogs.com/ysocean/p/11108933.html#_labelTop)

### 2、为什么要了解垃圾回收

　　文章开头，我们就说Java的垃圾回收是系统自动进行的，不需要我们程序员手动处理，那么我们为什么还要了解垃圾回收呢，？

　　其实这也是一个程序员进阶的过程，生产项目在运行过程中，很可能会存在内存溢出、内存泄露等问题，出现了这些问题，我们应该怎么排查？以及在生产服务器有限的资源上如何更好的分配Java运行时内存区域，提高系统运行效率等，我们必须知其然知其所以然。

　　PS：本篇博客只是介绍Java垃圾回收机制，关于排查内存泄漏、溢出，运行时内存区域参数调优等会在后面进行介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/11108933.html#_labelTop)

### 3、回收哪部分区域内存

　　还是结合上一篇博客[Java运行时内存结构](https://www.cnblogs.com/ysocean/p/9345622.html)，我们介绍了Java运行时的内存结构，其中程序计数器、虚拟机栈、本地方法栈这三个区域是线程私有的，随线程而生，随线程而灭，栈中的栈帧随着方法的进入和退出而有条不紊的执行着入栈和出栈操作，这几个区域的内存分配和回收都具备确定性，在方法结束或线程结束时，内存也就跟着回收了，所以不需要我们考虑。

　　那么现在就剩下**Java堆**和**方法区**了，这两块区域在编译期间我们并不能完全确定创建多少个对象，有些是在运行时期创建的对象，所以Java内存回收机制主要是作用在这两块区域。

[回到顶部](https://www.cnblogs.com/ysocean/p/11108933.html#_labelTop)

### 4、如何判断对象为垃圾对象

　　通过上面介绍了，我们了解了为什么要进行垃圾回收以及回收哪部分的垃圾，那么接下来我们怎么去区分哪些对象为垃圾呢？

　　换句话来说，我们如何判断哪些对象还“活着”，哪些对象已经“死了”，那些“死了”的对象占据的内存就是我们要进行回收的。



#### ①、引用计数算法

　　这种算法是这样的：给每一个创建的对象增加一个引用计数器，每当有一个地方引用它时，这个计数器就加1；而当引用失效时，这个计数器就减1。当这个引用计数器值为0时，也就是说这个对象没有任何地方在使用它了，那么这就是一个无效的对象，便可以进行垃圾回收了。

　　这种算法实现简单，而且效率也很高。但是**Java没有采用该算法来进行垃圾回收**，因为这种算法无法解决对象之间的循环引用问题。

　　下面我们就来构造一个循环引用的例子：

　　首先，有一个 Person 类，这个类有两个自引用属性，分别表示其父亲，儿子。

![img](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.algorithmproject.leetcode.demo.JVM;
 2 
 3 /**
 4  * Create by YSOcean
 5  */
 6 public class Person {
 7 
 8     private Byte[] _1MB = null;
 9 
10     public Person() {
11         /**
12          * 这个成员属性的作用纯粹就是占据一定内存，以便在日志中查看是否被回收
13          */
14         _1MB = new Byte[1*1024*1024];
15     }
16 
17 
18 
19     private Person father;
20     private Person son;
21 
22     public Person getFather() {
23         return father;
24     }
25 
26     public void setFather(Person father) {
27         this.father = father;
28     }
29 
30     public Person getSon() {
31         return son;
32     }
33 
34     public void setSon(Person son) {
35         this.son = son;
36     }
37 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　接着，我们通过Person类构造两个对象，分别是父亲，儿子，如下：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 public static void main(String[] args) {
 2 
 3     Person father = new Person();
 4     Person son = new Person();
 5     father.setSon(son);
 6     son.setFather(father);
 7 
 8     father = null;
 9     son = null;
10     
11     /**
12      * 调用此方法表示希望进行一次垃圾回收。但是它不能保证垃圾回收一定会进行，
13      * 而且具体什么时候进行是取决于具体的虚拟机的，不同的虚拟机有不同的对策。
14      */
15     System.gc();
16 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　首先，从第3-6行代码，其运行时内存结构图如下：

 　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701204956946-2075256854.png)

　　father对象和son对象，其引用计数第一个是栈内存指向，第二个就是其属性互相引用对方，所有引用计数器都是2。

　　接着我们看第8,9行代码，分别将这两个对象置为null，也就是去掉了栈内存指向。

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701205254800-62820611.png)

　　这时候其实这两个对象只是自己互相引用了，没有别的地方在引用它们，引用计数器为1，那么这两个对象按照引用计数算法实现的虚拟机就不会回收，可想而知，这是我们不能接受的。

　　所以J**ava虚拟机都没有使用该算法来判断对象是否存活，**我们可以通过增加打印虚拟机参数来验证。

　　我们将上面的man函数，增加如下Java虚拟机参数，用来打印gc信息。

```
-verbose:gc
```

　　在IDEA编辑器中，添加方式如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701205643994-1582909078.png)

 　运行结果如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701210220057-1975227399.png)

　　我们看到12201K->1088K(125952K)的输出，表示垃圾收集GC前有12201K，回收后剩下1088K，堆的总量为125952K，回收的内存为12201K-1088K = 11113K。

　　换句话说，上面的例子Java虚拟机是有进行垃圾回收的，所以，这也间接佐证了Java虚拟机并不是采用的引用计数法来判断对象是否是垃圾。

　　PS：这些参数信息详解也会在后面博客进行详细介绍。



#### ②、根搜索算法

　　我们这里直接给出结论：**在主流的商用程序中（Java，C#），都是使用根搜索算法（GC Roots Tracing）来判定对象是否存活。** 

　　该算法思路：通过一系列名为“GC Roots” 的对象作为终点，当一个对象到GC Roots 之间无法通过引用到达时，那么该对象便可以进行回收了。

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701211918177-969975856.png)

　　上图Object1,Object2,Object3,Object4到GC Roots是可达的，所以不会被作为垃圾回收。

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701212015278-596693041.png)

　　上图Object1，Object2，Object3这三个对象互相引用，但是到 GC Roots不可达，所以都会被垃圾回收掉。

　　那么有哪些对象可以作为 GC Roots 呢？

　　在Java语言中，有如下4中对象可以作为 GC Roots:

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191129110328037-459228623.png)

 

 　PS：红色的对象是要被当做垃圾回收的！

```
1 1、虚拟机栈（栈帧中的本地变量表）中引用的对象
2 2、方法区中的静态变量属性引用的对象
3 3、方法区中常量引用的对象
4 4、本地方法栈中（JNI）（即一般说的Native方法）的引用的对象
```

[回到顶部](https://www.cnblogs.com/ysocean/p/11108933.html#_labelTop)

### 5、如何进行垃圾回收

　　垃圾回收涉及到大量的程序细节，而且各个平台的虚拟机操作内存的方式也不一样，但是他们进行垃圾回收的算法是通用的，所以这里我们也只介绍几种通用算法。



#### ①、标记-清除算法

　　**算法实现**：分为标记-清除两个阶段，首先根据上面的根搜索算法标记出所有需要回收的对象，在标记完成后，然后在统一回收掉所有被标记的对象。

　　**缺点**：

　　1、效率低：标记和清除这两个过程的效率都不高。

　　2、容易产生内存碎片：因为内存的申请通常不是连续的，那么清除一些对象后，那么就会产生大量不连续的内存碎片，而碎片太多时，当有个大对象需要分配内存时，便会造成没有足够的连续内存分配而提前触发垃圾回收，甚至直接抛出OutOfMemoryExecption。

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701213406016-286379646.png)



#### ②、复制算法

　　为了解决标记-清除算法的两个缺点，复制算法诞生了。

　　**算法实现**：将可用内存按容量划分为大小相等的两块区域，每次只使用其中一块，当这一块的内存用完了，就将还活着的对象复制到另一块区域上，然后再把已使用过的内存空间一次性清理掉。

　　**优点**：每次都是只对其中一块内存进行回收，不用考虑内存碎片的问题，而且分配内存时，只需要移动堆顶指针，按顺序进行分配即可，简单高效。

　　**缺点**：将内存分为两块，但是每次只能使用一块，也就是说，机器的一半内存是闲置的，这资源浪费有点严重。并且如果对象存活率较高，每次都需要复制大量的对象，效率也会变得很低。

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701215336656-48891683.png)



#### ③、标记-整理算法

　　上面我们说过复制算法会浪费一半的内存，并且对象存活率较高时，会有过多的复制操作，效率低下。

　　如果对象存活率很高，基本上不会进行垃圾回收时，标记-整理算法诞生了。

　　**算法实现**：首先标记出所有存活的对象，然后让所有存活对象向一端进行移动，最后直接清理到端边界以外的内存。

　　**局限性**：只有对象存活率很高的情况下，使用该算法才会效率较高。

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701215456537-787486222.png)



#### ④、分代收集算法

 　当前商业虚拟机都是采用此算法，但是其实这不是什么新的算法，而是上面几种算法的合集。

　　算法实现：根据对象的存活周期不同将内存分为几块，然后不同的区域采用不同的回收算法。

　　　　1、对于存活周期较短，每次都有大批对象死亡，只有少量存活的区域，采用复制算法，因为只需要付出少量存活对象的复制成本即可完成收集；

　　　　2、对于存活周期较长，没有额外空间进行分配担保的区域，采用标记-整理算法，或者标记-清除算法。

　　比如，对于 HotSpot 虚拟机，它将堆空间分为如下两块区域：

　　![img](https://img2018.cnblogs.com/blog/1120165/201907/1120165-20190701221404790-1636826353.png)

　　堆有新生代和老年代两块区域组成，而新生代区域又分为三个部分，分别是 Eden,From Surivor,To Survivor ,比例是8:1:1。

　　新生代采用复制算法，每次使用一块Eden区和一块Survivor区，当进行垃圾回收时，将Eden和一块Survivor区域的所有存活对象复制到另一块Survivor区域，然后清理到刚存放对象的区域，依次循环。

　　老年代采用标记-清除或者标记-整理算法，根据使用的垃圾回收器来进行判断。

　　至于为什么要这样，这是由于内存分配的机制导致的，新生代存的基本上都是朝生夕死的对象，而老年代存放的都是存活率很高的对象。关于内存分配下篇博客我们会详细进行介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/11108933.html#_labelTop)

### 6、何时进行垃圾回收

　　理清了什么是垃圾，怎么回收垃圾，最后一点就是Java虚拟机何时进行垃圾回收呢？

　　程序员可以调用 System.gc()方法，手动回收，但是调用此方法表示希望进行一次垃圾回收。但是它不能保证垃圾回收一定会进行，而且具体什么时候进行是取决于具体的虚拟机的，不同的虚拟机有不同的对策。

　　其次虚拟机会自行根据当前内存大小，判断何时进行垃圾回收，比如前面所说的，新生代满了，新产生的对象无法分配内存时，便会触发垃圾回收机制。

　　这里需要说明的是宣告一个对象死亡，至少要经历两次标记，前面我们说过，如果对象与GC Roots 不可达，那么此对象会被第一次标记并进行一次筛选，筛选的条件是此对象是否有必要执行 finalize() 方法，当对象没有覆盖 finalize()方法，或者该方法已经执行了一次，那么虚拟机都将视为没有必要执行finalize()方法。

　　如果这个对象有必要执行 finalize() 方法，那么该对象将会被放置在一个有虚拟机自动建立、低优先级，名为 F-Queue 队列中，GC会对F-Queue进行第二次标记，如果对象在finalize() 方法中成功拯救了自己（比如重新与GC Roots建立连接），那么第二次标记时，就会将该对象移除即将回收的集合，否则就会被回收。



# [垃圾收集器](https://www.cnblogs.com/ysocean/p/11117365.html)



**目录**

- [1、垃圾收集器种类](https://www.cnblogs.com/ysocean/p/11117365.html#_label0)
- [2、Serial收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label1)
- [3、ParNew收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label2)
- [4、Parallel Scavenge收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label3)
- [5、Serial Old收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label4)
- [6、Parallel Old收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label5)
- [7、CMS收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label6)
- [8、G1收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label7)
- [9、ZGC 收集器](https://www.cnblogs.com/ysocean/p/11117365.html#_label8)
- [10、如何选择垃圾收集器　　](https://www.cnblogs.com/ysocean/p/11117365.html#_label9)
- [11、几个名词解释](https://www.cnblogs.com/ysocean/p/11117365.html#_label10)

 

------

　　上一篇博客我们介绍了[Java虚拟机垃圾回收](https://www.cnblogs.com/ysocean/p/11108933.html)，介绍了几种常用的垃圾回收算法，包括标记-清除，标记整理，复制等，这些算法我们可以看做是内存回收的理论方法，那么在Java虚拟机中，由谁来具体实现这些方法呢？

　　没错，就是本篇博客介绍的内容——垃圾收集器。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 1、垃圾收集器种类

　　事实上Java虚拟机规范对垃圾收集器应该如何实现，并没有任何的规定，所以不同的厂商、不同版本的虚拟机所提供的垃圾收集器都会有所不同，并且一般都会提供参数供用户根据自己的应用特点和要求组合出各个年代所使用的收集器。

　　下图是基于 Sun HotSpot 虚拟机1.6版 Update 22的虚拟机种类：

　![image-20200422111404407](/Users/houshaojie/Library/Application Support/typora-user-images/image-20200422111404407.png)

　　由上图我们可以总结出几个结论：

　　①、新生代垃圾收集器：Serial、ParNew、Parallel Scavenge；

　　　　老年代垃圾收集器：Serial Old（MSC）、Parallel Old、CMS；

　　　　整堆垃圾收集器：G1

　　②、垃圾收集器之间的连线表示可以搭配使用，有如下几种组合：

　　　　Serial/Serial Old、Serial/CMS、ParNew/Serial Old、ParNew/CMS、Parallel Scavenge/Serial Old、Parallel Scavenge/Parallel Old、G1；

　　③、串行收集器Serial：Serial、Serial Old

　　　　并行收集器 Parallel：Parallel Scavenge、Parallel Old

　　　　并发收集器：CMS、G1

　　ps:对于文章中有一些名词不理解的，可以先看本篇博客[最后一个小节](https://www.cnblogs.com/ysocean/p/11117365.html#_label9)。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 2、Serial收集器

　　这是一个最基本，历史最悠久的垃圾收集器，是JDK1.3之前新生代唯一的垃圾收集器。

　　该收集器有如下特点：

　　**①、作用于新生代**

　　由上图也可看出，这是一个新生代垃圾收集器，采用的垃圾回收算法是复制算法。

　　**②、单线程**

　　工作时只会使用一个CPU或者一条收集线程去完成工作。

　　**③、进行垃圾收集时，必须暂停所有工作线程**

　　也就是说使用Serial收集器进行垃圾回收时，别的工作线程都暂停，系统这时候会有卡顿现象产生。

　　**④、适用场景**

　　Serial 收集器由于没有线程交互的开销，对于限定单个CPU的环境，可以获得最高的单线程收集效率。

　　一般在用户的桌面场景中，分配给虚拟机管理的内存一般来说不会很大，收集几十兆或一两百兆的新生代，定顿时间可以控制在几十毫秒，只要不是频繁发生的，这点停顿是可以接受的。

　　所以 Serial 收集器对于运行在 Client 模式下的虚拟机是一种很好的选择。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 3、ParNew收集器

　　这个收集器其实就是Serial收集器的多线程版本。

　　也就是说其特点除了多线程，其余和Serial收集器一样，事实上，这两个收集器实现上也共用了很多代码。

　　**①、作用于新生代**

　　一个新生代垃圾收集器，采用的垃圾回收算法是复制算法。

　　**②、多线程**

　　弥补了Serial收集器单线程的缺陷。

　　**③、适用场景**

　　由于其多线程的特性，是大多数运行在 Server 模式下的虚拟机首选新生代垃圾收集器。

　　另外需要说明的是，能够与下面将要介绍的划时代垃圾收集器CMS（Concurrent Mark Sweep）配合使用，也是一个重要原因。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 4、Parallel Scavenge收集器

　　前面介绍的垃圾收集器关注点是尽可能缩小垃圾收集时的用户线程停顿时间。而 Parallel Scanvenge 收集器是为了达到一个可控制的吞吐量。

　　**吞吐量 = 运行用户代码的时间 / （运行用户代码的时间+垃圾收集时间）**

　　可以用下面两个参数进行精确控制：

　　-XX:MaxGCPauseMills 设置最大垃圾收集停顿时间

　　-XX:GCTimeRatio 设置吞吐量大小

　　**①、作用于新生代**

　　一个新生代垃圾收集器，采用的垃圾回收算法是复制算法。

　　**②、多线程**

　　并行的多线程垃圾收集器。

　　**③、吞吐量**

　　这个收集器可以精确控制吞吐量。

　　**④、适用场景**

　　设置垃圾收集停顿时间短适合需要与用户快速交互的程序；

　　而设置高吞吐量可以最高效的利用CPU效率，尽快的完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 5、Serial Old收集器

　　Serial Old 收集器是 Serial 收集器的老年代版本，特点如下：

　　**①、作用于老年代**

　　**②、单线程**

　　**③、使用标记-整理算法**

　　**④、进行垃圾收集时，必须暂停所有工作线程**

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 6、Parallel Old收集器

　　Parallel Old 是 Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。

　　**①、作用于老年代**

　　**②、多线程**

　　**③、使用标记-整理算法**

　　除了具有以上几个特点，比较关键的是能和新生代收集器 Parallel Scavenge 配置使用，获得吞吐量最大化的效果。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 7、CMS收集器

　　CMS，全称为 Concurrent Mark Sweep ，顾名思义并发的，采用标记-清除算法。另外也将这个收集器称为并发低延迟收集器（Concurrent Low Pause Collector）

　　这是一款跨时代的垃圾收集器，真正做到了垃圾收集线程与用户线程（基本上）同时工作。和 Serial 收集器的 Stop The World（妈妈打扫房间的时候，你不能再将垃圾丢到地上） 相比，真正做到了妈妈一边打扫房间，你一边丢垃圾。

　　**①、作用于老年代**

　　**②、多线程**

　　**③、使用标记-清除算法**

　　整个算法过程分为如下 4 步：

　　一、初始标记（CMS initial mark）：只是仅仅标记GC Root 能够直接关联的对象，速度很快，但是需要“Stop The World”　　

　　二、并发标记（CMS concurrent mark）：进行GC Root Tracing的过程，简单来说就是遍历Initial Marking阶段标记出来的存活对象，然后继续递归标记这些对象可达的对象。

　　三、重新标记（CMS Remark）：修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，需要“Stop The World”。这个时间一般比初始标记长，但是远比并发标记时间短。

　　四、并发清除（CMS concurrent sweep）：对上一步标记的对象进行清除操作。

　　由于整个过程最耗时的操作是第二（并发标记）、四步（并发清除），而这两步垃圾收集器线程是可以和用户线程一起工作的。所以整体来说，CMS垃圾收集和用户线程是一起并发的执行的。

　　**缺点：**

　　**①、对CPU资源敏感**

　　因为在并发阶段，会占用一部分CPU资源，从而导致应用程序变慢，总吞吐量会降低。

　　**②、产生浮动垃圾**

　　由于CMS并发清理阶段用户线程还在工作，这个时候产生的垃圾，CMS无法在本次收集中处理掉它们，只能留在下一次GC时再将其处理掉，这部分垃圾称为“浮动垃圾”。

　　**③、产生内存垃圾碎片**

　　因为采用的算法是标记-清除，很明显，会有空间碎片产生。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 8、G1收集器

　　这是当前收集器技术发展的最前沿的成果。可以实现在基本不牺牲吞吐量的前提下完成低停顿的内存回收，首发于JDK8中，是JDK9默认的垃圾回收器。

　　这是因为它并不像前面介绍的所有垃圾收集器是区分新生代，老年代的，它作用于全区域。将整个Java堆划分为多个大小固定的独立区域（Regin），并且跟踪这些区域的垃圾堆积面积，在后台维护一个优先级列表，每次根据允许的收集时间，优先回收垃圾最多的区域，这样保证了G1收集器在有限的时间内可以获得最高的收集效率。

　　它与前面讲的 CMS 垃圾收集器相比，有两个显著的改进：

　　**①、采用 标记-整理 的回收算法**

　　这样不会产生空间碎片

　　**②、可以精确的控制停顿时间**

　　能让使用者明确指定一个长度为M毫秒的时间片内，消耗在垃圾回收上的时间不超过 N 毫秒。

　　③、作用于整个Java堆

　　G1收集器不区分年轻代和老年代，是整堆垃圾收集器。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 9、ZGC 收集器

　　这是JDK11发布的一款垃圾收集器，是一个可扩展的低延迟垃圾收集器，有如下特性：

　　①、暂停时间不超过10毫秒

　　②、暂停时间不会随堆或实时设置大小而增加

　　③、处理堆范围从几百M到几TB。

　　看暂停时间，这是一款很逆天的垃圾收集器了，实际效果咋样，因为版本太高，博主也还没用到过。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 10、如何选择垃圾收集器　　

　　详细文档可以查看官方介绍，如下

　　https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html

　　这里我们翻译一下结论：

　　除非应用程序有相当严格的暂停时间要求，否则就让JVM自己选择垃圾收集器。并且可以适当优先调整堆的大小来提高性能。如果还不满足要求，则以下面四点作为指导：

　　1. 如果应用程序内存小于100M，那么使用选项选择串行收集器`-XX:+UseSerialGC`。

　　2. 如果应用程序将在单核处理器上运行，并且没有停顿时间的要求，选择串行`-XX:+UseSerialGC`或者 JVM 自己选

　　3. 如果允许停顿时间超过1秒，选择并行或 JVM 自己选

　　4. 如果响应时间比总吞吐量更重要，并且垃圾收集暂停必须保持短于大约1秒，则使用`-XX:+UseConcMarkSweepGC`或选择并发收集器`-XX:+UseG1GC`。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117365.html#_labelTop)

### 11、几个名词解释

　　**①、并行**

　　指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。

　　适合科学计算、后台处理等弱交互场景。

　　**②、并发**

　　指用户线程与垃圾收集器线程同时执行（但不一定是并行的，可能会交替执行），用户线程继续执行，而垃圾收集线程运行在另一块CPU上。

　　适合对响应快速的场景，比如Web。

　　**③、停顿时间**

　　垃圾收集器做垃圾回收中断应用执行的时间。

　　**④、吞吐量**

　　 吞吐量 = 运行用户代码的时间 / （运行用户代码的时间+垃圾收集时间）



# [JVM参数](https://www.cnblogs.com/ysocean/p/11109018.html)



**目录**

- [1、标准参数](https://www.cnblogs.com/ysocean/p/11109018.html#_label0)
- [2、X 参数](https://www.cnblogs.com/ysocean/p/11109018.html#_label1)
- 3、XX参数
  - [①、Boolean类型](https://www.cnblogs.com/ysocean/p/11109018.html#_label2_0)
  - [②、Key-Value类型](https://www.cnblogs.com/ysocean/p/11109018.html#_label2_1)
- [4、参数详解（持续更新）](https://www.cnblogs.com/ysocean/p/11109018.html#_label3)

 

------

　　JVM参数有很多，其实我们直接使用默认的JVM参数，不去修改都可以满足大多数情况。但是如果你想在有限的硬件资源下，部署的系统达到最大的运行效率，那么进行相关的JVM参数设置是必不可少的。下面我们就来对这些JVM参数进行详细的介绍。

　　JVM参数主要分为以下三种（可以根据书写形式来区分）：

[回到顶部](https://www.cnblogs.com/ysocean/p/11109018.html#_labelTop)

### 1、标准参数

　　标准参数，顾名思义，标准参数中包括功能以及输出的结果都是很稳定的，基本上**不会随着JVM版本的变化而变化**。

　　我们可以通过 -help 命令来检索出所有标准参数。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190805230407845-314882895.png)

　　关于这些命令的详细解释，可以参考官网：https://docs.oracle.com/javase/7/docs/technotes/tools/solaris/java.html

　　-help 也是一个标准参数，再比如使用比较多的 -version也是。

　　①**、-version**

　　显示Java的版本信息。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190805231431993-437234634.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/11109018.html#_labelTop)

### 2、X 参数

　　对应前面讲的标准化参数，这是非标准化参数。表示在将来的JVM版本中可能会发生改变，但是这类以 -X开始的参数变化的比较小。

　　我们可以通过 Java -X 命令来检索所有-X 参数。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190805232014055-165198698.png)

　　关于这些参数的介绍，其实上图的中文解释很清楚了，这里我们不作过多的介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/11109018.html#_labelTop)

### 3、XX参数

　　这是我们日常开发中接触到最多的参数类型。这也是非标准化参数，相对来说不稳定，随着JVM版本的变化可能会发生变化，主要用于**JVM调优**和debug。

　　**注意**：这种参数是我们后续介绍JVM调优讲解最多的参数。

　　该参数的书写形式又分为两大类：



#### ①、Boolean类型

　　格式：-XX:[+-]<name> 表示启用或者禁用name属性。

　　例子：-XX:+UseG1GC（表示启用G1垃圾收集器）



#### ②、Key-Value类型

　　格式：-XX:<name>=<value> 表示name的属性值为value。

　　例子：-XX:MaxGCPauseMillis=500（表示设置GC的最大停顿时间是500ms）

[回到顶部](https://www.cnblogs.com/ysocean/p/11109018.html#_labelTop)

### 4、参数详解（持续更新）

　　本节我们会持续更新罗列一些JVM参数。

**1、打印已经被用户或者当前虚拟机设置过的参数**

```
-XX:+PrintCommandLineFlags
```

　　比如：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190818114630059-1026208989.png)

**2、最大堆和最小堆内存设置**

　　**-Xms512M：设置堆内存初始值为512M**

　　**-Xmx1024M：设置堆内存最大值为1024M**

　　这里的ms是memory start的简称，mx是memory max的简称，分别代表最小堆容量和最大堆容量。但是别看这里是-X参数，其实这是-XX参数，等价于：

　　**-XX:InitialHeapSize**

　　**-XX:MaxHeapSize**

　　在通常情况下，服务器项目在运行过程中，堆空间会不断的收缩与扩张，势必会造成不必要的系统压力。所以在生产环境中，**JVM的Xms和Xmx要设置成一样的**，能够避免GC在调整堆大小带来的不必要的压力。　　

**3、Dump异常快照以及以文件形式导出**

　　**-XX:+HeapDumpOnOutOfMemoryError**

　　**-XX:HeapDumpPath**

　　堆内存出现OOM的概率是所有内存耗尽异常中最高的，出错时的堆内信息对解决问题非常有帮助，所以给JVM设置这个参数(**-XX:+HeapDumpOnOutOfMemoryError**)，让JVM遇到OOM异常时能输出堆内信息，并通过（**-XX:+HeapDumpPath**）参数设置堆内存溢出快照输出的文件地址，这对于特别是对相隔数月才出现的OOM异常尤为重要。

　　这两个参数通常配套使用：

```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./
```

**4、发送OOM后，执行一个脚本**

　　***\*-XX:OnOutOfMemoryError\****

　　**比如这样设置：**

```
-XX:OnOutOfMemoryError="C:\Program Files\Java\jdk1.8.0_152\bin\jconsole.exe"
```

　　表示发生OOM后，运行jconsole.exe程序。这里可以不用加“”，因为jconsole.exe路径Program Files含有空格。

　　利用这个参数，我们可以在系统OOM后，自定义一个脚本，可以用来发送邮件告警信息，可以用来重启系统等等。

**5、打印gc信息**

　　**①、打印GC简单信息**

　　　　**-verbose:gc**

　　　　**-XX:+PrintGC**

　　一个是标准参数，一个是-XX参数，都是打印详细的gc信息。通常会打印如下信息：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190817114045012-1363706259.png)

　　比如第一行，表示GC回收之前有12195K的内存，回收之后剩余1088K，总共内存为125951K

　　**②、打印详细GC信息**

　　**-XX:+PrintGCDetails**

　　**-XX:+PrintGCTimeStamps**

　　**![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190817114914297-1467879699.png)**

**6、指定GC日志以文件输出**

　　**-Xloggc:./gc.log**

　　这个在参数用于将gc日志以文件的形式输出，更方便我们去查看日志，定位问题。

**7、设置永久代大小**

　　**-XX:MaxPermSize=1280m**

　　在JDK1.7以及以前的版本中，只有Hotspot 才有Perm区，称为永久代，它在启动时固定大小，很难进行调优。

　　在某些情况下，如果动态加载类过多，容易产生Perm区的 OOM。比如某个实际 Web 工程中，因为功能点较多，在运行过程中，要不断动态加载很多类，就会出现类似错误：

　　"Exception in thread 'dubbo client x.x.connect' java.lang.OutOfMemoryError:PermGenspace"

　　为了解决这个问题，就需要在项目启动时，设定运行参数-XX:MaxPermSize。

　　**注意：在JDK1.8以后面的版本，使用元空间来代替永久代。在 JDK1.8以及后面的版本中，如果设定参数-XX:MaxPermSize，启动JVM不会报错，但是会提示：**

　　**Java Hotspot 64Bit Server VM warning：ignoring option MaxPermSize=1280m:support was removed in 8.0**

 

**8、垃圾收集器常用参数**

![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190817112922860-1733276796.png)

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190817112945938-1947293243.png)

 

 

参考文档：https://www.oracle.com/technetwork/java/javase/tech/vmoptions-jsp-140102.html

https://docs.oracle.com/javase/7/docs/technotes/tools/solaris/java.html

# [内存分配](https://www.cnblogs.com/ysocean/p/11117359.html)



**目录**

- [1、Minor GC 、Major GC 和 Full GC](https://www.cnblogs.com/ysocean/p/11117359.html#_label0)
- [1、对象优先在 Eden 上分配](https://www.cnblogs.com/ysocean/p/11117359.html#_label1)
- [2、大对象直接进行老年代](https://www.cnblogs.com/ysocean/p/11117359.html#_label2)
- [3、长期存活的对象将进入老年代](https://www.cnblogs.com/ysocean/p/11117359.html#_label3)
- [4、新生代Survivor 区相同年龄所有对象之和大于 Survivor 所有对象之和的一半，大于等于该年龄的对象进入老年代](https://www.cnblogs.com/ysocean/p/11117359.html#_label4)
- [5、空间分配担保原则](https://www.cnblogs.com/ysocean/p/11117359.html#_label5)

 

------

　　我们说Java是自动进行内存管理的，所谓自动化就是，不需要程序员操心，Java会自动进行**内存分配**和**内存回收**这两方面。

　　前面我们介绍过如何通过垃圾回收器来回收内存，那么本篇博客我们来聊聊如何进行分配内存。

　　对象的内存分配，往大方向上讲，就是堆上进行分配（但也有可能经过JIT编译后被拆散为标量类型并间接的在栈上分配），对象主要分配在新生代 Eden 区上，如果启动了本地线程分配缓冲，将按线程优先在 TLAB 上分配。少数情况下也可能会直接分配在老年代上（下面会详细介绍），分配的规则并不是百分之百固定的，其细节取决于当前使用哪一种垃圾收集器组合，还有虚拟机中与内存相关的参数设置。

　　本篇博客会介绍几条最普遍的内存分配规则。通过增加 -XX:+UseParallelGC 参数，表示使用的垃圾收集器是 Parallel Scavenge + Serial Old ，通过这两个垃圾收集器组合进行校验。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117359.html#_labelTop)

### 1、Minor GC 、Major GC 和 Full GC

　　下面会出现这几个概念，所以这里首先介绍一下。

　　**①、Minor GC**

　　也叫Young GC，指的是新生代 GC，发生在新生代（Eden区和Survivor区）的垃圾回收。因为Java对象大多是朝生夕死的，所以 Minor GC 通常很频繁，一般回收速度也很快。

　　**②、Major GC**

　　也叫Old GC，指的是老年代的 GC，发生在老年代的垃圾回收，该区域的对象存活时间比较长，通常来讲，发生 Major GC时，会伴随着一次 Minor GC，而 Major GC 的速度一般会比 Minor GC 慢10倍。

　　**②、Full GC**

　　指的是全区域（整个堆）的垃圾回收，通常来说和 Major GC 是等价的。　　

[回到顶部](https://www.cnblogs.com/ysocean/p/11117359.html#_labelTop)

### 1、对象优先在 Eden 上分配

　　大多数情况下，对象优先在 Eden 上分配。当 Eden 区没有足够的空间进行分配时，虚拟机将会发起一次 Minor GC(新生代GC)。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
package com.ys.algorithmproject.leetcode.demo.JVM;

/**
 * Create by YSOcean
 * 对象优先在Eden区上分配
 */
public class EdenTest {
    private static final int _1MB = 1024*1024;

    /**
     * 虚拟机参数设置：-XX:+UseParallelGC -XX:+PrintGCDetails -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
     * @param args
     */
    public static void main(String[] args) {
        byte[] a = new byte[2*_1MB];
        byte[] b = new byte[2*_1MB];
        byte[] c = new byte[2*_1MB];
        byte[] d = new byte[3*_1MB];
    }
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　运行时的虚拟机参数设置为：

```
-XX:+UseParallelGC -XX:+PrintGCDetails -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
```

　　①、 -XX:+UseParallelGC 参数，表示使用的垃圾收集器是 Parallel Scavenge + Serial Old ；

　　②、-XX:+PrintGCDetails 参数，表示打印详细的GC日志，便于我们查看GC情况

　　③、-Xms20M -Xmx20M 这两个参数分别表示设置最大堆，最小堆内存都是20M

　　④、-Xmn 参数表示设置新生代大小为 10M

　　⑤、-XX:SurvivorRatio=8 新生代中的 Eden 区和 Survivor 区的比值为8:1，注意 Survivor是有两个的。

　　运行打印的GC日志为：

![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190818121244818-1642105242.png)

　　我们首先分析设置的JVM参数，表示堆中内存为20M，新生代和老年代分别各占一半为10M，并且新生代的Eden区为8M，剩下两个 Survivor 各为 1M。

　　在看代码，首先分配了三个大小都为2M的对象 a,b,c。这时候新生代对象的 Eden区已经被占用了6M，这时候来了一个对象d，大小为3M，发现新生代Eden区已经不足以分配对象d了，于是发起一次Minor GC。GC期间虚拟机又发现现在已有3个 2MB对象无法全部放入Survivor空间（Survivor空间只有1MB），所以只好通过分配担保机制提前转移到老年代中，然后将这个对象d分配到新生代Eden区中。

　　我们查看日志，在eden区中，总共8192K的空间，被使用了38%，约等于3113K，大概就是对象d(3MB)的大小。其次在老年代中，总共10240K（10MB），被使用了6865K，大概也就是a,b,c这三个对象的大小（6MB）。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117359.html#_labelTop)

### 2、大对象直接进行老年代

　　通常大对象是指需要大量连续内存空间的Java对象，比较典型的就是那种很长的字符串以及数组。

　　系统中出现大量大对象是很影响性能的，这样会导致还有不少空间时就提前触发垃圾回收来放置这些对象。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
package com.ys.algorithmproject.leetcode.demo.JVM;

/**
 * Create by YSOcean
 * 大对象直接在老年代上分配
 */
public class OldTest {
    private static final int _1MB = 1024*1024;

    /**
     * 虚拟机参数设置：-XX:+UseParallelGC -XX:+PrintGCDetails -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
     * @param args
     */
    public static void main(String[] args) {
        byte[] a = new byte[8*_1MB];

    }
}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　运行时虚拟机参数还和上面一样，运行的GC日志如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190818124823152-745808027.png)

　　可以看到老年代 ParOldGen直接被使用了 8192K，而新生代只被占用了1820K。

　　PS：可以通过设置-XX:PretenureSizeThreshold 参数，大于这个参数设置值的对象直接在老年代中分配，但是这个参数只对 Serial 和 ParNew 这两款垃圾收集器有效，Parallel Scavenge 收集器不认识这个参数。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117359.html#_labelTop)

### 3、长期存活的对象将进入老年代

 　我们知道Java虚拟机是通过分代收集的思想来管理内存，新创建的对象通常放在新生代，除此之外，还有一些对象放在老年代。为了识别哪些对象放在新生代，哪些对象放在老年代，虚拟机给每个对象定义了一个年龄计数器(Age)，如果对象在新生代Eden创建，并经历一次 Minor GC 后仍然存活，并且能够被 Survivor 容纳的话，虚拟机会将该对象移动到 Survivor 区域，并将对象的年龄Age+1。

　　新生代对象每熬过一次 Minor GC，年龄就增加1，当它的年龄增加到一定阈值时（默认是15岁），就会被晋升到老年代中。

　　这个年龄阈值可以通过如下参数来设置（N表示晋升到老年代的阈值）：

```
-XX:MaxTenuringThreshold=N
```

　　验证代码如下：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
package com.ys.algorithmproject.leetcode.demo.JVM;

/**
 * Create by YSOcean
 * 新生代对象经过N次Minor GC后，晋升到老年代
 */
public class OldAgeTest {
    private static final int _1MB = 1024*1024;

    /**
     * 虚拟机参数设置：-XX:MaxTenuringThreshold=1 -XX:+UseParallelGC -XX:+PrintGCDetails -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8
     * @param args
     */
    public static void main(String[] args) {
        byte[] a = new byte[_1MB];
        System.gc();

    }

}
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　注意：这里我们设置 -XX:MaxTenuringThreshold=1，也就是经历一次gc，新生代对象就直接进入老年代了，然后手动调用了 System.gc() 方法，表示让虚拟机进行垃圾回收。打印的日志如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190819204634025-908108573.png)

　　注意看，代码中我们只创建了一个 1MB大小的对象，但是老年代占用了1999K的内存，而新生代确只有246K。

　　接下来可以将 -XX:MaxTenuringThreshold 参数设置的更大一点，来对比打印的日志，这里读者可以自己进行验证。

[回到顶部](https://www.cnblogs.com/ysocean/p/11117359.html#_labelTop)

### 4、新生代Survivor 区相同年龄所有对象之和大于 Survivor 所有对象之和的一半，大于等于该年龄的对象进入老年代

　　Java虚拟机并不会死板的根据上面第3点说的，设置-XX:MaxTenuringThreshold 的阈值，只有对象经历该阈值次GC后，才会进入到老年代。而是会根据新生代对象的年龄来动态的决定哪些对象可以进入到老年代。

　　也就是说，新生代经历一次 Minor GC 后，Survivor 区域存活对象的所有相同年龄之和大于整个 Survivor 区域的所有对象之和，那么该区域大于等于这个年龄的对象就会进入老年代，而无需等到 -XX:MaxTenuringThreshold 设置的阈值。

 

[回到顶部](https://www.cnblogs.com/ysocean/p/11117359.html#_labelTop)

### 5、空间分配担保原则

　　在前面介绍 [垃圾回收](https://www.cnblogs.com/ysocean/p/11108933.html) 时，我们介绍过现在Java虚拟机采用的是分代回收算法，新生代采用复制收集算法，而老年代采用标记整理，或者标记清除算法。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190819211653146-411821811.png)

　　新生代内存分为一块 Eden区，和两块 Survivor 区域，当发生一次 Minor GC时，虚拟机会将Eden和一块Survivor区域的所有存活对象复制到另一块Survivor区域，通常情况下，Java对象朝生夕死，一块 Survivor 区域是能够存放GC后剩余的对象的，但是极端情况下，GC后仍然有大量存活的对象，那么一块 Survivor 区域就会存放不下这么多的对象，那么这时候就需要老年代进行分配担保，让无法放入 Survivor 区域的对象直接进入到老年代，当然前提是老年代还有空间能够存放这些对象。但是实际情况是在完成GC之前，是不知道还有多少对象能够存活下来的，所以老年代也无法确认是否能够存放GC后新生代转移过来的对象，那么这该怎么办呢?

　　前面我们介绍的都是Minor GC,那么何时会发生 Full GC？

　　在发生 Minor GC 时，虚拟机会检测之前每次晋升到老年代的平均大小是否大于老年代的剩余空间，如果大于，则改为 Full GC。如果小于，则查看 HandlePromotionFailure 设置是否允许担保失败，如果允许，那只会进行一次 Minor GC，如果不允许，则也要进行一次 Full GC。

```
-XX:-HandlePromotionFailure
```

　　回到第一个问题，老年代也无法确认是否能够存放GC后新生代转移过来的对象，那么这该怎么办呢?

　　也就是取之前每一次回收晋升到老年代对象容量的平均大小作为经验值，然后与老年代剩余空间进行比较，来决定是否进行 Full GC，从而让老年代腾出更多的空间。

　　通常情况下，我们会将 HandlePromotionFaile 设置为允许担保失败，这样能够避免频繁的发生 Full GC。

 

# [虚拟机监控和分析工具——命令行](https://www.cnblogs.com/ysocean/p/11311667.html)



**目录**

- [1、jps：显示虚拟机进程](https://www.cnblogs.com/ysocean/p/11311667.html#_label0)
- [2、jstat：统计监视虚拟机信息工具](https://www.cnblogs.com/ysocean/p/11311667.html#_label1)
- [3、jinfo:实时的查看和调整虚拟机各项参数](https://www.cnblogs.com/ysocean/p/11311667.html#_label2)
- [4、jmap:内存映像工具](https://www.cnblogs.com/ysocean/p/11311667.html#_label3)
- [5、jstack：Java堆栈跟踪工具 ](https://www.cnblogs.com/ysocean/p/11311667.html#_label4)

 

------

　　通过前面的几篇博客，我们介绍了Java虚拟机的内存分配以及内存回收等理论知识，了解这些知识对于我们在实际生产环境中提高系统的运行效率是有很大的帮助的。但是话又说回来，在实际生产环境中，线上项目正在运行，我们怎么去监控虚拟机运行效率？又或者线上项目发生了OOM，异常堆栈信息，我们又怎么去抓取，然后怎么去分析定位问题呢？

　　本篇博客，我们就来介绍各种虚拟机监控和分析工具，当然都是命令行工具，不够直观，下篇博客我们会介绍各种可视化工具。

[回到顶部](https://www.cnblogs.com/ysocean/p/11311667.html#_labelTop)

### 1、jps：显示虚拟机进程

```
JVM Process Status Tools ，显示指定系统内所有的 HotSpot 虚拟机进程。
```

　　该命令有如下常用参数：

　　①、-l 

　　显示应用程序main类的完整包名称或应用程序的JAR文件的完整路径名。

　　②、-v

　　显示虚拟机启动时的JVM参数。

　　③、-m

　　显示虚拟机进程启动时传递给主类 main() 函数的参数。

　　比如，我在服务器上启动了一个Tomcat，如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190820200641843-2112367170.png)

　　然后，输入 jps 命令，打印信息如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190820200714344-692232240.png)

　　这里的 Bootstrap 便是启动的 Tomcat进程。可以加上 -v 参数，显示所有传递给 JVM的参数信息。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190820201543363-49183902.png)

　　PS：jps 命令默认是没有安装的，需要进行安装，具体安装步骤可以百度，我这里就不做详细介绍了。

　　jps更多详细信息，请参考官方文档：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html

[回到顶部](https://www.cnblogs.com/ysocean/p/11311667.html#_labelTop)

### 2、jstat：统计监视虚拟机信息工具

```
JVM Statistics Monitoring Tool，用于收集虚拟机各方面的运行数据。
```

　　jstat 是用于监视虚拟机各种运行时状态信息的命令行工具。它可以显示本地或远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行时数据，它是**运行时期定位虚拟机性能问题的首选工具**。但是终究只是命令行工具，后面我们会介绍图形化工具，更加直观。

　　该命令监控本地的格式如下：

　　**jstat -参数 vmid 采样间隔时间 采样次数**

　　①、常用参数有如下

 　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190820203351874-2091875600.png)

　　②、vmid

　　表示目标虚拟机的标识符，在Linux系统上可以通过上小节我们介绍的 jps 命令，前面输出的数字便是进程 PID。在windows平台上，可以通过任务管理器查看。

　　③、采样间隔时间

　　默认单位是毫毛，必须是正整数。

　　**实例1**：这里我们加入 -class 参数，查看类装载信息：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190820203925569-1409729786.png)

　　相关表头信息：

　　Loaded：加载的类数量。

　　Bytes：加载的类字节KB大小。

　　Unloaded：卸载的类数量。

　　Bytes：卸载的类字节KB大小。

　　Time：执行类加载和卸载操作所花费的时间。

 

　　jstat更多详细信息，请参考官方文档：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html

[回到顶部](https://www.cnblogs.com/ysocean/p/11311667.html#_labelTop)

### 3、jinfo:实时的查看和调整虚拟机各项参数

```
jinfo(Confiiguration Info for Java)：实时的查看和调整虚拟机各项参数
```

　　jinfo ，通过此命令，我们可以实时的查看和调整虚拟机的各项参数（包括显示指定或默认配置的）。

　　该命令格式如下：

　　jinfo [ 选项 ] pid

　　①、常用选项如下

　　一、没有选项

　　打印系统属性名称键值对。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190820205901149-1592990124.png)

　　二、-参数名称

　　打印指定参数的名称和值。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190820210235344-421974861.png)

　　三、-flag [+|-] 参数名称

　　启用或者禁用指定的布尔命令。

　　四、-flag name=value

　　设置参数name的值为value

　　五、-sysprops

　　打印Java属性名称键值对。

　　②、pid

　　进程号，和上面一样，可以通过jps命令获取。

　　 jinfo更多详细信息，请参考官方文档：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jinfo.html

[回到顶部](https://www.cnblogs.com/ysocean/p/11311667.html#_labelTop)

### 4、jmap:内存映像工具

```
jmap(Memory Map for Java)：用于生成堆存储快照
```

　　jmap主要用于获取堆存储快照文件，在生产环境中，发生OOM（堆内存溢出）异常时，我们可以通过这个快照文件来快速定位到具体代码位置。

　　这个命令还可以查询 finalize 队列，Java堆和永久代信息，如空间使用率、当前用的是哪种垃圾收集器等。

　　该命令格式如下：

　　jmap [参数] pid

　　①、常用参数如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190821202756420-1693076792.png)

　　对于堆内存溢出异常，在前面介绍[虚拟机参数](https://www.cnblogs.com/ysocean/p/11109018.html#_label3)时，我们介绍过，通过下面两个参数，也能够打印堆内存快照。

```
　　-XX:+HeapDumpOnOutOfMemoryError

　　-XX:HeapDumpPath
```

　　下面，我们通过如下代码，演示堆内存溢出异常：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 package com.ys.algorithmproject.leetcode.demo.JVM;
 2 
 3 import java.util.ArrayList;
 4 import java.util.List;
 5 
 6 /**
 7  * Create by YSOcean
 8  *
 9  */
10 public class JmapTest {
11     private static final int _1MB = 1024*1024;
12 
13     /**
14      * 虚拟机参数设置： -Xms20M -Xmx20M -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./
15      * @param args
16      */
17     public static void main(String[] args) {
18         List<Object> list = new ArrayList<>();
19         while(true){
20             list.add(new Object[_1MB]);
21         }
22     }
23 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　设置虚拟机参数后，然后运行这段代码，就会发生堆内存溢出异常，并在根目录下生成快照文件 java_pid10840.hprof。

 　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190821203810625-903488951.png)

　　那么，怎么通过 jmap 命令来生成堆内存快照呢？

```
jmap -dump:format=b,file=heap20190821.hprof  16823
```

　　后面的数字是进程PID，可以通过jps命令来获取。

　　得到堆内存快照了，那么我们怎么去查看呢？

　　在eclipse中，可以下载 **MAT** 工具，而在 IDEA中，可以下载 **JProfiler** 插件。实在不行，可以用我们下篇博客介绍的几个可视化工具，具体情况见下篇博客。

　　jmap更多详细信息，请参考官方文档：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html

[回到顶部](https://www.cnblogs.com/ysocean/p/11311667.html#_labelTop)

### 5、jstack：Java堆栈跟踪工具 

```
 Stack Trace for Java，用于生成虚拟机当前时刻的线程快照。
```

　　线程快照其实就是当前虚拟机每一条线程正在执行的堆栈的集合，通过线程快照可以用来定位线程出现长时间停顿的原因（线程间死锁、死循环、请求外部资源导致的长时间等待）。

　　该命令格式如下：

　　jstack [选项] pid

　　①、常用选项如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190826212928098-88887238.png)

参考文档：https://docs.oracle.com/javase/8/docs/technotes/tools/index.html



# [虚拟机监控和分析工具——可视化](https://www.cnblogs.com/ysocean/p/11415514.html)



**目录**

- 1、JConsole
  - [①、启动 JConsole](https://www.cnblogs.com/ysocean/p/11415514.html#_label0_0)
  - [②、监控界面介绍](https://www.cnblogs.com/ysocean/p/11415514.html#_label0_1)
  - [③、配置Tomcat远程监控](https://www.cnblogs.com/ysocean/p/11415514.html#_label0_2)
  - [④、配置远程jar包监控](https://www.cnblogs.com/ysocean/p/11415514.html#_label0_3)
-  2、JVisualVM
  - [①、启动 JVisualVM](https://www.cnblogs.com/ysocean/p/11415514.html#_label1_0)
  - [②、监控界面介绍](https://www.cnblogs.com/ysocean/p/11415514.html#_label1_1)
  - [③、插件机制](https://www.cnblogs.com/ysocean/p/11415514.html#_label1_2)
  - [④、配置远程连接](https://www.cnblogs.com/ysocean/p/11415514.html#_label1_3)
  - [⑤、使用文档](https://www.cnblogs.com/ysocean/p/11415514.html#_label1_4)

 

------

　　上篇博客我们介绍了[虚拟机监控和分析命令行工具](https://www.cnblogs.com/ysocean/p/11311667.html)，由于其不够直观，不是很容易排查问题，那么本篇博客我们就来介绍几个可视化工具。

[回到顶部](https://www.cnblogs.com/ysocean/p/11415514.html#_labelTop)

### 1、JConsole

　　JConsole（Java Monitoring and Management Console）是一款基于 JMX 的可视化监视和管理的工具。它管理部分的功能是针对 JMX MBean 进行管理，MBean 可以使用代码、中间件服务器的管理控制台或者所有符合 JMX 规范的软件进行访问。

　　JMX(Java Management Extensions)是一个为应用程序植入管理功能的框架，一套标准的代理和服务；MBean就是一种规范的JavaBean，通过集成和实现一套标准的Bean接口。



#### ①、启动 JConsole

　　这是我们JDK自带的监控工具，在JDK的安装目录bin下即可找到。

　　如果配置过JDK环境变量，在CMD命令提示符中输入 jconsole 也可直接打开。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827213057667-434262524.png)

　　这是一个可执行文件，直接双击即可打开。打开如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827213155961-1658759711.png)



#### ②、监控界面介绍

　　JConsole 这个监控工具可以监控本地进程以及远程进程，我们这里以监控本地进程为例，来介绍具体的监控界面。

　　点击本地进程下面的任意一栏，进入到监控界面。

　　**1、监控概览**

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827214421083-733683653.png)

　　这个界面是我们建立本地连接后，进入的第一个页面。显示的是整个虚拟机主要运行数据的概览，包括“堆使用情况”、“线程”、“类”、“CPU占用率”等四项信息的曲线图，这些曲线图是后面“内存”、“线程”、“类”页签的信息汇总，下面会分别介绍这几个页签。

　　**2、内存监控**

　　这个页签相当于上一篇博客介绍的jstat命令，不过这里是可视化的。用于监视虚拟机内存的一些变化趋势。

　　监视区域如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827221045164-337453861.png)

　　**3、线程监控**

　　这个页签相当于上篇博客介绍的可视化的jstat 命令。遇到线程停顿的时候可以使用这个页签进行监控分析。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827222658985-781896705.png)

　　另外，此页面左下角还有一个检测死锁的按钮，出现线程死锁后，点击此按钮，便会出现一个新的死锁页签。

　　比如，对于如下这段死锁代码：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     @GetMapping("/test2")
 2     public void test2() throws Exception{
 3         Object lock1 = new Object();
 4         Object lock2 = new Object();
 5 
 6         new Thread(()->{
 7             synchronized (lock1){
 8                 try {
 9                     Thread.sleep(1000);
10                 } catch (InterruptedException e) {
11                     e.printStackTrace();
12                 }
13                 synchronized (lock2){
14                     System.out.println("线程1结束运行");
15                 }
16             }
17         }).start();
18 
19         new Thread(()->{
20             synchronized (lock2){
21                 try {
22                     Thread.sleep(1000);
23                 } catch (InterruptedException e) {
24                     e.printStackTrace();
25                 }
26                 synchronized (lock1){
27                     System.out.println("线程2结束运行");
28                 }
29             }
30         }).start();
31     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这里创建了两把锁，lock1,lock2，创建了两个线程，线程1获取到lock1后，说你给我lock2，我就释放lock1；而线程2获取到lock2后，说你给我lock1，我就释放lock2。两个线程谁也不释放，于是便造成了死锁现象。

　　通过监控工具便可以检测到，如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827223516790-1922036927.png)

　　**4、类监控**

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827224013490-1651850789.png)

　　**5、VM概要**

　　展示一些JVM信息。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190827224130133-918692655.png)



#### ③、配置Tomcat远程监控

 　其实使用监控工具，我们很少对本地的程序进行监控，大多数情况都是对部署在远程Linux服务器上的程序进行监控，那么想要使用 JConsole这款工具进行远程监控，我们必须要进行一些配置。我们首先介绍对Tomcat的远程监控。

　　**1、配置catalina.sh**

　　在该文件下加入如下配置信息：

```
JAVA_OPTS="$JAVA_OPTS -Djava.rmi.server.hostname=192.168.146.200 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1099 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
```

　　-Dcom.sun.management.jmxremote 表示开启远程连接。

　　-Dcom.sun.management.jmxremote.port=9004 表示设置远程连接端口为1099

　　-Dcom.sun.management.jmxremote.authenticate=false 表示不需要密码验证

　　-Dcom.sun.management.jmxremote.ssl=fals 表示不需要开启ssl连接

　　-Djava.net.preferIPv4Stack=true 表示只支持IPV4地址

　　-Djava.rmi.server.hostname=192.168.146.200 表示监控的主机名为192.168.146.200

　　添加位置如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190829074001162-1132842741.png)

 

　　**2、建立连接**

　　通过上面的配置，启动Tomcat后，我们只需要在 JConsole 的远程连接界面，输入 192.168.146.200:9004 ，然后点击连接即可。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828213823223-1260877022.png)

　　**3、连接错误情况**

　　**如果无法连接，需要依次检测如下信息：**

　　①、配置的端口不能被占用，可以通过 netstat -tunlp|grep 1099 命令验证。

　　②、防火墙开启对上面设置端口的信任

　　③、通过 hostname -i 命令，如果打印的不是前面设置的ip地址，则需要通过 vim /etc/hosts 命令，将127.0.0.1 修改为本机IP地址。



#### ④、配置远程jar包监控

　　启动jar包的命令如下：

```
nohup java -Djava.rmi.server.hostname=192.168.146.200 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1089 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -jar jvm-0.0.1-SNAPSHOT.jar &
```

　　配置端口，ip地址，和远程监控Tomcat大体上是一样的，然后建立连接即可。

[回到顶部](https://www.cnblogs.com/ysocean/p/11415514.html#_labelTop)

###  2、JVisualVM

　　英文介绍为 All-in-One Java Troubleshooting Tool。听名字我们就知道这是一块功能很全，很强大的Java运行监视和故障处理工具，并且是官方主力发展的虚拟机故障处理工具，其性能分析比很多专业收费软件都不会逊色多少。



#### ①、启动 JVisualVM

　　和前面介绍的JConsole工具一样，这也是 JDK 自带的一个工具，在安装目录bin下，可以直接双击启动。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828225248185-19744372.png)

　　打开界面如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828225342040-1087511074.png)



#### ②、监控界面介绍

　　其实大体界面和JConsole差不多。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828231322744-1919723573.png)

　　抽样器可以对CPU，内存进行详细监控统计。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828231505272-1409652465.png)



#### ③、插件机制

　　JVisualVM 比较强大的地方在与可以安装各种插件，提供各种不同的功能。

　　点击上方菜单栏 工具---》插件：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828231700619-1466911609.png)

　　然后设置插件中心的地址：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828231955376-31554328.png)

　　这个地址，我们可以到这个网址上去获取：

　　https://visualvm.github.io/pluginscenters.html

　　选择对应的插件地址时，要根据我们的JDK版本来选定。

　　比如，我这边的JDK版本如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828232144288-1383607983.png)

　　那么选择的地址如下（152，介于131-221之间）：

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828232210598-144859154.png)

　　设置好下载地址后，我们这边选择需要的插件，点击安装即可！比如比较常用的插件 Visual GC(用来查看GC日志)

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190828232336096-1858435117.png)

 　安装完成之后，我们便可以在页签上看到这个新增的插件。

　　![img](https://img2018.cnblogs.com/blog/1120165/201908/1120165-20190829073409982-422961402.png)



#### ④、配置远程连接

　　不管是远程连接Tomcat还是jar包，都和介绍JConsole一模一样，详情请参考上面的配置。



#### ⑤、使用文档

　　对于JVisualvm，官方有详细的中文文档说明，如下：

　　https://visualvm.github.io/documentation.html





# [类文件结构](https://www.cnblogs.com/ysocean/p/11427535.html)



**目录**

- 1、Java虚拟机的两个特性
  - [①、语言无关性](https://www.cnblogs.com/ysocean/p/11427535.html#_label0_0)
  - [②、平台无关性](https://www.cnblogs.com/ysocean/p/11427535.html#_label0_1)
- 2、class 字节码文件介绍
  - [①、字节码文件](https://www.cnblogs.com/ysocean/p/11427535.html#_label1_0)
  - [②、javap 命令](https://www.cnblogs.com/ysocean/p/11427535.html#_label1_1)
- [3、无符号数和表](https://www.cnblogs.com/ysocean/p/11427535.html#_label2)
- [4、魔数](https://www.cnblogs.com/ysocean/p/11427535.html#_label3)
- [5、Class 文件的版本号](https://www.cnblogs.com/ysocean/p/11427535.html#_label4)
- 6、常量池
  - [①、常量池容量计数值](https://www.cnblogs.com/ysocean/p/11427535.html#_label5_0)
  - [②、常量池内容](https://www.cnblogs.com/ysocean/p/11427535.html#_label5_1)
- [7、访问标志](https://www.cnblogs.com/ysocean/p/11427535.html#_label6)
- [8、类索引、父类索引和接口索引集合](https://www.cnblogs.com/ysocean/p/11427535.html#_label7)
- [9、字段表集合](https://www.cnblogs.com/ysocean/p/11427535.html#_label8)
- [10、方法表集合](https://www.cnblogs.com/ysocean/p/11427535.html#_label9)
- [11、属性表集合](https://www.cnblogs.com/ysocean/p/11427535.html#_label10)

 

------

　　我们知道计算机是由晶体管、电路板等组装而成的电子设备，而这些电子设备其实只能识别0与1的信号。

　　那么问题来了，我们在操作系统上编写的Java代码（由字母、数字等各种符号组成），打包后部署到服务器上，是如何被计算机所识别并运行的呢？另外，操作系统有很多种，包括Windows系统，Linux系统，Mac OS系统等，而我们同样的Java代码，却可以不做任何处理在不同的系统上正常运行，这又是为啥呢？

　　带着这些疑问，你将会在下面的介绍中得到答案！！！

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 1、Java虚拟机的两个特性

　　在此系列博客[第一篇文章](https://www.cnblogs.com/ysocean/p/9345787.html)中，我们介绍到Java虚拟机的两个特性。



#### ①、语言无关性

　　对于Java语言，我们通过编辑器编写的Java代码，后缀一般是.java。通过javac编译器编译后，会变成.class结尾的字节码文件，只有编译后的.class文件，才能在Java虚拟机上运行。（解压部署在服务器上的jar包，全是编译后的class文件）

　　再比如对于 JRuby 语言，通过编辑器编写的代码后缀是.rb。通过jrubyc 编译器编译后，也会变成 .class 结尾的字节码文件，然后也能在Java虚拟机上运行。

　　在比如已正式成为Android官方支持开发语言的Kotlin。也可以编译成.class字节码文件，然后在虚拟机上运行。

　　我们可以用下面这幅图来表示：

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190902214020398-2130490371.png)

 

 　也就是说，不管你是什么语言，只要能通过某种手段生成合乎规范的.class字节码文件，其实就可以在Java虚拟机上运行，这就是语言无关性。



#### ②、平台无关性

　　Write once, run everywhere（一次编写，到处运行）这是Java语言诞生之处就宣传的一个口号。Java语言之所以能够跨平台运行，其实就是因为Java虚拟机对各个平台的适配，在不同的系统下安装不同的Java虚拟机，我们程序当然能够在不同的系统上运行。

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190902214614261-1530667755.png)

 

 　对于文章开头提出的问题，同样的程序能够在不同的系统上正常运行的原因，就是因为我们在不同的系统上安装了不同的Java虚拟机。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 2、class 字节码文件介绍

　　搞清楚了Java代码的跨平台原理，我们接着来介绍为什么编写的Java代码能够被计算机所识别。



#### ①、字节码文件

　　这其实是上面所说的语言无关性这个特性重要文件——class字节码文件的功劳。

　　Java所有的指令大概有 200 个左右，一个字节（8位）可以存储 256 种不同的信息，我们将一个这样的字节称为字节码（ByteCode）。

　　而 class 文件便是一组以 8 位字节为基础单位流的二进制流，各个数据项目严格按照顺序紧凑地排列在 class 文件之中，中间没有添加任何分隔符，所以整个class 文件中存储的内容几乎都是程序运行的必要数据，没有任何冗余。当遇到需要占用 8 位字节以上空间的数据项时，则会按照高位在前的方式分割成若干个 8 位字节进行存储。

　　比如，对于如下这段代码：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 /**
 2  * Create by YSOcean
 3  */
 4 public class ClassTest {
 5     private static int i = 0;
 6 
 7     public static void main(String[] args) {
 8         System.out.println(i);
 9     }
10 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　我们将生成的class 文件，通过十六进制编辑器打开（在IDEA中，可以下载HexView插件，安装完成后，选择这个class文件，右键 HexView）

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190903214808635-2060178663.png)

 

 　打开后的文件如下：（下面的介绍也都是以这张图为例）

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190903214850020-763386845.png)

 　下面我们会介绍这些十六进制分别代表什么意思。



#### ②、javap 命令

　　另外，为了更好的查看 Class 文件字节码结构，JDK 还为我们提供了一个命令行工具 javap。使用语法如下：

```
javap <options> <classes>
```

　　通过 javap -help 命令，可以查看相关参数作用：

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190909223213546-1910047276.png)

 　我们将 ClassTest.class 文件，通过 javap -v ClassTest.class 命令，执行后如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190909223340407-810991171.png)

 　这些内容下面也会详细介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 3、无符号数和表

　　在介绍这些十六进制之前，我们先介绍 Class 文件的数据类型。

　　Class 文件采用一种类似于 C 语言结构体的伪结构来存储，这种伪结构只有两种数据类型：**无符号数和表**。

**①、无符号数**

　　这是一种基本数据类型，以 u1,u2,u4,u8 来分别代表 1个字节、2个字节、4个字节、8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或按照 UTF-8 编码构成的字符串值。

**②、表**

　　表是由多个无符号数或其它表作为数据项所构成的复合数据类型，所有表都习惯行的以“_info”结尾。表用于描述有层次关系的复合结构数据。

　　整个 Class 文件本质上就是一张表，结构如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190905081737355-432037429.png)

 

 　PS：需要说明的是，由于 Class 文件结构没有任何分隔符，所以无论是每个数据项的的顺序还是数量，都是严格限定的，哪个字节代表什么含义，长度多少，先后顺序如何，都是不允许改变的。

　　下面，我们就来分别介绍这些数据项代表什么含义。　　

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 4、魔数

　　每个 class 文件的头 4 个字节称为魔数（Magic Number），它的唯一作用是：标识该文件是一个Java类文件。如果没有识别到该标志，则说明该文件不是Java类文件或者文件已受损。

　　由上图，我们可以看到前 4 个字节是 **cafe babe**。这是 Gosling 定义的一个魔法数，意思是 Coffee Baby。

　　其实很多文件存储标准中都使用魔数进行身份识别，比如图片gif或者jpeg，使用魔数而不是使用扩展名来进行识别主要是基于安全考虑，因为文件扩展名可以任意的改动。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 5、Class 文件的版本号

　　紧随魔数的 4 个字节存储的是 class 文件的版本号：第 5 和第 6 个字节是次版本号（Minor Version），第 7 和第 8 个字节是主版本号（Major Version）。

　　Java的版本号是从 45 开始的，JDK1.1 之后的每个 JDK 大版本发布主版本号向上加1（JDK1.0~JDK1.1使用了45.0~45.3的版本号），高版本的 JDK 能向下兼容以前版本的 Class 文件，但不能运行以后版本的 Class 文件，即使文件格式未发生变化。

　　上图第5、6、7、8个字节为 00 00 00 34。其十进制值为 52，是JDK8的内部版本号。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 6、常量池

　　紧随主版本号的是常量池入口，是class文件中第一个出现的表类型数据项目，也是占用Class文件空间最大的项目之一，更是Class文件结构中与其它项目关联最多的数据类型。



#### ①、常量池容量计数值

　　因为常量池中常量的数量是不固定的，所以在常量池的入口要放置一项 u2 类型的数据，代表常量池容量计数值（constant_pool_count）。

　　PS：注意，常量池容量计数值是从 1 开始的，而不是从 0 开始。将 0 空出来，是为了满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的意思。

　　Class 文件结构中，只有常量池的容量是从 1 开始的，其它的集合类型，都是从 0 开始的。

　　看上图的十六进制文件，常量池容量计数值为：0x0025，即十进制 37。这就表示常量池中有 36 项常量，索引值分别为 1~36(通过上面javap命令生成字节码文件可以很明显看出来有36个)

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190909221951117-1133475480.png)



#### ②、常量池内容

　　常量池主要存放两大类常量：

　　1、字面量（Literal）：字面量比较接近于 Java 语言层面的常量概念，比如 文本字符串、被声明为 final 的常量值等。

　　2、符号引用（Symbolic References）：符号引用属于编译原理方面的概念，包括下面三类常量：

　　　　类和接口的权限定名（Fully Qualified Name）

　　　　字段的名称和描述符（Descriptor）

　　　　方法的名称和描述符。

　　需要说明的是，Java代码在进行javac 编译的时候，并不像 C 和 C++ 那样有“连接”这一步骤，而是在虚拟机加载 Class 文件的时候进行动态连接。

　　也就是说，在 Class 文件中不会保存各个方法和字段的最终内存布局信息，因此这些字段和方法的符号引用不经过转换的话是无法被虚拟机使用的。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析并翻译到具体的内存地址之中。关于类的创建和动态连接的内容，下篇博客会详细介绍。

　　常量池中的每一项内容都是一个表，在JDK1.8中共有 14 种结构各不相同的表结构数据，每个表结构第一位是一个 u1 类型的标志位（tag，取值为1 到 18，缺少标志为 2、13、14、17 的数据类型）。代表当前这个常量属于哪种常量类型。

　　14 种常量类型所代表的具体含义如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190910074838462-1324015738.png)

 　接着看十六进制文件，紧跟常量池数量的十六进制是0x0a，这是一个标志位，0x0a的十进制数是10，查看常量池的项目表接口，表示的类型是 CONSTANT_Methodref_info。

 　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190924224345474-1115190622.png)

 

 　也就是说，接下来的u2类型0x0006，其十进制值为6，紧跟后面的u2类型十六进制为0x0017，其十进制值为23，这都是两个索引值，分别指向第索引值为6的常量和索引值为23的常量。

　　整个十六进制字节码就不一一进行推导了，下面是各个数据类型的结构：

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190924225521565-1815207971.png)

 

 　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190924225537140-277540503.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 7、访问标志

 　常量池结束后的两个字节表示访问标志（access_flags），这个标识用于识别一些类或接口层次的访问信息。

　　包括：这个 Class 是类还是接口；是否定义为 public 类型，是否定义为 abstract 类型；如果是类的话，是否被声明为 final 等。

　　具体的标志位及标志含义如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201909/1120165-20190924225921233-549353168.png)

 

 　上表定义了 8 个标志位，但是我们说访问标志是一个 u2 类型，一共有 32 个标志位可以使用，没有定义的标志位一律为 0 。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 8、类索引、父类索引和接口索引集合

　　类索引、父类索引和接口索引按顺序排列在访问标志之后。

　　类索引：用于确定这个类的全限类名 ，是一个 u2 类型的数据。

　　父类索引：用于确定这个类的父类全限类名，也是一个 u2 类型的数据。因为Java是单继承的，除了 java.lang.Object 类以外，所有的类都有父类。所以，除了Object 类以外，所有Java类的父类索引都不为0.

　　接口索引：用于描述这个类实现了哪些接口，是一组 u2 类型的数据集合，第一项为 u2 类型的接口计数器，表示实现接口的个数。如果没有实现任何接口，则为0。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 9、字段表集合

 　字段表（field_info）：描述接口或类中声明的变量。（不包括方法内部声明的变量）

　　描述的信息包括：

　　①、字段的作用域（public，protected，private修饰）

　　②、是类级变量还是实例级变量（static修饰）

　　③、是否可变（final修饰）

　　④、并发可见性（volatile修饰，是否强制从主从读写）

　　⑤、是否可序列化（transient修饰）

　　⑥、字段数据类型（8种基本数据类型，对象，数组等引用类型）

　　⑦、字段名称

　　前面5个修饰符，都是布尔值，用标志位来表示；后面两个字段名称和类型，是无法固定的，只能引用常量池中的常量来表示。

　　![img](https://img2018.cnblogs.com/blog/1120165/201910/1120165-20191016204550433-1527259240.png)

 

 　access_flags 是一个 u2 类型，表示各种修饰符。

　　![img](https://img2018.cnblogs.com/blog/1120165/201910/1120165-20191016204712857-582810092.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 10、方法表集合

　　Class 文件存储格式中对方法的描述和字段的描述基本上是一致的。也是依次包括：

　　访问标志（access_flags）、名称索引（name_index）、描述符索引（descriptor_index）、属性表集合数量（attributes_count）、属性表集合（attributes）

　　![img](https://img2018.cnblogs.com/blog/1120165/201910/1120165-20191016205805437-1637256926.png)

 

 

　　方法访问标志如下（access_flags）：

　　![img](https://img2018.cnblogs.com/blog/1120165/201910/1120165-20191016205843354-2103960452.png)

 

[回到顶部](https://www.cnblogs.com/ysocean/p/11427535.html#_labelTop)

### 11、属性表集合

　　在前面介绍的字段表集合、方法表集合中都包括了属性表集合（attributes），其实就是引用的这里。

　　根据《Java虚拟机规范第二版》中，预定义了 9 项虚拟机实现应当能够识别的属性。

　　![img](https://img2018.cnblogs.com/blog/1120165/201910/1120165-20191016210231595-1119218249.png)

 

 　对于每一个属性，它的名称要从常量池中引用一个 CONSTANT_Utf8_info 类型的常量来表示，其属性值的结构则是完全自定义的，只需要说明属性值所占用的位数长度即可。

　　![img](https://img2018.cnblogs.com/blog/1120165/201910/1120165-20191016210437172-1302665450.png) 

参考文档：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4　　



# [类加载过程](https://www.cnblogs.com/ysocean/p/11427536.html)



**目录**

- [1、类的生命周期](https://www.cnblogs.com/ysocean/p/11427536.html#_label0)
- [2、加载](https://www.cnblogs.com/ysocean/p/11427536.html#_label1)
- 3、验证
  - [①、文件格式验证](https://www.cnblogs.com/ysocean/p/11427536.html#_label2_0)
  - [②、元数据验证](https://www.cnblogs.com/ysocean/p/11427536.html#_label2_1)
  - [③、字节码验证](https://www.cnblogs.com/ysocean/p/11427536.html#_label2_2)
  - [④、符号引用验证](https://www.cnblogs.com/ysocean/p/11427536.html#_label2_3)
- [4、准备](https://www.cnblogs.com/ysocean/p/11427536.html#_label3)
- [5、解析](https://www.cnblogs.com/ysocean/p/11427536.html#_label4)
- [6、初始化](https://www.cnblogs.com/ysocean/p/11427536.html#_label5)

 

------

　　在上一篇文章中，我们详细的介绍了Java[类文件结构](https://www.cnblogs.com/ysocean/p/11427535.html)，那么这些Class文件是如何被加载到内存，由虚拟机来直接使用的呢？这就是本篇博客将要介绍的——类加载过程。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427536.html#_labelTop)

### 1、类的生命周期

　　类从被加载到虚拟机内存开始，到卸载出内存为止，其声明周期流程如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191106221645128-1364036120.png)

　　上图中红色的5个部分（加载、验证、准备、初始化、卸载）顺序是确定的，也就是说，类的加载过程必须按照这种顺序按部就班的开始。这里的“开始”不是按部就班的“进行”或者“完成”，因为这些阶段通常是互相交叉混合的进行的，通常会在一个阶段执行过程中调用另一个阶段。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427536.html#_labelTop)

### 2、加载

　　“加载”阶段是“类加载”生命周期的第一个阶段。在加载阶段，虚拟机要完成下面三件事：

　　①、通过一个类的全限定名来获取定义此类的二进制字节流。

　　②、将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。

　　③、在Java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口。

　　PS：类的全限定名可以理解为这个类存放的绝对路径。方法区是JDK1.7以前定义的运行时数据区，而在JDK1.8以后改为元数据区（Metaspace），主要用于存放被Java虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。详情可以参考这边该系列的第二篇文章——[运行时内存结构](https://www.cnblogs.com/ysocean/p/9345622.html)。

　　另外，我们看第一点——通过类的权限定名来获取定义此类的二进制流，这里并没有明确指明要从哪里获取以及怎样获取，也就是说并没有明确规定一定要我们从一个 Class 文件中获取。基于此，在Java的发展过程中，充满创造力的开发人员在这个舞台上玩出了各种花样：

　　1、从 ZIP 包中读取。这称为后面的 JAR、EAR、WAR 格式的基础。

　　2、从网络中获取。比较典型的应用就是 Applet。

　　3、运行时计算生成。这就是动态代理技术。

　　4、由其它文件生成。比如 JSP 应用。

　　5、从数据库中读取。

　　加载阶段完成后，虚拟机外部的二进制字节流就按照虚拟机所需的格式存储在方法区中，然后在Java堆中实例化一个 java.lang.Class 类的对象，这个对象将作为程序访问方法区中这些类型数据的外部接口。

　　注意，加载阶段与连接阶段的部分内容（如一部分字节码文件的格式校验）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始了。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427536.html#_labelTop)

### 3、验证

　　验证是连接阶段的第一步，作用是为了**确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。**

　　我们说Java语言本身是相对安全，因为编译器的存在，纯粹的Java代码要访问数组边界外的数据、跳转到不存在的代码行之类的，是要被编译器拒绝的。但是前面我们也说过，Class 文件不一定非要从Java源码编译过来，可以使用任何途径，包括你很牛逼，直接用十六进制编辑器来编写 Class 文件。

　　所以，如果虚拟机不检查输入的字节流，将会载入有害的字节流而导致系统崩溃。但是虚拟机规范对于检查哪些方面，何时检查，怎么检查都没有明确的规定，不同的虚拟机实现方式可能都会有所不同，但是大致都会完成下面四个方面的检查。



#### ①、文件格式验证

　　校验字节流是否符合Class文件格式的规范，并且能够被当前版本的虚拟机处理。

　　一、是否以魔数 0xCAFEBABE 开头。

　　二、主、次版本号是否是当前虚拟机处理范围之内。

　　三、常量池的常量中是否有不被支持的常量类型（检查常量tag标志）

　　四、指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量。

　　五、CONSTANT_Utf8_info 型的常量中是否有不符合 UTF8 编码的数据。

　　六、Class 文件中各个部分及文件本身是否有被删除的或附加的其他信息。

　　以上是一部分校验内容，当然远不止这些。经过这些校验后，字节流才会进入内存的方法区中存储，接下来后面的三个阶段校验都是基于方法区的存储结构进行的。



#### ②、元数据验证

　　第二个阶段主要是对字节码描述的信息进行语义分析，以保证其描述的信息符合Java语言规范要求。

　　一、这个类是否有父类（除了java.lang.Object 类之外，所有的类都应当有父类）。

　　二、这个类的父类是否继承了不允许被继承的类（被final修饰的类）。

　　三、如果这个类不是抽象类，是否实现了其父类或接口之中要求实现的所有普通方法。

　　四、类中的字段、方法是否与父类产生了矛盾（例如覆盖了父类的final字段、或者出现不符合规则的重载）



#### ③、字节码验证

　　第三个阶段字节码验证是整个验证阶段中最复杂的，主要是进行数据流和控制流分析。该阶段将对类的方法进行分析，保证被校验的方法在运行时不会做出危害虚拟机安全的行为。

　　一、保证任意时刻操作数栈中的数据类型与指令代码序列都能配合工作。例如不会出现在操作数栈中放置了一个 int 类型的数据，使用时却按照 long 类型来加载到本地变量表中。

　　二、保证跳转指令不会跳转到方法体以外的字节码指令中。

　　三、保证方法体中的类型转换是有效的。比如把一个子类对象赋值给父类数据类型，这是安全的。但是把父类对象赋值给子类数据类型，甚至赋值给完全不相干的类型，这就是不合法的。



#### ④、符号引用验证

　　符号引用验证主要是对类自身以外（常量池中的各种符号引用）的信息进行匹配性的校验，通常需要校验如下内容：

　　一、符号引用中通过字符串描述的全限定名是否能够找到相应的类。

　　二、在指定类中是否存在符合方法的字段描述符及简单名称所描述的方法和字段。

　　三、符号引用中的类、字段和方法的访问性（private、protected、public、default）是否可以被当前类访问。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427536.html#_labelTop)

### 4、准备

　　准备阶段是正式为**类变量**分配内存并设置**类变量**初始值的阶段，这些内存是在方法区中进行分配。

　　注意：

　　一、上面说的是类变量，也就是被 static 修饰的变量，不包括实例变量。实例变量会在对象实例化时随着对象一起分配在堆中。

　　二、初始值，指的是一些数据类型的默认值。基本的数据类型初始值如下（引用类型的初始值为null）：

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191107230747222-307539212.png)

 

 　比如，定义 public static int value = 123 。那么在准备阶段过后，value 的值是 0 而不是 123,把 value 赋值为123 是在程序被编译后，存放在类的构造器方法之中，是在初始化阶段才会被执行。但是有一种特殊情况，通过final 修饰的属性，比如 定义 public final static int value = 123，那么在准备阶段过后，value 就被赋值为123了。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427536.html#_labelTop)

### 5、解析

　　解析阶段是虚拟机将常量池中的符号引用替换为直接引用的过程。

　　符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义的定位到目标即可。符号引用与虚拟机实现的内存布局无关，引用的目标不一定已经加载到内存中。

　　直接引用（Direct References）：直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。直接引用是与虚拟机实现内存布局相关的，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那么引用的目标必定已经在内存中存在。

　　解析动作主要针对类或接口、字段、类方法、接口方法四类符号引用，分别对应于常量池的 CONSTANT_Class_info、CONSTANT_Fieldref_info、CONSTANT_Methodref_info、CONSTANTS_InterfaceMethodref_info四种类型常量。

[回到顶部](https://www.cnblogs.com/ysocean/p/11427536.html#_labelTop)

### 6、初始化

 　初始化阶段是类加载阶段的最后一步，前面过程中，除第一个加载阶段可以通过用户自定义类加载器参与之外，其余过程都是完全由虚拟机主导和控制。而到了初始化阶段，则开始真正执行类中定义的Java程序代码（或者说是字节码）。

　　在前面介绍的准备阶段中，类变量已经被赋值过初始值了，而初始化阶段，则根据程序员的编码去初始化变量和资源。

　　换句话来说，**初始化阶段是执行类构造器() 方法的过程**。

　　①、<clinit>() 方法 是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块中可以赋值，但是不能访问。

　　比如如下代码会报错：

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191117224046088-277331745.png)

 

 　但是你把第 14 行代码放到 static 静态代码块的上面就不会报错了。或者不改变代码顺序，将第 11 行代码移除，也不会报错。

　　②、<clinit>() 方法与类的构造函数（或者说是实例构造器<init>()方法）不同，它不需要显示的调用父类构造器，虚拟机会保证在子类的<init>()方法执行之前，父类的<init>()方法已经执行完毕。因此虚拟机中第一个被执行的<init>()方法的类肯定是 java.lang.Object。

　　③、由于父类的<clinit>() 方法先执行，所以父类中定义的静态语句块要优先于子类的变量赋值操作。

　　④、<clinit>() 方法对于接口来说并不是必须的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成<clinit>() 方法。

　　⑤、接口中不能使用静态语句块，但仍然有变量初始化的赋值操作，因此接口与类一样都会生成<clinit>() 方法。但接口与类不同的是，执行接口中的<clinit>() 方法不需要先执行父接口的<clinit>() 方法。只有当父接口中定义的变量被使用时，父接口才会被初始化。

　　⑥、接口的实现类在初始化时也一样不会执行接口的<clinit>() 方法。

　　⑦、虚拟机会保证一个类的<clinit>() 方法在多线程环境中被正确的加锁和同步。如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的<clinit>() 方法，其他的线程都需要阻塞等待，直到活动线程执行<clinit>() 方法完毕。如果在一个类的<clinit>() 方法中有很耗时的操作，那么可能造成多个进程的阻塞。

　　比如对于如下代码：

![img](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　运行结果如下：

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191117230014699-485676426.png)

 

 　线程1抢到了执行<clinit>() 方法，但是该方法是一个死循环，线程2将一直阻塞等待。

　　知道了类的初始化过程，那么类的初始化何时被触发呢？JVM大概规定了如下几种情况：

　　①、当虚拟机启动时，初始化用户指定的类。

　　②、当遇到用以新建目标类实例的 new 指令时，初始化 new 指定的目标类。

　　③、当遇到调用静态方法的指令时，初始化该静态方法所在的类。

　　④、当遇到访问静态字段的指令时，初始化该静态字段所在的类。

　　⑤、子类的初始化会触发父类的初始化。

　　⑥、如果一个接口定义了 default 方法，那么直接实现或间接实现该接口的类的初始化，会触发该接口的初始化。

　　⑦、使用反射 API 对某个类进行反射调用时，会初始化这个类。

　　⑧、当初次调用 MethodHandle 实例时，初始化该 MethodHandle 指向的方法所在的类。





# [双亲委派模型](https://www.cnblogs.com/ysocean/p/11878803.html)



**目录**

- [1、类加载器](https://www.cnblogs.com/ysocean/p/11878803.html#_label0)
- [2、双亲委派模型](https://www.cnblogs.com/ysocean/p/11878803.html#_label1)
- [3、双亲委派模型实现源码](https://www.cnblogs.com/ysocean/p/11878803.html#_label2)
- [4、自定义类加载器](https://www.cnblogs.com/ysocean/p/11878803.html#_label3)

 

------

　　在上一篇博客，我们介绍了[类加载过程](https://www.cnblogs.com/ysocean/p/11427536.html)，包括5个阶段，分别是“加载”，“验证”，“准备”，“解析”，“初始化”，如下图所示：

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191120204213326-1178311294.png)

 

　　本篇博客，我们来介绍Java虚拟机的双亲委派模型，在介绍之前，我先抛出一个问题：

　　我们知道，在JDK源码中，有各种Java自带的类，比如java.lang.String，java.util.List等，那么我们自己的项目中，能够写一个命名为java.lang.String.java 等JDK源码中存在的类，并且在项目中使用吗？

[回到顶部](https://www.cnblogs.com/ysocean/p/11878803.html#_labelTop)

### 1、类加载器

　　什么是类加载器？上篇博客我们介绍类加载过程中的第一个阶段——加载，作用是“通过一个类的全限定名来获取描述此类的二进制流”，那么这个加载过程就是由类加载器来完成的。

　　从Java虚拟机的角度出发，只存在两种不同的类加载器，一种是启动类加载器（Bootstrap ClassLoader），这个类加载器使用 C++ 语言实现，是虚拟机自身的一部分；另一种是所有其它的类加载器，这些类加载器都是由Java语言实现的。但是从Java开发人员的角度来看，类加载器可以细分为如下四种：

**①、启动类加载器（Bootstrap ClassLoader）**

　　负责将存放在 **/lib** 目录中的，或者被**-Xbootclasspath** 参数所指定的路径中的，并且是虚拟机按照文件名识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。**
**　　启动类加载器无法被Java程序直接引用。**
**

　　JDK 中的源码类大都是由启动类加载器加载，比如前面说的 java.lang.String，java.util.List等，需要注意的是，启动类 main Class 也是由启动类加载器加载。

**②、扩展类加载器（Extension ClassLoader）**

 　这个类加载器由 **sun.misc.Launcher$ExtClassLoader** 实现，负责加载**＜JAVA_HOME＞/lib/ext** 目录中的，或者被 **java.ext.dirs** 系统变量所指定的路径中的所有类库。

　　开发者可以直接使用扩展类加载器。

**③、应用程序类加载器（Application ClassLoader）**

　　由 **sun.misc.Launcher$AppClassLoader** 实现。由于这个类加载器是 ClassLoader.getSystemClassLoader() 方法的返回值，所以一般也称它为系统类加载器。

　　它负责加载用户类路径ClassPath上所指定的类库，开发者可以直接使用这个类加载器。如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

 　通常项目中自定义的类，都会放在类路径下，由应用程序类加载器加载。

**④、自定义类加载器（User ClassLoader）**

 　这是由用户自己定义的类加载器，一般情况下我们不会自定义类加载器，但有些特殊情况，比如JDBC能够通过连接各种不同的数据库就是自定义类加载器来实现的，具体用处会在后文详细介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/11878803.html#_labelTop)

### 2、双亲委派模型

　　回到文章开头提出的问题，如果有不法分子在你项目中构造了一个java.lang.String类，并在该类中植入了一些不良代码，但你自己浑然不知，以为使用的String类还是 rt.jar 包下的，那可能会给你系统造成不良的影响。

　　聪明的Java虚拟机实现者也想到了这个问题，于是，他们引入了 双亲委派模型来解决这个问题。

　　下面是双亲委派模型的加载流程机制：

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191120214652222-1119089831.png)

　　总结来说：**双亲委派机制就是如果一个类加载器收到了类加载请求，它首先不会自己尝试去加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有父类加载器反馈到无法完成这个加载请求（它的搜索范围没有找到这个类），子加载器才会尝试自己去加载。**

　　其实，这里叫双亲委派可能有点不妥，因为按道理来讲只有父加载器，这里的“双亲”是“parents”的直译，并不表示汉语中的父母双亲。另外，这里的父加载器也不是继承的关系。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 /**
 2  * Create by YSOcean
 3  */
 4 public class ClassLoadTest {
 5     public static void main(String[] args) {
 6         ClassLoader classLoader1 = ClassLoadTest.class.getClassLoader();
 7         ClassLoader classLoader2 = classLoader1.getParent();
 8         ClassLoader classLoader3 = classLoader2.getParent();
 9         System.out.println(classLoader1);
10         System.out.println(classLoader2);
11         System.out.println(classLoader3);
12     }
13 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　输出为：

　　![img](https://img2018.cnblogs.com/blog/1120165/201911/1120165-20191121212405623-1118554825.png)

　　那么知道了什么是双亲委派机制，双亲委派机制有什么好处呢?

　　回到上面提出的问题，如果你自定义了一个 java.lang.String类，你会发现这个自定义的String.java**可以正常编译，但是永远无法被加载运行**。因为加载这个类的加载器，会一层一层的往上推，最终由启动类加载器来加载，而启动类加载的会是源码包下的String类，不是你自定义的String类。

[回到顶部](https://www.cnblogs.com/ysocean/p/11878803.html#_labelTop)

### 3、双亲委派模型实现源码

　　可以打开 java.lang.ClassLoader 类，其 loadClass方法如下：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     protected Class<?> loadClass(String name, boolean resolve)
 2         throws ClassNotFoundException
 3     {
 4         synchronized (getClassLoadingLock(name)) {
 5             // First, check if the class has already been loaded
 6             Class<?> c = findLoadedClass(name);
 7             if (c == null) {
 8                 long t0 = System.nanoTime();
 9                 try {
10                     if (parent != null) {
11                         c = parent.loadClass(name, false);
12                     } else {
13                         c = findBootstrapClassOrNull(name);
14                     }
15                 } catch (ClassNotFoundException e) {
16                     // ClassNotFoundException thrown if class not found
17                     // from the non-null parent class loader
18                 }
19 
20                 if (c == null) {
21                     // If still not found, then invoke findClass in order
22                     // to find the class.
23                     long t1 = System.nanoTime();
24                     c = findClass(name);
25 
26                     // this is the defining class loader; record the stats
27                     sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
28                     sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
29                     sun.misc.PerfCounter.getFindClasses().increment();
30                 }
31             }
32             if (resolve) {
33                 resolveClass(c);
34             }
35             return c;
36         }
37     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　实现方式很简单，首先会检查该类是否已经被加载过了，若加载过了直接返回（默认resolve取false）；若没有被加载，则调用父类加载器的 loadClass方法，若父类加载器为空则默认使用启动类加载器作为父加载器。如果父类加载失败，则在抛出 ClassNotFoundException 异常后，在调用自己的 findClass 方法进行加载。

[回到顶部](https://www.cnblogs.com/ysocean/p/11878803.html#_labelTop)

### 4、自定义类加载器

　　先说说我们为什么要自定义类加载器？

**①、加密**

　　我们知道Java字节码是可以进行反编译的，在某些安全性高的场景，是不允许这种情况发生的。那么我们可以将编译后的代码用某种加密算法进行加密，加密后的文件就不能再用常规的类加载器去加载类了。而我们自己可以自定义类加载器在加载的时候先解密，然后在加载。

**②、动态创建**

　　比如很有名的动态代理。

**③、从非标准的来源加载代码**

　　我们不用非要从class文件中获取定义此类的二进制流，还可以从数据库，从网络中，或者从zip包等。

　　明白了为什么要自定义类加载器，接下来我们再来详述如何自定义类加载器。

　　通过第 3 小节的 java.lang.ClassLoader 类的源码分析，类加载时根据双亲委派模型会先一层层找到父加载器，如果加载失败，则会调用当前加载器的 findClass() 方法来完成加载。因此我们自定义类加载器，有两个步骤：

　　1、**继承 ClassLoader** 

　　**2、覆写 findClass() 方法**