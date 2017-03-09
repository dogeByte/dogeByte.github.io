---
layout: post
title:  "JDK 1.8 中的 HashMap 和 HashSet"
date:   2017-03-08 21:08:21
categories: JCF
tags: Java
---

# 目录
1. [HashMap 概述](#1)
2. [构造方法](#2)
	1. [数据结构](#2_1)
	2. [常量和字段](#2_2)
	3. [构造方法](#2_3)
3. [get 方法](#3)
4. [put 方法](#4)
	1. [put(K key, V value)](#4_1)
	2. [resize()](#4_2)
5. [HashSet](#5)

同 `TreeMap` 和 `TreeSet` 一样，`HashMap` 和 `HashSet` 在 Java 里也有着相同的实现，后者仅仅是对前者做了一层包装，也就是说 `HashSet` 内部维护着一个 `HashMap`。

<h1 id="1">HashMap 概述</h1>

`HashMap` 是基于哈希表的 `Map` 接口的实现。此实现提供所有可选的映射操作，并允许使用 `null` 值和 `null` 键。（最多只允许一条记录的键为 `null`，允许多条记录的值为 `null`）

除了非同步和允许使用 `null` 之外，`HashMap` 与 `Hashtable` 大致相同。但是 `Hashtable` 是遗留类，并发性能不如引入了分段锁的 `ConcurrentHashMap`。不建议在新代码中使用 `Hashtable`，不需要线程安全的场合可以用 `HashMap` 替换，需要线程安全的场合可以用 `ConcurrentHashMap` 替换。

与 `TreeMap` 不同的是， `HashMap` 不保证映射的顺序，特别是它不保证该顺序恒久不变。根据需要， `HashMap` 可能会对元素重新哈希，元素的顺序也会被重新打散，因此不同时间迭代同一个 `HashMap` ，元素的顺序可能会不同。

JDK 1.8 中对 `HashMap` 底层的实现进行了优化，例如引入红黑树的数据结构和针对扩容的优化等。

<h1 id="2">构造方法</h1>

<h3 id="2_1">数据结构</h3>

`HashMap` 使用哈希表来存储数据。根据对冲突的处理方式不同，哈希表有两种实现方式，一种是开放地址方式（Open addressing），另一种是冲突链表方式（Separate chaining with linked lists）。Java 中 `HashMap` 采用了冲突链表方式：当数据 `key` 被 hash 后，得到数组下标，把数据放在对应下标的桶中。如果两个 `key` 得到了相同的 hash 值，即发生了 hash 冲突，相同 hash 值的元素会被存储在一个链表里。但是当同一个桶中的元素较多，即 hash 值相同的元素较多时，通过 `key.equals` 方法对链表依次查找的效率很低。在 JDK 1.8 中，当链表长度超过阈值（8）时，会将链表转换为红黑树，利用红黑树快速增删改查的特点提高了 `HashMap` 的性能。

<h3 id="2_2">常量和字段</h3>

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 默认初始化容量
static final int MAXIMUM_CAPACITY = 1 << 30;        // 最大容量
static final float DEFAULT_LOAD_FACTOR = 0.75f;     // 默认负载因子
static final int TREEIFY_THRESHOLD = 8;             // 链表转为红黑树的阈值
static final int UNTREEIFY_THRESHOLD = 6;           // 红黑树退化为链表的阈值

transient Node<K,V>[] table;                        // 存储元素的数组
transient int size;                                 // 实际存储的元素个数
transient int modCount;                             // 结构变化的次数
int threshold;                                      // 扩容阈值
final float loadFactor;                             // 加载因子
```

`Node[] table` 即哈希桶数组，其中 `Node` 是 `HashMap` 的内部类。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) && Objects.equals(value, e.getValue())) {
                return true;
            }
        }
        return false;
    }
}
```

`Node` 实现了 `Map.Entry` 接口，本质是一个映射（键值对）。

`HashMap` 的实例有两个参数影响其性能：初始容量和加载因子。容量是哈希表中桶的数量，初始容量只是哈希表在创建时的容量。加载因子 `loadFactor` 是哈希表在其容量自动增加之前可以达到多满的一种尺度。`threshold` 是 `HashMap` 所能容纳的 `Node` 的最大个数，threshold = table.length * loadFactor。当哈希表中的条目数超出了阈值 `threshold` 时，则要对该哈希表进行 rehash 操作（重建内部数据结构），从而哈希表将具有大约两倍的桶数。

通常，默认加载因子 0.75 是在时间和空间成本上寻求一种折衷。加载因子过高（可以大于 1）虽然减少了空间开销，但同时也增加了查询成本（在大多数 `HashMap` 类的操作中，包括 `get` 和 `put` 操作，都反映了这一点）。在设置初始容量时应该考虑到映射中所需的条目数及其加载因子，以便最大限度地减少 rehash 操作次数。如果很多映射关系要存储在 `HashMap` 实例中，使用足够大的初始容量使得映射关系能更有效地存储。

`modCount` 记录 `HashMap` 内部结构发生变化的次数，主要用于迭代的快速失败：在迭代器创建之后，如果从结构上对映射进行修改，除非通过迭代器本身的 `remove` 方法，其他任何时间任何方式的修改，迭代器都将抛出 `ConcurrentModificationException`。因此，面对并发的修改，迭代器很快就会完全失败，而不冒在将来不确定的时间发生任意不确定行为的风险。

在 `HashMap` 中，哈希桶数组 `table` 的长度 `length` 必须是 2 的整数次幂，是一种非常规的设计。常规的设计是把桶的大小设计为素数，例如 `Hashtable` 的初始化桶大小为 11，这是因为素数导致哈希冲突的概率要小于合数。`HashMap`的这种非常规设计，主要是为了优化取模和扩容的操作。

<h3 id="2_3">构造方法</h3>

> - HashMap() 构造一个具有默认初始容量（16）和默认加载因子（0.75）的空 HashMap。
> 
> - HashMap(int initialCapacity) 构造一个带指定初始容量和默认加载因子（0.75）的空 HashMap。
> 
> - HashMap(int initialCapacity, float loadFactor) 构造一个带指定初始容量和加载因子的空 HashMap。
> 
> - HashMap(Map<? extends K,? extends V> m) 构造一个映射关系与指定 Map 相同的新 HashMap。

```java
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0) {
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    }
    if (initialCapacity > MAXIMUM_CAPACITY) {
        initialCapacity = MAXIMUM_CAPACITY;
    }
    if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    }
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

`tableSizeFor(int cap)` 方法得到一个大于 cap 的 2 的最小整数次幂。详细分析请参考 [ArrayDeque的构造方法](https://dogebyte.github.io/jcf/2017/02/25/JCF(4)-ArrayDeque.html#3)。

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

<h1 id="3">get 方法</h1>

> get(Object key) 返回指定键所映射的值；如果对于该键来说，此映射不包含任何映射关系，则返回 null。更确切地讲，如果此映射包含一个满足 (key == null ? k == null : key.equals(k)) 的从 k 键到 v 值的映射关系，则此方法返回 v；否则返回 null。（最多只能有一个这样的映射关系）返回 null 值并不一定表明该映射不包含该键的映射关系；也可能该映射将该键显示地映射为 null。可使用 containsKey 操作来区分这两种情况。

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  // 高位运算
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 && (first = tab[(n - 1) & hash]) != null) {
        // 桶中第一个元素匹配成功则直接返回（单个元素或多个元素）
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) {
            return first;
        }
        // 桶中存在多个相同 hash 的元素
        if ((e = first.next) != null) {
            // 桶中是红黑树
            if (first instanceof TreeNode) {
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            }
            // 桶中是链表
            do {
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                    return e;
                }
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

`HashMap` 的数据结构是数组+链表+红黑树的结合，所以我们希望这个 `HashMap` 里面的元素位置尽可能分布均匀，使得每个位置上的元素数量只有一个。这样当我们使用 hash 算法求得数组索引位置时，就可以立即找到对应的元素，而不需要遍历链表，大大优化了查询的效率。

`get(Object key)` 方法中用到的 hash 算法分为三步：取 `key.hashCode()`、高位运算、取模运算。

高位运算通过 `hashCode()` 的高 16 位异或低 16 位实现的，`(h = k.hashCode()) ^ (h >>> 16)` 操作可以在 `table.length` 比较小的时候，也能保证考虑到高低位都参与到 hash 的计算中，同时不会有太大的开销。

对于任意给定的 `key` ，只要它的 `hashCode()` 返回值相同，那么 `hash(Object key)` 方法所计算得到的 hash 值总是相同的。接下来把 hash 值对数组长度取模，这样使得元素在数组中分布均匀。`getNode` 方法中使用 `(length - 1) & hash` 来代替模运算：哈希桶数组 `table` 的长度 `length` 必须是 2 的整数次幂，则 length - 1 的二进制表示的低位全为 1，那么 `(length - 1) & hash` 操作相当于 hash 对 length 取模。

`getNode(int hash, Object key)` 方法查找元素时，首先根据 key 的 hash 值找到在数组中的索引，如果该索引位置上的桶中第一个元素匹配成功（根据 equals 方法），则直接返回；反之则根据桶中存放的是红黑树或链表，用相应的方法找到该元素。

<h1 id="4">put 方法</h1>

<h3 id="4_1">put(K key, V value)</h3>

> put(K key, V value) 在此映射中关联指定值与指定键。如果该映射以前包含了一个该键的映射关系，则旧值被替换。

```java
public V put(K key, V value) {
    // 计算 hash
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 1. table 为空则创建
    if ((tab = table) == null || (n = tab.length) == 0) {
        n = (tab = resize()).length;
    } 
    // 2. 计算数组索引位置
    if ((p = tab[i = (n - 1) & hash]) == null) {
        tab[i] = newNode(hash, key, value, null);
    } else {
        Node<K,V> e; K k;
        // 3. key 已存在则直接覆盖 value
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
            e = p;
        // 4. 红黑树插入元素
        } else if (p instanceof TreeNode) {
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 5. 链表插入元素
        } else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 链表长度大于 8，转为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) {
                        treeifyBin(tab, hash);
                    } 
                    break;
                }
                // 遍历链表过程中发现 key 存在则直接覆盖 value
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
                    break;
                }
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null) {
                e.value = value;
            }
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    // 6. 容量超过阈值则进行扩容
    if (++size > threshold) {
        resize();
    }
    afterNodeInsertion(evict);
    return null;
}
```

![put 方法](https://s25.postimg.org/uq4c7ypgv/JCF06-HashMap-01-put.png)

> 1. 判断 Node[] 数组 table 是否为 null 或空，为空则执行 resize() 进行创建
> 
> 2. 计算插入元素在数组 table 中的索引位置 i，如果该位置为空则直接新建节点添加，转向 6；如果不为空则转向 3
> 
> 3. 判断 table[i] 的首个元素是否与新插入元素相同（根据 hash 值和 key.equals()），如果相同直接覆盖 value，否则转向 4
> 
> 4. 判断 table[i] 是否为 treeNode，即是否为红黑树，如果是红黑树，则直接在树中插入新元素，否则转向 5
> 
> 5. 遍历 table[i] 链表，判断其长度是否大于 8，大于 8 则把链表转换为红黑树并在红黑树中执行插入操作，否则进行链表的插入操作。遍历过程中若发现 key 已经存在则直接覆盖 value
> 
> 6. 插入成功后，判断实际存储的元素个数 size 是否超过最大容量 threshold ，如果超过则执行 resize() 进行扩容

<h3 id="4_2">resize()</h3>

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 容量超出最大值则不再扩容
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 容量和扩容阈值增加一倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY) {
            newThr = oldThr << 1; // double threshold
        }
    }
    else if (oldThr > 0) {
        newCap = oldThr;
    // 使用默认参数初始化
    } else {
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的扩容阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把原数组中的每一个桶复制到新数组中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 桶中只有一个元素
                if (e.next == null) {
                    newTab[e.hash & (newCap - 1)] = e;
                // 桶中是红黑树
                } else if (e instanceof TreeNode) {
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 桶中是链表
                } else { // preserve order
                    // 该桶中索引值不变的元素链表的首尾"指针"
                    Node<K,V> loHead = null, loTail = null;
                    // 该桶中索引值增加原数组长度的元素链表的首尾"指针"
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 索引值不变
                        if ((e.hash & oldCap) == 0) {
                            // 首次添加
                            if (loTail == null) {
                                loHead = e;
                            // 第二次及以后的添加
                            } else {
                                loTail.next = e;
                            }
                            loTail = e;
                        // 索引值为原索引值+原数组长度
                        } else {
                            // 首次添加
                            if (hiTail == null) {
                                hiHead = e;
                            // 第二次及以后的添加
                            } else {
                                hiTail.next = e;
                            }
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 索引值不变的新桶
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 索引值为原索引值+原数组长度的新桶
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

`resize()` 方法分为两步，首先计算扩容后的容量并创建新数组，然后将原数组中的每一个桶复制到新数组中。

在 JDK 1.7 中，复制操作前需要根据每个元素的 hash 值，重新计算该元素在新数组中的位置（hash % newCap）。实际上，只要元素的 hash 值不变（key.hashCode() 不变），该元素在新数组中的位置索引只有两种可能：原索引值 或 原索引值+原数组长度。例如原数组长度 oldCap 为 32，那么：

#### 情形 1

hash & oldCap == 0 时，hash 的第 6 位一定为 0：

xxxx xxxx xxxx xxxx xxxx xxxx xx<span style="background:#99CC99;"><font color="red">0</font>x xxxx</span>

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;">10 0000</span>
———————————————————————————————————————

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;">00 0000</span>

在新数组中的位置为 hash % newCap ，即 hash & (newCap - 1)：

xxxx xxxx xxxx xxxx xxxx xxxx xx<span style="background:#99CC99;"><font color="red">0</font>x xxxx</span>

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;">11 1111</span>
———————————————————————————————————————

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;"><font color="red">0</font>x xxxx</span>

这与 hash & (oldCap - 1) 的结果相同，即扩容后此元素在新数组中的索引位置与原数组一致。

#### 情形 2

hash & oldCap != 0 时，hash 的第 6 位一定为 1：

xxxx xxxx xxxx xxxx xxxx xxxx xx<span style="background:#99CC99;"><font color="red">1</font>x xxxx</span>

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;">10 0000</span>
———————————————————————————————————————

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;"><font color="red">1</font>0 0000</span>

在新数组中的位置为 hash % newCap ，即 hash & (newCap - 1)：

xxxx xxxx xxxx xxxx xxxx xxxx xx<span style="background:#99CC99;"><font color="red">1</font>x xxxx</span>

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;">11 1111</span>
———————————————————————————————————————

0000 0000 0000 0000 0000 0000 00<span style="background:#99CC99;"><font color="red">1</font>x xxxx</span>

扩容后此元素在新数组中的索引位置为：原索引值+原数组长度。

于是，在 JDK 1.8 中，`resize()` 方法在复制原数组的每一个桶时，如果桶中只有一个元素，则直接根据 key.hash 计算它在新数组中的位置；如果桶中是链表，则不再重新计算桶中各个元素在新数组中的位置，而是判断新的索引值属于上述两种情形中的哪一种，将两种不同的索引值分别放到两个桶中并存入新数组；如果桶中是红黑树，复制的过程与链表类似，只是增加了对红黑树的修复过程。

<h1 id="5">HashSet</h1>

同 `TreeMap` 和 `TreeSet` 一样，`HashSet` 也是对 `HashMap` 的简单包装，对 `HashSet` 的函数调用都会转换成合适的 `HashMap` 中的方法，因此 `HashSet` 的实现逻辑也非常简单。

```java
public class HashSet<E> extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();
    public HashSet() {
        map = new HashMap<>();
    }
    public HashSet(Collection<? extends E> c) {
        map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
        addAll(c);
    }
    public HashSet(int initialCapacity, float loadFactor) {
        map = new HashMap<>(initialCapacity, loadFactor);
    }
    public HashSet(int initialCapacity) {
        map = new HashMap<>(initialCapacity);
    }
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
}
```