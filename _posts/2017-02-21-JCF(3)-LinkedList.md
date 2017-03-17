---
layout: post
title:  "LinkedList"
date:   2017-02-21 09:49:12
categories: JCF
tags: Java
---

# 目录
1. [构造方法](#1)
2. [set 方法和 get 方法](#2)
	1. [set 方法](#2_1)
	2. [get 方法](#2_2)
3. [add 方法](#3)
    1. [add(E e) 方法](#3_1)
    2. [add(int index, E element) 方法](#3_2)
    3. [addAll 方法](#3_3)
4. [remove 方法](#4)
5. [ArrayList 和 LinkedList](#5)

`ArrayList` 作为动态数组的模拟，使用的是连续内存空间来存储数据，带来了可随机访问数据元素这一便利的同时，也有着插入和删除效率低下的缺点。对于插入和删除操作频率较高的场合，应该考虑使用`LinkedList`。

`LinkedList` 同时实现了 `List` 接口和 `Deque` 接口，也就是说它既可以看作一个顺序容器，又可以看作一个队列（`Queue`），同时又可以看作一个栈（`Stack`）。

`LinkedList` 底层通过双向链表实现，允许放入 `null` 元素，每个节点用内部类 `Node` 表示。`LinkedList` 中 `Node` 的实例对象 `first` 和 `last` 引用分别指向链表的第一个和最后一个元素。注意 `LinkedList` 中没有哑元，当链表为空的时候 `first` 和 `last` 都指向 `null`。

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

可见，Java 语法中的引用充当了 C/C++ 语言中指针的角色。

`LinkedList` 的实现方式决定了所有跟下标相关的操作都是线性时间，而在首尾删除元素只需要常数时间。为了追求效率 `LinkedList` 也没有实现同步。如果多个线程同时访问一个 `LinkedList` 实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。结构上的修改是指任何添加或删除一个或多个元素的操作，或者显式调整底层数组的大小；仅仅设置元素的值不是结构上的修改。这一般通过对自然封装该列表的对象进行同步操作来完成。如果不存在这样的对象，则应该使用工具类 `Collections` 中的 `synchronizedList` 方法将该列表“包装”起来。这最好在创建时完成，以防止意外对列表进行不同步的访问：

```java
List list = Collections.synchronizedList(new LinkedList(...));
```

<h1 id="1">构造方法</h1>

> - LinkedList() 构造一个空列表。
> 
> - LinkedList(Collection<? extends E> c) 构造一个包含指定 Collection 中的元素的列表，这些元素按其 Collection 的迭代器返回的顺序排列。

```java
public LinkedList() {
}

public LinkedList(Collection<? extends E> c) {
    this();
    addAll(c);
}
```

<h1 id="2">set 方法和 get 方法</h1>

<h3 id="2_1">set 方法</h3>

> set(int index, E element) 方法将此列表中指定位置的元素替换为指定的元素。

```java
public E set(int index, E element) {
    checkElementIndex(index);    // index >= 0 && index < size
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}

Node<E> node(int index) {
    // assert isElementIndex(index);
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++) {
            x = x.next;
        }
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--) {
            x = x.prev;
        }
        return x;
    }
}
```

`node(int index)` 方法用于查找指定位置的 `Node<E>` 对象，由于链表是双向的，所以可以从前端查找，也可以从后端查找，具体寻找的方向取决于条件 `index < (size >> 1)`，即 `index` 更靠近前端还是后端。

除了使用 `set(int index, E element)` 方法，也可以先通过 `node(int index)` 方法找到对应下标元素的引用，然后修改 `Node<E>` 对象中 `item` 的值。

<h3 id="2_2">get 方法</h3>

```java
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

<h1 id="3">add</h1>

<h3 id="3_1">add(E e) 方法</h3>

> add(E e) 方法将指定元素添加到此列表的结尾。

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}

void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null) {
        first = newNode;    // 原链表为空
    } else {
        l.next = newNode;
    }
    size++;
    modCount++;
}
```

`add(E e)` 方法的逻辑非常简单，由于内部类 `Node` 的实例对象 `last` 指向链表末尾，在末尾插入元素的花费的是常数时间，只需要简单修改几个相关引用即可。

<h3 id="3_2">add(int index, E element) 方法</h3>

> add(int index, E element) 方法在此列表中指定的位置插入指定的元素。

```java
public void add(int index, E element) {
    checkPositionIndex(index);    // index >= 0 && index <= size
    if (index == size) {
        linkLast(element);        // 链表为空或插入到链表末尾
    } else {
        linkBefore(element, node(index));
    }
}

void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null) {
        first = newNode;    // 插入位置为 0
    } else {
        pred.next = newNode;
    }
    size++;
    modCount++;
}
```

`add(int index, E element)` 方法的逻辑稍显复杂，可以分成两步：

1. 根据 `index` 找到要插入的位置。
2. 修改引用，完成插入操作。

<h3 id="3_3">addAll 方法</h3>

> - addAll(Collection<? extends E> c) 方法添加指定 Collection 中的所有元素到此列表的结尾，顺序是指定 Collection 的迭代器返回这些元素的顺序。
> 
> - boolean addAll(int index, Collection<? extends E> c) 方法将指定 Collection 中的所有元素从指定位置开始插入此列表。

```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

public boolean addAll(int index, Collection<? extends E> c) {
    checkPositionIndex(index);    // index >= 0 && index <= size
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0) {
        return false;
    }
    Node<E> pred, succ;
    if (index == size) {          // 链表为空或插入到链表末尾
        succ = null;
        pred = last;
    } else {
        succ = node(index);
        pred = succ.prev;
    }
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null) {
            first = newNode;
        } else {
            pred.next = newNode;
        }
        pred = newNode;
    }
    if (succ == null) {          // 链表为空或插入到链表末尾
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }
    size += numNew;
    modCount++;
    return true;
}
```

<h1 id="4">remove 方法</h1>

> - remove() 方法获取并移除此列表的头（第一个元素）。
> 
> - E remove(int index) 方法移除此列表中指定位置处的元素。
> 
> - boolean remove(Object o) 方法从此列表中移除首次出现的指定元素（如果存在）。


```java
public E remove() {
    return removeFirst();
}

public E remove(int index) {
    checkElementIndex(index);    // index >= 0 && index < size
    return unlink(node(index));
}

public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

删除操作也可以分为两步

> 1. 找到要删除元素的引用。
> 
> 2. 修改相关引用，完成删除操作。

`remove(int index)` 方法和 `remove(Object o)` 方法在寻找被删除元素的引用时都是线性时间复杂度。第二步中相关引用的修改是通过

在寻找被删元素引用的时候remove(Object o)调用的是元素的equals方法，而remove(int index)使用的是下标计数，两种方式都是线性时间复杂度。在步骤2中，两个revome()方法都是通过 `unlink(Node<E> x)` 方法完成的，注意需要考虑被删除元素是链表首尾时的情况。

```java
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;
    if (prev == null) {    // 删除的是第一个元素
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }
    if (next == null) {    // 删除的是最后一个元素
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }
    x.item = null;         // 由 GC 回收
    size--;
    modCount++;
    return element;
}
```

<h1 id="5">ArrayList 和 LinkedList</h1>

<table width="100%" border="1">
<thead>
<tr align="center">
<td>类</td>
<td>内存占用</td>
<td>取值和修改</td>
<td>插入和删除</td>
</tr>
</thead>
<tbody>
<tr align="center">
<td>ArrayList</td>
<td>连续空间，50%增长</td>
<td>随机访问，O(1)</td>
<td>需要移动元素，O(n)</td>
</tr>
<tr align="center">
<td>LinkedList</td>
<td>离散空间，按需增长</td>
<td>顺序访问，O(n)</td>
<td>直接操作，O(1)</td>
</tr>
</tbody>
</table>