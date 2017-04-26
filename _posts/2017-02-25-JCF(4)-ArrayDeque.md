---
layout: post
title:  "ArrayDeque"
date:   2017-02-25 08:31:59
categories: JCF
tags: Java
---

# 目录
1. [Deque 接口](#1)
2. [ArrayDeque 概述](#2)
3. [构造方法](#3)
4. [add 方法和 offer 方法](#4)
	1. [addFirst(E e)](#4_1)
	2. [addLast(E e)](#4_2)
	3. [offerFirst(E e) 和 offerLast(E e)](#4_3)
5. [poll 方法和 remove 方法](#5)
	1. [pollFirst() 和 pollLast()](#5_1)
	2. [removeFirst() 和 removeLast()](#5_2)
6. [peek 方法和 get 方法](#6)

<h1 id="1">Deque 接口</h1>

说到栈和队列，首先要说 `Deque` 接口。`Deque` 的含义是“double ended queue”，即双端队列，它是一个线性 `Collection`，支持在两端插入和移除元素，因此它既可以当作栈使用，也可以当作队列使用。大多数 `Deque` 的实现对于它们能够包含的元素数没有固定限制，但此接口既支持有容量限制的双端队列，也支持没有固定大小限制的双端队列。

`Deque` 接口提供插入、移除和检查元素的方法。每种方法都存在两种形式：一种形式在操作失败时抛出异常，另一种形式返回一个特殊值（null 或false，具体取决于操作）。插入操作的后一种形式是专为使用有容量限制的 `Deque` 实现设计的；在大多数实现中，插入操作不能失败。

下表总结了上述的 12 种方法：

<p>
<table align="center" border="1" cellpadding="3">
<tr align="center">
<td rowspan="2">&nbsp;</td>
<td colspan="2"><b>第一个元素（头部）</b></td>
<td colspan="2"><b>最后一个元素（尾部）</b></td>
</tr>
<tr align="center">
<td>抛出异常</td>
<td>特殊值</td>
<td>抛出异常</td>
<td>特殊值</td>
</tr>
<tr align="center">
<td><b>插入</b></td>
<td><a href="#4_1">addFirst(e)</a></td>
<td><a href="#4_3">offerFirst(e)</a></td>
<td><a href="#4_2">addLast(e)</a></td>
<td><a href="#4_3">offerLast(e)</a></td>
</tr>
<tr align="center">
<td><b>移除</b></td>
<td><a href="#5_2">removeFirst()</a></td>
<td><a href="#5_1">pollFirst()</a></td>
<td><a href="#5_2">removeLast()</a></td>
<td><a href="#5_1">pollLast()</a></td>
</tr>
<tr align="center">
<td><b>检查</b></td>
<td><a href="#6">getFirst()</a></td>
<td><a href="#6">peekFirst()</a></td>
<td><a href="#6">getLast()</a></td>
<td><a href="#6">peekLast()</a></td>
</tr>
</table>
</p>

`Deque` 接口扩展了 `Queue` 接口。在将双端队列用作队列时，将得到 FIFO（先进先出）行为。将元素添加到双端队列的末尾，从双端队列的开头移除元素。从 `Queue` 接口继承的方法完全等效于 `Deque` 方法，如下表所示：

<p>
<table align="center" border="1" cellpadding="3">
<tr align="center">
<td width="15%"><b>Queue 方法</b></td>
<td width="20%"><b>等效 Deque 方法</b></td>
<td><b>说明</b></td>
</tr>
<tr align="center">
<td>add(e)</td>
<td><a href="#4_2">addLast(e)</a></td>
<td>将指定元素插入此双端队列的末尾，失败则抛出异常</td>
</tr>
<tr align="center">
<td>offer(e)</td>
<td><a href="#4_3">offerLast(e)</a></td>
<td>将指定元素插入此双端队列的末尾，失败则返回 false</td>
</tr>
<tr align="center">
<td>remove()</td>
<td><a href="#5_2">removeFirst()</a></td>
<td>获取并移除此双端队列的第一个元素，如果此双端队列为空，则抛出 NoSuchElementException</td>
</tr>
<tr align="center">
<td>poll()</td>
<td><a href="#5_1">pollFirst()</a></td>
<td>获取并移除此双端队列的第一个元素，如果此双端队列为空，则返回 null</td>
</tr>
<tr align="center">
<td>element()</td>
<td><a href="#6">getFirst()</a></td>
<td>获取但不移除此双端队列的第一个元素，如果此双端队列为空，则抛出 NoSuchElementException</td>
</tr>
<tr align="center">
<td>peek()</td>
<td><a href="#6">peekFirst()</a></td>
<td>获取但不移除此双端队列的第一个元素，如果此双端队列为空，则返回 null</td>
</tr>
</table>
</p>

双端队列也可用作 LIFO（后进先出）堆栈。应优先使用此接口而不是遗留 `Stack` 类。在将双端队列用作堆栈时，元素被推入双端队列的开头并从双端队列开头弹出。堆栈方法完全等效于 `Deque` 方法，如下表所示： 

<p>
<table align="center" border="1" cellpadding="3">
<tr align="center">
<td width="15%"><b>堆栈方法</b></td>
<td width="20%"><b>等效 Deque 方法</b></td>
<td><b>说明</b></td>
</tr>
<tr align="center">
<td>push(e)</td>
<td><a href="#4_1">addFirst(e)</a></td>
<td>将指定元素插入此双端队列的开头，失败则抛出异常</td>
</tr>
<tr align="center">
<td>无</td>
<td><a href="#4_3">offerFirst(e)</a></td>
<td>将指定元素插入此双端队列的开头，失败则返回 false</td>
</tr>
<tr align="center">
<td>pop()</td>
<td><a href="#5_2">removeFirst()</a></td>
<td>获取并移除此双端队列的第一个元素，如果此双端队列为空，则抛出 NoSuchElementException</td>
</tr>
<tr align="center">
<td>无</td>
<td><a href="#5_1">pollFirst()</a></td>
<td>获取并移除此双端队列的第一个元素，如果此双端队列为空，则返回 null</td>
</tr>
<tr align="center">
<td>无</td>
<td><a href="#6">getFirst()</a></td>
<td>获取但不移除此双端队列的第一个元素，如果此双端队列为空，则抛出  NoSuchElementException</td>
</tr>
<tr align="center">
<td>peek()</td>
<td><a href="#6">peekFirst()</a></td>
<td>获取但不移除此双端队列的第一个元素，如果此双端队列为空，则返回 null</td>
</tr>
</table>
</p>

与 `List` 接口不同，`Deque` 接口不支持通过索引访问元素。

虽然 `Deque` 实现没有严格要求禁止插入 `null` 元素，但建议最好这样做。这是因为各种方法会将 `null` 用作特殊的返回值来指示双端队列为空。

`Deque` 实现通常不定义基于元素的 `equals` 和 `hashCode` 方法，而是从 `Object` 类继承基于身份的 `equals` 和 `hashCode` 方法。

<h1 id="2">ArrayDeque 概述</h1>

`ArrayDeque` 和 `LinkedList` 是 `Deque` 接口的两个通用实现。`ArrayDeque` 类在用作堆栈时快于 `Stack`，在用作队列时快于 `LinkedList`。

`ArrayDeque` 内部维护着一个 `Object[]` 数组，为了满足可以同时在数组两端插入或删除元素的需求，该数组还必须是循环的，即数组的任何一点都可能被看作起点（`head`）或者终点（`tail`）。数组双端队列没有容量限制，它们可根据需要增加以支持使用。`ArrayDeque` 不是线程安全的，在没有外部同步时，它们不支持多个线程的并发访问。此外，禁止插入 `null` 元素。

```java
transient Object[] elements;
transient int head;
transient int tail;
```

![循环数组](https://s25.postimg.org/6vcxrk31r/JCF04-01.png)

`head` 指向队列的第一个有效元素，`tail` 指向尾端第一个可以插入元素的空位。因为 `Object[]` 是循环数组，所以 `head` 不一定总等于 0，`tail` 也不一定总是比 `head` 大。

<h1 id="3">构造方法</h1>

> - ArrayDeque() 构造一个初始容量能够容纳 16 个元素的空数组双端队列。
> 
> - ArrayDeque(Collection<? extends E> c) 构造一个包含指定 Collection 的元素的双端队列，这些元素按 Collection 的迭代器返回的顺序排列。
> 
> - ArrayDeque(int numElements) 构造一个初始容量能够容纳指定数量的元素的空数组双端队列。

```java
public ArrayDeque() {
    elements = new Object[16];
}

public ArrayDeque(int numElements) {
    allocateElements(numElements);
}

public ArrayDeque(Collection<? extends E> c) {
    allocateElements(c.size());
    addAll(c);
}

private void allocateElements(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;    // MIN_INITIAL_CAPACITY = 8
    // Find the best power of two to hold elements.
    // Tests "<=" because arrays aren't kept full.
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;
        if (initialCapacity < 0) {   // Too many elements, must back off
            initialCapacity >>>= 1;  // Good luck allocating 2 ^ 30 elements
        }
    }
    elements = new Object[initialCapacity];
}
```

`allocateElements(int numElements)` 方法用于分配一个指定的队列长度，如果给定的元素个数小于默认的最小初始化容量（8），则队列长度取 8 ；如果元素个数大于 8，则队列长度取大于元素个数的 2 的最小整数次幂，如：元素个数为 10，队列长度取 2^4=16 ；元素个数为 20，队列长度取 2^5=32。

代码中最小整数次幂的求法很巧妙，下面以 x=4100 为例说明这个过程。

我们知道 `int` 类型的数据占 4 个字节，即 32 位。4100 的二进制表示为 ：

0000 0000 0000 0000 000<span style="background:#99CC99;"><font color="red">1</font> 0000 0000 0100</span>

`x |= (x >>> 1);` 的意思是先将 x 右移一位（高位补零），然后把 x 和 移位后的 x 做或运算，结果重新赋值给 x。执行后 x 的最高 2 位都变为 1 ：

0000 0000 0000 0000 000<span style="background:#99CC99;"><font color="red">1 1</font>000 0000 0110</span>

执行 `x |= (x >>> 2);` 后，x 的最高 4 位都变为 1 ：

0000 0000 0000 0000 000<span style="background:#99CC99;"><font color="red">1 111</font>0 0000 0111</span>

执行 `x |= (x >>> 4);` 后，x 的最高 8 位都变为 1 ：

0000 0000 0000 0000 000<span style="background:#99CC99;"><font color="red">1 1111 111</font>0 0111</span>

执行 `x |= (x >>> 8);` 后，x 的所有有效位都变为 1 ：

0000 0000 0000 0000 000<span style="background:#99CC99;"><font color="red">1 1111 1111 1111</font></span>

由于 x 只有 13 个有效位，再执行 `x |= (x >>> 16);`，x 不会发生变化。此时 x 的全部有效位都为 1，自增后即可得到 2 的最小整数次幂，即 2^13=8192。

如果最初指定的元素个数过多，大于 2^30，执行移位/或运算/自增后会溢出，此时 `initialCapacity` 为 `1000 0000 0000 0000 0000 0000 0000 0000`，代表一个负数，应当右移一位分配 2^30 的空间。

这个算法不仅时间效率高，而且只用到了一个变量，真可谓是短小精悍。

<h1 id="4">add 方法和 offer 方法</h1>

<h3 id="4_1">addFirst(E e)</h3>

`addFirst(E e)` 的作用将指定元素插入此双端队列的开头，也就是在 `head` 的前面插入元素，在空间足够且下标没有越界的情况下，只需要 `elements[--head] = e;` 即可。

![addFirst(E e)](https://s25.postimg.org/ifycklmcv/JCF04-02.png)

实际上还需要考虑空间是否够用以及下标是否越界的问题。如果　`head` 为 0 之后接着调用 `addFirst(E e)`，虽然空间足够，但 `head` 为 -1，下标越界了。

```java
public void addFirst(E e) {
    if (e == null) {
        throw new NullPointerException();
    }
    elements[head = (head - 1) & (elements.length - 1)] = e; // 解决下标越界问题
    if (head == tail) {
        doubleCapacity(); // 解决空间不足问题
    }
}
```

`head = (head - 1) & (elements.length - 1)` 用于解决下标越界问题。根据对 `allocateElements(int numElements)` 方法的分析，`elements.length` 一定是 2 的整数次幂，那么 `elements.length - 1` 就是二进制低位全为 1，跟 `head - 1` 与运算之后就起到了取模的作用。如果 `head` ∈ [1, elements.length - 1]，则 head = head - 1 ； 如果 `head` 为负（只可能是 -1），则对其取相对于 `elements.length` 的补码，即 head = elements.length - 1。

空间问题是在元素插入之后解决的，因为 `tail` 总是指向下一个可插入的空位，也就意味着 `elements` 数组至少有一个空位，所以插入元素的时候不用考虑空间问题。

```java
private void doubleCapacity() {
    assert head == tail;
    int p = head;
    int n = elements.length;
    int r = n - p; // number of elements to the right of p
    int newCapacity = n << 1;
    if (newCapacity < 0) {
        throw new IllegalStateException("Sorry, deque too big");
    }
    Object[] a = new Object[newCapacity];
    System.arraycopy(elements, p, a, 0, r);
    System.arraycopy(elements, 0, a, r, p);
    elements = a;
    head = 0;
    tail = n;
}
```

扩容方法 `doubleCapacity()` 的逻辑是申请一个新的数组，大小为原数组的两倍，将原数组复制到新数组中。复制分两次进行，第一次复制 `head` 右边的元素，第二次复制 `head` 左边的元素。

![doubleCapacity()](https://s25.postimg.org/5d2q1bw4v/JCF04-03.png)

<h3 id="4_2">addlast(E e)</h3>

`addLast(E e)` 的作用将指定元素插入此双端队列的末尾，也就是在 `tail` 指向的位置插入元素，由于 `tail` 总是指向下一个可以插入的空位，因此只需要 `elements[tail] = e;` 即可。插入完成后再检查空间，如果空间不足，则调用 `doubleCapacity()` 进行扩容。

```java
public void addLast(E e) {
    if (e == null) {
        throw new NullPointerException();
    }
    elements[tail] = e;
    if ( (tail = (tail + 1) & (elements.length - 1)) == head) { // 处理下标越界问题
        doubleCapacity();  // 处理空间不足问题
    }
}
```

<h3 id="4_3">offerFirst(E e) 和 offerLast(E e)</h3>

`offer` 方法内部通过调用对应的 `add` 方法实现：

```java
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}

public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```

<h1 id="5">poll 方法和 remove 方法</h1>

<h3 id="5_1">pollFirst() 和 pollLast()</h3>

> - pollFirst() 获取并移除此双端队列的第一个元素；如果此双端队列为空，则返回 null。
> 
> - polllast() 获取并移除此双端队列的最后一个元素；如果此双端队列为空，则返回 null。

```java
public E pollFirst() {
    int h = head;
    @SuppressWarnings("unchecked")
    E result = (E) elements[h];
    if (result == null) {
        return null;     // 返回 null 意味着此双端队列为空
    }
    elements[h] = null;  // 垃圾回收
    head = (h + 1) & (elements.length - 1);     // 处理下标越界问题
    return result;
}

public E pollLast() {
    int t = (tail - 1) & (elements.length - 1); // 处理下标越界问题
    @SuppressWarnings("unchecked")
    E result = (E) elements[t];
    if (result == null) {
        return null;     // 返回 null 意味着此双端队列为空
    }
    elements[t] = null;  // 垃圾回收
    tail = t;
    return result;
}
```

<h3 id="5_2">removeFirst() 和 removeLast()</h3>

> - removeFirst() 获取并移除此双端队列的第一个元素。此方法与 pollFirst() 唯一的不同在于：如果此双端队列为空，它将抛出 NoSuchElementException。
> 
> - removeLast() 获取并移除此双端队列的最后一个元素。此方法与 pollLast() 唯一的不同在于：如果此双端队列为空，它将抛出 NoSuchElementException。

```java
public E removeFirst() {
    E x = pollFirst();
    if (x == null) {
        throw new NoSuchElementException();
    }
    return x;
}

public E removeLast() {
    E x = pollLast();
    if (x == null) {
        throw new NoSuchElementException();
    }
    return x;
}
```

<h1 id="6">peek 方法和 get 方法</h1>

> - peekFirst() 获取，但不移除此双端队列的第一个元素；如果此双端队列为空，则返回 null。
> 
> - peekLast() 获取，但不移除此双端队列的最后一个元素；如果此双端队列为空，则返回 null。
> 
> - getFirst() 获取，但不移除此双端队列的第一个元素。此方法与 peekFirst() 唯一的不同在于：如果此双端队列为空，它将抛出一个 NoSuchElementException。
> 
> - getLast() 获取，但不移除此双端队列的最后一个元素。此方法与 peeklast() 唯一的不同在于：如果此双端队列为空，它将抛出一个 NoSuchElementException。

```java
public E peekFirst() {
    return (E) elements[head];
}

public E peekLast() {
    return (E) elements[(tail - 1) & (elements.length - 1)];
}

public E getFirst() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[head];
    if (result == null) {
        throw new NoSuchElementException();
    }
    return result;
}

public E getLast() {
    @SuppressWarnings("unchecked")
    E result = (E) elements[(tail - 1) & (elements.length - 1)];
    if (result == null) {
        throw new NoSuchElementException();
    }
    return result;
}
```