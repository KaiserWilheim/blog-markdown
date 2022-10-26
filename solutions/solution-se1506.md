---
title: "S2OJ #1506. Antifloyd 题解"
date: 2022-08-26
tags:
	- 题解
	- 图论
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Antifloyd</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1506">S2OJ #1506</a></li></ul></div>

----

题目大意是，给定一个有向图的传递闭包，要求在保证该传递闭包不变的情况下用最少的边构造出来一个有向图，并输出其所用边数。

我们考虑传递闭包是怎么生成的，然后将这个过程倒过来做就可以了。

传递闭包代表着无向图的连通性关系。如果从一个点 $i$ 可以到达另一个点 $j$，那么在传递闭包上就会有 $i$ 到 $j$ 的一条边。

考虑三个点：$x$，$y$ 和 $z$。
如果我们现在有 $x$ 到 $y$ 的边和 $y$ 到 $z$ 的边，那么做完传递闭包之后我们就会有 $x$ 到 $z$ 的边。

这里我将两种不同的边用颜色进行了一个区分。
原有的边用蓝色表示，新的边用红色表示。

![se1506-1.png](https://s2.loli.net/2022/08/26/NhaeRXiyjfIPxc8.png)

这样我们可以将红色的边删去，同时还能保证图的传递闭包不变。
同时我们可以知道，所有的红色边都满足这样一个性质。
这样使得我们可以枚举点来找到红色边并删去。

而对于强连通分量则不一定了：
考虑四个点：$x$，$y$，$z$ 和 $w$。
同时传递闭包已经给出来了：

![se1506-2.png](https://s2.loli.net/2022/08/26/pw8FGNX14HqmPsA.png)

此时我们就不能通过简单的枚举法来找出哪些边是红色的边了。
因为你枚举出来的结果可能是全都是，导致强连通分量内部完全不连通。

但是对于强连通分量，我们可以给其一种替代方案——成环。
成环保证了其强连通分量的本质，同时使得其内部边数最少。

那么我们就可以跑Tarjan来缩点。
缩点之后，我们得到的就是一个DAG。
排除了SCC的影响之后，我们就可以 $O(n^3)$ 枚举点了。

最后我们只需要统计出来剩下的边数和所有大小不为1的SCC大小之和就是答案。

代码如下：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 250;
int n, m;
int g[N][N], G[N][N];
int dfn[N], low[N], cnt;
int sta[N], tt;
bool vis[N];
int scc[N], sc, sz[N];
void tarjan(int p)
{
	low[p] = dfn[p] = ++cnt;
	sta[++tt] = p, vis[p] = true;
	for(int i = 1; i <= n; i++)
	{
		if(!g[p][i] || i == p)continue;
		if(!dfn[i])
		{
			tarjan(i);
			low[p] = min(low[p], low[i]);
		}
		else if(vis[i])
		{
			low[p] = min(low[p], dfn[i]);
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
int main()
{
	scanf("%d", &n);
	for(int i = 1; i <= n; i++)
		for(int j = 1; j <= n; j++)
			scanf("%d", &g[i][j]);
	for(int i = 1; i <= n; i++)
		if(!dfn[i])tarjan(i);
	if(sc == 1)
	{
		printf("%d\n", sz[sc]);
		return 0;
	}
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= n; j++)
		{
			if(!g[i][j])continue;
			G[scc[i]][scc[j]] = 1;
		}
	}
	for(int i = 1; i <= sc; i++)
	{
		for(int j = 1; j <= sc; j++)
		{
			if(!G[i][j] || i == j)continue;
			for(int k = 1; k <= sc; k++)
			{
				if(!G[j][k] || i == k || j == k)continue;
				G[i][k] = 0;
			}
		}
	}
	int ans = 0;
	for(int i = 1; i <= sc; i++)
		for(int j = 1; j <= sc; j++)
			if(i != j && G[i][j])ans++;
	for(int i = 1; i <= sc; i++)
		if(sz[i] != 1)ans += sz[i];
	printf("%d\n", ans);
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>