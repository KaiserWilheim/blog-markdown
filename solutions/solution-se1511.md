---
title: "S2OJ #1511. 华二 题解"
date: 2022-09-12
tags:
	- 题解
	- 计数
	- 数学
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">华二</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1511">S2OJ #1511</a></li></ul></div>

----

首先，题目说只有互质的数可以交换，我们就可以手动列举出来哪些数字可以交换。
经过一通分析之后，我们可以看出1,5,7三个数字可以与所有数字交换，2,4,6,8之间不可以互相交换，3,6,9之间不可以互相交换。

我们将1,5,7三个数字单独拿出来考虑，等到最后再直接插进去。
剩下的只有上面这些数字了，我们可以看出这些数字形成了两个集合，集合内部是不可以交换的。
因为6出现在了两个集合之中，所以我们单独把其拿出来，后面再考虑。

那我们将6和8等价为2，将9等价为3，这样就只有2和3了，其方案数就是可重集合的排列数。
对于一个集合，我们记该集合里面2的数量为 $sum2$，3的数量为 $sum3$，那么其答案数就是 $\dfrac{(sum2+sum3)!}{sum2! \times sum3!}$。

然后就是考虑6的情况了。
因为6与2和3都不能交换，那么这些6会将这一长串2和3分割成几个小串，我们对于这些小串做可重集合的排列，然后将答案乘起来即可。

然后就是考虑1,5,7了。
因为2,3与6都不能交换，我们可以将其等价为6，然后与1,5,7做可重集合的排列。
这次得到的答案乘以上面的结果就是最终我们需要的答案了。

<b>**对为什么可以等价的说明**</b>

我们题目中说的是相邻两个数的交换，相应的，不相邻的两个数可以通过不断的与与其相邻的数交换来达到交换这一目的。

假设我们有一个数 $a$，想要跨过 $b$ 到达一个新的位置，那么在途中其会不可避免地与 $b$ 进行交换，而不互质的两个数是不可以交换的，由此可以推出两个不互质的数不可以互相越过。
这就验证了我们直接将6和8等价为2，将9等价为3是正确的，因为在同一个集合内的数不可以互相交换，而跨集合的数是可以互相交换的。这样，当前集合内同一元素的顺序是固定的，也就没有了对于排列的枚举。
同时也就验证了我们拿6当集合的分割点是正确的，因为2与3都不可以与6交换，也就不可以越过6来到另外一个集合了。

参考代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010;
const ll mod = 998244353;
ll qpow(ll a, int x)
{
	ll res = 1;
	while(x)
	{
		if(x & 1)res = (res * a) % mod;
		a = (a * a) % mod;
		x >>= 1;
	}
	return res;
}
int n;
int a[N];
ll fac[N], inv[N];
ll cnt1, cnt5, cnt7;
ll cnt2, cnt3, cnt6;
int main()
{
	const char endl = '\n';
	std::ios::sync_with_stdio(false);
	cin.tie(nullptr);
	cin >> n;
	for(int i = 1; i <= n; i++)
		cin >> a[i];
	fac[0] = 1;
	for(int i = 1; i <= 1000000; i++)
		fac[i] = fac[i - 1] * i % mod;
	inv[1000000] = qpow(fac[1000000], mod - 2);
	for(int i = 999999; i >= 0; i--)
		inv[i] = inv[i + 1] * (i + 1) % mod;
	ll ans = 1;
	for(int i = 1; i <= n; i++)
	{
		if(a[i] == 1)cnt1++;
		else if(a[i] == 5)cnt5++;
		else if(a[i] == 7)cnt7++;
		else if(a[i] == 2 || a[i] == 4 || a[i] == 8)
		{
			cnt2++;
			cnt6++;
		}
		else if(a[i] == 3 || a[i] == 9)
		{
			cnt3++;
			cnt6++;
		}
		else if(a[i] == 6)
		{
			cnt6++;
			ans = ans * fac[cnt2 + cnt3] % mod * inv[cnt2] % mod * inv[cnt3] % mod;
			cnt2 = cnt3 = 0;
		}
	}
	ans = ans * fac[cnt2 + cnt3] % mod * inv[cnt2] % mod * inv[cnt3] % mod;
	ans = ans * fac[cnt1 + cnt6 + cnt5 + cnt7] % mod * inv[cnt1] % mod * inv[cnt6] % mod * inv[cnt5] % mod * inv[cnt7] % mod;
	cout << ans << endl;
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>