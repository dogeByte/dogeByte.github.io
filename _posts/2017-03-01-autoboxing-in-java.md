---
layout: post
title:  "自动装箱的分析"
date:   2017-03-01 07:40:15
categories: Java
tags: Java
---

# 目录
1. [缓存](#1)
2. [比较](#2)
3. [重载与自动装箱](#3)
4. [自动装箱的弊端](#4)

今天逛论坛的时候看到一个比较有趣的问题，下面的代码的执行结果是什么：

```java
public static void main(String[] args) {
    Integer a = new Integer(3);
    Integer b = 3; 
    int c = 3;
    System.out.println(a == b);
    System.out.println(a == c);
    Integer f1 = 100, f2 = 100, f3 = 150, f4 = 150;
    System.out.println(f1 == f2);
    System.out.println(f3 == f4);
}
```

很明显这种问题涉及到自动装箱和自动拆箱，那么先看一下自动装箱和拆箱是什么时候发生的，是如何发生的。这里为了方便，结合 Integer 的源码来分析。

自动装箱，即将 char、int、float、double、long 等基本数据类型转为 Char、Integer、Float、Double、Long 等引用类型，一般来说，自动装箱/拆箱 在两种情况下会发生：赋值 和 方法调用。比如下面的代码：

```java
// 赋值时，会将基本数据类型转为引用类型。
Integer i = 3;
// 方法调用时，如果传入 isSmall(3, 5)，会自动把 3 和 5 转为 Integer。
public boolean isSmall(Integer a, Integer b) {
    return a < b;
}
```

那么，如何装箱和拆箱呢？在装箱的时候，会调用 valueOf() 方法，比如 Integer.valueOf()、Double.valueOf()、Boolean.valueOf() 方法等。而在拆箱的时候，会调用装箱类的 intValue() 方法。

<h1 id="1">缓存</h1>

这里有一个地方需要注意的是，在整型和长整型（int、long)装箱的时候，会将对象缓存起来，下次再用相同的值进行装箱的时候，会直接把这个缓存的对象返回。以 Integer 为例，代码如下：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];
    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;
        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);
        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }
    private IntegerCache() {}
}
```

上面的代码是说，如果 int 的值在某个范围内的话，会创建对于的对象并缓存起来，下次再用相同的 int 值来装箱时，就直接从缓存中获取这个对象，而不用重新创建。而这个范围，最低是 -128，而最高的值，默认是 127，但是可以自定义！这个最高的值是 JVM 虚拟机中的配置文件里面定义的，只要这个值大于 127 且小于 Integer.MAX_VALUE - (-low) -1，则可以使用定义的值作为缓存区间的上限。

还有一点，缓存其实使用的一个数组来存储，并没有缓存在常量池中。相比起来，Long 里面对于缓存的处理就简单粗暴了：

```java
private static class LongCache {
    private LongCache(){}
    static final Long cache[] = new Long[-(-128) + 127 + 1];
    static {
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Long(i - 128);
    }
}
```
<h1 id="2">比较</h1>

那么，基本数据类型和对应装箱类型是怎么比较的呢？这个其实看一下编译后的 .class 文件就知道了，比如

```java
public static void main(String[] args) {
    int a = 300;
    Integer b = 300;
    System.out.println(a == b);
}
```

编译为 .class 后就是：

```java
public static void main(String[] args) {
    short a = 300;
    Integer b = Integer.valueOf(300);
    System.out.println(a == b.intValue());
}
```

可以看见，基本数据类型和装箱类的比较，其实就是拆箱比值。

那么开头提到的问题就解决了：

第一个比较，因为 new Integer() 创建了一个新的对象，所以和 b 不是同一个对象，因此为 false。
第二个比较，因为基本数据类型和装箱类型的比较只比较值，因此两个相等，答案为 ture。
第三个比较，在装箱的时候，100 对应的装箱类是会被缓存的，因此 f2 和 f1 是同一个对象。答案是 true。
第四个比较，由于 150 大于默认的 127，因此装箱类并不会被缓存，f3 和 f4 不是同一个对象，答案是 false。

<h1 id="3">重载与自动装箱</h1>

自动装箱和自动拆箱会产生一个问题，在调用方法的时候可以自动装箱，那么重载的情况呢？比如下面的代码：

```java
public static void main(String[] args) {
    test(1);
    test(new Integer(1));
}
public static void test(int i) {
    System.out.println("This is primary :" + i);
}
public static void test(Integer box) {
    System.out.println("This is box :" + box);
}
```

实际运行一下就会知道，在有重载情况发生的时候，是不会发生自动装箱的。

<h1 id="4">自动装箱的弊端</h1>

有如下的代码：

```java
public static void main(String[] args) {
  	Long sum = 0L;
  	for (long i = 0; i < Integer.MAX_VALUE; i++) {
      sum += i;
  	}
  	System.out.println(sum);
}
```

运行的结果是正确的，但是会消耗大量内存，因为 sum 被声明称 Long 而不是 long，这意味着程序创建了大约 231231 个多余的 Long 对象。因此，要优先使用基本类型而不是装箱类型，要当心无意识的自动装箱。