---
title:  Java基础（七） - 泛型程序设计
tags:
  - Java
categories:
  - Java
comments: true
date: 2020-11-16 20:30:00






---

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201117215145.png)

<!--more-->

# Java基础（七） - 泛型程序设计

## 1、为什么要使用泛型程序设计

### 使用泛型的好处？

使用泛型会让你的程序更易读，更加安全。在取数据的时候，不需要进行强制类型转换，编译器知道要返回的类型，在存数据的时候，编译器也会检查，防止插入错误类型的对象，出现编译错误要比运行时出现类的强制类型转换异常好得多。

## 2、定义简单泛型类

定义泛型类的要求就是：引入一个类型变量T，用尖括号(<>)括起来，放在类名的后面。如果有多个类型变量，则使用逗号分隔。在实例化的时候用具体类型替换类型变量。

> 一般，Java库使用变量E表示集合的元素类型，K和V分别表示键和值的类型，T（或U、S）表示“任意类型”

```java
//类型变量T
public class Pair<T> {
    private T first;
    private T second;
    
    public Pair() {}
    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }
}
```

## 3、泛型方法

定义泛型方法时，类型变量放置在修饰符（下面代码中修饰符是public static)后面，并在返回类型的前面。泛型方法可以在普通类中定义，也可以在泛型类中定义。

```java
class ArrayAlg {
    public static <T> T getMiddle(T... a) {
        return a[a.length / 2];
    }
}

ArrayAlg.<String>getMiddle("xixi", "qq");
//调用泛型方法时，可以省略<String>类型参数
ArrayAlg.getMiddle("xixi", "qq");
```

## 4、类型变量的限定

有时候需要限制传进来的泛型只能是实现了某个接口的类，这时候就需要使用`extends`关键字进行限定。语法：

```java
//T应该是限定类型的子类型
//T和限定类型可以是类，也可以是接口
<T extends BoundingType>
    
//一个类型变量或通配符可以有多个限定
<T extends Comparable & Serializable， U extends BoundingType>
```

**需要注意：**

- **一个类型变量或通配符可以有多个限定，限定类型用`&`分隔，类型变量用`逗号`隔开**
- **在Java继承中，多个限定中可以有多个接口，但最多只能有一个限定是类，且如果有一个类作为限定，那它必须是限定列表中的第一个限定**

## 5、泛型代码和虚拟机

### 类型擦除

无论何时定义一个泛型参数，都会自动提供一个相应的`原始类型`。这个原始类型的名字就是去掉类型参数后的泛型类型名。类型变量会被`擦除`，并替换为第一个限定类型（或者，对于无限定的变量则替换为Object）

```java
public class Pair<T> {
    private T first;
    private T second;
}
//Pair<T>的原始类型如下所示
public class Pair {
    private Object first;
    private Object second;
}
```

如果有多个限定，则原始类型用第一个限定来替换类型变量

```java
public class Pair<T extends Comparable & Serialable> {
    private T first;
    private T second;
}
//Pair<T extends Comparable & Serialable>的原始类型如下所示
public class Pair {
    private Comparable first;
    private Comparable second;
}
```

### 转换泛型表达式

编写一个泛型方法调用时，如果泛型擦除了返回类型，编译器会插入强制类型转换。例如：

```java
Pair<Employee> buddies = ...;
Employee buddy = buddies.getFirst();
```

getFirst擦除类型后返回类型是Object，则便一起会自动插入强制类型转换，将这个方法转换为两条虚拟机指令：

1. 对原始方法`Pair.getFirst`的调用
2. 将返回的Object类型强制转换为Employee类型

### 转换泛型方法

### 调用遗留代码

## 6、限制与局限性

Java泛型有一些限制，大多数限制都是由类型擦除引起的：

1. 不能用基本类型实例化类型参数
2. 运行时类型查询只适用于原始类型
3. 不能创建参数化类型的数组
4. Varargs警告
5. 不能实例化类型变量
6. 不能构造泛型数组
7. 泛型类的静态上下文中类型变量无效
8. 不能抛出或捕获泛型类的实例
9. 可以取消对检查型异常的检查
10. 注意擦除后的冲突

## 7、泛型类型的继承规则



## 8、通配符类型

**extends：**

子类型限定：例如通配符类型`Pair<? extends Employee>`表示泛型Pair类型，它的类型参数是Employee的**子类**，如`Pair<manager>`，而不是`Pair<String>`

**super：**

超类型限定：例如`<? super Manager>`，限制为Manager的所有超类型

## 9、反射和泛型

 

















