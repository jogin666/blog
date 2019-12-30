## Java反射——Constructor

`Constructor`类是辅助Java反射的一个类，该类包含了类中所有改造函数的信息，通过该类可以动态生

成类的实例。

`Contructor`的获取方法

| 方法名                                                       | 所属类 | 描述                                         |
| ------------------------------------------------------------ | ------ | -------------------------------------------- |
| `Constructor<T> getConstructor(Class<?>... parameterTypes)`  | Class  | 根据指定参数类，获取指定公有的构造函数的实例 |
| `Constructor<?>[] getConstructors()`                         | Class  | 获取所有类中声明为公有的构造函数             |
| `Constructor<?>[] getDeclaredConstructors()`                 | Class  | 获取所有类中声明的构造函数                   |
| `Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes)` | Class  | 根据指定参数类，获取指定的构造函数的实例     |

`Constructor`常用的方法

* 实例、参数、权限

| 方法名                                               | 方法描述                    |
| ---------------------------------------------------- | --------------------------- |
| `T newInstance(Object ... initargs)`                 | 创建实例对象                |
| `Class<T> getDeclaringClass()`                       | 返回`Constructor`属于的类型 |
| `String getName()`                                   | 获取构造函数的方法名        |
| `int getParamterCount()`                             | 返回构造函数的参数个数      |
| `Class<?>[] getParamterTypes()`                      | 返回参数的类型              |
| `TypeVariable<Constructor<T>>[] getTypeParameters()` | 获取参数化类型（泛型）      |
| `boolean isVarArgs()`                                | 是否有参数                  |
| `Class<?>[] getExceptionTypes()`                     | 获取异常类型                |
| `int getModifiers()`                                 | 获取权限级别                |

* 关于注解和构造函数描述

| 方法名                                                       | 描述                                         |
| ------------------------------------------------------------ | -------------------------------------------- |
| `<T extends Annotation> T getAnnotation(Class<T> annotationClass)` | 获取指定的注解类型                           |
| `Annotation[] getDeclaredAnnotations`                        | 获取构造函数上的所有注解类型，以数组形式返回 |
| `Annotation[][] getParameterAnnotations()`                   | 获取构造函数的参数的注解类型                 |
| `String  toString()`                                         | 获取构造函数的字符串描述（泛型擦除）         |
| `String toGenericString()`                                   | 获取构造函数的字符串描述                     |

* 权限访问

| 方法                          | 描述                     |
| ----------------------------- | ------------------------ |
| `isAccessible()`              | 该构造函数是否可以被访问 |
| `setAccessible(boolean flag)` | 修改构造函数的访问权限   |

* `Object newInstance(Object ... initargs)`源码

```java
	@CallerSensitive
    public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException{
        if (!override) { //是否重载
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, null, modifiers);
            }
        }
                   //是否拥权限
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum"
                                               +"objects");
                   //构造函数执行者
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
                   //实例对象
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }
```

值得注意的是：override 是祖父类`AccessibleObject`继承而来的，Constructor的实例默认是false