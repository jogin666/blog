## spring boot 篇章（一）入门篇

#### 1、spring boot 概述

> **Build Anything with Spring Boot：**Spring Boot is the starting point for building all Spring-based applications. Spring Boot is designed to get you up and running as quickly as possible, with minimal upfront configuration of Spring.

上面是引自官网的一段话，大概是说： Spring Boot 是所有基于 Spring 开发的项目的起点。Spring Boot 的设计是为了让你尽可能快的跑起来 Spring 应用程序并且尽可能减少你的配置文件。

#### 2、 spring boot 是什么

* 是 spring 技术栈的一个大整合
* 简化 spring 应用开发的一个框架
* j2EE 开发的一站式解决方案

> 通俗易懂来说就是：
>
> - spring boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化 spring 应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置，减少开发人员的工作量。
> - 个人觉得 spring boot 不是什么新的框架，只是集成了市面上使用的框架，将一个个情景（框架）集成为一个个启动器，想用什么情景，只需在 *pom.xml* 文件中导入相关的启动器，就可以了，不在像从前那样配置一大堆配置信息。

#### 3、spring boot 的优势

* 无配置的集成主流框架
* 简单、快捷地搭建项目，无关过多的配置
* 极大提高了开发、部署效率。



#### 4、搭建 spring boot 项目

**4.1、使用 idea 新建项目**

* 选择 spring initializr ，然后选择默认的 url 点击【Next】：

![1](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/1.png)

* 修改一下醒目的默认信息

![2](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/2.png)

* 选择 web 启动器（这里提供 springboot 的所有启动器选择项，勾选自己所需的启动器，当然也可以之后，在 *pom.xml* 文件中导入）：

![3](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/3.png)

* 填写项目的信息，然后点击 【finish】，等待项目构建完成。

![4](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/4.png)

* 构成完的项目的目录结构如下：

![目录结构](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.png)



* *SpringBootApplication* ：项目启动类，带有 *public static void main（..）* 方法，用于启动项目
* *static* 目录：是静态资源文件的目录（css，html，js）
* *template* 目录：是 spring boot 支持的引擎模板存放的目录（spring boot 默认不再支持 jsp 文件了）
* *application.properties*：一个空的 properties 文件，可以根据需要添加配置属性（如：数据库连接信息）
* *SpringbootApplicationTests*：一个空的 Junit 测试类，它加载了一个使用 Spring Boot 字典配置功能的 Spring 应用程序上下文，用于单元测试（个人感觉相当于单元测试的启动类）。
* *pom.xml* ：项目依赖构建文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.zy</groupId>
    <artifactId>springboot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!-- web 启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--热部署-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <!--开发工具-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!--单元测试启动器-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

上面的 *pom.xml* 文件中，`<dependency>` 标签管理的依赖包没有指定版本，同时出现陌生一个的标签 `<parent>` 。`<parent>` 标签是在配置 *pom.xml* 文件的父级依赖，也就是在配置 Spring Boot 的父级依赖，将所有 spring boot 项目的所有依赖包的版本交给父级管理，这样开发人员不在需要管理项目依赖包的版本，避免了依赖冲突的问题。

如果想知道 spring boot 提供的是什么版本的依赖，可以点击 `<parent>` 标签下的 `<artifactId>spring-boot-dependencies</artifactId>` 进入父级文件，然后继续点击父级文件中相同的标签，进入到祖父级文件，尽可以看到 spring boot 提供的 jar 版本了。

*pom.xml* 文件中默认有两个模块：

- `spring-boot-starter` ：核心启动器，包括自动配置支持、日志和 YAML，如果引入了 `spring-boot-starter-web` web 模块可以去掉此配置，因为 `spring-boot-starter-web` 自动依赖了启动器。
- `spring-boot-starter-test` ：测试启动器：集成了 JUnit、Hamcrest、Mockito。



**4.2、 使用 maven 构建项目**

* 1、访问 http://start.spring.io/

- 2、选择构建工具 Maven Project、Java、Spring Boot 版本 2.2.4，选择相应的启动器，可参考下图所示：

![img](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/5.png)

- 3、点击 Generate Project 下载项目压缩包，解压，然后使用 idea 导入项目：File -> New -> Model from Existing Source.. -> 选择解压后的文件夹 -> OK，选择 Maven 一路 Next。

**4.3、编写 Controller**

```java
@RestController
public class DemoController {

    @RequestMapping("/hello")
    public String hello(){
        return "Hello SpringBoot";
    }
}
```

> 值得注意的是：在编写 Controller 时， 推荐编写在 *SpringbootApplication* 启动类目录的同级目录，或者其子目录下，当然也可以编写在其他目录，然后在 *SpringbootApplication* 启动类上，添加扫描注解，扫描该 Controller 类所在的包，更或者在启动类 *SpringBootApplication* 下使用 @Bean 注解向 spring boot 容器注入 Controller 类。

看一下 *@RestController* 注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController {
    @AliasFor(
        annotation = Controller.class
    )
    String value() default "";
}
```

> 从上面的注解中，可以知道，*@RestController*  注解就是 *@ResponseBody* + *@Controller* 注解的结合

启动项目，待控制台打印如下信息，标志 spring boot 项目启动成功：

![6](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/6.png)

> 在此说一下，spring boot 项目启动有三种方式：
>
> * 在idea中直接使用启动（最常用）
> * 使用mvn 命令来启动
> * 使用mvn编译，而后在class目录生成jar包，使用Java命令来启动

然后打开浏览器，输入 *localhost:8080/hello* ,可以成功的显示数据。

![7](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/7.png)

你可能会惊奇，什么都没有配置，包括没有配置 tomcat，就可以写出一个类似一个 springmvc 的 web demo。没错 spring boot 就是这么简洁，基本不用啥配置，就可以轻松的搭建一个 web demo，对于中间件，之所以在上面的项目中没有手动的去配置 Tomcat 服务器，是因为 Spring Boot 内置了 Tomcatspring boot 了，无需开发人员手动配置。

对比以前写的 web 项目，就可以更加深刻的知道 spring boot 的优势。对此，对配置文件烦恼的你，是不是深深的爱上了 spring boot 了啊。

入门篇完结。



参考资料：

<a herf="http://www.ityouknow.com/springboot/2016/01/06/spring-boot-quick-start.html">Spring Boot(一)：入门篇</a>

<a herf="https://www.cnblogs.com/wmyskxz/p/9010832.html">Spring Boot【快速入门】</a>



