---
title: 【OP2】the number of good subsets
date: 2022-03-22
tag: [dp,bitmask]
category: [One Problem]
math: true
---

状压，dp

<!--more-->

题目链接：[好子集的数目](https://leetcode-cn.com/problems/the-number-of-good-subsets/)

这题有个有趣的条件：数组中的元素的取值范围只有 $[1,30]$。在这个范围内只存在 $10$ 个质数，所以容易想到状压的做法。先统计每个数出现的次数，然后令 $dp[i][j]$ 表示选择 $[2,i]$ 中的数且用了 $j$ 状态中的质因子时的好子集数目，那么状态转移方程也很容易得出：

- 若 $i$ 有平方因子，则这个数是没用的，$dp[i][j]=dp[i-1][j]$
- 否则 $dp[i][j]=dp[i-1][j]+dp[i-1][k]$，其中 $k$ 为状态 $j$ 中去掉 $i$ 这个数包含的质因子形成的状态

初始状态就是选 $1$ 的方案数，即 $2^{cnt[1]}$。

然后还可以滚动数组优化一下空间，倒序枚举即可。

``` cpp
using ll = long long;
class Solution {
private:
    const ll mod = 1e9 + 7;
	const int p[10] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29};
public:
	ll ksm(ll b, ll k) {
		ll ret = 1;
		while (k) {
			if (k & 1) ret = ret * b % mod;
			k >>= 1, b = b * b % mod;
		}
		return ret;
	}
    int numberOfGoodSubsets(vector<int>& nums) {
    	vector<int> cnt(31);
    	vector<ll> dp(1 << 10, 0);
    	for (auto&& i: nums) cnt[i]++;
    	dp[0] = ksm(2, cnt[1]);
        for (int i = 2; i <= 30; ++i) {
        	if (!cnt[i]) continue;
        	int now = 0, x = i, flag = 1;
        	for (int j = 0; j < 10; ++j) {
        		if (x % (p[j] * p[j]) == 0) {
        			flag = 0;
        			break;
        		}
        		if (x % p[j] == 0) now |= (1 << j);
        	}
        	if (!flag) continue;
        	for (int st = (1 << 10) - 1; st >= 0; --st) {
        		if ((st & now) == now)
        			dp[st] = (dp[st] + dp[st ^ now] * cnt[i]) % mod;
        	} 
        }
        ll ans = accumulate(dp.begin() + 1, dp.end(), 0LL) % mod;
        return (int)ans;
    }
};
```



