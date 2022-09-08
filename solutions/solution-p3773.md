---
title: P3773 [CTSC2017] 吉夫特 题解
date: 2022-08-22
tags:
	- 题解
	- 数学
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">吉夫特</div>
<div id="problem-info-from">CTSC 2017</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3773">Luogu P3773</a></li><li><a href="https://loj.ac/p/2264">LibreOJ L2264</a></li><li><a href="https://uoj.ac/problem/300">UOJ #300</a></li><li><a href="https://darkbzoj.cc/problem/4903">BZOJ #4903</a></li></ul></div>

----

题目要求我们从一个数列中找出满足 $\displaystyle \prod_{i=2}^k C_{a_{b_{i-1}}}^{a_{b_i}}$ 为奇数的子序列个数。

看起来很难算，但是我们可以根据卢卡斯定理化简，以此迅速判断是否满足条件。

卢卡斯定理是 $C_a^b \bmod{p} \equiv C_{a \div p}^{b \div p} \times C_{a \bmod{p}}^{b \bmod{p}} \bmod{p}$。

因为此处 $p=2$，所以我们可以将其分解成为一串组合数，其中只包含 $C_0^0$、$C_0^1$、$C_1^0$ 和 $C_1^1$ 四种。
其中只有 $C_0^1 = 0$。

这样预示着符合条件的子序列会很多，不如求出不合法的状况然后从总数里面减去。
那么我们只需要看 $a_{b_{i-1}}$ 某一位是 $1$ 而 $a_{b_i}$ 这一位是 $0$ 的情况就好了。
对于一个数，这种情况只需要有与二进制位数奇偶性不同的次数就可以了。

但是枚举数字判断不是特别能过。

我们可以尝试换种方法，将满足上面性质的 $a_{b_i}$ 当成 $a_{b_{i-1}}$ 的子集，枚举子集即可，然后判断枚举出来的这个数字是否在其后面。
实际写的时候我们可以将所有的 $a_i$ 存进桶里，存储下来每一个 $a_i$ 对应的以其开头的子序列个数，然后统计答案。

复杂度 $O(3^{log a_{max}})$，实测可过。

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 250010;
int l[N], r[N], sum[N];
const int mod = 1e9 + 7;
int a[N], f[N];
int tong[N];
int main()
{
	int n;
	scanf("%dd", &n);
	for(int i = 1; i <= n; i++)
	{
		scanf("%d", &a[i]);
		tong[a[i]] = i;
	}
	int ans = 0;
	for(int i = n; i >= 1; i--)
	{
		f[i] = 1;
		for(int j = a[i] & (a[i] - 1); j; j = a[i] & (j - 1))
			if(tong[j] > i)f[i] = (f[i] + f[tong[j]]) % mod;
		ans = (ans + f[i]) % mod;
	}
	ans = (ans - n + mod) % mod;
	printf("%d\n", ans);
	return 0;
}
```





















