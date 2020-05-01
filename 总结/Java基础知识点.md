# Collections 工具类和 Arrays 工具类常见方法

## Collections

Collections 工具类常用方法:

1. 排序
2. 查找,替换操作
3. 同步控制(不推荐，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合)

### 排序操作

```
void reverse(List list)//反转
void shuffle(List list)//随机排序
void sort(List list)//按自然排序的升序排序
void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j)//交换两个索引位置的元素
void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面。
```

**示例代码:**

```
     ArrayList<Integer> arrayList = new ArrayList<Integer>();
		arrayList.add(-1);
		arrayList.add(3);
		arrayList.add(3);
		arrayList.add(-5);
		arrayList.add(7);
		arrayList.add(4);
		arrayList.add(-9);
		arrayList.add(-7);
		System.out.println("原始数组:");
		System.out.println(arrayList);
		// void reverse(List list)：反转
		Collections.reverse(arrayList);
		System.out.println("Collections.reverse(arrayList):");
		System.out.println(arrayList);


		Collections.rotate(arrayList, 4);
		System.out.println("Collections.rotate(arrayList, 4):");
		System.out.println(arrayList);

		// void sort(List list),按自然排序的升序排序
		Collections.sort(arrayList);
		System.out.println("Collections.sort(arrayList):");
		System.out.println(arrayList);

		// void shuffle(List list),随机排序
		Collections.shuffle(arrayList);
		System.out.println("Collections.shuffle(arrayList):");
		System.out.println(arrayList);

		// void swap(List list, int i , int j),交换两个索引位置的元素
		Collections.swap(arrayList, 2, 5);
		System.out.println("Collections.swap(arrayList, 2, 5):");
		System.out.println(arrayList);

		// 定制排序的用法
		Collections.sort(arrayList, new Comparator<Integer>() {

			@Override
			public int compare(Integer o1, Integer o2) {
				return o2.compareTo(o1);
			}
		});
		System.out.println("定制排序后：");
		System.out.println(arrayList);
```

### 查找,替换操作

```
int binarySearch(List list, Object key)//对List进行二分查找，返回索引，注意List必须是有序的
int max(Collection coll)//根据元素的自然顺序，返回最大的元素。 类比int min(Collection coll)
int max(Collection coll, Comparator c)//根据定制排序，返回最大元素，排序规则由Comparatator类控制。类比int min(Collection coll, Comparator c)
void fill(List list, Object obj)//用指定的元素代替指定list中的所有元素。
int frequency(Collection c, Object o)//统计元素出现次数
int indexOfSubList(List list, List target)//统计target在list中第一次出现的索引，找不到则返回-1，类比int lastIndexOfSubList(List source, list target).
boolean replaceAll(List list, Object oldVal, Object newVal), 用新元素替换旧元素
```

**示例代码：**

```
		ArrayList<Integer> arrayList = new ArrayList<Integer>();
		arrayList.add(-1);
		arrayList.add(3);
		arrayList.add(3);
		arrayList.add(-5);
		arrayList.add(7);
		arrayList.add(4);
		arrayList.add(-9);
		arrayList.add(-7);
		ArrayList<Integer> arrayList2 = new ArrayList<Integer>();
		arrayList2.add(-3);
		arrayList2.add(-5);
		arrayList2.add(7);
		System.out.println("原始数组:");
		System.out.println(arrayList);

		System.out.println("Collections.max(arrayList):");
		System.out.println(Collections.max(arrayList));

		System.out.println("Collections.min(arrayList):");
		System.out.println(Collections.min(arrayList));

		System.out.println("Collections.replaceAll(arrayList, 3, -3):");
		Collections.replaceAll(arrayList, 3, -3);
		System.out.println(arrayList);

		System.out.println("Collections.frequency(arrayList, -3):");
		System.out.println(Collections.frequency(arrayList, -3));

		System.out.println("Collections.indexOfSubList(arrayList, arrayList2):");
		System.out.println(Collections.indexOfSubList(arrayList, arrayList2));

		System.out.println("Collections.binarySearch(arrayList, 7):");
		// 对List进行二分查找，返回索引，List必须是有序的
		Collections.sort(arrayList);
		System.out.println(Collections.binarySearch(arrayList, 7));
```

### 同步控制

Collections提供了多个`synchronizedXxx()`方法·，该方法可以将指定集合包装成线程同步的集合，从而解决多线程并发访问集合时的线程安全问题。

我们知道 HashSet，TreeSet，ArrayList,LinkedList,HashMap,TreeMap 都是线程不安全的。Collections提供了多个静态方法可以把他们包装成线程同步的集合。

**最好不要用下面这些方法，效率非常低，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合。**

方法如下：

```
synchronizedCollection(Collection<T>  c) //返回指定 collection 支持的同步（线程安全的）collection。
synchronizedList(List<T> list)//返回指定列表支持的同步（线程安全的）List。
synchronizedMap(Map<K,V> m) //返回由指定映射支持的同步（线程安全的）Map。
synchronizedSet(Set<T> s) //返回指定 set 支持的同步（线程安全的）set。
```

### Collections还可以设置不可变集合，提供了如下三类方法：

```
emptyXxx(): 返回一个空的、不可变的集合对象，此处的集合既可以是List，也可以是Set，还可以是Map。
singletonXxx(): 返回一个只包含指定对象（只有一个或一个元素）的不可变的集合对象，此处的集合可以是：List，Set，Map。
unmodifiableXxx(): 返回指定集合对象的不可变视图，此处的集合可以是：List，Set，Map。
上面三类方法的参数是原有的集合对象，返回值是该集合的”只读“版本。
```

**示例代码：**

```
        ArrayList<Integer> arrayList = new ArrayList<Integer>();
        arrayList.add(-1);
        arrayList.add(3);
        arrayList.add(3);
        arrayList.add(-5);
        arrayList.add(7);
        arrayList.add(4);
        arrayList.add(-9);
        arrayList.add(-7);
        HashSet<Integer> integers1 = new HashSet<>();
        integers1.add(1);
        integers1.add(3);
        integers1.add(2);
        Map scores = new HashMap();
        scores.put("语文" , 80);
        scores.put("Java" , 82);

        //Collections.emptyXXX();创建一个空的、不可改变的XXX对象
        List<Object> list = Collections.emptyList();
        System.out.println(list);//[]
        Set<Object> objects = Collections.emptySet();
        System.out.println(objects);//[]
        Map<Object, Object> objectObjectMap = Collections.emptyMap();
        System.out.println(objectObjectMap);//{}

        //Collections.singletonXXX();
        List<ArrayList<Integer>> arrayLists = Collections.singletonList(arrayList);
        System.out.println(arrayLists);//[[-1, 3, 3, -5, 7, 4, -9, -7]]
        //创建一个只有一个元素，且不可改变的Set对象
        Set<ArrayList<Integer>> singleton = Collections.singleton(arrayList);
        System.out.println(singleton);//[[-1, 3, 3, -5, 7, 4, -9, -7]]
        Map<String, String> nihao = Collections.singletonMap("1", "nihao");
        System.out.println(nihao);//{1=nihao}

        //unmodifiableXXX();创建普通XXX对象对应的不可变版本
        List<Integer> integers = Collections.unmodifiableList(arrayList);
        System.out.println(integers);//[-1, 3, 3, -5, 7, 4, -9, -7]
        Set<Integer> integers2 = Collections.unmodifiableSet(integers1);
        System.out.println(integers2);//[1, 2, 3]
        Map<Object, Object> objectObjectMap2 = Collections.unmodifiableMap(scores);
        System.out.println(objectObjectMap2);//{Java=82, 语文=80}

        //添加出现异常：java.lang.UnsupportedOperationException
//        list.add(1);
//        arrayLists.add(arrayList);
//        integers.add(1);
```

## Arrays类的常见操作

1. 排序 : `sort()`
2. 查找 : `binarySearch()`
3. 比较: `equals()`
4. 填充 : `fill()`
5. 转列表: `asList()`
6. 转字符串 : `toString()`
7. 复制: `copyOf()`

### 排序 : `sort()`

```
		// *************排序 sort****************
		int a[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
		// sort(int[] a)方法按照数字顺序排列指定的数组。
		Arrays.sort(a);
		System.out.println("Arrays.sort(a):");
		for (int i : a) {
			System.out.print(i);
		}
		// 换行
		System.out.println();

		// sort(int[] a,int fromIndex,int toIndex)按升序排列数组的指定范围
		int b[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
		Arrays.sort(b, 2, 6);
		System.out.println("Arrays.sort(b, 2, 6):");
		for (int i : b) {
			System.out.print(i);
		}
		// 换行
		System.out.println();

		int c[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
		// parallelSort(int[] a) 按照数字顺序排列指定的数组(并行的)。同sort方法一样也有按范围的排序
		Arrays.parallelSort(c);
		System.out.println("Arrays.parallelSort(c)：");
		for (int i : c) {
			System.out.print(i);
		}
		// 换行
		System.out.println();

		// parallelSort给字符数组排序，sort也可以
		char d[] = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
		Arrays.parallelSort(d);
		System.out.println("Arrays.parallelSort(d)：");
		for (char d2 : d) {
			System.out.print(d2);
		}
		// 换行
		System.out.println();
```

在做算法面试题的时候，我们还可能会经常遇到对字符串排序的情况,`Arrays.sort()` 对每个字符串的特定位置进行比较，然后按照升序排序。

```
String[] strs = { "abcdehg", "abcdefg", "abcdeag" };
Arrays.sort(strs);
System.out.println(Arrays.toString(strs));//[abcdeag, abcdefg, abcdehg]
```

### 查找 : `binarySearch()`

```
		// *************查找 binarySearch()****************
		char[] e = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
		// 排序后再进行二分查找，否则找不到
		Arrays.sort(e);
		System.out.println("Arrays.sort(e)" + Arrays.toString(e));
		System.out.println("Arrays.binarySearch(e, 'c')：");
		int s = Arrays.binarySearch(e, 'c');
		System.out.println("字符c在数组的位置：" + s);
```

### 比较: `equals()`

```
		// *************比较 equals****************
		char[] e = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
		char[] f = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
		/*
		* 元素数量相同，并且相同位置的元素相同。 另外，如果两个数组引用都是null，则它们被认为是相等的 。
		*/
		// 输出true
		System.out.println("Arrays.equals(e, f):" + Arrays.equals(e, f));
```

### 填充 : `fill()`

```
		// *************填充fill(批量初始化)****************
		int[] g = { 1, 2, 3, 3, 3, 3, 6, 6, 6 };
		// 数组中所有元素重新分配值
		Arrays.fill(g, 3);
		System.out.println("Arrays.fill(g, 3)：");
		// 输出结果：333333333
		for (int i : g) {
			System.out.print(i);
		}
		// 换行
		System.out.println();

		int[] h = { 1, 2, 3, 3, 3, 3, 6, 6, 6, };
		// 数组中指定范围元素重新分配值
		Arrays.fill(h, 0, 2, 9);
		System.out.println("Arrays.fill(h, 0, 2, 9);：");
		// 输出结果：993333666
		for (int i : h) {
			System.out.print(i);
		}
```

### 转列表 `asList()`

```
		// *************转列表 asList()****************
		/*
		 * 返回由指定数组支持的固定大小的列表。
		 * （将返回的列表更改为“写入数组”。）该方法作为基于数组和基于集合的API之间的桥梁，与Collection.toArray()相结合 。
		 * 返回的列表是可序列化的，并实现RandomAccess 。
		 * 此方法还提供了一种方便的方式来创建一个初始化为包含几个元素的固定大小的列表如下：
		 */
		List<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
		System.out.println(stooges);
```

### 转字符串 `toString()`

```
		// *************转字符串 toString()****************
		/*
		* 返回指定数组的内容的字符串表示形式。
		*/
		char[] k = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
		System.out.println(Arrays.toString(k));// [a, f, b, c, e, A, C, B]
```

### 复制 `copyOf()`

```
		// *************复制 copy****************
		// copyOf 方法实现数组复制,h为数组，6为复制的长度
		int[] h = { 1, 2, 3, 3, 3, 3, 6, 6, 6, };
		int i[] = Arrays.copyOf(h, 6);
		System.out.println("Arrays.copyOf(h, 6);：");
		// 输出结果：123333
		for (int j : i) {
			System.out.print(j);
		}
		// 换行
		System.out.println();
		// copyOfRange将指定数组的指定范围复制到新数组中
		int j[] = Arrays.copyOfRange(h, 6, 11);
		System.out.println("Arrays.copyOfRange(h, 6, 11)：");
		// 输出结果66600(h数组只有9个元素这里是从索引6到索引11复制所以不足的就为0)
		for (int j2 : j) {
			System.out.print(j2);
		}
		// 换行
		System.out.println();
```



#final,static,this,super

## final 关键字

**final关键字主要用在三个地方：变量、方法、类。**

1. **对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。**
2. **当用final修饰一个类时，表明这个类不能被继承。final类中的所有成员方法都会被隐式地指定为final方法。**
3. 使用final方法的原因有两个。第一个原因是把方法锁定，以防任何继承类修改它的含义；第二个原因是效率。在早期的Java实现版本中，会将final方法转为内嵌调用。但是如果方法过于庞大，可能看不到内嵌调用带来的任何性能提升（现在的Java版本已经不需要使用final方法进行这些优化了）。类中所有的private方法都隐式地指定为final。

## static 关键字

**static 关键字主要有以下四种使用场景：**

1. **修饰成员变量和成员方法:** 被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。调用格式：`类名.静态变量名` `类名.静态方法名()`
2. **静态代码块:** 静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—>非静态代码块—>构造方法)。 该类不管创建多少对象，静态代码块只执行一次.
3. **静态内部类（static修饰类的话只能修饰内部类）：** 静态内部类与非静态内部类之间存在一个最大的区别: 非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：1. 它的创建是不需要依赖外围类的创建。2. 它不能使用任何外围类的非static成员变量和方法。
4. **静态导包(用来导入类中的静态资源，1.5之后的新特性):** 格式为：`import static` 这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法。

## this 关键字

this关键字用于引用类的当前实例。 例如：

```
class Manager {
    Employees[] employees;
     
    void manageEmployees() {
        int totalEmp = this.employees.length;
        System.out.println("Total employees: " + totalEmp);
        this.report();
    }
     
    void report() { }
}
```

在上面的示例中，this关键字用于两个地方：

- this.employees.length：访问类Manager的当前实例的变量。
- this.report（）：调用类Manager的当前实例的方法。

此关键字是可选的，这意味着如果上面的示例在不使用此关键字的情况下表现相同。 但是，使用此关键字可能会使代码更易读或易懂。

## super 关键字

super关键字用于从子类访问父类的变量和方法。 例如：

```
public class Super {
    protected int number;
     
    protected showNumber() {
        System.out.println("number = " + number);
    }
}
 
public class Sub extends Super {
    void bar() {
        super.number = 10;
        super.showNumber();
    }
}
```

在上面的例子中，Sub 类访问父类成员变量 number 并调用其其父类 Super 的 `showNumber（）` 方法。

**使用 this 和 super 要注意的问题：**

- 在构造器中使用 `super（）` 调用父类中的其他构造方法时，该语句必须处于构造器的首行，否则编译器会报错。另外，this 调用本类中的其他构造方法时，也要放在首行。
- this、super不能用在static方法中。

**简单解释一下：**

被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享。而 this 代表对本类对象的引用，指向本类对象；而 super 代表对父类对象的引用，指向父类对象；所以， **this和super是属于对象范畴的东西，而静态方法是属于类范畴的东西**。

## 参考

- https://www.codejava.net/java-core/the-java-language/java-keywords
- https://blog.csdn.net/u013393958/article/details/79881037

## static 关键字详解

## static 关键字主要有以下四种使用场景

1. 修饰成员变量和成员方法
2. 静态代码块
3. 修饰类(只能修饰内部类)
4. 静态导包(用来导入类中的静态资源，1.5之后的新特性)

### 修饰成员变量和成员方法(常用)

被 static 修饰的成员属于类，不属于单个这个类的某个对象，被类中所有对象共享，可以并且建议通过类名调用。被static 声明的成员变量属于静态成员变量，静态变量 存放在 Java 内存区域的方法区。

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 Non-Heap（非堆），目的应该是与 Java 堆区分开来。

HotSpot 虚拟机中方法区也常被称为 “永久代”，本质上两者并不等价。仅仅是因为 HotSpot 虚拟机设计团队用永久代来实现方法区而已，这样 HotSpot 虚拟机的垃圾收集器就可以像管理 Java 堆一样管理这部分内存了。但是这并不是一个好主意，因为这样更容易遇到内存溢出问题。

调用格式：

- `类名.静态变量名`
- `类名.静态方法名()`

如果变量或者方法被 private 则代表该属性或者该方法只能在类的内部被访问而不能在类的外部被访问。

测试方法：

```
public class StaticBean {

    String name;
    //静态变量
    static int age;

    public StaticBean(String name) {
        this.name = name;
    }
    //静态方法
    static void SayHello() {
        System.out.println("Hello i am java");
    }
    @Override
    public String toString() {
        return StaticBean{ +
                name=' + name + ''' + age + age +
                '}';
    }
}
public class StaticDemo {

    public static void main(String[] args) {
        StaticBean staticBean = new StaticBean(1);
        StaticBean staticBean2 = new StaticBean(2);
        StaticBean staticBean3 = new StaticBean(3);
        StaticBean staticBean4 = new StaticBean(4);
        StaticBean.age = 33;
        StaticBean{name='1'age33} StaticBean{name='2'age33} StaticBean{name='3'age33} StaticBean{name='4'age33}
        System.out.println(staticBean+ +staticBean2+ +staticBean3+ +staticBean4);
        StaticBean.SayHello();Hello i am java
    }

}
```

### 静态代码块

静态代码块定义在类中方法外, 静态代码块在非静态代码块之前执行(静态代码块—非静态代码块—构造方法)。 该类不管创建多少对象，静态代码块只执行一次.

静态代码块的格式是

```
static {    
语句体;   
}
```

一个类中的静态代码块可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果静态代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次。

![img](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-14/88531075.jpg)

静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问.

### 静态内部类

静态内部类与非静态内部类之间存在一个最大的区别，我们知道非静态内部类在编译完成之后会隐含地保存着一个引用，该引用是指向创建它的外围类，但是静态内部类却没有。没有这个引用就意味着：

1. 它的创建是不需要依赖外围类的创建。
2. 它不能使用任何外围类的非static成员变量和方法。

Example（静态内部类实现单例模式）

```
public class Singleton {
    
    //声明为 private 避免调用默认构造方法创建对象
    private Singleton() {
    }
    
   // 声明为 private 表明静态内部该类只能在该 Singleton 类中被访问
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getUniqueInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

当 Singleton 类加载时，静态内部类 SingletonHolder 没有被加载进内存。只有当调用 `getUniqueInstance() `方法从而触发 `SingletonHolder.INSTANCE` 时 SingletonHolder 才会被加载，此时初始化 INSTANCE 实例，并且 JVM 能确保 INSTANCE 只被实例化一次。

这种方式不仅具有延迟初始化的好处，而且由 JVM 提供了对线程安全的支持。

### 静态导包

格式为：import static

这两个关键字连用可以指定导入某个类中的指定静态资源，并且不需要使用类名调用类中静态成员，可以直接使用类中静态成员变量和成员方法

```
 //将Math中的所有静态资源导入，这时候可以直接使用里面的静态方法，而不用通过类名进行调用
 //如果只想导入单一某个静态方法，只需要将换成对应的方法名即可
 
import static java.lang.Math.;//换成import static java.lang.Math.max;具有一样的效果
 
public class Demo {
  public static void main(String[] args) {
 
    int max = max(1,2);
    System.out.println(max);
  }
}
```

## 补充内容

### 静态方法与非静态方法

静态方法属于类本身，非静态方法属于从该类生成的每个对象。 如果您的方法执行的操作不依赖于其类的各个变量和方法，请将其设置为静态（这将使程序的占用空间更小）。 否则，它应该是非静态的。

Example

```
class Foo {
    int i;
    public Foo(int i) { 
       this.i = i;
    }

    public static String method1() {
       return An example string that doesn't depend on i (an instance variable);
       
    }

    public int method2() {
       return this.i + 1;  Depends on i
    }

}
```

你可以像这样调用静态方法：`Foo.method1（）`。 如果您尝试使用这种方法调用 method2 将失败。 但这样可行：`Foo bar = new Foo（1）;bar.method2（）;`

总结：

- 在外部调用静态方法时，可以使用”类名.方法名”的方式，也可以使用”对象名.方法名”的方式。而实例方法只有后面这种方式。也就是说，调用静态方法可以无需创建对象。
- 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），而不允许访问实例成员变量和实例方法；实例方法则无此限制

### `static{}`静态代码块与`{}`非静态代码块(构造代码块)

相同点： 都是在JVM加载类时且在构造方法执行之前执行，在类中都可以定义多个，定义多个时按定义的顺序执行，一般在代码块中对一些static变量进行赋值。

不同点： 静态代码块在非静态代码块之前执行(静态代码块—非静态代码块—构造方法)。静态代码块只在第一次new执行一次，之后不再执行，而非静态代码块在每new一次就执行一次。 非静态代码块可在普通方法中定义(不过作用不大)；而静态代码块不行。

> 修正 [issue #677](https://github.com/Snailclimb/JavaGuide/issues/677)：静态代码块可能在第一次new的时候执行，但不一定只在第一次new的时候执行。比如通过 `Class.forName("ClassDemo")`创建 Class 对象的时候也会执行。

一般情况下,如果有些代码比如一些项目最常用的变量或对象必须在项目启动的时候就执行的时候,需要使用静态代码块,这种代码是主动执行的。如果我们想要设计不需要创建对象就可以调用类中的方法，例如：Arrays类，Character类，String类等，就需要使用静态方法, 两者的区别是 静态代码块是自动执行的而静态方法是被调用的时候才执行的.

Example：

```
public class Test {
    public Test() {
        System.out.print("默认构造方法！--");
    }

    //非静态代码块
    {
        System.out.print("非静态代码块！--");
    }

    //静态代码块
    static {
        System.out.print("静态代码块！--");
    }

    private static void test() {
        System.out.print("静态方法中的内容! --");
        {
            System.out.print("静态方法中的代码块！--");
        }

    }

    public static void main(String[] args) {
        Test test = new Test();
        Test.test();//静态代码块！--静态方法中的内容! --静态方法中的代码块！--
    }
}
```

上述代码输出：

```
静态代码块！--非静态代码块！--默认构造方法！--静态方法中的内容! --静态方法中的代码块！--
```

当只执行 `Test.test();` 时输出：

```
静态代码块！--静态方法中的内容! --静态方法中的代码块！--
```

当只执行 `Test test = new Test();` 时输出：

```
静态代码块！--非静态代码块！--默认构造方法！--
```

非静态代码块与构造函数的区别是： 非静态代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。也就是说，构造代码块中定义的是不同对象共性的初始化内容。

### 参考

- httpsblog.csdn.netchen13579867831articledetails78995480
- httpwww.cnblogs.comchenssyp3388487.html
- httpwww.cnblogs.comQian123p5713440.html



# Java的引用模型

本文通过探析Java中的引用模型，分析比较强引用、软引用、弱引用、虚引用的概念及使用场景，**知其然且知其所以然**，希望给大家在实际开发实践、学习开源项目提供参考。

## 1 Java的引用

对于Java中的垃圾回收机制来说，对象是否被应该回收的取决于该对象是否被引用。因此，引用也是JVM进行内存管理的一个重要概念。Java中是JVM负责内存的分配和回收，这是它的优点（使用方便，程序不用再像使用C语言那样担心内存），但同时也是它的缺点（不够灵活）。由此，Java提供了引用分级模型，可以**定义Java对象重要性和优先级，提高JVM内存回收的执行效率**。

关于引用的定义，在JDK1.2之前，如果reference类型的数据中存储的数值代表的是另一块内存的起始地址，就称为这块内存代表着一个引用；JDK1.2之后，Java对引用的概念进行了扩充，将引用分为强引用(Strong Reference)、软引用(Soft Reference)、弱引用(Weak Reference)、虚引用(Phantom Reference)四种。

软引用对象和弱应用对象主要用于：当内存空间还足够，则能保存在内存之中；如果内存空间在垃圾收集之后还是非常紧张，则可以抛弃这些对象。很多系统的缓存功能都符合这样的使用场景。

而虚引用对象用于替代不靠谱的finalize方法，可以获取对象的回收事件，来做资源清理工作。

## 2 对象生命周期

### 2.1 无分级引用对象生命周期

前面提到，分层引用的模型是用于内存回收，没有分级引用对象下，一个对象从创建到回收的生命周期可以简单地用下图概括：对象被创建，被使用，有资格被收集，最终被收集，阴影区域表示对象“强可达”时间：



![img](https:////upload-images.jianshu.io/upload_images/5618238-57f389f8a1adadc8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

对象生命周期(无分级引用)

### 2.2 有分级引用对象生命周期

JDK1.2引入java.lang.ref程序包之后，对象的生命周期多了3个阶段，软可达，弱可达，虚可达，这些状态仅适用于符合垃圾回收条件的对象，这些对象处于非强引用阶段，而且需要基于java.lang.ref包中的相关的引用对象类来指示标明。

- 软可达
   软可达对象用SoftReference来指示标明，并没有强引用，垃圾回收器会尽可能长时间地保留对象，但是会在抛出OutOfMemoryError异常之前收集它。
- 弱可达
   弱可达对象用WeakReference来指示标明，并没有强引用或软引用，垃圾回收器会随时回收对象，并不会尝试保留它，但是会在抛出OutOfMemoryError异常之前收集它。

假设垃圾收集器在某个时间点确定对象是弱可达的。 那时它将原子地清除该弱可达引用对象关联的对象。

- 虚可达
   虚可达对象用PhantomReference来指示标明，它已经被标记选中进行垃圾回收并且它的finalizer(如果有)已经运行。在这种情况下，术语“可达”实际上是用词不当，因为您无法访问实际对象。

![img](https:////upload-images.jianshu.io/upload_images/5618238-c77161498febd894.png?imageMogr2/auto-orient/strip|imageView2/2/w/690/format/webp)

分级引用作用时间在对象生命周期中的位置

对象生命周期图中出现三个新的可选状态会造成一些困惑。逻辑顺序上是从强可达到软，弱和虚，最终到回收，但实际的情况取决于程序创建的参考对象。但如果创建WeakReference但不创建SoftReference，则对象直接从强可达到弱到达最终到收集。

## 3 强引用

强引用就是指在程序代码之中普遍存在的，比如下面这段代码中的obj和str都是强引用：



```java
Object obj = new Object();
String str = "hello world";
```

只要强引用还存在，垃圾收集器永远不会回收被引用的对象，即使在内存不足的情况下，JVM即使抛出OutOfMemoryError异常也不会回收这种对象。

实际使用上，可以通过把引用显示赋值为null来中断对象与强引用之前的关联，如果没有任何引用执行对象，垃圾收集器将在合适的时间回收对象。

例如ArrayList类的remove方法中就是通过将引用赋值为null来实现清理工作的：



```java
    /**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

## 4 引用对象

介绍软引用、弱引用和虚引用之前，有必要介绍一下引用对象，
 引用对象是程序代码和其他对象之间的间接层，称为引用对象。每个引用对象都围绕对象的引用构造，并且不能更改引用值。

![img](https:////upload-images.jianshu.io/upload_images/5618238-ee7ecc54ad7dbff2.png?imageMogr2/auto-orient/strip|imageView2/2/w/743/format/webp)

引用对象提供get()来获得其引用值的一个强引用，垃圾收集器可能随时回收引用值所指的对象。
 一旦对象被回收，get()方法将返回null，要正确使用引用对象，下面使用SoftReference(软引用对象)作为参考示例：



```java
    /**
     * 简单使用demo
     */
    private static void simpleUseDemo(){
        List<String> myList = new ArrayList<>();
        SoftReference<List<String>> refObj = new SoftReference<>(myList);

        List<String> list = refObj.get();
        if (null != list) {
            list.add("hello");
        } else {
            // 整个列表已经被垃圾回收了，做其他处理
        }
    }
```

也就是说，使用时：

- 1、必须经常检查引用值是否为null
   垃圾收集器可能随时回收引用对象，如果轻率地使用引用值，迟早会得到一个NullPointerException。
- 2、必须使用强引用来指向引用对象返回的值
   垃圾收集器可能在任何时间回收引用对象，即使在一个表达式中间。



```java
    /**
     * 正确使用引用对象demo
     */
    private static void trueUseRefObjDemo(){
        List<String> myList = new ArrayList<>();
        SoftReference<List<String>> refObj = new SoftReference<>(myList);

        // 正确的使用，使用强引用指向对象保证获得对象之后不会被回收
        List<String> list = refObj.get();
        if (null != list) {
            list.add("hello");
        } else {
            // 整个列表已经被垃圾回收了，做其他处理
        }
    }

    /**
     * 错误使用引用对象demo
     */
    private static void falseUseRefObjDemo(){
        List<String> myList = new ArrayList<>();
        SoftReference<List<String>> refObj = new SoftReference<>(myList);

        // XXX 错误的使用，在检查对象非空到使用对象期间，对象可能已经被回收
        // 可能出现空指针异常
        if (null != refObj.get()) {
            refObj.get().add("hello");
        }
    }
```

- 3、必须持有引用对象的强引用
   如果创建引用对象，没有持有对象的强引用，那么引用对象本身将被垃圾收集器回收。
- 4、当引用值没有被其他强引用指向时，软引用、弱引用和虚引用才会发挥作用，引用对象的存在就是为了方便追踪并高效垃圾回收。

## 5 软引用、弱引用和虚引用

引用对象的3个重要实现类位于java.lang.ref包下，分别是软引用SoftReference、弱引用WeakReference和虚引用PhantomReference。

### 5.1 软引用

软引用用来描述一些还有用但非必需的对象。对于软引用关联着的对象，在系统将要发生抛出OutOfMemoryError异常之前，将会把这些对象列入回收范围之内进行第二次回收。如果这次回收还没有足够的内存，才会抛出OutOfMemoryError异常。在JDK1.2之后，提供了SoftReference类来实现软引用。

下面是一个使用示例：



```java
import java.lang.ref.SoftReference;

public class SoftRefDemo {
    public static void main(String[] args) {
        SoftReference<String> sr = new SoftReference<>( new String("hello world "));
        // hello world
        System.out.println(sr.get());
    }
}
```

JDK文档中提到：软引用适用于对内存敏感的缓存：每个缓存对象都是通过访问的 SoftReference，如果JVM决定需要内存空间，那么它将清除回收部分或全部软引用对应的对象。如果它不需要空间，则SoftReference指示对象保留在堆中，并且可以通过程序代码访问。在这种情况下，当它们被积极使用时，它们被强引用，否则会被软引用。如果清除了软引用，则需要刷新缓存。

实际使用上，要除非缓存的对象非常大，每个数量级为几千字节，才值得考虑使用软引用对象。例如：实现一个文件服务器，它需要定期检索相同的文件，或者需要缓存大型对象图。如果对象很小，必须清除很多对象才能产生影响，那么不建议使用，因为清除软引用对象会增加整个过程的开销。

### 5.2 弱引用

弱引用也是用来描述非必需对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发送之前。**当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象**。

在JDK1.2之后，提供了WeakReference类来实现弱引用。



```java
    /**
     * 简单使用弱引用demo
     */
    private static void simpleUseWeakRefDemo(){
        WeakReference<String> sr = new WeakReference<>(new String("hello world " ));
        // before gc -> hello world 
        System.out.println("before gc -> " + sr.get());

        // 通知JVM的gc进行垃圾回收
        System.gc();
        // after gc -> null
        System.out.println("after gc -> " + sr.get());
    }
```

可以看到被弱引用关联的对象，在gc之后被回收掉。
 有意思的地方是，如果把上面代码中的：



```java
WeakReference<String> sr = new WeakReference<>(new String("hello world "));
```

改为



```java
WeakReference<String> sr = new WeakReference<>("hello world ");
```

程序将输出



```rust
before gc -> hello world 
after gc -> hello world 
```

这是因为使用Java的String直接赋值和使用new区别在于：

- new 会在堆区创建一个可以被正常回收的对象。
- String直接赋值，例如：String  str = String( "Hello");
   JVM首先在string池内里面看找不找到字符串 "Hello"，找到，不做任何事情；
   否则，创建新的String对象，放到String常量池里面(常量池Hotspot1.7之前存于永生代，Hotspot1.7和1.7之后的版本存于堆区，通常不会被gc回收)。同时，由于遇到了new，还会在内存上（不是String常量池里面）创建String对象存储 "Hello"，并将内存上的（不是String池内的）String对象返回给str。

**WeakHashMap**
 为了更方便使用弱引用，Java还提供了WeakHashMap，功能类似HashMap，内部实现是用弱引用对key进行包装，当某个key对象没有任何强引用指向，gc会自动回收key和value对象。



```java
    /**
     *  weakHashMap使用demo
     */
    private static void weakHashMapDemo(){
        WeakHashMap<String,String> weakHashMap = new WeakHashMap<>();
        String key1 = new String("key1");
        String key2 = new String("key2");
        String key3 = new String("key3");
        weakHashMap.put(key1, "value1");
        weakHashMap.put(key2, "value2");
        weakHashMap.put(key3, "value3");

        // 使没有任何强引用指向key1
        key1 = null;

        System.out.println("before gc weakHashMap = " + weakHashMap + " , size=" + weakHashMap.size());

        // 通知JVM的gc进行垃圾回收
        System.gc();
        System.out.println("after gc weakHashMap = " + weakHashMap + " , size="+ weakHashMap.size());
    }
```

程序输出：



```java
before: gc weakHashMap = {key1=value1, key2=value2, key3=value3} , size=3
after: gc weakHashMap = {key2=value2, key3=value3} , size=2
```

WeakHashMap比较适用于缓存的场景，例如Tomcat的缓存就用到。

### 5.3  引用队列

介绍虚引用之前，先介绍引用队列：
 在使用引用对象时，通过判断get()方法返回的值是否为null来判断对象是否已经被回收，当这样做并不是非常高效，特别是当我们有很多引用对象，如果想找出哪些对象已经被回收，需要遍历所有所有对象。

更好的方案是使用引用队列，在构造引用对象时与队列关联，当gc（垃圾回收线程）准备回收一个对象时，如果发现它还仅有软引用(或弱引用，或虚引用)指向它，就会在回收该对象之前，把这个软引用（或弱引用，或虚引用）加入到与之关联的引用队列（ReferenceQueue）中。

**如果一个软引用（或弱引用，或虚引用）对象本身在引用队列中，就说明该引用对象所指向的对象被回收了**，所以要找出所有被回收的对象，只需要遍历引用队列。

当软引用（或弱引用，或虚引用）对象所指向的对象被回收了，那么这个引用对象本身就没有价值了，如果程序中存在大量的这类对象（注意，我们创建的软引用、弱引用、虚引用对象本身是个强引用，不会自动被gc回收），就会浪费内存。因此我们这就可以手动回收位于引用队列中的引用对象本身。



```java
    /**
     * 引用队列demo
     */
    private static void refQueueDemo() {
        ReferenceQueue<String> refQueue = new ReferenceQueue<>();

        // 用于检查引用队列中的引用值被回收
        Thread checkRefQueueThread = new Thread(() -> {
            while (true) {
                Reference<? extends String> clearRef = refQueue.poll();
                if (null != clearRef) {
                    System.out
                            .println("引用对象被回收, ref = " + clearRef + ", value = " + clearRef.get());
                }
            }
        });
        checkRefQueueThread.start();

        WeakReference<String> weakRef1 = new WeakReference<>(new String("value1"), refQueue);
        WeakReference<String> weakRef2 = new WeakReference<>(new String("value2"), refQueue);
        WeakReference<String> weakRef3 = new WeakReference<>(new String("value3"), refQueue);

        System.out.println("ref1 value = " + weakRef1.get() + ", ref2 value = " + weakRef2.get()
                + ", ref3 value = " + weakRef3.get());

        System.out.println("开始通知JVM的gc进行垃圾回收");
        // 通知JVM的gc进行垃圾回收
        System.gc();
    }
```

程序输出：



```java
ref1 value = value1, ref2 value = value2, ref3 value = value3
开始通知JVM的gc进行垃圾回收
引用对象被回收, ref = java.lang.ref.WeakReference@48c6cd96, value=null
引用对象被回收, ref = java.lang.ref.WeakReference@46013afe, value=null
引用对象被回收, ref = java.lang.ref.WeakReference@423ea6e6, value=null
```

### 5.4 虚引用

虚引用也称为幽灵引用或者幻影引用，不同于软引用和弱引用，虚引用不用于访问引用对象所指示的对象，相反，**通过不断轮询虚引用对象关联的引用队列，可以得到对象回收事件**。一个对象是否有虚引用的存在，完全不会对其生产时间构成影响，也无法通过虚引用来取得一个对象实例。虽然这看起来毫无意义，但它实际上可以用来做对象回收时**资源清理、释放**，它比finalize更灵活，我们可以基于虚引用做更安全可靠的对象关联的资源回收。

- finalize的问题
  - Java语言规范并不保证finalize方法会被及时地执行、而且根本不会保证它们会被执行
     如果可用内存没有被耗尽，垃圾收集器不会运行，finalize方法也不会被执行。
  - 性能问题
     JVM通常在单独的低优先级线程中完成finalize的执行。
  - 对象再生问题
     finalize方法中，可将待回收对象赋值给GC Roots可达的对象引用，从而达到对象再生的目的。

针对不靠谱finalize方法，完全可以使用虚引用来实现。在JDK1.2之后，提供了PhantomReference类来实现虚引用。

下面是简单的使用例子，通过访问引用队列可以得到对象的回收事件：



```java
    /**
     * 简单使用虚引用demo
     * 虚引用在实现一个对象被回收之前必须做清理操作是很有用的,比finalize()方法更灵活
     */
    private static void simpleUsePhantomRefDemo() throws InterruptedException {
        Object obj = new Object();
        ReferenceQueue<Object> refQueue = new ReferenceQueue<>();
        PhantomReference<Object> phantomRef = new PhantomReference<>(obj, refQueue);

        // null
        System.out.println(phantomRef.get());
        // null
        System.out.println(refQueue.poll());

        obj = null;
        // 通知JVM的gc进行垃圾回收
        System.gc();

        // null, 调用phantomRef.get()不管在什么情况下会一直返回null
        System.out.println(phantomRef.get());

        // 当GC发现了虚引用，GC会将phantomRef插入进我们之前创建时传入的refQueue队列
        // 注意，此时phantomRef对象，并没有被GC回收，在我们显式地调用refQueue.poll返回phantomRef之后
        // 当GC第二次发现虚引用，而此时JVM将phantomRef插入到refQueue会插入失败，此时GC才会对phantomRef对象进行回收
        Thread.sleep(200);
        Reference<?> pollObj = refQueue.poll();
        // java.lang.ref.PhantomReference@1540e19d
        System.out.println(pollObj);
        if (null != pollObj) {
            // 进行资源回收的操作
        }
    }
```

比较常见的，可以基于虚引用实现JDBC连接池，锁的释放等场景。
 以连接池为例，调用方正常情况下使用完连接，需要把连接释放回池中，但是不可避免有可能程序有bug，造成连接没有正常释放回池中。基于虚引用对Connection对象进行包装，并关联引用队列，就可以通过轮询引用队列检查哪些连接对象已经被GC回收，释放相关连接资源。[具体实现已上传github的caison-blog-demo仓库](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fcaison%2Fcaison-blog-demo)。

## 6 总结

对比一下几种引用对象的不同：

| 引用类型 | GC回收时间 |    常见用途    |    生存时间    |
| :------- | :--------: | :------------: | :------------: |
| 强引用   |    永不    | 对象的一般状态 | JVM停止运行时  |
| 软引用   | 内存不足时 |    对象缓存    | 内存不足时终止 |
| 弱引用   |    GC时    |    对象缓存    |    GC后终止    |

虚引用，配合引用队列使用，通过不断轮询引用队列获取对象回收事件。

虽然引用对象是一个非常有用的工具来管理你的内存消耗，但有时它们是不够的，或者是过度设计的 。例如，使用一个Map来缓存从数据库中读取的数据。虽然可以使用弱引用来作为缓存，但最终程序需要运行一定量的内存。如果不能给它足够实际足够的资源完成任何工作，那么错误恢复机制有多强大也没有用。

当遇到OutOfMemoryError错误，第一反应是要弄清楚它为什么会发生，也许真的是程序有bug，也许是可用内存设置的太低。

在开发过程中，应该制定程序具体的使用内存大小，而已要关注实际使用中用了多少内存。大多数应用程序在实际运行负载下，程序的内存占用会达到稳定状态，可以用此来作为参考来设置合理的堆大小。如果程序的内存使用量随着时间的推移而上升，很有可能是因为当对象不再使用时仍然拥有对对象的强引用。引用对象在这里可能会有所帮助，但更有可能是把它当做一个bug来进行修复。

文章所有涉及源码已经上传github，地址：[https://github.com/caison/caison-blog-demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fcaison%2Fcaison-blog-demo)

## 参考

[Java引用类型原理剖析](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Ffarmerjohngit%2Fmyblog%2Fissues%2F10)

[Java Reference Objects](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.kdgregory.com%2Findex.php%3Fpage%3Djava.refobj)

《深入理解Java虚拟机——JVM高级特性与最佳实践(第2版)》

[Java核心技术36讲](https://links.jianshu.com/go?to=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Fintro%2F82%3Fcode%3Dw8EZ6RGOQApZJ5tpAzP8dRzeVHxZ4q%2FfOdSbSZzbkhc%3D)

[深入拆解 Java 虚拟机](https://links.jianshu.com/go?to=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Fintro%2F108%3Fcode%3DXamkmJYooKBPKe8hT7otClsFDeVHb6rptMYHTdgiRPU%3D)

[Java 如何有效地避免OOM：善于利用软引用和弱引用](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.cnblogs.com%2Fdolphin0520%2Fp%2F3784171.html)

[Java幽灵引用的作用](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fimzoer%2Farticle%2Fdetails%2F8044900)

[oracle官方文档](https://links.jianshu.com/go?to=https%3A%2F%2Fdocs.oracle.com%2Fjavase%2F7%2Fdocs%2Fapi%2Fjava%2Flang%2Fref%2Fpackage-summary.html%23package_description)



# 日志打印规范及技巧

### 一、日志打印级别

- **DEBUG（调试）**
   **开发调试日志**。一般来说，在系统实际运行过程中，不会输出该级别的日志。因此，开发人员可以打印任何自己觉得有利于了解系统运行状态的东东。不过很多场景下，过多的DEBUG日志，并不是好事，建议是按照业务逻辑的走向打印。
- **INFO（通知）**
   INFO日志级别主要用于记录系统运行状态等关联信息。该日志级别，**常用于反馈系统当前状态给最终用户**。所以，在这里输出的信息，应该对最终用户具有实际意义，也就是最终用户要能够看得明白是什么意思才行。
- **WARN（警告）**
   WARN日志常用来表示**系统模块发生问题，但并不影响系统运行**。 此时，进行一些修复性的工作，还能把系统恢复到正常的状态。
- **ERROR（错误）**
   **此信息输出后，主体系统核心模块正常工作，需要修复才能正常工作**。 就是说可以进行一些修复性的工作，但无法确定系统会正常的工作下去，系统在以后的某个阶段，很可能会因为当前的这个问题，导致一个无法修复的错误（例如宕机），但也可能一直工作到停止也不出现严重问题。

### 二、日志打印规范

> **1. 【强制**】应用中不可直接使用日志系统 （Log 4 j 、 Logback） 中的 API ，而应依赖使用日志框架
>  SLF 4 J 中的 API ，使用门面模式的日志框架，有利于维护和各个类的日志处理方式统一。



```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
private static final Logger logger = LoggerFactory.getLogger(Abc.class);
```

> 1. 【强制】日志文件推荐至少保存 15 天，因为有些异常具备以“周”为频次发生的特点。

可以结合实际业务需求，基于按天，和按照容量配置appender。例如，按天保存接口对接基本关键数值记录日志，按照容量保存接口对接详细日志。

> 1. 【强制】应用中的扩展日志 （ 如打点、临时监控、访问日志等 ）
>
> - **命名方式：**appName _ logType _ logName . log 。
> - **日志类型（ logType），推荐分类有stats / desc / monitor / visit 等**。
> - **日志描述（logName）**。
>    这种命名的好处：通过文件名就可知道日志文件属于什么应用，什么类型，什么目的，也有利于归类查找。推荐对日志进行分类，如将错误日志和业务日志分开存放，便于开发人员查看，也便于通过日志对系统进行及时监控。
>    **正例： mppserver 应用中单独监控时区转换异常，如：mppserver _ monitor _ timeZoneConvert . log**

> 1. 【强制】对 trace / debug / info 级别的日志输出，必须使用条件输出形式或者使用占位符的方式。
>     说明： logger . debug( " Processing trade with id : " +  id + "  and symbol : " +  symbol)。如果日志级别是 warn ，上述日志不会打印，但是会执行字符串拼接操作，如果 symbol 是对象，会执行 toString() 方法，浪费了系统资源，执行了上述操作，最终日志却没有打印。
>     正例： （ 占位符 ）
>     logger.debug("Processing trade with id: {} and symbol : {} ", id, symbol);

> 1. 【强制】避免重复打印日志，浪费磁盘空间，务必在 log 4 j . xml 中设置 additivity = false 。
>     正例： <logger name="com.taobao.dubbo.config" additivity="false">

> 1. 【强制】异常信息应该包括两类信息：案发现场信息和异常堆栈信息。如果不处理，那么通过关键字 throws 往上抛出。
>     正例： logger.error(各类参数或者对象 toString  +  "_"  + e.getMessage(), e);

> 1. 【推荐】谨慎地记录日志。生产环境禁止输出 debug 日志 ； 有选择地输出 info 日志 ； 如果使用 warn 来记录刚上线时的业务行为信息，一定要注意日志输出量的问题，避免把服务器磁盘撑爆，并记得及时删除这些观察日志。
>     说明：大量地输出无效日志，不利于系统性能提升，也不利于快速定位错误点。记录日志时请思考：这些日志真的有人看吗？看到这条日志你能做什么？能不能给问题排查带来好处？

> 1. 【参考】可以使用 warn 日志级别来记录用户输入参数错误的情况，避免用户投诉时，无所适从。注意日志输出的级别， error 级别只记录系统逻辑出错、异常等重要的错误信息。如非必要，请不要在此场景打出 error 级别。

### 三、日志打印技巧

#### 问题排查的日志

- **对接外部的调用封装**：
   程序中对接外部系统与模块的依赖调用前后都记下日志，方便接口调试。出问题时也可以很快理清是哪块的问题

```java
LOG.debug("Calling external system:" + parameters);  
Object result = null;  
try {  
    result = callRemoteSystem(params);  
    LOG.debug("Called successfully. result is " + result);  
} catch (Exception e) {  
    LOG.warn("Failed at calling xxx system . exception : " + e);  
}  
```

- 状态变化：
   程序中重要的状态信息的变化应该记录下来，方便查问题时还原现场，推断程序运行过程

```java
boolean isRunning = true;  
LOG.info("System is running");  
//...  
isRunning = false;  
LOG.info("System was interrupted by " + Thread.currentThread().getName()); 
```

- 系统入口与出口：



```java
这个粒度可以是重要方法级或模块级。记录它的输入与输出，方便定位 
void execute(Object input) {  
    LOG.debug("Invoke parames : " + input);  
    Object result = null;  
      
    //business logical  
      
    LOG.debug("Method result : " + result);  
}  
```

- 业务异常：
   任何业务异常都应该记下来



```java
try {  
    //business logical  
} catch (IOException e) {  
    LOG.warn("Description xxx" , e);  
} catch (BusinessException e) {  
    LOG.warn("Let me know anything"，e);  
} catch (Exception e) {  
    LOG.error("Description xxx", e);  
}  
  
void invoke(Object primaryParam) {  
    if (primaryParam == null) {  
        LOG.warn(原因...);  
        return;  
    }  
} 
```

- 非预期执行：
   为程序在“有可能”执行到的地方打印日志。如果我想删除一个文件，结果返回成功。但事实上，那个文件在你想删除之前就不存在了。最终结果是一致的，但程序得让我们知道这种情况，要查清为什么文件在删除之前就已经不存在呢



```java
int myValue = xxxx;  
int absResult = Math.abs(myValue);  
if (absResult < 0) {  
    LOG.info("Original int " + myValue + "has nagetive abs " + absResult);  
}  
```

- 很少出现的else情况：
   else可能吞掉你的请求，或是赋予难以理解的最终结果



```java
Object result = null;  
if (running) {  
   result = xxx;  
} else {  
   result = yyy;  
   LOG.debug("System does not running, we change the final result");  
}  
```

#### 程序运行状态的日志

程序在运行时就像一个机器人，我们可以从它的日志看出它正在做什么，是不是按预期的设计在做，所以这些正常的运行状态是要有的。

- 程序运行时间：



```java
long startTime = System.currentTime();  
  
// business logical  
  
LOG.info("execution cost : " + (System.currentTime() - startTime) + "ms");　 
```

大批量数据的执行进度：



```java
LOG.debug("current progress: " + (currentPos * 100 / totalAmount) + "%"); 
```

关键变量及正在做哪些重要的事情：
 执行关键的逻辑，做IO操作等等



```java
String getJVMPid() {  
   String pid = "";  
   // Obtains JVM process ID  
   LOG.info("JVM pid is " + pid);  
   return pid;  
}  
  
void invokeRemoteMethod(Object params) {  
    LOG.info("Calling remote method : " + params);  
    //Calling remote server  
} 
```

### 四、需要规避的问题

- 频繁打印大数据量日志：
   当日志产生的速度大于日志文件写磁盘的速度，会导致日志内容积压在内存中，导致内存泄漏。
- 无意义的Log：
   日志不包含有意义的信息: 你肯定想知道的是哪个文件不存在吧



```java
File file = new File("xxx");  
if (!file.isExist()) {  
    LOG.warn("File does not exist"); //Useless message  
} 
```

- 混淆信息的Log：
   日志应该是清晰准确的: 当看到日志的时候，你知道是因为连接池取不到连接导致的问题么？



```java
Connection connection = ConnectionFactory.getConnection();  
if (connection == null) {  
    LOG.warn("System initialized unsuccessfully");  
}  
```

参考：
 《阿里巴巴开发手册》
 [Logger日志级别说明及设置方法、说明](http://blog.csdn.net/rogger_chen/article/details/50587920)
 [闲谈程序中如何打印log](http://langyu.iteye.com/blog/1147992)





# Java内存模型

这篇文章主要介绍模型产生的问题背景，解决的问题，处理思路，相关实现规则，环环相扣，希望读者看完这篇文章后能对Java内存模型体系产生一个相对清晰的理解，知其然而知其所以然。

## 1 内存模型产生背景

在介绍Java内存模型之前，我们先了解一下物理计算机中的并发问题，理解这些问题可以搞清楚内存模型产生的背景。物理机遇到的并发问题与虚拟机中的情况有不少相似之处，物理机的解决方案对虚拟机的实现有相当的参考意义。

### 物理机的并发问题

- **硬件的效率问题**

计算机处理器处理绝大多数运行任务都不可能只靠处理器“计算”就能完成，处理器至少需要与**内存交互**，如读取运算数据、存储运算结果，这个I/O操作很难消除(无法仅靠寄存器完成所有运算任务)。

由于计算机的存储设备与处理器的运算速度有几个数量级的差距，为了避免处理器等待缓慢的内存读写操作完成，现代计算机系统通过加入一层读写速度尽可能接近处理器运算速度的高速缓存。缓存作为内存和处理器之间的缓冲：将运算需要使用到的数据复制到缓存中，让运算能快速运行，当运算结束后再从缓存同步回内存之中。



![img](https:////upload-images.jianshu.io/upload_images/5618238-dcf0a15c8f1d6b83.png?imageMogr2/auto-orient/strip|imageView2/2/w/352/format/webp)

CPU高速缓存

- **缓存一致性问题**

基于高速缓存的存储系统交互很好地解决了处理器与内存速度的矛盾，但是也为计算机系统带来更高的复杂度，因为引入了一个新问题：**缓存一致性。**

在多处理器的系统中(或者单处理器多核的系统)，每个处理器(每个核)都有自己的高速缓存，而它们有共享同一主内存(Main Memory)。当多个处理器的运算任务都涉及同一块主内存区域时，将可能导致各自的缓存数据不一致。
 为此，需要各个处理器访问缓存时都遵循一些协议，在读写时要根据协议进行操作，来维护缓存的一致性。

![img](https:////upload-images.jianshu.io/upload_images/5618238-072900e229e4726b.png?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp)

缓存一致性

- **代码乱序执行优化问题**

为了使得处理器内部的运算单元尽量被充分利用，提高运算效率，处理器可能会对输入的代码进行乱序执行，处理器会在计算之后将乱序执行的结果重组，**乱序优化可以保证在单线程下该执行结果与顺序执行的结果是一致的**，但不保证程序中各个语句计算的先后顺序与输入代码中的顺序一致。

![img](https:////upload-images.jianshu.io/upload_images/5618238-08eef6d06ba90a7f.png?imageMogr2/auto-orient/strip|imageView2/2/w/611/format/webp)

代码执行乱序优化

乱序执行技术是处理器为提高运算速度而做出违背代码原有顺序的优化。在单核时代，处理器保证做出的优化不会导致执行结果远离预期目标，但在多核环境下却并非如此。

多核环境下， 如果存在一个核的计算任务依赖另一个核 计的算任务的中间结果，而且对相关数据读写没做任何防护措施，那么其顺序性并不能靠代码的先后顺序来保证，处理器最终得出的结果和我们逻辑得到的结果可能会大不相同。

![img](https:////upload-images.jianshu.io/upload_images/5618238-68688fe88106f3f1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

代码乱序执行优化的问题

以上图为例进行说明：CPU的core2中的逻辑B依赖core1中的逻辑A先执行

- 正常情况下，逻辑A执行完之后再执行逻辑B。
- 在处理器乱序执行优化情况下，有可能导致flag提前被设置为true，导致逻辑B先于逻辑A执行。

## 2 Java内存模型的组成分析

### 内存模型概念

为了更好解决上面提到系列问题，内存模型被总结提出，我们可以把内存模型理解为**在特定操作协议下，对特定的内存或高速缓存进行读写访问的过程抽象**。

不同架构的物理计算机可以有不一样的内存模型，Java虚拟机也有自己的内存模型。Java虚拟机规范中试图定义一种Java内存模型（Java Memory Model，简称JMM）来**屏蔽掉各种硬件和操作系统的内存访问差异**，以实现**让Java程序在各种平台下都能达到一致的内存访问效果**，不必因为不同平台上的物理机的内存模型的差异，对各平台定制化开发程序。

更具体一点说，Java内存模型提出目标在于，**定义程序中各个变量的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节**。此处的变量(Variables)与Java编程中所说的变量有所区别，它包括了实例字段、静态字段和构成数值对象的元素，但不包括局部变量与方法参数，因为后者是线程私有的。(如果局部变量是一个reference类型，它引用的对象在Java堆中可被各个线程共享，但是reference本身在Java栈的局部变量表中，它是线程私有的)。

### Java内存模型的组成

- 主内存
   Java内存模型规定了所有变量都存储在主内存(Main Memory)中（此处的主内存与介绍物理硬件的主内存名字一样，两者可以互相类比，但此处仅是虚拟机内存的一部分）。
- 工作内存
   每条线程都有自己的工作内存(Working Memory，又称本地内存，可与前面介绍的处理器高速缓存类比)，线程的工作内存中保存了该线程使用到的变量的主内存中的共享变量的副本拷贝。**工作内存是 JMM 的一个抽象概念，并不真实存在**。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。

Java内存模型抽象示意图如下：

![img](https:////upload-images.jianshu.io/upload_images/5618238-2f548f0663280dd7.png?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp)

Java内存模型抽象示意图

### JVM内存操作的并发问题

结合前面介绍的物理机的处理器处理内存的问题，可以类比总结出JVM内存操作的问题，下面介绍的Java内存模型的执行处理将围绕解决这2个问题展开：

- **1 工作内存数据一致性**
   各个线程操作数据时会保存使用到的主内存中的共享变量副本，当多个线程的运算任务都涉及同一个共享变量时，将导致各自的的共享变量副本不一致，如果真的发生这种情况，数据同步回主内存以谁的副本数据为准？
   Java内存模型主要通过一系列的数据同步协议、规则来保证数据的一致性，后面再详细介绍。
- **2 指令重排序优化**
   Java中重排序通常是编译器或运行时环境为了优化程序性能而采取的对指令进行重新排序执行的一种手段。重排序分为两类：**编译期重排序和运行期重排序**，分别对应编译时和运行时环境。
   同样的，指令重排序不是随意重排序，它需要满足以下两个条件：
  - 1 在单线程环境下不能改变程序运行的结果
     即时编译器（和处理器）需要保证程序能够遵守 as-if-serial 属性。通俗地说，就是在单线程情况下，要给程序一个顺序执行的假象。即经过重排序的执行结果要与顺序执行的结果保持一致。
  - 2 存在数据依赖关系的不允许重排序

多线程环境下，如果线程处理逻辑之间存在依赖关系，有可能因为指令重排序导致运行结果与预期不同，后面再展开Java内存模型如何解决这种情况。

## 3 Java内存间的交互操作

在理解Java内存模型的系列协议、特殊规则之前，我们先理解Java中内存间的交互操作。

### 交互操作流程

为了更好理解内存的交互操作，以线程通信为例，我们看看具体如何进行线程间值的同步：

![img](https:////upload-images.jianshu.io/upload_images/5618238-41e25652ff2651c9.png?imageMogr2/auto-orient/strip|imageView2/2/w/757/format/webp)

线程间交互操作

线程1和线程2都有主内存中共享变量x的副本，初始时，这3个内存中x的值都为0。线程1中更新x的值为1之后同步到线程2主要涉及2个步骤：

- 1 线程1把线程工作内存中更新过的x的值刷新到主内存中
- 2 线程2到主内存中读取线程1之前已更新过的x变量

从整体上看，这2个步骤是线程1在向线程2发消息，这个通信过程必须经过主内存。线程对变量的所有操作（读取，赋值）都必须在**工作内存中**进行。不同线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递均需要通过主内存来完成，实现各个线程共享变量的可见性。

### 内存交互的基本操作

关于主内存与工作内存之间的具体交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步回主内存之类的实现细节，Java内存模型中定义了下面介绍8种操作来完成。

虚拟机实现时必须保证下面介绍的每种操作都是原子的，不可再分的(对于double和long型的变量来说，load、store、read、和write操作在某些平台上允许有例外，后面会介绍）。

### 8种基本操作

![img](https:////upload-images.jianshu.io/upload_images/5618238-c34a03dc8b12f468.png?imageMogr2/auto-orient/strip|imageView2/2/w/732/format/webp)

8种基本操作

- lock (锁定)
   作用于**主内存**的变量，它把一个变量标识为一条线程独占的状态。
- unlock (解锁)
   作用于**主内存**的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
- read (读取)
   作用于**主内存**的变量，它把一个变量的值从主内存**传输**到线程的工作内存中，以便随后的load动作使用。
- load (载入)
   作用于**工作内存**的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
- use (使用)
   作用于**工作内存**的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时就会执行这个操作。
- assign (赋值)
   作用于**工作内存**的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- store (存储)
   作用于**工作内存**的变量，它把工作内存中一个变量的值传送到主内存中，以便随后write操作使用。
- write (写入)
   作用于**主内存**的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。

## 4 Java内存模型运行规则

### 4.1 内存交互基本操作的3个特性

在介绍内存的交互的具体的8种基本操作之前，有必要先介绍一下操作的3个特性，Java内存模型是围绕着在并发过程中如何处理这3个特性来建立的，这里先给出定义和基本实现的简单介绍，后面会逐步展开分析。

- **原子性(Atomicity)**
   **即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行**。即使在多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程所干扰。
- **可见性(Visibility)**
   **是指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值**。
   正如上面“交互操作流程”中所说明的一样，JMM是通过在线程1变量工作内存修改后将新值同步回主内存，线程2在变量读取前从主内存刷新变量值，这种**依赖主内存作为传递媒介**的方式来实现可见性。
- **有序性(Ordering)**
   有序性规则表现在以下两种场景: 线程内和线程间
  - 线程内
     从某个线程的角度看方法的执行，指令会按照一种叫“串行”（as-if-serial）的方式执行，此种方式已经应用于顺序编程语言。
  - 线程间
     这个线程“观察”到其他线程并发地执行非同步的代码时，由于指令重排序优化，任何代码都有可能交叉执行。唯一起作用的约束是：对于同步方法，同步块(synchronized关键字修饰)以及volatile字段的操作仍维持相对有序。

Java内存模型的一系列运行规则看起来有点繁琐，但总结起来，是**围绕原子性、可见性、有序性特征建立**。归根究底，是为实现共享变量的在多个线程的工作内存的**数据一致性**，多线程并发，指令重排序优化的环境中程序能如预期运行。

### 4.2 happens-before关系

介绍系列规则之前，首先了解一下happens-before关系：用于描述下2个操作的内存可见性：**如果操作A happens-before 操作B，那么A的结果对B可见**。happens-before关系的分析需要分为**单线程和多线程**的情况：

- **单线程下的 happens-before**
   字节码的先后顺序天然包含happens-before关系：因为单线程内共享一份工作内存，不存在数据一致性的问题。
   在程序控制流路径中靠前的字节码 happens-before 靠后的字节码，即靠前的字节码执行完之后操作结果对靠后的字节码可见。然而，这并不意味着前者一定在后者之前执行。实际上，如果后者不依赖前者的运行结果，那么它们可能会被重排序。
- **多线程下的 happens-before**
   多线程由于每个线程有共享变量的副本，如果没有对共享变量做同步处理，线程1更新执行操作A共享变量的值之后，线程2开始执行操作B，此时操作A产生的结果对操作B不一定可见。

为了方便程序开发，Java内存模型实现了下述支持happens-before关系的操作：

- 程序次序规则
   一个线程内，按照代码顺序，书写在前面的操作 happens-before 书写在后面的操作。
- 锁定规则
   一个unLock操作 happens-before 后面对同一个锁的lock操作。
- volatile变量规则
   对一个变量的写操作 happens-before 后面对这个变量的读操作。
- 传递规则
   如果操作A happens-before  操作B，而操作B又 happens-before 操作C，则可以得出操作A happens-before 操作C。
- 线程启动规则
   Thread对象的start()方法 happens-before 此线程的每个一个动作。
- 线程中断规则
   对线程interrupt()方法的调用 happens-before 被中断线程的代码检测到中断事件的发生。
- 线程终结规则
   线程中所有的操作都 happens-before 线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行。
- 对象终结规则
   一个对象的初始化完成 happens-before 他的finalize()方法的开始

### 4.3 内存屏障

Java中如何保证底层操作的有序性和可见性？可以通过内存屏障。

内存屏障是被插入两个CPU指令之间的一种指令，用来禁止处理器指令发生重排序（像屏障一样），从而保障**有序性**的。另外，为了达到屏障的效果，它也会使处理器写入、读取值之前，将主内存的值写入高速缓存，清空无效队列，从而保障**可见性**。

举个例子：



```cpp
Store1; 
Store2;   
Load1;   
StoreLoad;  //内存屏障
Store3;   
Load2;   
Load3;
```

对于上面的一组CPU指令（Store表示写入指令，Load表示读取指令），StoreLoad屏障之前的Store指令无法与StoreLoad屏障之后的Load指令进行交换位置，即**重排序**。但是StoreLoad屏障之前和之后的指令是可以互换位置的，即Store1可以和Store2互换，Load2可以和Load3互换。

常见有4种屏障

- LoadLoad屏障：
   对于这样的语句 Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
- StoreStore屏障：
   对于这样的语句 Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
- LoadStore屏障：
   对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被执行前，保证Load1要读取的数据被读取完毕。
- StoreLoad屏障：
   对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的（冲刷写缓冲器，清空无效化队列）。在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能。

Java中对内存屏障的使用在一般的代码中不太容易见到，常见的有volatile和synchronized关键字修饰的代码块(后面再展开介绍)，还可以通过Unsafe这个类来使用内存屏障。

### 4.4 8种操作同步的规则

JMM在执行前面介绍8种基本操作时，为了保证内存间数据一致性，JMM中规定需要满足以下规则：

- 规则1：如果要把一个变量从主内存中复制到工作内存，就需要按顺序的执行 read 和 load 操作，如果把变量从工作内存中同步回主内存中，就要按顺序的执行 store 和 write 操作。但 Java 内存模型只要求上述操作必须按顺序执行，而没有保证必须是连续执行。
- 规则2：不允许 read 和 load、store 和 write 操作之一单独出现。
- 规则3：不允许一个线程丢弃它的最近 assign 的操作，即变量在工作内存中改变了之后必须同步到主内存中。
- 规则4：不允许一个线程无原因的（没有发生过任何 assign 操作）把数据从工作内存同步回主内存中。
- 规则5：一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load 或 assign ）的变量。即就是对一个变量实施 use 和 store 操作之前，必须先执行过了 load 或 assign  操作。
- 规则6：一个变量在同一个时刻只允许一条线程对其进行 lock 操作，但 lock 操作可以被同一条线程重复执行多次，多次执行 lock 后，只有执行相同次数的 unlock 操作，变量才会被解锁。所以 lock 和 unlock 必须成对出现。
- 规则7：如果对一个变量执行 lock 操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行 load 或 assign 操作初始化变量的值。
- 规则8：如果一个变量事先没有被 lock 操作锁定，则不允许对它执行 unlock 操作；也不允许去 unlock 一个被其他线程锁定的变量。
- 规则9：对一个变量执行 unlock 操作之前，必须先把此变量同步到主内存中（执行 store 和 write 操作）

看起来这些规则有些繁琐，其实也不难理解：

- 规则1、规则2
   工作内存中的共享变量作为主内存的副本，主内存变量的值同步到工作内存需要read和load一起使用，工作内存中的变量的值同步回主内存需要store和write一起使用，这2组操作各自都是是一个固定的有序搭配，不允许单独出现。
- 规则3、规则4
   由于工作内存中的共享变量是主内存的副本，为保证数据一致性，当工作内存中的变量被字节码引擎重新赋值，必须同步回主内存。如果工作内存的变量没有被更新，不允许无原因同步回主内存。
- 规则5
   由于工作内存中的共享变量是主内存的副本，必须从主内存诞生。
- 规则6、7、8、9
   为了并发情况下安全使用变量，线程可以基于lock操作独占主内存中的变量，其他线程不允许使用或unlock该变量，直到变量被线程unlock。

### 4.5 volatile型变量的特殊规则

volatile的中文意思是不稳定的，易变的，用volatile修饰变量是为了保证变量的可见性。

#### volatile的语义

volatile主要有下面2种语义

#### 语义1 保证可见性

保证了不同线程对该变量操作的内存可见性。

这里保证可见性是不等同于volatile变量并发操作的安全性，保证可见性具体一点解释：

**线程写volatile变量的过程：**

- 1 改变线程工作内存中volatile变量副本的值
- 2 将改变后的副本的值从工作内存刷新到主内存

**线程读volatile变量的过程：**

- 1 从主内存中读取volatile变量的最新值到线程的工作内存中
- 2 从工作内存中读取volatile变量的副本

但是如果多个线程同时把更新后的变量值同时刷新回主内存，可能导致得到的值不是预期结果：

举个例子：
 定义volatile int count = 0，2个线程同时执行count++操作，每个线程都执行500次，最终结果小于1000，原因是每个线程执行count++需要以下3个步骤：

- 步骤1 线程从主内存读取最新的count的值
- 步骤2 执行引擎把count值加1，并赋值给线程工作内存
- 步骤3 线程工作内存把count值保存到主内存
   有可能某一时刻2个线程在步骤1读取到的值都是100，执行完步骤2得到的值都是101，最后刷新了2次101保存到主内存。

#### 语义2 禁止进行指令重排序

具体一点解释，禁止重排序的规则如下：

- 当程序执行到 volatile变量的读操作或者写操作时，在其前面的操作的更改肯定全部已经进行，且结果已经对后面的操作可见；在其后面的操作肯定还没有进行；
- 在进行指令优化时，不能将在对 volatile 变量访问的语句放在其后面执行，也不能把 volatile 变量后面的语句放到其前面执行。

普通的变量仅仅会保证该方法的执行过程中所有依赖赋值结果的地方都能获取到正确的结果，而不能保证赋值操作的顺序与程序代码中的执行顺序一致。

举个例子：



```php
volatile boolean initialized = false;

// 下面代码线程A中执行
// 读取配置信息，当读取完成后将initialized设置为true以通知其他线程配置可用
doSomethingReadConfg();
initialized = true;

// 下面代码线程B中执行
// 等待initialized 为true，代表线程A已经把配置信息初始化完成
while (!initialized) {
     sleep();
}
// 使用线程A初始化好的配置信息
doSomethingWithConfig();
```

上面代码中如果定义initialized变量时没有使用volatile修饰，就有可能会由于指令重排序的优化，导致线程A中最后一句代码 "initialized = true" 在 “doSomethingReadConfg()” 之前被执行，这样会导致线程B中使用配置信息的代码就可能出现错误，而volatile关键字就禁止重排序的语义可以避免此类情况发生。

#### volatile型变量实现原理

具体实现方式是在编译期生成字节码时，会在指令序列中增加内存屏障来保证，下面是基于保守策略的JMM内存屏障插入策略：

![img](https:////upload-images.jianshu.io/upload_images/5618238-1131bae3fd4c90ef.png?imageMogr2/auto-orient/strip|imageView2/2/w/641/format/webp)

volatile型变量内存屏障插入策略

- 在每个volatile写操作的前面插入一个StoreStore屏障。
   该屏障除了保证了屏障之前的写操作和该屏障之后的写操作不能重排序，还会保证了volatile写操作之前，任何的读写操作都会先于volatile被提交。
- 在每个volatile写操作的后面插入一个StoreLoad屏障。
   该屏障除了使volatile写操作不会与之后的读操作重排序外，还会刷新处理器缓存，使volatile变量的写更新对其他线程可见。
- 在每个volatile读操作的后面插入一个LoadLoad屏障。
   该屏障除了使volatile读操作不会与之前的写操作发生重排序外，还会刷新处理器缓存，使volatile变量读取的为最新值。
- 在每个volatile读操作的后面插入一个LoadStore屏障。
   该屏障除了禁止了volatile读操作与其之后的任何写操作进行重排序，还会刷新处理器缓存，使其他线程volatile变量的写更新对volatile读操作的线程可见。

#### volatile型变量使用场景

总结起来，就是“一次写入，到处读取”，某一线程负责更新变量，其他线程只读取变量(不更新变量)，并根据变量的新值执行相应逻辑。例如状态标志位更新，观察者模型变量值发布。

### 4.6 final型变量的特殊规则

我们知道，final成员变量必须在声明的时候初始化或者在构造器中初始化，否则就会报编译错误。
 final关键字的可见性是指：被final修饰的字段在声明时或者构造器中，一旦初始化完成，那么在其他线程无须同步就能正确看见final字段的值。这是因为一旦初始化完成，final变量的值立刻回写到主内存。

### 4.7 synchronized的特殊规则

通过 synchronized关键字包住的代码区域，对数据的读写进行控制：

- 读数据
   当线程进入到该区域读取变量信息时，对数据的读取也不能从工作内存读取，只能从内存中读取，保证读到的是最新的值。
- 写数据
   在同步区内对变量的写入操作，在离开同步区时就将当前线程内的数据刷新到内存中，保证更新的数据对其他线程的可见性。

### 4.8 long和double型变量的特殊规则

Java内存模型要求lock、unlock、read、load、assign、use、store、write这8种操作都具有原子性，但是对于64位的数据类型(long和double)，在模型中特别定义相对宽松的规定：允许虚拟机将没有被volatile修饰的64位数据的读写操作分为2次32位的操作来进行。也就是说虚拟机可选择不保证64位数据类型的load、store、read和write这4个操作的原子性。由于这种非原子性，有可能导致其他线程读到同步未完成的“32位的半个变量”的值。

不过实际开发中，Java内存模型强烈建议虚拟机把64位数据的读写实现为具有原子性，目前各种平台下的商用虚拟机都选择把64位数据的读写操作作为原子操作来对待，因此我们在编写代码时一般不需要把用到的long和double变量专门声明为volatile。

## 5 总结

由于Java内存模型涉及系列规则，网上的文章大部分就是对这些规则进行解析，但是很多没有解释为什么需要这些规则，这些规则的作用，其实这是不利于初学者学习的，容易绕进去这些繁琐规则不知所以然，下面谈谈我的一点学习知识的个人体会：

**学习知识的过程不是等同于只是理解知识和记忆知识，而是要对知识解决的问题的输入和输出建立连接**，知识的本质是解决问题，所以在学习之前要理解问题，理解这个问题要的输出和输出，而知识就是输入到输出的一个关系映射。知识的学习要结合大量的例子来**理解这个映射关系，然后压缩知识**，华罗庚说过：“把一本书读厚，然后再读薄”，解释的就是这个道理，先结合大量的例子理解知识，然后再压缩知识。

以学习Java内存模型为例：

- 理解问题，明确输入输出
   首先理解Java内存模型是什么，有什么用，解决什么问题
- 理解内存模型系列协议
   结合大量例子理解这些协议规则
- 压缩知识
   大量规则其实就是通过数据同步协议，保证内存副本之间的数据一致性，同时防止重排序对程序的影响。

希望对大家有帮助。

## 参考

《深入学习Java虚拟机》

[深入拆解Java虚拟机](https://links.jianshu.com/go?to=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Fintro%2F108%3Fcode%3DXamkmJYooKBPKe8hT7otClsFDeVHb6rptMYHTdgiRPU%3D)

[Java核心技术36讲](https://links.jianshu.com/go?to=https%3A%2F%2Ftime.geekbang.org%2Fcolumn%2Farticle%2F13484%3Fcode%3DXamkmJYooKBPKe8hT7otClsFDeVHb6rptMYHTdgiRPU%3D)

[Synchronization and the Java Memory Model ——Doug Lea](https://links.jianshu.com/go?to=http%3A%2F%2Fgee.cs.oswego.edu%2Fdl%2Fcpj%2Fjmm.html)

[深入理解 Java 内存模型](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.infoq.cn%2Farticle%2Fjava-memory-model-1)

[Java内存屏障和可见性](https://links.jianshu.com/go?to=https%3A%2F%2Fblog.csdn.net%2Fkuangzhanshatian%2Farticle%2Fdetails%2F47949059)

[内存屏障与synchronized、volatile的原理](https://www.jianshu.com/p/43af2cc32f90)



