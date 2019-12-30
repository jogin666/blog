## Hibernate篇章（五）缓存篇

在前面的几篇中，已将把hibernate将的差不多了，此篇章打算讲述hibernate的缓存是怎么实现的。



### 一、Hibernate实体类对象状态

和mybatis一样，hibernate也分为两级缓存，一级缓存的级别是session级别（私有缓存），二级缓存是session

共享界别（公共缓存，默认不开启），集合缓存，查询缓存。在开始讲述hibnenate的缓存之前，先讲一下对象状

态，这样更快的了解hibernate的一级缓存。

* 临时状态：new出来的对象且没有交给session管理的对象的状态，就是临时/瞬时状态的。

* 持久状态：使用session查询的数据或者使用save（）等方法将临时状态的对象交给session管理，在session

  的事务没有提交之前，实体类对象的属性发生的改变，都会在session提交事务时，记录或者更新到数据。

  ```java
  @Test
  void example( ){
      Session session = HibernateConfig.getSession();
      Query query2 = session.createQuery("from StudentEntity as stu where stu.id=?1");
      query2.setParameter(1,16);
      StudentEntity entity = (StudentEntity) query2.uniqueResult();
      System.out.println(entity);
  
      Transaction transaction = session.beginTransaction();
      entity.setGender("00000"); //该改变被更新到数据库中
      System.out.println(entity);
      transaction.commit();
      session.close();
  }
  ```

* 游离状态：在session关闭之后，其使用sessioon查询的数据，也就是实体类或者交给session管理的对象（由

  临时状态变为持久化状态的对象），不再受到session的管理，实体类的属性发生改变之后，是不会被session
  
  更新到数据库（因为会话已经结束了）。



### 二、Hibernate的一级缓存

Hibenate的一级缓存，也叫做session的缓存，将查询的数据放入到session的私有缓存中（不同的session的缓存

是不一样的）。上面所说的持久化对象，就是对象处于session的私有缓存中，session的私有缓存是由hibernate

来维护的，是默认开启的，用户如果想操作session的缓存，只能是通过使用hibernate提供的evit/clear方法操作

。一级缓存可以减少session对数据库的访问次数！  session关闭，其私有缓存（一级缓存）也跟着被销毁，或者

说清空！

一级缓存可以减少session访问数据库的原因如下：

* 对只开启一级缓存的hibernate，当发起请求时，，session会先在其私有缓存中查询数据，如数据存在，则返

  回，反之则向数据库发起请求，然后将数据写入到一级缓存中，用于下次查询获取。

* session对于其管理的持久化对象，当提交事务时，session会先将对象与缓存的值比较，如果发生变化，则向

  数据库发起更新/保存数据的请求，反之，是不会发起请求的。在提交事务之前，session只会将最终版的对象

  的属性值与内存中数值比较，无论对象之前变化了多少次数，都只会向数据库请求1次或者0次请求。

  ```java
  @Test
  void example( ){
      Session session = HibernateConfig.getSession();
      Query query2 = session.createQuery("from StudentEntity as stu where stu.id=?1");
      query2.setParameter(1,16);
      StudentEntity entity = (StudentEntity) query2.uniqueResult();
      System.out.println(entity);
  
      Transaction transaction = session.beginTransaction();
      entity.setGender("female");
      entity.setGender("male");  //只会尝试更新这个记录，如果数据库记录的也是male，则是不会跟新的
      System.out.println(entity);
      transaction.commit();
      session.close();
  }
  ```

对于一级缓存，hibernate提供了三个方法给开发人员使用：

* *Session.flush();*      强制一级缓存与数据库同步

  ```java
  Transaction transaction = session.beginTransaction();
  entity.setGender("female");
  session.flush();    //增加刷新缓存的语句，其他与上述代码不变
  entity.setGender("male");
  System.out.println(entity); 
  transaction.commit();  //控制台有两条update的sql语句
  session.close();
  ```

* *Session.evict(Object obj);*    在一级缓存中清空指定的对象的记录

* *Session.clear();*      清空一级缓存的缓存数据



### 三、Hibernate的二级缓存

hibernate的二级缓存是session共享界别（公共缓存），是基于应有的缓存（缓存框架），其存储的实体类的对

象）。hibernate为二级缓存提供了默认的实现（框架），同时也支持其他缓存架构（包括开发人员自己编写的）

hibernate的二级缓存是默认不开启的，如果想要使用hibernate的二级缓存，需要在hibernate的配置文件中声

明。

**3.1、hibernate的缓存配置信息**

```properties
#hibernate.cache.use_second_level_cache false二  级缓存默认不开启，需要手动开启

#hibernate.cache.use_query_cache true      默认开启查询缓存
  
#二级缓存框架的实现如下，其他的缓冲需要开发人员实现接口，然后在配置文件中配置

#hibernate.cache.provider_class org.hibernate.cache.EhCacheProvider

#hibernate.cache.provider_class org.hibernate.cache.EmptyCacheProvider

#hibernate.cache.provider_class org.hibernate.cache.HashtableCacheProvider 默认实现

#hibernate.cache.provider_class org.hibernate.cache.TreeCacheProvider

#hibernate.cache.provider_class org.hibernate.cache.OSCacheProvider

#hibernate.cache.provider_class org.hibernate.cache.SwarmCacheProvider
```

**3.2、开启二级缓存**

* 配置文件声明开启

```xml
<session-factory>
    <!--使用二级缓存-->
	<property name="hibernate.cache.use_second_level_cache">true</property>
</session-factory>
```

* 指定二级缓存要缓存的实体类

```xml
<session-facory> 
    <!--usage：指定该实体的缓存策略
        1. read-only   :只读
		2. read-write  ：读写
		3. nonstrict-read-write //严格读写
		4. transactional   ：基于事务的策略
    -->
    <class-cache usage="read-write" class="com.zy.hibernate.bean.ClazzEntity"/>
    <class-cache usage="only-read" class="com.zy.hibernate.bean.StudentEntity"/>
</session-factory>
```

在hibernate的配置文件声明好了之后，就可以在程序中使用二级缓存了，但是二级缓存的使用需要谨慎，毕竟范

围有些大，缓存中的数据不会及时跟新。贴一张hibernate的一二级缓存图：

![hibernate缓存](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Hibernate/images/hibernate%E7%BC%93%E5%AD%98.jpg)

### 四、hibernate的集合缓存和查询缓存

**4.1、开启集合缓存**

在一对多，或者多对多的关联映射中，hibernate默认是没有关联表的数据集合(Set,List)设置二级缓存的，只会将

数据集合放入到一级缓存中，如果需要将集合数据的放入二级缓存需要在hibernate的配置文件中声明：

```xml
<collection-cache collection="com.zy.hibernate.bean.StudentEntity.courses" usage="read-write"/>
```

**4.2、数据集合的迭代：List和Iterator**

* List：会一次把关联表中的数据查询全部出来放入到内存中

* Iterator ：会先发起一条查询关联表主键的sql语句，获取所有符合条件记录的主键，然后根据迭代，使用主键

  一条一条记录的查询，将查询的数据放入到缓存中。总共会发起N+1次查询请求，N是符合的数据记录，1是

  第一次查询主键发起的。

**4.3、开启查询缓存**

* 查询缓存的定义

```java
Query query2 = session.createQuery("from StudentEntity")
```

* hibernate默认也是不会将查询缓存存入二级缓存的，想将查询缓存放入到二级缓存，需要在配置文件中开启

```xml
<!-- 开启查询缓存 -->
<property name="hibernate.cache.use_query_cache">true</property>
```

* 或者在程序中声明

```
Query query = session.createQuery("from StudentEntity ").setCacheable(true);
```

其实说白了，要使用二级缓存，就需要在配置文件或者代码中为其声明，hibernate才会将数据放入到二级缓存中



参考资料：

<a href="https://www.cnblogs.com/zxf160/p/9489151.html">Hibernate二级缓存介绍</a>

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=7&sn=a8538b4f42d29ce059204e8db8fcfaaa&chksm=ebd74303dca0ca1572d1e3d66605548e62827df7f1d03f023927867c5487c920fc26762ec5bb&scene=21###wechat_redirect">Hibernate【缓存】知识要点</a>
