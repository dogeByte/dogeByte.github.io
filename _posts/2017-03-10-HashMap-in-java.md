---
layout: post
title:  "HashMap 源码分析"
date:   2017-03-10 06:17:27
categories: Java
tags: Java
---

# 目录 
1. [数据存储](#1)
    1. [数组](#1_1)
    2. [链表](#1_2)
    3. [哈希表](#1_3)
2. [源码分析](#2)
    1. [构造方法](#2_1)
    2. [存储](#2_2)
    3. [读取](#2_3)
3. [resize](#3)
4. [Fail-Fast机制](#4)
    1. [原理](#4_1)
    2. [解决方法](#4_2)
5. [其他常见问题](#5)

HashMap是基于哈希表的Map接口的非同步实现。此提供实现所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证顺序的一直不变。

在讲解HashMap之前，需要了解一下数据的存储。

<h1 id="1">数据存储</h1>

数据结构中有数组和链表来实现对数据的存储，但这两个基本上是两个极端。

<h3 id="1_1">数组</h3>

数组的存储区间是连续的，占用内存严重，空间复杂度很大。但是数组的二分查找时间复杂度小，为O(nlogn)；数组的特点是：寻址容易，插入和删除困难。

<h3 id="1_2">链表</h3>

链表的存储区间离散，占用内存比较宽松，故空间复杂度很小，但查询的时间复杂度很大。链表的特点是：寻址困难，插入和删除容易。

<h3 id="1_3">哈希表</h3>

那么我们能不能综合两者的特性，做出一种寻址容易，插入和删除也容易的数据结构呢？答案是肯定的，这就是我们要提及的哈希表。哈希表（Hash table)既满足了数据的查找方便，同时不占用太多的存储空间，使用也方便。

关于哈希表，[维基百科](https://zh.wikipedia.org/wiki/%E5%93%88%E5%B8%8C%E8%A1%A8)的相关说明如下:

> 若关键字为k，则其值存放在f(k)的存储位置上。由此，不需比较便可直接取得所查记录。称这个对应关系F为散列函数，按这个思想建立的表为散列表。
> 
> 对不同的关键字可能得到同一散列地址，即k1 != k2，而f(k1) = f(k2)，这种现象称为冲突（英语：Collision）。具有相同函数值的关键字对该散列函数来说称做同义词。综上所述，根据散列函数f(k)和处理冲突的方法将一组关键字映射到一个有限的连续的地址集（区间）上，并以关键字在地址集中的“像”作为记录在表中的存储位置，这种表便称为散列表，这一映射过程称为散列造表或散列，所得的存储位置称散列地址。
> 
> 若对于关键字集合中的任一个关键字，经散列函数映象到地址集合中任何一个地址的概率是相等的，则称此类散列函数为均匀散列函数（Uniform Hash function），这就是使关键字经过散列函数得到一个“随机的地址”，从而减少冲突。
哈希表有多种实现方法，下面详细说明一种最常用的方法——拉链法。也可以理解为“链表的数组”，如图：

![](http://static.zybuluo.com/wl9739/pzape4m6ai20rnmbc9pfwflz/Group%203.png)

从上图我们可以发现哈希表是由数组+链表组成的。一个长度为16的数组中，每个元素存储的都是一个链表的头节点。那么这些元素是按照什么规则存储到数组中的呢，一般情况下是通过hash(key)%len获得，也就是元素的key的哈希值对数组长度的取模得到。比如下图的哈希表中，12%16=12， 28%16=12， 108%16=12.所以12、28和108都存储在数组下标为12的位置。

![](http://static.zybuluo.com/wl9739/6tx8b8i6t7c67xbkp5tsybpt/Group%202.png)

HashMap其实也是一个线性的数组实现的，所以可以理解为其存储数据的容器就是一个线性数组。这可能让我们很不理解，一个线性数组是怎么实现按键值对来存取数据的呢？这里HashMap有做一些处理。

其实HashMap里面实现一个静态内部类Entry，其重要属性有key,value,next。

需要注意的一点：HashMap不是同步的。如果多个线程同时访问同一个Hashmap，而其中至少有一个线程在数据结构上（指添加或删除一个或多个映射关系的修改操作）修改了，则必须保存外部同步。

<h1 id="2">源码分析</h1>

源码是JDK 1.7

<h3 id="2_1">构造方法</h3>

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    // Find a power of 2 >= initialCapacity
    int capacity = 1;
    while (capacity < initialCapacity)
        capacity <<= 1;
    this.loadFactor = loadFactor;
    threshold = (int)(capacity * loadFactor);
    table = new Entry[capacity];
    init();
}
```

注意代码：table = new Entry[capacity];，很明显这就是Java中创建数组的方式。也就是说，在构造方法中，创建了一个Entry数组，其大小为capacity，那么Entry又是什么呢？我们看一下源码：

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    final int hash;
    ......
}
```

我们还是看重点部分，Entry是一个static class，其中包含了key和value，也就是键值对，另外还包含了一个next的Entry指针。我们可以得出结论：Entry就是数组中的元素，每个Entry其实就是一个key-value对，它持有一个指向下一个元素的引用，这就构成了链表。

<h3 id="2_2">存储</h3>

```java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V put(K key, V value) {
    // 允许其存放null的key和null的value，当key为Null时，调用putForNullKey方法，放到table[0]这个位置
    if (key == null)
        return putForNullKey(value);
    // 通过调用hash方法对key进行hash，打到hash之后的数值。其目的是为了尽可能的让键值对可以分到不同的痛(bucket)中
    int hash = hash(key.hashCode());
    // 根据上一步骤中求出的hash得到在数组中的索引是i
    int i = indexFor(hash, table.length);
    // 如果i处的Entry不为null，则通过其next指针不断遍历e元素的下一个元素。
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
/**
 * Returns index for hash code h.
 */
static int indexFor(int h, int length) {
    return h & (length-1);
}
```

我们看一下方法的注释：在注释中首先提到了，当我们put的时候，如果key存在了，那么新的value会替代旧的value，并且如果key存在的情况下，该方法返回的是旧的value，如果key不存在，那么返回null。

而indexFor()“这个方法非常巧妙，它通过 h & (table.length -1) 来得到该对象的保存位，而 HashMap 底层数组的长度总是 2 的 n 次方，这是 HashMap 在速度上的优化。在 HashMap 构造器中有如下代码：

```java
// Find a power of 2 >= initialCapacity
int capacity = 1;
while (capacity < initialCapacity)  
    capacity <<= 1;
```

这段代码保证初始化时 HashMap 的容量总是 2 的 n 次方，即底层数组的长度总是为 2 的 n 次方。当 length 总是 2 的 n 次方时，h& (length-1)运算等价于对 length 取模，也就是 h%length，但是 & 比 % 具有更高的效率。这看上去很简单，其实比较有玄机的，简单来说，当数组长度为 2 的 n 次幂的时候，不同的 key 算得得 index 相同的几率较小，那么数据在数组上分布就比较均匀，也就是说碰撞的几率小，相对的，查询的时候就不用遍历某个位置上的链表，这样查询效率也就较高了。

从上面的源码中可以看出，当我们往HashMap中put元素的时候，先根据key的hashCode重新计算hash值，根据hash值得到这个元素在数组中的位置(即下标)，如果该位置上已经存放有其他元素了，呢么在这个位置上的元素将以链表的形式存放，加入新的放在链头，最先加入的放在链尾。如果数组该位置上没有元素，就直接将该元素放到此数组中的该位置上。

我们再来看看addEntry()方法：

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    if (size++ >= threshold)
        resize(2 * table.length);
}
```

当系统决定存储HashMap中的key-value对时，完全没有考虑Entry中的value，仅仅只是根据key来计算并决定每个Entry的存储位置。我们完全可以把Map集合中的value当成key的附属，当系统决定了key的存储位置之后，value随之保存在那里即可。

还有一个hash()方法需要说明一下：

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).
    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

hash()方法根据key的hashCode重新计算一次散列。此算法加入了高位计算，防止地位不变，高位变化时，造成的hash冲突。

总的来说，当程序试图将一个key-value对放入HashMap中时，程序首先根据该Key的hashCode()返回值决定该Entry的存储位置：如果两个Entry的key的hashCode()返回值相同，那么他们的存储位置相同。如果这两个Entry的key通过equals比较返回true，新添加Entry的value将覆盖集合中原有Entry的value，但key不会覆盖。如果这两个Entry的key通过equals比较返回false，新添加的Entry将与集合中原有的Entry形成Entry链，而且新添加的Entry位于Entry链的头部——具体说明可以看addEntry()方法说明。

<h3 id="2_3>读取</h3>

```java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
 * key.equals(k))}, then this method returns {@code v}; otherwise
 * it returns {@code null}.  (There can be at most one such mapping.)
 *
 * <p>A return value of {@code null} does not <i>necessarily</i>
 * indicate that the map contains no mapping for the key; it's also
 * possible that the map explicitly maps the key to {@code null}.
 * The {@link #containsKey containsKey} operation may be used to
 * distinguish these two cases.
 *
 * @see #put(Object, Object)
 */
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
            return e.value;
    }
    return null;
}
/**
 * Returns the entry associated with the specified key in the
 * HashMap.  Returns null if the HashMap contains no mapping
 * for the key.
 */
final Entry<K,V> getEntry(Object key) {
    int hash = (key == null) ? 0 : hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

有了上面存储时的hash算法作为基础，理解这段代码就很容易了。从上面的源代码中可以看出：从HashMap中get元素时，首先计算key的hashCode，找到数组中对应位置的某一元素，然后通过key的equals方法在对应位置的链表中找到需要的元素。

总结一下，HashMap底层是将key-value作为一个整体进行处理，这个整体就是一个Entry对象，HashMap底层使用一个Entry[]数组来保存所有的key-value对。当需要存储一个key-value对象时，会根据Hash算法来决定其在数组中的位置，再根据equals()方法决定其在数组位置上的链表中的存储位置；当需要取出一个Entry时。也会根据Hash算法找到其在数组中的存储位置，在根据equals()方法从该位置上的链表中取出该entry。

<h1 id="3">resize</h1>

当HashMap中的元素越来愈多的时候，hash冲突的几率也就越来越高，因为数组的长度是固定的。所以为了提高查询的效率，就要对HashMap的数组进行扩容。数组扩容这个操作也会出现在ArrayList中，这是一个常用的操作，而在HashMap数组扩容后，最消耗性能的点就出现了：原数组中的数据必须重新计算器在新数组中的位置，并放进去，这就是resize。

那么HashMap什么时候进行扩容呢？当HashMap中的元素个数超过(数组大小*loadFactor)时，就会进行扩容。loadFactor的默认值是0.75，这是一个折中的取值。也就是说，默认情况下，数组的大小为16，那么当Hashmap中元素个数超过16*0.75=12的时候，就会把数组的大小扩展为2*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数就能有效的提高HashMap的性能。

HashMap包括如下几个构造方法：

HashMap(): 构造一个初始容量为16，负载因子为0.75的HashMap。
HashMap(int initialCapacity): 构建一个初始容量为initialCapacity，负载因子为0.75的HashMap。
HashMap(int initialCapacity, float loadFactor): 以指定初始容量、指定的负载因子创建一个HashMap。
负载因子loadFactor衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之越小。对于使用链表的散列表来说，查找一个元素的平均时间为O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。

HashMap的实现中，通过threshold字段来判断HashMap的最大容量：

```java
threshold = (int)(capacity * loadFactor)
```

结合负载因子的定义公式可知，threshold就是此loadFator和capacity对应下允许的最大元素数目，超过这个数目就重新resize，以降低实际负载因子。默认的负载因子0.75是对空间和时间效率的一个平衡。当容量超出此最大容量时， resize后的HashMap容量是之前的两倍。

<h1 id="4">Fail-Fast机制</h1>

<h3 id="4_1">原理</h3>

我们知道java.util.HashMap不是线程安全的，因此如果在使用迭代器的过程中有其他线程修改了map，那么将抛出ConcurrentModificationException，这就是所谓的fail-fast策略。

fail-fast机制是Java集合(Collection)中的一种错误机制。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变，那么线程A访问集合时，就会抛出ConcurrentModificationException异常，产生fail-fast事件。

这一策略在源码中是通过modCount域，modCount顾名思义就是修改次数，对HashMap内容（当然不仅仅是HashMap才有，其他例如ArrayList也有）的修改豆浆增加这个值（源码中很多操作方法里面都有modCount++这句)，那么在迭代器初始化过程中会将这个值赋给迭代器的expectedModCount。

```java
private abstract class HashIterator<E> implements Iterator<E> {
    Entry<K,V> next;        // next entry to return
    int expectedModCount;   // For fast-fail
    int index;              // current slot
    Entry<K,V> current;     // current entry
    HashIterator() {
        expectedModCount = modCount;
        if (size > 0) { // advance to first entry
            Entry[] t = table;
            while (index < t.length && (next = t[index++]) == null)
                ;
        }
    }
    public final boolean hasNext() {
        return next != null;
    }
    final Entry<K,V> nextEntry() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if ((next = e.next) == null) {
            Entry[] t = table;
            while (index < t.length && (next = t[index++]) == null)
                ;
        }
        current = e;
        return e;
    }
    public void remove() {
        if (current == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        Object k = current.key;
        current = null;
        HashMap.this.removeEntryForKey(k);
        expectedModCount = modCount;
    }
}
```

在迭代过程中，判断modCount跟expectedModCount是否相等，如果不相等就表示已经有其他线程修改了Map。

注意到modCount声明为Volatile，保证线程之间修改的可见性。

在HashMap的API中指出：

由所有HashMap类的“Collection视图方法”所返回的迭代器都是快速失败的：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的remove方法，其他任何时间任何方式的修改，迭代器都将抛出ConcurrentModificationException。因此，面对并发的修改，迭代器很快就会完全失败，并在将来的不确定的时间发生任意不确定的风险。

注意，迭代器的快速失败行为不能得到保障，一般来说，存在非同步的并发修改时，不可能做出任何确定的保证。快速失败迭代器尽最大的努力抛出ConcurrentModificationException，因此，编写依赖于此异常的程序的做法是错误的，正确的做法是：迭代器的快速失败行为应该仅用于检测程序错误。

<h3 id="4_2">解决方案</h3>

在上文中提到，fail-fast机制，是一种错误检测机制。它只能被用来检测错误，因为JDK并不保证fail-fast机制一定会发生。若在多线程环境下使用fail-fast机制的集合，建议使用“java.util.concurrent”包下的类去取代“java.util”包下的类。

HashMap的两种遍历方式

第一种：

```java
Map map = new HashMap<>();
Iterator iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry entry = (Map.Entry) iterator.next();
    Object k = entry.getKey();
    Object val = entry.getValue();
}
```
效率高，建议使用此方式！

第二种：

```java
Map map = new HashMap();
Iterator iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    Object key = iterator.next();
    Object val = map.get(key);
}
```
效率低，不建议使用此方式。

<h1 id="5">其他常见问题</h1>

HashMap对键的类型有什么要求？为什么常用字符串作为键。
HashMap的键需要重写hashCode()方法和equals()方法。
String类是不可变类型，而且已经重写了hashCode()方法和equals()方法，所以非常适合作为HashMap的键。