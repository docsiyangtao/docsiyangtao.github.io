# BigDecimal

在Java中金额计算一定不能使用Double、Float！！！一定要使用BigDecimal，即使是整数计算，也要养成习惯！！                                                     

## 精度丢失

浮点数进行计算会产生丢失精度的现象！这种舍入误差的主要原因是**浮点数值采用二进制系统表示**，而在**二进制系统中无法精确地表示分数1/10**。这就好像十进制无法精确地表示分数1/3一样。

如

```java
public void test() {
    System.out.println(2.0 - 1.9);
}
```

以上程序运行，控制台打印结果为`0.10000000000000009`，而不是0.1

## BigDecimal

Java专门提供了一个类**BigDecimal**，用于计算精确结果

### 常用构造方法

1. **BigDecimal(int val)**
2. **BigDecimal(long val)**
3. **BigDecimal(String val)**

```java
BigDecimal bd1 = new BigDecimal(1);
BigDecimal bd3 = new BigDecimal(3L);
BigDecimal bd4 = new BigDecimal("4.5");
```

不推荐使用double类型进行构造，因为浮点数无法准确的表示一个数，如1/10，前面已经说过，如

```java
BigDecimal bigDecimal = new BigDecimal(0.1);
System.out.println(bigDecimal);
```

打印结果为 0.1000000000000000055511151231257827021181583404541015625，构造方法中的参数（浮点数 0.1） 并不是我们理解上的0.1！

### 比较大小

对于BigDecimal对象，一般使用compareTo方法比较两个数的大小

```java
BigDecimal bd1 = new BigDecimal("1");
BigDecimal bd2 = new BigDecimal("2");
BigDecimal bd3 = new BigDecimal("2.0");

System.out.println(bd1.compareTo(bd2));     // -1：小于
System.out.println(bd2.compareTo(bd1));     // 1 ：大于
System.out.println(bd2.compareTo(bd3));     // 0 ：等于
```

### 常用计算方法

```java
BigDecimal bd1 = new BigDecimal("4.000");
BigDecimal bd2 = new BigDecimal("3");
// +
BigDecimal addRes = bd1.add(bd2);
// -
BigDecimal subRes = bd1.subtract(bd2);
// ×
BigDecimal mulRes = bd1.multiply(bd2);
// ÷
BigDecimal divRes = bd1.divide(bd2, RoundingMode.HALF_DOWN);

System.out.println(addRes);     // 7.000
System.out.println(subRes);     // 1.000
System.out.println(mulRes);     // 12.000
System.out.println(divRes);     // 1.333
```

注意除法较为特殊，需要指定另外一个参数“舍入规则”，如向正无穷大靠近、向负无穷大靠近、四舍五入，否则遇到无限循环小数时会抛出异常

其他加减乘方法可指定可不指定舍入规则

```java
throw new ArithmeticException("Non-terminating decimal expansion; " +
                                              "no exact representable decimal result.");
```

### 转换结果

BigDecimal对象可以转换为字符串、整形、浮点型，如

```java
BigDecimal bd = new BigDecimal("0.1");
int i = bd.intValue();
long l = bd.longValue();
double v = bd.doubleValue();
String s = bd.toString();

System.out.println(i);      // 0
System.out.println(l);      // 0
System.out.println(v);      // 0.1
System.out.println(s);      // 0.1
```

### 格式化

可使用类**DecimalFormat**对BigDecimal进行格式化

占位符`0`与占位符`#`

```java
DecimalFormat df1 = new DecimalFormat("00.00");
DecimalFormat df2 = new DecimalFormat("#.00");
DecimalFormat df3 = new DecimalFormat("##.##");

BigDecimal bd = new BigDecimal("0.3");
System.out.println(df1.format(bd));		// 00.30
System.out.println(df2.format(bd));		// .30
System.out.println(df3.format(bd));		// 0.3
```

从以上可以看出，**占位符`0`有多少个，最后的结果就有多少位，而`#`则只保留有效数字**

此外，**DecimalFormat对象也可以设置舍入规则**！

**案例：金额格式化**

小数点左边3为分隔，只保留有效数字，但最少为一位，右边则是保留两位数字，同时进行

```java
DecimalFormat df4 = new DecimalFormat(",##0.00");
df4.setRoundingMode(RoundingMode.UP);
BigDecimal bd2 = new BigDecimal("1233.230");
BigDecimal bd3 = new BigDecimal("13.232");
BigDecimal bd4 = new BigDecimal("0.232");
System.out.println(df4.format(bd2));        // 1,233.23
System.out.println(df4.format(bd3));        // 13.23
System.out.println(df4.format(bd4));        // 0.23
```
