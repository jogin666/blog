



## Spring Cloud 篇章（一）知识概念篇

> 前言：写在前面，本文转自 ：<a href="https://www.cnblogs.com/wmyskxz/p/10992686.html">你想了解的「SpringCloud」都在这里</a> 。在此篇博客之前，原作者有写过一篇
>
> [什么是微服务？](https://www.jianshu.com/p/5368af76a0f8)，有需要的，可以先去浏览，顺便帮原作者增加人气。



### 一、传统建构历史

> 部分引用自：[从架构演进的角度聊聊Spring Cloud都做了些什么？ - 纯洁的微笑](http://www.ityouknow.com/springcloud/2017/11/02/framework-and-springcloud.html)

**1.1、单体架构**

单体架构在微小企业比较常见，典型代表就是一个应用，一个数据库，一个web容器就可以跑起来。有两种常用情

况是可能选在单一架构的：① 在企业的发展初期，为了保证快速上线，采用开发单体应用比较简单且快速；② 是

传统奇特中垂直度比较高，访问压力较小的业务。此情况对技术要求比较低，单体应用的开发方便更层次人员接手

，而且也能满足需求。

下面是单体应用的架构图：

![img](https://upload-images.jianshu.io/upload_images/7896890-97c4c079986d5325.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在单体应用架构中，技术选型非常灵活，可以优先满足快速上线的要求，也便于快速跟进市场需求。



**1.2、垂直架构**

在单体应有架构发展一段时间后，公司的业务模式得到了认可，业务交易量也慢慢的大起来，这时候有些企业为了

应对更大的流量，就对原有的业务进行拆分，比如将单体应用拆分成：后台系统，前端系统，交易系统等。在这个

阶段，往往会将单体系统拆分为不同的层次，每个层次有对应的职责，UI层负责和用户交互，业务逻辑层负责具

体的业务功能，数据库层负责和业务层交互数据以及存储数据。

下面是垂直架构的架构图：

![img](https://upload-images.jianshu.io/upload_images/7896890-bed88a6d69bf4346.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**1.3、服务化架构**

如果公司进一步做大，垂直子系统会越来越多，系统和系统自检的调用关系呈指数上升的确实。在这样的背景下，

很多公司都会考录服务的SOA话化。**SOA代表面向服务的架构，将应有程序根据不同的职责划分为不同的模块**，不

同的模块直接通过特定的协议和接口进行交互。这样是整个系统切分成很多单个组件服务来完成请求，当流量过大

时通过水平扩张相应的组件来支撑，所有组件通过交互来满足整体的业务需求。SOA服务化的优点是：可以根据需

求通过网络对松散耦合的组粒度应用组件进行分布式部署，组合和使用。服务层是SOA的基础，可以被应用调用，

从而有效控制系统中与软件代理交互的认为依赖性。

服务化架构是一套松耦合的架构，服务的拆分原则是服务内部高内聚，服务之间低耦合。

下图是服务化架构图（SOA）：

![SOA服务架构](D:\Typora\projects\微服务、分布式\images\SOA服务架构.png)

在这个阶段可以使用 WebService 和 Dubbo 来服务治理。

可以发现从单体应用加厚到服务化架构，应用数量都在不断增加，慢慢下沉的旧城了基础组件，上浮的就成业务组

件。从上述也可以看出**架构的本质就是不断的拆分与重构** ：拆分就是把系统拆分成各个子系统/模块/组件，而拆分

首先需要姐姐每个组件的定位问题，然后才能划分边界，实现合理的拆分。合就是根据最终的要求，把各个分离的

组件有机的整合在一起。拆分的结果是开发人员能够做到业务聚焦，技能聚焦，实现开发的敏捷，合的结果就是让

系统编的柔性，可以因需求而变，实现业务敏捷。



**1.4、微服务架构**

**微服务是一种软件架构风格，它以专注于单一职责与小型功能区块为基础**，利用模块化的方式组合出复杂出的大型

应用程序，各功能区块使用与语言无关的API(例如 REST)集相互通讯，且每个服务可以被单独部署。微服务架构具

备以下三个核心特点：

* **微服务为大型系统而生**。随着业务的快速增长，系统的流量压力和复杂度也会快速上升。系统的可维护性和可

  拓展性成为架构设计的主要考虑因素，微服务架构设计理念通过拆分出单一职责的小型业务，和分而治的思想

  来实现复杂系统的优雅设计实现。

* **微服务架构是面向结果的**。微服务架构设计风格的产生并非是出自学术或者为标准而标准的设计的，而是在软

  件架构设计领域不断演进的过程中，为了解决实际工业界所遇到的问题而出现的架构设计风格。

* **专注于服务的可替代性来设计的**。微服务架构设计风格核心要解决的问题之一便是如何遍历的在大型系统中进

  行系统组件的维护和替换，且不影响整体系统的稳定性。

下面是基于SpringCloud的微幅架构图：

![img](https://upload-images.jianshu.io/upload_images/7896890-edd16c96b17343b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**1.5、微服务与SOA的区别在于：**

* 服务粒度不同：SOA服务是粗粒度的，微服务是更细维度的，小到一个子子模块，只要模块依赖的资源与其他

  模块没有关联，那么就可以单独成为一个微服务。

* 服务独立部署：每个服务都严格遵循独立打包部署的原则，互不影响。比如一台服务器可以部署多Docker实

  例，每个Docker实例可以部署一个微服务。

* 服务独立维护：m每个微服务都可以交给一个小团队甚至多个人来开发，测试，发布和运维，并对整个生命周

  期负责。

* 服务治理能力要求高：因为拆分为微服务之后，服务的数量变多，因此需要有一个统一的服务治理平台，来对

  各个服务进行管理。

  

### 二、引入 spring  Cloud

**2.1、什么是 Spring Cloud ？**

Spring 全家桶在 Java 开发中拥有举足轻重的地位，其中的一系列产品不仅仅大大简化和方便了 Java 的开发，其

中的 AOP 和 IOC 等一系列的理念也深刻地影响着 Java 程序员们。

Spring 全家桶产品众多，总结起来大概就是：

- **Spring** ： 通常指 Spring IOC。
- **Spring Framework**  ： 包含了 Spring IOC 和 Spring AOP，并实现与其它 J2EE 框架的整合。
- **Spring Boot** ： 是对 Spring Framework 的补充，简化搭建架构开发，致力于快速开发独立的 Spring 应用。
- **Spring Cloud**  ： 是基于 Spring Boot 设计的一套微服务规范，并增强了应用上下文。

官网的介绍：![img](https://upload-images.jianshu.io/upload_images/7896890-3267323e6d3b94bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**总结起来就是： Spring Cloud 是一系列框架的有序集合。**开发人员能够使用基于 Spring Boot 设计的 Spring 

Cloud 微服务框架，方便快速的搭建起一个可靠的、协调一致的分布式系统。



**2.2、为什么是 Spring Cloud？**

微服务的架构那么多，比如：Dubbo，Kubernetes，为什么就要使用 Spring Cloud 的呢？

* Spring Cloud 产出于 Spring 大家族，Spring 在企业级开发框架中无人匹敌，来头很大，可以保证后续的更

  新，完善。比如 Dubbo 现在就被淘汰了。

* 有 Spring Boot 这个独立干将可以省很多事，大大小小的活 Spring Boot 都搞的挺不错。

* 作为一个微服务治理的能手，考虑的很全面，几乎服务治理的方方面面都考虑到了，开箱即用，十分方便。

* Spring Cloud 活跃度很高，教程很丰富，遇到问题很容易找到解决方案。

* 轻轻松松几行代码就完成了熔断、均衡负载、服务中心的各种平台功能。



### 三、Spring Cloud 能够帮助开发人员做什么？

前面说到了，Spring Cloud 是一系列框架的集合，可以帮助开发人员解决分布式/微服务的各种问题，那么

Spring Cloud 究竟能帮助开发人员做什么呢？

**3.1、Spring Cloud 功能 **

Spring Cloud 基础功能：

* 服务治理 ：Spring Cloud Eureka
* 客户端负载均衡 ： Spring Cloud Ribbon
* 服务容错保护 ： Spring Cloud Hystrix
* 声明式服务调用 ：  Spring Cloud Feign
* API网关服务 ： Spring Cloud Zuul
* 分布式配置中心 ：  Spring Cloud Config

Spring Cloud 高级功能：

* 消息总线
* 消息驱动的微服务
* 分布式服务跟踪



**3.2、服务治理 ：Eureka**

微服务最重要的一点是就是 **无状态** ，也就是每个服务之间应该是独立的，所以当微服务架构搭建起来后，每个微

服务 之间的应该如何 **通讯** 成了首要目标。假设有服务A和服务B，服务A需要访问服务B的服务,在发起服务之前，

首先需要知道对方的 **IP地址**，才能访问对方提供的服务。如下：

![img](https://upload-images.jianshu.io/upload_images/7896890-d736e62854e9f949.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看起来，似乎并没有问题，但是如果 服务B 的 **IP地址** 变更之后，name只能手动去修改 服务A 的配置。一两个还

好，但是微服务如此之多，如果每个服务的 **IP地址** 变动之后，都要手动去更改，那效率是何其之低啊。于是 

Eureka 出现了，Eureka 是 Netflix 开源的一款提供服务注册和发现的产品，提供了完整的 Service Registry（服

务注册） 和 Service Discovery （服务发现）等功能，也是 Spring Cloud 体系中最重要最核心的组件之一。用大

白话讲，Eureka 就是一个服务中心，将所有可以提供的服务都注册到Eureka，实现管理，其它调用者需要服务时

就来注册中心（Eureka）获取，然后再进行调用，避免了服务之间的直接调用，方便后续的水平扩展、故障转移

等。如下图：

![img](https://upload-images.jianshu.io/upload_images/7896890-44cc612bd6a8a064.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然服务中心这么重要的组件一但挂掉将会影响全部服务，因此需要搭建 Eureka 集群来保持高可用性，生产中建

议最少两台。随着系统的流量不断增加，需要根据情况来扩展某个服务，Eureka 内部已经提供均衡负载的功能，

只需要增加相应的服务端实例既可。那么在系统的运行期间某个实例挂了怎么办？**Eureka 内容有一个心跳检测机**

**制，** 如果某个实例在规定的时间内没有进行通讯则会自动被剔除掉，避免了某个实例挂掉而影响服务。

> 因此使用了Eureka就自动具有了注册中心、负载均衡、故障转移的功能。如果想对Eureka进一步了解可以参
>
> 考这篇文章：[注册中心Eureka](http://www.ityouknow.com/springcloud/2017/05/10/springcloud-eureka.html)



**3.3、客户端负载均衡 ：Ribbon**

Ribbon 是一个基于 HTTP 和 TCP 客户端的负载均衡器。Ribben 可以通过在客户端中配置 RibbonServerList（服

务列表）去轮序访问提供服务的服务者，已达到负载均衡的作用。当 Ribbon 与 Eureka 联合使用时，

RibbonServerList（服务列表）会被 DiscoveryEnabledNIWSServerList 重写，扩展成**从 Eureka（注册中心）

中获取提供服务的服务者列表，同时也会使用 NIWSDiscoveryPing 来取代 Ping，其职责将由 Eureka（注册中

心）来确定服务端时候已经启动。

> 实战参考：<a href="http://blog.didispace.com/springcloud2/">Spring Cloud构建微服务架构（二）服务消费者</a> 



**3.4、服务容错保护 ： Hystrix**

在微服务架构中通常会有多个服务相互之间调用，基础服务的故障可能会导致级联故障，进而造成整个系统不可用

的情况，这种现象被称为 **服务雪崩效应。**服务雪崩效应是一种因 **服务提供者** 的不可用导致 **服务消费者** 的不可用,

并将不可用逐渐放大的过程。

如下图所示：A作为服务提供者，B为A的服务消费者，C和D是B的服务消费者。A不可用引起了B的不可用，并将

不可用像滚雪球一样放大到C和D时，雪崩效应就形成了。

如下图所示：A作为服务提供者，B为A的服务消费者，C和D是B的服务消费者。A不可用引起了B的不可用，并将

不可用像滚雪球一样放大到C和D时，雪崩效应就形成了。

![img](https://upload-images.jianshu.io/upload_images/7896890-2c62b2ffa85e86e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为防止上述情况出现，就需要整个服务机构具有 **故障隔离** 的功能，避免某一个服务挂掉而影响全局。在 Spring 

Cloud 中 Hystrix 提供 故障处理的功能。Hystrix  会在某个服务在连续调用 N 次不响应的情况下，立即改制调用

者调用失败，避免了调用者持续等待而影响整体的服务。Hystrix 也会在一定的时间间隔后会再次检查此服务，如

果服务恢复，则继续让该服务提供服务。

>  继续了解 Hystrix 可以参考：[熔断器Hystrix](http://www.ityouknow.com/springcloud/2017/05/10/springcloud-eureka.html)



**3.5、Hystrix 监控工具：Hystrix Dashboard 和 Turbine**

当熔断发生的后，程序需要迅速的响应以解决问题。避免故障进一步扩散，那么对熔断的监控就变得非常重要。熔

断的监控现在有两款工具：Hystrix-dashboard 和 Turbine。

Hystrix  DashBoard 是一款针对 Hystrix 进行实时监控的工具，通过 Hystrix Dashboard ，开发人员可以直观地

看到 Hystrix Command 的请求相应时间，请求成功率等数据。但是使用 Hystrix Dashborad  只能看到单个应用

内的服务信息，这明显是不够的。因此 Turbine 出现了，搭配 Hystrix Bashboard 使用，可以汇总系统内多个服

务的数据，并显示出来。效果图如下：

![img](https://upload-images.jianshu.io/upload_images/7896890-de80387262f335c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



**3.6、声明式服务调用：Fegin**

上面已经介绍了 Ribbon 和 Hystrix 了。可以发现：微服务的调用，需要 Rabbon 和 Hystrix 的搭配使用。为了**简**

**化开发**。于是 Spring Cloud Feign 被了出来，它基于 Netflix Feign 实现，整合了 Spring Cloud Ribbon 与 Spring 

Cloud Hystrix 。Fegin  除了整合这两者的强大功能之外，还提供了 **声明式的服务调用 **(不再通过RestTemplate)。

> Feign 是一种声明式、模板化的伪HTTP客户端。在 Spring Cloud 中使用 Feign, 开发人员可以做到使用HTTP
>
> 请求远程服务时能与调用本地方法一样的编码体验，开发者完全感知不到这是远程方法，更感知不到这是个 
>
> HTTP 请求。

下面就简单看看Feign是怎么优雅地实现远程调用的：

* 服务绑定

```java
// value --->指定请求注册中的服务
// fallbackFactory--->熔断器的降级提示
@FeignClient(value = "MICROSERVICECLOUD-DEPT",
             fallbackFactory = DeptClientServiceFallbackFactory.class)
public interface DeptClientService {

    // 采用Feign 开发人员可以使用SpringMVC的注解来对服务进行绑定！
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") long id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();

    @RequestMapping(value = "/dept/add", method = RequestMethod.POST)
    public boolean add(Dept dept);
}
```

* Feign 中使用熔断器

```java
/**
 * Feign中使用断路器
 * 这里主要是处理异常出错的情况(降级/熔断时服务不可用，fallback就会找到这里来)
 */
@Component // 不要忘记添加，不要忘记添加
public class DeptClientServiceFallbackFactory implements FallbackFactory<DeptClientService> {
    
    @Override
    public DeptClientService create(Throwable throwable) {
        return new DeptClientService() {
            @Override
            public Dept get(long id) {
                return new Dept().setDeptno(id).setDname("该ID：" + id + "没有没有对应的信"
                        +"息,Consumer客户端提供的降级信息,此刻服务Provider已经关闭")
                        .setDb_source("no this database in MySQL");
            }

            @Override
            public List<Dept> list() {
                return null;
            }

            @Override
            public boolean add(Dept dept) {
                return false;
            }
        };
    }
}
```

* 调用

![img](https://upload-images.jianshu.io/upload_images/7896890-151920f51a61863b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 引用自：<a href="https://juejin.im/post/5b83466b6fb9a019b421cecc#heading-12">外行人都能看懂的 Spring Cloud</a>



**3.7、API 网关服务 ：Zuul**

在微服务架构模式下，**后端服务的是实例数是动态的**，对于客户端而言很难发现动态改变的服务实例的访问地址信

息（服务者的注册名发生变化）。在基于微服务的项目中为了简化前端的调用逻辑，通常会引入 API Gateway 作

为轻量级网关，同时 API Gateway 中也会实现相关的认证逻辑（权限控制）从而简化内部服务之间相互调用的复

杂度。![img](https://upload-images.jianshu.io/upload_images/7896890-e3e021be95edbf77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 Spring Cloud 体系中支持 API Gateway 落地的技术就是 Zuul。Spring Cloud Zuul 路由是微服务架构中不可或

缺的一部分，其提供动态路由，监控，弹性，安全等的边缘服务。Zuul 是 Netflix 出品的一个基于 JVM 路由和服

务端的负载均衡器。其具体作用就是 **服务转发，接收并转发所有内外部的客户端调用**。使用 Zuul 可以作为资源的

统一访问入口，同时也可以在网关做一些权限校验等类似的功能。

> 具体使用请参考这边文章：[服务网关zuul](http://www.ityouknow.com/springcloud/2017/06/01/gateway-service-zuul.html)



**3.8、分布式配置中心 ：Config**

随着业务的不断发展，系统的 微服务 可能会越来越多，而**每一个微服务都会有自己的配置文件，**在研发过程中有

开发环境、测试环境、生产环境，因此每个微服务又会有对应至少三个不同环境的配置文件。这么多的配置文件，

如果需要修改某个公共服务的配置信息（或者服务集群所需的配置文件），如：缓存、数据库等，难免会产生混

乱，这个时候就需要引入 Spring Cloud 另外一个组件：Spring Cloud Config（文件配置中心）。

* Config 

  **Spring Cloud Config 是一个解决分布式系统的配置管理方案**，包含了 Client 和 Server 两个部分，Server 提

  供配置文件的储存，以接口的形式将配置文件的内容提供出去，Client 通过接口获取数据，并依据此数据初始

  化本身。其实就是 Server 端将所有的配置文件服务化，需要配置文件的服务实例去 Config Server 获取对应

  的数据。说白了就是 **将所有的配置文件统一整理，避免了配置文件碎片化**。 

  配置中心git实例参考：[配置中心git示例](http://www.ityouknow.com/springcloud/2017/05/22/springcloud-config-git.html)；

* Refresh

  如果服务运行期间改变配置文件，服务是不会得到最新的配置信息，需要解决这个问题就需要引入 Refresh。

  可以在服务的运行期间重新加载配置文件。 具体可以参考这篇文章：[配置中心svn示例和refresh](http://www.ityouknow.com/springcloud/2017/05/23/springcloud-config-svn-refresh.html)

* 配置中心集群

  当所有的配置文件都存储在配置中心的时候，配置中心就成为了一个非常重要的组件。**如果配置中心出现问题**

  **将会导致灾难性的后果，因此在生产中建议对配置中心做集群，来支持配置中心高可用性**。

  具体参考：[配置中心服务化和高可用](http://www.ityouknow.com/springcloud/2017/05/25/springcloud-config-eureka.html)



**3.9、消息总线：Bus**

上面的 Refresh 方案虽然可以解决单个微服务运行期间重载配置信息的问题，但是在真正的实践生产中，可能会

有 N 多的服务需要更新配置，如果每次依靠手动 Refresh 将是一个巨大的工作量，这时候 Spring Cloud 提出了另

外一个解决方案：Spring Cloud Bus

**Spring Cloud Bus 通过轻量消息代理连接各个分布的节点。**这会用在**广播状态**的变化（例如配置变化）或者其它

的消息指令中。Spring Cloud Bus 的一个核心思想是**通过分布式的启动器对Spring Boot应用进行扩展，也可以用**

**来建立一个或多个应用之间的通信频道。**目前唯一实现的方式是用 AMQP 消息代理作为通道。

Spring Cloud Bus 是轻量级的通讯组件，也可以用在其它类似的场景中。有了 Spring Cloud Bus 之后，当开发人

员改变配置文件提交到版本库中时，会自动的触发对应实例的 Refresh，具体的工作流程如下：

![img](https://upload-images.jianshu.io/upload_images/7896890-ff35d8cb86a33733.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也可以参考这篇文章来了解：[配置中心和消息总线](http://www.ityouknow.com/springcloud/2017/05/26/springcloud-config-eureka-bus.html)



**3.10、消息驱动的微服务：Stream**

Spring Cloud Stream 是一个用来为微服务应用构建消息驱动能力的框架。它可以基于 Spring Boot 来创建独立的

可用于生产的 Spring 应用程序。它通过使用 Spring Integration 来**连接消息代理中间件以实现消息事件驱动的微**

**服务应用。**

下图是官方文档中对于 Spring Cloud Stream 应用模型的结构图。从图中可以看到，Spring Cloud Stream 构建的

应用程序与消息中间件之间是通过绑定器 Binder 相关联的，绑定器对于应用程序而言起到了隔离作用，它使得不

同消息中间件的实现细节对应用程序来说是透明的。所以对于每一个 Spring Cloud Stream 的应用程序来说，它

不需要知晓消息中间件的通信细节，它只需要知道 Binder 对应用程序提供的概念去实现即可。如下图案例，在应

用程序和 Binder 之间定义了两条输入通道和三条输出通道来传递消息，而绑定器则是作为这些通道和消息中间件

之间的桥梁进行通信。

![img](https://upload-images.jianshu.io/upload_images/7896890-07f3ff1255dfc531.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Spring Cloud Stream 为一些供应商的消息中间件产品提供了个性化的自动化配置实现，并且引入了发布-订阅、

消费组以及消息分区这三个核心概念。简单的说，**Spring Cloud Stream 本质上就是整合了 Spring Boot 和 **

**Spring Integration，实现了一套轻量级的消息驱动的微服务框架。**通过使用 Spring Cloud Stream，可以有效地

简化开发人员对消息中间件的使用复杂度，让系统开发人员可以有更多的精力关注于核心业务逻辑的处理。由于 

Spring Cloud Stream 基于 Spring Boot 实现，所以它秉承了 Spring Boot 的优点，实现了自动化配置的功能帮忙

我们可以快速的上手使用，但是目前为止 Spring Cloud Stream 只支持 **RabbitMQ** 和 **Kafka** 两个著名的消息中

间件的自动化配置：

- 实战：<a href="http://blog.didispace.com/spring-cloud-starter-dalston-7-1/">Spring Cloud构建微服务架构：消息驱动的微服务（入门）【Dalston版】</a>



**3.10、分布式服务跟踪：Sleuth**

随着服务的越来越多，对调用链的分析会越来越复杂，如服务之间的调用关系、某个请求对应的调用链、调用之间

消费的时间等，**对这些信息进行监控就成为一个问题。**在实际的使用中我们需要监控服务和服务之间通讯的各项指

标，这些数据将是我们改进系统架构的主要依据。因此分布式的链路跟踪就变的非常重要，Spring Cloud 也给出

了具体的解决方案：Spring Cloud Sleuth 和 Zipkin

![img](https://upload-images.jianshu.io/upload_images/7896890-c68c2ef12cd4adae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Spring Cloud Sleuth 为服务之间调用提供链路追踪。通过 Sleuth 可以很清楚的了解到一个服务请求经过了哪些服

务，每个服务处理花费了多长时间。从而让我们可以很方便的理清各微服务间的调用关系。

Zipkin 是 Twitter 的一个开源项目，允许开发者收集 Twitter 各个服务上的监控数据，并提供查询接口

分布式链路跟踪需要 Sleuth + Zipkin 结合来实现，具体操作参考这篇文章：[分布式链路跟踪(Sleuth)](http://www.jianshu.com/p/c3d191663279)



**3.12总结**

从整体上来看一下Spring Cloud各个组件如何来配套使用：

![img](https://upload-images.jianshu.io/upload_images/7896890-eda069329a8b386d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从上图可以看出 Spring Cloud 各个组件相互配合，合作支持了一套完整的微服务架构。

- 其中 **Eureka** 负责服务的注册与发现，很好将各服务连接起来
- **Hystrix** 负责监控服务之间的调用情况，连续多次失败进行熔断保护。
- **Hystrix dashboard,Turbine** 负责监控 Hystrix 的熔断情况，并给予图形化的展示
- **Spring Cloud Config** 提供了统一的配置中心服务
- 当配置文件发生变化的时候，**Spring Cloud Bus** 负责通知各服务去获取最新的配置信息
- 所有对外的请求和服务，我们都通过 **Zuul** 来进行转发，起到 API 网关的作用
- 最后我们使用 **Sleuth + Zipkin** 将所有的请求数据记录下来，方便我们进行后续分析

Spring Cloud 从设计之初就考虑了绝大多数互联网公司架构演化所需的功能，如服务发现注册、配置中心、消息

总线、负载均衡、断路器、数据监控等。这些功能都是以插拔的形式提供出来，方便在系统架构演进的过程中，可

以合理的选择需要的组件进行集成，从而在架构演进的过程中会更加平滑、顺利。

微服务架构是一种趋势，Spring Cloud 提供了标准化的、全站式的技术方案，意义可能会堪比当前 Servlet 规范的

诞生，有效推进服务端软件系统技术水平的进步。

> 引用自：<a href="http://www.ityouknow.com/springcloud/2017/11/02/framework-and-springcloud.html">从架构演进的角度聊聊Spring Cloud都做了些什么？</a> 



### 四、Spring Cloud 版本

刚接触的「Spring Cloud」的童鞋可能会对它的版本感到奇怪，什么 `Angle`、`Brixton`、`Finchley`，这些都是

啥啊？「为什么会有这么多种看起来不同的 Spring Cloud？」

从上面讲述的内容可以知道：**Spring Cloud 是一个拥有诸多子项目的大型综合项目**（功能不止上面的介绍），原

则上其子项目也都维护着自己的发布版本号。那么每一个Spring Cloud的版本都会包含不同的子项目版本，**为了要**

**管理每个版本的子项目清单，避免版本名与子项目的发布号混淆，所以没有采用版本号的方式，而是通过命名的方式。**

这些版本名字采用了伦敦地铁站的名字，**根据字母表的顺序来对应版本时间顺序，**比如：最早的Release版本：

Angel，第二个Release版本：Brixton，以此类推……

当一个项目到达发布临界点或者解决了一个严重的 BUG 后就会发布一个 "service Release" 版本， 简称 SR（X）

版本，x 代表一个递增数字。

> 引用自：<a href="http://blog.didispace.com/springcloud-version/">聊聊Spring Cloud版本的那些事儿</a>



**Spring Cloud & Spring Boot 版本对照表**

通过查阅<a href="https://spring.io/projects/spring-cloud">官网</a>，可以看到一个「Release train Spring Boot compatibility」表：

| **Release Train** | **Boot Version** |
| :---------------- | :--------------- |
| Greenwich         | 2.1.x            |
| Finchley          | 2.0.x            |
| Edgware           | 1.5.x            |
| Dalston           | 1.5.x            |

上表可以看出，最新的「Spring Cloud」版本已经出到了 Greenwich... 每个版本都能查阅到当前版本所包含的子

项目，以及子项目的版本号，我们可以通过此来决定需要选择怎么样的版本。





参考资料：

① <a href="https://juejin.im/post/5b83466b6fb9a019b421cecc#heading-19">外行人都能看懂的SpringCloud，错过了血亏！</a>

②<a href="http://www.ityouknow.com/springcloud/2017/11/02/framework-and-springcloud.html">从架构演进的角度聊聊Spring Cloud都做了些什么？</a> 

③<a href="http://blog.didispace.com/springcloud-version/">聊聊Spring Cloud版本的那些事儿</a>

④<a href="http://blog.didispace.com/spring-cloud-learning/">Spring Cloud 从入门到精通</a>

⑤<a href="https://www.springcloud.cc/">Spring Cloud 中文网</a>