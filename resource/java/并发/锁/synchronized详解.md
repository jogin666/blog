## synchronized详解

写在前面：本文内容几乎来源于 <a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484198&idx=1&sn=4d8e372165bb49987a6243f17153a9b4&chksm=ebd74227dca0cb31311886f835092c9360d08a9f0a249ece34d4b1e49a31c9ec773fa66c8acc&scene=21###wechat_redirect">Java锁机制了解一下</a>



Java内存模式(JMM)围绕原子性，可见性，有序性以及Happen-before原则展开,传送门：

<a href="https://www.cnblogs.com/hujunzheng/p/5118256.html">JMM和happens-before原则</a>。Java的关键字volatile关键字保证只保证可见性和有序性，没有保证原子性，因而该

关键字只能作为辅助，而不能成为保证Java并发编程的结实的后盾。synchronized使用对象锁保证了原子性，可

见性和有序性，为并发编程提供了正确执行的有效保证。



### 一、synchronized锁

##### synchronized说明

每一个Java对象都有一个内置锁(监视器)，也就是对象锁。该内置锁是一种互斥锁，每一次开锁只允许一个线程进

入被锁住的临界资源，被锁住的临界资源是指所有使用同一对象锁的synchronized修饰的代码。Java的关键字

synchronized就是使用Java对象的内置锁，把代码块(方法)给锁起来，实现线程的同步。



##### synchronized的作用

开篇就说了保证了并发编程的原子性，可见性，有序性。

**原子性**：被锁住的代码只允许一个线程进入，不能多个线程同时访问，既不存在线程交替执行代码，进入被锁住的

方法的线程有一次执行完代码的权限。

**可见性**：线程执行完被锁住的代码后，其代码内修改的全局变量对其他线程是透明的。

**有序性**：实现了线程的同步



**synchronized的原理**

```java
public class Main{
	public synchronized void test1(){ //修饰方法
    }

    public void test2(){    
        synchronized (this){	 // 修饰代码块
        }
    }
}
```

来反编译看一下：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib1z9PUpsn1d3iblXxTRriaeWGwqAjWefkGlUr33eTu7rUgr4Vt5vv2bAt8ns5jZvCG8WialywK1ibOyqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**同步代码块**：

- `monitorenter`和`monitorexit`指令实现的

**同步方法**（在这看不出来需要看`JVM`底层实现）

- 方法修饰符上的`ACC_SYNCHRONIZED`实现。

synchronized底层是是**通过monitor和对象头实现的，每一个对对象都有自己的对象头，存储了很多信息，其**

**中一个信息标示是被哪一个线程持有**。(monitor也称为监视器，城管)。

**个线程持有**。

具体可参考：

- https://blog.csdn.net/chenssy/article/details/54883355
- https://blog.csdn.net/u012465296/article/details/53022317



##### synchronized修饰的范围

1、 普通方法 	2、静态方法	3、代码块

```java
public class Main {
    public synchronized void test1(){    //使用对象锁 调用当前方法的Main实例对象 
    }
    
	public void example1(){ 
        synchronized (this){  //使用对象 调用当前方法的Main实例对象 
        }
    }
    
    public synchronized static void test2(){ //使用类锁 Main.class
    }
    
    public void example2(){  //使用类锁  Main.class
        synchronized(Main.class){
        }
    }
}
```



**对象所和类锁**

synchronized修饰非静态方式和使用非`xx.class`形式锁代码块时，使用的是对象所；synchronized修饰静态方

法和使用`xx.class`锁形式锁住代码快时，使用的是类锁(类的字节码文件对象——Class<?>实例对象，也是一种对

象锁，只不过比较特殊)。因而类锁与对象锁并不会造成冲突。也就是说：**获取了类锁的线程和获取了对象锁的线**

**程是不冲突的**！

```java
public class SynchoronizedDemo {

    //synchronized修饰非静态方法
    public synchronized void function() throws InterruptedException {
        for (int i = 0; i <3; i++) {
            Thread.sleep(1000);
            System.out.println("function running...");
        }
    }
    //synchronized修饰静态方法
    public static synchronized void staticFunction()
            throws InterruptedException {
        for (int i = 0; i < 3; i++) {
            Thread.sleep(1000);
            System.out.println("Static function running...");
        }
    }

    public static void main(String[] args) {
        final SynchoronizedDemo demo = new SynchoronizedDemo();

        // 创建线程执行静态方法
        Thread t1 = new Thread(() -> {
            try {
                staticFunction();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        // 创建线程执行实例方法
        Thread t2 = new Thread(() -> {
            try {
                demo.function();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        // 启动
        t1.start();
        t2.start();
    }
}
```

输出结果：

![img](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib1z9PUpsn1d3iblXxTRriaeWGj83uyhtgpXrZIyROaFwJxU2LZerOKmFUjC7BRT5eWs8mwskLbhaLNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

结果证明：**类锁和对象锁是不会冲突的**！



**重入锁**

我们来看下面的代码：

```java
public class Widget {

    public synchronized void doSomething() {    // 锁住了
        //...
    }
}

public class LoggingWidget extends Widget {

    public synchronized void doSomething() {    // 锁住了
        System.out.println(toString() + ": calling doSomething");
        super.doSomething();
    }
}
```

1. 当线程A进入到`LoggingWidget`的`doSomething()`方法时，此时拿到了`LoggingWidget`实例对象的锁。

2. 随后在方法上又调用了父类Widget的`doSomething()`方法，它也被synchronized修饰。

3. 线程A拥有的`LoggingWidget`实例对象锁还没有释放，调用父类Widget的`doSomething()`方法是否还需要获

   取父类的对象锁？

答案：不需要。因为对象锁的持有者是线程，而不是调用。线程A已经拥有了`LoggingWidget`的实例对象锁了，

尝试进入父类的`doSomething()`方法时，如父类方法没有被其他线程上锁持有，则线程A开锁，把父类的方法一

并锁上持有，然后执行代码。这就是内置锁的可重入性。



 **锁的释放**

1. 当方法(代码块)执行完毕后会自动释放锁，不需要做任何的操作。
2. 当一个线程执行的代码出现异常时，其所持有的锁会自动释放。

- 不会由于异常导致出现死锁现象~



### 二、 synchronized的安全性

是不是对类中的每一个方法都有synchronized修饰，就能保证线程安全？

代码示例：

```java
package com.zy.study.thread.book;

public class Main {

    static int[] numbers={1,2,3,4,5,6,7,8,9,10};

    public synchronized void read() throws InterruptedException {
        for(int i=0;i<10;i++){
            System.out.print(numbers[i]+"\t");
            if (i==5){
                Thread.sleep(1000);
            }
        }
        System.out.println();
    }

    public synchronized void write(){
        for (int i=9;i>=0;i--){
            numbers[i]=-1;
        }
    }

}

class Test{
    public static void main(String args[]) throws InterruptedException {
        Main o=new Main();
        Main oo=new Main();

        new Thread(() -> {
            try {
                o.read();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        Thread.sleep(100);
        new Thread(()->oo.write()).start();
    }
}
//输出结果：1  	2	3	4	5	6	-1	-1	-1	-1	
```

答案是不一定安全，为什么会这样呢？

因为对象锁只能排斥拥有当前实例对象的线程进入临界资源而已，并不能排斥拥有其他实例对象的线程进入临

界资源。