---
title: P1251 餐巾计划问题 题解
date: 2022-03-22
tags:
	- 题解
	- 网络流
categories:
	题解
mathjax: true
---

Luogu P1251
LibreOJ #6008

网络流24题之一。

<!--more-->
----

题目要求我们安排餐巾的清洗分配。

可能是戴森球计划搭产线搭多了吧，这道题我一开始就把餐巾从一个地方到另一个地方（如从餐厅到快洗部）想成了一道流。

我们就可以把餐厅向快洗部和慢洗部连边。

但是呢，我们还需要处理快洗部和慢洗部需要的时间问题，还有处理所有的延时送洗的操作。

我们于是就考虑将每天的餐厅拆成不同的点。
当然，每天的快洗部和慢洗部也需要拆成不同的点。

这样就可以保证我们可以让餐巾进行时间穿梭了，而不再是只局限于一天之内。

还有，我们需要将餐巾进行最大化利用来省钱，毕竟每多存在一块餐巾，就多需要花费我们 $p$ 分钱。

我们考虑将拆分后的餐厅继续拆分，拆成一个输入和一个输出。输入代表今天餐厅的餐巾到底是从哪里运过来的，而输出就代表今天使用的餐巾的最终去处。

这时候，我们就已经需要了 $4n$ 个点了（不包括源点和汇点），其中餐厅输入，餐厅输出，快洗部和慢洗部各 $n$ 个点。

但是我们考虑简化一下。

我们考虑省去快洗部和慢洗部，可以看做自己内部员工洗干净了。
这样，我们可以直接连接某一天的输出和另一天的输入了。

最后开始连边。
1. 源点与餐厅输出连一条容量为当天用量，费用为0的边；
2. 汇点与餐厅输入连一条容量为当天用量，费用为0的边；
3. 餐厅输出与下一日的餐厅输出连一条容量为无限，费用为0的边。
4. 源点与餐厅输入连一条容量为无限，费用为 $p$ 的边；
5. 餐厅输出与 $n$ 天后的餐厅输入连一条容量为无限，费用为 $s$ 的边，代表快洗部；
6. 餐厅输出与 $m$ 天后的餐厅输入连一条容量为无限，费用为 $f$ 的边，代表慢洗部。

然后跑一个最小费用最大流即可，最终的费用就是答案。

上代码：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int p, x, xp, y, yp;
int need[N];
int s[N];
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a]; h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

int EK()
{
	int cost = 0;
	while(spfa())
	{
		int t = incf[T];
		cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
	return cost;
}

signed main()
{
	scanf("%lld", &n);
	S = 0, T = n * 2 + 1;
	memset(h, -1, sizeof h);
	for(int i = 1; i <= n; i++)scanf("%lld", &need[i]);
	scanf("%lld%lld%lld%lld%lld", &p, &x, &xp, &y, &yp);
	for(int i = 1; i <= n; i++)
	{
		int r = need[i];
		add(S, i, r, 0);
		add(n + i, T, r, 0);
		add(S, n + i, INF, p);
		if(i < n) add(i, i + 1, INF, 0);
		if(i + x <= n) add(i, n + i + x, INF, xp);
		if(i + y <= n) add(i, n + i + y, INF, yp);
	}
	printf("%lld\n", EK());
	return 0;
}

```

