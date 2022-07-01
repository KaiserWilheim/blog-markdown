---
title: P1505 [国家集训队] 旅游 题解
zh-CN: true
date: 2022-03-24
tags:
	- 题解
	- 树链剖分
categories:
	题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">旅游</div>
<div id="problem-info-from">国家集训队</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P1505">Luogu P1505</a></li></ul></div>

----

题目给定一棵 n 个节点的树，边带权，编号 $0 \sim n-1$，需要我们支持五种操作：

+ `C i w` 将输入的第 $i$ 条边权值改为 $w$
+ `N u v` 将 $u,v$ 节点之间的边权都变为相反数
+ `SUM u v` 询问 $u,v$ 节点之间边权和
+ `MAX u v` 询问 $u,v$ 节点之间边权最大值
+ `MIN u v` 询问 $u,v$ 节点之间边权最小值

我们可以使用树剖来解决这个问题。

但是树剖作用的是点权，而我们可以使用边权转点权的方法来解决。

我们将边权转入其下方的节点里面，在访问的时候忽略掉最顶端的节点即可。

还有区间取反。

我们考虑使用一个懒标记来异或维护一下，当我们需要访问的时候就让min和max取反并互换即可。

上代码：

``` cpp
#include<bits/stdc++.h> 
#define int long long
using namespace std;
const int N = 2e5 + 10;
int n, m;
int cnt, h[N], e[N << 1], ne[N << 1], val[N << 1];
int idx, fa[N], son[N], top[N], dep[N], dfn[N], sz[N], tmp[N], w[N];
struct SegTree
{
	int l, r;
	int sum, max, min, tag;
}tr[N << 3];
struct node
{
	int x, y;
}id[N];

inline void add(int x, int y, int w)
{
	ne[++cnt] = h[x]; e[cnt] = y; val[cnt] = w; h[x] = cnt;
}

inline void dfs1(int x, int f)
{
	dep[x] = dep[f] + 1; fa[x] = f; sz[x] = 1;
	for(int i = h[x]; i; i = ne[i])
	{
		int v = e[i];
		if(v == f) continue;
		dfs1(v, x);
		tmp[v] = val[i];
		sz[x] += sz[v];
		if(sz[son[x]] < sz[v]) son[x] = v;
	}
}

inline void dfs2(int x, int topf)
{
	dfn[x] = ++idx; w[idx] = tmp[x]; top[x] = topf;
	if(son[x]) dfs2(son[x], topf);
	for(int i = h[x]; i; i = ne[i])
	{
		int v = e[i];
		if(v == fa[x] || v == son[x]) continue;
		dfs2(v, v);
	}
}

inline void pushup(int p)
{
	tr[p].sum = tr[p << 1].sum + tr[p << 1 | 1].sum;
	tr[p].max = max(tr[p << 1].max, tr[p << 1 | 1].max);
	tr[p].min = min(tr[p << 1].min, tr[p << 1 | 1].min);
	return;
}

inline void pushdown(int p)
{
	auto &root = tr[p], &left = tr[p << 1], &rght = tr[p << 1 | 1];
	left.tag ^= 1; rght.tag ^= 1;
	left.sum = -left.sum; rght.sum = -rght.sum;
	left.max = -left.max; rght.max = -rght.max;
	left.min = -left.min; rght.min = -rght.min;
	swap(left.max, left.min);
	swap(rght.max, rght.min);
	root.tag = 0;
	return;
}

inline void build(int p, int l, int r)
{
	tr[p].l = l, tr[p].r = r;
	if(l == r)
	{
		tr[p].sum = tr[p].max = tr[p].min = w[l];
		return;
	}
	int mid = (l + r) >> 1;
	build(p << 1, l, mid);
	build(p << 1 | 1, mid + 1, r);
	pushup(p);
}

inline void change(int p, int q, int k)
{
	if((tr[p].l == q) && (tr[p].r == q))
	{
		tr[p].sum = tr[p].max = tr[p].min = k;
		return;
	}
	if(tr[p].tag) pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(q <= mid) change(p << 1, q, k);
	if(q > mid)  change(p << 1 | 1, q, k);
	pushup(p);
}

inline void segrev(int p, int l, int r)
{
	if((l <= tr[p].l) && (r >= tr[p].r))
	{
		tr[p].tag ^= 1;
		tr[p].sum = -tr[p].sum;
		tr[p].max = -tr[p].max;
		tr[p].min = -tr[p].min;
		swap(tr[p].max, tr[p].min);
		return;
	}
	if(tr[p].tag) pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid) segrev(p << 1, l, r);
	if(r > mid)  segrev(p << 1 | 1, l, r);
	pushup(p);
}

inline int segsum(int p, int l, int r)
{
	if((l <= tr[p].l) && (r >= tr[p].r))return tr[p].sum;
	int res = 0;
	if(tr[p].tag)pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)res += segsum(p << 1, l, r);
	if(r > mid)res += segsum(p << 1 | 1, l, r);
	return res;
}

inline int segmax(int p, int l, int r)
{
	if((l <= tr[p].l) && (r >= tr[p].r))return tr[p].max;
	int res = -2147483647;
	if(tr[p].tag)pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)res = max(res, segmax(p << 1, l, r));
	if(r > mid)res = max(res, segmax(p << 1 | 1, l, r));
	return res;
}

inline int segmin(int p, int l, int r)
{
	if((l <= tr[p].l) && (r >= tr[p].r))return tr[p].min;
	int res = 2147483647;
	if(tr[p].tag)pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)res = min(res, segmin(p << 1, l, r));
	if(r > mid)res = min(res, segmin(p << 1 | 1, l, r));
	return res;
}

inline void revpath(int x, int y)
{
	while(top[x] != top[y])
	{
		if(dep[top[x]] < dep[top[y]]) swap(x, y);
		segrev(1, dfn[top[x]], dfn[x]);
		x = fa[top[x]];
	}
	if(dep[x] > dep[y]) swap(x, y);
	if(x != y) segrev(1, dfn[x] + 1, dfn[y]);
}

inline int sumpath(int x, int y)
{
	int res = 0;
	while(top[x] != top[y])
	{
		if(dep[top[x]] < dep[top[y]]) swap(x, y);
		res += segsum(1, dfn[top[x]], dfn[x]);
		x = fa[top[x]];
	}
	if(dep[x] > dep[y]) swap(x, y);
	if(x != y) res += segsum(1, dfn[x] + 1, dfn[y]);
	return res;
}

inline int maxpath(int x, int y)
{
	int res = -2147483647;
	while(top[x] != top[y])
	{
		if(dep[top[x]] < dep[top[y]]) swap(x, y);
		res = max(res, segmax(1, dfn[top[x]], dfn[x]));
		x = fa[top[x]];
	}
	if(dep[x] > dep[y]) swap(x, y);
	if(x != y) res = max(res, segmax(1, dfn[x] + 1, dfn[y]));
	return res;
}

inline int minpath(int x, int y)
{
	int res = 2147483647;
	while(top[x] != top[y])
	{
		if(dep[top[x]] < dep[top[y]]) swap(x, y);
		res = min(res, segmin(1, dfn[top[x]], dfn[x]));
		x = fa[top[x]];
	}
	if(dep[x] > dep[y]) swap(x, y);
	if(x != y) res = min(res, segmin(1, dfn[x] + 1, dfn[y]));
	return res;
}

signed main()
{
	scanf("%lld", &n);
	for(int i = 1; i < n; i++)
	{
		int a, b, c;
		scanf("%lld%lld%lld", &a, &b, &c);
		add(++a, ++b, c);
		add(b, a, c);
		id[i].x = a;
		id[i].y = b;
	}
	dfs1(1, 0);
	dfs2(1, 1);
	build(1, 1, n);
	scanf("%lld", &m);
	while(m--)
	{
		char s[10];
		int a, b;
		scanf("%s%lld%lld", s, &a, &b);
		if(s[0] == 'C')
		{
			int target;
			if(dep[id[a].x] > dep[id[a].y]) target = id[a].x;
			else target = id[a].y;
			change(1, dfn[target], b);
		}
		else if(s[0] == 'N')
		{
			revpath(++a, ++b);
		}
		else if(s[0] == 'S')
		{
			printf("%lld\n", sumpath(++a, ++b));
		}
		else if(s[0] == 'M' && s[1] == 'A')
		{
			printf("%lld\n", maxpath(++a, ++b));
		}
		else if(s[0] == 'M' && s[1] == 'I')
		{
			printf("%lld\n", minpath(++a, ++b));
		}
	}
	return 0;
}
```

