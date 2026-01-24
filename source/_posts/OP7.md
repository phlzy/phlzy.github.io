---
title: 【OP7】Query Weight of Subtree
date: 2022-03-27
tag: [dsu on tree]
category: [One Problem]
math: true
---

dsu on tree

<!--more-->

这是别人问我的题，所以没有题目链接，名字也是我随便起的。（好像出自赛氪上的某场比赛）

有一棵包含 $n$ 个节点的树，节点编号从 $1$ 到 $n$，以 $1$ 为根，每条边都有一个权值。给你 $m$ 次询问，每次询问一个子树，在子树中对每种边权的值 $c$ 统计出现次数 $cnt_{c}$，求 $\sum(c\cdot cnt_c)^2$。

$n,m$ 以及边权范围均为 $1\times 10^5$。

容易发现可以将边权下放到点，并且这个平方和很好维护，增删一个权值都可以 $O(1)$ 完成。那么可以用 dsu on tree 来做，先预处理出所有节点的最大子树，然后 dfs 的时候先算所有小子树再算大子树，最后暴力合并小子树即可。时间复杂度 $O(n\log n)$。

``` cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
using pii = pair<int, int>;
const int maxn = 1e5 + 5;
ll weight[maxn], ans[maxn];
vector<int> g[maxn], id[maxn];
int sz[maxn], son[maxn];
void dfs0(int u) {
    sz[u] = 1;
    for (auto v : g[u]) {
        dfs0(v);
        sz[u] += sz[v];
        if (sz[v] > sz[son[u]])
            son[u] = v;
    }
}
int cnt[maxn], vis[maxn];
ll sum;
void clear(int u) {
    sum -= (weight[u] * cnt[weight[u]]) * (weight[u] * cnt[weight[u]]);
    cnt[weight[u]]--;
    sum += (weight[u] * cnt[weight[u]]) * (weight[u] * cnt[weight[u]]);
    for (auto v : g[u])
        clear(v);
}
void dfs(int u, int op) {
    for (auto v : g[u]) {
        if (v == son[u])
            continue;
        dfs(v, 0);
    }
    if (son[u]) {
        dfs(son[u], 1);
    }
    for (auto v : g[u]) {
        if (v == son[u])
            continue;
        dfs(v, 1);
    }
    if (!vis[u]) {
        for (auto x : id[u]) {
            ans[x] = sum;
        }
        vis[u] = 1;
    }
    sum -= (weight[u] * cnt[weight[u]]) * (weight[u] * cnt[weight[u]]);
    cnt[weight[u]]++;
    sum += (weight[u] * cnt[weight[u]]) * (weight[u] * cnt[weight[u]]);
    if (!op) {
        clear(u);
    }
}
void solve() {
    int n, m, c, q;
    cin >> n >> m >> c;
    for (int i = 1, u, v, w; i < n; ++i) {
        cin >> u >> v >> w;
        g[u].push_back(v);
        weight[v] = w;
    }
    for (int i = 1, x; i <= m; ++i) {
        cin >> x;
        id[x].push_back(i);
    }
    dfs0(1);
    dfs(1, 1);
    for (int i = 1; i <= m; ++i) {
        cout << ans[i] << '\n';
    }
}
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    solve();
    return 0;
}
```



