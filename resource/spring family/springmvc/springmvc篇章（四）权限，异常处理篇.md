## springmvc篇章（四）校验，拦截器，异常处理篇

写在前面，

#### 1、前言

在上两个篇章中，大致将了 springmvc 控制器的开发需要讲的点都讲述了一下。此篇章讲述 springmvc 是如何开发数据校验，处理异常和拦截器的。

#### 2、校验

springmvc 的检验准是以 JSR-303（JavaEE6 规范的一部分）校验规范为基准的，springmvc 使用的数据校验包是hibernate validator（其和 hibernate 的 orm 是没有关联的）。在 springmvc 中开发校验，需要在配置文件中配置相关的 bean：

* ① springmvc 的校验工厂（用于校验数据是否符合指定的规范）	
* ② 加载校验文件的资源类（加载指定数据规范的的文件）	
* ③ web 参数初始绑定器（加载校验器）	
* ④ 映射处理适配器（将请求交给控制器处理）。

导入相关依赖包

```xml
<!--校验-->
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.14.Final</version>
</dependency>

<dependency>
    <groupId>org.jboss.logging</groupId>
    <artifactId>jboss-logging</artifactId>
    <version>3.1.0.CR2</version>
</dependency>

<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
```

编写规范文件

```properties
name.length.error=名称长度的范围（1~30），请检查
email.format.error=邮件格式错误，请检查
birthday.is.notnull=出生日期不能为空，请输入
```

编写实体类和控制器

```java
public class UserEntity {

    @Size(min = 1,max = 30,message ="{name.length.error}" )
    private String username;

    private String password;

    @Email(message = "{email.format.error}")
    private String email;

    @NotNull(message = "{birthday.is.notnull}")
    private Date birthday;
    //getter and setter
}

/******************控制器**********************/
@Controller
public class ValidationController {

    @RequestMapping("/validation")
    @ResponseBody						//校验的参数必须使用 @Validated 注解修饰 
    public Map<String,String> validation(@Validated UserEntity 
                                         userEntity, BindingResult result){
        //获取所有的错误
        List<ObjectError> allErrors = result.getAllErrors();
        StringBuilder builder = new StringBuilder();
        for(ObjectError error:allErrors){
            builder.append(error.getDefaultMessage());
        }
        HashMap<String, String> map = new HashMap<>();
        map.put("error",String.valueOf(builder));
        return map;
    }

}
```

编写配置文件

```xml
<!--配置加载资源文件的 bean-->
<bean id="messageSource" class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <!--指定规范文件-->
    <property name="basenames">
        <list>
             <value>classpath:jsr</value>
        </list>
    </property>
     <property name="defaultEncoding" value="utf-8"/>
    <!-- 或者
	 <property name="fileEncoding">
  		<props>
    		 <prop key="classpath:jsr">utf-8</prop>
		</props> 
	<property>	
	-->
</bean>

<!--配置 web 属性初始绑定器-->
<bean id="bindingInitializer" class="org.springframework.web.bind.support.ConfigurableWebBindingInitializer">
    <property name="conversionService" ref="conversionService"/>
</bean>

<!--配置映射适配器-->
<bean id="mappingHandlerAdapter" class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="webBindingInitializer" ref="bindingInitializer"/>
</bean>

<!--配置 springmvc 本地校验工厂-->
<bean class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <!--校验器-->
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
    <!--加载规范文件的资源器-->
    <property name="validationMessageSource" ref="messageSource"/>
</bean>
```

如果在配置文件中，如果开启注解驱动，只需要在文件中配置加载资源文件的 bean，指定加载的规范文件便可以了（推荐使用）。

```xml
<!--开启注解驱动-->
<mvc:annotation-driven validator="validatorFactoryBean"/>
<!--开启默认处理的 servlet-->
<mvc:default-servlet-handler/>

<!--配置 springmvc 本地校验工厂-->
<bean  id="validatorFactoryBean" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
    <!--校验器-->
    <property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
    <!--加载规范文件的资源器-->
    <property name="validationMessageSource" ref="messageSource"/>
</bean>
```

>  遇到的问题：在配置 *ReloadableResourceBundleMessageSource* 时，配置的编码不对，
>
> `<property name="fileEncoding" value="utf-8"/>`造成 springmvc 相应的 json 文本是中文乱码的。
>
> 解决办法：将 fileEncoding 改为 defaultEncoding，或者这样配置 fileEncoding 
>
> ` <property name="fileEncoding">
>   	<props>
>     		<prop key="classpath:jsr">utf-8</prop>
> 	</props> 
> <property>	`

**分组校验**

* 需求

  有的时候，并不需要对们当前 JavaBean 配置的属性都进行校验，在控制器的某些方法中，只需要校验 JavaBean 的某些属性而已

* 解决办法

  使用 Hibernate Validatior 的分组校验

  hibernate validator 分组校验的步骤：① 定义分组接口（用于标识）;	② 在 JavaBean 中将要校验的数据进行分组；	③ 在控制器（Controller）的方法中使用定义好的校验分组

* 定义分组

```java
public interface FirstGroup {}
```

* 在 JavaBean 进行数据分组

```java
public class UserEntity {

    @Size(min = 1,max = 30,message ="{name.length.error}",groups = {FirstGroup.class})
    private String username;

    private String password;

    @Email(message = "{email.format.error}",groups = {FirstGroup.class})
    private String email;

    @NotNull(message = "{birthday.is.notnull}")
    private Date birthday;
	//getter and setter
}
```

* 在控制器（Controller）使用分组校验

```java
@RequestMapping(value = "/validation")
@ResponseBody
public Map<String,String> validationGroup(@Validated(value = {FirstGroup.class}) 
                                          UserEntity user,BindingResult result){
    List<ObjectError> allErrors = result.getAllErrors();
    StringBuilder builder = new StringBuilder();
    for(ObjectError error:allErrors){
        builder.append(error.getDefaultMessage()+",");
    }
    HashMap<String, String> map = new HashMap<>();
    map.put("error",String.valueOf(builder));
    return map;
}
```

#### 3、拦截器

在 springmvc 中，用户请求发起请求后，请求会被 *DispatherServlet*拦截，之后 *DispatherServlet* 调用*HandlerMapping* 查找相应的 控制器来处理请求。*HandlerMapping* 是使用拦截链（多个拦截器构成的拦截链）进行处理的。因此可以说，springmvc 的拦截器都是通过 *HandlerMapping*  来进行拦截操作的。

在 springmvc 想要自定义拦截器，只要实现 *HandlerInterceptor* ，在相应的方法中实现自己的拦截处理，然后在springmv 的配置文件中配置拦截器便可以使用了。

看一下 *HandlerInterceptor* 接口

```java
public interface HandlerInterceptor {
    
    /*在执行 控制器（Controller）的方法之前执行
      可进行用户认证，用户权限认证
      返回false表示拦截不会将请求放行，交给控制器小狐狸，返回true，表示将请求放行
    */
    boolean preHandle(HttpServletRequest var1, HttpServletResponse var2,
                      Object var3) throws Exception;

    /*在控制器执行方法的 return 之前（Controller 的方法已执行，准备 return 之前），执行该方法
      如果需要向页面提供一些公用 的数据或配置一些视图信息，使用此方法实现。需要从modelAndView入手
    */
    void postHandle(HttpServletRequest var1, HttpServletResponse var2, 
                    Object var3, ModelAndView var4) throws Exception;
	
    /* 控制器（Controller）执行完方法之后执行
     可用于 系统统一异常处理，系统日志记录，方法执行性能监控（在preHandle中设置一个时间点，在	
     afterCompletion设置一个时间，两个时间点的差就是执行时长）
    */
    void afterCompletion(HttpServletRequest var1, HttpServletResponse var2, 
                         Object var3, Exception var4) throws Exception;
}
```

编写自定义拦截器：

```java
public class FirstInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest,
                             HttpServletResponse httpServletResponse, Object o) throws Exception {
        System.out.println("FirstInterceptor：preHandle 执行了");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest,
                           HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        System.out.println("FirstInterceptor：postHandle 执行了");
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest,
                                HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {
        System.out.println("FirstInterceptor：afterCompletion 执行了");
    }
}
```

在 springmvc 配置文件中配置拦截器：

```xml
<mvc:interceptors>
    <mvc:interceptor>
        <!-- /** 可以拦截路径不管多少层-->
        <mvc:mapping path="/**"/>
        <bean class="com.zy.springmvc.interceptors.FirstInterceptor"/>
    </mvc:interceptor>
    <!--拦截是按照配置的顺序执行的-->
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="com.zy.springmvc.interceptors.SecondInterceptor"/>
    </mvc:interceptor>
</mvc:interceptors>
```

测试一下拦截器的执行顺序：

* *FirstInterceptor* 和 *SecondInterceptor* 都放行

```
FirstInterceptor：preHandle 执行了
SecondInterceptor：postHandle 执行了

SecondInterceptor：postHandle 执行了
FirstInterceptor：postHandle 执行了

SecondInterceptor：afterCompletion 执行了
FirstInterceptor：afterCompletion 执行了


总结：
执行preHandle是顺序执行。
执行postHandle、afterCompletion是倒序执行
```

- *FirstInterceptor* 放行 ，但 *SecondInterceptor* 不放行

```
FirstInterceptor：preHandle 执行了
SecondInterceptor：postHandle 执行了
FirstInterceptor：afterCompletion 执行了

总结：
如果有一个拦截器额 preHandle 不放行，那么当前拦截器的 postHandle、afterCompletion 不执行，之前的拦截器执行 afterCompletion。同时 controller 是不会执行。
```

- *FirstInterceptor* 不放行 ，但 *SecondInterceptor* 放行

```
FirstInterceptor：preHandle 执行了
```

#### 4、异常处理

对于异常，在 Java 中会将异常分为两类： ① 编译时期异常；	② 运行期异常

在以上两种异常中，对于运行运行时期的异常，开发人员是无法掌控的，只能通过代码质量、在系统测试时详细测试等方法，排除运行时期的异常。而对于编译时期的异常，则可以在代码手动处理异常可以 try/catch 捕获处理，或者向上层抛出，让上层代码进行处理。

在 springmvc 中，前端控制器 *DispatcherServlet* 拦截请求，交给 *HandlerMapping* 后、调用 *HandlerAdapter*  查找控制器 Controller，Controller 处理请求的过程中，如果遇到异常，便会使用系统中自定义统一的异常处理器，进行异常处理。

如果想自己定义异常，进行自定义异常处理，那么该怎么进行呢？springmvc统一处理异常有三种方式：

* *@ExceptionHandler* 注解

```
@Controller
public class ExceptionController {

    @ResponseBody
    @ExceptionHandler(value ={Exception.class} )
    public Map<String,String> handleException(Exception e){
        String message = e.getMessage();
        HashMap<String, String> error = new HashMap<>();
        error.put("error",message);
        return error;
    }
}
```

> *@ExceptionHandler* 注解作用于方法之上，参数是异常类型，一旦系统抛出这种类型的异常时，会引导到该方法来处理。但是它的缺陷很明显：处理异常的方法和出错的方法（或者异常最终抛出来的地方）必须在同一个Controller，不能全局控制。

* *@ControllerAdvice* 注解 + *@ExceptionHandler* 注解

  使用 *@ControllerAdvice* 和 *@ExceptionHandler* 可以全局控制异常，使业务逻辑和异常处理分隔开。

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Map<String,String> handleException(Exception e){
        String message = e.getMessage();
        HashMap<String, String> error = new HashMap<>();
        error.put("error",message);
        return error;
    }
}
```

* 实现 *HandlerExceptionResolver* 接口

```java
@Component  
public class MyExceptionResolver implements HandlerExceptionResolver{  

    //全局异常处理
    @Override
    public ModelAndView resolveException(HttpServletRequest request, 
                         HttpServletResponse response, Object handler,  Exception ex) {  
        //输出异常
        ex.printStackTrace();

        /*统一异常处理代码*/
        String message = null;  //异常信息
        MyException exception = null;
        //如果 ex 是自定义的异常，直接取出异常信息
        if(ex instanceof MyException){
            exception = (MyException)ex;
        }else{ 
            //针对非 MyException 异常，对这类重新构造成一个 MyException，异常信息为“未知错误”
            exception = new MyException("未知错误");
        }
        //错误 信息
        message = exception.getMessage();
        request.setAttribute("message", message);

        try {
            //转向到错误 页面
            request.getRequestDispatcher("/WEB-INF/jsp/error.jsp").
                forward(request, response);
        } catch (ServletException e) {   
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

        return new ModelAndView();
    }
}
```

既然 springmvc 有系统异常处理 和 开发人员自定义异常处理的两种方式，那么必然有优先级别：当系统抛出异常时，springmvc 会进行如下过程处理：

* ① springmvc 会先从配置文件中查找是否有配置异常处理的异常解析器 *HandlerExceptionResolver*（包括使用系统的异常解析器和实现接口的异常解析器）

* ② 如果有异常解析器，那么接下来就会判断该异常解析器能否处理当前发生的异常

* ③ 如果处理的话，那么编进行处理，然后将处理的结果交给视图渲染

* ④ 以上方式行不通时，就查看当前的抛出异常的 Controller 中有没有提供对应的异常处理方法，如果有，就使用 Controller 提供的方法进行异常处理，然后将处理的结果交给视图渲染。
* ⑤ 以上方式不行，springmvc 就会查看有没有使用 ControllerAdvice 注解，定义的全局异常处理器，如果有,尝试去处理异常，反之，就将异常抛出。

最后贴一下 springmvc 异常处理的过程：

![异常处理过程](https://github.com/jogin666/blog/blob/master/resource/spring%20family/springmvc/iamges/%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B.png)

​																			（图片来源参考资料）

此篇章完结。





参考资料：

<a href="https://www.cnblogs.com/chenzhubing/p/11438902.html">springMVC 统一异常处理</a>

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484008&idx=3&sn=5719448dab8d9c3d7e8c91261db4c1a2&chksm=ebd74369dca0ca7f51637c13b09579572d3e2960b5105decda4ecf7878f7b10346669f26e221&scene=21###wechat_redirect">pringMVC【校验器、统一处理异常、RESTful、拦截器】</a>