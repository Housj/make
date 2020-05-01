









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



# I/O体系从原理到应用

本文介绍操作系统I/O工作原理，Java I/O设计，基本使用，开源项目中实现高性能I/O常见方法和实现，彻底搞懂高性能I/O之道

## 基础概念

在介绍I/O原理之前，先重温几个基础概念：

- **(1) 操作系统与内核**

![img](https:////upload-images.jianshu.io/upload_images/5618238-ee23d455c41b9496.png?imageMogr2/auto-orient/strip|imageView2/2/w/430/format/webp)

**操作系统**：管理计算机硬件与软件资源的系统软件
 **内核**：操作系统的核心软件，负责管理系统的进程、内存、设备驱动程序、文件和网络系统等等，为应用程序提供对计算机硬件的安全访问服务

- **2 内核空间和用户空间**

为了避免用户进程直接操作内核，保证内核安全，操作系统将内存寻址空间划分为两部分：
 **内核空间（Kernel-space）**，供内核程序使用
 **用户空间（User-space）**，供用户进程使用
 为了安全，内核空间和用户空间是隔离的，即使用户的程序崩溃了，内核也不受影响

- **3 数据流**

![img](https:////upload-images.jianshu.io/upload_images/5618238-abf5fdf85c9e8889.png?imageMogr2/auto-orient/strip|imageView2/2/w/338/format/webp)

计算机中的数据是基于随着时间变换高低电压信号传输的，这些数据信号连续不断，有着固定的传输方向，类似水管中水的流动，因此抽象数据流(I/O流)的概念：**指一组有顺序的、有起点和终点的字节集合**，

抽象出数据流的作用：**实现程序逻辑与底层硬件解耦**，通过引入数据流作为程序与硬件设备之间的抽象层，面向通用的数据流输入输出接口编程，而不是具体硬件特性，程序和底层硬件可以独立灵活替换和扩展

![img](https:////upload-images.jianshu.io/upload_images/5618238-8c4dd0bb4d7e0a51.png?imageMogr2/auto-orient/strip|imageView2/2/w/362/format/webp)

## I/O 工作原理

### 1 磁盘I/O

典型I/O读写磁盘工作原理如下：

![img](https:////upload-images.jianshu.io/upload_images/5618238-e2f6c7cc9ec3d9e4.png?imageMogr2/auto-orient/strip|imageView2/2/w/484/format/webp)

> **tips**: DMA：全称叫直接内存存取（Direct Memory Access），是一种允许外围设备（硬件子系统）直接访问系统主内存的机制。基于 DMA 访问方式，系统主内存与硬件设备的数据传输可以省去CPU 的全程调度

值得注意的是：

- 读写操作基于系统调用实现
- 读写操作经过用户缓冲区，内核缓冲区，应用进程并不能直接操作磁盘
- 应用进程读操作时需阻塞直到读取到数据

### 2 网络I/O

这里先以最经典的阻塞式I/O模型介绍：

![img](https:////upload-images.jianshu.io/upload_images/5618238-dbc0c35b530791a7.png?imageMogr2/auto-orient/strip|imageView2/2/w/484/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/5618238-1da0221dd8a7109a.png?imageMogr2/auto-orient/strip|imageView2/2/w/584/format/webp)

> **tips**:recvfrom，经socket接收数据的函数

值得注意的是：

- 网络I/O读写操作经过用户缓冲区，Sokcet缓冲区
- 服务端线程在从调用recvfrom开始到它返回有数据报准备好这段时间是阻塞的，recvfrom返回成功后，线程开始处理数据报

## Java I/O设计

### 1 I/O分类

Java中对数据流进行具体化和实现，关于Java数据流一般关注以下几个点：

- **(1) 流的方向**
   从外部到程序，称为**输入流**；从程序到外部，称为**输出流**
- **(2) 流的数据单位**
   程序以字节作为最小读写数据单元，称为**字节流**，以字符作为最小读写数据单元，称为**字符流**
- **(3) 流的功能角色**

![img](https:////upload-images.jianshu.io/upload_images/5618238-711a259d84778cb2.png?imageMogr2/auto-orient/strip|imageView2/2/w/362/format/webp)

从/向一个特定的IO设备（如磁盘，网络）或者存储对象(如内存数组)读/写数据的流，称为**节点流**；
 对一个已有流进行连接和封装，通过封装后的流来实现数据的读/写功能，称为**处理流**(或称为过滤流)；

### 2 I/O操作接口

java.io包下有一堆I/O操作类，初学时看了容易搞不懂，其实仔细观察其中还是有规律：
 这些I/O操作类都是在**继承4个基本抽象流的基础上，要么是节点流，要么是处理流**

#### 2.1 四个基本抽象流

java.io包中包含了流式I/O所需要的所有类，java.io包中有四个基本抽象流，分别处理字节流和字符流：

- InputStream
- OutputStream
- Reader
- Writer

![img](https:////upload-images.jianshu.io/upload_images/5618238-5035c8b1b9c3e5d4.png?imageMogr2/auto-orient/strip|imageView2/2/w/457/format/webp)

#### 2.2 节点流

![img](https:////upload-images.jianshu.io/upload_images/5618238-01ee80f008853679.png?imageMogr2/auto-orient/strip|imageView2/2/w/384/format/webp)

节点流I/O类名由节点流类型 + 抽象流类型组成，常见节点类型有：

- File文件
- Piped 进程内线程通信管道
- ByteArray / CharArray (字节数组 / 字符数组)
- StringBuffer / String (字符串缓冲区 / 字符串)

节点流的创建通常是在构造函数传入数据源，例如：



```cpp
FileReader reader = new FileReader(new File("file.txt"));
FileWriter writer = new FileWriter(new File("file.txt"));
```

#### 2.3 处理流

![img](https:////upload-images.jianshu.io/upload_images/5618238-1f5b0f57c0f06b82.png?imageMogr2/auto-orient/strip|imageView2/2/w/361/format/webp)

处理流I/O类名由对已有流封装的功能 + 抽象流类型组成，常见功能有：

- **缓冲**：对节点流读写的数据提供了缓冲的功能，数据可以基于缓冲批量读写，提高效率。常见有BufferedInputStream、BufferedOutputStream
- **字节流转换为字符流**：由InputStreamReader、OutputStreamWriter实现
- **字节流与基本类型数据相互转换**：这里基本数据类型数据如int、long、short，由DataInputStream、DataOutputStream实现
- **字节流与对象实例相互转换**：用于实现对象序列化，由ObjectInputStream、ObjectOutputStream实现

处理流的应用了适配器/装饰模式，转换/扩展已有流，处理流的创建通常是在构造函数传入已有的节点流或处理流：

```csharp
FileOutputStream fileOutputStream = new FileOutputStream("file.txt");
// 扩展提供缓冲写
BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
 // 扩展提供提供基本数据类型写
DataOutputStream out = new DataOutputStream(bufferedOutputStream);
```

### 3 Java NIO

#### 3.1 标准I/O存在问题

Java NIO(New I/O)是一个可以替代标准Java I/O API的IO API(从Java 1.4开始)，Java NIO提供了与标准I/O不同的I/O工作方式，目的是为了解决标准 I/O存在的以下问题：

- **(1) 数据多次拷贝**

标准I/O处理，完成一次完整的数据读写，至少需要从底层硬件读到内核空间，再读到用户文件，又从用户空间写入内核空间，再写入底层硬件

此外，底层通过write、read等函数进行I/O系统调用时，需要传入数据所在缓冲区**起始地址和长度**。由于JVM GC的存在，导致对象在堆中的位置往往会发生移动，移动后传入系统函数的地址参数就不是真正的缓冲区地址了可能导致读写出错，为了解决上面的问题，使用标准I/O进行系统调用时，还会额外导致一次数据拷贝：把数据从JVM的堆内拷贝到堆外的连续空间内存(堆外内存)

所以总共经历6次数据拷贝，执行效率较低

![img](https:////upload-images.jianshu.io/upload_images/5618238-54528ba95009c708.png?imageMogr2/auto-orient/strip|imageView2/2/w/484/format/webp)

- **(2) 操作阻塞**

传统的网络I/O处理中，由于请求建立连接(connect)，读取网络I/O数据(read)，发送数据(send)等操作是线程阻塞的

```java
// 等待连接
Socket socket = serverSocket.accept();

// 连接已建立，读取请求消息
StringBuilder req = new StringBuilder();
byte[] recvByteBuf = new byte[1024];
int len;
while ((len = socket.getInputStream().read(recvByteBuf)) != -1) {
    req.append(new String(recvByteBuf, 0, len, StandardCharsets.UTF_8));
}

// 写入返回消息
socket.getOutputStream().write(("server response msg".getBytes()));
socket.shutdownOutput();
```

以上面服务端程序为例，当请求连接已建立，读取请求消息，服务端调用read方法时，客户端数据可能还没就绪(例如客户端数据还在写入中或者传输中)，线程需要在read方法阻塞等待直到数据就绪

为了实现服务端并发响应，每个连接需要独立的线程单独处理，当并发请求量大时为了维护连接，内存、线程切换开销过大

![img](https:////upload-images.jianshu.io/upload_images/5618238-df52af9d6af06e61.png?imageMogr2/auto-orient/strip|imageView2/2/w/311/format/webp)

#### 3.2 Buffer

**Java NIO核心三大核心组件是Buffer(缓冲区)、Channel(通道)、Selector**

Buffer提供了常用于I/O操作的字节缓冲区，常见的缓存区有ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, ShortBuffer，分别对应基本数据类型: byte, char, double, float, int, long, short，下面介绍主要以最常用的ByteBuffer为例，Buffer底层支持Java堆内(HeapByteBuffer)或堆外内存(DirectByteBuffer)

**堆外内存**是指与堆内存相对应的，把内存对象分配在JVM堆以外的内存，这些内存直接受操作系统管理（而不是虚拟机)，相比堆内内存，I/O操作中使用堆外内存的优势在于：

- 不用被JVM GC线回收，减少GC线程资源占有
- 在I/O系统调用时，直接操作堆外内存，可以节省堆外内存和堆内内存的复制成本

ByteBuffer底层堆外内存的分配和释放基于malloc和free函数，对外allocateDirect方法可以申请分配堆外内存，并返回继承ByteBuffer类的DirectByteBuffer对象：

```cpp
public static ByteBuffer allocateDirect(int capacity) {
    return new DirectByteBuffer(capacity);
}
```

堆外内存的回收基于DirectByteBuffer的成员变量Cleaner类，提供clean方法可以用于主动回收，Netty中大部分堆外内存通过记录定位Cleaner的存在，主动调用clean方法来回收；
 另外，当DirectByteBuffer对象被GC时，关联的堆外内存也会被回收

> **tips**: JVM参数不建议设置-XX:+DisableExplicitGC，因为部分依赖Java NIO的框架(例如Netty)在内存异常耗尽时，会主动调用System.gc()，触发Full GC，回收DirectByteBuffer对象，作为回收堆外内存的最后保障机制，设置该参数之后会导致在该情况下堆外内存得不到清理

堆外内存基于基础ByteBuffer类的DirectByteBuffer类成员变量：Cleaner对象，这个Cleaner对象会在合适的时候执行unsafe.freeMemory(address)，从而回收这块堆外内存

Buffer可以理解为一组基本数据类型，存储地址连续的的数组，支持读写操作，对应读模式和写模式，通过几个变量来保存这个数据的当前位置状态：capacity、 position、 limit：

- capacity 缓冲区数组的总长度
- position 下一个要操作的数据元素的位置
- limit 缓冲区数组中不可操作的下一个元素的位置：limit <= capacity

![img](https:////upload-images.jianshu.io/upload_images/5618238-6b687ff0b2a2ee5d.png?imageMogr2/auto-orient/strip|imageView2/2/w/429/format/webp)

#### 3.3 Channel

Channel(通道)的概念可以类比I/O流对象，NIO中I/O操作主要基于Channel：
 从Channel进行数据读取 ：创建一个缓冲区，然后请求Channel读取数据
 从Channel进行数据写入 ：创建一个缓冲区，填充数据，请求Channel写入数据

Channel和流非常相似，主要有以下几点区别：

- Channel可以读和写，而标准I/O流是单向的
- Channel可以异步读写，标准I/O流需要线程阻塞等待直到读写操作完成
- Channel总是基于缓冲区Buffer读写

Java NIO中最重要的几个Channel的实现：

- FileChannel： 用于文件的数据读写，基于FileChannel提供的方法能减少读写文件数据拷贝次数，后面会介绍
- DatagramChannel： 用于UDP的数据读写
- SocketChannel： 用于TCP的数据读写，代表客户端连接
- ServerSocketChannel: 监听TCP连接请求，每个请求会创建会一个SocketChannel，一般用于服务端

基于标准I/O中，我们第一步可能要像下面这样获取输入流，按字节把磁盘上的数据读取到程序中，再进行下一步操作，而在NIO编程中，需要先获取Channel，再进行读写



```java
FileInputStream fileInputStream = new FileInputStream("test.txt");
FileChannel channel = fileInputStream.channel();
```

> **tips**: FileChannel仅能运行在阻塞模式下，文件异步处理的 I/O 是在JDK 1.7 才被加入的 java.nio.channels.AsynchronousFileChannel



```java
// server socket channel:
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.bind(new InetSocketAddress(InetAddress.getLocalHost(), 9091));

while (true) {
    SocketChannel socketChannel = serverSocketChannel.accept();
    ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
    int readBytes = socketChannel.read(buffer);
    if (readBytes > 0) {
        // 从写数据到buffer翻转为从buffer读数据
        buffer.flip();
        byte[] bytes = new byte[buffer.remaining()];
        buffer.get(bytes);
        String body = new String(bytes, StandardCharsets.UTF_8);
        System.out.println("server 收到：" + body);
    }
}
```

#### 3.4 Selector

Selector(选择器) ，它是Java NIO核心组件中的一个，用于检查一个或多个NIO Channel（通道）的状态是否处于可读、可写。实现单线程管理多个Channel，也就是可以管理多个网络连接

Selector核心在于基于操作系统提供的I/O复用功能，单个线程可以同时监视多个连接描述符，一旦某个连接就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作，常见有select、poll、epoll等不同实现

![img](https:////upload-images.jianshu.io/upload_images/5618238-61dd67cedc78e7e9.png?imageMogr2/auto-orient/strip|imageView2/2/w/628/format/webp)

![img](https:////upload-images.jianshu.io/upload_images/5618238-da53b4240d8ca791.png?imageMogr2/auto-orient/strip|imageView2/2/w/638/format/webp)

Java NIO Selector基本工作原理如下：

- (1) 初始化Selector对象，服务端ServerSocketChannel对象
- (2) 向Selector注册ServerSocketChannel的socket-accept事件
- (3) 线程阻塞于selector.select()，当有客户端请求服务端，线程退出阻塞
- (4) 基于selector获取所有就绪事件，此时先获取到socket-accept事件，向Selector注册客户端SocketChannel的数据就绪可读事件事件
- (5) 线程再次阻塞于selector.select()，当有客户端连接数据就绪，可读
- (6) 基于ByteBuffer读取客户端请求数据，然后写入响应数据，关闭channel

示例如下，完整可运行代码已经上传github([https://github.com/caison/caison-blog-demo](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fcaison%2Fcaison-blog-demo))：



```java
Selector selector = Selector.open();
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.bind(new InetSocketAddress(9091));
// 配置通道为非阻塞模式
serverSocketChannel.configureBlocking(false);
// 注册服务端的socket-accept事件
serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

while (true) {
    // selector.select()会一直阻塞，直到有channel相关操作就绪
    selector.select();
    // SelectionKey关联的channel都有就绪事件
    Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();

    while (keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();
        // 服务端socket-accept
        if (key.isAcceptable()) {
            // 获取客户端连接的channel
            SocketChannel clientSocketChannel = serverSocketChannel.accept();
            // 设置为非阻塞模式
            clientSocketChannel.configureBlocking(false);
            // 注册监听该客户端channel可读事件，并为channel关联新分配的buffer
            clientSocketChannel.register(selector, SelectionKey.OP_READ, ByteBuffer.allocateDirect(1024));
        }

        // channel可读
        if (key.isReadable()) {
            SocketChannel socketChannel = (SocketChannel) key.channel();
            ByteBuffer buf = (ByteBuffer) key.attachment();

            int bytesRead;
            StringBuilder reqMsg = new StringBuilder();
            while ((bytesRead = socketChannel.read(buf)) > 0) {
                // 从buf写模式切换为读模式
                buf.flip();
                int bufRemain = buf.remaining();
                byte[] bytes = new byte[bufRemain];
                buf.get(bytes, 0, bytesRead);
                // 这里当数据包大于byteBuffer长度，有可能有粘包/拆包问题
                reqMsg.append(new String(bytes, StandardCharsets.UTF_8));
                buf.clear();
            }
            System.out.println("服务端收到报文：" + reqMsg.toString());
            if (bytesRead == -1) {
                byte[] bytes = "[这是服务回的报文的报文]".getBytes(StandardCharsets.UTF_8);

                int length;
                for (int offset = 0; offset < bytes.length; offset += length) {
                    length = Math.min(buf.capacity(), bytes.length - offset);
                    buf.clear();
                    buf.put(bytes, offset, length);
                    buf.flip();
                    socketChannel.write(buf);
                }
                socketChannel.close();
            }
        }
        // Selector不会自己从已selectedKeys中移除SelectionKey实例
        // 必须在处理完通道时自己移除 下次该channel变成就绪时，Selector会再次将其放入selectedKeys中
        keyIterator.remove();
    }
}
```

> **tips**: Java NIO基于Selector实现高性能网络I/O这块使用起来比较繁琐，使用不友好，一般业界使用基于Java NIO进行封装优化，扩展丰富功能的Netty框架来优雅实现

# 高性能I/O优化

下面结合业界热门开源项目介绍高性能I/O的优化

## 1 零拷贝

零拷贝(zero copy)技术，用于在数据读写中减少甚至完全避免不必要的CPU拷贝，减少内存带宽的占用，提高执行效率，零拷贝有几种不同的实现原理，下面介绍常见开源项目中零拷贝实现

### 1.1 Kafka零拷贝

Kafka基于Linux 2.1内核提供，并在2.4 内核改进的的sendfile函数 + 硬件提供的DMA Gather Copy实现零拷贝，将文件通过socket传送

函数通过一次系统调用完成了文件的传送，减少了原来read/write方式的模式切换。同时减少了数据的copy, sendfile的详细过程如下：

![img](https:////upload-images.jianshu.io/upload_images/5618238-4a47e3d4d662bc51.png?imageMogr2/auto-orient/strip|imageView2/2/w/488/format/webp)

基本流程如下：

- (1) 用户进程发起sendfile系统调用
- (2) 内核基于DMA Copy将文件数据从磁盘拷贝到内核缓冲区
- (3) 内核将内核缓冲区中的文件描述信息(文件描述符，数据长度)拷贝到Socket缓冲区
- (4) 内核基于Socket缓冲区中的文件描述信息和DMA硬件提供的Gather Copy功能将内核缓冲区数据复制到网卡
- (5) 用户进程sendfile系统调用完成并返回

相比传统的I/O方式，sendfile + DMA Gather Copy方式实现的零拷贝，数据拷贝次数从4次降为2次，系统调用从2次降为1次，用户进程上下文切换次数从4次变成2次DMA Copy，大大提高处理效率

Kafka底层基于java.nio包下的FileChannel的transferTo：



```csharp
public abstract long transferTo(long position, long count, WritableByteChannel target)
```

transferTo将FileChannel关联的文件发送到指定channel，当Comsumer消费数据，Kafka Server基于FileChannel将文件中的消息数据发送到SocketChannel

### 1.2 RocketMQ零拷贝

RocketMQ基于mmap + write的方式实现零拷贝：
 mmap() 可以将内核中缓冲区的地址与用户空间的缓冲区进行映射，实现数据共享，省去了将数据从内核缓冲区拷贝到用户缓冲区



```c
tmp_buf = mmap(file, len); 
write(socket, tmp_buf, len);
```

![img](https:////upload-images.jianshu.io/upload_images/5618238-c7f3a5a7e581c14b.png?imageMogr2/auto-orient/strip|imageView2/2/w/448/format/webp)

mmap + write 实现零拷贝的基本流程如下：

- (1) 用户进程向内核发起系统mmap调用
- (2) 将用户进程的内核空间的读缓冲区与用户空间的缓存区进行内存地址映射
- (3) 内核基于DMA Copy将文件数据从磁盘复制到内核缓冲区
- (4) 用户进程mmap系统调用完成并返回
- (5) 用户进程向内核发起write系统调用
- (6) 内核基于CPU Copy将数据从内核缓冲区拷贝到Socket缓冲区
- (7) 内核基于DMA Copy将数据从Socket缓冲区拷贝到网卡
- (8) 用户进程write系统调用完成并返回

RocketMQ中消息基于mmap实现存储和加载的逻辑写在org.apache.rocketmq.store.MappedFile中，内部实现基于nio提供的java.nio.MappedByteBuffer，基于FileChannel的map方法得到mmap的缓冲区：



```java
// 初始化
this.fileChannel = new RandomAccessFile(this.file, "rw").getChannel();
this.mappedByteBuffer = this.fileChannel.map(MapMode.READ_WRITE, 0, fileSize);
```

查询CommitLog的消息时，基于mappedByteBuffer偏移量pos，数据大小size查询：



```cpp
public SelectMappedBufferResult selectMappedBuffer(int pos, int size) {
    int readPosition = getReadPosition();
    // ...各种安全校验
    
    // 返回mappedByteBuffer视图
    ByteBuffer byteBuffer = this.mappedByteBuffer.slice();
    byteBuffer.position(pos);
    ByteBuffer byteBufferNew = byteBuffer.slice();
    byteBufferNew.limit(size);
    return new SelectMappedBufferResult(this.fileFromOffset + pos, byteBufferNew, size, this);
}
```

> **tips: transientStorePoolEnable机制**
>  Java NIO mmap的部分内存并不是常驻内存，可以被置换到交换内存(虚拟内存)，RocketMQ为了提高消息发送的性能，引入了内存锁定机制，即将最近需要操作的CommitLog文件映射到内存，并提供内存锁定功能，确保这些文件始终存在内存中，该机制的控制参数就是transientStorePoolEnable

因此，MappedFile数据保存CommitLog刷盘有2种方式：

- 1 开启transientStorePoolEnable：写入内存字节缓冲区(writeBuffer)  -> 从内存字节缓冲区(writeBuffer)提交(commit)到文件通道(fileChannel)  -> 文件通道(fileChannel) -> flush到磁盘
- 2 未开启transientStorePoolEnable：写入映射文件字节缓冲区(mappedByteBuffer) -> 映射文件字节缓冲区(mappedByteBuffer) -> flush到磁盘

RocketMQ 基于 mmap+write 实现零拷贝，适用于业务级消息这种小块文件的数据持久化和传输
 Kafka 基于 sendfile 这种零拷贝方式，适用于系统日志消息这种高吞吐量的大块文件的数据持久化和传输

> **tips:** Kafka 的索引文件使用的是 mmap+write 方式，数据文件发送网络使用的是 sendfile 方式

### 1.3 Netty零拷贝

Netty 的零拷贝分为两种：

- 1 基于操作系统实现的零拷贝，底层基于FileChannel的transferTo方法
- 2 基于Java 层操作优化，对数组缓存对象(ByteBuf )进行封装优化，通过对ByteBuf数据建立数据视图，支持ByteBuf 对象合并，切分，当底层仅保留一份数据存储，减少不必要拷贝

## 2 多路复用

Netty中对Java NIO功能封装优化之后，实现I/O多路复用代码优雅了很多：



```java
// 创建mainReactor
NioEventLoopGroup boosGroup = new NioEventLoopGroup();
// 创建工作线程组
NioEventLoopGroup workerGroup = new NioEventLoopGroup();

final ServerBootstrap serverBootstrap = new ServerBootstrap();
serverBootstrap 
     // 组装NioEventLoopGroup 
    .group(boosGroup, workerGroup)
     // 设置channel类型为NIO类型
    .channel(NioServerSocketChannel.class)
    // 设置连接配置参数
    .option(ChannelOption.SO_BACKLOG, 1024)
    .childOption(ChannelOption.SO_KEEPALIVE, true)
    .childOption(ChannelOption.TCP_NODELAY, true)
    // 配置入站、出站事件handler
    .childHandler(new ChannelInitializer<NioSocketChannel>() {
        @Override
        protected void initChannel(NioSocketChannel ch) {
            // 配置入站、出站事件channel
            ch.pipeline().addLast(...);
            ch.pipeline().addLast(...);
        }
    });

// 绑定端口
int port = 8080;
serverBootstrap.bind(port).addListener(future -> {
    if (future.isSuccess()) {
        System.out.println(new Date() + ": 端口[" + port + "]绑定成功!");
    } else {
        System.err.println("端口[" + port + "]绑定失败!");
    }
});
```

## 3 页缓存(PageCache)

页缓存（PageCache)是操作系统对文件的缓存，用来减少对磁盘的 I/O 操作，以页为单位的，内容就是磁盘上的物理块，页缓存能帮助**程序对文件进行顺序读写的速度几乎接近于内存的读写速度**，主要原因就是由于OS使用PageCache机制对读写访问操作进行了性能优化：

**页缓存读取策略**：当进程发起一个读操作 （比如，进程发起一个 read() 系统调用），它首先会检查需要的数据是否在页缓存中：

- 如果在，则放弃访问磁盘，而直接从页缓存中读取
- 如果不在，则内核调度块 I/O 操作从磁盘去读取数据，并读入紧随其后的少数几个页面（不少于一个页面，通常是三个页面），然后将数据放入页缓存中

![img](https:////upload-images.jianshu.io/upload_images/5618238-80de90f22a9b782f.png?imageMogr2/auto-orient/strip|imageView2/2/w/251/format/webp)

**页缓存写策略**：当进程发起write系统调用写数据到文件中，先写到页缓存，然后方法返回。此时数据还没有真正的保存到文件中去，Linux 仅仅将页缓存中的这一页数据标记为“脏”，并且被加入到脏页链表中

然后，由flusher 回写线程周期性将脏页链表中的页写到磁盘，让磁盘中的数据和内存中保持一致，最后清理“脏”标识。在以下三种情况下，脏页会被写回磁盘:

- 空闲内存低于一个特定阈值
- 脏页在内存中驻留超过一个特定的阈值时
- 当用户进程调用 sync() 和 fsync() 系统调用时

RocketMQ中，ConsumeQueue逻辑消费队列存储的数据较少，并且是顺序读取，在page cache机制的预读取作用下，Consume Queue文件的读性能几乎接近读内存，即使在有消息堆积情况下也不会影响性能，提供了2种消息刷盘策略：

- 同步刷盘：在消息真正持久化至磁盘后RocketMQ的Broker端才会真正返回给Producer端一个成功的ACK响应
- 异步刷盘，能充分利用操作系统的PageCache的优势，只要消息写入PageCache即可将成功的ACK返回给Producer端。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了MQ的性能和吞吐量

Kafka实现消息高性能读写也利用了页缓存，这里不再展开



《深入理解Linux内核 —— Daniel P.Bovet》

[Netty之Java堆外内存扫盲贴 ——江南白衣](https://links.jianshu.com/go?to=http%3A%2F%2Fcalvin1978.blogcn.com%2Farticles%2Fdirectbytebuffer.html)

[Java NIO？看这一篇就够了！ ——朱小厮 ](https://links.jianshu.com/go?to=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fc9tkrokcDQR375kiwCeV9w)

[RocketMQ 消息存储流程 ——  Zhao Kun(赵坤)](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.kunzhao.org%2Fblog%2F2018%2F03%2F12%2Frocketmq-message-store-flow)

[一文理解Netty模型架构 ——caison](https://www.jianshu.com/p/6681bfa36c4f)



# Netty

## Netty到底是什么

### 从HTTP说起

 有了Netty，你可以实现自己的HTTP服务器，FTP服务器，UDP服务器，RPC服务器，WebSocket服务器，Redis的Proxy服务器，MySQL的Proxy服务器等等。 

我们回顾一下传统的HTTP服务器的原理 

 1、创建一个ServerSocket，监听并绑定一个端口

 2、一系列客户端来请求这个端口

 3、服务器使用Accept，获得一个来自客户端的Socket连接对象 

4、启动一个新线程处理连接 

  4.1、读Socket，得到字节流

  4.2、解码协议，得到Http请求对象 

  4.3、处理Http请求，得到一个结果，封装成一个HttpResponse对象 

  4.4、编码协议，将结果序列化字节流 写Socket，将字节流发给客户端 

5、继续循环步骤3

HTTP服务器之所以称为HTTP服务器，是因为编码解码协议是HTTP协议，如果协议是Redis协议，那它就成了Redis服务器，如果协议是WebSocket，那它就成了WebSocket服务器，等等。 使用Netty你就可以定制编解码协议，实现自己的特定协议的服务器。

### NIO

上面是一个传统处理http的服务器，但是在高并发的环境下，线程数量会比较多，System load也会比较高，于是就有了NIO。

他并不是Java独有的概念，NIO代表的一个词汇叫着IO多路复用。它是由操作系统提供的系统调用，早期这个操作系统调用的名字是select，但是性能低下，后来渐渐演化成了Linux下的epoll和Mac里的kqueue。我们一般就说是epoll，因为没有人拿苹果电脑作为服务器使用对外提供服务。而Netty就是基于Java NIO技术封装的一套框架。为什么要封装，因为原生的Java NIO使用起来没那么方便，而且还有臭名昭著的bug，Netty把它封装之后，提供了一个易于操作的使用模式和接口，用户使用起来也就便捷多了。

说NIO之前先说一下BIO（Blocking IO）,如何理解这个Blocking呢？

1. 客户端监听（Listen）时，Accept是阻塞的，只有新连接来了，Accept才会返回，主线程才能继
2. 读写socket时，Read是阻塞的，只有请求消息来了，Read才能返回，子线程才能继续处理
3. 读写socket时，Write是阻塞的，只有客户端把消息收了，Write才能返回，子线程才能继续读取下一个请求

传统的BIO模式下，从头到尾的所有线程都是阻塞的，这些线程就干等着，占用系统的资源，什么事也不干。

那么NIO是怎么做到非阻塞的呢。它用的是事件机制。它可以用一个线程把Accept，读写操作，请求处理的逻辑全干了。如果什么事都没得做，它也不会死循环，它会将线程休眠起来，直到下一个事件来了再继续干活，这样的一个线程称之为NIO线程。用伪代码表示：







```
while true {
    events = takeEvents(fds)  // 获取事件，如果没有事件，线程就休眠
    for event in events {
        if event.isAcceptable {
            doAccept() // 新链接来了
        } elif event.isReadable {
            request = doRead() // 读消息
            if request.isComplete() {
                doProcess()
            }
        } elif event.isWriteable {
            doWrite()  // 写消息
        }
    }
}复制代码
```



### Reactor线程模型

**Reactor单线程模型**

一个NIO线程+一个accept线程：

![img](https://user-gold-cdn.xitu.io/2018/11/1/166cf694962300a8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**Reactor多线程模型**

![img](https://user-gold-cdn.xitu.io/2018/11/1/166cf69cdf959355?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**Reactor主从模型**



主从Reactor多线程：多个acceptor的NIO线程池用于接受客户端的连接

![img](https://user-gold-cdn.xitu.io/2018/11/1/166cf6b2c711bc0f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Netty可以基于如上三种模型进行灵活的配置。

### 总结

Netty是建立在NIO基础之上，Netty在NIO之上又提供了更高层次的抽象。

在Netty里面，Accept连接可以使用单独的线程池去处理，读写操作又是另外的线程池来处理。

Accept连接和读写操作也可以使用同一个线程池来进行处理。而请求处理逻辑既可以使用单独的线程池进行处理，也可以跟放在读写线程一块处理。线程池中的每一个线程都是NIO线程。用户可以根据实际情况进行组装，构造出满足系统需求的高性能并发模型。



##  为什么选择Netty 

如果不用netty，使用原生JDK的话，有如下问题：

1、API复杂

2、对多线程很熟悉：因为NIO涉及到Reactor模式

3、高可用的话：需要出路断连重连、半包读写、失败缓存等问题

4、JDK NIO的bug

而Netty来说，他的api简单、性能高而且社区活跃（dubbo、rocketmq等都使用了它）

## 什么是TCP 粘包/拆包 

### 现象

先看如下代码，这个代码是使用netty在client端重复写100次数据给server端，ByteBuf是netty的一个字节容器，里面存放是的需要发送的数据

```
public class FirstClientHandler extends ChannelInboundHandlerAdapter {    
    @Override    
    public void channelActive(ChannelHandlerContext ctx) {       
        for (int i = 0; i < 1000; i++) {            
            ByteBuf buffer = getByteBuf(ctx);            
            ctx.channel().writeAndFlush(buffer);        
        }    
    }    
    private ByteBuf getByteBuf(ChannelHandlerContext ctx) {        
        byte[] bytes = "你好，我的名字是1234567!".getBytes(Charset.forName("utf-8"));        
        ByteBuf buffer = ctx.alloc().buffer();        
        buffer.writeBytes(bytes);        
        return buffer;    
    }
}复制代码
```

从client端读取到的数据为：

![img](https://user-gold-cdn.xitu.io/2018/11/1/166cf7ca469a221e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

从服务端的控制台输出可以看出，存在三种类型的输出

1. 一种是正常的字符串输出。
2. 一种是多个字符串“粘”在了一起，我们定义这种 ByteBuf 为粘包。
3. 一种是一个字符串被“拆”开，形成一个破碎的包，我们定义这种 ByteBuf 为半包。

### 透过现象分析原因

应用层面使用了Netty，但是对于操作系统来说，只认TCP协议，尽管我们的应用层是按照 ByteBuf 为 单位来发送数据，server按照Bytebuf读取，但是到了底层操作系统仍然是按照字节流发送数据，因此，数据到了服务端，也是按照字节流的方式读入，然后到了 Netty 应用层面，重新拼装成 ByteBuf，而这里的 ByteBuf 与客户端按顺序发送的 ByteBuf 可能是不对等的。因此，我们需要在客户端根据自定义协议来组装我们应用层的数据包，然后在服务端根据我们的应用层的协议来组装数据包，这个过程通常在服务端称为拆包，而在客户端称为粘包。

拆包和粘包是相对的，一端粘了包，另外一端就需要将粘过的包拆开，发送端将三个数据包粘成两个 TCP 数据包发送到接收端，接收端就需要根据应用协议将两个数据包重新组装成三个数据包。

### 如何解决

在没有 Netty 的情况下，用户如果自己需要拆包，基本原理就是不断从 TCP 缓冲区中读取数据，每次读取完都需要判断是否是一个完整的数据包 如果当前读取的数据不足以拼接成一个完整的业务数据包，那就保留该数据，继续从 TCP 缓冲区中读取，直到得到一个完整的数据包。 如果当前读到的数据加上已经读取的数据足够拼接成一个数据包，那就将已经读取的数据拼接上本次读取的数据，构成一个完整的业务数据包传递到业务逻辑，多余的数据仍然保留，以便和下次读到的数据尝试拼接。 

而在Netty中，已经造好了许多类型的拆包器，我们直接用就好：

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="926"></svg>)

选好拆包器后，在代码中client段和server端将拆包器加入到chanelPipeline之中就好了:

如上实例中：

客户端：

```
ch.pipeline().addLast(new FixedLengthFrameDecoder(31));复制代码
```

服务端：

```
ch.pipeline().addLast(new FixedLengthFrameDecoder(31));复制代码
```

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="882"></svg>)

## Netty 的零拷贝

### 传统意义的拷贝

是在发送数据的时候，传统的实现方式是：

\1. `File.read(bytes)`
\2. `Socket.send(bytes)`
这种方式需要四次数据拷贝和四次上下文切换：

\1. 数据从磁盘读取到内核的read buffer

\2. 数据从内核缓冲区拷贝到用户缓冲区
\3. 数据从用户缓冲区拷贝到内核的socket buffer
\4. 数据从内核的socket buffer拷贝到网卡接口（硬件）的缓冲区



### 零拷贝的概念

明显上面的第二步和第三步是没有必要的，通过java的FileChannel.transferTo方法，可以避免上面两次多余的拷贝（当然这需要底层操作系统支持）

\1. 调用transferTo,数据从文件由DMA引擎拷贝到内核read buffer

\2. 接着DMA从内核read buffer将数据拷贝到网卡接口buffer

上面的两次操作都不需要CPU参与，所以就达到了零拷贝。

### Netty中的零拷贝

主要体现在三个方面：

**1、bytebuffer**

Netty发送和接收消息主要使用bytebuffer，bytebuffer使用对外内存（DirectMemory）直接进行Socket读写。

原因：如果使用传统的堆内存进行Socket读写，JVM会将堆内存buffer拷贝一份到直接内存中然后再写入socket，多了一次缓冲区的内存拷贝。DirectMemory中可以直接通过DMA发送到网卡接口

**2、Composite Buffers**

传统的ByteBuffer，如果需要将两个ByteBuffer中的数据组合到一起，我们需要首先创建一个size=size1+size2大小的新的数组，然后将两个数组中的数据拷贝到新的数组中。但是使用Netty提供的组合ByteBuf，就可以避免这样的操作，因为CompositeByteBuf并没有真正将多个Buffer组合起来，而是保存了它们的引用，从而避免了数据的拷贝，实现了零拷贝。

**3、对于FileChannel.transferTo的使用**

Netty中使用了FileChannel的transferTo方法，该方法依赖于操作系统实现零拷贝。

## Netty 内部执行流程

### 服务端：

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="794"></svg>)

![img](data:image/svg+xml;utf8,<?xml version="1.0"?><svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1280" height="653"></svg>)

1、创建ServerBootStrap实例

2、设置并绑定Reactor线程池：EventLoopGroup，EventLoop就是处理所有注册到本线程的Selector上面的Channel

3、设置并绑定服务端的channel

4、5、创建处理网络事件的ChannelPipeline和handler，网络时间以流的形式在其中流转，handler完成多数的功能定制：比如编解码 SSl安全认证

6、绑定并启动监听端口

7、当轮训到准备就绪的channel后，由Reactor线程：NioEventLoop执行pipline中的方法，最终调度并执行channelHandler 

### 客户端

![img](https://user-gold-cdn.xitu.io/2018/11/1/166cf951ce743191?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



![img](https://user-gold-cdn.xitu.io/2018/11/1/166cf957e59b5066?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


作者：lyowish
链接：https://juejin.im/post/5bdaf8ea6fb9a0227b02275a
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。