# 方法引用

## 概述

* 当要传递给lambda体的操作，**已经有实现的方法了**，可以使用方法引用
* 方法引用本质上就是lambda表达式，而lambda表达式作为函数式接口的实例，所以**方法引用也是函数式接口的实例**
* **实现接口的抽象方法的参数列表和返回值类型必须与方法引用的方法的参数列表和返回值类型保持一致**

## 格式

* **对象 : : 实例方法名**
* **类 : : 静态方法名**
* **类 : : 实例方法名**
* **类 : : new**

## 使用案例

0. 提供一个公共的Person类，便于测试

   ```java
   @Data
   @AllArgsConstructor
   @NoArgsConstructor
   public class Person {
       private String name;
       private Integer age;
       private String id;
   }
   ```

1. ### **对象 : : 实例方法名**

   获取实例对象的年龄

   ```java
   public class Demo1 {
       @Test
       public void test1() {
           Person p = new Person("张三", 14, "002");
           // 常规
           Supplier<Integer> supplier1 = new Supplier<Integer>() {
               @Override
               public Integer get() {
                   return p.getAge();
               }
           };
           
           // lambda
           Supplier<Integer> supplier2 = () -> p.getAge();
           
           // 方法引用：lambda表达式的形参和返回值与实例方法完全一致，只有方法名不一致
           Supplier<Integer> supplier3 = p::getAge;
           
           System.out.println(supplier3.get());	// 14
       }
   }
   ```

2. ### **类 : : 静态方法名**

   Person类中定义一个静态方法`getNominalAge`

   ```java
   public static Integer getNominalAge(Integer age) {
       return age + 1;
   }
   ```

   传入一个实际年龄即可获取虚岁

   ```java
   public class Demo {
       @Test
       public void test() {
           // 重写Function
           Function<Integer, Integer> function1 = new Function<Integer, Integer>() {
               @Override
               public Integer apply(Integer i) {
                   return Person.getNominalAge(i);
               }
           };
   		
           // lambda表达式
           Function<Integer, Integer> function2 = i -> Person.getNominalAge(i);
   
           // 方法引用：接口的实现依旧与Person类中的getNominalAge一致，可直接使用方法引用
           Function<Integer, Integer> function3 = Person::getNominalAge;
           
           System.out.println(function3.apply(2));		// 3
       }
   }
   ```

3. ### **类 : : 实例方法名**

   Person类中定义一个方法`ageDiff`

   ```java
   public Integer ageDiff(Person p) {
       return Math.abs(p.getAge() - this.getAge());
   }
   ```

   获取两个Person实例的年龄差

   ```java
   public class Demo {
       @Test
       public void test() {
           // 重写BiFunction
           BiFunction<Person, Person, Integer> biFunction1 = new BiFunction<Person, Person, Integer>() {
               @Override
               public Integer apply(Person p1, Person p2) {
                   return p1.ageDiff(p2);
               }
           };
           
           // lambda表达式
           BiFunction<Person, Person, Integer> biFunction2 = (p1, p2) -> p1.ageDiff(p2);
   
           /*
           这里lambda表达式与Person类中的ageDiff形参列表并不相同，但是仍可以用方法引用，
           因为Person类中的ageDiff形参虽然只有一个，但实际比较时参数this被隐藏了，
           this作为形参调用实例方法ageDiff，所以可以使用方法引用
           */
           BiFunction<Person, Person, Integer> biFunction3 = Person::ageDiff;
   
           Person zs = new Person("zs", 14, "001");
           Person ls = new Person("ls", 16, "001");
           System.out.println(biFunction3.apply(zs, ls));	// 2
       }
   
   }
   ```

4. ### **类 : : new**

    也称为**构造器引用**

    ```java
    public Person(String name) {
        this.name = name;
    }
    ```
    
    通过一个String类型的姓名构造一个Person对象
    
    ```java
    public class Demo {
        @Test
        public void test() {
            // 重写Function
            Function<String, Person> function1 = new Function<String, Person>() {
                @Override
                public Person apply(String s) {
                    return new Person(s);
                }
            };
    
            // lambda
            Function<String, Person> function2 = s -> new Person(s);
    
            // 构造方法默认返回一个Person对象（this），因此可以使用方法引用
            Function<String, Person> function3 = Person::new;
    
            System.out.println(function3.apply("ww"));	// Person(name=ww, age=null, id=null)
        }
    }
    ```
    
    这里有个坑：如果Person类中还有一个构造函数，假设为通过String类型的id构造Person
    
    ```java
    public Person(String id) {
        this.id = id;
    }
    ```
    
    此时就不能再使用lambda表达式及方法引用，因为Person类中有两个参数为String、返回值为Person的方法，无法判断到底是通过name构造还是通过id构造！
