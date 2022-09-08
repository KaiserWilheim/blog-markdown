---
title: CF1288F Red-Blue Graph 题解
date: 2022-08-21
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
<div id="problem-info-name">Red-Blue Graph</div>
<div id="problem-info-from">Educational Codeforces Round 80</div>
<div id="problem-info-difficulty">NOI / NOI+ / CTSC</div>
<div id="problem-info-color">#0e1d69</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/CF1288F">Luogu CF1288F</a></li><li><a href="http://codeforces.com/problemset/problem/1288/F">CF 1288 F</a></li></ul></div>

----

不知道怎么就有思路了（

首先看数据范围，$1 \leq n,m \leq 200$，应该是给网络流的复杂度。

对于一个红色的点，我们将其连接着的边中颜色不同的边一对对地消除了之后还剩一些红边，对于蓝点这样消除之后还剩一些蓝边。
那么红点应该对于红边有贡献，蓝点应该对于蓝边有贡献。

把这个绑定到流上面，就可以认为红点有流出，蓝点有流入，而没有颜色的点既可以流出也可以流入。而红边从左往右，蓝边从右往左，没有染色的边不走，所以既不往左也不往右。
我们需要保证这个“不少于”，那么二分图的点连向源（汇）点的边就有了下界，需要使用有源汇上下界最小费用可行流。
其实现思路与有源汇上下界可行流是一样的，同时也需要将bfs函数改为spfa来实现费用流。

----

具体实现的时候，我们将每一个原图中的边拆成两条边，一条红，一条蓝，容量为 $[0,1]$。
对于每一个点，如果其是红色点，那么连接源点，容量为 $[1,\infty]$；如果其是蓝色点，那么连接汇点，容量为 $[1,\infty]$；如果其没有颜色，那么其既连接源点，又连接汇点，容量都是 $[0,\infty]$。

然后就直接跑就可以了。

参考代码：

``` cpp
#include<bits/stdc++.h>
#define ll long long
using namespace std;
const int N = 10010, M = 200010;
const int INF = 1e9;
int m, n1, n2;
int S, T, s, t;
int R, B;
int h[N], e[M], f[M], w[M], ne[M], idx;
int A[N], tot;
int q[N], d[N], pre[N], incf[N];
bool st[N];
void add(int a, int b, int l, int r, int c)
{
	A[a] -= l, A[b] += l;
	e[idx] = b, f[idx] = r - l, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -c, ne[idx] = h[b], h[b] = idx++;
}
bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof(d));
	memset(incf, 0, sizeof(incf));
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;

		for(int i = h[t]; ~i; i = ne[i])
		{
			int v = e[i];
			if(f[i] && d[v] > d[t] + w[i])
			{
				d[v] = d[t] + w[i];
				pre[v] = i;
				incf[v] = min(f[i], incf[t]);
				if(!st[v])
				{
					q[tt++] = v;
					if(tt == N) tt = 0;
					st[v] = true;
				}
			}
		}
	}

	return incf[T] > 0;
}
void EK(int &flow, int &cost)
{
	flow = cost = 0;
	while(spfa())
	{
		int t = incf[T];
		flow += t, cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d%d%d%d", &n1, &n2, &m, &R, &B);
	s = 0, t = n1 + n2 + 1;
	S = n1 + n2 + 2, T = n1 + n2 + 3;
	string lc, rc;
	cin >> lc >> rc;
	for(int i = 1; i <= m; i++)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		add(a, b + n1, 0, 1, R);
		add(b + n1, a, 0, 1, B);
	}
	for(int i = 1; i <= n1; i++)
	{
		if(lc[i - 1] == 'R')add(s, i, 1, INF, 0);
		if(lc[i - 1] == 'B')add(i, t, 1, INF, 0);
		if(lc[i - 1] == 'U')add(s, i, 0, INF, 0), add(i, t, 0, INF, 0);
	}
	for(int i = 1; i <= n2; i++)
	{
		if(rc[i - 1] == 'R')add(i + n1, t, 1, INF, 0);
		if(rc[i - 1] == 'B')add(s, i + n1, 1, INF, 0);
		if(rc[i - 1] == 'U')add(s, i + n1, 0, INF, 0), add(i + n1, t, 0, INF, 0);
	}
	add(t, s, 0, INF, 0);
	for(int i = s; i <= t; i++)
	{
		if(A[i] > 0)tot += A[i], add(S, i, 0, A[i], 0);
		if(A[i] < 0)add(i, T, 0, -A[i], 0);
	}
	int flow, cost;
	EK(flow, cost);
	if(flow != tot)puts("-1");
	else
	{
		printf("%d\n", cost);
		for(int i = 0; i < m * 4; i += 4)
		{
			if(!f[i])putchar('R');
			else if(!f[i + 2])putchar('B');
			else putchar('U');
		}
	}
	return 0;
}
```

