## SimpleDateFormat简单介绍

原文地址：<a href="https://blog.csdn.net/u012939476/article/details/89217539">SimpleDateFormat 多线程不安全原因,及解决方案</a>





日常开发中，我们经常需要使用时间相关类，说到时间相关类，想必大家对`SimpleDateFormat`并不陌生。主要

是用它进行时间的格式化输出和解析，挺方便快捷的，但`SimpleDateFormat`并不是一个线程安全的类。在多线

程情况下，会出现异常，想必有经验的小伙伴也遇到过。下面我们就来分析分析`SimpleDateFormat`为什么不安

全？是怎么引发的？以及多线程下有那些`SimpleDateFormat`的解决方案？

先看看《阿里巴巴开发手册》对于`SimpleDateFormat`是怎么看待的：

```java
/*
5. SimpleDateFormat是线程不安全的类，一般不要定义为static变量，如果定义static变量，，必须枷锁，
或者使用DateUtils工具类。
正例：注意线程安全，使用DateUtils。亦推荐如下处理：
*/
private static final SimpleDateFormat df=new SimpleDateFormat(){
    @Override
    protected DateFormat intialValue(){
        return new SimpleDateFormat("yyyy-MM-dd");
    }
};
/* 如果是JDK8的应用，可以使用Instant代替Date，LocalDateTime代替Calendar，DateFormatter代替
SimpleDateFormat。 
```

> 附《阿里巴巴Java开发手册》v1.4.0(详尽版)下载链接：https://yfzhou.oss-cn-beijing.aliyuncs.com/blog/img/《阿里巴巴开发手册》v 1.4.0.pdf



### 问题场景复现

一般使用SimpleDateFormat的时候都是定义为static变量，避免频繁创建SimpleDateFormat的对象实例。

如下代码：

```java
public class SimpleDateFormatTest {

    private static final SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public static String formatDate(Date date){
        return sdf.format(date);
    }

    public static Date parse(String dateStr) throws ParseException {
        return sdf.parse(dateStr);
    }

    public static void main(String args[]){
        System.out.println(sdf.format(new Date()));
    }
}
```

是不是感觉没什么毛病？单线程下自然没毛病了，都是运用到多线程下就有大问题了。测试下：

```java
public static void main(String args[]) throws InterruptedException {
    ExecutorService service= Executors.newFixedThreadPool(100);
    for(int i=0;i<20;i++){
        service.execute(()->{
            for (int j=0;j<5;j++){
                try{
                    System.out.println(parse("2018-01-02 09:45:59"));
                } catch (ParseException e) {
                    e.printStackTrace();
                }
            }
        });
    }
    //等待上述的线程执行完
    service.shutdown();
    service.awaitTermination(1, TimeUnit.DAYS);
}
```

输出结果：

![img](https://img-blog.csdnimg.cn/20190411185133775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5Mzk0NzY=,size_16,color_FFFFFF,t_70)

你看这不崩了？部分线程获取的时间不对，部分线程直接报`java.lang.NumberFormatException: multiple points`错，线程直接挂死了。



### 多线程不安全的原因

因为我们把SimpleDateFormat定义static变量，n那么在多线程下SimpleDateFormat的实例就会别多个线程所共

享，B线程获取到A线程的时间，就会出现时间差异和其他问题。SimpleDateFormat和其父类DateFormat类也是

线程不安全的。

来看看SimpleDateFormat的format()方法的源码

```java
@Override
public StringBuffer format(Date date, StringBuffer toAppendTo,FieldPosition pos){
    pos.beginIndex = pos.endIndex = 0;
    return format(date, toAppendTo, pos.getFieldDelegate());
}
    
private StringBuffer format(Date date, StringBuffer toAppendTo,FieldDelegatede legate) {
    
    calendar.setTime(date);

    boolean useDateFormatSymbols = useDateFormatSymbols();

    for (int i = 0; i < compiledPattern.length; ) {
        int tag = compiledPattern[i] >>> 8;
        int count = compiledPattern[i++] & 0xff;
        if (count == 255) {
        	count = compiledPattern[i++] << 16;
        count |= compiledPattern[i++];
    }

    switch (tag) {
    	case TAG_QUOTE_ASCII_CHAR:
        	toAppendTo.append((char)count);
            break;

		case TAG_QUOTE_CHARS:
        	toAppendTo.append(compiledPattern, i, count);
            i += count;
            break;

		default:
        	subFormat(tag, count, delegate, toAppendTo, useDateFormatSymbols);
            break;
            }
      }
        return toAppendTo;
}
```

注意：calender.setTime(date);SimpleDateFormatde format方法实际就是操作的就是Calender。

因为SimpleDateFormat的实例对象被声明为static变量，那它的Calender的变量也就是一个共享的变量，可以被

多个线程访问。假设线程A执行完calender.setTime(date),把时间设置成2019-01-02，这是后被挂起，线程B获得

cpu的执行权，也开始执行calender.setTime(date)，把时间设置成2019-02-03.线程B被挂起，线程A重新获得cpu

的执行权，线程A继续走，calendar还会被继续使用(subFormat方法)，而这时calendar用的是线程B设置的值0了

而这就是引了发问题的根源，出现时间不对，线程挂死等等。



其实SimpleDateFormat源码上作者也给过我们提示：

* Date formats are not synchronized.
* It is recommended to create separate format instances for each thread.
* If multiple threads access a format concurrently, it must be synchronized
* externally

意思就是：

> 日期格式不同步。
> 建议为每个线程创建单独的格式实例。
> 如果多个线程同时访问一种格式，则必须在外部同步该格式



### 解决方案

1. 在需要到SimpleDateFormat的时候，创建实例，不要使用static修饰

```java
public static String formatDate(Date date){
    SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    return sdf.format(date);
}

public static Date parse(String dateStr) throws ParseException {
    SimpleDateFormat sdf=new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    return sdf.parse(dateStr);
}
```

如上代码，仅在需要用到的地方创建一个新的实例，就没有线程安全问题，不过也加重了创建对象的负担，会频繁地创建和销毁对象，效率较低。

2. 使用城管Synchronized

```java
public static String formatDate(Date date){
    synchronized (sdf) {
   		return sdf.format(date);
    }
}

public static Date parse(String dateStr) throws ParseException {
    synchronized (sdf) {
    	return sdf.parse(dateStr);
    }
}
```

简单粗暴，synchronized往上一套也可以解决线程安全问题，缺点自然就是并发量大的时候会对性能有影响，线程阻塞。

3. 使用ThreadLocal，保证线程各自拥有一个SimpleDateFormat实例

```java
private static ThreadLocal<DateFormat> threadLocal=new ThreadLocal<DateFormat>(){
    @Override
    protected DateFormat initialValue(){
        return new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    }
};

public static String formatDate(Date date){
    return threadLocal.get().format(date);
}

public static Date parse(String dateStr) throws ParseException {
    return threadLocal.get().parse(dateStr);
}
```

4. 使用jdk8之后的时间类，和时间格式类DateTimeFormatter类，线程安全且不可更改。

```java
private static DateTimeFormatter dtf=DateTimeFormatter.ofPattern("yyyy-MM:dd hh:mm:ss");
    
public static String formatDate(LocalDate date){
    return dtf.format(date);
}

public static LocalDate parse(String dateStr) throws ParseException {
    return LocalDate.parse(dateStr,dtf);
}
```

