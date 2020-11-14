---
title:  深入理解HashMap（五）hashmap的get方法
tags:
  - Java
  - JDK源码
  - HashMap
categories:
  - Java
comments: true
date: 2020-10-25 15:00:00
---

## get方法

get方法是相对比较简单的，如果看过前面几篇肯定不能理解get方法，以及里面的一些小细节。get方法里面又调用了getNode方法，如果getNode方法返回的是null，说明没找到这个key。

<!--more-->

## 源码

```java
/**
     * Returns the value to which the specified key is mapped,
     * or {@code null} if this map contains no mapping for the key.
     *
     * <p>More formally, if this map contains a mapping from a key
     * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
     * key.equals(k))}, then this method returns {@code v}; otherwise
     * it returns {@code null}.  (There can be at most one such mapping.)
     *
     * <p>A return value of {@code null} does not <i>necessarily</i>
     * indicate that the map contains no mapping for the key; it's also
     * possible that the map explicitly maps the key to {@code null}.
     * The {@link #containsKey containsKey} operation may be used to
     * distinguish these two cases.
     *
     * @see #put(Object, Object)
     */
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        //当table不为空，并且所在桶中不为空
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //先检查桶中的头节点是否为我们需要get的节点
            if (first.hash == hash && // always check first node
            	//满足key的地址相同或equals，返回头节点
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //桶中不止一个节点
            if ((e = first.next) != null) {
            	//桶中为树
                if (first instanceof TreeNode)
                	//在树中查找，这里有机会再讲吧
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
				//这里的do-while循环目的是遍历当前桶中的链表
                do {
                	//两个key的hash值相同，并且有相同地址或者equals的key，返回目标节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

```



