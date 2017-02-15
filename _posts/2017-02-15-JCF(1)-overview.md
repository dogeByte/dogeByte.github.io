---
layout: post
title:  "集合框架概述"
date:   2017-02-15 06:45:41
categories: Java集合框架
tags: Java
---

# 目录
1. [容器](#1)
2. [接口和实现类](#2)
3. [泛型](#3)
4. [迭代器](#4)

<h1 id="1">容器</h1>

在计算机科学中，容器是指实例为其他类的对象的集合的类、数据结构、或者抽象数据类型。换言之，它们以一种遵循特定访问规则的系统的方法来存储对象。容器的大小取决于其包含的对象（或元素）的数目。潜在的不同容器类型的实现可能在空间和时间复杂度上有所差别，这使得在给定应用场景中选择合适的某种实现具有灵活性。

Java Collections Framework（JCF）为 Java 开发者提供了通用的容器，始于 JDK 1.2 。 Java 容器里只能放对象，对于八种基本类型（`short` `int` `long` `float` `double` `boolean` `byte` `char`），需要将其包装后才能放到容器里。很多时候装箱和拆箱能够自动完成，这虽然会导致额外的性能和空间开销，但简化了设计和编程。

<h1 id="2">接口和实现类</h1>

为了规范容器的行为， JCF 定义了 14 种容器接口（collection interfaces），它们的关系如下图所示：

![JCF collection interfaces](https://s25.postimg.org/71cupf81b/JCF01.png)

`Map` 接口没有继承自 `Collection` 接口，因为 `Map` 表示的是关联式容器而不是集合。但 Java 提供了从 `Map` 转换到 `Collection` 的方法，可以方便地将 `Map` 切换到集合视图。

<h1 id="3">实现</h1>

上述接口的通用实现见下表：


<table border="1" width="100%">
<thead>
<tr align="center">
<td rowspan="2">接口</td><td colspan="2">实现类</td>
</tr>
<tr align="center">
<td>可变长数组</td>
<td>双向链表</td>
</tr>
</thead>
<tbody>
<tr align="center">
<td>List</td>
<td>ArrayList</td>
<td>LinkedList</td>
</tr>
<tr align="center">
<td>Deque</td>
<td>ArrayDeque</td>
<td>LinkedList</td>
</tr>
</tbody>
</table>

<br/>

<table border="1" width="100%">
<thead>
<tr align="center">
<td rowspan="2">接口</td><td colspan="3">实现类</td>
</tr>
<tr align="center">
<td>哈希表</td>
<td>平衡二叉树</td>
<td>哈希表+链表</td>
</tr>
</thead>
<tbody>
<tr align="center">
<td>Set</td>
<td>HashSet</td>
<td>TreeSet</td>
<td>LinkedHashSet</td>
</tr>
<tr align="center">
<td>Map</td>
<td>HashMap</td>
<td>TreeMap</td>
<td>LinkedHashMap</td>
</tr>
</tbody>
</table>

<h1 id="3">泛型</h1>

所有容器的内部存放的都是 `Object` 对象，泛型机制可以简化编程，由编译器完成类型的强制转换，并且可以在编译阶段而不是运行阶段发现问题。JDK 1.4 及以前版本不支持泛型，类型转换需要显式完成。

```java
List cars = new ArrayList();
cars.add("Audi");
cars.add("Benz");
cars.add("BMW");
for (int i = 0; i < cars.size(); i++) {
	String car = (String) cars.get(i);    // 显式类型转换
	System.out.println(car);
}
```

JDK 1.5 开始支持泛型的写法，省去了类型的强制转换，并且如果添加了错误类型的元素会编译失败。

```java
List<String> cars = new ArrayList<String>();
cars.add("Audi");
cars.add("Benz");
cars.add("BMW");
for (int i = 0; i < cars.size(); i++) {
	String car = cars.get(i);    // 隐式类型转换
	System.out.println(car);
}
```

JDK 1.7 可以在声明中指定泛型，创建时无需再次指定，编译器会自动推断目标类型。

```java
List<String> cars = new ArrayList<>();
```

<h1 id="4">迭代器</h1>

迭代器 `Iterator` 提供了遍历容器中元素的方法。只有容器本身清楚容器里元素的组织方式，因此迭代器只能通过容器本身得到。

```java
List<String> cars = new ArrayList<>();
cars.add("Audi");
cars.add("Benz");
cars.add("BMW");
for (Iterator<String> iterator = cars.iterator(); iterator.hasNext();) {
	String car = iterator.next();
	System.out.println(car);
}
```

JDK 1.5 引入了增强 for 循环，简化了遍历容器或数组时的写法。

```java
List<String> cars = new ArrayList<>();
cars.add("Audi");
cars.add("Benz");
cars.add("BMW");
for (String car : cars) {
	System.out.println(car);
}
```

上述代码的输出结果均为：

```
Audi
Benz
BMW
```