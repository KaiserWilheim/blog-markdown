---
title: CF949C Data Center Maintenance 题解
date: 2022-10-02
tags:
	- 题解
	- 建图
	- 图的连通性
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Data Center Maintenance</div>
<div id="problem-info-from">Codeforces Round #469 (Div. 1)</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/CF949C">Luogu CF949C</a></li><li><a href="https://codeforces.com/problemset/problem/949/C">CF 949 C</a></li></ul></div>

----

题目要求我们一天中的任何时候都可以下载到所有文件，这就需要我们存储同一份文件的两台服务器不能同时维护。

因为我们想要把某一个服务器的维护天窗强制向后推一个小时，那么我们就可以将这个时间的冲突转化为同余式 $x_u + 1 \equiv x_v \pmod{h}$。

通过上面的说明，我们还可以发现我们这个强制向后推的关系不是双向的，因为 $x_u + 1 \equiv x_v \pmod{h}$ 绝对不会使得 $x_v + 1$ 也 $\equiv x_u \pmod{h}$。

那我们就可以建立单向边，然后跑tarjan求出强连通分量并缩点。
因为强连通分量里面的每一个点都可以到达其他点，那么一旦这个点推迟了，其他所有点都需要推迟。

我们需要最小化需要推迟天窗的服务器数量，也就是需要最小化一个点能够到达的点的数量。
我们并不需要大费周章地统计出来缩点后每一个点能够到达的点的数量（当然这个数量是原图中点的数量），因为该图远没有那么稠密，只需要找到出度为0的点中最小的任意一个并输出之即可。
至少数据全都造成了这样，如果图稠密一点的话可能就需要统计出来每一个点能够到达的点集了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = 400010;
int n, m, k;
int h[N], e[M], ne[M], idx;
int mnt[N];
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
int low[N], dfn[N], cnt;
int sta[N], tt;
bool vis[N];
int scc[N], sz[N], sc;
void tarjan(int p)
{
	low[p] = dfn[p] = ++cnt;
	sta[++tt] = p, vis[p] = true;
	for(int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if(!dfn[j])
		{
			tarjan(j);
			low[p] = min(low[p], low[j]);
		}
		else if(vis[j])
		{
			low[p] = min(low[p], dfn[j]);
		}
	}
	if(dfn[p] == low[p])
	{
		++sc;
		while(sta[tt] != p)
		{
			scc[sta[tt]] = sc;
			sz[sc]++;
			vis[sta[tt]] = false;
			tt--;
		}
		scc[sta[tt]] = sc;
		sz[sc]++;
		vis[sta[tt]] = false;
		tt--;
	}
}
int oud[N];
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d%d", &n, &m, &k);
	for(int i = 1; i <= n; i++)
		scanf("%d", &mnt[i]);
	for(int i = 1; i <= m; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		if((mnt[u] + 1) % k == mnt[v] % k)add(u, v);
		if((mnt[v] + 1) % k == mnt[u] % k)add(v, u);
	}
	for(int i = 1; i <= n; i++)
		if(!dfn[i])tarjan(i);
	for(int i = 1; i <= n; i++)
	{
		int u = scc[i];
		for(int j = h[i]; ~j; j = ne[j])
		{
			int v = scc[e[j]];
			if(u != v)oud[u]++;
		}
	}
	int ans = 0;
	sz[0] = 998244353;
	for(int i = 1; i <= sc; i++)
	{
		if(oud[i] == 0)
		{
			ans = (sz[ans] > sz[i]) ? i : ans;
		}
	}
	printf("%d\n", sz[ans]);
	for(int i = 1; i <= n; i++)
		if(scc[i] == ans)printf("%d ", i);
	putchar('\n');
	return 0;
}
```

