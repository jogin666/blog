## Spring篇章（四）AOP 讲述篇



### 一、概念介绍

如果说 IoC 是 Spring 的核心，那么面向切面编程（AOP）就是 Spring 最为重要的功能之一了，在数据库事务中切面编程被广泛使用（事务使用 AOP 之后，事务的开启，提交，关闭都由 spring 管理，开发人员只需关业务）。



**1.1、AOP (Aspect Oriented Program) 面向切面编程**

在面向切面编程的思想里面，会将业务分为核心业务，和周边业务。

- 核心业务：系统中主要处理的业务，比如：登录、数据库的增删改等
- 周边业务：系统的主要处理的业务的附加业务，比如：登录日志，数据库的事务等等（重复用到的业务）周边功能在 Spring 的面向切面编程AOP思想里，被定义为切面。

> 在面向切面编程 AOP 的思想里面，核心业务功能和切面功能分别独立进行开发，然后把切面功能和核心业务功能 "编织" 在一起。其实AOP的本质就是动态代理，在执行的主要业务的方法之前，或者之后，或者前后执行周边业务。



**1.2、AOP 的目的**

AOP 将与业务无关，却为业务模块所提供服务的代码（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，减少开发人员的工作，在执行主要业务的代码前后，使用 “织入”的行为， 降低模块间的耦合度，有利于未来的可拓展性和可维护性。



**1.3、AOP 的相关概念**

- 切入点（PointCut）

  在什么类的什么方法（主要的业务的方法）上（where），织入周边功能。

- 通知（Advice）

  在切入点的前后（where 方法前/方法后/方法前后），要做什么事情（执行的周边业务）

- 切面（Aspect）

  由切入点和通知构成，即：切面 = 切入点 + 通知，其实就是：在什么时机（方法前/方法后/方法前后），什么

  地方（执行主要业务的方法），做什么事情（执行周边业务）。

- 织入（Weaving）

  把切面加入到对象。也就是创建出代理对象的过程。（由 Spring 来完成）



### 二、手动编写 AOP 代码

在《<a href="">Spring篇章（一）入门篇</a>》的时候，就演示过 spring  的 AOP 编程的例子。还没有看过的读者，可以先去看看，体验一下 spring 的 AOP 编程。由于这个篇章将要讲述的是 spring 的 AOP，那就模仿 spring 的事务管理。举例，在数据库的增删改操作时，总是先要开启时候，然后才能对数据进行操作，接着是提交事务，然后关闭连接（对于事务，其实 spring 已经提供方式管理，开发人员只需配置便可以使用了）。

为了更好的理解，直接用图来展示吧

![切面编程](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring/images/%E5%88%87%E9%9D%A2%E7%BC%96%E7%A8%8B.png)

图片有了，接下来就是代码实战了，在 spring aop 编程中，开发人员可以在 xml 配置文件中配置或者使用注解的方式来实现切面编程。

**1.使用配置文件的方式**

在编写基于 XML 配置文件的开发之前，先来了解一下 AOP 中可以配置的元素：

| AOP 配置元素          | 用途                             | 备注                         |
| :-------------------- | :------------------------------- | :--------------------------- |
| `aop:config`          | 顶层的 AOP 配置元素              | AOP 的配置是以它为开始的     |
| `aop:pointcut`        | 定义切点                         | ——                           |
| `aop:declare-parents` | 给通知引入新的额外接口，增强功能 | ——                           |
| `aop:advisor`         | 定义 AOP 的通知                  | 是一种很古老的方式，很少使用 |
| `aop:aspect`          | 定义一个切面                     | ——                           |
| `aop:before`          | 定义前置通知                     | ——                           |
| `aop:after`           | 定义后置通知                     | ——                           |
| `aop:around`          | 定义环绕（前后）通知             | ——                           |
| `aop:after-returning` | 定义返回通知                     | ——                           |
| `aop:after-throwing`  | 定义异常通知                     | ——                           |

- 创建 *CommitHelper* 类完成事务多的开启和关闭

```java
/*周边功能
*开启数据库事务
* 关闭数据库事务
*/
public class CommitHelper {

    public void open(){
        System.out.println("开始数据库事务");
    }

    public void close(){
        System.out.println("关闭数据库事务");
    }
}

```

- 创建 *PersonDaoImpl*  完成事务的提交

```java
/*核心功能
 * 保存数据
 */
public class PersonDaoImpl {

    public void save(){
        System.out.println("保存数据");
    }
}
```

- 创建 *PersonServiceImpl* 类 完成业务的处理

```java
public class PersonServiceImpl {

    @Autowired
    private PersonDaoImpl personDao;

    public void save(){
        personDao.save();
    }

    public PersonDaoImpl getPersonDao() {
        return personDao;
    }

    public void setPersonDao(PersonDaoImpl personDao) {
        this.personDao = personDao;
    }
}
```

- 在配置文件中配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="personServiceImpl" class="net.aspect.PersonServiceImpl">
        <property name="personDao" ref="personDao"/>
    </bean>
    
    <bean id="personDao" class="net.aspect.PersonDaoImpl"/>

    <bean id="commitHepler" class="net.aspect.CommitHelper"/>

    <aop:config>
        <!--切点-->
        <aop:pointcut id="personServiceCut" expression="execution(* net.aspect.PersonServiceImpl.save())"/>
        <!--切面-->
        <aop:aspect id="personServiceAspect" ref="commitHepler">
            <aop:before method="open" pointcut-ref="personServiceCut"/>
            <aop:after method="close" pointcut-ref="personServiceCut"/>
        </aop:aspect>
    </aop:config>
</beans>
```

- 单元测试

```java
public class PersonServiceImplTest {

    private ApplicationContext act;

    @Before
    public void setUp(){
        act=new ClassPathXmlApplicationContext("bean.xml");
    }

    @Test
    public void save() {
        PersonServiceImpl service = act.getBean("personServiceImpl", PersonServiceImpl.class);
        service.save();
    }
}
/*控制台打印结果

* 开始数据库事务
* 保存数据
* 关闭数据库事务
*/
```



**2.使用注解的方式**

在编写基于注解的切面编程开发之前，先来了解一下 AOP提供的注解：

| 注解              | 说明                                                         |
| :---------------- | :----------------------------------------------------------- |
| `@Pointcut`       | 定义切点                                                     |
| `@Aspect`         | 定义切面                                                     |
| `@Before`         | 前置通知，在连接点方法前调用                                 |
| `@Around`         | 环绕通知，它将覆盖原有方法，但是允许你通过反射调用原有方法，后面会讲 |
| `@After`          | 后置通知，在连接点方法后调用                                 |
| `@AfterReturning` | 返回通知，在连接点方法执行并正常返回后调用，要求连接点方法在执行过程中没有发生异常 |
| `@AfterThrowing`  | 异常通知，当连接点方法异常时调用                             |

- 在 *CommitHelper* 类中使用注解，定义一个切面类，注意切面类仍是一个 bean ，所以仍然是需要使用 @Component 注解的

```java
/*周边功能
*开启数据库事务
* 关闭数据库事务
*/
@Component
@Aspect
public class CommitHelper {

    @Before("execution(* net.aspect.PersonServiceImpl.save())")  //定义切点
    public void open(){
        System.out.println("开始数据库事务");
    }

    @After("execution(* net.aspect.PersonServiceImpl.save())") //定义切点
    public void close(){
        System.out.println("关闭数据库事务");
    }
}
```

- 如果觉得每次写 *@Before、@After* 等，都要重复编写切入点表达式，觉得很烦，代码不够优雅，可以使用  	*@pointCut* 注解解决这个问题。

```java
@Component
@Aspect
public class CommitHelper {

    //在切入点之前做些啥
    @Before("pointCut()")
    public void open(){
        System.out.println("开始数据库事务");
    }
	
    //在切入点执行完之后，做些啥
    @After("pointCut()")
    public void close(){
        System.out.println("关闭数据库事务");
    }
    
	// 指定切入点
    @Pointcut("execution(* net.aspect.PersonServiceImpl.save())")
    public void pointCut(){

    }
}
```

- 处理核心业务的 *PersonServiceImpl* 和 *PersonDaoImpl* 类如下

```java
/* 业务处理 */
@Service 
public class PersonServiceImpl {

    @Autowired
    private PersonDaoImpl personDao;

    public void save(){
        personDao.save();
    }
}

/*核心功能
 * 保存数据
 */
@Repository
public class PersonDaoImpl {

    public void save(){
        System.out.println("保存数据");
    }
}
```

- 在配置文件中，配置要扫描的包和启动 aop 代理模式

```xml
<!--开启aop代理-->
<aop:aspectj-autoproxy/>

<!--扫描包-->
<context:component-scan base-package="net.aspect"/>
```

- 单元测试

```java
public class PersonServiceImplTest {

    private ApplicationContext act;

    @Before
    public void setUp(){
        act=new ClassPathXmlApplicationContext("bean.xml");
    }

    @Test
    public void save() {
        PersonServiceImpl service = act.getBean("personServiceImpl", PersonServiceImpl.class);
        service.save();
    }
}
/*控制台打印结果

* 开始数据库事务
* 保存数据
* 关闭数据库事务
*/
```

**切点表达式介绍**

- 语法

```java
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)
```

- 符号讲解

  - “ ？” 号代表 0 或 1，可以不写
  - “ * ” 号代表任意类型，0 或 多
  - “ .. ” 号代表可变参数

- 参数讲解

  - modifiers-pattern ：修饰符，可以不写
  - ret-type-pattern ： 方法返回类型，必须要写
  - declaring-type-pattern ： 方法的声明类型，方法声明的类型，可以不写
  - name-pattern(param-pattern) ： 要匹配的名称（方法名），括号里面是方法的参数
  - throws-pattern ：方法抛出的异常，可以不写

  

- 例子演示

  - 指定了修饰符，任意返回类型的任意方法

  ```java
  execution(public * *(..))
  ```

  - 不指定修饰符，指定了任意返回类型的任意 set 方法

  ```java
  execution(* set(..))
  ```

  - 不指定修饰符，指定了任意返回类型的具体类的任意方法

  ```java
  execution(* com.zy.service.PersionService.*(..))
  ```

  - 指定具体包下任意类的任意方法

  ```java
  execution(* com.zy.service.*.*(..))
  ```

  - 指定具体包和其子包下的任意类的任意方法

  ```
  execution(* com.zy.service..*.*(..))
  ```

  

spring 的 aop 讲述篇完结。



参考资料：

①：<a href="https://www.cnblogs.com/wmyskxz/p/8835243.html">Spring(4)——面向切面编程（AOP模块）</a>

②：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483954&idx=1&sn=b34e385ed716edf6f58998ec329f9867&chksm=ebd74333dca0ca257a77c02ab458300ef982adff3cf37eb6d8d2f985f11df5cc07ef17f659d4&scene=21###wechat_redirect">Spring【AOP模块】就这么简单</a>
