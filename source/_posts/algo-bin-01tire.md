---
title: "博弈论: 01字典树"
categories: [Algorithm, Trick]
tags: [algorithm, tree]
mathjax: false
date: 2025-02-21
---
<!-- placeholder -->
<!-- more -->
```c++
#include <bits/stdc++.h>
using namespace std;
int n, i, u, v, w, head[100002], cnt = 1, cnt01 = 1, ch[100002 * 32][2], ans, sum[100002];
struct Edge{
    int to, next, w;
}e[100002 << 1];
void addEdge(int from, int to, int ww) {
    e[cnt].to = to;
    e[cnt].w = ww;
    e[cnt].next = head[from];
    head[from] = cnt++;
}
void addWt(int x) {
    bool m;
    for (int k = 31, s = 0; k >= 0; --k) {
        m = x & (1 << k);
        if (!ch[s][m]) {
            ch[s][m] = cnt01++;
        }
        s = ch[s][m];
    }
}
void dfs(int p, int fa, int xorw) {
    sum[p] = xorw;
    addWt(xorw);
    for (int k = head[p]; k; k = e[k].next) {
        if (e[k].to != fa) {
            dfs(e[k].to, p, xorw ^ e[k].w);
        }
    }
}
void find(int x) {
    bool m;
    int nans = 0;
    for (int k = 31, s = 0; k >= 0; --k) {
        m = x & (1 << k);
        if (ch[s][!m]) {
            nans <<= 1;
            ++nans;
            s = ch[s][!m];
        }
        else {
            nans <<= 1;
            s = ch[s][m];
        }
    }
    ans = max(ans, nans);
}
int main() {
    cin >> n;
    for (i = 1; i < n; ++i) {
        cin >> u >> v >> w;
        addEdge(u, v, w);
        addEdge(v, u, w);
    }
    dfs(1, 0, 0);
    for (i = 2; i <= n; ++i) {
        find(sum[i]);
    }
    cout << ans << endl;
}
```
