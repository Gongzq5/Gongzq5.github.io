---
title: LeetCode 87. Scramble String
tags: 
    - LeetCode
    - String
    - Algorithm
category: LeetCode
mathjax: true
---

## 题目

Given a string *s1*, we may represent it as a binary tree by partitioning it to two non-empty substrings recursively.

Below is one possible representation of *s1* = `"great"`:

```
    great
   /    \
  gr    eat
 / \    /  \
g   r  e   at
           / \
          a   t
```

To scramble the string, we may choose any non-leaf node and swap its two children.

For example, if we choose the node `"gr"` and swap its two children, it produces a scrambled string `"rgeat"`.

```
    rgeat
   /    \
  rg    eat
 / \    /  \
r   g  e   at
           / \
          a   t
```

We say that `"rgeat"` is a scrambled string of `"great"`.

Similarly, if we continue to swap the children of nodes `"eat"` and `"at"`, it produces a scrambled string `"rgtae"`.

```
    rgtae
   /    \
  rg    tae
 / \    /  \
r   g  ta  e
       / \
      t   a
```

We say that `"rgtae"` is a scrambled string of `"great"`.

Given two strings *s1* and *s2* of the same length, determine if *s2* is a scrambled string of *s1*.

**Example 1:**

```
Input: s1 = "great", s2 = "rgeat"
Output: true
```

**Example 2:**

```
Input: s1 = "abcde", s2 = "caebd"
Output: false
```



## Solution

> **283 / 283** test cases passed.
>
> Status: **Accepted**
>
> Runtime: **0 ms**

```cpp
class Solution {
public:
    bool isScramble(string s1, string s2) {
        if(s1==s2)
            return true;
            
        int len = s1.length();
        int count[26] = {0};
        for(int i=0; i<len; i++)
        {
            count[s1[i]-'a']++;
            count[s2[i]-'a']--;
        }
        
        for(int i=0; i<26; i++)
        {
            if(count[i]!=0)
                return false;
        }
        
        for(int i=1; i<=len-1; i++)
        {
            if( isScramble(s1.substr(0,i), s2.substr(0,i)) && isScramble(s1.substr(i), s2.substr(i)))
                return true;
            if( isScramble(s1.substr(0,i), s2.substr(len-i)) && isScramble(s1.substr(i), s2.substr(0,len-i)))
                return true;
        }
        return false;
    }
};
```

## 题解

可以分析一下，两个串是 `isScramble` 的话，那么其中一个串，是通过另一个串进行多次的交换得来的。交换可以交换两个相邻的字符，也可以交换相邻的子串。

类似的我们可以分析，如果两个串满足这个性质，那么我们可以把它们都分为两个长度相同的部分（前i位，记为s11, s21，和后n-i位，记为s12, s22）。

思路很简单，如果两个串是 `isScramble` 的话，那么肯定是上下相对应，或者交换后相对应（即满足isScramble的)。也就是说，我们可以通过分别验证这两个性质来确认其是否满足条件，只要有一个满足，那么它就是isScramble的。

在这样的思路上我们可以做一些优化，比如将每次递归的结果存起来，使得递归到已经算过的子串的时候可以直接获取答案而不是再次进行计算。

但是在实际使用中发现优化后的代码运行速度并不快，我推测这是因为当前递归程序的重复计算已经被消除了，相同的子串不会进行多次判断，因而完全无需进行记忆性的递归。

## 分析

* **时间复杂度：** 这是一个递归算法，每次循环n次，最多递归n层，时间复杂度为O(n!)，这是最坏情况
