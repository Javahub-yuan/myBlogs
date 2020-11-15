---
title:  Java基础（六） - 异常
tags:
  - Java
categories:
  - Java
comments: true
date: 2020-11-15 14:30:00





---

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201115190152.png)

<!--more-->

# Java基础（六） - 异常

异常处理的任务就是：将控制器从产生错误的地方转移到能够处理这种情况的错误处理器中。

## 1、异常分类

Java中，所有异常都是继承Throwable而来的，紧接着在下一层会分解为两个分支：Error和Exception。

- Error表示Java运行时系统内部错误和资源耗尽错误，当发生Error错误时，程序只能停止运行。

- Exception表示异常，下一层又会细分为两个分支：一个是运行时异常（RuntimeException），另一个时非运行时异常（非RuntimeException）。
  - 运行时异常（RuntimeException）：**如果出现RuntimeException异常，那么就一定是你的问题**，该异常一般是由编程错误导致的，这些运行时错误完全处于控制之中，通过检查代码逻辑，从一开始就可以避免发生RuntimeException，并且代码中也不应该声明RuntimeException或时从RuntimeException继承的这些异常
  - 非运行时异常（非RuntimeException）：程序本身没有问题，但由于像I/O错误这类问题导致的异常，例如：试图打开一个不存在的文件、读取文件内容为空等等，因此遇到这种无法处理的情况时，可以抛出一个异常告诉编译器可能发生什么错误。

![异常分类](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201115112301.png)

Java语言规范又将派生于Error类或RuntimeException类所有的异常称为**非检查型（unchecked）异常**，其他所有的异常称为**检查型（checked）异常**。

## 2、声明检查型异常

方法不仅要告诉编译器返回什么类型，还需要告诉编译器有可能发生什么错误，如果要抛出多个检查型异常，则每个异常类之间使用逗号隔开。

```java
class MyAnimation {
    public void drawImage(int i) throws FileNotFoundException, EOFException {
        ...
    }
}
```

一个方法必须声明所有可能抛出的**检查型异常**，而**非检查型异常**要么在你控制之外（Error），要么从一开始就应该通过代码逻辑避免发生（RuntimeException）,遇到下面四种情况需要抛出检查型异常：

​	☛调用了一个抛出检查型异常的方法，例如FileInputStream构造器。

​	☛检测到一个错误，并且使用throw语句抛出一个检查型异常

​	☛程序出现错误，例如a[-1]=0会抛出一个非检查型异常（ArrayIndexOutOfBoundsException）

​	☛Java虚拟机或运行时库出现内部错误。

需要注意，如果超类方法没有抛出任何检查型异常，则子类也不能抛出任何检查型异常。如果一个方法声明抛出一个异常，则实际抛出的异常可能属于这个异常类，也可能是属于这个异常类的任意一个子类。

## 3、如何抛出异常

注意throw和throws的区别

```java
class MyAnimation {
    public String readData() throws EOFException {
        ...
        throw new EOFException();
    }
}
```

## 4、创建异常类

创建自定义异常，只需要定义一个派生于Exception的类，或者派生于Exception的某个子类，如IOException。习惯上自定义异常类，应该包含两个构造器，一个是默认无参构造器，另一个是包含详细描述信息的构造器

```java
class MyException extends IOException {
    public MyException() {}
    public MyException(String msg) {
        super(msg);
    }
}
```

## 5、捕获异常

异常处理分为两种方式：一种是积极处理，就是使用`try catch finally`处理，另一种是消极处理，使用`throws`继续抛出异常

## 6、该选择哪种异常处理方式？

如果调用一个抛出检查型异常的方法，就必须处理这个异常，或者继续向上传递，哪种方式更好呢？一般经验来说，如果你知道如果处理这些异常，则进行捕获处理，否则就抛出继续向上传递，由方法调用者或在其他合适的地方进行处理。

## 7、finally子句

```java
var in = new FileInputStream(...);
try {
    //1
    code that might throw exception
    //2
} catch (IOException e) {
    //3
    thow error message
    //4
} finally {
    //5
    in.close();
}
//6
```

在上面这段代码中，有下列3种情况会执行finally子句：

1. 代码没有抛出异常。在这种情况下，程序首先执行try语句块中的全部代码，然后执行finally子句中的代码。随后，继续执行try语句块之后的第一条语句。也就是说，执行标注的**1、2、5、6**处。

2. 抛出一个在catch子句中捕获的异常。在上面的示例中就是IOException异常。在这种情况下，程序将执行try语句块中的所有代码，直到发生异常为止。此时，将跳过try语句块中的剩余代码，转去执行与该异常匹配的catch子句中的代码，最后执行finally子句中的代码。
   - 如果catch子句没有抛出异常，程序将执行try语句块之后的第一条语句。在这里，执行标注**1、3、4、5、6**处的语句。
   - 如果catch子句抛出了一个异常，异常将被抛回这个方法的调用者。在这里，执行标注**1、3、5**处的语句。

3. 代码抛出了一个异常，但这个异常不是由catch子句捕获的。在这种情况下，程序将执行try语句块中的所有语句，直到有异常被抛出为止。此时，将跳过try语句块中的剩余代码，然后执行finally子句中的语句，并将异常抛给这个方法的调用者。在这里，执行标注**1、5**处的语句。

## 8、finally子句包含return语句

当finally子句包含return语句时，将会出现一种意想不到的结果„假设利用return语句从try语句块中退出。在方法返回前，finally子句的内容将被执行。如果finally子句中也有一个return语句，这个返回值将会覆盖原始的返回值。请看一个复杂的例子：

```java
//parseInt("42")返回的是0，而不是42
//parseInt("sss")返回0，而抛出的NumberFormatException异常被“吞掉”
public static int parseInt(String s)
{
    try {
        return Integer.parseInt(s);
    } finally {
        return 0;
    }
}
```

加入调用parseInt("42")，看起来像是try代码块种返回整数42，不过，在方法真正返回之前，会执行finally子句，这就使得方法最后会返回0，而忽略原先的返回值。

并且，如果调用parseInt("sss")，则`Integer.parseInt(s);`会抛出一个异常，然后会执行finally子句，return语句甚至会“**吞掉**”这个异常

**finally子句主要作用就是用于清理关闭资源。不要把改变控制流程的语句(return、throw、break、continue)放在finally子句中**

## 9、try-with-Resource语句

使用`try-with-Resource语句`的前提条件是：资源需要实现AutoCloseable接口的类。以下两段代码是等价的

```java
//open a resource
try {
    //work with the resource
} finally {
    //close the resource
}

------------------------------------
    
try (Resource res = ...) {
    //work with the resource
}
```

第二段代码中**当try代码块退出时，会自动调用close方法关闭资源，和try-finally本质一样的，是开发者提供的一个语法糖，当要关闭多个资源时，在圆括号里面使用分号分隔**

## 10、使用异常的技巧

1. 异常处理不能代替简单的测试
2. 不要过分的细化异常
3. 充分利用异常层次结构
4. 不要压制异常
5. 在检测错误时，“苛刻”要比放任更好
6. 不要羞于传递异常（早抛出，晚捕获）