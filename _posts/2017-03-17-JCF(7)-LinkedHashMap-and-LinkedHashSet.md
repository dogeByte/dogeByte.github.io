---
layout: post
title:  "LinkedHashMap 和 LinkedHashSet"
date:   2017-03-17 15:36:12
categories: JCF
tags: Java
---

# 目录
1. [LinkedHashMap 概述](#1)
2. [构造方法](#2)
3. [get 方法](#3)
4. [put 方法](#4)
5. [remove 方法](#5)
6. [LinkedHashSet](#6)

`LinkedHashMap` 和 `LinkedHashSet` 在 Java 里也有着相同的实现，后者仅仅是对前者做了一层包装，也就是说 `LinkedHashSet` 内部维护着一个 `LinkedHashMap`。

<h1 id="1">LinkedHashMap 概述</h1>

`LinkedHashMap` 是 `Map` 接口的哈希表和链表实现，允许使用 null 值（一条）和 null 键（多条），具有可预知的迭代顺序。事实上， `LinkedHashMap` 是 `HashMap` 的直接子类，二者的不同之处在于，前者维护着一个运行于所有条目的双向链表。此链表定义了迭代顺序，该迭代顺序通常就是将键插入到映射中的顺序。需要注意的是，如果在映射中重新插入键，则插入顺序不受影响。

除了可以保证迭代顺序，这种结构还有一个好处：迭代 `LinkedHashMap` 时不需要像 `HashMap` 那样遍历整个 `table`，而只需要遍历双向链表即可。也就是说 `LinkedHashMap` 的迭代时间只与 `entry` 的个数相关，而与 `table.length` 无关。

`LinkedHashMap` 具有两个影响其性能的参数：初始容量和加载因子。它们的定义与 `HashMap` 极其相似，不同之处在于，选择较大的初始容量对 `LinkedHashMap` 的影响比对 `HashMap` 要小，因为 `LinkedHashMap` 的迭代时间不受容量的影响。

<h1 id="2">构造方法</h1>

> - LinkedHashMap() 构造一个带默认初始容量 (16) 和加载因子 (0.75) 的空插入顺序 LinkedHashMap 实例。
> 
> - LinkedHashMap(int initialCapacity) 构造一个带指定初始容量和默认加载因子 (0.75) 的空插入顺序 LinkedHashMap 实例。
> 
> - LinkedHashMap(int initialCapacity, float loadFactor) 构造一个带指定初始容量和加载因子的空插入顺序 LinkedHashMap 实例。
> 
> - LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) 构造一个带指定初始容量、加载因子和排序模式的空 LinkedHashMap 实例。通过指定 accessOrder 的值，可以控制访问顺序：对于访问顺序，为 true；对于插入顺序，则为 false。

> 
> - LinkedHashMap(Map<? extends K,? extends V> m) 构造一个映射关系与指定映射相同的插入顺序 LinkedHashMap 实例。


```java
public LinkedHashMap() {
    super();
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity, float loadFactor, boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}

public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}
```

通过如下方式可以得到一个与源 `Map` 迭代顺序相同的 `LinkedHashMap`：

```java
void foo(Map m) {
    Map copy = new LinkedHashMap(m);
}
```

<h1 id="3">get 方法</h1>

> get(Object key) 返回指定键所映射的值；如果对于该键来说，此映射不包含任何映射关系，则返回 null。更确切地讲，如果此映射包含一个满足 (key == null ? k == null : key.equals(k)) 的从 k 键到 v 值的映射关系，则此方法返回 v；否则返回 null。（最多只能有一个这样的映射关系）返回 null 值并不一定表明该映射不包含该键的映射关系；也可能该映射将该键显示地映射为 null。可使用 containsKey 操作来区分这两种情况。

```java
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null) {
        return null;
    }
    if (accessOrder) {
        afterNodeAccess(e);
    }
    return e.value;
}
```

其中 getNode(int hash, Object key) 方法继承自父类 `HashMap`。如果指定了访问顺序，则需要调用 afterNodeAccess(Node<K,V> e) 方法，将被访问的元素链接到链表的尾端。`LinkedHashMap` 重写了 `HashMap` 中的此方法。事实上，`HashMap` 中的 afterNodeAccess(Node<K,V> e) 方法内并没有任何语句。

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    // 指定了访问顺序且访问的元素不是尾节点
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null) {
            head = a;
        } else {
            b.after = a;
        }
        if (a != null) {
            a.before = b;
        } else {
            last = b;
        }
        if (last == null) {
            head = p;
        } else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

`LinkedHashMap.Entry` 是 `HashMap.Node` 的子类，在其基础上增加了 before 和 after 两个链表指针：

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

<h1 id="4">put 方法</h1>


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
        tab[i] = newNode(hash, key, value, null);  // 重写
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
            afterNodeAccess(e);  // 重写
            return oldValue;
        }
    }
    ++modCount;
    // 6. 容量超过阈值则进行扩容
    if (++size > threshold) {
        resize();
    }
    afterNodeInsertion(evict);   // 重写
    return null;
}
```

`LinkedHashMap` 在插入元素时，直接调用了父类 `HashMap` 中的 put 方法，不同之处在于重写了 newNode(int hash, K key, V value, Node<K,V> e)、afterNodeAccess(Node<K,V> e) 和 afterNodeInsertion(boolean evict) 三个方法：

```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null) {
        head = p;
    } else {
        p.before = last;
        last.after = p;
    }
}
```

可以看到，newNode 方法不仅新建了一个节点，还把这个节点链接到双向链表的末尾，这个操作维护了插入的顺序。

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

removeEldestEntry 直接返回 false，这意味着如果直接使用 `LinkedHashMap`，在插入新元素时，是不会删除最老的元素的。如果要实现 FIFO 替换策略的缓存，可以继承 `LinkedHashMap` 并重写 removeEldestEntry 方法：

```java
public class FIFOCache<K, V> extends LinkedHashMap<K, V> {
    private static final long serialVersionUID = 1L;
    private final int cacheSize;
    public FIFOCache(int cacheSize) {
        this.cacheSize = cacheSize;
    }
    @Override
    protected boolean removeEldestEntry(java.util.Map.Entry<K, V> eldest) {
        return size() > cacheSize;
    }
}
```

<h1 id="5">remove 方法</h1>

与 put 方法类似，`LinkedHashMap` 在删除元素时也是直接调用了父类 `HashMap` 的 remove 方法，并重写了其中的 afterNodeRemoval 方法：

```java
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null) {
        head = a;
    } else {
        b.after = a;
    }
    if (a == null) {
        tail = b;
    } else {
        a.before = b;
    }
}
```

显然，afterNodeRemoval 方法解除了被删除元素与双向链表的关联。

<h1 id="6">linkedHashSet</h1>

`LinkedHashSet` 内部维护着一个 `LinkedHashMap`，通过调用父类的[构造方法](https://dogebyte.github.io/jcf/2017/03/08/JCF(6)-HashMap-and-HashSet-in-JDK1.8.html#5) HashSet(int initialCapacity, float loadFactor, boolean dummy) 实现。

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {
    public LinkedHashSet() {
        super(16, .75f, true);
    }
    public LinkedHashSet(int initialCapacity) {
        super(initialCapacity, .75f, true);
    }
    public LinkedHashSet(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor, true);
    }
    public LinkedHashSet(Collection<? extends E> c) {
        super(Math.max(2*c.size(), 11), .75f, true);
        addAll(c);
    }
}
```