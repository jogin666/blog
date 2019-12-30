## LinkedList

### 一、ArrayList，LinkedList，Vetor介绍

1.1 区别

 	 ArrayList底层的数据结构是数组，线程不安全，(使用了快速失败机制)

  	LinkedList底层的数据节后是双链表，线程不安全，(使用了快速失败机制)

  	Vetor的底层数据结构是数组，线程安全，使用城管(synchronized修饰增删改的方法，因此开销增大，	  性能不够好)

1.2 联系

​	都是有序，可重复，允许值为null

### 二、LinkedList源码介绍

1. 继承图

![LinkedList](https://github.com/jogin666/blog/blob/master/resource/java/%E9%9B%86%E5%90%88/images/LinkedList.jpg)

图片来源参考资料  侵删

### 三、 常用方法介绍

1. ListNode类的实现的接口和成员

   ```java
   //实现了Queue接口  拥有队列的性质
   public class LinkedList<E> extends AbstractSequentialList<E>
           implements List<E>, Deque<E>, Cloneable, java.io.Serializable{
   
           //list元素个数
           transient int size = 0;
   
           //头结点
           transient Node<E> first;  //Node<E> 是内部类
   
           //尾部结点
           transient Node<E> last;
       }
   
       /************************分割线************************/
   	private static class Node<E> {
           E item;  //结点的value
           Node<E> next;	//下一个结点
           Node<E> prev;	//上一个结点
   
           Node(Node<E> prev, E element, Node<E> next) {
               this.item = element;
               this.next = next;
               this.prev = prev;
           }
       }
   }
   ```

2. 构造函数

   ```java
   public LinkedList() {  //无参数构造函数
   }
   
   public LinkedList(Collection<? extends E> c) {  //有参数的构造函数
       this();
       addAll(c);
   }
   
   public boolean addAll(Collection<? extends E> c) {
       return addAll(size, c); //在尾部添加集合  详情见下面介绍
   }
   
   ```

3. 添加元素boolean add(E e)/boolean add(int index,E e)

   ```java
   public boolean add(E e) {
       linkLast(e);
       return true;
   }
   
   void linkLast(E e) {
       final Node<E> l = last;
       final Node<E> newNode = new Node<>(l, e, null);
       last = newNode;
       if (l == null)
           first = newNode;
       else
           l.next = newNode;
       size++;
       modCount++;
   }
   
   public void add(int index, E element) {
       checkPositionIndex(index);  //数组下标越界检查
   
       if (index == size)
           linkLast(element);  //尾部添加
       else
           linkBefore(element, node(index));
   }
   
   //查找当前位于index的结点
   Node<E> node(int index) {
       // assert isElementIndex(index);
       if (index < (size >> 1)) { 		//分割两部分，缩小范围
           Node<E> x = first;
           for (int i = 0; i < index; i++)
               x = x.next;
           return x;
       } else {
           Node<E> x = last;
           for (int i = size - 1; i > index; i--)
               x = x.prev;
           return x;
       }
   }
   
   void linkBefore(E e, Node<E> succ) {
       // assert succ != null;
       final Node<E> pred = succ.prev;
       final Node<E> newNode = new Node<>(pred, e, succ);
       succ.prev = newNode;
       if (pred == null)
           first = newNode;
       else
           pred.next = newNode;
       size++;
       modCount++;
   }
   ```

4. boolean add(Collection<E> c)/addAll(int index,Conlection<E> c) 

   ```java
   public boolean addAll(Collection<? extends E> c) {
       return addAll(size, c);
   }
   
   public boolean addAll(int index, Collection<? extends E> c) {
       checkPositionIndex(index);  //数组越界下标检查
   
       Object[] a = c.toArray();
       int numNew = a.length;
       if (numNew == 0)
           return false;
   
       Node<E> pred, succ;  //前结点 后继结点
       if (index == size) {
           succ = null;
           pred = last; //尾部结点
       } else {
           succ = node(index);
           pred = succ.prev;
       }
   
       for (Object o : a) {
           @SuppressWarnings("unchecked") E e = (E) o;
           Node<E> newNode = new Node<>(pred, e, null);
           if (pred == null) 	//链表为空时
               first = newNode;
           else
               pred.next = newNode;
           pred = newNode;
       }
   
       if (succ == null) {  //在中间插入
           last = pred;
       } else {
           pred.next = succ;
           succ.prev = pred;
       }
   
       size += numNew;
       modCount++;
       return true;
   }
   ```

5. E  get(int index)

   ```boojava
   public E get(int index) {
       checkElementIndex(index);	//数组下标越界检查
       return node(index).item;
   }
   ```

6. 移除元素  E remove() /E remove(int index)/remove(Object obj)

   ```java
   public E remove() {    //在链表表头移除元素
       return removeFirst();
   }
   
   public E removeFirst() {
       final Node<E> f = first;
       if (f == null)
           throw new NoSuchElementException();
       return unlinkFirst(f);
   }
   
   private E unlinkFirst(Node<E> f) {
       // assert f == first && f != null;
       final E element = f.item;
       final Node<E> next = f.next;  
       f.item = null;
       f.next = null; // help GC
       first = next;
       if (next == null)
           last = null;
       else
           next.prev = null;
       size--;
       modCount++;
       return element;
   }
   
   public E remove(int index) {
       checkElementIndex(index); //数据下标越界检查
       return unlink(node(index));   //node(index)  上面add(int index)已介绍
   }
   
   public boolean remove(Object o) {
       if (o == null) {
           for (Node<E> x = first; x != null; x = x.next) {
               if (x.item == null) {
                   unlink(x);
                   return true;
               }
           }
       } else {
           for (Node<E> x = first; x != null; x = x.next) {
               if (o.equals(x.item)) {
                   unlink(x);
                   return true;
               }
           }
       }
       return false;
   }
   
   E unlink(Node<E> x) {
       // assert x != null;
       final E element = x.item;
       final Node<E> next = x.next;
       final Node<E> prev = x.prev;
   
       if (prev == null) {  //没有前驱结点 判断为表头
           first = next;
       } else {
           prev.next = next;
           x.prev = null;
       }
   
       if (next == null) {  //没有后驱结点 判断为尾结点
           last = prev;
       } else {
           next.prev = prev;
           x.next = null;
       }
   
       x.item = null;
       size--;
       modCount++;
       return element;
   }
   ```

7. 更新  E set(int index,E e)

   ```java
   public E set(int index, E element) {
       checkElementIndex(index);  //数据下标越界检查
       Node<E> x = node(index);
       E oldVal = x.item;
       x.item = element;
       return oldVal;
   }
   ```

### 四、Queue接口方法介绍

​	有继承图可以知道，LinkedList是实现了Queue接口，即LinkedList既是集合，也是队列。

 1. 队列方法图

    ![Queue接口方法图](https://github.com/jogin666/blog/blob/master/resource/java/%E9%9B%86%E5%90%88/images/Queue%E6%8E%A5%E5%8F%A3%E6%96%B9%E6%B3%95%E5%9B%BE.png)


参考资料：

https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484130&idx=1&sn=4052ac3c1db8f9b33ec977b9baba2308&chksm=ebd743e3dca0caf51b170fd4285345c9d992a5a56afc28f2f45076f5a820ad7ec08c260e7d39&scene=21###wechat_redirect
