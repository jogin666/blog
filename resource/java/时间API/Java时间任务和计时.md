## Java时间任务和新计时姿势

### 1、时间任务

```java
public class Main {

    public static void main(String args[]){
        MyTimer.task();
    }
}


//自定义任务，继承时间任务类TimeTask		 
class MyTask extends TimerTask{

    @Override
    public void run() {
        SimpleDateFormat sdf=new SimpleDateFormat();
        System.out.println("doing timerTask"+sdf.format(new Date()));
    }
}


//自定义任务触发器，维护一个时间触发器Timer
class MyTimer{
    
    public static void task(){
        Timer timer=new Timer(); //任务触发器
        /**
         * 待执行的任务
         * 延迟执行时间
         * 执行任务的时间间隔
         */
        timer.schedule(new MyTask(),1000,60000); //任务触发器触发函数
    }
}
```

### 2、计时新姿势

1. 传统的计时

```java
public class Example{
    
     public static void main(String args[]){
         
        long startTime = System.currentTimeMillis();
        //......
        long endTime = System.currentTimeMillis();
        System.out.println(endTime-startTime);
    }
}
```

2. Java计时新姿势——`StopWatch`类

   `StopWatch`是Spring的核心库中的类，想要使用该类，就先要导入spring的核心库。

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
```

`StopWatch`的使用例子：

```java
public class TimeCounter {
    
    static void count(){
        StopWatch clock=new StopWatch();
        clock.start("开始任务一");	//设置任务名
        doSomething();
        clock.stop();
        
        clock.start("开始任务二");  //设置任务名
        doSomething();
        clock.stop();
        
        System.out.println(clock.prettyPrint()); //使用StopWatch自带输出函数
    }

    private static void doSomething() {
      
        try {
           Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    public static void main(String args[]){
        count();
    }
}

//输出结果
-----------------------------------------
02001  050%  开始任务一
02000  050%  开始任务二
//用时时间  占用总时间比	任务名
```

