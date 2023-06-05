## 概述

* 把真正的函数式编程风格引入到Java中，java.util.stream包下
* 提供了一种高效且易于使用的处理数据（主要为集合类）的方式，可以进行处理集合，执行查找、过滤、映射数据等操作
* Stream不改变源对象，会返回一个新的Stream
* 代码书写干净、简洁、高效

## 操作步骤

![image-20220213083140235](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220213083140235.png)

1. 创建Stream

   一个数据源，如集合、数组，获取一个流

2. 中间操作

   一个中间操作过程，对数据源进行处理

3. 终止操作

   执行终止操作，就开始执行中间操作链，并产生结果，执行终止操作后，stream流不再可操作性，如：对同一stream流执行两次遍历

   ```java
   public void test() {
       int[] ints = new int[]{1, 2, 3};
       IntStream stream = Arrays.stream(ints);
       stream.forEach(System.out::println);
       stream.forEach(System.out::println);
   }
   ```
   
报错
   
![image-20220213092419050](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220213092419050.png)

## Stream的创建

1. 通过集合创建

2. 通过数组创建

3. Stream.of(T... values)

4. 并行流

5. 无限流

```java
public void test() {
    // 通过集合类创建
    Stream<Integer> listStream = Collections.singleton(1).stream();
    // 通过数组创建
    IntStream arrayStream = Arrays.stream(new int[] {1, 2, 3});
    // 通过Stream类创建
    Stream<Integer> streamStream = Stream.of(1, 2, 3);
    // 并行流
    Stream<Integer> parallelStream = Collections.singleton(1).parallelStream();
    // 无限流
    Stream<Integer> iterateStream = Stream.iterate(1, s -> s + 1);
}
```

## 中间操作

中间操作可以拼接形成一个流水线

为方便测试，创建Person类和初始化列表List\<Person>，详见文章底部附[1]

### 筛选与切片

1. Stream\<T> filter(Predicate<? super T> predicate); 过滤某些元素
2. Stream\<T> limit(long maxSize); 限制长度
3. Stream\<T> skip(long n); 跳过某些元素
4. Stream\<T> distinct(); 去重，通过hashCode和equals判定重复

```java
public void test() {
    List<Person> list = initList();
    Stream<Person> stream = list.stream()
        .distinct()
        .filter(p -> p.getAge() > 10)
        .skip(2)
        .limit(5);

}
```

### 操作

1. Stream<T> peek(Consumer<? super T> action); 对流中元素进行操作

```java
public void test11() {
    int[] rank = {1};
    List<Person> list = initList();
    Stream<Person> stream = list.stream()
        .peek(p -> p.setRank(rank[0]++));	// 设置排名
}
```

### 排序

1. Stream\<T> sorted(); 默认的排序，自然排序
2. Stream\<T> sorted(Comparator<? super T> comparator); 通过Comparator接口自定义排序规则

```java
public void test() {
    Stream<Integer> stream1 = Stream.of(2, 1, 5, 0, 3)
        .sorted();  // 0, 1, 2, 3, 5

    List<Person> list = initList();
    Stream<Person> stream2 = list.stream()
        .sorted(Comparator.comparingInt(Person::getAge));
}
```

### 映射

1. \<R> Stream\<R> map(Function<? super T, ? extends R> mapper); 将流中每个值转换成另一个值并拼接成新的流
2. \<R> Stream\<R> flatMap(Function<? super T, ? extends Stream<? extends R>> mapper); 将流中的每个值转换为另一个流，再把所有的流都拼接成一个流

```java
public void test() {
    List<Person> list = initList();
    Stream<Integer> ageStream1 = list.stream()
        .map(Person::getAge);
    IntStream ageStream2 = list.stream()
        .mapToInt(Person::getAge);

    Stream<String> hobbyStream = list.stream()
        .flatMap(p -> p.getHobbies().stream())  // 篮球 足球 围棋 棒球 唱歌 桌球 健身 网游 游泳
        .distinct();
}
```

## 终止操作

### 匹配

1. boolean allMatch / anyMatch / noneMatch(Predicate<? super T> predicate); 是否 全部 / 部分 / 不 匹配断言型函数定义的规则
2. Optional\<T> findFirst(); 查找第一个元素
3. Optional\<T> findAny(); 查找任意一个元素

```java
public void test() {
    List<Person> list = initList();
    boolean b1 = list.stream()
        .allMatch(p -> p.getAge() > 20);	// false
    boolean b2 = list.stream()
        .anyMatch(p -> p.getAge() > 20);	// false
    boolean b3 = list.stream()
        .noneMatch(p -> p.getAge() > 20);	// true
}
```

### 查找

1. long count(); 获取流中元素的个数
2. Optional\<T> max / min(Comparator<? super T> comparator); 查找流中最大 / 小的元素，Comparator类型函数

```java
public void test() {
    List<Person> list = initList();
    long count = list.stream()
        .count();
    Optional<Person> maxAgePerson = list.stream()
        .max(Comparator.comparingInt(Person::getAge));	// 年龄最大的Person
    Optional<Person> minAgePerson = list.stream()
        .max(Comparator.comparingInt(Person::getAge));
}
```

### 遍历

1. void forEach(Consumer<? super T> action); 接收消费型函数，循环所有的元素

```java
public void test() {
    List<Person> list = initList();
    list.stream()
        .forEach(System.out::println);
}
```

### 规约

将流中的元素都结合起来，得到一个值，如计算总和

1. T reduce(T identity, BinaryOperator\<T> accumulator); 将所有的元素都结合起来得到一个值，并指定初始值
2. Optional\<T> reduce(BinaryOperator\<T> accumulator); 将所有的元素都结合起来得到一个值，无指定初始值
3. \<R> R reduce(R identity, BiFunction<R, ? super P_OUT, R> accumulator, BinaryOperator\<R> combiner); 并行流特有方法，并行流中的元素进行reduce操作，初始值为第1个参数，再将并行流的结果进行参数3的reduce操作

```java
public void test() {
    Integer sum1 = Stream.of(1, 2, 3, 4, 5, 6)
        .reduce(2, Integer::sum);       // 23
    Optional<Integer> sum2 = Stream.of(1, 2, 3, 4, 5, 6)
        .reduce(Integer::sum);          // 21
    List<Integer> list1 = Arrays.asList(1, 2, 3, 4, 5, 6);
    List<Integer> list2 = new ArrayList<>();
    List<Integer> list3 = list1.parallelStream()
        .reduce(list2,
                (l1, l2) -> {
                    ArrayList<Integer> tempList = new ArrayList<>(l1);
                    tempList.add(l2);
                    return tempList;
                },
                (l1, l2) -> {
                    ArrayList<Integer> tempList = new ArrayList<>(l1);
                    tempList.addAll(l2);
                    System.out.println(tempList);
                    return tempList;
                });
    System.out.println();
    System.out.println(list3);
    
    /* list2为：
        [5, 6]
        [4, 5, 6]
        [2, 3]
        [1, 2, 3]
        [1, 2, 3, 4, 5, 6]

        [1, 2, 3, 4, 5, 6]
    */
}
```

### 收集

1. <R, A> R collect(Collector<? super T, A, R> collector); 将流转换为其他形式，用于给流的元素做汇总

```java
public void test() {
    List<Person> personList = initList();
    List<Person> list = personList.stream()
        .collect(Collectors.toList());
    Set<Person> set = personList.stream()
        .collect(Collectors.toSet());
    LinkedList<Person> linkedList = personList.stream()
        .collect(Collectors.toCollection(LinkedList::new));
}
```

Collectors的其他常见方法：

| 方法         | 返回类型       | 作用                            |
| ------------ | -------------- | ------------------------------- |
| toList       | List\<T>       | 收集为List                      |
| toSet        | Set\<T>        | 收集为Set                       |
| toCollection | Collection\<T> | 收集为创建的集合                |
| counting     | Long           | 计算流中元素的个数              |
| summingInt   | Integer        | 对流中元素的整数属性求和        |
| averagingInt | Double         | 计算流中元素Integer属性的平均值 |

## 拓展

流式编程DEBUG，在添加断点的时候可以单独为一个中间操作加上断点，也可以加上行断点，对于整个中间操作过程可以通过【Trace Current Stream Chain】查看

![流式编程Debug](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/%E6%B5%81%E5%BC%8F%E7%BC%96%E7%A8%8BDebug.gif)



附[1]：

Person类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Setter
@Getter
public class Person {

    private String no;
    private String name;
    private int age;
    private List<String> hobbies;
    private int rank;

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Person person = (Person) o;
        return no.equals(person.no);
    }

    @Override
    public int hashCode() {
        return Objects.hash(no);
    }
}
```

测试列表List\<Person>

```java
public static List<Person> initList() {
    Person zs = new Person("002", "zs", 15, Arrays.asList("篮球", "足球"), 0);
    Person ls = new Person("001", "ls", 12, Arrays.asList("围棋", "足球"), 0);
    Person ww = new Person("005", "ww", 16, Arrays.asList("棒球", "唱歌"), 0);
    Person zl = new Person("004", "zl", 12, Arrays.asList("篮球", "桌球"), 0);
    Person tq = new Person("003", "tq", 13, Arrays.asList("健身", "网游"), 0);
    Person zb = new Person("006", "zb", 18, Arrays.asList("游泳", "足球"), 0);
    return Arrays.asList(zs, ls, ww, zl, tq, zb);
}
```



<center>【END】</center>