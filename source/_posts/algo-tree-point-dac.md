---
title: "树的算法: 点分治"
categories: [Algorithm, Tree Algo]
tags: [algorithm, tree, devide and conquer]
mathjax: true
date: 2025-06-11
---
<!-- placeholder -->
<!-- more -->
### 概述

- 点分治适用于处理类似问题：需要维护/查询树中**任意两点的路径长度**(不含平行边)
  其输出往往不是给出两点求距离，而是**给出距离**查询**是否存在这样的路径长度/小于该距离的路径数量**等等
  因为所有路径都可通过**经过/不经过某点来分类**，点分治是比一般暴力更优的借助分治思想的另类暴力
- (朴素暴力法)：对每个结点都作为起点进行遍历，每次遍历$O(n)$，因此总共$O(n^2)$

### 算法结构

- 考虑一棵带权无根树，即**任意结点均可作为根结点且不影响答案**，任选一根结点$r$
  则任意两点的路径可分为**经过$r$**、**不经过$r$**的两类路径
  经过$r$的路径分为**以$r$为起点**、**不以$r$为起点**(也不为终点)两类路径；所有不以$r$为起点的路径可由前者排列组合求得(并不是前者所有路径的排列组合，**两条路径途径结点必须没有交集才可合并**为新的经过$r$的路径)
  所有**不经过$r$**的路径可由**递归**算法(其新的根结点为原根结点的孩子，且计算时**删去原根结点**)求得
  因此仅需要考虑经过$r$的路径如何计算
  
- 上述给出了分治的雏形(递归求解)，为了使分治不退化至$O(n)$的深度，一般分治需要使分割点尽量在序列中间；点分治也同理，需要使分治中心：**根结点为树的重心**，以尽量达到$O(\log n)$的递归深度
  **寻找重心过程**：详见[`6.6_树的重心`](6.6_树的重心.md)，由于需要多次查询重心、且每次查询的总结点数将变化，需要改动：
  
  - 添加`visited`数组，标识“某结点被删除”
  - 求子树的重心时，总结点数变为`size[son]`
  
- **合并过程**：关于所有经过$r$但不以$r$为起点的路径，先记录所有向下结点至根结点的距离(同时记录所属的子树)，合并时如**若所属子树不同直接相加**即可

- 一般可用桶/哈希表来记录路径，并支持在线查询(预处理后常数时间得到答案)
  也可以用**离线方式处理**一批查询(对每批查询都使用一次点分治)
  先对这批查询排序，然后使用双指针遍历一批路径(指计算完经过$r$的所有路径后遍历)，更新布尔数组即可
  为了方便排序+查询其所属子树，不宜对深搜时得到的路径数组进行改动，需要额外的数组再存储一遍路径(通过存结点编号来存路径，排序时修改排序函数即可)
  
- ```c++
  // 采用链式前向星存树(边编号为0表示空)
  // 以"查询是否存在长为K的路径"为例
  bool visited[VN], queryAns[QN];  // 已访问结点、查询结果Ans
  int size[VN], w[VN], center, // 树大小、树大小最大值、重心
   n, qn, queryK[QN], // 现在遍历子树的总结点数、查询个数、查询值K
   dis[VN], tmp[VN], cntTmp, // 一批路径个数不多于总结点数、tmp作为将路径排成连续序列的副本(存储编号来代表路径)
   from[VN];  // 所属子树
  // 寻找当前树重心(赋值给center), O(n)
  void getCenter(int rt, int fa, int curN) {
      size[rt] = 1;
      w[rt] = 0;
      for (int i = head[rt]; i; i = edge[i].next) {
    if (edge[i].to != fa && !visited[edge[i].to]) {
              getCenter(edge[i].to, rt, curN);
              size[rt] += size[edge[i].to];
              w[rt] = max(w[rt], size[edge[i].to]);
          }
      }
      if (!center) {
          w[rt] = max(w[rt], curN - size[rt]);
          if (w[rt] <= (curN >> 1)) {
              center = rt;
          }
      }
  }
  // 获取所有向下结点离center的距离, O(n)
  void getDistance(int rt, int fa) {
      for (int i = head[rt]; i; i = edge[i].next) {
          if (edge[i].to != fa && !visited[edge[i].to]) {
              dis[edge[i].to] = dis[fa] + edge[i].w;
        tmp[cntTmp++] = edge[i].to;  // 也可用tmp[0]代替cntTmp, 大差不差
              from[edge[i].to] = from[rt] ? from[rt] : edge[i].to;
              getDistance(edge[i].to, rt);
          }
      }
  }
  // 对离center距离进行合并+处理查询
  void calc() {
      int left, right;
      cntTmp = 0;
      // 部分题解在遍历子树时调用getDis(方便确定所属子树), 其实对getDis稍作改动即可简化
      dis[center] = 0; // 用来表示以center为起点的路径
      tmp[cntTmp++] = center;
      getDistance(center, 0);
      sort(tmp, tmp + cntTmp, [](const int l, const int r) -> bool {
          return dis[l] < dis[r];
      });
      for (int i = 0; i < qn; ++i) {
          l = 0;
          r = cntTmp - 1;
          if (!queryAns[i]) { // 已有查询结果的不用查
              while (l < r) { // 双指针遍历
                  // 前两个分支查以center为起点(和过大则右指针左移)
                  if (dis[tmp[l]] + dis[tmp[r]] > queryK[i]) {
                      --r;
                  }
                  else if (dis[tmp[l]] + dis[tmp[r]] < queryK[i]) {
                      ++l;
                  }
                  // 和相等又分两种, 可以合并/不可合并
                  else if (from[tmp[l]] == from[tmp[r]]) { // 不可合并
                      if (dis[tmp[l]] == dis[tmp[l + 1]]) ++l; // 两分支顺序无要求
                      else --r;
                  }
                  else { // 可合并
                      queryAns[i] = true;
                      break;
                  }
              }
          }
      }
  }
  // 递归进行所有子树的calc()
  void solve(int rt, int curN) {
      int tmpN;
      calc(); // 计算经过center的路径
      visited[rt] = 1; // 表示center已计算
      for (int i = head[rt]; i; i = edge[i].next) {
       // 子树比重心更大, 说明在原来的有根树中子树是重心的向上子树, 因此用树大小减去rt大小
       tmpN = size[edge[i].to] > size[rt] ? curN - size[rt] : size[edge[i].to];
          getCenter(edge[i].to, 0, tmpN);
          solve(center, tmpN);
      }
  }
  int main() {
      cin >> n >> m >> q;
      // 输入略
      getCenter(1, 0, n); // 计算整棵树重心
      solve(center, n);
      // 输出略
  }
