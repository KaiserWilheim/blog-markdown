---
title: K-D 树
date: 2022-08-19
tags:
	- 数据结构
categories:
	- 数据结构
---

一种高效处理 $k$ 维空间中的信息的数据结构。

<!-- more -->
K-D Tree 全称是叫 K-Dimentional Tree，是一棵二叉树。
一般我们用到K-D Tree的时候，这个 $k$ 是等于2的。

# 原理

我们就以 $k$ 等于2为例：

假设我们当前有一个二维~~空间~~平面，上面有一堆点；

<img src="https://s2.loli.net/2022/06/22/XMeF9ofAaBg1PUQ.png" width="60%">

然后我们需要对这个平面上的12个点建立一棵K-D树。

## 建立

建立的过程是这个样子的：

首先我们随便拿一维开始，就拿x这一维了。
然后我们找到这个维度中的中位数，在这个点上画一条直线，将平面分成两半：

<img src="https://s2.loli.net/2022/06/22/mhDXF6GO3tsW8Z5.png" width="60%">

然后继续建立，只不过我们为了在每一维都能够有较高的访问效率，这次我们换一维：

<img src="https://s2.loli.net/2022/06/22/a3DnuNJEPzYRHBd.png" width="60%">

然后继续，直到某一个长方形之内没有点了。

<img src="https://s2.loli.net/2022/06/22/p3osUmeyxLtgOCN.png" width="60%">

然后最终我们建立出来的树长这个样子：

![kd7.png](https://s2.loli.net/2022/08/18/QwBcK8F5q1IJZS7.png)

## 删除

K-D树的删除使用惰性删除，利用类似[替罪羊树](/OI/scapegoat-tree)的思想，暴力重构即可。

当然，还可以是朝鲜树，因为也是暴力重构。

## 插入

假设我们现在要在这里插入一个点 $N=(7,3)$。

<img src="https://s2.loli.net/2022/06/22/a9pGS4hXQL75bqZ.png" width="60%">

我们这样来检索：

我们从根节点开始。

首先它在当前节点 $A=(6,3)$ 的右边，所以走到右子树；
然后它在当前节点 $B=(8,5)$ 的下边，所以走到右子树；
然后它在当前节点 $J=(8,1)$ 的左边，所以走到左子树；

然后我们就发现，当前节点是个空节点。

于是就将 $N$ 加到当前节点上即可。

我们的树就变成了这个样子：

![kd8.png](https://s2.loli.net/2022/08/18/Z59VySfXmIgGoLF.png)

不要忘记给点 $N$ 加一条分割线，即使这根本在建树的时候完全体现不出来：

<img src="https://s2.loli.net/2022/06/22/xl3X8ZHzrkDa9Kf.png" width="60%">

# 维护信息

K-D Tree可以当做线段树来用，只不过每一个节点都维护的是一个 $k$ 维长方体。

还是刚才这个图，我们将每一个节点维护的矩形的范围定为这样一块：

<img src="https://s2.loli.net/2022/06/22/sA9RILfw3mPFr8X.png" width="60%">

上面就是 $K$ 这个节点维护的矩形范围。

其父节点，$F$，维护的范围如下：

<img src="https://s2.loli.net/2022/06/22/Vwvfgl6sFzpbGHa.png" width="60%">

我们可以看出，K-D Tree 维护的矩形范围十分类似线段树维护的区间范围，某一个节点维护的矩形一定会覆盖其两个子节点维护的矩形，且其两个子节点维护的矩形的并就是这个节点维护的矩形。

所以我们可以对其进行类似线段树上的操作，比如说统计区间和什么的。

# 代码实现

因为KDT是基于替罪羊树的，这里只看与替罪羊树不同的地方。

## pushup

因为维护信息不同，pushup也不一样了。

我们pushup的时候需要将当前节点维护的区间的范围也更新上。

``` cpp
void pushup(int p)
{
	tr[p].l[0] = tr[p].r[0] = tr[p].p.x[0];
	tr[p].l[1] = tr[p].r[1] = tr[p].p.x[1];
	tr[p].sum = tr[p].p.w;
	tr[p].sz = 1;
	if(tr[p].ls)
	{
		tr[p].l[0] = min(tr[p].l[0], tr[tr[p].ls].l[0]);
		tr[p].l[1] = min(tr[p].l[1], tr[tr[p].ls].l[1]);
		tr[p].r[0] = max(tr[p].r[0], tr[tr[p].ls].r[0]);
		tr[p].r[1] = max(tr[p].r[1], tr[tr[p].ls].r[1]);
		tr[p].sum += tr[tr[p].ls].sum;
		tr[p].sz += tr[tr[p].ls].sz;
	}
	if(tr[p].rs)
	{
		tr[p].l[0] = min(tr[p].l[0], tr[tr[p].rs].l[0]);
		tr[p].l[1] = min(tr[p].l[1], tr[tr[p].rs].l[1]);
		tr[p].r[0] = max(tr[p].r[0], tr[tr[p].rs].r[0]);
		tr[p].r[1] = max(tr[p].r[1], tr[tr[p].rs].r[1]);
		tr[p].sum += tr[tr[p].rs].sum;
		tr[p].sz += tr[tr[p].rs].sz;
	}
}
```

## 重构

我们重构的时候需要注意，不再是直接取当前区间的mid，而是用`nth_element`来取出当前区间的中位数。
为此，我们还需要分别给两个维度写两个比较函数。
更高维的可以尝试使用全局变量。

``` cpp
bool cmp0(Point a, Point b)
{
	return a.x[0] < b.x[0];
}
bool cmp1(Point a, Point b)
{
	return a.x[1] < b.x[1];
}
bool canrbu(int p)
{
	return (1.0 * max(tr[tr[p].ls].sz, tr[tr[p].rs].sz)) >= (alpha * tr[p].sz);
}
Point ldr[N];
int ldc;
void rbuunf(int p)
{
	if(!p)return;
	rbuunf(tr[p].ls);
	if(tr[p].p.w)
	{
		ldr[++ldc] = tr[p].p;
		rec[++tt] = p;
	}
	rbuunf(tr[p].rs);
}
int rbubld(int l, int r, int k)
{
	if(l > r)return 0;
	int mid = (l + r) >> 1;
	int p = newnode();
	nth_element(ldr + l, ldr + mid, ldr + r + 1, k ? cmp1 : cmp0);
	tr[p].p = ldr[mid];
	tr[p].ls = rbubld(l, mid - 1, k ^ 1);
	tr[p].rs = rbubld(mid + 1, r, k ^ 1);
	pushup(p);
	return p;
}
void rbuild(int &p, int k)
{
	ldc = 0;
	rbuunf(p);
	p = rbubld(1, ldc, k);
}
```

## 查询

这里是查询的矩形内的权值和。

对于一个矩形来说，我们需要看当前查询区间是否包含这个矩形。
如果完全包含的话就直接返回矩形权值和，如果不完全包含的话就需要往下递归了，而如果完全不包含的话就直接返回0即可。

``` cpp
inline bool inc(Point p, int x1, int y1, int x2, int y2)
{
	return (p.x[0] >= x1) && (p.x[0] <= x2) && (p.x[1] >= y1) && (p.x[1] <= y2);
}
inline bool inc(int p, int x1, int y1, int x2, int y2)
{
	return (tr[p].l[0] >= x1) && (tr[p].r[0] <= x2) && (tr[p].l[1] >= y1) && (tr[p].r[1] <= y2);
}
inline bool exc(int p, int x1, int y1, int x2, int y2)
{
	return (tr[p].l[0] > x2) || (tr[p].r[0] < x1) || (tr[p].l[1] > y2) || (tr[p].r[1] < y1);
}
int query(int p, int x1, int y1, int x2, int y2)
{
	if(!p)return 0;
	if(exc(p, x1, y1, x2, y2))return 0;
	if(inc(p, x1, y1, x2, y2))return tr[p].sum;
	int res = 0;
	if(inc(tr[p].p, x1, y1, x2, y2))res += tr[p].p.w;
	res += query(tr[p].ls, x1, y1, x2, y2);
	res += query(tr[p].rs, x1, y1, x2, y2);
	return res;
}
```

## 全部加起来

参考代码：

{% note success 参考代码 %}
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 500010;
int n, k;
struct Point
{
	int x[2], w;
	Point() {};
	Point(int _x, int _y, int _w)
	{
		x[0] = _x, x[1] = _y, w = _w;
	}
	friend bool operator == (const Point &a, const Point &b)
	{
		return (a.x[0] == b.x[0]) && (a.x[1] == b.x[1]);
	}
};
const double alpha = 0.75;
struct KDT
{
	int ls, rs;
	int l[2], r[2];//[l,u],[r,d]
	int sum, sz;
	Point p;
}tr[N];
int rt, idx = 0;
int rec[N], tt;

int newnode()
{
	if(tt)return rec[tt--];
	else return ++idx;
}

void pushup(int p)
{
	tr[p].l[0] = tr[p].r[0] = tr[p].p.x[0];
	tr[p].l[1] = tr[p].r[1] = tr[p].p.x[1];
	tr[p].sum = tr[p].p.w;
	tr[p].sz = 1;
	if(tr[p].ls)
	{
		tr[p].l[0] = min(tr[p].l[0], tr[tr[p].ls].l[0]);
		tr[p].l[1] = min(tr[p].l[1], tr[tr[p].ls].l[1]);
		tr[p].r[0] = max(tr[p].r[0], tr[tr[p].ls].r[0]);
		tr[p].r[1] = max(tr[p].r[1], tr[tr[p].ls].r[1]);
		tr[p].sum += tr[tr[p].ls].sum;
		tr[p].sz += tr[tr[p].ls].sz;
	}
	if(tr[p].rs)
	{
		tr[p].l[0] = min(tr[p].l[0], tr[tr[p].rs].l[0]);
		tr[p].l[1] = min(tr[p].l[1], tr[tr[p].rs].l[1]);
		tr[p].r[0] = max(tr[p].r[0], tr[tr[p].rs].r[0]);
		tr[p].r[1] = max(tr[p].r[1], tr[tr[p].rs].r[1]);
		tr[p].sum += tr[tr[p].rs].sum;
		tr[p].sz += tr[tr[p].rs].sz;
	}
}

bool cmp0(Point a, Point b)
{
	return a.x[0] < b.x[0];
}
bool cmp1(Point a, Point b)
{
	return a.x[1] < b.x[1];
}
bool canrbu(int p)
{
	return (1.0 * max(tr[tr[p].ls].sz, tr[tr[p].rs].sz)) >= (alpha * tr[p].sz);
}
Point ldr[N];
int ldc;
void rbuunf(int p)
{
	if(!p)return;
	rbuunf(tr[p].ls);
	if(tr[p].p.w)
	{
		ldr[++ldc] = tr[p].p;
		rec[++tt] = p;
	}
	rbuunf(tr[p].rs);
}
int rbubld(int l, int r, int k)
{
	if(l > r)return 0;
	int mid = (l + r) >> 1;
	int p = newnode();
	nth_element(ldr + l, ldr + mid, ldr + r + 1, k ? cmp1 : cmp0);
	tr[p].p = ldr[mid];
	tr[p].ls = rbubld(l, mid - 1, k ^ 1);
	tr[p].rs = rbubld(mid + 1, r, k ^ 1);
	pushup(p);
	return p;
}
void rbuild(int &p, int k)
{
	ldc = 0;
	rbuunf(p);
	p = rbubld(1, ldc, k);
}

void insert(int &p, Point v, int k)
{
	if(!p)
	{
		p = newnode();
		tr[p].ls = tr[p].rs = 0;
		tr[p].p = v;
		pushup(p);
		return;
	}
	if(v.x[k] <= tr[p].p.x[k])insert(tr[p].ls, v, k ^ 1);
	else insert(tr[p].rs, v, k ^ 1);
	pushup(p);
	if(canrbu(p))rbuild(p, k);
}

inline bool inc(Point p, int x1, int y1, int x2, int y2)
{
	return (p.x[0] >= x1) && (p.x[0] <= x2) && (p.x[1] >= y1) && (p.x[1] <= y2);
}
inline bool inc(int p, int x1, int y1, int x2, int y2)
{
	return (tr[p].l[0] >= x1) && (tr[p].r[0] <= x2) && (tr[p].l[1] >= y1) && (tr[p].r[1] <= y2);
}
inline bool exc(int p, int x1, int y1, int x2, int y2)
{
	return (tr[p].l[0] > x2) || (tr[p].r[0] < x1) || (tr[p].l[1] > y2) || (tr[p].r[1] < y1);
}
int query(int p, int x1, int y1, int x2, int y2)
{
	if(!p)return 0;
	if(exc(p, x1, y1, x2, y2))return 0;
	if(inc(p, x1, y1, x2, y2))return tr[p].sum;
	int res = 0;
	if(inc(tr[p].p, x1, y1, x2, y2))res += tr[p].p.w;
	res += query(tr[p].ls, x1, y1, x2, y2);
	res += query(tr[p].rs, x1, y1, x2, y2);
	return res;
}

int main()
{
	scanf("%d", &n);
	int lastans = 0, op;
	while(scanf("%d", &op), op != 3)
	{
		if(op == 1)
		{
			int x, y, w;
			scanf("%d%d%d", &x, &y, &w);
			x ^= lastans, y ^= lastans, w ^= lastans;
			insert(rt, Point(x, y, w), 0);
		}
		else if(op == 2)
		{
			int x1, y1, x2, y2;
			scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
			x1 ^= lastans, y1 ^= lastans, x2 ^= lastans, y2 ^= lastans;
			lastans = query(rt, x1, y1, x2, y2);
			printf("%d\n", lastans);
		}
	}
}
```
{% endnote %}

# 例题

## 洛谷 P4148 简单题

[题目链接](https://www.luogu.com.cn/problem/P4148)

上面放的代码就是该题的代码。

