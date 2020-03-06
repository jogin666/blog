## spring boot 篇章（二）yml 文件篇

#### 1、前言

spring boot 会自动加载放置在【src/main/resources】目录或者类路径的 /config 下全局配置文件，然后根据配置文件，配置项目工程。对于全局配置文件，spring boot 是支持常规扩展名为  *.properties* 的文件，但 spring boot 更推荐使用以支持 yaml 语言，扩展名 *.yml* 的文件。yaml 是以数据为中心的语言，在配置数据的时候具有面向对象的特征。

#### 2、体验 yaml

**2.1、初体验 yml 文件**

spring boot 的全局配置文件的作用是对一些默认配置的配置值进行修改。例如：修改默认访问端口

* 使用 properties 文件

```properties
server.port=8089
#访问路径  localhost:8080/springboot/
server.servlet.context-path=/springboot
```

* 使用 yml 文件

```yml
server:
  port: 8089
  #访问路径  localhost:8080/springboot/
  servlet:
    context-path: /springboot
```

> Tomcat 默认端口 8080 设置为 8089 ，并将默认的访问路径从 “`/`” 修改为 “`/springboot`” 时，使用 properties 文件和 yml 文件的区别如上。
>
> 注意：在使用 yml 文件时， 配置属性与属性值之前，需要在 “`:`” 后加一个空格，yml 文件时垂直分层的，同一垂直线是属于同一层的。

在 spring boot 的项目工程中，*application.properties* 和 *application.yml* 两个文件是可以同时使用的，形成互补的作用，但不推荐这样使用。

在相同优先级位置：同时有 *application.properties* 和 *application.yml*，那么*application.yml* 配置的属性就会覆盖 *application.properties* 里的属性。


**2.2、定义属性和属性值**

* 使用 yml 文件配置自定义属性，例如配置一个 Person 类的属性

```yml
server:
  port: 8089
  #访问路径  localhost:8080/springboot/
  servlet:
    context-path: /springboot

name: xaioming
age: 18
gender: male

#或者这样配置（引用 yml 文件中自定义的属性值）
personStr: "name: ${name},age: ${age},gender: ${gender}"
```

编辑 JavaBean

```java
@Component
public class Person {

    @Value("${name}")
    private String name;
    @Value("${gender}")
    private String male;
    @Value("${age}")
    private int age;
    @Value("${personStr}")
    private String personStr;

    //getter and setter
}
```

通过单元测试，两种方式都是可以拿到数据的。但是上面的配置存在一种缺点，这样会造成配置文件繁琐而且可能会造成类的臃肿，因为有许许多多的 *@Value* 注解。

为此 spring boot 提供了 *@ConfigurationProperties* 注解，配置前缀信息（指定注入属性），将配置文件中的配置信息，全部注入到配置类中，和之前学到注入一样，那就是配置配置类的属性名和配置文件中的属性名称一样，同时需要给配置文件中的属性一个前缀。

* 修改 yml 文件

```yml
person:
  name: xiaoming
  age: 18
  gender: male
#或者（行内写法）
person: {name: xiao,age: 18,gender: male}
```

* 修改配置类

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person1 {

    private String name;
    private String male;
    private int age;
    
     //getter and setter
}
```

经测试，可以获取到数据。

看一下 *@ConfigurationProperties*

```java
@Target({ElementType.TYPE, ElementType.METHOD}) //也可以用于注解方法，将信息注入方法参数
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ConfigurationProperties {
    @AliasFor("prefix")
    String value() default "";  //指定待注入信息的前缀

    @AliasFor("value")
    String prefix() default "";

    boolean ignoreInvalidFields() default false;  //是否忽略不可注入属性

    boolean ignoreUnknownFields() default true; //是否忽略未知属性
}
```

**2.3、定义容器属性**

* 数组

```yml
person:
  #普通数据
  name: xiaoming
  age: 18
  gender: male 
  
 #数组（多行写法）
  array:
    - 1
    - 2
    - 3
  # 或者 （行内写法）
  array: [1,2,3]
  
```

*  List 和 Set

```yml
person:
  #普通数据
  name: xiaoming
  age: 18
  gender: male 
  #.........
  
  #集合
  list:
    - 12
    - 23
    - 15
    - 12
  # 或者（行内写法）
  list: [12,23,15,12]
  
  #类型为 Dog 的集合（使用 [] 定位符的方式，给集合赋值）
  dogs[0]:
    name: 旺财
    age: 1
  dogs[1]:
    name: 二哈
    age: 2
```

* map

```yml
person:
  #普通数据
  name: xiaoming
  age: 18
  gender: male 
  #......... 
  
  #map（行内写法）
  map: {name: xiao,age: 18}
  #或者
  map:
    name: xiao
    age: 18
```

注意：如果Map类型的key包含非字母数字和-的字符，需要用[]括起来，比如：

```yml
map:
 '[name.s]': xiao
 age: 18
```

* JavaBean

```yml
person:
  #普通数据
  name: xiaoming
  age: 18
  gender: male 
  #.........
  
  #JavaBean
  dog:
    name: wangcai
    age: 2
```

Person1 类

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person1 {

    private String name;
    private String male;
    private int age;

    private int array[];
    private List<Integer> list;
    private Map<String,Object> map;
    private Dog dog;
    private List<Dog> dogs;
	//getter and setter
}
```

将单元测试时，可以成功拿到数据。

* 使用随机数，在 yml 文件中使用随机数，主要是使用`${random}`的配置方式，主要有一下几种：

```yml
num:
  value: '${random.value}'
  number: '${random.int}'
  bignumber: '${random.long}'
  number1: '${random.int(20)}'
  number2: '${random.int[10,20]}'
```

#### 3、环境配置

项目环境一般分为：① 开发环境	② 测试环境	③ 生产环境。不同环境，所需的配置可能是不一样的，比如说path 变量在开发环境中是：`http://localhost:8080`。而在生产环境中 path 路径可能是这样的`https://www.zy.com:8080`。对于环境中的配置信息的差异，在 spring boot 是很好解决的。

* 使用 *spring.profiles* 在同一个文件中定义环境

```yml
spring:
  profiles:
    active: dev  # 指定使用哪一个环境

---   #环境分割符
spring:
  profiles: dev

server:
  port: 8089
  servlet:
    context-path: /springboot

---
spring:
  profiles: test

server:
  port: 8080
  servlet:
    context-path: /springboot

---
spring:
  profiles: prod
server:
  port: 4568
  servlet:
    context-path:  /springboot
```

* 一般不会在同一个文件中配置多个环境，而是一个环境一个配置文件，在 *application.yml* 文件指定使用的环境文件，值得注意的：环境文件必须是以： *application- 环境* 的方式起名。

![环境配置图](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E7%8E%AF%E5%A2%83.png)

* 全局配置文件的加载顺序

| -    | 路径              |
| ---- | ----------------- |
| ——   | ./config/         |
| ——   | ./                |
| ——   | classpth:/config/ |
| ——   | classpath:/       |

优先级可参考下图：

![顺序](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E9%A1%BA%E5%BA%8F.png)



yml 文件篇完结。



参考资料：

<a herf="https://www.cnblogs.com/wmyskxz/p/9010832.html">Spring Boot【快速入门】</a>

<a herf="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484031&idx=2&sn=c586cd21312c720a4a45435ea18dc30a&chksm=ebd7437edca0ca68a8bdf98b962474b53e68372adc9e059964b55de60a0056aa204348f6206b&scene=21###wechat_redirect">SpringBoot就是这么简单</a>

<a herf="http://blog.didispace.com/spring-boot-learning-21-1-3/">Spring Boot 2.x基础教程：配置文件详解</a>

