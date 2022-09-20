---
title: P3174 [HAOI2009] 毛毛虫 题解
date: 2022-09-10
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
<div id="problem-info-name">毛毛虫</div>
<div id="problem-info-from">HAOI 2009</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3174">Luogu P3174</a></li></ul></div>

----

首先解读题目中的信息。
一条「毛毛虫」是由树上的一条链以及与该链直接相连但不在链上的节点组成的。
题目要求我们找出包含节点数最多的一条毛毛虫。

我们以此做一个DP。

为叙述方便，下面「最长链」的意义是包含节点数最多的「毛毛虫」，「次长链」代表包含节点次多的「毛毛虫」。
为区分正常意义与此处的意义，使用上面意义的词语会用直角括号包裹。

不管是什么树形DP我们都得DFS来遍历每一个点。
对于每一个点，我们记录下来其子树中以其为链顶的「最长链」，然后随着不断回溯不断更新每一个节点的「最长链」的值。

当然，答案链可能是两条竖直向上的链拼接在一起得到的，所以我们还需要记录以当前点为链顶的「次长链」，然后统计出来每一个点的答案，最后取较大的值。

<big>**代码实现：**</big>

对于每一个节点，我们记录一个`maxn`，一个`maxm`，和一个`sson`。
分别代表的是其所有以其子节点为链顶的「最长链」（但不含该节点）的最大值，
所有以其子节点为链顶的「次长链」（但不含该节点）的最大值，和该节点子节点的个数。
每一个节点的答案就是`maxn + maxm + sson + 1`。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 300010, M = N << 1;
int n, m;
int h[N], e[M], ne[M], idx;
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
int maxn[N], maxm[N];
int stre[N];
void dfs(int p, int fa)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		dfs(e[i], p);
		stre[p]++;
		int v = maxn[e[i]] + stre[e[i]];
		if(v > maxn[p])maxm[p] = maxn[p], maxn[p] = v;
		else maxm[p] = max(maxm[p], v);
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	for(int i = 1; i < n; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		add(u, v), add(v, u);
	}
	dfs(1, 0);
	int ans = 0;
	for(int i = 1; i <= n; i++)
		ans = max(ans, maxn[i] + maxm[i] + stre[i] + 1 + (i != 1));
	printf("%d\n", ans);
	return 0;
}
```
















































































































