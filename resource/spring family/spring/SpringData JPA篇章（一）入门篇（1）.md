## SpringData JPA篇章（一）入门篇（1）

*JPA （Java Persistence API）*，是Java持久化的API，是Sun官方在JDK5.0后提出的Java持久化规范（JSR 338），其

出现的主要目的是：其目的是以官方的身份来统一各种orm框架的规范，简化持久层开发，开发者只需面向JPA规

范的接口，不用在意底层使用的ORM框架，就是面向接口编程。JPA接口所在的包的路径为：*javax.persistence*。

SpringData JPA是SpringData中的一个子模块，其底层默认使用的orm框架是hibernate，而hibernate是实现jap

规范的。关于这三者的关系，这篇文章讲得不错：《<a href="https://blog.csdn.net/a772304419/article/details/79356613">SpringData Jpa、Hibernate、Jpa 三者之间的关系</a>》。

好了回归文章主题，本章节将介绍使用 *Spring Boot* 工程项目介绍。

**1、搭建开发环境**

在IDEA下，建立一个 *Spring Boot*工程，然后在 pom文件中，导入SpringData JPA启动器。

```xml
<dependencies>
    <!--spring boot 启动器-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    
    <!--spring boot 测试启动器-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

    <!--数据库驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <!--Web必要的-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!--spring data jpa-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!--jdbc-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
</dependencies>
```

**2、在resource目下建立一个 *application.yml* 文件**

springdata jap其底层的orm框架是hibernate

```yml
#服务器配置
server:
  port: 8087
  servlet:
    context-path: /demo/

#数据库配置
spring:
  datasource:
    username: root
    password: zy131456
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/lerarnsql?serverTimezone=GMT%2B8

  #设置hibernate的参数（与datasource是同一级的）
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

    #格式化sql语句（与hibernate 是同一级的）
    properties:
      hibernate:
        format_sql: true
```

**3、创建实体类**

```java
@Entity
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

有没有感觉似曾相识。对的，与hibernate篇章—注解篇中实体类注解是一模一样的。因为hibernate是面向JPA的

，是JPA 规范的实现，所以当然是一样的啦。而至于表，在*application.yml* 文件中，已配置了 *ddl-auto: update*，

会自动生成表的，十分方便。好了，环境已经搭好了，就是编写持久层的代码了。

* 命名查询

创建一个实现 Repository<T, ID> 接口的StudentRepository接口。如果不想继承接口，则可以使用

*@RepositoryDefinition* 注解来注解 *StudentRepository* 接口。这样的目的是使用 Repository 接口的命名查询功能

```java
public @interface RepositoryDefinition {
    Class<?> domainClass(); //实体类
    Class<? extends Serializable> idClass(); //实体类的主键类型
}
```

```java
//Repository<T, ID>中的 泛型 T 是实体类，泛型 ID 是 实体类 id 的类型
@Transactional
public interface StudentRepository extends Repository<StudentEntity,String> {

    //select * from student;
    List<StudentEntity> findAll();

    //select * from student where id=?1
    StudentEntity findById(String id);

    //select * from student where age < :age;
    StudentEntity findStuByAgeLessThan(int age);

    //select * from student where name like name%;
    StudentEntity findByNameStartsWith(String name);
}
```

上述的方法，是无需编写JPQL语句，只要方法名称符合jpa的命名规则，在可以直接外界使用了。JPA在编译时，

会自动根据方法名，自动解析成sql语句的，十分方便。（JPA使用的是JPQL语言，面向对象的查询语言）

spring data支持的关键字（更多查询匹配：<a href="https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-keywords">SpringData JPA官方文档</a> 。）

| 关键字            | 方法命名                       | sql where字句              |
| :---------------- | :----------------------------- | :------------------------- |
| And               | findByNameAndPwd               | where name= ? and pwd =?   |
| Or                | findByNameOrSex                | where name= ? or sex=?     |
| Is,Equals         | findById,findByIdEquals        | where id= ?                |
| Between           | findByIdBetween                | where id between ? and ?   |
| LessThan          | findByIdLessThan               | where id < ?               |
| LessThanEquals    | findByIdLessThanEquals         | where id <= ?              |
| GreaterThan       | findByIdGreaterThan            | where id > ?               |
| GreaterThanEquals | findByIdGreaterThanEquals      | where id > = ?             |
| After             | findByIdAfter                  | where id > ?               |
| Before            | findByIdBefore                 | where id < ?               |
| IsNull            | findByNameIsNull               | where name is null         |
| isNotNull,NotNull | findByNameNotNull              | where name is not null     |
| Like              | findByNameLike                 | where name like ?          |
| NotLike           | findByNameNotLike              | where name not like ?      |
| StartingWith      | findByNameStartingWith         | where name like '?%'       |
| EndingWith        | findByNameEndingWith           | where name like '%?'       |
| Containing        | findByNameContaining           | where name like '%?%'      |
| OrderBy           | findByIdOrderByXDesc           | where id=? order by x desc |
| Not               | findByNameNot                  | where name <> ?            |
| In                | findByIdIn(Collection<?> c)    | where id in (?)            |
| NotIn             | findByIdNotIn(Collection<?> c) | where id not  in (?)       |
| True              | findByAaaTue                   | where aaa = true           |
| False             | findByAaaFalse                 | where aaa = false          |
| IgnoreCase        | findByNameIgnoreCase           | where UPPER(name)=UPPER(?  |

命名查询是其也是有弊端的，①：只支持查询；② 约定大于配置，会造成方法名比较长；如 ：

*findByNameAndIdAndAgeLessThan(.......);*   ③：难以实现复杂查询。

* @query注解解析

```java
public @interface Query {
    String value() default ""; //JPQL

    String countQuery() default ""; //定义一个用于分页查询总数的查询语句
    /*
    @Query(value = "SELECT * FROM USERS WHERE LASTNAME = ?1",
    	countQuery = "SELECT count(*) FROM USERS WHERE LASTNAME = ?1",nativeQuery = true)
 	 Page<User> findByLastname(String lastname, Pageable pageable);
    */

    String countProjection() default ""; //定义为分页而生成的计数查询的投影部分

    boolean nativeQuery() default false; //是否使用原生的sql

    String name() default "";  //名字，默认使用实体类名.@Query的方法名字

    String countName() default ""; //定义查询总数的结果名称，方便分页使用
}
```

* @query注解中使用的JPQL语言，其语法和HQL语法无多大差别。

```java
//参训全部
@Query("select stu from StudentEntity stu")
//@Query(value = "from StudentEntity ")
List<StudentEntity> findAllStu();

//使用占位符匹配参数
@Query("select stu from StudentEntity stu where stu.name=?1 and age=?2")
StudentEntity findStuByNameAndAge(String name,int age);

@Modifying   //参数为实体类的匹配
@Query("update StudentEntity stu set stu.name=:#{#entity.name},age=:#{#entity.age}," +
            "gender=:#{#entity.gender} where stuId=:#{#entity.stuId}")
void updateStuById(@Param("entity") StudentEntity entity);

//使用命名参数    
@Modifying
@Query("delete from StudentEntity where stuId=:stuId")
void deleteById(@Param("stuId")String stuId );

//使用原生的sql
@Modifying
@Query(nativeQuery = true,
      value = "insert into student(stu_id,age,gender,name,id)values(:#{#entity.stuId}," +
       ":#{#entity.age},:#{#entity.gender},:#{#entity.name},:#{#entity.id})"
void save(@Param("entity")StudentEntity entity);
```

值得注意的是：JPQL是不支持 insert 语句的，同时，**增删改的方法必须使用 *@Modifying* 注解**，负责会抛出异

常。在使用增删改方法的服务类，如*Service*层记得加上**事务 *@Transactional* 注解**。

最后 贴一下项目工程目录：

![项目工程1](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/SpringData%20JPA/images/%E9%A1%B9%E7%9B%AE%E5%B7%A5%E7%A8%8B1.png)



参考资料：

<a herf="https://www.jianshu.com/p/d5961725af82">SpringData--简介(一)</a>

