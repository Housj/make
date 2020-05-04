# JVM面试题

## Part 1

### 一. Java类加载时机和过程

**类加载时机**
1. 创建类的实例，也就是new一个对象，隐式加载
2. 访问某个类或接口的静态变量，或者对该静态变量赋值
3. 调用类的静态方法
4. 反射（Class.forName("com.lyj.load")）,loaderClass 显示加载
5. 初始化一个类的子类（会首先初始化子类的父类）
6. JVM启动时标明的启动类，即文件名和类名相同的那个类  
   **除此之外，下面几种情形需要特别指出：**
   对于一个final类型的静态变量，如果该变量的值在编译时就可以确定下来，那么这个变量相当于“宏变量”。Java编译器会在编译时直接把这个变量出现的地方替换成它的值，因此即使程序使用该静态变量，也不会导致该类的初始化。反之，如果final类型的静态Field的值不能在编译时确定下来，则必须等到运行时才可以确定该变量的值，如果通过该类来访问它的静态变量，则会导致该类被初始化。

Java类加载需要经历一下7个过程：
1.加载
	加载是类加载的第一个过程，在这个阶段，将完成一下三件事情：
		• 通过一个类的全限定名获取该类的二进制流。
		• 将该二进制流中的静态存储结构转化为方法去运行时数据结构。
		• 在内存中生成该类的Class对象，作为该类的数据访问入口。
2.验证
	验证的目的是为了确保Class文件的字节流中的信息不会危害到虚拟机.在该阶段主要完成以下四种验证:
		•文件格式验证：验证字节流是否符合Class文件的规范，如主次版本号是否在当前虚拟机范围内，常量池中的常量是否有不被支持的类型.
		•元数据验证: 对字节码描述的信息进行语义分析，如这个类是否有父类，是否继承了不被继承的类等。
		•字节码验证：是整个验证过程中最复杂的一个阶段，通过验证数据流和控制流的分析，确定程序语义是否正确，主要针对方法体的验证。如：方法中的类型转换是否正确，跳转指令是否正确等。
		•符号引用验证：这个动作在后面的解析过程中发生，主要是为了确保解析动作能正确执行。
3.准备
	准备阶段是为类的静态变量分配内存并将其初始化为默认值，这些内存都将在方法区中进行分配。准备阶段不分配类中的实例变量的内存，实例变量将会在对象实例化时随着对象一起分配在Java堆中。
	public static int value=123;//在准备阶段value初始值为0初始化阶段才会变为123。
4.解析
	该阶段主要完成符号引用到直接引用的转换动作。解析动作并不一定在初始化动作完成之前，也有可能在初始化之后。
5.初始化
	初始化是类加载的最后一步，前面的类加载过程，除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全由虚拟机主导和控制。到了初始化阶段，才真正开始执行类中定义的Java程序代码。
6.使用
7.卸载

### 二. JVM加载Class文件的原理机制
​	Java语言是一种具有动态性的解释型语言，类（Class）只有被加载到JVM后才能运行。当运行指定程序时，JVM会将编译生成的.class文件按照需求和一定的规则加载到内存中，并组织成为一个完整的Java应用程序。这个加载过程是由类加载器完成，具体来说，就是由ClassLoader和它的子类来实现的。
​	类加载器本身也是一个类，其实质是把类文件从硬盘读取到内存中。
​	类的加载方式分为隐式加载和显示加载。
​		隐式加载指的是程序在使用new等方式创建对象时，会隐式地调用类的加载器把对应的类加载到JVM中。
​		显示加载指的是通过直接调class.forName()方法来把所需的类加载到JVM中。
任何一个工程项目都是由许多类组成的，当程序启动时，只把需要的类加载到JVM中，其他类只有被使用到的时候才会被加载，采用这种方法一方面可以加快加载速度，另一方面可以节约程序运行时对内存的开销。此外，在Java语言中，每个类或接口都对应一个.class文件，这些文件可以被看成是一个个可以被动态加载的单元，因此当只有部分类被修改时，只需要重新编译变化的类即可，而不需要重新编译所有文件，因此加快了编译速度。
在Java语言中，类的加载是动态的，它并不会一次性将所有类全部加载后再运行，而是保证程序运行的基础类（例如基类）完全加载到JVM中，至于其他类，则在需要的时候才加载。
类加载的主要分三大步：
​	•装载。根据查找路径找到相应的class文件，然后导入。
​	•链接。链接又可分为3个小步：
​		•检查，检查待加载的class文件的正确性。
​		•准备，给类中的静态变量分配存储空间。
​		•解析，将符号引用转换为直接引用（这一步可选）
​	•初始化。对静态变量和静态代码块执行初始化工作。

### 三. Java内存分配

•寄存器：我们无法控制
•静态域：static定义的静态成员
•常量池：编译时被确定并保存在.class文件中的（final）常量值和一些文本修饰的符号引用（类和接口的全限定名，字段的名称和描述符，方法和名称和描述符）。
•非RAM存储：硬盘等永久存储空间。
•堆内存：new创建的对象和数组，由Java虚拟机自动垃圾回收器管理,存取速度慢。
•栈内存：基本类型的变量和对象的引用变量（堆内存空间的访问地址），速度快，可以共享，但是大小与生存期必须确定，缺乏灵活性。

Java堆的结构是什么样子的？什么是堆中的永久代（PermGen space）?
	JVM的堆是运行时数据区，所有类的实例和数组都是在堆上分配内存。它在JVM启动的时候被创建。对象所占的堆内存是由自动内存管理系统也就是垃圾收集器回收。
	堆内存是由存活和死亡的对象组成的。存活的对象是应用可以访问的，不会被垃圾回收。死亡的对象是应用不可访问尚且还没有被垃圾收集器回收掉的对象。一直到垃圾收集器把这些对象回收掉之前，他们会一直占据堆内存空间。

### 四.GC是什么?为什么要有GC？

​	GC是垃圾收集的意思（Gabage Collection），内存处理是编程人员容易出现问题的地方，忘记或者错误的内存回收会导致程序或系统的不稳定甚至崩溃，Java提供的GC功能可以自动监测对象是否超过作用域从而达到自动回收内存的目的。

​	Java语言没有提供释放已分配内存的显示操作方法。

### 五.简述Java垃圾回收机制

​	在Java中，程序员是不需要显示的去释放一个对象的内存的，而是由虚拟机自行执行。在JVM中，有一个垃圾回收线程，它是低优先级的，在正常情况下是不会执行的，只有在虚拟机空闲或者当前堆内存不足时，才会触发执行，扫描那些没有被任何引用的对象，并将它们添加到要回收的集合中，进行回收。

### 六.对象是否存活判定方法

判断一个对象是否存活有两种方法：
	1.引用计数法
		所谓引用计数法就是给每一个对象设置一个引用计数器，每当有一个地方引用这个对象时，就将计数器加一，引用失效时，计数器就减一。当一个对象的引用计数器为零时，说明此对象没有被引用，也就是“死对象”,将会被垃圾回收.引用计数法有一个缺陷就是无法解决循环引用问题，也就是说当对象A引用对象B，对象B又引用者对象A，那么此时A、B对象的引用计数器都不为零，也就造成无法完成垃圾回收，所以主流的虚拟机都没有采用这种算法。

​	2.可达性算法（引用链法）

​		该算法的思想是：从一个被称为GCRoots的对象开始向下搜索，如果一个对象到GCRoots没有任何引用链相连时，则说明此对象不可用。
​		在Java中可以作为GCRoots的对象有以下几种：
​			• 虚拟机栈中引用的对象
​			• 方法区类静态属性引用的对象
​			• 方法区常量池引用的对象
​			• 本地方法栈JNI引用的对象
​	虽然这些算法可以判定一个对象是否能被回收，但是当满足上述条件时，一个对象比不一定会被回收。当一个对象不可达GCRoot时，这个对象并不会立马被回收，而是出于一个死缓的阶段，若要被真正的回收需要经历两次标记.
​	如果对象在可达性分析中没有与GCRoot的引用链，那么此时就会被第一次标记并且进行一次筛选，筛选的条件是：是否有必要执行finalize()方法。当对象没有覆盖finalize()方法或者已被虚拟机调用过，那么就认为是没必要的。如果该对象有必要执行finalize()方法，那么这个对象将会放在一个称为F-Queue的对队列中，虚拟机会触发一个Finalize()线程去执行，此线程是低优先级的，并且虚拟机不会承诺一直等待它运行完，这是因为如果finalize()执行缓慢或者发生了死锁，那么就会造成F-Queue队列一直等待，造成了内存回收系统的崩溃。GC对处于F-Queue
中的对象进行第二次被标记，这时，该对象将被移除”即将回收”集合，等待回收。

### 七.垃圾回收的优点和原理。并考虑2种回收机制。

​	Java语言中一个显著的特点就是引入了垃圾回收机制，使C++程序员最头疼的内存管理的问题迎刃而解，它使得Java程序员在编写程序的时候不再需要考虑内存管理。由于有个垃圾回收机制，Java中的对象不再有“作用域”的概念，只有对象的引用才有"作用域"。垃圾回收可以有效的防止内存泄露，有效的使用可以使用的内存。垃圾回收器通常是作为一个单独的低级别的线程运行，不可预知的情况下对内存堆中已经死亡的或者长时间没有使用的对象进行清楚和回收，程序员不能实时的调用垃圾回收器对某个对象或所有对象进行垃圾回收。

​	回收机制有分代复制垃圾回收和标记垃圾回收，增量垃圾回收。

### 八.垃圾回收器的基本原理是什么？有什么办法主动通知虚拟机进行垃圾回收？

​	对于GC来说，当程序员创建对象时，GC就开始监控这个对象的地址、大小以及使用情况。通常，GC采用有向图的方式记录和管理堆（heap）中的所有对象。通过这种方式确定哪些对象是”可达的”，哪些对象是”不可达的”。当GC确定一些对象为“不可达”时，GC就有责任回收这些内存空间。

​	可以。程序员可以手动执行System.gc()，通知GC运行，但是Java语言规范并不保证GC一定会执行。

### 九.Java中会存在内存泄漏吗，请简单描述。

​	所谓内存泄露就是指一个不再被程序使用的对象或变量一直被占据在内存中。Java中有垃圾回收机制，它可以保证对象不再被引用的时候，即对象变成了孤儿的时候，对象将自动被垃圾回收器从内存中清除掉。由于Java使用有向图的方式进行垃圾回收管理，可以消除引用循环的问题，例如有两个对象，相互引用，只要它们和根进程不可达的，那么GC也是可以回收它们的，例如下面的
代码可以看到这种情况的内存回收：

```java
public class GarbageTest{
/**
*@paramargs
*@throwsIOException
*/
public static void main(String[]args)throws IOException
{
//TODOAuto-generatedmethodstub
try{
gcTest();
}catch(IOExceptione){
//TODOAuto-generatedcatchblock
e.printStackTrace();
}
System.out.println("hasexitedgcTest!");
System.in.read();
System.in.read();
System.out.println("outbegingc!");
for(inti=0;i<100;i++)
{
System.gc();
System.in.read();
System.in.read();
}}
private static void gcTest()throws IOException{
System.in.read();
System.in.read();
Person p1=new Person();
System.in.read();
System.in.read();
Person p2=new Person();
p1.setMate(p2);
p2.setMate(p1);
System.out.println("beforeexitgctest!");
System.in.read();
System.in.read();
System.gc();
System.out.println("exitgctest!");
}
private static class Person
{
byte[]data=newbyte[20000000];
Perso nmate=null;
public void setMate(Person other)
{
mate=other;
}}}
```

​	Java中的内存泄露的情况：长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄露，尽管短生命周期对象已经不再需要，但是因为长生命周期对象持有它的引用而导致不能被回收，这就是Java中内存泄露的发生场景，通俗地说，就是程序员可能创建了一个对象，以后一直不再使用这个对象，这个对象却一直被引用，即这个对象无用但是却无法被垃圾回收器回收的，这就是java中可能出现内存泄露的情况，例如，缓存系统，我们加载了一个对象放在缓存中(例如放在一个全局map对象中)，然后一直不再使用它，这个对象一直被缓存引用，但却不再被使用。
​	检查Java中的内存泄露，一定要让程序将各种分支情况都完整执行到程序结束，然后看某个对象是否被使用过，如果没有，则才能判定这个对象属于内存泄露。如果一个外部类的实例对象的方法返回了一个内部类的实例对象，
这个内部类对象被长期引用了，即使那个外部类实例对象不再被使用，但由于内部类持久外部类的实例对象，这个外部类对象将不会被垃圾回收，这也会造成内存泄露。
​	下面内容来自于网上（主要特点就是清空堆栈中的某个元素，并不是彻底把它从数组中拿掉，而是把存储的总数减少，本人写得可以比这个好，在拿掉某个元素时，顺便也让它从数组中消失，将那个元素所在的位置的值设置为null即可）：
我实在想不到比那个堆栈更经典的例子了，以致于我还要引用别人的例子，下面的例子不是我想到的，是书上看到的，当然如果没有在书上看到，可能过一段时间我自己也想的到，可是那时我说是我自己想到的也没有人相信的。

```java
public class Stack{
private Object[] elements = new Object[10];
privateintsize=0;
publicvoidpush(Objecte){
ensureCapacity();
elements[size++]=e;
}
publicObjectpop(){
if(size==0)thrownewEmptyStackException();
returnelements[--size];
}
privatevoidensureCapacity(){
if(elements.length==size){
Object[]oldElements=elements;
elements=newObject[2*

elements.length+1];
System.arraycopy(oldElements,0,elements,0,
size);
}}}
```

​	上面的原理应该很简单，假如堆栈加了10个元素，然后全部弹出来，虽然堆栈是空的，没有我们要的东西，但是这是个对象是无法回收的，这个才符合了内存泄露的两个条件：无用，无法回收。
​	但是就是存在这样的东西也不一定会导致什么样的后果，如果这个堆栈用的比较少，也就浪费了几个K内存而已，反正我们的内存都上G了，哪里会有什么影响，再说这个东西很快就会被回收的，
​	有什么关系。下面看两个例子。

```java
public class Bad{
  public static Stack s=Stack();
    static{
    s.push(newObject());
    s.pop();//这里有一个对象发生内存泄露
    s.push(newObject());//上面的对象可以被回收了，等于是自愈了
  }
}
```

​	因为是static，就一直存在到程序退出，但是我们也可以看到它有自愈功能，就是说如果你的Stack最多有100个对象，那么最多也就只有100个对象无法被回收其实这个应该很容易理解，Stack内部持有100个引用，最坏的情况就是他们都是无用的，因为我们一旦放新的进取，以前的引用自然消失！
​	内存泄露的另外一种情况：当一个对象被存储进HashSet集合中以后，就不能修改这个对象中的那些参与计算哈希值的字段了，否则，对象修改后的哈希值与最初存储进HashSet集合中时的哈希值就不同了，在这种情况下，即使在contains方法使用该对象的当前引用作为的参数去HashSet集合中检索对象，也将返回找不到对象的结果，这也会导致无法从HashSet集合中单独删除当前对象，造成内存泄露。

### 十.深拷贝和浅拷贝。

​	简单来讲就是复制、克隆。
​		Person p=new Person(“张三”);
​	浅拷贝就是对 对象中的数据成员进行简单赋值，如果存在动态成员或者指针就会报错。
​	深拷贝就是对 对象中存在的动态成员或指针重新开辟内存空间。

### 十一.System.gc()和Runtime.gc()会做什么事情？

​	java.lang.System.gc()只是java.lang.Runtime.getRuntime().gc()的简写，两者的行为没有任何不同。

​	这两个方法用来提示JVM要进行垃圾回收。但是，立即开始还是延迟进行垃圾回收是取决于JVM的。

### 十二.finalize()方法什么时候被调用？析构函数(finalization)的目的是什么？

​	垃圾回收器（garbage collector）决定回收某对象时，就会运行该对象的finalize()方法但是在Java中很不幸，如果内存总是充足的，那么垃圾回收可能永远不会进行，也就是说filalize()可能永远不被执行，显然指望它做收尾工作是靠不住的。

​	那么finalize()究竟是做什么的呢？它最主要的用途是回收特殊渠道申请的内存。Java程序有垃圾回收器，所以一般情况下内存问题不用程序员操心。但有一种JNI（JavaNativeInterface）调用non-Java程序（C或C++）finalize()的工作就是回收这部分的内存。

### 十三.如果对象的引用被置为null，垃圾收集器是否会立即释放对象占用的内存？

​	不会，在下一个垃圾回收周期中，这个对象将是可被回收的。

### 十四.什么是分布式垃圾回收（DGC）？它是如何工作的？

​	DGC叫做分布式垃圾回收。RMI使用DGC来做自动垃圾回收。因为RMI包含了跨虚拟机的远程对象的引用，垃圾回收是很困难的。DGC使用引用计数算法来给远程对象提供自动内存管理。

### 十五.串行（serial）收集器和吞吐量（throughput）收集器的区别是什么？

​	吞吐量收集器使用并行版本的新生代垃圾收集器，它用于中等规模和大规模数据的应用程序。而串行收集器对大多数的小应用（在现代处理器上需要大概100M左右的内存）就足够了。

### 十六.在Java中，对象什么时候可以被垃圾回收？

​	当对象对当前使用这个对象的应用程序变得不可触及的时候，这个对象就可以被回收了。

### 十七.简述Java内存分配与回收策率以及MinorGC和MajorGC。

​	• 对象优先在堆的Eden区分配
​	• 大对象直接进入老年代
​	• 长期存活的对象将直接进入老年代
​	当Eden区没有足够的空间进行分配时，虚拟机会执行一次MinorGC。MinorGC通常发生在新生代的Eden区，在这个区的对象生存期短，往往发生Gc的频率较高，回收速度比较快；FullGC/MajorGC发生在老年代，一般情况下，触发老年代GC的时候不会触发MinorGC，但是通过配置，可以在FullGC之前进行一次MinorGC这样可以加快老年代的回收速度。

### 十八.JVM的永久代中会发生垃圾回收么？

​	垃圾回收不会发生在永久代，如果永久代满了或者是超过了临界值，会触发完全垃圾回收（FullGC）。
​	注：Java8中已经移除了永久代，新加了一个叫做元数据区的native内存区。

### 十九.Java中垃圾收集的方法有哪些？

​	标记-清除：这是垃圾收集算法中最基础的，根据名字就可以知道，它的思想就是标记哪些要被回收的对象，然后统一回收。这种方法很简单，但是会有两个主要问题：
​	1. 效率不高，标记和清除的效率都很低；
​	2. 会产生大量不连续的内存碎片，导致以后程序在分配较大的对象时，由于没有充足的连续内存而提前触发一次GC动作。
​	
​	复制算法：为了解决效率问题，复制算法将可用内存按容量划分为相等的两部分，然后每次只使用其中的一块，当一块内存用完时，就将还存活的对象复制到第二块内存上，然后一次性清除完第一块内存，再将第二块上的对象复制到第一块。但是这种方式，内存的代价太高，每次基本上都要浪费一般的内存。
​		于是将该算法进行了改进，内存区域不再是按照1:1去划分，而是将内存划分为8:1:1三部分，较大那份内存交Eden区，其余是两块较小的内存区叫Survior区。每次都会优先使用Eden区，若Eden区满，就将对象复制到第二块内存区上，然后清除Eden区，如果此时存活的对象太多，以至于Survivor不够时，会将这些对象通过分配担保机制复制到老年代中。（java堆又分为新生代和老年代）
​	
​  标记-整理：该算法主要是为了解决标记-清除，产生大量内存碎片的问题；当对象存活率较高时，也解决了复制算法的效率问题。
​	它的不同之处就是在清除对象的时候现将可回收对象移动到一端，然后清除掉端边界以外的对象，这样就不会产生内存碎片了。

 分代收集：现在的虚拟机垃圾收集大多采用这种方式，它根据对象的生存周期，将堆分为新生代和老年代。在新生代中，由于对象生存期短，每次回收都会有大量对象死去，那么这时就采用复制算法。老年代里的对象存活率较高，没有额外的空间进行分配担保。

### 二十.什么是类加载器，类加载器有哪些？

通过类的权限定名获取该类的二进制字节流的代码块叫做类加载器。
主要有一下四种类加载器：
	•启动类加载器（BootstrapClassLoader）用来加载Java核心类库，无法被Java程序直接引用。负责加载$JAVA_HOME中jre/lib/rt.jar
	•扩展类加载器（extensionsclassloader）：负责加载JRE的扩展目录lib/ext。Java虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载Java类。
	•系统类加载器（systemclassloader）：它根据Java应用的类路径（CLASSPATH）来加载Java类。一般来说，Java应用的类都是由它来完成加载的。可以通过ClassLoader.getSystemClassLoader()来获取它。
	•用户自定义类加载器，通过继承java.lang.ClassLoader类的方式实现。

类加载器加载Class大致要经过如下8个步骤：
	1.检测此Class是否载入过，即在缓冲区中是否有此Class，如果有直接进入第8步，否则进入第2步。
	2.如果没有父类加载器，则要么Parent是根类加载器，要么本身就是根类加载器，则跳到第4步，如果父类加载器存在，则进入第3步。
	3.请求使用父类加载器去载入目标类，如果载入成功则跳至第8步，否则接着执行第5步。
	4.请求使用根类加载器去载入目标类，如果载入成功则跳至第8步，否则跳至第7步。
	5.当前类加载器尝试寻找Class文件，如果找到则执行第6步，如果找不到则执行第7步。
	6.从文件中载入Class，成功后跳至第8步。
	7.抛出ClassNotFountException异常。
	8.返回对应的java.lang.Class对象。

### 二十一. 类加载机制
JVM的类加载机制主要有如下3种。
	全盘负责：所谓全盘负责，就是当一个类加载器负责加载某个Class时，该Class所依赖和引用其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入。
	
	双亲委派：所谓的双亲委派，则是先让父类加载器试图加载该Class，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类。通俗的讲，就是某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父加载器，依次递归，如果父加载器可以完成类加载任务，就成功返回；只有父加载器无法完成此加载任务时，才自己去加载。
	
	缓存机制:缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区中搜寻该Class，只有当缓存区中不存在该Class对象时，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓冲区中。这就是为很么修改了Class后，必须重新启动JVM，程序所做的修改才会生效的原因。

### 二十二. 类加载器双亲委派模型机制？

​	当一个类收到了类加载请求时，不会自己先去加载这个类，而是将其委派给父类，由父类去加载，如果此时父类不能加载，反馈给子类，由子类去完成类的加载。

   双亲委派机制，其工作原理的是，如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式，即每个儿子都很懒，每次有活就丢给父亲去干，直到父亲说这件事我也干不了时，儿子自己才想办法去完成。
     
   双亲委派机制的优势：采用双亲委派模式的是好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。其次是考虑到安全因素，java核心api中定义类型不会被随意替换，假设通过网络传递一个名为java.lang.Integer的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Integer，而直接返回已加载过的Integer.class，这样便可以防止核心API库被随意篡改。

### 二十三. 类的热替换(Hotswap)
​	springboot-devtools模块能够实现热部署,添加类.添加方法,修改配置文件,修改页面等,都能实现热部署.
其深层原理是使用了两个ClassLoder,一个ClassLoader加载哪些不会改变的类(第三方jar包),另一个ClassLoader加载会更改的类.称之为restart ClassLoader,这样在有代码更改的时候,原来的restart Classloader被丢弃,重新创建一个restart ClassLoader,由于需要加载的类相比较少,所以实现了较快的重启时间
实现自己的类加载器https://blog.csdn.net/Dome_/article/details/102710377
​	要想实现自己的类加载器，只需要继承ClassLoader类即可。而我们要打破双亲委派规则，那么我们就必须要重写loadClass方法，因为默认情况下loadClass方法是遵循双亲委派的规则的。

```java
	public class CustomClassLoader extends ClassLoader{

    private static final String CLASS_FILE_SUFFIX = ".class";

    //AppClassLoader的父类加载器
    private ClassLoader extClassLoader;

    public CustomClassLoader(){
        ClassLoader j = String.class.getClassLoader();
        if (j == null) {
            j = getSystemClassLoader();
            while (j.getParent() != null) {
                j = j.getParent();
            }
        }
        this.extClassLoader = j ;
    }

    protected Class<?> loadClass(String name, boolean resolve){

        Class cls = null;
        cls = findLoadedClass(name);
        if (cls != null){
            return cls;
        }
        //获取ExtClassLoader
        ClassLoader extClassLoader = getExtClassLoader() ;
        //确保自定义的类不会覆盖Java的核心类
        try {
            cls = extClassLoader.loadClass(name);
            if (cls != null){
                return cls;
            }
        }catch (ClassNotFoundException e ){

        }
        cls = findClass(name);
        return cls;
    }

    @Override
    public Class<?> findClass(String name) {
        byte[] bt = loadClassData(name);
        return defineClass(name, bt, 0, bt.length);
    }

    private byte[] loadClassData(String className) {
        // 读取Class文件呢
        InputStream is = getClass().getClassLoader().getResourceAsStream(className.replace(".", "/")+CLASS_FILE_SUFFIX);
        ByteArrayOutputStream byteSt = new ByteArrayOutputStream();
        // 写入byteStream
        int len =0;
        try {
            while((len=is.read())!=-1){
                byteSt.write(len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 转换为数组
        return byteSt.toByteArray();
    }

    public ClassLoader getExtClassLoader(){
        return extClassLoader;
    }
}
```

​	为什么要先获取ExtClassLoader类加载器呢？其实这里是借鉴了Tomcat里面的设计，是为了避免我们自定义的类加载器覆盖了一些核心类。例如java.lang.Object。

​	为什么是获取ExtClassLoader类加载器而不是获取AppClassLoader呢？这是因为如果我们获取了AppClassLoader进行加载，那么不还是双亲委派的规则了吗？



## Part 2

### 1、JVM垃圾回收的时候如何确定垃圾？是否知道什么是GC Roots

1. 什么是垃圾？

   内存中已经不再使用到的空间就是垃圾

2. 要进行垃圾回收，如何判断一个对象是否可以被回收

   - **引用计数法**：java中，引用和对象是由关联的。如果要操作对象则必须用引用进行。

     因此很显然一个简单的办法是通过引用计数来判断一个对象是否可以回收，简单说，给对象中添加一个引用计数器，每当有一个地方引用它，计数器加1，每当有一个引用失效时，计数器减1，任何时刻计数器数值为零的对象就是不可能再被使用的，那么这个对象就是可回收对象。

     但是它很难解决对象之间相互循环引用的问题

     JVM一般不采用这种实现方式。

   - **==枚举根节点做可达性分析（跟搜索路径）==**：

     为了解决引用计数法的循环引用问题，java使用了可达性分析的方法。

     所谓**GC ROOT**或者说**Tracing GC**的“根集合”就是一组比较活跃的引用。

     基本思路就是通过一系列“GC Roots”的对象作为起始点，从这个被称为GC Roots 的对象开始向下搜索，如果一个对象到GC Roots没有任何引用链相连时，则说明此对象不可用。也即**给定一个集合的引用作为根出发**，通过引用关系遍历对象图，能被便利到的对象就被判定为存活；没有被便利到的就被判定为死亡

#### （1）哪些对象可以作为GC Roots对象

- 虚拟机栈（栈帧中的局部变量区，也叫局部变量表）中应用的对象。
- 方法区中的类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI（native方法）引用的对象

### 2、如何盘点查看JVM系统默认值

**如何查看运行中程序的JVM信息**

jps查看进程信息  

jinfo -flag 配置项 进程号 

jinfo -flags 进程号   ====   查看所有配置

#### （1）JVM参数类型

1. 标配参

   `-version`  `-help`   

   各个版本之间稳定，很少有很大的变化

2. x参数

   `-Xint` `-Xcomp` `-Xmixed`

   - -Xint :解释执行
   - -Xcomp：第一次使用就编译成本地代码
   - -Xmixed：混合模式

3. **==xx参数==**

   - Boolean类型

     公式：`-XX+或者-某个属性值` +表示开启，-表示关闭

     **是否打印GC收集细节** -XX:+PrintGCDetails 开启   -XX:-PrintGCDetails 关闭

     **是否使用串行垃圾回收器**：-XX:-UseSerialGC

   - KV设值类型

     公式：-XX:属性key=属性值value

     case：

     -XX:MetaspaceSize=128m

     -XX:MaxTenuringThreshold=15

     -Xms----> -XX:InitialHeapSize

     -Xmx----> -XX:MaxHeapSize

#### （2）查看参数

- -XX:+PrintFlagsInitial

  查看初始默认

  ==java -XX:+PrintFlagsInitial -version==

- -XX:+PrintFlagsFinal

  查看修改后的 `:=`说明是修改过的

- -XX:+PrintCommandLineFlags

  查看使用的垃圾回收器



### 3、你平时工作用过的JVM常用基本配置参数有哪些

`-Xms` `-Xmx` `-Xmn` 

-Xms128m -Xmx4096m -Xss1024K -XX:MetaspaceSize=512m -XX:+PrintCommandLineFlags -XX:+PrintGCDetails -XX:+UseSerialGC

1. -Xms

   初始大小内存，默认为物理内存1/64，等价于-XX:InitialHeapSize

2. -Xmx

   最大分配内存，默认物理内存1/4，等价于-XX:MaxHeapSize

3. -Xss

   设置单个线程栈的大小，默认542K~1024K ，等价于-XX:ThreadStackSize

4. -Xmn

   设置年轻代的大小

5. -XX:MetaspaceSize

   设置元空间大小

   元空间的本质和永久代类似，都是对JVM规范中方法区的实现，不过元空间与永久代最大的区别在于：==元空间并不在虚拟机中，而是在本地内存中。==因此，默认元空间的大小仅受本地内存限制

6. -XX:+PrintGCDetails

   输出详细GC收集日志信息

   [名称：GC前内存占用->GC后内存占用(该区内存总大小)]

7. -XX:SurvivorRatio

   设置新生代中Eden和S0/S1空间的比例

   默认-XX:SurvivorRatio=8,Eden:S0:S1=8:1:1

8. -XX:NewRatio

   设置年轻代与老年代在堆结构的占比

   默认-XX:NewRatio=2  新生代在1，老年代2，年轻代占整个堆的1/3

   NewRatio值几句诗设置老年代的占比，剩下的1给新生代

9. -XX:MaxTenuringThreshold

   设置垃圾的最大年龄

   默认-XX:MaxTenuringThreshold=15

   如果设置为0，年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大的值，则年轻代对象回在Survivor区进行多次复制，这样可以增加对对象在年轻代的存活时间，增加在年轻代即被回收的概率。

10. -XX:+UseSerialGC

    串行垃圾回收器

11. -XX:+UseParallelGC

    并行垃圾回收器

### 4、强引用、软引用、弱引用、虚引用作用分别是什么

#### 4.1 强引用 Reference

当内存不足，JVM开始垃圾回收，对于强引用对象，就算出现了OOM也不会堆该对象进行回收。

强引用是我们最常见的普通对象引用，只要还有强引用指向一个对象，就能表面对象还“或者”，垃圾收集器不会碰这种对象。在Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，他是不可能被垃圾回收机制回收的，既是该对象以后永远都不会被用到 JVM也不会回收。因此强引用时造成java内存泄漏的主要原因之一。

对于一个普通对象，如果没有其他的引用关系，只要超过了引用的作用域或者显式的将应用（强）引用复制为null，一般认为就是可以被垃圾收集的了

#### 4.2 软引用  SoftReference

软引用是一种相对强引用弱化了一些作用，需要用`java.lang.ref.SOftReference`类来实现，可以让对象豁免一些垃圾收集。

当系统内存充足时他不会被回收，当内存不足时会被回收。

软引用通常在对内存敏感的程序中，比如高速缓存就有用到软引用，内存足够的时候就保留，不够就回收。

#### 4.3 弱引用 WeakReference

弱引用需要用java.lang.ref.WeakReference类来实现，比软引用的生存期更短

只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收。

- **谈谈WeakHashMap**

  key是弱引用

#### 4.4 虚引用PhantomReference

顾名思义，就是形同虚设，与其他集中不同，虚引用并不会决定对象的生命周期

如果一个对象持有虚引用，那么他就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，他不能单独使用也不能通过它访问对象，虚引用和引用队列（ReferenceQueeu)联合使用。

需应用的主要作用是跟踪对象被垃圾回收的状态。仅仅是提供了一种确保ui想被finalize以后，做某些事情的机制。

PhantomReference的get方法总是返回null，因此无法访问对应的引用对象。其意义在于说明一个对象已经进入finalization阶段，可以被gc回收，用来实现比finalization机制更灵活的回收操作

换句话说，设置虚引用关联的唯一目的，就是这个对象被收集器回收的时候收到一个系统通知或者后续添加进一步的处理。

java允许使用finalize()方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。

##### 4.4.1 引用队列Reference

创建引用的时候可以指定关联的队列，当gc释放对象内存的时候，会把引用加入到引用队列，如果程序发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动，相当于通知机制

当关联的引用队列中有数据的时候，意味着引用指向的对内存中的对象被回收。通过这种方式，jvm允许我们在对象被小回收，做一些我们自己想做的事情。

#### 4.5 适用场景

- 加入一个应用需要读取大量的本地图片，如果每次读取图片都从硬盘读取会严重影响性能，如果一次性全部加载到内存又可能造成内存溢出，这时可以用软引用解决这个问题

  设计思路：用一个HashMap来保存图片路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动共回收这些缓存图片对象所占的空间，避免OOM

  ```java
  Map<String, SoftReference<Bitmap>> imageCache = new HashMap<>();
  ```



### 5、请你谈谈对OOM的认识

- `java.lang.StackOverflowError`

  栈空间溢出 ，递归调用卡死

- `java.lang.OutOfMemoryError:Java heap space`

  堆内存溢出 ， 对象过大

- `java.lang.OutOfMemoryError:GC overhead limit exceeded`

  GC回收时间过长

  过长的定义是超过98%的时间用来做GC并且回收了而不倒2%的堆内存

  连续多次GC，都回收了不到2%的极端情况下才会抛出

  如果不抛出，那就是GC清理的一点内存很快会被再次填满，迫使GC再次执行，这样就恶性循环，

  cpu使用率一直是100%，二GC却没有任何成果

  ```java
  int i = 0;
  List<String> list = new ArrayList<>();
  try{
      while(true){
          list.add(String.valueOf(++i).intern());
      }
  }catch(Throwable e){
      System.out.println("********");
      e.printStackTrace();
      throw e;
  }
  ```

  

- `java.lang.OutOfMemoryError:Direct buffer memory`

  执行内存挂了

  写NIO程序经常使用ByteBuffer来读取或写入数据，这是一种基于通道（Channel）与缓存区（Buffer)的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作，这样能在一些场景中显著提高性能，因为避免了在java堆和native堆中来回复制数据

  ByteBuffer.allocate(capability) 第一种方式是分配JVM堆内存，属于GC管辖，由于需要拷贝所以速度较慢

  ByteBuffer.alloctedDirect(capability)分配os本地内存，不属于GC管辖，不需要拷贝，速度较快

  但如果不断分配本地内存，堆内存很少使用，那么jvm就不需要执行GC，DirectByteBuffer对象们就不会被回收，这时候堆内存充足，但本地内存可能已经使用光了，再次尝试分配本地内存救护i出现oom，程序崩溃

  

- `java.lang.OutOfMemoryError:unable to create new native thread`**==好案例==**

  - 应用创建了太多线程，一个应用进程创建了多个线程，超过系统承载极限
  - 你的服务器并不允许你的应用程序创建这么多线程，linux系统默认允许单个进程可以创建的线程数是1024，超过这个数量，就会报错

  解决办法

  降低应用程序创建线程的数量，分析应用给是否针对需要这么多线程，如果不是，减到最低

  修改linux服务器配置

- `java.lang.OutOfMemoryError:Metaspace`

  元空间主要存放了虚拟机加载的类的信息、常量池、静态变量、即时编译后的代码

  ```java
  static class OOMTest{}
  public static void main(String[] args){
      int i = 0;
      try{
         while(true){
             i++;
             Enhancer enhancer = new Enhancer();
             enhancer.setSuperclass(OOMTest.class);
             enhancer.setUseCache(false);
             enhancer.setCallback(new MethodInterceptor(){
                 @Override
                 public Object intercept(Object o,Method method,Object[] objects,
                                        MethodProxy methodProxy)throws Throwable{
                     return methodProxy.invokeSuper(o,args);
                 }
             });
             enhancer.create();
         } 
      } catch(Throwable e){
          System.out.println(i+"次后发生了异常");
          e.printStackTrace();
      }
  }
  ```

  

### 6、==GC垃圾回收算法和垃圾收集器的关系？分别是什么==

垃圾收集器就是算法的具体实现

#### 6.1 GC算法

- 引用计数
- 复制
- 标记清理
- 标记整理

#### 6.2 4种主要垃圾收集器

1. Serial 串行回收

   为单线程换进该设计且只是用过一个线程进行垃圾回收，会暂停所有的用户线程，不适合服务器环境

2. Paralle 并行回收

   多个垃圾收集线程并行工作，此时用户线程是暂停的，适用于科学计算/大数据处理首台处理等弱交互场景

3. CMS 并发标记清除

   用户线程和垃圾手机线程同时执行（不一定是并行，可能交替执行），不需要停顿用户线程，互联网公司多用它，适用堆响应时间有要求的场景

4. G1

   将堆内存分割城不同的区域然后并发的对其进行垃圾回收

5. ZGC



### 7、==怎么查看服务器默认的垃圾收集器是哪个？生产上如何配置垃圾收集器？对垃圾收集器的理解？==

1. 查看：`java -XX:+PrintCommandLinedFlags -version`

2. 配置：

   有七种：`UseSerialGC` `UseParallelGC` `UseConcMarkSweepGC` `UseParNewGC` `UseParallelOldGC` `UseG1GC`


![](images/1559370440(1).jpg)

![](images/1559370703(1).jpg)



### 8、==垃圾收集器==

#### 8.1 部分参数说明

1. DefNew：Default New Generation
2. Tenured：Old
3. ParNew：Parallel New Generation
4. PSYoungGen：Parallel Scavenge
5. ParOldGen：Parallel Old Generation

#### 8.2 Server/Client模式分别是什么意思

1. 适用范围：只需要掌握Server模式即可，Client模式基本不会用
2. 操作系统
   - 32位Window操作系统，不论硬件如何都默认使用Client的JVM模式
   - 32位其他操作系统，2G内存同时有2个cpu以上用Server模式，低于该配置还是Client模式
   - 64位only server模式

#### 8.3 新生代

1. **串行GC    (Serial)/(Serial Copying)**

   串行收集器：Serial收集器  1:1

   一个单线程的收集器，在进行垃圾收集的时候，必须暂停其他所有的工作线程知道它收集结束。

   串行收集器是最古老，最稳定以及效率最高的收集器，只使用一个线程去回收但其在进行垃圾收集过程中可能会产生较长的停顿（Stop-The-World 状态）。虽然在收集垃圾过程中需要暂停所有其他的工作线程，但是它简单高效，==对于限定单个CPU环境来说，没有线程交互的开销可以获得最高的单线程垃圾收集效率，因此Serial垃圾收集器依然是java虚拟机运行在Client模式下默认的新生代垃圾收集器。==

   对应的JVM参数是：**==-XX:+UseSerialGC==**

   ==开启后会使用：Serial(Young区)+Serial Old(Old区)的收集器组合==

   表示：==新生代、老年代都会使用串行回收收集器，新生代使用复制算法，老年代使用标记整理算法==

   DefNew   -----     Tenured

2. **并行GC    (ParNew)**

   并行收集器    N:1

   使用多线程进行垃圾回收，在垃圾收集时，会Stop-the-world暂停其他所有的工作线程知道它收集结束。

   ==ParNew收集器其实就是Serial收集器新生代的并行多线程版本，==最常见的应用场景时配合==老年代的CMS GC工作==，其余的行为和Serial收集器完全一样，ParNew垃圾收集器在垃圾收集过程成中童谣也要暂停所有其他的工作线程，他是很多==java虚拟机运行在Server模式下**新生代**的默认垃圾收集器。==

   常用对应JVM参数：**==-XX:+UseParNewGC==**  启用ParNew收集器，只影响新生代的收集，不影响老年代

   ==开启后会使用：ParNew(Young区)+Serial Old(Old 区)的收集器组合==，新生代使用复制算法，老年代采用标记-整理算法

   ParNew   ------   Tenured

   备注:

   ==**-XX:ParallelGCThreads**==  限制线程数量，默认开启和CPU数目相同的线程数

3. **并行回收GC     (Parallel)/(Parallel Scavenge)**

   并行收集器    N:N

   类似ParNew也是一个新生代垃圾收集器，使用复制算法，也是一个并行的多线程的垃圾收集器，俗称吞吐量有限收集器。串行收集器在新生代和老年代的并行化。

   他重点关注的是：

   - **可控制的吞吐量**（Thoughput=运行用户代码时间/(运行用户代码时间+垃圾收集时间)，业绩比如程序运行100分钟，垃圾收集时间1分钟，吞吐量就是99%）。高吞吐量意味着高效利用CPU时间，他多用于在后台运算而不需要太多交互的任务。
   - 自适应调节策略也是ParallelScavenge收集器与ParNew收集器的一个重要区别。（自适应调节策略：虚拟机会根据当前系统的运行情况手机性能监控信息，动态调整这些参数以提供最合适的停顿时间==(-XX:MaxGCPauseMillis)==或最大的吞吐量

   ==-XX:ParallelGCThreads=数字N==  表示启动多少个GC线程

   cpu>8  n=5/8

   cpu<8  n=实际个数

   ==**-XX:+UseParallelGC**==、==**-XX:+UseParallelOldGC**==

#### 8.4 老年代

1. **串行GC（Serial Old）/（Serial MSC）**

2. **并行GC（Parallel Old）/（Parallel MSC）**

   Parallel Scavenge的老年代版本，使用多线程的标记-整理算法，Parallel Old收集器在JDK1.6开始提供

   1.6之前，新生代使用ParallelScavenge收集器只能搭配年老代的Serial Old收集器，只能保证新生代的吞吐量优先，无法保证整体的吞吐量。在1.6之前（ParallelScavenge+SerialOld）

   Parallel Old正式为了在老年代同样提供吞吐量游戏的垃圾收集器，如果系统对吞吐量要求比较高，JDK1.8后可以考虑新生代Parallel Scavenge和年老代Parallel Old收集器的搭配策略

   常用参数：==**-XX:+UseParallelOldGC**==》》》》》新生代Paralle+老年代Paralle Old

3. **并发标记清除GC（CMS）**

   CMS收集器（Concurrent Mark Sweep：并发标记清除）是一种以获取最短回收停顿时间为目标的收集器。

   适合在互联网站或者B/S系统的服务器上，这列应用尤其中使服务器的响应速度，希望系统停顿时间最短。

   CMS非常适合堆内存大、CPU核数多的服务区端应用，也是G1出现之大型应用的首选收集器。

   并发标记清除收集器：ParNew+CMS+Serial Old

   CMS，并发收集低停顿，并发指的是与用户线程一起执行

   JVM参数：**==-XX:+UseConcMarkSweepGC==**，开启该参数后会自动将-XX:UseParNewGC打开

   开启该参数后，使用ParNew+CMS+Serial Old的收集器组合，Serial Old将作为CMS出错的后备收集器

   **4步过程：**

   - 初始标记（CMS initial mark）

     只是标记一下GC Roots能够直接关联的对象，速度很快，仍然需要暂停所有的工作线程。

   - 并发标记（CMS concurrent mark）和用户线程一起

     进行GC Roots跟踪过程，和用户线程一起工作，不需要暂停工作线程。主要标记过程，标记全部对象

   - 重新标记（CMS remark）

     为了修正并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程，由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正

   - 并发清除（CMS concurrent sweep）和用户线程一起

     清除GC Roots不可达对象，和用户线程一起工作，不需要暂停工作线程。基于标记结果，直接清理对象

     由于耗时最长的并发标记和并发清除过程中，垃圾收集线程可以和用户现在一起并发工作，所以总体上看来CMS收集器的内存回收和用户线程是一起并发的执行。

   **优缺点：**

   - 并发收集停顿低

   - 并发执行，cpu资源压力大

     由于并发进行，CMS在收集与应用线程会同时增加对堆内存的占用，也就是说，==CMS必须要在老年代堆内存用尽之前完成垃圾回收==，否则CMS回收失败时，将出发担保机制，串行老年代收集器将会以STW的方式进行一次GC，从而造成较大停顿时间。

   - 采用的标记清除算法会导致大量的碎片

     标记清除算法无法整理空间碎片，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制堆堆内存进行压缩。CMS也提供了参数==-XX:CMSFullGCsBeForeCompaction==(默认0，即每次都进行内存整理)来制定多少次CMS收集之后，进行一次压缩的FullGC。

#### 8.5 如何选择垃圾选择器

- 单CPU或小内存，单机内存

  -XX:+UseSerialGC

- 多CPU，需要最大吞吐量，如后台计算型应用

  -XX:+UseParallelGC    -XX:+UseParallelOldGC

- 多CPU，最求低停顿时间，需快速相应，如互联网应用

  -XX:+ParNewGC    -XX:+UseConcMarkSweepGC

| 参数                              | 新生代垃圾收集器   | 新生代算法         | 老年代垃圾收集器                                             | 老年代算法 |
| --------------------------------- | ------------------ | ------------------ | ------------------------------------------------------------ | ---------- |
| UseSerialGC                       | SerialGC           | 复制               | SerialOldGC                                                  | 标整       |
| UseParNewGC                       | ParNew             | 复制               | SerialOldGC                                                  | 标整       |
| UseParallelGC<br>UseParallelOldGC | Parallel[Scavenge] | 复制               | Parallel Old                                                 | 标整       |
| UseConcMarkSweepGC                | ParNew             | 复制               | CMS+Serial Old的收集器组合(Serial Old 作为CMS出错的后备收集器) | 标清       |
| UseG1GC                           | G1整体上采用标整   | 局部是通过复制算法 |                                                              |            |



### 9、==G1垃圾收集器==

将堆内存分割城不同的区域然后并发的对其进行垃圾回收

#### 9.1 其他收集器特点

- 年轻代和老年代是各自独立且了连续的内存块
- 年轻代收集使用单eden+S0+S1进行复制算法
- 老年代收集必须扫描真个老年代区域
- 都是以尽可能少而快速地执行GC为设计原则

#### 9.2 G1是什么

G1（Garbage-First）收集器，是一款面向服务端应用的收集器

应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能的满足垃圾收集暂停时间的要求

- 和CMS收集器一样，能与应用程序线程并发执行
- 整理空闲空间更快
- 需要更多的时间来预测GC停顿时间
- 不希望牺牲大量的吞吐性能
- 不需要更大的Java Heap

G1收集器的设计目标是取代CMS收集器，和CMS相比，在以下放木表现更出色：

- G1是一个由整理内存过程的垃圾收集器，不会产生很多内存碎片
- G1的Stop The World（STW）更可控，G1在停顿时间上添加了预测机制，用户可以指定期望停顿时间。

CMS垃圾收集器虽然减少了暂停应用程序的运行时间，但是他还是存在着内存碎片问题。于是为了取出内存碎片问题，同时又保留CMS垃圾收集器低暂停时间的优点，JAVA7发布了G1垃圾收集器。

主要改变的是Eden，Survivor和Tenured等内存区域不再是连续的了，而是变成了一个个大小一样的region（区域化），每个region从1M到32M不等。一个region有可能属于Eden，Survivor或Tenured内存区域。

##### 9.2.1 特点

1. G1能充分利用多CPU、多核环境优势，尽量缩短STW。
2. G1整体上采用标记-整理算法，局部是通过复制算法，不会产生内存碎片。
3. 宏观上G1之中不再区分年轻代和老年代。把内存划分成多个独立的子区域（Region），可以近似理解为一个棋盘
4. G1收集器里面讲整个的内存去都混合在一起了，但其本身依然在小范围内要进行年轻代和老年代的区分，保留了新生代和老年代，但它们不再是物理隔离，而是一部分Region的集合且不需要Region是连续的，也就是说依然会采用不同GC方式来处理不同的区域。
5. G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代与老年代的区别，也不需要完全独立的survivor（to space）堆做复制准备。G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换 

#### 9.3 底层原理

- Region区域化垃圾收集器

  最大好处是化整为零，避免全内存扫描，只需要按照区域来进行扫描即可

  区域化内存划片Region，整体编为了 一系列不连续的内存区域，避免了全内存区的GC操作。

  核心思想是讲整个堆内存区域分成大小相同的子区域，在JVM启动时会自动共设置这些子区域的大小

  在堆的使用上，G1并不要求对象的存储一定是物理上连续的，只要逻辑上连续即可，每个分区也不会固定地为某个代服务，可以按需在年轻代和老年代之间切换。启动时可以通过参数==-XX:G1HeapRegionSize==可指定分区大小（1~32M,且必须是2的幂），默认将整堆划分为2048个分区。

  大小范围在1-32M，最多能设置2048个区域，也即能够支持的最大内存为64G

  G1算法将堆划分为诺干个区域，他仍然属于分代收集器

  - 这些Region的一部分包含新生代，新生代的垃圾收集依然采用暂停所有应用线程的方式，将存活对象拷贝到老年代或Survivor空间，这些Region的一部分包含老年代，G1收集器通过将对象从一个区域复制到另外一个区域，完成了清理工作。这就意味着，在正常的处理过程中，G1完成了堆的压缩，这样也就不会有CMS内存碎片问题的存在了

  - 在G1中，还有一种特殊区域，**Humongous**区域，如果一个对象张勇的空间超过了分区容量50%以上，G1收集器就认为i这是一个巨型对象。这些巨型对象默认直接会被分配在年老代，但是如果他是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响。为了解决这个问题，G1划分了一个Humongous区，他用来专门存放巨型对象。如果一个H区装不下，那么G1就会寻找连续的H分区来存储。为了能找到连续的H区，有时候不得不启动Full GC

- 回收步骤

  针对Eden区进行收集，Eden区耗尽后会被触发，主要小区域收集+形成连续的内存块，避免内存碎片

  - Eden区的数据移动到Survivor区，假如出现Survivor区空间不够，Eden区数据就会晋升到Old区
  - Survivor区的数据移动到新的Survivor区，部分数据晋升到Old区
  - 最后Eden区收拾干净了，GC结束，用户的应用程序继续执行。

- 4步过程

  1. 初始标记：只标记GC Roots能直接关联到的对象
  2. 并发标记：进行GC Roots Tracing的过程
  3. 最终标记：修正并发标记期间，因程序运行导致标记发生变化的那一部分对象
  4. 筛选回收：根据时间来进行价值最大化的回收

#### 9.4 case



#### 9.5 常用配置参数（不是重点）

- **-XX:+UseG1Gc**

- **-XX:G1HeapRegionSize=n**

  设置的G1区域的大小，值是2的幂，范围是1-32MB，目标是根据最小的java堆大小划分出约2048个区域

- **-XX:MaxGCPauseMillis=n**

  最大GC停顿时间，这是个软目标，JVM将尽可能（但不保证）停顿小于这个时间

- **-XX:InitiatingHeapOccupancyRercent=n**

  堆占用了多少的时候就触发GC，默认45

- **-XX:ConcGcThreads=n**

  并发GC使用的线程数

- **-XX:G1ReservePercent=n**

  设置作为空闲空间的预留内存百分比，以降低目标空间溢出的风险，默认10%

#### 9.6 和CMS相比优势

1. G1不会产生内存碎片
2. 可以精确控制停顿。该收集器十八真个堆划分成多个固定大小的区域，每根据允许停顿的时间去收集垃圾最多的区域



### 10、生产环境服务器变慢，诊断思路和性能评估谈谈？

1. 整机：top 系统性能

   uptime 精简版

   load average：系统负载均衡 1min 5min 15min 系统的平均负载值 相加/3>60%压力够

2. CPU：vmstat

   - 查看CPU

     vmstat -n 2 3 第一个参数是采样的时间间隔数单位s，第二个参数是采样的次数

     - procs

       **r**：运行和等待CPU时间片的进程数，原则上1核CPu的运行队列不要超过2，真个系统的运行队列不能超过总核数的2倍，否则表示系统压力过大

       **b**：等待资源的进程数，比如正在等待磁盘I/O，网络I/O等

     - cpu

       **us**：用户进程消耗cpu时间百分比，us高，用户进程消耗cpu时间多，如果长期大于50%，优化程序

       **sy**：内核进程消耗的cpu时间百分比

       **us+sy**：参考值为80%，如果大于80，说明可能存在cpu不足

       **id**：处于空闲的cpu百分比

       **wa**：系统等待IO的cpu时间百分比

       **st**：来自于一个虚拟机偷取的cpu时间的百分比

   - 查看额外

     - 查看所有cpu核信息   mpstat -P ALL 2
     - 每个进程使用cpu的用量分解信息   pidstat -u 1 -p 进程编号

3. 内存：free

   查看内存   free -m   free -g

   pidstat -p 进程编号 -r 采样间隔秒数

4. 硬盘：df

   查看磁盘剩余空间  df -h

5. 磁盘IO：iostat

   - 磁盘I/O性能评估

     iostat -hdk 2 3

     - rkB/s每秒读取数据kb；
     - wkB/s每秒读写数据量kb
     - svctm I/O请求的平均服务时间，单位毫秒；
     - await I/O请求的平均等待时间，单位毫秒；值越小，性能越好；
     - ==util== 一秒中又百分几的时间用于I/O操作，接近100%时，表示磁盘带宽跑满，需要优化程序或加磁盘
     - rkB/s，wkB/s根据系统该应用不同回有不同的值，担忧规律遵循：长期、超大数据读写，肯定不正常，需要优化程序读取。
     - svctm的值与await的值很接近，表示几乎没有I/O等待，磁盘性能好，如果await的值远高于svctm的值，则表示I/O队列等待太长，需要优化程序或更换更快磁盘

   - pidstat -d 采样间隔秒数 -p 进程号

6. 网络IO：ifstat

   ifstat l



### 11、加入生产环境CPU占用过高，谈谈分析思路和定位？

结合Linux和JDK命令一块分析

#### 11.1 案例步骤

1. 先用top命令找出cpu占比最高的

2. ps -ef或者jps进一步定位，得知是一个怎样的后台程序惹事

   - jps -l
   - ps -ef|grep java|grep -v grep

3. 定位到具体线程或者代码

   - ps -mp 进程编号 -o Thread,tid,time

     定位到具体线程

     -m ：显示所有的线程

     -p pid 进程使用cpu的时间

     -o：该参数后是用户自定义格式

4. 将需要的线程ID转换为16禁止格式（英文小写格式）

   - printf "%x\n" 线程ID

5. jstack 进程Id|grep tid(16进制线程id小写英文) -A60

   查看运行轨迹，堆栈异常信息



### 12、对于JDK自带的JVM监控和性能分析工具你用过那些？一般怎么用？