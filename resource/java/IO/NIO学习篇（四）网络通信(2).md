## NIO学习篇（四）网络通信（2）代码实战

在上两篇《<a href="https://github.com/jogin666/blog/blob/master/resource/java/IO/NIO%E5%AD%A6%E4%B9%A0%E7%AF%87%EF%BC%88%E4%BA%8C%EF%BC%89%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1(1).md">NIO学习篇章（二）网络通信（1）入门</a>》和《<a href="https://github.com/jogin666/blog/blob/master/resource/java/IO/NIO%E5%AD%A6%E4%B9%A0%E7%AF%87%EF%BC%88%E5%9B%9B%EF%BC%89NIO%26epoll%E8%AE%B2%E8%A7%A3.md">NIO学习篇（四）NIO&epoll讲解</a>》中，介绍了NIO的三

个模式和模式的底层操作系统的实现，以及NIO和Linux系统的epoll之间的联系。相信看过之后，都大致明白了

NIO是怎么使用的了吧！本篇章将使用代码实践一下NIO的知识。



**1、Channel（管道）回顾**

Channel（管道）：用于源数据节点与目标结点的连接。在Java的NIO中负责缓冲区的数据传输。管道不负责与数

据打交道，因此需要配合缓冲区使用。

- 常用的管道类

  FileChanne：用于文件操作

  SocketChannel：用于TCP协议网络通信中客户端的数据运输

  SocketServerChannel：用于TCP协议网络通信中服务端的数据运输

  DataGramChannel：用于UDP协议网络通信的数据运输

  Pipe.SinkChannelhe Pipe.SourceChannel：用于线程之间的数据运输

**2、非阻塞I/O模型回顾**

在前面的两个篇章中结合Channel的子类的，可以看的出来，NIO常说的非阻塞是在网络通信中实现的，NIO在文

件中非阻塞是几乎用不到的。在前面的两个篇章中收到过，NIO的非阻塞模式的主要核心成员是：Buffer（缓冲区

）、Channel（管道）、Selector（选择器，重点）。在NIO的非阻塞模型中，Selector（选择器）会轮询检测其

注册列表上的连接（客户端与服务端的连接），如果连接发过来的请求携带有数据，则会创建一个线程处理该连接

的请求。

![Selector](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/Selector.png)

**3、基于TCP的网络通信代码演示**

为了更好地理解，先来演示一下NIO**在网络中是阻塞的状态代码**。

```java
//客户端
public class BlockClient {

    public static void main(String args[]) throws IOException {
        //建立与服务器的连接，然后获取连接的通道
        SocketChannel socketChannel = SocketChannel.open(
                new InetSocketAddress("127.0.0.1", 6666));
		//指定要发送的文件
        FileChannel fc = FileChannel.open(
            			Paths.get("D:\\Typora\\projects\\java基础\\IO\\images\\AIO.png"),
                		StandardOpenOption.READ);
        //创建一个大小为1M的缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        //将文件的内容读取到socketChannel的缓冲区中
        while (fc.read(buffer)!=-1){
            buffer.flip(); //切换读模式
            socketChannel.write(buffer);
            buffer.clear();//清除缓冲区
        }
        //关闭通道
        fc.close();
        socketChannel.close(); //并发送数据
    }
}
```

服务端

```java
public class BlockServer {
    public static void main(String args[]) throws IOException {
        //创建一个通道
        ServerSocketChannel serverChannel = ServerSocketChannel.open();
        //为通道指定端口号，获取该端口上的连接请求的数据
        serverChannel.bind(new InetSocketAddress(6666));
        //获取客户端发送来的数据的通道
        SocketChannel channel = serverChannel.accept();
        //创建缓冲区
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        //创建文件输出的通道
        FileChannel outChannel = FileChannel.open(Paths.get("6.jpg"),
                StandardOpenOption.CREATE_NEW, StandardOpenOption.WRITE);
        //读取客户端发来的数据
        while(channel.read(buffer)!=-1){
            buffer.flip();//切换成读模式
            outChannel.write(buffer);
            buffer.clear(); //清空缓冲区
        }
        //关闭
        outChannel.close();
        channel.close();
        serverChannel.close();
    }
}
```

程序执行完成之后，可以看到在项目的工程多了一个文件，其实上面的程序是阻塞的，因为服务端无法得知客户是否发送了数据。要告诉服务端，客户端已经写完数据，需要在通关闭之前加上socketChannel.shutdownOutput();向服务器表名客户端已经写完数据了。

![1](https://github.com/jogin666/blog/blob/master/resource/java/IO/images/1.png)

NIO非阻塞的代码演示：

客户端：

```java
public static void main(String args[]) throws IOException {
    SocketChannel socketChannel = SocketChannel.open(
        new InetSocketAddress("127.0.0.1", 6666));
    //切换成非阻塞模式
    socketChannel.configureBlocking(false);
    //获取选择器
    Selector selector = Selector.open();
    //监听通道上的服务端返回的数据
    socketChannel.register(selector,SelectionKey.OP_READ);
    //上传图片
    FileChannel fc = FileChannel.open(
        				 Paths.get("D:\\Typora\\projects\\java基础\\IO\\images\\AIO.png"),
                         StandardOpenOption.READ);
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    while (fc.read(buffer)!=-1){
        buffer.flip();
        socketChannel.write(buffer);
        buffer.clear();
    }
    while(selector.select()>0){

        Set<SelectionKey> keys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = keys.iterator();
        while (iterator.hasNext()){
            SelectionKey key = iterator.next();
            //读取服务器返回来的消息
            if (key.isReadable()){
                SocketChannel channel= (SocketChannel) key.channel();
                ByteBuffer byteBuffer=ByteBuffer.allocate(1024);
                int len=channel.read(byteBuffer);
                if (len>0){
                    byteBuffer.flip();
                    System.out.println(new String(byteBuffer.array(),0,len));
                }
            }
            iterator.remove();
        }
    }

}
```

服务端：

```java
public static void main(String args[]) throws IOException {

    //创建一个通道
    ServerSocketChannel serverChannel = ServerSocketChannel.open();
    //为通道设置成非阻塞
    serverChannel.configureBlocking(false);
    //为通道指定端口号，获取该端口上的连接请求的数据
    serverChannel.bind(new InetSocketAddress(6666));
    //创建选择器
    Selector selector = Selector.open();
    //设置监听指定端口的通道，并设置监听事件为： 通道接收 事件
    serverChannel.register(selector, SelectionKey.OP_ACCEPT);

    //选择存在连接，则不断轮询访问连接的状态
    while(selector.select()>0){
        //获取通道上建立连接的 选择键
        Set<SelectionKey> keys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = keys.iterator();
        while (iterator.hasNext()){
            SelectionKey key = iterator.next();
            if (!key.isValid()){ //连接是否已失效
                continue;
            }else if (key.isAcceptable()){
                SocketChannel channel = serverChannel.accept();
                channel.configureBlocking(false);
                //注册到选择器上-->拿到客户端的连接为了读取通道的数据(监听读就绪事件)
                channel.register(selector,SelectionKey.OP_READ);
            }else if (key.isReadable()){
                //读取客户端上传的数据
                SocketChannel channel=(SocketChannel)key.channel();
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                FileChannel outChannel = FileChannel.open(Paths.get("6.jpg"),
                                                          StandardOpenOption.CREATE_NEW, 
                                                          StandardOpenOption.WRITE);
                while (channel.read(buffer)>0){
                    buffer.flip();
                    outChannel.write(buffer);
                    buffer.clear();
                }
                //告诉客户端上传数据成功
                buffer.put("successful put a message".getBytes());
                buffer.flip();
                channel.write(buffer);
            }
            //移除已经轮序过的选择键
            iterator.remove();
        }
    }
}
```

结果就不演示了。下面就简单总结一下使用NIO时的要点：

1. 获取指定地址（服务地址，端口）的通道
2. 创建选择器Selector，并让选择监听通道上指定的监听事件
3. Selector不断轮询查看通道上的连接的状态（监听事件）是否发生
4. 如果状态改变，则根据状态值，做出相应的处理。
5. 处理完事件之后，移除事件。



**4、基于UDP协议的网络通信代码演示**（和tcp类似）

客户端：

```java
public static void main(String args[]) throws IOException {
    DatagramChannel channel = DatagramChannel.open();
    channel.bind(new InetSocketAddress("127.0.0.1",6666));
    channel.configureBlocking(false);
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    buffer.put("this is udp test".getBytes());
    buffer.flip();
    channel.read(buffer);
    channel.close();
}
```

服务端：

```java
public static void main(String args[]) throws IOException {
    DatagramChannel dgChannel = DatagramChannel.open();
    dgChannel.bind(new InetSocketAddress(6666));
    dgChannel.configureBlocking(false);
    Selector selector = Selector.open();
    dgChannel.register(selector, SelectionKey.OP_ACCEPT);
    while(selector.select()>0){
        Set<SelectionKey> keys = selector.selectedKeys();
        Iterator<SelectionKey> iterator = keys.iterator();
        while (iterator.hasNext()){
            SelectionKey key = iterator.next();
            if (key.isAcceptable()){
                //...
            }else if (key.isReadable()){
                //...
            }
        }
    }
}
```

**5、管道（Pipe）**

Java NIO管道是2个线程之间的单向数据连接。Pipe有一个sink管道和一个source管道。其中sink管道是用来存放线程写入数据的。而source管道是获取线程写入skin管道的数据。

```java
public static void main(String args[]) throws IOException, InterruptedException {
    final Pipe pipe = Pipe.open();
    final ByteBuffer buffer=ByteBuffer.allocate(1024);
    new Thread(()->{
        try {
            buffer.put("使用单线管道写入数据".getBytes());
            buffer.flip();
            Pipe.SinkChannel channel = pipe.sink();
            channel.write(buffer);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }).start();
    Thread.sleep(2000); //休眠2s
    new Thread(()->{
        try {
            Pipe.SourceChannel channel = pipe.source();
            buffer.flip();
            int len=channel.read(buffer);
            System.out.println(new String(buffer.array(),0,len));//使用单线管道写入数据
        } catch (IOException e) {
            e.printStackTrace();
        }
    }).start();
}
```

**6、AIO网络异步通信代码演示**

​      <a href="https://blog.csdn.net/wanbf123/article/details/78054852">AIO编程（NIO2.0）及代码实现</a>









参考资料：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484235&idx=1&sn=4c3b6d13335245d4de1864672ea96256&chksm=ebd7424adca0cb5cb26eb51bca6542ab816388cf245d071b74891dd3f598ccd825f8611ca20c&scene=21###wechat_redirect">JDK10都发布了，nio你了解多少？</a>
