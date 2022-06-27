---
title: 替罪羊树
date: 2022-05-21
tags:
	- 数据结构
	- 平衡树
categories:
	- 数据结构
mathjax: true
---

替罪羊树。

<!-- more -->

<div id="problem-card-vis">false</div>

替罪羊树是一种平衡树。

# 热身

平衡树一般都是一个二叉搜索树，其满足中序遍历得到的序列就是我们需要维护的原序列。

当然，二叉搜索树可以不平衡，这样就可以构造一个特殊的数据使之退化成为一条链。

那我们怎么定义一棵二叉搜索树“不平衡”呢？

这里需要引入一个概念：平衡指数 $\alpha$。

一棵二叉搜索树的平衡常数等于其子节点大小与其大小的比值。
这里取的是最大值。

平衡常数 $\alpha$ 的取值是 $\alpha \in \[ 0.5 , 1 \]$。
其两个边界代表了两个极端情况：

当 $\alpha = 1$ 时，我们不管怎样建造搜索树都会被认为是平衡的，因为其子节点的子树大小永远不可能超过其本身的子树大小。

当 $\alpha = 0.5$ 时，我们每一个节点的子节点的子树大小必须恰好是其本身的子树大小的一半。
AVL树就在尽力维持这样的平衡，这就导致其代码十分冗长，没有能在OI上有太多实际的应用。
在[这里](https://www.cs.usfca.edu/~galles/visualization/AVLtree.html)有一个AVL树的可视化。

红黑树比较特殊，通过放宽一些过于严苛的要求，其追求的是 $\alpha = \frac{2}{3}$，同时降低了常数和代码长度。
在[这里](https://www.cs.usfca.edu/~galles/visualization/RedBlack.html)有一个红黑树的可视化。

其他的平衡树都是通过一些思想来维持 $\alpha$ 的尽量低。

基于[这里](https://riteme.site/blog/2016-4-6/scapegoat.html)的数据，我们可以大概得知不同平衡树的 $\alpha$ 大小：

| | 1 | 2 | 3 | 4 | 5 | 平均 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Splay | 0.758 | 0.588 | 0.582 | 0.612 | 0.759 | 0.659 |
| Treap | 0.766 | 0.578 | 0.601 | 0.587 | 0.781 | 0.662 |
| FHQ-Treap | 0.914 | 0.860 | 0.613 | 0.678 | 0.803 | 0.773 |

可见，一般的平衡树都能将 $\alpha$ 维持到 0.6 到 0.8 范围内。

## 替罪羊树

替罪羊树最大的特点就是暴力。

怎么暴力呢？

替罪羊树会将不平衡的子树进行重构来保证其平衡。
而其判断子树平衡与否就是根据刚才讲的平衡因数 $\alpha$，只不过这里是人为设定的，称之为平衡常数。

# 思想

## 暴力重构

替罪羊树之所以能够平衡，是在于其重构时不是瞎重构，而是将被重构的子树重构为一棵**完全二叉树**。

当然我们都知道这样费时又费力，更何况还是暴力重构的。

所以我们认为设定的平衡常数 $\alpha$ 在此时就起到了决定性的作用。
当其值合适的时候，我们就可以将所有的时间复杂度均摊到一个 $O(\log n)$ 的水平。

具体如何暴力重构就不用太多赘述了，我们可以使用简单的方法来保证线性建树，然后将新建的树接过来即可。

## 查询

替罪羊树的查询与其他二叉搜索树一样，并且因为其没有对树进行修改，还不会导致产生重构操作，所以最终时间复杂度为 $O(\log n)$。

## 插入

替罪羊树的插入操作与其他的二叉搜索树差不多。只不过因为其导致了树形态的改变，我们在插入完回溯的过程中还需要判断一下是否需要重构。

当然，还会有一条链上多棵子树不平衡的情况。
我们可以将最大的子树重构，但是这样在实际写代码的时候会略显复杂。
如果你真的很懒的话，只需要在回溯的时候找到第一棵不平衡的树重构即可，并且据说这个样子对于时间复杂度的影响不会很大。

## 删除

替罪羊树使用惰性删除，只需要将对应节点上代表节点内数据数量的标记自减一即可。

对于一个节点内数据数量为0的点，我们会忽略对其的任何操作，并在下一次重构时将其丢弃掉，除非再有插入操作将其插入回去。

# 实现

## 节点

替罪羊树的一个节点内需要存储很多信息。

``` cpp
struct Scapegoat
{
	int ls, rs;
	int w, wn;
	int s, sz, sd;
}tr[N];
```

解释一下：

- `ls`&`rs`：左右儿子。
- `w`：节点权值。
- `wn`：节点内数据数量。
- `s`：子树内节点个数。
- `sz`：子树内数据个数。
- `sd`：子树内不计删除节点的节点个数。

我们这样来（重新）计算当前节点的子树大小：

``` cpp
void calc(int p)
{
	tr[p].s = tr[tr[p].ls].s + tr[tr[p].rs].s + 1;
	tr[p].sz = tr[tr[p].ls].sz + tr[tr[p].rs].sz + tr[p].wn;
	tr[p].sd = tr[tr[p].ls].sd + tr[tr[p].rs].sd + (tr[p].wn != 0);
}
```

## 重构

我们重构分两种情况：一是子树不平衡了，即左右子树之一的大小占其本身子树大小的比例超过 $\alpha$；二是被删除的节点太多了，这样也会影响效率。

首先我们需要判断是否需要重构：

``` cpp
bool canrbu(int p)
{
	if(!tr[p].wn)return false;
	if(alpha * tr[p].s <= double(max(tr[tr[p].ls].s, tr[tr[p].rs].s)))return true;
	if(double(tr[p].sd) <= alpha * tr[p].s)return true;
	return false;
}//can rebuild
```

一句话版：

``` cpp
bool canrbu(int k)
{
	return tr[k].wn && (alpha * tr[k].s <= ( double )max(tr[tr[k].ls].s, tr[tr[k].rs].s) || ( double )tr[k].sd <= alpha * tr[k].s);
}//can rebuild
```

然后就是重构的具体操作：

首先我们将需要重构的子树经中序遍历展开之后存入数组中，然后将新得到的数组二分建树。

``` cpp
void rbuunf(int &ldc, int p)
{
	if(!p)return;
	rbuunf(ldc, tr[p].ls);
	if(tr[p].wn)ldr[++ldc] = p;
	rbuunf(ldc, tr[p].rs);
}//rebuild-unfold
```
``` cpp
int rbubld(int l, int r)
{
	if(l >= r)return 0;
	int mid = (l + r) >> 1;
	tr[ldr[mid]].ls = rbubld(l, mid);
	tr[ldr[mid]].rs = rbubld(mid + 1, r);
	calc(ldr[mid]);
	return ldr[mid];
}//rebuild-build
```
``` cpp
void rbuild(int &p)
{
	int ldc = 0;
	rbuunf(ldc, p);
	p = rbubld(1, ldc);
}//rebuild
```

## 插入

插入时，我们需要找到对应节点并 `tr[p].wn++`。如果没有节点就新建一个，回溯时需要判断是否能够重构，如果可以的话就重构。

``` cpp
void insert(int &p, int k)
{
	if(!p)
	{
		p = ++cnt;
		if(!rt)rt = 1;
		tr[p].w = k;
		tr[p].ls = tr[p].rs = 0;
		tr[p].wn = tr[p].s = tr[p].sz = tr[p].sd = 1;
	}
	else
	{
		if(tr[p].w == k)tr[p].wn++;
		else if(tr[p].w < k)insert(tr[p].rs, k);
		else insert(tr[p].ls, k);
		calc(p);
		if(canrbu(p))rbuild(p);
	}
}
```

## 删除

替罪羊树使用惰性删除，找到对应节点之后只需要 `tr[p].wn--` 即可。当然，回溯时候遇到可以重构的节点时要重构。

``` cpp
void loschn(int &p, int k)
{
	if(!p)return;
	if(tr[p].w == k)
	{
		if(tr[p].wn)tr[p].wn--;
	}
	else
	{
		if(tr[p].w < k)loschn(tr[p].rs, k);
		else loschn(tr[p].ls, k);
	}
	calc(p);
	if(canrbu(p))rbuild(p);
}//löschen，delete是关键字就不用了
```

# 查询函数

然后就是一些其他的函数，都是在二分查找树上进行查找操作的。

## `upper_bound`

与原来upper_bound的用途一样，返回第一个大于其权值的位置。

``` cpp
int uprbnd(int p, int k)
{
	if(!p)
		return 1;
	else if(tr[p].w == k && tr[p].wn)
		return tr[tr[p].ls].sz + tr[p].wn + 1;
	else if(tr[p].w > k)
		return uprbnd(tr[p].ls, k);
	else 
		return tr[tr[p].ls].sz + tr[p].wn + uprbnd(tr[p].rs, k);
}
```

还有一个其反义函数，相当于是对当前序列反转之后的结果进行upper_bound，返回的是第一个小于其权值的位置。

查询某一个数字的排名的时候可以使用 `uprgtr(rt,k)+1`。

``` cpp
int uprgtr(int p, int k)
{
	if(!p)
		return 0;
	else if(tr[p].w == k && tr[p].wn)
		return tr[tr[p].ls].sz;
	else if(tr[p].w < k)
		return tr[tr[p].ls].sz + tr[p].wn + uprgtr(tr[p].rs, k);
	else
		return uprgtr(tr[p].ls, k);
}
```

## `getk`

getk函数返回的是当前排名上的数字。

``` cpp
int getk(int p, int k)
{
	if(!p)
		return 0;
	else if(tr[tr[p].ls].sz < k && k <= tr[tr[p].ls].sz + tr[p].wn)
		return tr[p].w;
	else if(tr[tr[p].ls].sz + tr[p].wn < k)
		return getk(tr[p].rs, k - tr[tr[p].ls].sz - tr[p].wn);
	else
		return getk(tr[p].ls, k);
}
```

## 前驱与后继

将上面两个函数结合起来就可以了。

``` cpp
int precsr(int p, int k)
{
	return getk(p, uprgtr(p, k));
}//precursor
int succsr(int p, int k)
{
	return getk(p, uprbnd(p, k));
}//successor
```

{% note success 封装好的结构体 %}
``` cpp
double alpha = 0.75;
struct Scapegoat
{
	int ls[N], rs[N];
	int w[N], wn[N];
	int s[N], sz[N], sd[N];

	int cnt, rt;
	void calc(int p)
	{
		s[p] = s[ls[p]] + s[rs[p]] + 1;
		sz[p] = sz[ls[p]] + sz[rs[p]] + wn[p];
		sd[p] = sd[ls[p]] + sd[rs[p]] + (wn[p] != 0);
	}
	bool canrbu(int p)
	{
		return wn[p] && (alpha * s[p] <= double(max(s[ls[p]], s[rs[p]])) || double(sd[p] <= alpha * s[p]));
	}
	int ldr[N];
	void rbuunf(int &ldc, int p)
	{
		if(!p)return;
		rbuunf(ldc, ls[p]);
		if(wn[p])ldr[ldc++] = p;
		rbuunf(ldc, rs[p]);
	}
	int rbubld(int l, int r)
	{
		if(l >= r)return 0;
		int mid = (l + r) >> 1;
		ls[ldr[mid]] = rbubld(l, mid);
		rs[ldr[mid]] = rbubld(mid + 1, r);
		calc(ldr[mid]);
		return ldr[mid];
	}
	void rbuild(int &p)
	{
		int ldc = 0;
		rbuunf(ldc, p);
		p = rbubld(0, ldc);
	}
	void insert(int &p, int k)
	{
		if(!p)
		{
			p = ++cnt;
			if(!rt)rt = 1;
			w[p] = k;
			ls[p] = rs[p] = 0;
			wn[p] = s[p] = sz[p] = sd[p] = 1;
		}
		else
		{
			if(w[p] == k)wn[p]++;
			else if(w[p] < k)insert(rs[p], k);
			else insert(ls[p], k);
			calc(p);
			if(canrbu(p))rbuild(p);
		}
	}
	void loschn(int &p, int k)
	{
		if(!p)return;
		if(w[p] == k)
		{
			if(wn[p])wn[p]--;
		}
		else
		{
			if(w[p] < k)loschn(rs[p], k);
			else loschn(ls[p], k);
		}
		calc(p);
		if(canrbu(p))rbuild(p);
	}
	int uprbnd(int p, int k)
	{
		if(!p)
			return 1;
		else if(w[p] == k && wn[p])
			return sz[ls[p]] + wn[p] + 1;
		else if(w[p] > k)
			return uprbnd(ls[p], k);
		else return sz[ls[p]] + wn[p] + uprbnd(rs[p], k);
	}
	int uprgtr(int p, int k)
	{
		if(!p)
			return 0;
		else if(w[p] == k && wn[p])
			return sz[ls[p]];
		else if(w[p] < k)
			return sz[ls[p]] + wn[p] + uprgtr(rs[p], k);
		else
			return uprgtr(ls[p], k);
	}
	int getk(int p, int k)
	{
		if(!p)
			return 0;
		else if(sz[ls[p]] < k && k <= sz[ls[p]] + wn[p])
			return w[p];
		else if(sz[ls[p]] + wn[p] < k)
			return getk(rs[p], k - sz[ls[p]] - wn[p]);
		else
			return getk(ls[p], k);
	}
	int precrs(int p, int k)
	{
		return getk(p, uprgtr(p, k));
	}
	int succsr(int p, int k)
	{
		return getk(p, uprbnd(p, k));
	}
}tr;
```
{% endnote %}

# 例题

洛谷上的[板子](https://www.luogu.com.cn/problem/P3369)：

示例代码：[`Luogu P3369-scapegoat`](https://gitee.com/kaiserwilheim/OIcodes/blob/master/Luogu/p3000-p3999/p3369/p3369_scapegoat.cpp)


