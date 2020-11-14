---
title:  Java基础（五） - 内部类
tags:
  - Java
categories:
  - Java
comments: true
date: 2020-11-14 14:30:00




---

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201114210852.png)

<!--more-->

# 内部类

内部类是定义在一个类中的类。为什么需要内部类，主要有两个原因：

- 内部类可以对同一个包中其他类隐藏
- 内部类方法可以访问定义在这个类的作用域中的数据，包括原本私有数据

内部类原先对于简洁的实现回调非常重要，不过现在lambda表达式可以做的更好，但内部类对于构建代码还是很有用的。

**内部类的分类**

- 成员内部类
- 静态内部类
- 局部（方法）内部类
- 匿名内部类

> 代码的属性或方法名字中，包含outer的表示为外部类，外部类中的属性或方法；包含inner的表示为内部类，内部类的属性或方法；

## 一、成员内部类

### **1、小结：**

1. 成员内部类当成Outer成员信息的存在

2. 内部类以及内部类中的方法可以被任何修饰符修饰，但是**只有当内部类和内部类中方法都被public修饰时，才能被其他类访问调用**

3. 内部类的内部**不能有静态信息**

4. 内部类也是类，可以继承、重写重载、实现接口，调用this、super

5. 外部类或其他类（其他类访问时修饰符必须是public）如果要访问成员内部类，需要new之后打点访问

6. 内部类与外部类的属性或方法发生冲突，直接调用是内部类，需要使用：**外部类名.this.属性或方法名**

   ```java
   OuterClass outer = new OuterClass();
   Inner inner = outer.new Inner();
   outer.outerShow();
   ```



### **2、外部类、内部类代码**

```java
@Data
public class OuterClass {

    private int outerVariable = 1;
    private int commonVariable = 2;
    private static int outerStaticVariable = 3;

    //外部成员方法
    public void outerMethod(){
        System.out.println("我是外部类的outerMethod方法");
    }

    //静态方法
    public static void outerStaticMethod(){
        System.out.println("我是外部类的outerStaticMethod静态方法");
    }

    //内部类
    public class Inner{
        private int commonVariable = 200;
        
        //内部类的内部不能有静态信息
        //private static int innerStaticVariable = 300;  error

//        public Inner() {
//        }

        //内部类的成员方法
        public void innerShow(){
            //当与外部类发生冲突的时候，直接引用属性名，是内部类属心
            System.out.println("commonVariable = " + commonVariable);
            //访问外部类同名属性： 外部类名.this.属性名
            System.out.println("commonVariable = " + OuterClass.this.commonVariable);
            //内部类访问外部属性
            System.out.println("outerVariable = " + outerVariable);
            //访问外部静态属性
            System.out.println("outerStaticVariable = " + outerStaticVariable);
            //访问外部方法
            outerMethod();
            outerStaticMethod();

        }
    }

    //外部类访问内部信息
    public void outerShow(){
        Inner inner = new Inner();
        System.out.println("inner.commonVariable = " + inner.commonVariable);
        inner.innerShow();
    }


    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        Inner inner = outer.new Inner();
//        inner.innerShow();
        outer.outerShow();
    }

}
```

### **3、其他类使用成员内部类**

```java
public static void main(String[] args) {
    //外部类对象
    Outer outer = new Outer();
    //创造内部类对象
    //只有当内部类修饰符为public时，其他类才可以通过外部类new出内部类，否则会报错
    Outer.Inner inner = outer.new Inner(); 		
    inner.innerShow();
    
    /*
        * 可在Outer中定义get方法，获得Inner对象,那么使用时，只需outer.getInnerInstance()即可。
        * public Inner getInnerInstance(Inner类的构造方法参数){
        *   return new Inner(参数);
        * }
        */
}

```



## 二、静态内部类

### **1、小结（与成员内部类区别）**

1. 成员内部类不能有静态信息，而静态内部类可以包含任意信息
2. 静态内部类的方法只能访问外部类的static信息，**内部类普通方法**可以访问内部类所有信息，**内部类static方法**访问内部类中信息时只能访问static的
3. 利用 **外部类.内部类  引用=new  外部类.内部类()**; 然后利用 **引用.成员信息(属性、方法)**调用
4. 访问内部类的静态信息，直接用  **外部类.内部类.静态信息**就可以
5. 静态内部类可以**独立存在**，不依赖其他外部类

### **2、外部类和内部类**

```java
@Data
public class OuterStaticClass {

    private int outerVariable = 1;

    private int commonVariable = 2;

    private static int outerStaticVariable = 3;

    static {
        System.out.println("Outer的静态代码块执行了");
    }

    //成员方法
    public void outerMethod() {
        System.out.println("我是外部类的outerMethod方法");
    }

    //静态方法
    public static void outerStaticMethod() {
        System.out.println("我是外部类的outerStaticMethod的方法");
    }


    //静态内部类
    public static class Inner {
        private int innerVariable = 10;
        private int commonVariable = 200;

        static {
            System.out.println("Outer.Inner的静态块执行了");
        }

        private static int innerStaticVariable = 300;

        //成员方法
        public void innerShow() {
            System.out.println("innerVariable = " + innerVariable);
            //System.out.println(outerVariable);  error
            System.out.println("内部的commonVariable = " + commonVariable);
            //System.out.println(OuterStaticClass.this.commonVariable);  error
            //outerMethod();    error
            outerStaticMethod();
        }

        //静态方法
        public static void innerStaticShow(){
            System.out.println("innerStaticVariable = " + innerStaticVariable);
            //System.out.println("内部的commonVariable = " + commonVariable);  error
            //System.out.println("innerVariable = " + innerVariable);   error
            outerStaticMethod();
        }
    }

    //外部类访问内部类
    public static void callInner(){
        System.out.println(Inner.innerStaticVariable);
        Inner.innerStaticShow();
    }


    public static void main(String[] args) {
//        OuterStaticClass.callInner();
        OuterStaticClass.Inner.innerStaticShow();

        System.out.println("--------------");

        OuterStaticClass.Inner oi = new OuterStaticClass.Inner();
        oi.innerShow();
    }
}

```

### 3、**其他类使用成员内部类**

```java
public class Other {
    public static void main(String[] args) {
        //访问静态内部类的静态方法，Inner类被加载,此时外部类未被加载，独立存在，不依赖于外围类。
        Outer.Inner.innerStaticShow();
        //访问静态内部类的成员方法
        Outer.Inner oi = new Outer.Inner();
        oi.innerShow();
    }
}

```



## 三、局部（方法）内部类

### **1、小结**

1. 局部内部类的类前不能有访问修饰符

2. 仅能够在此方法中使用

3. 无法创造静态信息

4. 可以直接访问方法内的局部变量和参数（有限制：https://blog.csdn.net/qq_38238296/article/details/83315263），但是不能改

   - 局部内部类访问局部变量时，变量必须用final修饰（JDK1.8之后Java可以隐式将局部变量修饰为final，final可以不用写）

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200710223320414.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzM1MTgxNDIw,size_16,color_FFFFFF,t_70)

5. 可以随意访问外部类的任意信息

### **2、外部类、内部类**

```java
/**
 *	外部类、内部类
 */
public class Outer {
    /**
     * 属性和方法
     */
    private int outerVariable = 1;
    /**
     * 外部类定义的属性
     */
    private int commonVariable = 2;
    /**
     * 静态的信息
     */
    private static int outerStaticVariable = 3;

    /**
     * 成员外部方法
     */
    public void outerMethod() {
        System.out.println("我是外部类的outerMethod方法");
    }

    /**
     * 静态外部方法
     */
    public static void outerStaticMethod() {
        System.out.println("我是外部类的outerStaticMethod静态方法");
    }

    /**
     * 程序的入口
     */
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.outerCreatMethod(100);
    }

    /**
     * 成员方法，内部定义局部内部类
     */
    public void outerCreatMethod(int value) {
        /**
         * 女性
         */
        boolean sex = false;

        /**
         * 局部内部类，类前不能有访问修饰符
         */
        class Inner {

            private int innerVariable = 10;
            private int commonVariable = 20;
			/**
			*	局部内部类方法
			*/
            public void innerShow() {
                System.out.println("innerVariable:" + innerVariable);
                //局部变量
                System.out.println("是否男性:" + sex);
                System.out.println("参数value:" + value);
                //调用外部类的信息
                System.out.println("outerVariable:" + outerVariable);
                System.out.println("内部的commonVariable:" + commonVariable);
                System.out.println("外部的commonVariable:" + Outer.this.commonVariable);
                System.out.println("outerStaticVariable:" + outerStaticVariable);
                outerMethod();
                outerStaticMethod();
            }
        }
        //局部内部类只能在方法内使用
        Inner inner = new Inner();
        inner.innerShow();
    }
}
```



## 四、匿名内部类

### **1、小结**

1. 匿名内部类长春被用来重写某个或某些方法
2. 匿名内部类没有访问修饰符
3. 使用时，new 的这个类需要存在，需要重新里面的方法
4. 匿名内部类访问方法参数和局部内部类有同样限制
5. 匿名内部类没有构造方法

### **2、定义接口**

```java
/**
*	接口中方法默认为public 
*/
public interface IAnimal{
	void speak();
}
```

### **3、匿名内部类使用**

```java
/**
*	外部内、内部类
*/
public class Outer {

    public static IAnimal getInnerInstance(String speak){
        return new IAnimal(){
            @Override
            public void speak(){
                System.out.println(speak);
            }};
        	//注意上一行的分号必须有
    }
    
    public static void main(String[] args){
    	//调用的speak()是重写后的speak方法。
        Outer.getInnerInstance("小狗汪汪汪！").speak();
    }
}

```



> 参考：https://blog.csdn.net/weixin_42762133/article/details/82890555    浅谈Java内部类
>
> ​			https://blog.csdn.net/qq_38238296/article/details/83315263			局部内部类访问局部变量时，变量必须用final修饰












