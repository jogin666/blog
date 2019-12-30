## Hibernate篇章（一）入门篇



### 一、前言 

其实本来不想写hinernate框架篇章的，因为现在普遍使用的都是mybatis和springdata jpa的，但由于在某个Java

群，看到有个五年Java经验的人说，hibernate框架仍需要了解和掌握，遂而想想还是应该要回顾一下。



### 二、介绍Hibernate

**2.1、hibernate是什么东东？**

同mybatis一样，hibernate也是一种ORM框架，用来实现Java实体对象与数据库之间建立某种映射，将数据库

表中的数据直接映射到Java的实体类对象中。常用与mvc框架中的持久城，与数据库打交道嘛。



**2.2、说一下ORM框架**

在上一个篇章中，没有对ORM框架进行解说，现在在这个篇章解说一下。ORM：Object—Relative—Database—

Mapping。其中主要作用：就是实现对象与数据库表中数据的映射，将数据封装到对象的属性中。



### 三、Hibernate快速入门



**3.1、创建maven工程**

使用idea建立一个maven工程，在配置文件pom.xml文件中到如hibernate的依赖包和单元测试包，如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zy</groupId>
    <artifactId>hibernate</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--hibernate 核心包-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>5.3.10.Final</version>
        </dependency>

        <!--单元测试包-->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.5.2</version>
        </dependency>

        <!--mysql 驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.17</version>
        </dependency>
    </dependencies>


</project>
```



**3.2、创建一张表**（mysql）

```sql
create table student(
  id     int auto_increment primary key,
  userId varchar(255) null,
  name   varchar(255) null,
  gender varchar(255) null,
  age    int          null
)
  engine = MyISAM;
```



**3.3、创建实体类**

```java
public class Student {

    private int id;
    private String userId;
    private String name;
    private String gender;
    private int age;
	//getter and getter
}
```



**3.4、在resource目录下创建实体与表的映射文件**

创建 *student.hbm.xml* 映射文件，文件必须是以 *.hbm.xml* 结尾，映射文件模板可在导入的依赖包或者官网查询。

在配置实体类与表的映射关系时，必须要主键。

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
    <!--将实体类与表建立管理映射-->
    <class name="com.zy.hibernate.entity.Student" table="student">
        <!--主键，采用自增策略，必须要有主键-->
        <id name="id" column="id">  
            <generator class="native"/>
        </id>
        <!--实体对象的属性对应的字段-->
        <property name="userId" column="userId" type="java.lang.String"/>
        <property name="gender" column="gender" type="java.lang.String"/>
        <property name="name" column="name" type="java.lang.String"/>
        <property name="age" column="age" type="java.lang.Integer"/>
    </class>
</hibernate-mapping>
```



**3.5、在resource目录下创建hibernate的配置文件**

创建 *hibernate.cfg.xml* 映射文件，文件必须是以 *.cfg.xml* 结尾，配置文件模板可在导入的依赖包或者官网查询。

并在文件中导入实体类与表的映射文件。

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>

    <!--会话工程-->
    <session-factory>
        <!--控制台显示sql-->
        <property name="hibernate.show_sql">true</property>
        <!--使用mysql的方言-->
        <property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>
        <!--用户名与密码-->
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">zy131456</property>
        <!--连接的urk-->
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/javaweb?serverTimezone=GMT%2B8</property>
        <!--sql格式化-->
        <property name="hibernate.format_sql">true</property>
        <!--数据库的驱动-->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        
        <!--数据源-->
 		<!--<property name="hibernate.connection.provider_class">
				org.hibernate.connection.C3P0ConnectionProvider
		</property>-->
        <property name="hibernate.c3p0.min_size">2</property>
        <property name="hibernate.c3p0.max_size">4</property>
        <property name="hibernate.c3p0.timeout">5000</property>
        <property name="hibernate.c3p0.max_statements">10</property>
        <property name="hibernate.c3p0.idle_test_period">30000</property>
        <property name="hibernate.c3p0.acquire_increment">2</property>
        
        <!--实体与表的映射文件-->
        <mapping resource="student.hbm.xml"/>
    </session-factory>
    
</hibernate-configuration>
```



**3.6、编写加载配置文件的工具类**

获取hibernate的信息，管理数据库的会话和事务。

```java
package com.zy.hibernate.configuration;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class HibernateConfig {

    private static SessionFactory factory; //会话工程
    private static Transaction transaction; //事务管理
    //使用ThrealLocal管理数据库会话
    private static final ThreadLocal<Session> threadLocal=new ThreadLocal<>();

    // 创建数据库会话工厂
    static {
        Configuration configuration=new Configuration();
        //没有参数，认加载src目录下的 hibernate.cfg.xml文件
        configuration.configure();
        //创建会话工程
        factory=configuration.buildSessionFactory();
    }

    public static Session getSession(){ //开启会话
        Session session = threadLocal.get();
        if (session==null){
            session=factory.openSession();
            threadLocal.set(session);
        }
        return session;
    }

    public static void close(){ //关闭会话
        Session session = threadLocal.get();
        if (session!=null){
            session.close();
            threadLocal.remove(); //会话已关闭，无需保留
        }
    }

    public static Session openTransaction(){ //同时开启会话和事务
        Session session = threadLocal.get();
        if (session==null){
            session=factory.openSession();
            transaction= session.beginTransaction(); 
            threadLocal.set(session);
        }
        return session;
    }

    public static void commitAndClose(){ //关闭会话和事务
        Session session = threadLocal.get();
        if (session!=null && transaction!=null){
            transaction.commit();
            session.close();
            threadLocal.remove(); //会话已关闭，无需保留
        }
    }
}
```



**3.7、创建dao类**

为了减少麻烦，没有创建接口.....值得注意的是：hibernate的增删更查都是使用主键的是实现，上边也说了，在配

置表与实体的关联映射时，必须要有主键。

```java
package com.zy.hibernate.dao;

import com.zy.hibernate.configuration.HibernateConfig;
import com.zy.hibernate.entity.Student;
import org.hibernate.Session;
import java.io.Serializable;

public class StudentDaoImpl {

    public boolean save(Student student){ //添加
        Session session = HibernateConfig.openTransaction();
        Serializable id = session.save(student);
        HibernateConfig.commitAndClose();
        if (id!=null){
            return true;
        }
        return false;
    }

    public void delete(int id){ //删除
        Student stu = findById(id);
        Session session = HibernateConfig.openTransaction();
        session.delete(stu);
        HibernateConfig.commitAndClose();
    }

    public Student findById(int id){ //查询
        Session session = HibernateConfig.getSession();
        Student student = session.find(Student.class, id);
        HibernateConfig.close();
        return student;
    }

    public void update(Student student){ //更新
        Session session = HibernateConfig.openTransaction();
        session.update(student);
        HibernateConfig.commitAndClose();
    }
}
```



**3.8、单元测试**

```java
class StudentDaoImplTest {

    private StudentDaoImpl studentDao;

    @BeforeEach
    void setUp() {
        studentDao=new StudentDaoImpl();
    }

    @Test
    void save() {
        Student student = new Student(); 
        student.setAge(18);
        student.setName("test");
        student.setGender("male");
        student.setUserId("000001");
        studentDao.save(student); //保存，主键自动增长
    }

    @Test
    void delete() {
        studentDao.delete(5); //根据id删除
    }

    @Test
    void findById() {
        Student student = studentDao.findById(5); //根据id查询
        System.out.println(student);
    }

    @Test
    void update() {
        Student stu = studentDao.findById(5);  //根据id查询
        stu.setGender("female");
        studentDao.update(stu);
    }
}
```

结果就不展示了，有兴趣的读者，请自行编写测试。



### 四、细节再讲解

**4.1、加载配置文件**

```java
// 创建数据库会话工厂
static {
    Configuration configuration=new Configuration();
    //没有参数，认加载src目录下的 hibernate.cfg.xml文件
    configuration.configure();
    //创建会话工程
    factory=configuration.buildSessionFactory();
}
```

如果没有指定加载配置文件，也就是无参数的话，默认加载src目下以 *.cfg.xml* 结尾的文件，如要指定参数，则需

要指定文件的全路径名称。



**4.2、查询和更新操作**

4.2.1 查询

上面说了，hibernate是通过主键实现crud操作的。查询时，hibernate会根据主键查询对应的记录，然后将查

询到据封装成实体类的对象。hibernate常用的两种查询方式如下：

- *get(Class<T> clazz,Object id)*
- *load(Class<T> clazz,Object id)*     支持懒加载



4.2.2 常用更新方法（实体对象没有设置主键且不采用策略，会抛出异常）

- *save(Object obj)*   【保存一个对象】

- *update(Object obj)* 【更新一个对象】

- *saveOrUpdate(Object obj)*  【保存或者更新的方法】, 参数：实体类如果没有设置主键值，则执行保存； 实体

  类对象有设置主键值，则执行更新操作; 



4.2.3 hibernate的事务

```java
public Session example(){
    Session session = threadLocal.get();
    if (session==null){
        session=factory.openSession();
        transaction= session.beginTransaction();  //开启事务
        /*
        	transaction=session.getTransaction(); //获得事务
        	transaction.begin(); //然后开启事务
             */
        threadLocal.set(session);
    }
    return session;

```



4.2.4 加载映射文件

hibernate提供在配置文件在家映射文件和手动加载文件两种方式。其中手动加载配置文件，也就是在代码中显示

加载关系映射文件，这样方式要求映射文件必须要和实体在同一个包，其文件名必须要和实体类名一致。一般用测

试。但值得注意的是：而maven工程，必须要把映射文件方法放入到resource目录下，所以maven工程无法使用

（个人测试所得结果，可能不准确）。

```java
public void example(){
    Configuration configuration = new Configuration();
    configuration.configure().addClass(Student.class);
}
```



### 五、Hibernate的HQL，QBC，SQL

**5.1、hql语言**

HQL： hibernate—query—language ，是hibernate提供的面向对象的查询语言，是面向对象和对象属性的查询

语言， HQL 查询被 Hibernate 翻译为原生的 SQL 语句然后向数据发起请求执行sql语句的操作。

```java
public void example(){
    Session session = HibernateConfig.getSession();
    Query quer1=session.createQuery("from Student"); //查询全部
    query1.list();
    //条件查询
    Query query2 = session.createQuery("from Student as stu where stu.id=:id");
    query2.setParameter("id",123);
    /*
     Query query2 = session.createQuery("from Student as stu where stu.id=?1");
     query2.setParameter(1,5);
     
     Query query3=session.createQuery("select stu from Student stu where stu.id=?1");
     query3.setParameter(1,5);
    */   
    query2.uniqueResult();
}
```

- hql的语法和sql语法没有多大差别，一般hql的格式为：***from 实体类名 [别名]  [条件]*  ; 或者 *select 别名 from***

  ***实体类名 别名 [条件]* **。查询单个字段的话，则使用 **别名 . 属性名 ** 。值得注意的是：**hibernate是不支持 **

  **select * 的格式，查询全部必须要使用  select 别名** 。

- hibernate是支持参数化的，有两种参数化方式。**①：使用参数别名   *: paramterName*  ，在 *setParamter* **

  **方法使用时，其中第一个参数为 *paramterName*，第二个参数为 参数值。  ②：使用占位符   *?1*   ,然后在 **

  ***setParamter* 方法的一个参数为占位符 1，第二个参数为 参数值。 占位符是从 1 开始的**。

- hql是支持三种连接查询的，和sql无多大差别。

```java
void example(){
    Session session = HibernateConfig.getSession();
    //内连接
    Query query1 = session.createQuery("from Student stu inner join Card card on stu.sId=card.id");
    //左外连接
    Query query2 = session.createQuery("from Student stu left join Card card on stu.sId=card.id");
    //右外连接
    Query query3 = session.createQuery("from Student stu right join Card card on stu.sId=card.id");
}
```

- hql的迫切连接查询，弥补了连接查询返回的Object数组。其迫切就是将符合条件的右表数据结合左表的数

  据，或者将符合条件的左表数据结合右表数据，封装成对象，然后返回。（前提是两表符合条件的字段，可

  以构成对象）

```java
void example(){
    Session session = HibernateConfig.getSession();
    Query query1 = session.createQuery("from Student s left join fetch Card c on s.sId=c.id");
    Query query2 = session.createQuery("from Student s right join fetch Card c on s.sId=c.id");
}
```

- hql的select查询，如只是查询几个字段，则返回的是Object数组，不会封装成对象的，但hql提供了将Object

  数组封装成对象的功能，但要有相应的构造函数提供。

```java
Query query = session.createQuery("select new Student(stu.name,stu.age )from Student stu");
```

- hql的聚合函数查询

```java
void example(){
    Session session = HibernateConfig.getSession();
    Query query = session.createQuery("select count(*) from Student");
    long count = (long) query.uniqueResult();
}
```

- hql 还支持命名查询，将常用的hql语句写在关联映射文件中，然后在程序中调用

```xml
<query name="findStudents">
    <![CDATA[ from Student stu where  stu.age<:age]]>  <!--特殊字符，需要转义-->
</query>
```

   程序调用：

```java
void example(){
    Session session = HibernateConfig.getSession();
    Query query = session.getNamedQuery("findStudnts");
}
```

- hql的分页

```java
void example(){
    Session session = HibernateConfig.getSession();
    Query query = session.createQuery("from Student");
    //获得滚动集
    ScrollableResults scroll = query.scroll();
    scroll.last();
    int count= scroll.getRowNumber() + 1; //获取总数

    //获取1-20 的记录
    query.setFirstResult(0);
    query.setMaxResults(20);
    List list = query.list();
}
```



**5.2、QBC查询**

QBC查询: query—by—criteria，完全面向对象的查询。其已经被淘汰了，一般都是使用hql的。

```java
void example(){
    Session session = HibernateConfig.getSession();
    Criteria criteria = session.createCriteria(Student.class);
    criteria.add(Restrictions.eq("id", 5));
    List list = criteria.list();
}
```



**5.3、本地sql 查询**

hibernate默认是使用hql查询的，但是也支持sql查询，但是编写的sql不能跨平台使用，因为配置文件制定了制定

数据库的方言。仅能对相应的数据库起作用。

```java
void example(){
    Session session = HibernateConfig.getSession();
    NativeQuery query = session.createSQLQuery("select * from student");
    //指定将数据封装成的实体类型
    query.addEntity(Student.class); //将查询的结果封装的类型，否则返回的是Object数组
    query.list();
}
```

最后贴一张 工程目录图：

![hibernate(1)工程目录](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Hibernate/images/hibernate(1)%E5%B7%A5%E7%A8%8B%E7%9B%AE%E5%BD%95.png)



参考资料：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=6&sn=e6b5be7f205ac5765d086966e446aeff&chksm=ebd74303dca0ca15bd553410e5e76c8e1457f175faeb9c61bc6a526977c98375242f1d2ab258&scene=21###wechat_redirect">Hibernate【查询详解、连接池、逆向工程】</a>

