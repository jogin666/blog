## Spring Cloud 篇章（三）项目篇（简易）

在上一篇《<a href="">Spring Cloud 篇章（二）快速入门篇</a>》快速搭建了一个十分简单的 Spring Cloud 的入门 demo，此篇

章根据使用根据 java3y 的（<a href="https://github.com/ZhongFuCheng3y/msc-Demo/tree/master/microservicecloud">msc- Demo</a>）资源，搭建一个有集群的 Spring Cloud  demo。 

github地址：https://github.com/jogin666/microservicecloud



### 一、准备工作 

**1.1、配置虚拟主机映射**

在 hosts 文件（*C:\Windows\System32\drivers\etc*） 增加三个主机映射

```
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
127.0.0.1 config-3344.com
```

**1.2、创建数据库**

创建三个mysql的数据库

```
create database cloudDB01;
create database cloudDB02;
create database cloudDB03;
```

**1.3、创建表**

在三个数据库创建一张数据表

```sql
CREATE TABLE Dept(
    deptno INT PRIMARY KEY,
    dname VARCHAR(50),
    db_source VARCHAR(50)
);
```

**1.4、为数据库表插入数据**

```sql
-- database 1

INSERT INTO dept (deptno, dname, db_source) VALUES (1, 'Java1y', '1');
INSERT INTO dept (deptno, dname, db_source) VALUES (2, 'Java2y', '1');
INSERT INTO dept (deptno, dname, db_source) VALUES (3, 'Java3y', '1');
INSERT INTO dept (deptno, dname, db_source) VALUES (4, 'Java4y', '1');
INSERT INTO dept (deptno, dname, db_source) VALUES (5, 'Java5y', '1');


-- database 2

INSERT INTO dept (deptno, dname, db_source) VALUES (1, 'Java1y', '2');
INSERT INTO dept (deptno, dname, db_source) VALUES (2, 'Java2y', '2');
INSERT INTO dept (deptno, dname, db_source) VALUES (3, 'Java3y', '2');
INSERT INTO dept (deptno, dname, db_source) VALUES (4, 'Java4y', '2');
INSERT INTO dept (deptno, dname, db_source) VALUES (5, 'Java5y', '2');


-- database 3

INSERT INTO dept (deptno, dname, db_source) VALUES (1, 'Java1y', '3');
INSERT INTO dept (deptno, dname, db_source) VALUES (2, 'Java2y', '3');
INSERT INTO dept (deptno, dname, db_source) VALUES (3, 'Java3y', '3');
INSERT INTO dept (deptno, dname, db_source) VALUES (4, 'Java4y', '3');
INSERT INTO dept (deptno, dname, db_source) VALUES (5, 'Java5y', '3');
```



### 二、模块之间的解释

**2.1、Eureka 服务注册中心（集群**）

* eureka_server1  [eureka7001.com]
* eureka_server2  [eureka7002.com]
* eureka_server3  [eureka7003.com]

**2.2、服务提供方**

* provider_dept1   [provider_dept_8001]
* provider_dept2   [provider_dept_8002]
* provider_dept3   [provider_dept_8003]
* provider_dept4   [provider_dept_8001_hystrix]   //使用熔断机制

**2.3、网关**（用于转发路由，服务过滤(安全验证)，限流等等）

* zuul_gateway_8080

**2.4、服务消费者**

* consumer1   // 使用restTemplate+ribbon的方式来测试(ribbon自定义了负载均衡算法)
* consumer_fegin   [consumer2]  //使用了Fegin（整合负载均衡 和 熔断机制）

```java
@SpringBootApplication
@EnableEurekaClient
//指定熔断处理类所在的包
@EnableFeignClients(basePackages = {"com.springcloud.api"})
//进行扫描
@ComponentScan("com.springcloud.api")
public class DeptConsumer_2_80 {

    public static void main(String args[]){
        SpringApplication.run(DeptConsumer_2_80.class,args);
    }
}
```

* consumer3_hystrix_dashboard [consumer3]    //监控消费方的指标(性能)

启动consumer-hystrix-dashboard项目，打开`http://localhost:9001/hystrix.stream`

![dashboard](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/dashboard.png)

可以监控：`microservicecloud-provider-dept-hystrix-8001`这个项目，于是在输入栏输入`http://localhost:8001/hystrix.stream`

![dashboard1](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/dashboard1.png)

随后，测试接口：`http://localhost:8001/dept/get/7` ，监控的数据就会变化了：![dashboard3](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/dashboard3.png)

**2.5 配置中心**

SpringCloud Config服务端(获取配置都从这里来拿)

- microservicecloud-config-3344

SpringCloud Config 客户端：

- microservicecloud-config-client-3355  
- microservicecloud-config-dept-client-8001   //提供服务
- microservicecloud-config-eureka-client-7001  //注册中心

最后退下项目工程图：

![项目工程（集群）](https://github.com/jogin666/blog/blob/master/resource/spring%20family/spring%20cloud/images/%E9%A1%B9%E7%9B%AE%E5%B7%A5%E7%A8%8B%EF%BC%88%E9%9B%86%E7%BE%A4%EF%BC%89.png)