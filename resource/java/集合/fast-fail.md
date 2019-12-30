## fast-fail

​	fast-fail：快速失败机制是java集合中一种错误机制。当多个线程对同一个集合内的元素进行修改的时候，有可

能就会产生快速失败(fast-fail)事件。它只能用来检测错误，并不代表fast-fail一定会发生。例如：当某一个线程A

通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出

ConcurrentModificationException异常，产生fail-fast事件。或者使用forEach循环删除元素时，也有可能发生

fast-fail机制。

### 一、快速失败机制

1. fast-fail 实例

   ```java
   public static void main(String args[]){
       ArrayList<Integer> list=new ArrayList<>();
       LinkedList<Integer> linkedList=new LinkedList<>();
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
   
   //Exception in thread "Thread-1" java.util.ConcurrentModificationException
   ```

2. 快速失败产生的原理与解析  

   ```java
   //此处采用的ArrayList源码
   private class Itr implements Iterator<E> { //ArrayList内部类
   
       int cursor;       // 下一个返回元素的下标
       int lastRet = -1; // 当前元素返回的下标
       int expectedModCount = modCount;  //快速失败机制实现的原理
   
       Itr() {}
   
       public boolean hasNext() {
           return cursor != size;
       }
   
       @SuppressWarnings("unchecked")
       public E next() {
           checkForComodification();
           int i = cursor;  
           if (i >= size)
               throw new NoSuchElementException();
           Object[] elementData = ArrayList.this.elementData;
           if (i >= elementData.length)
               throw new ConcurrentModificationException();
           //下一个
           cursor = i + 1;	
           return (E) elementData[lastRet = i];   //lastRet=i 指向获取元素的下标
       }
   
       public void remove() {  //移除lastRet下标的元素
           if (lastRet < 0)
               throw new IllegalStateException();
           checkForComodification();
   
           try {
               ArrayList.this.remove(lastRet);
               cursor = lastRet;
               lastRet = -1;
               expectedModCount = modCount;     //期待的操作总数与实际操作的总数
           } catch (IndexOutOfBoundsException ex) {
               throw new ConcurrentModificationException();
           }
       }
   
       //快速失败的实现，拥有的期望值与实际值比较，如果不等则说明，被另外线程操作过
       final void checkForComodification() {
           if (modCount != expectedModCount) //modCount是否在多线下修改过
               throw new ConcurrentModificationException();
       }
   
   
       @Override
       @SuppressWarnings("unchecked")
       public void forEachRemaining(Consumer<? super E> consumer) {
           Objects.requireNonNull(consumer);
           final int size = ArrayList.this.size;
           int i = cursor;
           if (i >= size) {
               return;
           }
           final Object[] elementData = ArrayList.this.elementData;
           if (i >= elementData.length) {
               throw new ConcurrentModificationException();
           }
           while (i != size && modCount == expectedModCount) {
               consumer.accept((E) elementData[i++]);
           }
           // update once at end of iteration to reduce heap write traffic
           cursor = i;
           lastRet = i - 1;
           checkForComodification();
       }
   
   }
   ```

3. modCount 值的变化过程

   ```java
   protected transient int modCount = 0;	// class AbstractList<E>
   
   private void ensureExplicitCapacity(int minCapacity) {   //确保容量
       modCount++;		//记录操作数
   
       // overflow-conscious code
       if (minCapacity - elementData.length > 0)
           grow(minCapacity);
   }
   
   public E remove(int index) {
       rangeCheck(index);
   
       modCount++;   //记录操作数
       E oldValue = elementData(index);
   
       int numMoved = size - index - 1;
       if (numMoved > 0)
           System.arraycopy(elementData, index+1, elementData, index,
                            numMoved);
       elementData[--size] = null; // clear to let GC do its work
   
       return oldValue;
   }
   
   public void clear() {
       modCount++;
   
       // clear to let GC do its work
       for (int i = 0; i < size; i++)
           elementData[i] = null;
   
       size = 0;
   }
   
   public void trimToSize() {
       modCount++;
       if (size < elementData.length) {
           elementData = (size == 0)
               ? EMPTY_ELEMENTDATA
               : Arrays.copyOf(elementData, size);
       }
   }
   ```
   

总结：快速失败机制就是容器的每一个修改操作，modCount操作就会增加1，在迭代器进行操作的时候，让线程拥有modCount的值和实际的modCount值比较，如果值是不相等的，就抛出异常。


### 二、迭代器

##### 1. ListItr迭代器 源码

```java
private class ListItr extends Itr implements ListIterator<E> {
    ListItr(int index) {  
        cursor = index;
    }

    public boolean hasPrevious() {
        return cursor != 0;   //父类的成员
    }

    public E previous() {
        checkForComodification();  //快速失败
        try {
            int i = cursor - 1;  //前一个元素
            E previous = get(i);
            //父类的成员
            lastRet = cursor = i;  //复制
            return previous;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public int nextIndex() {
        return cursor;
    }

    public int previousIndex() {
        return cursor-1;
    }

    public void set(E e) {
        if (lastRet < 0)
            throw new IllegalStateException();
        checkForComodification();	//快速失败

        try {
            AbstractList.this.set(lastRet, e);  //替换当前元素
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }

    public void add(E e) {
        checkForComodification();

        try {
            int i = cursor;
            AbstractList.this.add(i, e);
            lastRet = -1;
            cursor = i + 1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException ex) {
            throw new ConcurrentModificationException();
        }
    }
}

final void checkForComodification() {   //	快速失败   父类的方法 
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

   

   

   

   

   

   






