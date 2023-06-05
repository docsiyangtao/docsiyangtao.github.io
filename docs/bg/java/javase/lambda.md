## Lambda表达式

### 概述

lambda表达式，即**匿名函数**，指一类无需定义标识符（函数名）的函数或子程序。

格式为

```java
lambda形参列表（接口中的抽象方法的形参列表 -> lambda体（重写的抽象方法的方法体）
 
(parameters) -> { expression }
```

java中lambda表达式的本质就是**接口的实例**！

### 类型

1. 无参、无返回

```java
Runnable r = () -> System.out.println("hello world");
r.run();
```

2. 有参、无返回

```java
Consumer<String> consumer = (String s) -> {
    System.out.println("接收的参数为：" + s);
};
consumer.accept("123");
```

3. 省略数据类型

```java
Consumer<String> consumer1 = (s) -> {
    System.out.println("接收的参数为：" + s);
};
consumer1.accept("你好");
```

4. 省略括号

```java
Consumer<String> consumer2 = s -> {
    System.out.println("接收的参数为：" + s);
};
consumer2.accept("hello");
```

5. 多参、多执行语句、有返回

```java
Comparator<Integer> comparator = (i, j) -> {
    System.out.println(i);
    System.out.println(j);
    return i.compareTo(j);
};
System.out.println(comparator.compare(1, 2));
```

6. 只有一条语句，省略大括号

```java
Comparator<Integer> comparator1 = (i, j) -> i.compareTo(j);
System.out.println(comparator1.compare(2, 3));
```

### 总结

1. 形参类型可以省略
2. 单个参数括号可以省略
3. 只有一条执行语句可以省略大括号及return关键字（同时）
4. 依赖函数式接口

