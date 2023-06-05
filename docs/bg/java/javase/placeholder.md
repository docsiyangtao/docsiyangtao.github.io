## API

* String format(String format, Object... args)：使用本地语言环境，制定字符串格式和参数生成格式化的新字符串
* String format(Locale locale, String format, Object... args)：使用指定语言环境，制定字符串格式和参数生成格式化的新字符串

## 转换符

```java
public void test1() {
    // 换行
    System.out.printf("%n");
    // 字符串
    System.out.println(String.format("%s", "hello world"));		// hello world
    // 字符
    System.out.printf("%c %n", 'H');							// H
    // 布尔
    System.out.printf("%b %n", 1 > 2);							// false
    // 整形（10进制）
    System.out.printf("%d %n", 100 / 2);						// 50
    // 整形（16进制）
    System.out.printf("%x %n", 100);							// 64
    // 整形（8进制）
    System.out.printf("%o %n", 100);							// 144
    // 浮点型（10进制，保留6位小数）
    System.out.printf("%f %n", 100.00);							// 100.000000
    // 指定小数位数的浮点型（四舍五入）
    System.out.printf("%.2f %n", 6.148 / 3);					// 2.05
    // 浮点型（16进制）
    System.out.printf("%a %n", 100.0);							// 0x1.9p6 
    // 指数型
    System.out.printf("%e %n", 10.04);							// 1.004000e+01
    // 通用浮点型（10进制，保留4位小数）
    System.out.printf("%g %n", 10.04);							// 10.0400
    // 百分号
    System.out.printf("%d%% %n", 100);							// 100%
    // 散列码
    System.out.printf("%h", 123);								// 7b
}
```

## 转换符标志

```java
public void test2() {
    // +：显示正负号
    System.out.printf("%+d %+d%n", 100, -100);           // +100 -100
    // -：左对齐
    System.out.printf("%-5d %n", 23);                    // 23
    // 0：左补零至3位
    System.out.printf("%03d%n", 1);                      // 007
    // x：左补空格至x位
    System.out.printf("%8d%n", 7);                       //        7
    // ,：逗号分隔符
    System.out.printf("%,d%n", 1231233);                 // 1,231,233
    // #：为浮点数添加小数、为8进制、16进制数添加标志
    System.out.printf("%#f %#x %#o %n", 2.3, 123, 321);  // 2.300000 0x7b 0501
    // $：被格式化的参数索引
    System.out.printf("%1$d %2$s", 123, "222");          // 123 222
}
```

## 日期时间组合格式转换符

```java
public void test3() {
    Date date = new Date();
    // tc：全部日期和时间信息
    System.out.printf("%tc %n", date);    // 星期四 二月 09 22:04:02 CST 2023
    // tD：月/日/年格式
    System.out.printf("%tD %n", date);    // 02/09/23
    // tF：年-月-日格式
    System.out.printf("%tF %n", date);    // 2023-02-09
    // tr：HH:mm:ss PM格式（12时制）
    System.out.printf("%tr %n", date);    // 10:05:56 下午
    // tR：HH:mm格式（24时制）
    System.out.printf("%tR %n", date);    // 22:07
    // tT：HH:mm:ss PM格式（24时制）
    System.out.printf("%tT %n", date);    // 22:07:02
}
```

## 日期格式转换符

```java
public void test4() {
    Date date = new Date();
    // ta：星期（简称）
    System.out.printf(Locale.US, "%ta %n", date);   // Thu
    System.out.print(String.format(Locale.JAPAN, "%ta %n", date));  // 木
    System.out.printf("%ta %n", date);      // 星期四
    // tA：星期（全称）
    System.out.printf("%tA %n", date);      // 星期四
    // tb：月份（简称）
    System.out.printf("%tb %n", date);      // 二月
    // tB：月份（全称）
    System.out.printf("%tB %n", date);      // 二月
    // tC：年分前两位
    System.out.printf("%tC %n", date);      // 20
    // ty：年分后两位
    System.out.printf("%ty %n", date);      // 23
    // tj：一年中的第几天
    System.out.printf("%tj %n", date);      // 040
    // tm：月份（全）
    System.out.printf("%tm %n", date);      // 02
    // td：日（全）
    System.out.printf("%td %n", date);      // 09
    // te：月份的日（简）
    System.out.printf("%te %n", date);      // 9
}
```

## 时间格式转换符

```java
public void test5() {
    Date date = new Date();
    // tH：小时（两位，24小时制）
    System.out.printf("%tH %n", date);      // 22
    // tI：小时（两位，12小时制）
    System.out.printf("%tI %n", date);      // 10
    // tk：小时（不补零位，24小时制）
    System.out.printf("%tk %n", date);      // 22
    // tl：小时（不补零位，12小时制）
    System.out.printf("%tl %n", date);      // 10
    // tM：分钟（两位）
    System.out.printf("%tM %n", date);      // 42
    // tS：秒（两位）
    System.out.printf("%tS %n", date);      // 20
    // tL：毫秒（三位）
    System.out.printf("%tL %n", date);      // 020
    // tN：毫秒（九位）
    System.out.printf("%tN %n", date);      // 020000000
    // tp：上午或下午标记
    System.out.printf("%tp %n", date);      // 下午
    // tz：相对于GMT的RFC822时区的偏移量
    System.out.printf("%tz %n", date);      // +0800
    // tZ：时区缩写字符串
    System.out.printf("%tZ %n", date);      // CST
    // ts：时间戳（秒）
    System.out.printf("%ts %n", date);      // 1675953740
    // tQ：时间戳（毫秒）
    System.out.printf("%tQ %n", date);      // 1675953740020
}
```

## MessageFormat

* String format(String pattern, Object ... arguments)：使用逗号分隔多参数

```java
public void test6() {
    String msg = MessageFormat.format("{0} {1}", "Hello", "World");
    System.out.println(msg);    // Hello World
}
```
