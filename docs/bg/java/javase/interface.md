## 概述

* 接口是对类的一组需求描述，这些类要遵从接口描述的统一格式进行定义
* 所含方法全是public类型，所有变量都为常量（static  final），无需显式声明
* 接口中可以定义简单方法及静态方法，方法不能引用实例域
* 一个类实现一个接口后，必须重写接口中的全部方法，可通过`instanceof`检查类是否继承了接口

* 接口不能实例化，但可以声明接口变量

## 对比抽象类

* 一个类可以实现多个接口，但只能继承一个类

  ```java
  class MyCls implements MyItf1, MyItf2 {  }
  ```

* 作用不一致，实现接口关注的是类的“功能”，继承类关注的是类的“属性”，如一个Student类可以实现多个接口如work、study等，但只能继承一个类，如Person类，不能又继承Person类又继承Animal类

  ```java
  class Student extends Person implements Work, Study {  }
  ```


## Java8新特性

### 静态方法

* 接口中`可以定义静态方法，但不推荐`，因为违背了抽象规范的初衷
* 通常将静态方法放在伴随类中，如Collection的伴随类Collections
* 接口的静态方法`与普通类的静态方法使用无差别`，`Class.method()`

  ```java
  interface Vehicle {
      static void show() {
          System.out.println("Vehicle show.");
      }
  }
  
  class Test {
      @Test
      public void test() {
          Vehicle.show();		// Vehicle show.
      }
  }
  ```

### 默认方法

* 接口中使用default修饰的方法即是默认方法

  ```java
  public interface MyItf {
      default String getName() {
          return this.getClass().getName();
      }
  }
  ```

* 默认方法有方法体，且实现类可以不用重写该方法即可通过实例对象调用该方法

  ```java
  public class MyCls implements MyItf { 
  	@Test
      public void test() {
          System.out.println(new MyCls().getName());
      }
  }
  ```

* 当一个接口被实现以后，如果新增了新的方法，那么以前的实现类则必须重写新方法，但如果新方法被定义为默认方法，则以前的实现类就没必要重写新方法了，可以提高代码兼容性

* `当类实现多个接口，且多个接口中含有（返回值、方法名、形参列表）相同的默认方法，则实现类需要手动重写该方法`

  ```java
  public interface MyItf2 {
      default String getName() {
          return this.getClass().getName();
      }
  }
  
  public class MyCls implements MyItf, MyItf2 { 
      // 由于MyItf和MyItf2中含有相同的默认方法，因此需要重写
  	@Override
      public String getName() {
          // ...
      }
  }
  ```
  


## 类优先原则

* 当一个类集成了一个超类，同时实现了一个接口，且被实现的类和被继承的接口含有相同的方法，那么这个类只会考虑超类的方法，接口中的所有与超类相同的默认方法都将被忽略，这就是“类优先”原则

  ```java
  public abstract class MyAbsCls {
      public String getName() {
          return "MyAbsCls" + this.getClass().getName();
      }
  }
  
  public interface MyItf {
      default String getName() {
          return "MyItf: " + this.getClass().getName();
      }
  }
  
  public class Demo  {
      @Test
      public void test() {
  		 System.out.println(new Demo().getName());		// MyAbstClslive.yangtao.Demo
      }
  }
  ```

