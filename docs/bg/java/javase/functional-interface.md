# 函数式接口

## 概述

* 接口只有一个抽象方法的接口
* 因为只有一个抽象方法，所以重写的时候没有歧义，本质就是该接口的实例
* **@FunctionalInterface**：通过该注解，可以标识这个类是函数是接口（没有标注不代表不是），编译时会校验该接口是否只有一个抽象方法。
* JDK8中新增了包`java.util.function`，包含众多类

## 常见函数式接口

|                          函数式接口                          |         参数类型          |         返回类型          |                         用途                          |
| :----------------------------------------------------------: | :-----------------------: | :-----------------------: | :---------------------------------------------------: |
|                         Consumer\<T>                         |             T             |           void            |      对类型为T的对象的应用操作，void accept(T t)      |
|                         Supplier\<T>                         |             /             |             T             |              返回类型为T的对象，T get()               |
|                        Function<T, R>                        |             T             |             R             |   对类型为T的对象操作，返回R类型对象，R apply(T t)    |
|                        Predicate\<T>                         |             T             |          boolean          | 参数\<T>是否满足条件，返回布尔类型，boolean test(T t) |
|                     BiFunction<T, U, R>                      |           T, U            |             T             |          可以看做是两个参数的 Function<T, R>          |
|                      UnaryOperator\<T>                       |             T             |             T             |      Function<T, R>的子接口，参数和返回类型都为T      |
|                      BinaryOperator\<T>                      |           T, T            |             T             |       参数和返回类型都是T的BiFunction<T, U, R>        |
|                       BiConsumer<T, U>                       |           T, U            |           void            |                两个参数的Consumer\<T>                 |
|                      BiPredicate<T, U>                       |           T, U            |          boolean          |                两个参数的Predicate\<T>                |
| ToIntFunction\<T><br />ToLongFunction\<T><br />ToDoubleFunction\<T> |             T             | int<br />long<br />double |           分别计算int、long、double值的函数           |
| IntFunction\<R><br />LongFunction\<R><br />DoubleFunction\<R> | int<br />long<br />double |             R             |         参数分别为int、long、double类型的函数         |

## 使用案例

### Consumer\<T>：消费型

打印购物消费了多少钱

```java
class Test {
    public void shopping(double money, Consumer<Double> consumer) {
        consumer.accept(money);
    }

    @Test
    public void test() {
        //重写抽象方法
        shopping(3.00, new Consumer<Double>() {
            @Override
            public void accept(Double aDouble) {
                System.out.println("购物消费了 " + aDouble + " 元");
            }
        });

        //直接使用lambda表达式
        shopping(400, money -> System.out.println("购物消费了 " + money + " 元"));
    }
}
```

### Supplier\<T>：供给型

生成一个长度为n的随机数列表

```java
class Test {
    public List<Integer> addNum(Integer num, Supplier<Integer> supplier) {
        ArrayList<Integer> list = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            list.add(supplier.get());
        }
        return list;
    }
    
    @Test
    public void test3() {

        List<Integer> list1 = addNum(10, new Supplier<Integer>() {
            @Override
            public Integer get() {
                return (int) (Math.random() * 100);
            }
        });

        //lambda表达式简写
        List<Integer> list2 = addNum(10, () -> (int) (Math.random() * 100));
    }
}
```

### Function<T, R>：函数型

字符串处理，去除字符串中的空格、将所有字符转为小写，并加上前缀

```java
class Test {
    public String handler(String str, Function<String, String> function) {
        return function.apply(str);
    }

    @Test
    public void test() {
        String s = " bai d u .c om ";

        String res1 = handler(s, new Function<String, String>() {
            @Override
            public String apply(String s) {
                String s1 = s.toLowerCase();
                return s1.replace(" ", "");
            }
        });

        //lambda表达式简写
        String res2 = handler(s, s1 -> {
            String s2 = s.toLowerCase();
            return s2.replace(" ", "");
        });
    }
}
```

### Predicate\<T>：断言型

过滤list中的奇数

```java
class Test {
    public List<Integer> myFilter(List<Integer> list, Predicate<Integer> predicate) {
        ArrayList<Integer> res = new ArrayList<>();
        for (Integer i : list) {
            if (predicate.test(i)) {
                res.add(i);
            }
        }
        return res;
    }
    
    @Test
    public void test2() {
        Integer[] arr = new Integer[]{1, 2, 3, 4, 5};
        List<Integer> list = Arrays.asList(arr);
        List<Integer> res1 = myFilter(list, new Predicate<Integer>() {
            @Override
            public boolean test(Integer integer) {
                return integer % 2 == 0;
            }
        });

        //lambda表达式简写
        List<Integer> res2 = myFilter(list, integer -> integer % 2 == 0);
        System.out.println(res2);
    }
}
```
