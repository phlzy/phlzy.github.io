---
title: 【OP9】k-th-smallest-in-lexicographical-order
date: 2022-03-29
tag: [trie]
category: [One Problem]
math: true
---

来自力扣的一道好题

<!--more-->

题目链接：[字典序的第K小数字](https://leetcode-cn.com/problems/k-th-smallest-in-lexicographical-order/)


这题很容易产生在字典树上统计的思路，但是并不需要建出字典树，因为可以直接计算出一个子树的大小。于是直接用算出的子树大小和 $k$ 进行比较，判断是直接进入子树还是走向下一个兄弟节点。这种做法的时间复杂度是 $O(\log ^2 n)$ 的，因为 `ans++` 的操作每一层不会超过 $10$ 次。


``` cpp
class Solution {
using ll = long long;
public:
	ll calc(ll n, ll mx) {
		ll now = n, nxt = n + 1, cnt = 0;
		while (now <= mx) {
			cnt += min(mx + 1, nxt) - now;
			now *= 10, nxt *= 10;
		}
		return cnt;
	}
    int findKthNumber(int n, int k) {
		int ans = 1;
		while (k > 1) {
			int cnt = calc(ans, n);
			if (cnt < k)
				k -= cnt, ans++;
			else 
				k--, ans *= 10;
		}
		return ans;
    }
};
```





