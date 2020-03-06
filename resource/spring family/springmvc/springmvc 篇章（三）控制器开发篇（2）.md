## springmvc 篇章（三）控制器开发篇（2）

#### 1、前言

上一个篇章主要讲述了 springmvc 的控制器开发，但是还没有讲完，本篇章将继续讲述控制器。

#### 2、字符串转换日期类型

和 struts2 一样，springmvc 也是不能将 yyy-mm-dd hh:MM:ss 这种格式的字符串，自动解析成日期的，需要开发人员编写一部分相关代码的。

**2.1、单个 Controller 使用**

使用 *@InitBinder*  注解实现，在 Controller 中，添加一个方法，使用 *@InitBinder* 注解。

```java
@Controller
public class DateController {

    @InitBinder
    public void initBinder(ServletRequestDataBinder binder){
        binder.registerCustomEditor(Date.class,
                new CustomDateEditor(new SimpleDateFormat("yyyy-MM-dd"), 
                                     true));
    }
}
```

**2.2、多个 Contronller 使用**

* 使用老的方式，实现 *PropertyEditorRegistrar* 接口，重写 *registerCustomEditors* 方法。

编写实现类：CustomPropertyEditor

```java
public class CustomPropertyEditor implements PropertyEditorRegistrar {

    @Override
    public void registerCustomEditors(PropertyEditorRegistry registry) {
        registry.registerCustomEditor(
                Date.class,
                new CustomDateEditor(new SimpleDateFormat("yyyyy-MM-dd"),true));
    }
}
```

在配置文件，配置 bean：① 自定义的属性编辑器；② web  属性初始绑定器；③ 映射处理适配器

```xml
<!--配置自定义属性编辑器-->
<bean id="propertyEditor" class="com.zy.springmvc.util.CustomPropertyEditor"/>

<!--配置 web 属性初始绑定器-->
<bean id="bindingInitializer" 
      class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">
    <property name="propertyEditorRegistrars">
        <list>
            <ref bean="propertyEditor"/>
        </list>
    </property>
</bean>

<!--映射处理适配器-->
<bean id="mappingHandlerAdapter" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="webBindingInitializer" ref="bindingInitializer"/>
</bean>
```

> 映射处理器拦截到请求后，将请求交给 web 初始绑定器，然后绑定器将参数交给自定义属性编辑器处理属性

* 使用新的方式，自定义参数转换器，实现Converter接口来实现自定义参数转换（推荐使用）

编写自定转换器

```java
public class CustomPropertyConverter implements Converter<String, LocalDate> {

    @Override
    public LocalDate convert(String s) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
        return LocalDate.parse(s, formatter);
    }
}
```

在配置文件配置 bean：① 格式转化器工厂中注册自定义转换器； ② web  属性初始绑定器；③ 映射处理适配器

```java
<!--在格式转化器工厂中注册 自定义转换器-->
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
       <property name="converters">
           <list>
               <bean class="com.zy.springmvc.util.CustomPropertyConverter"/>
           </list>
       </property>
</bean>

<!--配置 web 属性初始绑定器-->
<bean id="bindingInitializer" class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">
        <property name="conversionService" ref="conversionService"/>
</bean>

<!--配置 映射适配器-->
<bean id="mappingHandlerAdapter" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
        <property name="webBindingInitializer" ref="bindingInitializer"/>
</bean>
```

如果是基于注解驱动的 `<mvc:annotation-driven>` 开发，只需这样配置便可以了（推荐使用）：

```xml
<mvc:annotation-driven conversion-service="conversionService"/>  
<!--在 格式转化器工厂中注册 自定义转换器-->
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <list>
            <bean class="com.zy.springmvc.util.CustomPropertyConverter"/>
        </list>
    </property>
</bean>
```

顺便提一下：值得注意的是：对于 Oracle 数据库，如果要插入时间的话，需要在 SQL 语句，使用 TimeStrap 时间戳插入进去，否则是行不通的。

#### 3、JSON 文本格式

JSON 是一种轻量级的文本交换格式，该格式的文本非常适合前后端交互，在 springmvc 中使用 JSON 文本格式，是要导入相应的依赖包，并在相关方法或者类使用 springmvc 提供的注解。

* 导入依赖包

```xml
<!--springmvc 默认支持的 json 核心包-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.8</version>
</dependency>

<!--springmvc 默认支持的 json 注解包-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.8</version>
</dependency>

<!--springmvc 默认支持的 json 数据绑定包-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>
</dependency>
```

编写测试方法

```java
@RequestMapping("/userInfo")
@ResponseBody
public UserEntity showUserInfo6(){
    UserEntity entity = new UserEntity();
    entity.setUsername("zheng");
    entity.setPassword("123456");
    return entity;
}
```

在配置文件，配置 json 注解处理适配器

```xml
<mvc:annotation-driven/>
<bean id="handlerAdapter" class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>
        </list>
    </property>
</bean>
```

或者这样配置（推荐使用）

```xml
<mvc:annotation-driven/>
<mvc:default-servlet-handler/>
```

运行本地服务

![json](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/json.png)

如果 web 端传到服务端的数据时 JSON 格式那该怎么办呢？对此，springmvc 也提供相应的注解 *@RequestBody*  将 JSON 文本格式的数据注入到参数中（模型类型）。

```java
@ResponseBody
@RequestMapping("/showUserInfo")
public UserEntity showUserInfo7(@RequestBody UserEntity userEntity){
    return userEntity;
}
```

![ResponseBody and RequestBody](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/ResponseBody%20and%20RequestBody.png)

​																			（图片来源参考资料）

#### 4、上传文件

在 springmvc 开发上传文件的功能，也是要导入先关的依赖包的，然后在配置文件中，配置文件解析器。

* 导入相关的依赖包

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>

<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.4</version>
</dependency>
```

编写 upload.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>测试文件上传</title>
</head>
<body>
<form action="upload" method="post" enctype="multipart/form-data" >
    <input type="file" name="picture">
    <input type="submit" value="submit">
</form>
</body>
</html>
```

编写 控制器的方法

```java
@RequestMapping("/upload")
@ResponseBody
public Map<String,String> showFileInfo(@RequestParam("picture") MultipartFile file){
    HashMap<String,String> map = new HashMap<>();
    String name = file.getName();
    map.put("name",name);
    String filename = file.getOriginalFilename();
    map.put("fileName",filename);
    return map;
}
```

在配置文件，配置 文件解析器

```xml
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!--上传文件大小-->
    <property name="maxUploadSize" value="104857600"/>
    <!--缓存大小-->
    <property name="maxInMemorySize" value="4096"/>
</bean>
```

运行本地服务：

![upload-file](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/upload-file.png)

springmvc 篇章（三）控制器开发篇（2）完结。



参考资料：

①：<a href="https://www.cnblogs.com/wmyskxz/p/8848461.html">Spring MVC【入门】就这一篇！</a>

②：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483973&idx=2&sn=483265ffa9087ca956ec2d637119a5f8&chksm=ebd74344dca0ca5298b894fbb706c26ee942a423e858e27679f06df4b83899e1a97cc9d5eb97&scene=21###wechat_redirect">SpringMVC【开发Controller】详解</a>

③：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484008&idx=2&sn=44e15b795eda5e1f112bf663cc146bf7&chksm=ebd74369dca0ca7fedadb2835d80896df76fa5279a9db38abccceb2b25c9ee95d549cc9010ed&scene=21###wechat_redirect">SpringMVC【参数绑定、数据回显、文件上传】</a>

