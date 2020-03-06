## springmvc 篇章（二）控制器开发篇（1）

#### 1、前言

在上一个篇章介绍了 springmvc 的入门，使用的是在配置文件中配置相应的映射处理。在这个篇章中，将使用注解的方式编写控制器，同时介绍控制器的开发。

#### 2、注解开发

**2.1 使用注解编写控制器**

继上一篇章的工程，编写一个新的控制器，并使用注解开发。

```java
@Controller
public class SimpleController {
    @RequestMapping("/hello")
    public ModelAndView handler(){
        ModelAndView mav = new ModelAndView("view.jsp");
        mav.addObject("message","hello springmvc");
        return mav;
    }
}
```

在 springmvc 的配置文件，将上一篇章中的配置全部注释掉。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!--开启注解模式-->
    <mvc:annotation-driven/>
    <!--扫描-->
    <context:component-scan base-package="com.zy.springmvc.controller"/>
</beans>
```

重新启动本地服务，在浏览器输入请求地址，结果是可以访问到的，就不展示了。

说一下在控制中使用到的两个注解：

 *@Controller*

```java
@Target({ElementType.TYPE}) //元素类型注解
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    //控制名
	String value() default "";

}
```

 *@RequestMapping*

```java
@Target({ElementType.METHOD, ElementType.TYPE}) //元素类型注解，方法类型注解
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping  //映射
public @interface RequestMapping {
    String name() default "";  //名称

    @AliasFor("path")
    String[] value() default {};  //映路径

    @AliasFor("value")
    String[] path() default {};  //映路径

    RequestMethod[] method() default {};  //Http请求方法，用于限定请求的方法类型

    String[] params() default {};  //请求参数（url后的键值对）
    /*使用方法：
    	现有 url 为 localhost8080:springmvc/hello?name=user&password=123456
    	@RequestMapping(path="/hello",param={"name=user","password=123456"})
    */

    String[] headers() default {}; //请求头

    //指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
    String[] consumes() default {};

    //指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回
    String[] produces() default {};
}

/*****************分割线*******************************/
//http 请求方法
public enum RequestMethod {
    GET,  HEAD, POST,
    PUT, PATCH, DELETE,
    OPTIONS, TRACE;

    private RequestMethod() {
    }
}
```

*@RequestMapping* 该注解不仅可以在方法上使用，还可以在在类上使用。如果  *@RequestMapping* 作用在类上，那么就相当于是给该类所有配置的映射地址前加上了一个地址（可以分模块开发）。

```java
@Controller
@RequestMapping("/simple")
public class SimpleController {
    @RequestMapping("/hello")
    public ModelAndView handler(){
       //.....
    }
}
//请求地址是 ：../simple/hello
```

> 无论基于注解和基于配置文件来开发 springmvc，都是是需要映射器、适配器和视图解析器参与工作的。  只是注解开发的映射器、适配器与配置文件开发略有不同，但是都是可以省略的。上面的配置文件中，使用  `<mvc:annotation-driven/>` 来代替了映射器和适配器的作用。

#### 3、解决中文乱码

在 springmvc  框架中，如果没有对编码进行任何的操作，那么在视图渲染的中文数据是乱码的。值得注意的是：在 springmvc 的控制器的方法中，使用 原生的 request 对象设置编码是解决不了中文乱码的问题的，原因是：springmvc 接收参数是要使用控制器中的无参构造方法的，再经过 handle() 方法的参数（参数名对应 web 端的参数名）注入 web 的参数值，如果是 model 类型，则使用 model 类型的 setter 方法注入。

springmvc 解决中文乱码问题的方法是：在 web.xml 文件中配置：

```xml
<!-- 编码过滤器 -->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>
        org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 值得注意的是：该过滤编码器只能解决POST的乱码问题

#### 4、获取 web 端参数

在上一个篇章说过，springmvc 是单例的，因此来获取 web 端的参数都是需要写在方法上的，但是没有说参数类型，现在来说一下，获取 web 端表单中的参数,，对于获取 web 端的数据，可以使用直接参数获取（方法参数名和 web 端的参数名一致），也可以使用原生的 servlet api（Request 对象） 获取，也可以使用 model 类型获取（javabean 属性与  web 端的参数名一致）。

**4.1、编写表单**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<html>
<body>
<form id="userInfoForm" name="userInfoForm" action="showUserInfo" method="post">
    <table border="1" width="350">
        <tr><td>用户名</td><td><input type="text" name="username"></td></tr>
        <tr><td>密码</td><td><input type="password" name="password"></td></tr>
        <tr><td colspan="2"><input type="submit" name="提交"></td></tr>
    </table>
</form>
</body>
</html>
```

**4.2、编写控制器**

```java
@Controller
public class UserController {

    //参数和 web 的参数名一致（简单参数）
    @RequestMapping("/showUserInfo") 
    public ModelAndView showUserInfo1(String username,String password){
        String userInfo="username: "+username+" password: "+password;
        ModelAndView mav = new ModelAndView("view.jsp");
        mav.addObject("message",userInfo);
        return mav;
    }
    
    /* 使用原生的 Servlet api
       不推荐使用，会造成耦合，建议使用 springmvc 自带的方式
    */
    @RequestMapping("/showUserInfo")
    public ModelAndView showUserIfo2(HttpServletRequest request, HttpServletResponse response){
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String userInfo="username: "+username+" password: "+password;
        ModelAndView mav = new ModelAndView("view.jsp");
        mav.addObject("message",userInfo);
        return mav;
    }

    //使用 Model 模型获取
    @RequestMapping("/showUserInfo")
    public ModelAndView showUserIfo3(UserEntity user){
        ModelAndView mav = new ModelAndView("view.jsp");
        mav.addObject("message",user.toString());
        return mav;
    }
}
/*****************分割线*******************************/
public class UserEntity {

    private String username;
    private String password;
	// getter and setter 
}
```

运行本地服务（经测试，三种方式都是成功获取到 web 端参数的）：

![springmvc 获取参数](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/springmvc%20%E8%8E%B7%E5%8F%96%E5%8F%82%E6%95%B0.png)

> 注：对于数组，springmvc 是支持的，在要再业务方法中，将参数：数组的名称和 web 端中的参数写成一致就可以了，但是 springmvc 中的业务方法是不支持 list 类型的，但是 springmvc 也给出相应的解决办法，那就是在套一层 JavaBean ，在 JavaBean 编写 list 类型（指的是泛型），并给出相应的 getter 和 setter 方法，在业务方法中，参数为上层 JavaBean 类型，springmvc 会自动注入的。对于业务方法使用到的多个 model 类型，springmvc 给出的方法也是 再套一层 JavaBean，其属性就是业务使用到多个 model 类型，同时给出相应的 getter 和 setter 方法，springmvc 会自动注入的。

#### 5、数据回显

在上一部分中，主要讲了 springmvc 获取 web 端中的参数，这部分将讲述一下 springmvc 的数据回显。

对于回显数据，springmvc 既然支持原生 servlet api 获取参数，那么也是支持 servlet api 回显数据的，可以是使用ModelAndView 对象回显数据， 也可以是 springmvc 比较推荐的使用 Molde 对象回显数据，也可以注解的方式：使用 *@ModelAttribute* 注解。

值得注意的是：以上的方式都是将数据写入到 request 对象的域中而已，然后在页面中从请求域中获取数据。

实践一下，将 UserController 控制类修改一下：

```java
@Controller
public class UserController {

    @RequestMapping("/showUserInfo")  //使用 ModelAndView
    public ModelAndView showUserInfo1(String username,String password){
        String userInfo="username: "+username+" password: "+password;
        ModelAndView mav = new ModelAndView("view.jsp");
        mav.addObject("message",userInfo);
        return mav;
    }

    @RequestMapping("/showUserInfo")  // 使用原生的 servlet api
    public String showUserIfo2(HttpServletRequest request, HttpServletResponse response){
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String userInfo="username: "+username+" password: "+password;
        request.setAttribute("message",userInfo);
        return "view.jsp";  // springmvc 的视图解析器会自动解析，找到匹配的视图
    }

    @RequestMapping("/showUserInfo")  //使用 Model
    public String showUserIfo3(UserEntity user, Model model){
        model.addAttribute("message",user.toString());
        return "view.jsp"; // springmvc 的视图解析器会自动解析，找到匹配的视图
    }
}
```

将测试，以上方式都是可以成功回显数据：

![springmvc 获取参数](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/springmvc%20%E8%8E%B7%E5%8F%96%E5%8F%82%E6%95%B0.png)

使用 *@ModelAttribute* 注解，将数据写入到请求域中

```java
//将请求的参数 使用 @ModelAttribute 注解，放到请求域中，然后在数据回显
@RequestMapping("/showUserInfo")
public String showUserInfo4(@ModelAttribute("userInfo") UserEntity userEntity){
    return "view.jsp";
}

@ModelAttribute("userInfo")  //会自动的将 map 的数据信息写入到请求域中
public Map<String, String> showUserInfo5(UserEntity userEntity){
    HashMap<String, String> userInfo = new HashMap<>();
    userInfo.put("userInfo",userEntity.toString());
    return userInfo;
}
```

看一下 *@ModelAttribute* 注解

```java
@Target({ElementType.PARAMETER, ElementType.METHOD}) //用于参数，方法注解
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ModelAttribute {
    @AliasFor("name")  
    String value() default "";

    @AliasFor("value")  //获取数据的 key
    String name() default "";

    boolean binding() default true;  //默认绑定数据
}
```

#### 6、转发与重定向

前面不管是地址 `/hello` 跳转到 `index.jsp` 还是提交表单的`showUserInfo` 跳转到 `view.jsp`，这些都是服务端的跳转，也就是 `request.getRequestDispatcher("地址").forward(request, response);`

那如何进行客户端跳转呢？那接下就演示客户端跳转：转发与重定向。

```java
@Controller
public class SimpleController {

    @RequestMapping("/hello")
    public ModelAndView handler(){
        ModelAndView mav = new ModelAndView("view.jsp");
        mav.addObject("message","hello springmvc");
        return mav;
    }

    @RequestMapping("/forward")
    public String forward(){
        return "/hello";
    }

    @RequestMapping("/redirect")
    public String redirect(){
        return "redirect:./hello";
    }
}
```

使用 `redirect:/hello` 就表示将要重定向到 `/hello` 这个路径，`/hello` 表示将请求转发到`/hello` 这个路径 ，重启服务器之后 ，在地址栏中输入：`localhost/forward` ，会自动跳转到 `/hello` 路径下：

![转发与重定向](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/%E8%BD%AC%E5%8F%91%E4%B8%8E%E9%87%8D%E5%AE%9A%E5%90%91.png)

当然对于重定向或者转发，也是可以使用 ModelAndView 类的实例完成的。

```java
@RequestMapping("/jump")
public ModelAndView jump(){
    ModelAndView modelAndView = new ModelAndView("/hello"); //转发
    // ModelAndView modelAndView = new ModelAndView("redirect:/hello"); //重定向
    return modelAndView;
}
```

#### 7、配置视图解释器

在 Spring MVC 的处理请求的流程中，视图解析器的功能就是负责定位视图，它接受一个由 DispaterServlet 传递过来的逻辑视图名来匹配一个特定的视图。

- 需求： 现在有一些页面不希望用户可以在浏览器输入访问地址，便直接访问到，例如有重要数据的页面，或者有模型数据支撑的页面。如果用户在浏览器输入地址，可以直接访问到页面的话，那么便有可能就会直接造成这会造成数据泄露，或者界面不美观（如输入：*localhost:8080/springmvc/view.jsp* 便直接可以访问 view。jsp，但是 jsp 没有从请求域中获取到数据，造成界面美观 ）...
- 解决方案：将 jsp 文件放置在【WEB-INF】文件夹下，【WEB-INF】是 Java Web 中默认的安全目录，是不允许用户直接访问的*（也就是说用户是无法通过 localhost/WEB-INF/ 这样的方式访问到的数据）*，在将 jsp 页面放置到【WEB-INF】目录下之后，需要在 springmvc 的配置文件中配置视图解析器，告诉 springmvc 视图所在的位置。

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!--前缀 视图文件所在目录的相对路径-->
    <property name="prefix" value="/WEB-INF/view/"/>
    <!--后缀 文件的扩展名-->
    <property name="suffix" value=".jsp"/>
</bean>
```

springmvc 篇章（二）控制器开发篇（1）完结。



参考资料：

①：<a href="https://www.cnblogs.com/wmyskxz/p/8848461.html">Spring MVC【入门】就这一篇！</a>

②：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483973&idx=2&sn=483265ffa9087ca956ec2d637119a5f8&chksm=ebd74344dca0ca5298b894fbb706c26ee942a423e858e27679f06df4b83899e1a97cc9d5eb97&scene=21###wechat_redirect">SpringMVC【开发Controller】详解</a>