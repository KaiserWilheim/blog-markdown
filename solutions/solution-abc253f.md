---
title: ABC253F Operations on a Matrix 题解
date: 2022-09-27
tags:
	- 题解
	- 线段树
	- 可持久化
	- 可持久化线段树
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Operations on a Matrix</div>
<div id="problem-info-from">ABC 253</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://atcoder.jp/contests/abc253/tasks/abc253_f">ABC 253 F</a></li><li><a href="https://sjzezoj.com/problem/1584">S2OJ #1584</a></li></ul></div>

----

题目要求我们维护一个矩阵，并支持对其在纵向的区间加、横向的单点修改和对于某一个元素当前值的查询。

直接维护矩阵可以让我们以最劣 $O(n^2q)$ 的时间复杂度黯然离场。

考虑这些修改对于当前格子的影响。
或者进一步说，我们需要求出来对于我们被询问的格子受到了哪些修改的影响，并将这些修改的影响叠加起来给出一个值。

因为横向是单点推平，我们在进行了一次推平之后，其之前的所有纵向上的区间加对其的影响就可以忽略了。
所以我们每次单点推平的时候记录一个时间戳，每一次询问到该点的时候把当前该点处区间加的影响减去时间戳对应的时间该点处区间加的影响，再加上推平的那个值，就是当前点的值了。

题外话：
场上以为横向的是区间推平，于是就写了一个线段树上去。
后面发现不是区间推平，于是把线段树的区间推平改成了单点推平。
然后发现不需要区间查询。
那我写个线段树有个啥子用处？
果断改成了数组。

然后就是可以随时拿出历史版本的线段树了，也就是可持久化线段树。
我们之间学的可持久化线段树是单点修改的，不需要懒标记，也没有pushdown什么的。十分简单。
这里我们需要支持区间修改，也就需要支持懒标记了。
但此时我们的标记不支持pushdown，因为很可能把后面的标记下放到过去的某个状态中。即使我们pushdown的时候为两个子节点新开两个节点记录状态，最终也会发现任何形式的pushdown都不是正确的。

正确的方式是标记永久化。

我实现的方式是，每一次修改的时候只累加懒标记，查询的时候因为是单点查询，在回溯的时候直接把一路上的懒标记加起来就可以了。

代码如下：

``` cpp 位于ChmTree结构体内
int segadd(int p, int l, int r, int k)
{
	int q = ++idx;
	tr[q] = tr[p];
	if(tr[p].l >= l && tr[p].r <= r)
	{
		tr[q].tag += k;
		return q;
	}
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)tr[q].ls = segadd(tr[p].ls, l, r, k);
	if(r > mid)tr[q].rs = segadd(tr[p].rs, l, r, k);
	return q;
}
int segsum(int p, int pos)
{
	if(tr[p].l == tr[p].r)return tr[p].tag;
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(pos <= mid)return segsum(tr[p].ls, pos) + tr[p].tag;
	if(pos > mid)return segsum(tr[p].rs, pos) + tr[p].tag;
}
```

然后就是主席树的其他基本操作了。

总体的代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
const int N = 200010;
int n, m;
struct ChmTree
{
	struct Node
	{
		int l, r;
		int ls, rs;
		int tag;
	};
	Node tr[N * 64];
	int root[N];
	int idx = 0;
	int build(int l, int r)
	{
		int p = ++idx;
		tr[p] = { l,r,0,0,0 };
		if(l == r)
		{
			return p;
		}
		int mid = (l + r) >> 1;
		tr[p].ls = build(l, mid);
		tr[p].rs = build(mid + 1, r);
		return p;
	}
	int segadd(int p, int l, int r, int k)
	{
		int q = ++idx;
		tr[q] = tr[p];
		if(tr[p].l >= l && tr[p].r <= r)
		{
			tr[q].tag += k;
			return q;
		}
		int mid = (tr[p].l + tr[p].r) >> 1;
		if(l <= mid)tr[q].ls = segadd(tr[p].ls, l, r, k);
		if(r > mid)tr[q].rs = segadd(tr[p].rs, l, r, k);
		return q;
	}
	int segsum(int p, int pos)
	{
		if(tr[p].l == tr[p].r)return tr[p].tag;
		int mid = (tr[p].l + tr[p].r) >> 1;
		if(pos <= mid)return segsum(tr[p].ls, pos) + tr[p].tag;
		if(pos > mid)return segsum(tr[p].rs, pos) + tr[p].tag;
	}
};
ChmTree col;
pair<int, int> row[N];
signed main()
{
	int q;
	scanf("%lld%lld%lld", &n, &m, &q);
	col.root[0] = col.build(1, m);
	for(int i = 1; i <= q; i++)
	{
		int op, l, r, k;
		scanf("%lld%lld%lld", &op, &l, &r);
		if(op == 1)
		{
			scanf("%lld", &k);
			col.root[i] = col.segadd(col.root[i - 1], l, r, k);
		}
		else if(op == 2)
		{
			row[l] = make_pair(r, i);
			col.root[i] = col.root[i - 1];
		}
		else
		{
			col.root[i] = col.root[i - 1];
			int sum = col.segsum(col.root[i], r);
			sum -= col.segsum(col.root[row[l].second], r);
			sum += row[l].first;
			printf("%lld\n", sum);
		}
	}
	return 0;
}
```

