## Spring Cloud 篇章（二） 快速入门篇

在上一篇章中，讲述了 Spring Cloud 的概念和相关组件，此篇章将讲述 Spring Cloud 的入门。此篇章内容是在上

一篇章的基础上讲述的，详情请看 《<a href="https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/Spring%20Cloud%20%E7%AF%87%E7%AB%A0%EF%BC%88%E4%B8%80%EF%BC%89%E7%9F%A5%E8%AF%86%E6%A6%82%E5%BF%B5%E7%AF%87.md">Spring Cloud 篇章（一）知识概念篇</a>》



**1、创建 Spring Boot 的项目工程**

在IDEA下，创建 Spring Boot 项目工程，在选项中选中 Spring  Cloud Server ，然后创建项目。

![springcloud1](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/springcloud1.png)

项目初始化完之后，在*pom.xml* 文件中添加 eureka 的启动器，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.demo</groupId>
    <artifactId>server_eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>server_eureka</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <!--spring cloud 版本-->
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>

    <dependencies>
        <!--web 启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!--eureka -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        
        <!--测试启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <!--版本依赖配置-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!--插件-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <!--仓库-->
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

</project>
```

**2、配置 eureka 注册中心**

然后在 *resourece* 目录下，将 *application.properties* 文件改为 *application.yml* 文件，文件内容如下：

```yml
server:
  port: 8888

eureka:
  instance:
    hostname: localhost   #主机名
  client:
    register-with-eureka: false  # false 不注册到eureka（注册中心）
    fetch-registry: false     # false  表示此端就是注册中心，负责维护服务实例
    service-url:    #提供注册服务的路径
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

在程序入口类，加上 *@EnableEurekaServer* 注解，接受其他服务的注册。

```java
@SpringBootApplication
@EnableEurekaServer  //注册中心
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

启动程序，在浏览器输入 *localhost:8888*, 出现如下界面表示搭建服务注册中心成功。

![eureka 注册中心界面](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/eureka%20%E6%B3%A8%E5%86%8C%E4%B8%AD%E5%BF%83%E7%95%8C%E9%9D%A2.png)

好了，注册中心已经搭建好了，接下来就是服务的问题了。注册两个服务，用户与订单，用户服务通过注册中心访

问订单服务提供的服务。

![服务调用](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/%E6%9C%8D%E5%8A%A1%E8%B0%83%E7%94%A8.png)

**3、配置服务（user-service、order-service）**

如上述创建 eureka 步骤意向一样，创建 user-service 服务，然后配置 user-service的信息，注册到 eureka 中心

配置文件 *application.yml* 文件如下：

```yml
server:
  port: 8889  #端口号

spring:
  application:
    name:  User-Service   #注册到 eureka 的中注册名

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8888/eureka    #注册到eureka注册中心
  instance:
    prefer-ip-address: true  #显示IP地址
```

在程序入口类，加上 *@EnabledEurekaClient* ，表明该类是提供服务的类。

```java
@EnableEurekaClient  ///本服务启动后会自动注册进eureka服务中
@SpringBootApplication
@EnableDiscoveryClient  //服务发现
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }

}
```

添加一个控制类，编写一个测试方法，用于测试一下。

```java
@RestController
public class UserController {
    
    @GetMapping("/user/test")
    public String test(){
        return "successful deployment";
    }
}
```

启动注册中心后，在启动 user-service。在浏览器中访问注册中心和user-service，可以发现，user-service 成功

注册到注册中心，提示也可以在浏览器访问user-service的服务。

![user-service](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/user-service.png)

和创建  user-service 一样，创建 order-service。其配置文件如下：

* *pom.xml*

```xml
<!--eureka 服务端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-eureka-client</artifactId>
</dependency>

<!--eureka 客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-eureka-client</artifactId>
</dependency>
```



```yml
server:
  port: 8890  #端口号

spring:
  application:
    name: Order-Service   #注册到 eureka 的中注册名

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8888/eureka    #注册到eureka注册中心
  instance:
    prefer-ip-address: true  #显示IP地址
```

按顺序先启动注册中心之后，在启动另外两个服务，在浏览器访问，可以得到：

![order_service](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/order_service.png)



**4、访问服务 : Ribbon**

在上述的三个步骤中，已经将 user_service 和 order_service 注册到了 注册中心，接下就是如何访问注册中心的

服务了。接下来，将使用的是 Spring Cloud 提供的负载均衡器——Ribbon 来接口回调访问资源，其中发起的

Http请求访问是 RESTful 形式（基于资源的请求）。

* 在 user-service 的 *pom.xml* 文件，导入 Ribbon 的启动器。

```xml
<!--负载均衡启动器-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```

创建配置类 *Beans*.java

```java
@Configuration 
public class Beans {
    @Bean
    @LoadBalanced
    public RestTemplate restTemplateBean(){
        return new RestTemplate();  //默认使用轮序访问法
    }
}
```

 在 *UserController.java* 文件新加两个方法。

```java
@RestController
public class UserController {

    @Autowired
    private RestTemplate template;

    private static final String HTTP_URL="ORDER-SERVICE";  //请求服务名称

    @GetMapping("/user/test")
    public String test() {
        return "successful deployment";
    }

    @GetMapping("/user/{orderId}") //调用不成功，报404 错误，哎
    public String findOrderById(@PathVariable("orderId") long orderId){
        
        //template.getForObject( 请求路径, 期望的类型，参数 )  RestTemplate的使用
        String str = template.getForObject("http://"+ HTTP_URL+"/order",String.class,orderId);
        return str;
    }

    @GetMapping("/user/orders")
    public String getOrders(){
        return template.getForObject("http://"+HTTP_URL+"/orders",String.class);
    }
}
```

在 Order-Service 的 *OrderController.java* 新增两个对应方法。

```java
@RestController
public class OrderController {

    @GetMapping("/order/test")
    public String test(){
        return "order_service is successful deployment";
    }

    @GetMapping("/order/{id}")
    public String findOrderById(@PathVariable("id") long id){
        return "order_service successfully find the order by id :"+id;
    }

    @GetMapping("orders")
    public String getOrders(){
        return "orders";
    }
}
```

启动服务，在浏览器输入:  *localhost:8889/user/orders*

![Ribbon负载均衡](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/Ribbon%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1.png)

**5、熔断器 ： Hystrix**

在 user-service 的 *pom.xml* 文件，导入 hystrix 的启动器。

```xml
<!--熔断保护启动器-->
<dependency>    
    <groupId>org.springframework.cloud</groupId>    
    <artifactId>spring-cloud-starter-hystrix</artifactId>    
</dependency>
```

* 在 *UserController.java* 编写指定处理方法

```java
@RestController
public class UserController {

    @Autowired
    private RestTemplate template;

    private static final String HTTP_URL="ORDER-SERVICE";  //请求服务名称

    @GetMapping("/user/test")
    public String test() {
        return "successful deployment";
    }
    

    @HystrixCommand(fallbackMethod = "handlerError")  //指定处理的方法
    @GetMapping("/user/{orderId}")
    public String findOrderById(@PathVariable("orderId") long orderId){
        //template.getForObject( 请求路径, 期望的类型，参数 )  RestTemplate的使用
        String str = template.getForObject("http://"+ HTTP_URL+"/order",String.class,orderId);
        return str;
    }

    
    
    @GetMapping("/user/orders")
    public String getOrders(){
        return template.getForObject("http://"+HTTP_URL+"/orders",String.class);
    }

    //出错处理方法
    public String handlerError(){
        return "can not handle this request";
    }
}
```

* 然后在程序的入口指定类，加 *@Hystrix* 注解

```java
@EnableEurekaClient  ///本服务启动后会自动注册进eureka服务中
@SpringBootApplication
@EnableDiscoveryClient  //服务发现
@Hystrix  //服务熔断
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }

}
```



**6、使用整合 负载均衡 和 熔断机制 的 Fegin**

在 user-service 的 *pom.xml* 文件，把都导入的 Ribbon 和hystrix  两个启动器给注释掉，然后导入 Fegin 启动器

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <version>1.4.6.RELEASE</version>
</dependency>
```

然后创建一个 *UserService* 接口，在接口使用 *@FeginClient* 实现 http  请求。

```java
@Service
//value ：指定访问的服务，fallback ：指定访问出错交给哪个类处理
@FeignClient(value = "order-service",fallback = HandlerHystrix.class)
public interface UserService {

    @GetMapping("/orders")
    String orders();

}
```

*HandlerHystrix.java* 文件如下

```java
@Component
public class HandlerHystrix implements FallbackFactory<UserService> {
    @Override
    public UserService create(Throwable throwable) {
        return new UserService() {
            @Override
            public String orders() {
                return "can not handler this request";
            }
        };
    }
}
```

在 user-service 的 *application.yml* 文件中声明开启 Fegin

```yml
feign:  #开启熔断
  hystrix:
    enabled: true
```

启动服务之后，在浏览器输入  *localhost:8889/user/orders* 仍可以访问到。



**7、网关 Zuul**

和之前建立 user-service 一样，新建一个 modul ： zuul-service ，在 *pom.xml* 文件导入 网关启动器。

```xml
<!--网关启动器-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zuul</artifactId>
    <version>1.4.7.RELEASE</version>
</dependency>
```

编写配置文件 ： *application.yml* 

```yml
#注册进eureka
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8888/eureka/

#配置网关端口号
server:
  port: 8080
spring:
  application:
    name: zuul-server

#配置网关转发详情
zuul:
  routes:
    api-a:   #第一个请求路径
      path: /user/**
      service-id: User-Service
    api-b:   #第二个请求路径
      path: /order/**
      service-id: Order-Service
```

程序入口类

```java
@EnableZuulProxy //开启网关代理
@SpringBootApplication
public class ZuulServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZuulServiceApplication.class, args);
    }

}
```

启动服务之后，在浏览器输入  *localhost:8080/user/orders* 仍可以访问到。



最后贴一张项目工程目录图：

![项目工程图](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/%E9%A1%B9%E7%9B%AE%E5%B7%A5%E7%A8%8B%E5%9B%BE.png)

