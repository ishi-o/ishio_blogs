---
title: "树的算法: LCA（最近公共祖先）"
categories: [Algorithm, Tree Algo]
tags: [algorithm, LCA]
mathjax: true
date: 2025-04-30
---
<!-- placeholder -->
<!-- more -->
## `LCA`算法

### `LCA()`性质

- `LCA(Lowest Common Ancestor)`，即最小公共祖先，$LCA()$接受一个结点集合，返回集合中所有结点的最小公共祖先结点
- $LCA(\{u\})=u$，即任意结点的`LCA`是自身
- $LCA(\{u,v\})=u\Leftrightarrow u是v的祖先$
- $LCA(S_1\cup S_2)=LCA(\{\ LCA(S_1),\ LCA(S_2)\ \})$
- 给定结点集合$S\subseteq V$，则$LCA(S)$在前序遍历中位于集合中所有结点之前，在后序遍历中位于所有结点之后
- $LCA({u,v})$必然在$u,v$最短路上，且$d(u,v)=h(u)+h(v)-2h(LCA(\{u,v\}))$(无权树)，有权树只需额外添加权重即可

### 朴素算法

很容易想到，当两结点处于同一深度时，若两结点相同，则该结点为所求

否则，目标结点为$LCA(直接祖宗结点)$

由此，需要先进行预处理，使两结点达到同一深度，即让深度较大的结点上移(经过的结点不可能是所求)

这样的算法要求结点结构能够访问父结点

```c++
struct node {
    int h, fa;
};
node tree[LEN]; // 以 下标 为结点编号
int lca(int u, int v) {
    int uh = tree[u].h, vh = tree[v].h; // 存储两结点的深度方便最短路计算
 if (uh < vh) {
        swap(u, v);
    }
    while (tree[u].h > tree[v].h) {
        u = tree[u].fa;
    }
    while (u != v) {
        u = tree[u].fa;
        v = tree[v].fa;
    }
    // uh + vh - 2 * tree[u].h   即为无权树两点间最短路
    return u;
}
```

时间复杂度：单次查询时，时间为$O(n)$

### 倍增算法

不难发现对于共同祖先这个性质，$LCA(\{u,v\})$的祖先一定是$u,v$的共同祖先，它的孩子一定不是$u,v$的共同祖先；回顾二分思想，若通过一个性质能对可能解集划分为两部分连续的区间，就能二分搜索出最优/最大/最小的具有该性质的解

倍增算法就是二分思想优化的朴素算法，在朴素算法中，只能一级一级跳，单次查询时间为$O(n)$

而倍增算法通过给每个结点多分配$O(\log h)$空间来存储比它高$2^j$处的祖先结点，达到倍增跳跃的效果

类比朴素算法，分为两步：

- 两结点先达到同一深度：先算出深度差，根据其二进制码决定在何时跳、跳多远
- 两结点共同跳跃，根据解的二分性质，取起始点为$\begin{cases}left=0\\right=h(u)\\mid=(left+right)>>1\end{cases}$

重申倍增算法和朴素算法的区别：

- 预处理时，每个结点额外消耗$O(\log h)$的时间和空间
- 查询目标方式使用二分查找，时间复杂度为$O(\log n)$
- 为了能快速到达目标，使用倍增祖先数组，使每次查询距离深度$h$的祖先的时间复杂度为$O(\log h)$
- 因此整个算法的预处理时间复杂度$O(n\log n)$，单次查询$O(\log n)$

```c++
int i, n, m, s, son, fa, f[500001][18];
struct node {
    int h, cnt, next[2];
};
node tree[500001];
void dfs() {
    int i, j;
    stack<int> stk;
    tree[s].h = 1;
    stk.push(s);
    while (!stk.empty()) {
        i = stk.top();
        stk.pop();
        for (j = tree[i].cnt - 1; j >= 0; --j) {
            stk.push(tree[i].next[j]);
            f[tree[i].next[j]][0] = i;
            tree[tree[i].next[j]].h = tree[i].h + 1;
        }
        j = son = 1;
        while (son < tree[i].h) {
            f[i][j] = f[f[i][j - 1]][j - 1];
            ++j;
            son <<= 1;
        }
    }
}
int jmp(int u, int h) {
    int i = 0;
    while (h) {
        if (h & 1) {
            u = f[u][i];
        }
        ++i;
        h >>= 1;
    }
    return u;
}
int lca(int u, int v) {
    if (tree[u].h < tree[v].h) {
        swap(u, v);
    }
    u = jmp(u, tree[u].h - tree[v].h);
    int left = 0, right = tree[u].h, mid;
    while (left < right) {
        mid = (left + right) >> 1;
        if (jmp(u, mid) == jmp(v, mid)) {
            right = mid - 1;
        }
        else {
            left = mid + 1;
        }
    }
    right = jmp(u, left);
    left = jmp(v, left);
    return left == right ? left : f[left][0];
}
int main() {
    cin >> n >> m >> s;
    queue<int> ans;
    for (i = 1; i < n; ++i) {
        cin >> son >> fa;
        tree[fa].next[tree[fa].cnt++] = son;
    }
    dfs();
    for (i = 0; i < m; ++i) {
        cin >> son >> fa;
        ans.push(lca(son, fa));
    }
    while (!ans.empty()) {
        cout << ans.front() << endl;
        ans.pop();
    }
}
```
