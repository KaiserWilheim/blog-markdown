---
title: P3047 [USACO12FEB] Nearby Cows 题解
date: 2022-09-08
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
<div id="problem-info-name">Nearby Cows</div>
<div id="problem-info-from">USACO 2012 February Gold</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3047">Luogu P3047</a></li></ul></div>

----

简述一下题意，题目要求我们对于每一个点求出距离其不超过 $k$ 的点的点权和。

操作很简单，我们只需要开一个DP数组，记录下来每一个点与其距离为 $j \in \\{ j|[0,k] \\}$ 的点权和即可。

首先我们DFS一遍，记录下来每一个点子树中与其距离为 $j \in \\{ j|[0,k] \\}$ 的点权和：

``` cpp 第一遍DFS
void dfs1(int p, int fa)
{
	f[p][0] = w[p];
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		dfs1(e[i], p);
		for(int j = 0; j < k; j++)
			f[p][j + 1] += f[e[i]][j];
	}
}
```

然后考虑其祖先节点对其的影响。

我们通过再一次的DFS来统计影响，所以我们只需要计算该节点的父亲对其的影响即可。

举个例子：
我们现在节点 $x$ 统计答案，其父亲为 $y$。
对于每一个与 $y$ 相距 $j \in \\{ j|[0,k-1] \\}$ 的节点，其与 $x$ 的距离为 $j+1$。

根据这个我们可以统计出 $y$ 对 $x$ 的影响，然后就可以得到答案了。

``` cpp 第二遍DFS
void dfs2(int p, int fa)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		for(int j = k; j >= 2; j--)
			f[e[i]][j] -= f[e[i]][j - 2];
		for(int j = 0; j < k; j++)
			f[e[i]][j + 1] += f[p][j];
		dfs2(e[i], p);
	}
}
```

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = N << 1;
int n, m;
int k;
int h[N], e[M], ne[M], idx;
int w[N];
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
int f[N][30];
void dfs1(int p, int fa)
{
	f[p][0] = w[p];
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		dfs1(e[i], p);
		for(int j = 0; j < k; j++)
			f[p][j + 1] += f[e[i]][j];
	}
}
void dfs2(int p, int fa)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		for(int j = k; j >= 2; j--)
			f[e[i]][j] -= f[e[i]][j - 2];
		for(int j = 0; j < k; j++)
			f[e[i]][j + 1] += f[p][j];
		dfs2(e[i], p);
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &k);
	for(int i = 1; i < n; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		add(u, v), add(v, u);
	}
	for(int i = 1; i <= n; i++)
		scanf("%d", &w[i]);
	dfs1(1, 0);
	dfs2(1, 0);
	for(int i = 1; i <= n; i++)
	{
		int sum = 0;
		for(int j = 0; j <= k; j++)sum += f[i][j];
		printf("%d\n", sum);
	}
	return 0;
}
```

