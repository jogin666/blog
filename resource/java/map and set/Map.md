## Map深入源码学习

1. Map 简单的介绍

   ​	Map是一个存储键值的容器，也称作映射表，根据键，来获取值。

   ​	其中特点：键值对中的键是唯一的，不能重复，而值是可以重复的。

2. Map常用大方法介邵

   ```java
   /*	
   1. 添加的功能 
   V put(K key,V value) //如果key 不存在插入键值对 返回null 如果存在择替换value，返回旧值
   putAll(Map<? extends K, ? extends V> m)
    //替换
   replace(K key,V value)
   repalce(K key, V oldValue,V newValue)
   
   2. 删除功能
   V remove(K key)	//根据键移除
   void  clear()		//清除全部
   boolean remove(K key,V value)
   
   3. 判断功能
   boolean containsKey(K key)	//是否包含某个键
   boolean conatiansValue(V value)	//是否包含某个值
   boolean isEmoty()	//是否为空
   
   4. 获取功能
   Set<Map.Entry<K key,V value> entrySet()  //键值对象集合
   Set<K> keySet()	//键集合
   Collection<V> values();	//value结合
   V get(K key)	//单个值
   */
   ```

   

3. 散列介绍

   - 所谓散列表就是根据算法为每一个对象算出一个整数，称之为散列码，根据散列码，把对象存放指定的位

     置上。获取对象时，根据对象的散列码便可以快速找到对象。

   - 散列冲突。散列冲突是无法避免的，只能尽可能减少。在Jdk1.8中，数组存满的时候，链表会成红黑树

     如果散列表太满了的话，需要对散列表进行再散列(也就是扩容)，然后把原有的数据放到新的散列表中，

     丢弃原有的散列表。



-  哈希(散列)冲突解决办法：

> - 开放地址法（再散列） //例如往后移  直到有空闲的地址
> - 链地址法（拉链法 ）// 同一哈希值 通过链表连接起来
> - 再哈希法   //使用几个哈希算法
> - 创建公共溢出区



4. 平衡二叉树介绍

   平衡二叉树的出现是为了防止二叉搜索树退化成链表(只有右子树或者左子树的极端情况），当结点的个数达

   要求时，通过左旋，右旋，左左旋，右右旋的操作，将二叉树变为平衡二叉树。其特点是对于**对于任意一个**

   **节点，其左子树和右子树的高度差不能超过1**

   

5. 红黑树介绍

   红黑树出现的原因是AVL树,也就是自平衡二叉树，为了保证任意一个结点的左右子树的高度差不能超过1，需

   要自旋。而在这个自旋的过程中，需要消耗的性能过大(需要多次自旋操作)。为了保证更好的性能，于是就有

   二红黑树。

   

   红黑树的特点：

   1. 每个结点或者是黑色，或者是红色

   2. 根节点必须是黑色

   3. 每一个叶子节点是黑色的（左右结点为空的结点）

   4. 如果一个节点是红色的，那么它的孩子结点都必须是黑色的

   5. 从任意一个结点到叶子节点经过的黑色结点是一样的

      ![红黑树](https://github.com/jogin666/blog/blob/master/resource/java/map%20and%20set/images/%E7%BA%A2%E9%BB%91%E6%A0%91.webp)

      图片来源于参考资料   侵删

      

   

   

   参考资料:

   <a href="https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484135&idx=1&sn=be2221572ffc82f5792dd4ef1ea8e309&chksm=ebd743e6dca0caf00f188cabafc73665b875bf1cbe92cf3626cedb4f80313bb20a7429b8ec3f&scene=21###wechat_redirect">Map集合、散列表、红黑树介绍</a>

   

   

   

   

   

   

   

   
