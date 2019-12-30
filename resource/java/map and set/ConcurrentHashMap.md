## ConcurrentHashMap深入源码学习

### 一、ConcurrenHashMap的特点

- jdk1.8底层数据结构是哈希表+链表+红黑树

- 支持线程的高并发操作，即线程安全，部分上锁+CAS(比较交换算法)实现的。

- 查询是不用加锁的，即非阻塞的，因为获取到的数据都是最新的，变量使用volatile关键字修饰

  例如get(Object key)方法获取value。

- key和value都不允许为null

  > 注：jdk1.8的ConcurrentHashMap的底层数据结构，有链表变为红黑树时，也是需要链表的结点超过8
  >
  > 个同时哈希表的大小64时，才会自动转成红黑树。

  

### 二、ConcurrentHashMap的结构

**1、底层数据结构**

![640?wx_fmt=png](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaIPf1sDCN5zcDdGsibZwyzy9rc81tfAsDb0FdjzHkBRu4jXRgLco0aDPXQXOqTiamFL9eOtC0g5RuwYw/640?wx_fmt=png)

图片来源：https://blog.csdn.net/weixin_44460333/article/details/86770169

**2、继承结构图**

![ConcurrentHashMap](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/%E7%BA%A2%E9%BB%91%E6%A0%91.webp)

**3、jdk1.7的ConcurrentHashMap的数据结构**

JDK1.7时的底层数据结构是Segment数组+哈希表+链表。Segment继承ReentrantLock重入锁，每一个Segment

都有一个锁，叫做锁分段。其内部维护一个哈希表，哈希表存放的HashEntry，也就是key和value的实体。在多线

程并发下，如果是要对数据进行修改，则Segment会上锁，是线程能完整修改数据。与其同时其他线程访问其他

的segment中的数据时，则是可以访问到，没有阻塞。因为每一个segment都是独立的，互不相干的。数据结构

如下图所示：

![img](https://upload-images.jianshu.io/upload_images/5220087-8c5b0cc951e61398.png?imageMogr2/auto-orient/strip|imageView2/2/w/767/format/webp)

图片来源于：https://www.jianshu.com/p/865c813f2726。



### 三、CAS比较交换算法和volatile关键字介绍

**3.1、CAS比较交换算法**

CAS（compare and swap）比较交换算法，是一种无锁算法。

该无锁算法需要有三个参数：①内存中的值value，②旧的期望值expectedValue，③待修改的新值newValue。

当且仅当value与expectedValue两个值是一致的时候，才会将内存值value更换成newValue。

> 例如：在高并发的情况下，多个线程同时使用CAS尝试去修改某一个值value，线程先是从内存获取value的
>
> 值，并将value设置成预期值expectedValue(类似快速失败机制)，然后尝试去更改value的值，在更改value的
>
> 值之前，线程会再次去获取内存value的值，将这次获取的value值和其拥有的expectedValue值比较，如果相
>
> 等则更换，反之线程被告知这次竞争失败，可以再次尝试或者啥都不做。最终只有一个线程可以完成value的
>
> 值更新操作。你可能会觉得很奇怪，为啥要获取两次value的值，答案是：因为CPU给线程分配的时间片不一
>
> 致，线程在时间片内可能完成不了更换操作，就被挂起。举例：线程A获取内存中value的值，然后就被挂
>
> 起，这是线程B获取到时间片后，完成value的更新。然后线程A再次获得时间片，尝试去修改value的值，却
>
> 发现value早已被修改，被告知竞争势失败。

**3.2、Volatile关键字介绍**

Volatile：易变的;无定性的;无常性的；

volatile经典总结：**volatile仅仅用来保证该变量对所有线程的可见性，但不保证原子性**。

- 保证可见性

  在多线程的情况下，被改关键字修饰的变量，其在任意时间被线程修改后的值，对所有线程都是透明的，线程获取变量的值，都是获取到最新的值。

- 不保证原子性

  修改一个变量，需要分三步走，①取值，②修改值，③赋值给变量；在并发的情况下，假如修改变量的方法没

  有上锁，当前线程被挂起，其他线程获得时间片后，也来修改该值，原子性被破坏，线程修改后的值与预期值

  不一致。又或者：有一变量值count为10，线程A，B同时获取count的值为10。线程A对count进行加1，

  count的值变为11。线程B也对count加1，count的值变为11。两次加1操作，结果本应该为12的，可结果却

  为11。这就是volatile不保证原子性的原因。

> volatile可以保证可见性的原因是：volatile关键字让JVM在内存设置了内存屏障，也就是当内存中的值被修改
>
> 后，将被修改后的值，强制写入到磁盘，刷新内存(将其值清除）。当有获取变量值的请求时，在内存中查询
>
> 不到，就要到磁盘中读取，同时放入到内存中，所以保证了获取到数据都是最新的。



### 四、ConcurrentHashMap的源码介绍

注：ConcurrentHashMap是Jdk容器中最复杂和最难看懂的类，需要足够的知识量才能看懂。。。。。。

虽然下面的链接也没有讲清楚，但可供参考。

https://www.jianshu.com/p/865c813f2726

https://blog.csdn.net/weixin_44460333/article/details/86770169

https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484161&idx=1&sn=6f52fb1f714f3ffd2f96a5ee4ebab146&chksm=ebd74200dca0cb16288db11f566cb53cafc580e08fe1c570e0200058e78676f527c014ffef41&scene=21###wechat_redirect
