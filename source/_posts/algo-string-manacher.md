---
title: "字符串算法: 马拉车（manacher）回文串统计"
categories: [Algorithm, String Algo]
tags: [algorithm, string, palindrome]
mathjax: true
date: 2025-04-26
---
<!-- placeholder -->
<!-- more -->
```c++
#include <bits/stdc++.h>
using namespace std;
int d[11000010 << 1], l = 0, r = -1, i, ans, n;
string t;
char s[11000010 << 1];
int main() {
    cin >> t;
    s[0] = '#';
    for (i = 0; i < t.size(); ++i) {
        s[(i << 1) + 1] = t[i];
        s[(i << 1) + 2] = '#';
    }
    n = (i << 1) + 3;
    for (i = 0; i < n; ++i) {
        if (i <= r) {
            d[i] = min(d[l + r - i], r - i);
        }
        while (i - d[i] >= 0 && i + d[i] < n && s[i - d[i]] == s[i + d[i]]) {
            ++d[i];
        }
        if (i + d[i] > r) {
            l = i - d[i];
            r = i + d[i];
        }
        ans = max(ans, d[i]);
    }
    cout << ans - 1 << endl;
}
```
