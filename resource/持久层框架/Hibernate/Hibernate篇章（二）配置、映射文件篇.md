## Hibernate篇章（二） 配置文件和映射文件介绍

在上一个篇章《 <a href="https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Hibernate/Hibernate%E7%AF%87%E7%AB%A0%EF%BC%88%E4%B8%80%EF%BC%89%E5%85%A5%E9%97%A8%E7%AF%87.md">Hibernate篇章（一）入门篇</a>》，介绍hibernate是如何使用的了。本篇章主要介绍hibernate的配

置文件和映射文件。

### 一、配置文件介绍

*hibernate.cfg.xml* 介绍

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
        <!--使用mysql的方言，使用对应的数据库的sql语言-->
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
		<property name="hibernate.connection.provider_class" value="org.hibernate.connection.C3P0ConnectionProvider"/>

        
        <property name="hibernate.hbm2ddl.auto">update</property>
     <!--
		1.update 若表不存在，根据映射文件的信息，生成表
		2.create 无论表是否存在，都会生成新的表（表存在，先删除，后创建）
		3.create-drop  创建SessionFactory时，创建表；关闭时（SessionFactory.close()）,删除表
		4.validate：检验映射文件中标的信息与数据库表的信息是否一致，不一致抛出异常
     -->

        <!--加载实体与表的映射文件-->
        <mapping resource="student.hbm.xml"/>
    </session-factory>
    

</hibernate-configuration>
```

在配置文件的 *<hibernate-configuration*  下用两个节点：*<session-factory* 和 *<security* ，一般情况是不会使用到权

限配置的 *<security* 的。所以主要介绍 *<hibernate-configuration* ，该节点有一个属性为 name，用来指定名字的。 

其节点下，常用的配置选项如上，都有注解。



### 二、关系映射文件

实体类数数据库标的关系映射文件

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping>
    <!--将实体类与表建立管理映射-->
    <class name="com.zy.hibernate.entity.Student" table="student">
        <!--主键，采用自增策略-->
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

在 *<hibernate-mapping* 节点中有 *package* 属性用来指定该配置文件中描述实体所在的包，使用 *package* 指定实体

的路径后，在 *<class* 节点的 *name* 属性直接写实体类的类名就行了。

```xml
<hibernate-mapping name="com.zy.hibernate.entity">
    <!--将实体类与表建立管理映射-->
    <class name="Student" table="student">
        <!--主键，采用自增策略-->
        <id name="id" column="id">
            <generator class="native"/>
            <!--
				1.identity: 自增长（用于mysql，db2）
				2.sequence：自增长（用于oracle，使用序列方法实现）
				3.native： 自增长，根据数据选择（1或者2）
				4.increment：  自增长（在集群中使用，存在并发问题）
				5.assgned：	开发手动创建主键的值
				6.uuid：主键生成策略为UUID生成的值（32位）
				7.foreign：使用另外一个相关联的对象的主键作为该对象主键。主要用于一对一关系中。
				//.......
			-->
        </id>
        <!--实体对象的属性对应的字段    type：指定类型   length：指定长度-->
        <property name="userId" column="userId" type="java.lang.String"/>
        <property name="gender" column="gender" type="java.lang.String"/>
        <property name="name" column="name" type="java.lang.String"/>
        <property name="age" column="age" type="java.lang.Integer" length="6"/>
    </class>
</hibernate-mapping>
```

在 *<hibernate-mapping* 节点还有一个常用属性 *auto-import* ，默认为 true，表示在编写该实体对应操作的hql语

言时，直接使用类名就可以了，如果为false，则需要类的全限定名。*<class* 下的 *<id* 节点是指定表的主键，其必

须要有的，在其下有一个子节点 *<generator*，使用指定主键的生成策略的，常用的策略在上述配置文件中已经介

绍。除了 *<id* 节点之外，还有一个复合主键 *<composite-id* 节点，表的多个字段为主键。Hibernate的复合主键会

将多个主键字段生成一个实体类。也就是如果一个表有复合主键，其表关联的实体类，需要将主键字段，额外抽取

成一个pojo类，然后在实体类中维护一个复合主键的pojo的引用。例如将学生的id和stuI的作为表的主键。则需要

将 id 和 stuId 额外抽取成一个 pojo：CompositeKey  ，然后在学生实体类Student中维护一个CompositeKey 的

引用。

- CompositeKey ，值得注意的是复合主键的pojo一定需要实现 Serializable 接口。

```java
public class CompositeKey implements Serializable { 
    private int id;
    private String stuId;
	//......
}
```

- StudentEntity，在hinerate中，实体类应要实现 Serializable接口。

```java
public class StudentEntity implements Serializable {

    private CompositeKey keyId;

    private String gender;
    private String name;
    private int age;
    //...
}
```

映射文件

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.zy.hibernate.entity">
     <!--复合主键-->
        <composite-id class="CompositeKey" name="keyId">
            <!--复合主键类的属性-->
            <key-property name="id" column="id" type="java.lang.Integer" length="6"/>
            <key-property name="stuId" column="stuId" type="java.lang.String" length="11"/>
        </composite-id>
        <!--实体类属性-->
        <property name="age" column="age" type="java.lang.Integer" length="6"/>
        <property name="name" column="name" type="java.lang.String" length="20"/>
        <property name="gender" column="gender" type="java.lang.String" length="6"/>
</hibernate-mapping>
```

### 三、懒加载

和mybatis一样，hibernate的懒加载就是需要到数据时，才会向数据库发起查询请求，加载数据库的数据，否则

是不会把数据加载到缓存的。其目的就是为了提供hibernate的性能。

- get()：及时加载，立刻向数据库查询，将数据加载到内存。
- load()：默认使用懒加载，需要数据的时，才向数据库发起请求加载数据。

如果想使用懒加载，需要在关系映射文件中配置。

```xml
<class name="" table="card2" lazy="false">
<set name="" lazy="true">
<list name="" lazy=""/>    
```

`lazy`有对应三个属性：

- true   开启懒加载

- false   关闭懒加载

- extra   只有在set、list等集合标签中使用，其目的是：在使用懒加载策略时，提升效率

  - 在真正使用数据的时候才向数据库发送查询的sql；

  - 调用集合的size()/isEmpty()方法，方法只是统计数量，不是真正询数据，不会发起数据库请求。

`lazy`懒加载异常：在Session关闭后，不能使用懒加载，否则会报出异常。就是session关闭之后，不能尝试去获

取其关联表的数据。

解决方法：

- 方式1： 先简单使用数据，让hibernate加载数据
- - dept.getDeptName();
- 方式2：强迫代理对象初始化，加载数据
- - Hibernate.initialize(dept);

- 方式3：关闭懒加载



单元测试不写了，有兴趣的读者可以自行研究一下。

最后贴一下工程目录：

![hibernate(2)工程木目录](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Hibernate/images/hibernate(2)%E5%B7%A5%E7%A8%8B%E6%9C%A8%E7%9B%AE%E5%BD%95.png)




参考资料：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=2&sn=e8b7ab76aeb7a4895eab63f410938be9&chksm=ebd74303dca0ca1558a1015a5baf35dcfd59a59756ce52ebd8db84797b5dbff28b95929ddc09&scene=21###wechat_redirect">Hibernate入门这一篇就够了</a>

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=7&sn=a8538b4f42d29ce059204e8db8fcfaaa&chksm=ebd74303dca0ca1572d1e3d66605548e62827df7f1d03f023927867c5487c920fc26762ec5bb&scene=21###wechat_redirect">Hibernate【缓存】知识要点</a>


