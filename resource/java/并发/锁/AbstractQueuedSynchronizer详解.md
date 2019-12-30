## AbstractQueuedSynchronizer详解

**一、初步认识AbstractQueuedSynchronized**

**1.1、 概述**

AbstractQueuedSynchronizer是java.util.concurrent.locks包下的一个抽象类，该类简称AQS（同步器），为

Java的所实现提供基础框架。其内部的实现依赖是原子变量state（同步状态，也就是线程获取资源使用的锁是使

用状态还是空闲状态，不同子类，state是不一样的）和先进先出的队列。AQS也为线程提供了两种资源的访问模

式：独占模式和共享模式。同时AQS中定义了ConditonObject内部类，在子类扩展时，该内部类为为实现提先进

先出的条件队列提供基础。

**1.2 设计思想**

AQS（同步器）的核心方法是`void acquire(int arg)`和`boolean release(int arg)`方法

```java
//使用到模板模式，方法延迟到子类中实现
//独占式
public final void acquire(int arg) {
    //如同步状态不允许允许操作
    if (!tryAcquire(arg) && 
        //如果当前线程不在队列中，则将线程添加到队列的尾部
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt(); //阻塞线程
}
public final boolean release(int arg) {
    //获取状态的线程释放状态成功
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h); //则解除队列中头结点的阻塞状态
        return true;
    }
    return false;
}

//共享式
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);  //访问资源
}
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared(); //释放
        return true;
    }
    return false;
}
```

从上面的操作思想中可以得知，AQS的实现依赖于AQS的状态，线程的阻塞与释放，FIFO队列的管理。

**1.2.1、原子性的同步状态state **

```java
private volatile int state; //使用volatile修饰，保证state的可见性

protected final int getState() {
    return state;
}
protected final void setState(int newState) {
    state = newState;
}
protected final boolean compareAndSetState(int expect, int update) {
    //使用CAS更改state的值
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update); //保证原子性
}
```

由源码可知，state使用volatile关键字修饰保证了可见性，在AQS向子类暴露修改state值的compareAndSetState

方法中适应CAS，确保了state修改操作的原子性，这样一来就确保了state的可见性，有序性和原子性。在AQS的

子类中（如锁，信号量等）必须根据AQS暴露出state相关的方法重新定义tryAcquire和tryReslease方法，以辅助

acquire和release操作。

**1.2.2、线程的阻塞与释放**

jdk的java.util.concurrent.locks包下的LockSupport为Java线程阻塞提供了支持。LockSupport的park方法阻塞当

前线程直到LockSupport的unpark方法被调用。unpark方法的调用是没有计数的。因此一个park方法调用前多次

调用unpark方法只会接触一个park操作。值得注意的是：park方法和unpark方法作用于线程而不是同步器(QAS)

**1.2.3、FIFO队列的管理**

AQS的线程队列是严格的FIFO队列，因此没有支持线程优先级的同步。同步队列的最佳选择是自身没有使用底层

锁来构造的非阻塞的数据结构。业界有两种选择：MSC锁和CLH锁。其中CLH一般是用于自旋，但相比MSC。CLH

更容易实现取消和超时，所以AQS选择了CLH作为实现基础。

同步队列介绍：

* CLH队列并不那么像队列，它的出队和入队与实际的业务使用场景密切相关。它是一个链表队列，通过AQS的

  两个字段head（头节点）和tail（尾节点）来存取，这两个字段是volatile类型，初始化的时候都指向了一个空

  节点。如下图：

  ![img](https://images2018.cnblogs.com/blog/289599/201808/289599-20180812201948093-418701167.png)

* 入队操作：CLH队列是FIFO对象，，故新的节点到来的时候，是要插入到当前队列的尾节点之后。当线程无法

  获取AQS的状态时（已被其他线程获取到），转而构造成链表加入到CLH队列中，而加入队列的过程必须是线

  程安全的，因此AQS提供了一个CAS方法，比较预期尾部结点与实际尾部结点时候一致，只要一致才能成功设

  置，反之自旋操作，当设置成功后，构造的结点才会与尾部结点简历关联。

  ![img](https://images2018.cnblogs.com/blog/289599/201808/289599-20180812202051742-1948605021.png)

* 出队操作：因为遵循FIFO规则，所以能成功获取AQS的同步状态必定是首部结点。首结点在释放同步状态时，

  会唤醒后继结点，而后继结点获取到同步状态成功后将自己设置为首结点。设置首结点的操作有获取AQS状态

  的线程来完成，由于只有一个线程能获取AQS的同步状态，所以设置首结点的方法不需要CAS操作，只需要将

  首结点的后继结点设置为首结点，同时断开该结点的前后继引用即可

  ![img](https://images2018.cnblogs.com/blog/289599/201808/289599-20180812202637706-938220186.png)

条件队列介绍：

* ConditionObject类实现了Condition接口，Condition接口提供了类似Object管程式的方法，如await、signal

  和signalAll操作，还扩展了带有超时、检测和监控的方法。ConditionObject类有效地将条件与其它同步操作

  结合到了一起。该类只支持Java风格的管程访问规则，这些规则中，**当且仅当当前线程持有锁且要操作的条件**

  **（condition）属于该锁时，条件操作才是合法的**。这样，一个ConditionObject关联到一个ReentrantLock上

  就表现的跟内置的管程（通过Object.wait等）一样了。两者的不同仅仅在于方法的名称、额外的功能以及用

  户可以为每个锁声明多个条件。

  ConditionObject类和AQS共用了内部结点，但也有独自的条件队列。ConditionObject会将符合条件的结点转移到同步队列中，不符合的条件的结点则留在条件队列中。

  signal操作从条件队列中将结点移到同步队列中。

  ![img](https://images2018.cnblogs.com/blog/289599/201808/289599-20180812202850477-387948953.png)

  await操作则将同步队列中的结点（当前线程结点）移到提交队列中

![img](https://images2018.cnblogs.com/blog/289599/201808/289599-20180812202959032-1029783103.png)



**1.2节内容来自：**<a href="https://www.cnblogs.com/iou123lg/p/9464385.html">[Java并发编程-看懂AQS的前世今生](https://www.cnblogs.com/iou123lg/p/9464385.html)</a>



### 二、AQS源码了解

**2.1 、AQS部分方法与队列介绍**

* AQS同步状态——请看1.2节前面介绍
* 获取AQS状态和释放AQS状态方法  请看1.2节首部。
* CLH队列

```java
static final class Node {
   
    static final Node SHARED = new Node();  //共享结点
    static final Node EXCLUSIVE = null;  //独占
    static final int CANCELLED =  1;    //线程被取消了
    static final int SIGNAL    = -1;   //后继线程需要唤醒
    static final int CONDITION = -2;	//等待condition唤醒
   
    static final int PROPAGATE = -3;

    volatile int waitStatus; //线程的等待状态 为上述几种状态

    volatile Node prev; //前继结点
    volatile Node next; //后继结点
    volatile Thread thread; //等待的线程
    
    Node nextWaiter;

    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {   }

    Node(Thread thread, Node mode) {    
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

