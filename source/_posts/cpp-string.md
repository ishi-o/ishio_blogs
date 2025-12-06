---
title: "Cpp STL: std::string"
date: 2024-03-14
categories: [Programming, C/Cpp]
tags: [Cpp STL, string]
---
<!-- placeholder -->
<!-- more -->
# `string`类

## `ctors`

```c++
// 定义:uint指size_type,即size_t,或许是unsigned int或unsigned long long
string();    // 默认ctor,创建长度为0的对象
string(uint n,char c); // 创建内容为n个字符c构成的字符串的对象
string(const char*); // 提供C风格字符串,创建内容为该字符串的对象
string(const char*,uint n);// 提供C风格字符串和一个长度,创建内容为字符串前n个字符的对象
string(const string&); // 复制构造函数,提供一个string对象,创建与该对象完全相同的新对象
// 创建内容为下标从idx开始数,一共n(默认为其长度)个字符的对象,即string(str,0)与string(str)等效
string(const string&,uint idx,uint n=npos);
string(It begin,It end);// 提供begin,end两指针或两迭代器,创建内容为[begin,end)的对象
string(string&&);  // 移动构造函数,可在临时对象将要析构而资源需要再利用时使用
string(initializer_list<char>);// 提供由单字符组成的列表,创建内容为它们整合而成的字符串的对象

// 示例
char CStr[6] = "12345"; // C风格字符串
string A_str;   // 默认ctor
string B_str("12345"); // string字符串
string C_str(CStr);  string C_str(B_str);
 // 拆子串的方法
string D_str(CStr,3) // D_str=="123" 新对象字符数为3,内容为前3个字符
string E_str(B_str,1); // E_str=="2345"新对象字符数为5(字符串长)-1,内容为下标1开始到结尾
string F_str(B_str,1,2);// F_str=="23"  新对象字符数为2,内容为下标1开始数,2个字符
auto it = B_str.begin();// type(it)==string::iterator
string G_str(it,it+3); // G_str=="123" 新对象字符数等于it+3-it,内容为it开始数,3个字符
```

与`C`风格字符串不同，`string`不带`'\0'`，在输入时也不需要考虑字符串的长度，`string`类会自动调整到刚刚好的空间，它所能容纳的长度为`string::npos=-1`(无符号整型的最大值)，在不同位数处理器下会有不同，可能是`unsigned long long`或`unsigned int`的最大值

这些构造函数使得需要创建子串对象时只需要知道起始点和子串字符数，就能很快算出参数

关于复制构造函数和移动构造函数的区别，以一个临时对象为例，假设还需要使用这个对象的内容，在被析构时，如果使用复制构造函数转移内容，需要：

1. 申请新的堆内存
2. 复制粘贴临时对象的内容
3. 释放临时对象的堆内存

而移动构造函数只需要将临时对象的堆内存转移给新对象，这个`ctor`通常由编译器自行选择使用，以优化性能，如果手动使用，需要注意它不保证`const`，可能造成问题

## 方法

### 重载运算符

关于部分重载运算符如`=,+=,==,[],<<,>>`等，较容易理解，不赘述

关于比较运算符`>,<,>=,<=`，它们调用的方法类似`C`中的`strcmp()`，是比较字符的`ASCII`码大小而非长度

`string`类没有定义`-=,-(),-(other)`运算符

需要注意用`[]`访问时，编译时不会抛出异常，即越界时有可能发生运行时错误

### 遍历访问与转换

使用`[]`时，完全靠自己谨慎来保证不出错，而`string`类提供`at(int)`方法，在越界时，会抛出`out_of_range`异常，这意味着可以`try_catch`处理它而非立刻终止程序(这里的越界指的是提供的参数大于等于`size()`，因为`string`尾部不含`'\0'`)

关于`length()`和`size()`方法，它们功能一样，都**返回总字符数**，`size()`的出现是为了配合`STL`容器要求的统一名称

所有`STL`都提供便于遍历的类似指针的迭代器，通常用以下方法获取`string`迭代器：

- `begin()`：返回指向**开头字符**的正向迭代器
- `end()`：返回指向**结尾字符的下一个位置**的正向迭代器
- `rbegin()`：返回指向**结尾字符**的**反向**迭代器(`reserve begin()`)
- `rend()`：返回指向**开头字符的上一个位置**的**反向**迭代器

在这些方法前加上`'c'`(例如`cbegin()`)，将返回**只读**迭代器，用于处理`const`修饰的对象，但事实上，这四个方法都有对应的重载版本，当该对象被`const`修饰时，自动使用这个版本返回只读迭代器，所以这个`'c'`可加可不加

那么遍历变得很简单，而不需要用`[]`或`at()`访问字符：

```c++
for (auto it = str.begin();it != str.end();it++){
    cout << *it;
}
```

在`std`中，`transform()`方法能够快速遍历`STL`容器并应用所提供的函数或函数指针：

```c++
// 定义:InIt指InputIterator,OutIt指OutputIterator
// 提供一元函数将[Ibegin,Iend)范围内元素都通过所提供的op,并存储在从Obegin开始的对象里
transform(InIt Ibegin,InIt Iend,OutIt Obegin,UnaryOperation);
// 提供二元函数将[Ibegin_1,Iend)和[Ibegin_2,Iend)范围内元素通过op,存储在从Obegin开始的对象里
transform(InIt Ibegin_1,InIt Iend,InIt Ibegin_2,OutIt Obegin,BinaryOperation);

// 示例
string a("abcdefg");
transform(a.begin(),a.end(),a.begin(),::toupper);// 将a的所有小写字母都变成大写,存到a里
```

### 增删改查

- **`insert()`**：用于前插入

  ```c++
  // 定义
  insert(uint idx,string&); // 将提供的string插到str[idx]前
  insert(uint idx,const char*);// 提供C风格字符串,会删除\0
  // 提供string引用,将它从下标idx_2开始数n个字符的子串插到str[idx_1]前
  // (经测试,虽然没有char*的函数重载,但传入char*是可行的)
  insert(uint idx_1,string&,uint idx_2,uint n);
  // 相当于上一条中,idx_2=0
  insert(uint idx,const char*,uint n);// 提供string&是不行的
  ```

- `append()`：用于后插入，完全可以被`operator+()`和`insert()`替代
- **`erase()`**：删除某字符，自动调整空间
