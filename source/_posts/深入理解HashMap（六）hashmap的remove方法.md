---
title:  深入理解HashMap（六）hashmap的remove方法
tags:
  - Java
  - JDK源码
  - HashMap
categories:
  - Java
comments: true
date: 2020-10-25 16:00:00
description: 本文主要分析HashMap中的remove()方法，remove方法也是核心且比较常用的方法之一
---

<!--more-->

## remove方法

remove方法中调用了removeNode方法，removeNode方法和putVal方法非常的像，这两者本来就是一个删一个增，所以在代码上有共性。我之前的文章中有更详细的说明，可以参考一下

## 源码

```java
/**
     * Removes the mapping for the specified key from this map if present.
     *
     * @param  key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

 final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        //当table不为空，并且hash对应的桶不为空时
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            //桶中的头节点就是我们要删除的节点
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                //用node记录要删除的头节点
                node = p;
            //头节点不是要删除的节点，并且头节点之后还有节点
            else if ((e = p.next) != null) {
            	//头节点为树节点，则进入树查找要删除的节点
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                //头节点为链表节点
                else {
                	//遍历链表
                    do {
                    	//hash值相等，并且key地址相等或者equals
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                             //node记录要删除的节点
                            node = e;
                            break;
                        }
                        //p保存当前遍历到的节点
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            //我们要找的节点不为空
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                	//在树中删除节点
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                //我们要删除的是头节点
                else if (node == p)
                    tab[index] = node.next;
                //不是头节点，将当前节点指向删除节点的下一个节点
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }

```

