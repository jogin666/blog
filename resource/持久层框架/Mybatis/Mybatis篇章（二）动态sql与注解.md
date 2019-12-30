## Mybatis篇章（二）动态sql语句与注解

继上一篇章《<a href="https://github.com/jogin666/Database-Persistence-Framework/blob/master/mybatis/Mybatis%E7%AF%87%E7%AB%A0%EF%BC%88%E4%B8%80%EF%BC%89%E5%85%A5%E9%97%A8%E7%AF%87.md">Mybatis篇章（一）入门篇</a>》之后，已经知道了Mybatis是如何使用的了 ，此篇章将介绍如何

在mybatis中使用动态sql。

**1、动态查询**

mybatis提供了<where> 节点和<if>节点配合使用，完成动态查询。例如：

```xml
<!--
	stuId,gender 是Map中的key，mybatis的<if>会自动获取Map的value，进行判定时候空
	<where>会自定组合成一个正常的 sql语句，条件符合就拼接，反之舍弃。
-->
<select id="findStuByContionds" parameterType="String" resultMap="studentMap">
    select * from mybatis
    <where>
        <if test="stuId!=null">
            and stuID=#{stuId}
        </if>
        <if test="gender!=null">
            and gender=#{gender}
        </if>
    </where>
</select>
<!--多个查询-->
<select id="findStuByContionds" parameterType="list" resultMap="studentMap">
    select * from mybatis
    <foreach collection="list",open="(" close=")" s
</select>
```

在 *StudentMapper* 接口中添加一个方法并在 *StudentMapperImpl* 实现类中实现，然后编写单元测试

```java
@Override
public Student findStuByConditions(Map<String, Object> conditions) {
    return session.selectOne("StudentMapper.findStuByConditions",conditions);
}
/************************分割线  单元测试 ************************/
@Test
public void testFindStuByConditions(){
    HashMap<String, Object> map = new HashMap<>();
    map.put("stuId","20161113033");
    map.put("gender","男");
    Student student = stuMapImpl.findStuByConditions(map);
    Assert.assertNotNull(student);
    System.out.println(student); 
//测试结果：Student{stuId='20161113033', name='李', gender='男', card=null}
}
```



**2、动态更新**

mybatis的动态更新是：<set>和<if>搭配使用实现的。

```xml
<!--
 将StudentMapper.xml的更新稍微修改一下，其他就不介绍了，看例子
 值得一说的是：每一个<if>中的内容末尾必须要有 逗号
 如果条件有多个，可以使用 <where> 节点。
-->
<update id="updateStuById" parameterType="Map">
    update mybatis
    <set>
        <if test="gender!=null and gender!=""">
            gender=#{gender},
        </if>
        <if test="name!=null">
            name=#{name},
        </if>
    </set>
    where stuID=#{stuId}
</update>
```

将单元测试中的根据id更新的方法稍微修改一下。

```java
@Override
public void updateStuById(Map<String, String> conditions) {
    session.update("StudentMapper.updateStuById",conditions);
    close();
}
/************************分割线  单元测试 ************************/
@Test
public void testUpdateStuById(){
    HashMap<String,String> condition = new HashMap<>();
    condition.put("stuId","20161113033");
    condition.put("name","Tom");
    condition.put("gender","女");
    stuMapImpl.updateStuById(condition);
}
```



**3、动态删除**

对于mybatis的删除主要的讲一下删除多个吧。回顾一下，在sql中，如果要删除多个，该怎么实现呢？那就是使用 *where id in ( x, x, x,)* 。为此mybatis提供一个节点 <foreach>循环遍历。

```xml
<delete id="deleteStuById" parameterType="list">
    delete from mybatis where stuID in
    <foreach collection="list" open="(" separator="," close=")" 
             index="index" item="stuId">
        #{stuId}
    </foreach>
</delete>
```

*collection* 指定传入的类型，类型有 list，array，Map[必须是参数名，不能为map]， *open* 和 *close* 拼接一个中括

号， *separator* 指定元素的分隔符，*index* 指定迭代次数，*item* 是元素别名。将 *StudentMapper* 接口的删除方法，

稍微修改一下，直接上代码。

```java
@Override
public void deleteStuById(List<String> stuIdList) {
    session.delete("StudentMapper.deleteStuById",stuIdList);
    close();
}
/************************分割线  单元测试 ************************/
@Test
public void testDeleteStu(){
    List<String> stuIds=new ArrayList<>(3);
    stuIds.add("20161113032");
    stuIds.add("20161113033");
    stuMapImpl.deleteStuById(stuIds);
}
```

想深入学习的话，传送门：<a href="https://blog.csdn.net/qq_36698956/article/details/90208459">Mybatis查询中使用foreach的三种用法</a>



**4、动态添加**

在将动态删除之前，先将一下，mybatis的sql片段吧。mybatis提供了sql片段，让开发人员可以将常用的sql语

句，提出出来成为一个片段，让在有需要的时候导入。很简单的例子：

```xml
<!--提出片段-->
<sql id="selectStuById">
    select mybatis where stuID=#{stuId}
</sql>
<!--引用片段-->
<select id="findStudent" resultMap="studentMap" parameterType="string">
    <include refid="selectStuById"/>
</select>
```

接下来就上动态增加的语句：

```xml
<insert id="addStudent" parameterType="student">
    insert into mybatis(<include refid='key'/>) values (<include refid="value"/>)
</insert>

<sql id="key">
    <trim suffixOverrides=","> <!--手动去除尾部逗号（因为无法得知什么地方停止）-->
        <if test="name!=null and name!=''">
            name,
        </if>
        <if test="gender!=null and gender!=''">
            gender,
        </if>
        <if test="stuId!=null and stuId!=''">
            stuId,
        </if>
    </trim>
</sql>
<sql id="value">
    <trim suffixOverrides=","> <!--手动去除尾部逗号-->
        <if test="name!=null and name!=''">
            #{name},
        </if>
        <if test="gender!=null and gender!=''">
            #{gender},
        </if>
        <if test="stuId!=null and stuId!=''">
            #{stuId},
        </if>
    </trim>
</sql>
```

代码就不演示了。



**5、基本注解使用**

mybatis是支持注解的，开发人员可以使用 *org.apache.ibatis.annotations* 包下，mybatis提供的注解。接下演示

基本注解（@select，@Insert，@Update，@Delete），在dao层新建一个EntityMapper，然后是有注解：

```java
public interface EntityMapper {

    //参数只有一个时，直接在注解中使用参数名
    @Select("select * from mybatis where stuID=#{stuId}")
    Student findStuById(String stuId);

	/*多个参数，可以使用占位符，mybatis只支持提供的两种占位符 arg0~argn 或者 param0 ~ paramn
		@Select("select * from mybatis where stuID=#{arg0} and name=#{arg1}")
	*/
    @Select("select * from mybatis where stuID=#{param}0 and name=#{param1}")
    Student findStuByConditions(String stuId,String name);

    //使用Map封装参数，在sql中使用的是 map的key（mybatis会自动获取key对应的value）
    @Select("select * from mybatis where stuID=#{stuId} and name=#{name}")
    Student findStuByCondition(Map<String,String> map);
    
     //参数多个时，可以使用@Param注解参数，然后在sql语句中使用注解的别名
    @Update("update mybatis set name=#{name} where stuID=#{stuId}")
    void updateStudent(@Param("stuId") String stuId,@Param("name") String name);
    
    @Select("select * from mybatis where stuID=#{stuId} and name=#{name}")
    Student findStuByCondition(Map<String,String> map);
    
    @Delete(" delete from mybatis where stuID=#{stuId}")
    void deleteStuById(String stuId);
    
    @Insert("insert into mybatis set(stuID,name,gender) values(#{stuId},#{name},#{gender})")
    void save(Student student);
}
```

至于测试，就演示了，有兴趣的话，可以编写单元测试。但要说的是：注解必须要搭配Mapper代理使用，同时在

在mybatis的配置文件的 <mappers 中配置 *<mapper class="com.zy.dao.EntityMapper"/*。想了解高

级注解的话：传送门走一波：<a href="https://blog.csdn.net/xushiyu1996818/article/details/89108210">mybatis 常用注解</a>。最后贴一张mybatis的常用注解：

![mybatis常用注解](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Mybatis/images/mybatis%E5%B8%B8%E7%94%A8%E6%B3%A8%E8%A7%A3.png)





参考资料：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483937&idx=2&sn=28c7827639bb6ac0296746c4c4343c59&chksm=ebd74320dca0ca36b763b3975665fc38a7e921f9ecaef1aaea3a7c757063a29222cd00b3d3b6&scene=21###wechat_redirect">Mybatis【入门】</a>

<a href="https://blog.csdn.net/qq_36698956/article/details/90208459">Mybatis查询中使用foreach的三种用法</a>

<a href="https://www.jb51.net/article/130279.htm">mybatis学习笔记之mybatis注解配置详解</a>
