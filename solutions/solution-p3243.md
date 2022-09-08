---
title: P3243 [HNOI2015] 菜肴制作 题解
date: 2022-09-05
tags:
	- 题解
	- 贪心
	- 拓扑排序
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">菜肴制作</div>
<div id="problem-info-from">HNOI2015</div>
<div id="problem-info-difficulty">普及+ /提高</div>
<div id="problem-info-color">#52c413</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3243">Luogu P3243</a></li><li><a href="https://loj.ac/p/2114">LibreOJ L2114</a></li><li><a href="https://www.acwing.com/problem/content/2226/">AcWing 2224</a></li><li><a href="https://darkbzoj.cc/problem/4010">BZOJ #4010</a></li></ul></div>

----

这里我们需要注意一下题目的要求。
我们不能直接求出来字典序最小的序列，因为题目要求的是：
> 1. 在满足所有限制条件下，1号菜肴优先制作。
> 2. 在满足上述条件下，2号菜肴优先制作。
> 3. 在满足上述条件下，3号菜肴优先制作。
> 4. 以此类推……

我们可以想到，对于这种比较方式，我们可以通过贪心地让最后的数最大来达到要求。
假设这个数为 $x$，其可以让所有小于 $x$ 的数尽量靠前，同时因为 $x$ 的位置固定了，那些大于 $x$ 的数也就没有比较的需求了。
因此我们可以将其转化为其反串的字典序最大。

对于这一些关系，我们可以将时间的流向反过来，同时将关系的内容变成“菜肴 $i$ 必须后于菜肴 $j$ 制作”的形式。
之后我们可以将这种关系看做一些单向边，然后建立其反图，跑一遍拓扑排序就可以了。
因为我们还需要字典序最大，所以我们采用堆而不是队列来存储数据，每一次贪心地从堆顶拿出元素放到答案序列的尾部即可。

然后就是关于特判无解的情况。
无解只有一种情况：图中出现了环。自环也算。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 300010, M = 1000010;
int n, m;
int h[N], e[M], ne[M], idx;
int deg[N];
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
priority_queue<int>q;
int ans[N], tot;
void solve()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	bool isValid = false;
	for(int i = 1; i <= m; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		if(u == v)isValid = true;
		deg[u]++;
		add(v, u);
	}
	if(isValid)
	{
		puts("Impossible!");
		return;
	}
	for(int i = 1; i <= n; i++)
		if(!deg[i])q.push(i);
	while(!q.empty())
	{
		int u = q.top();
		q.pop();
		ans[++tot] = u;
		for(int i = h[u]; ~i; i = ne[i])
			if(!(--deg[e[i]]))q.push(e[i]);
	}
	if(tot < n)
	{
		puts("Impossible!");
		return;
	}
	for(int i = n; i; i--)
		printf("%d ", ans[i]);
	putchar('\n');
}
int main()
{
	int T;
	scanf("%d", &T);
	while(T--)
	{
		memset(deg, 0, sizeof(deg));
		idx = 0;
		tot = 0;
		solve();
	}
	return 0;
}
```

