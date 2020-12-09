---
title:  LeetCode（一） - 数组
tags:
  - LeetCode
categories:
  - LeetCode
comments: true
date: 2020-12-8 20:30:00

---

永远不要把自己的快乐和希望寄托在别人身上，最终靠得住的,还是自己

<!--more-->



# LeetCode（一） - 数组

|                             题目                             |  思路  |     日期      |
| :----------------------------------------------------------: | :----: | :-----------: |
| [88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/) | 双指针 | 2020年12月8日 |
|    [66. 加一](https://leetcode-cn.com/problems/plus-one/)    |  数组  | 2020年12月9日 |

## 1、[88. 合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

### 题目描述

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201208231600.png)

### 解题思路

#### 方法一 : 合并后排序

最朴素的解法就是将两个数组合并之后再排序。该算法只需要一行(Java是2行)，时间复杂度较差，为`O((n + m)\log(n + m))`。这是由于这种方法没有利用两个数组本身已经有序这一点。

```java
class Solution {
  public void merge(int[] nums1, int m, int[] nums2, int n) {
    System.arraycopy(nums2, 0, nums1, m, n);
    Arrays.sort(nums1);
  }
}
```

**时间复杂度** :` O((n + m)\log(n + m))`。
**空间复杂度** : `O(1)`。

#### 方法二 : 双指针 / 从后往前

一开始思路是想到了有一种排序中（记不清了）合并两个有序数组的操作，有些类似，可以利用双指针分别遍历两个数组，又考虑到`nums1.length == m + n`，根据这个条件，可以考虑从后往前遍历，当遍历结束时，如果nums2没有遍历完，就直接将剩余元素移至nums1中即可

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int i = m - 1;
        int j = n - 1;
        int len = nums1.length - 1;
        
        for(;i >= 0 && j >= 0;) { 
            if (nums1[i] < nums2[j]) {
                nums1[len--] = nums2[j--];
            } else {
                nums1[len--] = nums1[i--];
            }
        }
        
        if(j >= 0) {
            System.arraycopy(nums2, 0, nums1, 0, j + 1);
        }
    }
}
```

- **时间复杂度** : `O(n + m)`。
- **空间复杂度** : `O(1)`。

## [66. 加一](https://leetcode-cn.com/problems/plus-one/)

### 题目描述

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201209233952.png)

### 解题思路

**注意点：**

- 从后往前加一，如果该位<10，则直接返回
- 如果遍历完仍没有返回，说明原数组中每位数字都变为0，例如原数组为999，则直接新建一个数组，首位置1即可

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int len = digits.length;
        for(int i = len - 1; i >= 0; i--) {
            digits[i]++;
            digits[i] %= 10;
            if(digits[i]!=0)
                return digits;
        }
        digits = new int[len + 1];
        digits[0] = 1;
        return digits;
    }
}
```

