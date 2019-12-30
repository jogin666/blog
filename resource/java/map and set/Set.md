## Set深入源码学习

### 一、Set类的子类及其特点

1. HashSet

   - 底层数据结构为哈希表+链表/红黑树
   - 非同步（线程不安全），不保证有序
   - 元素不能重复，允许为null

   > 注：HashSet的底层实现是HashMap，HashSet存储的元素为HashMap的key值，所有的value都是同一个Object的实例。

2. TreeSet

   - 底层数据结构为红黑树
   - 非同步，保证有序
   - 元素不能重复，且不能为null

   > 注：TreeSet的底层实现是TreeMap。TreeSet存储的元素为TreeSet的key值，所有的value都是同一个Object的实例。

3. LinkedHasgSet

   - 底层数据接口是哈希表（元素为链表的数据）+双链表实现的。
   - 非同步，保证有序
   - 元素不能重复，允许为null
   - 迭代时，内部使用的是双链表的迭代

   > 注：LinkedHashSet的底层实现是LinkedHashMap，LinkedHashSet存储的元素为LinkedHashSet的key值，所有的value都是同一个Object的实例。



### 二、HashSet介绍

**2.1、继承图**

![](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/HashSet%E7%BB%A7%E6%89%BF%E5%9B%BE.png)

**2.2 、HashSet的结构**

- 构造函数与成员

```java
public class HashSet<E> extends AbstractSet<E>
                implements Set<E>, Cloneable, java.io.Serializable{
	//HashMap的value                                   
	 private static final Object PRESENT = new Object();
     //底层实现是HashMap
	 private transient HashMap<E,Object> map;
    
     public HashSet() {
        map = new HashMap<>();
    }
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
    //.......
}
```

- 部分常用方法

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;  //保证所有的value都是相同的Object实例
}
public void clear() {
    map.clear();
}
public boolean isEmpty() {
    return map.isEmpty();
}
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
//......
```

> 注：可以看出，HashSet实际上就是封装了HashMap，操作HashSet元素实际上就是操作HashMap。

对HashMap还不太了解的话，传送门：<a href="https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/HashMap.md">HashMap深入源码学习</a>



### 三、TreeSet介绍

**3.1 继承图**

![TreeSet的继承图](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/TreeSet%E7%9A%84%E7%BB%A7%E6%89%BF%E5%9B%BE.png)

**3.2、TreeSet的结构**

- 成员与构造函数

```java
public class TreeSet<E> extends AbstractSet<E>
    				implements NavigableSet<E>, Cloneable, java.io.Serializable{
     
    private transient NavigableMap<E,Object> m;  //实际操作的就是TreeMap
	private static final Object PRESENT = new Object();//所有的value都是相同的Object实例

    TreeSet(NavigableMap<E,Object> m) {
        this.m = m;
    }
    public TreeSet(Comparator<? super E> comparator) {
       this(new TreeMap<>(comparator));
    }
    public TreeSet(SortedSet<E> s) {
        this(s.comparator());
        addAll(s);
    }
}
```

- 部分常用方法

```java
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
public  boolean addAll(Collection<? extends E> c) {

    if (m.size()==0 && c.size() > 0 && c instanceof SortedSet && m instanceof TreeMap) {
        SortedSet<? extends E> set = (SortedSet<? extends E>) c;
        TreeMap<E,Object> map = (TreeMap<E, Object>) m;
        Comparator<?> cc = set.comparator();
        Comparator<? super E> mc = map.comparator();
        if (cc==mc || (cc != null && cc.equals(mc))) {
            map.addAllForTreeSet(set, PRESENT);
            return true;
        }
    }
    return super.addAll(c);
}
public Comparator<? super E> comparator() {
    return m.comparator();
}
public Iterator<E> iterator() {
    return m.navigableKeySet().iterator();
}
```

> 可以知道：TreeSet实际上就是封装了TreeSet，操作TreeSet元素实际上就是操作TreeMap。

对TreeMap还不太了解的话，传送门：<a href="https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/TreeMap.md">TreeMap深入源码学习</a>



### 四、LinkedHashSet介绍

**4.1、继承图**

![LinkedHashSet继承图](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/LinkedHashSet%E7%BB%A7%E6%89%BF%E5%9B%BE.png)

**4.2、LinkedHashSet源码**

```java
public class LinkedHashSet<E> extends HashSet<E>
    					implements Set<E>, Cloneable, java.io.Serializable {

    private static final long serialVersionUID = -2851667679971038690L;

    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }

    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }

    public LinkedHashSet() {
        super(16, .75f, true);
    }

    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }

    @Override
    public Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 
        							Spliterator.DISTINCT | Spliterator.ORDERED);
    }
}

//super(....)方法
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
     map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

> 可以知道：LinkedHashSet实际上就是封装了LinkedHashMap，操作LinkedHashSet元素实际上就是操作LinkedHashMap。

对LinkedHashMap还不太了解的话，传送门：<a href="https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/LinkedHashMap.md">LinkedHashMap深入源码学习</a>

