## NO文件学习——文件读写篇

及上一篇章之后《 <a href="">传统IO系统学习</a> 》系统的学习Java的IO之后，现在开始学习进阶版的IO——NIO。为啥要学

习NIO呢？因为NIO相对传统的IO，其在 **文件操作** 和 **网络传送** 中有着重大的优势，尤其是网络传输方面。其实

传统的IO已经被重做过了，效率提高了不少，因此此篇章主要学习NIO的文件操作。

### 一、NIO快速入门

**1.1、传统IO与NIO的区别**

| IO类型 | 操作类型             | 是否阻塞 | 操作方向 | 辅助     |
| ------ | -------------------- | -------- | -------- | -------- |
| 传统IO | 面向流（Stream）     | 是       | 单向     | 无       |
|        | 面向缓冲区（buffer） | 否       | 双向     | selector |

传统IO是面向流的操作，一次一个字节的处理数据，读写只能单向操作。NIO是面向缓冲区的操作，是以缓冲区

（块）的形式处理数据，读写是双向操作的。单向：是指执行的IO操作只能是单方面的读取或者写入；双向指的

是双方可以向对方进行读取或者写入数据的操作。



**1.2、NIO操作文件的核心部分**

- **1.2.0、直接缓冲区与非直接缓冲区**

  linux为了保证内核的安全，不允许用户进程不能直接操作内核，于是将操作系统将虚拟空间分为两部分：**内**

  **核控件和用户空间**。因为虚拟空间有两部分，所以缓冲区也相应的被分成两种：**直接缓冲区和非直接缓冲区**。

  **直接缓冲区**：是指不经过用户地址空间，IO操作直接使用操作系统为物理磁盘上的文件创建的物理内存映射文

  件。**非直接缓冲区**：IO操作需要经过用户的地址和内核的地址空间的缓冲区。应用程序的IO读取操作先访问用

  户地址空间上的缓冲区，如数据不存在就告诉操作系统，系统就去内核地址空间上查询，查询找到，拷贝一份

  到用户地址空间上，查询不到就直接去物理磁盘上查询，然后读取到内核地址空间，接下来就复制一份到用户

  地址空间上。用户程序的写操作是先将数据写到用户地址空间，然后拷贝一份到内核的地址空间上，操作系统

  在写到物理磁盘上。

  ![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib00xmyDOO5dgS04fnMtGicQxmCa5niaSOBZU01yZSTxP7RLTXuNibXqKyqQGLiamvibUyYWq53NyiaCtc7g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  ![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib00xmyDOO5dgS04fnMtGicQxXd7mogVzFEJGbXFL65P8m8dhtfG0VVvQ52VKVkQth9Nw2VwicF2o3pA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  ​                  **图片来源参考资料**

  - **非直接字节缓冲区**：非直接字节缓冲区，则java虚拟机会尽最大努力在此缓冲区上执行本机I/O操作。也

    就是说，在每次调用基础操作系统的一个本机I/O操作之前或者之后，虚拟机都会尽量避免将缓冲区的内

    容复制到中间缓冲区上。（或从中间的缓冲区赋值内容）

  - **直接字节缓冲区**：直接字节缓冲区可以通过该类的allocateDirect()工厂方法来创建。此方法返回的胡冲去

    进行分配和取消分配的所需成本都高于非直接缓冲区。直接缓冲区的内容可以驻留在常规的垃圾回收堆之

    外，因此直接字节缓冲区的内容对应用程序的内存需求量造成的影响不大。所以，建议将直接缓冲区主要

    分配给那些易受系统基础的本机I/O操作影响的大型，持久的缓冲区。一般情况下，最好尽在直接缓冲区

    能在程序性能方面带来明显的好处时分配。

  - **直接缓冲区**：直接字节缓冲区还可以**FileChannel的map方法**将文件区域直接映射到内存中创建。该方法

    返回MappedByteBuffer。Java平台的实现有助于通过JNI从本机代码直接字节缓冲区。如果以上这些缓冲

    区中的某个缓冲区实例指的是不可访问的内存区域，则尝图访问该缓冲区不能更改的的内容时，则会抛出

    异常。

  - 字节缓冲区是直接缓冲区还是非直接缓冲区是可以通过调用isDirect方法确定。提供该方法是为了能够在

    性能关键性代码中执行显示缓冲区管理。

    

- **1.2.1、buffer和channel介绍**

  相对于传统IO，NIO是基于缓冲区操作的，其文件操作的核心是：buffer（缓冲区）和 channel（管道）。

  **buffer（缓冲区）是用来存放数据的，而channel（管道）的作用是运输缓冲区的数据**。你可以这样理解：

  channel是高铁轨道，而缓冲区是列车，装载数据的列车需要在在高铁轨道上运输，在1.1中说的双向，可以

  这样理解，列车在管道上既可以有上海开往广西，也能由广西开往上海。值得注意的是channel是不与数据打

  交道的，只有buffer才和数据打交道。

  

- **1.2.2、buffer（缓冲区）类介绍**

  Buffer是一个抽象类，常用的缓冲区类都是继承与该类。其内部维护了几个核心成员。

  ```java
  public abstract class Buffer {    
   	private int position = 0; 
      private int limit;	
      private int capacity;
      private int mark=-1; 
      //....
  }
  ```

  | 成员     | 描述                                                         |
  | -------- | ------------------------------------------------------------ |
  | position | 下一个要被读或写的元素的位置，由put方法和get方法自动更新。   |
  | limit    | 缓冲区的总数，初始值为capacity，使用 filp()方法后,变为数组有效元素的总个数，limit之后的数据不能读取。 |
  | capacity | 缓冲区的容量，初始值指定。一旦指定就不能修改。（底层是数据结构是final修饰的数组） |
  | mark     | 记录上一读写的位置，就是position的位置。（数组的下标）       |

  常用的Buffer的子类

  | 类名         | 描述                             |
  | ------------ | -------------------------------- |
  | ByteBuffer   | 提供Byte类型的缓冲区的相关操作   |
  | CharBuffer   | 提供Char类型的缓冲区的相关操作   |
  | ShortBuffer  | 提供Short类型的缓冲区的相关操作  |
  | IntBuffer    | 提供Int类型的缓冲区的相关操作    |
  | FloatBuffer  | 提供Float类型的缓冲区的相关操作  |
  | DoubleBuffer | 提供Double类型的缓冲区的相关操作 |
  | LongBuffer   | 提供Long类型的缓冲区的相关操作   |

  缓冲区顾名思义就是存放/获取数据的，所以缓冲区的常用操作就是put方法——想缓冲区写入数据和get方法从缓冲区读取数据了。代码演示：

  ```java
  public static void main(String args[]){
      ByteBuffer buffer =ByteBuffer.allocate(1024*2);//2M的缓冲区，使用的是直接缓冲区
      System.out.println();
      System.out.println("初始时-->limit--->"+buffer.limit());
      System.out.println("初始时-->position--->"+buffer.position());
      System.out.println("初始时-->capacity--->"+buffer.capacity());
      System.out.println("初始时-->mark--->" + buffer.mark());
  
      String s="this is a test";
      buffer.put(s.getBytes()); //想缓冲区写入数据
  
      System.out.println("put完之后-->limit--->"+buffer.limit());
      System.out.println("put完之后-->position--->"+buffer.position());
      System.out.println("put完之后-->capacity--->"+buffer.capacity());
      System.out.println("put完之后-->mark--->" + buffer.mark());
  
      buffer.flip();//切换成读取数据模式
      System.out.println("flip完之后-->limit--->"+buffer.limit());
      System.out.println("flip完之后-->position--->"+buffer.position());
      System.out.println("flip完之后-->capacity--->"+buffer.capacity());
      System.out.println("flip完之后-->mark--->" + buffer.mark());
  }
  /*输出结果
  初始时-->limit--->2048
  初始时-->position--->0
  初始时-->capacity--->2048
  初始时-->mark--->java.nio.HeapByteBuffer[pos=0 lim=2048 cap=2048]
  
  put完之后-->limit--->2048
  put完之后-->position--->14
  put完之后-->capacity--->2048
  put完之后-->mark--->java.nio.HeapByteBuffer[pos=14 lim=2048 cap=2048]
  
  flip完之后-->limit--->14
  flip完之后-->position--->0
  flip完之后-->capacity--->2048
  flip完之后-->mark--->java.nio.HeapByteBuffer[pos=0 lim=14 cap=2048]
  */
  ```

  有输出结果可以看出：写入数据后，分配缓冲区后，limit=capacity=2048，position=0；使用put方法写入数据后，position变为14，其他不变。切换成读之后，position变为0，也就是数据开始读取的位置，limit为实际数据的大小。切换完之后，皆可以开始读取数据了。

  ```java
  public static void main(String args[]){
      ByteBuffer buffer =ByteBuffer.allocate(1024*2);//2M的缓冲区
      System.out.println();
  
      String s="this is a test";
      buffer.put(s.getBytes()); //向缓冲区写入数据
      buffer.flip(); //切换成读模式
      byte[] bytes=new byte[12];
      buffer.get(bytes);
      System.out.println(new String(bytes,0,bytes.length)); //this is a test
  
      buffer.clear(); //重新写入数据时，想要"清空"缓冲区的数据
      String s1="second test";
      buffer.put(s1.getBytes());
  }
  ```

  在向buffer重新写入数据前，需要调用clear方法，“清空”缓冲区的数据（其实没用清除，只是将position重置为0，limit重置为capacity的大小），然后向缓冲区中写入数据。

**1.2.3、Channel类介绍**

Channel（管道）：用于源数据节点与目标结点的连接。在Java的NIO中负责缓冲区的数据传输。管道不负责与数

据打交道，因此需要配合缓冲区使用。

- 常用的管道类

  FileChanne：用于文件操作

  SocketChannel：用于TCP协议网络通信中客户端的数据运输

  SocketServerChannel：用于TCP协议网络通信中服务端的数据运输

  DataGramChannel：用于UDP协议网络通信的数据运输

文件的NIO代码演示（网络的留到网络篇介绍）：

- channel的获取

```java
public void getChannel(String filePath) throws IOException {
    FileInputStream fis = new FileInputStream(filePath);
    FileChannel fisChannel = fis.getChannel(); //管道

    FileChannel channel = FileChannel.open(Paths.get(filePath));
}
```

- 文件拷贝

```java
public void copyFile(String originFile,String targetFile) throws IOException {
    FileInputStream fis = new FileInputStream(originFile); //读取源文件
    targetFile=targetFile+"\\00.jpg";
    FileOutputStream fos = new FileOutputStream(targetFile);

    FileChannel fisChannel = fis.getChannel(); //获取读取运输文件管道
    FileChannel fosChannel = fos.getChannel(); //获取写入文件管道
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    while (fisChannel.read(buffer)!=-1){
        buffer.flip();//切换成读取模式
        fosChannel.write(buffer); //写入数据
        buffer.clear(); //清空缓冲区
    }
}
public static void main(String args[]) throws IOException {
    new Test().copyFile("C:\\Users\\99447\\Pictures\\Camera Roll\\1.jpg",
                        "C:\\Users\\99447\\Pictures\\Camera Roll");
}
```

- 使用内部才能映射文件的方式文件拷贝

```java
public void copyFile(String originFile,String targetFile) throws IOException {
    //以读取模式打开文件，并创建管道
    FileChannel fcIn = FileChannel.open(Paths.get(originFile), StandardOpenOption.READ);
    //以创建文件和写操作打开管道，如果文件存不存在，则创建文件，反之替代文件
    FileChannel fcOut = FileChannel.open(Paths.get(targetFile + "00.jpg"), 
                                         StandardOpenOption.CREATE_NEW, 
                                         StandardOpenOption.WRITE);
    //创建内存内存映射文件
    MappedByteBuffer inMapBuf = fcIn.map(FileChannel.MapMode.READ_ONLY, 0, fcIn.size());
    MappedByteBuffer outMapBuf = fcOut.map(FileChannel.MapMode.READ_WRITE, 
                                           0, fcOut.size());

    byte[] bytes=new byte[inMapBuf.limit()];
    //拷贝文件
    inMapBuf.get(bytes);
    outMapBuf.put(bytes);
}

public static void main(String args[]) throws IOException {
    new Test().copyFile("C:\\Users\\99447\\Pictures\\Camera Roll\\1.jpg",
                        "C:\\Users\\99447\\Pictures\\Camera Roll");
}
```

- 使用transferTo方法拷贝文件

```java
public void copyFile(String originFile,String targetFile) throws IOException {
    //以读取模式打开文件，并创建管道
    FileChannel fcIn = FileChannel.open(Paths.get(originFile), StandardOpenOption.READ);
    //以创建文件和写操作打开管道，如果文件存不存在，则创建文件，反之替代文件
    FileChannel fcOut = FileChannel.open(Paths.get(targetFile + "00.jpg"),
                                         StandardOpenOption.CREATE_NEW, 
                                         StandardOpenOption.WRITE);
    fcIn.transferTo(0,fcIn.size(),fcOut);
    fcIn.close();
    fcOut.close();
}
public static void main(String args[]) throws IOException {
    new Test().copyFile("C:\\Users\\99447\\Pictures\\Camera Roll\\1.jpg",
                        "C:\\Users\\99447\\Pictures\\Camera Roll");
}
```



**1.3、scatter和gather、字符集**

- scatter（分散读取）：将一个管道运输的数据分散读取到多个缓冲区中
- gather（聚集写入）；分散在多个缓冲区的内容汇聚写入到一个通道中。

代码演示：

```java
public ByteBuffer[] read(String filePath) throws IOException {
    FileChannel channel = FileChannel.open(Paths.get(filePath),
                                           StandardOpenOption.READ);
    ByteBuffer buf1=ByteBuffer.allocate(1024);
    ByteBuffer buf2=ByteBuffer.allocate(1024);
    ByteBuffer[] bufs={buf1,buf2};
    channel.read(bufs); //分散读取数据
    for (ByteBuffer buf:bufs){
        buf.flip(); //切换成读模式
    }
    return bufs;
}

public void write(ByteBuffer[] bufs,String targetFile) throws IOException {
    FileChannel channel = FileChannel.open(Paths.get(targetFile + "\\00.jpg"),
                                           StandardOpenOption.CREATE_NEW, 
                                           StandardOpenOption.WRITE);
    channel.write(bufs);
}

```

- 字符集：就是写入和读取的编码是要一致的

```java
public static void main(String args[]) throws CharacterCodingException {
    Charset gbkCs = Charset.forName("GBK");
    //获取格式编码器
    CharsetEncoder gbkCe = gbkCs.newEncoder();
    //获取合适解码器
    CharsetDecoder gbkCd = gbkCs.newDecoder();

    CharBuffer buffer = CharBuffer.allocate(1024);
    //存放数据
    buffer.put("this is a test");
    //切换读模式,加密读取
    buffer.flip();
    ByteBuffer byteBuffer = gbkCe.encode(buffer);
    for (int i=0;i<byteBuffer.limit();i++){
        System.out.print(byteBuffer.get()+"\t");
    }
    System.out.println();
    //切换读模式
    byteBuffer.flip();
    //解密读取缓冲区的数据
    CharBuffer charBuffer = gbkCd.decode(byteBuffer);
    for (int j=0;j<charBuffer.limit();j++){
        System.out.print(charBuffer.get()+"\t");
    }
}
/** 输出结果
116	104	105	115	32	105	115	32	97	32	116	101	115	116	
t	h	i	s	 	i	s	 	a	 	t	e	s	t	
*/
```





参考资料：

①：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484235&idx=1&sn=4c3b6d13335245d4de1864672ea96256&chksm=ebd7424adca0cb5cb26eb51bca6542ab816388cf245d071b74891dd3f598ccd825f8611ca20c&scene=21###wechat_redirect">JDK10都发布了，nio你了解多少？</a>

