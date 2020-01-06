## HTTP 常见面试题

写在前面。原文来源：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484979&idx=3&sn=37528f67d16315e5d49ad48ebf6bd53d&chksm=ebd74732dca0ce2446aa4a66e22f082d48b49f63301be748e61ce652b3a00de53d4f15cef3a7###rd">HTTP常见面试题(修订版)</a>



**1、Http与Https的区别**

* HTTP 的URL 以http:// 开头，而HTTPS 的URL 以https:// 开头
* HTTP 是不安全的，而 HTTPS 是安全的
* HTTP 标准端口是80 ，而 HTTPS 的标准端口是443
* 在OSI 网络模型中，HTTP工作于应用层，而HTTPS 的安全传输机制工作在传输层
* HTTP 无法加密，而HTTPS 对传输的数据进行加密
* HTTP无需证书，而HTTPS 需要CA机构的颁发的SSL



**2、什么是Http协议无状态协议?怎么解决Http协议无状态协议?**

- 无状态协议对于事务处理没有记忆能力**。**缺少状态意味着如果后续处理需要前面的信息

- 无状态，当客户端一次HTTP请求完成以后，客户端再发送一次HTTP请求，HTTP并不知道当前客户端是

  一个”老用户“。

- 对于无状态问题，使用Cookie来解决，Cookie就相当于一个通行证，第一次访问的时候给客户端发送一个

  Cookie，当客户端再次来的时候，拿着Cookie(通行证)，那么服务器就知道这个是”老用户“。



**3、URI和URL的区别**

3.1、URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。Web的资源是多种多样

的，如HTML文档、图像、视频片段、程序等都是一个来URI来定位的。URI一般由三部组成：①访问资源的命名机

制；②存放资源的主机名③；资源自身的名称，由路径表示，着重强调于资源。



3.2、URL，是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资  

源，也指明了如何locate这个资源。URL是 Internet 上用来描述信息资源的字符串，主要用在各种 WWW 客户程

序和服务器程序上，特别是著名的Mosaic。采用URL可以用一种统一的格式来描述各种信息资源，包括文件、服

务器的地址和目录等。URL一般由三部组成：①协议(或称为服务方式)；②存有该资源的主机IP地址(有时也包括端

口号)；③主机资源的具体地址。如目录和文件名等



3.3、URN，uniform resource name，统一资源命名，是通过名字来标识资源，比如 mailto:javanet@java.sun.c

om。URI是以一种抽象的，高层次概念定义统一资源标识，而URL和URN则是具体的资源标识的方式。URL和

URN都是一种URI。笼统地说，每个 URL 都是 URI，但不一定每个 URI 都是 URL。这是因为 URI 还包括一个子类

，即统一资源名称 (URN)，它命名资源但不指定如何定位资源。上面的 mailto、news 和 isbn URI 都是 URN 

的示例。



3.4、在Java的URI中，一个URI实例可以代表绝对的，也可以是相对的，只要它符合URI的语法规则。而URL类则

不仅符合语义，还包含了定位该资源的信息，因此它不能是相对的。在Java类库中，URI类不包含任何访问资源的

方法，它唯一的作用就是解析。相反的是，URL类可以打开一个到达资源的流。



**4、常用的HTTP方法有哪些？**

- GET： 用于请求访问已经被URI（统一资源标识符）识别的资源，可以通过URL传参给服务器。【获取数据】
- POST：用于传输信息给服务器，主要功能与GET方法类似，但一般推荐使用POST方式。【提交数据】
- PUT： 传输文件，报文主体中包含文件内容，保存到对应URI位置。【更新数据】
- DELETE：删除文件，与PUT方法相反，删除对应URI位置的文件。【删除数据】
- HEAD： 获得报文首部，与GET方法类似，只是不返回报文主体，一般用于验证URI是否有效。
- OPTIONS：查询相应URI支持的HTTP方法。



**5、HTTP请求报文与响应报文格式**

请求报文包含四部分：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2FsMVzfNxopOlFhKsfticyLWOKKLicial5cs5icYFeWF79IXUAGtFMcMiagdGV3SBee0RBfOG4uOia6STQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- a、请求行：包含请求方法、URI、HTTP版本信息
- b、请求首部字段
- c、请求内容实体
- d、空行

响应报文包含四部分：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2FsMVzfNxopOlFhKsfticyLAqOXeCHgG7SNTDicJIAMbakyOicJTd7DCFqofb70kxx91DR6EJsjibFHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- a、状态行：包含HTTP版本、状态码、状态码的原因短语
- b、响应首部字段
- c、响应内容实体
- d、空行



**6、HTTPS工作原理**

①：首先HTTP请求服务端生成证书，客户端对证书的有效期、合法性、域名是否与请求的域名一致、证书的公钥

（RSA加密）等进行校验；

②：客户端如果校验通过后，就根据证书的公钥的有效， 生成随机数，随机数使用公钥进行加密（RSA加密）；

③：消息体产生之后，对它的摘要进行MD5（或者SHA1）算法加密，此时就得到了RSA签名；

④：发送给服务端，此时只有服务端（RSA私钥）能解密。

⑤：解密得到的随机数，再用AES加密，作为密钥（此时的密钥只有客户端和服务端知道）。

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2FsMVzfNxopOlFhKsfticyLCttnI5zMBYiazg6C0vrOn79U4icugdAialERkpFMqnfVLiclGZPR9icaLiaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

具体的参考链接：http://blog.csdn.net/sean_cd/article/details/6966130



**7、一次完整的HTTP请求所经历的7个步骤**

HTTP通信机制是在一次完整的HTTP通信过程中，Web浏览器与Web服务器之间将完成下列7个步骤：

- 建立TCP连接

  >在HTTP工作开始之前，Web浏览器首先要通过网络与Web服务器建立连接，该连接是通过TCP来完成
  >
  >的，该协议与IP协议共同构建 Internet，即著名的TCP/IP协议族，因此Internet又被称作是TCP/IP网络。
  >
  >HTTP是比TCP更高层次的应用层协议，根据规则， 只有低层协议建立之后，才能进行更高层协议的连
  >
  >接，因此，首先要建立TCP连接，一般TCP连接的端口号是80。

- Web浏览器向Web服务器发送请求行

  > 建立了TCP连接，Web浏览器才会向Web服务器发送请求命令。

- Web浏览器发送请求头

- > 浏览器发送其请求命令之后，还要以头信息的形式向Web服务器发送一些别的信息，之后浏览器会发送一空白行来通知服务器，表明信息发送结束。

- Web服务器应答

- > 浏览器向服务器发出请求后，服务器会根据请求回送应答， HTTP/1.1 200 OK ，应答的第一部分是协议
  >
  > 的版本号和应答状态码。

- Web服务器发送应答头

- > 正如客户端会在请求发送关于自身的信息一样，服务器也会在应答时，向用户发送关于它自己的数据及被
  >
  > 请求的文档。

- Web服务器向浏览器发送数据

- > Web服务器向浏览器发送相应头信息后，也会发送一个空白行来表示头信息的发送到此为结束，接着就
  >
  > 会在Content-Type应答头信息描述文件格式和回送用户所请求的实际数据。

- Web服务器关闭TCP连接 

- > 一般情况下，Web服务器相应浏览器请求，向浏览器回送请求数据之后，是关闭TCP连接。然后如果浏览
  >
  > 器或者服务器在其头信息加入了这行代码：Connection:keep-alive，则TCP连接在发送后将仍然保持打
  >
  > 开状态，浏览器仍可以继续通过同一连接发送请求。保持连接能节省为每个请求建立新连接所需的时间，
  >
  > 也节约了网络带宽。

建立TCP连接->发送请求行->发送请求头->（到达服务器）发送状态行->发送响应头->发送响应数据->断TCP连接

最具体的HTTP请求过程：http://blog.51cto.com/linux5588/1351007



**8、HTTP1.1版本新特性**

- 默认使用持久连接节省通信量，只要客户端服务端任意一端没有明确提出断开TCP连接，就一直保持连接，可

  以发送多次HTTP请求

- 管线化，客户端可以同时发出多个HTTP请求，而不用一个个等待响应。

- 断点续传 ：实际上就是利用HTTP消息头使用分块传输编码，将实体主体分块传输



**9、HTTP优化方案**

- TCP复用：TCP连接复用是将多个客户端的HTTP请求复用到一个服务器端TCP连接上，而HTTP复用则是一个

  客户端的多个HTTP请求通过一个TCP连接进行处理。前者是负载均衡设备的独特功能；而后者是HTTP 1.1协

  议所支持的新功能，目前被大多数浏览器所支持。

- 内容缓存：将经常用到的内容进行缓存起来，那么客户端就可以直接在内存中获取相应的数据了。

- 压缩：将文本数据进行压缩，减少带宽

- SSL加速（SSL Acceleration）：使用SSL协议对HTTP协议进行加密，在通道内加密并加速

- TCP缓冲：通过采用TCP缓冲技术，可以提高服务器端响应时间和处理效率，减少由于通信链路问题给服务器造成的连接负担。

详情参考：

- http://blog.51cto.com/virtualadc/580832
- http://www.cnblogs.com/cocowool/archive/2011/08/22/2149929.html