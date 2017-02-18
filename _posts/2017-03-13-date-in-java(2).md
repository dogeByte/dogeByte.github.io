---
layout: post
title:  "Java 中的日期和时间（下）"
date:   2017-03-13 08:07:01
categories: Java
tags: Java
---

# 目录 
1. [枚举类 Month 和 DayOfWeek](#1)
    1. [Month](#1_1)
    2. [DayOfWeek](#1_2)
2. [日期和时间](#2)
    1. [LocalDate 和 LocalTime](#2_1)
    2. [LocalDateTime](#2_2)
3. [解析和格式化](#3)
4. [调节器](#4)
    1. [TemporalAdjusters](#4_1)
    2. [自定义调节器](#4_2)

[上集](https://dogebyte.github.io/java/2017/03/12/date-in-java(1).html)简单介绍了 Java 中的 `Date` 类， `Calendar` 类以及用于格式化的 `SimpleDateFormater` 类。使用这些类的时候我们会明显地感受到其中的不便之处，比如 `Calendar` 类的月份是从 0 开始计数的；日期格式输出不够友好，都需要使用 `SimpleDateFormater` 类来格式化；一些简单的日期计算也比较麻烦等等。所以就有了 joda-time 这种第三方库来简化 Java 对于日期和时间的操作。为了改变这种情况， jdk 8 中对日期和时间对处理就吸收了 joda-time 库的特性。

<h1 id="1">枚举类 Month 和 DayOfWeek</h1>

<h3 id="1_1">Month</h3>

`Calendar` 类的月份是从 0 开始计数的，因此月份的表示和计算比较复杂。 JDK 8 中为了改变这一现状，增加了枚举类 `Month` 来表示月份，甚至可以直接使用这个枚举类来进行月份的加减运算。

```java
public static Month of(int month) {
    if (month < 1 || month > 12) {
        throw new DateTimeException("Invalid value for MonthOfYear: " + month);
    }
    return ENUMS[month - 1];
}
```

`of(int month)` 方法用于创建一个 `Month` 对象。传入的参数范围为 [1,12] ，当传入的参数超出范围就会抛出异常。

```java
public int getValue() {
    return ordinal() + 1;
}
```

`getValue()` 方法返回该 `Month` 对象当前的值。一月份返回1，二月份返回2，依次类推。

```java
public Month plus(long months) {
    int amount = (int) (months % 12);
    return ENUMS[(ordinal() + (amount + 12)) % 12];
}
```

`plus(long months)` 方法用来计算月份的加法，传入的参数表示在该 `Month` 对象的基础上增加的月份，例如 12 月加上 2 个月，返回 2 月。

```java
public Month minus(long months) {
    return plus(-(months % 12));
}
```

`minus(long months)` 方法和 `plus(long months)` 方法是类似的，例如是 1 月减去 2 个月，返回 11 月。

```java
public int length(boolean leapYear) {
    switch (this) {
        case FEBRUARY:
            return (leapYear ? 29 : 28);
        case APRIL:
        case JUNE:
        case SEPTEMBER:
        case NOVEMBER:
            return 30;
        default:
            return 31;
    }
}
```

`length(boolean leapYear)` 方法和 `maxLength()` / `minLength()` 两个方法都是用来获取 `Month` 对象表示的该月的日期数。其中参数 `leapYear` 表示是否为闰年。这三个方法返回的结果在很多情况下都是一样的，返回的都是当月的日数，30或者31。只有二月份除外，当 `Month` 对象表示二月份时， `maxLength()` 和 `length(true)` 返回29， `minLength()` 和 `length(false)` 返回28。

```java
System.out.println(Month.DECEMBER);         // DECEMBER
System.out.println(Month.of(2));            // FEBRUARY
Month month = Month.FEBRUARY;        
System.out.println(month.getValue());       // 2
System.out.println(month.minus(3));         // NOVEMBER
System.out.println(month.plus(2));          // APRIL
System.out.println(month.length(false));    // 28
System.out.println(month.length(true));     // 29
```

有时候我们希望返回月份是中文，这时候需要调用 `getDisplayName(TextStyle style, Locale locale)` 方法，第一个参数是文本类型，表示希望显示完整的名称还是缩写，第二个参数表示地区。

```java
Month month = Month.APRIL;
System.out.println(month.getDisplayName(TextStyle.FULL, Locale.getDefault()));      // 二月
System.out.println(month.getDisplayName(TextStyle.SHORT, Locale.getDefault()));     // 二月
System.out.println(month.getDisplayName(TextStyle.NARROW, Locale.getDefault()));    // 2
System.out.println(month.getDisplayName(TextStyle.FULL, Locale.ENGLISH));           // February
System.out.println(month.getDisplayName(TextStyle.SHORT, Locale.ENGLISH));          // Feb
System.out.println(month.getDisplayName(TextStyle.NARROW, Locale.ENGLISH));         // F
}
```

<h3 id="1_2">DayOfWeek</h3>

枚举类 `DayOfWeek` 用来表示一周中的七天。常用的方法和枚举类 `Month` 几乎一致，包括静态方法 `of(int dayOfWeek)` 用于创建 `DayOfWeek` 对象； `getValue()` 方法用来获取该对象的值； `plus(long days)` 和 `minus(long days)` 方法用来进行日期计算。同样也可以使用 `getDisplayName(TextStyle style, Locale locale)` 来格式化输出。

```java
System.out.println(DayOfWeek.FRIDAY);       // FRIDAY
System.out.println(DayOfWeek.of(7));        // SUNDAY
DayOfWeek dayOfWeek = DayOfWeek.TUESDAY;
System.out.println(dayOfWeek.getValue());   // 2
System.out.println(dayOfWeek.plus(3));      // FRIDAY
System.out.println(dayOfWeek.minus(2));     // SUNDAY
System.out.println(dayOfWeek.getDisplayName(TextStyle.FULL, Locale.getDefault()));    // 星期二
System.out.println(dayOfWeek.getDisplayName(TextStyle.SHORT, Locale.getDefault()));   // 星期二
System.out.println(dayOfWeek.getDisplayName(TextStyle.NARROW, Locale.getDefault()));  // 二
System.out.println(dayOfWeek.getDisplayName(TextStyle.FULL, Locale.ENGLISH));         // Tuesday
System.out.println(dayOfWeek.getDisplayName(TextStyle.SHORT, Locale.ENGLISH));        // Tue
System.out.println(dayOfWeek.getDisplayName(TextStyle.NARROW, Locale.ENGLISH));       // T
}
```

由于 `Month` 和 `DayOfWeek` 只是枚举类，它们并不持有当前时间信息，所以不能使用这两个枚举类来解决“今天是星期几”或者“明天是几号”等问题了。

<h1 id="2">日期时间</h1>

> - LocalDate 类可以获取当前日期（不包含时间），并可以进行相应处理。
> 
> - LocalTime 类可以获取当前时间（不包含日期），并可以进行相应处理。
> 
> - LocalDateTime 类可以同时处理日期和时间。

<h3 id="2_1">LocalDate 和 LocalTime</h3>

根据不同的需求， `LocalDate` 提供了不同的创建方式，主要包括 `now()` 和 `of()` 这两个静态方法。

```java
LocalDatedate1 = LocalDate.now();
LocalDatedate2 = LocalDate.of(2017, 1, 1);
LocalDatedate3 = LocalDate.ofEpochDay(364);
System.out.println(date1);    // 2017-03-13
System.out.println(date2);    // 2017-01-01
System.out.println(date3);    // 1970-12-31
LocalTimetime1 = LocalTime.now();
LocalTimetime2 = LocalTime.now().withNano(0);
LocalTimetime3 = LocalTime.of(23, 59);
LocalTimetime4 = LocalTime.ofSecondOfDay(12 * 60 * 60);
System.out.println(time1);    // 21:52:22.719
System.out.println(time2);    // 21:52:22
System.out.println(time3);    // 23：59
System.out.println(time4);    // 12:00
```

`LocalDate` 提供了大量的方法来进行日期信息的获取和计算，主要可以分为四类：获取日期信息，修改日期信息，加减法运算和日期对象间的比较。

<table border="1">
<thead>
<tr>
<th>方法名</th>
<th>返回值</th>
<th>备注</th>
</tr>
</thead>
<tbody>
<tr>
<td>getYear()</td>
<td>int</td>
<td>获取当前日期的年份</td>
</tr>
<tr>
<td>getMonth()</td>
<td>Month</td>
<td>获取当前日期的月份对象</td>
</tr>
<tr>
<td>getMonthValue()</td>
<td>int</td>
<td>获取当前日期是第几月</td>
</tr>
<tr>
<td>getDayOfWeek()</td>
<td>DayOfWeek</td>
<td>表示该对象表示的日期是星期几</td>
</tr>
<tr>
<td>getDayOfMonth()</td>
<td>int</td>
<td>表示该对象表示的日期是这个月第几天</td>
</tr>
<tr>
<td>getDayOfYear()</td>
<td>int</td>
<td>表示该对象表示的日期是今年第几天</td>
</tr>
<tr>
<td>withYear(int year)</td>
<td>LocalDate</td>
<td>修改当前对象的年份</td>
</tr>
<tr>
<td>withMonth(int month)</td>
<td>LocalDate</td>
<td>修改当前对象的月份</td>
</tr>
<tr>
<td>withDayOfMonth(int dayOfMonth)</td>
<td>LocalDate</td>
<td>修改当前对象在当月的日期</td>
</tr>
<tr>
<td>isLeapYear()</td>
<td>boolean</td>
<td>是否是闰年</td>
</tr>
<tr>
<td>lengthOfMonth()</td>
<td>int</td>
<td>这个月有多少天</td>
</tr>
<tr>
<td>lengthOfYear()</td>
<td>int</td>
<td>该对象表示的年份有多少天（365或者366）</td>
</tr>
<tr>
<td>plusYears(long yearsToAdd)</td>
<td>LocalDate</td>
<td>当前对象增加指定的年份数</td>
</tr>
<tr>
<td>plusMonths(long monthsToAdd)</td>
<td>LocalDate</td>
<td>当前对象增加指定的月份数</td>
</tr>
<tr>
<td>plusWeeks(long weeksToAdd)</td>
<td>LocalDate</td>
<td>当前对象增加指定的周数</td>
</tr>
<tr>
<td>plusDays(long daysToAdd)</td>
<td>LocalDate</td>
<td>当前对象增加指定的天数</td>
</tr>
<tr>
<td>minusYears(long yearsToSubtract)</td>
<td>LocalDate</td>
<td>当前对象减去指定的年数</td>
</tr>
<tr>
<td>minusMonths(long monthsToSubtract)</td>
<td>LocalDate</td>
<td>当前对象减去注定的月数</td>
</tr>
<tr>
<td>minusWeeks(long weeksToSubtract)</td>
<td>LocalDate</td>
<td>当前对象减去指定的周数</td>
</tr>
<tr>
<td>minusDays(long daysToSubtract)</td>
<td>LocalDate</td>
<td>当前对象减去指定的天数</td>
</tr>
<tr>
<td>compareTo(ChronoLocalDate other)</td>
<td>int</td>
<td>比较当前对象和 other 对象在时间上的大小，返回值如果为正，则当前对象时间较晚，</td>
</tr>
<tr>
<td>isBefore(ChronoLocalDate other)</td>
<td>boolean</td>
<td>比较当前对象日期是否在 other 对象日期之前</td>
</tr>
<tr>
<td>isAfter(ChronoLocalDate other)</td>
<td>boolean</td>
<td>比较当前对象日期是否在 other 对象日期之后</td>
</tr>
<tr>
<td>isEqual(ChronoLocalDate other)</td>
<td>boolean</td>
<td>比较两个日期对象是否相等</td>
</tr>
</tbody>
</table>

列表中的参数 `ChronoLocalDate` 是一个接口， `LocalDate` 是它的实现类，所以可以直接传入一个 `LocalDate`对象。

```java
LocalDate localDate = LocalDate.now();
System.out.println(localDate.getYear());                 // 2017
System.out.println(localDate.getDayOfWeek());            // MONDAY
System.out.println(localDate.getDayOfMonth());           // 13
System.out.println(localDate.withMonth(3));              // 2017-03-13
System.out.println(localDate.minusWeeks(2));             // 2017-02-27
System.out.println(localDate.plusDays(10));              // 2017-03-23
LocalDate firstDayOfYear = LocalDate.of(2017, 1, 1);
System.out.println(localDate.compareTo(firstDayOfYear)); // 2
System.out.println(localDate.isAfter(firstDayOfYear));   // true
System.out.println(localDate.isEqual(firstDayOfYear));   // false
```

`LocalTime` 类中的常用方法和 `LocalDate` 类似，同样可以分为：获取时间信息，修改时间信息，加减法运算和时间对象间的比较，不过 `LocalTime` 类中没有 `isEqual()` 方法。

<h3 id="2_2">LocalDateTime</h3>

将日期和时间分开处理可能会有些不方便，这时可以使用 `LocalDateTime` 类。

`LocalDateTime` 类的创建方式也同 `LocalDate` 类和 `LocalTime` 类相似：

```java
LocalDateTime dateTime1 = LocalDateTime.now();
LocalDateTime dateTime2 = LocalDateTime.of(2008, 8, 8, 8, 8);
LocalDateTime dateTime3 = LocalDateTime.of(1993, 10, 26, 11, 10, 30);
System.out.println(dateTime1);    // 2017-03-13T22:20:53.431
System.out.println(dateTime2);    // 2008-08-08T08:08
System.out.println(dateTime3);    // 1993-10-26T11:10:30
```

通常需要在 `of()` 方法中传入6个 `int` 参数，分别表示年月日时分秒。关于月份，既可以传入 `Month` 对象，也可以传入 `int` 值。另外也可以传入 5 个参数，将秒省略了，或者也可以增加一个纳秒参数，变为传入 7 个参数。

`LocalDateTime` 类的其它常用方法同 `LocalDate` 类和 `LocalTime` 类也是类似的，不再赘述。

`LocalDateTime` 类可以方便地转换为 `LocalDate` 类或 `LocalTime` 类。

```java
LocalDateTime dateTime = LocalDateTime.now();
LocalDate date = dateTime.toLocalDate();
LocalTime time = dateTime.toLocalTime();
System.out.println(dateTime);    // 2017-03-13T22:28:33.816
System.out.println(date);        // 2017-03-13
System.out.println(time);        // 22:18:33.816
```

<h1 id="3">解析和格式化</h1>

在 JDK 7 的时代，想要格式化一个日期，只能用 `Date` 类，并且 `SimpleDateFormat` 类还存在线程安全隐患，而在 JDK 8 中，这些问题都不复存在了。

```java
LocalDate date = LocalDate.parse("2017-03-13");
LocalTime time = LocalTime.parse("22:34:45");
LocalDateTime dateTime = LocalDateTime.parse("2017-03-13T22:34:45");
System.out.println(date);         // 2017-03-13
System.out.println(time);         // 22:34:45
System.out.println(dateTime);     // 2017-03-13T22:34:45
```

也可以按照自定义的格式进行解析：

```java
String text = "2017/03/13 22:37:39";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
System.out.println(LocalDateTime.parse(text, formatter));    // 2017-03-13T22:37:39
```

日期和时间的格式化同样需要用到 `DateTimeFormatter` 类：

```java
LocalDateTime dateTime = LocalDateTime.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss");
System.out.println(dateTime.format(formatter));    // 2017/03/13 22:40:52
```

<h1 id="4">调节器</h1>

<h3 id="4_1">TemporalAdjusters</h3>

新版日期和时间的处理方法，与以前使用的 `Date` 和 `Calendar` 类最明显的区别就是调节器了。

```java
LocalDate date = LocalDate.now();
System.out.println("今天是 " + date + " " + date.getDayOfWeek().getDisplayName(TextStyle.FULL, Locale.getDefault()));    // 今天是 2017-03-14 星期二
System.out.println("下周二是 " + date.with(TemporalAdjusters.next(DayOfWeek.TUESDAY)));    // 下周二是 2017-03-21
System.out.println("本月的第一个周二是 " + date.with(TemporalAdjusters.firstInMonth(DayOfWeek.TUESDAY)));    // 本月的第一个周二是 2017-03-07
```

得到 `LocalDate` 对象后，调用 `with()` 方法，传入一个 `TemporalAdjusters` 对象即可。 `TemporalAdjusters` 类有许多静态方法来创建该对象：

> - firstDayOfMonth()
> 
> - lastDayOfMonth()
> 
> - firstDayOfNextMonth()
> 
> - firstDayOfYear()
> 
> - lastDayOfYear()
> 
> - firstDayOfNextYear()
> 
> - firstInMonth(DayOfWeek dayOfWeek)
> 
> - lastInMonth(DayOfWeek dayOfWeek)
> 
> - dayOfWeekInMonth(int ordinal, DayOfWeek dayOfWeek)
> 
> - next(DayOfWeek dayOfWeek)
> 
> - nextOrSame(DayOfWeek dayOfWeek)
> 
> - previous(DayOfWeek dayOfWeek)
> 
> - previousOrSame(DayOfWeek dayOfWeek)

如果这些方法不能满足需求，就需要自定义调节器了。

<h3 id="4_2">自定义调节器</h3>

自定义一个调节器很简单，创建一个类，实现 `TemporalAdjuster` 接口，重写 `adjustInto(Temporal temporal)` 方法。

假设一个场景，一个公司每个月清账两次，分别是本月 15 号和最后一天。如果恰逢周末，则提前到周五清帐。如何自定义一个调节器，计算下一次清帐时间呢？

```java
LocalDate date = LocalDate.of(2016, Month.DECEMBER, 20).with(new TemporalAdjuster() {
    @Override
    public Temporal adjustInto(Temporal temporal) {
        LocalDate date = LocalDate.from(temporal);
        int day = date.getDayOfMonth() < 15 ? 15 : date.lengthOfMonth();
        date = date.withDayOfMonth(day);
        if (DayOfWeek.SATURDAY.equals(date.getDayOfWeek()) || DayOfWeek.SUNDAY.equals(date.getDayOfWeek())) {
            date = date.with(TemporalAdjusters.previous(DayOfWeek.FRIDAY));
        }
        return temporal.with(date);
    }
});
System.out.println(date);    // 2016-12-30
```

2016年12月31日是周六，所以清账日期提前至周五。