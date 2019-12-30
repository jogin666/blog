## CLH锁和MCS锁介绍

### 一、背景介绍

写在前面，以下内容来源于：<a href="https://blog.csdn.net/claram/article/details/83828768">算法：CLH锁的原理及实现</a>



**1.1 SMP(Symmetric Multi-Processor)**

对称多处理器结构，它是相对非对称多处理技术而言的、应用十分广泛的并行技术。在这种架构中，一台计算机由

多个CPU组成，并共享内存和其他资源，所有的CPU都可以平等地访问内存、I/O和外部中断。虽然同时使用多个

CPU，但是从管理的角度来看，它们的表现就像一台单机一样。操作系统将任务队列对称地分布于多个CPU之上，

从而极大地提高了整个系统的数据处理能力。但是随着CPU数量的增加，每个CPU都要访问相同的内存资源，共享

资源可能会成为系统瓶颈，导致CPU资源浪费。

![SMP：对称多处理器结构](https://img-blog.csdnimg.cn/20181107180131269.png)



**1.2 NUMA(Non-Uniform Memory Access)**

非一致存储访问，将CPU分为CPU模块，每个CPU模块由多个CPU组成，并且具有独立的本地内

存、I/O槽口等，模块之间可以通过互联模块相互访问，访问本地内存（本CPU模块的内存）的

速度将远远高于访问远地内存(其他CPU模块的内存)的速度，这也是非一致存储访问的由来。

NUMA较好地解决SMP的扩展问题，当CPU数量增加时，因为访问远地内存的延时远远超过本地

内存，系统性能无法线性增加。

![NUMA：非一致存储访问](https://img-blog.csdnimg.cn/20181107180254509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NsYXJhbQ==,size_16,color_FFFFFF,t_70)



**1.3 CLH、MCS命名来源**

MCS：John Mellor-Crummey and Michael Scott

CLH：Craig，Landin andHagersten



### 二、CLH锁介绍

CLH是一种基于单向链表的高性能、公平的自旋锁。申请加锁的线程通过前驱节点的变量进行自旋。在前置节点解

锁后，当前节点会结束自旋，并进行加锁。在SMP架构下，CLH更具有优势。在NUMA架构下，如果当前节点与前

驱节点不在同一CPU模块下，跨CPU模块会带来额外的系统开销，而MCS锁更适用于NUMA架构。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181107191206507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NsYXJhbQ==,size_16,color_FFFFFF,t_70)

>  锁值：我把自旋条件定义为锁值 locked。locked == true 表示节点的处于加锁状态或者等待加锁状态，locked == false 表示节点处于解锁状态。

基于线程当前节点的前置节点的锁值（locked）进行自旋，locked == true 自旋，locked == false 加锁成功。

locked == true 表示节点处于加锁状态或者等待加锁状态。

locked == false 表示节点处于解锁状态。

每个节点在解锁时更新自己的锁值（locked），在这一时刻，该节点的后置节点会结束自旋，并进行加锁。



**加锁逻辑**

1. 获取当前线程的锁节点，如果为空则进行初始化。
2. 通过同步方法获取尾节点，并将当前节点置为尾节点，此时获取到的尾节点为当前节点的前驱节点。
3. 如果尾节点为空，则表示当前节点为第一个节点，加锁成功。
4. 如果尾节点不为空，则基于前驱节点的锁值（locked==true）进行自旋，直到前驱节点的锁值 locked == false。

**解锁逻辑**

1. 获取当前线程的锁节点，如果节点为空或者锁值（locked== false）则无需解锁，直接返回。

2. 使用同步方法为尾节点赋空值，赋值不成功则表示当前节点不是尾节点，需要将当前节点的 locked == false 

   已保证解锁该节点。如果当前节点为尾节点，则无需设置该节点的锁值。因为该节点没有后置节点，即使设置

   了，也没有实际意义。

**代码实现**

```java
public class CLHLock implements Lock {
    private AtomicReference<CLHNode> tail;
    private volatile ThreadLocal<CLHNode> threadLocal;

    public CLHLock() {
        this.tail = new AtomicReference<>();
        this.threadLocal = new ThreadLocal<>();
    }

    @Override
    public void lock() {
        CLHNode curNode = threadLocal.get(); 
        if(curNode == null){ //结点为空，则创建结点
            curNode = new CLHNode();
            threadLocal.set(curNode);
        }
        CLHNode predNode = tail.getAndSet(curNode); //设置当前为尾部结点
        if(predNode != null){ //前驱不为空，则不断自旋前驱结点是否释放锁，反之是首部结点，直接运行
            while (predNode.getLocked()){
            }
        }
    }

    @Override
    public void unlock() {
        CLHNode curNode = threadLocal.get();
        threadLocal.remove(); 
        if(curNode == null || curNode.getLocked() == false){//释放锁
            return;
        }
        if(!tail.compareAndSet(curNode, null)){ //最后一个结点放行后，将尾部结点设置为null
            curNode.setLocked(false);
        }
    }

    public static void main(String[] args) {
        final Lock clhLock = new CLHLock();
        for (int i = 0; i < 10; i++) {
            new Thread(new DemoTask(clhLock, i + "")).start();
        }
    }

    class CLHNode { //CLH没有显示后继结点
        private volatile boolean locked = true; //是否获得锁
        public boolean getLocked() {
            return locked;
        }
        
        public void setLocked(boolean locked) {
            this.locked = locked;
        }
    }
}
```



### 三、MCS锁介绍

写在前面，一下内容来源于：<a href="https://blog.csdn.net/claram/article/details/83858054">算法：MCS锁的原理及实现</a>

**MCS介绍**

MCS是一种基于单向链表的高性能、公平的自旋锁。申请加锁的线程通过当前节点的变量进行自旋（locked == 

true）。在前置节点解锁后，会修改后继结点（当前节点）的锁值（locked ==false），这一刻当（后继结点）前

节点会结束自旋，并进行加锁。在SMP架构下，CLH更具有优势。在NUMA架构下，如果后继结点（当前节点）与

前驱节点不在同一CPU模块下，CLH自旋频繁的访问前驱节点的变量，跨CPU模块的访问会带来额外的系统开销。

而MCS锁自旋访问的是当前节点的变量，不会产生跨CPU模块的访问，因此MSC锁更适用于NUMA架构。



![MCS队列锁](https://img-blog.csdnimg.cn/20181108104144728.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NsYXJhbQ==,size_16,color_FFFFFF,t_70)



> 锁值：我把自旋条件定义为锁值 locked。locked == true 表示节点的处于加锁状态或者等待加锁状态，locked == false 表示节点处于解锁状态。

1. 基于线程当前节点的锁值（locked）进行自旋，locked == true 自旋，locked == false 加锁成功。
2. locked == true 表示节点处于加锁状态或者等待加锁状态。

3. locked == false 表示节点处于解锁状态。
4. 每个节点在解锁时更新后置节点的锁值（locked），在这一时刻，该节点的后置节点会结束自旋，并进行加锁。

**加锁逻辑**

1. 获取当前线程的锁节点，如果为空则进行初始化。

2. 通过同步方法获取尾节点，并将当前节点置为尾节点，此时获取到的尾节点为当前节点的前驱节点。

3. 如果尾节点为空，则表示当前节点为第一个节点，加锁成功。

4. 如果尾节点不为空，则基于当前节点的锁值（locked==true）进行自旋，直到前驱节点解锁时，将当前节点的

   锁值置为 false，此时当前节点结束自旋并进行加锁。


**解锁逻辑**

1. 前驱结点获取当前线程的锁节点，如果节点为空或者锁值（locked== true）则无需解锁，直接返回。

2. 使用同步方法为尾节点赋空值，赋值不成功则表示当前节点不是尾节点，需要将当前节点的后置节点 locked 

   == false 已保证解锁该节点。如果当前节点为尾节点，则无需设置该节点的锁值。因为该节点没有后继节点，

   即使设置了，也没有实际意义。

**代码演示**

```java
public class MSCLock implements Lock {
    private AtomicReference<MSCNode> tail;
    private volatile ThreadLocal<MSCNode> threadLocal;

    public MSCLock() {
        this.tail = new AtomicReference<>();
        this.threadLocal = new ThreadLocal<>();
    }

    @Override
    public void lock() {
        MSCNode curNode = threadLocal.get();
        if (curNode == null) {
            curNode = new MSCNode(); //为当前线程创建结点
            threadLocal.set(curNode);
        }

        MSCNode predNode = tail.getAndSet(curNode);//获取尾部结点
        if (predNode != null) {
            predNode.setNext(curNode); //将获取的尾部结点设置为当前结点的前驱结点
            while (curNode.getLocked()) { //自旋
            }
        } else {
            curNode.setLocked(false); //当前结点是首结点
        }
    }

    @Override
    public void unlock() {
        MSCNode curNode = threadLocal.get();
        threadLocal.remove();
        if (curNode == null || curNode.getLocked() == true) {//线程仍在等待锁
            return;
        }
        //尝试判断该结点是否为尾部结点
        if (curNode.getNext() == null && 
            					!tail.compareAndSet(curNode, null)) {
            while (curNode.getNext() == null) {
            }
        }
        if (curNode.getNext() != null) {
            curNode.getNext().setLocked(false); //释放锁
            curNode.setNext(null); //丢弃当前线程
        }
    }

    public static void main(String[] args) {
        final Lock mscLock = new MSCLock();

        for (int i = 0; i < 10; i++) {
            new Thread(new DemoTask(mscLock, i + "")).start();
        }
    }

    class MSCNode {
        private volatile boolean locked = true; //是否获得锁
        private volatile MSCNode next = null; //显示的后继结点

        public boolean getLocked() {
            return locked;
        }

        public void setLocked(boolean locked) {
            this.locked = locked;
        }

        public MSCNode getNext() {
            return next;
        }

        public void setNext(MSCNode next) {
            this.next = next;
        }
    }
}
```

