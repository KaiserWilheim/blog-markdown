---
title: "S2OJ #993. Merge 题解"
date: 2022-08-30
tags:
	- 题解
	- 贪心
	- BFS
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Merge</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/996">S2OJ #996</a></li></ul></div>

----

题目要求我们把整张图按照某种给定的规则缩成一条链，并最大化其长度。
而这个规则是，每次找到两个不直接连边的点，我们新建一个点，然后将其与所有与前面两个点直接连边的点直接连边，然后删掉上面的两个点。

首先我们可以知道一个结论，就是图中如果出现了奇环就肯定无解。

因为对于一个奇环，我们不管怎么缩点，最终都会获得一个基于三元环的基环树。
而三元环中我们找不到任何两个点可以缩点，所以这个三元环只能随着缩点的进行留到最后，导致无解。

所以说我们可以判定，没有奇环就肯定有解。
而没有奇环可以转化为该图是一个二分图。
那么我们黑白染色一次就可以进行判定。

然后我们的策略就是，找到每一个连通块的直径，然后把所有的连通块的直径加起来就可以了。

直径的话，我们对连通块中的每一个点为起点进行一次BFS，取深度最大的点作为可能的直径，然后对连通块中每一个可能的直径取一个最大值就可以了。

代码如下：

``` cpp
#include<bits/stdc++.h>
#define ll long long
using namespace std;
const int N = 1000010, M = N << 1;
int n, m;
int h[N], e[M], ne[M], idx;
int color[N];
bool flag = true;
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
int cc[N], sc;
void paint(int s)
{
	queue<int>q;
	color[s] = 2;
	q.push(s);
	cc[s] = ++sc;
	while(!q.empty())
	{
		int p = q.front();
		q.pop();
		for(int i = h[p]; ~i; i = ne[i])
		{
			if(color[e[i]] == color[p])
			{
				flag = false;
				return;
			}
			if(color[e[i]])continue;
			color[e[i]] = color[p] ^ 1;
			cc[e[i]] = sc;
			q.push(e[i]);
		}
	}
}
int dis[N];
int dep[N];
void run(int s)
{
	memset(dis, 0, sizeof(dis));
	queue<int>q;
	dis[s] = 1;
	q.push(s);
	while(!q.empty())
	{
		int p = q.front();
		q.pop();
		for(int i = h[p]; ~i; i = ne[i])
		{
			if(dis[e[i]] && dis[e[i]] <= dis[p] + 1)continue;
			dis[e[i]] = dis[p] + 1;
			dep[cc[p]] = max(dep[cc[p]], dis[p]);
			q.push(e[i]);
		}
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= m; i++)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		add(a, b), add(b, a);

	}
	for(int i = 1; i <= n; i++)
	{
		if(!color[i])paint(i);
		if(!flag)
		{
			puts("-1");
			return 0;
		}
	}
	for(int i = 1; i <= n; i++)
		run(i);
	int sum = 0;
	for(int i = 1; i <= sc; i++)
		sum += dep[i];
	printf("%d\n", sum);
	return 0;
}
```

