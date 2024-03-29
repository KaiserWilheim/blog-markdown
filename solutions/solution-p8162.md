---
title: P8162 [JOI 2022 Final] 选举 题解
date: 2022-07-16
tags:
	- 题解
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name"><ruby>选举<rt>選挙で勝とう</rt></ruby></div>
<div id="problem-info-from">JOI 2022 Final</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P8162">Luogu P8162</a></li><li><a href="https://loj.ac/p/3664">LibreOJ L3664</a></li></ul></div>

----

我们首先可以得出一些结论。

首先，我们不会浪费时间。因为演讲时间可以是浮点数，不需要考虑取整的问题，所以如果我们只需要选票的话，我们只会在这个州演讲 $A_i$ 的时间，不会多也不会少。

其次，我们可以将存在多个人同时在不同州演讲的情况转化为只有多个人同时在同一个州演讲的情况。所以我们只需要考虑当前人数和在哪个州演讲就可以了。

然后，我们可以确定我们的演讲顺序。对于一个已经确定了的计划（即每一个州的演讲时间已经确定），我们肯定首先在所有能拿到协作者的州演讲完拿到协作者之后再去其他州演讲。

最后，我们可以将所有的州分为三类，分别是“协作州”、“支持州”和“反对州”。“协作州”指拿到了选票和协作者的州，“支持州”指只拿到了选票的州，而“反对州”指什么也没有拿到的州。
如果我们对所有的州按照 $B_i$ 为第一关键字，$A_i$ 为第二关键字来排序的话，我们可以发现，在最后一个“协作州”之前一定没有“反对州”。如果有的话，我们可以将两者调换一下位置，这样结果也不会变劣。

于是我们就可以想到一个DP，$f_{i,j,k}$ 代表前 $i$ 个州中有 $j$ 个“协作州”，$k$ 个“支持州”的最小时间。
同时根据上面的性质，我们可以忽略支持州，只枚举协作州，将 $O(n^3)$ 的DP优化到 $O(n^2)$。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1010;
int n, k;
struct State
{
	int a, b;
	bool operator < (const State &x)
	{
		return (~b && ~x.b) ? b < x.b : b>x.b;
	}
}p[N];
double f[N][N];
int tmp[N];
double get(int x)
{
	for(int i = 1; i <= n; i++)tmp[i] = p[i].a;
	for(int i = 0; i <= n; i++)
		for(int j = 0; j <= n; j++)
			f[i][j] = 1e9;
	f[0][0] = 0;//没有进行过任意一次演讲，耗时为 0
	double res = 1e9;
	for(int i = 1; i <= n; i++)
	{
		for(int j = 0; j <= min(x, i); j++)
		{
			f[i][j] = f[i - 1][j] + ((double)p[i].a / (x + 1));//不需要招协作者，就等到最后再演讲
			if(j >= 1 && p[i].b != -1)f[i][j] = min(f[i][j], f[i - 1][j - 1] + ((double)p[i].b / j));//需要招协作者，就立刻开始演讲罢
		}
	}
	for(int i = k; i <= n; i++)
		res = min(res, f[i][x]);
	for(int i = k - 1; i >= x; i--)
	{
		sort(tmp + i + 1, tmp + n + 1);
		double val = 0;
		for(int j = 1; j <= k - i; j++)val += tmp[i + j];
		res = min(res, f[i][x] + ((double)val / (x + 1)));//加上只要选票的耗时
	}
	return res;
}
int main()
{
	scanf("%d%d", &n, &k);
	for(int i = 1; i <= n; i++)
		scanf("%d%d", &p[i].a, &p[i].b);
	sort(p + 1, p + 1 + n);
	double ans = 1e9;
	for(int i = 0; i <= k; i++)//枚举招募协作者的数量
		ans = min(ans, get(i));
	printf("%.15lf\n", ans);
	return 0;
}
```

