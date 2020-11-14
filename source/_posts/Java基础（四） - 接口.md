---
title:  Java基础（四） - 接口
tags:
  - Java
categories:
  - Java
comments: true
date: 2020-11-14 14:00:00



---

<img src="https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201114205442.png" style="zoom:63%;" />

<!--more-->

# Java基础（四） - 接口

## 1、接口

接口是用来描述类应该做什么，而不指定具体如何做，一个类只能继承一个父类，但是可以实现一个或多个接口（Java不允许多重继承）。

接口中所有的方法都默认是`public`方法，所以声明方法可以省略访问修饰符，并且在Java8之前，接口中绝对不会实现方法（Java8后可以在接口中提供简单方法）。接口中不能包含实例字段，这也是接口与抽象类的区别之一，虽然不能包含实例字段，但接口中却可以包含常量，与方法类似，接口中的常量总是默认为`public static final`

```java
public interface A {
    void move(int x, int y);	//public method
}
//接口通过extends关键字扩展接口
public interface B extends A {
    double NUM = 10;	// public static final constant
    double getSalary();	//public method
}
```

## 2、接口属性

需要注意，与建立类的继承层次一样，通过`extends`关键字也可以扩展接口，如上代码，如果一个类实现的B接口，则需要实现接口A和B中的方法。实现类会自动继承接口中的常量，并可以直接引用

接口不能使用new运算符实例化一个接口，但却能声明接口变量，接口变量必须引用实现了这个接口的类对象

## 3、接口静态和私有方法

在Java8中，允许在接口中增加静态方法。在Java9中，接口中的方法可以是private，private方法可以是静态方法或实例方法。

静态方法如果没有实现，则编译器会报错

```java
public interface B extends A{
    double NUM = 10;
    double getSalary();

    //默认为public，所以省略了访问修饰符
    //静态方法如果没有实现，则编译器会报错
    static int getAge() {
        return 1;
    }
}
```

## 4、接口默认方法

可以为接口方法提供一个默认实现，但该方法必须用`default`修饰符标记，如果没有被default修饰，则会报错

```java
public interface B extends A{
    double NUM = 10;
    double getSalary();

    //默认方法，必须被default修饰
    default int getYear() {
        return 1;
    }
}
```

例如，在`Iterator`接口中，声明了一个remove方法,因为remove方法依赖于你要遍历访问的数据结构，所以方法没有具体实现，当你没有实现remove方法，且调用到的时候，则会抛出`UnsupportedOperationException("remove")`异常。不过，如果你的迭代器是只读，就不用操心实现remove方法了

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();

    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    ...
}
```

## 5、解决接口默认方法冲突

冲突情况：如果先在一个接口中将一个方法定义为默认方法，然后又在超类或另外一个接口中定义同样方法，则就会产生冲突。

1）**超类优先**：如果超类提供了一个具体方法，同名且有相同参数参数类型的默认方法会被忽略。

一个类继承了一个超类,同时实现了一个接口,并从超类和接口继承了相同的方法，在这种情况下,只会考虑超类方法,接口的所有默认方法都会被忽略.这正是”类优先”原则.

```java
//: 超类与接口方法冲突

//接口中冲突的默认方法都会被忽略
interface Named {
    default String getName() {
        return getClass().getName() + "_" + hashCode() + "    Named";
    }
}

class Base {
    public String getName() {
        return getClass().getName() + "_" + hashCode() + "   Base";
    }
}

public class Test2 extends Base implements Named {
    public static void main(String[] args) {
        Test2 t = new Test2();
        System.out.println(t.getName());
    }
}

```

2）**接口冲突**：如果一个超类接口提供了一个默认方法，另一个接口也提供了一个同名且参数类型（无论是否是默认参数）相同的方法。必须覆盖这个方法来解决冲突。

```java
//: 测试接口默认方法的冲突

interface Named {
    default String getName() {
        return getClass().getName() + "_" + hashCode() + "   Named";
    }
}

interface Base {
    default String getName() {
        return getClass().getName() + "_" + hashCode() + "   Base";
    }
}

public class Test implements Base, Named {
    //覆盖getName方法
    public String getName() {
        return "testName";
    }

    public static void main(String[] args) {
        Test t = new Test();
        System.out.println(t.getName());
    }
}

```

java设计者强调一致性.两个接口如何冲突并不重要.如果**至少一个接口提供了一个实现**,编译器就会报告一个错误,而程序员就要解决这个个二义性.

当然,如果**两个接口都没有为共享方法提供默认实现**,这里就不存在冲突.实现类可以有两个选择:实现这个方法,或者干脆不实现.如果不实现,那么这个实现类本身就是抽象的.

