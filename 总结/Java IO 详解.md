# File 类

**File 类：文件和目录路径名的抽象表示。**

**注意：File 类只能操作文件的属性，文件的内容是不能操作的。**

**1、File 类的字段**

**![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514100904191-792147519.png)**

 

　　我们知道，各个平台之间的路径分隔符是不一样的。

　　①、对于UNIX平台，绝对路径名的前缀始终为`"/"` 。 相对路径名没有前缀。 表示根目录的抽象路径名具有前缀`"/"`和空名称序列。

　　②、对于Microsoft Windows平台，包含驱动器说明符的路径名的前缀由后面跟着`":"`的驱动器号组成，如果路径名是绝对的，则可能后跟`"\\"` 。 UNC路径名的前缀为`"\\\\"` ; 主机名和共享名称是名称序列中的前两个名称　　　      没有有指定驱动器的相对路径名没有前缀。

　　那么为了屏蔽各个平台之间的分隔符差异，我们在构造 File 类的时候（如何构造，请看下面第二点），就可以使用上述 Java 为我们提供的字段。

```
System.out.println(File.separator);``//输出 \  ``    ``System.out.println(File.pathSeparator);``//输出 ;
```

　　那么我们可以看出：

　　　　**File.pathSeparator指的是分隔连续多个路径字符串的分隔符**

　　　　**File.separator是用来分隔同一个路径字符串中的目录的**

 

 

**2、File 类的构造方法**

![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514101614926-416484144.png)

如何使用上述构造方法，请看如下例子：

```
//不使用 Java 提供的分隔符字段，注意：这样写只能在 Windows 平台有效
        File f1 = new File("D:\\IO\\a.txt");
        //使用 Java 提供的分隔符
        File f2 = new File("D:"+File.separator+"IO"+File.separator+"a.txt");
        System.out.println(f1);//输出 D:\IO\a.txt  
        System.out.println(f2);//输出 D:\IO\a.txt
         
        //File(File parent, String child)
        //从父抽象路径名和子路径名字符串创建新的 File实例。
        File f3 = new File("D:");
        File f4 = new File(f3,"IO");
        System.out.println(f4); //D:\IO
         
        //File(String pathname)
        //通过将给定的路径名字符串转换为抽象路径名来创建新的 File实例。
        File f5 = new File("D:"+File.separator+"IO"+File.separator+"a.txt");
        System.out.println(f5); //D:\IO\a.txt
         
        //File(String parent, String child)
        //从父路径名字符串和子路径名字符串创建新的 File实例。
        File f6 = new File("D:","IO\\a.txt");
        System.out.println(f6); //D:\IO\a.txt
```

　　 

**3、File 类的常用方法**

　　①、**创建方法**

　　　　1.boolean createNewFile() 不存在返回true 存在返回false
　　　　2.boolean mkdir() 创建目录，如果上一级目录不存在，则会创建失败
　　　　3.boolean mkdirs() 创建多级目录，如果上一级目录不存在也会自动创建

 

　　②、**删除方法**

　　　　1.boolean delete() 删除文件或目录，如果表示目录，则目录下必须为空才能删除
　　　　2.boolean deleteOnExit() 文件使用完成后删除

 

　　③、**判断方法**

　　　　1.boolean canExecute()判断文件是否可执行
　　　　2.boolean canRead()判断文件是否可读
　　　　3.boolean canWrite() 判断文件是否可写
　　　　4.boolean exists() 判断文件或目录是否存在
　　　　5.boolean isDirectory()  判断此路径是否为一个目录
　　　　6.boolean isFile()　　判断是否为一个文件
　　　　7.boolean isHidden()　　判断是否为隐藏文件
　　　　8.boolean isAbsolute()判断是否是绝对路径 文件不存在也能判断

 

 　④、**获取方法**

　　　　1.String getName() 获取此路径表示的文件或目录名称
　　　　2.String getPath() 将此路径名转换为路径名字符串
　　　　3.String getAbsolutePath() 返回此抽象路径名的绝对形式
　　　　4.String getParent()//如果没有父目录返回null
　　　　5.long lastModified()//获取最后一次修改的时间
　　　　6.long length() 返回由此抽象路径名表示的文件的长度。
　　　　7.boolean renameTo(File f) 重命名由此抽象路径名表示的文件。
　　　　8.File[] liseRoots()//获取机器盘符
　　　　9.String[] list()  返回一个字符串数组，命名由此抽象路径名表示的目录中的文件和目录。
　　　　10.String[] list(FilenameFilter filter) 返回一个字符串数组，命名由此抽象路径名表示的目录中满足指定过滤器的文件和目录。

 

```
//File(File parent, String child)
        //从父抽象路径名和子路径名字符串创建新的 File实例。
        File dir = new File("D:"+File.separator+"IO");
        File file = new File(dir,"a.txt");
         
        //判断dir 是否存在且表示一个目录
        if(!(dir.exists()||dir.isDirectory())){
            //如果 dir 不存在，则创建这个目录
            dir.mkdirs();
            //根据目录和文件名，创建 a.txt文件
            file.createNewFile();
 
        }
        //返回由此抽象路径名表示的文件或目录的名称。 这只是路径名称序列中的最后一个名字。 如果路径名的名称序列为空，则返回空字符串。
        System.out.println(file.getName()); //a.txt
        //返回此抽象路径名的父null的路径名字符串，如果此路径名未命名为父目录，则返回null。
        System.out.println(file.getParent());//D:\IO
        //将此抽象路径名转换为路径名字符串。 结果字符串使用default name-separator character以名称顺序分隔名称。
        System.out.println(file.getPath()); //D:\IO\a.txt
```

　 

4、File 的一些技巧

　　①、打印给定目录下的所有文件夹和文件夹里面的内容　

```
public static void getFileList(File file){
        //第一级子目录
        File[] files = file.listFiles();
        for(File f:files){
            //打印目录和文件
            System.out.println(f);
            if(f.isDirectory()){
                getFileList(f);
            }
        }
    }
```

　　测试：

```
public static void main(String[] args) throws Exception {
        File f = new File("D:"+File.separator+"WebStormFile");
        getFileList(f);
    }
```



# 流的分类

**一、根据流向分为输入流和输出流：**

　　注意输入流和输出流是相对于程序而言的。

　　输出：把程序(内存)中的内容输出到磁盘、光盘等存储设备中

![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514213816254-1233083805.png)

 

   输入：读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中

![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514213836691-236122960.png)

　　综合起来：

　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514214041051-1934100856.png)

 

**二、根据传输数据单位分为字节流和字符流**

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514214350379-1847340179.png)

　　上面的也是 Java IO流中的四大基流。这四大基流都是抽象类，其他流都是继承于这四大基流的。

 

**三、根据功能分为节点流和包装流**

　　节点流：可以从或向一个特定的地方(节点)读写数据。如FileReader.

　　处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如BufferedReader.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。

 

 操作 IO 流的模板：

　　①、创建源或目标对象

　　　　输入：把文件中的数据流向到程序中，此时文件是 源，程序是目标

　　　　输出：把程序中的数据流向到文件中，此时文件是目标，程序是源

 

　　②、创建 IO 流对象

　　　　输入：创建输入流对象

　　　　输出：创建输出流对象

 

　　③、具体的 IO 操作

 

　　④、关闭资源

　　　　输入：输入流的 close() 方法

　　　　输出：输出流的 close() 方法

 

 

**注意：1、程序中打开的文件 IO 资源不属于内存里的资源，垃圾回收机制无法回收该资源。如果不关闭该资源，那么磁盘的文件将一直被程序引用着，不能删除也不能更改。所以应该手动调用 close() 方法关闭流资源**

 

最后这是 Java IO 流的整体架构图，下面几篇博客将会详细讲解这些流：

![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514215843894-1251221936.png)

 

# 字节输入输出流

​	那么这篇博客我们讲的是字节输入输出流：**InputStream、OutputSteam**(下图红色长方形框内)，红色椭圆框内是其典型实现（FileInputSteam、FileOutStream）

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514220758597-133475538.png)

 

 **1、字节输出流：OutputStream**

```
public abstract class OutputStream
　　　　　　extends Object
　　　　　　implements Closeable, Flushable
```

　　这个抽象类是表示字节输出流的所有类的超类。 输出流接收输出字节并将其发送到某个接收器。

　　方法摘要：

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514221231097-1842081032.png)

 

　　下面我们用 字节输出流 OutputStream 的典型实现 FileOutputStream 来介绍：

```
//1、创建目标对象，输出流表示把数据保存到哪个文件。不写盘符，默认该文件是在该项目的根目录下
        File target = new File("io"+File.separator+"a.txt");
        //2、创建文件的字节输出流对象,第二个参数是 Boolean 类型，true 表示后面写入的文件追加到数据后面，false 表示覆盖
        OutputStream out = new FileOutputStream(target,true);
        //3、具体的 IO 操作(将数据写入到文件 a.txt 中)
            /**
             * void write(int b):把一个字节写入到文件中
             * void write(byte[] b):把数组b 中的所有字节写入到文件中
             * void write(byte[] b,int off,int len):把数组b 中的从 off 索引开始的 len 个字节写入到文件中
             */
        out.write(65); //将 A 写入到文件中
        out.write("Aa".getBytes()); //将 Aa 写入到文件中
        out.write("ABCDEFG".getBytes(), 1, 5); //将 BCDEF 写入到文件中
        //经过上面的操作，a.txt 文件中数据为 AAaBCDEF
         
        //4、关闭流资源
        out.close();
        System.out.println(target.getAbsolutePath());
```

　　

 

 **2、字节输入流：InputStream**

```
public abstract class InputStream
　　extends Object
　　implements Closeable
```

　　这个抽象类是表示输入字节流的所有类的超类。

　　方法摘要：

　　**![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170514222801629-1229701558.png)**

　　下面我们用 字节输出流 InputStream 的典型实现 FileInputStream 来介绍：

 　

```
//1、创建目标对象，输入流表示那个文件的数据保存到程序中。不写盘符，默认该文件是在该项目的根目录下
            //a.txt 保存的文件内容为：AAaBCDEF
        File target = new File("io"+File.separator+"a.txt");
        //2、创建输入流对象
        InputStream in = new FileInputStream(target);
        //3、具体的 IO 操作（读取 a.txt 文件中的数据到程序中）
            /**
             * 注意：读取文件中的数据，读到最后没有数据时，返回-1
             *  int read():读取一个字节，返回读取的字节
             *  int read(byte[] b):读取多个字节,并保存到数组 b 中，从数组 b 的索引为 0 的位置开始存储，返回读取了几个字节
             *  int read(byte[] b,int off,int len):读取多个字节，并存储到数组 b 中，从数组b 的索引为 0 的位置开始，长度为len个字节
             */
        //int read():读取一个字节，返回读取的字节
        int data1 = in.read();//获取 a.txt 文件中的数据的第一个字节
        System.out.println((char)data1); //A
        //int read(byte[] b):读取多个字节保存到数组b 中
        byte[] buffer  = new byte[10];
        in.read(buffer);//获取 a.txt 文件中的前10 个字节，并存储到 buffer 数组中
        System.out.println(Arrays.toString(buffer)); //[65, 97, 66, 67, 68, 69, 70, 0, 0, 0]
        System.out.println(new String(buffer)); //AaBCDEF[][][]
         
        //int read(byte[] b,int off,int len):读取多个字节，并存储到数组 b 中,从索引 off 开始到 len
        in.read(buffer, 0, 3);
        System.out.println(Arrays.toString(buffer)); //[65, 97, 66, 0, 0, 0, 0, 0, 0, 0]
        System.out.println(new String(buffer)); //AaB[][][][][][][]
        //4、关闭流资源
        in.close();
```

　　

 

**3、用字节流完成文件的复制**

　　

```
/**
         * 将 a.txt 文件 复制到 b.txt 中
         */
        //1、创建源和目标
        File srcFile = new File("io"+File.separator+"a.txt");
        File descFile = new File("io"+File.separator+"b.txt");
        //2、创建输入输出流对象
        InputStream in = new FileInputStream(srcFile);
        OutputStream out = new FileOutputStream(descFile);
        //3、读取和写入操作
        byte[] buffer = new byte[10];//创建一个容量为 10 的字节数组，存储已经读取的数据
        int len = -1;//表示已经读取了多少个字节，如果是 -1，表示已经读取到文件的末尾
        while((len=in.read(buffer))!=-1){
            //打印读取的数据
            System.out.println(new String(buffer,0,len));
            //将 buffer 数组中从 0 开始，长度为 len 的数据读取到 b.txt 文件中
            out.write(buffer, 0, len);
        }
        //4、关闭流资源
        out.close();
        in.close();
```

 

 

# 字符输入输出流

File 类的介绍：http://www.cnblogs.com/ysocean/p/6851878.html

Java IO 流的分类介绍：http://www.cnblogs.com/ysocean/p/6854098.html

Java IO 字节输入输出流：http://www.cnblogs.com/ysocean/p/6854541.html

那么这篇博客我们讲的是字符输入输出流：**Reader、Writer**(下图红色长方形框内)，红色椭圆框内是其典型实现，图片显示错误（FileReader、FileWriter）

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516001208385-1494291878.png)

**①、为什么要使用字符流？**

　　因为使用字节流操作汉字或特殊符号语言的时候容易乱码，因为汉字不止一个字节，为了解决这个问题，建议使用字符流。

**②、什么情况下使用字符流？**

　　一般可以用记事本打开的文件，我们可以看到内容不乱码的。就是文本文件，可以使用字符流。而操作二进制文件（比如图片、音频、视频）必须使用字节流

 

 **1、字符输出流：FileWriter**

```
public abstract class Writer
　　extends Object
　　implements Appendable, Closeable, Flushable
```

　　用于写入字符流的抽象类

　　方法摘要：

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516002749791-191966665.png)

　　下面我们用 字符输出流 Writer 的典型实现 FileWriter 来介绍这个类的用法：

 

```
//1、创建源
        File srcFile = new File("io"+File.separator+"a.txt");
        //2、创建字符输出流对象
        Writer out = new FileWriter(srcFile);
        //3、具体的 IO 操作
            /***
             * void write(int c):向外写出一个字符
             * void write(char[] buffer):向外写出多个字符 buffer
             * void write(char[] buffer,int off,int len):把 buffer 数组中从索引 off 开始到 len个长度的数据写出去
             * void write(String str):向外写出一个字符串
             */
        //void write(int c):向外写出一个字符
        out.write(65);//将 A 写入 a.txt 文件中
        //void write(char[] buffer):向外写出多个字符 buffer
        out.write("Aa帅锅".toCharArray());//将 Aa帅锅 写入 a.txt 文件中
        //void write(char[] buffer,int off,int len)
        out.write("Aa帅锅".toCharArray(),0,2);//将 Aa 写入a.txt文件中
        //void write(String str):向外写出一个字符串
        out.write("Aa帅锅");//将 Aa帅锅 写入 a.txt 文件中
         
        //4、关闭流资源
        /***
         * 注意如果这里有一个 缓冲的概念，如果写入文件的数据没有达到缓冲的数组长度，那么数据是不会写入到文件中的
         * 解决办法：手动刷新缓冲区 flush()
         * 或者直接调用 close() 方法，这个方法会默认刷新缓冲区
         */
        out.flush();
        out.close();
```

　　

 

 **2、字符输入流：Reader**

```
public abstract class Reader
　　extends Object
　　implements Readable, Closeable
```

　　用于读取字符流的抽象类。

　　方法摘要：

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516004403244-494211105.png)

　　下面我们用 字符输入流 Reader 的典型实现 FileReader 来介绍这个类的用法：

 

```
//1、创建源
        File srcFile = new File("io"+File.separator+"a.txt");
        //2、创建字符输出流对象
        Reader in = new FileReader(srcFile);
        //3、具体的 IO 操作
            /***
             * int read():每次读取一个字符，读到最后返回 -1
             * int read(char[] buffer):将字符读进字符数组,返回结果为读取的字符数
             * int read(char[] buffer,int off,int len):将读取的字符存储进字符数组 buffer，返回结果为读取的字符数，从索引 off 开始，长度为 len
             *
             */
        //int read():每次读取一个字符，读到最后返回 -1
        int len = -1;//定义当前读取字符的数量
        while((len = in.read())!=-1){
            //打印 a.txt 文件中所有内容
            System.out.print((char)len);
        }
         
        //int read(char[] buffer):将字符读进字符数组
        char[] buffer = new char[10]; //每次读取 10 个字符
        while((len=in.read(buffer))!=-1){
            System.out.println(new String(buffer,0,len));
        }
         
        //int read(char[] buffer,int off,int len)
        while((len=in.read(buffer,0,10))!=-1){
            System.out.println(new String(buffer,0,len));
        }
        //4、关闭流资源
        in.close();
```

　　

 

 **3、用字符流完成文件的复制**

 

```
/**
         * 将 a.txt 文件 复制到 b.txt 中
         */
        //1、创建源和目标
        File srcFile = new File("io"+File.separator+"a.txt");
        File descFile = new File("io"+File.separator+"b.txt");
        //2、创建字符输入输出流对象
        Reader in = new FileReader(srcFile);
        Writer out = new FileWriter(descFile);
        //3、读取和写入操作
        char[] buffer = new char[10];//创建一个容量为 10 的字符数组，存储已经读取的数据
        int len = -1;//表示已经读取了多少个字节，如果是 -1，表示已经读取到文件的末尾
        while((len=in.read(buffer))!=-1){
            out.write(buffer, 0, len);
        }
         
        //4、关闭流资源
        out.close();
        in.close();
```

#  包装流

　　我们在 [Java IO 流的分类介绍](http://www.cnblogs.com/ysocean/p/6854098.html) 这篇博客中介绍知道：

　　**根据功能分为节点流和包装流（处理流）**

　　　　节点流：可以从或向一个特定的地方(节点)读写数据。如FileReader.

　　　　处理流：是对一个已存在的流的连接和封装，通过所封装的流的功能调用实现数据读写。如BufferedReader.处理流的构造方法总是要带一个其他的流对象做参数。一个流对象经过其他流的多次包装，称为流的链接。

 

**1、前面讲的字符输入输出流，字节输入输出流都是字节流。那么什么是包装流呢？**

　　①、包装流隐藏了底层节点流的差异，并对外提供了更方便的输入\输出功能，让我们只关心这个高级流的操作

　　②、使用包装流包装了节点流，程序直接操作包装流，而底层还是节点流和IO设备操作

　　③、关闭包装流的时候，只需要关闭包装流即可

 

**2、缓冲流**

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516201919650-1292046402.png)

　　缓冲流：是一个包装流，目的是缓存作用，加快读取和写入数据的速度。

　　字节缓冲流：BufferedInputStream、BufferedOutputStream

　　字符缓冲流：BufferedReader、BufferedWriter

**案情回放：我们在将字符输入输出流、字节输入输出流的时候，读取操作，通常都会定义一个字节或字符数组，将读取/写入的数据先存放到这个数组里面，然后在取数组里面的数据。这比我们一个一个的读取/写入数据要快很多，而这也就是缓冲流的由来。只不过缓冲流里面定义了一个 数组用来存储我们读取/写入的数据，当内部定义的数组满了（注意：我们操作的时候外部还是会定义一个小的数组，小数组放入到内部数组中），就会进行下一步操作。**　

　　　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516204535088-744856618.png)

下面是没有用缓冲流的操作：　　　

```java
//1、创建目标对象，输入流表示那个文件的数据保存到程序中。不写盘符，默认该文件是在该项目的根目录下
            //a.txt 保存的文件内容为：AAaBCDEF
        File target = new File("io"+File.separator+"a.txt");
        //2、创建输入流对象
        InputStream in = new FileInputStream(target);
        //3、具体的 IO 操作（读取 a.txt 文件中的数据到程序中）
            /**
             * 注意：读取文件中的数据，读到最后没有数据时，返回-1
             *  int read():读取一个字节，返回读取的字节
             *  int read(byte[] b):读取多个字节,并保存到数组 b 中，从数组 b 的索引为 0 的位置开始存储，返回读取了几个字节
             *  int read(byte[] b,int off,int len):读取多个字节，并存储到数组 b 中，从数组b 的索引为 0 的位置开始，长度为len个字节
             */
        //int read():读取一个字节，返回读取的字节
        int data1 = in.read();//获取 a.txt 文件中的数据的第一个字节
        System.out.println((char)data1); //A
        //int read(byte[] b):读取多个字节保存到数组b 中
        byte[] buffer  = new byte[10];//这里我们定义了一个 长度为 10 的字节数组，用来存储读取的数据
        in.read(buffer);//获取 a.txt 文件中的前10 个字节，并存储到 buffer 数组中
        System.out.println(Arrays.toString(buffer)); //[65, 97, 66, 67, 68, 69, 70, 0, 0, 0]
        System.out.println(new String(buffer)); //AaBCDEF[][][]
         
        //int read(byte[] b,int off,int len):读取多个字节，并存储到数组 b 中,从索引 off 开始到 len
        in.read(buffer, 0, 3);
        System.out.println(Arrays.toString(buffer)); //[65, 97, 66, 0, 0, 0, 0, 0, 0, 0]
        System.out.println(new String(buffer)); //AaB[][][][][][][]
        //4、关闭流资源
        in.close();
```

　　我们查看 缓冲流的 JDK 底层源码，可以看到，程序中定义了这样的 缓存数组,大小为 8192

　　**BufferedInputStream:**

　　　　　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516203834478-1571378891.png)

 

 

　　**BufferedOutputStream:**

　　　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516203907682-67129768.png)

```java
				//字节缓冲输入流
        BufferedInputStream bis = new BufferedInputStream(
                new FileInputStream("io"+File.separator+"a.txt"));
        //定义一个字节数组，用来存储数据
        byte[] buffer = new byte[1024];
        int len = -1;//定义一个整数，表示读取的字节数
        while((len=bis.read(buffer))!=-1){
            System.out.println(new String(buffer,0,len));
        }
        //关闭流资源
        bis.close();<br><br>
         
        //字节缓冲输出流
        BufferedOutputStream bos = new BufferedOutputStream(
                new FileOutputStream("io"+File.separator+"a.txt"));
        bos.write("ABCD".getBytes());
        bos.close();
        //字符缓冲输入流
        BufferedReader br = new BufferedReader(
                new FileReader("io"+File.separator+"a.txt"));
        char[] buffer = new char[10];
        int len = -1;
        while((len=br.read(buffer))!=-1){
            System.out.println(new String(buffer,0,len));
        }
        br.close();
         
        //字符缓冲输出流
        BufferedWriter bw = new BufferedWriter(
                new FileWriter("io"+File.separator+"a.txt"));
        bw.write("ABCD");
        bw.close();
```

 

 **3、转换流：把字节流转换为字符流**

　　**InputStreamReader:把字节输入流转换为字符输入流
**

　　**OutputStreamWriter:把字节输出流转换为字符输出流**

 　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516210907791-1320951490.png)

 

 用转换流进行文件的复制：

```java
				/**
         * 将 a.txt 文件 复制到 b.txt 中
         */
        //1、创建源和目标
        File srcFile = new File("io"+File.separator+"a.txt");
        File descFile = new File("io"+File.separator+"b.txt");
        //2、创建字节输入输出流对象
        InputStream in = new FileInputStream(srcFile);
        OutputStream out = new FileOutputStream(descFile);
        //3、创建转换输入输出对象
        Reader rd = new InputStreamReader(in);
        Writer wt = new OutputStreamWriter(out);
        //3、读取和写入操作
        char[] buffer = new char[10];//创建一个容量为 10 的字符数组，存储已经读取的数据
        int len = -1;//表示已经读取了多少个字符，如果是 -1，表示已经读取到文件的末尾
        while((len=rd.read(buffer))!=-1){
            wt.write(buffer, 0, len);
        }
        //4、关闭流资源
        rd.close();
        wt.close();
```

　　

 

 **4、内存流（数组流）：**

　　把数据先临时存在数组中，也就是内存中。所以关闭 内存流是无效的，关闭后还是可以调用这个类的方法。底层源码的 close()是一个空方法

　　　　　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516215136807-1864331477.png)

 

　　①、字节内存流：ByteArrayOutputStream 、ByteArrayInputStream

```
//字节数组输出流：程序---》内存
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        //将数据写入到内存中
        bos.write("ABCD".getBytes());
        //创建一个新分配的字节数组。 其大小是此输出流的当前大小，缓冲区的有效内容已被复制到其中。
        byte[] temp = bos.toByteArray();
        System.out.println(new String(temp,0,temp.length));
         
        byte[] buffer = new byte[10];
        ///字节数组输入流：内存---》程序
        ByteArrayInputStream bis = new ByteArrayInputStream(temp);
        int len = -1;
        while((len=bis.read(buffer))!=-1){
            System.out.println(new String(buffer,0,len));
        }
         
        //这里不写也没事，因为源码中的 close()是一个空的方法体
        bos.close();
        bis.close();
```

　　

　　②、字符内存流：CharArrayReader、CharArrayWriter

　　

```
//字符数组输出流
        CharArrayWriter caw = new CharArrayWriter();
        caw.write("ABCD");
        //返回内存数据的副本
        char[] temp = caw.toCharArray();
        System.out.println(new String(temp));
         
        //字符数组输入流
        CharArrayReader car = new CharArrayReader(temp);
        char[] buffer = new char[10];
        int len = -1;
        while((len=car.read(buffer))!=-1){
            System.out.println(new String(buffer,0,len));
        }
```

　　

　　③、字符串流：StringReader,StringWriter（把数据临时存储到字符串中）

 

```
//字符串输出流,底层采用 StringBuffer 进行拼接
        StringWriter sw = new StringWriter();
        sw.write("ABCD");
        sw.write("帅锅");
        System.out.println(sw.toString());//ABCD帅锅
 
        //字符串输入流
        StringReader sr = new StringReader(sw.toString());
        char[] buffer = new char[10];
        int len = -1;
        while((len=sr.read(buffer))!=-1){
            System.out.println(new String(buffer,0,len));//ABCD帅锅
        }
```

　　

 

 

**5、合并流：把多个输入流合并为一个流，也叫顺序流，因为在读取的时候是先读第一个，读完了在读下面一个流。**

 ![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170516221936260-20534969.png)

```
//定义字节输入合并流
        SequenceInputStream seinput = new SequenceInputStream(
                new FileInputStream("io/a.txt"), new FileInputStream("io/b.txt"));
        byte[] buffer = new byte[10];
        int len = -1;
        while((len=seinput.read(buffer))!=-1){
            System.out.println(new String(buffer,0,len));
        }
         
        seinput.close();
```

# 序列化与反序列化 对象流

**1、什么是序列化与反序列化？**

　　序列化：指把堆内存中的 Java 对象数据，通过某种方式把对象存储到磁盘文件中或者传递给其他网络节点（在网络上传输）。这个过程称为序列化。通俗来说就是将数据结构或对象转换成二进制串的过程

　　反序列化：把磁盘文件中的对象数据或者把网络节点上的对象数据，恢复成Java对象模型的过程。也就是将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程

 

**2、为什么要做序列化？**

　　①、在分布式系统中，此时需要把对象在网络上传输，就得把对象数据转换为二进制形式，需要共享的数据的 JavaBean 对象，都得做序列化。

　　②、服务器钝化：如果服务器发现某些对象好久没活动了，那么服务器就会把这些内存中的对象持久化在本地磁盘文件中（Java对象转换为二进制文件）；如果服务器发现某些对象需要活动时，先去内存中寻找，找不到再去磁盘文件中反序列化我们的对象数据，恢复成 Java 对象。这样能节省服务器内存。

 

**3、Java 怎么进行序列化？**

　　①、需要做序列化的对象的类，必须实现序列化接口：Java.lang.Serializable 接口（这是一个标志接口，没有任何抽象方法），Java 中大多数类都实现了该接口，比如：String，Integer

　　②、底层会判断，如果当前对象是 Serializable 的实例，才允许做序列化，Java对象 instanceof Serializable 来判断。

　　③、在 Java 中使用对象流来完成序列化和反序列化

　　　　**ObjectOutputStream**:通过 writeObject()方法做序列化操作

　　　　**ObjectInputStream**:通过 readObject() 方法做反序列化操作 

 

 **第一步：创建一个 JavaBean 对象**

```
public class Person implements Serializable{
    private String name;
    private int age;
     
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + "]";
    }
    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }
}　　
```

 

 **第二步：使用 ObjectOutputStream 对象实现序列化**

```
//在根目录下新建一个 io 的文件夹
        OutputStream op = new FileOutputStream("io"+File.separator+"a.txt");
        ObjectOutputStream ops = new ObjectOutputStream(op);
        ops.writeObject(new Person("vae",1));
         
        ops.close();
```

　　我们打开 a.txt 文件，发现里面的内容乱码，注意这不需要我们来看懂，这是二进制文件，计算机能读懂就行了。

错误一：如果新建的 Person 对象没有实现 Serializable 接口，那么上面的操作会报错：

　　　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170517224626150-2013853053.png)

**第三步：使用ObjectInputStream 对象实现反序列化**

　　**反序列化的对象必须要提供该对象的字节码文件.class**

```
				InputStream in = new FileInputStream("io"+File.separator+"a.txt");
        ObjectInputStream os = new ObjectInputStream(in);
        byte[] buffer = new byte[10];
        Person p = (Person) os.readObject();
        System.out.println(p);  //Person [name=vae, age=1]
        os.close();
```

　　

问题1：如果某些数据不需要做序列化，比如密码，比如上面的年龄？

解决办法：在字段面前加上 transient

```
	 private String name;//需要序列化
    transient private int age;//不需要序列化
```

　　那么我们在反序列化的时候，打印出来的就是Person [name=vae, age=0]，整型数据默认值为 0 

 

问题2：序列化版本问题，在完成序列化操作后，由于项目的升级或修改，可能我们会对序列化对象进行修改，比如增加某个字段，那么我们在进行反序列化就会报错：

解决办法：在 JavaBean 对象中增加一个 serialVersionUID 字段，用来固定这个版本，无论我们怎么修改，版本都是一致的，就能进行反序列化了

```
private` `static` `final` `long` `serialVersionUID = 8656128222714547171L;
```



# 随机访问文件流 

**1、什么是 随机访问文件流 RandomAccessFile?**

　　该类的实例支持读取和写入随机访问文件。 随机访问文件的行为类似于存储在文件系统中的大量字节。 有一种游标，或索引到隐含的数组，称为文件指针 ; 输入操作读取从文件指针开始的字节，并使文件指针超过读取的字节。 如果在读/写模式下创建随机访问文件，则输出操作也可用; 输出操作从文件指针开始写入字节，并将文件指针提前到写入的字节。 写入隐式数组的当前端的输出操作会导致扩展数组。 文件指针可以通过读取getFilePointer方法和由设置seek方法。

　　通俗来讲：我们以前讲的 IO 字节流，包装流等都是按照文件内容的顺序来读取和写入的。而这个随机访问文件流我们可以再文件的任意地方写入数据，也可以读取任意地方的字节。

 

我们查看 底层源码，可以看到：

```
public class RandomAccessFile implements DataOutput, DataInput, Closeable {
```

　　实现了 DataOutput类，DataInput类，那么这两个类是什么呢？

 

**2、数据流：DataOutput,DataInput**

　　①、DataOutput:提供将数据从任何Java基本类型转换为一系列字节，并将这些字节写入二进制流。 还有一种将`String`转换为modified UTF-8格式(这种格式会在写入的数据之前默认增加两个字节的长度)并编写结果字节系列的功能。

　　②、DataInput:提供从二进制流读取字节并从其中重建任何Java原语类型的数据。 还有，为了重建设施String从数据modified UTF-8格式。 

下面我们以其典型实现：DataOutputSteam、DataInputStream 来看看它的用法：

```
			 //数据输出流
        File file = new File("io"+File.separator+"a.txt");
        DataOutputStream dop = new DataOutputStream(new FileOutputStream(file));
        //写入三种类型的数据
        dop.write(65);
        dop.writeChar('哥');
        dop.writeUTF("帅锅");
        dop.close();
         
        //数据输入流
        DataInputStream dis = new DataInputStream(new FileInputStream(file));
        System.out.println(dis.read());  //65
        System.out.println(dis.readChar()); //哥
        System.out.println(dis.readUTF());  //帅锅
        dis.close();

```

　　

 

**3、通过上面的例子，我们可以看到因为 RandomAccessFile 实现了数据输入输出流，那么 RandomAccessFile 这一个类就可以完成 输入输出的功能了。**

randomAccessFile(File file,String mode)

RandomAccessFile(String name,String mode)

这里面第二个参数：String mode 有以下几种形式：（ps：为什么这里的值是固定的而不弄成枚举形式，不然很容易写错，这是因为随机访问流出现在枚举类型之前，属于Java 历史遗留问题）

　　![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170518190121494-861731929.png)

 

 第一种：用 随机流顺序读取数据

```
public class RandomAccessFileTest {
    public static void main(String[] args) throws Exception {
        File file = new File("io"+File.separator+"a.txt");
        write(file);
        read(file);
    }
     
    /**
     * 随机流读数据
     */
    private static void read(File file) throws Exception {
        //以 r 即只读的方法读取数据
        RandomAccessFile ras = new RandomAccessFile(file, "r");
        byte b = ras.readByte();
        System.out.println(b); //65
         
        int i = ras.readInt();
        System.out.println(i); //97
         
        String str = ras.readUTF(); //帅锅
        System.out.println(str);
        ras.close();
    }
 
    /**
     * 随机流写数据
     */
    private static void write(File file) throws Exception{
        //以 rw 即读写的方式写入数据
        RandomAccessFile ras = new RandomAccessFile(file, "rw");
        ras.writeByte(65);
        ras.writeInt(97);
        ras.writeUTF("帅锅");
         
        ras.close();
    }
 
}

```

　　

第二种：随机读取，那么我们先介绍这两个方法

Long  getFilePointer()

Void seek(long pos)

**这里所说的偏移量，也就是字节数。一个文件是有N个字节数组成，那么我们可以通过设置读取或者写入的偏移量，来达到随机读取或写入的目的。**

**我们先看看Java 各数据类型所占字节数：**

**![img](https://images2015.cnblogs.com/blog/1120165/201705/1120165-20170518193446025-600957924.png)**

**下面是 随机读取数据例子：**

```
/**
     * 随机流读数据
     */
    private static void read(File file) throws Exception {
        //以 r 即只读的方法读取数据
        RandomAccessFile ras = new RandomAccessFile(file, "r");
         
        byte b = ras.readByte();
        System.out.println(b); //65
        //我们已经读取了一个字节的数据，那么当前偏移量为 1
        System.out.println(ras.getFilePointer());  //1
        //这时候我们设置 偏移量为 5，那么可以直接读取后面的字符串（前面是一个字节+一个整型数据=5个字节）
        ras.seek(5);
        String str = ras.readUTF(); //帅锅
        System.out.println(str);
         
        //这时我们设置 偏移量为 0，那么从头开始
        ras.seek(0);
        System.out.println(ras.readByte()); //65
         
        //需要注意的是：UTF 写入的数据默认会在前面增加两个字节的长度
         
        ras.close();
    }
```

　　

 **随机流复制文件：**

```
/**
     * 随机流复制文件
     * @param fileA
     * @param B
     * @throws Exception
     */
    private static void copyFile(File fileA,File fileB) throws Exception{
         
        RandomAccessFile srcRA = new RandomAccessFile(fileA, "rw");
        RandomAccessFile descRA = new RandomAccessFile(fileB, "rw");
         
        //向 文件 a.txt 中写入数据
        srcRA.writeByte(65);
        srcRA.writeInt(97);
        srcRA.writeUTF("帅锅");
        //获取 a.txt 文件的字节长度
        int len = (int) srcRA.length();
        srcRA.seek(0);
        System.out.println(srcRA.readByte()+srcRA.readInt()+srcRA.readUTF());
         
        //开始复制
        srcRA.seek(0);
        //定义一个数组，用来存放 a.txt 文件的数据
        byte[] buffer = new byte[len];
        //将 a.txt 文件的内容读到 buffer 中
        srcRA.readFully(buffer);
        //再将 buffer 写入到 b.txt文件中
        descRA.write(buffer);
         
        //读取 b.txt 文件中的数据
        descRA.seek(0);
        System.out.println(descRA.readByte()+descRA.readInt()+descRA.readUTF());
        //关闭流资源
        srcRA.close();
        descRA.close();
    }
```

　　

ps：一般多线程下载、断点下载都可以运用此随机流