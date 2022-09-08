---
title: P3707 [SDOI2017] 相关分析 题解
date: 2022-07-20
tags:
	- 题解
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">相关分析</div>
<div id="problem-info-from">SDOI 2017</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3707">Luogu P3707</a></li><li><a href="https://loj.ac/p/2005">LibreOJ L2005</a></li><li><a href="https://www.acwing.com/problem/content/2587/">AcWing 2585</a></li><li><a href="https://darkbzoj.cc/problem/4821">BZOJ #4821</a></li></ul></div>

----

这道题需要我们维护一大坨东西。

# 准备与计算答案

首先我们把线性回归方程的公式拆一下：
（这里将 $\sum_{i=l}^r$ 省略为 $\sum$）

$$
\begin{align}
\hat{a} &= \frac{\sum (x_i - \bar{x})(y_i - \bar{y})}{\sum (x_i - \bar{x}^2)} \\\\
&= \frac{\sum (x_i y_i - y_i \bar{x} - x_i \bar{y} + \bar{x} \bar{y})}{\sum (x_i^2 - 2x_i \bar{x} + \bar{x}^2)} \\\\
&= \frac{\sum x_i y_i - \bar{x} \sum y_i - \bar{y} \sum x_i + (r-l+1) \bar{x} \bar{y}}{\sum x_i^2 - 2\bar{x} \sum x_i + (r-l+1)\bar{x}^2} \\\\
&= \frac{\sum x_i y_i - \frac{\sum x_i \sum y_i}{r-l+1} - \frac{\sum y_i \sum x_i}{r-l+1} + (r-l+1) \frac{\sum x_i \sum y_i}{(r-l+1)^2}}{\sum x_i^2 - 2\frac{\sum x_i \sum x_i}{(r-l+1)} + (r-l+1) (\frac{\sum x_i}{r-l+1})^2} \\\\
&= \frac{\sum x_i y_i - \frac{\sum x_i \sum y_i}{r-l+1}}{\sum x_i^2 - \frac{\sum x_i \sum x_i}{r-l+1}}
\end{align}
$$

最终我们需要维护的就是 $\sum x_i$，$\sum x_i^2$，$\sum y_i$ 和 $\sum x_i y_i$。
我们计算时的公式就是 $\hat{a} = \frac{\sum x_i y_i - \frac{\sum x_i \sum y_i}{r-l+1}}{\sum x_i^2 - \frac{(\sum x_i)^2}{r-l+1}}$。

# 区间加

我们考虑每一个量是怎么变化的：

$\sum (x_i+s)^2 = \sum (x_i^2 + 2sx_i + s^2) = \sum x_i^2 + 2s\sum x_i + (r-l+1)s^2$
$\sum (x_i+s)(y_i+t) = \sum (x_iy_i + sy_i + ts_i + st) = \sum x_iy_i + s\sum y_i + t\sum x_i + (r-l+1)st$
$\sum (x_i+s) = \sum x_i + (r-l+1)s$
$\sum (y_i+t) = \sum y_i + (r-l+1)t$

注意我们这里在维护前面两个二次的东西的时候需要我们一次的这两个量，所以我们需要注意维护的顺序。

# 区间修改

## 前置芝士

$\sum_{i=1}^n i^2 = \frac{n(n+1)(2n+1)}{6}$

## 转化

我们考虑将每一个点的值拆分成 $(i,i)$ 和 $(s,t)$ 两部分，而后者可以用区间加来完成。

这个 $i$ 就是当前点的编号。一开始我想的是每一个修改区间的 $i$ 从 $1$ 开始，傻乎乎地传了一个 $i$ 下去，然后wa了一堆……

对于一个完全被修改的区间 $[l,r]$，我们的四个量的新值如下：

$\sum x_i^2 = \sum i^2 = \sum_{i=1}^r i^2 - \sum_{i=1}^{l-1} = \frac{r(r+1)(2r+1)}{6} - \frac{(l-1)l(2l-1)}{6}$
$\sum x_iy_i = \sum i^2 = \frac{r(r+1)(2r+1)}{6} - \frac{(l-1)l(2l-1)}{6}$
$\sum x_i = \sum i = \frac{(r-l+1)(l+r)}{2}$
$\sum y_i = \sum i = \frac{(r-l+1)(l+r)}{2}$

然后需要清空懒标记，并打一个清空懒标记的标记。
然后就是区间加了。

# 注意事项

注意懒标记的维护顺序。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;
int n, m;
double x[N], y[N];
double calc(double x)
{
	return x * (x + 1) * (2 * x + 1) / 6;
}
struct Ret
{
	double xx, xy, x, y;
	friend Ret operator + (const Ret &a, const Ret &b)
	{
		return { a.xx + b.xx,a.xy + b.xy,a.x + b.x,a.y + b.y };
	}
};
struct SegTree
{
	int l, r;
	double xx, xy, x, y;
	double s, t;
	bool c;
}tr[N << 3];
void modadd(int p, double s, double t)
{
	double len = double(tr[p].r - tr[p].l + 1);
	tr[p].xx += s * s * len + 2 * s * tr[p].x;
	tr[p].xy += s * t * len + s * tr[p].y + t * tr[p].x;
	tr[p].x += s * len;
	tr[p].y += t * len;
	tr[p].s += s;
	tr[p].t += t;
}
void modchg(int p)
{
	double l = double(tr[p].l), r = double(tr[p].r);
	tr[p].xx = tr[p].xy = calc(r) - calc(l - 1);
	tr[p].x = tr[p].y = (r - l + 1) * (l + r) / 2;
	tr[p].c = true;
	tr[p].s = tr[p].t = 0;
}
void pushup(int p)
{
	tr[p].xx = tr[p << 1].xx + tr[p << 1 | 1].xx;
	tr[p].x = tr[p << 1].x + tr[p << 1 | 1].x;
	tr[p].y = tr[p << 1].y + tr[p << 1 | 1].y;
	tr[p].xy = tr[p << 1].xy + tr[p << 1 | 1].xy;
}
void pushdown(int p)
{
	if(tr[p].c)
	{
		modchg(p << 1), modchg(p << 1 | 1);
	}
	modadd(p << 1, tr[p].s, tr[p].t);
	modadd(p << 1 | 1, tr[p].s, tr[p].t);
	tr[p].c = false;
	tr[p].s = tr[p].t = 0;
}
void build(int p, int l, int r)
{
	tr[p].l = l, tr[p].r = r;
	if(l == r)
	{
		tr[p].xx = x[l] * x[r];
		tr[p].x = x[l];
		tr[p].y = y[l];
		tr[p].xy = x[l] * y[r];
		tr[p].s = tr[p].t = 0;
		tr[p].c = false;
		return;
	}
	int mid = (l + r) >> 1;
	build(p << 1, l, mid);
	build(p << 1 | 1, mid + 1, r);
	pushup(p);
}
void segadd(int p, int l, int r, double s, double t)
{
	if(tr[p].l >= l && tr[p].r <= r)
	{
		modadd(p, s, t);
		return;
	}
	pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)segadd(p << 1, l, r, s, t);
	if(r > mid)segadd(p << 1 | 1, l, r, s, t);
	pushup(p);
}
void segchg(int p, int l, int r, double s, double t)
{
	if(tr[p].l >= l && tr[p].r <= r)
	{
		modchg(p);
		modadd(p, s, t);
		return;
	}
	pushdown(p);
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)segchg(p << 1, l, r, s, t);
	if(r > mid)segchg(p << 1 | 1, l, r, s, t);
	pushup(p);
}
Ret segsum(int p, int l, int r)
{
	if(tr[p].l >= l && tr[p].r <= r)return { tr[p].xx,tr[p].xy,tr[p].x,tr[p].y };
	pushdown(p);
	Ret res = { 0,0,0,0 };
	int mid = (tr[p].l + tr[p].r) >> 1;
	if(l <= mid)res = res + segsum(p << 1, l, r);
	if(r > mid)res = res + segsum(p << 1 | 1, l, r);
	return res;
}
double ans(double l, double r)
{
	Ret res = segsum(1, l, r);
	double len = r - l + 1;
	return (res.xy - res.x * res.y / len) / (res.xx - res.x * res.x / len);
}
int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
		scanf("%lf", &x[i]);
	for(int i = 1; i <= n; i++)
		scanf("%lf", &y[i]);
	build(1, 1, n);
	while(m--)
	{
		int op, l, r;
		double s, t;
		scanf("%d%d%d", &op, &l, &r);
		if(op == 1)
		{
			printf("%.6lf\n", ans(l, r));
		}
		else if(op == 2)
		{
			scanf("%lf%lf", &s, &t);
			segadd(1, l, r, s, t);
		}
		else if(op == 3)
		{
			scanf("%lf%lf", &s, &t);
			segchg(1, l, r, s, t);
		}
	}
	return 0;
}
```
