---
title: P4588 [TJOI2018] 数学计算 题解
date: 2022-06-30
tags:
	- 题解
	- 线段树
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">数学计算</div>
<div id="problem-info-from">TJOI 2018</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4588">Luogu P4588</a></li><li><a href="https://loj.ac/p/2573">LibreOJ L2573</a></li><li><a href="https://www.acwing.com/problem/content/3004/">AcWing 3001</a></li></ul></div>

----

题目要求我们维护一个初始值为1的数字 $x$，要求支持对其乘以一个数和除以一个数，并在每一次操作后输出 $x \bmod{M}$。

同时，题目保证我们除以的数之前被乘过，且每一个被乘过的数至多只会被除一次。
这样就可以将除法操作转化为撤回一次乘法操作。

这样我们需要做的就是，维护一串数的连乘积，同时支持向这一串数里面加入数和去掉数。

我们可以尝试使用乘法逆元做，但是这个 $M$ 不一定是质数，所以还需要判整除，非常麻烦。
或者可以使用中国剩余定理强行来一波，但是会更麻烦。

我们可以尝试使用线段树来维护区间乘积。
首先将所有的的数值全都初始化成1，然后每一次做单点修改区间查询即可。
甚至不需要区间查询，只需要查询根节点的值就可以了。

代码如下：

``` cpp
#include <bits/stdc++.h>
using namespace std;
int n, mod;
const int N = 100010;
struct segtree
{
	int l, r;
	long long v;
}node[N << 2];
void build(int p, int l, int r)
{
	node[p].l = l;
	node[p].r = r;
	if (l == r)
	{
		node[p].v = 1;
		return;
	}
	int mid = (l + r) >> 1;
	build(p << 1, l, mid);
	build(p << 1 | 1, mid + 1, r);
	node[p].v = (node[p << 1].v * node[p << 1 | 1].v) % mod;
	return;
}
void segchg(int p, int ed, int k)
{
	if ((node[p].r == ed) && (node[p].l == ed))
	{
		node[p].v = k;
		return;
	}
	int mid = (node[p].r + node[p].l) >> 1;
	if (mid >= ed)segchg(p << 1, ed, k);
	if (mid < ed)segchg(p << 1 | 1, ed, k);
	node[p].v = (node[p << 1].v * node[p << 1 | 1].v) % mod;
	return;
}
int main()
{
	int t;
	scanf("%d", &t);
	while (t--)
	{
		int m;
		scanf("%d%d", &m, &mod);
		build(1, 1, N);
		int cnt = 1;
		for (int i = 1; i <= m; i++)
		{
			int op, x;
			scanf("%d%d", &op, &x);
			if (op == 1)
			{
				segchg(1, i + 1, x);
				printf("%lld\n", node[1].v % mod);
			}
			else if (op == 2)
			{
				segchg(1, x + 1, 1);
				printf("%lld\n", node[1].v % mod);
			}
		}
	}
	return 0;
}
```

