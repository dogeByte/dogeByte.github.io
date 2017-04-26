---
layout: post
title:  "探究 String.intern()"
date:   2017-02-15 06:45:41
categories: Java
tags: Java
---

# 目录
1. [intern 简介](#1)
2. [jdk6 和 jdk7 下 intern 的区别](#2)
    1. [jdk6 的 intern](#2_1)
    2. [jdk7 的 intern](#2_2)

<h1 id="1">intern 简介</h1>

众所周知，Java 中有 8 个基本数据类型和一个字符串类型 `String`。为了优化性能并节省内存，JVM 中提出了“常量池”这一概念，常量池就类似一个 Java 系统级别提供的缓存。

8种基本类型的常量池都是系统协调的，而 `String` 类型的常量池比较特殊，它的主要使用方法有两种：

1. 直接使用双引号声明出来的 `String` 对象，即字面值，会直接存储在常量池中；
2. 不是使用双引号声明出来的 `String` 对象，可以使用 `String` 提供的 `intern` 方法。

> `intern` 方法返回字符串对象的规范化表示形式。
> 
> 一个初始为空的字符串池，它由类 `String` 私有地维护。
> 
> 当调用 `intern` 方法时，如果池已经包含一个等于此 `String` 对象的字符串（用 `equals(Object)` 方法确定），则返回池中的字符串。否则，将此 `String` 对象添加到池中，并返回此 `String` 对象的引用。
> 
> 它遵循以下规则：对于任意两个字符串 s 和 t，当且仅当 `s.equals(t)` 为 `true` 时，`s.intern() == t.intern()` 才为 `true`。
> 
> 所有字面值字符串和字符串赋值常量表达式都使用 `intern` 方法进行操作。字符串字面值在 [Java Language Specification](https://docs.oracle.com/javase/specs/) 的 §3.10.5 定义。> 
> 
> 返回：一个字符串，内容与此字符串相同，但一定取自具有唯一字符串的池。

简言之，`intern` 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中，并返回其在池中的引用。

<h1 id="2">jdk6 和 jdk7 下 intern 的区别</h1>

分别试试这段代码的在 jdk6 和 jdk7 下的输出：

```java
String s1 = new StringBuilder().append("jing").append("bo").toString();
String s2 = new StringBuilder().append("ja").append("va").toString();
System.out.println(s1.intern() == s1);
System.out.println(s2.intern() == s2);
```

<h3 id="2_1">jdk6 的 intern</h3>

jdk6 的输出为 `false` / `false`。

执行 `intern` 方法时，会优先在常量池中查找是否已经存在相同的字符串。如果已经存在，则直接返回该字符串的引用；如果不存在，JVM 则会在常量池中生成该字符串，再将返回该字符串的引用。

在 jdk6 以及更早的版本中，字符串的常量池放在 Perm 区，而 Perm 区和堆内存是物理隔离的。`s1` 指向堆内存，`s1.intern()` 则指向常量池，对于 `s2` 和 `s2.intern()` 也是同理，理应返回两个 `false`。

<h3 id="2_2">jdk7 的 intern</h3>

jdk7 下情况发生了变化，输出为 `true` / `false`。

首先需要明确一点，Perm 区是一个类静态的区域，主要存储一些加载类的信息，常量池，方法片段等内容，默认大小只有 4m，一旦常量池中大量使用 `intern`，有可能产生`java.lang.OutOfMemoryError: PermGen space`。所以，在 jdk7 中，JVM 将常量池从 Perm 区移到了元空间，正因为如此，jdk7 及其以后版本的 `intern` 方法在实现上发生了较大的改变。

JDK 1.7后，intern方法还是会先去查询常量池中是否有已经存在，如果存在，则返回常量池中的引用，这一点与之前没有区别，区别在于，如果在常量池找不到对应的字符串，则不会再将字符串拷贝到常量池，而只是在常量池中生成一个对原字符串的引用。

根据上述分析，`s1` 和 `s1.intern()` 都指向堆内存，因此第一次输出 `true`。而字符串 `"java"` 在虚拟机初始化时就在常量池中存在，因此第二次依然输出 `false`。