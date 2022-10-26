---
title: "S2OJ #1541. 单调栈 题解"
date: 2022-09-12
tags:
	- 题解
	- 建图
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">单调栈</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/996">S2OJ #996</a></li></ul></div>

----

题目给定了一个长度为 $n$ 的「单调栈序列」，然后需要我们求出有多少个长度为 $n$ 的序列满足该序列的「单调栈序列」与题目给出的相同，同时每一个元素都在 $[1,m]$ 之间。

一个「单调栈序列」由 $n$ 个数构成，每一个数 $b_i$ 代表着对应位置的元素 $a_i$ 在加入单调栈后下面的元素的序号，也就是其前面第一个大于其的数字的序号。
如果 $b_i$ 为0代表着前面没有比它大的。

题目中给出的「单调栈序列」并不完整，缺失的地方用-1表示。

我们考虑根据这个序列建立一棵树。每一个点都向第一个大于其的数字连边，把其当做自己的父亲节点，同时把0也当成一个节点，这样就有 $n+1$ 个点和 $n$ 条边了，可以构成一棵树。
我们在这棵树上跑DP，就可以得出符合要求的数字个数了。

然后就是处理-1的情况了。
我们先将所有 $b_i=-1$ 的 $fa_i$ 设成0，至于为什么后面再说。

对于一个下标 $i$，我们将其与其对应的 $b_i$ 之间的这一段区间拎出来。
我们找到其中 $fa_j < b_i$ 的这些 $j$，手模一下可以发现这种 $j$ 一定是 $b_j = -1$ 的类型。

> Q：为什么？
> A：因为对于一个 $i$、$j$，满足 $i>j$ 且 $b_i > b_j$，这就表明 $a_{b_j}$ 肯定是大于 $a_{b_i}$ 的，因为 $j$ 选择的是 $a_{b_j}$ 而非 $a_{b_i}$。同时，这也说明 $a_j > a_{b_i}$。于是我们就可以得到一串不等式：$a_{b_j} > a_j > a_{b_i} > a_i$。那么 $i$ 就应该选择 $a_j$ 而非 $a_{b_i}$，而这与我们已知条件相左。
而我们将所有满足 $b_j = -1$ 的 $fa_j$ 都设成了0，那么要么 $j$ 这个位置无拘无束，要么 $a_j \leq a_i$。

所以我们遍历一遍 $(b_i,i)$ 这个区间，将所有满足 $fa_j < b_i$ 的 $fa_j$ 都设成 $i$。

同时我们需要区分边的类型，因为有些边是代表大于的，有些边是代表大于等于的。

下面是代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 310, M = 100010;
const ll mod = 1e9 + 7;
int n, m;
int a[N], b[N];
int fa[N];
ll dp[N][M];
ll ans;
int h[N], e[N], ne[N], idx;
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
void dfs(int u)
{
	for(int i = 1; i <= m + 1; i++)
		dp[u][i] = 1;
	for(int i = h[u]; ~i; i = ne[i])
	{
		int v = e[i];
		dfs(v);
		for(int j = 1; j <= m + (u == 0); j++)
		{
			if(j - a[v] < 0)continue;
			dp[u][j] = (dp[u][j] * dp[v][j - a[v]]) % mod;
		}
	}
	if(u != 0)
		for(int i = 1; i <= m + 1; i++)
			dp[u][i] = (dp[u][i] + dp[u][i - 1]) % mod;
}
int main()
{
	memset(h, -1, sizeof(h));
	const char endl = '\n';
	std::ios::sync_with_stdio(false);
	cin.tie(nullptr);
	cin >> n >> m;
	for(int i = 1; i <= n; i++)
		cin >> b[i];
	for(int i = 1; i <= n; i++)
	{
		a[i] = 1;
		if(b[i] == -1)
		{
			fa[i] = 0;
			continue;
		}
		for(int j = b[i] + 1; j < i; j++)
		{
			if(fa[j] <= b[i])
			{
				fa[j] = i;
				a[j] = 0;
			}
		}
		fa[i] = b[i];
	}
	for(int i = 1; i <= n; i++)
		add(fa[i], i);
	dfs(0);
	cout << dp[0][m + 1] << endl;
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>