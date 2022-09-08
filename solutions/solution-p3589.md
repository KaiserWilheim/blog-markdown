---
title: P3589 [POI2015] Kurs szybkiego czytania 题解
date: 2022-08-17
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
<div id="problem-info-name">Kurs szybkiego czytania</div>
<div id="problem-info-from">POI 2015</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3589">Luogu P3589</a></li><li><a href="https://darkbzoj.cc/problem/4377">BZOJ #4377</a></li></ul></div>

----

假设小串在大串中的某一个位置匹配上了，这时小串的开头是 $x$，那么小串的每一位都对应一个不等关系：

$$
\begin{cases}
a(i+x)+b \bmod{n} \in [0,p-1] & s_i = 0 \\\\
a(i+x)+b \bmod{n} \in [p,n-1] & s_i = 1
\end{cases}
$$

我们可以观察到，其中 $ai+b$ 这一部分对于每一个位置是固定的，我们将其提出来，并定义 $B = ai+b$。
化简后的式子如下：

$$
\begin{cases}
ax+B \bmod{n} \in [0,p-1] & s_i = 0 \\\\
ax+B \bmod{n} \in [p,n-1] & s_i = 1
\end{cases}
$$

将 $x$ 在模意义下拆成 $a'x'$，其中 $a'$ 为 $a'$ 在模意义下的乘法逆元。
化简后的式子如下：

$$
\begin{cases}
x'+B \bmod{n} \in [0,p-1] & s_i = 0 \\\\
x'+B \bmod{n} \in [p,n-1] & s_i = 1
\end{cases}
$$

我们还可以知道，因为 $a$ 与 $n$ 互质，所以每一个 $x'$ 都与 $x$ 一一对应，形成了一组双射关系，所以我们只需要求出 $x'$ 的所有可能取值的个数即可。
而 $x'$ 的取值范围也一目了然。

对于一个式子，我们有两种可能取值：
（这里用 $s_i = 1$ 来示范）

$$
\begin{cases}
x' \in [p-B,n-1-B] \\\\
x' \in [n-i-B,n-1] \cup [0,p-B+n]
\end{cases}
$$

这样的取值就像是数轴上的一些线段，而我们需要取的就是这些线段的交。
快速取线段交我们不会，但是我们可以将其转化成为其补集的并集的补集来计算。

同时还需要注意照顾到不合理的取值（$x \in [n-m+1,n-1]$），计算的时候需要将其剔除。

示例代码：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010, M = N << 1;
int n, m;
int a, b, p;
int l[N], r[N];
int pr[N];
int d[M], cnt;
int c[M];
int ans;
map<int, int>dic;
int main()
{
	scanf("%d%d%d%d%d", &n, &a, &b, &p, &m);
	string s;
	cin >> s;
	for(int i = 1; i <= m; i++)
	{
		int B = (1ll * i * a + b) % n;
		if(s[i - 1] == '1')
		{
			l[i] = p - B - 1;
			r[i] = n - B - 1;
			if(l[i] < 0)l[i] += n;
		}
		else
		{
			l[i] = n - B - 1;
			r[i] = p - B - 1;
			if(r[i] < 0)r[i] += n;
		}
		d[++cnt] = l[i], d[++cnt] = r[i];
	}
	for(int i = n - m + 1; i <= n - 1; i++)pr[n - i] = (1ll * a * i) % n;
	sort(d + 1, d + 1 + cnt);
	sort(pr + 1, pr + m);
	cnt = unique(d + 1, d + 1 + cnt) - d - 1;
	if(d[cnt] != n - 1)d[++cnt] = n - 1;
	for(int i = 1; i <= cnt; i++)
		dic.insert(make_pair(d[i], i));
	for(int i = 1; i <= m; i++)
	{
		c[dic[l[i]]]--;
		c[dic[r[i]]]++;
		if(l[i] > r[i])c[cnt]++;
	}
	d[0] = -1;
	for(int i = cnt, r = m - 1; i; i--)
	{
		c[i] += c[i + 1];
		int l = r;
		while(l && pr[l] > d[i - 1])l--;
		if(c[i] == m)ans += d[i] - d[i - 1] - r + l;
		r = l;
	}
	printf("%d\n", ans);
	return 0;
}
```

代码需要 $O(2)$。

思路是听 [$\text{Y}{\color{Red}\text{ouwike}}$](https://www.luogu.com.cn/user/261773) 大佬的讲解才弄懂的，大家快去模%%%


