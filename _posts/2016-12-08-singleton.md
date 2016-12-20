---
layout: post
title:  "单例模式的几种写法"
date:   2016-12-08 22:38:27
categories: Java
tags: Java
---

# 目录
1. [饿汉式](#1)
2. [懒汉式](#2)
3. [静态内部类](#3)
4. [枚举](#4)

<h1 id="1">饿汉式</h1>

单例模式应该是最简单的一种设计模式，最简单的饿汉式的写法如下：

```java
public class Singleton {
    private Singleton() {}
    private static final Singleton INSTANCE = new Singleton();
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

饿汉式非常简单，因为单例的实例被声明成 static 和 final 变量了，在第一次加载类到内存中时就会初始化，所以创建实例本身是线程安全的。

饿汉式的缺点在于它不是一种懒加载模式，单例会在类加载完成后立即被初始化，即使没有调用  getInstance() 方法。饿汉式的创建方式在一些场景中将无法使用，例如 Singleton 实例的创建是依赖参数或者配置文件的，在调用 getInstance() 方法之前必须调用某个方法设置参数给它，那样这种单例写法就无法使用了。

<h1 id="2">懒汉式</h1>

为了实现懒加载，可以使用懒汉式创建单例：

```java
public class Singleton {
    private Singleton () {}
    private static Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

这段代码简单明了，而且使用了懒加载模式，但是却存在致命的问题：当有多个线程并行调用 getInstance() 方法的时候，就会创建多个实例。最简单解决办法是将整个 getInstance() 方法设为同步（synchronized）：

```java
public class Singleton {
    private Singleton () {}
    private static Singleton instance;
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

这样虽然做到了线程安全，并且解决了多个实例的问题，但是它并不高效。在任何时候都只能有一个线程调用 getInstance() 方法，但是只有第一次创建实例时才需要同步，这时可以使用双重检验锁。

```java
public class Singleton {
    private Singleton () {}
    private static Singleton instance;
    public static Singleton getSingleton() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

这段代码的不完美之处在于 `instance = new Singleton()` 并非是一个原子操作，事实上在执行这句代码时， JVM 做了以下三件事：

1. 为 instance 分配内存；
2. 调用 Singleton 的构造函数来初始化成员变量；
3. 将 instance 对象指向分配的内存空间（此时 instance != null）

但是在 JVM 的即时编译器中存在指令重排序的优化，即上面的第二步和第三步的顺序是不能保证的，最终的执行顺序可能是 1-2-3 也可能是 1-3-2。如果是后者，则在 3 执行完毕且 2 未执行之前，CPU被线程二抢占，这时 instance 已经是非 null，所以线程二会直接返回尚未初始化的 instance，调用它则会报错。

此时只需要将变量 instance 声明为 volatile 就可以了。

```java
public class Singleton {
    private Singleton () {}
    private static volatile Singleton instance;
    public static Singleton getSingleton() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

使用 volatile 的主要原因是禁止指令重排序，在 volatile 变量的赋值操作后面会有一个内存屏障，读操作不会被重排序到内存屏障之前。此时，取操作必须在执行完 1-2-3 之后或者 1-3-2 之后，不存在执行到 1-3 然后取到值的情况。

<h1 id="3">静态内部类</h1>

```java
public class Singleton {
    private Singleton () {}
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```

这种写法仍然使用 JVM 本身机制保证了线程安全问题；由于 SingletonHolder 是私有的，除了 getInstance() 之外没有办法访问它，因此它也可以实现懒加载；同时读取实例的时候不会进行同步，没有性能缺陷。

<h1 id="4">枚举</h1>

```java
public enum Singleton {
    INSTANCE;
}
```

这时可以通过 Singleton.INSTANCE 来访问实例，创建枚举默认是线程安全的，而且还能防止反序列化导致重新创建新的对象。