---
title:  Java基础（一） - 基本程序设计结构
tags:
  - Java
categories:
  - Java
comments: true
date: 2020-11-9 20:00:00

---

<img src="https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201109221140.png" style="zoom: 75%;" />

<!--more-->

# Java基础（一） - 基本程序设计结构

## 1、Java里面的三种注释

Java里面的注释不会出现在可执行程序中，所以不必担心可执行代码会膨胀。不过在写代码时，为了代码优雅整洁，可读性强一些，也要注意代码的规范。Java中总共又3种标记注释的方式：

- 使用`//`，注释内容从`//`开始到本行末尾
- 使用`/* */`，注释界定符，可以将一段比较长的注释括起来
- 使用`/** */`，文档注释，使用JDK自带的javadoc工具，可以根据该注释自动生成文档

## 2、数据类型

Java是一种强类型语言，每一个变量都必须生命一种类型。在Java中，一共有8种基本类型，其中包括`4种整形、2种浮点类型、1种字符类型、1种boolean类型`：

|  类型   |      存储需求       |
| :-----: | :-----------------: |
|  byte   |        1字节        |
|  short  |        2字节        |
|   int   |        4字节        |
|  long   |        8字节        |
|  float  |        4字节        |
| double  |        8字节        |
|  char   | 2字节（和编码有关） |
| boolean |        1字节        |

需要注意的是，在表示long类型数值时，需要加一个后缀L或l，表示十六进制需要加前缀0x或0X，表示八进制需要加前缀0，从Java7开始，表示二进制可以加前缀0b或0B。在金融计算的时候为了精度要求，通常会选择使用BigDecimal大数类型数据。布尔类型和整数类型不能进行相互转换。

## 3、变量与常量

**声明变量**

Java中每一个变量都需要一个类型，在为变量命名时需注意符合Java命名规范。可以在一行中声明多个变量，如`int i, j;`。

**变量初始化**

使用未初始化的变量，程序则会报错。从Java10开始，对于局部变量，如果可以从变量的初始值推断出它的类型，就不需要声明类型。只需要使用关键字var而不许指定类型，如：

```java
var i = 10; //i is an int
var greeting = "Hello"; //greeting is a String
```

**常量**

被final修饰的变量就是常量，常量只能被赋值一次，一旦被赋值，就不能再改变了，命名时通常使用全大写。

如果希望在一个类的多个方法中使用，通常可以使用`static final`关键字设置一个类常量，且类常量通常设置在main方法外部。如果希望其他类的方法也可以使用这个常量，可以将常量的访问修饰符设置为`public`，如：

```java
public static final double CM_PER_INCH = 2.54;
```

**关于final修饰问题**

- 被final修饰的基本数据类型，如：byte,short, int,long等等，只能在声明或构造方法中复制，且赋值后不能改变
- 被final修饰的引用类型数据（不包括String和包装类型），地址不能改变，即这个引用只能引用这一个对象，但这个对象里面的值可以改变

## 4、运算符

**使用 +、-、\*、/、%、运算操作遵循规则**

只要两个操作数中有一个是double类型的，另一个将会被转换成double类型，并且结果也是double类型，如果两个操作数中有一个是float类型的，另一个将会被转换为float类型，并且结果也是float类型，如果两个操作数中有一个是long类型的，另一个将会被转换成long类型，并且结果也是long类型，否则（操作数为：byte、short、int 、char），两个数都会被转换成**int类型**，并且结果也是int类型。

```java
byte b1 = 1, b2 = 2, b3;
b3 = (b1 + b2) //error
```

**数学函数与常量**

在Math类中，包含了各种各样的数学函数，例如：

```java
Math.sqrt(); //求平方根
Math.pow(x, a); //求x的a次幂
```

**位运算符**

处理整数类型时，可以直接对整数各个位，使用位运算符，完成非常巧妙的操作，例如：

```java
& //与
| //或
! //非
^ //异或
>> 和 << //有符号右移，有符号左移
>>> //无符号右移，不存在无符号左移
```

## 5、字符串

**不可变字符串**

String类没有提供修改字符串中某个字符的方法。在Java文档中将String类对象称为`不可变的`，如字符串"hello"，只能永远包含h、e、l、l、o这些代码单元序列，不能修改这些值。不过却可以修改指向该字符串的指针，让指针指向另一个字符串。不可变字符串优点就是：编译器可以让字符串共享，提高效率。Java设计者认为共享带来的高效率远超过提取子串、拼接字符串所带来的低效率。

**String API**

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201109215244.png)

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201109215350.png)

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201109215442.png)

## 8、大数

如果基本的整数和浮点数不能满足需求，可以使用`BigInteger和BigDecimal`，这两个类可以实现任意精度的整数和浮点数运算。

**BigInteger**

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201109215516.png)

**BigDecimal**

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201109215551.png)

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201109215615.png)

## 9、数组

数组是一种数据结构，用来存储同一类型值的集合。声明数组：

```java
int[] a //只声明了变量a，并没有初始化
int[] a = new int[100]; //声明并初始化了一个可以存储100个整数的数组
int[] a = {1, 2, 3, 4, 5}; //简介形式
new int[]{1, 2, 3, 4, 5}; //匿名数组

//多维数组
double[][] balances;
balances = new double[NYEARS][NRATES];
int[][] magicSquare = {
    {1, 2, 3},
    {4, 5, 6},
    {7, 8, 9}
}
```

数组拷贝、排序：

```java
//copy
int[] copiedLuckyNumbers = Arrays.copyOf(oldLockyNumbers, newLength);

//sort 使用优化后的快速排序
int[] a = new int[100];
Arrays.sort(a);
```

