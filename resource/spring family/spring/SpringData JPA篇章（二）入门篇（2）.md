## SpringData JPA篇章（二）入门篇（2）

在上一篇章中，使用 Spring  Boot 项目讲了如何快速入门SpringData JPA，这个入门篇的序章讲的是如何使用 

Maven工程搭建SpringData JPA环境。在慕课网上有一个视频教程：

教程地址：https://www.imooc.com/learn/821

源码下载地址:https://img.mukewang.com/down/58e60b910001594b00000000.zip



**1、创建Maven工程**

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
    <!--spring-data jpa-->
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>1.8.0.RELEASE</version>
    </dependency>
 	<!--hibernate 实体类管理者-->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>4.3.6.Final</version>
    </dependency>
	 <!--mysql 驱动器-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.15</version>
    </dependency>

    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>RELEASE</version>
        <scope>test</scope>
    </dependency>

</dependencies>
```

**2、创建数据库配置文件 *db.properties***

```properties
url=jdbc:mysql://localhost:3306/javaweb?serverTimezone=GMT%2B8
user=root
password=zy131456
classDriver=com.mysql.cj.jdbc.Driver
```

**3、创建 spring 配置文件 *bean.xml* **

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

    <!--引用db配置文件-->
    <context:property-placeholder location="classpath:db.properties"/>
   
    <!--使用注解，配置扫描路劲-->
    <context:component-scan base-package="com.zy.study"/>

    <!--1.数据源-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${classDriver}"/>
        <property name="username" value="${user}"/>
        <property name="password" value="${password}"/>
        <property name="url" value="${url}"/>
    </bean>

    <!--2. 配置spring-data jpa的实体管理器 -->
    <bean id="entityManager" 
          class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        
        <!--jpa 适配器-->
        <property name="jpaVendorAdapter">
            <!--指定orm框架使用的是 hibernate-->
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>
        </property>
       
        <!--配置实体类的扫描路径-->
        <property name="packagesToScan" value="com.zy.study"/>
        
        <!--jap的属性配置-->
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.ejb.naming_strategy">
                    org.hibernate.cfg.ImprovedNamingStrategy
                </prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <prop key="hibernate.format_sql">true</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
            </props>
        </property>
    </bean>

    <!--3. 事务管理者-->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManager"/>
    </bean>

    <!--4. 配置支持事务注解-->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <!--5. 配置dao层的扫面路径-->
    <jpa:repositories base-package="com.zy.study" 
                      entity-manager-factory-ref="entityManager"/>

</beans>
```

**4、创建实体类**

```java
@Entity(name = "student") //为实体类起个别名，然后在dao接口就不必使用实体类名了
@Table(name ="student",schema = "javaweb")
public class StudentEntity {

    private String id;
    @Id
    @GenericGenerator(name = "my-uuid",strategy = "uuid")
    @GeneratedValue(generator = "my-uuid")
    @Column(name = "id",length = 32,nullable = false)
    public String getId() {
        return id;
    }
 
    private String stuId;
    @Basic
    @Column(name ="stu_id",length = 11,nullable = false)
    public String getStuId() {
        return stuId;
    }
   
    @Basic
    @Column(name = "name",length =20,nullable = false)
    private String name;
    public String getName() {
        return name;
    }

    private String gender;
    @Basic
    @Column(name = "gender",length = 6,nullable = false)
    public String getGender() {
        return gender;
    }

    private int age;
    @Basic
    @Column(name = "age",length = 3,nullable = false)
    public int getAge() {
        return age;
    }
	//省略 setter
}
```

**5、创建Dao类**

```java
//使用注解开发，可以不用继承
//@RepositoryDefinition(domainClass = StudentEntity.class,idClass = String.class)
public interface StudentRepository extends Repository<StudentEntity,String> {

    StudentEntity findStuByStuId(String id);  //没有注解的需要按照jpa的方法命名规则

    StudentEntity findStuByName(String name);

    @Query("select stu from student stu where stu.stuId=(select max(stuId) from student t1)")
    StudentEntity findStuByMaxId();

    //演示占位符
    @Query("select stu from student stu where stu.stuId=?1")
    StudentEntity findStuById(String stuId);

    //演示参数
    @Query("select stu from student stu where stu.name=:name")
    StudentEntity findStuByParamName(@Param("name") String name);

    //使用原生的sql语句查询
    @Query(nativeQuery = true,value = "select count(1) from student")
    Integer count();


    @Query("from student")
    List<StudentEntity> findAll();

    @Modifying
    @Query("update student stu set stu.name=?1 where stu.stuId=?2")
    void updateByStuId(String name,String stuId);

}
```

单元测试：

```java
public class StudentRepositoryTest {

    public ApplicationContext act;
    public StudentRepository repository;

    @Before
    public void setUp(){
        act=new ClassPathXmlApplicationContext("bean.xml");
        repository = act.getBean(StudentRepository.class);
    }

    @After
    public void tearDown(){
        act=null;
    }

    @Test
    public void testFindStuByStuId(){

        StudentEntity stu = repository.findStuByStuId("201611130");
//        Student stu=repository.findStuByName("Tom");
        System.out.println(stu);
    }

    @Test
    public void testFindStuByMaxId(){
        StudentEntity stu = repository.findStuByMaxId();
        System.out.println(stu);
    }

    @Test
    public void testFindStuById(){
        StudentEntity stu = repository.findStuById("201611130");
        System.out.println(stu);
    }

    @Test
    public void testFindStuByName(){
        StudentEntity stu = repository.findStuByName("小强");
        System.out.println(stu);
    }

    @Test
    public void testFindStuByParamName(){
        StudentEntity stu = repository.findStuByParamName("B");
        System.out.println(stu);
    }

    @Test
    public void testCount(){
        System.out.println(repository.count());
    }

    @Test
    public void testFindAll(){
        List<StudentEntity> students = repository.findAll();
        for (StudentEntity s:students){
            System.out.println(s);
        }
    }
}
```

最后贴一张项目工程图：

![项目工程2](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/SpringData%20JPA/images/%E9%A1%B9%E7%9B%AE%E5%B7%A5%E7%A8%8B2.png)

