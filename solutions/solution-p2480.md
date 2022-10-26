---
title: P2480 [SDOI2010] 古代猪文 题解
date: 2022-10-09
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
<div id="problem-info-name">古代猪文</div>
<div id="problem-info-from">SDOI 2010</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2480">Luogu P2480</a></li><li><a href="https://loj.ac/p/10229">LibreOJ L10229</a></li><li><a href="https://www.acwing.com/problem/content/215/">AcWing 213</a></li><li><a href="https://darkbzoj.cc/problem/1951">BZOJ #1951</a></li></ul></div>

----

题目要求我们求出 $g^{\sum_{d | n}C_n^d} \bmod{999911659}$ 的值。

因为 $999911659$ 是一个质数，我们可以根据费马小定理，将式子化简为 $g^{\sum_{d | n}C_n^d \bmod{999911658}} \bmod{999911659}$。

显然 $999911658$ 不是一个质数。将其分解，可得 $999911658 = 2 \times 3 \times 4679 \times 35617$。

卢卡斯定理解决不了模数非质数的情况，那么我们就需要把四个质因数都作为模数过一遍卢卡斯定理解决 $C_n^d$ 的值，算出 $\sum_{d | n}C_n^d$ 的值之后用中国剩余定理合并四个式子即可。
假设我们四次计算得到的结果分别是 $a_1$、$a_2$、$a_3$、$a_4$，那我们就需要解如下的同余方程组：

$$
\begin{cases}
x \equiv a_1 \pmod{2} \\\\
x \equiv a_2 \pmod{3} \\\\
x \equiv a_3 \pmod{4679} \\\\
x \equiv a_4 \pmod{35617}
\end{cases}
$$

得到的 $x$ 就是一个可以接受的数据范围了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define int long long
const int N = 100010;
const int mod = 999911658;
int kpd(int a, int b)
{
	return b ? kpd(b, a % b) : a;
}
int fac[N], inv[N];
int a[5], b[5] = { 0,2,3,4679,35617 };
int qpow(int a, int x, int p)
{
	int res = 1;
	while(x)
	{
		if(x & 1)res = (res * a) % p;
		a = (a * a) % p;
		x >>= 1;
	}
	return res;
}
void init(int p)
{
	fac[0] = 1;
	for(int i = 1; i < p; i++)
		fac[i] = fac[i - 1] * i % p;
	inv[p - 1] = qpow(fac[p - 1], p - 2, p);
	for(int i = p - 1; i > 1; i--)
		inv[i - 1] = inv[i] * i % p;
	inv[0] = 1;
}
int C(int n, int m, int p)
{
	if(n < m)return 0;
	return fac[n] * inv[m] % p * inv[n - m] % p;
}
int lucas(int n, int m, int p)
{
	if(n < m)return 0;
	if(!n)return 1;
	return lucas(n / p, m / p, p) * C(n % p, m % p, p) % p;
}
signed main()
{
	int n, g;
	scanf("%lld%lld", &n, &g);
	if(g % (mod + 1) == 0)
	{
		puts("0");
		return 0;
	}
	for(int k = 1; k <= 4; k++)
	{
		init(b[k]);
		for(int i = 1; i * i <= n; i++)
		{
			if(n % i == 0)
			{
				a[k] = (a[k] + lucas(n, i, b[k])) % b[k];
				if(i * i != n)
					a[k] = (a[k] + lucas(n, n / i, b[k])) % b[k];
			}
		}
	}
	int ans = 0;
	for(int i = 1; i <= 4; i++)
		ans = (ans + a[i] * (mod / b[i]) % mod * qpow(mod / b[i], b[i] - 2, b[i])) % mod;
	printf("%lld\n", qpow(g, ans, mod + 1));
	return 0;
}
```

