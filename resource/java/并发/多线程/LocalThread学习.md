## LocalThread深入源码学习



### 一、ThreadLocal类介绍

**1.1、概述**

*ThreadLocal* 类设计的目的为了让线程拥有属于自己的变量，保证线程使用的变量都是线程自己的，其他线程无法

访问不是自己的变量（避免线程下变量污染的情况）。简而言之就是：让线程拥有属于自己且只有自己可以访问的

变量。

**1.2、原理**

*Thread* 维护一个 *ThreadLocal.ThreadLocalMap* 的实例map，该map的键值对中的key值是 *ThreadLocal* 的实

例，value是线程的变量。当获取变量时，会先获取当前线程thread，然后从当前线程(thread)中获取维护的map

，然后用ThreadLocal的实例从map获取线程的变量。

**1.3、值得注意**

*ThreadLocal* 的提出，不是为解决并发或者变量共享的问题，只是为让线程用于属于自己的变量。ThreadLocal可

能会造成内存溢出的问题，但内存的溢出问题的原因是：没有手动删除map中的键值对，线程维护的

*ThreadLocalMap* 的生命周期和线程的生命周期是一样的，键值对占用的内存一致存在，而不是弱引用的问题。



### 二、ThreadLocal的深入学习

**2.1 ThreadLocal使用场景**

- ThreadLocal最常用的场景就是管理数据库的连接Connection了，数据库框架 Hibernate对Connection的管

  理就是使用ThreadLocal实现的。

- 开篇的时候介绍ThreadLocal类设计的目的为了让线程拥有属于自己的变量，保证线程使用的变量都是线程自

  己的。所以当线程想拥有自己的变量时，ThreadLocal就要出场了。

代码演示：

```java
@Repository
public class StudentDao{
    
    private static ThreadLocal<Session> local=new ThreadLocal<>();
    @Resource(name = "sessionFactory")
    private SessionFactory sessionFactory;
    
    public static Session getConnection() throws SQLException {
        //从数据源获取Session对象
        Session session=sessionFactory.openSession();; 
        //把Session放进ThreadLocal里面
        local.set(session);
        //返回Session对象
        return session;
     }

    //关闭数据库连接
    public static void removeSession() {
        //从线程中拿到Session对象
        Session session = local.get();
        try {
            if (session != null) {          
                //将该连接归还给连接池
                session.close();
                //移除线程的本地变量
                local.remove();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```



**2.2、深入源码查看实现原理**

- 存放变量 set(T value)

```java
public void set(T value) {
    Thread t = Thread.currentThread();  //获取当前线程
    ThreadLocalMap map = getMap(t); 
    if (map != null)
        map.set(this, value); //以当前ThreadLocal的实例作为key，变量作为value存放
    else
        createMap(t, value); 
}

ThreadLocalMap getMap(Thread t) { //获取线程维护的ThreadLocalMap的实例
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) { //创建ThreadLocalMap并存放变量
    t.threadLocals = new ThreadLocalMap(this, firstValue); 
}

ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY]; //默认初始容量为16
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1); //计算存放的地址
    table[i] = new Entry(firstKey, firstValue); //Entry维护键值对
    size = 1;
    setThreshold(INITIAL_CAPACITY); //设置阈值
}
```

- 获取变量 `T get()`

```java
public T get() {
    Thread t = Thread.currentThread(); //获取当前线程
    ThreadLocalMap map = getMap(t); //获取ThreadLocalMap
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this); //用ThreadLocal作为key获取变量
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result; //返回结果
        }
    }
    return setInitialValue(); //map为null ，则返回null
}
```

- 移除变量 

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this); //用ThreadLocal实例作为key移除变量
}
```



原理总结： 

1. 每个Thread维护着一个ThreadLocal.ThreadLocalMap的引用

2. ThreadLocalMap使用Entry来进行存储键值对（key为ThreadLocal的实例，value为变量）

3. 调用ThreadLocal的set()方法时，实际上就是往ThreadLocalMap存放值，key是ThreadLocal对象，

   值是传递进来的对象

4. 调用ThreadLocal的get()方法时，实际上就是往ThreadLocalMap获取值，key是ThreadLocal对象

5. ThreadLocal本身并不存放储值**，它只是**作为一个key来让线程从ThreadLocalMap获取value。



### 三、内存泄露问题

ThreadLocal的对象关系引用图：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib1a3zqXxNAm6O584NJmiar2AVN56p77CkAyxjvhyaZ27HORCZbFeOc1zHcXI1e7CqJqaYiahbW5LBEw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**图片来源参考资料。**

ThreadLocal造成内存溢出的问题的原因是：线程维护的ThreadLocalMap的生命周期和线程的生命周期是一样长

的，键值对占用的内存一致存在，需要手动删除map中的键值对，如果没有则很有可能会造成内存溢出的问题，

而不是弱引用的问题。因此是要手动remove()删除。



参考资料：<a href="https://www.cnblogs.com/jasongj/p/8079718.html">[Java进阶（七）正确理解Thread Local的原理与适用场景](https://www.cnblogs.com/jasongj/p/8079718.html)</a>

​				   <a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484118&idx=1&sn=da3e4c4cfd0642687c5d7bcef543fe5b&chksm=ebd743d7dca0cac19a82c7b29b5b22c4b902e9e53bd785d066b625b4272af2a6598a0cc0f38e&scene=21###wechat_redirect">ThreadLocal就是这么简单</a> 
