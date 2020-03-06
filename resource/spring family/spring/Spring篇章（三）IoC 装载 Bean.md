## Spring篇章（三）IoC 容器装载 Bean



### 一、概述

前面两个篇章已经介绍了 Spring IoC 的理念和设计，这一篇文章将介绍的是如何将开发出来的 Bean 装配到 

Spring IoC 容器中。对于 IoC 装配 Bean,Spring 提供了三种方法进行配置：① 在 xml 配置文件中显示装配；② 在

类上使用注解装配； ③：隐式 Bean 的发现机制和自动装配原则

方式的选择原则：

* 基于约定大于配置的原则，优先使用隐式 Bean 的发现机制和自动装配原则。

  > 好处：解耦合，简单又不是灵活

* 其次，在类上使用注解装配

  > 好处：避免 XML 配置的泛滥，更容易开发，也可以避免 Bean 歧义（一个接口多个实现类，直接使用注
  >
  > 解声明类的别名，然后根据别名获取）。

* 最后，使用XML 方式配置

  > 好处：简单易懂（当然，特别是对于初学者）



### 二、装配方法介绍

**2.1、使用 xml 配置**

2.1.1、在前面两个篇章中，都是使用的是 xml 配置，相信大家都会知道怎么配置了吧，但是还是在这重新讲一下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="student" class="com.zy.bean.Student" name="stu">
        <!--在配置文件注入成员的值时，相应类必须给出相应的 getter 和 setter 方法，否则报错-->
        <property name="name" value="tom"/>
        <property name="age" value="18"/>
        <property name="gender" value="male"/>
    </bean>
    <!--
		id : 唯一标识,必须以字母开头
		class：类的全路径名
		name：类的名字（别名）,id没有声明，name声明的话，IoC 会使用 name 作为唯一标识。
		property： 元素是定义类的属性，其中的 name 属性定义的是属性的名称，而 value 是它的值，
		如果属性是对象的话，则使用 ref 来制定所依赖的对象 
	-->
    <bean id="school" class="com.zy.bean.School">
        <property name="name" value="sspu"/>
        <property name="address" value="上海"/>
        <property name="students">
            <list>
                <!-- ref: 指定所注入的对象 -->
                <ref bean="student"/>
            </list>
        </property>
    </bean>
</beans>	
```

对于 bean 节点中的 name 和 id 的联系和区别,详情请看：<a href="https://blog.csdn.net/SEVENY_/article/details/88029611">Spring配置中的id和name的区别</a> 。说一下，bean 节

点中常用其他属性。

* abstract ：默认为 false，为 true 时，表名该 bean 是一个抽象的 bean，不会被 IoC 容器初始化和实例。

* parent：指明该 bean 的父 bean（可继承父bean 的属性）

  > 一般 abstract 和 parent 是在两个bean 中搭配使用的。
  >
  > 详情：<a href="https://www.cnblogs.com/mfrank/p/10753127.html">bean标签中的属性（二）你可能还不够了解的 abstract 属性和 parent 属性</a>

* scope： prototype|singleton (作用域是：单例还是多例)

* lazy-init：false|true，是否为懒初始化，默认 false

* init-method：初始化 Bean 的时候，执行该类的某个方法

* destroy-method： 在销毁 Bean 的时候，执行该类的某个方法

* factory-method：工厂化方法（指定实例该 bean 的工厂方法）

* factory-bean：指定生产该 bean 的工厂 bean

  > 非静态工厂：需要使用：factory-bean 和 factory-method 搭配使用；
  >
  > 静态工厂只需要使用：destroy-method 就可以了



其实当 bean 的属性值为对象时，也可以这样装配 bean。

```xml
<bean id="class" class="com.zy.bean.Class">
    <property name="name" value="A1"/>
    <property name="student">
        <bean id="stu0" class="com.zy.bean.Student"/>
    </property>
</bean>
```



2.1.2、演示静态工厂和非静态工厂获取 bean。

使用静态工厂

* 创建 Student 的静态工厂

```java
public class StudentFactory {
    
    public static Student getStudentBean(){
        return new Student();
    }
}
```

* 在 *application.xml* 文件中增加

```xml
<bean id="stu1" class="com.zy.factory.StudentFactory" factory-method="getStudentBean"/>
```

使用非静态工厂

* 创建 Student 的非静态工厂

```java
public class StudentFactory1 {

    public Student getStudentBean(){
        return new Student();
    }
}
```

* 在 *application.xml* 文件中增加

```xml
<bean id="studentFactory" class="com.zy.factory.StudentFactory1"/>
<bean id="stu2" class="com.zy.bean.Student" factory-bean="studentFactory" 
      factory-method="getStudentBean"/>
```

以上的单元测试都可以通过，就不演示了。



2.1.3、Java 容器装配（集合，Set，Map，Array，Properties）

接下来演示当 bean 的属性是容器或者是 Properties 时，如何装配 bean。首先创建一个类 IoC Entity

```java
public class IoCEntity {

    private List<String> list;
    private int[] array;
    private Map<String,Integer> map;
    private Set<String> set;
    private Properties properties;
	//getter and setter
}
```

在 *application.xml* 文件装配该 bean 。

```xml
<bean id="iocEntity" class="com.zy.bean.IoCEntity">
    <property name="list">
        <!--集合-->
        <list>
            <value>test</value>
            <value>txt</value>
            <!--当值为对象时：
               <ref bean="bean1"/>
                <ref bean="bean2"/>
            -->
        </list>
    </property>
    <!--数组-->
    <property name="array">
        <array>
            <value>12</value>
            <value>23</value>
           <!--当值为对象时：
              <ref bean="bean1"/>
               <ref bean="bean2"/>
            -->
        </array>
    </property>
    <!--set-->
    <property name="set">
        <set>
            <value>student</value>
            <value>school</value>
            <!--当值为对象时：
                    <ref bean="bean1"/>
                    <ref bean="bean2"/>
            -->
        </set>
    </property>
    <!--properties-->
    <property name="properties">
        <props>
            <prop key="prop1">123</prop>
            <prop key="prop2">456</prop>
        </props>
    </property>
    <!--map-->
    <property name="map">
        <map>
            <entry key="123" value="123"/>
            <entry key="456" value="456"/>
            <!--
               <entry key-ref="对象" value-ref="对象"/>
              -->
        </map>
    </property>
</bean>
```

**总结：**

- List 属性为对应的 `<list>` 元素进行装配，然后通过多个 `<value>` 元素设值

  > 当值为对象时，使用 `<ref bean="bean1"/>`

- Map 属性为对应的 `<map>` 元素进行装配，然后通过多个 `<entry>` 元素设值，只是 `entry` 包含一个键值对

  (key-value)，然后使用 key-value 分别设置键和值。

  > 当键值都为对象时，使用`< entry kkey-ref="bean1" value-ref="bean2"/>` 来设置

- Properties 属性为对应的 `<props>` 元素进行装配，通过多个 `<prop>` 元素设值，只是 `properties` 元素有

  一个必填属性 `key` ，指定对应的属性。

  > 一般情况下，该属性的值不会是对象的。

- Set 属性为对应的 `<set>` 元素进行装配，然后通过多个 `<value>` 元素设值

  > 当值为对象时，使用 `<ref bean="bean1"/>` 

- 对于数组而言，可以使用 `<array>` 设置值，然后通过多个 `<value>` 元素设值。

  > 当值为对象时，使用 `ref bean="bean1"/>`



2.1.4、使用带参数的构造函数装载 bean

* 带有参数的构造函数的 Class 类

```java
public class Class {

    private String name;
    private Student student;
    
    public Class(String name,Student student){
        this.name=name;
        this.student=student;
    }
    //getter and setter
}
```

* 在 *application.xml* 文件装配该 bean 。

```xml
<bean id="class1" class="com.zy.bean.Class">
    <constructor-arg index="0" value="A2"/>
    <constructor-arg index="1" ref="student"/>
    <!--
        index: 指定参数的小标 value： 值
							ref： 指定对象的bean
    -->
</bean>
```

* c 命名空间 

XML配置创建构造函数有参数的 Bean 的时，`<constructor-arg>` 该节点太长了，不方便使用 ，为了简化配

置，Spring来提供了 c 名称空间（`<constructor-arg>` 的简化）。

在使用 c 名称空间之前，需要导入 *xmlns:c="http://www.springframework.org/schema/c"* 声明使用。

```xml
<bean id="class2" class="com.zy.bean.Class" c:_0="A2" c:student="student"/>

<!-- c 名称空间支持属性名，也支持参数索引，因为在 XML 中不允许数字作为属性的第一个字符，
	因此必须要添加一个下划线来作为前缀。-->
```

* p 命名空间

p 命名空间，也就是 `<property>` 结点的简称。使用 p 命名空间之前，也是需要导入

 *xmlns:p="http://www.springframework.org/schema/p"* 的声明。

```xml
<bean id="class" class="com.zy.bean.Class" p:name="A1" p:student-ref="student"/> 
<!--p 名称空间只支持属性名的形式-->
```

> 注意：使用命名空间之后，必须是在 `<bean>` 节点内注入属性，更多的命名空间用法，请到 spring 官网查阅

* 导入 spring 配置文件

在 时间开发中，一般是一个模块一个 spring 配置文件，这样容易管理，也不会泳衣造成配置文件臃肿。对于将其

它的配置文件导入到 spring 的总配置文件中，spring 提供了 `<import>` 节点导入文件。

```xml
<import resource="application2.xml"/>
```



**2.2、使用注解装配 bean**

上面将了如何在 xml 文件中装载 bean，接下就讲述如何使用注解的方式，让 spring 扫描指定包内的类，然后装

载到 IoC 容器中。spring 提供两种方式，两种方式来让 Spring IoC 容器发现 bean：

- 组件扫描：通过定义资源的方式，让 Spring IoC 容器扫描对应的包，从而把 bean 装配进来。
- 自动装配：通过注解定义，使得一些依赖关系可以通过注解完成。

使用注解的好处：

* 可以减少 xml 的配置，避免 xml 文件臃肿难以维护
* 功能更加强大，既能实现 XML 的功能，也提供了自动装配的功能

2.2.1、使用 @Component 装配 Bean

在指定的类上，使用 @Component 注解该类，Spring IoC 会扫描指定的包，然后把带有 @ Component 注解的

类当一个 bean 实例，然后装载到 IoC 容器中。新建一些用于测试的类。

```java
@Component
public class Person {
    
    @Value("zhangsan")
    private String name;
    @Value("18")
    private int age;
    @Value("male")
    private String gender;
	//getter and setter
}
```

@Component 注解和 @Value 注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Component {
    String value() default "";  //被注解类的名字，默认采用被注解的全称，但一个首字母小写 如：person
}
/***********分割线**********/
public @interface Value {
    String value();  //属性的值
}
```

在 Spring IoC 扫描到 Person 类，然后装配到 IoC 容器初始化时，会生成如一个 person的 实例，类似如下：

```xml
<bean id="person" class="com.zy.entity.Person">
	<property name="name" value="zhangsan"/>
    <property name="age" value="18"/>
    <property name="gender" value="male"/>
</bean>
```

但是现在只是注解声明了这个类，并不能使用，因为 Spring IoC 并不知道这个 Bean 的存在（无法管理），这个

时候可以在 spring 的配置文件中配置要扫描的包路径，然后 IoC 就会扫面注入：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
	
    <!--扫描指定的包-->
    <context:component-scan base-package="com.zy.entity"/>
</beans>
```

单元测试：

```java
@Test
public void testComponent(){
    act=new ClassPathXmlApplicationContext("bean.xml");
    Person person = act.getBean("person", Person.class);
    Assert.assertNotNull(person);
    System.out.println(person);
    //Person{name='zhangsan', age=18, gender='male'}
}
```

也可以使用一个 PersonConfig 类（与 Peson 处于同一个目录下）去告诉 Spring IoC ：

```java
//代表进行扫描（默认是扫描当前包的路径），扫描所有带有 @Component 注解的类，然后装配到 IoC 容器中。
@ComponentScan 
public class PersonConfig {
    
}
```

单元测试：

```java
@Test
public void testComponent(){
    act=new AnnotationConfigApplicationContext(Person.class);
    //act=new AnnotationConfigApplicationContext("com.zy.entity");
    Person person = act.getBean("person", Person.class);
    Assert.assertNotNull(person);  
    System.out.println(person); //结果是一样的
}
```

看一下 @ComponentScan 注解常用的属性

```java
public @interface ComponentScan {
    @AliasFor("basePackages") //别名
    String[] value() default {}; ////指定所扫描的包，

    @AliasFor("value")
    String[] basePackages() default {}; //指定所扫描的包

    Class<?>[] basePackageClasses() default {}; //指定所扫描的类
    
    boolean lazyInit() default false; //是否懒初始化
	//......
}
```

> 注意：@Value 只能注入简单类型，无法注入对象。



2.2.2、使用 @Autowried 自动装配

由于  @Value 无法注入对象，因此 Spring 提供了 @Autowried 注解，注入对象。所谓自动装配技术是一种 **由 **

**Spring 自己发现对应的 Bean，自动完成装配工作的方式（Spring 会根据待注入属性的类型去寻找定义的 Bean **

**然后将其注入）**。接下来，使用代码体验一下：

```java
@Component
public class House {
    
    @Autowired
    private Person person;
    @Value("sspu")
    private String address;

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "House{" +
                "person=" + person +
                ", address='" + address + '\'' +
                '}';
    }
}
```

单元测试

```java
@Test
public void testComponent(){
    act=new AnnotationConfigApplicationContext("com.zy.entity");
    House house = act.getBean("house", House.class);
    Assert.assertNotNull(house);
    System.out.println(house);
    //House{person=Person{name='zhangsan', age=18, gender='male'}, address='sspu'}
}
```

有控制台可以知道，spring 在装载 bean 时，会自动完成 bean 属性的注入工作的责任。@Autowired 注解不仅仅

能配置在属性之上，还允许方法配置常见的 Bean 的 setter 方法也可以使用它来完成注入，总之一切需要 Spring I

oC 去寻找 Bean 资源的地方都可以用到，例如：

```java
@Autowired  //也是
public Car(Person person){
    this.person=person;
}
```

> 在大部分的配置中都推荐使用这样的自动注入来完成，这是 Spring IoC 帮助我们自动装配完成的，这样使得
>
> 配置大幅度减少，满足约定优于配置的原则，增强程序的健壮性。

看一下 @Autowried 注解

```java
@Documented
public @interface Autowired {
    boolean required() default true;  
    //是否是强制性注入，默认为 true，如果为 false，当找不到注入类型时，则不注入
}
```



2.2.2.1、自动装配的歧义性（@Primary和@Qualifier）

在 Spring IoC  容器中一个类型存在两个 bean 时，spring 在自动装配时，就不知道该注入哪一个 bean 的，然后

抛出异常，例如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.zy.entity"/>
    <bean id="person1" class="com.zy.entity.Person">
        <property name="name" value="wangwu"/>
        <property name="age" value="20"/>
        <property name="gender" value="male"/>
    </bean>
</beans>
```

使用注解有一个 Person 的 bean，现在在 *application.xml* 文件中又配置了一个 bean，这样在 House 类的自动注

入中，spring IoC 容器无法知道应该注入哪一个bean 的实例。

* @Primary 注解：

  代表首要的，当 Spring IoC 检测到有多个相同类型的 Bean 资源的时候，会优先注入**使用该注解的类**。

  问题：该注解只是解决了首要的问题，但是并没有选择性的问题

* @Qualifier 注解：

  歧义性产生的一个重要的原因是 Spring 在寻找依赖注入的时候是按照类型注入引起的。除了按类型查找 

  Bean。@Qualifier 提供了使用名字和类型查找资源的解决方法。

  ```java
  @Autowired
  @Qualifier("person") //只用value 属性，强制性要写 bean 的id 或者 name
  private Person person;
  ```

其实歧义性的解决，其实也可以使用 Java 提供的 **@Resource** 注解来解决，该注解有一个 name 属性用来指明所

要注入的资源名，spring 可以识别该注解，并将该注解的类装载到 spring 的 IoC 容器中 。



**2.3、使用约定大于配置的方式——@Bean 装载 bean**

当需要引用第三方包的（jar 文件），而且往往并没有这些包的源码，这时无法为这些包的类加入 `@Component`  

注解，让它们变成开发环境中的 Bean 资源。对此 Spring 提供了 @Bean 注解，该注解用于注解方法（方法返回

的是对象），使其方法返回的对象成为 Spring IoC中 Bean 资源。体验一下。

```java
package com.zy.entity;
//注意： @Configuration 注解相当于 XML 文件的根元素，必须要有，才能解析其中的 @Bean 注解
@Configuration
public class Beans {

    @Bean("house")
    public House getHouse(){
        return new House();
    }

    @Bean("person0")
    public Person getPersonBean(){
        return new Person();
    }
}

/*********************分割线******************************/
@Component
public class Card {

    @Resource(name="person0")
    private Person person;

    @Resource(name="house0")
    private House house;

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Card{" +
                "person=" + person +
                ", house=" + house +
                ", name='" + name + '\'' +
                '}';
    }
}
```

单元测试（测试通过）：

```java
@Test
public void testComponent(){
    act=new AnnotationConfigApplicationContext("com.zy.entity");
    Card card = act.getBean("card", Card.class);
    System.out.println(card);
}
```

来看一下 @Configuration 注解 和 @Bean 注解

```java
public @interface Configuration {
    String value() default ""; //该配置的名称
}

/*********************分割线******************************/
public @interface Bean {
    @AliasFor("name") //别名
    String[] value() default {}; //bean 的名字

    @AliasFor("value")
    String[] name() default {}; //bean 的名字

    Autowire autowire() default Autowire.NO; /自动装载类型

    String initMethod() default ""; //初始化bean的实例时，执行的方法

    String destroyMethod() default "(inferred)"; //销毁 bean 的实例时，执行的方法
}

/*********************分割线******************************/
public enum Autowire {
    NO(0),
    BY_NAME(1), //按照名字查找
    BY_TYPE(2); //按照类型查找

    private final int value;

    private Autowire(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }

    public boolean isAutowire() {
        return this == BY_NAME || this == BY_TYPE;
    }
}
```



**2.4、Bean 的作用域**

**在默认的情况下，Spring IoC 容器只会对一个 Bean 创建一个实例**，但有时候，希望获取的是多个实例，为此 

spring 提供了 `@Scope` 注解或者 `<bean>` 元素中的 `scope` 属性来设置，例如：

```xml
// XML 中设置作用域
<bean id="" class="" scope="prototype" />
// 使用注解设置作用域
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

Spring 提供了 5 种作用域，它会根据情况来决定是否生成新的对象：

| 作用域类别              | 描述                                                         |
| :---------------------- | :----------------------------------------------------------- |
| singleton(单例)         | 在Spring IoC容器中仅存在一个Bean实例 （默认的scope）         |
| prototype(多例)         | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时 ，相当于执行new XxxBean()：不会在容器启动时创建对象 |
| request(请求)           | 用于web开发，将Bean放入request范围 ，request.setAttribute("xxx") ， 在同一个request 获得同一个Bean |
| session(会话)           | 用于web开发，将Bean 放入Session范围，在同一个Session 获得同一个Bean |
| globalSession(全局会话) | 一般用于 Porlet 应用环境 , 分布式系统存在全局 session 概念（单点登录），如果不是 porlet 环境，globalSession 等同于 Session |

在开发中主要使用 `scope="singleton"`、`scope="prototype"`，对于 MVC 中的 Action 使用的是 prototype类

型，其他使用 singleton，Spring 容器会自动管理 Action 对象的创建,此时把 Action 的作用域设置为 prototype。



### 三、 Spring IoC 容器装载 bean 方式的总结

注解和XML配置是可以混合使用的，JavaConfig 和 XML也是可以混合使用。如果JavaConfig的配置类是分散的，

则可以再创建一个更高级的配置类（root），然后使用 @Import 来将配置类进行组合，如果XML的配置文件是分

散的，我们也是创建一个更高级的配置文件（root），然后使用`<import resource=""/>`来将配置文件组合

如果想在 JavaConfig 引用 XML 的 bean 资源，则使用 @ImportResource 注解。而在 XML 想引用JavaConfig

使用`<bean>`节点就可以了。

* @Import 注解

```java
public @interface Import {

	Class<?>[] value();  //配置其他 JavaConfig 类
}
```

* @ImportResource 注解

```java
public @interface ImportResource {

	@AliasFor("locations")  //别名
	String[] value() default {};  //xml 文件的路劲

	@AliasFor("value")
	String[] locations() default {};  //xml 文件的路劲
	
    // bean 定义解释器
	Class<? extends BeanDefinitionReader> reader() default BeanDefinitionReader.class;

}
```

IoC 容器装载 Bean 篇章完结。



<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483942&idx=1&sn=f71e1adeeaea3430dd989ef47cf9a0b3&chksm=ebd74327dca0ca3141c8636e95d41629843d2623d82be799cf72701fb02a665763140b480aec&scene=21###wechat_redirect">Spring入门这一篇就够了</a>

<a href="https://www.cnblogs.com/wmyskxz/p/8830632.html">Spring(3)——装配 Spring Bean 详解</a>

