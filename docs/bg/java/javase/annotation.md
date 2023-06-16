# 注解

## 概述

- 给开发者看的标记叫注释，而给程序看的标记叫注解，识别注解这种标记并结合反射的机制，可以自定义标记信息处理流程，使得代码更加简洁易用
- 从**JDK 5.0**开始，Java增加了对元数据(MetaData)的支持，也就是Annotation(注解)。
- Annotation其实就是**代码里的特殊标记**，这些标记可以在**编译**、**类加载**、**运行时**被读取，并执行相应的处理。
- 通过使用Annotation,程序员可以在不改变原有逻辑的情况下,在源文件中嵌入一些补充信息。代码分析工具、开发工具和部署工具可以通过这些补充信息进行验证或者进行部署。
- Annotation可以像修饰符一样被使用，可用于修饰**包**、**类**、**构造器**、**方法**、**成员变量**、**参数**、**局部变量**的声明，这些信息被保存在Annotation的“name=value”对中。
- 在JavaSE中，注解的使用目的比较简单，例如标记过时的功能，忽略警告等。在JavaEE/Android中注解占据了更重要的角色，例如用来配置应用程序的任何切面，代替JavaEE旧版中所遗留的繁冗代码和XML配置等。

## 功能

1. **生成文档相关的注解**。如@author、@version、@since等。
2. **编译时进行格式检查**。如@Override、@Deprecated、@SuppressWarnings
3. **跟踪代码依赖性，实现替代配置文件功能**。如@GetMapping、@ResponseBody

## 常见注解

- @Override：编译过程中检查是否重写父类或实现接口的方法，如果被标记的方法不是被重写的方法，则编译不通过。注意：重写方法不一定要加这个注解，但被这个注解标记的方式一定是重写了的方法！
- @Deprecated：被此注解标记的类或方法或构造器已经是过时了，不推荐再使用。
- @SuppressWarnings：抑制编译器警告。

## 元注解

- 用于**修饰其他注解的注解**，JDK5.0提供了4个标准的meta-annotation，分别为

- - @Retention：指定被修饰的Annotation的生命周期，使用时可指定**生命周期状态**

- - - RetentionPolicy.SOURCE：编译时校验，**编译后丢弃**
    - RetentionPolicy.CLASS：保留在**编译后的class文件中**，但虚拟机运行时不保留
    - RetentionPolicy.RUNTIME：保留在class文件中，**运行时保留**，便于用反射获取

- - @Target：指定被修饰的Annotation**可使用在哪些元素类型上**。如类、方法、方法参数、成员变量、构造器、包等。
  - @Documented：被其修饰的Annotation将被javadoc工具**提取成文档**（javadoc默认不保留注解），即文档中保留了注解。
  - @Inherited：被其修饰Annotation具有**继承性**。如父类被一个注解标记，则子类也将被该注解标记（可通过反射获取）。

## 自定义注解

1. 注解的声明类型为**@interface**
2. **添加元注解**，指明生命周期、可用于的元素类型、是否保留在javadoc文档中以及是否有继承性等
3. 成员变量以无参数方法的形式来声明，**类型可为数组**
4. 可指定成员变量的**默认值**（**default**关键字）
5. 如果只有一个参数成员，建议使用参数名为value，这样使用时就可以直接传值，如@MyAnno(xxx)，而不用套用格式（@MyAnno(value = xxx)），如果次此参数指定了默认值，则可以省略传值使用默认值，如（@MyAnno）
6. 没有成员定义的Annotation成为**标记**，包含成员变量的Annotation成为**元数据Annotation**
7. **自定义注解必须配上注解的信息处理流程才有意义**

如：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Inherited
public @interface MyAnnotation {
    String value() default "123";
}
```

## JDK8注解新特性

### 可重复注解

一个注解可多次使用在同一个元素上。如：在需要重复的注解@MyAnnotation中加上@Repeatable注解，成员值为MyAnnotations.class，同时在@MyAnnotations中，成员变量必须为MyAnnotation的数组对象，这样即可实现一个注解可多次使用在同一个元素上

```java
@Repeatable(MyAnnotations.class)
public @interface MyAnnotation {
    String value() default "123";
}
public @interface MyAnnotations {
    MyAnnotation[] value();
}
@MyAnnotation("2333")
@MyAnnotation("567")
private void method() {
}
```

而在JDK8以前只能以值传递的形式实现同样的效果

```java
public @interface MyAnnotation {
    String value() default "123";
}
public @interface MyAnnotations {
    MyAnnotation[] value();
}
@MyAnnotations({@MyAnnotation("233"), @MyAnnotation("567")})
private void method() {
}
```

**注意：MyAnnotations的声明周期要大于MyAnnotation，但MyAnnotations的使用范围应是MyAnnotation使用范围的子集，且是否可重复@Interited必须同时声明或不声明！**

### 类型注解

ElementType.TYPE_PARAMETER：表示该注解能写在类型变量的声明语句中（如泛型声明）

```java
@Target({ElementType.TYPE_PARAMETER})
public @interface MyAnnotation {
    String value() default "123";
}
class Cls<@MyAnnotation T> {
}
```

ElementType.TYPE_USE：表示该注解能写在使用类型的任何语句中

```java
@Target({ElementType.TYPE_USE})
public @interface MyAnnotation {
    String value() default "123";
}
public void show() throws @MyAnnotation RuntimeException {
    ArrayList<@MyAnnotation String> list = new ArrayList<>();
    int i = (@MyAnnotation int) 10L;
}
```
