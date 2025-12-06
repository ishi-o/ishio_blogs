---
title: "算法竞赛: 常数优化之快读快写"
categories: [Algorithm, Trick]
tags: [algorithm]
mathjax: false
date: 2025-04-07
---
<!-- placeholder -->
<!-- more -->
### 快读快写

- 快读有两套模板：

  - 使用内置`getchar()`

    ```c++
    int read() {
        int x = 0, sig = 1;
        char ch = getchar();
        while (!isdigit(ch)) {
            if (ch == '-') {
                sig = -1;
            }
            ch = getchar();
        }
        while (isdigit(ch)) {
            x = x * 10 + (ch - '0');
            ch = getchar();
        }
        return sig * x;
    }
    ```

  - 使用文件`fread()`加速`getchar()`：此后不能再通过终端输入数据，但在`OI`是可行的因为输入数据都是文件

    ```c++
    // 即, 一次性将文件所有内容读入buf, 然后让p1(文件开头)递增到p2(文件结尾)处, 每次调用递增一次并返回字符
    // 这里的判断条件分为两部分
    // 1: p1 == p2   初始状态时, 会立刻执行条件2读入文件
    // 2: p2 = (p1 = buf) + fread(), p1 == p2 执行后, p1在文件开头, p2在文件结尾
    // 读入后, 若p1 == p2则立刻返回EOF, 否则返回第一个字符, p1递增
    // 此后调用gc(), 除非到达文件结尾, 否则不会进入2, 直接返回*p1后递增
    // 到达文件结尾后, 由于stdin没有字符了, 这里调用fread()也没有变化, p1 == p2 == buf
    // fread()最多调用2次, 相当于将文本一次性读到缓冲区
    #define gc() (p1 == p2 && (p2 = (p1 = buf) + fread(buf, 1, 1 << 20, stdin), p1 == p2) ? EOF : *p1++)
    char buf[1 << 23], *p1 = buf, *p2 = buf;
    int read() {
        int x = 0, sig = 1;
        char ch = gc();
        while (!isdigit(ch)) {
            if (ch == '-') {
                sig = -1;
            }
            ch = gc();
        }
        while (isdigit(ch)) {
            x = x * 10 + (ch - '0');
            ch = getchar();
        }
        return sig * x;
    }

- 快写：

  ```c++
  void write(long long x) {
      if (x < 0) {
          putchar('-');
          x = -x;
      }
      if (x > 9) {
          write(x / 10);
      }
      putchar(x % 10 + '0');
  }
