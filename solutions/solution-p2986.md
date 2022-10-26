---
title: P2986 [USACO10MAR] Great Cow Gathering 题解
date: 2022-10-11
tags:
	- 题解
	- 换根DP
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Great Cow Gathering</div>
<div id="problem-info-from">USACO 2010 March Gold</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2986">Luogu P2986</a></li></ul></div>

----

简述一下题目：
题目给定了一张带有边权的树，要求我们对于每一个点 $u$ 求出 $\displaystyle \sum_{v \in S} c_v \times \operatorname{dis}(u,v)$，并输出所有点的答案中的最小值。

首先我们可以很简单地通过一次DFS求出来树根的答案，就是 $\displaystyle \sum_{v \in S} c_v \times \operatorname{dis}(1,v)$。

对于每一个节点，我们记录两个信息：该节点的答案`ans`和其子树内的 $c_i$ 之和`sum`。
当我们DFS回溯到该节点 $u$ 的时候，我们的答案就是 $\displaystyle \sum_{v \in \{\text{son of} u\}} \operatorname{ans}(v) + \operatorname{sum}(v) \times \operatorname{dis}(u,v)$。

为了不与最终的答案冲突，这里将初步的答案命名成了`dis`。

``` cpp 第一次DFS代码
ll sum[N], dis[N];
ll ans[N];
void dfs1(int p, int fa)
{
	sum[p] = c[p];
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		dfs1(e[i], p);
		sum[p] += sum[e[i]];
		dis[p] += dis[e[i]] + sum[e[i]] * w[i];
	}
}
```

然后考虑怎么转移到其他节点。
这里就可以使用换根DP来求解了。

考虑我们从一个节点转移到其的一个子节点。
当前我们已经求出了当前这个节点的答案，此时我们所有的奶牛已经到达了当前的节点。
我们转移到其子节点的时候，可以等价成把所有的奶牛移动到这个子节点。
那么在该子节点子树中的所有奶牛都少走了 $(u,v)$ 这个边，而剩下的所有奶牛都多走了这一条边。
那么我们第二遍DFS的代码如下：

``` cpp 第而次DFS代码
void dfs2(int p, int fa)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		ans[e[i]] = ans[p] + (k - 2 * sum[e[i]]) * w[i];
		dfs2(e[i], p);
	}
}
```

然后我们的答案就出来了，求个最小值输出即可。

完整代码如下：

``` cpp
#define _CRT_SECURE_NO_WARNINGS
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = N << 1;
int n, m;
int k;
int h[N], e[M], ne[M], idx;
int c[N], w[M];
void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
ll sum[N], dis[N];
ll ans[N];
void dfs1(int p, int fa)
{
	sum[p] = c[p];
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		dfs1(e[i], p);
		sum[p] += sum[e[i]];
		dis[p] += dis[e[i]] + sum[e[i]] * w[i];
	}
}
void dfs2(int p, int fa)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		ans[e[i]] = ans[p] + (k - 2 * sum[e[i]]) * w[i];
		dfs2(e[i], p);
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d", &n);
	for(int i = 1; i <= n; i++)
	{
		scanf("%d", &c[i]);
		k += c[i];
	}
	for(int i = 1; i < n; i++)
	{
		int u, v, w;
		scanf("%d%d%d", &u, &v, &w);
		add(u, v, w), add(v, u, w);
	}
	dfs1(1, 0);
	ans[1] = dis[1];
	dfs2(1, 0);
	ll res = INT64_MAX;
	for(int i = 1; i <= n; i++)
		res = min(res, ans[i]);
	printf("%lld\n", res);
	return 0;
}
```

