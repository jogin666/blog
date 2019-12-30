## NIO学习——入门篇

### 一、开篇介绍

**1.1、引言**

及上一篇章之后《 <a href="https://github.com/jogin666/blog/blob/master/resource/java/IO/NIO%E5%AD%A6%E4%B9%A0%E7%AF%87%EF%BC%88%E4%B8%80%EF%BC%89%E6%96%87%E4%BB%B6%E6%93%8D%E4%BD%9C.md">NIO学习篇（一）文件操作</a> 》已经大致了解了NIO是如何进行文件读取和写入操作，以及大致

的实现原理。此篇将开启NIO的网络通信的学习篇章。

**1.2、理清BIO，NIO，AIO概念**

在正式学习NIO之前，需要理清BIO，NIO，AIO这三个概念。对于这三个概念，网上有一大堆文章介绍，有些

讲的是蛮好的，有些就讲的令人难受了。因此还是很有必要将一下的。

* **BIO（同步阻塞I/O）**

  数据的读取或写入必须是在同一个线程中完整操作。一个线程在执行I/O操作时，必须要等待操作系统完成就

  绪工作后，才能执行，再此期间线程都是阻塞状态。举个例子：银行ATM机排队领钱，排队领钱的人，必须

  要等之前的人全部领完钱之后（准备就绪工作），才能执行，在此之间，无法离开去做其他事情。

* **NIO（同步非阻塞I/O）**

  同时支持同步阻塞和同步非阻塞的两种模式，再此介绍同步非阻塞模式：程序执行发起I/O操作时，操作系统

  提供函数，返回状态，应有程序不断轮询访问状态标识，查看操作系统是否准备就绪。举个例子：银行柜台排

  队领钱，在银行的机器上获取到排队编号之后，发现前面还有很多人，可以先去做其他事情，不必一直等待，

  **只不过中间需要不时返回查看是否轮到自己**。

* **AIO（异步非阻塞I/O）**

  数据的读取和操作不再同一线程中执行。应用程序发起IO读取操作后，系统立即返回一个状态，标识读取请求

  成功。然后应用程序就可以去做其他事情了。待操作系统准备就绪后，会执行回调函数告诉应用程序已经准备

  好了。举个例子：银行柜台排队领钱，在银行的机器上，输入手机号码获取到排队编号之后就可以做其他的事

  情，不必等待，准备轮到你的时候，银行机器系统会自动发送一条短信告诉你——快轮到你了，快回来。

**1.3、同步和异步的区别**

同步和异步的区别是针对操作系统内核和应用程序而言的。同步I/O：是指用户程序发出I/O操作之后，线程不断等

待或者轮询去查看I/O操作是否就绪，而异步I/O：是指用户程序发出I/O操作之后，不必等待或者轮询去查看，当

I/O操作完成了之后，系统会自动告知用户程序I/O操作完成了。

**1.4、阻塞和非阻塞的区别**

阻塞和非阻塞的是针对进程访问数据而言的，根据I/O操作的状态来采用不同的方式。简单的说，就是I/O操作的读

取或者写入操作的实现方式，在阻塞方式下，执行读取或者写入操作，线程会一直等待，直到完成操作或者抛出异

常之后，返回状态值（**BIO**）。而非阻塞方式下，执行到读取操作或者写入操作时，会立即返回一个状态值，执行

后或者出现异常又会返回以一个状态值（**NIO**）。

**1.5、代码演示**

1. BIO代码演示

   在网络的初期，网民很少，服务器完全无压力，那是一个属于BIO大展身手的时代，采用Socket实现网络通信

   ，一个线程处理一个请求，也就是一个请求到来之后，服务器创建一个线程全程跟踪请求，完成客户端的请求

   ```java
   class BIOServer{
       public static void main(String args[]) throws IOException {
           final int port=8080;
           ServerSocket serverSocket = new ServerSocket(); //创建服务器会话
           serverSocket.bind(new InetSocketAddress("localhost", port));//绑定端口
           while (true){ //等待请求
               Socket socket=serverSocket.accept(); //接受客户端请求
               //创建一个线程处理客户端的一个请求，无论请求是否是要求处理事情
               new Thread(()->{/*....*/}) 
           }
       }
   }
   
   class client{
       public static void main(String args[]) throws IOException {
           Socket socket = new Socket();//
           socket.connect(new InetSocketAddress("localhost",8080)); //与服务器简历连接
           //......发送请求的内容
       }
   }
   ```

2. NIO代码演示

   上面在介绍NIO的时候，只介绍了NIO是使用单个线程轮询访问请求的状态。其实NIO还有另一个优势，那就

   是IO多路复用。IO复用的关键点就是在客户端与服务器成功连接后，服务器不是立马创建线程跟踪处理客户端

   的请求。而是先将 **连接** 注册到选择器（**selector**）上，同时创建一个轮询访问请求状态是否发生变化的线

   程。当轮序线程发生注册器上请求连接的状态有发生改变时（连接上有请求数据），才会创建一个线程来处理

   请求，也是有效请求才会创建线程处理，没有数据的请求是不会创建线程处理的。

   ```java
   class NioServer{
   
       final int port=8080;
       final int timeout=5000;
       public void nioAccept() throws IOException {
           Selector selector=Selector.open(); //选择器
           //信道
           ServerSocketChannel channel = ServerSocketChannel.open();
           //信道绑定端口
           channel.bind(new InetSocketAddress(port));
           //设置信道为非阻塞模式
           channel.configureBlocking(false);
           //将选择绑定到信道,并设置为接受
           channel.register(selector,SelectionKey.OP_ACCEPT);
           
           while(true){ //不断的轮询访问
               
               Set<SelectionKey> keys = selector.selectedKeys(); //获取所有的连接
               Iterator<SelectionKey> iterator = keys.iterator(); 
               while (iterator.hasNext()){
                   SelectionKey key = iterator.next();
                   if (key.isValid()){ //连接仍有校
                       //....
                   }else if (key.isAcceptable()){ //接受
                       //....
                   } else if (key.isReadable()){ //读状态
                       //....
                   }else if (key.isWritable()){ //写状态
                       //....
                   }
                   iterator.remove();//已经处理的事件，就扔掉
               }
           }
       }
   }
   ```

   总结一下：NIO的大致处理过程（网络通信方面）：

   1. 定义一个选择器，selector（轮询访问连接的状态，每一个连接的状态是否发生变化）
   2. 定义一个服务器端套接字信道，ServerSocketChannel，并配置为非阻塞的。
   3. 将信道绑到指定的端口号
   4. 将信道注册到选择器上，并把感兴趣的操作设置为OP_ACCEPT。
   5. 不断轮询访问注册到选择器的**连接**的状态是否发生变化
   6. 连接状态发生变化，创建线程跟踪处理。

3. AIO代码演示

   AIO真正的异步，读写当进行读写操作时，只要直接使用API提供的read和write异步方法即可，读写操作完成

   后，会自动调用回调函数，告知应用程序。对于读操作而言，操作系统会将可读的流传入read方法的缓冲区，

   并通知应用程序。而对于写操作，操作系统将写入流的数据写入完毕时，操作系统也会自动通知应用程序。

   ```java
   class AioExample{
   
       public void example() throws IOException {
           //获取异步信道
           AsynchronousServerSocketChannel assc=AsynchronousServerSocketChannel.open();
           //非阻塞方法，注册回调函数，只能接受一个连接
           assc.accept(null, new CompletionHandler<AsynchronousSocketChannel, Object>() {
               @Override
               public void completed(AsynchronousSocketChannel result, 
                                     Object attachment) {
                   //执行成功的回调函数
               }
               @Override
               public void failed(Throwable exc, Object attachment) {
                   //执行失败的回调函数
               }
           });
       }
   }
   ```
> 值得注意的是：AIO是jdk1.7引入，不同的操作系统的底层实现原理是不一致的，比如：Linux是epoll，而window的是ocp



### 二、底层实现

上述说了，不同的操作系统的底层实现原理是不一样的。linux的底层实现原理是epoll，而window的是ocp。接下

来会讲解linnux操作系统的实现。

**2.1、文件描述符**

Linux的内核会将所有的外部设备的看成一个文件，操作设备就如同操作文件一样。Linux对一个文件的读写会调

用内核提供的系统命令（api），返回一个file descriptor（fd，文件描述符）。而对一个socket的读写也会有相应

的描述符，成为socket fd（socket文件的描述符），描述符就是一个数字，指向内核的一个结构体（文件路径，

数据区等一些属性）。即**在Linux下对文件的操作是利用文件描述符（file descriptor）来实现的。**

**2.2、用户空间和内核控件**

为了保证用户进程不能直接操作内核，保证内核的拿权，操作系统将虚拟空间分成两部分：内核控件和用户空间

**2.3、I/O操作运行过程**

下面以I/O操作的读取数据（read为例），说明I/O的操作过程：

![IO](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/IO.png)

①：应用程序调用操作系统提供的read接口的API

②：系统查询用户空间是否有数据，有则返回数据。没有，进入第三步

③：操作系统等待数据准备（查看内核空间有没有数据，则等待数据传入，有则进入第四步）

④：操作系统内核地址空间拷贝数据到用户地址空间上

⑤：应用程序发现用户地址空存有数据了，读取。

可以发现，如果应用程序调用read方法读取数据时，用户地址空间若不存在数据，则是需要等待的，且这个等待

是必须的——从内核空间中找数据，再将内核空间的数据拷贝到用户空间的。

以上内容来源于：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484235&idx=1&sn=4c3b6d13335245d4de1864672ea96256&chksm=ebd7424adca0cb5cb26eb51bca6542ab816388cf245d071b74891dd3f598ccd825f8611ca20c&scene=21###wechat_redirect">JDK10都发布了，nio你了解多少？</a>

**2.4、IO模型介绍**

* 阻塞I/O模型（BIO）

  进行BIO操作时，用户空间上调用recvfrom函数(经socket接收数据)，此函数等到直到数据包获取到并成功拷

  贝到用户地址空间上或者发生错误时才会返回，**recvfrom函数在此期间会一直等待（阻塞）**。![BIO](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/BIO.png)

* 非阻塞I/O模型（NIO）

  进行NIO操作时，用户空间上调用recvfrom函数(经socket接收数据)，revcfrom从应用到内核的过程，如果没

  有数据返回，就返回一个EWOULDBLOCK状态标识，用户进程不断的轮行检查该状态，根据状态做出相应的

  操作，如数据到来则处理数据。

![NIO](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/NIO.png)

* I/O复用模型（NIO）

  Linux下的I/O复用模型的本质是：调用操作系统调用select/poll/epoll/pselect其中的一个函数，传入多个文件

  描述符和等待时间，如果其中有一个文件符准备就绪，则返回，反之阻塞直到等待超时。![IO复用模型](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/IO%E5%A4%8D%E7%94%A8%E6%A8%A1%E5%9E%8B.png)

  过程解释一下：①用户进程调用上述方法之一，进程被阻塞。②同时内核会监视select负责的所有socket。③

  当其中一个socket返回可操作状态时（读/写），select就会返回。④然后用户进程就会调用read操作，将数

  据从内核中拷贝，操作数据。  IO复用模型就是使用某种机制让一个进程能同时等待多个I/O，当I/O的文件描

  述符进入可操作性状态时，函数就会返回标识给用户进程，然后用户进程根据状态标识操作。值得注意：

  **select/epoll函数的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接**。

  

* 异步非阻塞I/O模型

  异步非阻塞I/O模型是一种处理与I/O重叠进行的模型。该模型的读（read）请求发起后便会立即返回状态，标

  识读（read）已经成功发起。应用程序是非阻塞的，进程进行其他处理操作。当后台执行read操作时（数据

  拷贝到用户地址空间），就会执行一个基于线程的回调函数完成本次的I/O读操作。

  ![AIO](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/AIO.png)



参考资料： <a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484235&idx=1&sn=4c3b6d13335245d4de1864672ea96256&chksm=ebd7424adca0cb5cb26eb51bca6542ab816388cf245d071b74891dd3f598ccd825f8611ca20c&scene=21###wechat_redirect">JDK10都发布了，nio你了解多少？</a>

