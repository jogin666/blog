## HashMap 介绍

### 一、HashMap特点：

* 允许键和值都可以为null，不保证有序（散列表）

* 初始容量默认为16，加载因子为0.75f (默认哈希表存储量占总容量的75%时，需要扩容)

  注：初始容量要适中，如果过大，查找速度变慢；若果过小，需要多次再散列。装载因子也要适中，初始值大

  了，散列次数减少，但发生哈希冲突的可能性增大；过小，发生冲突的可能性减少，散列次数增多。

* 底层数据结构是哈希表    数组+红黑树/链表+链地址法。

  注：Java的实现的哈希表在本质是在一块连续的地址控件上存放数据(数组)，每一次存放数据的地址(数组下标)

  都是根据指定的算法hash()计算出来的，不是顺序存放的。如果哈希值是相同的，则判断地址空间中是否已经

  存放了该元素，如果已经存放了，丢弃当前元素，反之则使用链表的结构，在该地址空间存放元素。

  HashMap的列上的**链表的长度超过8时且哈希表的大小超过64**，会自动将链表转成红黑树，当红黑树的结点

  个数小于等于6时，会自动退化成链表。

* 底层数组的大小必须是2的幂次方

* 多线程不安全(采用快速失败机制)

### 二、 继承图

![HashMap继承图](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/HashMap%E7%BB%A7%E6%89%BF%E5%9B%BE.png)

### 三、HashMap源码介绍

1. 成员介绍

   ```java
   	public class HashMap<K,V> extends AbstractMap<K,V>
           			implements Map<K,V>, Cloneable, Serializable {
      
      //常量
   		//默认的容量        			
           static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
   		//最大容量
           static final int MAXIMUM_CAPACITY = 1 << 30;
   		//默认的装载因子
           static final float DEFAULT_LOAD_FACTOR = 0.75f;
   		//黑红树阈值 链表超过8个结点 编程红黑树
           static final int TREEIFY_THRESHOLD = 8;
   		//链表阈值  红黑树的结点减少到6个结点时  退化成链表
           static final int UNTREEIFY_THRESHOLD = 6;
   
   		//同转化为属性结构的最小容量（？？）
           static final int MIN_TREEIFY_CAPACITY = 64;
           
    /**************       ****************************/
           transient Node<K,V>[] table;   //哈希表
            
           transient Set<Map.Entry<K,V>> entrySet;		//键值对集合
            
           transient int size;	//元素个数
            
           transient int modCount;	//fast-fail
            
           final float loadFactor;	//加载因子
            
           int threshold;	//在散列的阈值 
           
           //........
       }
   ```
   

   
2. 部分构造函数介绍

   参数为Map类型的构造函数

   ```java
   public HashMap(int initialCapacity, float loadFactor) {
       if (initialCapacity < 0) 
           throw new IllegalArgumentException("Illegal initial capacity: " +
                                              initialCapacity);
       if (initialCapacity > MAXIMUM_CAPACITY)
           initialCapacity = MAXIMUM_CAPACITY;
       if (loadFactor <= 0 || Float.isNaN(loadFactor))
           throw new IllegalArgumentException("Illegal load factor: " +
                                              loadFactor);
       this.loadFactor = loadFactor;
       this.threshold = tableSizeFor(initialCapacity); //散列阈值
   }
   
   static final int tableSizeFor(int cap) {	//返回一个最接近参数且大于参数的2的幂次方的值
       int n = cap - 1;
       n |= n >>> 1; //无符号右移1位
       n |= n >>> 2;
       n |= n >>> 4;
       n |= n >>> 8;
       n |= n >>> 16;
       return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
   }
   ```

   值得注意的是：阈值被赋值为最接近指定初始容量且大于初始容量的2的幂次方的值，是给初始化哈希表的大小准备的

   

   

   参数为Map类型的构造函数

   ```java
   //参数为Map的构造构造函数
   public HashMap(Map<? extends K, ? extends V> m) {
       this.loadFactor = DEFAULT_LOAD_FACTOR;
       putMapEntries(m, false);
   }
   
   /**
   1.旧表为null，则计算阈值，
   2.旧表不为空，且map的个数大于旧表的阈值，则散列
   3.将map上的数据一一哈希表上
   */																//evict  驱逐
   final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
       int s = m.size();
       if (s > 0) {
           if (table == null) {  //哈希表没有初始化
               float ft = ((float)s / loadFactor) + 1.0F;
               int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                        (int)ft : MAXIMUM_CAPACITY);
               if (t > threshold)
                   threshold = tableSizeFor(t);	//计算阈值
           }
      else if (s > threshold)  //哈希表不为空且元素个数大于阈值
      		resize();  //再散列
   
           for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
               K key = e.getKey();
               V value = e.getValue();
               putVal(hash(key), key, value, false, evict);
           }
       }
   }
   
   /**再散列
   1. 若旧表有数据，则判断旧表的数据容量是否小于整数的最大值，如果小于则新表的容量取旧表容量的两倍，
      然后如果旧表的容量大于等于16，且新表的容量小于最大值，则将新表的阈值设置成旧表阈值的两倍
      
   2.如果旧表为空且旧表的阈值不为0，则将新表的容量设置成一个最接近指定初始容量且最接近初始容量2的幂次方 
   
   3.旧表为空且阈值<=0,将新表的容量设置成默认容量和计算新表的阈值
   
   4. 再次判断新表的阈值是否为0，若是，则计算新表的阈值 threshold=capacity*loadFactory
   
   5.如果旧表有数据，将旧表上的数据移到新表上
     5.1 判断子表是否红黑树的节点，如是，使用红黑树的方法
     5.2 如果是链表结点则根据结点的哈希值与旧表容量的与运算后，结果是否为0，判断该节点是否会偏移
     偏移产生的原因是：在旧表存放的数据时，计算哈希值的是这样的
     (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); key得到哈希值与哈希值右移16位，得  	到高16位后做异或运算，才返回给结点存放的哈希地址。原因在hash(Object key)方法介绍
   */
   final Node<K,V>[] resize() {
       Node<K,V>[] oldTab = table;
       int oldCap = (oldTab == null) ? 0 : oldTab.length;
       int oldThr = threshold;
       int newCap, newThr = 0;
   
       if (oldCap > 0) {
           if (oldCap >= MAXIMUM_CAPACITY) { //已是最大值，无法再散列
               threshold = Integer.MAX_VALUE;  //整数最大值
               return oldTab;
           }
           else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                   oldCap >= DEFAULT_INITIAL_CAPACITY) //容量大于等于默认容量且翻倍小于整数最大
   
               newThr = oldThr << 1; // 新阈值为老阈值的双倍 新的哈希表容量也是旧的双倍
       }
       else if (oldThr > 0)
           newCap = oldThr;  //如果元哈希表容量<=0,赋予计算出来的2的幂次方的大小
       else {              
           newCap = DEFAULT_INITIAL_CAPACITY;	
           newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); 
       }
       if (newThr == 0) {
           float ft = (float)newCap * loadFactor; //重新赋予阈值
           //短路运算
           newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                     (int)ft : Integer.MAX_VALUE);
       }
       threshold = newThr;
       @SuppressWarnings({"rawtypes","unchecked"})
       Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
       table = newTab;
       if (oldTab != null) {
           for (int j = 0; j < oldCap; ++j) {
               Node<K,V> e;
               if ((e = oldTab[j]) != null) {
                   oldTab[j] = null; 
                   if (e.next == null)	//只用一个节点
                       newTab[e.hash & (newCap - 1)] = e; //计算哈希值与存放
                   
                   else if (e instanceof TreeNode) //红黑树节点
                       ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
   
                   else { // 链表
                       Node<K,V> loHead = null, loTail = null;
                       Node<K,V> hiHead = null, hiTail = null;
                       Node<K,V> next;
                       do {
                           next = e.next;
    						//将原有的链表拆分
                           if ((e.hash & oldCap) == 0) {
                               if (loTail == null)
                                   loHead = e;
                               else
                                   loTail.next = e;
                               loTail = e;
                           }
                           else {  //哈希值位移
                               if (hiTail == null)
                                   hiHead = e;
                               else
                                   hiTail.next = e;
                               hiTail = e;
                           }
                       } while ((e = next) != null);
                       if (loTail != null) { //存放新的链表
                           loTail.next = null;
                           newTab[j] = loHead;
                       }
                       if (hiTail != null) {
                           hiTail.next = null;
                           newTab[j + oldCap] = hiHead; //产生位移的链表
                       }
                   }
               }
           }
       }
       return newTab;
   }
   ```

   链表拆分解释：内容引自美团点评技术博客。新表散列之后容量是旧表的两倍，所以原旧表上的数据在新表上

   的存放位置要么不变，要么在相对原位置偏移2次幂的位置。如下表图解释：n为table的容量大小，图(a)表示

   扩容前的key1和key2确定索引的过程**(哈希值与表的大小做与运算）**，图(b)表编示的是扩容后(n的扩大两倍）

   的确定索引的过程。

   ![img](https://upload-images.jianshu.io/upload_images/5679451-a8f2e1917e6c188a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

   ​							                                图1

   因为n变为2倍，那么n-1的mask范围在高位多了1bit(红色)，因此索引需要这样确定：

   ![img](https://upload-images.jianshu.io/upload_images/5679451-b2d09c67373f217a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1064/format/webp)

   因此，在扩充HashMap的时候，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引

   没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图 ：

   ![img](https:////upload-images.jianshu.io/upload_images/5679451-d193fbb84c53dd09.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

   以上解析内容来源于：<a href="https://www.jianshu.com/p/ee0de4c99f87">一文读懂HashMap</a>

   

3. 添加  `V put(K key,V value)`

   ```java
   public V put(K key, V value) {
       return putVal(hash(key), key, value, false, true);
   }
   
   /*计算哈希码
   将key得到的哈希值h和h右移16位得到最高位最与运算的原因是在计算e存放的地址公式为
   tab[i = (n - 1) & hash]，n-1是存放地址的范围，列如哈希表的大小为16，存放方位为0-15。由上面的图1可以知道往往只和计算的哈希值后4位有关，这样就很大程度上会造成哈希冲突，在计算哈希值时，将哈希值与哈希值右移16位后，高位和地位做与运算，这就增加了随机性，减少了碰撞冲突的可能性！。 
   */
   static final int hash(Object key) {
       int h;
       return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   }
   
   /*存放元素
   1. 哈希表若为空，则哈希散列
   2.n-1与key计算出来的哈希值做与运算，检测是否发生哈希冲突，不发生则存放元素
   3.发生哈希冲突，检测链表是否只有一个节点，是，则连在在其之后
   4.不是只有一个节点，则判断是否为红黑树节点，是则调用树的方法实现添加元素
   5.不是，则遍历链表，在遍历时，检测key是否链表上的结点一致，是，退出循环，更新结点的值。不是则添加在链表的最后面，然后检测是否转成红黑树。
   6.判定哈希表的大小是否大于阈值，大于则需要哈希散列
   */
   final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                  boolean evict) {
       Node<K,V>[] tab; Node<K,V> p; int n, i;
       if ((tab = table) == null || (n = tab.length) == 0) //哈希表为空，则哈希散列
           n = (tab = resize()).length;
       if ((p = tab[i = (n - 1) & hash]) == null) //判断计算出来的存放位置上是有有元素
           tab[i] = newNode(hash, key, value, null); //没有则添加
       
       //发生哈希冲突
       else {
           Node<K,V> e; K k;
           //key与地址链表上的头结点的key是否相等
           if (p.hash == hash &&       
               ((k = p.key) == key || (key != null && key.equals(k))))
               //若key值相同，则替换元素
               e = p;
           else if (p instanceof TreeNode) //是否为红黑树结点
               e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
           else {
               for (int binCount = 0; ; ++binCount) {
                   if ((e = p.next) == null) {
                       p.next = newNode(hash, key, value, null);
                       if (binCount >= TREEIFY_THRESHOLD - 1) //是否大于结点长度阈值
                           treeifyBin(tab, hash); //链表要转变为红黑树
                       break;
                   }
                   //链表上存在相同的key，退出循环
                   if (e.hash == hash &&
                       ((k = e.key) == key || (key != null && key.equals(k))))
                       break;
                   p = e;
               }
           }
           if (e != null) { //可以相同退出循环，  新的元素替换旧的元素
               V oldValue = e.value;
               if (!onlyIfAbsent || oldValue == null)
                   e.value = value;
               afterNodeAccess(e);
               reurn oldValue;
           }
       }
       ++modCount;
       //哈希表的大小是否超过阈值，若是，则哈希散列
       if (++size > threshold)
           resize();
       afterNodeInsertion(evict);
       return null;
   }
   ```

4. 获取  `V get(Object key)`

   ```java
   public V get(Object key) {
       Node<K,V> e;
       return (e = getNode(hash(key), key)) == null ? null : e.value;
   }
   /*
   1.若哈希表不为空，则根据哈希值取头结点，判断头结点是否符合条件，哈希表为空则返回null，结束。
   2.若头结点不符合条件，判定头结点是否有下一个结点，没有返回null，结束。
   3.取头结点的下一个结点，判断该节点是否为红黑树结点，是则调用红黑树的方法查询
   4.不是红黑树的结点，则开始遍历链表查询是否有符合条件的，有就返回，没有则最终返回null；
   */
   final Node<K,V> getNode(int hash, Object key) {
       Node<K,V>[] tab; 
       Node<K,V> first, e;
       int n; K k;
       //哈希表不为空，则根据哈希值取头结点
       if ((tab = table) != null && (n = tab.length) > 0 &&
           (first = tab[(n - 1) & hash]) != null) { 
          
           if (first.hash == hash &&   //头结点是否为要取的结点
               ((k = first.key) == key || (key != null && key.equals(k))))
               return first;
           if ((e = first.next) != null) { //头结点是否有下一个
               
               if (first instanceof TreeNode) //头结点是否为红黑树结点
                   return ((TreeNode<K,V>)first).getTreeNode(hash, key);
               //遍历链表
               do {
                   if (e.hash == hash &&
                       ((k = e.key) == key || (key != null && key.equals(k))))
                       return e;
               } while ((e = e.next) != null);
           }
       }
       return null;
   }
   ```

5. 移除  `V remove(Object key)`

   ```java
   public V remove(Object key) {
       Node<K,V> e;
       return (e = removeNode(hash(key), key, null, false, true)) == null ?
           null : e.value;
   }
   
   /*
   1. 哈希表为空返回null，结束
   2. 取到头结点后，判定头结点是否是要删除的结点
   3. 头结点不为结点，则判断头结点是否为红黑树结点，是则使用红黑树的方法查找
   4. 头不是红黑树结点，则遍历链表，查询结点
   5. 是否查到要删除的结点，是，分三种情况删除：红黑树结点，头结点，非头结点
   */
   final Node<K,V> removeNode(int hash, Object key, Object value,
                                  boolean matchValue, boolean movable) {
       Node<K,V>[] tab; 
       Node<K,V> p; 
       int n, index;
       //若表不为空，且根据哈希值可以得到结点
       if ((tab = table) != null && (n = tab.length) > 0 &&
           (p = tab[index = (n - 1) & hash]) != null) {
           
           Node<K,V> node = null, e;
           K k; V v;
           //头结点是否符合条件
           if (p.hash == hash &&
               ((k = p.key) == key || (key != null && key.equals(k))))
               node = p;
           else if ((e = p.next) != null) { //头结点有后继结点
               
               if (p instanceof TreeNode) //结点是否为红黑树结点
                   node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
               else {
                   //遍历链表
                   do {
                       if (e.hash == hash &&
                           ((k = e.key) == key ||
                            (key != null && key.equals(k)))) {
                           node = e;
                           break;
                       }
                       p = e;
                   } while ((e = e.next) != null);
               }
           }
           //找到结点之后
           if (node != null && (!matchValue || (v = node.value) == value ||
                                (value != null && value.equals(v)))) {
               if (node instanceof TreeNode) //结点是红黑树结点
                   ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
               else if (node == p)	//头结点
                   tab[index] = node.next; 
               else
                   p.next = node.next; //非头结点
               ++modCount;
               --size;
               afterNodeRemoval(node);
               return node;
           }
       }
       return null;
   }
   ```

6. 更新 `boolean replace(K key,V oldValue,V newValue)`

   ```java
   @Override
   public boolean replace(K key, V oldValue, V newValue) {
       Node<K,V> e; V v;
       if ((e = getNode(hash(key), key)) != null &&
           ((v = e.value) == oldValue || (v != null && v.equals(oldValue)))) {
           e.value = newValue;
           afterNodeAccess(e);
           return true;
       }
       return false;
   }
   ```



### 四、HashMap与HashTable的区别

>从存储结构和实现来讲基本上都是相同的。它和HashMap的最大的不同是它是线程安全的,l另外HashTable不
>
>允许key和value为null，Hashtable是个过时的集合类，不建议在新代码中使用，不需要线程安全的场合可以
>
>用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

| 不同方面         | HashMap          | HashTable                            |
| ---------------- | ---------------- | ------------------------------------ |
| 数据结构         | 数组+链表+红黑树 | 数组+链表                            |
| 父类             | AbstractMap      | Dirctionary                          |
| 是否线程安全     | 否               | 是                                   |
| 默认初始容量     | 16               | 11                                   |
| 扩容方式         | 原始容量*2       | 原始容量*2+1                         |
| 数组长度要求     | 2的幂次方        | 无                                   |
| 数组索引确定方式 | (n-1)&hash       | (hash & 0x7FFFFFFF) % tab.length     |
| 遍历方式         | Iterator(迭代器) | Iterator(迭代器)和Enumerator(迭代器) |
| 数据遍历顺序     | 索引从小到大     | 索引从大到小                         |

Hashtable具体阅读源码可参考：

- https://blog.csdn.net/panweiwei1994/article/details/77427010
- https://blog.csdn.net/panweiwei1994/article/details/77428710

HashMap与HashTable的区别的内容来源于：<a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484139&idx=1&sn=bb73ac07081edabeaa199d973c3cc2b0&chksm=ebd743eadca0cafc532f298b6ab98b08205e87e37af6a6a2d33f5f2acaae245057fa01bd93f4&scene=21###wechat_redirect">HashMap就是这么简单【源码剖析】</a>





参考资料：

[HashMap就是这么简单【源码剖析】](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484139&idx=1&sn=bb73ac07081edabeaa199d973c3cc2b0&chksm=ebd743eadca0cafc532f298b6ab98b08205e87e37af6a6a2d33f5f2acaae245057fa01bd93f4&scene=21###wechat_redirect)

<a href="https://www.jianshu.com/p/ee0de4c99f87">一文读懂HashMap</a>

