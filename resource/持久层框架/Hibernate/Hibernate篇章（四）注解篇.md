## Hibernate篇章（四）注解篇

在前面的三个篇章中，已经介绍hibernate的入门，配置文件，表关联映射了，此篇章将介绍hibernate的注解，

在实体上使用注解。在这之前，先了解hiberna的逆向工程吧。传送门：<a href="https://www.cnblogs.com/java-class/p/6208356.html">[IDEA 中生成 Hibernate 逆向工程实践](https://www.cnblogs.com/java-class/p/6208356.html)</a>



**1. 实体类**

在IDEA使用逆向工程生成的实体类如下

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
	//equals and hashCode
}

```



**2.生成注解的解释**

- @Entity： 表名该类是实体类，该注解有一个name属性，为实体类起名字

- @Table：将实体类与数据库中的表映射关联起来

  ```java
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Table {
      String name() default ""; //表名
  
      String catalog() default ""; 
  
      String schema() default ""; //数据库名
   
      UniqueConstraint[] uniqueConstraints() default {};  //外键约束
  
      Index[] indexes() default {}; //索引
  }
  ```

- Id：注解属性，表名该属性是主键

- @Basic  ：默认是Entity类的属性的默认注解。

  ```java
  public @interface Basic {
      FetchType fetch() default FetchType.EAGER;  //加载类型，默认立即加载，也支持来加载
  
      boolean optional() default true; //是否为空
  }
  ```

- @Column： 将实体类属性与表的字段关联映射

  ```java
  @Target({ElementType.METHOD, ElementType.FIELD})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Column {
      String name() default "";
  
      boolean unique() default false;  //是否为唯一
  
      boolean nullable() default true;  //是有为空
  
      //表示在使用“INSERT”脚本插入数据时，是否需要插入该字段的值。
      boolean insertable() default true; 
  
      boolean updatable() default true;  //更新时是否用此字段
  
      String columnDefinition() default "";  //字段定义
  
      String table() default ""; //表名
  
      int length() default 255; //长度
  
      //precision属性和scale属性表示精度，当字段类型为double时，precision表示数值的总长度，scale表示小数点所占的位数。
      int precision() default 0;
      int scale() default 0;
  }
  ```



**3、其他常用注解**

- @Temporal：声明日期类型。

  ```java
  @Temporal(TemporalType.TIMESTAMP) // 是用来定义日期类型
  private Date publicationDate; // 出版日期
  
  /*********分割线**********/
  public @interface Temporal {
      TemporalType value();
  }
  /*********分割线**********/
  public enum TemporalType {
      DATE, //日期：年月日
      TIME, //时间： 时分秒
      TIMESTAMP; //时间戳 
  
      private TemporalType() {
      }
  }
  ```

- `@Type`：指定Hibernate提供的类型

  ```java
  @Type(type="double") //
  private Double price; 
  
  /*********分割线**********/
  @Target({ElementType.FIELD, ElementType.METHOD})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Type {
      String type();
      Parameter[] parameters() default {}; //参数
  }
  /*********分割线**********/
  @Target({})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Parameter {
      String name();
      String value();
  }
  ```

- @GeneratedValue：主键生成策略

  ```java
  @Id
  @GeneratedValue //默认程序控制
  private int id;
  
  /*********分割线**********/
  @Target({ElementType.METHOD, ElementType.FIELD})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface GeneratedValue {
      
      GenerationType strategy() default GenerationType.AUTO;
      String generator() default ""; //指定自定义的主键生成策略
  }
  /*********分割线**********/
  public enum GenerationType {
      TABLE, //使用一个特定的数据库表格来保存主键
      
      SEQUENCE,  //使用数据库底层的序列生成主键，oracle
      
      IDENTITY,  //梓增长，高并发，会可能出错
      
      AUTO;  //：主键由程序控制
  
      private GenerationType() {
      }
  }
  ```

- @GenericGenerator：主键生成器，指定主键生成策略，结合@GeneratedValue使用

  ```java
  @Id
  @GenericGenerator(name="myuuid", strategy="uuid") // 声明一种主键生成策略(uuid)
  @GeneratedValue(generator="myuuid") // 引用自定义主键生成策略 uuid
  private String id;
  
  /*********分割线**********/
  @Target({ElementType.PACKAGE, ElementType.TYPE, ElementType.METHOD, ElementType.FIELD})
  @Retention(RetentionPolicy.RUNTIME)
  @Repeatable(GenericGenerators.class)
  public @interface GenericGenerator {
      String name(); //策略名，用于引用
  
      String strategy();  //策略
  
      Parameter[] parameters() default {}; //参数
  }
  ```

- @Transient：声明实体类属性只是暂时性，是不会写入到数据库的表中

  ```java
  @Transient
  private String demo; // 属性不会被在表中生成
  ```



**4、关联映射**

- @OneToOne ：一对一 （基于主键的关联映射）

  ```java
  public @interface OneToOne {
      Class targetEntity() default void.class; //目标类
  
      CascadeType[] cascade() default {};  //级联
  
      FetchType fetch() default FetchType.EAGER;  //加载类型
  
      boolean optional() default true; //是否为空
  
      String mappedBy() default "";  //控制权限（相当于inverse），指定权限交给指定名字的实体类对象维护，用于一对多中一的那方。
  
      boolean orphanRemoval() default false;  //是否清除关联表的数据，还是只解除关系，默认解除关系。常用一对多关系
  }
  /*********分割线**********/
  public enum CascadeType {
      ALL, //全部
      PERSIST, //级联保存
      MERGE, //级联更新
      REMOVE, //级联删除
      REFRESH, //级联刷新（获取），重新获取A时，也会重新获取A关联的数据
      DETACH; //级联脱管/游离操作，存在外键约束时，先删除外键约束，然后在删除对象
  
      private CascadeType() {
      }
  }
  ```

- @OneToMany：一对多

  ```java
  @Target({ElementType.METHOD, ElementType.FIELD})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface OneToMany {
      Class targetEntity() default void.class;
  
      CascadeType[] cascade() default {};
  
      FetchType fetch() default FetchType.LAZY;
  
      String mappedBy() default "";
  
      boolean orphanRemoval() default false;
  }
  ```

- @ManyToOne

  ```java
  @Target({ElementType.METHOD, ElementType.FIELD})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface ManyToOne {
      Class targetEntity() default void.class;
  
      CascadeType[] cascade() default {};
  
      FetchType fetch() default FetchType.EAGER;
  
      boolean optional() default true;
  }
  ```

- @MantToMany

  ```java
  @Target({ElementType.METHOD, ElementType.FIELD})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface ManyToMany {
      Class targetEntity() default void.class;
  
      CascadeType[] cascade() default {};
  
      FetchType fetch() default FetchType.LAZY;
  
      String mappedBy() default "";
  }
  ```

- @JoinColumn

  ```java
  public @interface JoinColumn {
      String name() default "";  //外键列
  
      String referencedColumnName() default ""; //关联表的主键，也是外键关联的列
  
      boolean unique() default false; //是否唯一值
   
      boolean nullable() default true; //是否能为控制
  
      boolean insertable() default true;
  
      boolean updatable() default true;
  
      String columnDefinition() default "";
  
      String table() default "";
      
  	//外键约束
      ForeignKey foreignKey() default @ForeignKey(ConstraintMode.PROVIDER_DEFAULT);
  }
  ```

- @JoinTable

  ```java
  @Target({ElementType.METHOD, ElementType.FIELD})
  @Retention(RetentionPolicy.RUNTIME)
  public @interface JoinTable {
      String name() default "";
  
      String catalog() default "";
  
      String schema() default "";
  
      JoinColumn[] joinColumns() default {};  //关联字段
  
      JoinColumn[] inverseJoinColumns() default {};  //另一个表的关联字段
  
      ForeignKey foreignKey() default @ForeignKey(ConstraintMode.PROVIDER_DEFAULT);
  
      ForeignKey inverseForeignKey() default 
          @ForeignKey(ConstraintMode.PROVIDER_DEFAULT);
  
      UniqueConstraint[] uniqueConstraints() default {}; //唯一约束
  
      Index[] indexes() default {};
  }
  ```

**5、关系映射注解使用**

在将如何使用关联映射之前，先创建四张表，student表，class表，card表，courses表，sc表（选课）；

student与card是一对一的关系，student与class是多对一的关系，class和student是一对多的关系，student和

course是多对多的关系。

- class表

  ```sql
  create table class(
    id         int auto_increment  primary key,
    class_name varchar(20) charset utf8 not null
  );
  ```

- student表

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

- card表

  ```sql
  create table card2(
    id     int         not null primary key,
    number varchar(11) not null,
    constraint card_student_sid_fk foreign key (id) references student (sid)
  );
  
  ```

- courses表

  ```sql
  create table courses(
    id          int auto_increment primary key,
    course_name varchar(11) null
  );
  
  ```

- sc表

  ```sql
  create table sc(
    courseId int null,
    sId      int not null,
    constraint sc_coueses_id_fk foreign key (courseId) references courses (id),
    constraint sc_student_sid_fk foreign key (sId) references student (sid)
  );
  
  ```

  使用反向工程生成实体类。

- 多对一

  ```java
  @ManyToOne
  @JoinColumn(name = "classId", referencedColumnName = "id")
  public ClazzEntity getClazzByClassId() {
      return clazzByClassId;
  }
  /*
  @JoginColumn关联两个表，name指定一对多中一的那一方表的外键，referencedColumnName指定：多的那一方的表的主键
  */
  
  ```

- 一对多

  ```java
  @OneToMany(targetEntity=StudentEntity.class)
  private Set<StudentEntity> students;
  
  ```

- 多对多

  ```java
  @ManyToMany(targetEntity=CoursesEntity.class)
  @JoinTable(name="sc",joinColumns={@JoinColumn(name="sId",referencedColumnName="sid")},
       inverseJoinColumns={@JoinColumn(name="courseId",referencedColumnName="id")})
  private Set<CoursesEntity> courses;
  /*
  @JoinTable中的name 指定中间表（sc） joinColumns指定sc表与student表的JoinColumn，inverseJoinColumns指定sc表盒courses表的联系，从而根据sid对应的courseId获取courses表的数据
  */
  
  ```

- 一对一

  基于外键的映射

  ```java
  @OneToOne(targetEntity=Card2Entity.class)
  private Card2Entity card;
  
  /*********分割线**********/
  
  @OneToOne(targetEntity=StudentEntity.class，,mappedBy="card2")
  @JoinColumn(name="sid",referencedColumnName="sid")
  @Cascade(CascadeType.SAVE_UPDATE)
  private StudentEntity Student;
  
  ```

  基于主键的映射

  ```java
  @OneToOne(targetEntity=Card2Entity.class，mappedBy="card2") // 
  @PrimaryKeyJoinColumn
  private StudentEntity Student;
  
  /*********分割线**********/
  
  @OneToOne(targetEntity=StudentEntity.class) // 
  @PrimaryKeyJoinColumn
  public Card2Entity card;
  
  ```

  

参考资料：

<a href="https://blog.csdn.net/qq_38977097/article/details/81355840">Hibernate 框架学习——Hibernate注解</a>

<a href="https://www.cnblogs.com/xiaobaizhiqian/p/7932561.html">Hibernate---实体类注释简介</a>

<a href="https://www.cnblogs.com/zzmb/p/7733677.html">Hibernate注解详解(超全面不解释)</a>

<a href="https://blog.51cto.com/12402717/2087876">SSH框架之Hibernate5专题9：Hibernate注解式开发
糖醋白糖</a>

<a href="https://www.cnblogs.com/qlqwjy/p/9545453.html">Hibernate注解开发、注解创建索引</a>
