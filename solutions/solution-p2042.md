---
title: P2042 [NOI2005] 维护数列 题解
date: 2022-04-18
tags:
	- 题解
	- 平衡树
categories:
	题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">维护数列</div>
<div id="problem-info-from">NOI 2005</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2042">Luogu P2042</a></li><li><a href="https://www.acwing.com/problem/content/957/">AcWing 955</a></li></ul></div>

----

题目要求我们维护一个数列，支持下面六种操作：

1. 插入：在当前数列的第 $posi$ 个数字后面插入 $c_1,c_2,\cdots,c_tot$ 共计 $tot$ 个数字；
2. 删除：从当前数列的第 $posi$ 个数字开始连续删除 $tot$ 个数字；
3. 推平：从当前数列的第 $posi$ 个数字开始的连续 $tot$ 个数字改为 $c$；
4. 翻转：将区间 $\[ posi,posi+tot-1 \]$ 翻转；
5. 求和：求出区间 $\[ posi,posi+tot-1 \]$ 内所有数字的和；
6. 求最大子序列和：求出当前数列中最大的一段子序列并输出其序列和。

看见区间翻转我就想到了平衡树。

幸好上面需要我们支持的操作都是可以利用平衡树来维护的。

我们在维护区间翻转的时候就用到了懒标记的思想，这里我们也可以使用懒标记。

我们的结构体长的是这个样子：

``` cpp
struct Node
{
	int s[2], p, v;
	int rev, same;
	int sz, sum, ms, ls, rs;
}tr[N];
```

其中分别维护的是：

- `s[2]`：左右儿子；
- `p`：父亲；
- `v`：当前节点的值；
- `rev`：翻转懒标记；
- `same`：推平懒标记；
- `sz`：子树大小；
- `sum`：区间和；
- `ms`：区间最大子序列和；
- `ls`：区间最大前缀和；
- `rs`：区间最大后缀和。

我们初始化的时候这样初始化：

``` cpp
void init(int _v, int _p)
{
	s[0] = s[1] = 0;
	p = _p, v = _v;
	rev = same = 0;
	sz = 1;
	sum = ms = v;
	ls = rs = max(v, 0);
}
```

然后是区间合并：

``` cpp
void pushup(int x)
{
	auto &u = tr[x], &l = tr[u.s[0]], &r = tr[u.s[1]];
	u.sz = l.sz + r.sz + 1;
	u.sum = l.sum + r.sum + u.v;
	u.ls = max(l.ls, l.sum + u.v + r.ls);
	u.rs = max(r.rs, r.sum + u.v + l.rs);
	u.ms = max(max(l.ms, r.ms), l.rs + u.v + r.ls);
}
```

维护当前区间的最大子序列可以看[这里](/OI/segment-tree/#%E5%8C%BA%E9%97%B4%E6%9C%80%E5%A4%A7%E5%AD%90%E6%AE%B5%E5%92%8C)的思路。

再然后是标记的下传：

``` cpp
void pushdown(int x)
{
	auto &u = tr[x], &l = tr[u.s[0]], &r = tr[u.s[1]];
	if(u.same)
	{
		u.same = u.rev = 0;
		if(u.s[0]) l.same = 1, l.v = u.v, l.sum = l.v * l.sz;
		if(u.s[1]) r.same = 1, r.v = u.v, r.sum = r.v * r.sz;
		if(u.v > 0)
		{
			if(u.s[0]) l.ms = l.ls = l.rs = l.sum;
			if(u.s[1]) r.ms = r.ls = r.rs = r.sum;
		}
		else
		{
			if(u.s[0]) l.ms = l.v, l.ls = l.rs = 0;
			if(u.s[1]) r.ms = r.v, r.ls = r.rs = 0;
		}
	}
	else if(u.rev)
	{
		u.rev = 0, l.rev ^= 1, r.rev ^= 1;
		swap(l.ls, l.rs), swap(r.ls, r.rs);
		swap(l.s[0], l.s[1]), swap(r.s[0], r.s[1]);
	}
}
```

区间推平的优先级是先于区间翻转的，且如果这个区间整体被推平了，那么这个区间翻转了和没翻转没有什么区别，所以每一次标记下传的时候只需要下传两者其一即可。

然后是六种操作：

1. 插入一整段区间的操作与一开始建树的时候差不多。
	我们需要首先预留出来新区间的位置，比如说 $posi+1$ 节点的左子树。此时这棵树长这个样子：
	![p2042-1.bmp](\pics\p2042-1.bmp)
	我们就是要在肩头指向的位置插入我们的新序列。
	然后就像一开始的时候那样，基于新读入的序列建立一棵平衡树，根节点为 $u$。
	然后让 $u$ 成为 $posi+1$ 的左儿子，就像这样：
	![p2042-2.bmp](\pics\p2042-2.bmp)
	现在我们就在 $posi$ 后面插入了一段新的序列。
2. 删除一整段区间的操作与正常的删除操作没有什么区别。
	这里就不讲了。
3. 区间推平与区间翻转的道理类似，都是给整个区间打上一个标记，同时区间推平还会额外修改一下区间和、区间最大子段和、区间最大前缀和与区间最大后缀和，别的就没有什么好说的了。
4. 区间求和的思路与区间翻转、区间推平类似，只不过不是修改而是查询。截出来询问区间之后直接输出值即可。
5. 上面的五种操作都需要截出对应区间，但是询问区间最大子段和只需要访问当前根节点即可。

最后放一下总的代码：

{% note success 示例代码 %}
``` cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 500010, INF = 1000000000;
int n, m;
struct Node
{
	int s[2], p, v;
	int rev, same;
	int sz, sum, ms, ls, rs;
	void init(int _v, int _p)
	{
		s[0] = s[1] = 0;
		p = _p, v = _v;
		rev = same = 0;
		sz = 1;
		sum = ms = v;
		ls = rs = max(v, 0);
	}
}tr[N];
int rt, recycle[N], tt;
int w[N];
void pushup(int x)
{
	auto &u = tr[x], &l = tr[u.s[0]], &r = tr[u.s[1]];
	u.sz = l.sz + r.sz + 1;
	u.sum = l.sum + r.sum + u.v;
	u.ls = max(l.ls, l.sum + u.v + r.ls);
	u.rs = max(r.rs, r.sum + u.v + l.rs);
	u.ms = max(max(l.ms, r.ms), l.rs + u.v + r.ls);
}
void pushdown(int x)
{
	auto &u = tr[x], &l = tr[u.s[0]], &r = tr[u.s[1]];
	if(u.same)
	{
		u.same = u.rev = 0;
		if(u.s[0]) l.same = 1, l.v = u.v, l.sum = l.v * l.sz;
		if(u.s[1]) r.same = 1, r.v = u.v, r.sum = r.v * r.sz;
		if(u.v > 0)
		{
			if(u.s[0]) l.ms = l.ls = l.rs = l.sum;
			if(u.s[1]) r.ms = r.ls = r.rs = r.sum;
		}
		else
		{
			if(u.s[0]) l.ms = l.v, l.ls = l.rs = 0;
			if(u.s[1]) r.ms = r.v, r.ls = r.rs = 0;
		}
	}
	else if(u.rev)
	{
		u.rev = 0, l.rev ^= 1, r.rev ^= 1;
		swap(l.ls, l.rs), swap(r.ls, r.rs);
		swap(l.s[0], l.s[1]), swap(r.s[0], r.s[1]);
	}
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
			if((tr[y].s[1] == x) ^ (tr[z].s[1] == y)) rotate(x);
			else rotate(y);
		rotate(x);
	}
	if(!k) rt = x;
}
int get_k(int k)
{
	int u = rt;
	while(u)
	{
		pushdown(u);
		if(tr[tr[u].s[0]].sz >= k) u = tr[u].s[0];
		else if(tr[tr[u].s[0]].sz + 1 == k) return u;
		else k -= tr[tr[u].s[0]].sz + 1, u = tr[u].s[1];
	}
	return 0;
}
int build(int l, int r, int p)
{
	int mid = (l + r) >> 1;
	int u = recycle[tt--];
	tr[u].init(w[mid], p);
	if(l < mid) tr[u].s[0] = build(l, mid - 1, u);
	if(mid < r) tr[u].s[1] = build(mid + 1, r, u);
	pushup(u);
	return u;
}
void del(int u)
{
	if(tr[u].s[0]) del(tr[u].s[0]);
	if(tr[u].s[1]) del(tr[u].s[1]);
	recycle[++tt] = u;
}
int main()
{
	for(int i = 1; i < N; i++) recycle[++tt] = i;
	scanf("%d%d", &n, &m);
	tr[0].ms = w[0] = w[n + 1] = -INF;
	for(int i = 1; i <= n; i++) scanf("%d", &w[i]);
	rt = build(0, n + 1, 0);
	char op[20];
	while(m--)
	{
		scanf("%s", op);
		if(!strcmp(op, "INSERT"))
		{
			int posi, tot;
			scanf("%d%d", &posi, &tot);
			for(int i = 0; i < tot; i++) scanf("%d", &w[i]);
			int l = get_k(posi + 1), r = get_k(posi + 2);
			splay(l, 0), splay(r, l);
			int u = build(0, tot - 1, r);
			tr[r].s[0] = u;
			pushup(r), pushup(l);
		}
		else if(!strcmp(op, "DELETE"))
		{
			int posi, tot;
			scanf("%d%d", &posi, &tot);
			int l = get_k(posi), r = get_k(posi + tot + 1);
			splay(l, 0), splay(r, l);
			del(tr[r].s[0]);
			tr[r].s[0] = 0;
			pushup(r), pushup(l);
		}
		else if(!strcmp(op, "MAKE-SAME"))
		{
			int posi, tot, c;
			scanf("%d%d%d", &posi, &tot, &c);
			int l = get_k(posi), r = get_k(posi + tot + 1);
			splay(l, 0), splay(r, l);
			auto &son = tr[tr[r].s[0]];
			son.same = 1, son.v = c, son.sum = c * son.sz;
			if(c > 0) son.ms = son.ls = son.rs = son.sum;
			else son.ms = c, son.ls = son.rs = 0;
			pushup(r), pushup(l);
		}
		else if(!strcmp(op, "REVERSE"))
		{
			int posi, tot;
			scanf("%d%d", &posi, &tot);
			int l = get_k(posi), r = get_k(posi + tot + 1);
			splay(l, 0), splay(r, l);
			auto &son = tr[tr[r].s[0]];
			son.rev ^= 1;
			swap(son.ls, son.rs);
			swap(son.s[0], son.s[1]);
			pushup(r), pushup(l);
		}
		else if(!strcmp(op, "GET-SUM"))
		{
			int posi, tot;
			scanf("%d%d", &posi, &tot);
			int l = get_k(posi), r = get_k(posi + tot + 1);
			splay(l, 0), splay(r, l);
			printf("%d\n", tr[tr[r].s[0]].sum);
		}
		else printf("%d\n", tr[rt].ms);
	}
	return 0;
}
```
{% endnote %}


