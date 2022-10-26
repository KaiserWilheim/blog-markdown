---
title: CF920F SUM and REPLACE 题解
date: 2022-09-29
tags:
	- 题解
	- 并查集
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">SUM and REPLACE</div>
<div id="problem-info-from">Educational Codeforces Round 37</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/CF920F">Luogu CF920F</a></li><li><a href="https://codeforces.com/problemset/problem/920/F">CF 920 F</a></li></ul></div>

----

我们可以注意到，当 $x \leq 2$ 的时候， $\operatorname{d}(x)$ 就与 $x$ 相等了。
之后不管再怎么修改，我们这些 $x \leq 2$ 的位置都保持不变。

那么我们可以利用这样一个性质，来减少我们修改的次数。

因为我们遇到这些不需要修改的点就可以直接跳过，那我们可以通过一种方式来动态地维护出来当前点及后面的第一个需要修改的点。

我们可以使用并查集来维护。

听起来貌似是很突兀，但是我们可以发现使用并查集来维护刚好特别适合。

开始的时候每一个点都可能可以被修改，所以初始状态是整个区间都没有「联通」。
随着修改的逐渐发生，我们可以修改的点不断减少，区间的「连通性」也不断增大。

具体实现的时候，我们每一个不能修改的点都向其下一个点连边，这样我们在访问到一个不能修改的点的时候就直接跳到第一个能够修改的点进行修改。

至于询问的话我们直接用树状数组进行维护就可以了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010;
#define lowbit(x) (x&(-x))
int n, m, k;
ll tr[N];
void segadd(int x, int c)
{
	for(int i = x; i <= n; i += lowbit(i))
		tr[i] += c;
}
ll segsum(int x)
{
	ll res = 0;
	for(int i = x; i; i -= lowbit(i))
		res += tr[i];
	return res;
}
int a[N];
int d[N];
int p[N];
int find(int x)
{
	if(p[x] != x)p[x] = find(p[x]);
	return p[x];
}
int main()
{
	for(int i = 1; i <= 1000000; i++)
		for(int j = i; j <= 1000000; j += i)
			d[j]++;//预处理因数
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
	{
		scanf("%d", &a[i]);
		segadd(i, a[i]);
		p[i] = i;
	}
	p[n + 1] = n + 1;
	while(m--)
	{
		int op, l, r;
		scanf("%d%d%d", &op, &l, &r);
		if(op == 1)
		{
			for(int i = l; i <= r;)
			{
				segadd(i, d[a[i]] - a[i]);
				a[i] = d[a[i]];
				if(a[i] <= 2)p[i] = i + 1;
				if(i == find(i))i++;
				else i = p[i];
			}
		}
		else
		{
			printf("%lld\n", segsum(r) - segsum(l - 1));
		}
	}
	return 0;
}
```

