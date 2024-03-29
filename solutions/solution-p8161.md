---
title: P8161 [JOI 2022 Final] 自学（自習） 题解
date: 2022-07-13
tags:
	- 题解
	- 贪心
	- 二分
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name"><ruby>自学<rt>自習</rt></ruby></div>
<div id="problem-info-from">JOI 2022 Final</div>
<div id="problem-info-difficulty">普及/提高-</div>
<div id="problem-info-color">#ffc116</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P8161">Luogu P8161</a></li><li><a href="https://loj.ac/p/3663">LibreOJ L3663</a></li></ul></div>

----

首先我们可以确定，如果我们某一门课自学的效果比老师讲的效果都好的话就不需要去上课了，所以我们让 $a_i = \max(a_i,b_i)$。

根据题目给定的数据范围，我们肯定不能用暴力。
我们需要一个至少 $O(n \log n)$ 的算法。

数据结构虽然能带一个 $\log$，但是我没有想到任何能够维护这个信息的数据结构。

那就只剩二分了。
我们来观察一下到底可不可以二分。

我们二分就只能二分答案了，也就是预期的最低熟练度。
而我们能够发现，一旦我们一个预期熟练度 $k$ 能够被满足，那么 $\leq k$ 的所有预期熟练度都可以被满足，这就为我们的二分提供了保障。

那么我们怎么判断一个 $k$ 能否被满足呢？

我们考虑这样一个思路：

我们遍历每一门课程，记录一下这门课的空余时间 $t_i$。$t_i$ 为正时表示该门课可以空出来的时间为 $t_i$ 节课，而为负时表示该门课需要的额外的时间为 $-t_i$ 节课。
- 如果 $ma_i \geq k$，那就意味着这门课不需要听满 $m$ 节，只需要听 $\lceil \dfrac{k}{a_i} \rceil$ 节即可，这给我们留出了 $m - \lceil \dfrac{k}{a_i} \rceil$ 节课的空余时间。令 $t_i = m - \lceil \dfrac{k}{a_i} \rceil$。
- 如果 $ma_i < k$，那么我们就需要用额外的时间来补这门课，所需的额外时间为 $\lceil \dfrac{k - ma_i}{b_i} \rceil$。令 $t_i = -\lceil \dfrac{k - ma_i}{b_i} \rceil$。

最后将所有的 $t_i$ 加起来，看总和是否为负数。如果为负，那就意味着我们需要花比我们所拥有的时间更多的时间来学习，而这是不可能的，代表着当前最小值不能达成。

----

一个小细节：不开`long long`会炸。
同时判断函数中间可能会出现爆`long long`的情况，并且二分区间右端点必须为 $10^{18}$ 及以上（$m \leq 10^9,a_i \leq 10^9$），请大家酌情考虑使用`__int128`。

示例代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 300010;
ll n, m;
ll a[N], b[N];
ll calc(ll a, ll b)
{//不需要double的ceil
	if (a % b == 0)return a / b;
	else return a / b + 1;
}
bool chq(ll k)
{
	__int128 tot = 0;
	for (int i = 1; i <= n; i++)
	{
		if (m * a[i] >= k)tot += m - calc(k, a[i]);
		else tot -= calc((k - m * a[i]), b[i]);
	}
	return tot >= 0;
}
int main()
{
	scanf("%lld%lld", &n, &m);
	for (int i = 1; i <= n; i++)
		scanf("%lld", &a[i]);
	for (int i = 1; i <= n; i++)
	{
		scanf("%lld", &b[i]);
		a[i] = max(a[i], b[i]);
	}
	ll l = 1, r = 1e18;
	ll ans;
	while (l <= r)
	{
		ll mid = (l + r) >> 1;
		if (chq(mid))l = mid + 1, ans = mid;
		else r = mid - 1;
	}
	printf("%lld\n", ans);
	return 0;
}
```

