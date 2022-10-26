---
title: P1999 高维正方体 题解
date: 2022-08-21
tags:
	- 题解
	- 数学（题目）
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">高维正方体</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P1999">Luogu P1999</a></li></ul></div>

----

一道考试题，写一下当时的思路。

首先我们进行一个列表：

|   | 0 | 1 | 2 | 3 |
|:-:|:-:|:-:|:-:|:-:|
| 0 | 1 | - | - | - |
| 1 | 2 | 1 | - | - |
| 2 | 4 | 4 | 1 | - |
| 3 | 8 | 12 | 6 | 1 |

首先我们可以发现一个规律，就是 $n$ 位立方体所拥有的的点的个数是 $2^n$，所拥有的 $n-1$ 维立方体的个数是 $2n$。

然后就没有发现规律。

于是我就尝试再举出一个例子，看看能不能找出规律。

![](https://img.catium.top:10000/upload/757281.png)

这是一个超立方体的展开图。

数了一下，发现这个玩意有8个正方体，24个面，32个边，16个顶点。

现在我们可以扩充一下这个表了：

|   | 0 | 1 | 2 | 3 | 4 |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 0 | 1 | - | - | - | - |
| 1 | 2 | 1 | - | - | - |
| 2 | 4 | 4 | 1 | - | - |
| 3 | 8 | 12 | 6 | 1 | - |
| 4 | 16 | 32 | 24 | 8 | 1 |

貌似还是没有什么规律。

尝试搞出来超立方体的面数与正方体的面数的关系。
发现其每一个面都会由两个正方体共享，于是就是 $\frac{6 \times 8}{2}$。
由此往后面推，每一个边都会由三个正方体共享，于是就是 $\frac{12 \times 8}{3} $。而正方体中的每一条边都会由两个面共享，所以12就等于 $\frac{4 \times 6}{2}$。

定义一个 $f_{n,m}$，代表 $n$ 维立方体拥有的 $m$ 维正方体数量。

于是 $f_{4,1} = f_{3,1} \times \frac{2 \times 4}{3} = f_{2,1} \times \frac{2 \times 3}{2} \times \frac{2 \times 4}{3}$。
而 $f_{n,n-1} = 2n$。

于是我们就得到了一个关于 $f_{n,m}$ 的公式：

$$
f_{n,m} = \frac{n!}{(n-m)!m!} \times 2^{n-m-1} \times 2(m+1)
$$

于是我们像预处理组合数一样预处理出来阶乘和阶乘的逆元就可以了。

参考代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const ll mod = 1e9 + 7;
ll fac[100010], inv[100010];
ll qpow(ll a, ll x)
{
	ll res = 1;
	while(x)
	{
		if(x & 1)res = (res * a) % mod;
		a = (a * a) % mod;
		x >>= 1;
	}
	return res % mod;
}
ll f(int n, int m)
{
	if(m == n)
	{
		return 1ll;
	}
	else if(m == n - 1)
	{
		ll res = (2 * n) % mod;
		return res;
	}
	else if(m == 0)
	{
		return qpow(2, n);
	}
	else
	{
		ll res = (fac[n] * inv[m + 1]) % mod * inv[n - m] % mod;
		res = (res * qpow(2, n - m - 1)) % mod;
		res = (res * f(m + 1, m)) % mod;
		return res;
	}
}
int main()
{
	fac[0] = 1;
	for(ll i = 1; i <= 100000; i++)
	{
		fac[i] = fac[i - 1] * i % mod;
	}
	inv[0] = 1; inv[100000] = qpow(fac[100000], mod - 2);
	for(int i = 99999; i >= 1; i--)
	{
		inv[i] = 1 * inv[i + 1] * (i + 1) % mod;
	}
	int n, m;
	scanf("%d%d", &n, &m);
	if(n < m)puts("0");
	else printf("%lld\n", f(n, m));
	return 0;
}
```

