---
title: P2375 [NOI2014] 动物园 题解
date: 2022-07-02
tags:
	- 题解
	- 字符串
	- KMP
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">动物园</div>
<div id="problem-info-from">NOI 2014</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2375">Luogu P2375</a></li><li><a href="https://loj.ac/p/2246">LibreOJ L2246</a></li><li><a href="https://www.acwing.com/problem/content/1002/">AcWing 1000</a></li><li><a href="https://uoj.ac/problem/5">UOJ #5</a></li><li><a href="https://darkbzoj.cc/problem/3670">BZOJ #3670</a></li></ul></div>

----

题目要求我们求出来一个 $num$ 数组。对于一个字符串来说，其 $num$ 的值就是其所有不重叠的 $border$ 的个数。而 $num$ 数组就是对这个字符串的所有前缀求 $num$。

这个有一点难想，我们不如先想一下弱化版的，即不考虑重叠这个因素了。

那么这个弱化版的 $num$ 数组其实就是字符串每一个前缀的 $border$ 个数，也就是其在 $fail$ 树上的深度。

那么如果我们再考虑上重叠的部分，那我们就对于一个长度为 $i$ 的前缀找到一个 $j$，使得长度为 $j$ 的前缀是长度为 $i$ 的前缀的一个 $border$。
因为 $\pi[i] < i$，所以我们找到的最大的 $j$ 的 $\pi[j]$ 就是我们的 $num[i]$ 了。

----

然后我们还需要一个优化。
因为我们如果碰到了一个奇怪的字符串，里面有 $10^6$ 个 `a`，那么我们就会被卡成 $O(n^2)$ 级别的。

我们可以尝试利用 KMP 的思想，尝试减小重复的递归。

那么我们可以将枚举的变量 $j$ 放到循环外面，结束循环的时候不初始化之，就不需要在不用枚举 $< j$ 的部分时枚举 $< j$ 的部分了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const ll mod = 1e9 + 7;
void solve()
{
	string s;
	cin >> s;
	ll n = (ll)s.length();
	vector<ll>ne(n + 1), num(n + 1);
	ll j = 0;
	num[1] = 1;
	for (ll i = 1; i < n; i++)
	{
		while (j > 0 && s[i] != s[j])j = ne[j];
		if (s[i] == s[j])j++;
		ne[i + 1] = j;
		num[i + 1] = num[j] + 1;
	}
	ll ans = 1;
	j = 0;
	for (int i = 1; i < n; i++) 
	{
		while (j > 0 && s[i] != s[j])j = ne[j];
		if (s[i] == s[j])j++;
		while (j * 2 > (i + 1))j = ne[j];
		ans = (ans * (num[j] + 1)) % mod;
	}
	cout << ans << endl;
	return;
}
int main()
{
	const char endl = '\n';
	std::ios::sync_with_stdio(false);
	cin.tie(nullptr);
	int T;
	cin >> T;
	while (T--)
	{
		solve();
	}
	return 0;
}
```

