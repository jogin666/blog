

## 学习 HTTP2和HTTPS

写在前面，原文地址：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484302&idx=1&sn=5fafcb988463b5b2df9120552b6dc3f8&chksm=ebd7428fdca0cb99d5ed60296100b315c4ecaefc901fb5bb5448f902c6f41b0fa0dc18d5ee06&scene=21###wechat_redirect">HTTP2和HTTPS来不来了解一下？</a>



### 一、HTTP协议的今生来世

最近在看博客的时候，发现有的面试题已经考HTTP2了，于是我就顺着去了解一下。

到现在为止，HTTP协议已经有三个版本了：① HTTP1.0    ② HTTP1.1   ③  HTTP/2

下面就简单聊聊他们三者的区别，以及整理一些必要的额外知识点。



### 二、Http 版本之间的区别

HTTP1.0 和 HTTP1.1 最主要的区别就是：**HTTP1.1默认是持久化连接！**

HTTP1.0默认是短连接：![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemHOwW3sL5UAYv5kfdByQhb5ibuK3kgNOABDfviaJy06ic1eRxteyr0WeSg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

简单来说就是：每次与服务器交互，都需要新开一个连接！

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemvgzbJWWjqibtTsFialZKnhulVeCoU7QmZX64eXCAHINle2CoWYcQ3qicg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemicONRh6woRiaice1N5AkSia7RjPUedhUicNmgJzTM2mAXlRsKprqibb9hn7A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP 协议是基于 TCP 的，而 TCP 每次都要经过**三次握手，四次挥手，慢启动** 等步骤，这都需要消耗非常多的资

源！试想一下：如果每请求一张图片，一个CSS文件了或者一个JS文件，都需要新开一个 Http 连接，这得耗费多

少资源啊。为了解决短链接的缺陷，于是在 HTTP1.1 中就默认使用持久化连接了：**建立一次连接，可以发起**

**多个请求**！(如果阻塞了，还是会开新的TCP连接的)。

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemiamB8ibsA1FZdLcSkca8YjSyBD3rnMf13TibwJlfvSK114ShppnJvnUUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

相对于持久化连接还有另外比较重要的改动：

- HTTP 1.1 增加host字段

- HTTP 1.1 中引入了范围请求（Chunked transfer-coding），实现断点续传的功能。( 实际上就是在HTTP消

  息头使用分块传输编码，将实体主体分块进行传输 )

- HTTP 1.1 管线化(pipelining)理论，客户端在一次连接中不需要等待服务器相应，可以连续发起多个请求。

  （之前是服务器响应后，才能重新发起请求）

- - 但是 pipelining 仅仅是**限于理论场景下**，大部分桌面浏览器仍然会**默认选择关闭** HTTP pipelining 功能。
  - 所以现在使用 HTTP1.1 协议的应用，还是有**可能会开多个TCP连接**的。

参考资料：<a href="https://www.cnblogs.com/gofighting/p/5421890.html">HTTP1.0和HTTP1.1的区别</a>



### 三、HTTP2介绍

在说HTTP2之前，不如先直观比较一下 HTTP2 和 HTTP1.1 的区别：

- https://http2.akamai.com/demo

![img](https://mmbiz.qpic.cn/mmbiz_gif/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeem3drXG9fEKvVGEFEIrhqV27B2SxGibTm3ZKLphTXwIyB8icic5icWhbyfqA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

上面也已经说了，HTTP 1.1 虽然提出了管线化(pipelining)理论，但是仅仅是限于理论的阶段上，这个功能默认还

是没有被广泛使用，多数浏览是默认关闭的。

管线化(pipelining)和非管线化的**区别**：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemhVrib5l2daeFVQAEIVWK2yGn1UzvEfYYXXWCGxXib6pHHVaGCD2AFmMw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemrmZuVZMTAl5zicJukuywvFYZMiavyw0LBkziaq5FbbbicWLeLRPOXUvu4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> HTTP Pipelining其实是把多个HTTP请求放到一个TCP连接中 一 一发送，而在发送过程中不需要等待服务器
>
> 对前一个请求的响应；只不过，**客户端还是要按照发送请求的顺序来接收响应！**
>
> 就像在超市收银台或者银行柜台排队时一样，你并不知道前面的**顾客**是干脆利索的还是会跟收银员/柜员磨蹭
>
> 到世界末日（不管怎么说，服务器（即收银员/柜员）是要按照顺序处理请求的，如果**前一个请求非常耗时**
>
> **（顾客磨蹭）**，那么后续请求都会受到影响。

- 在HTTP1.0 时，发送一次请求，需要**等待服务端响应之后**才可以继续发送下一个请求。

- 在HTTP1.1 时，发送一次请求，是不需要等待服务端响应的，直接可以发送下一个请求，但是服务器回送数

  据给客户端时，客户端还是需要按照**响应的顺序**来一 一接收的。

- 所以说，无论是 HTTP1.0 还是 HTTP1.1 提出的 Pipelining 理论，都会出现**阻塞**的情况。从专业的名词上说这

  种情况，叫做**线头阻塞**（Head of line blocking）简称：HOLB



**HTTP1.1 和 HTTP2 区别**

HTTP2 与 HTTP1.1最重要的区别就是 **解决了线头阻塞的** 问题。其中最重要的改动是：**多路复用 (Multiplexing)**

多路复用意味着线头阻塞将不在是问题，多路复用允许同时通过单一的 HTTP/2 连接发起 **多重的请求-响应消息**，

HTTP1.1 的合并多个请求为一个的优化将不再适用。上述讲了：HTTP1.1 中的 Pipelining 是没有付诸于实际的，

HTTP1.1 为了**减少**HTTP请求，将多个请求合并为一个，比如：Spriting(多个图片合成一个图片)，内联Inlining(将

图片 的原始数据嵌入在CSS文件里面的URL里），拼接Concatenation(一个请求要求下载完多个JS文件)，分片

Sharding (将请求分配到各个主机上) ……。



使用了 HTTP2 之后，可能如下图：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemqUVhTDUO0shiasqkkwib4eIAia6U6VX4EVFp4ZzbL1JPCcekQlNHwoKicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP2 所有性能增强的核心都在于 **新的二进制分帧层** (不再以文本格式来传输了)，它定义了如何封装 http 消息并

在客户端与服务器之间传输。

使用了HTTP2可能是这样子的：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemqUVhTDUO0shiasqkkwib4eIAia6U6VX4EVFp4ZzbL1JPCcekQlNHwoKicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP2所有性能增强的核心在于**新的二进制分帧层**(不再以文本格式来传输了)，它定义了如何封装http消息并在客

户端与服务器之间传输。

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemBz50EHfzfXHje06D4ialibWpK3bY8NOjYu0Wib9jFrqjMegtpGzufUK6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

看上去协议的格式和 HTTP1.x 完全不同了，但**实际上 HTTP2 并没有改变 HTTP1.x 的语义**，只是把原来 HTTP1.x

的 header 和 body 两部分用 **frame重新封装了** 而已 。![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemgYXqL5rTvMicTxd8wUSiaTAoH6Soj6Za0cXZoQXcuZvvibgdLTCRHVv6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP2 连接上**传输的每个帧都关联到一个“流”**。流是一个独立的，双向的帧序列可以通过一个 HTTP2 的连接在服

务端与客户端之间不断的交换数据。

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeembxke1sr7QjDiaaAl836pEscScKrReJ5KaTJoQISeqHdlh31lmcUBdEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

实际上运输时：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemdeqGb0YSr9qcdibzDJPPshSYJMl92kAGc6UGibDNGYmYHkXRKOibmE1Tw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP2还有一些比较重要的改动：

- 使用 HPACK 对 HTTP/2 头部压缩
- 服务器推送： <a href="https://segmentfault.com/a/1190000015773338">HTTP/2之服务器推送(Server Push)最佳实践</a>
- 流量控制 ：针对传输中的**流**进行控制(TCP默认的粒度是针对连接)
- 流优先级（Stream Priority）：它被用来告诉**对端哪个流更重要**。



总结：

HTTP1.1新改动：

- **持久连接**
- 请求管道化
- 增加缓存处理（新的字段如cache-control）
- 增加Host字段、支持断点传输等

HTTP2新改动：

- 二进制分帧
- **多路复用**
- 头部压缩
- 服务器推送



参考资料：

- HTTP2 GitBook电子书(中文版)：https://legacy.gitbook.com/book/ye11ow/http2-explained/details
- HTTP/2.0 相比1.0有哪些重大改进？https://www.zhihu.com/question/34074946
- HTTP/2 新特性浅析：https://segmentfault.com/a/1190000002765886
- HTTP2学习资料：https://imququ.com/post/http2-resource.html
- HTTP2简介和基于HTTP2的Web优化：http://caibaojian.com/toutiao/6641
- http2原理入门：https://blog.qingf.me/?p=600
- HTTP/2 对现在的网页访问，有什么大的优化呢？体现在什么地方？https://www.zhihu.com/question/24774343/answer/96586977
- HTTP/2笔记之流和多路复用：http://www.blogjava.net/yongboy/archive/2015/03/19/423611.aspx





### 四、HTTPS再次回顾

在将数HTTPS之前，首先还是来解释一下基础的东东：

- 对称加密：

- - 加密和解密都是用同一个密钥

- 非对称加密：

- - 加密用公开的密钥，解密用私钥
  - (私钥只有自己知道，公开的密钥大家都知道)

- 数字签名：

- - 验证传输的内容**是对方发送的数据**
  - 发送的数据**没有被篡改过**

- 数字证书（Certificate Authority）简称CA

- - 认证机构证明是**真实的服务器发送的数据**。

通讯之路的发展：

- 远古时代：在服务器与客户端的通信中，通信内容是直接传输

- - 内容被看得一清二楚，毫无隐私可言

- 上古时期：使用对称加密的方式来保证传输的数据只有双方知道

- - 但存在一个问题：**密钥不能通过网络传输** (因为没有加密之前，都是不安全的)，在通信之前必须要知道彼

    此的加密方式，用来解密。

- 中古时期：为了解决上古时期通信之间的问题，于是用到了非对称加密

- - 发送信息方使用公开的秘钥对数据加密，接受使用私有的秘钥对数据解密。

- 近代：由于有非对称性加密，解决多个机器的通信问题，但是由于黑客的技术过硬，加密后的数据有可能被黑

  客拦截，篡改数据之后，再发送给服务器，服务器的数据不完整，或者解析不了，无法做出响应。

- 现代：为了解决近代数据可能被篡改的问题，于是数字签名技术被提出来，用于解决数据被篡改的问题。数字

  签名其实也可以看做是**非对称加密的手段一种**，具体是这样的：得到原信息 hash值，用**私钥**对 hash值加密，

  另一端用**公钥**解密，最后比对hash值是否该变。如果变了就说明被篡改了。(一端用私钥加密，另一端用公钥

  解密，也确保了来源)

- 目前现在：好像使用了数字签名就万无一失了，但是还是问题。在对数据进行非对称加密的时候，是使用**公钥**

  **进行加密的**。如果**公钥被伪造了**，后面的数字签名其实就毫无意义了。讲到底：**还是可能会被中间人攻击**的。

  于是就有了**CA认证机构来确认公钥的真实性**！

  

对于数字签名和CA认证还是不太了解参考一下

- 阮一峰：http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html
- 什么是数字签名和证书？https://www.jianshu.com/p/9db57e761255



回到HTTPS，HTTPS其实就是在HTTP协议下多加了一层SSL协议 (ps:现在都用TLS协议了)![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemjskq9afcmX5udE6b1icyTTsXfwDYCn9iak2wZrbGeP8RD7zMibOHHrJVA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTPS采用的是**混合方式加密**：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeem6x7GmoTMHdMGOgNrgN9oGEhrfpz7a86voOlOOUIR4QtRlQ0uIsTcLQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

过程是这样子的：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemNnhjMvQj14geltKk2a0ZzVuqPT0tuiadkNNeA5VIfmAAuLBiaZhnwmIw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemTDsp4ibhWJgFId48D5wZy5wB4Eg6sTtYTXiaqeNd4QSiaHhDs5qIrrNZw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- 用户向web服务器发起一个安全连接的请求

- 服务器返回经过CA认证的数字证书，证书里面包含了服务器的public key(公钥)

- 用户拿到数字证书，用自己浏览器内置的CA证书解密得到服务器的public key

- 用户用服务器的public key加密一个用于接下来的对称加密算法的密钥，传给web服务器

- - 因为只有服务器有private key可以解密，所以**不用担心中间人拦截这个加密的密钥**

- 服务器拿到这个加密的密钥，解密获取密钥，再使用对称加密算法，和用户完成接下来的网络通信

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemNmf8reicdLwgElic2zl5bako8w83SJKIVsDePLSnic4wxdskjYibefWJYQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

所以相比HTTP，HTTPS 传输更加安全

- （1） 所有信息都是加密传播，黑客无法窃听。
- （2） 具有校验机制，一旦被篡改，通信双方会立刻发现。
- （3） 配备身份证书，防止身份被冒充。





参考资料：

- <a href="https://www.zhihu.com/question/52493697/answer/131015846">数字签名、数字证书、SSL、https是什么关系?</a>
- <a href="https://zhuanlan.zhihu.com/p/36981565">浅谈SSL/TLS工作原理</a>
- <a href=" https://tech.upyun.com/article/192/HTTPS%E7%B3%BB%E5%88%97%E5%B9%B2%E8%B4%A7%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9AHTTPS%20%E5%8E%9F%E7%90%86%E8%AF%A6%E8%A7%A3.html">HTTPS </a>
- <a href="https://www.cnblogs.com/powertoolsteam/p/http2https.html">网站HTTP升级HTTPS完全配置手册</a>