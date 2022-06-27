---
title: L2980. [THUSCH 2017] 大魔法师 题解
date: 2022-04-17
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
<div id="problem-info-name">大魔法师</div>
<div id="problem-info-from">THUSCH 2017</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P7453">Luogu P7453</a></li><li><a href="https://loj.ac/p/2980">LibreOJ L2980</a></li></ul></div>

----

首先看题：

题目要求我们在 $\[ 1,n \]$ 的区间上维护三个值 $A_i,B_i,C_i$，要求支持查询区间和和下面六中操作：

1. 区间内 $A_i = A_i + B_i$；
2. 区间内 $B_i = B_i + C_i$；
3. 区间内 $C_i = C_i + A_i$；
4. 区间内 $A_i = A_i + k$（$k$ 给定）；
5. 区间内 $B_i = B_i \times k$（$k$ 给定）；
6. 区间内 $C_i = k$（$k$ 给定）。

显然这道题需要使用线段树来维护区间操作。
但是我们不能给每一个操作附上一个懒标记，最后懒标记下放的时候还不得麻烦死。

我们可以尝试一下转化一下我们的操作。

众所周知，一个矩阵乘以一个单位矩阵 $I$ 还等于它本身。
单位矩阵的形状是这样的：$\begin{bmatrix}1 & & &\\\\ & 1 & & \\\\ & & \ddots & \\\\ & & & 1 \end{bmatrix}$

如果我们将 $A_i,B_i,C_i$ 三个数值化作一个矩阵的话，那么这个矩阵 $\begin{bmatrix} A_i & B_i & C_i \end{bmatrix}$ 所对应的单位矩阵是这个样子的：$\begin{bmatrix}1&&\\\\&1&\\\\&&1\end{bmatrix}$。

而 $\begin{bmatrix} A_i & B_i & C_i \end{bmatrix}$ 想要变成 $\begin{bmatrix} A_i+B_i & B_i & C_i \end{bmatrix}$ 的话可以让它乘以一个 $\begin{bmatrix}1&&\\\\1&1&\\\\&&1\end{bmatrix}$。$(2,1)$ 处多出来的那个1就代表着给结果行矩阵的第**1**个位置加上**1**个原先的行矩阵矩阵第**2**个位置的值，也就相当于是给结果的 $A_i$ 加上了一个 $B_i$。

同理，第二个操作乘以的是一个 $\begin{bmatrix}1&&\\\\&1&\\\\&1&1\end{bmatrix}$，第三个操作乘以的是一个 $\begin{bmatrix}1&&1\\\\&1&\\\\&&1\end{bmatrix}$。

然后就是可恶的第四个操作。我们需要给 $A_i$ 加上一个 $k$，但是我们无法从刚才的房费里面推出来一个类似的方法。

于是我们可以考虑将我们维护的矩阵由 $\begin{bmatrix} A_i & B_i & C_i \end{bmatrix}$ 变为 $\begin{bmatrix} A_i & B_i & C_i & 1 \end{bmatrix}$。
这样我们就只需要乘以一个 $\begin{bmatrix}1&&&\\\\&1&&\\\\&&1&\\\\k&&&1\end{bmatrix}$ 即可。

为了适配我们新的需要维护的矩阵，前面三个就变成了 $\begin{bmatrix}1&&&\\\\1&1&&\\\\&&1&\\\\&&&1\end{bmatrix}$，$\begin{bmatrix}1&&&\\\\&1&&\\\\&1&1&\\\\&&&1\end{bmatrix}$ 和 $\begin{bmatrix}1&&1&\\\\&1&&\\\\&&1&\\\\&&&1\end{bmatrix}$。

第五个操作的思路也很简单，只需要乘以一个 $\begin{bmatrix}1&&&\\\\&k&&\\\\&&1&\\\\&&&1\end{bmatrix}$ 即可。

第六个操作有点难，我们可以先把 $C_i$ 变成0，通过乘以一个 $\begin{bmatrix}1&&&\\\\&1&&\\\\&&&\\\\&&&1\end{bmatrix}$，然后再乘以一个 $\begin{bmatrix}1&&&\\\\&1&&\\\\&&1&\\\\&&k&1\end{bmatrix}$，就可以给 $C_i$ 赋成 $k$ 了。
最终效果跟直接乘以 $\begin{bmatrix}1&&&\\\\&1&&\\\\&&&\\\\&&k&1\end{bmatrix}$ 效果一样。

于是我们的线段树只需要维护一个区间乘和区间求和即可。

矩阵结构体：

``` cpp
struct Matrix
{
	int n, m;
	ll a[5][5];
	Matrix() {}
	Matrix(int _n, int _m) :n(_n), m(_m)
	{
		memset(a, 0, sizeof(a));
	}
	Matrix operator + (const Matrix &rhs)
	{
		Matrix res(n, m);
		for(int i = 1; i <= n; i++)
			for(int j = 1; j <= m; j++)
				res.a[i][j] = (a[i][j] + rhs.a[i][j]) % p;
		return res;
	}
	Matrix operator * (const Matrix &rhs)
	{
		Matrix res(n, rhs.m);
		for(int i = 1; i <= n; i++)
			for(int j = 1; j <= rhs.m; j++)
				for(int k = 1; k <= m; k++)
					res.a[i][j] = (res.a[i][j] + a[i][k] * rhs.a[k][j]) % p;
		return res;
	}
};
```

总代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 250010;
const ll p = 998244353;
struct Matrix
{
	int n, m;
	ll a[5][5];
	Matrix() {}
	Matrix(int _n, int _m) :n(_n), m(_m)
	{
		memset(a, 0, sizeof(a));
	}
	Matrix operator + (const Matrix &rhs)
	{
		Matrix res(n, m);
		for(int i = 1; i <= n; i++)
			for(int j = 1; j <= m; j++)
				res.a[i][j] = (a[i][j] + rhs.a[i][j]) % p;
		return res;
	}
	Matrix operator * (const Matrix &rhs)
	{
		Matrix res(n, rhs.m);
		for(int i = 1; i <= n; i++)
			for(int j = 1; j <= rhs.m; j++)
				for(int k = 1; k <= m; k++)
					res.a[i][j] = (res.a[i][j] + a[i][k] * rhs.a[k][j]) % p;
		return res;
	}
};
Matrix num[N];
struct SegTree
{
	int l, r;
	Matrix sum, tag;
}tr[N << 2];
Matrix base1, base2, base3, base4, base5, base6;
Matrix base;
void pushdown(int p)
{
	auto &root = tr[p], &left = tr[p << 1], &rght = tr[p << 1 | 1];
	left.sum = left.sum * root.tag;
	rght.sum = rght.sum * root.tag;
	left.tag = left.tag * root.tag;
	rght.tag = rght.tag * root.tag;
	root.tag = base;
	return;
}
void build(int p, int l, int r)
{
	tr[p].l = l, tr[p].r = r;
	if(l == r)
	{
		tr[p].sum = num[l];
		tr[p].tag = base;
		return;
	}
	int mid = (l + r) >> 1;
	build(p << 1, l, mid);
	build(p << 1 | 1, mid + 1, r);
	tr[p].sum = tr[p << 1].sum + tr[p << 1 | 1].sum;
	tr[p].tag = base;
	return;
}

void segadd(int p, int l, int r, Matrix k)
{
	if(tr[p].l >= l && tr[p].r <= r)
	{
		tr[p].tag = tr[p].tag * k;
		tr[p].sum = tr[p].sum * k;
		return;
	}
	pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)segadd(p << 1, l, r, k);
	if(r > mid)segadd(p << 1 | 1, l, r, k);
	tr[p].sum = tr[p << 1].sum + tr[p << 1 | 1].sum;
	return;
}

Matrix segsum(int p, int l, int r)
{
	if(tr[p].l >= l && tr[p].r <= r)return tr[p].sum;
	pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	Matrix res(4, 4);
	if(l <= mid)res = res + segsum(p << 1, l, r);
	if(r > mid)res = res + segsum(p << 1 | 1, l, r);
	return res;
}

void init()
{
	base = Matrix(4, 4);
	for(int i = 1; i <= 4; i++)
		base.a[i][i] = 1;
	/*
	base1:| base2:| base3:
	1 0 0 | 1 0 0 | 1 0 1
	1 1 0 | 0 1 0 | 0 1 0
	0 0 1 | 0 1 1 | 0 0 1
	*/
	base1 = base;
	base1.a[2][1] = 1;
	base2 = base;
	base2.a[3][2] = 1;
	base3 = base;
	base3.a[1][3] = 1;
	/*
	base4:  | base5:  | base6:
	1 0 0 0 | 1 0 0 0 | 1 0 0 0
	0 1 0 0 | 0 v 0 0 | 0 1 0 0
	0 0 1 0 | 0 0 1 0 | 0 0 0 0
	v 0 0 1 | 0 0 0 1 | 0 0 v 1
	*/
	base4 = base;
	base5 = base;
	base6 = base;
	base6.a[3][3] = 0;
}
int main()
{
	init();
	int n;
	scanf("%d", &n);
	for(int i = 1; i <= n; i++)
	{
		num[i] = Matrix(1, 4);
		scanf("%lld%lld%lld", &num[i].a[1][1], &num[i].a[1][2], &num[i].a[1][3]);
		num[i].a[1][4] = 1;
	}
	build(1, 1, n);
	int m;
	scanf("%d", &m);
	while(m--)
	{
		int op, l, r, v;
		scanf("%d%d%d", &op, &l, &r);
		if(op == 1)
		{
			segadd(1, l, r, base1);
		}
		else if(op == 2)
		{
			segadd(1, l, r, base2);
		}
		else if(op == 3)
		{
			segadd(1, l, r, base3);
		}
		else if(op == 4)
		{
			scanf("%d", &v);
			base4.a[4][1] = v;
			segadd(1, l, r, base4);
		}
		else if(op == 5)
		{
			scanf("%d", &v);
			base5.a[2][2] = v;
			segadd(1, l, r, base5);
		}
		else if(op == 6)
		{
			scanf("%d", &v);
			base6.a[4][3] = v;
			segadd(1, l, r, base6);
		}
		else if(op == 7)
		{
			Matrix res = segsum(1, l, r);
			printf("%lld %lld %lld\n", res.a[1][1], res.a[1][2], res.a[1][3]);
		}
	}
	return 0;
}
```

