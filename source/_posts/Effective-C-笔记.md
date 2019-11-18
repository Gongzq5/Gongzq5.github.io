---
title: Effective C++ 笔记
date: 2019-09-21 15:53:14
tags:
category:
---



<!-- more -->

[TOC]

# 导读

## 01. `explicit`关键字

通常会将一个构造函数声明为`explicit`，可以防止被隐式转换。

```cpp
class B {
public:
    explicit B(int x=0);
};
void DoSomething(B objB);

// 此时若没有explicit，那么
DoSomething(3); // 是可以运行的
// 有explicit，那么便不可以了，只能写成
DoSomething(B(3)); // 这个样子
```



## 02. copy 方法

有两种

```cpp
class B {
public:
    B();
    B(const B&);
    B& operator=(const B&);
};
```

如果声明了新的对象，那么一定会调用一个构造函数：

```cpp
B b1; // 调用默认构造函数
B b2 = b1; // 调用copy构造函数
```

而如果只是单纯的赋值，才会调用`=`函数：

```cpp
B b1, b2; // 都会调用默认构造函数
b2 = b1; // 此时会调用赋值函数
```



# 正文

## 条款01：视C++为一个语言联邦

把C++看作四个子语言的联合体

* C
* 面向对象的机制
* 模板（template），泛型
* STL

每个部分都有自己的规约，当你切换的时候可能会被要求遵循不同的规约。



## 条款02：用`const,enum,inline`而不是`define`

推荐使用`const`定义常量而不是宏，值得注意的是，类内部定义时静态常量时，推荐使用类外的定义式和初始化。

> 类内的初始化可能不能被识别，视编译器而定

```cpp
// 声明文件
class B {
private:
    // 这里的叫做声明式
    static const int AIntVal;
    // 对某些编译器而言，支持整数常量的类内初始化，比如：
    // static const int AIntVal = 2;
    // 请注意，也只有整数类型才可以，下边的double不可以
    static const double ADoubleVal;
};

// 实现文件
// 下面的则是定义式
const int B::AIntVal = 2; 
// 如果之前有类内初始化，那么类外也可以只写一个定义式：
// const int B::AIntVal;
// 如果不需要对这个变量进行取地址操作，定义式也可以不写
// 但是这也要视编译器的实现而定，有些编译器会强制要求提供一个定义式
const double B::ADoubleVal = 2.5;
```



一个类内初始化的必要应用场景是在类的编译期间需要使用这个常量的值，比如：

```cpp
class B {
    static const int NameLen = 2;
    char name[NameLen];
};
```

此时`NameLen`变量便不可以在类外初始化了，而如果编译器不支持类内初始化，此时我们可以使用`enum`关键字。

```cpp
class B {
    enum { NameLen = 5 };
    char name[NameLen];
};
```

这样声明的`NameLen`和`#define`得到的常量的行为是十分相似的，比如不能使用取地址等操作。



最后是使用`inline`关键字，宏函数太过丑陋，因此使用`inline`搭配`template`是更好的做法。

```cpp
#define CALL_WITH_MAX(a, b) \
    (f((a) > (b) ? (a) : (b)))

// 丑陋的括号，而且哪怕是这样还是会出现问题，比如有人会用
int a = 2;
CALL_WITH_MAX(a++, 0);
a = 2;
CALL_WITH_MAX(a++, 4);
// 这两次调用后a的值是不一样的

// 所以最好使用
template<typename T>
inline void CALL_WITH_MAX(const T& a, const T& b) {
    f(a > b ? a : b);
} 
```



而且`inline`函数还会提供一个函数的作用域，而`define`里如果添加了新的变量，会造成作用域的混乱。当然也是有解决办法的：

```cpp
#define TEST_FUNC(a) \
    do { \
        int inner = 0; \
    } while(0);
```



## 条款03 尽可能的使用`const`关键字

先写写`const`的用法啊

| const的位置                | 含义                                           |
| -------------------------- | ---------------------------------------------- |
| `const int a = 2;`         | `a`的值不能被修改                              |
| `const int *p = &a;`       | `p`不会被修改，即`p`这个指针不会指向其他地方了 |
| `int* const p = &a;`       | 即`p`这个指针指向的值不可以更改                |
| `const int* const p = &a;` | 即`p`这个指针指向的值和自己都不能发生更改。    |

`const`还可以放在类内