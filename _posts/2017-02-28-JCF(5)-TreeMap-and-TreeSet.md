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
	3. [颜色重绘和树旋转](#2_3)
3. [构造方法](#3)
4. [get 方法](#4)
5. [put 方法](#5)
6. [remove 方法](#6)
7. [TreeSet](#7)

`TreeMap` 和 `TreeSet` 在 Java 里有着相同的实现，后者仅仅是对前者做了一层包装，也就是说 `TreeSet` 内部维护着一个 `TreeMap`。

<h1 id="1">TreeMap 概述</h1>

`TreeMap` 实现了 `SortedMap` 接口，该映射根据其键（`key`）的自然顺序进行排序，或者根据创建映射时提供的 `Comparator` 进行排序，具体取决于使用的构造方法。

`TreeMap` 底层通过红黑树实现，此实现为 `containsKey` 、 `get` 、 `put` 和 `remove` 操作提供受保证的 log(n) 时间开销。`TreeMap` 中使用内部类 `Entry` 表示红黑树的一个节点：

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

同样，`TreeMap` 不是同步的。如果多个线程同时访问一个映射，并且其中至少一个线程从结构上修改了该映射，则其必须外部同步。（结构上的修改是指添加或删除一个或多个映射关系的操作；仅改变与现有键关联的值不是结构上的修改。）这一般是通过对自然封装该映射的对象执行同步操作来完成的。如果不存在这样的对象，则应该使用 `Collections.synchronizedSortedMap` 方法来“包装”该映射。最好在创建时完成这一操作，以防止对映射进行意外的不同步访问：

```java
SortedMap map = Collections.synchronizedSortedMap(new TreeMap(...));
```

<h1 id="2">红黑树</h1>

<h3 id="2_1">约束条件</h3>

红黑树是每个节点都带有颜色属性的二叉查找树，颜色为红色或黑色。

![红黑树](https://s25.postimg.org/b0hghrs6n/JCF05-TreeMap-01-red-black-tree.png)

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
    } else {                        // 右子树为空
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

<h3 id="2_3">颜色重绘和树旋转</h3>

因为每一个红黑树也是一个特化的二叉查找树，因此红黑树上的只读操作与普通二叉查找树上的只读操作相同。然而，在红黑树上进行插入操作和删除操作会导致不再匹配红黑树的性质。恢复红黑树的性质需要少量 O(log n) 的颜色重绘（实际是非常快速的）和不超过三次树旋转（对于插入操作是两次）。虽然插入和删除很复杂，但操作时间仍可以保持为 O(log n) 次。

左旋转的过程是将 x 的右子节点绕 x 逆时针旋转，使得 x 的右子节点成为 x 的父节点，同时修改相关节点的引用。树旋转之后，二叉查找树的属性仍然满足。

![左旋转](https://s25.postimg.org/rpiwdoorz/JCF05-TreeMap-02-rotate-left.png)

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

![右旋转](https://s25.postimg.org/4pc91cqy7/JCF05-TreeMap-03-rotate-right.png)

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
        return getEntryUsingComparator(key);                // 使用指定的比较器顺序
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

`get(Object key)` 方法根据指定的 `key` 值返回对应的 `value`，该方法调用了 `getEntry(Object key)` 得到相应的 `Entry`，然后返回 `entry.value`。`getEntry(Object key)` 的逻辑是根据 `key` 的自然顺序（或者比较器顺序）对二叉查找树进行查找，直到找到满足`k.compareTo(p.key) == 0` 的 `Entry`。

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
    if (cpr != null) {                      // 使用指定的比较器顺序
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0) {
                t = t.left;                 // 向左找
            } else if (cmp > 0) {
                t = t.right;                // 向右找
            } else {
                return t.setValue(value);
            }
        } while (t != null);
    }
    else {                                  // 使用 key 的自然顺序
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0) {
                t = t.left;                 // 向左找
            } else if (cmp > 0) {
                t = t.right;                // 向右找
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

`put(K key, V value)` 方法首先会查看根节点是否为空，如果为空则将新节点置于根节点上。由于 `Entry` 中 `color` 字段默认为 `BLACK`，此时满足性质 2 和性质 5。

如果根节点不为空，则继续查找 `TreeMap` 中是否包含该 `key`，查找过程类似于`getEntry(Object key)`。如果已经包含则替换旧值并返回；如果没有找到则会插入新的 `Entry`，插入之后可能破坏了红黑树的约束条件，还需要调用 `fixAfterInsertion(Entry<K,V> x)` 进行调整（颜色重绘和树旋转）。

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
`fixAfterInsertion(Entry<K,V> x)` 方法首先将新插入的节点标记为红色。如果设为黑色，就会导致根节点到叶子节点的所有路径中的某一条路上多一个额外的黑色节点，这个是很难调整的。但是设为红色节点后，可能会导致出现两个连续红色节点的冲突，那么可以通过颜色重绘和树旋转来调整。接下来要进行什么操作取决于其他临近节点的颜色。注意，在根节点存在的情况下：

> - 性质 1 和性质 3 总是保持着。
> 
> - 性质 4 只在增加红色节点、重绘黑色节点为红色，或做旋转时受到威胁。
> 
> - 性质 5 只在增加黑色节点、重绘红色节点为黑色，或做旋转时受到威胁。

如果新节点的父节点是黑色，则性质 4 依然有效（新节点是红色的）。性质 5 也未受到威胁，尽管新节点有两个黑色叶子子节点，但由于新节点是红色，通过它的每个子节点的路径就都有同通过它所取代的黑色的叶子的路径同样数目的黑色节点，所以依然满足性质 5。在这种情形下，树仍是有效的，无需调整。

如果新节点的父节点是红色，则破坏了红黑树的性质 4。此时新节点一定有祖父节点。因为如果父节点是根节点，那父节点就应当是黑色。所以新节点总有一个叔父节点，尽管叔父节点可能是叶子节点。

以下的示意图中，N 代表 `fixAfterInsertion(Entry<K,V> x)` 方法中被调整的节点 `x`，P 代表 N 的父节点，U 代表 N 的叔父节点。

情形 2 和情形 3 中，均假定节点 N 的父节点 P 是其祖父节点 G 的左子节点。节点 N 的父节点 P 是其祖父节点 G 的右子节点，则将情形 2 和情形 3 中的左右调换，得到情形 4 和情形 5。

#### 情形 1：

<img src="https://s25.postimg.org/q69q3he27/JCF05-TreeMap-04-insert-case1.png" alt="插入后调整的情形 1" align="right"/>

节点 N 的父节点 P 和叔父节点 U 二者都是红色，在这种情形下，可以将父节点 P 和叔父节点 U 重绘为黑色，并将祖父节点 G 重绘为红色。现在新节点 N 有了一个黑色的父节点 P。因为通过父节点 P 或叔父节点 U 的任何路径都必定通过祖父节点 G，在这些路径上的黑色节点数目没有改变，性质 5 得到了维持。但是，红色祖父节点 G 可能是根节点，这违反了性质 2 ；也有可能祖父节点 G 的父节点是红色，这违反了性质 4。为了解决这个问题，需要在祖父节点 G 上递归地进行整个调整过程，即把祖父节点 G 当作新节点 N 进行各种情形的检查。

#### 情形 2：

<img src="https://s25.postimg.org/5anfs8hv3/JCF05-TreeMap-05-insert-case2.png" alt="插入后调整的情形 2" align="right"/>

节点 N 的父节点 P 是其祖父节点 G 的左子节点，父节点 P 是红色而叔父节点 U 是黑色或缺少，节点 N 是其父节点 P 的右子节点。在这种情形下，需要对父节点 P 进行一次左旋转，调换新节点 N 和其父节点 P 的角色；接着，按情形 3 处理原来的父节点 P 以解决仍然失效的性质 4。注意这个改变会导致某些路径通过它们以前不通过的新节点 N（如 1 号叶子节点）或不通过节点 P（如 3 号叶子节点），但由于这两个节点都是红色的，所以性质 5 仍有效。如果新节点 N 的父节点 P 是祖父节点 G 的右子节点，则将情形 2 中的左右对调得到情形 4。

#### 情形 3：

<img src="https://s25.postimg.org/rnzrss86n/JCF05-TreeMap-06-insert-case3.png" alt="插入后调整的情形 3" align="right"/>

节点 N 的父节点 P 是其祖父节点 G 的左子节点，父节点 P 是红色而叔父节点 U 是黑色或缺少，节点 N 是其父节点 P 的左子节点。在这种情形下，需要对祖父节点 G 进行一次右旋转，在旋转产生的树中，原来的父节点 P 变成了新节点 N 和原来的祖父节点 G 的父节点。原来的祖父节点 G 一定是黑色，否则父节点 P 就不可能是红色（如果 P 和 G 都是红色就违反了性质 4）。调换原来的父节点 P 和祖父节点 G 的颜色，生成的树满足性质 4。性质 5 也仍然保持满足，因为通过这三个节点中任何一个的所有路径以前都通过原来的祖父节点 G，现在它们都通过原来的父节点 P。在各自的情形下，原来的父节点 P 都是三个节点中唯一的黑色节点。如果新节点 N 的父节点 P 是祖父节点 G 的右子节点，则将情形 3 中的左右对调得到情形 5。

<h1 id="6">remove 方法</h1>

> remove(Object key) 如果此 TreeMap 中存在该键的映射关系，则将其删除。

```java
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null) {
        return null;
    }
    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}
```

`remove(Object key)` 方法首先调用 `getEntry(Object key)` 方法找到 `key` 值对应的 `entry`，然后调用 `deleteEntry(Entry<K,V> entry)` 方法删除对应的 `entry`。

```java
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;
    if (p.left != null && p.right != null) {  // 被删除节点的两个子节点都非空
        Entry<K,V> s = successor(p);  
        p.key = s.key;                        // 把后继节点的值转移给被删除节点
        p.value = s.value;
        p = s;                                // 需要删除的节点变为它的后继节点
    }
    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);
    if (replacement != null) {                // 被删除节点只有一个非空子节点
        // Link replacement to parent
        replacement.parent = p.parent;
        if (p.parent == null) {
            root = replacement;
        } else if (p == p.parent.left) {
            p.parent.left  = replacement;
        } else {
            p.parent.right = replacement;
        }
        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;
        // Fix replacement
        if (p.color == BLACK) {               // 被删除节点为黑色，需要调整树结构
            fixAfterDeletion(replacement);
        }
    } else if (p.parent == null) {            // 整棵树只有一个节点
        root = null;
    } else {                                  // 被删除节点的两个子节点都为空
        if (p.color == BLACK) {               // 被删除节点为黑色，需要调整树结构
            fixAfterDeletion(p);
        }
        if (p.parent != null) {
            if (p == p.parent.left) {
                p.parent.left = null;
            } else if (p == p.parent.right) {
                p.parent.right = null;
            }
            p.parent = null;
        }
    }
}
```

如果被删除节点 p 有两个非空子节点，那么问题可以被转化为删除另一个最多只有一个非空子节点的节点。对于二叉查找树，在删除有两个非空子节点的节点的时候，需要找到它的后继节点 s，并把后继节点 s 的值转移到被删除节点 p 中，然后删除后继节点 s。由于被删除节点 p 的两个子节点都非空，所以后继节点 s 只可能是节点 p 的右子树中的最小元素，这意味这后继节点 s 的左子节点一定为空。因为只是复制了一个值，不违反任何性质，这就把问题简化为如何删除最多只有一个非空子节点的节点。

余下的部分中，只需要讨论删除最多只有一个非空子节点的节点的情况，而无须关心这个节点是最初要删除的节点 p 还是它的后继节点 s（将 p 指向 s）。

如果被删除节点 p 只有一个非空子节点 replacement，则用这个非空子节点 replacement 替代被删除节点 p，并将 p 的父子引用全部清空；如果被删除节点 p 的两个子节点都为空，则直接清空相关引用。

注意，只有当被删除节点 p 是黑色的时候，才会触发调整方法 `fixAfterDeletion(Entry<K,V> x)`。因为如果删除一个红色节点，它的父节点和子节点一定都是黑色的，所以可以简单地用它的黑色子节点替换它，这并不会破坏性质 3 和性质 4。而通过被删除节点的所有路径只是少了一个红色节点，这样可以继续保证性质 5。

```java
private void fixAfterDeletion(Entry<K,V> x) {
    while (x != root && colorOf(x) == BLACK) {
        if (x == leftOf(parentOf(x))) {
            Entry<K,V> sib = rightOf(parentOf(x));
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);                    // 情形 1
                setColor(parentOf(x), RED);              // 情形 1
                rotateLeft(parentOf(x));                 // 情形 1
                sib = rightOf(parentOf(x));              // 情形 1
            }
            if (colorOf(leftOf(sib)) == BLACK && colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);                      // 情形 2
                x = parentOf(x);                         // 情形 2
            } else {
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);        // 情形 3
                    setColor(sib, RED);                  // 情形 3
                    rotateRight(sib);                    // 情形 3
                    sib = rightOf(parentOf(x));          // 情形 3
                }
                setColor(sib, colorOf(parentOf(x)));     // 情形 4
                setColor(parentOf(x), BLACK);            // 情形 4
                setColor(rightOf(sib), BLACK);           // 情形 4
                rotateLeft(parentOf(x));                 // 情形 4
                x = root;                                // 情形 4
            }
        } else { // symmetric
            Entry<K,V> sib = leftOf(parentOf(x));
            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);                    // 情形 5（情形 1 的镜像）
                setColor(parentOf(x), RED);              // 情形 5（情形 1 的镜像）
                rotateRight(parentOf(x));                // 情形 5（情形 1 的镜像）
                sib = leftOf(parentOf(x));               // 情形 5（情形 1 的镜像）
            }
            if (colorOf(rightOf(sib)) == BLACK && colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);                      // 情形 6（情形 2 的镜像）
                x = parentOf(x);                         // 情形 6（情形 2 的镜像）
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);       // 情形 7（情形 3 的镜像）
                    setColor(sib, RED);                  // 情形 7（情形 3 的镜像）
                    rotateLeft(sib);                     // 情形 7（情形 3 的镜像）
                    sib = leftOf(parentOf(x));           // 情形 7（情形 3 的镜像）
                }
                setColor(sib, colorOf(parentOf(x)));     // 情形 8（情形 4 的镜像）
                setColor(parentOf(x), BLACK);            // 情形 8（情形 4 的镜像）
                setColor(leftOf(sib), BLACK);            // 情形 8（情形 4 的镜像）
                rotateRight(parentOf(x));                // 情形 8（情形 4 的镜像）
                x = root;                                // 情形 8（情形 4 的镜像）
            }
        }
    }
    setColor(x, BLACK);                                  // 情形 0
}
```

情形 0 是一种简单的情况，被删除节点是黑色，只有一个非空子节点且为红色。如果只是去除这个黑色节点，并用它的红色子节点代替的话，会破坏性质 5 ；但是如果将红色子节点重绘为黑色，则原来通过它的所有路径将通过它的黑色子节点，这样可以继续保持性质 5。

以下的示意图中，N 代表 `fixAfterDeletion` 方法中被调整的节点 `x`，P 代表 N 的父节点，S（sib）代表 N 的兄弟节点，SL 和 SR 分别代表 S 的左右子节点。

情形 1 ~ 4 中，均假定 N 是它的父节点 P 的左子节点。如果 N 是 P 的右子节点，则将情形 1 ~ 4 中的左右调换，得到情形 5 ~ 8。

#### 情形 1：

<img src="https://s25.postimg.org/7xgqeed7z/JCF05-TreeMap-07-delete-case1.png" alt="删除后调整的情形 1"  align="right"/>

N 是它的父节点 P 的左子节点，且兄弟节点 S 是红色。在这种情形下，需要调换父节点 P 和兄弟节点 S 的颜色，然后在父节点 P 上做左旋转，把红色兄弟节点 S 转换为 N 的祖父节点。此时，尽管所有路径上黑色节点的数目没有改变，但是 N 有了一个黑色的兄弟节点 SL（它的新兄弟 SL 是黑色，因为 SL 曾经是红色节点 S 的一个子节点）和一个红色的父节点 P，所以接下来需要按情形 2 、情形 3 或情形 4 来处理。如果 N 是它的父节点 P 的右子节点，则将情形 1 中的左右对调得到情形 5。

#### 情形 2：

<img src="https://s25.postimg.org/kq4uebotr/JCF05-TreeMap-08-delete-case2.png" alt="删除后调整的情形 2"  align="right"/>

N 是它的父节点 P 的左子节点，N 的兄弟节点 S 、侄节点 SL 和 SR 都是黑色。在这种情形下，只需要简单地将兄弟节点 S 重绘为红色。此时，通过节点 S 的所有路径，即以前不通过 N 的那些路径，都少经过了一个黑色节点。由于在 `deleteEntry(Entry<K,V> p)` 方法中删除了黑色节点 N 的初始父节点 p（当 p 只有一个非空子节点时；如果 p 的子节点都为空，那么在调整前没有任何一个节点被删除，树结构不可能发生变化，这意味着 N 的父节点 P 只能为一定为红色，即不属于情形），使通过 N 的所有路径都少了一个黑色节点，这使得经过 N 和 S 的路径相对平衡了起来。但是，现在通过 P 的所有路径比不通过 P 的路径少了一个黑色节点，所以仍然违反性质 5。要修正这个问题，还需要从情形 1 开始，在 P 上做重新调整。如果 N 是它的父节点 P 的右子节点，则将情形 2 中的左右对调得到情形 6。

#### 情形 3：

<img src="https://s25.postimg.org/f38hgumb3/JCF05-TreeMap-09-delete-case3.png" alt="删除后调整的情形 3"  align="right"/>

N 是它的父节点 P 的左子节点，N 的兄弟节点 S 是黑色，S 的左子节点 SL 是红色，右子节点 SR 是黑色。在这种情形下，需要调换 S 和它的左子节点 SL 的颜色，然后在 S 上做右旋转，这样 S 的左子节点 SL 成为 S 的父节点和 N 的新兄弟节点。此时，所有路径上仍有同样数目的黑色节点，但是 N 有了一个黑色兄弟节点 SL，而 SL 的右子节点是红色，所以我们进入了情形 4。节点 N 和它的父节点 P 都不受这个变换的影响，所以在图中略去。如果 N 是它的父节点 P 的右子节点，则将情形 3 中的左右对调得到情形 7。

#### 情形 4：

<img src="https://s25.postimg.org/czy296mi7/JCF05-TreeMap-10-delete-case4.png" alt="删除后调整的情形 3"  align="right"/>

N 是它的父节点 P 的左子节点，N 的兄弟节点 S 是黑色，S 的右子节点 SR 是红色。在这种情形下，需要调换节点 P 和节点 S 的颜色，并将 S 的右子节点 SR 重绘为黑色。然后在 N 的父节点 P 上做左旋转，使 N 的兄弟节点 S 成为 N 的父节点。现在，通过 N 的所有路径都增加了一个黑色节点：要么是由于 N 的父节点 P 从红色变成黑色，要么是由于 P 原本就是黑色而 S 被增加为一个黑色祖父。此时，如果一个路径不通过 N，则有两种可能性：

1. 它通过 N 的新兄弟节点，那么它以前和现在都必定会经过节点 S 和节点 P，而它们只是调换了颜色，所以这些路径保持了同样数目的黑色节点。

2. 它通过 N 的新叔父节点，即 S 的右子节点 SR，那么它以前会经过节点 P 、节点 S 和它的右子节点 SR，但是现在只经过节点 S 和它的右子节点 SR。由于节点 S 的颜色和原来的节点 P 相同，并且节点 SR 的颜色从红色改变为黑色，因此调整后的效果是这些路径也经过了同样数目的黑色节点。

上述两种可能性中，这些路径上的黑色节点数目都没有改变，所以性质 5 得以恢复。如果 N 是它的父节点 P 的右子节点，则将情形 4 中的左右对调得到情形 8。

因此，`fixAfterDeletion(Entry<K,V> x)` 的总体思想是：首先将情形 1 转换成情形 2，或者转换成情形 3 和情形 4。当然，这并不意味着调整过程一定是从情形 1 开始。

> 1. 如果是由情形 1 之后进入的情形 2，那么情形 2 之后一定会退出循环（x 为红色）
> 
> 2. 一旦进入情形 3 和情形 4，一定会退出循环（x 为 root）

<h1 id="7">TreeSet</h1>

`TreeSet` 是对 `TreeMap` 的简单包装，对 `TreeSet` 的函数调用都会转换成合适的 `TreeMap` 中的方法，因此 `TreeSet` 的实现逻辑非常简单。

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable {
    private transient NavigableMap<E,Object> m;
    private static final Object PRESENT = new Object();
    public TreeSet() {
        this(new TreeMap<E,Object>());
    }
    public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }
    public boolean add(E e) {
        return m.put(e, PRESENT) == null;
    }
    public boolean remove(Object o) {
        return m.remove(o) == PRESENT;
    }
    public Iterator<E> iterator() {
        return m.navigableKeySet().iterator();
    }
}
```