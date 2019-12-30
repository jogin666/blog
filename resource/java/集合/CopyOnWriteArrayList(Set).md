## CopyOnWrriteArrayList/Set

### 一、CopyOnWriteArrayList功能简介

CopyOnWriteArrayList是juc包提供多线程并发安全的ArrayList，根据类名，可以推断出，容器元素的修改操作是

基于复制原有数组进行操作的。当数组进行更新，添加，删除时，会在相应的方法中显示加锁，同时会对原数组拷

贝一份数组，操作在拷贝数组中进行，操作完成后，在将修改后的数组替换原数组的元素，

数组数据的读取是不用拷贝数组和加锁的。基于CopyOnWriteArraylist的特性，可以得出它是采用空间换时间的

策略，尽可能的保证性能，该类适用于读多写少的操作。



### 二、回顾Vector和Collections.synchronizedList()方法

Vector是ArrayList的前身，且Vector是线程安全的类，因为Vector类中的 几乎每一个方法都有城管

(synchronized)来修饰，确保容器的线程安全。

```java
public synchronized int capacity() {
	return elementData.length;
}

public synchronized int size() {
	return elementCount;
}

public synchronized boolean isEmpty() {
    return elementCount == 0;
}

public synchronized int indexOf(Object o, int index) {
    if (o == null) {
        for (int i = index ; i < elementCount ; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = index ; i < elementCount ; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

 Collections.synchronizedList(List<T> list)是让List子类线程安全的，其线程安全的原理也是使用城管

(synchronized)来实现的，几乎在每一个方法中对实例对象加锁，与Vector的区别是：synchronized关键字是修饰 

list这个对象实例，而不是方法。

```java
 public E get(int index) {
     synchronized (mutex) {return list.get(index);}
 }
public E set(int index, E element) {
    synchronized (mutex) {return list.set(index, element);}
}
public void add(int index, E element) {
    synchronized (mutex) {list.add(index, element);}
}
public E remove(int index) {
    synchronized (mutex) {return list.remove(index);}
}

public int indexOf(Object o) {
    synchronized (mutex) {return list.indexOf(o);}
}
public int lastIndexOf(Object o) {
    synchronized (mutex) {return list.lastIndexOf(o);}
}

public boolean addAll(int index, Collection<? extends E> c) {
    synchronized (mutex) {return list.addAll(index, c);}
}
```

其实用synchronized修饰方法和synchronized修饰代码块没有本质区别，二者都是使用的是对象锁。一个线程在

获得一个实例的对象锁之后，其他线程就不能使用同一个实例调用有synchronized修饰的方法，因为只有拥有对

象锁的线程有"钥匙"打开锁,或者换句话说，一个类所有使用同一对象锁的synchronized修饰代码为一个临界资

源。不太了解Synchronized的话，传送门：<a href="">Synchronized详解</a>

```java
class Test{

    public  void read1() throws InterruptedException {
        synchronized (this){
            System.out.print(Thread.currentThread().getName()+"开始执行：");
            int[] array=new int[]{1,2,3,4,5,6,7,8,9,10};
            int i=0;
            for (int num:array){
                System.out.print(num+"\t");
                if (i++==5){
                    Thread.sleep(5000); //进入睡眠让其他线程执行
                }
            }
            System.out.println();
        }
    }

    public  synchronized void read2() throws InterruptedException {
        String[] array=new String[]{"11","22","33","44","55"};
        System.out.print(Thread.currentThread().getName()+"开始执行：");
        for (int i=0;i<5;i++){
            System.out.print(array[i]+"\t");
            if (i==2){
                Thread.sleep(5000);   //进入睡眠让其他线程执行
            }
        }
        System.out.println();
    }

    public synchronized void read3(){
        System.out.print(Thread.currentThread().getName()+"开始执行：");
        char[] chars=new char[]{'a','b','c','d','e'};
        for (char ch:chars){
            System.out.print(ch+"\t");
        }
    }

    public static void main(String args[]) throws InterruptedException {
        LockStudy lockStudy = new LockStudy();
        new Thread(()-> {
            try {
                lockStudy.read1();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"线程1").start();

        new Thread(()->{
            try {
                lockStudy.read2();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"线程2").start();

        Thread.sleep(1000);//让上述线程执行

        new Thread(()-> {
            lockStudy.read3();
        },"线程3").start();

    }

}
```

>线程1开始执行：1	2	3	4	5	6	7	8	9	10	
>线程3开始执行：a	b	c	d	e	
>线程2开始执行：11	22	33	44	55	//线程2是最后执行，因为使用的是同一对象锁



### 三、CopyOnWriteArrayList/Set介绍

由于老一代的线程类的加锁粒度限制于方法，仍然存在着多线程安全的问题，继而涌现出了

Collections.synchronized...(... ...)等方法加锁粒度更小，直接限制于对象的实例，但是这又出现了性能上的问题，

因为Collections.synchronized...(... ...)等方法，把对应的容器类的每一个操作方法的内部都用上了锁。于是，Java

就在juc包下推出线程安全的容器类。



##### CopyAndWrite简称cow，其定义是：

> 如果多个调用者(callers)同时请求相同的资源(如内存或磁盘上的数据存储)，它们会获取同一个指向相同资源
>
> 的指针，如其中某一个调用者**试图修改资源内容**时，系统会拷贝一份资源的**专用副本(private copy**)给调用
>
> 者，修改资源的调用者会在专用副本上修改资源，而其他调用者维护的指针指向的资源是保持不变的。调用
>
> 者修改资源后，将修改的资源更新到原来的资源。**优点**是如果调用者**没有修改该资源，就不会有副本**
>
> （private copy）被建立，因此多个调用者只是读取操作时可以**共享同一份资源**。



##### CopyOnWriteArrayList的源码简介

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final long serialVersionUID = 8673264195747942595L;

    //可从如梭对象
    final transient ReentrantLock lock = new ReentrantLock();

  	//不可被序列化，对所有线程可见
    private transient volatile Object[] array;  //存放元素的数据

    final Object[] getArray() { //获取数组
        return array;
    }

	//设置数组
    final void setArray(Object[] a) {
        array = a;
    }
    //......
}
```

CopyOnWriteArrayList的底层数据结构和ArrayList一样都是数组，使用可重入锁ReentrantLock的实例对象显示

加锁，其允许数组值为null。

##### CopyOnWriteArrayList的修改数组的方法

1. `boolean add(E e)`/`boolean add(int i,E e)`

   ```java
   public boolean add(E e) {
       final ReentrantLock lock = this.lock; //获取锁
       lock.lock(); //上锁
       try {
           Object[] elements = getArray(); //获取数组的引用
           int len = elements.length;
           Object[] newElements = Arrays.copyOf(elements, len + 1); //拷贝数组
           newElements[len] = e; //拷贝的数组添加元素
           setArray(newElements); //修改数组
           return true;
       } finally {
           lock.unlock(); //解锁
       }
   }
   
   public void add(int index, E element) {
       final ReentrantLock lock = this.lock;
       lock.lock();
       try {
           Object[] elements = getArray();
           int len = elements.length;
           if (index > len || index < 0) //索引检查
               throw new IndexOutOfBoundsException("Index: "+index+
                                                   ", Size: "+len);
           Object[] newElements;
           int numMoved = len - index;
           if (numMoved == 0)
               newElements = Arrays.copyOf(elements, len + 1);
           else {
               newElements = new Object[len + 1];
               //拷贝数组
               System.arraycopy(elements, 0, newElements, 0, index);
               System.arraycopy(elements, index, newElements, index + 1,
                                numMoved);
           }
           newElements[index] = element; //添加元素
           setArray(newElements); //设置数组
       } finally {
           lock.unlock();
       }
   }
   ```

2. `void clear()`

   ```java
   public void clear() {
       final ReentrantLock lock = this.lock;
       lock.lock();
       try {
           setArray(new Object[0]);
       } finally {
           lock.unlock();
       }
   }
   ```

3. `E remove(int index)`

   ```java
   public E remove(int index) {
       final ReentrantLock lock = this.lock;
       lock.lock();
       try {
           Object[] elements = getArray();
           int len = elements.length;
           E oldValue = get(elements, index);
           int numMoved = len - index - 1;
           if (numMoved == 0)
               setArray(Arrays.copyOf(elements, len - 1));
           else {
               Object[] newElements = new Object[len - 1];
               System.arraycopy(elements, 0, newElements, 0, index);
               System.arraycopy(elements, index + 1, newElements, index,
                                numMoved);
               setArray(newElements);
           }
           return oldValue;
       } finally {
           lock.unlock();
       }
   }
   ```

4. `E set(int index, E element)`

   ```java
    public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            E oldValue = get(elements, index);
   
            if (oldValue != element) {
                int len = elements.length;
                Object[] newElements = Arrays.copyOf(elements, len);
                newElements[index] = element;
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
   ```


5. `int size()/E get(int index)`

   ```java
   public int size() {
       return getArray().length;
   }
   
   public E get(int index) {
       return get(getArray(), index);
   }
   
    private E get(Object[] a, int index) {
           return (E) a[index];
       }
   ```

可以看出CopyOnWriteArrayList类修改数组的操作都是先显示加锁，然后拷贝数组一份，在拷贝的数组上修改数

据后，在将拷贝的数组跟新到原来的数组中。而读取的操作时不用加锁的，直接在存储元素的数组上操作。



##### CopyOnWriteArrayList的for-Each

在前面的快速失败机制原理的讲解中，可以了解到Java的List，Map，Set在迭代器迭代的过程中，多线程对容器

类里的数据进行修改时，容易会造成快速失败事件。传送门：<a href="">快速失败机制讲解</a>

```java
public static void main(String args[]){
    ArrayList<Integer> list=new ArrayList<>();
    list.add(3);list.add(8);list.add(5);list.add(4);
    list.add(2);list.add(6);list.add(7);list.add(9);
    list.add(1);

    for(int num:list){   //反编译后 使用的是迭代器遍历
        list.remove((Integer)2);
    }

    //或者
    new Thread(()->{
        add((int)Math.random()*10,list);
    }).start();

    new Thread(()->print(list)).start();

}

static void add(int e,List<Integer> list){
    list.add(e);
}

static void print(List<Integer> list){
    Iterator<Integer> iterator=list.iterator();
    while(iterator.hasNext()){
        System.out.print(iterator.next()+"\t");
    }
}
//上述代码就很容易造成快速失败事件，抛出ConcurrentModificationException异常。
```

而CopyOnWriteArrayList就不会造成快速失败事件，直接抛出异常，源码解析：

```java
public Iterator<E> iterator() {
    return new COWIterator<E>(getArray(), 0);
}

static final class COWIterator<E> implements ListIterator<E> {

    private final Object[] snapshot; //数组的引用

    private int cursor; //数据遍历的下标

    private COWIterator(Object[] elements, int initialCursor) {
        cursor = initialCursor;
        snapshot = elements; //源数组
    }

    //........

    public void remove() {
        throw new UnsupportedOperationException();
    }

    public void set(E e) {
        throw new UnsupportedOperationException();
    }

    public void add(E e) {
        throw new UnsupportedOperationException();
    }

    //....
}
```

由源码可以，CopyOnWriteArrayList的内部实现的迭代器遍历的是原数组，且不支持修改数组的操作，既会抛出

其他异常，而不会造成快速失败事件。



##### CopyOnWriteArraySet

CopyOnWriteArraySet的原理就是CopyOnWriteArrayList。其底层为会一个CopyOnWriteArrayList的实例对象。

其元素为null，但不可重复。

```java
public CopyOnWriteArraySet() {
    al = new CopyOnWriteArrayList<E>();
}
public boolean add(E e) {
    return al.addIfAbsent(e);
}
public boolean addIfAbsent(E e) { //不存在则加入
    Object[] snapshot = getArray();
    return indexOf(e, snapshot, 0, snapshot.length) >= 0 ? false :
    addIfAbsent(e, snapshot);
}
```



##### 总结：

* CopyOnWriteArrayList是线程安全的类

* 牺牲性能和数据的实时的一致性换取安全性，但同时用空间换时间的策略，尽可保证性能(数据的修改都是先

  拷贝数组，数组越大，消耗的性能越大)

* 适用于多线程下读多写少的情况

* 只保证数据最终的一致性，不保证实时的一致性。(线程A在拷贝的数组上正在修改数组(已修改部分数据)，线

  程B开始读取源数组上的数据,数据不一致)

* 元素允许为null，可重复

​		

参考资料:

https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484380&idx=1&sn=c1aa61d29818005f886c61575d2a5d36&chksm=ebd742dddca0cbcb4c8d5792cccfb774a268e64d2b65a94dd0781c17a73bfadf4f930c33f4be&scene=21###wechat_redirect

