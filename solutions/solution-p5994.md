---
title: P5994 [PA2014] Kuglarz 题解
date: 2022-08-07
tags:
	- 题解
	- 最小生成树
	- 建图
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Kuglarz</div>
<div id="problem-info-from">PA 2014</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P5994">Luogu P5994</a></li><li><a href="https://darkbzoj.cc/problem/3714">BZOJ #3714</a></li></ul></div>

----

首先我们可以想到一个性质，就是我们在知道了 $[l,r_1]$ 和 $[l,r_2]$ 的奇偶性了之后，我们就知道了 $[r_1+1,r_2]$ 的奇偶性。

这与我们连边很像，所以我们可以尝试将其转化为“连接了 $(l,r_1)$ 和 $(l,r_2)$ 之后，$(r_1+1,r_2)$ 就联通了”的这种形式，这样我们就可以对其使用一些图论有关的算法来求解了。

但是这个+1就很不方便，不符合我们正常连边时候的性质，我们需要想办法将其去掉。
我们可以将询问 $[l,r]$ 的奇偶性转化为连接 $(l-1,r)$，这样我们就符合了连通性的规则了。

现在我们需要知道每一个点的奇偶性，就相当于是让图中的所有点都联通。同时还需要让我们的总花费最小，这就相当于是让图中的边权和最小。

于是我们就直接跑最小生成树即可。

代码如下：

``` cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 2010, M = 4000010;
#define int long long
int p[N];
int find(int x)
{
	if(p[x] != x)p[x] = find(p[x]);
	return p[x];
}
struct edge
{
	int a, b, w;
	bool operator < (const edge &a) const
	{
		return w < a.w;
	}
}e[M];
signed main()
{
	int n, m = 0;
	scanf("%lld", &n);
	for(int i = 1; i <= n; i++)
	{
		for(int j = i; j <= n; j++)
		{
			int x;
			scanf("%lld", &x);
			e[m++] = { i - 1,j,x };
		}
	}
	for(int i = 1; i <= n; i++)p[i] = i;
	sort(e, e + m);
	int ans = 0;
	for(int i = 0; i < m; i++)
	{
		int a = e[i].a, b = e[i].b, w = e[i].w;
		int pa = find(a), pb = find(b);
		if(pa != pb)
		{
			p[pa] = pb;
			ans += w;
		}
	}
	printf("%lld\n", ans);
	return 0;
}
```

