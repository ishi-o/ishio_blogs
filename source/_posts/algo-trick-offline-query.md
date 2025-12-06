---
title: "算法竞赛: 离线查询"
categories: [Algorithm, Trick]
tags: [algorithm, trick]
mathjax: true
date: 2024-11-26
---
<!-- placeholder -->
<!-- more -->
### 离线查询

- 离线查询，即将题目问的多组问题缓存下来，再进行一定顺序的求解
  通常用于，一些查询的**结果是另一些查询结果的一部分**的情况；甚至于**如果先求解后者，反而很难得到前者**的情况
  具有这些性质的查询，在离线下来后通常要按某个查询属性进行排序，然后迭代求解
  它类似一个小技巧，很难总结出一些模板，或者方法论，所以下面给出一些例题帮助理解：

#### [[GZOI2017] 配对统计](https://www.luogu.com.cn/problem/P5677)

- 问题可以抽象成，给你一堆二元组$[x,y],x<y$(原题通过排序可以轻松求出所有符合要求的数对)，你需要在每次**区间$[l,r]$查询**后，给出满足$l\le x,y\le r$的数对的个数
  - 由于是区间查询，考虑用树状数组或线段树；在本题，用权值树状数组$tr[i]$维护左值$x==i$的数对个数；但是，树状数组只能保证维护左值在区间当中，不关心数对的右值
  - 因此通过离线查询，对所有二元组、待查询区间的**右值/右端点大小**进行排序，因此查询区间的右端点是单调递增的；对每个查询，将所有右值不超过该查询区间右端点的数对插入到树状数组中
    这样，树状数组中所有数对对应的右值都不会超过当前查询的右端点$r$
    那么如何使数对的左值处于当前查询区间中呢？通过权值树状数组查询$[l,r]$元素和即可(即左值在$[l,r]$区间内的数对个数)；同时由于$y>x$，因此也满足了使数对的右值不小于左端点$l$

- ```c++
  #define lbit(x) (x - (x & (x - 1)))
  #include <bits/stdc++.h>
  using namespace std;
  int n, m, tr[300006], cnt;
  long long ans;
  void add(int x) {
      while (x <= n) {
          ++tr[x];
          x += lbit(x);
      }
  }
  int query(int x) {
      int ans = 0;
      while (x) {
          ans += tr[x];
          x -= lbit(x);
      }
      return ans;
  }
  int query(int l, int r) {
      return query(r) - query(l - 1);
  }
  struct Info{
      int a, id;
      bool operator<(const Info& oth) const {
          return a < oth.a;
      }
  }a[300006];
  struct Pair{
      int l, r, id;
      bool operator<(const Pair& oth) const {
          return r < oth.r;
      }
  }p[300006], goodp[300006 << 1];
  int main() {
      cin >> n >> m;
      if (n == 1) {
          cout << 0;
          return 0;
      }
      for (int i = 0; i < n; ++i) {
          cin >> a[i].a;
          a[i].id = i + 1;
      }
      sort(a, a + n);
      goodp[cnt++] = {min(a[0].id, a[1].id), max(a[0].id, a[1].id)};
      goodp[cnt++] = {min(a[n - 1].id, a[n - 2].id), max(a[n - 1].id, a[n - 2].id)};
      for (int i = 1; i < n - 1; ++i) {
          if (abs(a[i].a - a[i - 1].a) == abs(a[i].a - a[i + 1].a)) {
              goodp[cnt++] = {min(a[i].id, a[i + 1].id), max(a[i].id, a[i + 1].id)};
              goodp[cnt++] = {min(a[i].id, a[i - 1].id), max(a[i].id, a[i - 1].id)};
          }
          else if (abs(a[i].a - a[i - 1].a) < abs(a[i].a - a[i + 1].a)) {
              goodp[cnt++] = {min(a[i].id, a[i - 1].id), max(a[i].id, a[i - 1].id)};
          }
          else {
              goodp[cnt++] = {min(a[i].id, a[i + 1].id), max(a[i].id, a[i + 1].id)};
          }
      }
      sort(goodp, goodp + cnt);
      for (int i = 0; i < m; ++i) {
          cin >> p[i].l >> p[i].r;
          p[i].id = i + 1;
      }
      sort(p, p + m);
      int j = 0;
      for (int i = 0; i < m; ++i) {
          while (j < cnt && goodp[j].r <= p[i].r) {
              add(goodp[j].l);
              ++j;
          }
          ans += 1ll * query(p[i].l, p[i].r) * p[i].id;
      }
      cout << ans;
  }
