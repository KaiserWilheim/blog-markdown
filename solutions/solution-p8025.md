---
title: P8025 [ONTAK2015] Związek Harcerstwa Bajtockiego 题解
date: 2022-07-22
tags:
	- 题解
	- 树链剖分
	- 最近公共祖先
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Związek Harcerstwa Bajtockiego</div>
<div id="problem-info-from">ONTAK2015</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P8025">Luogu P8025</a></li><li><a href="https://darkbzoj.cc/problem/4281">BZOJ #4281</a></li></ul></div>

----

这道题的题意已经很简洁了，我们就不需要解释了。

对于每一次操作，我们可以分为两种情况：能到达与不能到达。

能到达的直接跳到对应的点即可，不能到达的就直接模拟直接跳就可以了。

因为数据范围是 $10^6$ 级别的，我们尝试优化复杂度。

首先我们可以使用树剖来求出来两个节点的LCA，这样我们就可以快速判断两者之间的距离。

对于跳不到的情况我们还可以继续往下分为三种情况：跳不到LCA，刚好跳到LCA和跳过了LCA。

对于第一种情况，我们直接跳重链直到跳不动重链为止，然后利用重链上DFS序连续的性质直接输出结束节点编号；
对于第二种情况直接输出LCA；
对于第三种情况，我们从目标节点开始向上跳 $dis(x,y)-t$ 步即可，而具体的实现与第一种情况无异。

至此，我们就完美解决了所有问题。

总复杂度为 $O(n \log n)$。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010, M = N << 1;
int n, m;
int h[N], e[M], ne[M], idx;
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
int dep[N], fa[N], sz[N], son[N];
int id[N], nw[N], top[N], cnt;
void dfs1(int p, int father, int depth)
{
	dep[p] = depth, fa[p] = father, sz[p] = 1;
	for (int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if (j == father)continue;
		dfs1(j, p, depth + 1);
		sz[p] += sz[j];
		if (sz[j] > sz[son[p]])son[p] = j;
	}
}
void dfs2(int p, int t)
{
	id[p] = ++cnt, nw[cnt] = p, top[p] = t;
	if (!son[p])return;
	dfs2(son[p], t);
	for (int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if (j == fa[p] || j == son[p])continue;
		dfs2(j, j);
	}
}
int lca(int p, int q)
{
	while (top[p] != top[q])
	{
		if (dep[top[p]] < dep[top[q]])swap(p, q);
		p = fa[top[p]];
	}
	if (dep[p] < dep[q])swap(p, q);
	return q;
}
int jump(int p, int k)
{
	while (dep[p] - dep[top[p]] + 1 <= k)
	{//当可以跳完整条重链
		k -= dep[p] - dep[top[p]] + 1;
		p = fa[top[p]];
	}
	return nw[id[p] - k];
}
int main()
{
	memset(h, -1, sizeof(h));
	int u;
	scanf("%d%d%d", &n, &u, &m);
	for (int i = 1; i < n; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		add(u, v), add(v, u);
	}
	dfs1(1, 0, 1);
	dfs2(1, 1);
	while (m--)
	{
		int v, step;
		scanf("%d%d", &v, &step);
		int x = lca(u, v);
		int dis1 = dep[u] - dep[x], dis2 = dep[v] - dep[x];
		if (step >= dis1 + dis2)
		{//能跳到
			u = v;
			printf("%d ", u);
		}
		else if (step == dis1)
		{//只能跳到LCA
			u = x;
			printf("%d ", u);
		}
		else if (step < dis1)
		{//连LCA都跳不到
			u = jump(u, step);
			printf("%d ", u);
		}
		else
		{//能跳过LCA
			u = jump(v, dis2 - (step - dis1));
			printf("%d ", u);
		}
	}
	return 0;
}
```
