## Hibernate篇章（三）多表关联篇

上两个篇章《<a href="">Hibernate篇章（一）入门篇</a>》和《<a href="https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Hibernate/Hibernate%E7%AF%87%E7%AB%A0%EF%BC%88%E4%BA%8C%EF%BC%89%E9%85%8D%E7%BD%AE%E3%80%81%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6%E7%AF%87.md">Hibernate篇章（二）配置、映射文件介绍</a>》分别讲述hibernate

的快速入门和hibernate的配置文件，映射文件，所以这个篇章将要讲述的是hibernate的是如何使用关联映射文

件实现多表关联的。表关联的方式有：①：一多一 	②：一对多	③：多对多。

### 一、实现一对一关系

现在要讲解hibernate的一对一的关系，在第一篇章，已将创建一张学生表。因此需要再创建一张表了，考虑到学

生与学生卡是一对一的关系，因此就创建一张学生表吧。基于hibernate的一对一关系，创建学生表会有两种方式

创建：

①：在学生卡（card）表使用外键约束来维护其和学生表（student）的一对一关系

②：在学生卡（card）表中使用 主键+外键 的方式来维护其和学生表（student）的一对一关系。



**1. 使用第一种方式，外键约束**

先使用第一种方式来创建一对一的关系。但在这之前要考虑好，应该使用哪一方作为维护关系的一方？由于使用的

是外键约束，即是不能使用学生（Student）作为维护方的，为啥呢？在此篇章的入门篇，就讲述了hibernate默

认实现的crud操作是使用主键实现的，student表是没有card表的主键（更善查改操作，使用 student.id=card.id 

进行外连接的操作的），所以没有办法实现维护的，无法对card进行关联，而使用Card来维护的话，card表有

student表的主键（外键）。

创建学生卡表：（使用外键）

```sql
create table card(
  id     int auto_increment primary key,
  sid    int         not null,
  number varchar(11) not null,
  constraint card_student_fk
    foreign key (sid) references student (sid)
);
create index card_student_fk_idx on card (sid);
```

创建学生卡实体类：

```java
public class Card implements Serializable{

    private int id;
    private String number;
   	private Student stu;  //维护一对一的关系
	//setter and getter
}
```

创建关系映射文件：

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.zy.hibernate.entity">
    <class name="Card" table="card">
        
        <id name="id" column="id" length="10">
            <generator class="native"/>
        </id>
        <property name="number" column="number" length="11" type="java.lang.String"/>
       
        <!--维护一对一的关系
			name: Student的引用
			sid：指定存贮stu主键的字段，也就是外键列
			unique：sid字段唯一值 否则抛出异常
			cascade： 联级增加，更新
  		-->
        <many-to-one name="stu" column="sid" class="Student" unique="true" cascade="save-update"/>
    </class>
</hibernate-mapping>
```

为什么不是一对一呢？ one-to-one是基于主键映射的，不符合。同时 one-to-one 执行sql的顺序是先执行card表

的sql，然后执行student的sql。（例如：在增加student和card时，就会造成card的外键列为空。因为hibernate 

默认是插入全部列，而sid外键列是依赖student表的主键的），one-to-one  对于 以上两个要求是不能满足的。

回顾学生表：

```sql
create table student(
  sid     int auto_increment primary key,
  userId varchar(255) null,
  name   varchar(255) null,
  gender varchar(255) null,
  age    int          null
);
```

修改学生实体：（不需要创建Card的引用，因为维护方是Card）

```java
public class Student implements Serializable{
    private int sId;
    private String userId;
    private String name;
    private String gender;
    private int age;
	//getter and setter
}
```

修改关系映射表：

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.zy.hibernate.entity">
    <!--将实体类与表建立管理映射-->
    <class name="Student" table="student">
        <!--主键，采用自增策略-->
        <id name="sId" column="id">
            <generator class="native"/>
        </id>
        <!--实体对象的属性对应的字段-->
        <property name="userId" column="userId" type="java.lang.String"/>
        <property name="gender" column="gender" type="java.lang.String"/>
        <property name="name" column="name" type="java.lang.String"/>
        <property name="age" column="age" type="java.lang.Integer" length="6"/>
    </class>
</hibernate-mapping>
```

编写单元测试：

```java
public class EntityTest {

    private Session session;

    @BeforeEach
    void setUp(){
         session = HibernateConfig.openTransaction();
    }

    @Test
    void save(){
        //创建学生
        Student stu = new Student();
        stu.setGender("male");
        stu.setAge(26);
        stu.setUserId("123456");
        stu.setName("tt");
        //创建学生卡
        Card card=new Card();
        card.setNumber("123456");
        card.setStu(stu);
        //保存学生
        session.save(card);
    }

    @Test
    void find(){
        Card card = session.find(Card.class, 4);
        System.out.println(card);
    }

    @AfterEach
    void close(){
        HibernateConfig.commitAndClose();
    }
}
/*控制台输出：
Hibernate: 
    select
        card0_.id as id1_0_0_,
        card0_.number as number2_0_0_,
        card0_.sid as sid3_0_0_ 
    from
        Card card0_ 
    where
        card0_.id=?
Hibernate: 
    select
        student0_.sid as sid1_1_0_,
        student0_.userId as userId2_1_0_,
        student0_.gender as gender3_1_0_,
        student0_.name as name4_1_0_,
        student0_.age as age5_1_0_ 
    from
        student student0_ 
    where
        student0_.sid=?
Card{id=4, number='123456', stu=Student{sId=9, userId='123456', name='tt', gender='male', age=26}}
*/
```

**2. 采用第二种方式，既是外键又是主键**

使用Card2进行维护一对一的关系，原因如上述。

```sql
create table card2(
  id     int auto_increment primary key,
  sid    int         not null,
  number varchar(11) not null,
  constraint card_student_fk
    foreign key (sid) references student (id)
);

create index card_student_fk_idx on card (sid);
```

创建Card2实体类：

```java
public class Card2 {
    private int id;
    private String number;
    private Student stu;
	//getter and setter
}
```

创建映射文件：

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.zy.hibernate.entity">

    <class name="Card2" table="card2">
        <id name="id" column="id">
            <!--使用外键策略，依赖于student表的主键-->
            <generator class="foreign">
                <param name="property">stu</param>
            </generator>
        </id>
        <property name="number" column="number" type="java.lang.String" length="11"/>

        <!--one-to-one: 是基于主键映射的
            constrained="true"： 指定在主键有外键约束,会先执行关联表的sql，然后执行本表的sql
         -->
        <one-to-one name="stu" class="Student" cascade="save-update" constrained="true"/>
    </class>
</hibernate-mapping>
```

单元测试就不展示了。



### 二、多对一的关系

因为是一对多的关系，就创建一张学生表吧（一个学生在一个班级，一个班级有多个学生）。

```sql
create table class(
  id        int auto_increment	primary key,
  className varchar(20) charset utf8 not null,
);
```

修改一下学生表：（学生表要求有班级表的主键，关联嘛）

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



在一对多的关系中，哪一方维护关系都可以，但一般是使用多的一放维护关系，减少hibernatede更新操作。

* 一对多

创建班级实体类

```java
public class Class implements Serializable {

    private int id;
    private String className;
    // private List<Student> studentList;
    private Set<Student> studentSet; 
    // getter and setter
}    
```

创建映射文件：

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.zy.hibernate.entity">
    <class name="Class" table="class">
        <id name="id" length="11" column="id">
            <generator class="native"/>
        </id>
        <property name="className" column="class_name" type="java.lang.String" length="20"/>
        <!--
            因为使用的是Set存储，所以对应的要是用 set 节点，set节点
            name： 实体类对应属性名字
            table： 指定关联的表
			
			<set>的子节点
            key： 指定关联表中的外键
            one-to-many： 一对多的关系；其class属于是指将表数据要封装成的类型
        -->
        <set name="studentSet" table="student" cascade="save-update">
            <key column="classId"></key>
            <one-to-many class="Student"/>
        </set>
        <!--
		    <list> 是有序集合，根据<index>或者<list-index>指定字段排序		
			<list name="studentList" table="student" cascade="save-update">          
            	<key column="classId"></key>          
            	<index column="sid"/>         
            	<one-to-many class="Student"/>
        	</list>
		-->
    </class>
</hibernate-mapping>
```

hibernate还提供了map存储数据，等需要到时，再回来填坑。

创建学生实体类：（维护方是 Class），关联映射文件内容不变。

```java
public class Student implements Serializable{

    private int sId;
    private String userId;
    private String name;
    private String gender;
    private int age;
    // getter and setter
}    
```

在单元测试类中增加测试方法

```java
@Test
void find(){
    System.out.println(session.find(Class.class,3));
}

@Test
void saveClass(){
    Class classEntity= new Class();
    classEntity.setClassName("终极一班");
    Student stu1 = new Student();
    stu1.setName("小明");
    stu1.setAge(12);
    Student stu2 = new Student();
    stu2.setAge(18);
    stu2.setName("小李");
    classEntity.getStudentSet().add(stu1);
    classEntity.getStudentSet().add(stu2);
    session.save(classEntity);
}
/*控制台输出：
Hibernate: 
    select
        class0_.id as id1_2_0_,
        class0_.class_name as class_na2_2_0_ 
    from
        class class0_ 
    where
        class0_.id=?
Hibernate: 
    select
        studentset0_.classId as classId6_3_0_,
        studentset0_.sid as sid1_3_0_,
        studentset0_.sid as sid1_3_1_,
        studentset0_.userId as userId2_3_1_,
        studentset0_.gender as gender3_3_1_,
        studentset0_.name as name4_3_1_,
        studentset0_.age as age5_3_1_ 
    from
        student studentset0_ 
    where
        studentset0_.classId=?
Class{id=3, className='终极一班', studentSet=[Student{sId=15, userId='null', name='小明', gender='null', age=12, classId=0}, Student{sId=16, userId='null', name='小李', gender='null', age=18, classId=0}]}
*/
```

在这里说一下：一对多关系中的一个 inverse属性吧，该属性有只能选择 true|false 默认为false，也只能是在一

对多中的 一 的那一方使用。Inverse属性：表示控制权是否转移。为 false，表示控制权不转移，在增删改查时，

维护两个表之间的关系。如果为true，则表示控制权交给多的那一方使用，让其在crud时，维护两个表之间的关

系，当交出控制权后，其不能对另一张的数据进行删除，或者解除数据之间的联系（外键）。但仍可以保存数据，

但不会进行数据关联（外键列为空），也可以查询。 inverse的优先级高于cascade的。

详细阅读，请传送门走一波：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=5&sn=3cd626ef18d0c1ac5d81dac393b745b8&chksm=ebd74303dca0ca15e29693486169255e83dd3a1bc1b5d373916560522095cd41a129b7051aaf&scene=21###wechat_redirect">Hibernate【inverse和cascade属性】知识要点</a>



* **多对一** 

  在Student实体类中 增加 *private Class classEntity;*  并给出相应的 getter and setter 方法，然后在关系映射文

  件中 *<class>* 节点下增加：*<many-to-one name="classEntity" column="classId"*

  *class="Class" cascade="save-update"/>*  ，这句代码在一对一的时，已经解释过了。单元测试也不展示了，其

  结果和一对一关系类似。
  
  

值得说一下的是：

- 配置一对多与多对一，      这种叫“双向关联”
- 只配置一对多，           叫“单项一对多”
- 只配置多对一，           叫“单项多对一”



### 三、多对多的关系

创建一张课程表，因为学生与课程时是多对多的关系，一个学生可以生多个课程，一个课程可以有多个学生来上

课。在多对多的关系中，一般创建一个表为维护多对多的关系，但在Java实体类中，一般是不会为第三张表创建实

体类的，只会为数据表创建实体类。

创建课程表：

```sql
create table course(
  courseId    varchar(32) not null primary key,
  course_name varchar(30) null,
  course_type varchar(30) null,
  stuId       varchar(11) not null,
  coureseId   varchar(32) not null
);
```

创建sc表，学生选课表：

```sql
create table sc(
  courseId int null,
  sId      int not null,
  constraint sc_coueses_id_fk foreign key (courseId) references coueses (id),
  constraint sc_student_sid_fk foreign key (sId) references student (sid)
);
```

使用Course作为维护方

* 创建Course实体类：

```java
public class Course {
    
    private int id;
    private String courseName;
    private Set<Student> studentSet;
	//getter and setter
}
```

* 创建关系映射文件：

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.zy.hibernate.entity">
    <class table="courses" name="Course">
        <id name="id" column="id">
            <generator class="native"/>
        </id>
        <property name="courseName" column="course_name" length="20" type="java.lang.String"/>
        
        <!-- set 节点中的 table： 指定sc 表 维护多对多的关联表-->
        <set name="studentSet" table="sc">
            <!--指定sc表维护courses表主键的外键字段 -->
            <key column="courseId"></key>
            <!--指定维护student表主键的外键字段，关联student表，class是指将数据封装的类型-->
            <many-to-many column="sId" class="Student"/>
        </set>
    </class>
</hibernate-mapping>
```

使用Student作为维护方：

* 修改学生实体类：

```java
public class Student {

    private int sId;
    private String userId;
    private String name;
    private String gender;
    private int age;
    private Class classEntity;
    private Set<Course> courseSet;
    //getter and setter
}
```

* 关系映射文件：

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.zy.hibernate.entity">
    <!--将实体类与表建立管理映射-->
    <class name="Student" table="student">
        <!--主键，采用自增策略-->
        <id name="sId" column="sid">
            <generator class="native"/>
        </id>
        <!--实体对象的属性对应的字段-->
        <property name="userId" column="userId" type="java.lang.String"/>
        <property name="gender" column="gender" type="java.lang.String"/>
        <property name="name" column="name" type="java.lang.String"/>

        <property name="age" column="age" type="java.lang.Integer" length="6"/>
        
         <!-- set 节点中的 table： 指定sc 表 维护多对多的关联表-->
        <set name="courseSet" table="sc">
             <!--指定sc表维护student表主键的外键字段 -->
            <key column="sId"></key>
             <!--指定维护course表主键的外键字段，关联courses表，class是指将数据封装的类型-->
            <many-to-many column="courseId" class="Course"/>
        </set>

        <many-to-one name="classEntity" column="classId" class="Class" unique="true" cascade="save-update"/>
    </class>

</hibernate-mapping>
```

单元测试：（在sc表盒courses表中添加几条数据，然后单元测试查询，以Course作为维护方）

```java
@Test
void saveCourse(){
    Course course = session.find(Course.class, 1);
    System.out.println(course);
}
/*控制台输出：
Hibernate: 
    select
        course0_.id as id1_3_0_,
        course0_.course_name as course_n2_3_0_ 
    from
        courses course0_ 
    where
        course0_.id=?
Hibernate: 
    select
        studentset0_.courseId as courseId2_4_0_,
        studentset0_.sId as sId1_4_0_,
        student1_.sid as sid1_5_1_,
        student1_.userId as userId2_5_1_,
        student1_.gender as gender3_5_1_,
        student1_.name as name4_5_1_,
        student1_.age as age5_5_1_,
        student1_.classId as classId6_5_1_ 
    from
        sc studentset0_ 
    inner join
        student student1_ 
            on studentset0_.sId=student1_.sid 
    where
        studentset0_.courseId=?
Course{id=1, courseName='数学', studentSet=[com.zy.hibernate.entity.Student@6342d610, com.zy.hibernate.entity.Student@74174a23]}
*/
```

终于写完了，花了将近一天时间，心累。 拓展阅读：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=4&sn=2d8e9ee24886dc859a65605ba0ddf941&chksm=ebd74303dca0ca153fccc522fe9adf6e985e419c17717aed092235810b4685161d3df0c481c9&scene=21###wechat_redirecthttps://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=4&sn=2d8e9ee24886dc859a65605ba0ddf941&chksm=ebd74303dca0ca153fccc522fe9adf6e985e419c17717aed092235810b4685161d3df0c481c9&scene=21###wechat_redirect">Hibernate【映射】续篇</a>



参考资料： <a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=3&sn=8f21078918d5950fe468be8e62d39b91&chksm=ebd74303dca0ca1576bf3a7f44f837468eb23ac465ac94e39dd497a9423e7a3d090b566ea1b9&scene=21###wechat_redirect">Hibernate【映射】知识要点</a>
