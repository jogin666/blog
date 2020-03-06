## mysql（二）数据的持久性

#### 1、前言

上一篇写 mysql 的基本知识，这个篇章是我在浏览博客时，看到比较好的一篇文章，遂而转载一下。

原文地址：<a href="https://mp.weixin.qq.com/s?__biz=MzUzMjczMzY5MQ==&mid=2247484044&idx=1&sn=4f8ac35fc67354cdf84ff317a6a4df2a&chksm=faaf8918cdd8000e8c6e4c4abc889f09f74988004b0a77c07718e4e9c40dc35dc36e4b6168c0&mpshare=1&scene=23&srcid=0220SuQRKu12Iz5LFzAdxF0z&sharer_sharetime=1582180978381&sharer_shareid=a8ee705dc28d6aaab271b797da5bc9c5#rd">MySQL 是如何实现 ACID 中的 D 的？</a>

#### 2、博客原文

假如在 mysql 数据库上执行一条 sql 语句：

```sql
update user set age = 18 where user_id = 345981
```

mysql 会直接去磁盘修改数据吗？ 明显不会，磁盘 IO 太慢了，如果每个请求过来，mysql 都要写磁盘，那磁盘肯定是扛不住的。那写内存？把数据从磁盘 load 到内存，然后修改内存里的数据。这样也不行，万一 mysql 宕机了那内存就没有了，数据就再也找不回来了。

对数据的修改到底是写磁盘，还是写内存？这是很多中间件都会遇到的问题，一个中间件怎么分布式，怎么搞可用，都会到这个问题 。写磁盘，嫌太慢？写内存，又不安全？

那 mysql 到底是怎么解决这个问题的呢？

mysql 的解决方案是：**既写磁盘又写内存。**数据写内存，另外再往磁盘写 redo log。



**redo log 是啥**

在执行上面的 sql 语句时，mysql 会判断内存中有没有 user_id = 345981 的数据，没有，则去磁盘找到这条数据所在的「页」，把整页数据都加载到内存，然后找到 user_id = 345981 的 row 数据，把内存中这行数据的 age 设置为 18。

执行完，这时的内存数据时新的，正确的，而磁盘的数据时旧的，已经过时了，所以我们将这使的磁盘对应的也数据，称之为「脏页」。

这里补充一个知识点：**mySQL 是按页为单位来读取数据的**，一个页里面有很多行记录，从内存刷数据到磁盘，也是以页为单位来刷。

为了防止宕机，数据就丢失，于是 mysql 会**把执行 sql 后，对页中的内存就行了什么修改**，记录了下来，保存到磁盘，也就是 redo log。写完 redo log 之后，mysql 就会认为事务提交成功，数据已经被持久化了（ACID 中的 D ），然后在 mysql 空闲的时候，在内存的数据刷到次磁盘。

如果在内存数据刷到磁盘之前，mysql 宕机了，怎么办？ 这时，在 mysql 重启后，会自动把「脏页」的数据，load 到内存中，然后根据 redo log 日志上的记录，修改内存中的数据，再刷磁盘就好了，脏页就变成「干净页」了。

如果你是扛精，可能会说，万一写内存成功，但是把 redo log 写到磁盘失败了呢？对于这个问题，mysql 已将给出了解决方案，这点后面在讨论「两阶段提交」时再讨论。

你也许会觉得，redo log 还是要写磁盘，那不还是很慢？并不是，把 redo log 写到磁盘，比一般的写磁盘要快，原因有：

- 一般写磁在盘，都是「随机写」，而 redo log，是「顺序写」
- mysql 在写 redo log 上做了优化，比如「组提交」

这些后面我们会陆续展开。



**redo log 怎么存储**

mysql 官方文档有几句话：

> *the redo log encodes requests to **change table data** that result from SQL statements or low-level API calls.*

> *The redo log is a **disk-based data** structure **used during crash recovery to correct data**written by incomplete transactions.*

> *By default, the redo log is physically represented on disk by **two files** named `ib_logfile0`and `ib_logfile1`.*

> *MySQL writes to the redo log files in a **circular** fashion.*

从这几句话，我们大致可以get到：

- redo log 记录了 sql 语句以及其他 api，对表数据产生的变化，也就是说，redo log 记录的是数据的 **物理变化**，这也是它和后面要讲的 binlog 一个最大的区别，binlog 记录的是数据的 **逻辑变化**，这也是为什么 redo log 可以用来 crash recovery，而 binlog 不能的原因之一。
- redo log 是存储在磁盘的，用于 crash recovery 后修正数据的，也就是我们常说的故障恢复，比如掉电，宕机等等
- redo log 默认有两个文件
- redo log 是循环写的（circular）

根据最后两个小点，我们大致可以画出这样一个图：

![redo log](https://github.com/jogin666/blog/blob/master/resource/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9F%A5%E8%AF%86/mysql/images/redo_log.png)

另外还有两个参数：

- *innodb_log_file_size* : 设置每个 redo log 文件的大小，默认是 50331648 byte ，也就是 48 m
- *innodb_log_files_in_group* : 设置每个 redo log 文件的数量，默认是 2，最大值是 100

常说事务具有 ACID 四个特性，其中 D（durability），数据持久性，意味着，**一旦事务提交，它的状态就必须保持提交，不能回滚**，哪怕你系统宕机了、奔溃了，你也要想办法把事务做到提交，把数据给我保存进去：

> *Durability guarantees that once a transaction has been committed, it will **remain committed** even in the case of a system failure (e.g., power outage or crash).*

这确实很严格，但看起来也很好实现，就是每次都把数据写到磁盘，然后再告诉客户端，事务提交成功了。

但如果要追求高性能呢？把数据写到内存呢？**所以说，innodb 在实现高性能写数据的同时，利用 redo log，实现了事务的持久性。**



**binlog**

讲完了 redo log，我们再来聊聊 binlog。还是这一条 update 语句：

```sql
update user set age = 18 where user_id = 345981
```

在这条 update 语句执行的时候，除了生成 redo log，还会生成 binlog。binlog 和 redo log 有很多不同点，有一点是一定要知道的：就是 redo log 知识 innodb 存储引擎的功能，而 binlog 是 mysql  server 层的功能，也就是说：redo log 只在使用了 innodb 引擎的 mysql 上才有，而 binlog 是 mysql 有的，无论使用了什么引擎。

![innodb 引擎](https://github.com/jogin666/blog/blob/master/resource/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9F%A5%E8%AF%86/mysql/images/innodb%E5%BC%95%E6%93%8E.png)

那 binlog 有什么作用呢？

mysql 之所以把 binlog 放在了 server 层，说明 binlog 提供了一些通用的能力，比如：**数据还原**。DBA 总说，他能把 MySQL 的数据还原到任意时刻，怎么还原？假设你在周三晚上八点的时候，不小心把一张表的数据都清空了，怎么办？这时候 DBA 就会找到最近的一次「全量备份」，然后重放一次最近的全量备份，到周三的晚上八点这段时间的 binlog，于是你的数据就还原回来了。

binlog 还有另一个作用：**主从复制**，主库把 binlog 发给从库，从库把 binlog 保存了下来，然后去执行它，这样就实现了主从同步。当然，我们还能让自己的业务应用，去监听主库的 binlog，当数据库的数据发生变动时，去做特定的事情，比如进行数据实时统计。



**两段提交**

最后，那么当我执行一条 update 语句时，redo log 和 binlog 是在什么时候被写入的呢？这就有了常说的「两阶段提交」：

- 写入：redo log (prepare)
- 写入：binlog
- 写入：redo log（commit）



为什么 redo log 要分两个阶段： prepare 和 commit ？redo log 就不能一次写入吗？我们分两种情况讨论：

- 先写 redo log，再写 binlog
- 先写 binlog，再写 redo log



1、先写 redo 再写 binlog

这样就会出现 redo log 写入到磁盘了，但是 binlog 还没有写入到磁盘，，于是当发生 crash recovery 时，恢复后，主库可以使用 redo 恢复数据后，但是由于没有写入到 binlog，从库是不能同步数据的，主库比从库“新”，造成主从不一致。

2、先写 binlog，再写 redo log

跟上一种情况类似，很容易知道，这样会反过来，造成从库比主库“新”，也会造成主从不一致

而两阶段提交，就解决这个问题，crash recovery 时：

- 如果 redo log 已经 commit，那毫不犹豫的，把事务提交
- 如果 redo log 处于 prepare，则去判断事务对应的 binlog 是不是完整的
  - 是，则把事务提交
  - 否，则事务回滚

两阶段提交，其实是为了保证 redo log 和 binlog 的逻辑一致性。



**未完待续**

总结一下 ：

- redo log：innodb 在实现高性能写数据的同时，利用  redo log，实现了事务 ACID 中的 D：持久性
- binlog：mysql 的数据还原、主从复制，都依赖 binlog 来实现
- 两阶段提交：为了保证 redo log 和 binlog 的一致性

看似一条简单的 update 语句，MySQL 在这背后其实做了很多事情。

MySQL 是一个把单机性能发挥到极致的数据库，这也是为什么出现了那么多分布式数据库，MySQL 依然是很多公司的首选的原因吧。

当然这篇文章也只是个引子，很多细节，还没有展开。

比如看完两阶段提交，你们可能还有疑问，为什么不能先写 redo log，再写 binlog，如果发生 crash，重启后判断 binlog 是不是完整的，不完整，则回滚事务不就好了？

其实两阶段提交是经典的分布式系统问题，很多分布式系统也在用，包括上面讲的两阶段提交也只是一个粗略的提交过程，拆分的再细一点，应该是这样：

![两段提交](https://github.com/jogin666/blog/blob/master/resource/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9F%A5%E8%AF%86/mysql/images/%E4%B8%A4%E6%AE%B5%E6%8F%90%E4%BA%A4.png)

我们后面可以再深入研究下。

还有，每次事务提交，redo log 都是要写磁盘的，MySQL 怎么优化 redo log 的写入？

组提交、LSN 是什么？

最后再贴一张 innodb 架构图：

![img](https://github.com/jogin666/blog/blob/master/resource/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9F%A5%E8%AF%86/mysql/images/innodb%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

今天其实只讲了这张图的右边的 redo log，其他的像 change buffer、double write buffer、undo log、log buffer 等等，都是些什么？

你会发现，OMG，MySQL 怎么这么复杂？

大概是，越是看起来运转顺畅的系统，背后越是有复杂的机制来支撑吧。

别看人家看起来很轻松，其实人家背后很努力的。
