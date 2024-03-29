---
title: P3050 [USACO12MAR] Large Banner 题解
date: 2022-08-16
tags:
	- 题解
	- 数学（题目）
	- 莫比乌斯反演
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->

<div id="problem-card-vis">true</div>
<div id="problem-info-name">Large Banner</div>
<div id="problem-info-from">USACO 2012 March Gold</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3050">Luogu P3050</a></li><li><a href="https://darkbzoj.cc/problem/2619">BZOJ #2619</a></li></ul></div>

----

机房大佬 [$\text{Y}{\color{Red}\text{ouwike}}$](https://www.luogu.com.cn/user/261773) 有一个 $O(n \log n)$ 的解法，但是祂不屑于写题解，就让我来帮他写了。

----

首先我们考虑怎样才能让一个线段除了端点之外不经过整点。
这里我们不考虑位置，只考虑其形状。那就可以让线段的一个点在 $(0,0)$，另一个点在 $(i,j)$。
可以发现，当 $i$ 与 $j$ 互质，即 $\gcd(i,j) = 1$ 的时候，该线段才不会经过整点。
那么这样的线段的个数如下：

$$
\sum_{i=1}^n \sum_{j=1}^m [\gcd(i,j)=1]
$$

注意到我们线段的端点不一定需要在原点上，并且线段的方向也有两种，那么完整的式子如下：

$$
2 \sum_{i=1}^n \sum_{j=1}^m [\gcd(i,j)==1] (n-i+1)(m-j+1)
$$

考虑莫比乌斯反演：

$$
2 \sum_{i=1}^n \sum_{j=1}^m \sum_{p|i,p|j} \mu(p) (n-i+1)(m-j+1)
$$

现在我们可以开始考虑线段长度的限制了。
这个限制我们直接加在 $j$ 上，让式子变成这个样子：

$$
2 \sum_{i=1}^n \sum_{j=\lceil \sqrt{l^2-i^2} \rceil}^{\lfloor \sqrt{h^2-i^2} \rfloor} \sum_{p|i,p|j} \mu(p) (n-i+1)(m-j+1)
$$

将带有 $p$ 的求和号拿到带有 $j$ 的求和号的前面，并将枚举 $j$ 改为枚举 $\frac{j}{p}$：

$$
2 \sum_{i=1}^n \sum_{p|i} \mu(p) \sum_{j=\lceil \frac{\sqrt{l^2-i^2}}{p} \rceil}^{\lfloor \frac{\sqrt{h^2-i^2}}{p} \rfloor} (n-i+1)(m-jp+1)
$$

将最后的多项式拆开：

$$
2 \sum_{i=1}^n \sum_{p|i} \mu(p) \sum_{j=\lceil \frac{\sqrt{l^2-i^2}}{p} \rceil}^{\lfloor \frac{\sqrt{h^2-i^2}}{p} \rfloor} (n-i+1)(m+1)-(n-i+1)jp
$$

将其拆成两个式子。
第一个式子如下：

$$
2 \sum_{i=1}^n \sum_{p|i} \mu(p) \sum_{j=\lceil \frac{\sqrt{l^2-i^2}}{p} \rceil}^{\lfloor \frac{\sqrt{h^2-i^2}}{p} \rfloor} (n-i+1)(m+1)
$$

发现带有 $j$ 的求和号的后面那一部分与 $j$ 无关，将其提出来：

$$
\begin{align}
& 2 \sum_{i=1}^n \sum_{p|i} \mu(p) (n-i+1)(m+1) \sum_{j=\lceil \frac{\sqrt{l^2-i^2}}{p} \rceil}^{\lfloor \frac{\sqrt{h^2-i^2}}{p} \rfloor} 1  \\\\
=& 2 \sum_{i=1}^n \sum_{p|i} \mu(p) (n-i+1)(m+1)(\lfloor \frac{\sqrt{h^2-i^2}}{p} \rfloor - \lceil \frac{\sqrt{l^2-i^2}}{p} \rceil+1)
\end{align}
$$

第二个式子如下：

$$
\begin{align}
& 2 \sum_{i=1}^n \sum_{p|i} \mu(p) \sum_{j=\lceil \frac{\sqrt{l^2-i^2}}{p} \rceil}^{\lfloor \frac{\sqrt{h^2-i^2}}{p} \rfloor} (n-i+1)jp \\\\
=& 2 \sum_{i=1}^n \sum_{p|i} \mu(p) p (n-i+1) \sum_{j=\lceil \frac{\sqrt{l^2-i^2}}{p} \rceil}^{\lfloor \frac{\sqrt{h^2-i^2}}{p} \rfloor} j
\end{align}
$$

而最后一个求和号可以用等差数列求和公式来做。

同时注意一个边界条件：当线段长度为 $1$ 的时候，相邻两个点之间连的边也是可以的，需要特判来进行修正。

----

我们在预处理莫比乌斯函数的时候就是 $O(n \log n)$ 的时间复杂度，而枚举的时候只枚举了一个 $i$ 和 $i$ 的所有约数，枚举约数的时间复杂度是单个 $O(\log n)$ 的，总体来说也是 $O(n \log n)$。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1e6 + 10;
int mu[N], prime[N], vis[N];
vector<int> d[N];
int n, m, mod;
int l, h;
void moebius(int n)
{
	mu[1] = 1;
	for(int i = 2; i <= n; ++i)
	{
		if(!vis[i]) prime[++prime[0]] = i, mu[i] = mod - 1;
		for(int j = 1; j <= prime[0] && i * prime[j] <= n; ++j)
		{
			vis[i * prime[j]] = 1;
			if(i % prime[j] == 0)
			{
				mu[i * prime[j]] = 0;
				break;
			}
			mu[i * prime[j]] = mod - mu[i];
		}
	}
}
int sum(int n)
{
	return ((1ll * (1 + n) * n) >> 1) % mod;
}
int solve()
{
	int ans = 0;
	for(int i = 1; i <= n; ++i)
	{
		int lim = min(m, (int)sqrt(1ll * h * h - 1ll * i * i));
		int lim2 = max(1, (int)ceil(sqrt(1ll * l * l - 1ll * i * i)));
		for(auto p : d[i])
		{
			if(lim / p >= (lim2 - 1) / p + 1)
				ans = (ans + 1ll * (n - i + 1) * (m + 1) % mod * mu[p] % mod * ((lim / p) - ((lim2 - 1) / p + 1) + 1) % mod) % mod;
		}
	}
	for(int i = 1; i <= n; ++i)
	{
		int lim = min(m, (int)sqrt(1ll * h * h - 1ll * i * i));
		int lim2 = max(1, (int)ceil(sqrt(1ll * l * l - 1ll * i * i)));
		for(auto p : d[i])
		{
			if(lim / p >= (lim2 - 1) / p + 1)
				ans = (ans + mod - 1ll * (n - i + 1) * p % mod * mu[p] % mod * (sum(lim / p) + mod - sum(((lim2 - 1) / p + 1) - 1)) % mod) % mod;
		}
	}
	return (ans + ans) % mod;
}

int main()
{
	scanf("%d%d", &n, &m);
	scanf("%d%d", &l, &h);
	scanf("%d", &mod);
	moebius(n);
	for(int i = 1; i <= n; ++i)
		for(int j = i; j <= n; j += i)
			d[j].push_back(i);

	int ans = solve();
	if(l == 1)
		ans = (ans + (1ll * n * (m + 1) % mod + 1ll * (n + 1) * m % mod) % mod) % mod;

	printf("%d\n", ans);
	return 0;
}
```


