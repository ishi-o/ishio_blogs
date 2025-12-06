---
title: "字符串算法: KMP匹配"
categories: [Algorithm, String Algo]
tags: [algorithm, string, match]
mathjax: true
date: 2024-10-23
---
<!-- placeholder -->
<!-- more -->
## `KMP`算法

### 暴力算法的不足

假设主串为`T`，模式串为`P`，要找出第一个/共有多少个`P`子串，最简单的办法是暴力搜索

```c++
int indexOf(const string& t, const string& p) {
    int i = 0, j = 0, tl = t.length(), pl = p.length();
 while (i < tl && j < pl) {
        if (t[i] == p[j]) {
            ++i;
            ++j;
        }
        else {
            i = i - j + 1;
            j = 0;
        }
    }
    return j == pl ? i - j : -1;
}
```

容易算出，该回溯算法的时间复杂度为`O(pl*tl)`，算法慢在每次的回溯点为`i-j+1`，而事实上若能找到一个更合适的位置回溯，就能减少重复比较

### 同一个串的前缀和后缀

定义一个字符串的前缀为`s[0:i),i<end`，即在末尾至少删除一个字符后剩下的子串

后缀同理，为在开头至少删除一个字符后剩下的子串；规定两种子串不为空串

例如字符串`abcd`的前缀集合为`{a,ab,abc}`，后缀集合为`{d,cd,bcd}`

如果同一个串的某一前缀和某一后缀相匹配，就可以将前缀平移至后缀，如$\begin{align}&abc\underline {ab}\\&\ \ \ \ \ \ \underline {ab}cab\end{align}$(前后缀$ab$匹配)

`KMP`算法的核心思想就是**同一个串的前后缀匹配**，实现**忽略**掉**匹配失败部分**的效果

将上方看作主串`T=="abcabeeee"`的子串，下方看作模式串`P=="abcabf"`

在暴力算法中，当遍历到$\begin{align}&abcab\underline eeee\\&abcab\underline f\end{align}$的最后一个字符`f`时，需要回溯到$\begin{align}&a\underline bcabeeee\\&\ \ \underline abcabf\end{align}$

但如果应用上前后缀匹配，**已匹配串**为$abcab$，前后缀$ab$匹配，直接回溯到$\begin{align}&abc\textcolor{red}{ab}\underline eeee\\&\ \ \ \ \ \ \textcolor{red}{ab}\underline cabf\end{align}$

实际情况中，需要取**最长**公共前后缀(取最长，匹配失败时仍可以继续遍历较短的情况；而不取最长，就会漏掉最长的情况)

假设提前知道了所有子串的最长公共前后缀的长度，可将暴力算法改写成：

```c++
int indexOf(const string& t, const string& p) {
    int i, j;
    while (i < tl && j < pl) {
        if (t[i] == p[j]) {
            ++i;
            ++j;
        }
        else {
            if (j == 0) { // j回溯到0, 且和i不匹配, 则i肯定不是答案
                ++i;
            }
            j = 子串的最长公共前后缀的长度; // 直接回溯到最长前缀子串的后一个字符
        }
    }
    return j == pl ? i - j : -1;
}
```

该过程时间复杂度为`O(tl)`

### 求出每个前缀子串的最长公共前后缀长

要使用上述算法，需要提前知道**所有子串`p[0:j)`的**最长公共前后缀长，即预处理

这里的子串刚好是模式串的前缀子串，共有`pl`个(长度从零到`pl-1`)，故用`next[pl]`数组存储

`KMP`算法的一大妙处为快速求出子串的公共前后缀长，实现`O(pl)`的预处理时间

核心为模式串的子串**自己和自己匹配**，对于子串`s=p[0:j)`，新主串为`s[end-k:end)`，新模式串为`s[0,k)`

这里用到动态规划思想(`KMP`算法的精髓)，回溯时要用到已求的最长公共前后缀长

基础情况为`next[0]=next[1]=0`(长度为`0`和`1`的字符串，没有前后缀子串，故长度为零)

```c++
int indexOf(const string& t, const string& p) {
    ...
    int next[pl] {};// 除next[0,1]==0外, 其它也置为0, 没有公共前后缀时就不需要修改
    i = 1;
    j = 0;
    while (i < pl - 1) { // 遍历剩下的前缀子串, 因为j一定小于i, 循环条件简化
        if (p[i] == p[j]) {
            next[++i] = ++j; // 数组索引为子串长度, 值为该子串的最长公共前后缀的长度
        }
        else {
            if (j == 0) {
                ++i;
            }
            j = next[j]; // 回溯到已求出的最长公共前后缀长
        }
    }
    ...
}
```

这里为了代码简洁，将下标和长度混用了

事实上整部分就是主算法的翻版，只不过需要存储最长公共前后缀长、主串偏移一个字符，可以说会写主算法就会写预处理

### 最终代码

```c++
int indexOf(const string& t, const string& p) {
    static int next[10000]; // 实际为了更快，不采用变长数组而是静态定长数组
    int i = 1, j = 0, tl = t.length(), pl = p.length();
    while (i < pl - 1) {// 如果需要算出p自身的最长前后缀长, 这里不用减1, 然而该算法不需要该信息
        if (p[i] == p[j]) {
            next[++i] = ++j;
        }
        else {
            if (j == 0) {
                next[++i] = 0; // 采用静态数组, 因此需手动置0
            }
            j = next[j];
        }
    }
    i = j = 0;
    while (i < tl && j < pl) {
        if (t[i] == p[j]) {
            ++i;
            ++j;
        }
        else {
            if (j == 0) {
                ++i;
            }
            j = next[j];
        }
        // if (j == pl) { // 如果需要求出所有匹配成功的p串, 在j满时j回溯
        //    // 回溯前记录i - j
        //     j = next[j]; // while条件可删去j < pl
        // }
    }
    return j == pl ? i - j : -1;
}
```
