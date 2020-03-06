

## spring boot 篇章（三）探索自动配置篇

#### 1、前言

在 spring boot 的篇章的入门篇时，对项目环境的搭建，无须编写各种的配置文件，也无需在 *pom.xml* 文件中，导入各种各样的依赖。

将要编写的类，建在在程序启动的同级目录或者其子目录下，使用注解后，也无需配置扫描类所在包，spring boot 也会自动将类注入到容器中。

一个带有 *@SpringBootApplication* 注解的程序启动类，运行类中的  *public static void main(String []args)* 方法，就能把整个项目启动起来。

同时 spring boot 集成其他框架，使用起来也贼方便，只需要在程序启动类，使用 *@EnableXXXXX* 注解，就能轻松整合其他框架，十分方便！

为了探究 spring boot 到底是如何实现自动配置的，因此有了本文。

#### 2、从程序启动类开始研究

```java
@SpringBootApplication
public class SpringbootApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootApplication.class, args);
    }
}
```

 可以发现，程序启动类就只单单使用 *@SpringBootApplication* 注解，点击查看

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
	//.....
}
```

在 *@SpringBootApplication* 注解中，使用其他三个重要的注解：

* *@SpringBootConfiguration* ：该注解是 spring boot 配置注解，标志这是一个 spring boot 应用，查看可以发现底层是使用了 *@Configuration* 注解，其功能就是：支持用 *JavaConfig* 的方式配置加载被注解的类。
* *@EnableAutoConfiguration* ：自动配置注解，其功能就是：自动开启配置功能（**重点**）。
* *@ComponentScan* ：该扫面注解，默认扫描该使用该注解的类所在的包和其子包，将带有注解的类，加载到 IoC 容器中。在上面中，指定了筛选条件，只有符合条件的类，才会被加载到容器中。

### 3、从注解入手

在上面中，知道了实现 spring boot 可以自动配置的重要注解是：*@EnableAuttoConfiguration* ，在 spring boot 中有一句很出名的话语，那就是 **“约定大于配置”**，其实这一句话的实现就是使用 *@EnableAutoConfiguration* 注解实现的，该注解将会自动载入 spring boot 应用程序所需要的默认配置。

* 看一下 *@EnableAuttoConfiguration*

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

在该注解中，可以看到新的两个注解和一个该注解的常量：

* *@AutoConfigurationPackage* ： 自动配置包注解，该注解的底层是使用了 *@Import* 注解
* *@Import* ：导入注解，其中能就是：指定一个类，用于给 IOC 容器导入组件。

- *ENABLED_OVERRIDE_PROPERTY* ： 用于加载 spring boot 的默认配置。

上面说了 *@AutoConfigurationPackage* 的底层使用了 *@Import* 注解，看一下该注解：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}

/****************分割线*****************************/
public @interface Import {
    Class<?>[] value(); //指定导入的类
}
```

点击 *Register.class* ,发现该类是 *AutoConfigurationPackages* 类的静态内部类

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    //在注册列表注册 Bean 的定义信息 
    public void registerBeanDefinitions(AnnotationMetadata metadata,
                                        BeanDefinitionRegistry registry) {
        AutoConfigurationPackages.register(registry, 
              (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName());
    }

    //加载程序启动类所在包的信息
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(
            new AutoConfigurationPackages.PackageImport(metadata));
    }
}
```
添加断点，调试可以看到：

![自动配置1](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE1.png)

经调试，发现 *@AutoConfigurationPackage* 注解的大致作用： 在默认的情况下就是将：将程序启动类(使用了*@SpringBootApplication* 注解的类)所在包及其子包下，使用注解的类（组件）扫描，注册 IoC 容器中。注意该  *@AutoConfigurationPackage*  和 *@ComponentScan* 的工能是不太一样的：

* *@ComponentScan* 注解的职责是：默认扫描该使用该注解的类所在的包和其子包，将带有开发中常用的*@Controller/@Service/@Component/@Repository* 等注解的类，加载到 IoC 容器中。
* *@AutoConfigurationPackage* 注解的职责是：默认扫描程序启动类所在包及其子包下，带有注解的类，加载到 IoC 容器中，这里的注解是指除 *@Controller/@Service/@Component/@Repository* 注解之外的注解，例如：使用 jpa 的 *@Entity/@Table/@JoinTable* 等注解的类（组件）。

#### 4、从类入手

在 *@EnableAuttoConfiguration* 注解上的 *@Import* 注解，指定 *AutoConfigurationImportSelector*  类，那就来看一下 *AutoConfigurationImportSelector* 类，发现该类还是蛮多方法的，根据方法名和其上注解，得知 *selectImports* 就是导入配置的主要方法 ，添加断点，调试。

![自动配置2](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE2.png)



在*getCandidateConfigurations(annotationMetadata, attributes)*方法中， 查看参数，其获取到信息如下：

![自动配置3](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE3.png)



继续断点调试，发现 spring boot 是使用 *SpringFactoryLoader* 加载器来完成

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, 
                                                  AnnotationAttributes attributes) {

    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
        this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());

    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");

    return configurations;
}

//筛选条件，只有条件符合，才会被加载
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;
}
```



继续断点调试：

![自动配置4](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE4.png)



到了这一步，spring boot 自动装配的原理就清晰了起来：

* 使用 *@Import* 注解指定 *AutoConfigurationImportSelector* 为加载资源的类
* *AutoConfigurationImportSelector*  使用 *SpringFactoryLoader*  类，扫描 spring boot 项目中所有 jar 路径下的 *META-INF/spring.factories*，将其文件中数据（key-value）封装到 *Properties* 对象中（也只有 key 是以*EnableAutoConfiguration* 结尾的数据 ，才会被封装到 *Properties* 对象中）。
* 然后将 *Properties* 对象的属性信息（key-value）存放到 map 中，然后将 map 中的信息放入到 List 中，返回给 *AutoConfigurationImportSelector* 。
* *AutoConfigurationImportSelector* 获取到 List 后，对 List 中的数据进行筛选过滤，将符合条件的数据，放入到内部类 *AutoConfigurationEntry* 中的 `List<String> configurations`，最后使用 *StringUtils* 将 list 中的数据放入到字符串数组中，并返回，spring boot 就会将获取到的字符串信息（类的全限定名），将类加载到 IoC 容器中。



有兴趣研究 spring boot 自动配置类信息的话，请自行去查看（*spring.factories* 文件只在 spring 的 jar 中）：

![自动配置5](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20boot/images/%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE5.png)





最后总结一下：

* spring boot 会使用 *@ComponentScan* 注解，扫描该注解下的类所在包和其子包，将带有 spring 所提供注解的类（组件）加载到 IoC 容器中。
* spring boot 会使用 *@EnableAuttoConfiguration* 注解，将项目启动类所在的包其子包下带有注解的类（组件）加载到 IoC 容器中。
* spring boot  在 *@EnableAuttoConfiguration* 注解使用 *@Import*  注解指定 *AutoConfigurationImportSelector*  类扫描所有 jar 路径下的 *META-INF/spring.factories*，获取其配置信息（也就是项目启动所需要的类），加载到 IoC 容器中。

该篇章完结。



参考资料：

<a href="https://docs.spring.io/spring-boot/docs/2.2.0.BUILD-SNAPSHOT/reference/html/using-spring-boot.html#using-boot-structuring-your-code">spring boot 官方文档</a>。

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484637&idx=1&sn=956c14daacc3e09367d9c27458b09f7f&chksm=ebd745dcdca0ccca6c173d32b6f8299f61d950990ee7c6eb2ec676f5ce0ad9b0ba306306a952###rd">SpringBoot自动配置原理！</a>