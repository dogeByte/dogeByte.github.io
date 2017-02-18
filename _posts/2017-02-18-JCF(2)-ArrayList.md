---
layout: post
title:  "ArrayList"
date:   2017-02-18 08:34:35
categories: JCF
tags: Java
---

# 目录
1. [构造方法](#1)
2. [set 和 get](#2)
3. [add](#3)
4. [remove](#4)
5. [序列化](#5)
6. [迭代器](#6)

`ArrayList` 继承抽象类 `AbstractList` 并实现接口 `List` 。实际上，集合框架的主体架构是以类为主体的，而不是接口。如果知道类、抽象类和接口之间的区别与联系，就能明白这里引入抽象类的原因。简言之，如果不引入抽象类，每个实现了 `Collection` 接口的类都必须实现该接口中的 10 多个方法，而通过继承抽象类 `AbstractList`，去除了重复代码。

`ArrayList` 是一个顺序容器，即元素存放的顺序与放入的顺序相同，并且允许放入 `null` 元素，底层通过一个 `Object[]` 数组 `elementData` 实现。 `elementData` 之所以没有定义为 `private` ，是为了简化其内部类的访问。

```java
transient Object[] elementData;
```

每个 `ArrayList` 都有一个容量（capacity），表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动将底层数组扩容。

需要注意的是， `ArrayList` 不是线程安全的。如果多个线程同时访问一个 `ArrayList` 实例，而其中至少一个线程从结构上修改了列表，那么它必须保持外部同步。结构上的修改是指任何添加或删除一个或多个元素的操作，或者显式调整底层数组的大小；仅仅设置元素的值不是结构上的修改。这一般通过对自然封装该列表的对象进行同步操作来完成。如果不存在这样的对象，则应该使用工具类 `Collections` 中的 `synchronizedList` 方法将该列表“包装”起来。这最好在创建时完成，以防止意外对列表进行不同步的访问：

```java
List list = Collections.synchronizedList(new ArrayList(...));
```

<h1 id="1">构造方法</h1>

`ArrayList<E>` 共有三个重载的构造方法：

```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
    }
}

public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```


> - ArrayList() 构造一个初始容量为 10 的空列表。
> 
> - ArrayList(int initialCapacity) 构造一个具有指定初始容量的空列表。
> 
> - ArrayList(Collection<? extends E> c) 构造一个包含指定 Collection 元素的列表，这些元素是按照该 Collection 的迭代器返回它们的顺序排列的。

<h1 id="2">set 和 get</h1>

`ArrayList` 的 `set` 方法非常简单，直接对 `Object[]` 数组的指定位置赋值即可。

```java
public E set(int index, E element) {
    rangeCheck(index);    // 下标越界检查，越界则抛出 IndexOutOfBoundsException
    E oldValue = elementData(index);
    elementData[index] = element;
    return oldValue;
}
```

`get` 方法同样很简单，唯一要注意的是由于底层数组是 `Object[]` ，得到元素后需要进行类型转换。

```java
public E get(int index) {
    rangeCheck(index);
    return elementData(index);
}

@SuppressWarnings("unchecked")
E elementData(int index) {
    return (E) elementData[index];
}
```

<h1 id="3">add</h1>

添加元素可能会导致容量 capacity 不足，因此在添加元素之前，都需要调用 `ensureCapacityInternal(int minCapacity)` 方法进行剩余空间检查，如果需要则自动扩容。扩容操作最终是通过 `grow(int minCapacity)` 方法完成的。

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);    // 扩容后容量变为原来的 1.5 倍
    if (newCapacity - minCapacity < 0) {
        newCapacity = minCapacity;
    }
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        newCapacity = hugeCapacity(minCapacity);
    }
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);    //扩容并复制
}
```

`grow(int minCapacity)` 方法中新建了一个容量为原数组的 1.5 倍的 `Object[]` 数组，并将原数组的数据复制到新数组中，实现了容器扩容的操作。由于 GC 自动管理了内存，这里也就不需要考虑原数组内存释放的问题。

> - add(E e) 方法将指定的元素添加到此列表的尾部。
> 
> - add(int index, E element) 方法将指定的元素插入此列表中的指定位置。


```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

public void add(int index, E element) {
    rangeCheckForAdd(index);
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}
```

`add(E e)` 方法以分摊的固定时间运行，也就是说，添加 n 个元素需要 O(n) 时间。而 `add(int index, E e)` 需要先将插入点后的元素后移一位，然后才能完成插入操作。

> - addAll(Collection<? extends E> c) 方法按照指定 Collection 的迭代器所返回的元素顺序，将该 Collection 中的所有元素添加到此列表的尾部。
> 
> - addAll(int index, Collection<? extends E> c) 方法从指定的位置开始，将指定 Collection 中的所有元素插入到此列表中。

```java
public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    return numNew != 0;
}

public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);
    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount
    int numMoved = size - index;
    if (numMoved > 0) {
        System.arraycopy(elementData, index, elementData, index + numNew, numMoved);
    }
    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

与 `add` 方法类似，在插入之前也需要进行空间检查，如果需要则自动扩容；如果从指定位置插入，也会存在移动元素的情况。 `addAll` 方法的时间复杂度不仅跟插入元素的数量有关，也跟插入的位置有关。

<h1 id="4">remove</h1>

> - remove(int index) 方法移除此列表中指定位置上的元素。
> 
> - remove(Object o) 方法移除此列表中首次出现的指定元素（如果存在）。
> 
> - removeRange(int fromIndex, int toIndex) 方法移除列表中索引在 fromIndex（包括）和 toIndex（不包括）之间的所有元素。

`remove` 方法是 `add` 方法的逆操作，需要将删除点之后的元素向前移动。需要注意的是，为了让 GC 起作用，必须显式地将移除的位置设置为 `null` ，否则会有内存泄漏的风险。

```java
public E remove(int index) {
    rangeCheck(index);
    modCount++;
    E oldValue = elementData(index);
    int numMoved = size - index - 1;
    if (numMoved > 0) {
        System.arraycopy(elementData, index+1, elementData, index, numMoved);
    }
    elementData[--size] = null;    // clear to let GC do its work
    return oldValue;
}

public boolean remove(Object o) {
    if (o == null) {
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

protected void removeRange(int fromIndex, int toIndex) {
    modCount++;
    int numMoved = size - toIndex;
    System.arraycopy(elementData, toIndex, elementData, fromIndex, numMoved);
    // clear to let GC do its work
    int newSize = size - (toIndex-fromIndex);
    for (int i = newSize; i < size; i++) {
        elementData[i] = null;
    }
    size = newSize;
}
```

<h1 id="5">序列化</h1>

`ArrayList` 实现了接口 `Serializable` ，但是内部 `Object[]` 数组 `elementData` 却使用了 `transient` 修饰符。 `transient` 修饰符的作用正是让被其修饰的变量不参与正常的序列化中，那么 `ArrayList` 是如何实现序列化的呢？

```java
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException{
    // Write out element count, and any hidden stuff
    int expectedModCount = modCount;
    s.defaultWriteObject();
    // Write out size as capacity for behavioural compatibility with clone()
    s.writeInt(size);
    // Write out all elements in the proper order.
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

private void readObject(java.io.ObjectInputStream s) throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;
    // Read in size, and any hidden stuff
    s.defaultReadObject();
    // Read in capacity
    s.readInt(); // ignored
    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        ensureCapacityInternal(size);
        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

对于实现了 `Serializable` 接口的类，可以直接使用 `ObjectOutputStream` 类和 `ObjectInputStream` 类进行对象的序列化和反序列化。当我们的类中包含有 `writeObject(ObjectOutputStream s)` 和 `readObject(ObjectOutputStream s)` 这两个方法时， `ObjectOutputStream` 就会调用类中的 `writeObject(ObjectOutputStream s)` 方法。但是我们会发现 `writeObject(ObjectOutputStream s)` 方法是私有的，同时该方法在 `Object` 中并不存在，那么 `ObjectOutputStream` 作为外部类是如何找到 `writeObject(ObjectOutputStream s)` 方法并调用的呢？事实上，此处正是使用的反射机制。之所以如此操作，是为了提高效率，毕竟只需要序列化 `elementData` 中 `size` 大小的变量，而不是 `capacity` 大小的变量。

<h1 id="6">迭代器</h1>

细心的话可以注意到 `add` 方法和 `remove` 方法中经常会出现一个变量 `modCount` ，它其实是 modifyCount 的缩写，表示 `ArrayList` 对象被修改的次数。这个变量记录的只是修改 `size` 的次数，而 `ArrayList` 对象中变量的替换则不会改变 `modCount` 的值。那么 `ArrayList` 类为什么要辛苦地去维护这个 `modCount` 变量值呢？如果使用增强 for 循环来遍历 `ArrayList` 中的元素，并试图删除其中的某些元素，那么一定会遇到 `ConcurrentModificationException` 异常。背后的原理就在于 `remove` 方法会修改 `modCount` 的值，而整个 `foreach` 循环过程则不允许 `modCount` 值发生改变。

`ArrayList` 中的 `iterator`方法 和 `listIterator` 方法返回的迭代器是快速失败的：在创建迭代器之后，除非通过迭代器自身的 `remove` 或 `add` 方法从结构上对列表进行修改，否则在任何时间以任何方式对列表进行修改，迭代器都会抛出 `ConcurrentModificationException` 。因此，面对并发的修改，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

注意，迭代器的快速失败行为无法得到保证，因为一般来说，不可能对是否出现不同步并发修改做出任何硬性保证。快速失败迭代器会尽最大努力抛出 `ConcurrentModificationException` 。因此，为提高这类迭代器的正确性而编写一个依赖于此异常的程序是错误的做法：迭代器的快速失败行为应该仅用于检测 bug 。