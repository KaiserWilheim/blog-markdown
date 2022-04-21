---
title: Splay
zh-CN: true
date: 2022-01-07
tags:
	- 数据结构
	- 平衡树
categories: 
	数据结构
mathjax: true
---

Splay.

（因为写挂了而郁闷不已）

<!--more-->

Splay是一种很好地维护一棵二叉搜索树的方法。
如果不知道什么是二叉搜索树，请看[此页面](https://oi-wiki.org/ds/bst/)。

# 基本思想

Splay的基本思想是，通过一系列的旋转操作，维持整棵二叉搜索树的平衡。

# 旋转

对于一个节点，我们可以对它进行旋转。

## 左旋与右旋

对于这样一个图：

<img src="https://s2.loli.net/2022/01/07/i8JVbaGELdNQOAx.png" alt="splay1.png" width="40%" />

其中x节点有两个子树，A与B；x节点的父亲y节点还有一棵子树C；z节点是y节点的父亲。
我们将x节点旋转（这里x节点是其父亲y节点的左儿子，所以我们会将其右旋），结果是这个样子的：

<img src="https://s2.loli.net/2022/01/07/hlkn8QXvHYgCtyr.png" alt="splay2.png" width="40%" />

当然，如果我们把x旋转回去的话，那就是左旋操作了。

总体来看是这个样子：

![splay3.png](/pics/splay3.png)

在进行旋转操作时，我们需要保持其中序遍历序列不变。

在刚刚的右旋操作中，我们来分析一下我们需要改变的边：

<img src="https://s2.loli.net/2022/03/23/uqVYj9LzJvDKZMn.png" alt="splay4.png" width="40%" />

就是这三条标红的边。

那么对于这三条边，我们分别进行重构操作。

代码如下：

``` cpp
int k = tr[y].s[1] == x;//x是y的哪个儿子
tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;//重构z-x边
tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].p = y;//重构y-B边
tr[x].s[k ^ 1] = y, tr[y].p = x;//重构x-y边
```

# 核心函数

我们在向splay中插入一个数之后，会强制将其旋转到根。
而在刚才的示例中，我们看到了，我们将x右旋之后，x就向上走了一点。
而经过不断的旋转，我们就可以让x节点走到根。

当然，我们也不是随便瞎转，因为旋转操作也是需要复杂度的。
而我们的最后目标是达到尽量小的复杂度。
引用闫学灿的一句话：
> “如果我们瞎转的话，就达不到$O(\log n)$的复杂度了。”
所以我们要根据x所处的位置来制定不同的旋转方案。

首先，我们对于x可能出现的几种情况分析一下：

0. x就是目标节点。
那么就不用转了。

1. x是目标节点的子节点。
那我们直接转一下x就可以了。

对于x的父亲也不是目标节点的情况，我们也分两种情况讨论。

2. x的父亲也不是目标节点，且x与其父亲的所在子树类型相同。
可以理解为x，x的父亲和x的父亲的父亲三个节点在一条直线上。
这样的话，我们就先旋转x的父节点，再旋转x。

3. x的父亲也不是目标节点，且x与其父亲的所在子树类型不同。
可以理解为x，x的父亲和x的父亲的父亲三个节点的连线是一条折线。
这样的话，我们旋转两次x。

这样不断判断，直到x到达目标节点。

# 基本操作

## 插入

``` cpp
int insert(int v)
{
	int u = root, p = 0;
	while(u) p = u, u = tr[u].s[v > tr[u].v];//如果节点非空，就一直向下跳对应的子树
	u = ++idx;//开一个新的节点
	if(p) tr[p].s[v > tr[p].v] = u;//如果非根节点，那么就更新一下子节点信息
	tr[u].init(v, p);//初始化节点值
	splay(u, 0);//旋转到根
	return u;
}
```

## 查询第k个值

这里查询的是splay中序遍历得到的序列中第k个值。

``` cpp
int get_k(int k)
{
	int u = root;
	while(true)
	{
		if(tr[tr[u].s[0]].size >= k)//在左子树中
		{
			u = tr[u].s[0];
		}
		else if(tr[tr[u].s[0]].size + 1 == k)//就是这个点了！
		{
			return tr[u].v;
		}
		else//搜索右子树
		{
			k -= tr[tr[u].s[0]].size + 1;
			u = tr[u].s[1];
		}
	}
	return -1;
}
```

# 代码

给个洛谷[板子题](https://www.luogu.com.cn/problem/P3391)的代码:

它这里面要求区间翻转，那么我们在进行每一次旋转操作时，我们首先将左边界的前驱旋转至根节点，接着再把右边界的后继旋转至根节点的下面，此时右边界的后继的左子树就是我们所要翻转的区间了。
我们顺便增加一个`flag`标记，用来标记翻转次数。

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;
int n, m;
struct Node
{
	int s[2], f, v;
	int size, flag;
	void init(int _v, int _f)
	{
		v = _v, f = _f;
		size = 1;
	}
}tr[N];
int rt, idx;
void pushup(int x)
{
	tr[x].size = tr[tr[x].s[0]].size + tr[tr[x].s[1]].size + 1;
}
void pushdown(int x)
{
	if(tr[x].flag)
	{
		swap(tr[x].s[0], tr[x].s[1]);
		tr[tr[x].s[0]].flag ^= 1;
		tr[tr[x].s[1]].flag ^= 1;
		tr[x].flag = 0;
	}
}
void rotate(int x)
{
	int y = tr[x].f, z = tr[y].f;
	int k = tr[y].s[1] == x;
	tr[z].s[tr[z].s[1] == y] = x, tr[x].f = z;
	tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].f = y;
	tr[x].s[k ^ 1] = y, tr[y].f = x;
	pushup(y), pushup(x);
}
void splay(int x, int k)
{
	while(tr[x].f != k)
	{
		int y = tr[x].f, z = tr[y].f;
		if(z != k)
		{
			if((tr[y].s[1] == x) ^ (tr[z].s[1] == y))
			{
				rotate(x);
			}
			else
			{
				rotate(y);
			}
		}
		rotate(x);
	}
	if(!k) rt = x;
}
void insert(int v)
{
	int u = rt, f = 0;
	while(u) f = u, u = tr[u].s[v > tr[u].v];
	u = ++idx;
	if(f) tr[f].s[v > tr[f].v] = u;
	tr[u].init(v, f);
	splay(u, 0);
}
int get_k(int k)
{
	int u = rt;
	while(true)
	{
		pushdown(u);
		if(tr[tr[u].s[0]].size >= k)
		{
			u = tr[u].s[0];
		}
		else if(tr[tr[u].s[0]].size + 1 == k)
		{
			return u;
		}
		else
		{
			k -= tr[tr[u].s[0]].size + 1;
			u = tr[u].s[1];
		}
	}
	return -1;
}
void output(int u)
{
	pushdown(u);
	if(tr[u].s[0]) output(tr[u].s[0]);
	if(tr[u].v >= 1 && tr[u].v <= n) printf("%d ", tr[u].v);
	if(tr[u].s[1]) output(tr[u].s[1]);
}
int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 0; i <= n + 1; i++) insert(i);
	while(m--)
	{
		int l, r;
		scanf("%d%d", &l, &r);
		l = get_k(l), r = get_k(r + 2);
		splay(l, 0), splay(r, l);
		tr[tr[r].s[0]].flag ^= 1;
	}
	output(rt);
	return 0;
}
```

