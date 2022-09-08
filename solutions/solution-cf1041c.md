---
title: CF1041C Coffee Break 题解
date: 2022-09-04
tags:
	- 题解
	- 贪心
	- STL
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Coffee Break</div>
<div id="problem-info-from">Codeforces Round #509 (Div. 2)</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/CF1041C">Luogu CF1041C</a></li><li><a href="https://codeforces.com/problemset/problem/1041/C">CF 1041 C</a></li></ul></div>

----

题目大概说的就是，给定 $n$ 个数和一个 $k$，让我们把这 $n$ 个数划分到尽量少的集合中去。每一个集合需要满足一个条件，就是其中任意两个数的差必须大于 $k$。

我们可以贪心地每一次取出当前集合内的最小值，并不断查找最小可以选取的值并选取之，直到无法再选取新值为止。

查找最小可以选取的值可以使用`lower_bound()`，那我们需要一个数据结构，支持快速查询当前最小值。
因为数据保证不会有重复的数，我们可以使用`set<int>`来进行存储。

总复杂度 $O(n \log n)$。

参考代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 200010;
int n;
ll m, k;
struct Data
{
	ll v;
	int id;
	Data(ll _v, int _id) { this->v = _v, this->id = _id; }
	bool operator < (const Data &a)const
	{
		return v < a.v;
	}
};
set<Data>s;
int pos[N];
int main()
{
	scanf("%d%lld%lld", &n, &m, &k);
	for(int i = 1; i <= n; i++)
	{
		ll x;
		scanf("%lld", &x);
		s.emplace(x, i);
	}
	int now = s.begin()->v, cnt = 1;
	pos[s.begin()->id] = 1;
	s.erase(s.begin());
	for(int i = 2; i <= n; i++)
	{
		Data x(now + k + 1, 0);
		set<Data>::iterator it = s.lower_bound(x);
		if(it != s.end())
		{
			pos[it->id] = cnt;
			now = it->v;
			s.erase(it);
			continue;
		}
		++cnt;
		it = s.begin();
		pos[it->id] = cnt;
		now = it->v;
		s.erase(it);
	}
	printf("%d\n", cnt);
	for(int i = 1; i <= n; i++)
		printf("%d ", pos[i]);
	putchar('\n');
	return 0;
}
```

