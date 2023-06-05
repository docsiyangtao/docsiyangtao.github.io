## 概述

* Date类和Calendar类存在问题：可变性、偏移性、Calendar无法格式化、线程不安全、无法处理闰秒等
* 新的API：java.time，包含：本地日期（LocalDate）、本地时间（LocalTime）、本地日期时间（LocalDateTime）、时区（ZonedDateTime）、持续时间（Duration）
* Date类也增加了toInstant()方法用于把Date转换成新的表示形式

## 使用举例

注：无特殊说明一般LocalDateTime实例的方法在LocalDate或LocalTime中有相同的或类似的，由于篇幅原因只列举LocalDateTime的API

### 创建日期时间

```java
public void test() {
    // 获取当前时间
    LocalDateTime time1 = LocalDateTime.now();
    LocalDateTime time2 = LocalDateTime.now(ZoneId.systemDefault());
    LocalDateTime time3 = LocalDateTime.now(Clock.systemDefaultZone());

    // 指定时间
    LocalDateTime time4 = LocalDateTime.of(2022, 2, 2, 13, 34, 56, 233);

    // 从字符串解析时间
    LocalDateTime time5 = LocalDateTime.parse("2022-03-21T09:23:55.233");
    LocalDateTime time6 = LocalDateTime.parse("2022-02-23 07:12:45", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));

    // 时间戳
    LocalDateTime time7 = LocalDateTime.ofInstant(Instant.now(), ZoneId.systemDefault());
    LocalDateTime time8 = Instant.ofEpochMilli(1656866702010L).atZone(ZoneId.systemDefault()).toLocalDateTime();
}
```

### 获取年月日时分秒

以时间`2022-02-15T21:23:54.401`为例

```java
public void test() {
    LocalDateTime localDateTime = LocalDateTime.now();      // 2022-02-15T21:23:54.401
    int year = localDateTime.getYear();                     // 2022
    Month month = localDateTime.getMonth();                 // FEBRUARY，英文
    int monthValue = localDateTime.getMonthValue();         // 2，value为数值
    int dayOfYear = localDateTime.getDayOfYear();           // 46
    int dayOfMonth = localDateTime.getDayOfMonth();         // 15
    DayOfWeek dayOfWeek = localDateTime.getDayOfWeek();     // TUESDAY
    int hour = localDateTime.getHour();                     // 21
    int minute = localDateTime.getMinute();                 // 23
    int second = localDateTime.getSecond();                 // 54
    int nano = localDateTime.getNano();                     // 401000000，纳秒
}
```

### 修改日期时间

LocalDateTime的修改并不是修改原有的时间，而是返回一个新的想同类型的日期时间，可直接修改对应的年月日时分秒，也可使用with，接收指定日期进行修改

```java
public void test() {
    LocalDateTime localDateTime = LocalDateTime.now();
    LocalDateTime localDateTime1 = localDateTime.withYear(2023);
    LocalDateTime localDateTime2 = localDateTime.withMonth(3);
    LocalDateTime localDateTime3 = localDateTime.withDayOfMonth(23);
    LocalDateTime localDateTime4 = localDateTime.withHour(12);
    LocalDateTime localDateTime5 = localDateTime.withMinute(23);
    LocalDateTime localDateTime6 = localDateTime.withSecond(33);
    LocalDateTime localDateTime7 = localDateTime.withNano(233000000);
    LocalDateTime localDateTime8 = localDateTime.withDayOfYear(233);
}
```

### 时间矫正器

根据一个日期获取指定的时间，如本月第一天、最后一天、下个月第一天等等指定日期，都是返回新的本地时间

```java
public void test() {
    // 本月第一天
	localDateTime.with(TemporalAdjusters.firstDayOfMonth());
    // 本月最后一天
    localDateTime.with(TemporalAdjusters.lastDayOfMonth());
    // 下个月第一天
    localDateTime.with(TemporalAdjusters.firstDayOfNextMonth());
    // 今年第一天
    localDateTime.with(TemporalAdjusters.firstDayOfYear());
    // 今年最后一天
    localDateTime.with(TemporalAdjusters.lastDayOfYear());
    // 明年第一天
    localDateTime.with(TemporalAdjusters.firstDayOfNextYear());
    // 本月第一个周一
    localDateTime.with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY));
    // 本月最后一个周一
    localDateTime.with(TemporalAdjusters.lastInMonth(DayOfWeek.MONDAY));
    // 本月开始算起，第n个周几，-1表示本月最后一个
    localDateTime.with(TemporalAdjusters.dayOfWeekInMonth(-1, DayOfWeek.MONDAY));
    // 下一个周一
    localDateTime.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
    // 下一个周二，如果当前是周二则返回当前日期
    localDateTime.with(TemporalAdjusters.nextOrSame(DayOfWeek.TUESDAY));
    // 上一个周一
    localDateTime.with(TemporalAdjusters.previous(DayOfWeek.MONDAY));
    // 上一个周二，如果当前是周二则返回当前日期
    localDateTime.with(TemporalAdjusters.previousOrSame(DayOfWeek.TUESDAY));
}
```

### 日期比较

两个日期比较，是否在前或在后

```java
public void test() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime future = LocalDateTime.of(2022, 4, 2, 9, 18, 22, 233);
    System.out.println(now.isBefore(future));
    System.out.println(now.isAfter(future));
}
```

### 时间量

注意时间量有正负之分

```java
public void test() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime future = LocalDateTime.parse("2022-6-30 23:59:59", DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM));
    // 通过时间比较创建
    Duration duration1 = Duration.between(now, future);
    // 手动创建，可通过API或指定单位及数量
    Duration duration2 = Duration.ofDays(20);
    Duration duration3 = Duration.of(10, ChronoUnit.DAYS);
    // 通过字符串解析创建
    Duration duration4 = Duration.parse("PT1089H45M");
    // 常用API
    Duration duration5 = Duration.between(future, now);
    // 取绝对值
    Duration duration6 = duration5.abs();
    // 与其他时间比较
    int i = duration6.compareTo(duration5);
    // 是否相同
    boolean equals = duration6.equals(duration5);
    // 等份划分
    Duration duration7 = duration6.dividedBy(2);
    // 取纳秒，与getNano相同
    long nano = duration6.get(ChronoUnit.NANOS);
    // 取秒
    long second = duration6.get(ChronoUnit.SECONDS);
    // 判断是否为负
    boolean negative = duration6.isNegative();
    // 是否为0
    boolean isZero = duration6.isZero();
    // 减去特定时间，可指定单位及数量
    Duration duration8 = duration6.minus(duration7);
    Duration duration9 = duration6.minusDays(2);
    Duration duration10 = duration6.minus(3, ChronoUnit.DAYS);
    // 返回负数
    Duration negated = duration6.negated();
    // 增加时间，可指定单位及数量
    Duration duration11 = duration6.plus(2, ChronoUnit.DAYS);
    Duration duration12 = duration6.plusHours(3);
    Duration duration13 = duration6.plus(duration2);
    // 转换为天数、时分秒等（向下取整）
    long days = duration6.toDays();
    // 转换为纳秒
    long nanos = duration6.toNanos();
    // 修改秒数，不修改纳秒数
    Duration duration14 = duration6.withSeconds(20);
    // 修改纳秒，但不修改其他单位如时分秒
    Duration duration15 = duration6.withNanos(2333);
}
```

### 时期时间量

```java
public void test() {
    LocalDate now = LocalDate.now();
    LocalDate future = LocalDate.parse("2022-12-31", DateTimeFormatter.ISO_DATE);
    // 通过比较创建
    Period period = now.until(future);
    // 手动创建
    Period period1 = Period.ofDays(10);
    Period period2 = Period.of(2, 3, 1);
    Period period3 = Period.ofWeeks(2);
    // 字符串解析
    Period period4 = Period.parse("P2M2D");
   	// 是否相等
    boolean equals = period4.equals(period1);
    // 相加
    Period period5 = period4.plus(period2);
    // 只接受YEARS、MONTHS、DAYS，与getYears等效
    long l = period4.get(ChronoUnit.YEARS);
    // 获取月份、天数等
    int months = period4.getMonths();
    int days = period4.getDays();
    // 是否为负数
    boolean negative = period4.isNegative();
    // 是否为零
    boolean zero = period4.isZero();
    // 相减
    Period minus = period4.minus(period1);
    // 减去月数，可选年、日
    Period period6 = period4.minusMonths(2);
    // 取反
    Period negated = minus.negated();
    // 标准化
    Period period7 = period4.normalized();
    // 相加
    Period period8 = period4.plus(period5);
    // 加若干天
    Period period9 = period4.plusDays(2);
    // 从指定的日期中减去次周期
    LocalDate localDate = (LocalDate) period4.subtractFrom(LocalDate.now());
    // 修改月数等
    Period period10 = period4.withMonths(4);
}
```

### 时间加减

```java
public void test() {
    LocalDateTime now = LocalDateTime.now();
    LocalDateTime localDateTime1 = now.plusYears(2);
    LocalDateTime localDateTime2 = now.plusMonths(2);
    LocalDateTime localDateTime3 = now.plusWeeks(2);
    LocalDateTime localDateTime4 = now.plusDays(2);
    LocalDateTime localDateTime5 = now.plusHours(2);
    LocalDateTime localDateTime6 = now.plusMinutes(2);
    LocalDateTime localDateTime7 = now.plusSeconds(2);
    LocalDateTime localDateTime8 = now.plusNanos(2);
    LocalDateTime LocalDateTime9 = now.plus(20, ChronoUnit.DAYS);
    LocalDateTime localDateTime0 = now.plus(Duration.ofSeconds(200));

    LocalDateTime localDateTime11 = now.minusYears(2);
    LocalDateTime localDateTime12 = now.minusMonths(2);
    LocalDateTime localDateTime13 = now.minusWeeks(2);
    LocalDateTime localDateTime14 = now.minusDays(2);
    LocalDateTime localDateTime15 = now.minusHours(2);
    LocalDateTime localDateTime16 = now.minusMinutes(2);
    LocalDateTime localDateTime17 = now.minusSeconds(2);
    LocalDateTime localDateTime18 = now.minusNanos(2);
    LocalDateTime LocalDateTime19 = now.minus(20, ChronoUnit.DAYS);
    LocalDateTime localDateTime10 = now.minus(Duration.ofSeconds(200));
}
```

### 时区时间

```java
public void test() {
    // 默认时区
    ZonedDateTime zonedDateTime1 = ZonedDateTime.now();
    ZonedDateTime zonedDateTime2 = ZonedDateTime.now(ZoneId.systemDefault());
    // 指定时区
    ZonedDateTime newYorkTime = ZonedDateTime.now(ZoneId.of("America/New_York"));
    // 时区转换
    ZonedDateTime shanghaiTime = newYorkTime.withZoneSameInstant(ZoneId.of("Asia/Shanghai"));
    // 转换为本地时间
    LocalDateTime localDateTime = newYorkTime.toLocalDateTime();
}
```

### 时间戳

```java
public void test() {
    // 中时区的时间
    Instant now = Instant.now();
    // 转为时区时间：东八区
    OffsetDateTime offsetDateTime = now.atOffset(ZoneOffset.ofHours(8));
    // 毫秒时间戳
    long epochMilli = now.toEpochMilli();
    // 秒时间戳
    long epochSecond = now.getEpochSecond();
    // 当前纳秒
    int nano = now.getNano();
    // 实例化：通过给定的毫秒数/秒数
    Instant instant = Instant.ofEpochSecond(1675888882);
    // 通过时间戳创建时区时间
    ZonedDateTime zonedDateTime = instant.atZone(ZoneId.systemDefault());
}
```

### 时间转换

```java
public void test12() {
    LocalDateTime localDateTime = LocalDateTime.now();
    // 转换为时间戳
    Instant instant = localDateTime.atZone(ZoneId.systemDefault()).toInstant();
    LocalDate localDate = localDateTime.toLocalDate();
    LocalTime localTime = localDateTime.toLocalTime();
    // 时间截断至天：2022-06-24T00:00
    LocalDateTime localDateTime1 = localDateTime.truncatedTo(ChronoUnit.DAYS);
    // 时间截断至时：2022-06-24T20:00
    LocalDateTime localDateTime2 = localDateTime.truncatedTo(ChronoUnit.HOURS);
}
```

### 格式化

可使用系统默认的格式化，一般其中FormatStyle.MEDIUM格式为 yyyy-MM-dd HH:mm:ss ，也可以自定义格式，用于格式化或解析

```java
public void test() {
    LocalDateTime now = LocalDateTime.now();
    //2022-06-23T23:20:54.762
    DateTimeFormatter.ISO_DATE_TIME.format(now);
    //2022-06-23
    DateTimeFormatter.ISO_DATE.format(now);
    //23:20:54.762
    DateTimeFormatter.ISO_TIME.format(now);
    //20220623
    DateTimeFormatter.BASIC_ISO_DATE.format(now);
    //2022-W25-4
    DateTimeFormatter.ISO_WEEK_DATE.format(now);
    //22-6-23 下午11:20
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT).format(now);
    //2022-6-23 23:20:54
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM).format(now);
    //2022年6月23日 下午11时20分54秒
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG).format(now);
    //2022年6月23日 星期四 下午11时20分54秒 CT，需指定地区
    DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL).withZone(ZoneId.systemDefault()).format(now);
    //2022-06-23 23:20:54
    DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss").format(now);
    //2022-06-23 星期四 23:20:54
    DateTimeFormatter.ofPattern("yyyy-MM-dd EE HH:mm:ss", Locale.CHINA).format(now);
    //2022-06-23T15:20:54.789Z
    DateTimeFormatter.ISO_INSTANT.format(Instant.now());
}
```

### 转换

新API之间的转换

![image-20220405165658319](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220405165658319.png)

旧API转换为新API

| 类                                  | To 遗留类                             | From 遗留类                         |
| :---------------------------------- | ------------------------------------- | ----------------------------------- |
| Instant 与 Date                     | Date.from(instant)                    | date.toInstant()                    |
| Instant 与 Timestamp                | Timestamp.from(instant)               | timstamp.toInstant()                |
| ZonedDateTime 与 GregorianCanlendar | GregorianCalendar.from(zonedDateTime) | gregorianCalendar.toZonedDateTime() |
| LocalDate 与 Time(sql)              | Date.valueOf(localDate)               | date.toLocalDate()                  |
| LocalTime 与 Time(sql)              | Date.valueOf(localDate)               | date.toLocalTime()                  |
| LocalTime 与 Timestamp              | Timestamp.valueOf(localDateTime)      | timestamp.toLocalDateTime()         |
| ZonedId 与 TimeZond                 | Timezone.getTimeZone(id)              | timeZone.toZoneId()                 |
| DateTimeFormatter 与 DateFormat     | formatter.toFormat()                  | 无                                  |

