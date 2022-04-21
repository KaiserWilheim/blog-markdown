---
title: 拉格朗日插值
zh-CN: true
date: 2022-03-17
tags:
	- 数学
categories:
	数学
mathjax: true
---

拉格朗日插值。

<!--more-->

在学完函数以后，AJ想让我们求出一个函数，使得其带入自然数时生成的序列时给定的，并让武嘉给出了一个数列。

崇尚<ruby>野獣<rt>やじゅう</rt></ruby><ruby>先輩<rt>せんぱい</rt></ruby>的武嘉给出了这样的一个序列：

$$
1,3,5,7,9,114514
$$

光看前五项貌似可以构建出 $f(x)=2x-1$ 来糊过去，但是第6项就不好办了。

王哥说：“这都不会，快用**拉格朗日插值**！”
(P.S.: 其实我们班里初三的时候真的有讲拉插)

----

引入完毕。

拉格朗日插值法可以很快地将一系列的点值转化为一个经过所有给定点值的函数（或者称多项式）。

拉格朗日给出的方法是这样的：

首先我们给出两个二次多项式，分别称之为 $f(x)$ 与 $g(x)$。
两者的系数分别为 $a_f,b_f,c_f$ 与 $a_g,b_g,c_g$。

假设有一个函数 $h(x)$ ，它等于 $f(x)+g(x)$，那么

$$
\begin{align}
h(x) &= f(x)+g(x) \\\\
&= a_fx^2+b_fx+c_f + a_gx^2+b_gx+c_g \\\\
&= (a_f+a_g)x^2 + (b_f+b_g)x + (c_f+c_g)
\end{align}
$$

$h(x)$ 的每一项系数竟然是 $f(x)$ 与 $g(x)$ 该项系数之和。

我们再来看代入数值之后的结果。

我们不难看出，对于每一个 $x$ ，其对应的函数值 $f(x)$，$g(x)$ 与 $h(x)$ 满足 $h(x)=f(x)+g(x)$。

然后拉格朗日就想，对于每一个点值，我们分别构造一个函数，使得其刚好经过当前点值代表的点，而其他点值处均为 $0$。
我们最后将所有的函数加起来，就得到了完美经过每个点值的函数了。

而每一个函数又怎么求呢？

我们先想个简单一点的，让这个函数在当前 $x$ 处为 $1$，其他 $x$ 处都为 $0$。

首先，我们为了表示这些点值且与自由元 $x$ 区分，设其分别为 $(X_i,Y_i)$，总共有 $n$ 个，当前函数为 $F_k(x)$，代表着第 $k$ 个点值。

那么，我们让函数为 $\displaystyle \prod_{i=1,i \neq k}^n (x - X_i)$，这样可以使除当前点之外的数都是 $0$。

然后，我们将整个函数除以代入 $X_k$ 时的值，使 $F_k(X_k)=1$。
那么整个函数就变为了 $\displaystyle \prod_{i=1,i \neq k}^n \frac{x - X_i}{X_k - X_i}$。

最后再乘上 $Y_k$，就可以得出我们当前的函数：$\displaystyle F_k(x) = Y_k \prod_{i=1,i \neq k}^n \frac{x - X_i}{X_k - X_i}$

再把所有的东西加起来，得到

$$
G(x) = \sum_{k=1}^n \bigg( Y_k \prod_{i=1,i \neq k}^n \frac{x - X_i}{X_k -X_i} \bigg)
$$

----

或者说，我们对于每一个点值列一个方程，将所有的系数作为未知量来求解。

对于一个 $n$ 次多项式，我们一共有 $n+1$ 个系数。我们设这些系数分别为 $a_0,a_1,a_2,a_3,\cdots ,a_n$。

我们如果有一个点值 $(x,y)$，那么我们就可以根据其列出一个方程：

$$
a_0 + a_1 x + a_2 x^2 + a_3 x^3 + \cdots + a_n x^n = y
$$

首先，众所周知，我们需要不少于 $n$ 个方程才能求出 $n$ 个未知数。
其次，我们利用克拉默法则来接方程式的时候，也需要形成行列式才能解出来值。

所以说，我们给定 $n+1$ 个点值，能唯一地确定一个 $n$ 次多项式。

----

所以，我们如果用拉格朗日插值法来计算AJ的课后作业的话，得到的式子就是这个样子的：

$$
\begin{align}
G(x) &= 1·\frac{(x-2)(x-3)(x-4)(x-5)(x-6)}{(1-2)(1-3)(1-4)(1-5)(1-6)} 
\\\\ &+ 3·\frac{(x-1)(x-3)(x-4)(x-5)(x-6)}{(2-1)(2-3)(2-4)(2-5)(2-6)} 
\\\\ &+ 5·\frac{(x-1)(x-2)(x-4)(x-5)(x-6)}{(3-1)(3-2)(3-4)(3-5)(3-6)} 
\\\\ &+ 7·\frac{(x-1)(x-2)(x-3)(x-5)(x-6)}{(4-1)(4-2)(4-3)(4-5)(4-6)} 
\\\\ &+ 9·\frac{(x-1)(x-2)(x-3)(x-4)(x-6)}{(5-1)(5-2)(5-3)(5-4)(5-6)} 
\\\\ &+ 114514·\frac{(x-1)(x-2)(x-3)(x-4)(x-5)}{(6-1)(6-2)(6-3)(6-4)(6-5)} \\\\
&= \frac{114503}{120} x^5 - \frac{114503}{8} x^4 + \frac{1946551}{24} x^3 - \frac{1717545}{8} x^2 + \frac{15687031}{60} x - 114504
\end{align}
$$

这里有一个 $O(n^2)$ 求出最后得出的多项式的系数的代码：

``` cpp
#include<bits/stdc++.h>
#define ll long long 
const int N = 5050;
const ll mod = 998244353;
using namespace std;
ll qpow(ll a, ll t = mod - 2)
{
	ll ans = 1;
	while(t)
	{
		if(t & 1)ans = ans * a % mod;
		a = a * a % mod;
		t >>= 1;
	}return ans;
}
int n;
ll x[N], y[N], c[N], fs[N], g[N], f[N];
int main()
{
	scanf("%d", &n);
	for(int i = 0; i < n; i++)
		scanf("%lld%lld", &x[i], &y[i]);
	for(int i = 0; i < n; i++)
	{
		c[i] = 1;
		for(int j = 0; j < n; j++)
			if(i != j)
				c[i] = c[i] * (x[i] - x[j]) % mod;
		c[i] = qpow(c[i]) * y[i] % mod;
	}
	fs[0] = 1;
	for(int i = 0; i < n; i++)
	{
		for(int j = n; j; j--)
			fs[j] = (fs[j - 1] + fs[j] * (mod - x[i])) % mod;
		fs[0] = fs[0] * (mod - x[i]) % mod;
	}
	for(int i = 0; i < n; i++)
	{
		ll buf = qpow(mod - x[i]);
		g[0] = fs[0] * buf % mod;
		for(int j = 1; j < n; j++)
			g[j] = (fs[j] - g[j - 1]) * buf % mod;
		for(int j = 0; j < n; j++)
			f[j] = (f[j] + c[i] * g[j]) % mod;
	}
	for(int i = 0; i < n; i++)
		printf("%lld ", (f[i] + mod) % mod);
	return 0;
}
```

[$\blacktriangleright$](https://www.luogu.org/blog/command-block/zong-la-cha-dao-kuai-su-cha-zhi-qiu-zhi)




























































