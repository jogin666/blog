## Mybatis篇章（七） 常见面试题

写在前面：文章转载于：<a href="">Mybatis面试题</a>



**1、#{}和${}的区别是什么？**

在Mybatis中，有两种占位符：

- #{} 解析获取传递进来的参数的数据值
- ${} 解析获取传递进来的参数的数据值，然后拼接到sql语句上
- #{}是预编译处理，${}是字符串拼接，相当于mysql的concat函数
- ${}有注入问题，#{}没有注入问题（安全，推介使用）。



**2、当实体类中的属性名和表中的字段名不一样 ，怎么办 ？**

- 在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致
- 使用mybatis的 *resultMap* ，将表的字段和实体成员关联起来（推荐使用）



**3、如何获取自动生成的(主)键值?**

需求：

- user对象插入到数据库后，新记录的主键要通过user对象返回，通过user获取主键值。

解决思路：

- 通过LAST_INSERT_ID()获取刚插入记录的自增主键值，在insert语句执行后，执行select LAST_INSERT_ID()就可以获取自增主键。

mysql:

```xml
<insert id="insertUser" parameterType="cn.itcast.mybatis.po.User">       
    <selectKey keyProperty="id" order="AFTER" resultType="int">  
        select LAST_INSERT_ID()        
    </selectKey>        
    INSERT INTO USER(username,birthday,sex,address) 
    							VALUES(#{username},#{birthday},#{sex},#{address})    
</insert>
```



**4、在mapper中如何传递多个参数?**

- 使用占位符思想：在映射文件中使用 #{0}，#{1}，#{2} 代表传递进来的第几个参数（0位第一个，依次类推）

- @param注解方式：在接口的方法中使用@param注解参数，给参数一个别名，然后在sql语句中使用

```java
public interface usermapper { 
    User findUser(@param(“username”) string username, 
                    @param(“password”) string password); 
}
/*
<select id=”selectuser” resulttype=”user”> 
    select id, username, password from User where username = #{username} 
                                                            and password = #{password} 
</select>
*/
```

- 使用Map集合作为参数来装载，然后在sql使用map的key。如 password=#{password}

```java
public List<User> paginatedUser(int st,int e){
 	Map<String, Integer> map = new HashMap();
    map.put("start", start);
    map.put("end", end);
    return sqlSession.selectList("StudentID.pagination", map);
}
/*
    <select id="pagination" parameterType="map" resultMap="studentMap">
    	<!--根据key自动找到对应Map集合的value-->
        select * from students limit #{start},#{end};
    </select>
*/
```



**5、Mybatis动态sql是做什么的？都有哪些动态sql？能简述一下动态sql的执行原理不？**

- Mybatis动态sql可以让开发人员在xml映射文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql

- 的功能。

- Mybatis提供了9种动态sql标签：trim（头尾处理）、where（条件处理）、set（更新）、foreach（遍历）

  if（条件判定） 、[choose、when、otherwise] （三者实现 if-else if-lelse 的功能 ）、bind （bind标签可以

  使用OGNL表达式创建一个变量并将其绑定到上下文中）    

- 其执行原理为，使用OGNL从sql参数对象中计算表达式的值，根据表达式的值动态拼接sql，以此来完成动态

  sql的功能。



**6、Mybatis的Xml映射文件中，不同的xml映射文件，id是否可以重复？**

- 统一命名空间，id可重复，反之不行（会覆盖），因为*statement* 由*namespace+id* 确定的

 

**7、为什么说Mybatis是半自动ORM映射工具？它与全自动的区别在哪里？**

- Hibernate属于全自动ORM映射工具，使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。
- 而Mybatis在查询关联对象或关联集合对象时，需要手动编写sql来完成，所以，称之为半自动ORM映射工具。



**8、通常一个xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？**

- 在xml的映射文件中的<mapper>节点的 *namespace* 将 xml映射文件和接口绑定联系，然后接口的方法与映射文件中的节点的id一致，参数类型和返回类型一致。
- Mapper接口如没有实现类的，使用Mapper代理时，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MappedStatement。
- Dao接口里的方法，是不能重载的，因为Mapper的代理类是使用 全限名+方法名的保存和寻找策略。



**9、Mybatis比IBatis比较大的几个改进是什么**？

- .有接口绑定,包括注解绑定sql和xml绑定Sql
- b.动态sql由原来的节点配置变成OGNL表达式,
- c. 在一对一,多对一的时候引进了association,在一对多的时候引入了collection节点,不过都是在resultMap里面配置



**10、接口绑定有几种实现方式,分别是怎么实现的?**

- 注解绑定,在接口的方法上面加上@Select@Update等注解里面包含Sql语句来绑定。
- 通过xml里面写SQL来绑定，指定xml映射文件里面的namespace必须为接口的全路径名。



**11、Mybatis是如何进行分页的？分页插件的原理是什么？**

- Mybatis使用RowBounds对象进行分页，它是针对ResultSet结果集执行的内存分页，而非物理分页，可以在

  sql内直接书写带有物理分页的参数来完成物理分页功能，也可以使用分页插件来完成物理分页。

- 分页插件的基本原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的

  sql，然后重写sql，根据dialect方言，添加对应的物理分页语句和物理分页参数。

举例：`select * from student`，拦截sql重写为：`select t.* from （select * from student）t limit 0，10`



**12、简述Mybatis的插件运行原理，以及如何编写一个插件**

Mybatis仅可以编写针对ParameterHandler、ResultSetHandler、StatementHandler、Executor这4种接口

的插件，Mybatis使用JDK的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这4

种接口对象的方法时，就会进入拦截方法，具体就是InvocationHandler的invoke()方法，当然，只会拦截那些你

指定需要拦截的方法。实现Mybatis的Interceptor接口并复写intercept()方法，然后在给插件编写注解，指定要拦

截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。



**13、Mybatis是否支持延迟加载？如果支持，它的实现原理是什么？**

Mybatis仅支持 *association* 关联对象和 *collection* 对象集合的延迟加载，*association* 支持一对一的关联查询，

collection* 支持一对多的查询。延迟加载需要在mybatis的配置文件中手动配置，通过

*lazyLoadingEnabled=true|false* 来配置时候启用延迟加载。其原理是：底层使用CGLIB创建目标对象的代理对象，

当调用目标方法时，进入拦截器方法，比如调用 *a.getB().getName()* 方法时，拦截器的 *invoke()* 方法发现 *a.getB()* 

返回的是null，name就会单独发送事先保存好的查询关联B对象的sql语句，把对象B查询出来，然后调用 

*a.setB(b)* ，于是a对象的b成员就有值了，接着完成 *a.getB().getName()* 方法的调用（几乎使用的框架都是这样实现

的） 。



**14、Mybatis都有哪些Executor执行器？它们之间的区别是什么？**

Mybatis有三种基本的Executor执行器：SimpleExecutor、ReuseExecutor、BatchExecutor。

- SimpleExecutor：每执行一次update或select，就开启一个Statement对象，用完立刻关闭Statement对象。

- ReuseExecutor：执行update或select，以sql作为key查找Statement对象，存在就使用，不存在就创建，用

  完后，不关闭Statement对象，而是放置于Map内，供下一次使用。简言之，就是重复使用Statement对象。

- BatchExecutor：执行update（没有select，JDBC批处理不支持select），将所有sql都添加到批处理中

  （addBatch()），等待统一执行（executeBatch()），它缓存了多个Statement对象，每个Statement对象都

  是addBatch()完毕后，等待逐一执行executeBatch()批处理。与JDBC批处理相同。

作用范围：Executor的这些特点，都严格限制在SqlSession生命周期范围内。



**15、MyBatis与Hibernate有哪些不同？**

mybatis是一个不完全自动的ORM框架，因为mybatis需要开发人员编写sql语句。为mybatis提供了动态的sql语

句和注解sql语句两种方式给开发人员灵活配置业务要执行的sql语句，然后将查询结果封装成指定的返回类型。

mybatis简单易学，学习门槛较低。因为开发人员直接编写sql语句，可严格控制sql语句执行的性能，整体灵活度

高。mybatis非常适合对关系型数据库模型要求不高的软件开发工程（类软件需求变化频繁），例如：互联网软

件。企业运营类的软件等。但mybatis灵活是以无法做到数据库无关性的为条件的（使用官方sql语句，没有使用

数据库方言），如果需要实现支持多种数据库的软件则需要自定义多套sql映射文件，工作量大。

hibernate对象/关系映射能力强，数据库无关性好（配置文件配置使用数据库方言，业务使用hql语句），适合关

系模型要求的软件开发（例如需求固定的定制化软件）。使用hibernate开发可以减少很多开发工作量，提高开发

效率。hibernate的缺点是：学习门槛高，要精通就更难了，需要知道怎么设计O/R映射，如何在性能和对象模型

进行权衡，用好hibernate需要经验的积累和摄入学习才行。

总之，按照用户的需求在有限的资源环境下只要能做出维护性、扩展性良好的软件架构都是好架构，所以框架只有

适合才是最好的。
