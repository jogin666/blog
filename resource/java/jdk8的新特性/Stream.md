## Java数据流(Stream)简介

流是 Java8 中 API 的新成员，它允许你以**声明式**的方式操作数据集合的数据。Stream在操作数据的时候，是以

数据流的形式操作，而不是单个单个元素的操作结合中的数据。

观看此文章需要有Lambda和函数式的基础，传送门 <a href="https://github.com/jogin666/blog/blob/master/resource/java/jdk8%E7%9A%84%E6%96%B0%E7%89%B9%E6%80%A7/Lambda%E5%92%8C%E5%87%BD%E6%95%B0%E5%BC%8F.md">Lambda表达式和函数式</a>

### 体验数据流

```java
public void example(){
	//传统的数组遍历
    for(int num: new int[]{1,2,3}){
        System.out.println(num);
    }
    //以流的方式遍历数组  forEach传入的是IntConsumer函数式接口的实例
     Arrays.stream(new int[]{1, 2, 3}).forEach(System.out::println);
}
```

### Stream的特点

##### 1、 操作数据是 内部迭代 的方式

​		操作数据集合中的数据时，开发人员无需单个单个的取元素，进行所需要的操作处理，直接使用Stream的

​		API，在数据集合中进行操作处理。

##### 2、 只能遍历一次

​		和迭代器一样，流只能遍历一次。当流遍历完之后，流已经被消耗完了，也就是流中的数据已经是被消耗

完，**不能再次操作被消耗数据完的流，否则抛出异常**。**值得注意的是**：每次从数据集合获取流，只是从源数据

集合中拷贝数据，构成一条数据流，**操作流中的数据不会影响到源数据**。

##### 3、支持并行操作

​    并行操作，就是内部多线程操作流中的数据。在数据量的时候，十分便捷。并行流的内部使用了默认的 

ForkJoinPool 分支/合并框架，它的默认线程数量就是你的处理器数量，这个值是由 

`Runtime.getRuntime().availableProcessors()` 得到的（当然卡法人员也可以全局设置这个值）。



### Stream的获取

 stream是数据流，继而便可以知道，流的获取都是与数据结合有关的。因而开发人员可以从数组，collection,

等数据容器和Stream的5个方法获取流。

```java
public void getStream(){
        IntStream stream1 = Arrays.stream(new int[]{1, 2, 3});  //数组 中获取整型流
    	//list中获取字符串流
        Stream<String> stream2 = Arrays.asList("z", "x", "c", "v").stream()
        Stream<Object> stream3 = new HashSet<Object>().stream();//set 中获取对象流
    
        Stream<String> stream4=Stream.empty();  //空的数据流
    	 //创建初始数据的流
        Stream<Integer> stream5 = Stream.of(1, 2, 3); 
     	//根据供给函数式接口的操作，返回一个没有限制供给的流
    	Stream.generate(()->Math.random()).forEach(System.out::println);
    	//把两个流合成一个流，并返回
    	Stream stream6=Stream.concat(Stream.of(12,13,10),Stream.of(1,2,3));
    	//通过Stream中的iterate方法创建,会返回一个无限有序值的Stream对象
     	Stream.iterate(12,(str)->str).forEach(System.out::println);
    }
```



### Stream接口的API介绍

Stream的API有很多，可将这些API分为量两类：中间操作和结束操作。**中间操作并不会立即执行(懒执行)，而**

**是需要等到流要执行最终操作时，执行之前所有的中间操作 **。

```java
public void example(){
	Stream.of(3,1,5,6,2).sort(); //sort()是一个中间操作，并不立即执行操作
    //forEach(...)是结束操作，流的操作sort()才会执行。
    Stream.of(3,1,5,6,2).sort().forEach(System.out::println); 
}
```

**中间操作：又可以分为无状态操作和有状态操作**。无状态的操作是指此刻的操作时不受前面的操作影响的，而有状

态的中间操作就是受之前操作的影响，需要所有元素操作完之后才得到最终结果。比如sum()就是是有状态操作，

最终的结果需要前面的操作结果的参与；

**结束操作：又分为短路操作和非短路操作**。短路操作是指找到任一符合条件的结束流操作，返回结果。而非短路运算是要取出流中的所有满足条件数据流中的元素。

##### 中间操作

| 方法                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Stream<T> distinct();`                                      | 返回一个没有不重复元素的流                                   |
| `Stream<T> filter(Predicate<? super T> predicate)`           | 根据Predicate定义的判定规则与过滤流的元素，返回所有符合规则元素的流 |
| `<R> Stream<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)` | 扁平化：将流中的元素(数组，集合)分别映射成流，然后汇聚成一条总流，并返回。 |
| `IntStream flatMapToInt(Function<? super T, ? extends IntStream> mapper)` //其他类型方法类型 | 扁平化，返回IntStream                                        |
| `Stream<T> limit(long maxSize);`                             | 返回指定个数元素形成新的流                                   |
| `<R> Stream<R> map(Function<? super T, ? extends R> mapper)` | 根据函数式接口Function定义的操作，返回操作结果的流           |
| `IntStream mapToInt(ToIntFunction<? super T> mapper)`        | 根据函数式接口ToIntFunction定义的操作，返回操作结果的IntStream |
| `Stream<T> peek(Consumer<? super T> action)`                 | 消费流的元素，并返回新的流，流元素不变                       |
| `Stream<T> skip(long n)`                                     | 返回抛弃前指定个数元素后，使用剩下的元素组成新的Stream       |
| `Stream<T> sorted()`                                         | 返回排序后的Stream对象                                       |

##### 非中间操作

| 方法                                                     | 描述                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| `boolean allMatch(Predicate<? super T> predicate)`       | 判定流中的所有元素是否符合定义的判定规则                     |
| `boolean anyMatch(Predicate<? super T> predicate)`       | 判定流中是否存在符合判定队则的元素                           |
| `<R, A> R collect(Collector<? super T, A, R> collector)` | 使用 Collectors（java.util.stream.Collectors）来进行各种 reduction 操作 |
| `long count();`                                          | 获取流元素的个数                                             |
| `Optional<T> findAny();`                                 | 返回任意一个元素的Optional对象，如果无元素返回的是空的Optioanl。 |
| `Optional<T> findFirst()`                                | 返回有序流的第一个元素的Optioanl对象；如果无元素返回的是空的Optional； 如果Stream是无序的，那么任何元素都可能被返回。 |
| `void forEach(Consumer<? super T> action)`               | 消费流中的所有元素                                           |
| `void forEachOrdered(Consumer<? super T> action)`        | 有序输出流中的元素                                           |
| `Optional<T> max(Comparator<? super T> comparator)`      | 根据比较器的比较规则，返回value是流中比较出来最大的元素Optional对象 |
| `Optional<T> min(Comparator<? super T> comparator)`      | 与max方法相反                                                |
| `boolean noneMatch(Predicate<? super T> predicate)`      | 如果所有元素均不满足传入的Predicate定义的规则时返回True，否则False |
| `T reduce(T identity, BinaryOperator<T> accumulator)`    | 归约，指定初值和操作定义，返回所有流元素操作的结果           |
| `Object[] toArray()`                                     | 返回所有元素的数组                                           |



### 数据流常用的方法

* filter方法示例

```java
public void example(){
	Stream<String> stream=Stream.of("test","text","people","team","person");
    //过滤没有包含 t 的单词
    stream.filter(str->str.conatins("t")).forEach(System.out::println); //test,text,team
    stream.close(); //关闭流
}
```

* map方法示例

```java
public void example(){
	Stream<String> s=Stream.of("a","b","c");
    //为流中的每一个元素追加".txt"
	s.map(str->str.concat(".txt")).forEach(System.out::println);
	s.close();
}
```

* flatMap方法示例

```java
public void example(){
	List<String> words = Arrays.asList("Hello", "World");
	words.stream()
     	.map(s -> s.split(""))   //拆分单词
     	.flatMap(Arrays::stream) //单词扁平化流，将每个数组的流汇成流
     	.distinct() //去重复
     	.forEach(System.out::println); //输出流的元素
}
```

* reduce方法示例

```java
public void eample(){
    //求和
	Integer num = Stream.of(1, 2, 3, 5, 6).reduce(0, Integer::sum); //底层应该是有元素迭代
    System.out.println(num); //17
}
```

* collect方法示例

```java
public void example(){
	 Stream<String> s1=Stream.of("test", "team", "speak", "task", "sun");
    List<String> list = s1.filter(str -> str.startsWith("t"))
        					.collect(Collectors.toList());
}
```

* parallel方法示例

```java
public void test(){
    
	System.out.println(String.format("此台计算机的核数：%d",
											Runtime.getRuntime().availableProcessors()));
    int max_value=100_000_000;

    Random random=new Random();
    List<Integer> list=new ArrayList<>(max_value);
    for (int i=0; i<max_value; i++){
        list.add(random.nextInt(100));
    }

    long preTime=System.currentTimeMillis();
    //串行
    list.stream().reduce((a,b)->a+b).ifPresent(System.out::println);
    System.out.println("单线程所需要的时间："+(System.currentTimeMillis()-preTime));

    //平行流
    preTime=System.currentTimeMillis();
    list.stream().parallel().reduce(Integer::sum).ifPresent(System.out::println);
     System.out.println("多线程所需要的时间："+(System.currentTimeMillis()-preTime));
}
/*输出

此台计算机的核数：4
655253781
单线程所需要的时间：4360
655253781
多线程所需要的时间：649
*/
```



参考文章：

1. https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247485508&idx=2&sn=a686a128ccbcfa1fcc000d8b9de14155&chksm=ebd74945dca0c05378c3083c6efda294ea11db25705436d08a6d6af4e82993cac99804ee1553&token=2078489135&lang=zh_CN###rd

2. https://blog.csdn.net/icarusliu/article/details/79495534#31-stream

