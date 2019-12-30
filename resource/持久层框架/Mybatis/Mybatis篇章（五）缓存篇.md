## Mybatis篇章（五）缓存篇

在前面的四个篇章中，已经将mybatis讲的差不多了，此篇章将介绍mybatis的缓存策略，废话不多说，开始！



**1、mybatis缓存**

mybatis提供了一级缓存和二级缓存策略。其中一级缓存是 *SqlSession* 级别的，每一个 *SqlSession* 只能方位自己的

缓存，不能访问其它 *SqlSession* 的缓存，也就是每一个sql会话都有自己私有独立的缓存区。而二级缓存是*Mapper* 

级别的，同一个 *Mapper* 下的sql会话是可以共享缓存的（多个 *SqlSession* 去执行同一个映射文件下的sql查询查询

语句，获取到的是Mapper缓存中的数据）。如下图：

![mybatis缓存](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/images/mybatis%E7%BC%93%E5%AD%98.png)

当在dao层执行sq查询l语句时，mybatis会先查看时候开启二级缓存（默认不开启），如果开启二级缓存，则尝试

从二级缓存中查询是否有符合条件的数据，有则返回数据，没有则在一级缓存中查询，有则返回数据，没有则执行

想数据库发起执行sql语句查询请求，然后获取数据。执行该顺序的原因是：开启二级缓存后，每一个 *SqlSession*  

在执行查询的操作后，mybatis会将查询的结构放在一级缓存中，但是在关闭 *SqlSession* 后，mybatis会将该 

*SqlSession* 缓存的数据放入到二级缓存中。



**1、一级缓存**

mybatis是默认开启一级缓存的，因此开发人员不要进行配置。上面说了，mybatis的一级缓存是 *SqlSession* 级别

的。在mybatis中， *SqlSession* 第一次发出sql查询请求后，查询*SqlSession* 缓存中是否有符合条件的数据，有则返

回数据，没有，则向数据库发起sql查询请求，然后myabtis会将查询到的数据写入到 *SqlSession* 的私有缓存中。

同一个 *SqlSession* 再次执行相同的sql查询时，直接从 *SqlSession* 的私有缓存中捞取数据返回。若 *SqlSession* 执行

修改，删除，更新的sql操作，那么mybatis会将 *SqlSession* 的私用缓存的数据全部清空（防止脏读），然后如果

执行查询，再向数据库发起查询请求，然后将结果写入到缓存中。

![一级缓存](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/images/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98.png)

想一下，mybatis的一级缓存使用什么数据结构存储数据，该数据结构能根据查询的sql，快速获取到数据。当然

是Map的数据结构，该map的key是由  *hashcode+sql+sql输入参数+sql输出参数（sql的唯一标识）*构成的，而 

value便是 *查询的结果*。



**2、二级缓存**

mybatis只默认开启一级缓存，二级缓存是默认关闭的。因为原因是二级缓存是一个 *namespace* 级别的缓存，如

果在不同的 *namespace*下操作同一SQL 语句，可能导致缓存中的数据不正确。在进行多表联查的时候，也可能会

导致二级缓存中的数据不正确（要慎用）。

* 首先在mybatis的配置文件中进行开启二级缓冲的的配置

```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

* 在对相应的映射文件中启用缓存

```xml
<cache></cache>
```

上述就是开启二级缓存的步骤，值得注意的是：mybatis的二级缓存可以将内存的数据写入到数据库（磁盘），存

在对象的序列化与反序列化的操作，因此每一张表对应的实体类都需要实现 *java.io.serializable* 接口，否则程序会

抛出 *java.io.NotSerializableException* 异常。

![二级缓存](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/images/%E4%BA%8C%E7%BA%A7%E7%BC%93%E5%AD%98.png)

二级缓存也是使用map存储数据。



**3、禁用二级缓存、刷新缓存**

开启缓存的目的就是提高程序的性能，但是性能的提升是以抛弃数据的实时性为代价的。开启缓存后，即使数据库

的数据有修改，但程序查询获取到的的结果仍是缓存中的数据（存在一定风险）。因此对于某些经常变动的数据，

mybatis提供了 *useCache=false* 来禁用当前select语句使用二级缓存，即每次查询都会发出sql去查询。*useCache* 

默认情况是true，即该sql使用二级缓存。

```xml
<select id="findStuById" parameterType="string" resultType="student" useCache="false">     
    select * from mybatis where stuID=#{stuId}
</select>
```

为了防止发生脏读，mybatis提供了 *flushCache* 单独配置强制性刷新缓存。

```xml
<update id="updateUser" parameterType="cn.itcast.mybatis.po.User" flushCache="false">
    update user set username=#{username},birthday=#{birthday},sex=#{sex},address=#{address} where id=#{id}
</update>
```

* 在select语句中：

  flushCache默认为false，表示任何时候语句被调用时，都不会去清空本地缓存和二级缓存。

  useCache默认为true，表示会将本条语句的结果将使用二级缓存。

* 当为insert、update、delete语句时：

  flushCache默认为true，表示任何时候语句被调用，都会导致本地缓存和二级缓存被清空。
  

**4、了解Mybatis的缓存参数**

>   flushInterval（刷新间隔）可以被设置为任意的正整数，而且它们代表一个合理的毫秒形式的时间段。默认
>
> 情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新。
>
> size（引用数目）可以被设置为任意正整数，要记住你缓存的对象数目和你运行环境的可用内存资源数目。默
>
> 认值是1024。
>
> readOnly（只读）属性可以被设置为true或false。只读的缓存会给所有调用者返回缓存对象的相同实例。因
>
> 此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存会返回缓存对象的拷贝（通过序列化）
>
> 。这会慢一些，但是安全，因此默认是false。
>
> 
>
> 如下例子：
> <cache  eviction="FIFO"  flushInterval="60000"  size="512"  readOnly="true"/>
>
> 这个更高级的配置创建了一个 FIFO 缓存,并每隔 60 秒刷新,存数结果对象或列表的 512 个引用,而且返回的对
>
> 象被认为是只读的,因此在不同线程中的调用者之间修改它们会导致冲突。可用的收回策略有, 默认的是 LRU:
>
> 1.LRU – 最近最少使用的:移除最长时间不被使用的对象。
>
> 2.FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
>
> 3.SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
>
> 4.WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。  

第四点内容来源于：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483937&idx=5&sn=4a049d7461b67c4135183db09ec97bcb&chksm=ebd74320dca0ca3691081597ac9db2447d51250d7aa819009231760977dd932b43a116fe44ba&scene=21###wechat_redirect">Mybatis【缓存、代理、逆向工程】</a>



参考资料：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483937&idx=5&sn=4a049d7461b67c4135183db09ec97bcb&chksm=ebd74320dca0ca3691081597ac9db2447d51250d7aa819009231760977dd932b43a116fe44ba&scene=21###wechat_redirect">Mybatis【缓存、代理、逆向工程】</a>

<a href="https://blog.csdn.net/qq_35673617/article/details/80543873">mybatis开启二级缓存</a>

<a href="https://blog.csdn.net/codejas/article/details/79547932">浅谈MyBatis二级缓存</a>
