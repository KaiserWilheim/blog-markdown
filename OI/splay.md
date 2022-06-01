---
title: Splay
zh-CN: true
date: 2022-05-24
tags:
	- 数据结构
	- 平衡树
categories: 
	数据结构
mathjax: true
---

Splay.

<!--more-->
（2022年5月24日重构）

Splay是一种很好地维护一棵二叉搜索树的方法。
如果不知道什么是二叉搜索树，请看[此页面](https://oi-wiki.org/ds/bst/)。

# 基本函数

Splay的基本思想是，通过一系列的旋转操作，维持整棵二叉搜索树的平衡。

首先，我们假设我们需要维护的结构体是这个样子的：

``` cpp
struct Node
{
	int s[2], fa;
	int v, w;
	int sz;
	void init(int _v, int _fa)
	{
		v = _v, fa = _fa;
		sz = w = 1;
	}
}tr[N];
```

解释：

- `s[2]`：左右子树。
- `fa`：父亲。
- `v`：节点权值。
- `w`：节点大小。
- `sz`：节点及其子树大小。

还有一些其他的东西就一一列举了，比如懒标记等等。

## 旋转

Splay之所以能够维持其平衡，依赖的就是这样的简单旋转操作。

### 左旋与右旋

对于这样一个图：

<img src="https://s2.loli.net/2022/01/07/i8JVbaGELdNQOAx.png" alt="splay1.png" width="40%" />

其中x节点有两个子树，A与B；x节点的父亲y节点还有一棵子树C；z节点是y节点的父亲。
我们将x节点旋转（这里x节点是其父亲y节点的左儿子，所以我们会将其右旋），结果是这个样子的：

<img src="https://s2.loli.net/2022/01/07/hlkn8QXvHYgCtyr.png" alt="splay2.png" width="40%" />

当然，如果我们把x旋转回去的话，那就是左旋操作了。

总体来看是这个样子：

<img src="https://s2.loli.net/2022/05/24/WQhxLqiVDOIzrmc.png" alt="splay3.png" width="80%" />

在进行旋转操作时，我们需要保持其中序遍历序列不变。

在刚刚的右旋操作中，我们来分析一下我们需要改变的边：

<img src="https://s2.loli.net/2022/03/23/uqVYj9LzJvDKZMn.png" alt="splay4.png" width="40%" />

就是这三条标红的边。

那么对于这三条边，我们分别进行重构操作。

代码如下：

``` cpp
int k = tr[y].s[1] == x;//x是y的哪个儿子
tr[z].s[tr[z].s[1] == y] = x, tr[x].fa = z;//重构z-x边
tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;//重构y-B边
tr[x].s[k ^ 1] = y, tr[y].fa = x;//重构x-y边
```

旋转完成之后，因为我们改变了树的结构，所以我们需要重新计算x和y的大小，有时候还有需要维护的其他信息。
注意这里需要先维护较低的y，再维护较高的x。

所以总的函数是这个样子的：

``` cpp
void pushup(int p)
{
	tr[p].sz = tr[p].w;
	if(tr[p].s[0])tr[p].sz += tr[tr[p].s[0]].sz;
	if(tr[p].s[1])tr[p].sz += tr[tr[p].s[1]].sz;
}
void rotate(int x)
{
	int y = tr[x].fa, z = tr[y].fa;
	int k = tr[y].s[1] == x;
	tr[z].s[tr[z].s[1] == y] = x, tr[x].fa = z;
	tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
	tr[x].s[k ^ 1] = y, tr[y].fa = x;
	pushup(y), pushup(x);
}
```

## 核心函数

我们在向splay中插入一个数之后，会强制将其旋转到根。
而在刚才的示例中，我们看到了，我们将x右旋之后，x就向上走了一点。
而经过不断的旋转，我们就可以让x节点走到根。

当然，我们也不是随便瞎转，因为旋转操作也是需要复杂度的。
而我们的最后目标是使之均摊之后达到尽量小的复杂度。
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

同时我们需要注意，因为我们根节点是随着我们的不断旋转而不断变化的，所以我们需要即使更新根节点的信息。

代码如下：

``` cpp
void splay(int x, int k)
{
	while(tr[x].fa != k)
	{
		int y = tr[x].fa, z = tr[y].fa;
		if(z != k)
		{
			if((tr[y].s[1] == x) ^ (tr[z].s[1] == y))
				rotate(x);
			else
				rotate(y);
		}
		rotate(x);
	}
	if(!k) rt = x;
}
```

# 其他操作

我们假设我们需要维护的是一棵可重序列的二叉搜索树。

## 插入

Splay的插入操作是比较复杂的。
假设我们需要向树中插入一个元素 $k$，那么我们分下列几种情况讨论：

- 如果树是空的，那就直接插入根节点。
- 如果当前节点的权值等于 $k$，那就增加当前节点的大小，并更新其与其父亲的信息。
- 否则就按照二叉搜索树的性质继续向下面的节点查找。

最后不要忘记将节点旋转到根。

``` cpp
void insert(int k)
{
	if(!rt)
	{
		tr[++idx].init(k, 0);
		rt = idx;
		return;
	}
	int p = rt, fa = 0;
	while(true)
	{
		if(tr[p].v == k)
		{
			tr[p].w++;
			pushup(p), pushup(fa);
			splay(p, 0);
			break;
		}
		fa = p;
		p = tr[p].s[tr[p].v < k];
		if(!p)
		{
			tr[++idx].init(k, fa);
			tr[fa].s[tr[fa].v < k] = idx;
			pushup(fa);
			splay(idx, 0);
			break;
		}
	}
}
```

## 删除

Splay不使用惰性删除，其删除操作也是比较复杂的。
假设我们想要删除节点 $x$。

首先，我们将 $x$ 旋转到根。

然后我们分类讨论：

- 如果 $x$ 的大小不为1，那就减少其大小。
- 否则直接合并其两棵子树。

合并两棵树的操作很简单。如果我们假设需要合并的两棵树 $x$ 和 $y$ 中，$x$ 的最大值大于 $y$ 的话，只需要将的 $x$ 的最大值旋转到根，同时将 $y$ 设置为其根节点的右子树即可。

``` cpp
void loschn(int k)
{
	get_rk(k);
	if(tr[rt].w > 1)//节点内部包含多个相同元素
	{
		tr[rt].w--;
		pushup(rt);
		return;
	}
	if(!tr[rt].s[0] && !tr[rt].s[1])//全树上下只剩这一个点了！
	{
		tr[rt].clear();
		rt = 0;
		return;
	}
	if(!tr[rt].s[0])//没有左子树，这个点是整棵树最小的点了
	{
		int p = rt;
		rt = tr[rt].s[1];
		tr[rt].fa = 0;
		tr[p].clear();
		return;
	}
	if(!tr[rt].s[1])//没有右子树，这个点是整棵树最大的点了
	{
		int p = rt;
		rt = tr[rt].s[0];
		tr[rt].fa = 0;
		tr[p].clear();
		return;
	}
	//一般情况
	int p = rt, x = precrs();
	tr[tr[p].s[1]].fa = x;
	tr[x].s[1] = tr[p].s[1];
	tr[p].clear();
	pushup(rt);
}
```

## 查询k的排名

``` cpp
int get_rk(int k)
{
	int res = 0, p = rt;
	while(true)
	{
		if(k < tr[p].v)//在左子树中
		{
			p = tr[p].s[0];
		}
		else
		{
			if(tr[p].s[0])res += tr[tr[p].s[0]].sz;
			if(k == tr[p].v)//就是这个点了！
			{
				splay(p, 0);
				return res + 1;
			}
			//搜索右子树
			res += tr[p].w;
			p = tr[p].s[1];
		}
	}
	return -1;
}
```

## 查询排名为k的数

``` cpp
int get_k(int k)
{
	int p = rt;
	while(true)
	{
		if(tr[tr[p].s[0]].sz >= k)//在左子树中
		{
			p = tr[p].s[0];
		}
		else
		{
			k -= tr[p].w;
			if(tr[p].s[0])k -= tr[tr[p].s[0]].sz;
			if(k <= 0)//就是这个点了！
			{
				splay(p, 0);
				return tr[p].v;
			}
			//搜索右子树
			p = tr[p].s[1];
		}
	}
	return -1;
}
```

## 查询k的前驱或后继

前驱定义为小于这个数的最大数，后继定义为大于这个数的最小数。

我们的思路是，先将其插入进去，这样它就会到根节点；然后查询其左子树内的最大值或右子树内的最小值即可。

``` cpp
int precsr()
{
	int p = tr[rt].s[0];
	if(!p)return p;
	while(tr[p].s[1])p = tr[p].s[1];
	splay(p, 0);
	return p;
}
int succsr()
{
	int p = tr[rt].s[1];
	if(!p)return p;
	while(tr[p].s[0])p = tr[p].s[0];
	splay(p, 0);
	return p;
}
```

# 例题

## 维护可重有序序列

洛谷[板子题](https://www.luogu.com.cn/problem/P3369)，要求我们支持维护一个有序序列，并支持上面讲的六种操作。

示例代码：[`Luogu P3369-splay`](https://gitee.com/kaiserwilheim/OIcodes/blob/master/Luogu/p3000-p3999/p3369/p3369_splay.cpp)

## 维护不可重序列，支持区间翻转

给个洛谷[板子题](https://www.luogu.com.cn/problem/P3391)的代码:

它这里面要求区间翻转，那么我们在进行每一次旋转操作时，我们首先将左边界的前驱旋转至根节点，接着再把右边界的后继旋转至根节点的下面，此时右边界的后继的左子树就是我们所要翻转的区间了。
我们顺便增加一个`flag`标记，用来标记翻转次数。

示例代码：[`Luogu P3391`](https://gitee.com/kaiserwilheim/OIcodes/blob/master/Luogu/p3000-p3999/p3391/p3391.cpp)
