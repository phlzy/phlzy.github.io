---
title: 【OP1】Islands War
date: 2022-03-21
tag: [greedy]
category: [One Problem]
math: true
---

贪心

<!--more-->


题目链接：[Islands War](https://atcoder.jp/contests/abc103/tasks/abc103_d)

$n$ 个点形成一条链，给你 $m$ 个点对，让你删掉若干条边使这些点对都不连通，问最少删多少条边。

可以直观的感受到，如果存在一组两两相交的区间，那么只需要删一条边，即最左边的右端点所在的边。

于是容易得到一个 $O(m\log m)$ 的贪心做法，对所有区间按右端点排序即可。

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define F first
#define S second
#define endl '\n'
using pii = pair<int, int>;
const int maxn = 1e5 + 5;
pii seg[maxn];
int main(){
	int n, m;
	cin >> n >> m;
	for(int i = 1; i <= m; ++i){
		cin >> seg[i].F >> seg[i].S;
	}
	sort(seg + 1, seg + m + 1, [](pii a, pii b){
		return a.S < b.S;
	});
	int now = 0, ans = 0;
	for(int i = 1; i <= m; ++i){
		if(now <= seg[i].F){
			now = seg[i].S;
			ans++;
		}
	}
	cout << ans << endl;
	return 0;
}
```

如果预处理一下每个点需要断的最近右端点，可以把复杂度优化到线性：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define endl '\n'
const int maxn = 1e5 + 5;
int a[maxn];
int main(){
	int n, m, L = maxn;
	cin >> n >> m;
	if(!m){
		cout << 0 << endl;
		return 0;
	}
	for(int i = 1, l, r; i <= m; ++i){
		cin >> l >> r;
		L = min(L, l);
		if(a[l] > r || !a[l]) a[l] = r;
	}
	int now = a[L], ans = 1;
	for(int i = 1; i <= n; ++i){
		if(!a[i]) continue;
		if(i >= now){
			ans++;
			now = a[i];
		} else if(now > a[i]){
			now = a[i];
		}
	}
	cout << ans << endl;
	return 0;
}
```

如果 $n$ 个点形成一棵树，将删边改成删点，也可以用类似的方法来做，只需将点对的 LCA 按深度排序后从深的开始删即可。