---
title: P2469 [SDOI2010] 星际竞速 题解
date: 2022-08-10
tags:
	- 题解
	- 网络流
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">星际竞速</div>
<div id="problem-info-from">SDOI 2010</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2469">Luogu P2469</a></li><li><a href="https://www.acwing.com/problem/content/2501/">AcWing 2499</a></li><li><a href="https://darkbzoj.cc/problem/1927">BZOJ #1927</a></li></ul></div>

----

题目要求我们求出，从任意一个星球开始走，经过所有星球恰好一次的最短时间。

我们先看一看题目给了我们什么信息：

首先是有 $n$ 颗星球，每一颗星球只能经过一次，我们可以拆点。

然后是正常的星际航道，每一个星际航道都有自己的通过所需时间，这个就加进图中的边里面去。

然后每一颗星球都有一个自己的定位时间，与当前所在位置无关。且我们一开始必须使用这种方式来到达某一个星球。这个不是特别好想出来，暂时搁置。

最后还有一个要求，就是我们只能从编号较小的星球跳到编号较大的星球。

我们从上面的要求里面可以看出来是一个费用流，但是我们并没有建图的思路。

----

我们可以这样想：

既然我们需要每一个星球都经过恰好一次，那么我们可以对这个经过的方式进行分类讨论。

我们有两种方法能经过这里：

1. 航道
2. 跃迁

至于我们是从哪个星球来的，之后又要去哪里，已经不重要了，我们只关心我们是怎么到当前这一颗星球的。

我们仍然将星球拆成两个点，只不过这一次我们的两个点不再在中间连边来限制流量了。
他们有了其他的意义：出发和到达。

顾名思义，代表出发的点连接源点，并向与之有着星际航道联通的星球的到达点连边。代表到达的点连接汇点，同时还连接源点，代表直接跃迁到了这里。

每一条边的容量均为1，费用根据边的类型而定。除了代表跃迁的边和代表星际航道的边费用分别是 $a_i$ 和 $w_i$，其余各边费用都为0。

参考代码：

``` cpp
#include<bits/stdc++.h>
#define ll long long
using namespace std;
const int N = 100010, M = 200010;
const int INF = 1e9;
int m, n;
int S, T;
int h[N], e[M], ne[M], f[M], w[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];
void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while (hh != tt)
	{
		int t = q[hh++];
		if (hh == N) hh = 0;
		st[t] = false;

		for (int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if (f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if (!st[ver])
				{
					q[tt++] = ver;
					if (tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

void EK(int& flow, int& cost)
{
	flow = cost = 0;
	while (spfa())
	{
		int t = incf[T];
		flow += t, cost += t * d[T];
		for (int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	S = 0, T = n * 2 + 1;
	for (int i = 1; i <= n; i++)
	{
		int a;
		scanf("%d", &a);
		add(S, i, 1, 0);//代表从这个点出发
		add(S, i + n, 1, a);//代表直接跃迁到达这个点
		add(i + n, T, 1, 0);//到了这个点，打个卡
	}
	for (int i = 1; i <= m; i++)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		if (a > b)swap(a, b);//编号小的连编号大的
		add(a, b + n, 1, c);//出发连到达，从a出发走星际航道到达b
	}
	int flow, cost;
	EK(flow, cost);
	printf("%d\n", cost);
	return 0;
}
```

