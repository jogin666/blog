## Java时间类学习

##### Date介绍

在jdk8之前，`java.util.Date`是开发人员常用获取时间的工具类，该类是以1970年1月1日作为时间的起点，计算时间，获取当前的系统时间。

1. Date获取系统时间方式

```java
public void showDate(){
    Date date=new Date(); //获取当前系统的时间
    Date date=new Date(50000);	//时间起点上便宜50s后时间
}
```

2. `long getTime()`

```java
public static void main(String args[]){
    Date date=new Date();
    long time = date.getTime();  //获取当前时间到起点时间的毫秒数
    //System.currentTimeMillis()  两个方法等价
    System.out.println(time);
}
```

3. 时间比较方法

   | 方法                              | 描述                            |
   | --------------------------------- | ------------------------------- |
   | `boolean after(Date when)`        | 当前时间是否在指定的时间之前    |
   | `boolean before(Date when)`       | 当前时间是否在指定的时间之后    |
   | `int compareTo(Date anotherDate)` | 当前时间是否早于/晚于指定的时间 |

   

#### `SimpleDateFormat`介绍

`SimpleDateFormat`是抽象类`DateFormat`的子类。`DateFormat`是日期/时间格式转化的类，它以与语言无关的

方式格式化并解析日期或时间。

##### `SimpleDateFormat`常用的方法

```java
public static void main(String args[]) throws ParseException {
    Date date=new Date();
    //有参构造函数，指定格式    //有参构造函数，指定格式
    SimpleDateFormat dateFormat=new SimpleDateFormat("yyyy-MM:dd hh:mm:ss");  

    String format = dateFormat.format(date);//格式日期
    System.out.println(format);

    Date date1=dateFormat.parse(format);   //将符合格式的字符串形式的日期转化为日期实例
}
```



### Calendar 日历类

Calendar类即日历类，常用于进行“翻日历”，比如下个月的今天是几号。通过Calendar开发人员可以获取到当前时间的年，月，日，时，分，秒，毫秒，当前时间，周几等。

```java
public static void main(String args[]) throws ParseException {
    Calendar calendar=Calendar.getInstance();
    int year=Calendar.YEAR;
    int month=Calendar.MONTH;
    int hour=Calendar.HOUR;
    int minute=Calendar.MINUTE;
    int second=Calendar.SECOND;
    Date date=calendar.getTime();
}
```

当然除了上述的所讲的，该类仍然有其他常用的方法，具体得具体业务。



### 值得注意的是

Date、SimpleDateFormat、Calendar都是线程不安全的类，很容易出错，因为大部分操作都与静态成员有关。





### JDK8的时间新特性

jdk8之后，java将时间这个概念划分的更加清晰，划分出，时区，日期，日期时间，时间，年，月，日等大量线程

安全且不可改变的时间类，这些时间类都在`java.lang.Time`包下。



##### 常用的时间类

* Instant ：本质是一个时间戳
* LocalDate：日期类，获取日期和设置日期。
* LocalTime：时间类，获取系统的时间
* LocalDateTime：日期时间类，操作日期和时间。
* ZonedDateTime：时区类，获取时区的日期时间。
* ChronoUnit：计算时间的工具
* TemporalAccessor：计算时间的工具
* ..........

##### 方法概览：

`java.lang.time`包下的API提供了大量相关的方法，这些方法一般有一致的方法前缀：

* of：静态工厂方法。
* parse：静态工厂方法，关注于解析。
* get：获取值。
* is：用于比较。
* with：不可变的setter等价物。
* plus：加一些量到某个对象。
* minus：从某个对象减去一些量。
* to：转换到另一个类型。
* at：把这个对象与另一个对象组合起来，例如：date.atTime(time)。
  

##### 部分时间代码演示

```java
//时间获取
public static void main(String args[]) throws ParseException {
    //获取时间
    LocalDateTime dateTime=LocalDateTime.now();
    //打印时间
    System.out.println(dateTime); //2019-11-13T17:55:50.959
    //获取格式时间的实例对象，线程安全类
    DateTimeFormatter formatter=DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
    System.out.println(formatter.format(dateTime)); //2019-11-13 05:55:50

    //获取年份
    Year year = Year.of(2019);
    //是否为闰年
    System.out.println(year.isLeap()); //false

    //获取2019年开始后的第33天的日期
    LocalDate date=year.atDay(33);
    System.out.println(date); //2019-02-02

    //获取指定的时间,11点整
    LocalTime time=LocalTime.of(11,0,0);
    System.out.println(time); //11:00

    //将日期与时间整合在一起
    LocalDateTime localDateTime=time.atDate(date);
    System.out.println(formatter.format(localDateTime)); //2019-02-02 11:00:00
}



/******************************分割线***************************/
//时间格式化与计算：
public static void main(String args[]) throws ParseException {
    //获取周一
    DayOfWeek dayOfWeek=DayOfWeek.of(1);
    System.out.println(dayOfWeek); //MONDAY

    //计算两个日期之间的时间间隔（小时），也可以使用其他时间单位
    LocalDateTime dateTime1 = LocalDateTime.of(2019, 11, 13, 11, 0, 0);
    LocalDateTime dateTime2 = LocalDateTime.of(2019, 11, 12, 11, 0, 0);
    long between = ChronoUnit.HOURS.between(dateTime1, dateTime2); 
    System.out.println(between); //-24

    //格式化时间
    DateTimeFormatter dateTimeFormatter2 = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    TemporalAccessor temporalAccessor = dateTimeFormatter2.parse("2019-11-13");
    System.out.println( LocalDate.from(temporalAccessor)); // 2019-11-13

    LocalDate localDate=LocalDate.now();
    //某月的第一天日期
    LocalDate date=localDate.with(TemporalAdjusters.firstDayOfMonth());// 2019-11-01
    System.out.println(date);

    //某月的第一个星期一的日期
    TemporalAdjuster adjuster=TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY);
    LocalDate date2 = localDate.with(adjuster);
    System.out.println(date2); //2019-11-04

    //计算下一个星期一的日期
    LocalDate date1=localDate.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
    System.out.println(date1); //2019-11-18
}
```

### 新旧时间之间的转换

```java
public static void main(String args[]) throws ParseException {

    Instant now = Instant.now();
    Date date = Date.from(now);
    now=date.toInstant();
    now=Calendar.getInstance().toInstant();
}
```

值得注意的是：输出instant时可能会出现与实际时间差8个小时的情况，这是因为Date和Calender输出时是按照

中国所在的时区（UTC\GMT+8）输出的，而instant输出是按照相对于GMT的时间输出的，所以会相差8个小时。




参考文章：https://www.jianshu.com/p/61e470e65d31

