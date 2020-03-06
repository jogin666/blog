## springmvc 篇章（一）入门篇

#### 1、前言

SpringMVC 是 Spring 家族的一员，Spring是将现在开发中流行的组件进行组合而成的一个框架！它用在基于MVC 的表现层开发。在刚开始学习 JavaWeb 的时候，相信大家都学过基于 Servlet + JSP + Java Bean 的 MVC 架构吧 

![mvc 架构图](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/mvc%20%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

在上图的 MVC 模式中，用户发起的请求会先到达 控制层 Servlet ，然后根据请求进行业务处理，然后到达数据层，将数据库的数据记录封装成对象的实体类，然后在视图层将数据展示出来，这种模式就叫做 mvc 模式。

* mvc 中的 m 代表模型（Model），在数据获取层中将数据封装成实体类。
* mvc 中的 v 代表视图（View），在控制获取处理好的数据之后，将数据展示。
* mvc 中的 c 代表控制器，处理用户请求，把处理好的数据显示到视图上。

#### 2、spring mvc 架构

**2.1 架构**

spring 为了解决持久层中一直没有处理好的数据库事务的编程，同时为了迎合 NoSql 的强势崛起，spring 在 web 模块中 spring mvc 给出解决方案：

![spring mvc 架构](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/spring%20mvc%20%E6%9E%B6%E6%9E%84.png)

​																			（图片来源参考资料）

传统的模型层被拆分为了业务层(Service)和数据访问层（DAO,Data Access Object）。在 Service 层可以通过 Spring 的声明式事务操作数据访问层的数据库操作，同时在 Service 层上还允许开发人员访问 NoSQL ，这样不仅能够满足异军突起的 NoSQL 的使用，也大提高互联网系统的性能。

**2.2、特点：**

* 结构松散，Spring MVC 架构支持使用多种视图
* 松耦合，各个模块分离
* 与 Spring 无缝集成



#### 3、springmvc 第一个程序

了解了 springmvc 的架构，那么现在开始 springmvc 的 hello 吧，使用 Idea 开发工具，创建一个 maven 工程，在 *pom.xml* 文件中，导入 springmvc 的依赖。

```xml
<dependencies>
    <!--spring 的核心依赖包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>

    <!--spring webmvc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>

    <!--spring web-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>

    <!--spring web通信-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-websocket</artifactId>
        <version>5.1.8.RELEASE</version>
    </dependency>

    <!--spring web mvc -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc-portlet</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>

    <!--request and response-->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
    </dependency>
</dependencies>
```

配置 web.xml 文件

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 	"http://java.sun.com/dtd/web-app_2_3.dtd" >
<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <!--上文参数，设置 spring 配置文件的位置-->
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:application.xml</param-value>
  </context-param>

  <!--监听 springmvc 上下文的加载器-->
  <listener>
    <listener-class>org.springframework.web.context.ContextLoader</listener-class>
  </listener>

  <!--配置中心 servlet：请求转发器-->
  <servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--初始化参数，加载 springmvc 配置文件-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:dispatcher-servlet.xml</param-value>
    </init-param>
    <!--设置启动级别-->
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <!--拦截所有请求吧-->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

编写 controller 类（控制器），处理请求。

```java
//实现 springmvc 的 Controller 接口，处理用户发起的请求
public class HelloController implements Controller {
    
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, 
                         HttpServletResponse httpServletResponse) throws Exception {
        /*ModelAndView 模型视图类
          指定处理完数据后，负责展示数据的视图
         */
        ModelAndView mav = new ModelAndView("view.jsp");
        //添加所要展示的数据
        mav.addObject("message","Hello World——springmvc");
        return mav;
    }
}
```

在 springmvc  的配置文件中配置 处理请求映射 的 bean。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置请求映射处理器-->
    <bean id="simpleUrlHandlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        <property name="mappings">
            <props>
                <!--key：为请求路径  value 为：controller 类的 bean 名，负责处理请求 -->
                <prop key="/hello">helloController</prop>
            </props>
        </property>
    </bean>

    <!--配置控制器的 bean-->
    <bean id="helloController" class="com.zy.springmvc.controller.HelloController"/>

</beans>
```

编写视图（view.jsp）

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" isELIgnored="false"%>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h2>12${message}</h2>
</body>
</html>
```

启动本地服务，其结果：

![springmvc：hello](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/springmvc%EF%BC%9Ahello.png)

可能会遇到的问题：

①：找不到 *javax.servlet.http.HttpServletResponse* 和 *javax.servlet.http.HttpServletRequest*

​	解决办法：在 pom.xml 文件中导入相关依赖包

②：view.jsp 只显示 ${messsage}

​	解决办法：在 view.jsp wen 文件的文件声明中，写入 *isELIgnored="false"*

#### 4、springmvc 的相应过程

通过第三部分的介绍，已经可以成功的运行了 demo了，接下来就是介绍 springmvc 的相应过程，下图就是 springmvc 在相应用户请求时的流程：

![springmvc 相应过程](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/springmvc%20%E7%9B%B8%E5%BA%94%E8%BF%87%E7%A8%8B.png)

​																			（图片来源参考资料）

**4.1 DispatcherServlet**

当用户在浏览器发起请求后，请求首先会来到在 *web.xml* 文件中配置的请求分发器，在配置中，本人设置是拦截所有的请求，然后将拦截到请求的 url 发送给 springmvc 的 url 映射处理器。

```xml
<!--配置 servlet 请求转发器-->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--初始化参数，加载 springmvc 配置文件-->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:dispatcher-servlet.xml</param-value>
    </init-param>
    <!--设置启动级别-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <!--拦截所有请求吧-->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

**4.2、映射处理器**

映射处理器接受到 DispatcherServlet 发来的 url 后，其会根据请求所携带的 URL 信息来进行决策，将该请求根据在其配置中映射关系，交给相应的控制器处理请求，因为在一个应用中，可定会有多个控制器，一个模块对应一个控制器。其实映射的公用就是：将什么样的请求，交给什么样的控制器来处理请求。

```xml
<!--配置请求映射处理器-->
<bean id="simpleUrlHandlerMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <!--key：为请求路径  value 为：controller 类处理请求 -->
            <prop key="/hello">helloController</prop>
        </props>
    </property>
</bean>

<!--配置控制器的 bean-->
<bean id="helloController" class="com.zy.springmvc.controller.HelloController"/>
```

【映射处理器默认是不可配置的】，只需要在对应的控制器 bean 的属性 name 指定相应的请求映射就可以了。

```xml
<!--配置控制器的 bean-->
<bean id="helloController" name="/hello" 
      class="com.zy.springmvc.controller.HelloController"/>
```

> 注：映射处理器在将请求交给指定的 控制器类 来处理请求的之前，核心控制器（DispatcherServlet）会使用适配器去找该控制类类是否实现了Controller接口（或者是否是使用了 @Controller 注解）【默认可省略的】
>
> 也就是说：适配器的作用：就是去找确认控制类是否是合法的 Controller 接口的类。
>
> ```java
>  <!-- 适配器【可省略】 -->    <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean>
> ```

**4.3、控制器**

在映射处理中选择好了相应的控制器后， DispatcherServlet 会将请求发送给选中的控制器，到了控制器，请求会卸下其负载（用户提交的请求）等待控制器处理完这些信息，并返回一个 ModelAndView 类的实例，该实例包处理的数据和指定显示数据的逻辑视图；

```java
//实现 springmvc 的 Controller 接口，处理用户发起的请求
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) throws Exception {
        /*模型视图类
          指定处理完数据后，负责展示数据的视图
         */
        ModelAndView mav = new ModelAndView("view.jsp");
        //添加所要展示的数据
        mav.addObject("message","Hello World——springmvc");
        return mav;
    }
}
```

**4.4 视图解析器**

DispatcherServlet 得到控制器返回的 ModelAndView 的实例后，会将该实例交给视图解释器来解析处理，然后在视图渲染结果，这样以来，控制器就不会和特定的视图相耦合。注意传递给 DispatcherServlet 的视图名并不一定表示某个特定的 jsp/html。它可以传递的仅仅是一个逻辑名称，这个名称将会用来查找产生结果的真正视图。因为 DispatcherServlet 将会使用视图解析器（view resolver）为逻辑视图名匹配一个特定的视图实现【 对于视图解析器，如果使用的是绝对的真实路径，那默认是可以省略配置视图解析器，因为springmvc 有默认的实现，但如果使用的是逻辑路径，那么就必须对其配置视图解释器，否则SpringMVC是找不到对应的路径的 】

**4.5、视图**

以上过程后， DispatcherServlet 已经知道由哪个视图渲染结果了，那请求的任务基本上也就完成了。在视图将模型中的数据渲染出结果，这个输出结果会通过响应对象传递给客户端。

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8" isELIgnored="false"%>
<html>
<head>
    <title>Title</title>
</head>
<body>
<h2>12${message}</h2>
</body>
</html>
```

#### 5、控制器

想一下，springmvc 是如接受 web 端传来的参数的？学过 struts2 的同学，struts2 的控制类是多例的，对于接受web 端的参数，只需要在控制类上写对应的成员变量，给出对应的set和get方法，struts2就会把参数封装到对应的成员变量中，是非常方便的。而且因为是多例的，不会出现数据的非法性。

而 springmvc 的控制器是单例的，是不可能使用成员变量来进行接收的【因为会有多个用户访问，就会出现数据不合理性】，因此 springmvc 只能通过方法的参数来进行接收对应的参数，这样才能保证不同的用户对应不同的数据！

对于 springmvc ，如果仅仅是跳转到某个视图上，是可以省略控制器和业务方法。只需在配置文件中，配置的控制器继承于 *ParameterizableViewController* 类，同时将 name 属性配置为 请求资源的路径就可以了。

```xml
<bean id="helloController2" name="/hello" 
      class="org.springframework.web.servlet.mvc.ParameterizableViewController">
    <!--转到真实的视图-->
    <property name="viewName" value="view.jsp"/>
</bean>
```

springmvc 篇章（一）入门篇完结。



参考资料：

①：<a href="https://www.cnblogs.com/wmyskxz/p/8848461.html">Spring MVC【入门】就这一篇！</a>

②：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483973&idx=1&sn=dda2252f37e5eb6db90db636a65c40bf&chksm=ebd74344dca0ca52d671fc0fa072bcc80892bfb5801ceaab6a4754036d246f5bef960c1840bd&scene=21###wechat_redirect">SpringMVC入门就这么简单</a>