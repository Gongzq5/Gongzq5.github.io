---
title: Leetcode 152. Maximum Product Subarray
tags:
    - LeetCode 
    - Array 
    - Algorithm 
category:  LeetCode 
mathjax: true
---

## 题目

Given an integer array `nums`, find the contiguous subarray within an array (containing at least one number) which has the largest product.

**Example 1:**

```
Input: [2,3,-2,4]
Output: 6
Explanation: [2,3] has the largest product 6.
```

**Example 2:**

```
Input: [-2,0,-1]
Output: 0
Explanation: The result cannot be 2, because [-2,-1] is not a subarray.
```



## 题解

这道题的目标是找到最大乘积的子串，还是比较复杂的，看题目就感觉是一道动态规划，我们**首先肯定想到的是这样的递推公式**：
$$
maxProduct[i] = max\{ maxProduct[i-1] * nums[i] , nums[i] \}
$$
其中，`maxProduct[i]`表示以第`i`位结尾的所有子串的最大乘积，所以要么选择`nums[i]`乘进去得到更大的值，要么不选择它，而是从他开始重新计算一个子串。

**这个解法会遇到一个问题**，比如`[1, 2, -3, -5]`，在面对两个负数的时候，我们的解法遇到-3时就会把它去掉而在-3的地方重新计算。即，`maxProduct[i] = nums[i] = 3` ，但是，如果我们保留`maxProduct[2] = maxProduct[i-1]*nums[i] = -6`，那么我们在计算到-5时就可以得到更大的值30。

**考虑这样的情况，我们要改造我们的算法**，我们在保留最大的乘积的同时，保留一个最小的负值，期望这个负值可以让最终得到另一个负值时可以得到更大的计算结果。即：

当`nums[i]`为正值时：
$$
postiveMax[i] = max \{ nums[i] , postiveMax[i-1]*nums[i] \} \\
negtiveMin[i] = min \{ nums[i] , negtiveMin[i-1]*nums[i] \}
$$
很简单，最小的负值就是当前的最小负值乘上`nums[i]`或者本身（重新开始），最大的正值也就是当前的最大正值乘上`nums[i]`或者本身（重新开始）。

当`nums[i]`为负值时：
$$
postiveMax[i] = max\{ nums[i] , negtiveMin[i-1]*nums[i] \}  \\
negtiveMin[i] = min\{ nums[i] , postiveMax[i-1]*nums[i] \};
$$
此时，最大正值可以通过最大的负值乘以当前的负`nums[i]`得来，也就解决了上面的问题。

下面我们进行实现：

## Solution 1

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {  
        int re = nums[0];
        vector<int> postiveMax(nums);
        vector<int> negtiveMin(nums);
        for (int i = 1; i < nums.size(); i++) {
            if ( nums[i] < 0 ) {
                postiveMax[i] = max( nums[i] , negtiveMin[i-1]*nums[i] );
                negtiveMin[i] = min( nums[i] , postiveMax[i-1]*nums[i] );
            } else {
                postiveMax[i] = max( nums[i] , postiveMax[i-1]*nums[i] );
                negtiveMin[i] = min( nums[i] , negtiveMin[i-1]*nums[i] );
            }
            re = max(re, postiveMax[i]);
        }
        return re;
    }    
};
```

还可以进行一些小小的优化使得代码量更加减少：

```cpp
class Solution {
public:
    int maxProduct(vector<int>& nums) {  
        int re = nums[0];
        vector<int> postiveMax(nums), negtiveMin(nums);
        for (int i = 1; i < nums.size(); i++) {            
            postiveMax[i] = max( postiveMax[i-1]*nums[i] , 
                                max( nums[i] , negtiveMin[i-1]*nums[i] ) );
            negtiveMin[i] = min( negtiveMin[i-1]*nums[i] , 
                                min( nums[i] , postiveMax[i-1]*nums[i] ) );
            re = max(re, postiveMax[i]);
        }
        return re;
    }    
};
```

把判断`nums[i]`正负性的一步去掉了，因为我们其实无需关心到底是取了哪一个，我们只要它拿到最小和最大即可了。

## 分析

我们分析这里的复杂度，发现空间复杂度是O(n)，时间复杂度也是O(n)。

这个解法还是可以更加优化一下的，空间复杂度可以降为O(1)，即每次只保留前一次(`max[i-1]`)的结果就好了， 不需要知道前(`max[i-2, i-3, ...]`)的内容。