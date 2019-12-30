## LinkedHashMap深入源码学习

### 一、LinkedHashMap的特点

* 继承HashMap

* 数据结构是哈希表+双链表
* 允许key和value为null，非线程安全，快速失败机制
* 数据是有序的，因为底层使用了双链表，遍历时使用的链表遍历
* 装载因子和初始容量对LinkedHashMap影响是很大

### 二、LinkedHashMap的数据结构图和结构图

**结构继承图**：

![HashMap继承图](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/LinkeddHashMap.png)

**LinkedHashMap数据结构图**：

![LinkedHashMap数据结构图](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/LinkeddHashMap.png)

图片来源与参考资料



### 二、LinkedHashMap结构

**1、成员**

```java
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>{
    transient LinkedHashMap.Entry<K,V> head; //双链表的头结点
    transient LinkedHashMap.Entry<K,V> tail; //尾部结点
	//是否使用LUC——最近经常使用算法，也就是对链表的顺序做出修改，把经常是有的放在链表的后面，节省查找时间
    final boolean accessOrder;
    
    //内部链表结点的实现,继承了HashMap的内部结点，有三个指针，next，before，after
     static class Entry<K,V> extends HashMap.Node<K,V> { 
        Entry<K,V> before, after; 
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
}
```

**2、部分构造函数**

```java
public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}
public LinkedHashMap(int initialCapacity, float loadFactor,boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

/**
1.旧表为null，则计算阈值，
2.旧表不为空，且map的个数大于旧表的阈值，则散列
3.将map上的数据移到哈希表上
*/	
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { //哈希表为空
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t); //计算2的幂次方的数
        }
        else if (s > threshold) //大于阈值
            resize(); //哈希散列
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) { //迭代存放如哈希表中
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

**3、添加  `V put(K key,V value)`**

开篇讲述了Linked继承了HashMap且之重写了部分的方法，重写方法中没有putval方法，即LinkedHashMap直接

使用HashMap的put方法。没有看过HashMap源码的同学，请进入传送门<a href="">HashMap深入源码学习</a>。

当使用到父类的putVal方法创建结点newNode(....)时，使用的是子类(LinkedHashMap)的newNode方法。

```java
//父类HashMap的方法
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0) //哈希表为空，则哈希散列
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) //判断计算出来的存放位置上是有有元素
        tab[i] = newNode(hash, key, value, null); //没有则添加
    //.......
}
//LinkedHashMap的方法
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p = new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}
```

为啥会到用过子类的方法呢？这就是Java里面的就近原则。当子类与父类有相同的签名方法时，就近原则调用子类

的方法。

```java
public class Main {

    public void test(){
        example();
    }

    public void example(){
        System.out.println("this is father's method");
    }
}

class MainSon extends Main{

    @Override
    public void example() {
        System.out.println("this is son's method");
    }

    public void check(){
        test();
    }
    
    public static void main(String args[]){
        new MainSon().check();
    }
}
//输出结果 ：this is son's method
```

**4、获取`V get(Object key)`**

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)  //使用父类HashMap的getNode方法
        return null;
    if (accessOrder) //是否允许使用LUR，是的话，将该节点放到双链表的尾部，有兴趣的同学可以自行测试
        afterNodeAccess(e);
    return e.value;
}

void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p = (LinkedHashMap.Entry<K,V>)e;
        LinkedHashMap.Entry<K,V> b = p.before, a = p.after;
        p.after = null;  //目标的结点的后继结点为null
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p; //tail重新只会尾部结点
        ++modCount;
    }
}
```

LinkedHas和Map的LUR最LinkedHashMap并没有多大的作用，因为LinkedHashMap的获取结点的方式仍是使用

父类HashMap的getNode方法——从表头向链表尾查找的,而且LinkedHashMap也没有其他的方法使用到。

有兴趣的同学可以去查看LURMap也就是LinkedHashMap的扩展：

<a href="https://blog.csdn.net/exceptional_derek/article/details/11713255">如何用LinkedHashMap实现LRU缓存算法</a>

<a href="https://www.php.cn/java-article-362041.html">详解LinkedHashMap如何保证元素迭代的顺序</a>

<a href="https://www.jianshu.com/p/1a66529e1a2e">LinkedHashMap实现LRU原理解析</a>

<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ%3D%3D&chksm=ebd639d5dca1b0c3ba5a26bd46d265544f4fdd468df6465e54d93da230c3457d4947e79eaf0c&idx=1&mid=2247485177&sn=93cfa2c2e6f3e5092e5850bdb5ea4cc3">集合系列—LinkedHashMap源码分析</a>



**5、`E remove(Object key)`**

LinkedHashMap也没有重写父类HashMap的方法，所以仍是使用父类的remove(Object key)方法,但是

LinkedHashMap重写了父类的`afterNodeRemoval(Node<K,V> e)`方法

```java
//HashMap的方法
final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
    //........
    if (node != null && (!matchValue || (v = node.value) == value ||
                                     (value != null && value.equals(v)))) {
        if (node instanceof TreeNode)
            ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
        else if (node == p)
            tab[index] = node.next;
        else
            p.next = node.next;
        ++modCount;
        --size;
        afterNodeRemoval(node); //LinkedHashMap重写的方法
        return node;
    }
}

void afterNodeRemoval(Node<K,V> e) { //在链表中删除结点
    LinkedHashMap.Entry<K,V> p =(LinkedHashMap.Entry<K,V>)e；
    LinkedHashMap.Entry<K,V> b = p.before, a = p.after;
    p.before = p.after = null; 
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

**`6、Set<Map.Entry> KeySet() `**

```java
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new LinkedKeySet();
        keySet = ks;
    }
    return ks;
}

final class LinkedKeySet extends AbstractSet<K> {
	//.....
    public final Iterator<K> iterator() {
        return new LinkedKeyIterator();
    }
    //....
}

final class LinkedKeyIterator extends LinkedHashIterator implements Iterator<K> {
    public final K next() { 
        return nextNode().getKey();
    }
}

abstract class LinkedHashIterator {
    LinkedHashMap.Entry<K,V> next,current;
    int expectedModCount;

    LinkedHashIterator() {
        next = head; //双链表的表头
        expectedModCount = modCount;
        current = null;
    }

    public final boolean hasNext() {
        return next != null;
    }

    final LinkedHashMap.Entry<K,V> nextNode() {
        LinkedHashMap.Entry<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        current = e;
        next = e.after;
        return e;
    }

    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException(); //使用父类的删除方法。
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
}
```

LinkedHashMap的迭代器维护底层双链表的表头，在遍历时使用的链表的遍历。而双链表上结点是哈希表上的结点。



参考资料：<a herf="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484143&idx=1&sn=8d94f92f26b87dcea9181a04c78a9743&chksm=ebd743eedca0caf84ede91e6cd295664f92d2b167068d4c4a15f8cb4002844b819378f1e8e3d&scene=21###wechat_redirect">LinkedHashMap就这么简单【源码剖析】</a>
