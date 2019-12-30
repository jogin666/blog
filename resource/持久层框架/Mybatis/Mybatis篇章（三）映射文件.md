## Mybatis篇章（三）映射文件

在前面两个篇章《<a href="https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/Mybatis%E7%AF%87%E7%AB%A0%EF%BC%88%E4%B8%80%EF%BC%89%E5%85%A5%E9%97%A8%E7%AF%87.md">Mybatis篇章（一）入门篇</a>》和《<a href="https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/Mybatis%E7%AF%87%E7%AB%A0%EF%BC%88%E4%BA%8C%EF%BC%89%E5%8A%A8%E6%80%81sql%E4%B8%8E%E6%B3%A8%E8%A7%A3.md">Mybatis篇章（二）动态sql语句</a>》，主要介绍mybatis的入门

使用操作（crud）和讲解了mybatis的动态sql语句和sql片段，接下来这一篇章将讲述MyBatis的实体与表之间关

联的映射文件。



**1、MappedStatement对象**

映射文件mapper.xml是以statement对为单位管理文件中的sql语句的，每一个statement都已映射文件的

namespace的值和节点id值确定的，每个sql语句被执行时，都会被封装成一个MappedStatement对象，然后在

Mybatis创建的代理类执行MappedStatement对象的方法。



**2、占位符和转义字符**

在Mybatis中，有两种占位符 \#{} 和 ${}

- \#{}解析传递进来的参数的数据值
- ${}解析传递进来的参数的数值后，然后拼接sql语句（相当于myql的concat函数），常结合like 使用。

```xml
<select id="findStudents" resultMap="studentMap" parameterType="string">
	select * from mybatis where stuID like '%${stuId}' <!--获取值后，然后拼接sql-->
</select>
```

- 转义字符。mybatis提供了一些常用的转义字符和转义字符的方法。

```sql
<![CDATA[ sql语句,例如：select * from user where  age >=18 ]] ">=" 是需要转义的字符
```



**3、别名  typeAliases**

有看过上一篇的读者，应该有注意到，在最后的mybatis配置文件中，多了几个节点。其中一个如下：

```xml
<!--为实体类指定别名-->
<typeAliases>
    <typeAlias type="com.zy.entity.Student" alias="student"/> <!--为具体的类起别名-->
    <package name="com.zy.entity"/>  <!--省略包的路径，在Mapper的返回类型中直接使用类名-->
</typeAliases>
```

上述节点的作用就是：为实体类别名，然后在mapper.xml文件中直接使用别名，省却指定具体返回类型的类路

径。如果没有起别名：要指定返回的类型需要这样写：*resultType="com.zy.entity.Student"*，每个要求返回一个

Student类型的节点，都这样写，那不是很烦？而有了别名就可以这样子写了：*resultType=student*，十分简洁，方

便。如果多个实体类都在包下，那直接在 *typeAlias* 节点下使用 *<package name="com.zy.entity"/* ，

指定省略返回类型的类路径，直接在指定返回的类型中直接使用类名，例如：*ResultType=Student*。值得注意的

是，package 节点必须在 typeAliases 节点之下。常用类型的别名如下图：

![这里写图片描述](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib3W3VSvKKR7ialreweHt7HP0iak4wrHJByib09y9L7ibVTYmVFvpt5iaDjZppQiaAuXUBJaFYg8zcyuZgvw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图片来源于参考资料。



**4、mapper 导入mapper.xml**

<mappers>是mybatis配置文件用来导入映入文件的节点，直接使用resource指定映射文件的全路径名。

```xml
<mappers>
    <mapper resource="mapper/StudentMapper.xml"/>
     <!--<mapper resource="com.zy.dao.StudentMapper"/>-->
</mappers>
```

被注释的语句是条件的，只有在：①xxMapper.xml和 xxMapper.java 两个文件都在同一目录下才行，并且尽量

满足两个文件的文件名是一样的。



**5、ResultMap 和 ResultType 返回属性**

ResultMap和ResultType都是指定执行sql语句后，返回的结果类型。在上一篇章，只使用到ResultType属性，没

有使用到ResultMap的属性，那先来了解ResultMap吧。使用上一章节的项目工程。在StudentMapper.xml文件

为尾部添加如下：

```xml
<mapper namespace="StudentMapper1">
    <resultMap id="studentMap" type="student">
        <result column="name" property="name"/>
        <result column="gender" property="gender"/>
        <result column="stuID" property="stuId"/>
    </resultMap>
</mapper>
```

如上述文件中，创建一个ResultMap，在ResultMap中指定返回的结果类型的属性，其中id是唯一值，type指定返

回的类型，column指定的是表的字段，property指定返回类型的属性。也就是column和property将表字段的数

据和返回类型的属性关联起来。你可能会迷惑？ResultType不就可以完成了吗？为什么还有需要ResultMap呢？

在开发中如果表的字段和实体的属性是对应的，那么就不会使用到ResultMap了。但是在实际开发中，表的字段不

一和实体类的成员名是一致的，所以就需要使用ResultMap将表的数据和实体关联起来。而且ResultMap的实际用

处不限于此，ResultMap还可以继承，以及和 association或者collection结合使用，实现多表之间的连接查询。

- ResultType适用于：实体类成员名和表字段名一致。

- ReultMap适用于：实体类成员名和表字段名不一致，或多表连接查询。

  

**6、association**  连接查询（一对一）

*association* 结合 *ResultMap* 将关联查询的表的数据映射到实体类的一个引用类型上。首先创建一张表（学生卡表

```sql
create table card(
  id     int auto_increment primary key,
  number varchar(11) not null,
  money  float       not null,
  year   int         not null
);
```

然后创建Card表的实体类Card

```java
public class Card {
    private String number;
    private int year;
    private float money;
    //getter and setter
}
```

在Student类中增加Card的成员: *private Card card;* 和对应的 getter、setter 方法。在StudentMapper.xml文件添

加一个 *select* 节点查询和新添加一个继承 *studentMap* 的 *ResultMap* 节点，新增内容如下图：

```xml
<select id="findStuAndCard" resultMap="stuCardMap" parameterType="string">
    select * from mybatis m,card c where m.stuID=c.number and m.stuID=#{stuId}
</select>

<resultMap id="studentMap" type="student">
    <result column="name" property="name"/>
    <result column="gender" property="gender"/>
    <result column="stuID" property="stuId"/>
</resultMap>

<resultMap id="stuCardMap" type="student" extends="studentMap">
    <association property="card" javaType="Card">
        <result property="money" column="money"/>
        <result property="number" column="number"/>
        <result property="year" column="year"/>
    </association>
</resultMap>
```

*asociation* 节点中javaType指定查询结果返回的类型。

然后在StudentMapper接口添加一个查询方法和StudentMapperImpl实现添加的方法，在单元测试中，测试一下

```java
@Override
public Student findStuAndCard(String stuId) {
    return session.selectOne("StudentMapper.findStuAndCard",stuId);
}
/*********************分割线 单元测试*********************/
@Test
public void testFindStuAndCard(){
    Student student = stuMapImpl.findStuAndCard("20161113033");
    Assert.assertNotNull(student);
    System.out.println(student); 
}    
/*测试结果：
Student{stuId='20161113033', name='李', gender='男', card=[number='20161113033', year=4, money=0.0]} */
```



**7、collection** 连接查询（一对多 或者 多对多）

*collection* 结合 *ResultMap*  将关联查询的表的数据映射到实体类的维护的一个List集合中。考虑到学生和课程是多

对多的关系，因此建立一张选修课程表 scourse（简单几个字段），一个stuId在scourse表有多个记录（一对多关系，虽然

举的例子不恰当，但明白意思就好）。

```sql
create table scourse(
  id           int auto_increment primary key,
  courseId     varchar(32) not null,
  course_name  varchar(26) not null,
  course_type varchar(6)  not null,
  stuId		  varchar(11)  not null
);
```

创建SCourse实体类

```java
public class SCourse {

    private String courseId;
    private String courseName;
    private String courseType;
    private String stuId;
    //getter and setter
}
```

在Student实体类添加 *private List<SCourse> courses;* 并给出相应的 *getter 和 setter* 方法StudentMapper.xml文

件添加一个 *select* 查询节点和新添加一个继承 *studentMap* 的 *ResultMap *节点，新增内容如下图：

```xml
<select id="findCourseByStuId" resultMap="stuCourseMap" parameterType="string">
    select * from mybatis m,scourse sc where m.stuID=#{stuId} and m.stuID=sc.stuId;
</select>

<resultMap id="stuCourseMap" type="student" extends="studentMap">
    <collection property="courses" ofType="SCourse">
        <result property="stuId" column="stuId"/>
        <result property="courseName" column="course_name"/>
        <result property="courseType" column="course_type"/>
        <result property="courseId" column="courseId"/>
    </collection>
</resultMap>
```

*collection* 节点中的ofType指定查询结果的每条记录封装成的类型。

在StudentMapper添加一个查询方法，并在StudentMapperImpl中实现该方法，在单元测试中测试。

```java
@Override
public Student findCourseByStuId(String stuId) {
    return session.selectOne("StudentMapper.findCourseByStuId",stuId);
}
/*********************分割线 单元测试*********************/
@Test
public void testFindCourseByStuId(){
    Student student = stuMapImpl.findCourseByStuId("20161113033");
    Assert.assertNotNull(student);
    List<SCourse> courses = student.getCourses();
    Assert.assertNotEquals(0,courses.size());
    for(SCourse course:courses){
        System.out.println(course);
    }
}
/*测试结果
SCourse{courseId='000001', courseName='高数上册', courseType='数学', stuId='20161113033'}
SCourse{courseId='000002', courseName='线代', courseType='数学', stuId='20161113033'}
*/
```



**8、延迟加载**

在数据查询的时候，为了提高数据库的查询速率，都应该使用单表查询，因为单表的查询速度快。但不是所有的业

务都可以由单表查询完成。就如上面的第6点，根据stuId查询学生时，也要获取到该学生的正在选修的课程，这就

需要多表关联查询了，但有时候未必需要到学生的选课信息，如果一查询学生，也要把学生的选课信息查询出来，

岂不是很浪费性能。延迟加载就是为了解决这个问题，有需要的时候才去加载关联表的数据，不需要的时候，就不

查询。mybatis的延迟加载在mybatis配置文件中手动启用，并在 *reslultMap* 中的 *asociation* 和 *collection* 节点中

设置。 在Mybatis配置文件启用懒加载，需要如下声明：

```xml
<!--启用懒加载设置-->
<settings>
    <setting name="lazyLoadingEnabled" value="ture"/>  <!--启用延迟加载，默认false-->
     <!--设置按需加载（需要时加载，按层级加载），默认true-->
    <setting name="aggressiveLazyLoading" value="false"/> 
</settings>
```

| 设置项                | 描述                                                         | 默认值 |
| --------------------- | ------------------------------------------------------------ | ------ |
| lazyLoadingEnabled    | 全局性设置是否懒加载，设置为false时，默认立即加载关联查询查询的数据。 | false  |
| aggressiveLazyLoading | 全局性设置是否按需加载，设置为true时，把把当前关联的数据全部加载（例如A表关联B表，B表关联C表，查询A，也会把BC的数据捞出来），设置false时，则按需（层次）加载，需要到时才会加载数据。 | true   |

***fetchType***：由于上述的设置都是针对全局的，不够灵活。如：有ABC三张表，A关联B表和C表（B,C处于同一层

级），现在要求是：只要加载B表数据，而不要加载C表数据。显然按需加载是不能解决这个需求的，因此*mybatis*

提供 *fectchType* 抓取指定的关联表的数据。*fetchType* 只能在 *ResultMap* 的 *association* 和 *collection* 中进行设置指

定抓取的表（其实也就是*association* 和 *collection*  节点的属性而已）。*fetchType* 有两种设置：

- eager: 立即抓取指定关联表的数据，封装成指定的pojo类型。
- lazy：懒加载，需要的时候才会加载。

值得注意的是：fetchType属性的优先级是高于全局属性的，会覆盖全局的设置（mybatis配置文件中的设置）。

> 想深入学习的话，传送门：<a href="https://www.jianshu.com/p/6f5b42d52d38">延迟加载(lazyLoadingEnabled、aggressiveLazyLoading、fetchType)</a>

延迟加载实践演示：（新建一个 *CardMapper.xml* 文件）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="CardMapper">
    
    <select id="findCardById" resultMap="cardMap">
        select * from card where number=#{number}
    </select>
    <resultMap id="cardMap" type="com.zy.entity.Card">
        <result column="number" property="number"/>
        <result column="money" property="money"/>
        <result column="year" property="year"/>
    </resultMap>
</mapper>
```

然后在 *StudentMapper.xml* 文件中添加如下如下内容：

```xml
<select id="findStuAndLazyLoadCard" resultMap="lazyLoadCard" parameterType="string">
    select * from mybatis where stuID=#{stuId}
</select>

<resultMap id="lazyLoadCard" type="student" extends="studentMap">
    <association property="card" select="CardMapper.findCardById" column="stuID"/>
</resultMap>
```

值得说一下：*select* 指定了使用哪个statement 来执行延迟加载，statement是由 全路径名+namespace的值+

*select* 节点的id值确定的（同一文件下直接使用节点的id，同包直接使用 namespace + id）。 column：指

定了哪个字段的值，作为statement条件的参数。因为card表中的*number* 和student表中的 *stuID* 是对应的，所以 

采用stuID。在 *StudentMapper* 接口添加一个方法并在实现类中实现，然后在单元测试：

```java
@Override
public Student findStuAndLazyLoadCard(String stuId) {
    return session.selectOne("StudentMapper.findStuAndLazyLoadCard",stuId);
}
@Test
public void testFindStuAndLazyLoadCard(){
    Student student = stuMapImpl.findStuAndLazyLoadCard("20161113033");
    Assert.assertNotNull(student);
    System.out.println(student.getCard());
}
/*测试结果：[number='20161113033', year=4, money=0.0]
*/
```



**9、主键策略**

Mybatis也提供了一些常用的主键生成策略，下面介绍两种常用的：

- 自增策略：<insert 节点中使用  *useGeneratedKeys="true"* 表明使用自增策略，*keyProperty* 指定字段。

```xml
<insert id="addStudent" parameterType="Student" useGeneratedKeys="true" keyProperty="id">
    insert into mybatis(name,gender,stuID) values(#{name},#{gender},#{stuId})
</insert>　
```

- UUID生成策略

```xml
<insert id="inster" parameterType="***">
    <!--keyProperty 指定字段，order 指定先后顺序，resultType指定类型-->
  	<selectKey keyProperty="id" order="BEFORE" resultType="string">
        select uuid()
    </selectKey>
    <!--insert 语句-->
</insert>　
```

- 获取生成的主键

```xml
<insert id="inster" parameterType="***">	
    <selectKey keyProperty="id" order="AFTER" resultType="int">
        select LAST_INSERT_ID()
    </selectKey>
     <!--insert 语句-->
</insert>　
```

> 通过LAST_INSERT_ID()获取刚插入记录的自增主键值，在insert语句执行后，执行select LAST_INSERT_ID()就可以获取自增主键。

最后贴一下：项目工程结构：

![mybatis工程结构](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/images/mybatis%E5%B7%A5%E7%A8%8B%E7%BB%93%E6%9E%842.png)

参考资料：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483937&idx=3&sn=977f8e1eeb0d4e46bab6d6140e856c83&chksm=ebd74320dca0ca3648e351f2f3d5196842e64d2e8ba14722ec2548da46df7e88832765e67f87&scene=21###wechat_redirect">Mybatis【配置文件】</a>

<a href="https://www.jianshu.com/p/6f5b42d52d38">延迟加载(lazyLoadingEnabled、aggressiveLazyLoading、fetchType)</a>
