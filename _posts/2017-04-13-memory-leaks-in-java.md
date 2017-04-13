---
layout: post
title:  "Java 中的内存泄漏"
date:   2017-04-13 14:56:52
categories: Java
tags: Java
---

理论上， Java 中因为存在垃圾回收机制（GC）不会存在内存泄露问题。而在实际中，可能会存在无用但可达的对象，这些对象不能被 GC 回收，因此也会导致内存泄露的发生。

```java
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(Object e) {
		ensureCapacity();
		elements[size++] = e;
	}

	public Object pop() {
		if (size == 0) {
			throw new EmptyStackException();
		}
		return elements[--size];
	}

	private void ensureCapacity() {
		if (elements.length == size) {
			elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}
}
```

这段代码实现了一个栈（stack）结构，似乎并没有任何问题，但是它的确存在着“内存泄露”的隐患。问题在于，栈中弹出的对象并不会被当成垃圾回收，即使使用栈的程序不再引用这些对象，原因是栈的内部维持着对这些对象的过期引用（obsolete reference）。

在上例中，凡是在 `elements` 数组的“活动部分”之外的任何引用都是过期的，所谓活动部分是指 `elements` 中下标小于 `size` 的那些元素。

在支持垃圾回收的语言中，内存泄露是很隐蔽的，这种内存泄露其实就是无意识的对象保持。如果一个对象引用被无意识的保留起来了，那么垃圾回收器不会处理这个对象，也不会处理该对象引用的其他对象，即使这样的对象只有少数几个，也可能会导致很多的对象被排除在垃圾回收之外，从而对性能造成重大影响，甚至造成 `OutOfMemoryError` 。

这类问题有一个很简单的修复方法，就是一旦对象引用过期，就立即手动将其清空。

```java
public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null;
	return result;
}
```

清除过期引用的另一个好处是，如果它们以后又被错误的解除引用，程序立即会抛出 `NullPointerException` 。

既然存在上述问题，那么是不是要手动清除每一个不再使用的对象引用呢？其实没有这个必要，因为它会使代码变得混乱，“清空对象引用应该是一种例外，而不是一种规范行为”。消除过期引用的最好方法是让包含该引用的变量结束其生命周期。

在上述例子中，由于栈内部维护着存储池（数组），所以垃圾回收器不会将过期的引用当做垃圾，因为它认为数组中的所有对象引用都同等有效，那么在这种情况下，只有程序员知道哪些是有效的，哪些是过期的，就需要手工清空这些过期的数组元素。一般而言，只要是由类自己管理内存，就需要警惕内存泄漏的问题。一旦元素被释放掉，则该元素中包含的任何对象引用都应该被清空。

内存泄漏的另一个常见来源是缓存，一旦把对象引用放入缓存中，它就很容易被遗忘。例如 Hibernate 的 `Session` （一级缓存）中的对象属于持久态，垃圾回收器是不会回收这些对象的，然而这些对象中可能存在无用的垃圾对象，如果不及时关闭（`close`）或清空（`flush`）一级缓存就可能导致内存泄露。

如果一个缓存要实现只要在缓存之外存在对某个项的键的引用，该项就有意义的话，就可以使用 `WeakHashMap` 代表缓存。因为当缓存中的项过期的时候，它们就会自动被删除掉。

更常见的情况是“缓存项的生命周期是否有意义”。随着时间的推移，缓存中的某些项会变得越来越没有价值，在这种情况下，缓存应该时不时地清除掉无用的项。这种工作可以交给一个后台线程（`Timer` 或 `ScheduledThreadPoolExecutor`）来完成，也可以在为缓存添加新条目的时候顺便进行。利用 `LinkedHashMap` 类的 `removeEldestEntry` 方法可以很容易地实现后一种方案。

内存泄漏的第三种常见来源是监听器和其他回调函数。如果客户在你实现的 API 中注册回调，但却没有显式地取消注册，这就容易产生积聚。确保回调立即被当作垃圾的最佳方法是只保存它们的弱引用（weak reference），例如只它们保存为 `WeakHashMap` 中的键。