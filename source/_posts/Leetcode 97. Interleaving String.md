---
title: Leetcode 97. Interleaving String
tags: 
    - LeetCode
    - String
    - Algorithm
category: LeetCode
mathjax: true
---


Given *s1, s2, s3*, find whether *s3* is formed by the interleaving of *s1* and *s2*.

**Example 1:**

```
Input: s1 = "aabcc", s2 = "dbbca", s3 = "aadbbcbcac"
Output: true
```

**Example 2:**

```
Input: s1 = "aabcc", s2 = "dbbca", s3 = "aadbbbaccc"
Output: false
```

## 题解

使用动态规划求解。

首先我们分解子问题。如果s3是s1和s2的交错字符串的话，那么一定有以下的一种情况

* s3的最后一位和s2的最后一位相同，此时s3前边的子串(除去最后一位)一定是**s2的前边的子串与s1的交错字符串。**
* s3的最后一位和s1的最后一位相同，此时s3前边的子串(除去最后一位)一定是**s1的前边的子串与s2的交错字符串。**

这个很容易想到，因为构造出交错的字符串的时候肯定是从s1选1个char，又从s2选1个char，所以如果当前是交错的串的话，去掉最后一位相同的位之后，前边的串肯定也是根据这个规则构造来的，所以也是交错的字符串。

因此我们可以写出迭代方程：

$$
length·of·s3 = l \\
length·of·s2 = n \\
length·of·s1 = m \\
isInterleave(s3[0...l]) = \\
        (s3[l] == s2[n] ^ isInterleave(s1[0...m], s2[0...n-1], s3[0...l-1])) \\
    or  (s3[l] == s1[m] ^ isInterleave(s1[0...m-1], s2[0...n], s3[0...l-1])) \\
$$

基于此方程我们写出如下的解：

## Solution

```cpp
class Solution {
public:
    bool isInterleave(string s1, string s2, string s3) {
        int m = s1.length(), n = s2.length(), l = s3.length();
        if (m + n != l) return false;
        vector<vector<bool>> dp(m+1, vector<bool>(n+1, false));
        for (int i = 0; i < m+1; i++) {
            for (int j = 0; j < n+1; j++) {
                if (i == 0 && j == 0) dp[0][0] = true;
                else if (i == 0) dp[i][j] = dp[i][j-1] && s2[j-1] == s3[i+j-1];
                // when i = 0, j = 1
                // dp[0][1] = dp[0][0] and s2[1] == s3[1]
                else if (j == 0) dp[i][j] = dp[i-1][j] && s1[i-1] == s3[i+j-1];
                // when i = 1, j = 0
                // dp[1][0] = dp[0][0] and s1[1] == s3[1]
                else dp[i][j] = (dp[i-1][j] && s1[i-1]==s3[i+j-1]) || (dp[i][j-1] && s2[j-1]==s3[i+j-1]);
                // when i = 1, j = 1
                // dp[1][1] = 
                //      | dp[0][1] && s1[1] == s3[2]
                //      | dp[1][0] && s2[1] == s3[1]
            }
        }
        return dp[m][n];
    }
};
```