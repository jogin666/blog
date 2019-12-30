## SpringData JPA篇章（三）Repository篇

前面的两个篇章分别用 maven 项目工程和 Spring Boot 工程讲述了，如何快速入门 SpringData JPA。此篇章将讲

述 SpringData JPA 的 Repository 接口及其子接口。



### 一、Repository 介绍

先来看一下，SpringData JPA的 Repository 接口和其子接口的关系

![jpa接口详情](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/SpringData%20JPA/images/jpa%E6%8E%A5%E5%8F%A3%E5%9B%BE%20.jpg)

在前两篇中说了，要使用SpringData JPA 的功能，则需要让dao层的接口继承 Repository 接口，或者用 

@RepositoryDefinition 注解类注解接口，其目的就是要使用 Repository 接口提供的命名查询功能，减少开发人

员的工作量。Repository接口有四个常用的子接口，分别提供了额外的功能。

* Repository 接口 ：命名查询

* CrudRepository 接口 ：数据库的crud操作

* PagingAndSortingRepository 接口：在curd的基础上增加了数据排序和分页的功能

* QueryByExampleExecutor 接口 ：提供了动态JPQL查询

* JpaRepository 接口 ： 优化/适配 上述的接口，适配其父类接口的返回值（Iterator）。

* @NoRepositoryBean 注解 ：该注解的接口不会被单独创建实例，只会作为其他接口的父接口而被使用。

  

**1.1、CrudRepository  接口介绍**

```java
@NoRepositoryBean					//继承repository接口，有命名查询功能
public interface CrudRepository<T, ID> extends Repository<T, ID> {
   
    <S extends T> S save(S entity); //保存单个对象
    
    <S extends T> Iterable<S> saveAll(Iterable<S> entities); //保存多个对象
    
    Optional<T> findById(ID id); //根据id查询
    
    boolean existsById(ID id); //根据id判断是否存在
    
    Iterable<T> findAll(); //查询全部
    
    Iterable<T> findAllById(Iterable<ID> ids); //根据多个id 查询
    
    long count();  //查询表的总记录
    
    void deleteById(ID id);
    
    void delete(T entity); //删除
    
    void deleteAll(Iterable<? extends T> entities); //删除多个
    
	void deleteAll(); //删除全部
}
```

**1.2、PagingAndSortingRepository 接口介绍**

```java
@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
	
    Iterable<T> findAll(Sort sort);  //提供排序
    
    Page<T> findAll(Pageable pageable); //提供分页
}
```

单元测试：（使用创建的 Spring Boot 工程项目）

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class StudentPagingdAndSortingRepositoryTest {

    @Autowired
    private StudentPagingdAndSortingRepository repository;

    //分页查询
    @Test
    public void findAll(){
        Sort.Order descOrder=new Sort.Order(Sort.Direction.DESC,"stuId"); //定义顺序
        Sort sort=new Sort(descOrder); //定义排序
        Iterable<StudentEntity> iterable = repository.findAll(sort);
        iterable.forEach(System.out::println);
    }

    //带排序的分页查询
    @Test
    public void sortedAndPage(){
        Sort.Order order=new Sort.Order(Sort.Direction.DESC,"age");
        Sort sort=new Sort(order);
        //index 是从0开始的，
        Pageable pageable=new PageRequest(0,5,sort);  //排序好的分页
        Page<StudentEntity> page = repository.findAll(pageable);
        System.out.println( page.getTotalElements());;//总记录数
        System.out.println(page.getTotalPages()); //总页数
        System.out.println(page.getNumber()); //当前页数
        System.out.println(page.getContent()); //内容
    }
}
```

**1.3、JpaRepository 接口介绍**

```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, 
QueryByExampleExecutor<T> {
	
    List<T> findAll(); //查询全部

	List<T> findAll(Sort sort); //查询全部并排序
    
    List<T> findAllById(Iterable<ID> ids); //根据多个id查询多个记录
    
    <S extends T> List<S> saveAll(Iterable<S> entities); //保存多个实体
    
    void flush(); //强制刷新缓存，更新到数据库
    
    <S extends T> S saveAndFlush(S entity); //刷新缓存并保存，将最新的数据跟新到数据库
    
    void deleteInBatch(Iterable<T> entities);  //批处理删除操作
    
    void deleteAllInBatch() //删除全部
        
    T getOne(ID id); //根据id查询
    
    @Override   //Example<实体类性> 支持动态查询
	<S extends T> List<S> findAll(Example<S> example); //根据实例查询
    
    @Override
	<S extends T> List<S> findAll(Example<S> example, Sort sort); //根据实例查询把并排序
}
```

* QueryByExampleExecutor 接口介绍

```java
//改接口实现了 动态查询  根据Example的泛型属性值动态生成JPQL语言  其中T是实体类
public interface QueryByExampleExecutor<T> {
    <S extends T> Optional<S> findOne(Example<S> var1);   //查询单个
 
    <S extends T> Iterable<S> findAll(Example<S> var1);  //查询多个
 
    <S extends T> Iterable<S> findAll(Example<S> var1, Sort var2);  //查询多个并排序
 
    <S extends T> Page<S> findAll(Example<S> var1, Pageable var2); //查询多个并分页
 
    <S extends T> long count(Example<S> var1); //查询匹配的数量
 
    <S extends T> boolean exists(Example<S> var1);  //是否存在
}
```

单元测试：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class StudentJpaRepositoryTest {

    @Autowired
    private StudentJpaRepository repository;

    @Test
    public void findAll(){
        StudentEntity entity = new StudentEntity();
        entity.setGender("male");
        entity.setAge(22);  //不设置年龄，默认为0
        Example<StudentEntity> example = Example.of(entity);

        List<StudentEntity> students = repository.findAll(example);
        System.out.println(students.size());
        students.forEach(System.out::println);
    }
}
```

**1.4、JpaSpecificationExecutor 接口介绍**

```java
/**
 * Interface to allow execution of {@link Specification}s based on the JPA criteria API	
 	该接口封装了 JPA的准则，也是实现了 JPA criteria API。
 */
public interface JpaSpecificationExecutor<T> {

	Optional<T> findOne(@Nullable Specification<T> spec);  //单个查询

	List<T> findAll(@Nullable Specification<T> spec); //查询全部

	Page<T> findAll(@Nullable Specification<T> spec, Pageable pageable); //分页

	List<T> findAll(@Nullable Specification<T> spec, Sort sort); //排序

	long count(@Nullable Specification<T> spec);  //查询总数
}
```

单元测试：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class StudentJSExecutorTest {

    @Autowired
    private StudentJSExecutor executor;

    /**
     * 筛选结果和分页
     */
    @Test
    public void findAll(){
        //排序
        Sort sort=new Sort(Sort.Direction.DESC,"stuId");
        /**
         * root 查询的类型
         * criteriaQuery  查询条件
         * criteriaBuilder 构建Predicate
         */
        Specification<StudentEntity> specification= (root, criteriaQuery, criteriaBuilder) -> {
            Path path = root.get("age");
            Predicate predicate = criteriaBuilder.gt(path, 20);
            return predicate;
        };
        //index 是从o开始的，
        Pageable pageable = PageRequest.of(0,2,sort);
        Page<StudentEntity> page = executor.findAll(specification,pageable);
        System.out.println( page.getTotalElements());;//总记录数
        System.out.println(page.getTotalPages()); //总页数
        System.out.println(page.getNumber()); //当前页数
        System.out.println(page.getContent());
    }
}
```

**1.5、@NameQuery 注解**

*@NameQuery* 是 SpringData JPA 为开发人员提供的命名注解，就是将常用的 JPQL 语句抽取出来，然后在相应的

实体类中声明，在需要时直接使用就好了（注意：SpringData JPA 会根据接口的方法会自动匹配，然后调用）。

```java
@Repeatable(NamedQueries.class)
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NamedQuery {
    String name();  //名称

    String query(); //JPQL 语句

    LockModeType lockMode() default LockModeType.NONE;  //上锁模式

    QueryHint[] hints() default {};  
}
/***************************分割线******************/
@NamedQueries(value={@NamedQuery(name="findAll",query = "select stu from StudentEntity stu")})
```

**1.6、关系映射**

在写hibernate的注解时，当时没有了解到其实hibernate使用的关联注解就是 JAP的定义的注解，其关联映射就

是使用jpa映射的，详情请查看：《<a href="https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/Hibernate/Hibernate%E7%AF%87%E7%AB%A0%EF%BC%88%E5%9B%9B%EF%BC%89%E6%B3%A8%E8%A7%A3%E7%AF%87.md">Hibernate篇章（四）注解篇</a>》



SpringData JPA篇章完结，最后贴一张项目工程目录：

![项目工程3](https://github.com/jogin666/blog/blob/master/resource/%E6%8C%81%E4%B9%85%E5%B1%82%E6%A1%86%E6%9E%B6/SpringData%20JPA/images/%E9%A1%B9%E7%9B%AE%E5%B7%A5%E7%A8%8B3.png)



推荐阅读与参考资料：

<a href="https://blog.csdn.net/Ditto_zhou/article/details/80830970">JPA SQL 查询、结果集映射(@NamedNativeQuery、@ColumnResult注解说明)</a>

<a href="https://www.cnblogs.com/linhuaming/p/11823952.html">[Spring Data JPA 提供的各种Repository接口作用](https://www.cnblogs.com/linhuaming/p/11823952.html)</a>
