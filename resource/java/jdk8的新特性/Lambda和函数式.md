## Java8新特性——Lambda和函数式

### 1、Lambda表达式初步认识

直接上代码

```java
//创建线程
public void example1(){
    //传统写法
    new Thread(new Runnable(){
        @Override
        public void run(){
            System.ot.println("this is a test");
        }
    });
    //lambda表达式的使用
    new Thread(()->{System.ot.println("this is a lambda test");})
}

//list集合的遍历
public void example2(){
    List<String> list = new ArrayList<>();
    list.add("1");
    list.add("2");
    list.add("2");
    list.add("3");
    //传统遍历  for循环和迭代器
    for(String str:list){  
        System.out.println(str);
    }
    //lambda表达式
    list.forEach(s->System.out.println(s));
}

//map集合的遍历
public void example3(){
    //传统方式
    Map<String, String> hashMap = new HashMap<>();
    hashMap.put("1", "11");
    hashMap.put("2", "22");
    for(Map.Entry<String,String> entry:hashMap.entrySet()){
        System.out.println(entry.getKey()+":"+entry.getValue());
    }
    //lambda表达式
    hashMap.forEach((s1,s2)-> System.out.println(s1+":"+s2));
}
```



### 2、lambda表达式的语法规则

##### 2.1 lambda的语法形式

```java
/* 
	()->{} 
	其中()是用来输入参数的，例如 (int var1,String var2),前面是参数类型，后面是参数引用，多个参数之间
	使用逗号隔开，更可以这样写法(var1,var2),参数类型可以不写，只用一个参数是，可以省略中括号()。
	{}花括号是整个方法体，方法执行执行的操作。当只有一条语句时，{}可以省略不写。
	
	然而这还不是最简的，由于此处只是进行打印，调用了System.out中的println静态方法对输入参数直接进行打
	印，因此可以简化成以下写法：
	Consumer<Integer> c=System.out::println;  //System.out 指定类 ::println 指定方法
    c.accept(12);
	它表示的意思就是针对输入的参数将其调用System.out中的静态方法println进行打印。
   
	
	()->{} 为什么能会被执行编译器所识别呢？
	因为函数式编程接口都只有一个抽象方法，因此在采用这种写法时，编译器会将这段函数编译后当作该抽象方法的实
	现。因此，=后面的函数体我们就可以看成是accept函数的实现。
*/
```



### 3、 函数式接口

##### 3.1 函数式接口注解

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface {}
```

​	每一个函数式接口被函数式注解所注解，表明该方法是函数式接口。

##### 3.2 函数式接口

* Consumer 消费接口，消费的规则与实现由开发人员定义

```java
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t); //待实现的消费方法

    default Consumer<T> andThen(Consumer<? super T> after) { //返回一个定义好消费顺序的消费组合
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); }; 
        /*
            (T t)->{} 是定义一个消费者，而
            {
                accept(t);//执行当前消费者的消费
                after.accept(t);//执行传入进来的消费者的消费
            } 
        */
    }
}
/****************************实例代码****************************/
public static void main(String args[]){
    Consumer<String> consumer1=s -> System.out.print("name：Sun"+'\t');
    Consumer<String> consumer2=s -> System.out.println("age:18");
    consumer1.andThen(consumer2).accept("");
    // 输出  name：Sun	age:18
}
```

* Function 函数式编程接口

```java
@FunctionalInterface
public interface Function<T, R> {
	
    //函数接口定义操纵
    R apply(T t);

	//返回一个函数执行组合的函数
    //以第一个function的 apple(T t) 处理的输出结果作为下一个function的 apple(T t)输入
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v)); //先执行before的apply方法，然后执行本身的
    }
    
   //返回一个函数执行组合的函数
   //以第一个function的 apple(T t) 处理的输出结果作为下一个function的 apple(T t)输入
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t)); 
    }

    //identity方法会返回一个不进行任何处理的Function，即输出与输入值相等；
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
/****************************实例代码****************************/
public static void main(String args[]){
    Function<Integer,Integer> fun1=num->++num;
    Function<Integer,Integer> fun2=num->num*2;

    System.out.println(fun1.compose(fun1).apply(1)); //3  先执行1*2 然后执行2+1

    System.out.println(fun2.andThen(fun1).apply(2));//5  先执行2*2  然后执行4+1

    System.out.println(Function.identity().apply("12")); //12
}
```

* Predicate 函数式判定接口

```java
@FunctionalInterface
public interface Predicate<T> {
	//判定方法
    boolean test(T t);

    //两个判定规则的与操作
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    
    //对开发人员的判定操作的结果取反
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    //两个判定规则的或操作
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
	
    //对对象就行判空
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}

/****************************实例代码****************************/
public static void main(String args[]){
    Predicate<String> predicate1=o->"test".equals(o);
    Predicate<String> predicate2=o->o.startsWith("t");

    System.out.println(predicate1.test("test"));  //true
    System.out.println(predicate1.and(predicate2).test("test")); //true

    System.out.println(predicate1.or(predicate2).test("false")); //false
    System.out.println(predicate1.negate().test("test")); //false
    System.out.println(Predicate.isEqual(null).test("12")); //false
}
```

* Supplier 函数供给接口(又返回值)

```java
@FunctionalInterface
public interface Supplier<T> {
	//返回操作结果
    T get();
}
```

* UnaryOperator

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    //返回一个不进行任何处理的UnaryOperator，即输出与输入值相等
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
```



参考文章：

https://blog.csdn.net/icarusliu/article/details/79495534#31-stream
