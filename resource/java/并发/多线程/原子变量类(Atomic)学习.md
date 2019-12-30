## 原子类(Atomic）知识整体学习



### 一、预备知识

**1.1 悲观锁和乐观锁**

- 悲观锁和乐观锁不是特指某种锁，而是在多线程并发的情况下为保证数据的安全性所采取的的策略。悲观锁策

  略总是假设当前处于最坏的情况，如果不上锁数据的完整性就会被破坏。而乐观锁策略是基于一种冲突检查的

  方法，检测到冲突是就会告诉线程操作失败。

- Synchronized就是一种常见的悲观锁。CAS机制（比较交换）就是一种常见的乐观锁。

- 如果对于Synchronized还不太了解的话，传送门:<a href="">Synchronized详解</a>



**1.2 CAS知识回顾**

CAS(Compare and swap)比较交换，是原子操作的一种，可用于多线程编程中实现以检测不冲突的则更新数据的

操作。CAS需要三个参数：①内存中的值value，②旧的期望值expectedValue，③待修改的新值newValue。

当且仅当value与expectedValue两个值是一致的时候，才会将内存值value更换成newValue。

> 例如：在高并发的情况下，多个线程同时使用CAS尝试去修改某一个value值，线程先是从内存获取value的
>
> 值，并将value设置成预期值expectedValue(类似快速失败机制)，然后尝试去更改value的值，在更改value的
>
> 值之前，线程会再次去获取内存value的值，将这次获取的value值和其拥有的expectedValue值比较，如果相
>
> 等则更换，反之线程被告知这次竞争失败(**可以再次尝试**或者啥都不做）。最终只有一个线程可以完成value值
>
> 的更新操作。你可能会觉得很奇怪，为啥要获取两次value的值？答案是：因为CPU给线程分配的时间片不一
>
> 致，线程在时间片内可能完成不了更换操作，就被挂起。举例：线程A获取内存中value的值，然后就被挂起
>
> ，这时线程B获取到时间片了，然后完成value的更新。然后线程A再次获得时间片，尝试去修改value的值，
>
> 却发现value早已被修改，被告知竞争势失败。

代码演示（简单示意）

```java
pullic class AtomicStudy{
    int value=0;    
    public static void main(String args[]){
        AtomicStudy atomic = new AtomicStudy();
        CyclicBarrier barrier = new CyclicBarrier(5);
        for (int i=0;i<5;i++){
            new Thread(()->{
                try {
                    int expectedValue=atomic.getValue();
                    barrier.await(); //拦截线程，然后统一放行，高度模拟同时竞争
                    atomic.updateValue(expectedValue,6);
                } catch (InterruptedException|BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },i+"").start();
        }
    }

    public int getValue(){
        return this.value;
    }

    public void updateValue(int expectedValue,int newValue){
        int oldValue=value;
        String name = Thread.currentThread().getName();
        if (oldValue==expectedValue){
            value=newValue;
            System.out.println("线程"+name+"竞争成功"+" value:"+value);
        }else{
            System.out.println("线程"+name+"竞争失败");
        }
    }
    /*輸出結果
    线程1竞争失败
    线程3竞争失败
    线程4竞争失败
    线程0竞争失败
    线程2竞争成功 value:6
    */
}
```

值得注意的是：当线程被告知失败的时候，线程可以失败重试。如何失败重试呢？那就是当线程被告知失败了之

后，再次获取value的内存值，设置新的期待值，再次发起重试。

代码演示（简单示意）

```java
//把主体代码修改一下，其他保持不变
public static void main(String args[]){
    AtomicClassStudy atomic = new AtomicClassStudy();
    CyclicBarrier barrier = new CyclicBarrier(2);
    for (int i=0;i<2;i++){
        new Thread(()->{
            try {
                int expectedValue=atomic.getValue();
                barrier.await(); //拦截线程，然后统一放行，高度模拟同时竞争
                boolean success = atomic.updateValue(expectedValue, 6);
                while (!success){
                    success=atomic.updateValue(atomic.getValue(),66);
                }
            } catch (InterruptedException|BrokenBarrierException e) {
                e.printStackTrace();
            }
        },i+"").start();
    }
}
public boolean updateValue(int expectedValue,int newValue){
    int oldValue=value;
    String name = Thread.currentThread().getName();
    if (oldValue==expectedValue){
        value=newValue;
        System.out.println("线程"+name+"竞争成功"+" value:"+value);
        return true;
    }else{
        System.out.println("线程"+name+"竞争失败");
    }
    return false;
}
/*输出结果
线程1竞争成功 value:6
线程0竞争失败
线程0竞争成功 value:66
*/
```

CAS的核心就是：原子性操作，上面的代码虽然是先比较，再修改(Compare and Swap)的两个步骤，但是原子性

的。因为原子性操作**并不定义为一步到位，如果一系列的操作是连续的，不能被中断的操作，就能称为原子性操作**



### 二、原子变量类介绍

有第一部分内容的铺垫，相信大家都知道了什么是乐观锁和悲观锁，以及知道它们的大致是怎么实现的了。现在回

到原子变量类介绍。

**2.1、 提出问题**

在此之前还是先来个例子铺垫。还是多线程安全——共享变量自增的问题。

```java
public class AtomicClassStudy {
    public static void main(String args[]) throws InterruptedException {
        ExecutorService service = Executors.newFixedThreadPool(20);
        Counter counter=new Counter();
        for (int i=0;i<10000;i++){
            service.execute(()->counter.increase());
        }
        //等待线程所有线程执行
        service.shutdown();
        service.awaitTermination(1, TimeUnit.SECONDS);  
        System.out.println("count->"+counter.getCount());
    }
    
    static class Counter{
        int count=0;   
     	//volatile int count=0; //保证了变量的可见性，不保证变量的原子性
        public int getCount() {
            return count;
        }

        public void increase(){
            count++;
        }
    }
}
```

在多线程并发的情况，结果不尽人意。有count->9999，count->9993，count->9997的，count->10000。为啥

呢？因为count++；并非是原子性操作。一个变量值的修改需要经过三个步骤：①取值，②修改值，③赋值。一个

线程可能前两个步骤就挂了（时间片到了），或者**两个线程 同时 去修改count的值且都修改成功（变量修改的方**

**法没有加锁），那么count的值只会增加1，而不是增加了2（线程数量依赖于机器与操作系统）。**



**2.2 、解决问题办法**

2.2.1 使用**volatile**关键字修饰。结果失败，原因是：volatile关键字只能保证可见性，不保证原子性。

- 保证可见性

  在多线程的情况下，被改关键字修饰的变量，其在任意时间被线程修改后的值，对所有线程都是透明的，线程

  获取变量的值，都是获取到最新的值。

- 不保证原子性

  在**例2.1**的代码中，当两个线程同时去修改变量count的值且都成功，那么count的值只会增加1，而不是增加

  了2。例如：count值为1，由于volatile保证了可见性，线程A和线程B，能同时获取count的值为1（修改变量

  的方法没有上锁）。线程A对count进行加1，count的值变为。线程B也对count加，count的值也是变为2。两

  次加1操作结果本应该为2的，可结果却为1，结论是volatile不会保证原子性。

> volatile可以保证可见性的原因是：volatile关键字让JVM在内存设置了内存屏障，也就是当内存中的值被修改
>
> 后，将被修改后的值，强制写入到磁盘，并刷新内存(将其值清除）。当有获取变量值的请求时，在内存中查
>
> 询不到，就要到磁盘中读取，同时放入到内存中，所以保证了获取到数据都是最新的。



2.2.2 使用锁

- 悲观锁——对修改共享变量值修改的方法上锁，结果肯定是成功的。但是使用锁就意味放弃部分性能，对于一

  个共享变量的修改就需要上锁，未免小题大做了吧。

- 乐观锁——使用CAS机制（自旋），结果也是成功的。但是CAS也存在一个缺点，那就是ABA问题。ABA问题

  可以这样描述：

  ①现在有一个共享变量count=10，同时有三个线程A，B，C 要对count就行操作。

  ②线程A，B同时获取到count的值为10，然后线程A被挂起。

  ③线程B使用CAS将count的值改成15。

  ④修改成功后，线程C使用CAS获取到的count值为15，然后将count的值改成为10。

  ⑤最后线程A获得CPU执行权，比较count的值，发现匹配是成功的，然后将count的值改成 -1；

  上述情况是可以正常执行的，但是这样执行存在很大的安全隐患，线程A没有得知线程B，C对count做了啥，

  就对count做出修改，很容易出现问题。

代码演示：

```java
public class AtomicStudy {
    int value=10;
    public static void main(String args[]) throws InterruptedException {
        AtomicStudy atomic = new AtomicStudy();
        new Thread(()->{
            atomic.updateValue(atomic.getValue(),-1);
        },"A").start();
        
        new Thread(()->{
            atomic.updateValue(atomic.getValue(),15);
        },"B").start();
        
        Thread.sleep(1000);  //模拟线程B先执行
        new Thread(()->{
            atomic.updateValue(atomic.getValue(),10);
        },"C").start();   
    }

    public int getValue(){
        return this.value;
    }

    public void updateValue(int expectedValue,int newValue){
        int oldValue=value;
        String name = Thread.currentThread().getName();
        if ("A".equals(name)){ //模拟线程A被挂起
           try {
               Thread.sleep(5000);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
        }
        if (oldValue==expectedValue){
            value=newValue;
            System.out.println("线程"+name+"竞争成功"+" value:"+value);
        }else{
            System.out.println("线程"+name+"竞争失败");
        }
    }
}
/*输出结果
线程B竞争成功 value:15
线程C竞争成功 value:10
线程A竞争成功 value:-1
*/
```



**2.3、Atomic变量原子类**

在java.util.concurrent.atomic路径下存在Jdk提供的原子变量类，截图如下：

![Atomic原子变量类图](https://github.com/jogin666/blog/blob/master/resource/java/%E5%B9%B6%E5%8F%91/images/Atomic%E5%8E%9F%E5%AD%90%E5%8F%98%E9%87%8F%E7%B1%BB%E5%9B%BE.png)

有上面截图可以对原子变量类大致分为如下几类：

- 基本类型

  AtomicInteger  	//整形

  AtomicBoolean	 //布尔型

  AtomicLong   	 //长整型

  DoubleAccumulator 	//双浮点型（ jdk1.8）

  DoubleAdder   	//双浮点型 （jdk1.8）

  LongAccumulator  //长整型（ jdk1.8）

  LongAdder  //长整型（ jdk1.8）

  > 其中DoubleAccumulator 和 DoubleAdder  是双浮点型的补充。而LongAccumulator  和 LongAdder是
  >
  > 对AtomicLong类的改进，弥补其性能不够高的缺点。jdk1.8版本推荐使用LongAdder代替AtmicLong。
  >
  > 为啥呢？AtmoicLong在并发的情况使用CAS被告知失败后，会补不断的自旋尝试修改，直到操作成功，
  >
  > 这是很浪费CPU资源的。而LongAdder：内部核心数据value**分离**成一个数组(Cell)，每个线程访问时,通
  >
  > 过哈希等算法映射到其中一个数字进行计数，而最终的计数结果，则为这个数组的**求和累加**。

有兴趣的同学请传送门走一波：

①<a href="https://zhuanlan.zhihu.com/p/45489739">AtomicLong与LongAdder性能对比 </a> 

②<a href="https://zhuanlan.zhihu.com/p/38288416">Java-Concurrent（二）-LongAdder源码详解</a>



- 数组类型

  AtomicIntegerArray	//整型数组

  AtomicLongArray		//长整型数组

  AtomicReferenceArray  //引用类型数组

- 引用类型

  AtomicReference	//引用类型

  AtomicStampedReference：带有版本号的引用类型

  AtomicMarkableReference：带有标记位的引用类型

- 对象属性类型

  AtomicIntegerFieldUpdater：对象的属性是整型

  AtomicLongFieldUpdater：对象的属性是长整型

  AtomicReferenceFieldUpdater：对象的属性是引用类型



查看原子类的源码可以发现：Atomic原子变量类的内部都维护一个Unsafe类的实例对象，几乎所有的操作都是有

Unsafe实例完成的，或者说Atomic原子变量类都是Unsafe的包装类。查看Unsafe源码，可以发现Unsafe是使用

CAS完成修改值的。

```java
//第一个参数是对象的引用	对象的地址值，	第三个参数是变化值
public native void putInt(Object var1, long var2, int var4);

public native void putObject(Object var1, long var2, Object var4);

public native void putBoolean(Object var1, long var2, boolean var4);

//第一个参数是对象的引用	对象的地址值，	第三个参数是期待值，第四个参数是更新值，
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2,long var4, long var6);
```

Atomic原子变量类的实现原理：原子变量类内部维护一个Unsafe实例对象，操作Atomic原子变量类，就是操作

Unsafe。而Unsafe的底层是C实现的。C代码调用汇编，生成一条CPU的cmpxchg指令，完成操作。即：CAS本质

是一条CPU指令，不会中断，所以说CAS操作是原子性操作。

**2.3  原子变量的使用代码演示**

- 解决共享变量的自增问题

```java
public class AtomicStudy {
    
    public static void main(String args[]) throws InterruptedException {
        AtomicInteger count=new AtomicInteger(0);
        //核心线程等于最大线程数=20的线程池
        ExecutorService service = Executors.newFixedThreadPool(20); 
        for (int i=0;i<10000;i++){
            service.execute(()->count.incrementAndGet());
        }
        service.awaitTermination(2, TimeUnit.SECONDS); //等待两秒钟
        service.shutdown(); //结束线程池
        System.out.println(count.get()); //结果肯定为10000 
    }
}

/****************************分割线 源码****************************/
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1; //返回内存的值+1的结果
}
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    //采用自旋
    do {
        var5 = this.getIntVolatile(var1, var2); //获取内存值
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4)); //CAS

    return var5; //返回获取内存中的值
}
```

记得遇到不会的API多看看源码哦



- 解决CAS的ABA问题

  解决这个问题，只需要使用JDK提供的AtomicStampedReference和AtomicMarkableReference类皆可以了。

  AtomicStampedReference类和AtomicMarkableReference类为对象提供版本化，每一次操作设置一个版本

  号，CAS比较不再是对象的内存值，而是对象的版本号，如果版本号与预期值一致，就修改，反之告知失败。

查看源码：

```java
public class AtomicStampedReference<V> {

    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

    private volatile Pair<V> pair;
    //.....
 	public void set(V newReference, int newStamp) {
        Pair<V> current = pair;
        //比较对象 和 版本 是否一致
        if (newReference != current.reference || newStamp != current.stamp)
            this.pair = Pair.of(newReference, newStamp);
    }  
    //比较的Pair
    public boolean compareAndSet(V   expectedReference, V   newReference,
                                 			int expectedStamp,int newStamp) {
        Pair<V> current = pair;
        return expectedReference == current.reference &&  expectedStamp == current.stamp 
           			 && ((newReference == current.reference && newStamp == current.stamp)
                 	|| casPair(current, Pair.of(newReference, newStamp)));
    }
}
```



参考资料： <a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484416&idx=1&sn=540c0714263f8ee8b80ba90535162657&chksm=ebd74501dca0cc179e66c34cf3fa647f18860c670b47b0612fac0cb2c26b6cb17ad6824f0808&token=465096859&lang=zh_CN###rd">还在用Synchronized？Atomic你了解不？</a>

