# 简介与安装



**目录**

- [1、Nginx 的简介](https://www.cnblogs.com/ysocean/p/9384877.html#_label0)
- [2、Nginx 的常用功能](https://www.cnblogs.com/ysocean/p/9384877.html#_label1)
- 3、Nginx 安装
  - [①、下载地址](https://www.cnblogs.com/ysocean/p/9384877.html#_label2_0)
  - [②、Windows 版本安装](https://www.cnblogs.com/ysocean/p/9384877.html#_label2_1)
  - [③、Linux 版本安装](https://www.cnblogs.com/ysocean/p/9384877.html#_label2_2)

 

------

　　说到 Nginx ，可能大家最先想到的就是其负载均衡以及反向代理的功能。没错，这也是当前使用 Nginx 最频繁的两个功能，但是 Nginx 可不仅仅只有这两个功能，其作用还是挺大的，本系列博客就来慢慢解开 Nginx 神秘的面纱。

[回到顶部](https://www.cnblogs.com/ysocean/p/9384877.html#_labelTop)

### 1、Nginx 的简介

　　Nginx 是由俄罗斯人 Igor Sysoev 设计开发的，开发工作从2002 年开始，第一次公开发布在 2004 年 10 月 4 日。

　　官方网站为：http://nginx.org/ 。它是一款免费开源的高性能 HTTP 代理服务器及反向代理服务器（Reverse Proxy）产品，同时它还可以提供 IMAP/POP3 邮件代理服务等功能。它高并发性能很好，官方测试能够支撑 5 万的并发量；运行时内存和 CPU 占用率低，配置简单，容易上手，而且运行非常稳定。

[回到顶部](https://www.cnblogs.com/ysocean/p/9384877.html#_labelTop)

### 2、Nginx 的常用功能

　　其实 Nginx 的功能特别多，这里我只介绍几个常用的功能，具体的大家可以参考[官网](http://nginx.org/en/)介绍。

　　**①、反向代理**

　　这是 Nginx 服务器作为 WEB 服务器的主要功能之一，客户端向服务器发送请求时，会首先经过 Nginx 服务器，由服务器将请求分发到相应的 WEB 服务器。正向代理是代理客户端，而反向代理则是代理服务器，Nginx 在提供反向代理服务方面，通过使用正则表达式进行相关配置，采取不同的转发策略，配置相当灵活，而且在配置后端转发请求时，完全不用关心网络环境如何，可以指定任意的IP地址和端口号，或其他类型的连接、请求等。

　　**②、负载均衡**

　　这也是 Nginx 最常用的功能之一，负载均衡，一方面是将单一的重负载分担到多个网络节点上做并行处理，每个节点处理结束后将结果汇总返回给用户，这样可以大幅度提高网络系统的处理能力；另一方面将大量的前端并发请求或数据流量分担到多个后端网络节点分别处理，这样可以有效减少前端用户等待相应的时间。而 Nginx 负载均衡都是属于后一方面，主要是对大量前端访问或流量进行分流，已保证前端用户访问效率，并可以减少后端服务器处理压力。

　　**③、Web 缓存**

　　在很多优秀的网站中，Nginx 可以作为前置缓存服务器，它被用于缓存前端请求，从而提高 Web服务器的性能。Nginx 会对用户已经访问过的内容在服务器本地建立副本，这样在一段时间内再次访问该数据，就不需要通过 Nginx 服务器向后端发出请求。减轻网络拥堵，减小数据传输延时，提高用户访问速度。

[回到顶部](https://www.cnblogs.com/ysocean/p/9384877.html#_labelTop)

### 3、Nginx 安装

　　关于 Nginx 的安装，分为在 Windows 平台和 Linux 平台安装，Windows 版本的 Nginx 服务器在效率上要比 Linux 版本的 Nginx 服务器差一些，而且实际使用的一般都是 Linux 平台的 Nginx 服务器。所以后期我们介绍时也会以 Linux 版本的为主。



#### ①、下载地址

　　Nginx 下载地址：http://nginx.org/en/download.html

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729191301014-1758800936.png)

　　开发版本主要用于 Nginx 软件项目的研发，稳定版本说明可以作为 Web 服务器投入商业应用。这里我们选择当前稳定版本：**nginx-1.14.0**



#### ②、Windows 版本安装

　　我们将上一步下载的 nginx-1.14.0.zip 文件解压到当前目录。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729192109716-560612925.png)

　　解压目录如下：

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729192204644-36563477.png)

　　下面对这个目录下的主要文件夹进行介绍：

　　1、conf 目录：存放 Nginx 的主要配置文件，很多功能实现都是通过配置该目录下的 nginx.conf 文件，后面我们会详细介绍。

　　2、docs 目录：存放 Nginx 服务器的主要文档资料，包括 Nginx 服务器的 LICENSE、OpenSSL 的 LICENSE 、PCRE 的 LICENSE 以及 zlib 的 LICENSE ，还包括本版本的 Nginx服务器升级的版本变更说明，以及 README 文档。

　　3、html 目录：存放了两个后缀名为 .html 的静态网页文件，这两个文件与 Nginx 服务器的运行相关。

　　4、logs 目录：存放 Nginx 服务器运行的日志文件。

　　5、nginx.exe：启动 Nginx 服务器的exe文件，如果 conf 目录下的 nginx.conf 文件配置正确的话，通过该文件即可启动 Nginx 服务器。

　　**一、启动 nginx**

　　双击解压之后目录中的 nginx.exe 文件，出现一闪而过的画面，则启动成功。

　　然后在浏览器中输入 http://localhost 或者 http://localhost:80 出现如下界面即启动成功。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729212236786-1123186926.png)

　　ps:该页面即是上面解压目录中 html 目录下的 index.html 文件。

 　**二、关闭 nginx**

　　进入到解压之后的目录，输入如下命令：

```
1 nginx.exe -s stop
```

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729222251887-1052720801.png)

　　或者也可以打开任务管理器，找到 nginx 的进程，直接右键结束。



#### ③、Linux 版本安装

　　选择的 Linux 系统为 CentOS6.8。

　　**一、安装 nginx 环境**

```
1 yum install gcc-c++
2 yum install -y pcre pcre-devel
3 yum install -y zlib zlib-devel
4 yum install -y openssl openssl-devel
```

　　对于 gcc，因为安装nginx需要先将官网下载的源码进行编译，编译依赖gcc环境，如果没有gcc环境的话，需要安装gcc。

　　对于 pcre，prce(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。

　　对于 zlib，zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。

　　对于 openssl，OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。

　　**二、编译安装**

　　首先将下载的 nginx-1.14.0.tar.gz 文件复制到 Linux 系统中，然后解压：

```
1 tar -zxvf nginx-1.14.0.tar.gz
```

　　接着进入到解压之后的目录，进行编译安装。

```
1 ./configure --prefix=/usr/local/nginx
2 make
3 make install
```

　　注意：指定 /usr/local/nginx 为nginx 服务安装的目录。

　　**三、启动 nginx**

　　进入到 /usr/local/nginx 目录，文件目录显示如下：

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729220525790-1171040940.png)

　　接着我们进入到 sbin 目录，通过如下命令启动 nginx：

```
./nginx
```

　　当然你也可以配置环境命令，这样在任意目录都能启动 nginx。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729220731705-1475271343.png)

　　Linux 没有消息就好消息，不提示任何信息说明启动成功。

　　或者也可以输入如下命令，查看 nginx 是否有服务正在运行：

```
ps -ef | grep nginx
```

　　然后我们在浏览器输入Linux系统的IP地址，出现windows安装成功的界面即可。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729221057300-343596218.png)

　　**四、关闭 nginx**

　　有两种方式：

　　方式1：快速停止

```
1 cd /usr/local/nginx/sbin
2 ./nginx -s stop
```

　　此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。不太友好。

　　方式2：平缓停止

```
1 cd /usr/local/nginx/sbin
2 ./nginx -s quit
```

　　此方式是指允许 nginx 服务将当前正在处理的网络请求处理完成，但不在接收新的请求，之后关闭连接，停止工作。

　　**五、重启 nginx**

　　方式1：先停止再启动

```
1 ./nginx -s quit
2 ./nginx
```

　　相当于先执行停止命令再执行启动命令。

　　方式2：重新加载配置文件

```
1 ./nginx -s reload
```

　　通常我们使用nginx修改最多的便是其配置文件 nginx.conf。修改之后想要让配置文件生效而不用重启 nginx，便可以使用此命令。

　　**六、检测配置文件语法是否正确**

　　方式1：通过如下命令，指定需要检查的配置文件

```
nginx -t -c /usr/local/nginx/conf/nginx.conf
```

　　方式2：通过如下命令，不加 -c 参数，默认检测nginx.conf 配置文件。

```
nginx -t 
```





# nginx.conf 配置文件



**目录**

- [1、nginx.conf 的主体结构](https://www.cnblogs.com/ysocean/p/9384880.html#_label0)
- [2、全局块](https://www.cnblogs.com/ysocean/p/9384880.html#_label1)
- [3、events 块](https://www.cnblogs.com/ysocean/p/9384880.html#_label2)
- 4、http 块
  - [①、http 全局块](https://www.cnblogs.com/ysocean/p/9384880.html#_label3_0)
  - [②、server 块](https://www.cnblogs.com/ysocean/p/9384880.html#_label3_1)

 

------

　　上一篇博客我们将 nginx 安装在 /usr/local/nginx 目录下，其默认的配置文件都放在这个目录的 conf 目录下，而主配置文件 nginx.conf 也在其中，后续对 nginx 的使用基本上都是对此配置文件进行相应的修改，所以本篇博客我们先大致介绍一下该配置文件的结构。

　　![img](https://images2018.cnblogs.com/blog/1120165/201807/1120165-20180729225520207-404971432.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/9384880.html#_labelTop)

### 1、nginx.conf 的主体结构

　　打开此文件，内容如下：

​		开头的表示注释内容，我们去掉所有以 # 开头的段落，精简之后的内容如下：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 worker_processes  1;
 2 
 3 events {
 4     worker_connections  1024;
 5 }
 6 
 7 
 8 http {
 9     include       mime.types;
10     default_type  application/octet-stream;
11 
12 
13     sendfile        on;
14 
15     keepalive_timeout  65;
16 
17     server {
18         listen       80;
19         server_name  localhost;
20 
21         location / {
22             root   html;
23             index  index.html index.htm;
24         }
25 
26         error_page   500 502 503 504  /50x.html;
27         location = /50x.html {
28             root   html;
29         }
30 
31     }
32 
33 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

 　根据上述文件，我们可以很明显的将 nginx.conf 配置文件分为三部分：

[回到顶部](https://www.cnblogs.com/ysocean/p/9384880.html#_labelTop)

### 2、全局块

　　从配置文件开始到 events 块之间的内容，主要会设置一些影响nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的 worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。

　　比如上面第一行配置的：

```
worker_processes  1;
```

　　这是 Nginx 服务器并发处理服务的关键配置，worker_processes 值越大，可以支持的并发处理量也越多，但是会受到硬件、软件等设备的制约，这个后面会详细介绍。

[回到顶部](https://www.cnblogs.com/ysocean/p/9384880.html#_labelTop)

### 3、events 块

　　比如上面的配置：

```
events {
    worker_connections  1024;
}
```

　　events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

　　上述例子就表示每个 work process 支持的最大连接数为 1024.

　　这部分的配置对 Nginx 的性能影响较大，在实际中应该灵活配置。

[回到顶部](https://www.cnblogs.com/ysocean/p/9384880.html#_labelTop)

### 4、http 块

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1 http {
 2     include       mime.types;
 3     default_type  application/octet-stream;
 4 
 5 
 6     sendfile        on;
 7 
 8     keepalive_timeout  65;
 9 
10     server {
11         listen       80;
12         server_name  localhost;
13 
14         location / {
15             root   html;
16             index  index.html index.htm;
17         }
18 
19         error_page   500 502 503 504  /50x.html;
20         location = /50x.html {
21             root   html;
22         }
23 
24     }
25 
26 }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　这算是 Nginx 服务器配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里。

　　需要注意的是：http 块也可以包括 **http全局块**、**server 块**。



#### ①、http 全局块

　　http全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。



#### ②、server 块

　　这块和虚拟主机有密切关系，虚拟主机从用户角度看，和一台独立的硬件主机是完全一样的，该技术的产生是为了节省互联网服务器硬件成本。后面会详细介绍虚拟主机的概念。

　　每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机。

　　而每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块。

　　**1、全局 server 块**

　　最常见的配置是本虚拟机主机的监听配置和本虚拟主机的名称或IP配置。

　　**2、location 块**

　　一个 server 块可以配置多个 location 块。

　　这块的主要作用是基于 Nginx 服务器接收到的请求字符串（例如 server_name/uri-string），对虚拟主机名称（也可以是IP别名）之外的字符串（例如 前面的 /uri-string）进行匹配，对特定的请求进行处理。地址定向、数据缓存和应答控制等功能，还有许多第三方模块的配置也在这里进行。



# nginx 反向代理



**目录**

- [1、代理](https://www.cnblogs.com/ysocean/p/9392908.html#_label0)
- [2、正向代理](https://www.cnblogs.com/ysocean/p/9392908.html#_label1)
- [3、反向代理](https://www.cnblogs.com/ysocean/p/9392908.html#_label2)
- [4、Nginx 反向代理](https://www.cnblogs.com/ysocean/p/9392908.html#_label3)
- 5、Nginx 反向代理相关指令介绍
  - [①、listen](https://www.cnblogs.com/ysocean/p/9392908.html#_label4_0)
  - [②、server_name](https://www.cnblogs.com/ysocean/p/9392908.html#_label4_1)
  - [③、location](https://www.cnblogs.com/ysocean/p/9392908.html#_label4_2)
  - [④、proxy_pass](https://www.cnblogs.com/ysocean/p/9392908.html#_label4_3)
  - [⑤、index](https://www.cnblogs.com/ysocean/p/9392908.html#_label4_4)

 

------

　　Nginx 服务器的反向代理服务是其最常用的重要功能，由反向代理服务也可以衍生出很多与此相关的 Nginx 服务器重要功能，比如后面会介绍的负载均衡。本篇博客我们会先介绍 Nginx 的反向代理，当然在了解反向代理之前，我们需要先知道什么是代理以及什么是正向代理。

[回到顶部](https://www.cnblogs.com/ysocean/p/9392908.html#_labelTop)

### 1、代理

　　在Java设计模式中，代理模式是这样定义的：给某个对象提供一个代理对象，并由代理对象控制原对象的引用。

　　可能大家不太明白这句话，在举一个现实生活中的例子：比如我们要买一间二手房，虽然我们可以自己去找房源，但是这太花费时间精力了，而且房屋质量检测以及房屋过户等一系列手续也都得我们去办，再说现在这个社会，等我们找到房源，说不定房子都已经涨价了，那么怎么办呢？最简单快捷的方法就是找二手房中介公司（为什么？别人那里房源多啊），于是我们就委托中介公司来给我找合适的房子，以及后续的质量检测过户等操作，我们只需要选好自己想要的房子，然后交钱就行了。

　　代理简单来说，就是如果我们想做什么，但又不想直接去做，那么这时候就找另外一个人帮我们去做。那么这个例子里面的中介公司就是给我们做代理服务的，我们委托中介公司帮我们找房子。

　　Nginx 主要能够代理如下几种协议，其中用到的最多的就是做Http代理服务器。

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180905232339438-913760288.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/9392908.html#_labelTop)

### 2、正向代理

　　弄清楚什么是代理了，那么什么又是正向代理呢？

　　这里我再举一个例子：大家都知道，现在国内是访问不了 Google的，那么怎么才能访问 Google呢？我们又想，美国人不是能访问 Google吗（这不废话，Google就是美国的），如果我们电脑的对外公网 IP 地址能变成美国的 IP 地址，那不就可以访问 Google了。你很聪明，VPN 就是这样产生的。我们在访问 Google 时，先连上 VPN 服务器将我们的 IP 地址变成美国的 IP 地址，然后就可以顺利的访问了。

　　这里的 VPN 就是做正向代理的。正向代理服务器位于客户端和服务器之间，为了向服务器获取数据，客户端要向代理服务器发送一个请求，并指定目标服务器，代理服务器将目标服务器返回的数据转交给客户端。这里客户端是要进行一些正向代理的设置的。

　　PS：这里介绍一下什么是 VPN，VPN 通俗的讲就是一种中转服务，当我们电脑接入 VPN 后，我们对外 IP 地址就会变成 VPN 服务器的 公网 IP，我们请求或接受任何数据都会通过这个VPN 服务器然后传入到我们本机。这样做有什么好处呢？比如 VPN 游戏加速方面的原理，我们要玩网通区的 LOL，但是本机接入的是电信的宽带，玩网通区的会比较卡，这时候就利用 VPN 将电信网络变为网通网络，然后在玩网通区的LOL就不会卡了（注意：VPN 是不能增加带宽的，不要以为不卡了是因为网速提升了）。

　　可能听到这里大家还是很抽象，没关系，和下面的反向代理对比理解就简单了。

[回到顶部](https://www.cnblogs.com/ysocean/p/9392908.html#_labelTop)

### 3、反向代理

　　反向代理和正向代理的区别就是：**正向代理代理客户端，反向代理代理服务器。**

　　反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址。

　理解这两种代理的关键在于代理服务器所代理的对象是什么，正向代理代理的是客户端，我们需要在客户端进行一些代理的设置。而反向代理代理的是服务器，作为客户端的我们是无法感知到服务器的真实存在的。

　　总结起来还是一句话：**正向代理代理客户端，反向代理代理服务器。**

[回到顶部](https://www.cnblogs.com/ysocean/p/9392908.html#_labelTop)

### 4、Nginx 反向代理

　　范例：使用 nginx 反向代理 www.123.com 直接跳转到127.0.0.1:8080

　　①、启动一个 tomcat，浏览器地址栏输入 127.0.0.1:8080，出现如下界面

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180905235823351-2004614694.png)

 

　　②、通过修改本地 host 文件，将 www.123.com 映射到 127.0.0.1

```
127.0.0.1 www.123.com
```

　　将上面代码添加到 Windows 的host 文件中，

　　配置完成之后，我们便可以通过 www.123.com:8080 访问到第一步出现的 Tomcat初始界面。

　　那么如何只需要输入 www.123.com 便可以跳转到 Tomcat初始界面呢？便用到 nginx的反向代理。

　　③、在 nginx.conf 配置文件中增加如下配置：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
1     server {
2         listen       80;
3         server_name  www.123.com;
4 
5         location / {
6             proxy_pass http://127.0.0.1:8080;
7             index  index.html index.htm index.jsp;
8         }
9     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　如上配置，我们监听80端口，访问域名为www.123.com，不加端口号时默认为80端口，故访问该域名时会跳转到127.0.0.1:8080路径上。

　　我们在浏览器端输入 www.123.com 

　　④、总结

　　其实这里更贴切的说是通过nginx代理端口，原先访问的是8080端口，通过nginx代理之后，通过80端口就可以访问了。

[回到顶部](https://www.cnblogs.com/ysocean/p/9392908.html#_labelTop)

### 5、Nginx 反向代理相关指令介绍



#### ①、listen

　　该指令用于配置网络监听。主要有如下三种配置语法结构：

　　一、配置监听的IP地址

```
listen address[:port] [default_server] [setfib=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [deferred]
    [accept_filter=filter] [bind] [ssl];
```

　　二、配置监听端口

```
listen port[default_server] [setfib=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] 
    [deferred] [bind] [ipv6only=on|off] [ssl];
```

　三、配置 UNIX Domain Socket

```
listen unix:path [default_server]  [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] 
    [deferred] [bind] [ssl];
```

　　上面的配置看似比较复杂，其实使用起来是比较简单的：

```
1 listen *:80 | *:8080 #监听所有80端口和8080端口
2 listen  IP_address:port   #监听指定的地址和端口号
3 listen  IP_address     #监听指定ip地址所有端口
4 listen port     #监听该端口的所有IP连接
```

　　下面分别解释每个选项的具体含义：

　　1、address:IP地址，如果是 IPV6地址，需要使用中括号[] 括起来，比如[fe80::1]等。

　　2、port:端口号，如果只定义了IP地址，没有定义端口号，那么就使用80端口。

　　3、path:socket文件路径，如 var/run/nginx.sock等。

　　4、default_server:标识符，将此虚拟主机设置为 address:port 的默认主机。（在 nginx-0.8.21 之前使用的是 default 指令）

　　5、 setfib=number:Nginx-0.8.44 中使用这个变量监听 socket 关联路由表，目前只对 FreeBSD 起作用，不常用。

　　6、backlog=number:设置监听函数listen()最多允许多少网络连接同时处于挂起状态，在 FreeBSD 中默认为 -1,其他平台默认为511.

　　7、rcvbuf=size:设置监听socket接收缓存区大小。

　　8、sndbuf=size:设置监听socket发送缓存区大小。

　　9、deferred:标识符，将accept()设置为Deferred模式。

　　10、accept_filter=filter:设置监听端口对所有请求进行过滤，被过滤的内容不能被接收和处理，本指令只在 FreeBSD 和 NetBSD 5.0+ 平台下有效。filter 可以设置为 dataready 或 httpready 。

　　11、bind:标识符，使用独立的bind() 处理此address:port，一般情况下，对于端口相同而IP地址不同的多个连接，Nginx 服务器将只使用一个监听指令，并使用 bind() 处理端口相同的所有连接。

　　12、ssl:标识符，设置会话连接使用 SSL模式进行，此标识符和Nginx服务器提供的 HTTPS 服务有关。



#### ②、server_name

　　该指令用于虚拟主机的配置。通常分为以下两种：

　　**1、基于名称的虚拟主机配置**

　　语法格式如下：

```
server_name   name ...;
```

　　一、对于name 来说，可以只有一个名称，也可以有多个名称，中间用空格隔开。而每个名字由两段或者三段组成，每段之间用“.”隔开。

```
server_name 123.com www.123.com
```

　　二、可以使用通配符“*”，但通配符只能用在由三段字符组成的首段或者尾端，或者由两端字符组成的尾端。

```
server_name *.123.com www.123.*
```

　　三、还可以使用正则表达式，用“~”作为正则表达式字符串的开始标记。

```
server_name ~^www\d+\.123\.com$;
```

　　该表达式“~”表示匹配正则表达式，以www开头（“^”表示开头），紧跟着一个0~9之间的数字，在紧跟“.123.co”，最后跟着“m”($表示结尾)

　　以上匹配的顺序优先级如下：

```
1 ①、准确匹配 server_name
2 ②、通配符在开始时匹配 server_name 成功
3 ③、通配符在结尾时匹配 server_name 成功
4 ④、正则表达式匹配 server_name 成功
```

　　**2、基于 IP 地址的虚拟主机配置**

　　语法结构和基于域名匹配一样，而且不需要考虑通配符和正则表达式的问题。

```
server_name 192.168.1.1
```



#### ③、location

　　该指令用于匹配 URL。

　　语法如下：

```
1 location [ = | ~ | ~* | ^~] uri {
2 
3 }
```

　　1、= ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配成功，就停止继续向下搜索并立即处理该请求。

　　2、~：用于表示 uri 包含正则表达式，并且区分大小写。

　　3、~*：用于表示 uri 包含正则表达式，并且不区分大小写。

　　4、^~：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location 块中的正则 uri 和请求字符串做匹配。

　　注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。



#### ④、proxy_pass

　　该指令用于设置被代理服务器的地址。可以是主机名称、IP地址加端口号的形式。

　　语法结构如下：

```
proxy_pass URL;
```

　　URL 为被代理服务器的地址，可以包含传输协议、主机名称或IP地址加端口号，URI等。

```
proxy_pass  http://www.123.com/uri;
```



#### ⑤、index

　　该指令用于设置网站的默认首页。

　　语法为：

```
index  filename ...;
```

　　后面的文件名称可以有多个，中间用空格隔开。

```
index  index.html index.jsp;
```

　　通常该指令有两个作用：第一个是用户在请求访问网站时，请求地址可以不写首页名称；第二个是可以对一个请求，根据请求内容而设置不同的首页。



# nginx 负载均衡



**目录**

- [1、负载均衡的由来](https://www.cnblogs.com/ysocean/p/9392912.html#_label0)
- 2、Nginx实现负载均衡
  - [①、普通轮询算法](https://www.cnblogs.com/ysocean/p/9392912.html#_label1_0)
  - [②、基于比例加权轮询](https://www.cnblogs.com/ysocean/p/9392912.html#_label1_1)
  - [③、基于IP路由负载](https://www.cnblogs.com/ysocean/p/9392912.html#_label1_2)
  - [④、基于服务器响应时间负载分配](https://www.cnblogs.com/ysocean/p/9392912.html#_label1_3)
  - [⑤、对不同域名实现负载均衡](https://www.cnblogs.com/ysocean/p/9392912.html#_label1_4)

 

------

　　在上一篇博客我们介绍了 Nginx 一个很重要的功能——代理，包括正向代理和反向代理。这两个代理的核心区别是：正向代理代理的是客户端，而反向代理代理的是服务器。其中我们又重点介绍了反向代理，以及如何通过 Nginx 来实现反向代理。那么了解了Nginx的反向代理之后，我们要通过Nginx的反向代理实现另一个重要功能——负载均衡。

[回到顶部](https://www.cnblogs.com/ysocean/p/9392912.html#_labelTop)

### 1、负载均衡的由来

　　早期的系统架构，基本上都是如下形式的：

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908121550571-375749058.png)

 

　　客户端发送多个请求到服务器，服务器处理请求，有一些可能要与数据库进行交互，服务器处理完毕后，再将结果返回给客户端。

　　这种架构模式对于早期的系统相对单一，并发请求相对较少的情况下是比较适合的，成本也低。但是随着信息数量的不断增长，访问量和数据量的飞速增长，以及系统业务的复杂度增加，这种架构会造成服务器相应客户端的请求日益缓慢，并发量特别大的时候，还容易造成服务器直接崩溃。很明显这是由于服务器性能的瓶颈造成的问题，那么如何解决这种情况呢？

　　我们首先想到的可能是升级服务器的配置，比如提高CPU执行频率，加大内存等提高机器的物理性能来解决此问题，但是我们知道[摩尔定律](https://www.cnblogs.com/ysocean/p/7641540.html)的日益失效，硬件的性能提升已经不能满足日益提升的需求了。最明显的一个例子，天猫双十一当天，某个热销商品的瞬时访问量是极其庞大的，那么类似上面的系统架构，将机器都增加到现有的顶级物理配置，都是不能够满足需求的。那么怎么办呢？

　　上面的分析我们去掉了增加服务器物理配置来解决问题的办法，也就是说纵向解决问题的办法行不通了，那么横向增加服务器的数量呢？这时候集群的概念产生了，单个服务器解决不了，我们增加服务器的数量，然后将请求分发到各个服务器上，将原先请求集中到单个服务器上的情况改为将请求分发到多个服务器上，将负载分发到不同的服务器，也就是我们所说的**负载均衡**。

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908131139969-1777205215.png)

 

　　负载均衡完美的解决了单个服务器硬件性能瓶颈的问题，但是随着而来的如何实现负载均衡呢？客户端怎么知道要将请求发送到那个服务器去处理呢？

[回到顶部](https://www.cnblogs.com/ysocean/p/9392912.html#_labelTop)

### 2、Nginx实现负载均衡

　　Nginx 服务器是介于客户端和服务器之间的中介，通过上一篇博客讲解的反向代理的功能，客户端发送的请求先经过 Nginx ，然后通过 Nginx 将请求根据相应的规则分发到相应的服务器。

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908131700302-1529457842.png)

　　主要配置指令为上一讲的 pass_proxy 指令以及 upstream 指令。负载均衡主要通过专门的硬件设备或者软件算法实现。通过硬件设备实现的负载均衡效果好、效率高、性能稳定，但是成本较高。而通过软件实现的负载均衡主要依赖于均衡算法的选择和程序的健壮性。均衡算法又主要分为两大类：

　　静态负载均衡算法：主要包括轮询算法、基于比率的加权轮询算法或者基于优先级的加权轮询算法。

　　动态负载均衡算法：主要包括基于任务量的最少连接优化算法、基于性能的最快响应优先算法、预测算法及动态性能分配算法等。

　　静态负载均衡算法在一般网络环境下也能表现的比较好，动态负载均衡算法更加适用于复杂的网络环境。

　　例子：

#### ①、普通轮询算法

　　这是Nginx 默认的轮询算法。

```
例子：两台相同的Tomcat服务器，通过 localhost:8080 访问Tomcat1，通过 localhost:8081访问Tomcat2，现在我们要输入 localhost 这个地址，可以在这两个Tomcat服务器之间进行交替访问。
```

　　一、分别修改两个Tomcat服务器的端口为8080和8081。然后再修改Tomcat的首页，使得访问这两个页面时能够区分。如下：

　　修改端口号文件为 server.xml ：

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908145855894-2044798993.png)

　　修改首页的路径为：webapps/ROOT/index.jsp

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908150006858-630938275.png)

　　修改完成之后，分别启动这两个Tomcat服务器，然后分别输入相应的地址端口号：

　　输入地址：localhost:8081

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908150514476-1375755078.png)

 

 　输入地址：localhost:8080

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908150452914-866570966.png)

 　二、修改 nginx 的配置文件 nginx.conf 

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     upstream OrdinaryPolling {
 2     server 127.0.0.1:8080;
 3     server 127.0.0.1:8081;
 4     }
 5     server {
 6         listen       80;
 7         server_name  localhost;
 8 
 9         location / {
10             proxy_pass http://OrdinaryPolling;
11             index  index.html index.htm index.jsp;
12         
13         }
14     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　三、启动 nginx。然后在浏览器输入localhost 地址，观看页面变化：

 　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908152509125-1859907953.gif)



#### ②、基于比例加权轮询

　　上述两台Tomcat服务器基本上是交替进行访问的。但是这里我们有个需求：

　　**由于Tomcat1服务器的配置更高点，我们希望该服务器接受更多的请求，而 Tomcat2 服务器配置低，希望其处理相对较少的请求。**

　　那么这时候就用到了加权轮询机制了。

　　nginx.conf 配置文件如下：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     upstream OrdinaryPolling {
 2     server 127.0.0.1:8080 weight=5;
 3     server 127.0.0.1:8081 weight=2;
 4     }
 5     server {
 6         listen       80;
 7         server_name  localhost;
 8 
 9         location / {
10             proxy_pass http://OrdinaryPolling;
11             index  index.html index.htm index.jsp;
12         
13         }
14     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　其实对比上面不加权的轮询方式，这里在 upstream 指令中多了一个 weight 指令。该指令用于配置前面请求处理的权重，默认值为 1。

　　也就是说：第一种不加权的普通轮询，其实其加权值 weight 都为 1。

　　下面我们看页面相应结果：

　　![img](https://images2018.cnblogs.com/blog/1120165/201809/1120165-20180908153300294-418662600.gif)

 

 　明显 8080 端口号出现的次数更多，试验的次数越多越接近我们配置的比例。



#### ③、基于IP路由负载

　　我们知道一个请求在经过一个服务器处理时，服务器会保存相关的会话信息，比如session，但是该请求如果第一个服务器没处理完，通过nginx轮询到第二个服务器上，那么这个服务器是没有会话信息的。

　　最典型的一个例子：用户第一次进入一个系统是需要进行登录身份验证的，首先将请求跳转到Tomcat1服务器进行处理，登录信息是保存在Tomcat1 上的，这时候需要进行别的操作，那么可能会将请求轮询到第二个Tomcat2上，那么由于Tomcat2 没有保存会话信息，会以为该用户没有登录，然后继续登录一次，如果有多个服务器，每次第一次访问都要进行登录，这显然是很影响用户体验的。

　　这里产生的一个问题也就是集群环境下的 session 共享，如何解决这个问题？

　　通常由两种方法：

　　1、第一种方法是选择一个中间件，将登录信息保存在一个中间件上，这个中间件可以为 Redis 这样的数据库。那么第一次登录，我们将session 信息保存在 Redis 中，跳转到第二个服务器时，我们可以先去Redis上查询是否有登录信息，如果有，就能直接进行登录之后的操作了，而不用进行重复登录。

　　2、第二种方法是根据客户端的IP地址划分，每次都将同一个 IP 地址发送的请求都分发到同一个 Tomcat 服务器，那么也不会存在 session 共享的问题。

　　而 nginx 的基于 IP 路由负载的机制就是上诉第二种形式。大概配置如下：

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     upstream OrdinaryPolling {
 2     ip_hash;
 3     server 127.0.0.1:8080 weight=5;
 4     server 127.0.0.1:8081 weight=2;
 5     }
 6     server {
 7         listen       80;
 8         server_name  localhost;
 9 
10         location / {
11             proxy_pass http://OrdinaryPolling;
12             index  index.html index.htm index.jsp;
13         
14         }
15     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　注意：我们在 upstream 指令块中增加了 ip_hash 指令。该指令就是告诉 nginx 服务器，同一个 IP 地址客户端发送的请求都将分发到同一个 Tomcat 服务器进行处理。



#### ④、基于服务器响应时间负载分配

　　根据服务器处理请求的时间来进行负载，处理请求越快，也就是响应时间越短的优先分配。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     upstream OrdinaryPolling {
 2     server 127.0.0.1:8080 weight=5;
 3     server 127.0.0.1:8081 weight=2;
 4     fair;
 5     }
 6     server {
 7         listen       80;
 8         server_name  localhost;
 9 
10         location / {
11             proxy_pass http://OrdinaryPolling;
12             index  index.html index.htm index.jsp;
13         
14         }
15     }
```

![复制代码](https://common.cnblogs.com/images/copycode.gif)

　　通过增加了 fair 指令。



#### ⑤、对不同域名实现负载均衡

 　通过配合location 指令块我们还可以实现对不同域名实现负载均衡。

![复制代码](https://common.cnblogs.com/images/copycode.gif)

```
 1     upstream wordbackend {
 2     server 127.0.0.1:8080;
 3     server 127.0.0.1:8081;
 4     }
 5 
 6     upstream pptbackend {
 7     server 127.0.0.1:8082;
 8     server 127.0.0.1:8083;
 9     }
10 
11     server {
12         listen       80;
13         server_name  localhost;
14 
15         location /word/ {
16             proxy_pass http://wordbackend;
17             index  index.html index.htm index.jsp;
18         
19         }
20     location /ppt/ {
21             proxy_pass http://pptbackend;
22             index  index.html index.htm index.jsp;
23         
24         }
25     }
```



# Nginx解决跨域问题

spring 里面服务端配置@CrossOrigin(value="*")，所有主机都允许访问

什么是跨域问题？

跨域，指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器对 JavaScript 施加的安全限制。

## 什么是同源？

所谓同源是指，域名，协议，端口均相同

http://www.funtl.com --> http://admin.funtl.com 跨域 http://www.funtl.com --> http://www.funtl.com 非跨域 http://www.funtl.com --> http://www.funtl.com:8080 跨域 http://www.funtl.com --> https://www.funtl.com 跨域

## 如何解决跨域？

### 使用 CORS（跨资源共享）解决跨域问题

CORS 是一个 W3C 标准，全称是"跨域资源共享"（Cross-origin resource sharing）。它允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 AJAX 只能同源使用的限制。

CORS 需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE 浏览器不能低于 IE10。整个 CORS 通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS 通信与同源的 AJAX 通信没有差别，代码完全一样。浏览器一旦发现 AJAX 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信（在 header 中设置：Access-Control-Allow-Origin）

### 使用 JSONP 解决跨域问题

JSONP（JSON with Padding）是 JSON 的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题。由于同源策略，一般来说位于 server1.example.com 的网页无法与 server2.example.com 的服务器沟通，而 HTML 的 <script> 元素是一个例外。利用 <script> 元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。用 JSONP 抓到的资料并不是 JSON，而是任意的 JavaScript，用 JavaScript 直译器执行而不是用 JSON 解析器解析（需要目标服务器配合一个 callback 函数）。

### CORS 与 JSONP 的比较

CORS 与 JSONP 的使用目的相同，但是比 JSONP 更强大。

JSONP 只支持 GET 请求，CORS 支持所有类型的 HTTP 请求。JSONP 的优势在于支持老式浏览器，以及可以向不支持 CORS 的网站请求数据。

### 使用 Nginx 反向代理解决跨域问题

以上跨域问题解决方案都需要服务器支持，当服务器无法设置 header 或提供 callback 时我们就可以采用 Nginx 反向代理的方式解决跨域问题。

以下为文件上传的跨域配置方案：

```conf
user  nginx;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen 80;
        server_name upload.myshop.com;
        add_header 'Access-Control-Allow-Origin'  '*';
        add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range';
        location / {
            proxy_pass  http://192.168.0.104:8888;
            if ($request_method = 'OPTIONS') {
                add_header Access-Control-Allow-Origin  *;
                add_header Access-Control-Allow-Headers X-Requested-With;
                add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,PATCH,OPTIONS;
                # 解决假请求问题，如果是简单请求则没有这个问题，但这里是上传文件，首次请求为 OPTIONS 方式，实际请求为 POST 方式
                # Provisional headers are shown.
                # Request header field Cache-Control is not allowed by Access-Control-Allow-Headers in preflight response.
                add_header Access-Control-Allow-Headers DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range;
                return 200;
            }
        }
    }
}
```



#Nginx配置Https

	## Http与Https的区别

​	HTTP：是互联网上应用最为广泛的一种网络协议，是一个客户端和服务器端请求和应答的标准（TCP），用于从WWW服务器传输超文本到本地浏览器的传输协议，它可以使浏览器更加高效，使网络传输减少。
​	HTTPS：是以安全为目标的HTTP通道，简单讲是HTTP的安全版，即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。HTTPS协议的主要作用可以分为两种：一种是建立一个信息安全通道，来保证数据传输的安全；另一种就是确认网站的真实性。

HTTPS和HTTP的区别主要如下：

​	1、https协议需要到ca申请证书，一般免费证书较少，因而需要一定费用。
​	2、http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。
​	3、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
​	4、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

## 使用openssl生成证书

​	openssl是目前最流行的SSL密码库工具，其提供了一个通用、健壮、功能完备的工具套件，用以支持SSL/TLS协议的实现。
​	比如生成到：/usr/local/ssl

```
openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /usr/local/ssl/nginx.key -out 	/usr/local/ssl/nginx.crt
```

生成过程：

```
# openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout /u    sr/local/ssl/nginx.key -out /usr/local/ssl/nginx.crt
Generating a 2048 bit RSA private key
...............................................................................+    ++
...............+++
writing new private key to '/usr/local/ssl/nginx.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:beijing
Locality Name (eg, city) [Default City]:beijing
Organization Name (eg, company) [Default Company Ltd]:xxxx
Organizational Unit Name (eg, section) []:xxxx
Common Name (eg, your name or your server's hostname) []:xxxx（一般是域名）
Email Address []:xxxx@xxxx.com
# ll
total 8
-rw-r--r--. 1 root root 1391 Apr 21 13:29 nginx.crt
-rw-r--r--. 1 root root 1704 Apr 21 13:29 nginx.key
```

## Nginx安装http_ssl_module模块

​	Nginx如果未开启SSL模块，配置Https时提示错误。

```
nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module in /usr/local/nginx/conf/nginx.conf:xxx
```

​	Nginx缺少http_ssl_module模块，编译安装的时候带上--with-http_ssl_module配置就行了。

​	本场景是服务器已经安装过nginx，但是未安装http_ssl_module。

1.进入到源码包，如：

```
cd /app/download/nginx-1.12.2
```

2.configure：

```
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

3.make：

```
make
```

4.不需要执行make install，否则就覆盖安装了。
5.备份原有的nginx，如：

```
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx_bak
```

6.然后将刚刚编译好的nginx覆盖掉原有的nginx（nginx需要停止）

```
cp ./objs/nginx /usr/local/nginx/sbin/
```

7.查看安装情况：

```
/usr/local/nginx/sbin/nginx -V
```

```
nginx version: nginx/1.12.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
```

四、nginx配置https
贴部分配置信息：

```
  server {
 
        listen	80;
		server_name www.yourdomain.com;
	    rewrite ^(.*) https://$server_name$1 permanent; #http 跳转 https
    }
server {
	listen 443 ssl;
	server_name www.yourdomain.com;
	ssl_certificate /usr/local/ssl/nginx.crt;
	ssl_certificate_key /usr/local/ssl/nginx.key;
	ssl_session_cache    shared:SSL:1m;
	ssl_session_timeout  5m;
	#禁止在header中出现服务器版本，防止黑客利用版本漏洞攻击
	server_tokens off;
	#如果是全站 HTTPS 并且不考虑 HTTP 的话，可以加入 HSTS 告诉你的浏览器本网站全站加密，并且强制用 HTTPS 访问
	fastcgi_param   HTTPS               on;
	fastcgi_param   HTTP_SCHEME         https;
	access_log /usr/local/nginx/logs/httpsaccess.log;
}
```

先检验配置的对不对：

     /usr/local/nginx/sbin/nginx -t
      1. nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
      2. nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful



重启nginx：

```
/usr/local/nginx/sbin/nginx -s reload
```





# Nginx 动静分离

**1.什么是动静分离?**

将动态请求和静态请求区分访问, 

**2.为什么要做动静分离?**

静态由Nginx处理, 动态由PHP处理或Tomcat处理.... 因为Tomcat程序本身是用来处理jsp代码的,但tomcat也能处理静态资源. tomcat本身处理静态效率不高,还会带来资源开销.

**3.如何实现动静分离?**

Nginx根据客户端请求的url来判断请求的是否是静态资源，如果请求的url包含jpg、png，则由Nginx处理。 如果请求的url是.php或者.jsp等等，这个时候这个请求是动态的，将转发给tomcat处理。 总结来说，Nginx是通过url来区分请求的类型，并转发给不同的服务端。

**4.单机实现动静分离实战**

​    [root@web01 ~]# yum install java tomcat -y     

​	 [root@web01 ~]# mkdir /usr/share/tomcat/webapps/ROOT        -->主要站点根目录     

​	 [root@web01 ~]# vi /usr/share/tomcat/webapps/ROOT/index.jsp    

```
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
    <html>
      <head>
        <title>Nginx+Tomcat</title>
      </head>
      <body>
          <%
            Random rand = new Random();
            out.println("<h2>动态资源</h2>");
            out.println(rand.nextInt(99)+100);
        %>
        <h2>静态图片</h2>
        <img src="nginx.png" />
      </body>
    </html>
```

[root@web01 ~]# wget -O /usr/share/tomcat/webapps/ROOT/nginx.png http://nginx.org/nginx.png [root@web01 ~]# systemctl start tomcat

tomcat监听在8080端口上:    

配置Nginx  [root@web01 conf.d]# cat ds.oldxu.com.conf  

```
server {
    listen 80;
    server_name ds.oldxu.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
    }
    location ~* \.(png|gif|jpg|mp4)$ {
        root /images;
        expires 1d;
    }
}
```

本地测试加demo项目名， /Users/houshaojie/nginx/下放的也是demo

```
location /demo/static/ {
            root  /Users/houshaojie/nginx/;
            autoindex on;
        }
```

**5.集群实现动静分离实战**

[root@lb01 conf.d]# cat  proxy_ds.oldxu.com.conf

```
upstream java {
    server 172.16.1.7:8080;
} 
upstream static {
    server 172.16.1.8:80;
}
server {
    listen 80;
    server_name ds.oldxu.com;
    location / {
        proxy_pass http://java;
        include proxy_params;
    }

    location ~* \.(png|gif|jpeg)$ {
        proxy_pass http://static;
        expires 2d;
        include proxy_params;
    }
}
```



#Nginx  expires缓存和gzip压缩配置

ngnix通过配置，告知浏览器，返回数据的有效时间，浏览器根据数据的有效时间，确认是否到服务器请求。如果数据没有超过有效期，就使用浏览器缓存的数据。

```
server{
    location ~ \.(jpeg|jpg|png|css)${
    #缓存一天
    expires  1d;
   }
}

配置gzip压缩		gzip on

http的协议版本		gzip_http_version 1.0

如果是IE的话，就关闭压缩	gzip_disable 'MSIE[1-6]'

需要压缩的文件格式		gzip_types image/jpg
```



# Nginx基本配置和日志

Nginx的主配置文件是nginx.conf，主要组成如下：

```
#全局区   有一个工作子进程，一般设置为CPU数*核数
worker_processes  1;

events {
          #一般配置nginx进程与连接的特性    如一个word能同时允许多少连接，一个子进程最大允许连接1024个连接 总数=cpu*1024
          worker_connections  1024;
}
keepalive_timeout 65;  #超过65秒说明服务器不可用
#配置HTTP服务器配置段
http {
            #配置虚拟主机段
    server 127.0.0.1:38181;
            server{
                 #监听端口  80
                 listen  80；
                 #域名解析    localhost
                 server_name  test.sergei.com;
                 #配置默认访问页
                 location / {
                      #代理转发时，如果在proxy_pass后面的url加/，表示绝对根路径；如果没有/，表示相对路径，把匹配的路径部分也给代理走。https://blog.csdn.net/zhongzh86/article/details/70173174
proxy_pass http://oms;
proxy_set_header Host admin.xiaodaibang.com;
proxy_set_head X-Forwarded-For $proxy_add_x_forwarded_for;
#网站根目录
                      #root   /Users/houshaojie/nginx/web;  #相对路径，相对nginx根目录，也可写成绝对路径
#alias   static/;请求到这个下面去寻找
                       index  index.html;     #默认跳转到index.html页面
                }
         }
}
```

## Nginx日志格式

打开nginx.conf配置文件：

```
vim /usr/local/nginx/conf/nginx.conf
```


日志部分内容：

```
access_log  logs/access.log  main;
```

日志生成的到Nginx根目录logs/access.log文件，默认使用“main”日志格式，也可以自定义格式。

默认“main”日志格式：

![image-20200422165505986](/Users/houshaojie/Library/Application Support/typora-user-images/image-20200422165505986.png)

 参数明细表：

```
$remote_addr 客户端的ip地址(代理服务器，显示代理服务ip)
$remote_user 用于记录远程客户端的用户名称（一般为“-”）
$time_local 用于记录访问时间和时区
$request 用于记录请求的url以及请求方法
$status 响应状态码，例如：200成功、404页面找不到等。
$body_bytes_sent 给客户端发送的文件主体内容字节数
$http_user_agent 用户所使用的代理（一般为浏览器）
$http_x_forwarded_for 可以记录客户端IP，通过代理服务器来记录客户端的ip地址
$http_referer 可以记录用户是从哪个链接访问过来的
```

我这里需要在/usr/local/Celler/nginx 下手动创建logs文件夹
查看日志命令tail -f /usr/local/Celler/nginx/logs/access.log

自定义某一个server配置的日志，使用“main”日志格式。

日志生成的到Nginx根目录logs/access.log文件，默认使用“main”日志格式，也可以自定义格式。
执行命令：nginx-s reload
执行命令：tail -100f /usr/local/Celler/nginx/logs/abc.access.log

## Nginx日志分隔

nginx的日志文件没有rotate功能。编写每天生成一个日志，我们可以写一个nginx日志切割脚本来自动切割日志文件。
第一步就是重命名日志文件，不用担心重命名后nginx找不到日志文件而丢失日志。在你未重新打开原名字的日志文件前，nginx还是会向你重命名的文件写日志，Linux是靠文件描述符而不是文件名定位文件。
第二步向nginx主进程发送USR1信号。nginx主进程接到信号后会从配置文件中读取日志文件名称，重新打开日志文件(以配置文件中的日志名称命名)，并以工作进程的用户作为日志文件的所有者。重新打开日志文件后，nginx主进程会关闭重名的日志文件并通知工作进程使用新打开的日志文件。工作进程立刻打开新的日志文件并关闭重名名的日志文件。然后你就可以处理旧的日志文件了。[或者重启nginx服务]。

nginx日志按每分钟自动切割脚本如下：
新建shell脚本：vi/usr/local/software/nginx/nginx_log.sh

        #!/bin/bash
        #设置日志文件存放目录
        LOG_HOME="/usr/local/software/nginx/logs/"
    
        #备分文件名称
        LOG_PATH_BAK="$(date -d yesterday +%Y%m%d%H%M)".abc.access.log
    
        #重命名日志文件
        mv ${LOG_HOME}/abc.access.log ${LOG_HOME}/${LOG_PATH_BAK}.log
    
        #向nginx主进程发信号重新打开日志 
        kill -USR1 `cat /usr/local/software/nginx/logs/nginx.pid`

创建crontab设置作业

设置日志文件存放目录crontab -e

```
*/1 * * * *  sh /usr/local/software/nginx/nginx_log.sh
```

















