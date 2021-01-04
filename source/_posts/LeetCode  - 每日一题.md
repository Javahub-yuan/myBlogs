---
title: LeetCode - 每日一题
tags:
  - LeetCode
categories:
  - LeetCode
comments: true
date: 2020-12-14 20:30:00


---

永远不要把自己的快乐和希望寄托在别人身上，最终靠得住的,还是自己

<!--more-->

# LeetCode - 每日一题

|      日期      |            题目             |   标签   |
| :------------: | :-------------------------: | :------: |
| 2020年12月14日 | [49. 字母异位词分组](#jump) |          |
| 2020年12月15日 |     738. 单调递增的数字     | 贪心算法 |
| 2020年12月16日 |        290. 单词规律        |  哈希表  |
| 2020年12月21日 |   746. 使用最小花费爬楼梯   | 动态规划 |

## [1. 字母异位词分组](#jump)

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201214223301.png)

### 方法一：排序

由于互为字母异位词的两个字符串包含的字母相同，因此对两个字符串分别进行排序之后得到的字符串一定是相同的，故可以将排序之后的字符串作为哈希表的键。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            char[] array = str.toCharArray();
            //对char数组进行排序
            Arrays.sort(array);
            String key = new String(array);
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/group-anagrams/solution/zi-mu-yi-wei-ci-fen-zu-by-leetcode-solut-gyoc/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

复杂度分析

时间复杂度：O(nklogk)，其中 n 是 strs 中的字符串的数量，kk 是 strs 中的字符串的的最大长度。需要遍历 n 个字符串，对于每个字符串，需要 O(klogk) 的时间进行排序以及 O(1) 的时间更新哈希表，因此总时间复杂度是 O(nklogk)。

空间复杂度：O(nk)，其中 n 是 strs 中的字符串的数量，k 是 strs 中的字符串的的最大长度。需要用哈希表存储全部字符串。



**关键点：**

1. 根据题目要求，互为字母异位词成立条件就是包含字母必须相同，所以可以根据这一点，结合哈希表求解

2. 对String排序时，先拆为char类型数组，再进行排序`char[] array = str.toCharArray(); Arrays.sort(array); `，排序过后再组合为String类型`String key = new String(array);`

3. map经典场景：判断key值是否存在，如果不存在则存入。可以使用`getOrDefault`方法优化

   ```java
   //old
   if(map.containsKey(key)) {
       map.get(key).add(str);
   } else {
       List<String> list = new ArrayList<>();
       list.add(str);
       map.put(key, list);
   }
   //new
   List<String> list = map.getOrDefault(key, new ArrayList<String>());
   list.add(str);
   ```

4. map的三种遍历方法

   ```java
   Map<String, List<String>> map = new HashMap<>();
   
   //值遍历
   for (List<String> list : map.values()) {
   }
   
   //键遍历
   for (String s : map.keySet()) {
       List<String> value = map.get(s);
   }
   
   //键值对遍历
   for (Map.Entry<String, List<String>> entry : map.entrySet()) {
       entry.getKey();
       entry.getValue();
   }
   ```

### 方法二：计数

由于互为字母异位词的两个字符串包含的字母相同，因此两个字符串中的相同字母出现的次数一定是相同的，故可以将每个字母出现的次数使用字符串表示，作为哈希表的键。

由于字符串只包含小写字母，因此对于每个字符串，可以使用长度为 2626 的数组记录每个字母出现的次数。需要注意的是，在使用数组作为哈希表的键时，不同语言的支持程度不同，因此不同语言的实现方式也不同。

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<String, List<String>>();
        for (String str : strs) {
            int[] counts = new int[26];
            int length = str.length();
            for (int i = 0; i < length; i++) {
                counts[str.charAt(i) - 'a']++;
            }
            // 将每个出现次数大于 0 的字母和出现次数按顺序拼接成字符串，作为哈希表的键
            StringBuffer sb = new StringBuffer();
            for (int i = 0; i < 26; i++) {
                if (counts[i] != 0) {
                    sb.append((char) ('a' + i));
                    sb.append(counts[i]);
                }
            }
            String key = sb.toString();
            List<String> list = map.getOrDefault(key, new ArrayList<String>());
            list.add(str);
            map.put(key, list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/group-anagrams/solution/zi-mu-yi-wei-ci-fen-zu-by-leetcode-solut-gyoc/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

复杂度分析

时间复杂度：O(n(k+∣Σ∣))，其中 n 是 strs 中的字符串的数量，k 是 strs 中的字符串的的最大长度，Σ 是字符集，在本题中字符集为所有小写字母，∣Σ∣=26。需要遍历 n 个字符串，对于每个字符串，需要 O(k) 的时间计算每个字母出现的次数，O(∣Σ∣) 的时间生成哈希表的键，以及 O(1) 的时间更新哈希表，因此总时间复杂度是 O(n(k+∣Σ∣))。

空间复杂度：O(n(k+∣Σ∣))，其中 n 是 strs 中的字符串的数量，k 是 strs 中的字符串的最大长度，Σ 是字符集，在本题中字符集为所有小写字母，∣Σ∣=26。需要用哈希表存储全部字符串，而记录每个字符串中每个字母出现次数的数组需要的空间为 O(∣Σ∣)，在渐进意义下小于 O(n(k+∣Σ∣))，可以忽略不计。

## 2.单调递增的数字

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201215233829.png)

题解

```java
class Solution {
    public int monotoneIncreasingDigits(int N) {
        char[] strN = Integer.toString(N).toCharArray();
        int i = 1;
        while (i < strN.length && strN[i - 1] <= strN[i]) {
            i += 1;
        }
        if (i < strN.length) {
            while (i > 0 && strN[i - 1] > strN[i]) {
                strN[i - 1] -= 1;
                i -= 1;
            }
            for (i += 1; i < strN.length; ++i) {
                strN[i] = '9';
            }
        }
        return Integer.parseInt(new String(strN));
    }
}

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/monotone-increasing-digits/solution/dan-diao-di-zeng-de-shu-zi-by-leetcode-s-5908/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

高人竟在我身边(我他么直接人傻掉了)

```java
public int monotoneIncreasingDigits(int N) {
    for (int i = N, j = 9, k = 1; i > 0; i /= 10, k *= 10)
        if (j < (j = i % 10))// 如果后一位比前一位小
            // 以332为例，第1次走到这一步的时候 i=33,k=10, 329进入递归
            // 第2次走到这一步的时候 i=3,k=100, 299进入递归
            return monotoneIncreasingDigits(i * k - 1);
    // 299递归出口
    return N;
}
```

## 3.单词规律

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201216230208.png)

题解

```java
public boolean wordPattern(String pattern, String s) {
    Map<Character, String> map = new HashMap<>();
    List<String> list = new ArrayList<>();

    String[] sArray = s.split(" ");
    if(pattern.length() != sArray.length) return false;

    for (int i = 0; i < sArray.length; i++) {
        char c = pattern.charAt(i);
        if(map.containsKey(c)) {
            if(!map.get(c).equals(sArray[i])) {
                return false;
            }
        } else {
            if(list.contains(sArray[i])) {
                return false;
            }
            map.put(c, sArray[i]);
            list.add(sArray[i]);
        }
    }
    return true;

}
```

官方

```java
class Solution {
    public boolean wordPattern(String pattern, String str) {
        Map<String, Character> str2ch = new HashMap<String, Character>();
        Map<Character, String> ch2str = new HashMap<Character, String>();
        int m = str.length();
        int i = 0;
        for (int p = 0; p < pattern.length(); ++p) {
            char ch = pattern.charAt(p);
            if (i >= m) {
                return false;
            }
            int j = i;
            while (j < m && str.charAt(j) != ' ') {
                j++;
            }
            String tmp = str.substring(i, j);
            if (str2ch.containsKey(tmp) && str2ch.get(tmp) != ch) {
                return false;
            }
            if (ch2str.containsKey(ch) && !tmp.equals(ch2str.get(ch))) {
                return false;
            }
            str2ch.put(tmp, ch);
            ch2str.put(ch, tmp);
            i = j + 1;
        }
        return i >= m;
    }
}

//时间复杂度：O(n+m)，其中 nn 为 pattern 的长度，m 为 str 的长度。插入和查询哈希表的均摊时间复杂度均为 O(n+m)。每一个字符至多只被遍历一次。

//空间复杂度：O(n+m)，其中 n 为 pattern 的长度，m 为 str 的长度。最坏情况下，我们需要存储 pattern 中的每一个字符和 str 中的每一个字符串。

作者：LeetCode-Solution
链接：https://leetcode-cn.com/problems/word-pattern/solution/dan-ci-gui-lu-by-leetcode-solution-6vqv/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```



## [4. 使用最小花费爬楼梯](https://leetcode-cn.com/problems/min-cost-climbing-stairs/)

> **题目：**
>
> 数组的每个索引作为一个阶梯，第 `i`个阶梯对应着一个非负数的体力花费值 `cost[i]`(索引从0开始)。
>
> 每当你爬上一个阶梯你都要花费对应的体力花费值，然后你可以选择继续爬一个阶梯或者爬两个阶梯。
>
> 您需要找到达到楼层顶部的最低花费。在开始时，你可以选择从索引为 0 或 1 的元素作为初始阶梯。
>
> **示例 1:**
>
> ```
> 输入: cost = [10, 15, 20]
> 输出: 15
> 解释: 最低花费是从cost[1]开始，然后走两步即可到阶梯顶，一共花费15。
> ```
>
>  **示例 2:**
>
> ```
> 输入: cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
> 输出: 6
> 解释: 最低花费方式是从cost[0]开始，逐个经过那些1，跳过cost[3]，一共花费6。
> ```
>
> **注意：**
>
> 1. `cost` 的长度将会在 `[2, 1000]`。
> 2. 每一个 `cost[i]` 将会是一个Integer类型，范围为 `[0, 999]`。

**方法一：动态规划**

- 标签：动态规划

- 和“每次上楼梯可以走1步也可以走2步，一共有多少种走法”类似

- ![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/20201221231050.png)

- 

  ```java
  class Solution {
      public int minCostClimbingStairs(int[] cost) {
          int n = cost.length;
          int[] dp = new int[n + 1];
          dp[0] = dp[1] = 0;
          for (int i = 2; i <= n; i++) {
              dp[i] = Math.min(dp[i - 1] + cost[i - 1], dp[i - 2] + cost[i - 2]);
          }
          return dp[n];
      }
  }
  
  //时间复杂度和空间复杂度都是 O(n)
  ```

**方法二：滚动数组优化**

- 标签：滚动数组，双指针

- 注意到当 i≥2 时，dp[i] 只和 dp[i−1] 与 dp[i−2] 有关，因此可以使用滚动数组的思想，将空间复杂度优化到 O(1)

  ```java
  class Solution {
      public int minCostClimbingStairs(int[] cost) {
          int n = cost.length;
          int prev = 0, curr = 0;
          for (int i = 2; i <= n; i++) {
              int next = Math.min(curr + cost[i - 1], prev + cost[i - 2]);
              prev = curr;
              curr = next;
          }
          return curr;
      }
  }
  
  //时间复杂度 O(n), n是cost数组长度
  //空间复杂度都是 O(1)
  ```

  

