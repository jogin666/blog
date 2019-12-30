## TreeMap深入源码学习

### 一、TreeMap特点

* 底层数据结构为红黑树，查找时间的复杂度保证为log(n)

* key和value不能为null

* 线程不安全，快速失败机制

* 有序的，默认是按key的自然排序(类似小顶堆)

  注：因为实现了继承SortedMap接口的NavigableMap接口，提供了导航定位和导航操作的功能，

  排序的实现是Comparator或者Comparable来比较key来实现的，可以更换比较规则，传入自定
  
  义的comparactor，同时key类型必须是要实现comparactor的。
  
  

### 二、TreeMap的继承结构图

![TreeMap](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/TreeMap.png)

### 三、TreeMap的结构

##### 1、成员

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
	//排序规则
    private final Comparator<? super K> comparator;
	//红黑树的根节点
    private transient Entry<K,V> root;
	//TreeMap的大小
    private transient int size = 0;
	//快速失败机制的实现
    private transient int modCount = 0;
    //是否红色
    private static final boolean RED   = false;
    private static final boolean BLACK = true; //是否黑色
    //.......
    
    static final class Entry<K,V> implements Map.Entry<K,V> {  //红黑树的结点
        K key;
        V value;
        Entry<K,V> left;  //左结点	
        Entry<K,V> right;	//右节点
        Entry<K,V> parent;	//父节点
        boolean color = BLACK; //颜色

        Entry(K key, V value, Entry<K,V> parent) {
            this.key = key;
            this.value = value;
            this.parent = parent;
        }
		//......
    }
 }
```

**2、构造函数**

```java
public TreeMap() {
    comparator = null;
}
//自定排序规则
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
//参数为Map
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}
//参数为SortedMap
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

**3、`putAll(Map<? extends K, ? extends V> map) `**

```java
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        if (c == comparator || (c != null && c.equals(comparator))) {
            ++modCount;
            try {
                buildFromSorted(mapSize, map.entrySet().iterator(),
                                null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }
    super.putAll(map); //最终父类遍历使用子类(TreeMap)的put(Object key,Object Val)方法存放元素
}

private void buildFromSorted(int size, Iterator<?> it, java.io.ObjectInputStream str,
                                 V defaultVal) throws  java.io.IOException, 
					ClassNotFoundException {                        
    this.size = size;
    root = buildFromSorted(0, 0, size-1, computeRedLevel(size),it, str, defaultVal);
}

private static int computeRedLevel(int sz) { //计算红色结点的层数
    int level = 0;
    for (int m = sz - 1; m >= 0; m = m / 2 - 1)
        level++;
    return level;
}

//二分查找
private final Entry<K,V> buildFromSorted(int level, int lo, int hi, int redLevel,
                  	Iterator<?> it, java.io.ObjectInputStream str,V defaultVal)
    				throws  java.io.IOException, ClassNotFoundException {

    if (hi < lo)  return null; //

    int mid = (lo + hi) >>> 1;

    Entry<K,V> left  = null;
    if (lo < mid) //递归
        left = buildFromSorted(level+1, lo, mid - 1, redLevel,it, str, defaultVal);
    K key;
    V value;
    if (it != null) {
        if (defaultVal==null) {
            Map.Entry<?,?> entry = (Map.Entry<?,?>)it.next();
            key = (K)entry.getKey();
            value = (V)entry.getValue();
        } else {
            key = (K)it.next();
            value = defaultVal;
        }
    } else { // use stream
        key = (K) str.readObject();
        value = (defaultVal != null ? defaultVal : (V) str.readObject());
    }

    Entry<K,V> middle =  new Entry<>(key, value, null);

    if (level == redLevel)
        middle.color = RED;

    if (left != null) {
        middle.left = left;
        left.parent = middle;
    }

    if (mid < hi) { //递归
        Entry<K,V> right = buildFromSorted(level+1, mid+1, hi, redLevel,
                                           it, str, defaultVal);
        middle.right = right;
        right.parent = middle;
    }

    return middle;
}
```

**4 、`V put(K key, V value)`**

```java
/**存放值
1. 若红黑树的根为空，则先判断key是否为null，不为null创建树根
2. 红黑树的树根不为空且比较器(comparator)不为null，则使用比较器比较树上结点的key与key，若相同则替换value结束，反之继续遍历树
3.若1,2都不成立，则默认使用key的自然比较，遍历红黑树判断，树上结点的key与key，若相同则替换value结束，反之继续遍历树
4. 若2或者3没有找到有相同的key，则创建结点，添加到红黑树上，添加的位置在遍历红黑树时，已经确定结点的parent，是左子树还是右子树，则由cmp的值确定。
5.最后添加上去的结点，可能破坏了红黑树的平衡性，因此需要修复红黑树的平衡性fixAfterInsertion(e)
*/
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); //强制检查key是否为空

        root = new Entry<>(key, value, null); //创建根节点
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    Comparator<? super K> cpr = comparator; 
    if (cpr != null) { //比较器不为空
        do {
            parent = t; 
            cmp = cpr.compare(key, t.key); //根据规则比较树节点的key与待插入key的大小
            if (cmp < 0)
                t = t.left; //小于遍历左子树
            else if (cmp > 0)
                t = t.right; //大于遍历右子树
            else
                return t.setValue(value); //相等替换结点值
        } while (t != null);
    }
    else {
        if (key == null) //可以不能为null
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key; //默认使用自然排序
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    Entry<K,V> e = new Entry<>(key, value, parent);  //红黑树不存在该节点，则创建结点
    if (cmp < 0) 
        parent.left = e; //左子树
    else
        parent.right = e; //右子树
    fixAfterInsertion(e); //红黑树自平衡
    size++;
    modCount++;
    return null;
}
```

红黑树平衡自平衡介绍：` fixAfterInsertion(Entry<K,V> x)`

```java
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;

    while (x != null && x != root && x.parent.color == RED) {
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    root.color = BLACK;
}
```

**5、获取：`V get(Object key)`**

```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}
/*
1.若比较器(comparator)不为空，则使用比较器比较查询，返回的值大于0则遍历比较右子树，小于0则遍历比较左子树，为0则是要获取的目标
2. 若比较器为空，则默认使用key来比较查询
3. 如果查不到，则返回null
*/
final Entry<K,V> getEntry(Object key) {
    //如果比较器comparator不为空，则使用比较器去查找，类似下面代码，只不过使用comparactor比较
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
    Comparable<? super K> k = (Comparable<? super K>) key; //比较器为空，则默认使用key比较
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}
```

**6、 删除` V remove(Object key)`**

```java
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p); //删除红黑树结点
    return oldValue;
}
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // If strictly internal, copy successor's element to p and then make p
    // point to successor.
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    if (replacement != null) {
        // Link replacement to parent
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // Fix replacement
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // return if we are the only node.
        root = null;
    } else { //  No children. Use self as phantom replacement and unlink.
        if (p.color == BLACK)
            fixAfterDeletion(p);

        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```

**7、迭代器PrivateEntryIterator**

```java
//代码跟踪，最终可知TreeMap的内部迭代器几乎都是继承PrivateEntryIterator
abstract class PrivateEntryIterator<T> implements Iterator<T> {
    Entry<K,V> next; //下一个结点
    Entry<K,V> lastReturned; //保留上一个返回的结点
    int expectedModCount; //快速错误机制

   //.....
    public final boolean hasNext() {
        return next != null;
    }

    final Entry<K,V> nextEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        next = successor(e); //获取下一个结点
        lastReturned = e;
        return e;
    }

    final Entry<K,V> prevEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        next = predecessor(e); //获取上一个结点
        lastReturned = e;
        return e;
    }

  //.....
}
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}

static <K,V> Entry<K,V> predecessor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.left != null) {
        Entry<K,V> p = t.left;
        while (p.right != null)
            p = p.right;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.left) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

> 由 successor(Entry<K,V> t)方法和 predecessor(Entry<K,V> t)两个方法可知，在红黑树的遍历中，使用的中
>
> 序遍历。successor(e)首先尝试获取e的右子树上最小/最大的结点(由comparactor决定)，也就是最近e结点的
>
> 左结点。若右子树为空，则需要回溯e的祖先结点，直到是父节点的右孩子或者p为root，返回当前直结点。
>
> predecessor(e)首先尝试获取e的左子树上最大/最小的结点，也就是最远离e的右节点。若左子树为空，则需
>
> 要回溯e的祖先结点，知道p为其父节点的左孩子或p为root。



注：红黑树是在是太难了，等哪天要啃红黑树时，再回来解释源码。我会回来的！！！
