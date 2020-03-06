## Spring 篇章（七）事务篇（2）



#### 1、前言

在上一个篇章，介绍了 spring 事务的分类，原理，隔离和传播行为，因此在这个篇章，主要将一下 spring 的编程式事务和声明式事务，并且实践一下。 顺便说一下，事务管理都是在 service 层控制的，为啥呢？因为service层是业务逻辑层，需要调用多个 dao 层中的方法，service的方法一旦执行成功，那么说明该功能没有出错，如果出错，就会滚。

#### 2、编程式事务

编程式事务是需要开发人员在自主编写代码控制事务，如：在某个代码点手动编程开启事务，手动编程关闭事务。其优点就是：控制的事务的粒度比较细，同时也比较灵活，但是开发起来就比较繁琐，需要开发人员收懂开启，提交，关闭事务。

#### 3、声明式事务

spring 提供对事务的控制管理就叫做声明式事务控制，有两种方式实现：①：在配置文件中配置；②：在 service 层代码中使用注解配置 。spring 提供的声明式事务是基于 AOP 切面编程实现的，因此其的耦合度是比较低的，随用随配，不想使用就移除，灵活度也是不错的。但其控制事务的粒度是比较粗的（基于方法，因为切面编程拦截的是方法）。

因为 spring 提供的是声明式编程，所以其相应提供了事务的管理类，事务管理器事务管理器类又分为两种，因为JDBC 的事务和 Hibernate 的事务是不一样的。对于 Jdbc，sprig 提供的是：*DataSourceTransactionManager*；对于 Hibernate，spring 提供的是：*HibernateTransactionManager*。（mybatis 框架使用的是 jdbc 事务）

#### 4、spring 事务的实践

该实践是基于 spring 的 jdbc 的，在实践之前需要导入所依赖的包。

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

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.3.13.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.3.13.RELEASE</version>
</dependency>
<!--c3p0 数据源-->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>RELEASE</version>
</dependency>
```

**基于配置文件配置的事务管理**

编写 StudentDaoImpl 类

```java
@Repository
public class StudentDaoImpl {

    @Autowired
    private JdbcTemplate template;

    //保存
    public int save(Student student){
        return template.update("insert into student(id,age, gender, name, stu_id) values('"+student.getId()+"'," +
                "'"+student.getAge()+"','"+student.getGender()+"','"+student.getName()+"','"+student.getStuId()+"')");
    }

    //删除
    public int delete(String stuId){
        return template.update("delete from student where stu_id= "+stuId);
    }

    //查询
    public Student findStuById(String stuId){
        Student student= template.queryForObject("select * from student where stu_id= " + stuId, (resultSet, i) -> {
            Student stu = new Student();
            while (resultSet.next()) {
                stu.setName(resultSet.getString("name"));
                stu.setName(resultSet.getString("gender"));
                stu.setAge(resultSet.getInt("age"));
            }
            return stu;
        });
        return student;
    }
}
```

编写 StudentServceImpl 类

```java
@Service
public class StudentServiceImpl {
    
    @Autowired
    private StudentDaoImpl studentDao;
    
    public int save(Student student){
        return studentDao.save(student);
    }
    
    public int delete(String stuId){
        return studentDao.delete(stuId);
    }
    
    public Student findStuById(String stuId){
        return studentDao.findStuById(stuId);
    }
}
```

在配置文件中，配置事务。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx 
         http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop 
         http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="net.dao"/>

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/javaweb?serverTimezone=GMT%2B8"></property>
        <property name="user" value="root"></property>
        <property name="password" value="zy131456"></property>
        <property name="initialPoolSize" value="3"></property>
        <property name="maxPoolSize" value="10"></property>
        <property name="maxStatements" value="100"></property>
        <property name="acquireIncrement" value="2"></property>
    </bean>

    <bean class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    
	 <!--事务管理类-->
    <bean id="manager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置如何管理事务-->
    <tx:advice id="interceptor" transaction-manager="manager">
        <!--配置事务属性-->
        <tx:attributes>   
            <!--传播属性-->
            <tx:method name="*" propagation="REQUIRED"/>
            <!--是否是只读-->
            <tx:method name="*" read-only="false"/>
            <!--出现什么，进行事务回滚-->
            <tx:method name="*" rollback-for=""/>
            <!--事物隔离界别-->
            <tx:method name="*" isolation=""/>
            <!--出现什么，不进行事务回滚-->
            <tx:method name="" no-rollback-for=""/>
            <!--超时-->
            <tx:method name="" timeout=""/>
        </tx:attributes>
    </tx:advice>

     <!--配置 aop -->
    <aop:config>
        <!--切点-->
        <aop:pointcut id="pt" expression="execution(* net.dao.StudentServiceImpl.*(..))"/>
        <!--配置使用的事务管理建议-->
        <aop:advisor advice-ref="interceptor" pointcut-ref="pt"/>
    </aop:config>
</beans>
```

单元测试就不展示了。

**基于注解的事务管理**

在配置文件中，开启事务注解驱动。

```xml
<bean id="manager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!--开启事务注解，指定使用的事务管理类-->
<tx:annotation-driven transaction-manager="manager"/>
```

然后在 service 层相应的服务类中，使用 *@Transactional* 注解。

```java
@Service
@Transactional  //事务注解
public class StudentServiceImpl {

    @Autowired
    private StudentDaoImpl studentDao;

    public int save(Student student){
        return studentDao.save(student);
    }

    public int delete(String stuId){
        return studentDao.delete(stuId);
    }

    public Student findStuById(String stuId){
        return studentDao.findStuById(stuId);
    }
}
```

看一下： *@Transactional* 注解

```java
public @interface Transactional {
    //指定事务管理类
    @AliasFor("transactionManager")  
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";
	
    Propagation propagation() default Propagation.REQUIRED; //传播类型
	
    Isolation isolation() default Isolation.DEFAULT; //隔离类型
	
    int timeout() default -1;  //超时时间，-1 是无线等待
	
    boolean readOnly() default false;//是否只读

    Class<? extends Throwable>[] rollbackFor() default {};  //指定出现特定异常，回滚事务

    String[] rollbackForClassName() default {};	//指定出现哪些错误，回滚事务

    Class<? extends Throwable>[] noRollbackFor() default {};  //指定出现特定异常，不回滚事务

    String[] noRollbackForClassName() default {}; 	//指定出现哪些错误，不回滚事务
}
```

事务传播的类型，详情请看 《<a href="">Spring 篇章（六）事务篇(1)</a>》

```java
public enum Propagation {
    REQUIRED(0),  
    SUPPORTS(1),
    MANDATORY(2),
    REQUIRES_NEW(3), 
    NOT_SUPPORTED(4),
    NEVER(5),
    NESTED(6);

    private final int value;

    private Propagation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }
}
```

事务的隔离级别，详情请看 《<a href="">Spring 篇章（六）事务篇(1)</a>》

```java
public enum Isolation {
    DEFAULT(-1),  //使用数据库默认的事务隔离界别
    READ_UNCOMMITTED(1),   //读未提交
    READ_COMMITTED(2),  //读提交
    REPEATABLE_READ(4), //可重复读
    SERIALIZABLE(8);	//序列化

    private final int value;

    private Isolation(int value) {
        this.value = value;
    }

    public int value() {
        return this.value;
    }
}
```

 spring 篇章（七）事务篇（2）完结。