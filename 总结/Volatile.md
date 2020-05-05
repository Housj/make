​	Java内存模型简称JMM（Java Memory Model），是Java虚拟机所定义的⼀种抽象规范，⽤来屏蔽不同硬件和操作系统的内存访问差异，让java程序在各种平台下都能达到⼀致的内存访问效果。

Java内存模型需要解释⼏个概念：

# 1.主内存（Main Memory）

主内存可以简单理解为计算机当中的内存，但⼜不完全等同。主内存被所有的线程所享，对于⼀个共享变量（⽐如静态变量，或是堆内存中的实例）来说，主内存当中存储它的“本尊”。

# 2.⼯作内存（Working Memory）

​	⼯作内存可以简单理解为计算机当中的CPU⾼速缓存，但⼜不完全等同。每⼀个线程拥有⾃⼰的⼯作内存，对于⼀个共享变量来说，⼯作内存当中存储了它的“副本”。

​	线程对共享变量的所有操作都必须在⼯作内存进⾏，不能直接读写主内存中的变量。不同线程之间也⽆法访问彼此的⼯作内存，变量值的传递只能通过主内存来进⾏。以上说的这些可能有点抽象，⼤家来看看下⾯这个例⼦：

对于⼀个静态变量

static int s = 0；

线程A执⾏如下代码：

s = 3；

那么，JMM的⼯作流程如下图所示：通过⼀系列内存读写的操作指令（JVM内存模型共定义了8种内存操作指令，以后会细讲），线程A把静态变量 s=0 从主内存读到⼯作内存，再把 s=3 的更新结果同步到主内存当中。从单线程的⻆度来看，这个过程没有任何问题。这时候我们引⼊线程B，执⾏如下代码：

System.out.println("s=" + s);引⼊线程B以后，当线程A⾸先执⾏，更⼤的可能是出现下⾯情况：此时线程B从主内存得到的s值是3，理所当然输出 s=3，这种情况不难理解。但是，有较⼩的⼏率出现另⼀种情况：因为⼯作内存所更新的变量并不会⽴即同步到主内存，所以虽然线程A在⼯作内存当中已经把变量s的值更新成3，但是线程B从主内存得到的变量s的值仍然是0，从⽽输出 s=0。

# 保证可见性

volatile关键字具有许多特性，其中最重要的特性就是保证了⽤volatile修饰的变量对所有线程的可⻅性。这⾥的可⻅性是什么意思呢？当⼀个线程修改了变量的值，新的值会⽴刻同步到主内存当中。⽽其他线程读取这个变量的时候，也会从主内存中拉取最新的变量值。

为什么volatile关键字可以有这样的特性？这得益于java语⾔的先⾏发⽣原则（happensbefore）。先⾏发⽣原则在维基百科上的定义如下：

​	在计算机科学中，先⾏发⽣原则是两个事件的结果之间的关系，如果⼀个事件发⽣在另⼀个事件之前，结果必须反映，即使这些事件实际上是乱序执⾏的（通常是优化程序流程）。

​	这⾥所谓的事件，实际上就是各种指令操作，⽐如读操作、写操作、初始化操作、锁操作等等。

先⾏发⽣原则作⽤于很多场景下，包括同步锁、线程启动、线程终⽌、volatile。我们这⾥只列举出volatile相关的规则：

对于⼀个volatile变量的写操作先⾏发⽣于后⾯对这个变量的读操作。

回到上述的代码例⼦，如果在静态变量s之前加上volatile修饰符：

volatile static int s = 0；线程A执⾏如下代码：

s = 3；

这时候我们引⼊线程B，执⾏如下代码：

System.out.println("s=" + s);

当线程A先执⾏的时候，把s = 3写⼊主内存的事件必定会先于读取s的事件。所以线程B的输出⼀定是s = 3。



# 不保证原子性

开启10个线程，每个线程当中让静态变量count⾃增100次。执⾏之后会发现，最终count的结果值未必是1000，有可能⼩于1000。使⽤volatile修饰的变量，为什么并发⾃增的时候会出现这样的问题呢？这是因为count++这⼀⾏代码本身并不是原⼦性操作，在字节码层⾯可以拆分成如下指令：

getstatic //读取静态变量（count）

iconst_1 //定义常量1

iadd //count增加1

putstatic //把count结果同步到主内存

虽然每⼀次执⾏ getstatic 的时候，获取到的都是主内存的最新变量值，但是进⾏iadd的时候，由于并不是原⼦性操作，其他线程在这过程中很可能让count⾃增了很多次。这样⼀来本线程所计算更新的是⼀个陈旧的count值，⾃然⽆法做到线程安全。

# 什么时候适合⽤volatile呢？

1.`运⾏结果并不依赖变量的当前值`，或者能够确保只有单⼀的线程修改变量的值。

2.变量`不需要与其他的状态变量共同参与不变约束`。

第⼀条很好理解，就是上⾯的代码例⼦。第⼆条是什么意思呢？可以看看下⾯这个场景：

```java
volatile static int start = 3;
volatile static int end = 6;

//线程A执⾏如下代码：
while (start < end){
//do something
}

//线程B执⾏如下代码：
start+=3;
end+=3;
```

​	这种情况下，⼀旦在线程A的循环中执⾏了线程B，start有可能先更新成6，造成了⼀瞬间start == end，从⽽跳出while循环的可能性。什么是指令重排？

# 禁止指令重排

​	指令重排是指JVM在编译Java代码的时候，或者CPU在执⾏JVM字节码的时候，对现有的指令顺序进⾏重新排序。

​	指令重排的⽬的是为了在不改变程序执⾏结果的前提下，优化程序的运⾏效率。需要注意的是，这⾥所说的不改变执⾏结果，指的是不改变单线程下的程序执⾏结果。

然⽽，指令重排是⼀把双刃剑，虽然优化了程序的执⾏效率，但是在某些情况下，会影响到多线程的执⾏结果。我们来看看下⾯的例⼦：

```java
boolean contextReady = false;
在线程A中执⾏:context = loadContext();
contextReady = true;
在线程B中执⾏:
while( ! contextReady ){
sleep(200);
}

doAfterContextReady (context);
```

以上程序看似没有问题。线程B循环等待上下⽂context的加载，⼀旦context加载完成，

contextReady == true的时候，才执⾏doA!erContextReady ⽅法。

但是，如果线程A执⾏的代码发⽣了指令重排，初始化和contextReady的赋值交换了顺序：

```java
boolean contextReady = false;

//在线程A中执⾏:

contextReady = true;

context = loadContext();

//在线程B中执⾏:

while( ! contextReady ){

sleep(200);

}
```

doAfterContextReady (context);这个时候，很可能context对象还没有加载完成，变量contextReady 已经为true，线程B直接跳出了循环等待，开始执⾏doA!erContextReady ⽅法，结果⾃然会出现错误。

需要注意的是，这⾥java代码的重排只是为了简单示意，真正的指令重排是在字节码指令的层⾯。什么是内存屏障？

# 内存屏障

内存屏障（Memory Barrier）是⼀种CPU指令，维基百科给出了如下定义：

​	内存屏障也称为内存栅栏或栅栏指令，是⼀种屏障指令，它使CPU或编译器对屏障指令之前和之后发出的内存操作执⾏⼀个排序约束。 这通常意味着在屏障之前发布的操作被保证在屏障之后发布的操作之前执⾏。内存屏障共分为四种类型：

`LoadLoad屏障`：

抽象场景：Load1; LoadLoad; Load2

Load1 和 Load2 代表两条读取指令。在Load2要读取的数据被访问前，保证Load1要读取的数据被读取完毕。

`StoreStore屏障`：

抽象场景：Store1; StoreStore; Store2

Store1 和 Store2代表两条写⼊指令。在Store2写⼊执⾏前，保证Store1的写⼊操作对其它处理器可⻅

`LoadStore屏障`：

抽象场景：Load1; LoadStore; Store2

在Store2被写⼊前，保证Load1要读取的数据被读取完毕。

`StoreLoad屏障`：

抽象场景：Store1; StoreLoad; Load2

在Load2读取操作执⾏前，保证Store1的写⼊对所有处理器可⻅。StoreLoad屏障的开销是四种屏障中最⼤的。volatile做了什么？



在⼀个变量被volatile修饰后，JVM会为我们做两件事：

1.在每个volatile写操作前插⼊StoreStore屏障，在写操作后插⼊StoreLoad屏障。

2.在每个volatile读操作前插⼊LoadLoad屏障，在读操作后插⼊LoadStore屏障。

或许这样说有些抽象，我们看⼀看刚才线程A代码的例⼦：

```
boolean contextReady = false;

在线程A中执⾏:

context = loadContext();

contextReady = true;
```

我们给contextReady 增加volatile修饰符，会带来什么效果呢？

由于加⼊了StoreStore屏障，屏障上⽅的普通写⼊语句 context = loadContext() 和屏障下⽅的volatile写⼊语句 contextReady = true ⽆法交换顺序，从⽽成功阻⽌了指令重排序。

volatile特性之⼀：	

​	保证变量在线程之间的可⻅性。可⻅性的保证是基于CPU的内存屏障指令，被JSR-133抽象为happens-before原则。

volatile特性之⼆：

​	阻⽌编译时和运⾏时的指令重排。编译时JVM编译器遵循内存屏障的约束，运⾏时依靠CPU屏障指令来阻⽌重排。



⼏点补充：

1. 关于volatile的介绍，本⽂很多内容来⾃《深⼊理解Java虚拟机》这本书。有兴趣的同学可以去看看。

2. 在使⽤volatile引⼊内存屏障的时候，普通读、普通写、volatile读、volatile写会排列组合出许多不同的场景。我们这⾥只简单列出了其中⼀种，有兴趣的同学可以查资料进⼀步学习其他阻⽌指令重排的场景。
3. volatile除了保证可⻅性和阻⽌指令重排，还解决了long类型和double类型数据的8字节赋值问题。这个特性相对简单，本⽂就不详细描述了。