## 深入了解Spring事物

写在前面：原文地址  <a href="https://blog.csdn.net/mawenshu316143866/article/details/81281443 ">深入理解Spring事务的基本原理、传播属性、隔离级别</a>



### 一、事物的基本要素（ACID）

事务是指多个基本操作单元组成的合集，其中多个基本操作单元是整体不可分割的。一个事无要么成功，要么失

败，别无选择。事务必须遵守四个原则(ACID)：

* 原子性(Atomicity)：是指一个事务要么全部执行，要么不执行。也就是一个事务不能执行一半而停止。比如说

  转钱，账户A转钱给账户B，账户A转出了钱，账户B必须要收到正确的金额。不能账户A转出了钱，而账户B没

  有收到钱。

* 一致性(Consistency)：在事务执行前数据库处于正确的状态，那事务执行完后，数据库也要处于正确的状态

  ，即数据完整约束性不能被破坏。还是转账例子：在开始转钱前，账户A和账户B有明确的总额，那账户A转出

  钱后，账户B需要收到正确的金额数，即转完账后，账户A和账户B的总额仍不变。不能出现账户A转出钱而账

  户B没有到钱，两个账户的总额发生变化。

* 隔离性(Isolation)：并发事务执行之间互不影响，在一个事务内部的操作对其他事物是不产生影响的，这需要

  事务的隔离级别来制定隔离性。

* 持久性(Durability)：事务一旦执行成功，数据库里的数据的改变必须是永久的，要被记录到磁盘，不能因为故

  障或意外而造成数据丢失或不一致。

### 二、事务的分类

1. 数据库事务分为本地事务和全局事务
   * 本地事务：普通事务，独立一个数据库，能保证在该数据库上所有操作的ACID。
   
   * 分布式事务：涉及多个数据库源的事务，即跨越多台同类数据库和异类数据库的事务（由每台数据库的本
   
     地事务组成），分布式事务的目的保证这些本地事务的所有操作的ACID，让事务可以跨越多台数据库。
2. Java事务类型分为 JDBC 和 JTA 事务
   * JDBC 事务：即上面所说的数据库上的本地事务，通过Connection实例对象操作管理
   
   * JTA 事务：JTA(Java Tansaction API)是Java事务的API，是JavaEE数据库事务规范，JTA只提供数据库事务
   
     管理的接口，具体的实现有数据库厂商提供，JTA事务比JDBC更加强大，支持分布式事务。
3. 声明式事务和编程式事务，参考：<a href="https://blog.csdn.net/liaohaojian/article/details/70139151">Spring事务管理实现方式之编程式事务与声明式事务详解</a>
   * 声明式事务：通过xml文件配置或者注解实现
   * 编程式事务：在业务逻辑的代码中手动自行实现，粒度更小。

### 三、事务的基本原理

Spring事务的本质就是数据库的对事务的支持。数据库厂商实现的事务API，然后spring使用，如果没有数据库对

事务的支持，Spring也是无法提供事务的功能。对于纯JDBC形式操作数据库，想要用到事务，可以按照以下步骤

进行：

```java
/*
1. 加载数据库驱动（厂商提供）：Class.forName("......")
2. 获取数据库连接：Connection con=DriverManager.getConnection()
3. 执行CRUD操作
4. 提交事务 con.commit()/回滚事务con.rollback();
5. 关闭连接 con.close()
*/
```

使用Spring的事物管理功能，开发人员不在需要步骤2和步骤4，Spring会自动完成步骤2和4的功能。要理解

Spring自动完成开启事务和关闭事务的功能，需要整体理解Spring的事物管理实现的原理。下面开始简单的介绍

下，以注解方式为例子：

1. 配置文件开启注解驱动，在dao类上或者方法上通过@Transactional标识，启动。

2. Spring启动后，在将类扫面注入容器时，会解释类上的注解和方法上的注解，并为有注解标识的类或方法生成

   代理，如果是@Transaction，则根据该注解的里的相关参数进行相关配置注入，在业务逻辑进行CRUD之前和

   之后，Spring生成的代理会自动开启，提交和关闭事务，发生异常则回滚事务。

3. 真正的数据库层的事务提交和回滚是通过bin log和redo log实现的。

### 四、数据库的隔离级别

| 隔离级别                   | 隔离级别的代码 | 脏读 | 不可重复读 | 幻读 |
| -------------------------- | -------------- | ---- | ---------- | ---- |
| 读未移交(read-uncommitted) | 0              | 是   | 是         | 是   |
| (read-committed)           | 1              | 是   | 是         | 否   |
| 可重复读(repeatable-read)  | 2              | 是   | 否         | 否   |
| 串行化(serializable)       | 3              | 否   | 否         | 否   |

脏读：一个事务对数据进行了修改，但未提交，另一事务可以读取到修改但未移交的数据（删除，更新数据）。如

果第一个事务进行事物提交，则第二个事务读取到的数据不合法，为脏数据。

不可重复读：在一个事物内对某一个数据进行读取两次，但在两次的间隔内，另一个事务对数据进行了修改，数据

库数据发生了变化，导致前事务两次读取的数据不一致。即不保证一个事务内多次读取到同一数据是一致的。

幻读：一个事务内两次读取数据，读取到的数据量不一致，即读取到的数据量或多了，或少了（在事物两次读取数

据的间隔内，另一个事物对数据进行了增删）。

总结：不可重复读侧重于修改，表内锁住符合条件的数据就可以解决；幻读侧重于增删，解决需要锁住整张表。

**上述内容总结：**

隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大。

大多数的数据库默认隔离级别为 Read Commited，比如 SqlServer、Oracle

少数数据库默认隔离级别为：Repeatable Read 比如： MySQL InnoDB

补充：

* **数据丢失新：**两个事务同时更新同一数据时，最后一个事务更新会覆盖掉第一个事务的更新，从而造成第一个

  事务的更新没有记录（或者说是失败），这是由于没有加锁造成的。

* SQL规范所规定的标准，不同的数据库具体的实现可能会有些差异

* mysql中默认事务隔离级别是可重复读时并不会锁住读取到的行

* 事务隔离级别为读提交时，写数据只会锁住相应的行

* 事务隔离级别为可重复读时，如果有索引（包括主键索引）的时候，以索引列为条件更新数据，会存在间隙锁

  间隙锁、行锁、下一键锁的问题，从而锁住一些行；如果没有索引，更新数据时会锁住整张表。

* 事务隔离级别为串行化时，读写数据都会锁住整张表

* 隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大，鱼和熊掌不可兼得啊。对于

  多数应用程序，可以优先考虑把数据库系统的隔离级别设为Read Committed，它能够避免脏读取，而且具有
  
  较好的并发性能。尽管它会导致不可重复读、幻读这些并发问题，在可能出现这类问题的个别场合，可以由应
  
  用程序采用悲观锁或乐观锁来控制。
  
  

### 五、Spring的隔离级别

| 常量                                 | 解释                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| ISOLATION_DEFAULT(默认)              | 这是`PlatfromTransactionManager `默认的隔离级别，使用数据库默认的事务隔离级别。 |
| ISOLATION_READ_UNCOMMITTED(读未提交) | 事务最低级别，允许一个事物读取另一个事务修改但未提交的数据，会发生脏读，不可重复读和幻读的级别错误。 |
| ISOLATION_READ_COMMITTED(读提交)     | 该级别保证一个事务无法读取令一个事务修改但未提交的数据，只有事务提交后才能读取。 |
| ISOLATION_REPEATABLE_READ(可重复读)  | 该级别可以防止脏读，重复读，但可能出现幻读的情况。           |
| ISOLATION_SERIALIZABLE(串行化)       | 花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。 |

1. ISOLATION_DEFAULT：使用底层数据库的默认隔离级别，数据库管理人员设置是什么就是什么。

2. ISOLATION_READ_UNCOMMITTED(未提交读)：Spring最低隔离级别，事务为提交，其他事物可以读取到该

   事务为提交的修改的数据。(发生脏读，重复读，幻读)。

3. ISOLATION_READ_COMMITTED(提交读)：不会发生脏读，但可能发生重复读，幻读。sql server默认级别。

4. ISOLATION_REPEATABLE_READ（可重复读）：保证一个事物内多次读取同一数据，取到的内容是一致的，

   不发生 脏读和不可从复读，有可能发生幻读。

5. ISOLATION_SERIALIZABLE(串行化)：代价最高的隔离界别，但防止脏读，不可重复读和幻读。

   

### 六、Spring事务的传播

1. Spring事务传播类型介绍

| 常量                      | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Propagation.REQUIRED      | 支持当前事务，若当前不存在事务，则创建事务。这是Spring默认的事务的传播，也是最常见的选择。 |
| Propagation.REQUIRES_NEW  | 新建食物，如果当前存在事务，则把当前事务挂起。新建的事物和被挂起的事物没有任何关系，是两个独立的事物，外层事务失败回滚，不能回滚内层事务执行的结果。内层失败回滚则抛出异常，外层事物捕获，也可以不做回滚操作。 |
| Propagation.SUPPORTS      | 支持当前事务，如果当前不存在事务，则已非事务的方式执行       |
| Propagation.MANDATORY     | 支持当前事务，如果当前不存在事务，就抛出异常                 |
| Propagation.NOT_SUPPORTED | 以非事务的当时执行，如果当前存在事务，则事务被挂起           |
| Propagation.NEVER         | 以非事务的方式执行，若果当前存在事务，则抛出异常             |
| Propagation.NESTED        | 如果一个活动的事务存在，则运行在一个嵌套的事务中。如果没有活动事务，则按REQUIRED属性执行。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对`DataSourceTransactionManager`事务管理器起效。 |

2. Spring事务传播代码演示

```java
public class StudentDaoImpl{
    @Transactional
	public void serviceA(){
        serviceB();
    }
    @Transactional
    public void serviceB(){
        //.....
    }
}
/*Propagation.REQUIRED（spring 默认的事务传播行为）
 如上述代码，serviceA()方法上已经定义了事务的传播为：PROPAGATION_REQUIRED，在serviceA调用
 serviceB方法()时，spring判定serviceB()方法已经运行在serviceA()方法的事务内，则不会再起新的事务，
 把serviceB()加入到serviceA()起的事务内。反之如果serviceA()没有声明有事务，则serviceB()方法会创
 建一个事务。这时如果methodA()或者在methodB()内的任何地方出现异常，该事务都会被回滚。
*/
```

```java
public class StudentDaoImpl{
    @Transactional
	public void serviceA(){
       try{
            serviceB();
        }catch (Exception e){
            //.....
        }
    }
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void serviceB(){
        //......
    }
}
/*Propagation.REQUIRES_NEW
 serviceA()方法上已经定义了spring默认的事务传播方式，而serviceB()则定义的事务为：
 Propagation.REQUIRES_NEW，在serviceA调用serviceB方法()时，serviceB()把会serviceA()起的事务挂起
 另起一个新的事务，待serviceB()起的事务完成以后，serviceA()的事务才能继续执行。
 这时因为serviceA()和serviceB()分别起的两个事务是相互独立的，也就是说serviceA()失败回滚，是不能回 
 回滚serviceB() 所提交的事务，而serviceB()失败回滚抛出异常，serviceA()捕获后，也可以不做回滚操作。
*/
```

```java
public class StudentDaoImpl{
    @Transactional(propagation = Propagation.SUPPORTS)
    public void serviceA(){
        serviceB();
    }
    public void serviceB(){
        //.......
    }
}
/*Propagation.SUPPORTS
 serviceA()方法上已经定义了事务的传播为：Propagation.SUPPORTS，	执行serviceB()方法时，spring检测到 
 serviceA()方法已经起了事务，则会把serviceB()方法加入到事务内。反之如果serviceA()没有起事务， 
 serviceB()方法会按照非事务方式执行。
*/
```

```java
public class Main {
    @Transactional
    public void serviceA(){
        serviceB();
    }
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void serviceB(){
        //.......
    }
}
/*Propagation.NOT_SUPPORTED
 serviceA()方法上已经定义了spring默认的事务传播，执行serviceB()时，因为serviceB()的事务传播
 定义为：Propagation.NOT_SUPPORTED，所以serviceA()起的事务会被挂起，待serviceB()以非事务的方式
 执行完后，在以事务的方式执行serviceA()。
*/
```

```java
public class Main {

    public void serviceA(){
        serviceB();
    }
    @Transactional(propagation = Propagation.MANDATORY)
    public void serviceB(){
        //.......
    }
}
/*Propagation.MANDATORY
  serviceA()方法上没有定义事务传播，而seriveB()上定义了Propagation.MANDATORY，那么在调用到 
  serviceB()时，spring检测到serviceA()没有定义事务，则会抛出异常，反之serviceA()定义了事务，则不会。
*/
```

```java
public class Main {
    public void serviceA(){
        serviceB();
    }
    @Transactional(propagation = Propagation.NEVER)
    public void serviceB(){
        //.......
    }
}
/*Propagation.NEVER
 serviceA()方法上已经定义了spring默认的事务传播，执行serviceB()时，因为serviceB()的事务传播
 定义为：Propagation.NEVER，spring检测seriveA()已经起了事务，则会抛出异常，反之，则不会抛出异常，以
 非事务的方式执行serviceB();
*/
```

```java
public class Main {
	@Transactional
    public void serviceA(){
       try{
            serviceB();
       }catch(Exception e){
           //....
       }
    }
    @Transactional(propagation = Propagation.NESTED)
    public void serviceB(){
        //.......
    }
}
/*Propagation.NESTED
 serviceA()方法上已经定义了spring默认的事务传播，serviceB()定义了事务的传播为：Propagation.NESTED
 ，那么在执行到serviceB()时，spring检测到serviceA()起了事务，则会把servicB()纳入serviceA()的事务
 中，反之serviceA()没有起事务，则serviceB()会起一个事物。
 值得注意的是：当serviceB()嵌套在serviceA()起的事务里时，serviceB()如果发生错误，则serviceB()里执行
 的所有操作都会被回滚到起点。而serviceA()会捕获serviceB()的异常，再决定是否也要回滚操作。
*/
```

### 七、总结

对于项目中需要使用到事务的地方，我建议开发者还是使用spring的TransactionCallback接口来实现事务，不要

盲目使用spring事务注解，如果一定要使用注解，那么一定要对spring事务的传播机制和隔离级别有个详细的了

解，否则很可能发生意想不到的效果。

![img](https://img-blog.csdn.net/20180202232828622?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRHVzdGluX0NEUw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)根据底层所使用的不同的持久化 API 或框架，使用如下：

- **DataSourceTransactionManager**：适用于使用JDBC和MyBatis进行数据持久化操作的情况，在定义时需要

  提供底层的数据源作为其属性，也就是 **DataSource**。

- **HibernateTransactionManager**：适用于使用Hibernate进行数据持久化操作的情况，与 

  HibernateTransactionManager 对应的是 **SessionFactory**。

- **JpaTransactionManager**：适用于使用JPA进行数据持久化操作的情况，与 JpaTransactionManager 对应的

  是 **EntityManagerFactory**。