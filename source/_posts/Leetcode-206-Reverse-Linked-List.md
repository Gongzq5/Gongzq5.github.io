---
title: Leetcode 206. Reverse Linked List
date: 2019-10-09 20:53:20
tags: 
- Leetcode
- Linked List
category: 
- Leetcode
---



# 题目

Reverse a singly linked list.

**Example:**

```
Input: 1->2->3->4->5->NULL
Output: 5->4->3->2->1->NULL
```

**Follow up:**

A linked list can be reversed either iteratively or recursively. Could you implement both?



简单来说，将一个链表反转过来（还推荐使用迭代和递归两种方法来实现）



# 题解

## 迭代法

### 代码

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        ListNode* prev = nullptr;
        while (head) {
            ListNode* tmpNext = head->next;
            head->next = prev;
            prev = head;
            head = tmpNext;
        }
        return prev;
    }
};
```

### 分析

画图分析：

每个部分的蓝色是我们预先定义的一些变量，下边的 红色->黄色->绿色 是我们反转链表的步骤（标注了1，2，3以方便理解）。

![1570627740137](C:\Users\76519\AppData\Roaming\Typora\typora-user-images\1570627740137.png)



## 递归法

### 代码

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (head == nullptr || head->next == nullptr) return head;
        ListNode* node = reverseList(head->next);
        head->next->next = head;
        head->next = nullptr;
        return node;
    }
};
```



### 分析

简单来说，递归法的思路是这样的

* 首先我们将头部后边的反转；
* 然后我们将后边的尾巴连接到头上；
* 然后把头的`next`指向`nullptr`；

