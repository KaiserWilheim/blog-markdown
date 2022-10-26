---
title: P4336 [SHOI2016] 黑暗前的幻想乡 题解
date: 2022-08-23
tags:
	- 题解
	- 矩阵树定理
	- 容斥
	- 数学（题目）
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">黑暗前的幻想乡</div>
<div id="problem-info-from">SHOI 2016</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4336">Luogu P4336</a></li><li><a href="https://loj.ac/p/2027">LibreOJ L2027</a></li><li><a href="https://www.acwing.com/problem/content/2795/">AcWing 2793</a></li><li><a href="https://darkbzoj.cc/problem/4596">BZOJ #4596</a></li></ul></div>

----

# 前置芝士

## Kirchhoff 矩阵树定理

Kirchhoff矩阵树定理解决了一个问题：对于一个确定的无向图，其究竟有多少个生成树？

对于一个无向图，我们拥有其邻接矩阵 $\bf{A}$。
这里的邻接矩阵允许重边，第 $i$ 行第 $j$ 列的值代表着点 $i$ 到点 $j$ 有几条边。
不允许自环。

我们定义一个无向图的度数矩阵 $\bf{D}$ 为，第 $i$ 行第 $i$ 列上的数字是点 $i$ 的度数，其余的格子都为 $0$ 的矩阵。

我们定义一个图的 Kirchhoff矩阵 ${\bf{K}} = {\bf{D}} - {\bf{A}}$。

这个矩阵同时去掉任意一行一列，剩下的这个子矩阵的行列式的绝对值，就是该无向图的生成树个数。

## 行列式

行列式可以被理解为列向量夹的几何体的体积，用 $\det({\bf{A}})$ 表示 $\bf{A}$ 这个方阵的行列式。

这里不讲太细，细一点的可以去看我的另外一篇博客。

在这里，我们只需要知道几个简单的点：

0. 每一个方阵都有自己的行列式，公式是 $\displaystyle \det({\bf{A}}) = \sum_{\sigma \in S_n} \operatorname{sgn}(\sigma) \prod_{i=1}^n a_{i,\sigma(i)}$。
1. 三角矩阵的行列式是可以以很小的复杂度计算出来的。具体方法是其对角线之积。
2. 方阵的行列式有如下几个性质：
	1. 矩阵转置，行列式不变；
	2. 矩阵行（列）交换，行列式取反；
	3. 矩阵行（列）相加减，行列式不变；
	4. 矩阵行（列）所有元素同时乘以数 $k$，行列式也乘 $k$。
3. 通过上面的几个操作（其实就是高斯消元）可以将一个矩阵消为一个三角矩阵，而反过来则可以从三角矩阵变回原矩阵。

那么我们存储一个系数，在消元的时候维护这个系数，直到最后得到三角矩阵的时候再将系数乘在这个我们能够简单求出的行列式上，就可以得到原矩阵的行列式了。

# 题目解读

题目要求我们求出一个无向图的生成树数量，并且该生成树需要满足其中的每一条边都属于一个不同的集合。

那么我们考虑容斥，将“恰好”改为“至多”。

这样我们枚举由 $n-1$ 个集合构成的无向图的时候会记录上由 $n-2$ 个集合构成的无向图的结果，需要减去这部分的影响。
然后枚举由 $n-2$ 个集合构成的无向图，同时会记录上由 $n-3$ 个集合构成的无向图的结果。
这样一直容斥下去，直到最后枚举由 $1$ 个集合构成的无向图的结果的时候，容斥就可以停止了。

每一次枚举的时候，我们都需要跑矩阵树定理。每一次跑矩阵树定理都是 $O((n-1)^3)$ 的，我们需要跑 $2^{n-1}$ 次，所以最终的复杂度是 $O(2^{n-1}(n-1)^3)$ 的。

参考代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 20, M = N * N;
const ll mod = 1e9 + 7;
int n, maxn;
int m[N];
int u[N][M], v[N][M];
int sz[(1 << 18)];
ll krh[N][N];
ll qpow(ll a, ll x)
{
	ll res = 1;
	while(x)
	{
		if(x & 1)res = (res * a) % mod;
		a = (a * a) % mod;
		x >>= 1;
	}
	return res;
}
inline ll det()
{
	ll res = 1;
	int flag = 0;
	for(int i = 1; i <= n - 1; i++)
	{
		if(krh[i][i] == 0)
		{
			for(int j = i + 1; j <= n - 1; j++)
			{
				if(krh[j][i] == 0)continue;
				flag ^= 1;
				for(int k = 1; k <= n - 1; k++)
					swap(krh[i][k], krh[j][k]);
			}
		}
		for(int j = i; j <= n - 1; j++)
		{
			if(krh[j][i] == 0)continue;
			ll inv = qpow(krh[j][i], mod - 2);
			res = (res * krh[j][i]) % mod;
			for(int k = i; k <= n - 1; k++)
				krh[j][k] = (krh[j][k] * inv) % mod;
		}
		for(int j = i + 1; j <= n - 1; j++)
		{
			if(krh[j][i] == 0)continue;
			for(int k = i; k <= n - 1; k++)
				krh[j][k] = (krh[j][k] - krh[i][k] + mod) % mod;
		}
	}
	for(int i = 1; i <= n - 1; i++)
		res = (res * krh[i][i]);
	return (flag) ? (mod - res) % mod : res;
}
int main()
{
	scanf("%d", &n);
	maxn = (1 << (n - 1)) - 1;
	for(int i = 1; i < n; i++)
	{
		scanf("%d", &m[i]);
		for(int j = 1; j <= m[i]; j++)
			scanf("%d%d", &u[i][j], &v[i][j]);
	}
	for(int i = 1; i <= maxn; i++)
		sz[i] = sz[i >> 1] + (i & 1);
	ll res = 0;
	for(int i = 1; i <= maxn; i++)
	{
		memset(krh, 0, sizeof(krh));
		for(int j = 1, p = i; p; p >>= 1, j++)
		{
			if((p & 1) == 0)continue;
			for(int k = 1; k <= m[j]; k++)
			{
				int U = u[j][k], V = v[j][k];
				krh[U][U]++, krh[V][V]++;
				krh[U][V] = (krh[U][V] + mod - 1) % mod;
				krh[V][U] = (krh[V][U] + mod - 1) % mod;
			}
		}
		res = (res + mod + det() * ((n - sz[i]) % 2 ? 1 : -1)) % mod;
	}
	printf("%lld\n", res);
	return 0;
}
```

