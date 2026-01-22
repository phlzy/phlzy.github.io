---
title: Kuhn–Munkres Algorithm
date: 2022-07-22
tag: [algorithm]
category: [Algorithm and DataStructure]
math: true
---

二分图匹配问题与 KM 算法

<!--more-->

二分图匹配的概念很容易理解：假设现在有一个二分图 $G=(V,E)$，对于它的一个子图 $M=(V_M,E_M)$，如果边集 $E_M$ 的所有边均没有共同的顶点，$M$ 就是 $G$ 的一个匹配。

对于一个匹配 $M$，其边数 $|E_M|$ 就是匹配数。二分图的最大匹配问题就是让你找出一个匹配数最大的匹配。

## 增广路

在找最大匹配之前，先尝试解决一个简单的问题：如果当前找到的匹配有优化的空间（存在更大的匹配），如何优化当前解？

如果这个问题可以比较简便的解决，那么就能够一步步得到最大匹配了。

我们定义**交错路径**为这样的一种路径：从一个未匹配点出发，交替经过非匹配边与匹配边形成的路径。

定义**增广路**为终点也是非匹配边的**交错路径**。

容易发现，只要把增广路里的匹配边变成非匹配边，非匹配边变成匹配边，那么就增加了一个匹配。这就是增广的过程。

**Berge's lemma**：匹配 $M$ 为图 $G=(V, E)$ 的最大匹配当且仅当 $G$ 中不存在 $M$-增广路。

必要性是显而易见的，用反证法很容易证明。

不太严谨地证明一下充分性：反证法，若不存在增广路且有一个更大的匹配 $M_1$，那么取两个匹配的边集的对称差，里面就是一些链和偶数长度的环。显然位于 $M_1$ 中的点的数量更多，所以可以找到一个从 $M_1$ 中的点出发的交错路，这就是增广路了。

## Hungarian Algorithm

有了增广路定理的基础后，匈牙利算法的原理就更容易理解了。逐一枚举二分图左部的顶点，每次寻找一条增广路，只要找到增广路就进行增广，并更新匹配结果；否则意味着能与这个点匹配的右部的点已经被耗尽了，之前占用这些右部的点的左部的点也转不到别的地方去，所以这个点已经彻底没用了，直接枚举下一个即可。

参考代码以[UOJ#78. 二分图最大匹配](https://uoj.ac/problem/78)为例：

``` cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 500 + 5;
vector<int> g[maxn];
int matchL[maxn], matchR[maxn], vis[maxn];
int L, R, k;
bool augment(int u) {
    for (auto &v : g[u]) {
        if (vis[v])
            continue;
        vis[v] = true;
        if (!matchR[v] || augment(matchR[v])) {
            matchR[v] = u;
            matchL[u] = v;
            return true;
        }
    }
    return false;
}
int Hungarian() {
    int cnt = 0;
    for (int u = 1; u <= L; ++u) {
        memset(vis, 0, sizeof(vis));
        if (augment(u))
            ++cnt;
    }
    return cnt;
}
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cin >> L >> R >> k;
    for (int i = 1, u, v; i <= k; ++i) {
        cin >> u >> v;
        g[u].push_back(v);
    }
    cout << Hungarian() << endl;
    for (int i = 1; i <= L; ++i) {
        cout << matchL[i] << " ";
    }
    return 0;
}
```

显而易见，匈牙利算法的时间复杂度是 $O(|V||E|)$。

## Hopcroft–Karp Algorithm

二分图的最大匹配问题容易转化为最大流问题，所以也可以用网络流算法解决。由于二分图的特殊性，用 Dinic 算法求最大匹配的时间复杂度仅为 $O(\sqrt{|V|}|E|)$。Dinic 算法的一大特点是多路增广，其实匈牙利算法也可以用多路增广优化到同样的复杂度，这就是 Hopcroft–Karp 算法。

具体内容心情好了再写。

## Kuhn–Munkres Algorithm

未完待续

