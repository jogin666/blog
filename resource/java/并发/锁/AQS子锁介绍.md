## AQS子锁锁介绍

上面主要讲了AQS的实现思想与设计（博客转载），本片就来学习学习AQS主要常用的子类ReentrantLock（重入

锁） 和  ReetrantReadWriteLock（读写锁）。



### 一、ReentrantLock锁介绍

**1、ReetrantLock的知识概要**

* 和synchronized类似，是互斥锁，但比其灵活
* 需要手动手动声明加锁和释放锁
* 支持公平锁（相对而言）和非公平锁两种锁，可根据构造函数定义
* 公平锁(是相对公平的)  ，线程加锁时，检测该线程是否位于双向队列的第一个结点

**1.2、例子演示**

```java
public void example(){
    ReentrantLock lock=new ReentrantLock();
    lock.lock(); //尝试获取锁
    try{
        //....
    }finally{
        lock.unlock();  //释放锁
    }
}
```

**1.3、类结构**

![ReentrantLock类结构图](https://github.com/jogin666/blog/blob/master/resource/java/%E5%B9%B6%E5%8F%91/images/ReentrantLock%E7%B1%BB%E7%BB%93%E6%9E%84%E5%9B%BE.png)

![ReentrantLock内部类锁结构图](https://github.com/jogin666/blog/blob/master/resource/java/%E5%B9%B6%E5%8F%91/images/ReentrantLock%E5%86%85%E9%83%A8%E7%B1%BB%E9%94%81%E7%BB%93%E6%9E%84%E5%9B%BE.png)

> 由图可以猜出,ReentrantLock的内部实现是应该是由Sync完成的，而Sync是继承AQS的，也就说AQS是实现锁的基础框架。

**1.4、Sync类和其子类**

> 十分值得注意的是：在Sync中，**同步器的状态是锁重入的次数**

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;

   //获取锁留的实现由子类实现
    abstract void lock();

   //获取非公平锁
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread(); //获取当前线程
        int c = getState();
        if (c == 0) {//同步器处于空闲状态（没有线程斥锁）
            if (compareAndSetState(0, acquires)) { //尝试上锁
                setExclusiveOwnerThread(current);//设置为当前线程是锁拥有者
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {  //线程要求重入锁
            int nextc = c + acquires;
            if (nextc < 0) // 溢出
                throw new Error("Maximum lock count exceeded");
            setState(nextc); //更改同步器状态
            return true;
        }
        return false;
    }
	//释放锁
    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        //当前线程不是拥有锁的线程，就抛出异常（HCL锁的实现)
        if (Thread.currentThread() != getExclusiveOwnerThread()) 
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) { //释放成功
            free = true;
            setExclusiveOwnerThread(null); //锁为空闲状态
        }
        setState(c);
        return free;
    }
    //........
}
//非公平锁
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    final void lock() {
        if (compareAndSetState(0, 1))
            //获取锁成功，则设置为当前线程是锁拥有者
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);  //失败则调用AQS的方法，而AQS则会调用该类实现的tryAcquire方法
    }
    
    protected final boolean tryAcquire(int acquires) {
        //调用Sync的方法尝试获取锁，如果失败，则当前线程会被AQS放入到同步队列中
        return nonfairTryAcquire(acquires); 
    }
}

static final class FairSync extends Sync {
     private static final long serialVersionUID = -3000897897090466540L;

     final void lock() {
         acquire(1); //使用AQS的方法
     }
    //AQS调用的方法
     protected final boolean tryAcquire(int acquires) { 
         final Thread current = Thread.currentThread(); //获取当前线程
         int c = getState(); 
         if (c == 0) { //同步器空闲状态
			/*翻看源码得知：
			 hasQueuedPredecessors方法判断当前线程是否位于CLH同步队列中的第一个。
             如果是则返回flase，否则返回true。
             */
             if (!hasQueuedPredecessors() && compareAndSetState(0, acquires)) {
                 setExclusiveOwnerThread(current);
                 return true;
             }
         }
         else if (current == getExclusiveOwnerThread()) {
             int nextc = c + acquires;
             if (nextc < 0)
                 throw new Error("Maximum lock count exceeded");
             setState(nextc);
             return true;
         }
         return false;
     }
 }
```

**1.5、构造函数**

```java
public ReentrantLock() {
    sync = new NonfairSync(); //默认非公平锁
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync(); //true：使用公平锁 ，反之非公平锁
}
```

**1.6、主要方法**

* 上锁 `void lock()`

```java
public void lock() {
    sync.lock(); //详情请看1.4节
}
```

* 解锁 `void unlock()`

```java
public void unlock() {
    sync.release(1); //使用父类的方法
}
public final boolean release(int arg) {
    if (tryRelease(arg)) { //使用Sync的方法 见1.4节
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h); //解除CLH队列中的首部结点的线程的等待
        return true;
    }
    return false;
}
```



### 二、ReentrantReadWriteLock介绍

**2.1、知识概要**

* 该类内部有两个锁：读锁与写锁，适合读多写少的加锁操作	
* 读写是共享的，也就是多个线程可以同时访问临界资源
* 写锁是互斥的，只允许一个线程访问临界资源
* 写锁可以进入读锁（读锁是共享的），而读锁不能进入写锁（写锁是互斥的）
* 线程使用读锁获取到的数据都是最新的（volatile关键字修饰）
* 写锁可以降级为读锁，而读锁不能升级为写锁
* 读锁不支持条件对象，写锁支持条件对象
* 读写锁都支持公平锁与非公平锁两种模式。当公平获取读锁时，需要当前线程写锁完成后才能获取读锁；当公平获取写锁，当前线程无其他锁。

**2.2、类结构**

![ReentrantReadWriteLock类结构图](https://github.com/jogin666/blog/blob/master/resource/java/%E5%B9%B6%E5%8F%91/images/ReentrantReadWriteLock%E7%B1%BB%E7%BB%93%E6%9E%84%E5%9B%BE.png)

> WriteLock和ReadLock实现Lock接口，完成读写加锁的操作。

**2.2读写锁介绍**

* ReadLock类源码

```java
 public static class ReadLock implements Lock, java.io.Serializable {
     private final Sync sync;
     protected ReadLock(ReentrantReadWriteLock lock) {
         sync = lock.sync;
     }
     public void lock() {
         sync.acquireShared(1); //共享
     }
     public void unlock() {
         sync.releaseShared(1); //释放锁
     }
     public boolean tryLock() {
         return sync.tryReadLock(); 
     }
     //........
 }
/* 可以看出ReadLock的实现还是有Sync子类（公平锁/非公平锁）实现的*/
```

* WriteLock类源码

```java
public static class WriteLock implements Lock, java.io.Serializable {
	private final Sync sync;

    protected WriteLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }
    public void lock() {
        sync.acquire(1); //独占锁
    }
    public boolean tryLock( ) {
        return sync.tryWriteLock();
    }
    public void unlock() {
        sync.release(1); //释放锁
    }
    //.....
}
/* 可以看出WriteLock的实现还是有Sync子类（公平锁/非公平锁）实现的*/
```

**2.3、Sync类与其子类**

* Sync类源码

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
   
    //读锁和写锁状态表示，高16位表示共享（读）状态，低16位表示独占（写）状态
    static final int SHARED_SHIFT   = 16;
   
    static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
    static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1; //最大重入次数
    static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;
	
    //.....
    abstract boolean readerShouldBlock();
    abstract boolean writerShouldBlock();
	//释放独占锁
    protected final boolean tryRelease(int releases) {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        int nextc = getState() - releases;
        //判断锁状态的低16位（写锁状态）是否为0，如果为0则返回true，否则返回false.
        boolean free = exclusiveCount(nextc) == 0;
        if (free)
            setExclusiveOwnerThread(null); //释放锁
        setState(nextc); //设置锁状态
        return free;
    }

    /*独占锁的获取需要三个条件
    1. 当前线程获是CLH队列中下一个要获取锁的线程且不存在读锁
    2. 同一线程获取写锁不能超过最大次数（重入次数）
    3. 锁处于空闲状态
    */
    protected final boolean tryAcquire(int acquires) {
       
        Thread current = Thread.currentThread(); 
        int c = getState();
        int w = exclusiveCount(c);
        if (c != 0) {    //锁处于空闲状态
            //存在读锁或者当前线程不处于队列中的首部，返回false
            if (w == 0 || current != getExclusiveOwnerThread())
                return false;
            //锁已经饱和，达到重入次数的最大值
            if (w + exclusiveCount(acquires) > MAX_COUNT) 
                throw new Error("Maximum lock count exceeded");    
            //符合条件
            setState(c + acquires); 
            return true;
        }
        //非空闲，或者获取锁失败 则阻塞当前线程
        if (writerShouldBlock() || !compareAndSetState(c, c + acquires))
            return false;
        setExclusiveOwnerThread(current);
        return true;
    }

    protected final boolean tryReleaseShared(int unused) {
        Thread current = Thread.currentThread();
        if (firstReader == current) {
            if (firstReaderHoldCount == 1)
                firstReader = null; //更新读锁持有者，
            else
                firstReaderHoldCount--; //更新重入次数
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                rh = readHolds.get();
            int count = rh.count;
            if (count <= 1) {
                readHolds.remove();
                if (count <= 0)
                    throw unmatchedUnlockException();
            }
            --rh.count;
        }
        for (;;) { //自旋操作
            int c = getState();
            int nextc = c - SHARED_UNIT;
            if (compareAndSetState(c, nextc))           
                return nextc == 0;
        }
    }
    
    /*读锁获取需要满足三个条件
    1. 当前不存在写锁 或者当写锁由当前线程持有
    2. 当前线程不被阻塞，且是读锁持有的候选线程（CLH的首部结点）
    3. 当前线程的冲入次数没有超过最大值
    */
    protected final int tryAcquireShared(int unused) {    
        Thread current = Thread.currentThread();
        int c = getState();
        //存在写锁且写锁非本线程拥有，则返回-1
        if (exclusiveCount(c) != 0 && getExclusiveOwnerThread() != current)
            return -1;
        int r = sharedCount(c);
        //线程没有被阻塞且重入次数为超过最大值，则上锁
        if (!readerShouldBlock() && r < MAX_COUNT 
            				&& compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) { //是否第一次加锁
                firstReader = current; 
                firstReaderHoldCount = 1;
            } else if (firstReader == current) { //重入
                firstReaderHoldCount++; //重入次数+1
            } else {
                //非 firstReader 读锁重入计数更新，读锁重入计数缓存，基于ThreadLocal实现
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return 1;
        }
        //上诉操作失败，则调用fullTryAcquireShared方法
        return fullTryAcquireShared(current);
    }
	//......
}
```

* 公平锁 FairSync

```java
static final class FairSync extends Sync { 
    //FairSync严格遵循FIFO，如果有前驱节点，则返回false，否则返回true。
    final boolean writerShouldBlock() {
        return hasQueuedPredecessors();
    }
    final boolean readerShouldBlock() {
        return hasQueuedPredecessors();
    }
}
```

* 非公平锁

```java
static final class NonfairSync extends Sync {
   //
    final boolean writerShouldBlock() {
        return false; // 总是阻塞，进入丢列
    }
    final boolean readerShouldBlock() {
        return apparentlyFirstQueuedIsExclusive();
    }
}
```



读写锁好难啊，目前只能先这样介绍了！以后技术能力提高且需要多线程的知识时，我会回来的！



参考资料：

<a href="https://www.cnblogs.com/zaizhoumo/p/7782941.html">[Java并发编程--ReentrantReadWriteLock](https://www.cnblogs.com/zaizhoumo/p/7782941.html)</a>

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484206&idx=1&sn=9722748c0308b3e56220be1c9d939ad7&chksm=ebd7422fdca0cb39ac7825e565ac4e7ed7fd77638da1a931f916d3b6c06ef50beb5c085510bf&scene=21###wechat_redirect">Lock锁子类了解一下</a>