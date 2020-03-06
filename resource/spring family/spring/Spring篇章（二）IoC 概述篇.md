## Spring篇章（二）IoC 概述篇

卸载前面，此篇章主要的内容来源于 <a href="https://www.cnblogs.com/wmyskxz/p/8824597.html">Spring(2)——Spring IoC 详解</a> ，部分内容根据个人见解修改。

![img](https://upload-images.jianshu.io/upload_images/7896890-34e6864b15c793ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 一、Spring IoC 概述

**1.1、IoC （Inverse of Control）：控制反转**

- 读作 “反转控制”，可能更好了解，其不是什么技术，而是一种设计思想，就是将原本程序中手动创建对象的控

  制权，交由Spring框架来管理。

- 正控：若要使用某个对象，需要自己去负责对象的创建

- 反控：若要使用某个对象，只需要从 Spring 容器中获取需要使用的对象，不关心对象的创建过程，也就是把

  创建对象的控制权反转给了Spring框架

- 好莱坞法则：Don’t call me ,I’ll call you

**1.2、举个例子**

控制反转显然是一个抽象的概念，举一个鲜明的例子来说明，会更改了解。

在现实生活中，人们要用到一样东西的时候，第一反应就是去找到这件东西，比如想喝新鲜橙汁，在没有饮品店的

情况，最直观的做法就是：买果汁机、买橙子，然后炸橙汁。值得注意的是：这些都是自己**“主动”创造**的过程，也

就是说一杯橙汁需要自己创造。

![IoC（1）](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring/images/IoC%EF%BC%881%EF%BC%89.png)

然而今天，由于饮品店的盛行，想喝橙汁时，第一想法就转换成了找到饮品店的联系方式，通过电话等渠道描述个

人需要、地址、联系方式等，下订单等待，过一会儿就会有人送来橙汁了。

![IoC（2）](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring/images/IoC%EF%BC%882%EF%BC%89.png)

请注意，以上都没有“主动”去创造橙汁，橙汁是由饮品店创造的，个人需要时，直接交付需求，然后就可以获取。

**1.3、Spring IoC 阐述**

上述的例子已经很好的说明了控制反转的理念，重新再理一下控制反转的概念：控制反转是一种通过描述（在 

Java 中可以是 XML 或者注解）并通过第三方（Spring）去产生或获取特定对象的方式。好处就是：降低对象之间

的耦合，开发人员不需要知道一个类的具体实现，只需要知道它有什么用就好了（直接向 IoC 容器拿）。在主动

创建的模式中，责任归于开发者，而在被动的模式下，责任归于 IoC 容器。基于这样的被动形式，就可以说对象

被控制反转了（也可以说是反转了控制）。



### 二、Spring IoC 容器

在上一篇章中，就说过了Spring 会提供 **IoC 容器 **来管理和容纳开发人员所开发的各种各样的 Bean（类），然后

从 IoC 容器中获取容器所装配的 Bean。

**2.1、Spring IoC 容器的设计**

Spring IoC 容器的设计主要是基于以下两个接口：①：BeanFactory；	②：ApplicationContext 。

ApplicationContext 是 BeanFactory 的子接口之一，换句话说：BeanFactory 是 Spring IoC 容器所定义的最底层

接口，而 ApplicationContext 是其最高级接口之一，并对 BeanFactory 功能做了许多的扩展，所以在绝大部分的

工作场景下，都会使用 ApplicationContext 作为 Spring IoC 容器。

![ApplicationContext的继承图](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring/images/ApplicationContext%E7%9A%84%E7%BB%A7%E6%89%BF%E5%9B%BE.png)

**2.2、BeanFactory 介绍**

从上图中可以知道， BeanFactory 位于设计的最底层，它提供了 Spring IoC 最底层的设计，为此，先来看一下该

类中提供了哪些方法：

![BeanFactory](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring/images/BeanFactory.png)

- BeanFactory 提供了对应了多个 getBean(...) 方法来获取在 Spring IoC 容器装配的 Bean。

  - 按照类型获取 bean：factory.getBean(Bean.class); 	

    **注意：**高方法要求在 Spring IoC 容器中只配置了一个这种类型的实例，否则报错。(不知道获取哪一个)

  - 按照 bean 的名字获取bean：factory.getBean("beanName");   

    **注意**：这种方法不太安全，因为IDE 不会检查其安全性（关联性） 

  - 按照名字和类型获取 bean： factory.getBean("beanName", Bean.class); （推荐使用）

- isSingleton(....) ：用于判断是否单例，如果判断为真，表示该 Bean 在容器中是作为一个唯一单例存在的。

- isPrototype(....)：用于判断是否是多例，如果判断为真，表示每从容器中获取 Bean 时，容器就生成一个新的

  实例。 

  > 注意在默认情况下，isSingleton 为 ture，而 isPrototype 为 false

- getType(....)： 按 Java 类型匹配的方式获取。

- getAliases(....)：获取装备 bean时设置的别名

以上就是 Spring IoC 最底层的设计，所有关于 Spring IoC 的容器将会遵守它所定义的方法。使用 BeanFactroy 的

实现类，获取 Spring IoC 容器装配的 Bean。

```java
//加载Spring的资源文件
Resource resource = new ClassPathResource("applicationContext.xml");
//创建IOC容器对象【IOC容器=工厂类+applicationContext.xml】
BeanFactory beanFactory = new XmlBeanFactory(resourc
```



**2.3、ApplicationContext** 获取 IoC  容器装配的 bean

根据 ApplicationContext 的类继承关系图，可以知道 ApplicationContext 接口扩展了许多的接口，因此它的功能

十分强大，所以在实际应用中常使用的是 ApplicationContext 接口，而不是 BeanFactory，因为 BeanFactory 的

方法和功能较少，而 ApplicationContext 的方法和功能较多。

在上一个篇章《<a href="">Spring 篇章（一）入门篇</a>》中的编写第一个 Spring 程序部分，就使用了 ApplicationContext 的

子类 ClassPathXmlApplicationContext，来获取 IoC 容器中装配的 Bean。

1. 首先在 Resource 目录下，创建 Spring 的配置文件 *application.xml*

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

2. 然后在单元测试类中，使用 ClassPathXmlApplicationContext 初始化 Spring 的 IoC 容器， 然后获取 bean。

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

   

**2.4、ApplicationContext 常见的实现类**

- ClassPathXmlApplicationContext： 读取 classpath 中的资源

  ```java
  ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
  ```

- FileSystemXmlApplicationContext：读取指定路径的资源

  ```java
  ApplicationContext ac = new               FileSystemXmlApplicationContext("c:/applicationContext.xml");
  ```

- XmlWebApplicationContext： 需要在Web的环境下才可以运行

  ```java
  XmlWebApplicationContext ac = new XmlWebApplicationContext(); // 这时并没有初始化容器
  ac.setServletContext(servletContext); // 需要指定ServletContext对象
  ac.setConfigLocation("/WEB-INF/applicationContext.xml"); // 指定配置文件路径，开头的斜线表示Web应用的根目录
  ac.refresh(); // 初始化容器
  ```

- AnnotationConfigApplicationContext：读取指定扫描的包中的使用注解的类或扫描指定的类

  ```java
  public void testConfig(){
  /* 在 StudentConfig 类上使用 @ComponentScan 注解，指定要扫描 StudentConfig类所在包中的所有带注 	解的类，然后装配到 Spring 的 IoC 容器中 */
      ApplicationContext context = new 
          AnnotationConfigApplicationContext(StudentConfig.class);
      StudentService service = context.getBean("studentService", StudentService.class);
      Assert.assertNotNull(service);
      System.out.println(service);
  }
  ```

- AnnotationConfigWebApplicationContext：读取指定扫描的包中的使用注解的类或扫描指定的类（专门为

  Web 准备，使用方式和 AnnotationConfigApplicationContext 类似）。



**2.5、BeanFactory 和 ApplicationContext 的区别**

- **BeanFactory：**是Spring中最底层的接口，只提供了最简单的IoC功能,负责配置，创建和管理bean。

  > 在应用中，一般不使用 BeanFactory，而推荐使用ApplicationContext（应用上下文），原因如下。 

- **ApplicationContext：**

  - 继承了 BeanFactory，拥有了基本的 IoC 功能；

  - 除此之外，ApplicationContext 还提供了其他功能：① 支持国际化；② 支持消息机制；③ 支持统一的资

    源加载；   ④ 支持AOP功能；

### 三、Spring IoC 的容器的初始化和依赖注入

虽然 Spring IoC 容器的生成十分的复杂，但是大体了解一下 Spring IoC 初始化的过程还是必要的。这对于理解 

Spring 的一系列行为是很有帮助的。IoC 容器装配 Bean，分为两大步骤：先定义 Bean，然后是 Bean 的初始化

和依赖注入。

- Bean 的定义分为 3 步：

  - Resource 定位

  > Spring IoC 容器先根据开发者的配置，进行资源的定位，在 Spring 的开发中，通过 XML 或者注解都是十分常见的方式，定位的内容是由开发者提供的。

  - BeanDefinition 的载入

  > 将 Resource 定位到的信息，保存到 Bean 定义（BeanDefinition）中，此时并不会创建 Bean 的实例

  - BeanDefinition 的注册

  > 将 BeanDefinition 的信息发布到 Spring IoC 容器中,**但注意**：此时仍然没有对应的 Bean 的实例。

做完了上述的 3 步，Bean 就在 Spring IoC 容器中被定义了，但还没有没有被初始化，更没有完成依赖注入，也

就是没有注入其配置的资源给 Bean，Bean 还不能完全使用。

- 初始化和依赖注入

  对于初始化和依赖注入，Spring Bean 还有一个配置选项：lazy-init（是否初始化发布到 Spring IoC 的

  Bean）。在没有任何配置的情况下，默认值为 default，也就是 Spring IoC 默认会自动初始化 Bean。如果设

  置为 true，那么只有使用  getBean 方法获取 Bean 时，IoC 容器才会进行 Bean 的初始化，完成依赖注入。

  

IoC概述篇完结。
