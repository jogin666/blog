## Hibernate篇章（七）Hibernate面试问题

写在前面，原文来源：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483906&idx=1&sn=16e43bebd697ad451e685038219ae2b9&chksm=ebd74303dca0ca1554311b16939713b4b23a9634ae614b302b8d748431aba6bc9aad83fadfbc&scene=21###wechat_redirect">Hibernate面试题大全</a>



**1、Hibernate工作原理及为什么要用？**

1. 读取并解析配置文件
2. 读取并解析映射信息，创建SessionFactory
3. 打开Sesssion
4. 创建事务Transation
5. 持久化操作
6. 提交事务
7. 关闭Session
8. 关闭SesstionFactory

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib05dv6goQSrS1nqGXq86dZUsEceku7k1nEms7c9m4Igliaw5tAUtbpC8XkDkTNgs0MXetHUAgcftpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

使用Hibernate框架就不用我们写很多繁琐的SQL语句。Hibernate实现了ORM，能够将对象映射成数据库表，从

而简化我们的开发！



**2、Hibernate是如何延迟加载(懒加载)?**

通过设置属性`lazy`进行设置是否需要懒加载。当Hibernate在查询数据的时候，hibernate并不会立即从数据库加

载数据放入内存中；当程序真正对数据的操作时，才会向数据库发起查询请求，就实现了延迟加载，节省了服务器

的内存开销，从而提高了服务器的性能。



**3、hibernate的三种状态之间如何转换**

Hibernate中对象的状态：

- 临时/瞬时状态： 对象被创建，但没有被ession的管理
- 持久化状态：对象已被保存到数据库，且在被ession的管理
- 游离状态：对象已被保存到数据库中，但已经不受session的管理



**4、比较hibernate的三种检索策略优缺点**

4.1、立即检索：

- 优点： 对应用程序完全透明，会把对象关联的数据全部加载到内存中，方便航到与它关联的对象（无论对象

  处于什么状态）。

- 缺点： 1.数据库查询访问次数过多；2.可能加载不程序需要的数据，浪费内存空间；

- 立即检索:`lazy=false`；

4.2、延迟检索：

- 优点： 程序需要真正访问数据时，才向从数据库请求载数据；减少数据库访问次数和内存，提高检索性能。

- 缺点： 对程序不够透明，应用程序如果希望访问游离状态代理类实例，必须保证其关联的对象在持久化状态

  时已经被初始化；

- 延迟加载：`lazy=true`；

**4.3、迫切左外连接检索：**

- 优点： 1. 对应用程序完全透明，会把对象关联的数据全部加载到内存中，方便航到与它关联的对象（无论对

  象处于什么状态）	2. 使用了外连接，数据库查询访问次数较少；

- 缺点： 1 . 可能加载不程序需要的数据，浪费内存空间；  2. 复杂的数据库表连接也会影响检索性能；

- 预先抓取： `fetch=“join”`；



**5、hibernate都支持哪些缓存策略**

usage的属性有4种：

- 放入二级缓存的对象，只读(Read-only);
- 非严格的读写(Nonstrict read/write)
- 读写； 放入二级缓存的对象可以读、写(Read/write)；
- 基于事务的策略(Transactional)



**6、ibernate里面的sorted collection 和ordered collection有什么区别**

* sorted collection：是在 内存中通过Java比较器 进行排序的

* ordered collection 是在 数据库中通过order by  进行排序的

* 对于比较大的数据集，为了避免在内存中对它们进行排序而出现 Java中的OutOfMemoryError，

  最好使用ordered collection。



**7、说下Hibernate的缓存机制**

7.1、 一级缓存：

- Hibenate中一级缓存，也叫做session的缓存，它可以在session范围内减少数据库的访问次数！  只在

  session范围有效！ Session关闭，一级缓存失效！

- 只要是持久化对象状态的，都受Session管理，也就是说，都会在Session缓存中。

- Session的缓存由hibernate维护，用户不能操作缓存内容； 如果想操作缓存内容，必须通过hibernate提供的

  evit/clear方法操作。

7.2、二级缓存：

- 二级缓存是基于应用程序的缓存，所有的Session都可以使用。

- Hibernate提供的二级缓存有默认的实现，且是一种可插配的缓存框架！如果用户想用二级缓存，只需要在

  hibernate.cfg.xml中配置即可； 不想用，直接移除，不影响代码。

- 如果用户觉得hibernate提供的框架框架不好用，自己可以换其他的缓存框架或自己实现缓存框架都可以。

- Hibernate二级缓存：存储的是常用的类。

![这里写图片描述](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib05dv6goQSrS1nqGXq86dZUb6f4l7W33NRViah0ZwEWLE9A3f6rpoy4YZ2UhFTRSibLhicqAAKq6icIoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**8、Hibernate的查询方式有几种**

- 对象导航查询(objectcomposition)

- HQL查询

- - 属性查询
  - 参数查询、命名参数查询
  - 关联查询
  - 分页查询
  - 统计函数

- Criteria 查询

- SQLQuery本地SQL查询



**9、如何优化Hibernate？**

- 数据库设计调整
-  HQL优化
- API的正确使用(如根据不同的业务类型选用不同的集合及查询API)
- 主配置参数(日志，查询缓存，fetch_size, batch_size等)
- 映射文件优化(ID生成策略，二级缓存，延迟加载，关联优化)
-  一级缓存的管理
- 针对二级缓存，还有许多特有的策略

详情可参考资料：https://www.cnblogs.com/xhj123/p/6106088.html



**10、谈谈Hibernate中inverse的作用**

inverse权限控制，维护表之间的关系，属性默认是false, 在多对多关系中就是两端都来维护关系。

- 比如Student和Teacher是多对多关系，用一个中间表TeacherStudent维护。Gp)

- 如果Student这边inverse=”true”, 那么关系由另一端Teacher维护，就是说当插入Student时，不会操作

  TeacherStudent表（中间表）。只有Teacher插入或删除时才会触发对中间表的操作。所以两边都

  inverse=”true”是不对的，会导致任何操作都不触发对中间表的影响；当两边都inverse=”false”或默认时，会

  导致在中间表中插入两次关系。

如果表之间的关联关系是“一对多”的话，那么inverse只能在“一”的一方来配置！

详情可参考：https://zhongfucheng.bitcron.com/post/hibernate/hibernate-inversehe-cascadeshu-xing-zhi-shi-yao-dian



**11、JDBC、hibernate 、 ibatis 三者的区别**

11.1、jdbc:手动

- 手动写sql

- delete、insert、update要将对象的值一个一个取出传到sql中,不能直接传入一个对象。

- select:返回的是一个resultset，要从ResultSet中一行一行、一个字段一个字段的取出，然后封装到一个对象

  中，不直接返回一个对象。

11.2、 ibatis的特点:半自动化

- sql要手动写
- delete、insert、update:直接传入一个对象
- select:直接返回一个对象

11.3、hibernate:全自动

- 不写sql,自动封装
- delete、insert、update:直接传入一个对象
- select:直接返回一个对象



**12、在数据库中条件查询速度很慢的时候,如何优化?**

1. 建立表的索引
2. 减少表之间的关联
3. 优化sql，尽量让sql很快定位数据，不要让sql做全表查询，应该走索引,把数据量大的表排在前面
4. 简化查询字段，没用的字段不要，已经对返回结果的控制，尽量返回少量数据

详情可参考：https://mp.weixin.qq.com/s?timestamp=1520300404&src=3&ver=1&signature=W6Fo7aDHiJtK4ecUcnSJ4h9bN0vRAcTPKBTgLWSJDsMcdQReJC487RYzUIU9UFYQdmgLFyss9cKifM*GFp*CEVLtaLlwjj2HaDOjsCRkTnwfVlUY5cDhSyRi-c8leheofZJVnu6wYQ3IvT*hYyVB1pQCqqnuXIWERaksjXuyNP8=



**13、什么是SessionFactory,她是线程安全么**

Hibrenate的SessionFactory是单例数据存储和线程安全的，以至于可以多线程同时访问。一个SessionFactory 在

启动的时候只能建立一次。SessionFactory应该包装各种单例以至于它能很简单的在一个应用代码中储存.



**14. get和load区别**

* get() 立即查询   load()懒加载

* get如果没有找到会返回null， load如果没有找到会抛出异常。

* get会先查一级缓存， 再查二级缓存，然后查数据库；load会先查一级缓存，如果没有找到，就创建代理对

  象， 等需要的时候去查询二级缓存和数据库。



**15、merge的含义**

- 如果session中存在相同持久化标识(identifier)的实例，用用户给出的对象的状态覆盖旧有的持久实例
- 如果session没有相应的持久实例，则尝试从数据库中加载，或创建新的持久化实例,最后返回该持久实例
- 用户给出的这个对象没有被关联到session上，它依旧是脱管的

详情可参考：http://cp3.iteye.com/blog/786019



**16、persist和save的区别**

- persist不保证立即执行，可能要等到flush；
- persist不更新缓存；
- save, 把一个瞬态的实例持久化标识符，及时的产生,它要返回标识符，所以会立即执行Sql insert
- 使用 save() 方法保存持久化对象时，该方法返回该持久化对象的标识属性值(即对应记录的主键值)；
- 使用 persist() 方法来保存持久化对象时，该方法没有任何返回值。

参考资料：http://blog.csdn.net/u010739551/article/details/47253881



**17、主键生成 策略有哪些**

17.1、自动生成策略

- identity  自增长(mysql,db2)

- sequence  自增长(序列)， oracle中自增长是以序列方法实现

- native  自增长【会根据底层数据库自增长的方式选择identity或sequence】

- - mysql数据库是identity
  - oracle数据库， 使用sequence序列的方式实现自增长

- increment  自增长(会有并发访问的问题，一般在服务器集群环境使用会存在问题。)

17.2、手动指定主键的值

- assigned   

17.3、UUID生成的值

- **uuid**

17.4、foreign(外键的方式)



**18、简述hibernate中getCurrentSession和openSession区别**

- getCurrentSession会绑定当前线程，而openSession不会。当把hibernate交给spring来管理之后，使用事务

  配置，有事务的线程会自动绑定当前的工厂里面的每一个session，而openSession是创建一个新session。

- getCurrentSession事务是有spring来控制的，而openSession需要手动开启和手动提交事务，

- getCurrentSession是不需要手动关闭的，因为工厂会自己管理，而openSession是需要手动关闭。

- getCurrentSession需要手动设置绑定事务的机制，有三种设置方式，jdbc本地的Thread、JTA、第三种

  是spring提供的事务管理机制org.springframework.orm.hibernate4.SpringSessionContext，而且srping默

  认使用该种事务管理机制



**19、Hibernate中的命名SQL查询指的是什么?**

- 命名查询指的是用`<sql-query>`标签在映射文档中定义的SQL查询，然后通过使用

  Session.getNamedQuery()方法进行调用。命名查询让开发人员重复编写hql语句。

- Hibernate中的命名查询既可以在映射文件编写，也可以使用注解来定义。Hibernate提供了@NameQuery定

  义单个的命名查询，@NameQueries定义多个命名查询。



**20、为什么在Hibernate的实体类中要提供一个无参数的构造器这一点非常重要？**

每个Hibernate实体类必须包含一个 无参数的构造器, 因为Hibernate框架要使用Reflection API，通过调用

Class.newInstance()来创建实体类的实例。如果在实体类中找不到无参数的构造器，这个方法就会抛出一个

InstantiationException异常。同时不建议将Hibernate的实体类定义为final类，因为 Java不允许对final类进行扩

展，所以定义为final类后，Hibernate就无法再使用代理，限制提升性能的手段。
