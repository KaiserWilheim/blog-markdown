---
title: P3452 [POI2007] BIU-Offices 题解
date: 2022-10-13
tags:
	- 题解
	- 建图
	- 生成树
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">BIU-Offices</div>
<div id="problem-info-from">POI 2007</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3452">Luogu P3452</a></li><li><a href="https://loj.ac/p/2649">LibreOJ L2649</a></li></li><li><a href="https://darkbzoj.cc/problem/1098">BZOJ #1098</a></li></ul></div>

----

假如说把两人互相知道对方的电话号码看做无向边，把大楼看做集合，那么题目中说的“在不同楼里工作需要相互知道电话号码”可以看做“在不同集合中的点需要直接连边”。
那么在同一集合里面的点就绝不能直接连边，因为我们需要最大化集合的数量。否则我们就有可能将直接连边的两个点中的其中一个单独分到一个集合内。

那么我们看一下原图的补图，在同一个集合内的就必须不能相互到达在补图中就意味着能相互到达。
那么我们就可以将题目转化成为补图中求连通块的个数。

因为一旦原图十分稀疏，那么补图就会是接近 $O(n^2)$ 的水平了，所以我们不能直接遍历补图。
而根据题目的信息，原图的稠密程度不是很高，补图会十分稠密，再次否决了直接遍历补图的想法。

因为我们只需要求出连通块相关的信息，那我们就可以只求出来补图的生成树。
什么生成树呢？
考虑一下一个叫做「BFS树」的东西。
顾名思义，其依赖的是一次BFS，复杂度 $O(n+m)$，可以通过本题。

我们需要建立一个链表，存储所有的点，来快速遍历我们没有加入到生成树中的节点。

每一次我们遍历到一个节点的时候，我们将其第一个不能直接走到的点与其连上边即可。

具体操作看代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = 4000010;
struct List//链表
{
	int pr, ne;
};
int n, m;
int h[N], e[M], ne[M], idx;
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
bool vis[N], cov[N];
int sz[N], sc;
List re[N];
void del(int x)
{
	re[re[x].ne].pr = re[x].pr;
	re[re[x].pr].ne = re[x].ne;
}
void bfs(int s)
{
	vis[s] = true;
	sz[++sc] = 0;
	queue<int>q;
	q.push(s);
	while(!q.empty())
	{
		int p = q.front();
		q.pop();
		for(int i = h[p]; ~i; i = ne[i])
			if(!vis[e[i]])
				cov[e[i]] = true;
		for(int i = re[0].ne; i != -1; i = re[i].ne)
		{//遍历链表
			if(!cov[i])
			{
				vis[i] = true;
				sz[sc]++;
				del(i);
				q.push(i);
			}
			else cov[i] = false;
		}
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	re[0] = { -1,1 };
	for(int i = 1; i <= n; i++)
		re[i] = { i - 1,i + 1 };
	re[n].ne = -1;
	for(int i = 1; i <= m; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		add(u, v), add(v, u);
	}
	for(int i = 1; i <= n; i++)
		if(!vis[i])bfs(i);
	sort(sz + 1, sz + 1 + sc);
	printf("%d\n", sc);
	for(int i = 1; i <= sc; i++)
		printf("%d ", sz[i]);
	putchar('\n');
	return 0;
}
```

