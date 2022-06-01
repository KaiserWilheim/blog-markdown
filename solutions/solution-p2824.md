---
title: P2824 [HEOI2016/TJOI2016] 排序 题解
date: 2022-05-27
tags:
	- 题解
	- 线段树
	- 二分
categories:
	- 题解
mathjax: true
---

Luogu P2824
LibreOJ #2055

HEOI 2016
TJOI 2016

<!-- more -->
----

一看见排序我们就感觉开始复杂起来了，毕竟排序是一个复杂而又缓慢的过程。

我们来想简单一点的排序。

如果我们对一个排列进行升序或者降序排序应该很简单，就是将这个排列所包含的数进行正向或者反向输出即可。

假如我们每一次排列的时候都是对一个排列排序该多好啊。

可惜这样无法满足我们接下来的询问，毕竟这样对其没有任何影响，该得不出来还是得不出来。

那么如果我们对的是一个01序列排序呢？

也很简单，就把1全部放在一边，0放在另一边即可。

那这样有什么可以利用的性质呢？

我们可以设想，当我们把大于一个值的数字全部变成1，其余的变成0，再排序，询问的时候得到的就是当前位置比这个值大还是小。

然后我们发现，我们可以利用这个东西进行二分。

我们每一次指定一个值，按照上面的步骤得到询问的位置是0还是1，然后根据答案二分即可。

但是我们每一次还是需要进行枚举，时间仍然没有达到我们的要求。

既然是区间修改了，那么我们就可以使用——线段树！

每一次询问当前区间内有多少个1，然后根据上面的想法区间覆盖即可，而询问区间内1的个数可以使用维护区间和的方式来得出。

然后就显而易见了。

``` cpp
#define _CRT_SECURE_NO_WARNINGS
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;
int n, m;
int q;
int a[N];
struct SegTree
{
	int l, r;
	int sum, tag;
}tr[N << 3];
void pushup(int p)
{
	tr[p].sum = tr[p << 1].sum + tr[p << 1 | 1].sum;
}
void pushdown(int p)
{
	auto &root = tr[p], &left = tr[p << 1], &rght = tr[p << 1 | 1];
	if(root.tag != -1)
	{
		left.sum = (left.r - left.l + 1) * root.tag;
		rght.sum = (rght.r - rght.l + 1) * root.tag;
		left.tag = root.tag;
		rght.tag = root.tag;
		root.tag = -1;
	}
}
void build(int p, int l, int r)
{
	tr[p].l = l, tr[p].r = r;
	if(l == r)
	{
		return;
	}
	int mid = (l + r) >> 1;
	build(p << 1, l, mid);
	build(p << 1 | 1, mid + 1, r);
}
void init(int p, int k)
{
	tr[p].tag = -1;
	if(tr[p].l == tr[p].r)
	{
		tr[p].sum = (a[tr[p].l] >= k);
		tr[p].tag = -1;
		return;
	}
	init(p << 1, k);
	init(p << 1 | 1, k);
	pushup(p);
}
void segchg(int p, int l, int r, int k)
{
	if(tr[p].l >= l && tr[p].r <= r)
	{
		tr[p].sum = (tr[p].r - tr[p].l + 1) * k;
		tr[p].tag = k;
		return;
	}
	pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)segchg(p << 1, l, r, k);
	if(r > mid)segchg(p << 1 | 1, l, r, k);
	pushup(p);
}
int segsum(int p, int l, int r)
{
	if(tr[p].l >= l && tr[p].r <= r)return tr[p].sum;
	pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	int res = 0;
	if(l <= mid)res += segsum(p << 1, l, r);
	if(r > mid)res += segsum(p << 1 | 1, l, r);
	return res;
}
int op[N], L[N], R[N];
int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
		scanf("%d", &a[i]);
	for(int i = 1; i <= m; i++)
		scanf("%d%d%d", &op[i], &L[i], &R[i]);
	scanf("%d", &q);
	build(1, 1, n);
	int ans = -1;
	int l = 1, r = n;
	while(l <= r)
	{
		int mid = (l + r) >> 1;
		init(1, mid);
		for(int i = 1; i <= m; i++)
		{
			int cnt = segsum(1, L[i], R[i]);
			if(op[i] == 0)
			{
				segchg(1, L[i], R[i] - cnt, 0);
				segchg(1, R[i] - cnt + 1, R[i], 1);
			}
			else if(op[i] == 1)
			{
				segchg(1, L[i], L[i] + cnt - 1, 1);
				segchg(1, L[i] + cnt, R[i], 0);
			}
		}
		int pos = segsum(1, q, q);
		if(pos == 1)
		{
			l = mid + 1;
			ans = mid;
		}
		else
		{
			r = mid - 1;
		}
	}
	printf("%d\n", ans);
	return 0;
}
```

