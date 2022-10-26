---
title: P6419 [COCI2014-2015#1] Kamp 题解
date: 2022-10-05
tags:
	- 题解
	- 树形DP
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Kamp</div>
<div id="problem-info-from">COCI2014-2015#1</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P6419">Luogu P6419</a></li><li><a href="https://loj.ac/p/2806">LibreOJ L2806</a></li></ul></div>

----

首先我们需要注意一点，就是这个车可以装下所有人，所以我们不需要每一次都返回开始的那个点。
其次，我们在送完最后一个人之后也不需要返回开始的那个点。

所以，我们需要一个树形DP来维护这些东西。
我们最终的答案就是遍历所有有值的点再回到当前节点的路程，再减去以当前节点为链顶的，到有值的点最长链。
显而易见的，我们需要维护两个值：遍历当前点子树内所有有值的点再返回当前节点需要走的路程，以及以该节点为链顶的，到有值的点的最长链。

DFS如下：

``` cpp 第一遍DFS
ll sz[N], flen[N], slen[N], son[N];
ll f[N], g[N];
void dfs1(int p, int fa)
{
	sz[p] = c[p];
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		int v = e[i];
		dfs1(v, p);
		sz[p] += sz[v];//子树内有值的节点个数
		if(sz[v])//子节点没有值的的话就不需要更新了
		{
			g[p] += g[v] + 2 * w[i];//一条边只需要走两次
			if(flen[v] + w[i] >= flen[p])//更新最长链
				slen[p] = flen[p], flen[p] = flen[v] + w[i], son[p] = v;
			else slen[p] = max(slen[p], flen[v] + w[i]);
		}
	}
}
```

当我们从1号节点结束了第一遍DFS之后，1号节点的答案就直接出来了。

然后，我们可以再进行一遍DFS，用父节点的答案更新子节点的答案。

更新节点答案的时候，我们有三种可能：

1. 该子节点子树内没有有值的点。
	那么 $f_v = f_u + 2 w_i$。
	最长链不需要更新，因为下面没有有值的点。
2. 该子节点子树外没有有值的点。
	那么 $f_v$ 就可以直接继承第一遍DFS时的答案。
	最长链也不需要更新，因为父节点没有最长链来更新。
3. 一般情况。

一般情况就麻烦许多了。

其答案好说，$f_v = f_u + 2w_i - 2w_i$，化简一下得 $f_v = f_u$。
然而最长链不好维护。
根据维护最长链时的通用做法，我们维护最长链的同时还需要维护一个次长链，以防父节点的最长链刚好会经过该节点，造成答案错误。
所以我们需要在记录最长链的同时记录一下次长链和最长链经过的儿子节点。

``` cpp 第二遍DFS
void dfs2(int p, int fa)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		int v = e[i];
		if(!sz[v])f[v] = f[p] + 2 * w[i], flen[v] = flen[p] + w[i];
		else if(m == sz[v])f[v] = g[v];
		else
		{
			f[v] = f[p];
			if(flen[v] < flen[p] + w[i] && son[p] != v)
				slen[v] = flen[v], flen[v] = flen[p] + w[i], son[v] = p;
			else if(flen[v] < slen[p] + w[i])
				slen[v] = flen[v], flen[v] = slen[p] + w[i], son[v] = 1;
			else if(slen[v] < flen[p] + w[i] && son[p] != v)
				slen[v] = flen[p] + w[i];
			else if(slen[v] < slen[p] + w[i])
				slen[v] = slen[p] + w[i];
		}
		dfs2(v, p);
	}
}
```

然后直接输出该节点为根的答案减去最长链的长度即可。

代码如下：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 500010, M = N << 1;
int n, m;
int h[N], e[M], ne[M], idx;
ll w[M], c[N];
void add(int a, int b, ll c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
ll sz[N], flen[N], slen[N], son[N];
ll f[N], g[N];
void dfs1(int p, int fa)
{
	sz[p] = c[p];
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		int v = e[i];
		dfs1(v, p);
		sz[p] += sz[v];
		if(sz[v])
		{
			g[p] += g[v] + 2 * w[i];
			if(flen[v] + w[i] >= flen[p])
				slen[p] = flen[p], flen[p] = flen[v] + w[i], son[p] = v;
			else slen[p] = max(slen[p], flen[v] + w[i]);
		}
	}
}
void dfs2(int p, int fa)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		int v = e[i];
		if(!sz[v])f[v] = f[p] + 2 * w[i], flen[v] = flen[p] + w[i];
		else if(m == sz[v])f[v] = g[v];
		else
		{
			f[v] = f[p];
			if(flen[v] < flen[p] + w[i] && son[p] != v)
				slen[v] = flen[v], flen[v] = flen[p] + w[i], son[v] = p;
			else if(flen[v] < slen[p] + w[i])
				slen[v] = flen[v], flen[v] = slen[p] + w[i], son[v] = 1;
			else if(slen[v] < flen[p] + w[i] && son[p] != v)
				slen[v] = flen[p] + w[i];
			else if(slen[v] < slen[p] + w[i])
				slen[v] = slen[p] + w[i];
		}
		dfs2(v, p);
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	for(int i = 1; i < n; i++)
	{
		int a, b;
		ll c;
		scanf("%d%d%lld", &a, &b, &c);
		add(a, b, c), add(b, a, c);
	}
	for(int i = 1; i <= m; i++)
	{
		int x;
		scanf("%d", &x);
		c[x]++;
	}
	dfs1(1, 0);
	f[1] = g[1];
	dfs2(1, 0);
	for(int i = 1; i <= n; i++)
		printf("%lld\n", f[i] - flen[i]);
	return 0;
}
```

