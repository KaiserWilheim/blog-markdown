---
title: CF436E Cardboard Box 题解
date: 2022-08-20
tags:
	- 题解
	- 反悔贪心
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Cardboard Box</div>
<div id="problem-info-from">Zepto Code Rush 2014</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/CF436E">Luogu CF436E</a></li><li><a href="https://codeforces.com/problemset/problem/436/E">CF 436 E</a></li></ul></div>

----

题目说，我们可以花费 $a_i$ 的代价拿到1分，或者可以花费 $b_i$ 的代价拿到两分。我们需要求出得到 $k$ 分的代价和方案。

一眼反悔贪心问题，但是做法却不是那么显然。

一个根据之前反悔贪心能得出的做法就是，把 $a_i$ 扔进一个小根堆里面，然后选中了一个 $a_i$ 之后就将其所对应的 $b_i - a_i$ 扔进小根堆里面，取 $k$ 次堆顶。

但是有一种情况就是，有可能你取 $b_j$ 更优，但是 $a_j$ 没能取到导致 $b_j$ 也取不到。

我们就可以分开考虑，将所有的 $a_i$ 加入一个小根堆，之后将所有的 $b_i - a_i$ 加入另一个小根堆，再拿一个数组记录下哪些已经被选中了，然后每一次取的时候权衡一下到底是取 $a_i$ 最好还是取 $b_i$ 最好。

每一次决策的时候都需要将已经选过了的全部清掉，防止选到重复的。

参考代码：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 300010, M = N << 1;
int n, m;
int st[N];
ll a[M];
bool vis[M];
ll sum;
priority_queue<pair<ll, int> >q, t;
void dump(priority_queue<pair<ll, int> > &q)
{
	while(!q.empty() && vis[q.top().second])q.pop();
}
int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 0; i < n; i++)
	{
		scanf("%lld%lld", &a[i], &a[i + n]);
		a[i + n] -= a[i];
		q.push(make_pair(-a[i], i));
		t.push(make_pair(-a[i] - a[i + n], i));
	}
	while(m--)
	{
		dump(q), dump(t);
		int i = q.top().second;
		q.pop();
		dump(q);
		if(m && !t.empty() && a[i] - q.top().first >= -t.top().first)
		{
			q.push(make_pair(-a[i], i));
			i = t.top().second;
			t.pop();
		}
		if(i < n)q.push(make_pair(-a[i + n], i + n));
		sum += a[i];
		st[i % n]++;
		vis[i] = true;
	}
	printf("%lld\n", sum);
	for(int i = 0; i < n; i++)
		printf("%d", st[i]);
	puts("");
	return 0;
}
```

