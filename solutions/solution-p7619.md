---
title: P7619 [COCI2011-2012#2] RASPORED 题解
date: 2022-05-23
tags:
	- 题解
	- 贪心
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">RASPORED</div>
<div id="problem-info-from">COCI 2011-2012 #2</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4643">Luogu P4643</a></li><li><a href="https://darkbzoj.cc/problem/3179">BZOJ #3179</a></li></ul></div>

----

我们首先考虑没有修改的情况。

首先，我们假设已经给所有的居民分配好了一个烘焙披萨的方案，并用 $pos_i$ 代表第 $i$ 位居民的披萨是第 $pos_i$ 个烘焙的。

这样的话，我们设 $C_i$ 为从第 $i$ 名居民身上获取的小费数额（如果为负数则表示需要向该位居民支付的数额的相反数），那么就会有如下的式子：

$$
C_i = L_i - \sum_{j=1}^{pos_i} T_j
$$

那么对于 Mirko 收获小费的总和 $\sum_{i=1}^n C_i$，我们可以推一下式子：

$$
\begin{aligned}
\sum_{i=1}^n C_i &= \sum_{i=1}^n (L_i - \sum_{j=1}^{pos_i} T_j) \\\\
&= \sum_{i=1}^n L_i - \sum_{i=1}^n \sum_{j=1}^{pos_i} T_j \\\\
&= \sum_{i=1}^n L_i - \sum_{i=1}^n T_i \times (n- pos_i + 1)
\end{aligned}
$$

最后一步是这样推出来的：

对于 $T_i$ 来说，它会在计算所有 $pos_j \geq pos_i$ 的 $j$ 的时候被计算到，所以总共就是 $\sum_{j=pos_i}^n T_i$，即 $T_i \times (n-pos_i + 1)$。

----

然后看带修改的。

我们看刚才推出来的式子，可以看出我们能够对于 $L$ 与 $T$ 分开考虑，而事实上我们也就是这样做的。

我们对于一开始没有被修改时的数据计算出一个 $suml = \sum_{i=1}^n L_i$，然后再计算出一个 $sumt = \sum_{i=1}^n T_i \times (n- pos_i + 1)$。  
我们最终输出的数值就是 $suml-sumt$。

假设我们有了一个新的修改，将 $(L_x,T_x)$ 修改为 $(L_y,T_y)$。

对于 $suml$，其只需要变为 $suml-L_x+L_y$ 即可。

对于 $sumt$，我们可以看成首先从数列里面删除了一个 $T_x$，然后再插入了一个 $T_y$；其中 $T_x$ 删除前的位置是 $pos_x$，$T_y$ 删除后的位置是 $pos_y$。

我们可以将删除和插入分开讨论，也可以只讨论改变位置的元素。

如果分开讨论删除和插入的话，我们的分析过程是这个样子的：

1. 对于 $\forall T_i < T_x$，我们的 $pos_i$ 不会变，但是 $n$ 会因删除而减小 $1$；而对于 $\forall T_i \geq T_x$，我们的 $pos_i$ 和 $n$ 都会减小 $1$ 而最终抵消；对于 $T_x$，我们需要减去它的贡献。

所以我们的 $sumt$ 在删除 $T_x$ 之后会变成这个样子：
$$
sumt \to sumt - T_x \times (n - pos_x + 1) - \sum_{T_i < T_x} T_i
$$

2. 对于 $\forall T_i < T_x$，我们的 $pos_i$ 不会变，但是 $n$ 会因插入而增大 $1$；而对于 $\forall T_i \geq T_x$，我们的 $pos_i$ 和 $n$ 都会增大 $1$ 而最终抵消；对于 $T_y$，我们需要加上它的贡献。

所以我们的 $sumt$ 在插入 $T_y$ 之后会变成这个样子：
$$
sumt \to sumt + T_x \times (n - pos_y + 1) + \sum_{T_i < T_x} T_i
$$

如果只讨论改变位置的元素的话，我们的分析过程是这个样子的：

1. 如果 $T_x < T_y$，那么对于 $\{T_i | pos_i \in (pos_x,pos_y)\}$，其 $pos_i$ 会减少 $1$，从而导致 $sumt$ 减少 $\sum_{pos_i \in (pos_x,pos_y)} T_i$。

1. 如果 $T_x > T_y$，那么对于 $\{T_i | pos_i \in (pos_y,pos_x)\}$，其 $pos_i$ 会增加 $1$，从而导致 $sumt$ 增加 $\sum_{pos_i \in (pos_x,pos_y)} T_i$。

总的来看，我们的变化量可以看做 $\sum_{T_i < T_x} T_i - \sum_{T_i < T_x} T_i$。

再加上 $T_y$ 的贡献，减去$T_x$ 的贡献，我们推出的式子跟上面的是一样的。

----

于是我们就需要一种数据结构，支持

1. 插入和删除元素
2. 查询小于一个元素的数字个数
3. 查询小于一个元素的数字之和

树状数组、权值线段树和平衡树均可。

我这里使用的是替罪羊树。

对于不知道替罪羊树的人，我在这里安利一下[我的博客](/OI/scapegoat-tree)，同时也给出[OI-Wiki](https://oi-wiki.org/ds/sgt/)关于替罪羊树的讲解。

替罪羊树虽然比较慢，但是所有的操作时间复杂度均摊之后都是 $O(\log n
)$ 级别的。

这里粘一下代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 200010;
ll n, m;
double alpha = 0.75;
struct Scapegoat
{
	int ls, rs;
	ll w, wn;
	int s, sz, sd;
	ll sum;
}tr[N];
int cnt, rt;
void calc(int p)
{
	tr[p].s = tr[tr[p].ls].s + tr[tr[p].rs].s + 1;
	tr[p].sz = tr[tr[p].ls].sz + tr[tr[p].rs].sz + tr[p].wn;
	tr[p].sd = tr[tr[p].ls].sd + tr[tr[p].rs].sd + (tr[p].wn != 0);
	tr[p].sum = tr[tr[p].ls].sum + tr[tr[p].rs].sum + tr[p].wn * tr[p].w;
}
bool canrbu(int p)
{
	return tr[p].wn && (alpha * tr[p].s <= ( double )max(tr[tr[p].ls].s, tr[tr[p].rs].s) ||
	( double )tr[p].sd <= alpha * tr[p].s);
}//can rebuild
int ldr[N];
void rbuunf(int &ldc, int p)
{
	if(!p)return;
	rbuunf(ldc, tr[p].ls);
	if(tr[p].wn)ldr[ldc++] = p;
	rbuunf(ldc, tr[p].rs);
}//rebuild-unfold
int rbubld(int l, int r)
{
	if(l >= r)return 0;
	int mid = (l + r) >> 1;
	tr[ldr[mid]].ls = rbubld(l, mid);
	tr[ldr[mid]].rs = rbubld(mid + 1, r);
	calc(ldr[mid]);
	return ldr[mid];
}//rebuild-build
void rbuild(int &p)
{
	int ldc = 0;
	rbuunf(ldc, p);
	p = rbubld(0, ldc);
}//rebuild
void insert(int &p, ll k)
{
	if(!p)
	{
		p = ++cnt;
		if(!rt)rt = 1;
		tr[p].w = k;
		tr[p].ls = tr[p].rs = 0;
		tr[p].wn = tr[p].s = tr[p].sz = tr[p].sd = 1;
		tr[p].sum = k;
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
void loschn(int &p, ll k)
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
ll uprgtr(int p, ll k)
{
	if(!p)
		return 0;
	else if(tr[p].w == k && tr[p].wn)
		return tr[tr[p].ls].sz;
	else if(tr[p].w < k)
		return tr[tr[p].ls].sz + tr[p].wn + uprgtr(tr[p].rs, k);
	else
		return uprgtr(tr[p].ls, k);
}//相当于是使用greater<>函数排序之后的upper_bound
//输出的结果是小于某个元素的数的个数
ll uprsum(int p, ll k)
{
	if(!p)
		return 0;
	else if(tr[p].w == k && tr[p].wn)
		return tr[tr[p].ls].sum;
	else if(tr[p].w < k)
		return tr[tr[p].ls].sum + tr[p].wn * tr[p].w + uprsum(tr[p].rs, k);
	else
		return uprsum(tr[p].ls, k);
}//跟上面差不多，输出的是小于某个元素的数之和
ll suml, sumt;
ll l[N], t[N];
int main()
{
	scanf("%lld%lld", &n, &m);
	int temp[N];
	for(int i = 1; i <= n; i++)
	{
		scanf("%lld%lld", &l[i], &t[i]);
		suml += l[i];
		insert(rt, t[i]);
		temp[i] = t[i];
	}
	sort(temp + 1, temp + 1 + n);
	for(int i = 1; i <= n; i++)
		sumt += (n - i + 1) * temp[i];
	printf("%lld\n", suml - sumt);
	while(m--)
	{
		int a;
		ll b, c;
		scanf("%d%lld%lld", &a, &b, &c);
		suml -= l[a] - b;
		l[a] = b;
		ll sum1 = uprsum(rt, t[a]), cnt1 = uprgtr(rt, t[a]);
		loschn(rt, t[a]);
		insert(rt, c);
		ll sum2 = uprsum(rt, c), cnt2 = uprgtr(rt, c);
		sumt += c * (n - cnt2) + sum2 - t[a] * (n - cnt1) - sum1;
		t[a] = c;
		printf("%lld\n", suml - sumt);
	}
	return 0;
}
```

