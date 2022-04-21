---
title: P1486 [NOI2004] 郁闷的出纳员 题解
date: 2022-03-23
tags:
	- 题解
	- 平衡树
categories:
	题解
mathjax: true
---

Luogu P1486
LibreOJ #10145

NOI 2004

<!--more-->
----

这道题要求我们完成两个操作：插入和查询。

我们最多分别有 $10^5$ 次插入和 $10^5$ 次查询。
而在通常是方法中，查询和插入的复杂度都是很高的，尤其是查询前还需要进行一次排序。

所以我们需要降低复杂度。

正好，平衡树可以帮助我们减轻这个负担。

我们尝试使用平衡树来维护一个有序的序列，以避免任何不必要的排序操作，城区排序需要的复杂度。

FHQ的无旋Treap可以过，具体实现见[这篇博客](https://oi.baoshuo.ren/luogu-p1486/)；STL貌似也能水过，具体实现见[这篇博客](https://oi.baoshuo.ren/luogu-p1486/)；而我们这里采用的是Splay。

对于插入，我们就像平常插入一样，不断递归直到可以插入为止。
同时注意：如果这个员工还没有来就发现自己的工资不足工资下界，那么他就不会来了。我们直接忽略即可。

对于修改工资，我们存储一个 $\Delta$ 以方便随时加减工资。
假设某个员工的工资是 $k$ ，那么我们存到平衡树里面的数据是 $k - \Delta$。
$\Delta$ 初始为0，我们每次加工资或减工资只需要对 $\Delta$ 进行更改就可以了。

同时我们还需要注意离职的员工。每一次减工资的时候，我们都需要看一下有哪些员工需要离职。
方法很简单：查找工资下界对应的节点——旋上来——清空子树。

对于查找，我们只需要在平衡树的节点上面维护一个 $size$，到时候直接查找就行了。
查找的思路是这个样子的：
我们使用一个参数 $k$，从根节点开始搜索。
当当前的左子树的 $size \geq k$ 时，证明我们的目标节点在左子树里面，我们递归搜索左子树。
如果当前的左子树的 $size + 1 = k$ 时，证明我们的目标节点就是我们当前搜索到的节点，直接`return`。
其他情况就是证明我们的目标节点在右子树里面，我们把当前的 $k$ 减去左子树的 $size$，再减去代表当前节点的 $1$，然后带着新的 $k$ 去递归搜索右子树。

上代码：

{% note warning %}
注意：我这里使用了非ASCII字符`Δ`做变量名，这种情况下只能在C++20的情况下通过编译，请根据需要修改变量名。
{% endnote %}

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010, INF = 1e9;
int n, m, Δ;

struct Node
{
	int s[2], p, v;
	int size;
	void init(int _v, int _p)
	{
		v = _v, p = _p;
		size = 1;
	}
}tr[N];

int root, idx;

void pushup(int x)
{
	tr[x].size = tr[tr[x].s[0]].size + tr[tr[x].s[1]].size + 1;
}

void rotate(int x)
{
	int y = tr[x].p, z = tr[y].p;
	int k = tr[y].s[1] == x;
	tr[z].s[tr[z].s[1] == y] = x, tr[x].p = z;
	tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].p = y;
	tr[x].s[k ^ 1] = y, tr[y].p = x;
	pushup(y), pushup(x);
}

void splay(int x, int k)
{
	while(tr[x].p != k)
	{
		int y = tr[x].p, z = tr[y].p;
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
	if(!k) root = x;
}

int insert(int v)
{
	int u = root, p = 0;
	while(u) p = u, u = tr[u].s[v > tr[u].v];
	u = ++idx;
	if(p) tr[p].s[v > tr[p].v] = u;
	tr[u].init(v, p);
	splay(u, 0);
	return u;
}

int get(int v)
{
	int u = root, res;
	while(u)
	{
		if(tr[u].v >= v)
		{
			res = u;
			u = tr[u].s[0];
		}
		else
		{
			u = tr[u].s[1];
		}
	}
	return res;
}

int get_k(int k)
{
	int u = root;
	while(true)
	{
		if(tr[tr[u].s[0]].size >= k)
		{
			u = tr[u].s[0];
		}
		else if(tr[tr[u].s[0]].size + 1 == k)
		{
			return tr[u].v;
		}
		else
		{
			k -= tr[tr[u].s[0]].size + 1;
			u = tr[u].s[1];
		}
	}
	return -1;
}

int main()
{
	scanf("%d%d", &n, &m);
	int L = insert(-INF), R = insert(INF);
	int tot = 0;
	while(n--)
	{
		char op[10];
		int k;
		scanf("%s%d", &op, &k);
		if(op[0] == 'I')
		{
			if(k >= m)
			{
				k -= Δ;
				insert(k);
				tot++;
			}
		}
		else if(op[0] == 'A')
		{
			Δ += k;
		}
		else if(op[0] == 'S')
		{
			Δ -= k;
			R = get(m - Δ);
			splay(R, 0), splay(L, R);
			tr[L].s[1] = 0;
			pushup(L), pushup(R);
		}
		else
		{
			if(tr[root].size - 2 < k)
			{
				puts("-1");
			}
			else
			{
				printf("%d\n", get_k(tr[root].size - k) + Δ);
			}
		}
	}
	printf("%d\n", tot - (tr[root].size - 2));
	return 0;
}
```

