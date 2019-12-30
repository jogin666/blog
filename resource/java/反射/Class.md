## Java反射——Class

放在前面：本文根据 <a href="https://blog.csdn.net/dufufd/article/details/80537638">Java中Class对象详解</a> 部分改动而来，侵删。



##### 一、Class

每一个`class`都有一个`Class`对象，包括Java的基本类型，数组。每一个Class对象都包含对应class的所有信息，

也就是java文件编译后的字节码文件的信息，如果说class是对象的抽象和数据的集合，那Class便是对class抽象的

和数据的集合。



Class类没有公共的构造函数，因为Class的实例对象都是在class加载器加载class时，有`JVM`调用class加载器中的

`defineClass`方法自动创建Class实例的。因此不能显示的声明Class对象。



Java中所有的每一个类在第一使用的时候，都都被先被JVM动态的加载到内存中(懒加载)，各自生成有且仅有一个

Class类。只用在使用到对应的类时，Java的JVM才会去加载类的信息，否在是不会去加载的，因为加载需要耗费

资源。而第一使用：是指第一次new 一个实例或者第一次使用到类的静态成员。

在类的加载阶段，JVM会首先检查要加载的类是否已经被加载过。如果没有被加载，类加载器会根据路类的全称

（全限定名）查找对应的.class文件。在类的字节码文件被加载的时候，会被检查其是否已经损坏，或者包含不良

的Java代码。一旦字节码文件被加载完，就会生成一个Class类。



一个字节码文件被加载到内存需要经过三个步骤：

1. 加载：类加载器同类的全限定名查询对应的.class文件，然后获取文件字节流所代表的的静态存储节后转化为

   方法调用数据接口，根据字节流的信息在Java的堆中生产对应的Class。

2. 连接：在链接阶段将验证.class文件的字节流信息时候符合JVM的要求，为字节流中的静态域分配内存，并设

   置类成员的初始默认值，如果有必要的话，将常量池中的信息引用转化为直接引用。

3. 初始化：到此开始真正执行类中定义的Java代码。执行该类的静态初始器和静态初始块，如果该类有父类的

   话，则优先对其父类进行初始化。



##### Class的获取方法

1. Class.forname("类的全限定名")
2. object.getClass();  //实例对象
3. 类名.class 

```java
package com.example;

public Class Exmaple{
    
	public static void main(String[] args){
        Class c2=Example.class;
		Class c1=Class.forName("com.example.Exmaple");
        Class c3=new Example().getClass();’
        System.out.printl(c1==c2 && c2==c3); //true  
    }
    
    /*值得注意的是：
    1. 用.class来创建对Class对象的引用时，不会自动地初始化该Class对象的成员，类对象的初始化阶段被延迟到了对静态方法或者非常数静态域首次引用时才执行。"loading example"并不会再指定到Class c2=Example.class;打印，而是在执行第二语句的时候打印出来。
    
    2. 其实对于任意一个Class对象，都需要由它的类加载器和这个类本身一同确定其在就Java虚拟机中的唯一性，也就是说，即使两个Class对象来源于同一个Class文件，只要加载它们的类加载器不同，那这两个Class对象就必定不相等。java虚拟机中使用双亲委派模型来组织类加载器之间的关系，来保证Class对象的唯一性。*/
    
    static{   //静态块，静态成员在类加载的时候，被加载到静态域
        System.out.println("loading example")
    }
     
}
```



##### Class 常用的方法

获取Field、Constructor、Method、Annotation的常用方法在相关类中介绍。

| 方法名                                                       | 描述                                         |
| ------------------------------------------------------------ | -------------------------------------------- |
| `Class<?> forName(String className)`                         | 根据全限定名获取类类型                       |
| `<U> Class<? extends U> asSubclass(Class<U> clazz)`          | 将一个Class对象转换成为指定了泛型的Class对象 |
| `String getName()`                                           | 获取全限定名                                 |
| `getSimpleName()`                                            |                                              |
| `Class<?> forName(String name, boolean initialize,                               ClassLoader loader)` | 根据全限定名指定类加载获取类类型             |
| `Class<?>[] getInterfaces()`                                 | 获取类的继承的接口                           |
| `ClassLoader getClassLoader()`                               | 获取其类加载器                               |
| `getClass()`                                                 | 获取Class的一个实例对象引用                  |
| `isInterface()`                                              | 是否是接口类型                               |
| `newInstance()`                                              | 实例一个对象                                 |
| `getSuperClass()`                                            | 获取其父类                                   |



##### Class与泛型

Class的引用就是器所指向的对象的确切类型，也就是确定是一个类型的引用类型，例如：

 Class obj=Integer.class。在JDK5出现后，允许开发人员对Class引用所指向的Class对象的类型进行限定，也就是

说可以对Class对象使用泛型语法。通过泛型语法，可以让编译器强制Class对象范围的类型检查。

```java
	public void example(){
        Class<? extends Number> tClass; //限定tClass的引用只能是 Number和其子类的类型引用
    }
```

