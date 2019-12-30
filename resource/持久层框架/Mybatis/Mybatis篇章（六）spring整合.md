## Mybatis篇章（六）Spring整合

前面的五个篇章已经将mybatis的基本开发都介绍了，此篇章讲述如何将mybatis与spring整合。

新建立一个 *maven* 项目吧！

**1、引入所有的jar包**

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
    <!--mybatis 的jar包-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.3</version>
    </dependency>
    <!--mysql 数据库连接的jar包-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.18</version>
    </dependency>
</dependencies>
```

**2、创建一张表**（mysql）

```sql
create table student(
  id     int auto_increment primary key,
  userId varchar(255) null,
  name   varchar(255) null,
  gender varchar(255) null,
  age    int          null
) engine = MyISAM;
```

**3、创建表的实体类**

```java
public class Student {
    private String stuId;
    private String name;
    private int age;
    private String gender;
	//getter and setter
}
```

**4、创建表与实体的映射文件**

在 resourse 创建 studentMapper.xml文件（也可以使用注解）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="StudentMapper">
    
    <resultMap id="studentMap" type="student">
        <result property="stuId" column="stuId"/>
        <result property="age" column="age"/>
        <result property="gender" column="gender"/>
        <result property="name" column="name"/>
    </resultMap>
</mapper>
```

**5、创建mybatis的配置文件**

在 resourse 创建 mybatis-config.xml 文件，导入studentMaper.xml文件 （数据库配置交给spring容器管理）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <typeAliases>
        <typeAlias type="com.zy.bean.Student" alias="student"/>
    </typeAliases>
    <mappers>
        <mapper resource="studentMapper.xml"/>
    </mappers>
</configuration>
```

**6、配置数据库的连接信息**

在 resourse 创建 db.properties 文件，配置连接数据库的信息

```properties
driver=com.mysql.cj.jdbc.Driver
url= jdbc:mysql://localhost:3306/javaweb?allowPublicKeyRetrieval=true&useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8&useSSL=false
username=root
#你的密码
password=******
```

**7、创建spring的配置文件**

在 resourse 创建 bean.xml 文件，导入db.properties文件，然后配置管理数据库的信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx 
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd">

    <!--导入数据库配置信息-->
    <context:property-placeholder location="db.properties" />

    <!--使用c3p0 数据源管理数据库连接-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${driver}"/>
        <property name="password" value="${password}"/>
        <property name="user" value="${username}"/>
        <property name="jdbcUrl" value="${url}"/>
    </bean>

    <!--配置数据库会话工厂，加载mybatis的配置文件，管理数据库连接-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="configLocation" value="mybatis-config.xml"/>
    </bean>

    <!--配置数据域的事务管理 使用spring的jdbc，（默认是提交事务的，个人测试所得）-->
    <bean id="transactionManager" 
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置spring的事务传播类型-->
    <tx:advice transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
</beans>
```

好了项目环境搭建完成了，接下就是测试。创建 *StudentMapper* 接口并将 studentMaper.xml中<mapper>的namespace设置成接口的全限定名，改完后添加一个*select* 节点查询。

```java
public interface StudentMapper {
    Student findStudentById(String stuId);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zy.dao.StudentMapper">

    <resultMap id="studentMap" type="student">
        <result property="stuId" column="userId"/>
        <result property="age" column="age"/>
        <result property="gender" column="gender"/>
        <result property="name" column="name"/>
    </resultMap>

    <select id="findStudentById" resultMap="studentMap" parameterType="string">
        select * from student where userId=#{stuId};
    </select>
</mapper>
```

编写一个单元测试。

```java
public class StudentMapperTest {

    @Test
    public void findStudentById(){
        ClassPathXmlApplicationContext act = new ClassPathXmlApplicationContext("bean.xml");
        SqlSessionFactory factory = act.getBean("sessionFactory", SqlSessionFactory.class);
        SqlSession session=factory.openSession();
        StudentMapper mapper = session.getMapper(StudentMapper.class);
        Student student = mapper.findStudentById("20161113033");
        Assert.assertNotNull(student);
        System.out.println(student);
    }
}
```

mybatis篇章完结。



