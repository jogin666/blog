## Java反射--Method的学习

**Method** 类是辅助Java反射的一个类，类位于`java.lang.reflect`包中。该类**可以动态获取类中的所有方法的信息，动态的调用对象的方法，Method不仅用于反射，也常被用于Java的代理**。

- Method获取的方法

  | 方法名                                                       | 所属类 | 方法描述                                 |
  | ------------------------------------------------------------ | ------ | ---------------------------------------- |
  | `getMethods()`                                               | Class  | 获取类中声明的**公有**方法，并以数组返回 |
  | `getDeclareMethods()`                                        | Class  | 获取类中声明的所有方法                   |
  | `getDeclaredMethod(String name, Class<?>... parameterTypes)` | Class  | 根据方法名和参数，获取指定的方法         |
  | `getMethod(String name, Class<?>... parameterTypes)`         | Class  | 根据方法名和参数或指定的公有方法         |



- 获取方法上的注解

  | 方法名                                                       | 方法描述                                                     |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | `<T extends Annotation> T  getAnnotation(Class<T> annotationClass)` | 如果存在该元素的指定类型的注释，则返回这些注释，否则返回 null。 |
  | `Annotation getDeclareAnnotations()`                         | 获取方法上的所有注解                                         |
  | `getDefaultValue()`                                          | 获取方法上的注解的默认值                                     |
  | `getParameterAnnotations()`                                  | 返回表示按照声明顺序对此 Method 对象所表示方法的形参进行注释的那个数组的数组 |



- 参数与返回值

  | 方法名                         | 描述                                |
  | ------------------------------ | ----------------------------------- |
  | `Class<?> getParamterTypes()`  | 获取方法的参数类型                  |
  | `Class<?> getExceptionTypes()` | 获取方法抛出的异常类型              |
  | `Class<?> getReturnType()`     | 获取方法的返回值类型                |
  | `int getModifiers()`           | 获取方法的权限级别，public、private |
  | `String getName()`             | 获取方法名                          |
  | `toGenericString()`            | 获取放全名的字符串                  |
  | `boolean isVarArgs()`          | 方法是否带有可变参数                |

- 权限方面

  `Method.setAccessible(Method method,Boolean t)` 修改指定的方法的权限访问机制，让方法

  可以访问。

- **`Object invoke(Object obj,Object... args)`** 调用指定**对象**上有指定**参数**的方法，该方法指

  多态。

  ```java
  public class Main{
      class People{
          public void speak(){
              System.out.println("people.speak");
          }
      }
  
      class Chinese extends People{
          @Override
          public void speak(){
              System.out.println("chinese speak");
          }
      }
  
      public static void main(String[] args){
          Chinese chinese=new Chinese();
          People people=new People();
  
          Method pmethod=People.class.getMethod("speak");
          Method cmethod=Chinese.class.getMethd("speak");
  
          pmethod.invoke(people); //people.speak
          pmethod.invoke(chinese); //chinese speak
          //子类重载父类的方法后，调用父类的方法，会转跳到子类重载的方法，支持多态
  
          cmethod.invoke(chinese); //chinese speak
          cmethod.invoke(people); //抛出IllegalArgumentException: object is 
          // not an instance of declaring class
  
      }
  }
  ```



- `Object invoke(Object obj,Object... args)`源码实现

  ```java
  @CallerSensitive
  public Object invoke(Object obj, Object... args)
      throws IllegalAccessException, IllegalArgumentException,
  InvocationTargetException {
      if (!override) { //方法是否被重载，重载的话，子类肯定有权限访问
  
          //快速检查方法的权限是否是public 
          if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
              //获取调用者的类型
              Class<?> caller = Reflection.getCallerClass();
              //检查调用者是否有权限访问方法
              checkAccess(caller, clazz, obj, modifiers);	//检查权限
          }
      }
      MethodAccessor ma = methodAccessor;             // read volatile
      if (ma == null) {
          ma = acquireMethodAccessor(); //获取执行者
      }
      return ma.invoke(obj, args);
  }
  
  private MethodAccessor acquireMethodAccessor() {
      // First check to see if one has been created yet, and take it
      // if so
      MethodAccessor tmp = null;
      if (root != null) tmp = root.getMethodAccessor();
      if (tmp != null) {
          methodAccessor = tmp;
      } else {
          //通过方法实例从映射工厂中获取方法执行者
          tmp = reflectionFactory.newMethodAccessor(this);
          setMethodAccessor(tmp);
      }
  
      return tmp;
  }
  
  MethodAccessor getMethodAccessor() {
      return methodAccessor;
  }
  
  
  void setMethodAccessor(MethodAccessor accessor) {
      methodAccessor = accessor;
      // Propagate up
      if (root != null) {
          root.setMethodAccessor(accessor);
      }
  }
  
  Method copy() {  //拷贝方法
      Method res = new Method(clazz, name, parameterTypes, returnType,
                              exceptionTypes, modifiers, slot, signature,
                              annotations, parameterAnnotations, 
                              annotationDefault);
      res.root = this;
      res.methodAccessor = methodAccessor;  //多个实例的methodAccesor都是同时同一引用
      return res;
  }
  ```

  override是Method继承祖父类Accessibale的成员，通过override可以快速初步检测是否有权限访问。在Method中的实例对象中，override默认是false的，需要调用父类的`setAccessiable(Method method,boolean t)`修改权限。顺便提一下，Field类的实例对象的override默认也是false；

  ```java
  //传入一个Method的数组，修改method的访问权限
  public static void setAccessible(AccessibleObject[] array, boolean flag) 
      throws SecurityException {   
      SecurityManager sm = System.getSecurityManager();   
      if (sm != null) 
          sm.checkPermission(ACCESS_PERMISSION);   
      for (int i = 0; i < array.length; i++) {     
          setAccessible0(array[i], flag);    
      }
  }
  
  private static void setAccessible0(AccessibleObject obj, boolean flag)
      throws SecurityException{
      if (obj instanceof Constructor && flag == true) {
          Constructor<?> c = (Constructor<?>)obj;
          if (c.getDeclaringClass() == Class.class) {
              throw new SecurityException("Cannot make a java.lang.Class" +
                                          " constructor accessible");
          }
      }
      obj.override = flag;
  }
  ```

**总结**：

1. Method支持多态，也就是向上转型，

2. 框架在执行`method.invoke()`，会在获取method实例对象时，一般先调用一次`setAccessable(true)`，使得后面每次调用invoke()时，节省一次方法修饰符的判断，略微提升性能。业务允许的情况下，Field同样可以如此操作。





参考文章：https://blog.csdn.net/wenyuan65/article/details/81145900

