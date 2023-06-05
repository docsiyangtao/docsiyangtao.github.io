## 概述

Optional类，位于java.util包下，可以保存类型T的值，代表这个值存在；或者保存null，表示这个值不存在。使用这种方式可以更好的表达值是否存在，从而避免空指针。

## 使用

### 创建

1. static \<T> Optional\<T> of(T t) { ... } ：创建一个Optional实例，t必须不为空
2. static\<T> Optional\<T> empty() { ... } ：创建一个Optional实例，t可以为空
3. static \<T> Optional\<T> ofNullable(T t) { ... } ：创建一个空的Optional实例

```java
@Test
public void test() {
    Person person = new Person();
    Optional<Person> optionalPerson1 = Optional.of(person);
    
    person = null;
    Optional<Person> optionalPerson2 = Optional.ofNullable(person);
    
    Optional<Person> optionalPerson3 = Optional.empty();
    
    System.out.println(optionalPerson1);	// Optional[Person{name='null', age=0}]
    System.out.println(optionalPerson2);	// Optional.empty
    System.out.println(optionalPerson3);	// Optional.empty
}
```

### 判断

1. boolean isPresent() { ... } ：判断是否包含对象
2. void ifPresent(Consumer<? super T> consumer) { ... } ：如果包含对象，则对象作为参数，执行Consumer接口的实现代码

```java
@Test
public void test2() {
    Person person = new Person();
    
    Optional<Person> optionalPerson = Optional.ofNullable(person);
    System.out.println(optionalPerson.isPresent());     // true

    optionalPerson.ifPresent(System.out::println);      // Person{name='null', age=0}
}
```

### 获取

1. T get() { ... } ：如果Optional实例含有对象则返回，否则报错NoSuchElementException
2. T orElse(T other) { ... } ：如果Optional实例含有对象则返回，否则返回指定的other对象\<T>
3. T orElseGet(Supplier<? extends T> other) { ... } ：如果Optional实例含有对象则返回，否则返回由Supplier接口提供的对象
4. \<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X { ... } ：如果Optional实例含有对象则返回，否则抛出由Supplier接口实现提供的异常

```java
@Test
    public void test3() {
        Person person = new Person();
        Optional<Person> optionalPerson1 = Optional.of(person);
        System.out.println(optionalPerson1.get());       // Person{name='null', age=0}

        person = null;
        Optional<Person> optionalPerson2 = Optional.ofNullable(person);
        //System.out.println(optionalPerson2.get());      // java.util.NoSuchElementException: No value present

        Person person1 = optionalPerson2.orElse(new Person());
        System.out.println(person1);         // Person{name='null', age=0}

        Person person2 = optionalPerson2.orElseGet(Person::new);
        System.out.println(person2);         // Person{name='null', age=0}

        System.out.println(optionalPerson2.orElseThrow(() ->
                new RuntimeException("对象为空")
        ));                                  // java.lang.RuntimeException: 对象为空
    }
```

