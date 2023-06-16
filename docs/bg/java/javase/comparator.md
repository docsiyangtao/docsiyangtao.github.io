# Java比较器

java中的基本数据类型，如int、long类型数据，可以使用 > < =进行比较大小，但是对于对象数据类型，则不能简单地通过比较符号进行比较，需要对象的类实现比较接口重写排序规则或通过Comparator进行定制排序。

## Comparable接口

含有方法compareTo(obj)，

- 如果当前对象this大于形参obj，则返回正数
- 如果当前对象this小于形参obj，则返回负数
- 如果当前对象this等于形参obj，则返回零

## 自定义类实现Comparable接口

自定义Person类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class Person {
    private Long id;
    private String name;
    private int age;
}
```

Person类如果需要比较大小，需要实现Comparable接口，重写CompareTo方法，给定具体的排序规则，如Person对象按年龄从小到大进行排序

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class Person implements Comparable {

    private Long id;
    private String name;
    private int age;

    @Override
    public int compareTo(Object o) {
        if (!(o instanceof Person)) {
            throw new RuntimeException("不能进行比较");
        }
        Person p = (Person) o;
        if (this.age > p.age) {
            return 1;
        } else if (this.age == p.age) {
            return 0;	// 还可以再次定义排序规则
        } else {
            return -1;
        }
    }
}
```

## Comparator定制排序

可以将Comparator这个接口作为参数传递给排序方法（如Collections.sort或Arrays.sort），重写方法compare(Object obj1, Object obj2)，比较obj1和obj2

- obj1大于obj2，则返回正数
- obj1小于obj2，则返回负数
- obj1等于obj2，则返回零

适用于对象没有实现java.lang.Comparable接口、或实现了Comparable接口但想重新定义排序规则的情况，如对person列表进行排序，规则为年龄相同，按id从小到大排序，年龄不同，按年龄从小到大进行排序

```java
public void test() {
    List<Person> personList = PERSON_LIST;
    personList.sort((o1, o2) -> {
        if (o1.getAge() != o2.getAge()) {
            // 年龄相同
            return Integer.compare(o1.getAge(), o2.getAge());
        }
        return Long.compare(o1.getId(), o2.getId());
    });
    System.out.println(personList);
}
```

## 两者对比

一种是通过类实现接口、重写比较方法，这样在每次使用的时候就不用重写排序方法，方便、但灵活性较差；

另一种则是在排序时重写排序方法，每次进行排序比较时需要重写比较方法，麻烦、但灵活性较高。