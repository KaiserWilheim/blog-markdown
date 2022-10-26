---
title: "S2OJ #1449. 保KDA 题解"
date: 2022-09-24
tags:
	- 题解
	- 分数规划
	- 背包
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">保KDA</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1449">S2OJ #1449</a></li></ul></div>

----

题目要求我们帮nekko战神在被对方打死和被队友打死之间权衡出来一种方案，使得nekko战神获得最高的KDA值。

假设nekko战神不能一次不死，那么我们的KDA（用 $\frac{p}{q}$ 表示）就等于 $\frac{\sum (a_i + c_i)}{\sum (b_i + d_i)}$。
我们将其转化为一个判断性问题，直接二分求解。
那么我们可以将 $\frac{\sum (a_i + c_i)}{\sum (b_i + d_i)} \geq \frac{p}{q}$ 转化为 $q\sum (a_i+c_i) - p\sum (b_i+d_i) \geq 0$。
然后我们可以将其转化为一个背包问题，选这个物品的代价是 $q(a_i+c_i)-pb_i$，不选的代价是 $-pd_i$。

我们二分答案的时候，因为 $\sum (a_i + c_i)$ 和 $\sum (b_i + d_i)$ 都不大于1000，我们可以将所有的分数枚举出来，然后排个序在数组内枚举；也可以二分小数，得到足够精度的答案之后枚举分母。
更快也更通用的方法是在施特恩-博罗考(Stern-Brocot)树上二分。这种方法在当前数据范围下，使用`scanf()`和`printf()`的时候能够在保证不TLE的情况下达到40000左右的精度。
~（而且还在「最快代码」榜上以405ms的成绩排第一）~

但是强大的nekko战神可以一次不死并且超神。
这时我们就需要处理一下nekko战神不死的情况。
我们给我们的背包加上一维0/1，变成 $f[i][j][0/1]$，代表当前nekko战神已经考虑完了前 $i$ 个任务，总用时为 $j$，并且死过了/没死过的最大收益。

最后我们只需要对于nekko战神死了和没死的两种情况分别找到一个可行的用时即可。

具体转移如下：

``` cpp
for(int j = 0; j <= m; j++)
{
	f[i][j][(d[i] != 0) ? 1 : 0] = max(f[i][j][(d[i] != 0) ? 1 : 0], f[i - 1][j][0] - d[i] * p);
	f[i][j][1] = max(f[i][j][1], f[i - 1][j][1] - d[i] * p);
}
for(int j = t[i]; j <= m; j++)
{
	f[i][j][(b[i] != 0) ? 1 : 0] = max(f[i][j][(b[i] != 0) ? 1 : 0], f[i - 1][j - t[i]][0] - b[i] * p + a[i] * q);
	f[i][j][1] = max(f[i][j][1], f[i - 1][j - t[i]][1] - b[i] * p + a[i] * q);
}
```

完整代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1010;
struct frac
{
	int p, q;
	frac(int _p, int _q) { p = _p, q = _q; }
};
int n, m;
int t[N], a[N], b[N], d[N];
int f[N][N][2];
bool chq(frac mid)
{
	int p = mid.p, q = mid.q;
	memset(f, 0xc1, sizeof(f));
	f[0][0][0] = 0;
	for(int i = 1; i <= n; i++)
	{
		for(int j = 0; j <= m; j++)
		{
			f[i][j][(d[i] != 0) ? 1 : 0] = max(f[i][j][(d[i] != 0) ? 1 : 0], f[i - 1][j][0] - d[i] * p);
			f[i][j][1] = max(f[i][j][1], f[i - 1][j][1] - d[i] * p);
		}
		for(int j = t[i]; j <= m; j++)
		{
			f[i][j][(b[i] != 0) ? 1 : 0] = max(f[i][j][(b[i] != 0) ? 1 : 0], f[i - 1][j - t[i]][0] - b[i] * p + a[i] * q);
			f[i][j][1] = max(f[i][j][1], f[i - 1][j - t[i]][1] - b[i] * p + a[i] * q);
		}
	}
	for(int j = 0; j <= m; j++)
	{
		if(f[n][j][1] >= 0)return true;
		if(f[n][j][0] - p >= 0)return true;
	}
	return false;
}
int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
	{
		int c;
		scanf("%d%d%d%d%d", &t[i], &a[i], &b[i], &c, &d[i]);
		a[i] += c;
	}
	frac l(0, 1), r(1, 0);
	while(1)
	{
		frac mid(l.p + r.p, l.q + r.q);
		if(mid.p > 1000 || mid.q > 1000)break;
		if(chq(mid))l = mid;
		else r = mid;
	}
	printf("%d %d\n", l.p, l.q);
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>