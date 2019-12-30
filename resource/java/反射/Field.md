## Java——反射Field类简单学习

Java 反射是一种常用的技术手段， 通过加载类的字节码的方式来获取相关类的一些信息 比如成员变量， 成员方法等。

**Field** 类是辅助Java反射的一个类，该类**可以动态获取类中的所有成员变量信息，也可以改变类的成员变量**。

* Field的实例对象获取的四中方式

  |             方法名             | 所属类 | 描述                                                    |
  | :----------------------------: | :----- | :------------------------------------------------------ |
  |         `getFields()`          | Class  | 获取类或者接口中public修饰的成员变量，返回的是Field数组 |
  |      `getDeclareFields()`      | Class  | 获取类或者接口中所有成员变量，返回的是Field数组         |
  |    `getField(String name)`     | Class  | 根据指定使用public修饰的类成员变量名，获取变量名的值    |
  | `getDeclareField(String name)` | Class  | 根据指定的类成员变量名，获取变量名的值                  |

  

* Field类常用的方法

  |                            方法名                            | 方法描述                                                     |
  | :----------------------------------------------------------: | ------------------------------------------------------------ |
  |                         `getType()`                          | 获取类成员的类型(泛型擦除)                                   |
  |                      `getGenericType()`                      | 获取类成员的类型（保留泛型）                                 |
  |                    `getDeclaringClass()`                     | 获取成员的声明的类型                                         |
  |                      `getGenericType()`                      | 获取类成员的签名属性，如果没有返回成员的类型                 |
  |                         `getName()`                          | 获取类成员的变量名                                           |
  |                      `get(Object obj)`                       | 获取指定的类成员的值，                                       |
  |               `set(Object obj,Object Value )`                | 设置指定对象的obj的值                                        |
  |                       `getModifiers()`                       | 以整数形式返回类成员上修饰符，如public，private              |
  |                      `isEnumConstant()`                      | 判断类成员是否是枚举类型                                     |
  |                  `setAccessible(boolean t)`                  | 修改安全机制，是否可以修改类的私有成员变量和常量的值，该方法是其父类`AccessibleObject`的方法 |
  |                       `siAccessible()`                       | 该成员时候可被访问                                           |
  | `isAnnotationPresent(Class<? extends Annotation> annotationClass)` | 判断类成员是有用指定类型的注解类型                           |
  |                  `getDeclaredAnnotations()`                  | 获取该成员上的所有注解，以数据形式返回，没有返回空数组，包括该成员的类型的类注解 |
  |                      `getAnnotations()`                      | 获取该成员上的所有注解，以数据形式返回，没有返回空数组，     |
  |                     `getAnnotatedTye()`                      | 获取类成员变量上的注解类型                                   |

  ##### 值得注意的是： #####

  set(Object obj, Object value)时， 新value 和原旧value 的类型不一致时，会导致类型转换异常【反射获取或者修改一个变量的值时， 编译器不会再自动拆装箱， 一些类型转换需要自己手动完成】