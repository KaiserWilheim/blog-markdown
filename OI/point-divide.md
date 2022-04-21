---
title: 点分治
zh-CN: true
date: 2022-03-05
tags:
	- 算法
categories:
	算法
mathjax: true
---

点分治以及点分树。

<!--more-->

# 点分治

点分治，顾名思义，是一种分治类算法。

点分治的核心思想是，将一棵树不断地通过删点来分为更小的子树，不断递归下去直至无法继续分治。

## 时间复杂度

同时具有类似思想的还有边分治，即通过截断边的方式来将树分为两棵更小的树，但是一个菊花图就可以将其卡到T得飞起。
所以说边分治无法有效地保证其时间复杂度。

但是点分治是如何保证时间复杂度的呢？

### 树的重心

点分治通过寻找树的重心并删除之来保证其时间复杂度为 $O(n \log n)$。

为什么呢？

有这样一条引理说得好：树的重心的子树大小最多不超过 $\frac{n}{2}$ ，同时 $n$ 指当前树的大小。

所以我们就可以保证，每一次最少可以将最大的子树大小削减为之前的一半，也就保证了最多分治 $\log n$ 次。

## 方法

点分治与其他分治的思路相近，大致如下：

1. 将所要分治的区间分为多段，递归处理子树内问题。
2. 尝试处理横跨子树的问题。

举个栗子：

以[AcWing #252 树](https://www.acwing.com/problem/content/254/),[Luogu P4178 Tree](https://www.luogu.com.cn/problem/P4178)为例。

这道题让我们求出树上长度不超过 $k$ 的路径有多少条。

我们的思路是这个样子的：

我们把所有路径分为两类，一类是两端点均在同一子树内的，一类是两端点横跨两个子树的。
对于第一种，我们可以通过递归处理来解决。
而对于第二种，我们可以发现，这样的路径一定经过我们进行分治的点，也就是当前树的重心。对此，我们考虑枚举当前分治重心的子树内，与其距离为 $i$ 的点的个数，最后再依据 $\sum_{i=1}^{\lfloor \frac{k}{2} \rfloor} num[i] \times num[k-i]$ 统计出所有长度为 $k$ 的路径的个数。

参考代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 40010, M = N * 2;
int n, m;
int h[N], e[M], w[M], ne[M], idx;
bool st[N];
int p[N], q[N];

void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

int get_size(int u, int fa)//求子树大小
{
	if(st[u]) return 0;
	int res = 1;
	for(int i = h[u]; ~i; i = ne[i])
		if(e[i] != fa)
			res += get_size(e[i], u);
	return res;
}

int get_wc(int u, int fa, int tot, int &wc)//求重心
{
	if(st[u]) return 0;
	int sum = 1, ms = 0;
	for(int i = h[u]; ~i; i = ne[i])
	{
		int j = e[i];
		if(j == fa) continue;
		int t = get_wc(j, u, tot, wc);
		ms = max(ms, t);
		sum += t;
	}
	ms = max(ms, tot - sum);
	if(ms <= tot / 2) wc = u;
	return sum;
}

void get_dist(int u, int fa, int dist, int &qt)
{
	if(st[u]) return;
	q[qt++] = dist;
	for(int i = h[u]; ~i; i = ne[i])
		if(e[i] != fa)
			get_dist(e[i], u, dist + w[i], qt);
}

int get(int a[], int k)
{
	sort(a, a + k);
	int res = 0;
	for(int i = k - 1, j = -1; i >= 0; i--)
	{
		while(j + 1 < i && a[j + 1] + a[i] <= m) j++;
		j = min(j, i - 1);
		res += j + 1;
	}
	return res;
}

int calc(int u)
{
	if(st[u]) return 0;
	int res = 0;
	get_wc(u, -1, get_size(u, -1), u);
	st[u] = true;

	int pt = 0;
	for(int i = h[u]; ~i; i = ne[i])
	{
		int j = e[i], qt = 0;
		get_dist(j, -1, w[i], qt);
		res -= get(q, qt);
		for(int k = 0; k < qt; k++)
		{
			if(q[k] <= m) res++;
			p[pt++] = q[k];
		}
	}
	res += get(p, pt);

	for(int i = h[u]; ~i; i = ne[i]) res += calc(e[i]);
	return res;
}

int main()
{
	scanf("%d", &n);
	memset(st, 0, sizeof st);
	memset(h, -1, sizeof h);
	idx = 0;
	for(int i = 1; i <= n - 1; i++)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		add(a - 1, b - 1, c), add(b - 1, a - 1, c);
	}
	scanf("%d", &m);
	printf("%d\n", calc(0));
	return 0;
}
```

# 边分治优化

之前我们提到过边分治无法应对菊花图。

但是，我们可以考虑将树的结构进行优化。
我们向图中添加多个虚点，这些点没有实际意义，根据题目的要求赋上对结果无影响的值。
这些点可以将一个点与其过多的儿子分开，达到二叉树的级别，确保了边分治能够保持 $O(n \log n)$ 的复杂度。


# 点分树

点分治能解决很多问题，但是对于带修还多次询问的这种问题却不是特别擅长。
如果我们保证数据正确性，每一次询问都重新计算一遍，那么我们的时间复杂度将会变为 $O(n^2 \log n)$ 级别的。

但是有一种方法可以解决这样的问题，那就是点分树。

点分树就是将点分治的过程来建成一棵树，适用于带修还多次询问，并且对树的结构没有要求的问题。

我们考虑将每一次分治的问题分为两种，一种是与选定节点在同一子树内的，一种是不在同一子树内的。
不在同一子树内的节点，我们考虑使用一种 $\log$ 级别的算法来求出来；
在同一个子树内的节点，我们考虑继续递归。

最后直到选定节点为此时树的重心。此时，我们也有一个 $\log$ 级别的算法来求解剩下的问题。

因此，动态点分治的时间复杂度是 $O(n \log^2 n)$ 级别的。

这里我们以[AcWing #2261 开店](https://www.acwing.com/problem/content/2228/),[Luogu P3241 HNOI 开店](https://www.luogu.com.cn/problem/P3241)为例。

参考代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 150010, M = N << 1;
int n, m, A;
int h[N], e[M], w[M], ne[M], idx;
int age[N];
bool st[N];
struct Father
{
	int u, num;
	ll dis;
};
vector<Father>f[N];
struct Son
{
	int age;
	ll dis;
	bool operator < (const Son &a) const
	{
		return age < a.age;
	}
};
vector<Son>son[N][3];

void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

int get_sz(int u, int fa)//求子树大小
{
	if(st[u])return 0;
	int res = 1;
	for(int i = h[u]; ~i; i = ne[i])
		if(e[i] != fa)
			res += get_sz(e[i], u);
	return res;
}

int get_zx(int u, int fa, int tot, int &zx)//求重心
{
	if(st[u])return 0;
	int sum = 1, ms = 0;
	for(int i = h[u]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		int t = get_zx(e[i], u, tot, zx);
		ms = max(ms, t);
		sum += t;
	}
	ms = max(ms, tot - sum);
	if(ms <= tot / 2)zx = u;
	return sum;
}

void get_dis(int u, int fa, ll dis, int zx, int k, vector<Son> &p)
{
	if(st[u])return;
	f[u].push_back({ zx,k,dis });
	p.push_back({ age[u],dis });
	for(int i = h[u]; ~i; i = ne[i])
	{
		if(e[i] == fa)continue;
		get_dis(e[i], u, dis + w[i], zx, k, p);
	}
}

void calc(int u)
{
	if(st[u])return;
	get_zx(u, -1, get_sz(u, -1), u);
	st[u] = true;
	for(int i = h[u], k = 0; ~i; i = ne[i])
	{
		if(st[e[i]])continue;
		auto &p = son[u][k];
		p.push_back({ -1,0 });
		p.push_back({ A + 1,0 });
		get_dis(e[i], -1, w[i], u, k, p);
		k++;
		sort(p.begin(), p.end());
		for(int i = 1; i < p.size(); i++)
			p[i].dis += p[i - 1].dis;
	}
	for(int i = h[u]; ~i; i = ne[i])calc(e[i]);
}

ll query(int u, int l, int r)
{
	ll res = 0;
	for(auto &t : f[u])
	{
		int g = age[t.u];
		if((g >= l) && (g <= r))res += t.dis;
		for(int i = 0; i < 3; i++)
		{
			if(i == t.num)continue;
			auto &p = son[t.u][i];
			if(p.empty())continue;
			int a = lower_bound(p.begin(), p.end(), Son({ l,-1 })) - p.begin();
			int b = lower_bound(p.begin(), p.end(), Son({ r + 1,-1 })) - p.begin();
			res += t.dis * (b - a) + p[b - 1].dis - p[a - 1].dis;
		}
	}
	for(int i = 0; i < 3; i++)
	{
		auto &p = son[u][i];
		if(p.empty())continue;
		int a = lower_bound(p.begin(), p.end(), Son({ l,-1 })) - p.begin();
		int b = lower_bound(p.begin(), p.end(), Son({ r + 1,-1 })) - p.begin();
		res += p[b - 1].dis - p[a - 1].dis;
	}
	return res;
}

int main()
{
	scanf("%d%d%d", &n, &m, &A);
	for(int i = 1; i <= n; i++)scanf("%d", &age[i]);
	memset(h, -1, sizeof(h));
	for(int i = 0; i < n - 1; i++)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		add(a, b, c);
		add(b, a, c);
	}
	calc(1);
	ll res = 0;
	while(m--)
	{
		int u, a, b;
		scanf("%d%d%d", &u, &a, &b);
		int l = (a + res) % A, r = (b + res) % A;
		if(l > r)swap(l, r);
		res = query(u, l, r);
		printf("%lld\n", res);
	}
	return 0;
}
```


















































































