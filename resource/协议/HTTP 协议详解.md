## Http协议详解



### 一、什么是Http协议

**1、定义**

超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。所有的WWW

文件都必须遵守这个标准。它是TCP/IP协议的一个**应用层协议**。简单来说，**HTTP协议就是客户端和服务器交互的**

**一种通迅的格式**。



例子: 在浏览器点击一个链接后，浏览器向服务发送请求，服务器根据请求做出相应。

原理：当在浏览器中点击这个链接的时候，**浏览器会向服务器发送一段文本**，**告诉服务器，浏览器请求的资源。服**

**务器收到请求后，就返回一段文本给浏览器，浏览器会将该文本解析，然后显示出来。**浏览器发送的文本和服务器

发送的文本都是这段文本都是遵循HTTP协议规范的。



### 二、Http连接

Http连接是基于TCP的，当客户端发送一个http请求时，需要先建立TCP连接，即**三次握手**。

1. 客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认；

2. 服务器收到syn包，必须确认客户的SYN，同时自己也发送一个SYN包，即SYN+ACK包，此时服务器进入

   SYN_RECV状态；

3. 客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK，客户端和服务器进入ESTABLISHED状态，完成

   三次握手。

 SYN：(Synchronize Sequence Numbers) 即是同步序列编号
 ACK：(Acknowledge character) 即是确认字符

![img](https:////upload-images.jianshu.io/upload_images/1903970-3ed1c686f17de5eb.png?imageMogr2/auto-orient/strip|imageView2/2/w/910/format/webp)



**2.1、HTTP1.0和HTTP1.1的区别**

* HTTP1.0协议中，客户端与web服务器建立连接后，只能获得一个web资源【短连接，获取资源后就断开连接

* HTTP1.1协议，允许客户端与web服务器建立连接后，在一个连接上获取多个web资源【保持连接，长连接】



**2.2、http的的短链接是无连接，长/短链接都是无状态的。**

- 无连接：客户端与服务器每次连接只处理一个请求。服务器接受完客户端的请求，开始处理，然后响应客

  户端，在客户端收到应答之后，就断开连接。其目的就是为了节省传输时间，听过处理事务的能力。

- 无状态：HTTP协议是无状态协议。协议的无状态是指：对于事务处理没有记忆能力。其好处就是：如果

  后续的连接请求不需要之前提供的信息，响应就会比较快。这种方式也有坏处：如果后续的处理需要用到

  之前的信息,则必须要重传,这样就导致了每次连接传输的数据量增大。

- 无状态的处理：为了解决HTTP的无状态特性,于是出现了 Cookie 和 Session 技术。

  

**2.3、持久连接：**

http的持久连接的出现是为了解决一个Http连接要能够处理多个请求的问题，比如说：访问一个网页，该网页有非

常多的图片。假设一个图片就算是一个HTTP请求了。那么在中途中就不断地建立TCP连接、获取图片、断开TCP连

接。这是十分耗费资源的，http的长连接就是为解决这种情况而诞生的。长连接为 “管线化” 方式发送成为了可能 :

**在一次HTTP连接过程，不需要等待服务器响应请求，就能够继续发送第二次请求**。



**2.4、提升传输效率**

在说明之前，首先需要知道什么是实体主体，实体主体就是作为数据在HTTP中传输的数据。一般地，**实体主体可**

**以等价为报文主体，报文主体是HTTP中的一部分**。

如果不使用任何手段，服务器返回的数据实体主体是原样返回的，这样传输的速率是有些慢的，可以使用以下三种

方法提高传输效率。

- **使用压缩技术把实体主体压小，在客户端再把数据解析**

- **使用分块传输编码，将实体主体分块传输，当浏览器解析到实体主体就能够显示了。**

- **使用范围请求**，这种请求只会下载资源的一部分，获取部分数据后，在接着发起请求获取值后的数据，如下载

- 文件是可以暂停值后接着下载的。

  

### 三、Http请求

 一个完整的 http 请求应该包含：请求行、请求头、空行和请求数据等四个部分。

* 请求行：分为三个部分：请求方法、请求地址和协议版本

  * 请求方法： POST、GET、DELETE、PUT、OPTIONS、HEAD、TRACE。比较常用的是 GET 和 POST 方法

    一般在网页中点击连接都是发起的是 GET 方法的请求，网页中的表单数据一般也都是使用 POST 方法提

    交的请求。GET 发起的请求，其请求携带的数据是拼接在URL的后面，数据的大小不能超过1K，而POST

    的请求携带的数据是卸载请求实体内容中的，数据量的大小是没有限制的。

* 请求头

  * Accept：text/html，image/* 【浏览器告诉服务器，其支持的数据类型】

  * Accept-Charset：ISO-88591-1 【浏览器告诉服务器，其支持的字符集】
  * Accept-Encoding：gzip，compress 【浏览器告诉服务器，其支持的压缩格式】

  * Accept-Language：en-us,zh-cn 【浏览器告诉服务器，其的语言环境】

  * Cache-Control: no-cache 【浏览器告诉服务器，缓存控制情况】
  * Connection: Upgrade    【浏览器告诉服务器，连接情况】
  * Host: live.github.com    【浏览器告诉服务器，访问的主机地址】
  * Origin: https://github.com   【浏览器告诉服务器，访问的原始主机地址】
  * Cookie      【浏览器告诉服务器，携带的cookie】
  * Date: Tue, 11 Jul 2000 18:23:51 GMT【浏览器告诉服务器，请求的时间】

* 请求数据：

  * 如果是 GET 请求，则请求数据拼接在 URL 的后面
  * 如果是 POST 请求，则请求的数据卸载请求的实体内容中。



一个完整的 Http 相应应该包含四个部分：状态行、相应头，空行、实体内容

* 一个状态行：用于描述服务器响应请求的处理结果 ，其格式为： HTTP版本号　状态码　原因叙述

* 多个消息头：用于描述服务器的基本信息和数据的描述，服务器通过数据内容的描述信息，告知客户端如何处

  理服务器返回的数据。

* 实体内容：服务器向客户端回送的数据。



Http相应状态码：服务器对请求的处理结果，是一个三位的十进制数。响应状态码分为5类

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2FPYiazSBurrVsbt1mkHFfDx5D2BjHv8U2hOAeXDJcALicibrHQbBGQBX4YsvHeicTEKmEF3IqIRjWAA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

响应头：

- Location: http://www.it315.org/index.jsp 【服务器告诉浏览器**要跳转到哪个页面**】
- Server:apache tomcat【服务器告诉浏览器，服务器的型号是什么】
- Content-Encoding: gzip 【服务器告诉浏览器**数据压缩的格式**】
- Content-Length: 80 【服务器告诉浏览器回送数据的长度】
- Content-Language: zh-cn 【服务器告诉浏览器，服务器的语言环境】
- Content-Type: text/html; charset=GB2312 【服务器告诉浏览器，**回送数据的类型**】
- Last-Modified: Tue, 11 Jul 2000 18:23:51 GMT【服务器告诉浏览器该资源上次更新时间】
- Refresh: 1;url=http://www.it315.org【服务器告诉浏览器要**定时刷新**】
- Content-Disposition: attachment; filename=aaa.zip【服务器告诉浏览器**以下载方式打开数据**】
- Transfer-Encoding: chunked 【服务器告诉浏览器数据以分块方式回送】
- Set-Cookie:SS=Q0=5Lb_nQ; path=/search【服务器告诉浏览器要**保存Cookie**】
- Expires: -1【服务器告诉浏览器**不要设置缓存**】
- Cache-Control: no-cache 【服务器告诉浏览器**不要设置缓存**】
- Pragma: no-cache 【服务器告诉浏览器**不要设置缓存**】
- Connection: close/Keep-Alive 【服务器告诉浏览器连接方式】
- Date: Tue, 11 Jul 2000 18:23:51 GMT【服务器告诉浏览器回送数据的时间】



### 四、HTTPS简述

由于 Http 在协议安全方面是存在缺陷的，其缺点表现在：

* 通信使用明文【没有对数据进行加密】
* 不验证通信方身份，无论是客户端还是服务端，都是可以随意通信的
* 无法确保报文的完整性 【有被篡改的可能性】

为了解决 Http 协议的问题，于是就出现 Https 协议（其实就是会用了 SSL安全协议的 Http 协议）。HTTPS使用

的是共享密钥和公开私有密钥混合来进行加密的。由于公开私有密钥需要太多的资源，不可能一直以公开私有密

钥进行通信。因此，HTTP在建立通信线路的时先使用公开私有密钥，当建立完连接后，就会使用共享密钥进行加

密和解密。对于认证方面，HTTPS 是基于第三方的认证机构来获取认可证书的、因此可以从认证该服务器判断请

求是否为合法的。但这样，客户端方面就需要自己购买认证证书、这实施起来难度是很大的【认证证书需要钱】。

所以，一般的网站都是使用表单认证的，这是目前使用最广泛的客户端认证方式。



参考资料：

<a href="https://www.jianshu.com/p/a5afb451be1e">HTTP请求、Socket连接、TCP连接的关系</a>

<a href="https://www.jianshu.com/p/7c8b4576e4bb">HTTP协议详解</a>

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484979&idx=2&sn=abe78c7ce58c15cb8b2e26602802e096&chksm=ebd74732dca0ce24b00b10ed3948801bc1ab0fdfa3cdb478b21d1048a4e5564bbda2316b31bb###rd">HTTP就是这么简单(修订版)</a>