## mysql（四）ACID

#### 1、前言

学过数据库的都知道事务的四大特性：ACID 吧，那 mysql 是如何实现这四大特性的呢？

此篇章讲述 mysql 是如何实现事务的四大特性。

#### 2、事务的四大特性 ACID

* **Atomicity** （原子性）

  是指一个事务是不可分割的单位，事务里的操作，全部执行，要么不执行。也就是一个事务不能执行一半而停止。比如说：转钱，账户A转钱给账户 B，账户 A 转出了钱，账户 B 必须要收到正确的金额。不能账户A转出了钱而账户B没有收到钱。

* **Consistency** （一致性）

  是指在事务执行前数据库处于正确的状态，那事务执行完后，数据库也要处于正确的状态，即数据完整约束性不能被破坏。还是转账例子：在开始转钱前，账户A和账户B有明确的总额，那账户A转出钱后，账户B需要收到正确的金额数，即转完账后，账户A和账户B的总额仍不变。不能出现账户A转出钱而账户B没有到钱，两个账户的总额发生变化。

* **Isolation** （隔离性）

  是指在并发事务在执行之间互不影响，在一个事务内部的操作对其他事务是不产生影响的，这需要事务的隔离级别来制定隔离性。

* **Durability** （持久性）

  是指事务一旦执行成功，数据库里的数据的改变必须是永久的，要被记录到磁盘，不能因为故障或意外而造成数据丢失或不一致。

#### 3、mysql 如何实现事务的特性

> 以下内容来源于：<a href="https://mp.weixin.qq.com/s?__biz=MzIwMDgzMjc3NA==&mid=2247484533&idx=1&sn=c15b19bff49b46b6af31dca1646c524a&chksm=96f6661ca181ef0a5061e9c2c0bd5912733bfa733f028f357e997ea4d963431dca9eeefd54c5&mpshare=1&scene=23&srcid=0221GioIvPLjqczUm2RNKwur&sharer_sharetime=1582265947845&sharer_shareid=a8ee705dc28d6aaab271b797da5bc9c5#rd">程序员，知道Mysql中事务ACID的原理吗?</a>

* 原子性

  利用 Innodb 引擎中的回滚日志 *undo log*，当事务回滚时能够撤销所有已经成功执行的sql语句，其会记录数据库执行 sql 前的数据，当事务失败或者调用 *rollback*，需要事务回滚时，就根据 *undo log* 中的记录，进行数据库数据复原操作。

  - (1)当 delete 一条数据的时候，就需要记录这条数据的信息，回滚的时候，insert这条旧数据
  - (2)当 update 一条数据的时候，就需要记录之前的旧值，回滚的时候，根据旧值执行update操作
  - (3)当 insert 一条数据的时候，就需要这条记录的主键，回滚的时候，根据主键执行delete操作

* 一致性

  这需要从两个层面来说：数据库层面和应用层方面。

  ① 数据库层面：通过事务的原子性、隔离性、持久性来保持事务的一致性。也就是说：一致性是事务的目的，而原子性、隔离性和持久性，是为实现一致性，数据库所使用的手段。数据库必须要实现 AID 三大特性，才有可能实现一致性。例如，原子性无法保证，显然一致性也无法保证。

  ② 应用层面：通过代码判断数据库数据是否有效，然后决定回滚还是提交数据！

* 隔离性

  使用利用的是锁和MVCC机制。

  表名`t_balance`

  ![img](http://mmbiz.qpic.cn/mmbiz_png/SYoYmIOcI5oc7c7gqDQh2yS1Wv3DWvQuk8zL2drcNJmibDibOjo43U58RKicfYkuZ4Ekcot5vrxL6RCE08XmhJ6Xg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=0)

  其中id是主键，user_id为账户名，balance为余额。还是以转账两次为例，如下图所示

  ![img](http://mmbiz.qpic.cn/mmbiz_png/SYoYmIOcI5oc7c7gqDQh2yS1Wv3DWvQuKmj8zWxibW6iaiciciaenwxkHLRLHmibJtfO1HBA3b2Y5lMfAn9YCqWAd8YQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=0)至于 MVCC,即多版本并发控制 (Multi Version Concurrency Control) ,一个行记录数据有多个版本的快照数据，这些快照数据存放在 *undo log* 回滚日志中。

  如果一个事务读取的行正在做 DELELE 或者 UPDATE 操作，读取操作不会等行上的锁释放，而是读取该行的快照版本。

  MVCC 机制在可重复读 (Repeateable Read) 和读已提交 (Read Commited) 的表现形式不同，就不赘述了。但是有一点是说明一下的：在事务隔离级别为读已提交 (Read Commited) 时，一个事务能够读到另一个事务已经提交的数据，是不满足隔离性的。但是当事务隔离级别为可重复读 (Repeateable Read) 中，是满足隔离性的。

* 持久性

  是利用 Innodb 的 *redo log*。正如之前说的，mysql 是先把磁盘上的数据加载到内存中，在内存中对数据进行修改，再刷回磁盘上。如果此时突然宕机，内存中的数据就会丢失，会将 mysql 的操作，记录到 *redo log* 日志中（该日志是要落盘的），宕机重启后，然后将数据 load 内存，根据 *redo log* 中的记录，更新内存的数据然后待 mysql 空闲时，在将内存的数据刷到磁盘中。

  

完结。