# 反射

## 概述

Reflection（反射）是被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。

加载完类之后，在堆内存的方法区中就产生了一个class类型的对象（一个类只有一个class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为 反射。

## 动态&静态

动态语言是一类在运行时可以改变其结构的语言:例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其他结构上的变化。通俗点说就是在运行时代码可以根据某些条件改变自身结构。主要动态语言:Object-C、C#、JavaScript、PtHP、Pythoh、Erlang。

静态语言与动态语言相对应的，运行时结构不可变的语言就是静态语言。如Java、C++。

Java不是动态语言，但Java可以称之为“准动态语言”。即Java有一定的动态性，我们可以利用反射机制、字节码操作获得类似动态语言的特性。Java的动态性让编程的时候更加灵活！

## 应用

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时获取泛型信息
- 在运行时调用任意一个对象的成员变量和方法
- 在运行时处理注解
- 生成动态代理

## 关于Class

### Class的理解

1. java文件经过javac命令编译以后生成了.class字节码文件
2. 通过java命令解释运行字节码文件，会将字节码文件加载到内存中
3. 加载到内存中的类，就成为运行时类，即类java.lang.Class的一个实例
4. 不只是外部类，各种内部类、接口、数组、枚举、注解、基本数据类型、void都是Class对象

### 获取

1. 通过运行时类的属性

```java
Class<Person> clazz1 = Person.class;
```

1. 通过运行时类的对象

```java
Person p = new Person();
Class<? extends Person> clazz2 = p.getClass();
```

1. 通过全类名加载，**此种方式更能体现反射的动态性**

```java
Class<?> clazz3 = Class.forName("com.pgmyt.domain.Person");
```

1. 通过类的加载器加载

```java
Class<?> clazz4 = this.getClass().getClassLoader().loadClass("com.pgmyt.domain.Person");
```

4种方式获取的类（Class对象）是同一个，因为载到内存中的运行时类，会缓存一定的时间，在此时间内，不同方式获取的Class对象都是同一个

```java
System.out.println(clazz1 == clazz2);   // true
System.out.println(clazz1 == clazz3);   // true
System.out.println(clazz1 == clazz4);   // true
```

不止这样，只要元素类型和维度一样，就是同一个类

```java
int[] arr1 = new int[10];
int[] arr2 = new int[100];
System.out.println(arr1.getClass() == arr2.getClass());	  // true
```

### 类的加载过程

见 JVM/类加载子系统

### 类加载器

见 JVM/类加载子系统

## API

### 创建实例

```java
public void test() throws Exception {
    Class<Person> clazz = Person.class;
    // 本质上还是调用类的空参构造器，无满足权限的空参构造器则直接抛出异常
    Person person = clazz.newInstance();
}
```

### 属性结构

```java
public void test() throws NoSuchFieldException, IllegalAccessException {
    Class<ArrayList> clazz = ArrayList.class;
    // 获取自身及父类中访问权限为public的属性
    Field[] fields = clazz.getFields();

    // 获取当前类所有访问权限的属性，但不包含父类中声明的属性
    Field[] declaredFields = clazz.getDeclaredFields();

    // 获取具体的属性：可能会抛出异常NoSuchFieldException，找不到此属性
    Field size = clazz.getDeclaredField("size");

    // 权限修饰符：java/lang/reflect/Modifier.java
    int modifiers = size.getModifiers();

    // 数据类型
    Class<?> type = size.getType();

    // 名称
    String name = size.getName();
}
```

### 方法结构

```java
public void test() throws Exception {
    Class<ArrayList> clazz = ArrayList.class;
    // 获取当前运行时类及其父类中所有访问权限为public的方法
    Method[] methods = clazz.getMethods();

    // 获取当前类所有访问权限的方法，但不包含父类中声明的方法
    Method[] declaredMethods = clazz.getDeclaredMethods();

    // 访问修饰符
    Method add = clazz.getMethod("add", Object.class);
    int modifiers = add.getModifiers();

    // 返回类型
    Class<?> returnType = add.getReturnType();

    // 方法名
    String name = add.getName();

    // 形参列表
    Parameter[] parameters = add.getParameters();

    // 方法抛出的异常（由此对象表示的基础可执行文件引发的异常）
    Class<?>[] exceptionTypes = add.getExceptionTypes();

	// 方法抛出的异常（由此可执行对象引发的异常）
	Type[] genericExceptionTypes = add.getGenericExceptionTypes();

    // 注解
    Annotation[] annotations = add.getAnnotations();
}
```

### 注解

```java
@Test
public void test() throws Exception {
    Class<TestClient> clazz = TestClient.class;
    Method test = clazz.getMethod("test");
	// 获取方法的注解
    Test annotation = test.getAnnotation(Test.class);

	// 获取当前运行时类的所有注解
    Annotation[] annotations = clazz.getAnnotations();

	// 同理可获取方法形参的注解
	Parameter[] parameters = test.getParameters();

	// 获取注解中属性的值
    long timeout = annotation.timeout();
	Class<? extends Throwable> expected = annotation.expected();
}
```

### 获取父类

```java
public void test() {
    Class<ArrayList> clazz = ArrayList.class;
    // 当前运行时类的父类
    Class<? super ArrayList> superclass = clazz.getSuperclass();

    // 带泛型的父类
    Type genericSuperclass = clazz.getGenericSuperclass();
    
    // 获取泛型类型
    ParameterizedType parameterizedType = (ParameterizedType) genericSuperclass;
    Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
}
```

### 获取接口

```java
public void test() {
    Class<ArrayList> clazz = ArrayList.class;
    // 获取当前运行时类实现的接口
    Class<?>[] interfaces = clazz.getInterfaces();
}
```

### 所在包

```java
public void test() {
    Class<ArrayList> clazz = ArrayList.class;
	// 获取当前运行时类所在的包
    Package aPackage = clazz.getPackage();
}
```

### 属性相关方法

```java
public void test() throws Exception {
    Class<Person> clazz = Person.class;
    // 获取指定的属性
    Field age = clazz.getDeclaredField("age");
    Person zs = new Person(1L, "zs", 23);
    
    // 如果属性的访问权限非public，则需要设置可访问性
    age.setAccessible(true);

    // 设置对象的属性值
    age.set(zs, 18);

    // 获取对象的属性值
    Object ageObj = age.get(zs);
}
```

### 调用方法

```java
public void test() throws Exception {
    Class<Person> clazz = Person.class;
    Person zs = new Person(1L, "zs", 23);

    // 获取当前运行时泪的私有方法
    Method privateShow = clazz.getDeclaredMethod("privateShow", String.class);

    // 设置可访问性
    privateShow.setAccessible(true);

    // 调用对象的方法（含返回值）
    Object invoke = privateShow.invoke(zs, "123");
    
    // 对于静态方法，方法的参数则是类对象，而不是运行时类对象
    Method staticShow = clazz.getDeclaredMethod("staticShow");
    staticShow.setAccessible(true);
    staticShow.invoke(clazz);
}
```

### 调用构造器

```java
public void test() throws Exception {
    Class<Person> clazz = Person.class;

	// 无参构造：本质上还是调用类的空参构造器
    Person person = clazz.newInstance();

	// 获取类的有参构造器时，参数为构造器参数类型类对象
    Constructor<Person> declaredConstructor = clazz.getDeclaredConstructor(long.class, String.class, int.class);

	// 调用构造器，创建实例
	declaredConstructor.setAccessible(true);
	Person person = declaredConstructor.newInstance(1L, "zs", 12);
}
```

### 总结

获取到了当前运行时类对象，则可以使用该对象创建实例、设置属性、调用方法即构造器，同时还可以获取类的结构信息，通过这些信息做进一步处理。

在获取类对象的属性、方法、构造器时应注意参数的准确性，避免NoSuchMethodException、NoSuchFieldException、InvocationTargetException、InstantiationException，同时还需要注意私有属性、方法、构造器的可访问性问题，获取私有的需要使用getDeclaredXxxxx，如果需要进一步的操作，还需设置可访问性xxx.setAccessible(true);
