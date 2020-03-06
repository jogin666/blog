## Spring篇章（一）入门篇



### 一、认识 Spring 框架

![img](https://upload-images.jianshu.io/upload_images/7896890-34e6864b15c793ec.png?imageMogr2/auto-orient/strip|imageView2/2/w/773/format/webp)

**1.1 、认识 Spring框架**

Spring 被创建的目的是用来代替更加重量级的企业级 Java 技术，Spring 现在是使用最广泛的框架，其**成功来源于**

**理念，而不是技术本身**，其理念包括 **IoC (Inversion of Control，控制反转)** 和 **AOP(Aspect Oriented **

**Programming，面向切面编程)**。



**1.2 、什么是 Spring**

- Spring 是一个**轻量级的 DI / IoC 和 AOP 容器的开源框架**
- Spring 是一个**轻量级的 DI / IoC 和 AOP 容器的开源框架**，可以随时安装与卸用



**1.3、Spring 的特点**

- 基于 POJO 轻量级和最小侵入式开发
- 通过依赖注入和面向接口实现松耦合
- 基于切面和惯例进行声明式编程
- 通过切面和模板减少样板式代码 
- 方便集成其他框架（如MyBatis、Hibernate）
- 降低 Java 开发难度



**1.4、术语介绍**

- 侵入式概念

  - 侵入式 ：是指需要继承框架中特定的类或者实现特定的接口，才能增强该类的功能。（如 Struts2 框架）
  - 非侵入：无需继承框架提供的任何类，就能够增强类的功能。（如：Mybatis框架）

- 轻量级和重量级

  轻量级是相对于重量级而言的，**轻量级一般就是非入侵性的、所依赖的东西非常少、资源占用非常少、部署简**

  **单等等**，其实就是**比较容易使用**，而**重量级正好相反**。

- 容器：

  在日常生活中容器就是一种盛放东西的器具，从程序设计角度看就是**装对象的的对象**，因为存在**放入、拿出等**

  操作，所以容器还要**管理对象的生命周期**。



**1.5、Spring 可以帮开发人员干啥**

- Spring  能根据配置文件帮助开发人员创建及组装对象之间的依赖关系。

- Spring  的面向切面编程能帮助开发人员 无耦合的实现日志记录，性能统计，安全控制。

- Spring   能非常简单的帮开发人员管理数据库事务。

- Spring   集成了第三方数据访问框架（如Hibernate、JPA）无缝集成，同时也提供一套 JDBC 访问模板来实现

  数据库访问操作。

- Spring  集成了第三方Web（如Struts1/2、JSF）框架，也提供了一套 Spring MVC框架，来方便 web 层搭建

- Spring  容易整合第三方框架。例如 Java EE（如Java Mail、任务调度），缓存框架等等。



### 二、Spring 的 IOC （控制反转）和 DI（依赖注入）

**2.1、IOC （控制反转）**

IOC 不是什么技术，而是一种**设计思想**，就是**将原本在程序中手动创建对象的控制权，交由Spring框架来管理。**

也就是类的实例，不再由开发人员类中创建，管理其声明周期，而是由 Spring 来创建，管理对象的声明周期。在

哪个类中使用到其他的类的实例，只需要在配置文件中指定，或者使用注解注入，坐享其成。例如 ：午饭不想自

己动手做，就把午饭的任务交给外卖平台，在平台中订单，就可以坐享其成。控制反转是通过外部容器完成的，**而**

**Spring 提供了一个容器用来装载实体类的实例，一般将这个容器叫做：IOC容器.**



如果还是不太了解IOC控制反转的作用的话，来看一下别人优秀的回答：

> ioc的思想最核心的地方在于，资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很
>
> 多好处。**第一，资源集中管理，实现资源的可配置和易管理**。**第二，降低了使用资源双方的依赖程度，也就**
>
> **是我们说的耦合度**。
>
> 也就是说，甲方要达成某种目的不需要直接依赖乙方，它只需要达到的目的告诉第三方机构就可以了，比如
>
> 甲方需要一双袜子，而乙方它卖一双袜子，它要把袜子卖出去，并不需要自己去直接找到一个卖家来完成袜
>
> 子的卖出。它也只需要找第三方，告诉别人我要卖一双袜子。这下好了，甲乙双方进行交易活动，都不需要
>
> 自己直接去找卖家，相当于程序内部开放接口，卖家由第三方作为参数传入。甲乙互相不依赖，而且只有在
>
> 进行交易活动的时候，甲才和乙产生联系。反之亦然。这样做什么好处么呢，甲乙可以在对方不真实存在的
>
> 情况下独立存在，而且保证不交易时候无联系，想交易的时候可以很容易的产生联系。甲乙交易活动不需要
>
> 双方见面，避免了双方的互不信任造成交易失败的问题。**因为交易由第三方来负责联系，而且甲乙都认为第**
>
> **三方可靠。那么交易就能很可靠很灵活的产生和进行了**。这就是ioc的核心思想。生活中这种例子比比皆
>
> 是，支付宝在整个淘宝体系里就是庞大的ioc容器，交易双方之外的第三方，提供可靠性可依赖可灵活变更交
>
> 易方的资源管理中心。另外人事代理也是，雇佣机构和个人之外的第三方。
> ==========================update===========================
>
> 在以上的描述中，诞生了两个专业词汇，依赖注入和控制反转所谓的依赖注入，则是，甲方开放接口，在它
>
> 需要的时候，能够讲乙方传递进来(注入)所谓的控制反转，甲乙双方不相互依赖，交易活动的进行不依赖于甲
>
> 乙任何一方，整个活动的进行由第三方负责管理。

内容来源：<a href="https://www.zhihu.com/question/23277575/answer/24259844">Spring IoC有什么好处呢？</a>

**知乎@Intopass的回答：**

1. 不用自己组装，拿来就用。
2. 享受单例的好处，效率高，不浪费空间。
3. 便于单元测试，方便切换mock组件。
4. 便于进行AOP操作，对于使用者是透明的。
5. 统一配置，便于修改。



**2.2、依赖注入**

依赖注入，就是 Spring 用来管理对象的对象之间的依赖关系。当一个类依赖另一个类的实例，但另外一个类依赖

其他类的实例，在第一个类使用依赖实例时，Spring 会自动依赖注入依赖实例的依赖。比喻说：A 类的实例依赖 

B 类的实例，而 B 类的实例依赖于 C 类的实例，在向 A 类的实例注入依赖 B 类的实例之前，Spring 会自动向 B 类

的实例注入依赖 C 类的实例。有了 Spring 管理，在开发中，开发人员不在需要创建 A 类，B 类，C 类的实例，然

后手动注入（setter）了，十分方便。

> 在 Spring 框架中， 对象的创建，声明周期的管理，以及对象与对象之间的依赖关系都是由 Spring 来管理
>
> 的，开发人员无需自己动手实现，只需要在配置文件声明好，或者使用注解。个人觉得 IoC 和 DI 其实是同一
>
> 个概念的不同角度描述，只不过 DI 相对 IoC 而言，**明确描述了“被注入对象依赖 IoC 容器配置依赖对象”**。



### 三、Spring 框架结构

![spring 框架图](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring/images/spring%20%E6%A1%86%E6%9E%B6%E5%9B%BE.jpg)

- **Data Access/Integration**：包含有 JDBC（数据库连接）、ORM（关系映射）、OXM（对象与xml映射）、

  JMS（Java Message Sevice） 和 Transaction（事务管理）模块。

  > spring 对数据库的支持

- **Web** ：包含了Web、Web-Servlet、WebSocket、Web-Porlet模块。

  > spring 对 web 的支持

- **AOP**：提供了一个符合AOP联盟标准的面向切面编程的实现。

  > spring 实现切面编程的支持

- **Core Container(核心容器)**： 包含有Beans、Core、Context 和 SpEL 模块。

  > spring 的核心功能，其中 beans 和 core 主演实现控制反转和依赖注入， context 是 spring 的上下文， 
  >
  > SpEl 是表达式语言模块，提供了在运行期间查询和操作对象图的强大能力。

- **Test**： 支持使用 JUnit 和 TestNG 对 Spring 组件进行测试。



### 四、 编写第一个 Spring 程序

使用 Spring 的 Core 模块编写一个 Spring 的程序。使用 maven 工程建立项目。

```xml
<dependencies>
    <!--core jar-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>
    <!--beans jar-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>
    <!--context（上下文） jar-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>
    <!--el 表达式 ajr-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-expression</artifactId>
        <version>4.3.18.RELEASE</version>
    </dependency>
    <!--日志包-->
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.1.3</version>
    </dependency>
</dependencies>
```

体验一把 spring 的  IoC，创建一个用于演示的类：Student

```java
public class Student {
    
    private String name;
    private String gender;
    private int age;
    //getter and setter
}
```

创建 Spring 的配置文件：*application.xml* ，并在文件中装载 演示类 Student，知道怎么编写的话，可以在官网查

看： *spring-framework-3.2.5.RELEASE\docs\spring-framework-reference\htmlsingle\index.html* 找到XML配置文件

的约束，如果编译器是IDEA 的话，可以选择创建 IDEA 自带的 Spring 配置文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.zy.bean.Student" name="stu">
        <!--在配置文件注入成员的值时，相应类必须给出相应的 getter 和 setter 方法，否则报错-->
        <property name="name" value="tom"/>
        <property name="age" value="18"/>
        <property name="gender" value="male"/>
    </bean>
    <!--
		id : 唯一标识
		class：类的全路径名
		name：类的名字
	-->
</beans>
```

对于 bean 节点中的 name 和 id 的联系和区别,详情请看：<a href="https://blog.csdn.net/SEVENY_/article/details/88029611">Spring配置中的id和name的区别</a>

编写一个测试类，用于体验 spring 的IoC。

```java
public class IOCTest {

    //spring 的上下文
    private ApplicationContext act;

    @Before
    public void setUp(){
        //加载 spring 的配置文件
        act=new ClassPathXmlApplicationContext("application.xml");
    }

    @Test
    public void testIOC(){
        Student student = act.getBean("student", Student.class);
        Assert.assertNotNull(student);
        System.out.println(student); //Student{name='tom', gender='male', age=18}
    }
}
```

有了单元测试的演示，相信大家会已经清除 Spring 的 IoC 概念了吧 —— 把创建对象的控制权交给 spring，对象

的生命周期由 spring 来管理，想要获取类的对象，直接从 Spring 的 IoC 容器中获取对象。

![img](https://upload-images.jianshu.io/upload_images/7896890-bb752724e10e0df2.png?imageMogr2/auto-orient/strip|imageView2/2/w/503/format/webp)

图片来源于参考资料。

体验了 spring 的 IoC，接着来体验一把 spring 的 DI（依赖注入）。创建一个 School 类。

```java
public class School {

    private String name;
    private String address;
    private List<Student> students;
	// getter and setter
}
```

在 spring 的配置文件中，装载 School 类。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="com.zy.bean.Student" name="stu">
        <!--在配置文件注入成员的值时，相应类必须给出相应的 getter 和 setter 方法，否则报错-->
        <property name="name" value="tom"/>
        <property name="age" value="18"/>
        <property name="gender" value="male"/>
    </bean>
    <!--
		id : 唯一标识
		class：类的全路径名
		name：类的名字
	-->

    <bean id="school" class="com.zy.bean.School">
        <property name="name" value="sspu"/>
        <property name="address" value="上海"/>
        <!--装载集合-->
        <property name="students">
            <list>
                <!--ref : 引用对象-->
                <ref bean="student"/>
            </list>
        </property>
    </bean>
</beans>
```

在单元测试类，增加一个测试方法。

```java
public class IOCTest {

    //spring 的上下文
    private ApplicationContext act;

    @Before
    public void setUp(){
        //加载 spring 的配置文件
        act=new ClassPathXmlApplicationContext("application.xml");
    }

    @Test
    public void testIOC(){
        Student student = act.getBean("student", Student.class);
        Assert.assertNotNull(student);
        System.out.println(student);
    }

    @Test
    public void testSchool(){
        School school = act.getBean("school", School.class);
        Assert.assertNotNull(school);
        System.out.println(school);
//School{name='sspu', address='上海', students=[Student{name='tom', gender='male', age=18}]}
    }
}
```

由于输出结果，可以知道 spring 会自动实现依赖注入（DI），无需开发人员的实现。看到这个例子，相信大家更

加清楚 IoC 和 DI 的概念了吧。其实： IoC 和 DI 其实是同一个概念的不同角度描述，DI 相对 IoC 而言，明确描述

了“被注入对象依赖 IoC 容器配置依赖对象”。

**IoC 实现的过程：**

- 读取配置文件/类上的注解（如果是有注解开发的话），拿到类的全限定名，然后使用反射的API，基于类名创

  建类的实例。然后如果使用的 xml 的话，就使用 setter 方法注入。如果是使用注解的话，就是 Field 类的 API

  获取权限之后注入属性。 



### 五、Spring AOP 简介

如果说 IoC 是 Spring 的核心，那么面向切面编程（AOP）就是 Spring 最为重要的功能之一了，在数据库事务中

切面编程被广泛使用（事务使用 AOP 之后，事务的开启，提交，关闭都由 spring 管理，开发人员只需关业务）。



**5.1、AOP (Aspect Oriented Program) 面向切面编程**

在面向切面编程的思想里面，会将业务分为核心业务，和周边业务。

- 核心业务：系统中主要处理的业务，比如：登录、数据库的增删改等

- 周边业务：系统的主要处理的业务的附加业务，比如：登录日志，数据库的事务等等（重复用到的业务）周边

  功能在 Spring 的面向切面编程AOP思想里，即被定义为切面

在面向切面编程 AOP 的思想里面，核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 

"编织" 在一起，其实就是动态代理，在执行的主要业务的方法之前，或者之后，或者前后 执行周边业务。



**5.2、AOP 的目的**

AOP 将与业务无关，却为业务模块所提供服务的代码（例如事务处理、日志管理、权限控制等）封装起来，便于

减少系统的重复代码，减少开发人员的工作，在执行主要业务的代码前后，使用 “织入”的行为， 降低模块间的耦

合度，有利于未来的可拓展性和可维护性。



**5.3、AOP 的相关概念**

- 切入点（PointCut）

  在什么类的什么方法（主要的业务的方法）上（where），织入周边功能。

- 通知（Advice）

  在切入点的前后（where 方法前/方法后/方法前后），要做什么事情（执行的周边业务）

- 切面（Aspect）

  由切入点和通知构成，即：切面 = 切入点 + 通知，其实就是：在什么时机（方法前/方法后/方法前后），什么

  地方（执行主要业务的方法），做什么事情（执行周边业务）。

- 织入（Weaving）

  把切面加入到对象。也就是创建出代理对象的过程。（由 Spring 来完成）



**5.4、体验 AOP 编程**

演示登录，然后在控制台输出登录日志。

首先在 *pom.xml* 文件添加依赖包的引用

```xml
<!--Aop jar-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.3.18.RELEASE</version>
</dependency>
<!--aop 织入 jar-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.18.RELEASE</version>
</dependency>
```

接着创建 Login 类和 LoggerUtil 类

```java
public class Login {
    public void login(String username,String password){
        if ("admin".equals(username) && "123456".equals(password)){
            System.out.println("用户："+username+" 要登录系统");
        }
    }
}

public class LoggerUtil {
    public void log(ProceedingJoinPoint joinPoint) throws Throwable {
        String method = joinPoint.getSignature().getName();
        System.out.println("log start：待执行核心业务的方法: "+method);
        //执行核心业务
        joinPoint.proceed();
        System.out.println("login end ：用户成功登录系统 ");
    }
}
```

然后在 *application.xml* 文件中，装载 Login类和 LoggerUtil 类，以及配置 AOP。

```xml
<bean id="logger" class="com.zy.aop.LoggerUtil"/>
<bean id="login" class="com.zy.aop.Login"/>

<!-- 配置AOP -->
<aop:config>
    <!-- where：在哪些地方（主要的业务功能） -->
    <aop:pointcut id="loggerCutPoint" expression="execution(* com.zy.aop.Login.*(..))"/>

    <!-- what:做什么增强（ref：指定周边业务的执行类） -->
    <aop:aspect id="logAspect" ref="logger">
        <!-- when:在什么时机（方法前/后/前后） method：指定处理周边业务的方法-->
        <aop:around pointcut-ref="loggerCutPoint" method="log"/>
    </aop:aspect>
</aop:config>
```

在单元测类中，添加测试方法：

```java
@Test
public void login(){
    Login login = act.getBean("login", Login.class);
    login.login("admin","123456");
}
/*控制台输出结果
log start：待执行核心业务的方法: login
用户：admin要登录系统
login end ：用户成功登录系统 
*/
```

入门篇完结。



参考资料：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483942&idx=1&sn=f71e1adeeaea3430dd989ef47cf9a0b3&chksm=ebd74327dca0ca3141c8636e95d41629843d2623d82be799cf72701fb02a665763140b480aec&scene=21###wechat_redirect">Spring入门这一篇就够了</a>

<a href="https://www.jianshu.com/p/1af66a499f49">Spring学习(1)——快速入门</a>
