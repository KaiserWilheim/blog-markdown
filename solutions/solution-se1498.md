---
title: "S2OJ #1498. 换乘 题解"
date: 2022-08-26
tags:
	- 题解
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">换乘</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1498">S2OJ #1498</a></li></ul></div>

----

我们一眼就可以看出来可以分层图跑最短路。
当然，是忽略数据范围的情况下。

首先看一下分层图跑最短路怎么建图。

每一个颜色单独建一层。每一层的内部连对应颜色的费用为0的边，同时层间对应的点之间也连边，费用为1。
1号点和n号点对应层之间费用为0。
双向边。
然后跑1到n的最短路即可。

不过这样点数和边数都会超标。

我们进行一些简化。

首先，我们将原图中的点建立出来。将原图中的每一个点与所有带着颜色的对应点连去1回0的双向边。
然后将原来的分层图中哪些没有在边中出现过的点删去，只保留有边连着的点。
这些非原图中的点需要进行一发离散化，以保证空间的充分利用。
这样可以大幅度减少我们的空间用量，在不保证连通性的情况下最多可以拿出来 $2 \times 10^6$ 个点和 $3 \times 10^6$ 条边。

答案就是1号点到n号点的最短路。

因为我们的边的边权只有0和1，我们可以跑BFS，时间复杂度是 $O(n+m)$。

下面是跑最短路的代码，也是可以过的。

``` cpp
#include <bits/stdc++.h>
#include <unordered_map>
using namespace std;
const int N = 1000010, M = 4000010;
int n, m;
int h[N], e[M], ne[M], w[M], idx;
void add(int a, int b, int c)
{
	e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx++;
}
struct node
{
	int dis, u;
	bool operator > (const node &a) const
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
	while(!q.empty())
	{
		int u = q.top().u;
		q.pop();
		if(vis[u])continue;
		vis[u] = 1;
		for(int i = h[u]; ~i; i = ne[i])
		{
			int v = e[i], c = w[i];
			if(dis[v] > dis[u] + c)
			{
				dis[v] = dis[u] + c;
				if(!vis[v])q.push({ dis[v],v });
			}
		}
	}
}
vector<int>vec[N];
map<pair<int, int>, int>dic;
int cnt;
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	cnt = n;
	for(int i = 1; i <= m; i++)
	{
		int u, v, p;
		scanf("%d%d%d", &u, &v, &p);
		if(!dic.count(make_pair(u, p)))
		{
			dic.insert(make_pair(make_pair(u, p), ++cnt));
			vec[u].push_back(cnt);
		}
		if(!dic.count(make_pair(v, p)))
		{
			dic.insert(make_pair(make_pair(v, p), ++cnt));
			vec[v].push_back(cnt);
		}
		int uu = dic.find(make_pair(u, p))->second;
		int vv = dic.find(make_pair(v, p))->second;
		add(uu, vv, 0), add(vv, uu, 0);
	}
	for(int i = 1; i <= n; i++)
	{
		if(vec[i].empty())continue;
		for(int u : vec[i])
			add(i, u, 1), add(u, i, 0);
	}
	dijkstra(1);
	if(dis[n] == 0x3f3f3f3f)puts("-1");
	else printf("%d\n", dis[n]);
	return 0;
}
```

