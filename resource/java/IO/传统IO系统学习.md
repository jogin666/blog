## 传统IO学习

写在前面：文章转载于：<a href="https://zhuanlan.zhihu.com/p/28286559">Java IO，硬骨头也能变软</a>（有修改）

IO作为Java的基础模块中的一大模块，学习是很有必要的。稍微浏览了网上的文章，发现还蛮多优秀的文章。接下

跟随知乎作者：小明 的优秀文章 《<a href="https://zhuanlan.zhihu.com/p/28286559">Java IO，硬骨头也能变软</a>》一起学习IO吧！！

### 一、前言

先看一张网上流传的 [java.io](https://link.zhihu.com/?target=http%3A//java.io) 包的类结构图：

![preview](https://pic2.zhimg.com/v2-c4759b2779487e7fe647973776a09a9d_r.jpg)

当你看到这幅图的时候，我相信，你跟我一样内心是崩溃的。

有些人不怕枯燥，不怕寂寞，硬着头皮看源码，但是，能坚持下去全部看完的又有几个呢！

然而，就算源码全部看完看懂，过不了几天，脑子里也会变成一团浆糊。

因为这里的类实在太多了。可能我们反复看，反复记，也很难做到清晰明白。

他就像是一块超级硬的骨头，怎么啃都啃不烂。

面对这样的做法，要坚决对他说，**NO**。



### 二、分类

我的做法是找出他们的共性，给他们分类，只记典型，触类旁通。

上面的图虽然有分类，但是还不够细，而且没有总结出方便记忆的规律，所以我们要重新整理和归类。

这篇文章中，使用了两种分时给他们分组，目的是更全面的了解共性，帮助记忆。

**2.1、分类一：按操作方式（类结构）分类**

* 字节流和字符流
  * 字节流（类名有Stream的）：以字节为单位，每次读入读出是**8位数据**，可以读取**任何数据类型**。
  * 字符流（类名有Reader/Writer的）：以字符为单位，每次读入读出是**16位数据**，只能读取**字符类型**。
* 输出流和输入流
  * 输出流：从内存中读取数据到文件，只能进行写操作。
  * 输入流：从文件中读取数据到内存，只能进行读操作。 

>注意：这里的出和入，都是相对于系统内存而言的。

* 节点流和处理流
  * 节点流：直接与数据源相连，读入读取操作。
  * 处理流：与节点流一块使用，在节点流的基础上，再套接一层，套接在节点流上的就是处理流。（包装模式，读节点流进行包装）

> 为什么需要处理流呢？直接是有节点流，读写不方便（直连数据源，IO是耗时操作），为了更快的读写文件，所以对采用包装模式，对节点流包装。



**按操作方式分类结构图**

根据以上分类，以及jdk的说明，我们可以画出更详细的类结构图，如下：

![IO图1](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/IO%E5%9B%BE1.png)

> 值得注意：处理流是对结点流的包装。

  **分类说明**

1. 读取字节流InputStream：

   输入字节流的类结构图可见上图，可以看出：

   FileInputStream：是三种基本介质流，分别从Byte数组，StringBuffer和本地文件中读取数据。

   ByteArrayInputStream：用于临时内存操作，以字节数组作为容器，作为临时存放从内存中获取的数据。

   PipeInputStream：用于从线程公用的管道中读取数据。需要搭配PipeOutputStream使用，共同完成管道的读取写入操作。

   DataInputStream：用于从文件中读取基本类型的数据值。

   未列出的ObjectInputStream ：从本地文件中读取引用类型的数据，和FilterInputStream 的子类都是装饰流（装饰器模式的主角）

2. 输出字节流OutputStream：

   输出字节流的类结构图可见上图，可以看出：

   FileOutputStream：是两种基本的介质流，向本地文件写入数据

   ByteArrayOutputStream：用于临时内存操作，以字节数组作为容器，作为临时存放要写入本地文件的数据

   PipedOutputStream：用于搭配PipeInputStream想线程公用的管道中读取/写入数据。

   DataOutputStream：用于向本地文件中写入基本类型的数据值。

   ObjectOutputStream：向本地的文件写入引用类型所拥有的数据值。

> 字节流的输入和输入对照图

![img](https://pic2.zhimg.com/80/v2-a6820c3095e62e25d3d56dd033225fd1_hd.png)

3. 字符输入流Reader：

   在上面的类结构图中可以看出：

   FileReader：文件读取

   PipeReader：从线程共享的管道中读取数据

   CharArrayReader：以数组作为容器从内存中读取字节数据

   CharReader、StringReader 是两种基本的介质流，它们分别将Char 数组、String中读取数据。

   BufferedReader ：很明显就是一个装饰器，它和其子类负责装饰其它Reader 对象。

   InputStreamReader： 是字节流和字符流的桥梁，它将字节流转变为字符流。

4. 字符输出流Writer：

   在上面的类结构图中可以看出：

   FileWriter：将数据写入到本地文件中

   PipeWriter：想线程共享的管道总写入数据

   CharArrayReader：以数组作为容器，存放待写入文件的字节数据

   CharArrayWriter、StringWriter 是两种基本的介质流，它们分别向Char 数组、String 中写入数据。

   BufferWriter：很明显就是一个装饰器，它和其子类负责装饰其它Writer对象。

   OutputStreamWriter： 是字节流和字符流的桥梁，它将字节流转变为字符流。

> 字符输出流和输入流的对照图

![img](https://pic1.zhimg.com/80/v2-1d5951f34695a0a02535824bde228af8_hd.png)

5. 字节流和字符流转换

   - 特点

   可对读取到的字节数据经过指定编码转换成字符；

   可对读取到的字符数据经过指定编码转换成字节；   

   - 何时转换流

   当字节和字符之间有转换动作时；

   流操作的数据需要编码或解码时。

6. 转换代码演示

    读取流（节点流）转为读流（处理流）

```java
void example(@NotNull String filePath) throws IOException {
    File file = new File(filePath); //创建文件
    FileInputStream fis = new FileInputStream(file); //节点流
    InputStreamReader isr=new InputStreamReader(fis); //搭建桥梁
    Reader reader=new BufferedReader(isr); //处理流
    //流要关闭
    fis.close();
    isr.close();
    reader.close();
}
```

写入流（结点流）转为写入（处理流），和上面类似，就不再此演示了。



**2.2、分类二：按照操作对象**

按照操作对象分类结构图：

   ![img](https://pic3.zhimg.com/80/v2-1a7a2ae7ed9a13910aecebbed9a00e72_hd.png)



**分类说明**

* 对文件进行操作（节点流）：

  FileInputStream（字节输入流）

  FileOutputStream（字节输出流）

  FileReader（字符输入流）

  FileWriter（字符输出流）

* 对管道进行操作（节点流）：

  PipeInputStream

  PipeOutputStream

  PipeReader

  PipeWriter

* 字节/字符数组流（节点流）：

  ByteArrayInputStream

  ByteArrayOutputStream

  CharArrayReader

  CharArrayWriter

  是在内存中开辟了一个字节或字符数组。

* Buffered缓冲流（处理流）

  BufferedInputStream，

  BufferedOutputStream，

  BufferedReader,

  BufferedWriter,
  是带缓冲区的处理流，缓冲区的作用的主要目的是：避免每次和硬盘打交道，提高数据访问的效率。

* 转化为流（处理流）

  InputStreamReader：把字节流转化成字符流；

  OutputStreamWriter：把字节流转化成字符流。

  为字节流和字符流的转换搭建桥梁

* 基本类型数据流（处理流）：用于读取/写入基本数据类型的数值

  DataInputStream

  DataOutputStream

  处理基本基本类型的读取和写入

* 打印流

  PrintStream，

  PrintWriter，
  一般是打印到控制台，可以进行控制打印的地方。

* 对象流（处理流）：

* ObjectInputStream，对象反序列化；

* ObjectOutputStream，对象序列化；
  把封装的对象直接输出，而不是一个个在转换成字符串再输出。

* 合并流（处理流）：

* SequenceInputStream：可以认为是一个工具类，将两个或者多个输入流当成一个输入流依次读取。

### 三、其他类：RandomAccessFile

该对象并不是流体系中的一员，其封装了字节流，同时还封装了一个缓冲区（字符数组），通过内部的指针来操作字符数组中的数据。 该对象特点：

1. 该对象只能操作文件，所以构造函数接收两种类型的参数：a.字符串文件路径；b.File对象。
2. 该对象既可以对文件进行读操作，也能进行写操作，在进行对象实例化时可指定操作模

>  **注意：**
> 该对象在实例化时，如果要操作的文件不存在，会自动创建；如果文件存在，写数据未指定位置，会从头开始写，即覆盖原有的内容。 可以用于多线程下载或多个线程同时写数据到文件



> 注：如果向单个深入了解的话，传送门：<a href="https://blog.csdn.net/panweiwei1994/article/details/78046000">Java8 I/O源码-目录</a>



