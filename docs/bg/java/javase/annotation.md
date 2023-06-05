## 概述

Annotation，JDK1.5开始增加的队元数据的支持。作为代码的特殊标记，可以在编译、类加载、运行时被读取，并执行相应的处理，可实现在不改变原有逻辑的情况下，在源文件中嵌入一些补充信息。

可修饰包、类、构造器、方法、成员变量、参数、局部变量，信息保存在Annotation的"name = value"对中。

## 文档的注解

@author标明开发该类模块的作者，多个作者之间使用,分割

@version标明该类模块的版本

@see参考转向，也就是相关主题since从哪个版本开始增加的

@param对方法中某参数的说明，如果没有参数就不能写

@return对方法返回值的说明，如果方法的返回值类型是void就不能写

@exception对方法可能抛出的异常进行说明，如果方法没有用throws显式抛出的异常就不能写

其中：

@param @return 和 @exception这三个标记都是只用于方法的。param的格式要求:param形参名形参类型形参说明

@return的格式要求:@return返回值类型返回值说明exception的格式要求:exception异常类型异常说明param和

@exception可以并列多个

## 编译时注解

@Override限定重写父类方法，该注解只能用于方法

@Deprecated用于表示所修饰的元素（类、方法等）已过时。

@SuppressWarnings抑制编译器警告

## 元注解

用于修饰其他注解的注解

### @Retention

* 指明所修饰的注解的生命周期

* 取值：

  ```java
  public enum RetentionPolicy {
      SOURCE,		//java源文件中有效，编译器丢弃
      CLASS,		//class文件中保留，JVM丢弃，默认取值
      RUNTIME		//运行时有效，JVM可通过反射获取该注解
  }
  ```

### @Target

* 指明被修饰的注解可修饰哪些元素

* 取值：

  ```java
  public enum ElementType {
      TYPE,					//类
      FIELD,					//属性
      METHOD,					//方法
      PARAMETER,				//参数
      CONSTRUCTOR,			//构造器
      LOCAL_VARIABLE,			//局部变量
      ANNOTATION_TYPE,		//注解类
      PACKAGE,				//包
      TYPE_PARAMETER,			//泛型
      TYPE_USE				//使用类型的任何语句
  }
  ```

### @Documented

* 该元注解修饰的注解类将被javadoc工具提取成文档，默认情况下javadoc不包括注解
* 定义为Documented的注解必须设置Retention值为RUNTIME

### @Inherited

* 被其修饰的注解将具有继承性，如Class B继承了Class A，Class A使用了被@Inherited修饰的注解X，那么Class B也可以使用注解X

## 自定义注解

1. 注解声明为 @interface
2. 内部定义成员，通常使用value表示，可指定默认值
3. 如果没有成员，则只起标识作用
4. 自定义注解必须配上注解的信息处理流程（使用反射）才有意义

```java
public @interface MyAnnotation {
    String value() default "hello";
}
```

## JDK8新特性

### JDK8以前

同一注解修饰同一类型不能重复，需定义一个新的注解，值类型为注解类型

```java
public @interface MyAnnotations {
    MyAnnotation[] value();
}

@MyAnnotations({@MyAnnotation("hello"), @MyAnnotation("hi")})
public class Demo {
}
```

### 新特性

#### 重复注解

给注解@MyAnnotation添加新的注解@Repeatable，成员值为新定义的注解类 MyAnnotations.class，这样，@MyAnnotation就可以重复使用，注意，两个注解的生命周期及作用范围必须一致，同时两者的@Inherited一致

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(MyAnnotations.class)
public @interface MyAnnotation {
    String value() default "hello";
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MyAnnotations {
    MyAnnotation[] value();
}
```

@MyAnnotation可重复使用

```java
@MyAnnotation("hello")
@MyAnnotation("hi")
public class Demo {
}
```

#### 泛型注解

```java
@Target({ElementType.TYPE_PARAMETER})
public @interface MyAnnotation {

    String value() default "hello";

}
```

```java
class Generic<@MyAnnotation T> {
    
}
```

#### 类型注解

* 任何使用类型的地方都可以使用注解

```java
@Target({ElementType.TYPE_USE})
@Repeatable(MyAnnotations.class)
public @interface MyAnnotation {

    String value() default "hello";

}
```

```java
class Generic<@MyAnnotation T> {
    public void show() throws @MyAnnotation RuntimeException {
        ArrayList<@MyAnnotation  String> strings = new ArrayList<>();
        int num = (@MyAnnotation int) 10L;
    }
}
```

