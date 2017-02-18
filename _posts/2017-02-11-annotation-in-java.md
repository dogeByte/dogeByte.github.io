---
layout: post
title:  "注解（annotation）详解"
date:   2017-02-11 10:05:23
categories: Java
tags: Java
---

# 目录
1. [Java中的注解](#1)
    1. [标准注解](#1_1)
    2. [元注解](#1_2)
2. [自定义注解](#2)
3. [使用自定义注解](#3)
4. [注解处理器](#4)

注解是为了在代码中添加信息的一种方式，可以使我们在之后的某个时刻获取并使用这些数据。

<h1 id="1">Java中的注解</h1>

Java目前内置了三种标准注解和四种元注解。

<h3 id="1_1">标准注解</h3>

标准注解就是说可以直接在代码中使用的，并且有约定好含义的注解。定义在Java中的三种元注解如下：

> @Override，表示当前的方法定义将覆盖父类中的方法。如果你不小心拼写错误，或者方法参数不对，那么编译器就会发出错误提示。
> 
> @Deprecated，表示该方法是过时的，不建议使用。如果你坚持要使用的话，编译器会发出警告信息。
> 
> @SuppressWarnings，编译器不会对使用该注解的方法发出任何警告信息。

<h3 id="1_2">元注解</h3>

Java中的元注解有4个。元注解是用来定义、生成其他注解的：

@Target

表示该注解可以用在什么地方。可能的ElementType参数包括：

> TYPE：类，接口（包括注解类）或枚举类声明。
> 
> FIELD：域声明（包括枚举对象）
> 
> METHOD：方法声明
> 
> PARAMETER：参数声明
> 
> CONSTRUCTOR：构造器声明
> 
> LOCAL_VARIABLE：局部变量声明
> 
> ANNOTATION_TYPE：注解类型声明
> 
> PACKAGE：包声明

@Retention

表示需要在什么级别保存该注解信息。可选的Retention参数包括：

> SOURCE：注解信息在编译阶段就会被丢弃
> 
> CLASS：注解信息保留在class文件中，但是会被VM丢弃。
> 
> RUNTIME：VM在运行期也会保留注解，因此可以通过反射机制读取注解信息。

@Documented

将此注解包含在Javadoc中。

@Inherited

允许子类继承父类中的注解。

<h1 id="2">自定义注解</h1>

既然元注解是生成注解的注解，那么元注解又是怎么生成的呢？当然还是用元注解！我们来看一下元注解`@Documented`的源码：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Documented {
}
```

可以看到，这里使用了三个元注解来生成了`@Documented`这个元注解。除了@符号外，定义一个注解很像定义一个空接口。一般来说，注解中都会包含一些元素来表示某些值，这样在分析处理注解时，就能使用这些值，但是也有像`@Documented`这种没有元素的注解，称之为标记注解。

下面我们定义一个有元素的注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    public int id();
    public String description() default "no description";
}
```
注解`@UseCase`包含int元素id，以及一个String元素description，注解元素可用类型如下所示：

> 基本数据类型
> 
> String
> 
> Class
> 
> enum
> 
> Annotation
> 
> 以上类型的数组

注意到元素description含有一个默认值"no description"，一般来说，对于非基类型的元素，无论是在源码中声明，还是在注解接口中定义默认值时，都不能用null作为期默认值。一般用一些特殊的值表示其不存在，比如空字符串或负数。比如：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SimulatingNull {
    public int id() default -1;
    public String description() default "";
}
```

<h1 id="3">使用自定义注解</h1>

下面，我们就看看如何在实际的代码中使用`@UseCase`注解：

```java
public class PasswordUtils {
    
    @UseCase(id = 47, description = "Password must contain at least one numeric")
    public boolean validatePassword(String password) {
        return password.matches("\\w*\\d\\w*");
    }
    
    @UseCase(id = 49)
    public String encryptPassword(String password) {
        return new StringBuilder(password).reverse().toString();
    }
    
    @UseCase(id = 49, description = "New passwords, can't equal previously used ones")
    public boolean checkForNewPassword(List<String> prevPasswords, String password) {
        return !prevPasswords.contains(password);
    }
}
```

注解的元素在使用时表现为键值对的形式，并且需要置于`@UseCase`声明之后的括号内。

<h1 id="4">注解处理器</h1>

```java
public class UseCaseTracker {
    public static void trackUseCases(List<Integer> useCases, Class<?> cl) {
        for (Method m : cl.getDeclaredMethods()) {
            UseCase uc = m.getAnnotation(UseCase.class);
            if (uc != null) {
                System.out.println("Found Use Case: " + uc.id() + " " + uc.description());
                useCases.remove(new Integer(uc.id()));
            }
        }
        for (int i : useCases) {
            System.out.println("Warning: Missing use case - " + i);
        }
    }
    public static void main(String[] args) {
        List<Integer> useCase = new ArrayList<Integer>();
        Collections.addAll(useCase, 47, 48, 49, 50);
        trackUseCases(useCase, PasswordUtils.class);
    }
}
```

运行结果如下：

```java
Found Use Case: 49 no description
Found Use Case: 47 Password must contain at least one numeric
Found Use Case: 49 New passwords, can't equal previously used ones
Warning: Missing use case - 48
Warning: Missing use case - 50
```

这个程序用到了两个反射的方法：`getDeclaredMethods()`和`getAnnotation()`，它们都属于`AnnotatedElement`接口（Class, Method与Field等类都实现了该接口）。`getAnnotation()`方法返回指定类型的注解对象，在这里就是UseCase。如果被注解的方法上没有该类型的注解，则返回null值。然后我们通过调用`id()`和`description()`方法从返回的UseCase对象中提取元素的值。其中，`encriptPassword()`方法在注解的时候没有指定description的值，因此通过`description()`方法取得的是默认值no description。