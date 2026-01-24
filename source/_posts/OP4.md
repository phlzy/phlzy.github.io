---
title: 【OP4】League of Leesins
date: 2022-03-24
tag: [constructive,topological sort]
category: [One Problem]
math: true
---

思维

<!--more-->

题目链接：[League of Leesins](https://codeforces.com/contest/1255/problem/C)

容易注意到只出现一次的数必然可以作为第一个元素，然后从这个三元组开始搜下去就可以得到答案。具体实现方式很多，这里就不写了。

其实这题还有一个比较有趣的做法，可以根据给出的三元组对数字连边，最后进行一次拓扑排序即可。

``` cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 2e5 + 5;

int deg[maxn], vis[maxn];
int n;
vector<int> g[maxn];
void add(int u, int v) {
    g[u].push_back(v);
    g[v].push_back(u);
}

void work(int h) {
    queue<int> Q;
    Q.push(h);
    vis[h] = 1;
    while (!Q.empty()) {
        int u = Q.front();
        Q.pop();
        cout << u << " ";
        for (int v : g[u]) {
            if (vis[v] == 0) {
                deg[v]--;
                if (deg[v] == 1) {
                    Q.push(v);
                    vis[v] = 1;
                }
            }
        }
    }
}

int main() {
    cin >> n;
    for (int i = 0, x, y, z; i < n - 2; i++) {
        cin >> x >> y >> z;
        add(x, y), add(x, z), add(y, z);
        deg[x]++, deg[y]++, deg[z]++;
    }
    int a = 0, b = 0;
    for (int i = 1; i <= n; i++) {
        if (deg[i] == 1) {
            if (a == 0)
                a = i;
            else
                deg[i] += 2;
        }
    }
    for (int i : g[a]) {
        if (deg[i] == 2)
            b = i;
    }
    for (int i = 1; i <= n; i++) {
        if (deg[i] == 2 && i != b)
            deg[i]++;
    }
    work(a);
    return 0;
}
```



