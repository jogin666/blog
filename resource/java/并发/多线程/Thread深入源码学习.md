## Thread深入源码学习

并发使用的多线程或线程池在本质上是由Thread类的实例进行操作的，要了解高并发则很有必要要了解Thread。

若对线程不太了解的话，传送门：<a href="">多线程入门</a>

### 一、Thread类的特点

1. 线程名不能为空（手动设置或者底层设置）
2. 在守护线程内创建的线程也是守护线程
3. 线程有优先级别，级别越高越有可能获得CPU的执行权，初始线程的级别是相等的，

> 注：守护线程的作用是在后台执行不重要的任务，如回收内存等。守护线程会在jvm没有任一个非守护线程
>
> 运行时，自动结束，值得注意的是：守护线程不占资源。 在设置线程的优先级时，如果当前线程存在于线程
>
> 组，则设置的优先级需要小于线程组的优先级。



### 二、Thread源码介绍

**2.0、Thread整体结构**

```java
public class Thread implements Runnable {
    /* 线程注册 */
    private static native void registerNatives();
    
    static {
        registerNatives();
    }

    private volatile String name; //线程名
    private int priority; //线程优先级
    private Runnable target; //Runnalbe的实现类
    private boolean     daemon = false; //默认不是守护线程
    private ThreadGroup group; //线程组
    private ClassLoader contextClassLoader; //上下文加载器
    private long tid;  //线程Id
	// ........注：只选取常用且重要的成员介绍
    
    private static int threadInitNumber; //线程编号
    private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }
}   
```

**2.1、部分构造函数介绍**

```java
public Thread() {
    init(null, null, "Thread-" + nextThreadNum(), 0);//nextThreadNum()返回线程的编号
}
public Thread(Runnable target) {
    init(null, target,"Thread-" + nextThreadNum(),0); //“Thread”+线程编号构成默认生成的线程名 
}
public Thread(Runnable target, String name) {
    init(null, target, name, 0);
}
public Thread(String name) {
    init(null, null, name, 0);
}

private void init(ThreadGroup g, Runnable target, String name,long stackSize, 
                  AccessControlContext acc,boolean inheritThreadLocals) {
    //线程名不能为空
    if (name == null) {
        throw new NullPointerException("name cannot be null");
    }
    this.name = name;

    Thread parent = currentThread();  //当前线程是创建线程的父线程
    SecurityManager security = System.getSecurityManager();  //获取安全器
    if (g == null) {
        //从安全器中获取线程组
        if (security != null) {
            g = security.getThreadGroup(); //
        }
        if (g == null) { //安全器获取不到，则取父类的线程组
            g = parent.getThreadGroup();
        }
    }
    g.checkAccess(); //检查线程组的权限

    if (security != null) {
        if (isCCLOverridden(getClass())) {
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
        }
    }

    g.addUnstarted();

    this.group = g; //线程组
    this.daemon = parent.isDaemon(); //是否是守护线程
    this.priority = parent.getPriority(); //优先级
    if (security == null || isCCLOverridden(parent.getClass()))
        this.contextClassLoader = parent.getContextClassLoader();
    else
        this.contextClassLoader = parent.contextClassLoader;
    
    this.inheritedAccessControlContext =
        acc != null ? acc : AccessController.getContext();
    
    this.target = target; //Runnable
    setPriority(priority); //设置优先级
    if (inheritThreadLocals && parent.inheritableThreadLocals != null)
        this.inheritableThreadLocals =
        ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
    /* Stash the specified stack size in case the VM cares */
    this.stackSize = stackSize; //线程栈的大小

    tid = nextThreadID(); //线程Id
}
```

> 注：设置线程名，开发人员可以使用底层的实现，也可以在创建线程时，指定线程线程名，也可在创建线程
>
> 后，使用 `public final synchronized void setName(String name)`方法设置线程名。



**2.2、线程的优先级**

- 线程是有优先级的，但不是说，线程的优先级越高，其就能被放在前面执行。线程的优先级越高，只能代表

  该线程获得CPU执行权的概率越高。线程的优先级高度依赖于操作系统，每一个操作系统的定义是不一样的。

```java
public final static int MIN_PRIORITY = 1; //最低级别

//默认级别
public final static int NORM_PRIORITY = 5;

//最高级别
public final static int MAX_PRIORITY = 10;
```

- 设置线程的优先级

```java
public final void setPriority(int newPriority) {
    ThreadGroup g;
    checkAccess(); //是否有权限操作
    //要处于最高级别与最低级被之间
    if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) { 
        throw new IllegalArgumentException();
    }
    if((g = getThreadGroup()) != null) {
        if (newPriority > g.getMaxPriority()) {
            newPriority = g.getMaxPriority(); //最高只能为线程组的最大优先级
        }
        setPriority0(priority = newPriority);
    }
}
private native void setPriority0(int newPriority);
```

> 值得注意的是：线程的优先级不能高于线程组的优先级



**2.3、守护线程**

**守护线程的定义：**

- 为其他线程提供服务的线程，例如：垃圾回收线程就是守护线程，JVM会为main线程启动一个垃圾回收线程。

**守护线程的特点：**

- JVM没有任一非守护线程执行时，所有守护线程就会自动结束。因为没有了非守护线程执行，jJVM就会退出。
- 在守护线程下创建的线程也是守护线程。

**值得注意的是：**

- 设置守护线程需要在线程启动前设置，使用`setDaemon(boolean on)`方法设置，如果在线程启动后设置线

  程为守护线程，则会抛出异常。

**守护线程下创建的线程也为守护线程的原因：**

```java
private void init(ThreadGroup g, Runnable target, String name,long stackSize, 
                    AccessControlContext acc,boolean inheritThreadLocals) {
    //.....
    this.group = g; //线程组
   
    this.daemon = parent.isDaemon(); //当前线程是否为守护线程取决于父类线程是否守护线程
    
    this.priority = parent.getPriority(); //优先级
   //....                  
}
```

**设置守护线程一定要再线程执行前设置的原因：**

```java
public final void setDaemon(boolean on) {
    checkAccess();
    if (isAlive()) { //如果线程为已经启动，则抛出异常
        throw new IllegalThreadStateException();
    }
    daemon = on;
}

public final native boolean isAlive();
```



**2.4、线程的常用方法**

- `void sleep(long millis)`

```java
/*线程休眠方法
该方法会让线程进入休眠，也就是进入计时等待状态，但线程不会放弃所分配到资源，只是让出所获得的的时间片，让
其他线程有机会执行，计时时间过了之后，线程重新进入就绪状态。该方法是线程控制自身的流程，抛出可能被打断异常。
*/
public static native void sleep(long millis) throws InterruptedException;
```

代码示例：

```java
public static void main(String args[]){
    Thread thread=new Thread(()->{});
    long sTime=System.currentTimeMillis();
    thread.start();
    try {
        thread.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(System.currentTimeMillis()-sTime); //2001
}
```



- void wait(long timeout)`

```java
/*Object类的方法
该方法用于进程之间的通信，使用时，线程会放弃已获得的资源和时间片，但该方法必须要在同步中使用，
同时需要搭配notify()或者notifyAll()方法唤醒，或者设置指定等待时间，让线程等待指定时间。
*/
public final native void wait(long timeout) throws InterruptedException;
```

代码示例：

```java
public void test(){
    Thread t1=new Thread(()->{int i=0;while (i++<1000000){}});
    Thread t2=new Thread(()->{int i=0;while (i++<1000){}});
    long sTime=System.currentTimeMillis();
    t1.start();
    t2.start();
    try {
        synchronized (t1){ // 只能同步使用
            t1.wait(2000);
        }
        synchronized (t2){ //只能同步使用
            t2.wait(5000);
        }
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(System.currentTimeMillis()-sTime);
}
```



- `void yield()`

```java
/*
该方法用于线程之间的调度，放弃线程所分配到时间片，但是不会放弃线程所获得的的资源。
线程会处于就绪状态（有可能刚让出时间片，就又再次获得时间片)
*/
public static native void yield();
```



- `void join(long millis)`

```java
/*线程调用该方法后，其必须执行完成后，其他线程才能执行。其原理使用循环使用wait()方法实现，
*/
public final synchronized void join(long millis)throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) { 
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay); //循环使用wait()设置等待时间，等待时间会一秒一秒减少
            now = System.currentTimeMillis() - base;
        }
    }
}
```

- `void interrupt()`

```java
/*该方法为线程设置中断标志，让线程自主决定什么结束。——线程检测自身有设置中断标志后，进入结束的阶段。
  值得注意的是：设置线程中断标志，首先要检测线程是否处于阻塞状态，如果是，抛出异常，线程退出阻塞状态，
  中断标志设置不成功(为阻塞的线程设置中断标志毫无意义)。然后检测是否有权限设置中断标志。
*/
public void interrupt() {
    if (this != Thread.currentThread())
        checkAccess();

    synchronized (blockerLock) {
        Interruptible b = blocker;
        if (b != null) {
            interrupt0();           //设置中断标志
            b.interrupt(this); //设置要中断的线程
            return;
        }
    }
    interrupt0();
}
```

- `boolean interrupted()`

```java
/*如果线程存在中断标志，则取消设置的中断标志
*/
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}
```

- `boolean isInterrupted()`

```java
/*检测是否有设置中断标志
*/
public boolean isInterrupted() {
    return isInterrupted(false);
}
```

示例代码：

```java
public void test() throws InterruptedException {
    Thread t1=new Thread(()->{int i=0;while (i++<1000000){}});
    t1.start();
    System.out.println(t1.isInterrupted()); //false

    t1.interrupt(); //设置中断标志

    System.out.println(t1.isInterrupted());//检测是否设中断标志  //true

    t1.interrupted(); //若存在中断标志，则取消中断标志

    Thread.sleep(1000); //暂停1s
    System.out.println(t1.isInterrupted()); //false
}
```



**被舍弃的方法**

```java
/**
 * stop() 立刻结束线程，在xx.stop()后的代码都不会执行，十分暴力
 * 一般抛出ThreadDead异常，但可以不用捕获。因为该方法存在重大缺陷，被设置成已过时
 */

/**
 * 线程的暂停与恢复  suspend() 与resume()需要一起使用   已过时
 * this.suspend()   当前线程被暂停，但独占资源的不会释放，因而容易造成死锁
 * this.resume()    暂停的线程被恢复执行，
 * 由于是独占资源，  1.在多线程的请款下容易造成死锁  2.多线程下，数据不同步
 */
```



> 值得注意的是：阻塞状态和挂起状态是两个不一样的状态。 挂起状态：主动让出时间片  有可能放弃所分配的
>
> 资源。而阻塞：被动挂起，放弃时间片，不放弃内存。所以阻塞可能会造成死锁。



**2.5 线程的生命周期**

```java
 public enum State {
 	 NEW, //新建
     RUNNABLE, //就绪
     BLOCKED, //阻塞
     WAITING,//等待
     TIMED_WAITING,//计时等待
     TERMINATED; //结束
 }
```

线程状态变化图：

![线程状态](https://github.com/jogin666/blog/blob/master/resource/java/%E5%B9%B6%E5%8F%91/images/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.png)

根据线程常用的方法，上面的图片可以画成：

![线程执行过程](https://github.com/jogin666/blog/blob/master/resource/java/%E5%B9%B6%E5%8F%91/images/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81%E5%8F%98%E5%8C%96.png)





参考资料：

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484190&idx=1&sn=ab7301e393aa7762be9ef80d30c5fb7a&chksm=ebd7421fdca0cb09f4a880064a8610416df414ea25284e6d5142ea659e4e7e669632cfed4050&scene=21###wechat_redirect">Thread源码剖析</a>

