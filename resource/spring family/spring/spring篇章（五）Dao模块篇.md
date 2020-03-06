## Spring 篇章（五）Dao模块篇

#### 1.前言

在上一个篇章中，讲述了 spring 中的切面编程——Aop，因此这个篇章将要介绍 spring 的 Dao模块。对于 JDBC，相信对于大家都不会陌生，在初学的时候，肯定写过很多的 JDBC 代码或者模板。

#### 2.传统 JDBC 回顾

```java
public class DBCon {

    // 数据驱动类的路径
    private final String driverName="com.microsoft.sqlserver.jdbc.SQLServerDriver";
    //数据库连接字段
    private final String url="jdbc:sqlserver://localhost:1433;DatabaseName=Dream";
    //用户名
    private final String userName="sa";
    //用户密码
    private final String passWord="123456";
    //数据库连接对象
    private Connection con=null;
    //数据库查询集
    private ResultSet rs=null;

    /**
     * 数据库连接
     */
    public DBCon() {
        try {
            Class.forName(driverName);
            con= DriverManager.getConnection(url,userName,passWord);
        } catch(ClassNotFoundException e) {
            System.err.println("装载 JDBC驱动程序失败。" );
            e.printStackTrace();
        } catch(SQLException e) {
            System.err.println("无法连接数据库" );
            e.printStackTrace();
        }
    }

    /**
     * 数据库查询
     */
    public ResultSet executeQuery(String sql, String[] str) {
        try {
            PreparedStatement pst=getPst(sql,str);
            rs = pst.executeQuery();
        } catch (SQLException e) {
            System.err.println("发生异常：" + e.getMessage());
            System.err.println("异常SQL语句：" + sql);
        }
        return rs;
    }

    /**
     * 数据库的增删改
     */
    public int executeUpdate(String sql, String str[]) {
        int rowCount = 0;  //操作影响的行数
        try {
            PreparedStatement pst=getPst(sql,str);
            rowCount = pst.executeUpdate();
        } catch (SQLException e) {
            System.err.println("发生异常：" + e.getMessage());
            System.err.println("异常SQL语句：" + sql);
        }
        return rowCount;
    }

    /**
     * 关闭数据库
     */
    public void close() {
        try {
            con.close();
            con = null;
        } catch (SQLException e) {
            System.out.println("数据库关闭操作出现异常"+e.getMessage());
        }
    }

    /**
     * 获取PreparedStatement
     */
    private PreparedStatement getPst(String sql, String str[]) throws SQLException {
        PreparedStatement pst = con.prepareStatement(sql);
        if (str != null) {
            for (int i = 0; i < str.length; i++) {
                pst.setString(i + 1, str[i]);
            }
        }
        return pst;
    }

}
```

上面的代码就是本人在当初学 Java 数据库编程时，编写的 DbUtils 工具类，还蛮使用的，有了工具类之后，之后的每次数据库操作，都可以直接使用，唯一不足的地方就是查询的结果集需要自己封装成实体类，在增删改的时候，需要手动拼接 sql 语句。

于是乎，接下来就是用到 apache 的开源工具包：dbutils；

```java
@Repository
public class UserDaoImpl {

    //数据库的工具类
    private QueryRunner runner=new QueryRunner();

    //获取数据库连接
    private Connection getConnection(){
        return DBHelper.getConn();
    }

    //关闭数据库连接
    private void close(){
        DBHelper.close();
    }

    //增加用户
    public boolean addUser(Object [] userInfo){
        int rowNum=0;
        String sql="insert into user(useraccount) values (?)";
        try{
            rowNum=runner.execute(getConnection(),sql,userInfo);
        }catch (SQLException e){
            System.out.println("异常sql语句"+sql);
            System.out.println("·查询出错"+e.getMessage());
        }finally {
            close();
        }

        if (rowNum!=0){
            return true;
        }
        return false;
    }

    //更新用户信息
    public boolean updateUser(Object [] userInfo){
        String sql="update user set username=?,gender=?,headImg=?,brithday=?,selfIntroduction=? where useraccount=?";
        int rowNum=0;
        try{
            rowNum=runner.update(getConnection(),sql,userInfo);
        }catch (SQLException e){
            System.out.println("异常sql语句"+sql);
            System.out.println("·查询出错"+e.getMessage());
        }finally {
            close();
        }
        if (rowNum==1){
            return true;
        }
        return false;
    }
    
    //by account 查询用户信息
    public UserEntity findUserByAccount(String account){
        String sql="select * from user where useraccount=?";
        try{
            return (UserEntity)runner.query(getConnection(),sql,new BeanHandler<>(UserEntity.class),new Object[]{account});
        }catch (SQLException e) {
            System.out.println("异常sql语句"+sql);
            System.out.println("·查询出错"+e.getMessage());
        }finally {
            close();
        }
        return null;
    }
}
```

apache 的开源工具包 dbutils，相对于自己编写的 DbUtils 工具类确实比较好用，ubuitils 工具包中的类，会自动将查询到的结果集封成实体类，自动将参数编织到 sql 语句中，不用开发人员去负责。

#### 3.Spring 的 JDBC

要使用 spring  Dao 模块的 JDBC ，必须导入相应的依赖包。

```xml
<dependency>
    <!--jdbc 所依赖的包-->
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.3.13.RELEASE</version>
</dependency>

<dependency>
     <!--事务 所依赖的包-->
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.3.13.RELEASE</version>
</dependency>
```

像传统的 jdbc 一样，需要将数据库连接的信息封装在在工具类一样，使用 spring Dao 模块中的 jdbc时，也是需要将这些数据库资源配置在相应的 bean 中。spring 对于第三方的数据库连接池是有很好的支持的，因此配置的 bean 是有两种方式，一种是 spring 自带的 bean，另外一种便是第三方的数据库连接池了。

**3.1、使用 spring 自带的内置类 **

spring Dao 模块为开发人员提供数据库的连接类 *org.spring.jdbc.datasource.SimpleDriverDataSource*，由于使用的是 spring 自带的内置类或者是第三方的包，使用 xml 配置方式是比较方便的，因此在此使用是 xml 配置文件中配置装载连接数据信息的 bean。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="url" value="jdbc:mysql://localhost:3306/javaweb?serverTimezone=GMT%2B8"/>
        <property name="username" value="root"/>
        <property name="password" value="zy131456"/>
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    </bean>

    <context:component-scan base-package="net.dao"/>
</beans>
```

编写 dao 类

```java
@Component
public class DaoTest {

    @Autowired
    private DataSource dataSource;

    public Student findStuById(String stuId) throws SQLException {
        String sql="select * from student where stu_id= "+stuId;
        Connection connection = dataSource.getConnection();
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery(sql);
        Student student=new Student();
        while (resultSet.next()){
            student.setName(resultSet.getString("name"));
            student.setName(resultSet.getString("gender"));
            student.setAge(resultSet.getInt("age"));
        }
        return student;
    }
}
```

编写单元测试

```java
public class DaoTestTest {

    private ClassPathXmlApplicationContext act;

    @Before
    public void setUp() throws Exception {
        act=new ClassPathXmlApplicationContext("dao.xml");
    }

    @Test
    public void findStuById() throws SQLException {
        DaoTest daoTest = act.getBean("daoTest", DaoTest.class);
        Student stu = daoTest.findStuById("123456");
        System.out.println(stu);
    }
}
/*控制台打印结果
Student{name='male', gender='null', age=22}
*/
```

**3.2、使用数据库第三方连接池**

导入 c3p0  数据数据源的依赖包

```xml
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>RELEASE</version>
</dependency>
```

在 spring 的配置文件中 配置 c3p0 数据源的 bean，替换原来的 dataSource。

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/javaweb?serverTimezone=GMT%2B8"></property>
    <property name="user" value="root"></property>
    <property name="password" value="zy131456"></property>
    <property name="initialPoolSize" value="3"></property>
    <property name="maxPoolSize" value="10"></property>
    <property name="maxStatements" value="100"></property>
    <property name="acquireIncrement" value="2"></property>
</bean>
```

dao 类 和单元测试类不变，测试结果是一样的。

#### 4. spring 的 jdbcTemplate

Spring 中提供了一个 JdbcTemplate 类，它自己已经封装了一个 DataSource 类型的变量，开发人员在使用 spring 提供的 JdbcTeplate 类时，只需配置其 dataSource 变量就可以使用了。

```xml
<bean class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

重新编写 DaoTest 类，虽然 JdbcTemplate 这个类提供了很多的查询方法，但一般常用的方法是使用 RowMapper将结果集手动封装成实体类的方法。

```java
@Component
public class DaoTest {

    @Autowired
    private JdbcTemplate template;

    /**
     *查询数据
     */
    public Student findStuById(String stuId){
        Student student= template.queryForObject("select * from student where stu_id= " + stuId, (resultSet, i) -> {
            Student stu = new Student();
            while (resultSet.next()) {
                stu.setName(resultSet.getString("name"));
                stu.setName(resultSet.getString("gender"));
                stu.setAge(resultSet.getInt("age"));
            }
            return stu;
        });
        return student;
    }

    /**
     * 增加一条数据
     */
    public void add(Student student) {
        this.template.update("INSERT INTO student(stu_id,name) VALUES(?,?)",
                student.getStuId(), student.getName());
    }

    /**
     * 更新一条数据
     */
    public void update(Student student) {
        this.template.update("UPDATE student SET name = ? WHERE stu_id = ?",
                student.getName(), student.getStuId());
    }

    /**
     * 删除一条数据
     */
    public void delete(String stuId) {
        this.template.update("DELETE FROM student WHERE id = ?",stuId);
    }
}
```

在这里说一下，如果需要将查询的结果集不止是一条记录，那么使用的是 *queryForList(..)* 方法，而不是使用*queryForMap(..)* 方法，因为该方法只能封装一行的数据，如果封装多行的数据、就会报错！同时个人感觉 spring 提供的 JdbcTemplate 这个数据库工具类，还没有 Apache 提供 dbutils 工具包好用。



spring篇章（五）Dao 模块篇完结。



参考资料：

①：<a href="https://www.cnblogs.com/wmyskxz/p/8845799.html">Spring(5)——Spring 和数据库编程</a>

②：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483965&idx=1&sn=2cd6c1530e3f81ca5ad35335755ed287&chksm=ebd7433cdca0ca2a70cb8419306eb9b3ccaa45b524ddc5ea549bf88cf017d6e5c63c45f62c6e&scene=21###wechat_redirect">Spring【DAO模块】知识要点</a>

