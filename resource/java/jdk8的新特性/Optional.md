# Optional类

一句话介绍Optional类Optinal是一个容器类，存放的类型只能是null或者是非null，来防止NPE(空指针异常)问题。

观看此文章需要有Lambda和函数式的基础，传送门<a href="https://github.com/jogin666/Java/blob/master/java8新特性/Lambda和函数式.md">Lambda表达式和函数式</a>

### 源码介绍

```java
public final class Optional<T> {
   
    private static final Optional<?> EMPTY = new Optional<>();//内部维护一个value为null的实例

    private final T value; //元素

    private Optional() {
        this.value = null;
    }

    public static<T> Optional<T> empty() {	//创建一个value为null的实例
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }

    public static <T> Optional<T> of(T value) {//创建一个value不能为null的实例，为null抛出异常
        return new Optional<>(value);
    }
    
    public static <T> Optional<T> ofNullable(T value) { //根据value的类型，返回一个实例
        return value == null ? empty() : of(value);
    }

    public T get() {  //获取value，为空时抛出异常
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }

    public boolean isPresent() { //判定value是否为null
        return value != null;
    }

    public void ifPresent(Consumer<? super T> consumer) {  //进行一个消费value的操作
        if (value != null)
            consumer.accept(value);
    }

    public Optional<T> filter(Predicate<? super T> predicate) { 
        Objects.requireNonNull(predicate);
        if (!isPresent()) //value为null，返回实例本身
            return this;
        else		//判定定义的判定规则返回对应的实例
            return predicate.test(value) ? this : empty();
    }

    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();  //value为null，创建一个value为null的实例，并返回
        else {
             //根据maper的操作返回值，创建并返回相应的实例
            return Optional.ofNullable(mapper.apply(value));
        }
    }

    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper); //要求mapper不能为null
        if (!isPresent())
            return empty(); //value为空时，返回一个value=null的实例
        else {
             //根据maper的操作返回值，创建并返回相应的实例
            return Objects.requireNonNull(mapper.apply(value));
        }
    }

    //value为空时，返回指定的值，否则返回value的值
    public T orElse(T other) {
        return value != null ? value : other;
    }
    
 	//value为空时，从定义的供给类中获取值，否则返回value的值
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
    
    //value为空时，抛出指定的异常，否则返回value的值
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (!(obj instanceof Optional)) {
            return false;
        }

        Optional<?> other = (Optional<?>) obj;
        return Objects.equals(value, other.value);
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(value);
    }

    @Override
    public String toString() {
        return value != null
            ? String.format("Optional[%s]", value)
            : "Optional.empty";
    }
}
```



### 实例操作：

1. 创建Optional实例

```java
public void newOptioal(){
	Optional<String> opl1=Optional.empty(); //创建value为空的实例
	Optional<String> opl2=Optional.of("12"); //创建指定值不为null的实例
    Optional<String> opl3=Optional.ofNullable(null); //根据指定的类型创建对应的实例
} 
```

2. 判定value和获取value的值

```java
public void example(){
    Optional<String> opl2=Optional.of("12");
    opl2.isPresent();//判定value是否为null
    opl2.get(); //获取value的值，value为null抛出异常
}
```

3. 使用map方法提取值

```java
static class User{
    private String username="12";

    public String getUsername(){
        return this.username;
    }
}

public static void main(String args[]){
    User user=new User();
    Optional<User> optional=Optional.ofNullable(user);
    //返回一个value为username值的Optional实例
    Optional<String> optional1=optional.map(User::getUsername); 
    System.out.println(optional1.get()); //12
}
```

4. orElse、orEelseGet、orElseThrow方法

```java
public static void main(String args[]){
    Optional<Integer> opl=Optional.empty(); //创建value为null的实例
    
    System.out.println(opl.orElse(12)); //value为空，返回指定12
    System.out.println(opl.orElseGet(()->10)); //value为null，返回供给提供的值
    //value为空，抛出定义好的异常
    System.out.println(opl.orElseThrow(()-> new RuntimeException("出错了")));
}
```

5. 使用filter()方法过滤

```java
public void example(){
	Optional<String> optional = Optional.of("xxxx@163.com");
     //不符合判定，返回一个value为null的实例
	optional = optional.filter(str -> str.contains("qq"));
}
```

6. 使用map和orElse简化if-elsefe分支

```java
//传统写法
public String toUpperUserName1(User user){
	 if(user!=null){
     	String username = user.getUsername();
         if(username!=null){
         	return username.toUpperCase();
         }else{
         	return null;
         }
     }
     return null;
}

//使用Optional  代码简洁美观
public String toUpperUserName2(User user){
    Optional<User> opl=Optional.ofNullable(user);
    return opl.map(User::getUsername)
                .map(String::toUpperCase)
                .orElse(null);
}
```

