## Mybatis篇章（一）入门篇



### 一、Mybatis介绍

**1.1、Mybatis是什么东东？**

MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google 

code，并且改名为MyBatis。是一个基于Java的持久层框架，也是ORM的一种实现框架，封装数据库底层的JDBC

操作。

**1.2、为什么使用Mybatis**

现有市面上的持久层框架存在较多的缺陷，如：hibernate框架，在处理简单的业务时，hibernate提供的hql语

句，减少了很多开发人员的工作量，用起来十分舒服。但是：在处理复杂的业务时，其缺陷就很容易被放大，hql

是很难写也很难理解的，不想传统的sql语句用起来方便。原生JDBC呢？其过程很容易简单，固定了步骤，但开发

人员开发起来麻烦，维护也不方便（需要在dao层中编写sql语句）。而dbutils和springDao类似，对JDBC进行了

一层封装，比原生JDBC好用些，但差不了多少。为了解决上述的问题，于是Ibatis框架被开发出来了（mybatis前

身），其在hibernate框架和jdbc中取得了平衡点，深受广大开发人员的喜爱。



### 二、Mybatis入门

**1. 建立maven工程**

建立maven工程，在配置文件pom.xml中导入mybatis的依赖包和单元测试包。

```xml
<dependencies>
    <!--单元测试-->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
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

    <!--动态代理包-->
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>3.2.10</version>
    </dependency>

</dependencies>
```

**2. 创建一张测试表**（mysql）

```sql
create table mybatis(
  Id     int auto_increment primary key,
  name   varchar(20) charset utf8 null,
  gender char(2) charset utf8     null,
  stuID  varchar(11)              null
);
```

**3.在resource目录下创建mybatis的配置文件和连接数据库的必备信息的db.properties**

- mybatis的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration> 
     <!--导入数据配置文件-->
    <properties resource="db.properties"/>
    <!--配置数据库环境-->
    <environments default="mysql"> <!--指定默认连接环境-->
        <!--环境1-->
        <environment id="mysql">
            <transactionManager type="jdbc"></transactionManager> <!--jdbc 事务管理-->
            <dataSource type="pooled">
                <property name="driver" value="${driver}"/>         <!--数据库驱动-->
                <property name="url" value="${url}"/>               <!--连接数据库的url-->
                <property name="username" value="${username}"/>     <!--数据库的用户名-->
                <property name="password" value="${password}"/>     <!--密码-->
            </dataSource>
        </environment>
        <!--环境2-->
        <environment id="mysql2">
            <transactionManager type="jdbc"></transactionManager>
            <dataSource type="pooled">
                <property name="driver" value=""/>
                <property name="url" value=""/>
                <property name="username" value=""/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>
</configuration>
```

简单说一下：①：envrionments是配置数据库环境的，其内部可以配置多个环境，但是需要在default指定默认使

用的环境  ②：environment的 id 是唯一的，不可重复，其内部指定了事务管理方式，数据源类型，和连接数据库

的必备的信息。如果不懂有什么，可以crtl+左键，点击节点进入文件浏览。

- 数据库连接信息 db.properties文件

```properties
driver=com.mysql.cj.jdbc.Driver
url= jdbc:mysql://localhost:3306/javaweb?allowPublicKeyRetrieval=true&useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8&useSSL=false

username=root
#你的密码
password=******  
```

**4、测试数据库连接**

- 创建读取mybatis配置文件的工具类

```java
public class MybatisConfiguration {
	//让线程拥有自己的变量
    private final static ThreadLocal<SqlSession> threadLocal=new ThreadLocal<>();
    private static SqlSessionFactory factory; //数据库连接会话工厂

    static {	//类初始就加载，
        try{
            //获取指定文件的输入流
            Reader reader= Resources.getResourceAsReader("mybatis-configuration.xml");
            //创建SqlFactoryBuilder的实例
            SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
            //获取会话工厂
            factory=builder.build(reader);
        }catch (Exception e){
            e.printStackTrace();
            System.out.println("获取mybatis配置文件失败。");
        }
    }
	//获取连接会话
    public static SqlSession getSession(){
        SqlSession session=threadLocal.get();
        if (session==null)
            session=factory.openSession();
        return session;
    }
	//关闭连接会话
    public static void close(){
        SqlSession session=threadLocal.get();
        if (session!=null){
            session.close(); //返回连接池
            threadLocal.remove(); 
        }
    }
}
```

- 创建Junit单元测试（测试项目环境是否部署成功）

```java
@Test
public void getSession() {
    Connection connection = MybatisConfiguration.getSession().getConnection();
    Assert.assertNotNull(connection);
    System.out.println("连接成功"); //输出成功连接
}
```

好了，项目环境已经好了，接下来就是mybatis的操作入门了。



**5、创建实体类**

```java
public class Student {

    private String stuId;
    private String name;
    private String gender;

    public Student() {
    }

    public Student(String stuId, String name, String gender) {
        this.stuId = stuId;
        this.name = name;
        this.gender = gender;
    }
    //getter and setter ....
}
```



**6、创建表与实体的映射文件**（处理表与实体之间的映射，和sql语句的编写）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper>
</mapper>
```

做完以上六个步骤之后，现在来到了如何对数据库表里的数据进行操作的步骤了。Mybatis对此步骤提供了两种常

见的方式，这两种方式都是基于代理实现的。

- **Mapper代理方式**，编写到接口，但不用写dao的实现类。在mapper文件（处理实体与表之间的映射）的

  *<mapper>*节点的属性namesapce 的值指定为接口全限定名（将接口的代理者与sql语句的执行者绑在一起

  ）。**值得注意的是**：*select，update* 等节点的id必须要dao接口的方法名一 一 对应，参数类型和返回类型需

  要一致，还有mybatis默认是开启事务的，但不会提交事务，需要手动提交。

1. 编写接口

```java
public interface StudentMapper {

    Student findStuById(String stuId); //根据id查询学生

    List<Student> findAll(); //查询全部学生

    void updateStuById(Map<String,String> conditions);	//根据id，根性学生姓名

    void deleteStuById(String stuId); //根据id删除学生

    Student findStu(Map<String,Object> conditions); //多个条件值查询学生

    void insertStu(Student stu); //新增一个学生
}
```

2. 编写mapper映射文件 

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">       
<!--使用命名空间将xml与接口绑在一起-->
<mapper namespace="com.zy.dao.StudentMapper">

    <select id="findAll" resultType="Student">
        select * from mybatis;
    </select>

    <select id="findStuById" parameterType="string" resultType="student">
        select * from mybatis where stuID=#{stuId}
    </select>

    <select id="findStu" parameterType="Map" resultType="student">
        select * from mybatis where stuID=#{stuId} and name=#{name}
    </select>

    <update id="updateStuById" parameterType="Map">
        update mybatis set name=#{name} where stuID=#{stuId}
    </update>

    <delete id="deleteStuById" parameterType="string">
        delete from mybatis where stuID=#{stuId}
    </delete>

    <insert id="insertStu" parameterType="student">
        insert into mybatis(name,gender,stuID) values(#{name},#{gender},#{stuId})
    </insert>
</mapper>
```

- 编写dao接口和dao的实现类，在到实现类中使用SqlSession操作crud。同时mapper文件的namesapce不用

  指定dao接口。在实现类中记得手动提交事务，mapper映射文件修改如下：

```xml
<mapper namespace="StudentMapper">  <!--只需要将namespace修改，其他不变-->
```

上述两种方式，都需要在配置文件中导入关系映射文件（mapper），配置文件变化如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--导入数据配置文件-->
    <properties resource="db.properties"/>

    <!--为实体类指定别名-->
    <typeAliases>
        <typeAlias type="com.zy.entity.Student" alias="student"/> <!--为具体的类起别名-->
        <package name="com.zy.entity"/>  <!--省略包的路径，在Mapper的返回类型中直接使用类名-->
    </typeAliases>
	
    <!--其他不变-->
    
    <mappers>
        <mapper resource="mapper/StudentMapper.xml"/>
    </mappers>
</configuration>
```

好了，两种方式已将介绍了，可能有点懵了，但没关系，接下，将展示全过程。首先是第一种方式：mapper映射

文件和接口绑定，采用上述第一种方式。然后创建单元测试类：

```java
public class StudentMapperTest {

    private SqlSession session;
    private StudentMapper stuMapper;
    @Before
    public void setUp(){
        session= MybatisConfiguration.getSession();     //mybatis 默认是开启事务的
        stuMapper=session.getMapper(StudentMapper.class); // //重点，重点，重点 
    }
    @Test
    public void testFindAll(){
        List<Student> students = stuMapper.findAll();
        Assert.assertNotNull(students);
        Assert.assertNotEquals(0,students.size());
        for (Student stu:students){
            System.out.println(stu);
        }
    }
    @Test
    public void testFindStuById(){
        Student stu = stuMapper.findStuById("20161113032");
        Assert.assertNotNull(stu);
        System.out.println(stu);
    }
    @Test
    public void testFindStu(){
        HashMap<String, Object> condition = new HashMap<>();
        condition.put("stuId","20191113033");
        condition.put("name","李");
        Student stu = stuMapper.findStu(condition);
        Assert.assertNotNull(stu);
        System.out.println(stu);
    }
    @Test
    public void testInsertStu(){
        Student student = new Student("20161110000","tony","男");
        stuMapper.insertStu(student);
        close();
    }
    @Test
    public void testUpdateStuById(){
        HashMap<String,String> condition = new HashMap<>();
        condition.put("stuId","20161110000");
        condition.put("name","Tom");
        stuMapper.updateStuById(condition);
        close();
    }
    @Test
    public void testDeleteStuById(){
        stuMapper.deleteStuById("20161110000");
        close();
    }
    //提交事务，关闭会话
    private void close(){
        session.commit();  //手动提交事务
        MybatisConfiguration.close();
    }
}
```

上述代码中，**执行session.getMapper(StudentMapper.class)语句时mybatis将会创建一个 StudentMapper**

**接口的代理类（接口实现类），该实现类中的方法，将会执行mapper映射文件中节点id与方法名对应sql语句，**

如 *select、update、insert* 等节点的语句。



第二种方式，在将中mapper映射文件的namespace属性修改后，接口StudentMappper不变，然后需要创建dao

的实现类，在实现了中完成数据库表数据的操作。（**感觉比较常用**）

```java
public class StudentMapperImpl implements StudentMapper {

    private SqlSession session;

    public StudentMapperImpl(){
        session= MybatisConfiguration.getSession();   //mybatis 默认是开启事务的
    }
    @Override
    public Student findStuById(String stuId) {
        return session.selectOne("StudentMapper.findStuById",stuId);
    }
    @Override
    public List<Student> findAll() {
        return session.selectList("StudentMapper.findAll");
    }
    @Override
    public void updateStuById(Map<String, String> conditions) {
        session.update("StudentMapper.updateStuById",conditions);
        close();
    }
    @Override
    public void deleteStuById(String stuId) {
        session.delete("StudentMapper.deleteStuId",stuId);
        close();
    
    @Override
    public Student findStu(Map<String,Object > conditions) {
        return session.selectOne("StudentMapper.findStu",conditions);
    }
    @Override
    public void insertStu(Student stu) {
        session.insert("StudentMapper.insertStu");
        close();
    }
    //提交事务，关闭会话
    private void close(){
        session.commit();  //手动提交事务
        MybatisConfiguration.close();
    }
}
```

单元测试类，就不写了。有兴趣的与实践实践一下。稍微讲一下重点知识：StudentDao的实现类没有直接操作数

据库表的数据，是使用数据库会话SqlSession操作的。在调用SQLSession相应的API时，例如：

*session.selectOne("StudentMapper.findStuById",stuId);* 第一个参数是**Mapper映射文件 namespace的值和 . 和**

***select* 节点的id组成的，第二参数就条件值了**。



讲到这里，数据库的crud都使用mybatis过了一篇。相信大家都入门了。总结一下关键点------mybatis的流程：

1. 在mybatis配置文件配置连接数据库的必要信息

2. 使用Reader读取配置文件，并通过SqlSessionFactoryBuilder对象创建SqlSessionFactory对象

3. 为每个线程维护自己的数据库连接会话（ThreadLocal），mybatis默认是开启事务的。

4. 创建实体与表的映射文件，指定命名空间，然后在配置文件中导入关系映射文件

5. 在dao的实现类中，获取线程的会话SqlSession，并使用相应的API操作（读取映射文件中的节点id，从而读取

   SQL语句）。

6. 增删改需要手动提交事务，然后关闭资源。

   

最后贴一下项目工程结构吧！

![mybatis工程结构](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/images/mybatis%E5%B7%A5%E7%A8%8B%E7%BB%93%E6%9E%84.png)



参考文章：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483937&idx=2&sn=28c7827639bb6ac0296746c4c4343c59&chksm=ebd74320dca0ca36b763b3975665fc38a7e921f9ecaef1aaea3a7c757063a29222cd00b3d3b6&scene=21###wechat_redirect">Mybatis【入门】</a>
