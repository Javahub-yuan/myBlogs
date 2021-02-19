---
title:  Java基础（九） - 函数式编程
tags:
  - Java
categories:
  - Java
comments: true
date: 2021-2-19 20:00:00
---

- 函数式接口
- 如何使用lambda表达式实现函数式接口
- Java中几种内置的函数式接口的使用方法

<!--more-->

## 1 Java8函数式编程语法入门

使用Lambda表达式可以表示函数式接口的实现，在JAVA 8 之前一般是用匿名类实现的

```java
//函数式接口
public interface GreetingService {
    void sayMessage(String str);
}


//函数式编程(只针对函数式接口才可行，函数式接口(Functional Interface)就是有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。)
//使用Lambda表达式来表示该接口的一个实现(注：JAVA 8 之前一般是用匿名类实现的)：
GreetingService greetingService = str -> System.out.println(str + " hello!");
greetingService.sayMessage("poly");


//之前默认匿名类实现
GreetingService service = new GreetingService() {
    @Override
    public void sayMessage(String str) {
        System.out.println(str + " hello!");
    }
};
service.sayMessage("join");
```

- 输入：->前面的部分，即被()包围的部分。此处只有一个输入参数，实际上输入是可以有多个的，如两个参数时写法：(a, b);当然也可以没有输入，此时直接就可以是()。
- 函数体：->后面的部分，即被{}包围的部分；可以是一段代码。
- 输出：函数式编程可以没有返回值，也可以有返回值。如果有返回值时，需要代码段的最后一句通过return的方式返回对应的值。

## 2 Java函数式接口

Java中有**预先定义的函数式接口**，可以在编程时直接使用，比较方便

### 2.1 Consumer

Consumer是一个函数式编程接口； 顾名思义，Consumer的意思就是消费，即针对某个东西我们来使用它，因此它包含有一个有输入而无输出的accept接口方法；
除accept方法，它还包含有andThen这个方法；

- **andThen：**这个方法就是指定在调用当前Consumer后是否还要调用其它的Consumer；

```java
System.out.println("--------------Consumer:代表了接受一个输入参数并且无返回的操作--------------");
Consumer<String> c1 = x -> System.out.println("第一次执行:" + x);
Consumer<String> c2 = x -> System.out.println("第二次执行:" + x);

//先执行完c1，之后再执行c2
c1.andThen(c2).accept("test");
//连续多次执行c1
c1.andThen(c1).andThen(c1).accept("test1");
```

### 2.2 Function

Function也是一个函数式编程接口；它代表的含义是“函数”，而函数经常是有输入输出的，因此它含有一个apply方法，包含一个输入与一个输出；
除apply方法外，它还有compose与andThen及indentity三个方法，其使用见下述示例；

```java

System.out.println("--------------Function<T,R>:接受一个输入参数，返回一个结果--------------");
System.out.println("--------------apply、compose、andThen、indentity方法--------------");
Function<Integer, Integer> f = x -> x++;
Function<Integer, Integer> g = x -> x * 2;

/**
 * 下面表示在执行F时，先执行G，并且执行F时使用G的输出当作输入。
 * 相当于以下代码：
 * Integer a = g.apply(1);
 * System.out.println(f.apply(a));
 */
System.out.println(f.compose(g).apply(1));

/**
 * 表示执行F的Apply后使用其返回的值当作输入再执行G的Apply；
 * 相当于以下代码
 * Integer a = f.apply(1);
 * System.out.println(g.apply(a));
 */
System.out.println(f.andThen(g).apply(1));

/**
 * identity方法会返回一个不进行任何处理的Function，即输出与输入值相等；
 */
System.out.println(Function.identity().apply("aaa"));

```

### 2.3 Predicate

Predicate为函数式接口，predicate的中文意思是“断定”，即判断的意思，判断某个东西是否满足某种条件； 因此它包含test方法，根据输入值来做逻辑判断，其结果为True或者False。
它的使用方法示例如下：

```java
System.out.println("--------------Predicate<T> 接受一个输入参数，返回一个布尔值结果。--------------");
System.out.println("--------------包含test方法--------------");
Predicate<String> p1 = x -> x.equals("test");
Predicate<String> p2 = x -> x.startsWith("t");

/**
 * negate: 用于对原来的Predicate做取反处理；
 * 如当调用p.test("test")为True时，调用p.negate().test("test")就会是False；
 */
assertTrue(p1.test("test"));
assertFalse(p1.negate().test("test"));

/**
 * and: 针对同一输入值，多个Predicate均返回True时返回True，否则返回False；
 */
assertTrue(p1.and(p2).test("test"));
assertFalse(p1.and(p2).test("test111"));
assertFalse(p1.and(p2).test("hello"));

/**
 * or: 针对同一输入值，多个Predicate只要有一个返回True则返回True，否则返回False
 */
assertTrue(p1.or(p2).test("test"));
assertTrue(p1.or(p2).test("test111"));
assertFalse(p1.or(p2).test("hello"));
```



## 3 参考

1.[Java8新特性学习-函数式编程(Stream/Function/Optional/Consumer)](https://blog.csdn.net/icarusliu/article/details/79495534)

2.[Java 8 函数式接口](https://www.runoob.com/java/java8-functional-interfaces.html)

3.[Java 8 Optional 类](https://www.runoob.com/java/java8-optional-class.html)

4.[理解、学习与使用 JAVA 中的 OPTIONAL](https://www.cnblogs.com/zhangboyu/p/7580262.html)









