---
title: 【OP3】Antenna Coverage
date: 2022-03-23
tag: [dp]
category: [One Problem]
math: true
---

dp

<!--more-->

题目链接：[Antenna Coverage](https://codeforces.com/contest/1253/problem/E)

数轴上有 $n$ 个天线，每个天线都有一定的辐射范围，可以支付 $k$ 的费用让某个天线的辐射半径增加 $k$，可以任意执行修改操作，问覆盖区间 $[1,m]$ 的最少费用。

各种贪心似乎都是不行的。观察数据范围，$O(nm)$ 的复杂度可以通过。尝试 dp，令 $dp[i]$ 表示覆盖到 $i$ 位置的最少费用，如果 $i$ 已经被覆盖，直接从 $dp[i-1]$ 转移，否则枚举之前的所有天线 $j$ 作为覆盖当前位置的天线，假设这种情况下延长的半径为 $k$，则从 $dp[j-r_j-k-1]$ 处转移。

``` cpp
#include <bits/stdc++.h>
using namespace std;
using pii = pair<int, int>;
const int maxn = 1e5 + 5;
int dp[maxn];
pii p[maxn];
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n, m;
    cin >> n >> m;
    for (int i = 1, x, s; i <= n; ++i) {
        cin >> x >> s;
        p[i] = {max(0, x - s), min(m, x + s)};
    }
    dp[0] = 0;
    for (int i = 1; i <= m; ++i) {
        dp[i] = i;
        for (int j = 1; j <= n; ++j) {
            if (p[j].first <= i && p[j].second >= i)
                dp[i] = min(dp[i], dp[i - 1]);
            else if (p[j].second < i) {
                int d = i - p[j].second;
                dp[i] = min(dp[i], dp[max(0, p[j].first - d - 1)] + d);
            }
        }
    }
    cout << dp[m] << '\n';
}
```

这是一种很经典的题型，我已经遇见过很多类似的题了。

