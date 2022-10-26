---
title: P2446 [SDOI2010] 大陆争霸 题解
date: 2022-10-07
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
<div id="problem-info-name">大陆争霸</div>
<div id="problem-info-from">SDOI 2010</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2446">Luogu P2446</a></li><li><a href="https://www.acwing.com/problem/content/2497/">AcWing 2495</a></li><li><a href="https://darkbzoj.cc/problem/1922">BZOJ #1922</a></li></ul></div>

----

一道看上去显然是最短路，但是实际上并不是那么好想出来的最短路。

因为我们有无限多个机器人，所以我们可以假设我们向每一个城市都派出了无限多个机器人，这样足以解除所有的结界并摧毁该城市。

首先，我们每一个城市可以记录下来两个时间：机器人到达其的时间和其不再受结界保护了的时间。
我们最终的答案就是两者取最大值。

机器人到达其的时间我们可以用最短路来求解，不再受结界保护了的时间就是保护其的城市中最晚被到达的那一个的到达时间。

同时我们需要魔改一下最短路的算法，像拓扑排序一样统计一下每一个节点的入度，只不过这里的「入度」只针对于保护与被保护的关系。
一旦某一个节点被完全暴露在我们面前，我们就可以将其加入队列中，后面再继续访问之。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 10010, M = 4000010;
int n, m, k;
int h[N], e[M], ne[M], idx;
ll w[M];
void add(int a, int b, ll c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
ll arr[N], acc[N];
vector<int>pro[N];
int ind[N];
struct Node
{
	int u;
	ll dis;
	bool operator > (const Node &a) const
	{
		return dis > a.dis;
	}
};
ll dis[N], vis[N];
priority_queue<Node, vector<Node>, greater<Node>>q;
void dijkstra(int s)
{
	memset(dis, 63, sizeof(dis));
	memset(arr, 63, sizeof(arr));
	dis[s] = arr[s] = acc[s] = 0;
	q.push({ s,0 });
	while(!q.empty())
	{
		int p = q.top().u;
		q.pop();
		if(vis[p])continue;
		vis[p] = 1;
		for(int i = h[p]; ~i; i = ne[i])
		{
			int v = e[i];
			if(arr[v] > dis[p] + w[i])
			{
				arr[v] = dis[p] + w[i];
				if(!ind[v])
				{
					dis[v] = max(acc[v], arr[v]);
					q.push({ v,dis[v] });
				}
			}
		}
		for(int v : pro[p])
		{
			acc[v] = max(acc[v], dis[p]);
			ind[v]--;
			if(!ind[v])
			{
				dis[v] = max(acc[v], arr[v]);
				q.push({ v,dis[v] });
			}
		}
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= m; i++)
	{
		int u, v;
		ll w;
		scanf("%d%d%lld", &u, &v, &w);
		add(u, v, w);
	}
	for(int i = 1; i <= n; i++)
	{
		int k, x;
		scanf("%d", &k);
		while(k--)
		{
			scanf("%d", &x);
			ind[i]++;
			pro[x].push_back(i);
		}
	}
	dijkstra(1);
	printf("%lld\n", dis[n]);
	return 0;
}
```

