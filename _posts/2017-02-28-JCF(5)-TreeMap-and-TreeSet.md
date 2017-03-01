---
layout: post
title:  "TreeMap 和 TreeSet"
date:   2017-02-28 07:49:00
categories: JCF
tags: Java
---

# 目录
1. [TreeMap 概述](#1)
2. [红黑树](#2)
	1. [约束条件](#2_1)
	2. [查找后继节点](#2_2)
	3. [颜色调换和树旋转](#2_3)
3. [构造方法](#3)
4. [get 方法](#4)
5. [put 方法](#5)
6. [remove 方法](#6)

之所以把 `TreeMap` 和 `TreeSet` 放在一起讲解，是因为二者在 Java 里有着相同的实现，后者仅仅是对前者做了一层包装，也就是说 `TreeSet` 内部维护着一个 `TreeMap` 。

<h1 id="TreeMap 概述"></h1>

`TreeMap` 实现了 `SortedMap` 接口，该映射根据其键（`key`）的自然顺序进行排序，或者根据创建映射时提供的 `Comparator` 进行排序，具体取决于使用的构造方法。

`TreeMap` 底层通过红黑树实现，此实现为 `containsKey` 、 `get` 、 `put` 和 `remove` 操作提供受保证的 log(n) 时间开销。 `TreeMap` 中使用内部类 `Entry` 表示红黑树的一个节点：

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value；
    Entry<K,V> left;         // 左子节点
    Entry<K,V> right;        // 右子节点
    Entry<K,V> parent;       // 父节点
    boolean color = BLACK;   // 颜色
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }
}
```

同样， `TreeMap` 不是同步的。如果多个线程同时访问一个映射，并且其中至少一个线程从结构上修改了该映射，则其必须外部同步。（结构上的修改是指添加或删除一个或多个映射关系的操作；仅改变与现有键关联的值不是结构上的修改。）这一般是通过对自然封装该映射的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 `Collections.synchronizedSortedMap` 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的不同步访问：

```java
SortedMap map = Collections.synchronizedSortedMap(new TreeMap(...));
```

<h1 id="2">红黑树</h1>

<h3 id="2_1">约束条件</h3>

红黑树是每个节点都带有颜色属性的二叉查找树，颜色为红色或黑色。

![红黑树](https://s25.postimg.org/swj20oar3/JCF05-TreeMap-01-red-black-tree.png)

除了二叉查找树的一般要求以外，对于任何有效的红黑树还有如下的额外要求（性质）：

> 1. 节点是红色或黑色。
> 
> 2. 根节点是黑色。
> 
> 3. 所有叶子节点都是黑色。
> 
> 4. 从每个叶子节点到根节点的所有路径上不能有两个连续的红色节点。
> 
> 5. 从任一节点到其每个叶子节点的所有简单路径都包含相同数目的黑色节点。

这些约束确保了红黑树的关键特性：红黑树是一种近似平衡的二叉查找树，它能保证从根节点到叶子节点的最长可能路径不多于最短可能路径的两倍长。这是因为，最短的可能路径全是黑色节点，最长的可能路径有交替的红色和黑色节点。同时，从任一节点到其每个叶子节点的所有简单路径都包含相同数目的黑色节点，这就表明了没有路径能多于任何其他路径的两倍长。

<h3 id="2_2">查找后继节点</h3>

一个节点的后继节点是指大于它的最小元素，寻找后继节点分为两种情况：

> 1. 节点的右子树不为空，则后继节点是其右子树中的最小元素。
> 
> 2. 节点的右子树为空，则后继节点是其第一个向左走的祖先。

```java
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null) {
        return null;
    } else if (t.right != null) {   // 右子树不为空
        Entry<K,V> p = t.right;
        while (p.left != null) {
            p = p.left;
        }
        return p;
    } else {    // 右子树为空
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

<h3 id="2_3">颜色调换和树旋转</h3>

因为每一个红黑树也是一个特化的二叉查找树，因此红黑树上的只读操作与普通二叉查找树上的只读操作相同。然而，在红黑树上进行插入操作和删除操作会导致不再匹配红黑树的性质。恢复红黑树的性质需要少量 O(log n) 的颜色调换（实际是非常快速的）和不超过三次树旋转（对于插入操作是两次）。虽然插入和删除很复杂，但操作时间仍可以保持为 O(log n) 次。

左旋转的过程是将 x 的右子节点绕 x 逆时针旋转，使得 x 的右子节点成为 x 的父节点，同时修改相关节点的引用。树旋转之后，二叉查找树的属性仍然满足。

![左旋转](https://s25.postimg.org/a5h4qig6n/JCF05-TreeMap-02-rotate-left.png)

```java
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> r = p.right;
        p.right = r.left;
        if (r.left != null) {
            r.left.parent = p;
        }
        r.parent = p.parent;
        if (p.parent == null) {
            root = r;
        } else if (p.parent.left == p) {
            p.parent.left = r;
        } else {
            p.parent.right = r;
        }
        r.left = p;
        p.parent = r;
    }
}
```

右旋转的过程是将 x 的左子节点绕 x 顺时针旋转，使得 x 的左子节点成为 x 的父节点，同时修改相关节点的引用。树旋转之后，二叉查找树的属性仍然满足。

![右旋转](https://s25.postimg.org/quikmfcrz/JCF05-TreeMap-03-rotate-right.png)

```java
private void rotateRight(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> l = p.left;
        p.left = l.right;
        if (l.right != null) {
            l.right.parent = p;
        }
        l.parent = p.parent;
        if (p.parent == null) {
            root = l;
        } else if (p.parent.right == p) {
            p.parent.right = l;
        } else {
            p.parent.left = l;
        }
        l.right = p;
        p.parent = l;
    }
}
```

<h1 id="3">构造方法</h1>

> - TreeMap() 使用键的自然顺序构造一个新的、空的树映射。
> 
> - TreeMap(Comparator<? super K> comparator) 构造一个新的、空的树映射，该映射根据给定比较器进行排序。
> 
> - TreeMap(Map<? extends K,? extends V> m) 构造一个与给定映射具有相同映射关系的新的树映射，该映射根据其键的自然顺序进行排序。
> 
> - TreeMap(SortedMap<K,? extends V> m) 构造一个与指定有序映射具有相同映射关系和相同排序顺序的新的树映射。

```java
public TreeMap() {
    comparator = null;
}

public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}

public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}

public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

<h1 id="4">get 方法</h1>

> get(Object key) 返回指定键所映射的值，如果对于该键而言，此映射不包含任何映射关系，则返回 null。

```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p == null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    if (comparator != null) {
        return getEntryUsingComparator(key);  // 使用指定的比较器顺序
    }
    if (key == null) {
        throw new NullPointerException();
    }
    @SuppressWarnings("unchecked")
    Comparable<? super K> k = (Comparable<? super K>) key;  // 使用 key 的自然顺序
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0) {
            p = p.left;    // 向左找
        } else if (cmp > 0) {
            p = p.right;   // 向右找
        } else {
            return p;
        }
    }
    return null;
}
```

`get(Object key)` 方法根据指定的 `key` 值返回对应的 `value` ，该方法调用了 `getEntry(Object key)` 得到相应的 `Entry` ，然后返回 `entry.value` 。 `getEntry(Object key)` 的逻辑是根据 `key` 的自然顺序（或者比较器顺序）对二叉查找树进行查找，直到找到满足`k.compareTo(p.key) == 0` 的 `Entry` 。

<h1 id="5">put 方法</h1>

> put(K key, V value) 将指定值与此映射中的指定键进行关联。如果该映射以前包含此键的映射关系，那么将替换旧值。

```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check
        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) { // 使用指定的比较器顺序
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0) {
                t = t.left;   // 向左找
            } else if (cmp > 0) {
                t = t.right;  // 向右找
            } else {
                return t.setValue(value);
            }
        } while (t != null);
    }
    else { // 使用 key 的自然顺序
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0) {
                t = t.left;   // 向左找
            } else if (cmp > 0) {
                t = t.right;  // 向右找
            } else {
                return t.setValue(value);
            }
        } while (t != null);
    }
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0) {
        parent.left = e;
    } else {
        parent.right = e;
    }
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

`put(K key, V value)` 方法首先会查看根节点是否为空，如果为空则将新节点置于根节点上。由于 `Entry` 中 `color` 字段默认为 `BLACK` ，此时满足性质 2 和性质 5 。

如果根节点不为空，则继续查找 `TreeMap` 中是否包含该 `key` ，查找过程类似于`getEntry(Object key)` 。如果已经包含则替换旧值并返回；如果没有找到则会插入新的 `Entry` ，插入之后可能破坏了红黑树的约束条件，还需要调用 `fixAfterInsertion(Entry<K,V> x)` 进行调整（颜色调换和树旋转）。

```java
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;
    while (x != null && x != root && x.parent.color == RED) { // 父节点为红色，需要调整
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);          // 情形 1
                setColor(y, BLACK);                    // 情形 1
                setColor(parentOf(parentOf(x)), RED);  // 情形 1
                x = parentOf(parentOf(x));             // 情形 1
            } else {
                if (x == rightOf(parentOf(x))) {       // 情形 2
                    x = parentOf(x);                   // 情形 2
                    rotateLeft(x);                     // 情形 2
                }
                setColor(parentOf(x), BLACK);          // 情形 3
                setColor(parentOf(parentOf(x)), RED);  // 情形 3
                rotateRight(parentOf(parentOf(x)));    // 情形 3
            }
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);          // 情形 1
                setColor(y, BLACK);                    // 情形 1
                setColor(parentOf(parentOf(x)), RED);  // 情形 1
                x = parentOf(parentOf(x));             // 情形 1
            } else {
                if (x == leftOf(parentOf(x))) {        // 情形 4（情形 2 的镜像）
                    x = parentOf(x);                   // 情形 4（情形 2 的镜像）
                    rotateRight(x);                    // 情形 4（情形 2 的镜像）
                }
                setColor(parentOf(x), BLACK);          // 情形 5（情形 3 的镜像）
                setColor(parentOf(parentOf(x)), RED);  // 情形 5（情形 3 的镜像）
                rotateLeft(parentOf(parentOf(x)));     // 情形 5（情形 3 的镜像）
            }
        }
    }
    root.color = BLACK;
}
```
`fixAfterInsertion(Entry<K,V> x)` 方法首先将新插入的节点标记为红色。如果设为黑色，就会导致根节点到叶子节点的所有路径中的某一条路上多一个额外的黑色节点，这个是很难调整的。但是设为红色节点后，可能会导致出现两个连续红色节点的冲突，那么可以通过颜色调换和树旋转来调整。接下来要进行什么操作取决于其他临近节点的颜色。注意，在根节点存在的情况下：

> - 性质 1 和性质 3 总是保持着。
> 
> - 性质 4 只在增加红色节点、重绘黑色节点为红色，或做旋转时受到威胁。
> 
> - 性质 5 只在增加黑色节点、重绘红色节点为黑色，或做旋转时受到威胁。

如果新节点的父节点是黑色，则性质 4 依然有效（新节点是红色的）。性质 5 也未受到威胁，尽管新节点有两个黑色叶子子节点，但由于新节点是红色，通过它的每个子节点的路径就都有同通过它所取代的黑色的叶子的路径同样数目的黑色节点，所以依然满足性质 5 。在这种情形下，树仍是有效的，无需调整。

如果新节点的父节点是红色，则破坏了红黑树的性质 4 。此时新节点一定有祖父节点。因为如果父节点是根节点，那父节点就应当是黑色。所以新节点总有一个叔父节点，尽管叔父节点可能是叶子节点。

#### 情形 1：

<img src="https://s25.postimg.org/q69q3he27/JCF05-TreeMap-04-insert-case1.png" alt="插入后调整的情形 1" align="right"/>

新插入节点为 N ，如果父节点 P 和叔父节点 U 二者都是红色，则可以将它们两个重绘为黑色，并将祖父节点 G 重绘为红色。现在新节点 N 有了一个黑色的父节点 P 。因为通过父节点 P 或叔父节点 U 的任何路径都必定通过祖父节点 G ，在这些路径上的黑色节点数目没有改变，性质 5 得到了维持。但是，红色祖父节点 G 可能是根节点，这违反了性质 2 ；也有可能祖父节点 G 的父节点是红色，这违反了性质 4 。为了解决这个问题，需要在祖父节点 G 上递归地进行整个调整过程，即把祖父节点 G 当作新节点 N 进行各种情形的检查。

#### 情形 2：

<img src="https://s25.postimg.org/5anfs8hv3/JCF05-TreeMap-05-insert-case2.png" alt="插入后调整的情形 2" align="right"/>

父节点 P 是红色而叔父节点 U 是黑色或缺少，新节点 N 是其父节点 P 的右子节点，父节点 P 是祖父节点 G 的左子节点。在这种情形下，需要对父节点 P 进行一次左旋转，调换新节点 N 和其父节点 P 的角色；接着，按情形 5 处理原来的父节点 P 以解决仍然失效的性质 4 。注意这个改变会导致某些路径通过它们以前不通过的新节点 N（如 1 号叶子节点）或不通过节点 P（如 3 号叶子节点），但由于这两个节点都是红色的，所以性质 5 仍有效。如果新节点 N 的父节点 P 是祖父节点 G 的右子节点，则将情形 2 中的左右对调得到情形 4 。

#### 情形 3：

<img src="https://s25.postimg.org/rnzrss86n/JCF05-TreeMap-06-insert-case3.png" alt="插入后调整的情形 3" align="right"/>

父节点 P 是红色而叔父节点 U 是黑色或缺少，新节点 N 是其父节点 P 的左子节点，父节点 P 是祖父节点 G 的左子节点。在这种情形下，需要对祖父节点 G 进行一次右旋转，在旋转产生的树中，原来的父节点 P 变成了新节点 N 和原来的祖父节点 G 的父节点。原来的祖父节点 G 一定是黑色，否则父节点 P 就不可能是红色（如果 P 和 G 都是红色就违反了性质 4）。调换原来的父节点 P 和祖父节点 G 的颜色，生成的树满足性质 4 。性质 5 也仍然保持满足，因为通过这三个节点中任何一个的所有路径以前都通过原来的祖父节点 G ，现在它们都通过原来的父节点 P 。在各自的情形下，原来的父节点 P 都是三个节点中唯一的黑色节点。如果新节点 N 的父节点 P 是祖父节点 G 的右子节点，则将情形 3 中的左右对调得到情形 5 。

<h1 id="6">remove 方法</h1>

