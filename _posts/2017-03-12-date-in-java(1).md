---
layout: post
title:  "Java 中的日期和时间 ( 上 ) "
date:   2017-03-12 17:19:32
categories: Java
tags: Java
---

# 目录 
1. [Date](#1)
	1. [构造方法](#1_1)
	2. [其它常用方法](#1_2)
2. [Calendar](#2)
	1. [构造方法](#2_1)
	2. [get() 方法](#2_2)
	3. [set() 方法](#2_3)
	4. [add() 和 roll() 方法](#2_4)
	5. [其它常用方法](#2_5)
	6. [Date 和 Calendar 的关系](#2_6)
3. [SimpleDateFormat](#3)

自从 JDK 1.0 开始， Java 就提供了 `Date` 来处理时间和日期，作为老古董自然有很多东西是过时的。然后出现了 `Calendar` 来解决了很多问题，但是 `Calendar` 使用比较复杂，并且有些反人类的地方。直到 Java 8 中 `LocalDateTime` 中的出现，它吸收了 Joda-Time 库的经验，使得 Java 处理时间和日期变得比较“人性化”了。本篇介绍 Java 中的 `Date` 、 `Calendar` ，以及 `SimpleDateFormat` 的使用。

<h1 id="1">Date</h1>

<h3 id="1_1">构造方法</h3>

Date 类的构造方法如下：

```java
public Date() {
	this(System.currentTimeMillis());
}

public Date(long date) {
	fastTime = date;
}

@Deprecated
public Date(int year, int month, int date) {
	this(year, month, date, 0, 0, 0);
}

@Deprecated
public Date(int year, int month, int date, int hrs, int min) {
	this(year, month, date, hrs, min, 0);
}

@Deprecated
public Date(int year, int month, int date, int hrs, int min, int sec) {
	int y = year + 1900;
	// month is 0-based. So we have to normalize month to support Long.MAX_VALUE.
	if (month >= 12) {
		y += month / 12;
		month %= 12;
	} else if (month < 0) {
		y += CalendarUtils.floorDivide(month, 12);
		month = CalendarUtils.mod(month, 12);
	}
	BaseCalendar cal = getCalendarSystem(y);
	cdate = (BaseCalendar.Date) cal.newCalendarDate(TimeZone.getDefaultRef());
	cdate.setNormalizedDate(y, month + 1, date).setTimeOfDay(hrs, min, sec, 0);
	getTimeImpl();
	cdate = null;
    }
```

可以看到6个构造方法中，有4个已经被添加了 `@Deprecated` 的注解了，尚未过时的只有两个： `Date()` 和 `Date(long date)` 。其中，第一个构造方法调用的是 `System.currentTimeMillis()` 方法，这个方法返回的是一个 `long` 整数，表示从 GMT 1970年1月1日 00:00:00 到现在所经历的毫秒数。这个毫秒数很重要，无论是在 `Date` 类中还是 `Calendar` 类中，这个毫秒数都是计算时间日期的基准。

怎么获得这个 `long` 的毫秒数呢？需要调用 `getTime()` 方法，注意并不是调用 `toString()` 方法，它们两者的差异可以从下面的代码中体现出来：

```java
public static void main(String[] args) {
    Date date = new Date();
    System.out.println(date.getTime());     // 1489320281826
    System.out.println(date.toString());    // Sun Mar 12 20:04:41 CST 2017
}
```

而第二个构造方法的参数 `date` ，表示创建的 `Date` 对象与 GMT 1970年1月1日 00:00:00 的时间差。

```java
Date date = new Date(60 * 60 * 1000);
System.out.println(date.getTime());     // 36000000
System.out.println(date.toString());    // Thu Jan 01 09:00:00 CST 1970
```

为什么不是 1 点呢？原因在于 CST ( China Standard Time ) 是指的北京时间，而 GMT ( Greenwich Mean Time )是指的是格林尼治标准时间。由于北京处于东八区，比 GMT 早8小时，所以打印的时间指的是北京时间 1970 年 1 月 1 日上午 9 点。

<h3 id="1_2">其它常用方法</h3>

谈完了构造方法，下面说说 `Date` 这个古老的类中可以使用的方法。

> public long getTime() ：返回从 GMT 1970年1月1日 00:00:00 到这个类创建的毫秒数。
> 
> public void setTime(long time) ：设置该 Date 对象的时间。参数 time 表示从 GMT 1970年1月1日 00:00:00 后的毫秒数。
> 
> public boolean after(Date when) ：测试该日期是否是在指定的日期之后。
> 
> public boolean before(Date when) ：测试该日期是否是在指定的日期之前。

`Date` 类中的构造方法和常用的未过时的方法基本就是这些了。看了之后是不是觉得其实 `Date` 类能用的东西很少？很多日期和时间的操作都很难实现，这时候就要使用到 `Calendar` 类，或者使用 JDK 8 中的日期时间包。

<h1 id="2">庞大的 Calendar</h1>

<h3 id="2_1">构造方法</h3>

与 `Date` 类不同， `Calendar` 类是一个抽象类，其直接子类是 `GregorianCalendar` 。既然 `Calendar` 类是抽象类，需要使用 `Calendar calendar = Calendar.getInstance();` 来创建一个 `Calendar` 对象。和 `Date` 类一样，创建好对象之后，便包含了当前日期和时间的信息。

实际上， `getInstance()` 这个静态方法还有三个重载方法，它们的源码如下：

```java
public static Calendar getInstance() {
	return createCalendar(TimeZone.getDefault(), Locale.getDefault(Locale.Category.FORMAT));
}

public static Calendar getInstance(TimeZone zone) {
	return createCalendar(zone, Locale.getDefault(Locale.Category.FORMAT));
}

public static Calendar getInstance(Locale aLocale) {
	return createCalendar(TimeZone.getDefault(), aLocale);
}

public static Calendar getInstance(TimeZone zone, Locale aLocale) {
	return createCalendar(zone, aLocale);
}
```

显然，这 4 个重载方法的区别在于时区和地区的不同。而它们都是通过创建子类 `GregorianCalendar` 的对象来返回一个 `Calendar` 对象，这其实是多态的一种应用，也是设计模式中工厂模式的应用。

<h3 id="2_2">get() 方法</h3>

`Date` 对象可以通过 `getTime()` 方法和 `toString()` 方法，能很容易的取到对应的时间信息。而在 `Calendar` 对象中，获取到时间和日期并不这么直接。在 `Calendar` 对象中， `getTime()` 方法返回的是一个 `Date` 对象，至于 `toString()` 方法的输出结果……

```
java.util.GregorianCalendar[time=1489323573250,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null],firstDayOfWeek=1,minimalDaysI
nFirstWeek=1,ERA=1,YEAR=2017,MONTH=2,WEEK_OF_YEAR=11,WEEK_OF_MONTH=3,DAY_OF_MONTH=12,DAY_OF_YEAR=71,DAY_OF_WEEK=1,DAY_OF_WEEK_IN_MONTH=2,AM_PM=1,HOUR=8,HOUR_OF_DAY=20,MINUTE=59,SECOND=33,MILLISECOND=250,ZONE_OFFSET=28800000,DST_OFFSET=0]
```

这从侧面反映了 `Calendar` 比 `Date` 强大，能提供更多的信息。其实在 `Calendar` 类中，有一点和 `Date` 类是一样的：它们都会保存一个毫秒值。 `Calendar` 类对于时间日期的各种计算也都是基于这个值来的。 `toString()` 方法返回的第一个值就是它，足以说明这个值的重要性。

要获取 `Calendar` 对象所提供的信息，需要调用 `get()` 方法。

```java
public int get(int field) {
	complete();
	return internalGet(field);
}

protected final int internalGet(int field) {
	return fields[field];
}
```

在 `complete()` 方法中，会调用 `computeTime()` 和 `computeFields()` 方法，这两个方法都是抽象方法，由子类 `GregorianCalendar` 实现。在 `computeTime()` 方法中，计算出年月日时分秒等时间相关的信息，而在` computeFields()` 方法中获取到对应的时区地区等信息。这两个方法会将所计算出来的信息保存在 `int` 类型的数组 `fields` 。fields数组保存的信息如下所示：

```java
public static final int ERA = 0;
public static final int YEAR = 1;
public static final int MONTH = 2;
public static final int WEEK_OF_YEAR = 3;
public static final int WEEK_OF_MONTH = 4;
public static final int DATE = 5;
public static final int DAY_OF_MONTH = 5;
public static final int DAY_OF_YEAR = 6;
public static final int DAY_OF_WEEK = 7;
public static final int DAY_OF_WEEK_IN_MONTH = 8;
public static final int AM_PM = 9;
public static final int HOUR = 10;
public static final int HOUR_OF_DAY = 11;
public static final int MINUTE = 12;
public static final int SECOND = 13;
public static final int MILLISECOND = 14;
public static final int ZONE_OFFSET = 15;
public static final int DST_OFFSET = 16;
```

`get()` 方法中的参数就是上述 18 个常量之一。其中：


> ERA 表示该日期是在公元元年之前还是之后，返回 0 表示在公元元年之前，返回 1 表示在公元元年之后。
> 
> AM_PM 表示该时间是在中午 12 点之前还是之后，返回 0 表示在中午 12 点之前，返回 1 表示在中午 12 点之后。
> 
> DST_OFFSET 表示该时间距夏令时的毫秒数。

<h3 id="2_3">set() 方法</h3>

```java
public void set(int field, int value)
{
	// If the fields are partially normalized, calculate all the fields before changing any fields.
	if (areFieldsSet && !areAllFieldsSet) {
		computeFields();
	}
	internalSet(field, value);
	isTimeSet = false;
	areFieldsSet = false;
	isSet[field] = true;
	stamp[field] = nextStamp++;
	if (nextStamp == Integer.MAX_VALUE) {
		adjustStamp();
	}
}

public final void set(int year, int month, int date)
{
	set(YEAR, year);
	set(MONTH, month);
	set(DATE, date);
}

public final void set(int year, int month, int date, int hourOfDay, int minute)
{
	set(YEAR, year);
	set(MONTH, month);
	set(DATE, date);
	set(HOUR_OF_DAY, hourOfDay);
	set(MINUTE, minute);
}

public final void set(int year, int month, int date, int hourOfDay, int minute, int second)
{
	set(YEAR, year);
	set(MONTH, month);
	set(DATE, date);
	set(HOUR_OF_DAY, hourOfDay);
	set(MINUTE, minute);
	set(SECOND, second);
}
```

原理很简单，就是将我们传入的值存放到 `fields` 数组中。调用 `set()` 方法会改变当前 `Calendar` 对象的日期和时间。

```java
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.MONTH, 3);
System.out.println(calendar.get(Calendar.MONTH));           // 3
calendar.set(2017, Calendar.MARCH, 12);
System.out.println(calendar.get(Calendar.YEAR));            // 2017
System.out.println(calendar.get(Calendar.MONTH));           // 2
System.out.println(calendar.get(Calendar.DAY_OF_MONTH));    // 12
}
```

由于月份是从 0 开始计数的，所以计算月份的时候需要特别小心。这就是 `Calendar` 反人类的地方之一。

<h3 id="2_4">add() 和 roll() 方法</h3>

使用 `get()` 和 `set()` 方法可以获取当前日期和时间，也可以设置 `Calendar` 对象的日期和时间。但是一些简单的日期时间计算就需要使用到 `add()` 和 `roll()` 方法了。

`add(int field, int amount)` 方法接受两个参数，第一个参数表示希望改变的字段，这个字段是 `fields` 数组中的某一个，第二个参数是改变的值。使用 `add()` 方法需要注意几点：

> 月份是从 0 开始计数的。
> 
> 第二个参数如果为正，则数据增加；如果为负，则数据减少。
> 
> 当增加的数值超过法定额度时，会自动向更高一级的单位加 1 。

`roll()` 方法和 `add()` 类似，二者最主要的区别在于 `roll()` 方法中不会进行进位和退位运算。

<h3 id="2_5">其它常用方法</h3>

`getMaximum(int field)` 和 `getMinimum(int field)` 两个方法，返回 `Calendar` 对象的 `fields` 数组中对应数据的最大值和最小值。例如：

```java
Calendar calendar = Calendar.getInstance();
System.out.println(calendar.getMinimum(Calendar.DAY_OF_MONTH));    // 1
System.out.println(calendar.getMaximum(Calendar.DAY_OF_MONTH));    // 31
```

`getActualMaximum(int field)` 和 `getActualMinimum(int field)` 两个方法同上面两个方法相比，多了一个 Actual 单词，表示返回 `Calendar` 对象的 `fields` 数组中，在当前日期的环境条件下，对应数据的最大值和最小值。例如：

```java
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.MONTH, 1);
System.out.println(calendar.getMinimum(Calendar.DAY_OF_MONTH));    // 1
System.out.println(calendar.getMaximum(Calendar.DAY_OF_MONTH));    // 28
```

注意，这里设置的是二月。 `fields` 数组中的各个字段的最大值和最小值如下：

```java
private static int[] maximums = new int[] { 1, 292278994, 11, 53, 6, 31,         366, 7, 6, 1, 11, 23, 59, 59, 999, 14 * 3600 * 1000, 7200000 };
private static int[] minimums = new int[] { 0, 1, 0, 1, 0, 1, 1, 1, 1, 0,         0, 0, 0, 0, 0, -13 * 3600 * 1000, 0 };
```

<h3 id="2_6">Date 和 Calendar 的关系</h3>

`Calendar` 作为 `Date` 类的补充和替代，当然得和 `Date` 进行相互转化。

> 使用 getTime() 方法，从 Calendar 对象中获取 Date 对象。
> 使用 setTime() 方法，将 Date 对象的值赋给 Calendar 对象。

`Date` 的 6 个构造方法中，有 4 个已经过时了，其实它们是被 `Calendar` 类的相关方法替代了。

> Date(int year, int month, int date) 被 Calendar.set(year + 1900, month, date) 替代。
> 
> Date(int year, int month, int date, int hrs, int min) 被 Calendar.set(year + 1900, month, date, hrs, min) 替代。
> 
> Date(int year, int month, int date, int hrs, int min, int sec) 被 Calendar.set(year + 1900, month, date, hrs, min, sec) 替代。
> 
> Date(String s) 被 DateFormat.parse(String s) 替代。

<h1 id="3">SimpleDateFormat</h1>

有了 `Date` 和 `Calendar` ，我们能获取并设置日期和时间，也能对日期和时间进行简单的计算。 `SimpleDateFormat` 可以对日期和时间进行格式化。

`SimpleDateFormat` 的构造方法如下：

```java
public SimpleDateFormat() {
	this("", Locale.getDefault(Locale.Category.FORMAT));
	applyPatternImpl(LocaleProviderAdapter.getResourceBundleBased().getLocaleResources(locale).getDateTimePattern(SHORT, SHORT, calendar));
}

public SimpleDateFormat(String pattern)
{
	this(pattern, Locale.getDefault(Locale.Category.FORMAT));
}

public SimpleDateFormat(String pattern, Locale locale)
{
	if (pattern == null || locale == null) {
		throw new NullPointerException();
	}
	initializeCalendar(locale);
	this.pattern = pattern;
	this.formatData = DateFormatSymbols.getInstanceRef(locale);
	this.locale = locale;
	initialize(locale);
}

public SimpleDateFormat(String pattern, DateFormatSymbols formatSymbols)
{
	if (pattern == null || formatSymbols == null) {
		throw new NullPointerException();
	}
	this.pattern = pattern;
	this.formatData = (DateFormatSymbols) formatSymbols.clone();
	this.locale = Locale.getDefault(Locale.Category.FORMAT);
	initializeCalendar(this.locale);
	initialize(this.locale);
	useDateFormatSymbols = true;
}
```

字符串 `pattern` 代表日期和时间格式化的样式，例如：

```java
String pattern = "yyyy/MM/dd HH:mm:ss";
SimpleDateFormat simpleDateFormat = new SimpleDateFormat(pattern);
System.out.println(simpleDateFormat.format(new Date()));    // 2017/03/12 21:43:59
```

`pattern` 中的字符含义如下表，注意区分大小写：

<table border="1">
<thead>
<tr>
<th>Letter</th>
<th>Date or Time Component</th>
<th>Presentation</th>
<th>Examples</th>
</tr>
</thead>
<tbody>
<tr>
<td>G</td>
<td>Era designator</td>
<td>Text</td>
<td>AD</td>
</tr>
<tr>
<td>y</td>
<td>Year</td>
<td>Year</td>
<td>1996; 96</td>
</tr>
<tr>
<td>Y</td>
<td>Week year</td>
<td>Year</td>
<td>2009; 09</td>
</tr>
<tr>
<td>M</td>
<td>Month in year (context sensitive)</td>
<td>Month</td>
<td>July; Jul; 07</td>
</tr>
<tr>
<td>L</td>
<td>Month in year (standalone form)</td>
<td>Month</td>
<td>July; Jul; 07</td>
</tr>
<tr>
<td>w</td>
<td>Week in year</td>
<td>Number</td>
<td>27</td>
</tr>
<tr>
<td>W</td>
<td>Week in month</td>
<td>Number</td>
<td>2</td>
</tr>
<tr>
<td>D</td>
<td>Day in year</td>
<td>Number</td>
<td>189</td>
</tr>
<tr>
<td>d</td>
<td>Day in month</td>
<td>Number</td>
<td>10</td>
</tr>
<tr>
<td>F</td>
<td>Day of week in month</td>
<td>Number</td>
<td>2</td>
</tr>
<tr>
<td>E</td>
<td>Day name in week</td>
<td>Text</td>
<td>Tuesday; Tue</td>
</tr>
<tr>
<td>u</td>
<td>Day number of week (1 = Monday, …, 7 = Sunday)</td>
<td>Number</td>
<td>1</td>
</tr>
<tr>
<td>a</td>
<td>Am/pm marker</td>
<td>Text</td>
<td>PM</td>
</tr>
<tr>
<td>H</td>
<td>Hour in day (0-23)</td>
<td>Number</td>
<td>0</td>
</tr>
<tr>
<td>k</td>
<td>Hour in day (1-24)</td>
<td>Number</td>
<td>24</td>
</tr>
<tr>
<td>K</td>
<td>Hour in am/pm (0-11)</td>
<td>Number</td>
<td>0</td>
</tr>
<tr>
<td>h</td>
<td>Hour in am/pm (1-12)</td>
<td>Number</td>
<td>12</td>
</tr>
<tr>
<td>m</td>
<td>Minute in hour</td>
<td>Number</td>
<td>30</td>
</tr>
<tr>
<td>s</td>
<td>Second in minute</td>
<td>Number</td>
<td>55</td>
</tr>
<tr>
<td>S</td>
<td>Millisecond</td>
<td>Number</td>
<td>978</td>
</tr>
<tr>
<td>z</td>
<td>Time zone</td>
<td>General time zone</td>
<td>Pacific Standard Time; PST; GMT-08:00</td>
</tr>
<tr>
<td>Z</td>
<td>Time zone</td>
<td>RFC 822 time zone</td>
<td>-0800</td>
</tr>
<tr>
<td>X</td>
<td>Time zone</td>
<td>ISO 8601 time zone</td>
<td>-08; -0800; -08:00</td>
</tr>
</tbody>
</table>

`SimpleDateFormat` 类除了能将 `Date` 对象进行格式化返回字符串，也能将一个日期字符串解析一个 `Date` 对象。

```java
String s = "2017#03#12#21#49#28";
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy#MM#dd#HH#mm#ss");
Date date = null;
try {
	date = simpleDateFormat.parse(s);
} catch (ParseException e) {
	e.printStackTrace();
}
System.out.println(date);    // Sun Mar 12 21:49:28 CST 2017
```

注意，数据之间需要用 `#` 来分割。

最后说一点， `SimpleDateFormat` 类不是线程安全的，如果有多个线程需要使用到 `SimpleDateFormat` 对象，建议每个线程单独创建，如果多个线程要获取同一个 `SimpleDateFormat` 对象，记得要加锁。