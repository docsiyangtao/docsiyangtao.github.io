# 枚举类的使用

## 入门

### 概念

* 类的对象是有限个的、确定的。
* 如：一周有：Monday、...、Sunday；季节有：Spring、...、Winter

### 用途

* 当需要定义一组常量时
* 需要实现单例模式时

### 定义枚举类

#### JDK5.0之前

定义一个Season类

```java
class Season {
    private final String seasonName;
    private final String seasonDesc;

    public Season(String seasonName, String seasonDesc) {
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }
    
    public static final Season SPRING = new Season("春天", "春暖花开");
    public static final Season SUMMER = new Season("夏天", "夏日炎炎");
    public static final Season AUTUMN = new Season("秋天", "秋高气爽");
    public static final Season WINTER = new Season("冬天", "冰天雪地");
    
}
```

使用：类名.属性名

```
Season.SPRING;	//("春天", "春暖花开")
Season.SUMMER;
Season.AUTUMN;
Season.WINTER;
```

#### JDK5.0以后

```java
enum Season {
    SPRING("春天", "春暖花开"),
    SUMMER("夏天", "夏日炎炎"),
    AUTUMN("秋天", "秋高气爽"),
    WINTER("冬天", "冰天雪地");

    private final String seasonName;
    private final String seasonDesc;

    Season(String seasonName, String seasonDesc) {
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    @Override
    public String toString() {
        return "Season{" +
                "seasonName='" + seasonName + '\'' +
                ", seasonDesc='" + seasonDesc + '\'' +
                '}';
    }
}
```

### 常用方法

| 方法                | 描述                          |
| ------------------- | ----------------------------- |
| values()            | 返回枚举类的对象数组          |
| valueOf(String str) | 根据str返回枚举类中对应的对象 |
| toString            | 枚举常量的名称                |

```
public class Demo {
    public static void main(String[] args) {

        Season[] values = Season.values();
        System.out.println(Arrays.toString(values));

        System.out.println(Season.valueOf("WINTER"));

        System.out.println(Season.SPRING);
    }
}
```

输出：

![image-20220726011506730](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220726011506730.png)

## 实现类

枚举类的每个对象可以分别重写实现类的方法

```java
interface itf {
    void show();
}

enum Season implements itf{
    SPRING("春天", "春暖花开") {
        @Override
        public void show() {
            System.out.println("SPRING的show");
        }
    },
    SUMMER("夏天", "夏日炎炎") {
        @Override
        public void show() {
            System.out.println("SUMMER的show");
        }
    },
    AUTUMN("秋天", "秋高气爽") {
        @Override
        public void show() {
            System.out.println("AUTUMN的show");
        }
    },
    WINTER("冬天", "冰天雪地") {
        @Override
        public void show() {
            System.out.println("WINTER的show");
        }
    };
    
    @Override
    public void show() {
        System.out.println("总体的show");
    }
    
    ...
}
```

```java
public class Demo {
    public static void main(String[] args) {
        Season.SPRING.show();
    }
}
```

![image-20220726011449156](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220726011449156.png)
