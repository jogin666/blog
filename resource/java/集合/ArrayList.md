## ArrayList

​		对于List的集合类，在开发中，我们主要常使用的类是ArrayList，LinkedList，但是作为ArrayList的前身，我们仍需要了解Vetor这个类。

### 一、ArrayList，LinkedList，Vetor介绍

1.1 区别
- ArrayList底层的数据结构是数组，线程不安全，(使用了快速失败机制)

- LinkedList底层的数据节后是双链表，线程不安全，(使用了快速失败机制)

- Vetor的底层数据结构是数组，线程安全，使用城管(synchronized)修饰增删改的方法，开销增大性能不够好


1.2 联系

​	都是有序，可重复，允许值为null

### 二、ArrayList源码介绍

##### 1.继承图

![ArrayList](https://github.com/jogin666/blog/blob/master/resource/java/%E9%9B%86%E5%90%88/images/ArrayList.jpg)

图片来源参考资料，侵删

### 2.源码

##### 2.0 成员与属性

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L; //序列化ID
    
    //默认的数组大小
    private static final int DEFAULT_CAPACITY = 10;

	//在指定初始容量为0时，elementData的实际数组
    private static final Object[] EMPTY_ELEMENTDATA = {}; 
	//elementData默认的数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
	//存储元素的数组
    transient Object[] elementData; // non-private to simplify nested class access

    private int size; //数组中有效的元素个数
    
    //......
}
```



#####	 2.1  构造函数源码解释

```java
//有参数的构造函数
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;  //初始容量为0时，返回EMPTY_ELEMENTDATA
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}

//无参数的构造函数
public ArrayList() {
    // 未指定容量，默认返回的是
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

//参数为集合构造函数
public ArrayList(Collection<? extends E> c) {
    //转化为数组
    elementData = c.toArray();  
    if ((size = elementData.length) != 0) {
        // 是否为Object[]类型
        if (elementData.getClass() != Object[].class)    //数据类型转换
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 返回空数组
        this.elementData = EMPTY_ELEMENTDATA;
    }
}

//*************************************分割线*****************************************  
//Arrays.copyOf()
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> 												newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)  //数组类型是否一样
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);  //转换
    //复制一份
    System.arraycopy(original, 0, copy, 0,Math.min(original.length, newLength)); 
    return copy;
}

```



##### 3. add(E e) 

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;   
    return true;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {   //是否为默认的数组
        return Math.max(DEFAULT_CAPACITY, minCapacity);  //取最大值  10
    }
    return minCapacity;   //数组所需最大的容量
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;   //快速错误机制
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)  //判断是否需要扩容
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code   内存溢出？
    int oldCapacity = elementData.length;  
    int newCapacity = oldCapacity + (oldCapacity >> 1);  //右移一位 减少一半  即扩容1.5倍
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);  //
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);  //将原来的元素复制到新的数组，																	//抛弃就数组
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow  
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;   //   MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}
```



##### 4  add(int index,E e) 

```java
public void add(int index, E element) {
    rangeCheckForAdd(index);   //检查index 是否越界

    ensureCapacityInternal(size + 1);  // Increments modCount!!  //确定是否需要扩容
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);     //将index-1之后的数组元素后移一位 
    elementData[index] = element;  //赋值
    size++;
}

private void rangeCheckForAdd(int index) {
    if (index > size || index < 0)
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```



##### 5. addAdd(Collection c)/addAll(int index,Collection c)

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);  //下标检查

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);  //将index-1之后的元素后移index+numNew位

    System.arraycopy(a, 0, elementData, index, numNew); //将a数据的元素复制到elementData
    size += numNew;
    return numNew != 0;
}      
```

##### 6.remove(int index)/ remove(Object obj)   

// 删除元素时不会减少容量，**若希望减少容量则调用trimToSize()**

```java
public E remove(int index) {
    rangeCheck(index);  

    modCount++;
    E oldValue = elementData(index);  //获取待移除的元素

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);  //index-1之后的元素前移一位

    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

private void rangeCheck(int index) {
    if (index >= size)   //负数抛出数组下标异常
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}


public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);   //index-1之后的元素前移一位

    elementData[--size] = null; // clear to let GC do its work
}
```



##### 7.  E  get(int index)

```java
public E get(int index) {
    rangeCheck(index); //下标检查

    return elementData(index);
}
```



##### 8. E set(int index,E e)

```java
public E set(int index, E element) {
    rangeCheck(index); //下标检查

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

### 三、 Vector

​		Verctor是ArrayList的前身，是jdk1.2的类了，比较老旧的一个集合类。其底层也是数据实现的，与ArrayList

的最大区别是Vector是线程安全的。为什么安全呢？答案是使用了：城管(synchronized)修饰了修改底层数组的方

法。由于使用城管，性能消耗比较大，因而被淘汰了。

```java
//增加
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

//移除
public synchronized boolean removeElement(Object obj) {
    modCount++;
    int i = indexOf(obj);
    if (i >= 0) {
        removeElementAt(i);
        return true;
    }
    return false;
}

//获取
public synchronized E get(int index) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    return elementData(index);
}

//修改
public synchronized E set(int index, E element) {
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);

    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}

```

在要求非同步的情况下，我们一般都是使用ArrayList来替代Vector的了~

如果想要ArrayList实现同步，可以使用Collections的方法：`List list = Collections.synchronizedList(new ArrayList(...));`，就可以实现同步了，也是使用了城管。

还有另一个区别：**ArrayList在底层数组不够用时在原来的基础上扩展0.5倍，也就是变为原来的1.5倍，Vector**

**是扩展1倍，变为原来的2倍。**




参考资料:

​	https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484130&idx=1&sn=4052ac3c1db8f9b33ec977b9baba2308&chksm=ebd743e3dca0caf51b170fd4285345c9d992a5a56afc28f2f45076f5a820ad7ec08c260e7d39&scene=21###wechat_redirect
