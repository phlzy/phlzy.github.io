---
title: 【OP8】Wrapping Chocolate
date: 2022-03-28
tag: [greedy]
category: [One Problem]
math: true
---

贪心

<!--more-->

题目链接：[Wrapping Chocolate](https://atcoder.jp/contests/abc245/tasks/abc245_e)

$n$ 盒巧克力和 $m$ 个盒子有各自的长和宽，巧克力只能放进长宽均不小于自己的盒子，不能转过来放，问能不能把所有巧克力都放进去。

只要想办法把一个维度消掉，就很容易解决这个问题。

可以把盒子和巧克力放一起，对宽度从大到小排序，盒子永远放在最前面。这样枚举的时候宽度的要求就直接满足了，然后维护一下可用的盒子的长度即可。


``` cpp
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
const int maxn = 2e5 + 5;
struct node {
    int w, l, type;
    bool operator<(const node &other) const {
        if (w == other.w)
            return type < other.type;
        return w < other.w;
    }
} a[maxn << 1];

void solve() {
    int n, m;
    cin >> n >> m;
    for (int i = 1; i <= n; ++i) {
        cin >> a[i].w;
        a[i].type = 0;
    }
    for (int i = 1; i <= n; ++i) {
        cin >> a[i].l;
    }
    for (int i = 1; i <= m; ++i) {
        cin >> a[i + n].w;
        a[i + n].type = 1;
    }
    for (int i = 1; i <= m; ++i) {
        cin >> a[i + n].l;
    }
    sort(a + 1, a + n + m + 1);
    multiset<int> s;
    for (int i = n + m; i > 0; --i) {
        if (a[i].type) {
            s.insert(a[i].l);
        } else {
            auto it = s.lower_bound(a[i].l);
            if (it == s.end()) {
                cout << "No" << endl;
                return;
            }
            s.erase(it);
        }
    }
    cout << "Yes" << endl;
}
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int T = 1;
    // cin >> T;
    while (T--) {
        solve();
    }
}
```



