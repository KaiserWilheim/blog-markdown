---
title: P4127 [AHOI2009] 同类分布 题解
date: 2022-10-17
tags:
	- 题解
	- 数位DP
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">同类分布</div>
<div id="problem-info-from">AHOI 2009</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4127">Luogu P4127</a></li><li><a href="https://www.acwing.com/problem/content/2894/">AcWing 2891</a></li><li><a href="https://darkbzoj.cc/problem/1799">BZOJ #1799</a></li></ul></div>

----

根据题目描述我们可以推测出其是一道典型的数位DP题目。
但是我们并不知道如何设计DP。

因为题目要求我们统计的数是满足其数位和能整除其自身，所以我们可以从这两个信息来入手。

但是，题目中的数字最大是 $10^{18}$ 级别的，是绝对无法维护的。

那么我们可以对其进行取模。
最理想也最简单的模数就是数位和了。
但是数位和在DP过程中是会变化的，我们应该选取一个定值作为模数才能保证我们程序的正确性。

观察数据范围，我们可以发现我们的数位和最多不过就是 $9 \times 18$，是可以进行枚举的。

那我们记录当前数字对枚举的模数取模的结果和当前数位和，最后看是否匹配就好了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 160, M = 20;
ll l, r;
int a[M], len;
ll mod;
ll dp[M][N][N];
ll dfs(int pos, int sum, ll now, bool lim)
{
	if(pos > len)return (now == 0 && sum == mod) ? 1 : 0;
	if(!lim && dp[pos][sum][now] != -1)return dp[pos][sum][now];
	ll res = 0;
	int top = lim ? a[len - pos + 1] : 9;
	for(int i = 0; i <= top; i++)
		res += dfs(pos + 1, sum + i, (10 * now + i) % mod, i == top && lim);
	dp[pos][sum][now] = res;
	return res;
}
ll calc(ll x)
{
	len = 0;
	while(x)
	{
		a[++len] = x % 10;
		x /= 10;
	}
	ll res = 0;
	for(mod = 1; mod <= 9 * len; mod++)
	{
		memset(dp, -1, sizeof(dp));
		res += dfs(1, 0, 0, true);
	}
	return res;
}
int main()
{
	scanf("%lld%lld", &l, &r);
	printf("%lld\n", calc(r) - calc(l - 1));
	return 0;
}
```

