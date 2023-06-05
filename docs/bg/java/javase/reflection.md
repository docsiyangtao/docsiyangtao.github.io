# 反射机制

## 概念

Reflection（反射）是被视为动态语言的关键，反射机制允许`程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法`。

加载完类之后，在堆内存的方法区中就产生了一个class类型的对象（一个类只有一个Class对象），这个亚象就包含了完整的类的结构信息。`我们可以通过这个对象看到类的结构`。这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为：反射。

传统：

```mermaid
graph LR;
A(引入需要的包类名称) --> B(通过new实例化) --> C(取得实例化对象)
```

反射：

```mermaid
graph LR;
A(实例化对象) --> B(getClass方法) --> C(得到完成的包类名称)
```

## 功能

* 判断对象所属类
* 构造一个类的对象
* 判断一个类具有的成员变量和方法
* 获取反省过信息
* 调用一个对象的成员变量和方法
* 运行时处理注解
* 生成动态代码

## API

### 获取Class

```java
//1. 调用运行时类的属性
Class clazz1 = Person.class;

//2. 捅过运行时类的对象
Person person = new Person();
Class clazz2 = person.getClass();

//3. Class的静态方法：forName(String classPath)
Class clazz3 = Class.forName("demo01.Person");

//4. 使用类的加载器
ClassLoader classLoader = Demo.class.getClassLoader();
Class clazz4 = classLoader.loadClass("demo01.Person");
```

### 创建实例

