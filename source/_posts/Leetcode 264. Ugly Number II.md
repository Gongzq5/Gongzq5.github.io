---
title: Leetcode 264. Ugly Number II
tags: 
- LeetCode
- Algorithm
category: LeetCode
mathjax: true
---

Write a program to find the `n`-th ugly number.

Ugly numbers are **positive numbers** whose prime factors only include `2, 3, 5`. 

**Example:**

```
Input: n = 10
Output: 12
Explanation: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 is the sequence of the first 10 ugly numbers.
```

**Note:**  

1. `1` is typically treated as an ugly number.
2. `n` **does not exceed 1690**.



## Solution 

```cpp
class Solution {
public:
    int nthUglyNumber(int n) {
        int i2 = 0, i3 = 0, i5 = 0;
        vector<int> dp {1};       
        for (int i = 0; i < n; i++) {            
            int m = min( min(dp[i2]*2, dp[i3]*3), dp[i5]*5 );
            dp.push_back(m);
            if (m == dp[i2]*2) i2++;
            if (m == dp[i3]*3) i3++;
            if (m == dp[i5]*5) i5++;
        }
        return dp[n-1];
    }
};
```



## 题解

应该大家看着还是挺迷茫的，我们从题目开始思考，要找的是质因数只有2、3、5的数，那么我们可以产生两种思路，**一种是验证的思路**，对于每个数进行验证，判断他是否有2、3、5之外的质因子；**或者是采用构造的思路**，从某个数开始构造满足这个条件的数字。

明显第一种思路耗时很久，毕竟现在还是没有很好地判断因子分解的算法。从另一种思路出发，我们可以这样考虑：所有以2、3、5为因子的数相乘并不会产生其他的素数作为因子，我们可以简单地推导一下

1. 很明显素数 $p$ 和 $q$ 相乘时，$pq$不存在其他的素数作为它的因子；
2. $ p^i q^j $明显也没有其他素数作为因子

因此这个算法应该是成立的！

因此，又为了排序的目的，我们逐个取最小值产生`ugly number`的数组。

如下：

```cpp
t2 = 0, t3 = 0, t5 = 0 
uglg numbers [0] = 1
for i from 1 to n
	uglg numbers [i] = min( uglg numbers [t2], 
							uglg numbers [t3], 
							uglg numbers [t5])
return uglg numbers [n-1] 
```

