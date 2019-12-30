## Hibernate篇章（六）Spring整合篇章

前面五个篇章已经将hibernate的基本开发都已经讲完了，此篇章将讲述spring是如何整合hibernate的。

重新创建一个maven工程吧。

**1、在pom文件导入依赖包**

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
    <!-- spring 核心包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.2.2.RELEASE</version>
    </dependency>
    <!-- spring bean包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
        <version>5.2.2.RELEASE</version>
    </dependency>
    <!--spring 上下文包-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.2.RELEASE</version>
    </dependency>
    <!-- spring 关系映射包 -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-orm</artifactId>
        <version>5.2.2.RELEASE</version>
    </dependency>
    <!-- spring 事务管理-->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-tx</artifactId>
        <version>5.2.2.RELEASE</version>
    </dependency>
    <!-- c3p0 数据源-->
    <dependency>
        <groupId>com.mchange</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.5.4</version>
    </dependency>
    <!--hibernate 的jar包-->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>RELEASE</version>
    </dependency>
    <!--mysql 数据库连接的jar包-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.18</version>
    </dependency>

    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjrt</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>RELEASE</version>
    </dependency>
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>RELEASE</version>
    </dependency>

</dependencies>
```

**2、创建student表**

```sql
create table student(
  sid     int auto_increment primary key,
  userId  varchar(255) null,
  name    varchar(255) null,
  gender  varchar(255) null,
  age     int          null,
  classId int          null,
  constraint student_class_id_fk foreign key (classId) references class (id)
);
```

**3、使用发现代理创建实体与关系映射文件**

* 实体类

```java
@Entity
@Table(name = "student", schema = "javaweb")
public class StudentEntity {
    private int sid;
    private String userId;
    private String name;
    private String gender;
    private Integer age;

    @Id
    @Column(name = "sid", nullable = false)
    public int getSid() {
        return sid;
    }

    public void setSid(int sid) {
        this.sid = sid;
    }

    @Basic
    @Column(name = "userId", nullable = true, length = 255)
    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    @Basic
    @Column(name = "name", nullable = true, length = 255)
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Basic
    @Column(name = "gender", nullable = true, length = 255)
    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    @Basic
    @Column(name = "age", nullable = true)
    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
    //equal and hashCode
}
```

* 关系映射文件

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>

    <class name="com.zy.sh.entity.StudentEntity" table="student" schema="javaweb">
        <id name="sid">
            <column name="sid" sql-type="int(11)"/>
        </id>
        <property name="userId">
            <column name="userId" sql-type="varchar(255)" not-null="true"/>
        </property>
        <property name="name">
            <column name="name" sql-type="varchar(255)" not-null="true"/>
        </property>
        <property name="gender">
            <column name="gender" sql-type="varchar(255)" not-null="true"/>
        </property>
        <property name="age">
            <column name="age" sql-type="int(11)" not-null="true"/>
        </property>
    </class>
</hibernate-mapping>
```

**4、创建 StudentDao接口和实体类**

```java
public interface StudentDao {

    void save(StudentEntity stu);
}
/***********************************分割线***************************/
public class StudentDaoImpl implements StudentDao{

    private SessionFactory factory;

    public void save(StudentEntity stu) {
        Session session = factory.openSession();
        Transaction transaction = session.beginTransaction();
        session.save(stu);
        transaction.commit();
        session.close();
    }

    public SessionFactory getFactory() {
        return factory;
    }

    public void setFactory(SessionFactory factory) {
        this.factory = factory;
    }
}
```

**5、创建StudentService接口和实现类**

```java
public interface StudentService {

    void save(StudentEntity stu);
}
/***********************************分割线***************************/
public class StudentServiceImpl implements StudentService {

    private StudentDao studentDao;


    public void save(StudentEntity stu) {
        studentDao.save(stu);
    }

    public StudentDao getStudentDao() {
        return studentDao;
    }

    public void setStudentDao(StudentDao studentDao) {
        this.studentDao = studentDao;
    }
}
```

好了，以上准备得拆不多了。接下就是spring整合hibernate了。spring整合hibernate有两种方式，一种是数据源

和实体类关系映射文件仍交给hibernate的配置文件管理，Spring只管理数据库会话工厂。另外一种就是Spring全

盘接手，不需要在编写hibernate的配置文件。

**6、整合**

* 使用第一种方式，spring只负责管理会话工厂，事务啥的都交给hibernate管理。 

spring配置文件 （bean1.xml）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="sessionFactory" 
          class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="configLocation" value="hibernate.cfg.xml"/>
    </bean>

    <bean class="com.zy.sh.dao.StudentDaoImpl" id="studentDao">
        <property name="factory" ref="sessionFactory"/>
    </bean>

    <bean class="com.zy.sh.service.StudentServiceImpl" id="studentService">
        <property name="studentDao" ref="studentDao"/>
    </bean>
</beans>
```

hibernate的配置文件

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--控制台显示sql-->
        <property name="hibernate.show_sql">true</property>
        <!--使用mysql的方言-->
        <property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>
        <!--用户名与密码-->
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">zy131456</property>
        <!--连接的url-->
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/javaweb?serverTimezone=GMT%2B8</property>
        <!--sql格式化-->
        <property name="hibernate.format_sql">true</property>
        <!--数据库的驱动-->
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>

        <property name="hibernate.hbm2ddl.auto">update</property>
        <mapping resource="StudentEntity.hbm.xml"/>
        <mapping class="com.zy.sh.entity.StudentEntity"/>
    </session-factory>
</hibernate-configuration>
```

创建单元测试类：

```java
class StudentServiceImplTest {

    private ApplicationContext act;
    @BeforeEach
    void setUp() {
        act=new ClassPathXmlApplicationContext("bean.xml");
    }

    @Test
    void save() {
        StudentEntity entity=new StudentEntity();
        entity.setAge(22);
        entity.setGender("female");
        entity.setUserId("666666");
        entity.setName("entity");
        StudentService service = act.getBean(StudentServiceImpl.class, "studentService");
        service.save(entity);
    }
}
```

* spring全盘接管hibernate的责任，不需要编写hibernate的配置文件。

spring配置文件（bean2.xml）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx 
       http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop 
       https://www.springframework.org/schema/aop/spring-aop.xsd">
	
    <context:property-placeholder location="db.properties"/>
    <!--c3p0数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${driver}"></property>
        <property name="jdbcUrl" value="${url}"></property>
        <property name="user" value="root"></property>
        <property name="password" value="zy131456"></property>
        <property name="initialPoolSize" value="3"></property>
        <property name="maxPoolSize" value="10"></property>
        <property name="maxStatements" value="100"></property>
        <property name="acquireIncrement" value="2"></property>
    </bean>

    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="hibernateProperties">
            <!--接管hibernate配置-->
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
            </props>
        </property>
        <!--接管映射文件-->
        <property name="mappingLocations">
            <list>
                <value>StudentEntity.hbm.xml</value>
            </list>
        </property>
    </bean>

    <!--事务管理，没有采用注解事务的话，需要手动提交事务，和mybatis不一样，因为事务管理者不一样-->
    <bean id="manager"
          class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>

    <!--代理-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>

    <!--开启注解事务-->
    <tx:annotation-driven transaction-manager="manager"/>
    <!--扫描指定的包-->	
    <context:component-scan base-package="com.zy.sh.*"/>
</beans>
```

db.properties文件

```properties
driver=com.mysql.cj.jdbc.Driver
url= jdbc:mysql://localhost:3306/javaweb?allowPublicKeyRetrieval=true&useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8&useSSL=false
username=root
#你的密码
password=zy131456
```

dao的接口实现类（采用注解）

```java
@Repository("studentDao")
public class StudentDaoImpl implements StudentDao{

    @Autowired
    private SessionFactory factory;

    public void save(StudentEntity stu) {
        Session session = factory.openSession();
        session.save(stu);
    }
}
```

service接口的实现类（采用注解）

```java
@Transactional //使用事务注解，spring采用ThreadLocal存放session，会自动开启和提交事务，不必手动设置
@Service("studentService")
public class StudentServiceImpl implements StudentService {

    @Autowired
    private StudentDao studentDao;

    public void save(StudentEntity stu) {
        studentDao.save(stu);
    }

    public StudentDao getStudentDao() {
        return studentDao;
    }
}
```

单元测试

```java
class StudentServiceImplTest {

    private ApplicationContext act;

    @BeforeEach
    void setUp() {
        act=new ClassPathXmlApplicationContext("bean2.xml");
    }

    @Test
    void save() {
        StudentEntity entity=new StudentEntity();
        entity.setAge(22);
        entity.setGender("female");
        entity.setUserId("666666");
        entity.setName("entity");
        StudentServiceImpl service = act.getBean("studentService", 
                                                 StudentServiceImpl.class);
        System.out.println(service);
        service.save(entity);
    }

}
```



最后贴一张工程结构图：

![spring-hibernate](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Hibernate/images/spring-hibernate.jpg)
