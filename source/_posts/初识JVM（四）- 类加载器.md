---
title:  初识JVM（四）- 类加载器
tags:
  - JVM
categories:
  - JVM
comments: true
date: 2020-11-8 10:00:00

---

<img src="https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/wallhaven-oxrr67.jpg" style="width:70%;" />

<!--more-->

# 类加载器

## 回顾一下类加载过程

一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，是他的整个生命周期，其中要经历七个阶段，分别是：**加载、验证、准备、解析、初始化、使用、卸载**，其中**验证、准备、解析三个部分统称为连接**

![类加载过程](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201107174246.png)

## 类与类加载器

在类加载中，类加载器**通过一个类的全限定名来获取描述该类的二进制字节流**，并且类加载器位于Java虚拟机外部，允许应用程序自己决定如何去获取所需要的类。类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远超类加载阶段。

**对于任意一个类，都必须由加载他的类加载器和这个类本身一起共同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间**。更通俗一些解释：比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则即使两个类来源于同一个Class文件，被同一个Java虚拟机加载，但是加载他们的类加载器不同，那这两个类就必定不相同。

比较两个类是否“相等”包括：`Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法、instanceof关键字`返回结果

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`：

1. **BootstrapClassLoader(启动类加载器)** ：最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者或被 `-Xbootclasspath`参数指定的路径中的所有类。因为是由C++实现，所以无法被Java程序直接引用，使用的时候可以直接使用null来代替。
2. **ExtensionClassLoader(扩展类加载器)** ：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。
3. **AppClassLoader(应用程序类加载器)** :也称为系统类加载器，面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。

## 双亲委派模型

双亲委派模型要求除了顶层的启动类加载器外，其余类加载器都应该有自己的父类加载器。不过，类加载器之间的父子关系一般不是通过继承实现的，而是用过使用组合关系来复用类加载器的代码。

双亲委派模型工作过程：每一个类都有一个对应它的类加载器。系统中的 ClassLoder 在协同工作的时候会默认使用 **双亲委派模型** 。即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。加载的时候，首先会把该请求委派该父类加载器的 `loadClass()` 处理，因此所有的请求最终都应该传送到顶层的启动类加载器 `BootstrapClassLoader` 中。当父类加载器无法处理时，才由自己来处理。当父类加载器为null时，会使用启动类加载器 `BootstrapClassLoader` 作为父类加载器。

![类加载器双亲委派模型](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201108100845.png)



##  双亲委派模型实现源码分析

双亲委派模型的实现代码非常简单，逻辑非常清晰，都集中在 `java.lang.ClassLoader` 的 `loadClass()` 中，相关代码如下所示。

```java
private final ClassLoader parent; 
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查请求的类是否已经被加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {//父加载器不为空，调用父加载器loadClass()方法处理
                        c = parent.loadClass(name, false);
                    } else {//父加载器为空，使用启动类加载器 BootstrapClassLoader 加载
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                   //抛出异常说明父类加载器无法完成加载请求
                }
                
                if (c == null) {
                    long t1 = System.nanoTime();
                    //自己尝试加载
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

##  双亲委派模型的好处

双亲委派模型保证了Java程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。如果没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现多个不同的 `Object` 类。

## 自定义加载器与破坏双亲委派模型

**自定义加载器的话，需要继承 `ClassLoader` 。如果我们不想打破双亲委派模型，就重写 `ClassLoader` 类中的 `findClass()` 方法即可，无法被父类加载器加载的类最终会通过这个方法被加载。但是，如果想打破双亲委派模型则需要重写 `loadClass()` 方法， `loadClass()`方法里面包含了双亲委派的具体逻辑**







