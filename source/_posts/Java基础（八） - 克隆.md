---
title:  Java基础（八） - 克隆
tags:
  - Java
categories:
  - Java
comments: true
date: 2020-11-17 20:30:00







---

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201117230300.png)

<!--more-->

# Java基础（八） - 克隆

## 1、回顾什么是拷贝

拷贝就是为对象引用变量建立副本，原变量和副本都是对同一个对象的引用，这说明，对任一个变量改变都会影响另外一个。

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201117224759.png)

## 2、Java克隆

克隆，即复制一个对象，该对象的属性与被复制的对象一致，如果不使用Object类中的clone方法实现克隆，可以自己new出一个对象，并对相应的属性进行数据，这样也能实现克隆的目的。

但当对象属性较多时，这样的克隆方式会比较麻烦，所以Object类中实现了clone方法，用于克隆对象。对于每一个类，需要确定：

1. 默认的clone方法是否满足需求
2. 是否可以在可变的对象上调用clone来修补默认的clone方法
3. 是否不该使用clone

实际上，第三个选项是默认选项。如果选择第一项或第二项，则类必须：

1. 实现Cloneable接口
2. 重新定义clone方法，并指定public访问修饰符

> Object的clone方法声明为protected，所以代码中不能直接调用clone方法，原因是因为Object类对你的对象一无所知，只能逐个字段的进行拷贝，如果对象中含有子对象的引用（对象类型的参数），则拷贝字段的引用与原引用将指向同一个对象，这样一来，原对象和克隆对象仍会共享一些信息

### 浅克隆

**浅克隆**：复制对象时仅仅复制对象本身，包括基本类型属性，但该对象的属性引用其他对象时（对象类型属性），该引用对象不会被复制，即拷贝出来的对象与被拷贝出来的对象中的属性引用的对象是同一个。

```java
class Employee implements Cloneable {
    private String name;
    private double salary;
    private Date hireDay;
    
    public Employee clone() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }
}
```

**默认克隆操作是“浅克隆”，并没有克隆对象中引用的其他对象（如图中String类型的name，Date类型的hireDay）**

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201117225934.png)

### 浅克隆会有什么影响吗？

这要看具体情况，如果原对象和克隆对象共享的子对象是**不可变的**，那么这种共享就是安全的。例如，子对象是一个不可变的String类，String对象是不可修改的对象，每次修改其实都是新建一个新的对象，而不是在原有的对象上修改，所以当修改String属性时其实是新开辟一个空间存储String对象，并把引用指向该内存，而克隆出来的对象的String属性还是指向原有的内存地址，所以String对象在浅克隆中也表现得与基本属性一样。

不过通常子对象都是可变的，为了保证安全，必须重新定义clone方法建立深克隆，同时克隆出所有子对象。

### 深克隆

深克隆：复制对象本身的同时，也复制对象包含的引用指向的对象，即修改被克隆对象的任何属性都不会影响到克隆出来的对象。

```java
class Employee implements Cloneable {
    private String name;
    private double salary;
    private Date hireDay;
    
    //重新定义clone方法建立深克隆,也复制对象包含的引用指向的对象
    public Employee clone() throws CloneNotSupportedException {
        Employee clone = (Employee) super.clone();
        
        clone.hireDay = (Date)hireDay.clone();
        
        return clone;
    }
}
```



## 3、其他克隆方式

上面手动重写clone方法的方式在关联对象少的情况是可取的，假设关联的对象里面也包含了对象，就需要层层修改，比较麻烦。不推荐这样使用！

### 利用fastJson实现克隆

这种方式依赖于阿里巴巴的fastJSON包。思路就是先序列化，之后再反序列化

```java
@Test
public void testFastJsonClone() {
    AccountDetail detail = new AccountDetail(1L, "1048791780@qq.com");
    Account account = new Account(1L, "小何", detail);

    //深克隆：序列化，再反序列化
    Account deepClone = JSON.parseObject(JSON.toJSONString(account), Account.class);
}
```

### 利用ObjectStream(推荐)

可以将方法放置在工具类中，方便使用

```java
private <T> T deepClone(T t) throws IOException, ClassNotFoundException {
    // First serializing the object and its state to memory using ByteArrayOutputStream instead of FileOutputStream.
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    ObjectOutputStream out = new ObjectOutputStream(bos);
    out.writeObject(t);
    
    // And then deserializing it from memory using ByteArrayOutputStream instead of FileInputStream.
    // Deserialization process will create a new object with the same state as in the serialized object,
    ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
    ObjectInputStream in = new ObjectInputStream(bis);
    return (T) in.readObject();
}
```

