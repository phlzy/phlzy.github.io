---
title: 【OP6】Lots of Parabolas
date: 2022-03-26
tag: [ternary search]
category: [One Problem]
math: true
---

三分

<!--more-->

题目链接：[Lots of Parabolas](https://codeforces.com/gym/103483/problem/H)

学长分享的一道有趣的题。抛物线开口指向的区域称为抛物线的内部，给出很多抛物线方程，请你输出一个位于所有抛物线内部的点。

可以根据二次项系数的正负将所有抛物线分为向上和向下的两类。对于向上的抛物线，取最大值后可以形成一个先减后增的单峰函数 $f$，对于向下的抛物线，取最小值后会形成一个先增后减的单峰函数 $g$。显然，这个点会存在于这两个单峰函数的图像之间。将这两个单峰函数相减，形成的函数依然是单峰函数。三分这个单峰函数，其极值点处应该是最有可能成为答案的地方，在这里取 $f$ 和 $g$ 的平均值作为答案的纵坐标。

``` cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn = 1e5 + 5;
const double eps = 1e-8;
struct node {
    double a, b, c;
};
vector<node> up, down;
double f(double x) {
    double ret = -1e30;
    for (auto &i : up) {
        ret = max(ret, i.a * x * x + i.b * x + i.c);
    }
    return ret;
}
double g(double x) {
    double ret = 1e30;
    for (auto &i : down) {
        ret = min(ret, i.a * x * x + i.b * x + i.c);
    }
    return ret;
}
double calc(double x) { return f(x) - g(x); }
int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    int n;
    cin >> n;
    for (int i = 1, a, b, c; i <= n; ++i) {
        cin >> a >> b >> c;
        if (a > 0)
            up.push_back({a, b, c});
        else
            down.push_back({a, b, c});
    }
    double l = -1e9, r = 1e9;
    while (fabs(l - r) > eps) {
        double midl = (2 * l + r) / 3, midr = (l + 2 * r) / 3;
        if (calc(midl) < calc(midr))
            r = midr;
        else
            l = midl;
    }
    cout << fixed << setprecision(6) << l << ' ' << (f(l) + g(l)) / 2.0 << endl;
    return 0;
}
```



