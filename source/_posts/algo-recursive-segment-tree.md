---
title: "数据结构: 递归式线段树"
categories: [Algorithm, Data Structure]
tags: [algorithm, data structure, tree]
mathjax: true
date: 2025-04-10
---
<!-- placeholder -->
<!-- more -->
## 递归式线段树

### 概述

- 线段树基于**分治思想**，本质是将分治过程中产生的**子区间解存储在树中**，以达到**快速查询**的效果

  典型如求解**最大连续子数组**，虽然运用贪心算法是更快的单次算法，但如果大区间不变，而只是进行**多次的子区间解查询**时，线段树比贪心算法更快

  若加上简单的维护，则能使其很简单地支持区间修改，同时不用消耗更多的资源

- 适用的修改操作：所有符合结合律的运算，例如比大小、求和等
  这也是和树状数组的区别，**树状数组**是**维护所有前缀区间**的信息来查询任意子区间的信息，而线段树是直接**维护一些有代表性的子区间**的信息并将它们**组合**来得到任意子区间的信息，因此不需要运算的元素拥有逆元

- 适用问题：要求维护**连续子区间**的问题，例如`RMQ`(区间最大/最小查询)和`RSQ`(区间求和查询)问题

### 递归式线段树的组成

- 基本结构：

  - 假设根结点编号为$1$，则所有结点$i$维护一个子区间$[l,r]$的信息，其左儿子$2i$维护$\begin{align}\left[l,\left\lfloor\frac{l+r}{2}\right\rfloor\right]\end{align}$，右儿子$2i+1$维护另一半区间
  - 叶子结点维护长度为一的区间信息，它们没有儿子
  - 递归式线段树不存在度为$1$的结点，且**是一棵平衡二叉树**
    但它**不一定是完全二叉树**，不过没什么影响就是了

- 建树：

  - 分治过程通常通过递归实现：如果结点的区间长度不为$1$，则进行上述递归，最后合并左、右儿子返回的值作为本结点的值，然后返回给父结点
    否则，直接返回这个长度为$1$的区间的值(即原数组中对应的值)
  - 空间消耗：有用的叶子结点即$n$个，同时存在着无用的叶子结点，在**不存在无用的叶子**结点时，**深度达到最大**，为$\left\lceil\log n\right\rceil$，则以最大深度为深度的**满二叉树数组**一定可以满足下标的分配，即**空间为$2^{\lceil\log n\rceil+1}-1$**
    一个容易记的形式是，因为$\lceil\log n\rceil\le\lfloor\log n\rfloor+1$，因此最大空间$<2^{\lfloor\log n\rfloor+2}<4·2^{\log n}=4n$，直接取$4n$即可

- 区间操作：

  - 区间查询：同样通过分治来得到答案，不过一部分分治不需要深入到叶子结点
    将待查询区间分为两种：完全覆盖当前结点维护的区间(即当前结点区间是待查询区间的子集)、以及另一种区间
    对于前者，不需要继续递归，直接返回维护的值即可
    对于后者，分为向左儿子递归和向右儿子递归的部分，如果查询区间的左端小于结点区间的中间点，则向左儿子递归；向右儿子递归同理，需要查询区间的右端大于结点区间的中间点；人话就是保证递归下去的区间和待查区间有交集
    最后合并来自各个结点的极大区间信息即可
    可以证明，这些极大区间最多有$O(\log n)$个，即区间查询操作是$O(\log n)$的
  - 区间修改：需要维护一个**懒惰标记**，**类似写时拷贝**的概念，取若干个构成待修改区间的结点进行修改，需要修改时将结点的`lazy`置`1`，只修改这个结点的值，但不去修改子结点的值
    具体来说，将这次修改看作区间查询，将待修改区间分为**若干个极大的子区间**，所有被置`lazy`的区间都是待修改区间的极大子集(同时告知需要增加/减去的值)，对同一个子区间的多次连续修改只需要将值叠加即可；当然，最后需要像合并更新上面的结点
    直到下一次查询，如果待查询区间不完全覆盖当前区间(即**需要递归时**)，则要将当前结点的**懒惰标记下放**以保证同步
    注意这里的**懒惰标记并非表示本结点未修改，而是标记子结点未修改**
    为了**防止懒惰标记溢出**，通常在区间修改时也需要下放标记

- 不同操作的线段树：

  - 区间内所有值赋为`k`：懒惰标记初始化为输入范围外的`k`来表示没有赋值
  - 区间内所有值加`k`：懒惰标记初始化为`0`
  - 区间内所有值乘`k`：懒惰标记初始化为`1`
  - 三种操作结合：赋值操作是优先级最高的，乘法操作会影响以前的加法操作，加法操作对其他操作没什么影响
    - 对于极大子集(即不着急下放标记的那个分支)：
      若为赋值，则应初始化乘法标记、加法标记，因为它们没有意义了
      若为乘法，则让乘法标记乘上`k`，同时会影响以前的加法操作，需要使加法标记乘上`k`
      若为加法，直接让加法标记加上`k`即可
    - 对于下放操作：
      若赋值标记存在，则应首先下放，同时初始化左右儿子的乘法、加法标记
      由于乘法标记、加法标记在初始状态下放时，不会对信息造成影响，所以不需要判断是否存在这些标记；且加法标记在之前已经经过处理，可以直接加到左右儿子区间信息上
      拿维护最大值举例，下放乘法、加法标记时，左右儿子的最大值需乘上`p`的乘法标记、再加上`p`的加法标记；左右儿子的乘法标记直接乘以`p`的乘法标记；左右儿子的加法标记先乘上`p`的乘法标记、再加上`p`的加法标记；最后初始化`p`的乘法、加法标记

- 区间加、区间求和的模板实现：

  ```c++
  #include <bits/stdc++.h>
  using namespace std;
  int n;
  long long b[100010];
  struct Info {
      int l, r, s, lz; // s表sum
  }t[100010 << 2];
  void build(int p, int l, int r) {
      t[p].l = l; // 初始化结点p管辖的区间
      t[p].r = r;
      if (l == r) { // 到叶子结点, 可以初始化了
          t[p].ans = b[l];
          return;
      }
      build(p << 1, l, l + r >> 1); // 左边
      bulid(p << 1 | 1, (l + r >> 1) + 1, r); // 右边
      t[p].s = t[p << 1].s + t[p << 1 | 1].s;  // 可以将合并操作单独写一个函数
  }
  void pushdown(int p) {
      if (t[p].lz) { // 下放标记
          t[p << 1].s += (t[p << 1].r - t[p << 1].l + 1) * t[p].lz;
          t[p << 1 | 1].s += (t[p << 1 | 1].r - t[p << 1 | 1].l + 1) * t[p].lz;
          t[p << 1].lz += t[p].lz;
          t[p << 1 | 1].lz += t[p].lz;
          t[p].lz = false;
      }
  }
  void update(int p, int l, int r, int v) {
   if (l <= t[p].l && r >= t[p].r) {
          t[p].s += (t[p].r - t[p].l + 1) * v;
          t[p].lz += v;
          return;
      }
      pushdown(p);
      int m = t[p].l + t[p].r >> 1;
      if (l <= m) update(p << 1, l, r, v);
      if (r > m) update(p << 1 | 1, l, r, v);
      t[p].s = t[p << 1].s + t[p << 1 | 1].s;
  }
  long long query(int p, int l, int r) {
      if (l <= t[p].l && r >= t[p].r) {
          return t[p].s;
      }
      pushdown(p);
      int m = t[p].l + t[p].r >> 1;
      long long ans = 0;
      if (l <= m) ans += query(p << 1, l, r);
      if (r > m) ans += query(p << 1 | 1, l, r);
      return ans;
  }
  ```

### 常见区间信息的设计

- 首先要判断一道题是否能用线段树来做：

  - 对题目要求的信息，是否满足**区间可加性**；即是否能由子区间的值求出父区间的值
  - 对要求区间修改的线段树，是否满足**懒惰标记可加性**和区间信息易改，即子结点的懒惰标记可以由父结点下放得到，而父结点可以快速求出整个区间进行某种修改后的值(而不需要递归)
    如果不能，看是否能降为暴力单点修改(适用于经过有限次修改后就不再改变的信息)

- 区间信息及懒惰标记的设计：

  - 设计区间信息取决于要求什么样的区间信息：
    通常要考虑到怎么通过维护某些**额外的信息**：使得**区间修改时，区间信息易改**
    一般是将修改前和修改后的待求信息进行对比，来得出额外信息：
    例如区间加+区间和：$\begin{align}s=\sum a_i、s'=\sum(a_i+d)\end{align}$，因此额外信息为**区间长度**
    区间加+区间平方和：$\begin{align}s=\sum a_i^2、s'=\sum(a_i+d)^2=s+2d\sum a_i+\sum d^2\end{align}$，因此额外信息为**区间和、区间长度**
  - 设计懒惰标记取决于要进行什么样的区间操作：
    懒惰标记要能唯一地标识两个信息，即子区间要不要改、子区间怎么改
    例如区间加就将上述两个信息合并，因为懒惰标记为$0$时，自然表示不用改
    其它如区间赋值，因为$0$不代表不用改，所以需要：一个布尔值标识要不要改，一个值表示要赋的值
    或将这个值**初始化设为范围外的值**，来表示不用改

- 一些常见的区间操作及它们的组合：

  - 区间赋值、区间加：不再赘述
  - 区间乘：下放乘标记时直接相乘即可，因为整数的乘法是半群；此外，乘法标记初始化为$1$

- 一些常见的区间信息：

  - 区间`hash`：区间哈希一般用于比较两个区间是否相等，用树状数组也可
    学过字符串哈希就知道，求一个区间的基本哈希值是将它看作$p$进制数，即
    $hash(a[1,n])=a_1p^{n-1}+a_2p^{n-2}+\cdots+a_{n-1}p+a_n$
    这其实有点像前缀和，只是多了一个$p$的幂的因子，通过简单推导可以得出：
    $hash(a[1,r])=hash(a[1,l-1])\times p^{r-l+1}+hash(a[l,r])$
    也就是：$hash(a[l,r])=hash(a[l,m])\times p^{r-m}+hash(a[m+1,r])$
    这里不考虑如何解决冲突，选择一个**大质数**作为模数和**比$max(a)$大的$p$**即可
    再不济，就用双值哈希

- 特定区间的最大连续子区间和(所有连续子区间中，元素和最大的子区间)：

  ```c++
  #include <bits/stdc++.h>
  using namespace std;
  int i, n, m, k, a, b, gd[500005];
  struct Info {
      int lm, rm, am, ll, rr, sum;
  }tr[500005 << 2];
  void edit(Info& p, Info sl, Info sr) {
      p.lm = max(sl.lm, sl.sum + sr.lm);
      p.rm = max(sr.rm, sr.sum + sl.rm);
      p.sum = sl.sum + sr.sum;
      p.am = max(max(sl.am, sr.am), (sl.rm < 0 && sr.lm) ? max(sl.rm, sr.lm) : max(0, sl.rm) + max(0, sr.lm));
  }
  void build (int p, int l, int r) {
      tr[p].ll = l;
      tr[p].rr = r;
      if (l == r) {
          tr[p].lm = tr[p].rm = tr[p].am = tr[p].sum = gd[l];
          return;
      }
      int m = (l + r) >> 1;
      build(p << 1, l, m);
      build(p << 1 | 1, m + 1, r);
      edit(tr[p], tr[p << 1], tr[p << 1 | 1]);
  }
  void update(int p, int t, int v) { // 单点修改
      if (tr[p].ll == tr[p].rr && tr[p].ll == t) {
          tr[p].lm = tr[p].rm = tr[p].am = tr[p].sum = v;
          return;
      }
      int m = tr[p].ll + tr[p].rr >> 1;
      if (t <= m) {
          update(p << 1, t, v);
      }
      else {
          update(p << 1 | 1, t, v);
      }
      edit(tr[p], tr[p << 1], tr[p << 1 | 1]);
  }
  Info query(int p, int l, int r) {
      if (l <= tr[p].ll && r >= tr[p].rr) {
          return tr[p];
      }
      int m = tr[p].ll + tr[p].rr >> 1;
      if (l <= m && r > m) {
          Info ans;
          edit(ans, query(p << 1, l, r), query(p << 1 | 1, l, r));
          return ans;
      }
      else if (l <= m) {
          return query(p << 1, l, r);
      }
      else {
          return query(p << 1 | 1, l, r);
      }
  }

较复杂线段树：

```c++
#include <bits/stdc++.h>
using namespace std;
int n, m, i, opt, L, R;
bool a[100006];
struct Info{
    bool fil, filv, swt;
    int l, r, lm[2], rm[2], am[2], sum, len;
    Info() {
        fil = swt = lm[0] = lm[1] = rm[0] = rm[1] = am[0] = am[1] = sum = len = 0;
    }
}tr[100006 << 2];
void merge(Info& p, Info l, Info r) {
    p.lm[0] = l.lm[0] + (l.lm[0] == l.len ? r.lm[0] : 0);
    p.lm[1] = l.lm[1] + (l.lm[1] == l.len ? r.lm[1] : 0);
    p.rm[0] = r.rm[0] + (r.rm[0] == r.len ? l.rm[0] : 0);
    p.rm[1] = r.rm[1] + (r.rm[1] == r.len ? l.rm[1] : 0);
    p.am[0] = max(l.rm[0] + r.lm[0], max(l.am[0], r.am[0]));
    p.am[1] = max(l.rm[1] + r.lm[1], max(l.am[1], r.am[1]));
    p.len = l.len + r.len;
    p.sum = l.sum + r.sum;
}
void build(int p, int l, int r) {
    tr[p].l = l;
    tr[p].r = r;
    if (l == r) {
        tr[p].len = 1;
        tr[p].lm[0] = tr[p].rm[0] = tr[p].am[0] = !a[l];
        tr[p].lm[1] = tr[p].rm[1] = tr[p].am[1] = tr[p].sum = a[l];
        return;
    }
    build(p << 1, l, l + r >> 1);
    build(p << 1 | 1, (l + r >> 1) + 1, r);
    merge(tr[p], tr[p << 1], tr[p << 1 | 1]);
}
void pushdown(int p) {
    if (tr[p].fil) {
        tr[p << 1].fil = tr[p << 1 | 1].fil = true;
        tr[p << 1].swt = tr[p << 1 | 1].swt = false;
        tr[p << 1].filv = tr[p << 1 | 1].filv = tr[p].filv;
        tr[p << 1].lm[tr[p].filv] = tr[p << 1].rm[tr[p].filv] = tr[p << 1].am[tr[p].filv] = tr[p << 1].len;
        tr[p << 1].lm[!tr[p].filv] = tr[p << 1].rm[!tr[p].filv] = tr[p << 1].am[!tr[p].filv] = 0;
        tr[p << 1].sum = tr[p].filv ? tr[p << 1].len : 0;
        tr[p << 1 | 1].lm[tr[p].filv] = tr[p << 1 | 1].rm[tr[p].filv] = tr[p << 1 | 1].am[tr[p].filv] = tr[p << 1 | 1].len;
        tr[p << 1 | 1].lm[!tr[p].filv] = tr[p << 1 | 1].rm[!tr[p].filv] = tr[p << 1 | 1].am[!tr[p].filv] = 0;
        tr[p << 1 | 1].sum = tr[p].filv ? tr[p << 1 | 1].len : 0;
        tr[p].fil = false;
    }
    if (tr[p].swt) {
        tr[p << 1].swt = !tr[p << 1].swt;
        swap(tr[p << 1].am[0], tr[p << 1].am[1]);
        swap(tr[p << 1].rm[0], tr[p << 1].rm[1]);
        swap(tr[p << 1].lm[0], tr[p << 1].lm[1]);
        tr[p << 1].sum = tr[p << 1].len - tr[p << 1].sum;
        tr[p << 1 | 1].swt = !tr[p << 1 | 1].swt;
        swap(tr[p << 1 | 1].am[0], tr[p << 1 | 1].am[1]);
        swap(tr[p << 1 | 1].rm[0], tr[p << 1 | 1].rm[1]);
        swap(tr[p << 1 | 1].lm[0], tr[p << 1 | 1].lm[1]);
        tr[p << 1 | 1].sum = tr[p << 1 | 1].len - tr[p << 1 | 1].sum;
        tr[p].swt = false;
    }
}
void update(int p, int l, int r, int mode) {
    if (l <= tr[p].l && r >= tr[p].r) {
        switch (mode) {
            case 0: case 1:
                tr[p].fil = true;
                tr[p].filv = mode;
                tr[p].lm[mode] = tr[p].rm[mode] = tr[p].am[mode] = tr[p].len;
                tr[p].lm[!mode] = tr[p].rm[!mode] = tr[p].am[!mode] = 0;
                tr[p].sum = mode ? tr[p].len : 0;
                tr[p].swt = false;
            break;
            case 2:
                tr[p].swt = !tr[p].swt;
                swap(tr[p].am[0], tr[p].am[1]);
                swap(tr[p].rm[0], tr[p].rm[1]);
                swap(tr[p].lm[0], tr[p].lm[1]);
                tr[p].sum = tr[p].len - tr[p].sum;
            break;
        }
        return;
    }
    pushdown(p);
    int m = tr[p].l + tr[p].r >> 1;
    if (l <= m) {
        update(p << 1, l, r, mode);
    }
    if (r > m) {
        update(p << 1 | 1, l, r, mode);
    }
    merge(tr[p], tr[p << 1], tr[p << 1 | 1]);
}
Info query(int p, int l, int r) {
    if (l <= tr[p].l && r >= tr[p].r) {
        return tr[p];
    }
    pushdown(p);
    int m = tr[p].l + tr[p].r >> 1;
    Info ans1, ans2, ans;
    if (l <= m) {
        ans1 = query(p << 1, l, r);
    }
    if (r > m) {
        ans2 = query(p << 1 | 1, l, r);
    }
    merge(ans, ans1, ans2);
    return ans;
}
int main() {
    cin >> n >> m;
    for (i = 1; i <= n; ++i) {
        cin >> a[i];
    }
    build(1, 1, n);
    while (m--) {
        cin >> opt >> L >> R;
        switch (opt) {
            case 0: case 1: case 2:
                update(1, L + 1, R + 1, opt);
            break;
            case 3:
                cout << query(1, L + 1, R + 1).sum << endl;
            break;
            case 4:
                cout << query(1, L + 1, R + 1).am[1] << endl;
            break;
        }
    }
}
```

### 动态开点线段树

- 使用`p << 1`和`p << 1 | 1`作为结点的左右儿子编号，显然是很浪费空间的(因为部分结点可能从不会被读、被写)
  所以一个自然的优化是，用我们维护的`cnt`记录当前的总结点数，然后需要创建新的结点时(即需要访问从未维护过的子区间时)将`cnt+1`作为其编号，并找到有直接父子关系的结点，维护其`ls`或`rs`(就像存储二叉树那样)
  这样的话，通常只需要$4m\log n$($m$为查询次数)的空间，如果$m\le10^5、n\le10^9$，那么大概需要$10^7$空间；当然，如果$n$较小，那是没有必要用动态开点的(除非要实现更复杂的数据结构)
  需要说明，区间信息必须**能快速地初始化**，否则如果要依赖子结点的区间信息，那么用动态开点就没有意义了
  
- 代码实现：和一般线段树相比，不需要建树，只多了一步判断是否需要新增结点的操作
  很多参数的设计是需要根据题意修改的，这里仅提供一个区间加、求区间和的模板

  ```c++
  int n, cnt, rt;  // 以1作为根结点
  long long pre[MAXN]; // 假设已经预处理好了前缀和, 注意这样无法处理n过大的情况
  const int LEN = 适合的大小;
  int ls[LEN], rs[LEN];
  struct Info{
      long long s, lz; // 因为无法事先初始化, 所以结点就不存l, r了
  }t[LEN];
  void init(int& p, int l, int r) {
      p = ++cnt;
   t[p].s = pre[r] - pre[l - 1];
  }
  void pushdown(int p, int l, int r) {
      if (t[p].lz) {
          int m = l + r >> 1;
          if (!ls[p]) init(ls[p], l, m);  // 子结点未创建
          if (!rs[p]) init(rs[p], m + 1, r);
          t[ls[p]].s += t[p].lz * (m - l + 1);
          t[rs[p]].s += t[p].lz * (r - m);
          t[p].lz = 0;
      }
  }
  void update(int& p, int l, int r, int L, int R, int v) {
      if (!p) init(p, L, R);
      if (l <= L && r >= R) {
          t[p].s += v * (R - L + 1);
          t[p].lz += v;
          return;
      }
      pushdown(p, L, R);
      int m = L + R >> 1;
      if (l <= m) update(ls[p], l, r, L, m, v);
      if (r > m) update(rs[p], l, r, m + 1, R, v);
      t[p].s = t[ls[p]].s + t[rs[p]].s;
  }
  void query(int& p, int l, int r, int L, int R) {
      if (!p) init(p, L, R);
      if (l <= L && r >= R) {
          return t[p].s;
      }
      pushdown(p, L, R);
      int m = L + R >> 1;
   long long ans = 0;
      if (l <= m) ans += query(ls[p], l, r, L, m);
      if (r > m) ans += query(rs[p], l, r, m + 1, R);
      return ans;
  }
