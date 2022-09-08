---
title: P1954 [NOI2010] 航空管制 题解
date: 2022-09-06
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
<div id="problem-info-name">航空管制</div>
<div id="problem-info-from">NOI 2010</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P1954">Luogu P1954</a></li><li><a href="https://loj.ac/p/3760">LibreOJ L3760</a></li><li><a href="https://www.acwing.com/problem/content/981/">AcWing 979</a></li><li><a href="https://darkbzoj.cc/problem/2008">BZOJ #2008</a></li></ul></div>

----

本题的思路与[[HNOI2015] 菜肴制作](/solutions/solution-p3243)很像。
同样是将约束条件当成单向边建反图之后跑拓扑排序，但是这里还需要考虑最晚起飞时间的限制。
因为我们是倒序枚举的时间，所以最晚起飞时间可以转化为最早起飞时间。
我们可以拿一个桶来存储下每一个最晚起飞时间对应的飞机集合，当我们枚举到这个时间之后，将所有的飞机放入堆里。
当然，因为要求的是可行解，所以这里的堆可以换成队列。

对于最早可能的起飞序号，我们可以对每一个飞机做一遍拓扑排序。
这一次做拓扑排序的时候，我们需要让当前选定的这个飞机尽量在队尾，也就是尽量晚出队。
具体实现方式就是在每一次插入新的队尾的时候，看一看能否将当前这个飞机（也就是队尾的前一个元素）与队尾交换位置。

题目保证有解了，就不需要判定无解的情况了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 2010, M = 100010;
int n, m;
int h[N], e[M], ne[M], idx;
int deg[N];
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
int ans[N], tot;
int tmp[N];
vector<int>v[N];
void solve(int u)
{
	int q[N];
	int hh = 1, tt = 0;
	for(int i = 1; i <= n; i++)tmp[i] = deg[i] + 1;
	for(int t = n; t; t--)
	{
		if(!v[t].empty())
		{
			for(auto j : v[t])
			{
				if(!(--tmp[j]))
				{
					q[++tt] = j;
					if(hh < tt && q[tt - 1] == u)swap(q[tt], q[tt - 1]);
				}
			}
		}
		int p = q[hh++];
		if(p == u)
		{
			printf("%d ", t);
			break;
		}
		for(int i = h[p]; ~i; i = ne[i])
		{
			if(!(--tmp[e[i]]))
			{
				q[++tt] = e[i];
				if(hh < tt && q[tt - 1] == u)swap(q[tt], q[tt - 1]);
			}
		}
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
	{
		int x;
		scanf("%d", &x);
		v[x].push_back(i);
	}
	for(int i = 1; i <= m; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		deg[u]++;
		add(v, u);
	}
	priority_queue<int>q;
	for(int i = 1; i <= n; i++)tmp[i] = deg[i] + 1;
	for(int t = n; t; t--)
	{
		if(!v[t].empty())
		{
			for(auto j : v[t])
			{
				tmp[j]--;
				if(!tmp[j])q.push(j);
			}
		}
		int u = q.top();
		q.pop();
		ans[t] = u;
		for(int i = h[u]; ~i; i = ne[i])
			if(!(--tmp[e[i]]))q.push(e[i]);
	}
	for(int i = 1; i <= n; i++)
		printf("%d ", ans[i]);
	putchar('\n');
	for(int i = 1; i <= n; i++)
		solve(i);
	putchar('\n');
	return 0;
}
```

