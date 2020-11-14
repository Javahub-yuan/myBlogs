---
title:  深入理解HashMap（四）hashmap的resize方法
tags:
  - Java
  - JDK源码
  - HashMap
categories:
  - Java
comments: true
date: 2020-10-25 14:00:00
---

## resize方法

上一篇讲了put方法，在put方法的开头时会检查数组是否初始化，以及最后需要检查容量是否超过阈值，如果未初始化或越界需要进行扩容。resize方法就是对hashmap进行扩容。

通过阅读resize()方法源码，可以解答设计者在编写时考虑过的问题，例如：

- 什么时候进行resize操作？
- 扩容后的新数组容量为多大比较合适？
- 节点在转移的过程中是一个个节点复制还是一串一串的转移？

<!--more-->

## 源码

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        //判断是否超过最大值，超过就不再扩容，将临界值设为最大返回
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //没有超过最大值，就扩容为原来的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    /**
     * 在第一次带参数初始化时候会有这种情况
     * 
     * 从构造方法我们可以知道
     * 如果没有指定initialCapacity, 则不会给threshold赋值, 该值被初始化为0
     * 如果指定了initialCapacity, 该值被初始化成大于initialCapacity的最小的2的次幂
     * 这里这种情况指的是原table为空，并且在初始化的时候指定了容量，
     * 则用threshold作为table的实际大小
    */
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    //构造方法中没有指定容量，则使用默认值
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算指定了initialCapacity情况下的新的 threshold
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;


    /**
     * 从以上操作我们知道, 初始化HashMap时, 
     *  如果构造函数没有指定initialCapacity, 则table大小为16
     *  如果构造函数指定了initialCapacity, 则table大小为threshold,即大于指定initialCapacity的最小的2的整数次幂
    
     *  从下面开始, 初始化table或者扩容, 实际上都是通过新建一个table来完成
    */ 

    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    //桶中只有一个节点，直接放入新桶中
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    //桶中为红黑树,则进行红黑树操作 
                    //todo 红黑树操作之后再分析
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    //桶中为链表，对链表进行操作   
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    
                    //对链表进行遍历
                    //第一次进入循环时，e是一个链表的头结点，这个循环的作用就是遍历整个链表，将链表里面的节点按照"需要移动位置"和"不需要移动位置"分为两个链表
                    do {
                        next = e.next;
                        //  (e.hash & oldCap) == 0 判断得到的是 元素的在数组中的位置是否需要移动,
                        //  示例会在下面讲

                        //扩容之后元素不需要移动位置
                        //模拟三次进入操作
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                // 第一次进入时loTail != null，确定首元素    此时 e   -> aa  ; 执行后loHead-> aa
                                loHead = e;
                            else
                                //第二次进入时loTail != null  此时loTail-> aa  ;    e  -> bb ;  执行后loTail.next -> bb;而loHead和loTail是指向同一块内存的，所以loHead.next 地址为 bb 
                                //第三次进入时loTail != null  此时loTail-> bb  ;    e  -> cc ;  执行后loTail.next 地址为 cc;loHead.next.next = cc
                                loTail.next = e;
                            // 第一次进入时      e   -> aa  ; 执行后loTail-> aa loTail指向了和 loHead 相同的内存空间
                            // 第二次进入时      e   -> bb  ; 执行后loTail-> bb loTail指向了和 loHead.next相同的内存空间  
                            // 第三次进入时      e   -> cc  ; 执行后loTail-> cc loTail指向了和 loHead.next.next相同的内存
                            loTail = e;
                        }

                        //扩容之后元素需要移动位置,括号里面操作和上面类似
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);

                    //链表元素相对位置没有变化,不会倒置; 实际是对对象的内存地址进行操作
                    if (loTail != null) {
                        loTail.next = null;
                        //将没有发生位置变化的链表插入新数组中
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        //将发生位置变化的链表插入新数组中
                        //新数组中位置 = 旧数组位置(j) + 旧数组长度(oldCap)
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## resize()拆分链表

resize最重要的操作之一就是对链表的拆分了，那么resize是如何拆分链表的呢？

方法中定义了4个变量：loHead, loTail ,hiHead , hiTail，这四个变量从字面意思可以看出应该是两个头节点，两个尾节点。

```java
 Node<K,V> loHead = null, loTail = null;
 Node<K,V> hiHead = null, hiTail = null;
```

那么为什么需要两个链表的头尾节点呢？看一张图就明白了：



[![BmCBy4.png](https://s1.ax1x.com/2020/10/25/BmCBy4.png)](https://imgchr.com/i/BmCBy4)

这张图中index=2的桶中有四个节点，在未扩容之前，它们的 `(n - 1) & hash, 其中n=8` 都等于2。在扩容之后，它们之中2、18还在一起，10、26却换了一个桶。这就是这句代码的含义：选择出扩容后在同一个桶中的节点。

```java
//选择出扩容后在原来桶中的节点,节点不需要移动位置
if ((e.hash & oldCap) == 0) {
    if (loTail == null)
        loHead = e;
    else
        loTail.next = e;
    loTail = e;
}
//选择出扩容后不在原来桶中的节点,节点需要移动位置
else {
    if (hiTail == null)
        hiHead = e;
    else
        hiTail.next = e;
    hiTail = e;
}
```

我们这时候的oldCap = 8，2的二进制为：0010，8的二进制为：1000，0010 & 1000 =0000
10的二进制为：1010，1010 & 1000 = 1000，
18的二进制为：10010, 10010 & 1000 = 0000，
26的二进制为：11010，11010 & 1000 = 1000，
从与操作后的结果可以看出来，2和18应该在同一个桶中，10和26应该在同一个桶中。

所以lo和hi这两个链表的作用就是保存原链表拆分成的两个链表。

## 移动链表

下面这段代码是将拆分完的链表放进桶里的操作，比较简单，只需要将头节点放进桶里就ok了。

- 对于扩容后在原来桶中的节点，还放回原来位置newTab[j]，相当于之前那张图中的2；
- 对于扩容后不在原来桶中的节点,节点需要移动位置，数组中新位置就是`新数组中位置 = 旧数组位置(j) + 旧数组长度(oldCap)，即 newTab[j + oldCap]`，相当于之前那张图中的10。

```java
if (loTail != null) {
    loTail.next = null;
    newTab[j] = loHead;
}
if (hiTail != null) {
    hiTail.next = null;
    newTab[j + oldCap] = hiHead;
}
```

## 总结

最后我们再来总结一下之前提到的3个问题，

1. **什么时候进行resize操作？**

   有两种情况会进行resize：1、初始化table；2、在size超过threshold之后进行扩容

2. **扩容后的新数组容量为多大比较合适**？

   扩容后的数组应该为原数组的两倍，并且这里的数组大小必须是2的幂

3. **节点在转移的过程中是一个个节点复制还是一串一串的转移**？

   从源码中我们可以看出，扩容时是先找到拆分后处于同一个桶的节点，将这些节点连接好，然后把头节点存入桶中即可