---
title: 【OP5】Equal Tree Sums
date: 2022-03-25
tag: [constructive,tree]
category: [One Problem]
math: true
---

构造

<!--more-->

题目链接：[Equal Tree Sums](https://codeforces.com/contest/1656/problem/E)

这是昨晚比赛中没有做出来的一道题。让你给一棵树的所有节点赋值，使其删去任意一个节点后产生的连通块权值和均相等。

当时手画了几种简单的情况，发现可以对树二染色后交替赋正值和负值，但没能构造出赋值的方案。

其实很简单，染色后每条边的两端颜色必然不同，可以让每条边的总贡献均为零，即每个点的权值的绝对值都是其度数，这样删去一个点就相当于将这个点的权值平均流到每个边上去了，每个连通块的权值之和就都是 $1$ 或者 $-1$。

``` cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1e5 + 5;
vector<int> g[maxn];
int ans[maxn];
void dfs(int u, int f, bool neg) {
    ans[u] = g[u].size();
    if (neg)
        ans[u] = -ans[u];
    for (auto v : g[u]) {
        if (v == f)
            continue;
        dfs(v, u, !neg);
    }
}
void solve() {
    int n;
    cin >> n;
    for (int i = 1, u, v; i < n; ++i) {
        cin >> u >> v;
        g[u].push_back(v), g[v].push_back(u);
    }
    dfs(1, 0, false);
    for (int i = 1; i <= n; ++i) {
        cout << ans[i] << " \n"[i == n];
        g[i].clear();
    }
}
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T = 1;
    cin >> T;
    while (T--) {
        solve();
    }
}
```



