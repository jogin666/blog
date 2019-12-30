## 集合（Collection）

1. 集合与数组的区别

   1.1 长度的区别

   ​	  	集合的长度是可变的，运行时动态增加

   ​	   数组的长度是不可变的，编译时分配好

   1.2 存储的数据类型
   
   ​		集合存储的数据类型是引用类型，基本类型会自动封装成对象类型
   
   ​		数组的存储的数据类型既可以是引用类型，也可以是基本类型
   
2. 所需要掌握的集合

   ​	  List： ArrayList，LinkedList，Vector (有序的，即取出顺序与存入的顺序一样的，可重复，允许为null)

   ​	  Set： HashSet(无序的，不能重复，允许为null)   LinkedHashSet(有序的)，TreeSet(有序的，自然排序)

   ​	  Queue： LinkedList，PriorityQueue 		(先进先出[有序的]，可重复，允许为null)

3. Collection常用的方法

   3.1 添加

   ​	  boolean add(Object obj)

   ​	  boolean addAll(Collection c);

   3.2 删除

   ​	 boolean remove(Object obj);

   ​   boolean removeAll(Collection c);

   3.3 判断

   ​	 boolean contains(Object obj)

   ​	 boolean containsAll(Collection c);

   ​   boolean isEmpty();

   3.4 迭代

   ​    Iterator  iterator()

   3.5 交集

      boolean retianAll(Collection c)	在源集合中保留源集合和集合c的交集，返回及是判源集合是否有发生变动

4.  迭代器Iterator


> Collection 继承了Iterator接口，这样每一个Collection的子接口，如List，Queue，Set就不需要各自单
>
> 独 继承了。Iterator 的实现是在每一个Collection的实现类中，由内部类实现的。
>
> Iterator的方法
>
> ​	1. boolean hasNext(); 
>
> ​    2. boolean next();
>
> ​    3.  boolean remove();

​    

   

   

   

   

   

   

   



