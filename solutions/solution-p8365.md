---
title: P8365 [LNOI2022] 吃 题解
date: 2022-07-07
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
<div id="problem-info-name">吃</div>
<div id="problem-info-from">LNOI 2022</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P8365">Luogu P8365</a></li><li><a href="https://loj.ac/p/3737">LibreOJ L3737</a></li></ul></div>

----

一道很有趣的贪心题目。

首先我们可以确定，我们一定是先做加法再做乘法，这样才可以使得数最大。

我们考虑将食物按照 $a_i$ 的大小分为两类：
- 一类是 $a_i = 1$ 的，这种食物只能利用它的 $b$，在一开始就全部加到初始体重上面，我们记这个处理好的值为 $B$。
- 另一类是 $a_i \geq 2$ 的。这种食物里面最多用一个 $b$。因为如果两个食物 $(a_i,b_i)$ 和 $(a_j,b_j)$，我们都用了其 $b$ 属性，且 $b_i \geq b_j$，那么还不如只用 $b_i$，然后用 $a_j$，这样的得数 $(B+b_i)a_j$ 比两者都用 $b$ 的得数 $B+b_i+b_j$ 更优。

设第二类食物的 $\prod a_i = A$，那么我们一个 $b$ 都不选的结果就是 $AB$。
如果我们选择一个 $b$，那么其价值就会变为 $\frac{B+b_i}{a_i}A$。
我们枚举找到这个最大的 $\frac{B+b_i}{a_i}$ 即可。
当然，如果我们枚举到的所有食物都会让这个 $\frac{B+b_i}{a_i} < B$，那我们还不如不选，此时 $b_i = 0,a_i = 1$。

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010;
const ll mod = 1e9 + 7;
int n;
ll a[N], b[N];
int main()
{
	scanf("%d", &n);
	ll ans = 1;
	a[0] = 1;
	for (int i = 1; i <= n; i++)
		scanf("%lld", &a[i]);
	for (int i = 1; i <= n; i++)
	{
		scanf("%lld", &b[i]);
		if (a[i] == 1)ans = (ans + b[i]) % mod;
	}
	int maxn = 0;
	for (int i = 1; i <= n; i++)
	{
		if (a[i] == 1)continue;
		if ((ans + b[i]) * a[maxn] > (ans + b[maxn]) * a[i])
			maxn = i;
	}
	ans = (ans + b[maxn]) % mod;
	for (int i = 1; i <= n; i++)
	{
		if (i == maxn)continue;
		ans = (ans * a[i]) % mod;
	}
	printf("%lld\n", ans);
	return 0;
}
```
