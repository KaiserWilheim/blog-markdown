---
title: P4551 最长异或路径 题解
date: 2022-04-24
tags:
	- 题解
	- 字典树
categories:
	- 题解
mathjax: true
---

Luogu P4551

<!-- more -->
----

做这道题需要一个有关异或的性质，就是：

假设我们有两个数 $x$ 和 $y$，不论 $x$ 和 $y$ 是多少，我们都有 $x \oplus y \oplus y = x$，即一个数异或另一个数两次仍为自身。

那我们需要找的最长异或路径，可以转化为两个点到根的路径的异或值的最大值。

因为对于两个点 $x$ 和 $y$，我们需要求的就是 $\operatorname{dis}(x,\operatorname{lca}(x,y)) \oplus \operatorname{dis}(y,\operatorname{lca}(x,y))$。我们可以将 $\operatorname{dis}(1,x) \oplus \operatorname{dis}(1,y)$ 分解为 $\operatorname{dis}(1,\operatorname{lca}(x,y)) \oplus \operatorname{dis}(\operatorname{lca}(x,y),x) \oplus \operatorname{dis}(1,\operatorname{lca}(x,y)) \oplus \operatorname{dis}(1,\operatorname{lca}(x,y))$，然后利用性质转化为 $\operatorname{dis}(x,\operatorname{lca}(x,y)) \oplus \operatorname{dis}(y,\operatorname{lca}(x,y))$。

而所有的 $\operatorname{dis}(1,x)$ 只需要使用一次DFS即可求出。

然后就是找异或最大值。

我们将所有的 $\operatorname{dis}(1,x)$ 放进一棵Trie树里面，然后就可以寻找异或最大值了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010, M = N << 1;
int n;
struct TrieTree
{
	int s[2];
	int occ;
}tr[N * 32];
int tot = 1;
int w[M], h[N], e[M], ne[M], idx;
int dis[N];
int ans;
void add(int a, int b, int v)
{
	e[idx] = b;
	w[idx] = v;
	ne[idx] = h[a];
	h[a] = idx++;
	return;
}
void insert(int x)
{
	for(int i = 30, p = 1; i >= 0; i--)
	{
		int c = ((x >> i) & 1);
		if(!tr[p].s[c])tr[p].s[c] = ++tot;
		p = tr[p].s[c];
	}
	return;
}
void get(int x)
{
	int res = 0;
	for(int i = 30, p = 1; i >= 0; i--)
	{
		int c = ((x >> i) & 1);
		if(tr[p].s[c ^ 1])
		{
			p = tr[p].s[c ^ 1];
			res |= (1 << i);
		}
		else
		{
			p = tr[p].s[c];
		}
	}
	ans = max(ans, res);
	return;
}
void dfs(int p, int fa)
{
	insert(dis[p]);
	get(dis[p]);
	for(int i = h[p]; ~i; i = ne[i])
	{
		int v = e[i];
		if(v == fa)continue;
		dis[v] = dis[p] ^ w[i];
		dfs(v, p);
	}
	return;
}
int main()
{
	cin >> n;
	memset(h, -1, sizeof(h));
	for(int i = 1; i < n; i++)
	{
		int a, b, v;
		scanf("%d%d%d", &a, &b, &v);
		add(a, b, v);
		add(b, a, v);
	}
	dfs(1, 0);
	cout << ans << endl;
	return 0;
}
```
