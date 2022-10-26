---
title: "S2OJ #1586. 口粮输送 题解"
date: 2022-09-26
tags:
	- 题解
	- 最小生成树
	- 状压DP
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">口粮输送</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1586">S2OJ #1586</a></li></ul></div>

----

首先我们可以想到一个结论。
我们手模一下就可以知道，如果一个图有合法的解，我们一定可以构造出来一个合法的解，保证其中用到的每一条边都最多被走过一次。

基于这个，我们可以构造出来一个基于贪心的方案。
我们构建出来一个最小生成树，然后从叶子结点开始运输口粮。
如果当前节点缺口粮，就从父节点运过来；而如果当前节点有富余的口粮，那就无论怎样都向上输送，直到最终该节点的口粮数量与需要的口粮数量相等为止。
基于这个规则，我们到达根节点的时候，已经满足了出根节点之外所有节点的要求了。
如果此时根节点也满足要求的话，我们就可以判定这个方案是一个合法的解。

但是，这个方案是错误的。
因为我们很可能可以找到不止一棵最小生成树，而很可能你找到的这一棵正好不满足要求。

况且，我们可以不使用生成树上的所有边。

我们观察如何可以使一个点集 $S$ 满足我们的要求：

1. 其本身 $S$ 满足要求。
2. 其可以被分成两个点集 $T_1$ 与 $T_2$，其中 $T_1 \cap T_2 = \varnothing$，$T_1 \cup T_2 = S$。

这样，我们可以将 $S$ 不断细分，将其分割成数个连通块。
这些连通块内部是符合要求的，就像我们之前说的那个贪心方案一样，基于其最小生成树。
但是也有不同的地方。
我们不再需要通过DFS来判断是否可行，因为我们需要保证这个连通块的最小生成树中的每一条边都被用到了。
其剩余的口粮就是 $\sum\limits_{i \in T} a_i - \sum\limits_{i \in T} b_i - \sum\limits_{e \in MST(T)} w_e$，我们需要满足这个数字大于零。
由于我们的 $n \leq 20$，我们就可以枚举子集来求出是否有合法方案。

总时间复杂度 $O(T3^n)$。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = N << 1;
int n, m;
struct Edge
{
	int u, v;
	ll w;
	bool operator < (const Edge &rhs) const
	{
		return w < rhs.w;
	}
};
Edge e[M];
int nd[N], hv[N];
bool f[N], vld[N];
int p[N];
int find(int x)
{
	if(p[x] != x)p[x] = find(p[x]);
	return p[x];
}
bool chq(int S)
{
	vector<Edge>v;
	for(int i = 1; i <= m; i++)
		if((S >> (e[i].u - 1) & 1) && (S >> (e[i].v - 1) & 1))
			v.push_back(e[i]);
	sort(v.begin(), v.end());
	for(int i = 1; i <= n; i++)p[i] = i;
	ll sum = 0;
	for(int i = 1; i <= n; i++)
		if(S >> (i - 1) & 1)
			sum += hv[i] - nd[i];
	for(auto i : v)
	{
		int u = find(i.u), v = find(i.v);
		if(u != v)
		{
			p[u] = v;
			sum -= i.w;
		}
	}
	int rt = 0;
	for(int i = 1; i <= n; i++)
		if(S >> (i - 1) & 1)
			if(p[i] == i) { rt = i; break; }
	for(int i = 1; i <= n; i++)
		if(S >> (i - 1) & 1)
			if(p[i] != rt)return false;
	return sum >= 0;
}
void solve()
{
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= m; i++)
	{
		int u, v; ll w;
		scanf("%d%d%lld", &u, &v, &w);
		e[i] = { u,v,w };
	}
	for(int i = 1; i <= n; i++)
		scanf("%d%d", &hv[i], &nd[i]);
	for(int S = 1; S < (1 << n); S++)
		f[S] = false, vld[S] = chq(S);
	f[0] = true;
	for(int S = 1; S < (1 << n); S++)
		for(int T = S; T; T = (T - 1) & S)
			if(vld[T] && f[S ^ T])f[S] = true;
	if(f[(1 << n) - 1])puts("Yes");
	else puts("No");
	return;
}
int main()
{
	int T;
	scanf("%d", &T);
	while(T--)
	{
		solve();
	}
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>