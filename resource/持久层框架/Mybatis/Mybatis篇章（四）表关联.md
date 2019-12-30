## Mybatis篇章（四）表关联

这是MyBatis篇章的第四篇了，本来这篇想讲的是Mybatis的缓存的。但是想了想还是讲一下Mybatis的多关联

和表之间的映射吧！回顾上一篇章《<a href="https://github.com/jogin666/Database-Persistence-Framework/blob/master/mybatis/Mybatis%E7%AF%87%E7%AB%A0%EF%BC%88%E4%B8%89%EF%BC%89%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6.md">Mybatis篇章（三）映射文件</a>》，在讲 *resultMap* 的时候，其实已经将

Mybatis的多表关联给讲了，现在重新再理一遍吧。

**1、回顾表结构**（虽然表建的不是很好......）

* mybatis表（相当于学生表.....		）

```sql
create table mybatis(
  Id     int auto_increment primary key,
  name   varchar(20) charset utf8 null,
  gender char(2) charset utf8     null,
  stuID  varchar(11)              null
);
```

* card表

```sql
create table card(
  id     int auto_increment primary key,
  number varchar(11) not null,
  money  float       not null,
  year   int         not null
);
```

* scourse表

```sql
create table scourse(
  id          int auto_increment primary key,
  courseId    varchar(32) not null,
  course_name varchar(26) not null,
  course_type varchar(6)  not null,
  stuId       varchar(11) not null
);
```

上述表之间的关系是：mybatis和card是一对一的关系（一个学生一张学生卡）；mybatis和scourse是多对多的

关系（一个学生可以选择多门课程，一个课程有多个学生上）。可以发现少了一个一对多的关系，而此篇章要讲表

关联 ，因此补充一对多的关系。考虑到一个班级有多个学生，一个学生只能属于一个班级，因此创建一个学生班

级表（class) ，同时在学生表中追加一个字段(classId)。

```sql
create table class(
  id 		int auto_increment primary key,
  classId	varchar(6) not null,
  class_name	varchar(26) not null,  
)
```

**2、回顾创建的实体类**

好了，几个关系都有了，回顾之前创建表的实体，考虑到表之间的联系，因此，实体类可能需要有些变动。

- Student实体类(多了一个属性classId，上面追加的)

```java
public class Student {

    private String stuId;
    private String name;
    private String classId; //新增的
    private String gender;
    private Card card;	//维护 一对一的关系
    private List<SCourse> courses;  //维护 多对多的关系
	//getter and setter
}
```

- Card实体类

```java
public class Card {

    private int id;
    private String number;
    private int year;
    private float money;
	//getter and setter
}
```

- SCourse 选课类

```java
public class SCourse {

    private String courseId;
    private String courseName;
    private String courseType;
    private String stuId;
    private List<Student> students; //维护 多对多的关系
	//getter and setter
}
```

- 创建SC表的实体类 SCEntity，Class表的实体类 Class

```java
public class SCEntity {
  
    private String stuId;
    private String classId;
    //getter and setter
}
/**************************分割线*************************************/
public class Class{
    private String classId;
    private String className;
    private String className;
    private List<Student> students; //维护 一对多的关系
    //getter and setter
}
```

**3、表之间的关联映射**

在关系映射Mapper.xml文件配置表之间的关联映射。Mybatis是如何维护**一对一**的关系，如果看过上一篇章，你

立马能想到，使用 *association*，没错！mybatis就是通过 *association* 维护的。 回顾一下*StudentMapper.xml*文件

```xaml
<select id="findStuAndCard" resultMap="stuCardMap" parameterType="string">
    select * from mybatis m,card c where m.stuID=c.number and m.stuID=#{stuId}
</select>

<resultMap id="stuCardMap" type="student" extends="studentMap">
    <association property="card" javaType="Card">
        <result property="money" column="money"/>
        <result property="number" column="number"/>
        <result property="year" column="year"/>
    </association>
</resultMap>
```

在回顾一下代码：

```java
@Override
public Student findStuAndCard(String stuId) {
    return session.selectOne("StudentMapper.findStuAndCard",stuId);
}
/**************************分割线 单元测试**********************************************/
@Test
public void testFindStuAndCard(){
    Student student = stuMapImpl.findStuAndCard("20161113033");
    Assert.assertNotNull(student);
    System.out.println(student.getCard());
}
//测试结果： [number='20161113033', year=4, money=0.0]
```

而**多对多**的关系，相信你能猜到了吧，使用 *collection*，没错！Mybatis就是使用 *collection* 维护多对多的干系的。 

在回顾上一篇章中的*StudentMapper.xml*文件的配置。

```xml
<select id="findCourseByStuId" resultMap="stuCourseMap" parameterType="string">
    select * from mybatis m,scourse sc where m.stuID=#{stuId} and m.stuID=sc.stuId;
</select>

<resultMap id="stuCourseMap" type="student" extends="studentMap">
    <collection property="courses" ofType="SCourse" fetchType="lazy">
        <result property="stuId" column="stuId"/>
        <result property="courseName" column="course_name"/>
        <result property="courseType" column="course_type"/>
        <result property="courseId" column="courseId"/>
    </collection>
</resultMap>
```

回顾一下代码：

```java
@Override
public Student findCourseByStuId(String stuId) {
    return session.selectOne("StudentMapper.findCourseByStuId",stuId);
}
/**************************分割线 单元测试**********************************************/
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
/*测试结果：
SCourse{courseId='000001', courseName='高数上册', courseType='数学', stuId='20161113033'}
SCourse{courseId='000002', courseName='线代', courseType='数学', stuId='20161113033'}
SCourse{courseId='000003', courseName='概率论', courseType='数学', stuId='20161113033'}
*/
```

至于一对多，就不说了，因为多对多，就是由一对多变过来的（虽然学生与课程是多对多的，但是本质上两个一对

多的关系，一个学生可以上多门课程，一门课程可以有多个学生）。懂了多对多关系，那便懂了一对多了。顺便说

一下，某些情况下，其实多对多，或者一对多并不一定需要到 *asociation* 和 *collection* 的，只要 *resultMap* 能将查

询的数据封装成返回类型就行了（ *association*  和 *collection* 就是实现数据封装指定类型）。



**4、使用注解实现关联关系**

 一下使用第二篇章中的EntityMapper接口实现。

- 一对一，修改一下 *findStuById* 上的注解，如下。*@Results*  相当于关系映射文件中的 *resultMap*，*@result* 将

  实体类的成员和表的字段关联起来，但又有额外的功能：*@Result*  可以实现一对一，一对多的关联关系，在实

  现关联关系时，*column* 指定是传递哪个字段的值，最作为参数，one指定一对第一，而 *@One*  的 *select* 指定

  查询的方法（不同包下，需要 全限定类名 . 方法 的形式，注意有个 点）。

```java
@Select("select * from mybatis where stuID=#{stuId}")
@Results(value = {
    @Result(column = "name",property = "name"),
    @Result(column = "gender",property = "gender"),
    @Result(column = "stuID",property = "stuId"),
    @Result(column = "stuID",property = "card",one = @One(
        select = "findCardById",fetchType = FetchType.EAGER))
})
Student findStuById(String stuId);

@Select("select * from card where number=#{number}")
Card findCardById(String number);
```

- 一对多，同上，只需要将 *one* 属性改为 *many=@Many(.....)*

```java
@Select("select * from mybatis where stuID=#{stuId}")
@Results(value = {
    @Result(column = "name",property = "name"),
    @Result(column = "gender",property = "gender"),
    @Result(column = "stuID",property = "stuId"),
    @Result(column = "stuID",property = "card",one = @One(
        select = "findCardById",fetchType = FetchType.LAZY)),
    @Result(column = "stuID",property ="courses",many=@Many(
        select = "findStuCourseById",fetchType =FetchType.EAGER))
})
Student findStuById(String stuId);

@Select("select * from scourse where stuId=#{stuId}")
List<SCourse> findStuCourseById(String stuId);
```

单元测试：

```java
public class EntityMapperTest {

    private SqlSession session;
    private EntityMapper mapper;

    @Before
    public void setUp() throws Exception {
        session= MybatisConfiguration.getSession();
        mapper=session.getMapper(EntityMapper.class);
    }

    @Test
    public void findStuById(){
        Student student = mapper.findStuById("20161113033");
        Assert.assertNotNull(student);
        System.out.println(student);
        List<SCourse> courses = student.getCourses();
        for (SCourse course:courses){
            System.out.println(course);
        }
    }
}
/*测试结果：
Student{stuId='20161113033', name='黄', gender='男', card=[number='20161113033', year=4, money=0.0]}
SCourse{courseId='000001', courseName='null', courseType='null', stuId='20161113033'}
SCourse{courseId='000002', courseName='null', courseType='null', stuId='20161113033'}
SCourse{courseId='000003', courseName='null', courseType='null', stuId='20161113033'}
*/
```

最后，个人感觉还是配置文件比较实在些。



参考资料：



