---
title: "图的算法: SCC（强连通分量）"
categories: [Algorithm, Graph Algo]
tags: [algorithm, graph]
mathjax: false
date: 2025-02-11
---
<!-- placeholder -->
<!-- more -->
```c++
#include <bits/stdc++.h>
using namespace std;
bool vis[10004];
int head[10004], cnt = 1, w[10004], sw[10004], i, n, m, u, v, dfn[10004], rt[10004], scecnt = 1, schead[10004], ncnt, aans[10004];
stack<int> stk;
struct Edge{
    int from, to, next;
}e[100004], se[100004];
void getScc(int p) {
    dfn[p] = rt[p] = ++ncnt;
    vis[p] = true;
    stk.push(p);
    for (int k = head[p]; k; k = e[k].next) {
        if (!dfn[e[k].to]) {
            getScc(e[k].to);
            rt[p] = min(rt[p], rt[e[k].to]);
        }
        else if (vis[e[k].to]) {
            rt[p] = min(rt[p], rt[e[k].to]);
        }
    }
    if (rt[p] == dfn[p]) {
        while (stk.top() != p) {
            rt[stk.top()] = dfn[p];
            vis[stk.top()] = false;
            sw[rt[p]] += w[stk.top()];
            stk.pop();
        }
        vis[p] = false;
        sw[rt[p]] += w[p];
        stk.pop();
    }
}
void getSccgraph() {
    for (int k = 1; k <= m; ++k) {
        if (rt[e[k].from] != rt[e[k].to]) {
            se[scecnt].to = rt[e[k].to];
            se[scecnt].next = schead[rt[e[k].from]];
            schead[rt[e[k].from]] = scecnt++;
        }
    }
}
int dfs(int p) {
    if (vis[p]) {
        return aans[p];
    }
    vis[p] = true;
    if (!schead[p]) {
        return aans[p] = sw[p];
    }
    int ans = 0;
    for (int k = schead[p]; k; k = se[k].next) {
        ans = max(ans, dfs(se[k].to) + sw[p]);
    }
    return aans[p] = ans;
}
int main() {
    cin >> n >> m;
    for (i = 1; i <= n; ++i) {
        cin >> w[i];
    }
    for (i = 1; i <= m; ++i) {
        cin >> u >> v;
        e[cnt].from = u;
        e[cnt].to = v;
        e[cnt].next = head[u];
        head[u] = cnt++;
    }
    for (i = 1; i <= n; ++i) {
        if (!dfn[i]) {
            getScc(i);
        }
    }
    getSccgraph();
    int realans = 0;
    for (i = 1; i <= n; ++i) {
        if (dfn[i] == rt[i]) {
            realans = max(realans, dfs(rt[i]));
        }
    }
    cout << realans << endl;
}
```
