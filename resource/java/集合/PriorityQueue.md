## PriorityQueue

优先级队列：默认自然排序，可自定义排排序规则。

底层实现：数组，不允许元素为：null，默认数组大小为11，线程不安全，快速失败机制。

数据结构：堆 (默认小顶堆)

### 一、继承图

![Queue继承图](https://github.com/jogin666/blog/blob/master/resource/java/%E9%9B%86%E5%90%88/images/Queue.png)

### 二、PriorityQueue 成员

```java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {

    private static final long serialVersionUID = -7720805057305804111L;

    private static final int DEFAULT_INITIAL_CAPACITY = 11;  //默认数组大小

    transient Object[] queue; //存储元素的数组

    private int size = 0;	//实际元素个数

    private final Comparator<? super E> comparator; //比较规则
    
    transient int modCount = 0; // 快速失败机制
    
    //..................
}
```

### 三、常用方法介绍

1. 主要的构造函数 

```java

public PriorityQueue(int initialCapacity,
                     Comparator<? super E> comparator) {
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}

public PriorityQueue(Collection<? extends E> c) {
    if (c instanceof SortedSet<?>) {	//是否SortedSet
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;  
        this.comparator = (Comparator<? super E>) ss.comparator();
        initElementsFromCollection(ss);
    }
    else if (c instanceof PriorityQueue<?>) {	//是否是ProrityQueue
        PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        initFromPriorityQueue(pq);
    }
    else {		//简单的集合
        this.comparator = null;
        initFromCollection(c);
    }
}

public PriorityQueue(SortedSet<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initElementsFromCollection(c);
}

private void initFromPriorityQueue(PriorityQueue<? extends E> c) {
    if (c.getClass() == PriorityQueue.class) {
        this.queue = c.toArray();
        this.size = c.size();
    } else {
        initFromCollection(c);
    }
}

private void initFromCollection(Collection<? extends E> c) {
    initElementsFromCollection(c);
    heapify();  //排序
}

private void initElementsFromCollection(Collection<? extends E> c) {
    Object[] a = c.toArray(); 
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, a.length, Object[].class);  //转为Object[].class
    int len = a.length;
    if (len == 1 || this.comparator != null) //无法比较null
        for (int i = 0; i < len; i++)
            if (a[i] == null)
                throw new NullPointerException();
    this.queue = a;
    this.size = a.length;
}

private void heapify() {   //使用堆排序
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        siftDown(i, (E) queue[i]);
}

private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
        siftDownComparable(k, x);
}

private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {  //同一个子堆循环比较
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo((E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = key;
}

@SuppressWarnings("unchecked")
private void siftDownUsingComparator(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = queue[child];  //左堆点
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)  //左右堆点数值比较
            c = queue[child = right];	//取小的
        if (comparator.compare(x, (E) c) <= 0) //左堆点值是否小于目标
            break;	
        queue[k] = c;
        k = child;
    }
    queue[k] = x;
}
```

2. 增加  boolean add(E e)

```java
public boolean add(E e) {
    return offer(e);
}

public boolean offer(E e) {  //尾部插入元素
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);  //扩容
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e); //升序
    return true;
}

private void grow(int minCapacity) {  
    int oldCapacity = queue.length;
    // Double size if small; else grow by 50%   尽可能取最小容量
    int newCapacity = oldCapacity + ((oldCapacity < 64) ? 
                                     (oldCapacity + 2) :
                                     (oldCapacity >> 1));
    // overflow-conscious code
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    queue = Arrays.copyOf(queue, newCapacity);
}

private static int hugeCapacity(int minCapacity) {  //超大容量
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}

private void siftUp(int k, E x) {   //是否需要升序重建堆
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}

private void siftUpComparable(int k, E x) {    
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (key.compareTo((E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = key;
}

@SuppressWarnings("unchecked")
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

3. E peek( )  / poll( )

```java
@SuppressWarnings("unchecked")
public E peek() {   //获取头部元素
    return (size == 0) ? null : (E) queue[0];
}

@SuppressWarnings("unchecked")
public E poll() {   //移除并返回头部元素
    if (size == 0)
        return null;
    int s = --size;  //
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x); 
    return result;
}
```

4. 移除元素  boolean remove(Object obj) / removeAt(int index)

```java
public boolean remove(Object o) {
    int i = indexOf(o);
    if (i == -1)
        return false;
    else {
        removeAt(i);
        return true;
    }
}

private E removeAt(int i) {
    // assert i >= 0 && i < size;
    modCount++;
    int s = --size;
    if (s == i) // removed last element
        queue[i] = null;
    else {
        E moved = (E) queue[s]; //取出最后一个元素
        queue[s] = null;
        siftDown(i, moved);  //尝试在i 插入moved  然后下虑
        if (queue[i] == moved) {	//moved仍是最后一个
            siftUp(i, moved);	//执行上虑
            if (queue[i] != moved)  
                return moved;
        }
    }
    return null;
}
```




