---
title: P4568 [JLOI2011] 飞行路线 题解
date: 2022-07-21
tags:
	- 题解
	- 最短路
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">飞行路线</div>
<div id="problem-info-from">JLOI 2011</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4568">Luogu P4568</a></li><li><a href="https://www.acwing.com/problem/content/2956/">AcWing 2953</a></li><li><a href="https://darkbzoj.cc/problem/2763">BZOJ #2763</a></li></ul></div>

----

一眼最短路，但是多了一些条件：可以最多将最短路上 $k$ 条边的权值变为 $0$。

因为我们求的是两个点之间的最短路，只需要最小化这两个点之间的距离即可，这样我们就不能先跑最短路再改边权。

我们考虑换种建图方式。

我们考虑将免费的机票和付费的机票分开连边。

我们在原图的基础上再建立 $k$ 层同样的图，分别代表使用了 $1,2,3,\cdots,k$ 次免费机票时的状态。
相邻两层之间也连边，边权为 $0$。对于原图中的每一条边 $(u,v)$，我们同时从第 $i$ 层的 $u$ 向第 $i+1$ 的 $v$、从第 $i$ 层的 $v$ 向第 $i+1$ 层的 $u$ 连上一条边权为 $0$ 的有向边，代表使用了免费的机票。

我们就从第 $0$ 层的 $s$ 向第 $k$ 层的 $t$ 跑最短路。

同时，以防某些奇葩数据使得我们没用完 $k$ 张免费机票就到了 $t$，我们把相邻两层之间的 $t$ 也连上边权为 $0$ 的有向边。

代码如下：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 500010, M = 5000010;
int n, m;
int k;
int h[N], e[M], ne[M], w[M], idx;
void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
struct node
{
	int dis, u;
	bool operator > (const node& a) const
	{
		return dis > a.dis;
	}
};
int dis[N], vis[N];
priority_queue<node, vector<node>, greater<node> >q;
void dijkstra(int s)
{
	memset(dis, 63, sizeof(dis));
	dis[s] = 0;
	q.push({ 0,s });
	while (!q.empty())
	{
		int u = q.top().u;
		q.pop();
		if (vis[u])continue;
		vis[u] = 1;
		for (int i = h[u]; ~i; i = ne[i])
		{
			int v = e[i], c = w[i];
			if (dis[v] > dis[u] + c)
			{
				dis[v] = dis[u] + c;
				if (!vis[v])q.push({ dis[v],v });
			}
		}
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d%d", &n, &m, &k);
	int s, t;
	scanf("%d%d", &s, &t);
	for (int i = 1; i <= m; i++)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		add(a, b, c), add(b, a, c);
		for (int j = 1; j <= k; j++)
		{
			add(a + n * (j - 1), b + n * j, 0);
			add(b + n * (j - 1), a + n * j, 0);
			add(a + n * j, b + n * j, c);
			add(b + n * j, a + n * j, c);
		}
	}
	for (int i = 1; i <= k; i++)
		add(t + n * (i - 1), t + n * i, 0);
	dijkstra(s);
	printf("%d\n", dis[t + n * k]);
	return 0;
}
```

