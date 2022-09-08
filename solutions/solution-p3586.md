---
title: P3586 [POI2015] Logistyka 题解
date: 2022-04-11
tags:
	- 题解
	- 线段树
categories:
	题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Logistyka</div>
<div id="problem-info-from">POI 2015</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3586">Luogu P3586</a></li><li><a href="https://darkbzoj.cc/problem/4378">BZOJ #4378</a></li></ul></div>

----

# 思路

这道题想让我们维护一个数列，支持对其的修改与询问。

题目给出了两种操作：

1. `U k a` 将序列中第 $k$ 个数修改为 $a$。
2. `Z c s` 在这个序列上，每次选出 $c$ 个正数，并将它们都减去 $1$，询问能否进行 $s$ 次操作。

回答询问的思路是这个样子的：

我们假设在这个数列里面，$\( 0,s \)$ 范围内的数有 $x$ 个，$\[ s, \infty \)$ 范围内的数有 $y$ 个。

如果 $x+y < c$ 的话就绝对不行，直接返回 `NIE`。

我们贪一下心，反正题目询问的是可行性，不如就先让着 $y$ 个数先顶上。

如果这 $y$ 个数都顶上去之后就可以完成任务，甚至还有些富余，就可以直接返回 `TAK`。
而如果 $y < c$，那么我们就需要继续往下讨论。

我们考虑让剩下的数顶上去。

如果这些数字的和小于 $s \times c$，那就绝对完不成任务。
反之则一定完得成任务。

QED.

----

所以，我们需要维护两个信息，一是大于某个数的数有多少个，二是大于某个数的所有数之和。

我们可以使用树状数组。

但是我不会，所以就用动态开点权值线段树了。

我们考虑结构体里面存什么：

首先我们需要存区间左右端点（按个人情况）和左右儿子。
我们还需要维护区间内数的个数。

为了不多写数据结构，我也将第二个要求写进了线段树里面。

于是我的结构体长的是这个样子：

``` cpp
struct SegTree
{
	int l, r;
	int ls, rs;
	ll sum;
	ll tot;
}
```

其中 `sum` 维护的是当前区间内数的个数，`tot` 维护的是当前区间内所有数的和。两者同时维护，但是查询的时候是分开的。

线段树部分代码：

``` cpp
struct SegTree
{
	int l, r;
	int ls, rs;
	ll sum;
	ll tot;
}tr[N << 3];
int idx;
void segadd(int p, int pos, ll k)
{
	if(tr[p].l == tr[p].r)
	{
		tr[p].sum += k;
		tr[p].tot = tr[p].sum * tr[p].l;
		return;
	}
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(pos <= mid)
	{
		if(!tr[p].ls)
		{
			tr[p].ls = ++idx;
			tr[tr[p].ls].l = tr[p].l;
			tr[tr[p].ls].r = mid;
		}
		segadd(tr[p].ls, pos, k);
	}
	else if(pos > mid)
	{
		if(!tr[p].rs)
		{
			tr[p].rs = ++idx;
			tr[tr[p].rs].l = mid + 1;
			tr[tr[p].rs].r = tr[p].r;
		}
		segadd(tr[p].rs, pos, k);
	}
	tr[p].sum = tr[tr[p].ls].sum + tr[tr[p].rs].sum;
	tr[p].tot = tr[tr[p].ls].tot + tr[tr[p].rs].tot;
	return;
}

ll segsum(int p, int l, int r)
{//求区间内数的个数
	if(!p)return 0;
	if(tr[p].l >= l && tr[p].r <= r)return tr[p].sum;
	int mid = (tr[p].l + tr[p].r) >> 1;
	ll res = 0;
	if(l <= mid)res += segsum(tr[p].ls, l, r);
	if(r > mid)res += segsum(tr[p].rs, l, r);
	return res;
}
ll segtot(int p, int l, int r)
{//求区间内所有数的和
	if(!p)return 0;
	if(tr[p].l >= l && tr[p].r <= r)return tr[p].tot;
	int mid = (tr[p].l + tr[p].r) >> 1;
	ll res = 0;
	if(l <= mid)res += segtot(tr[p].ls, l, r);
	if(r > mid)res += segtot(tr[p].rs, l, r);
	return res;
}
```

# 代码

（需要O2）

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010;
int n, m;
ll a[N];
struct SegTree
{
	int l, r;
	int ls, rs;
	ll sum;
	ll tot;
}tr[N << 3];
int idx;
void segadd(int p, int pos, ll k)
{
	if(tr[p].l == tr[p].r)
	{
		tr[p].sum += k;
		tr[p].tot = tr[p].sum * tr[p].l;
		return;
	}
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(pos <= mid)
	{
		if(!tr[p].ls)
		{
			tr[p].ls = ++idx;
			tr[tr[p].ls].l = tr[p].l;
			tr[tr[p].ls].r = mid;
		}
		segadd(tr[p].ls, pos, k);
	}
	else if(pos > mid)
	{
		if(!tr[p].rs)
		{
			tr[p].rs = ++idx;
			tr[tr[p].rs].l = mid + 1;
			tr[tr[p].rs].r = tr[p].r;
		}
		segadd(tr[p].rs, pos, k);
	}
	tr[p].sum = tr[tr[p].ls].sum + tr[tr[p].rs].sum;
	tr[p].tot = tr[tr[p].ls].tot + tr[tr[p].rs].tot;
	return;
}

ll segsum(int p, int l, int r)
{
	if(!p)return 0;
	if(tr[p].l >= l && tr[p].r <= r)return tr[p].sum;
	int mid = (tr[p].l + tr[p].r) >> 1;
	ll res = 0;
	if(l <= mid)res += segsum(tr[p].ls, l, r);
	if(r > mid)res += segsum(tr[p].rs, l, r);
	return res;
}
ll segtot(int p, int l, int r)
{
	if(!p)return 0;
	if(tr[p].l >= l && tr[p].r <= r)return tr[p].tot;
	int mid = (tr[p].l + tr[p].r) >> 1;
	ll res = 0;
	if(l <= mid)res += segtot(tr[p].ls, l, r);
	if(r > mid)res += segtot(tr[p].rs, l, r);
	return res;
}
int main()
{
	scanf("%d%d", &n, &m);
	int maxn = 0;
	tr[++idx] = { 0,100000001,0,0,0,0 };
	while(m--)
	{
		string op;
		int x, k;
		cin >> op;
		scanf("%d%d", &x, &k);
		if(op[0] == 'U')
		{
			if(k > 0)
			{
				segadd(1, k, 1);
			}
			if(a[x] > 0)
			{
				segadd(1, a[x], -1);
			}
			a[x] = k;
			maxn = max(maxn, k);
		}
		else if(op[0] == 'Z')
		{
			ll pos = segsum(1, 1, maxn), cnt = segsum(1, 1, k);
			if(pos < x)
			{
				puts("NIE");
				continue;
			}
			if(pos - cnt >= x)
			{
				puts("TAK");
				continue;
			}
			ll tot = segtot(1, 1, k);
			if(tot >= (x + cnt - pos) * k)
				puts("TAK");
			else
				puts("NIE");
		}
	}
	return 0;
}
```
