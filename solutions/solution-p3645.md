---
title: P3645 [APIO2015] 雅加达的摩天楼 题解
date: 2022-05-27
tags:
	- 题解
	- 最短路
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">雅加达的摩天楼</div>
<div id="problem-info-from">APIO 2015</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3645">Luogu P3645</a></li><li><a href="https://loj.ac/p/2887">LibreOJ L2887</a></li><li><a href="https://uoj.ac/problem/111">UOJ #111</a></li><li><a href="https://darkbzoj.cc/problem/4070">BZOJ #4070</a></li></ul></div>

----

题目说，只要一栋楼里面的一只doge知道了消息，那么他就可以在不产生任何代价的情况下将这条消息传递给这栋楼里面的所有doge。

利用这一条信息，我们可以将doge的行动转化为摩天楼之间的边，一只doge能够前往的两栋楼之间会直接或间接联通。

于是我们就可以想到跑最短路。图上的所有边权均为1，从0到1的最短路即为传递消息的代价。

然后再看数据范围：

$N \in \[1,30000 \] , M \in \[ 2,30000 \] , P_i \in \[ 1,30000 \]$

当头一棒。

我们可怜的最短路算法承受不起如此多的边（$O(n^2)$ 级别），我们甚至可以将这张图看做一张完全图来计算时间复杂度，而结果必然是爆炸。

我们如果一开始分析错误了，把它当做费用流来跑SPFA的话，最多能拿到36分(UOJ&LibreOJ)/75分(Luogu)。

我们如果使用链式前向星连边、建边，跑Dijkstra的话，可以拿到36分(UOJ&LibreOJ)/80分(Luogu)的好成绩。

然后我们会发现上面会MLE，于是选用vector存边。
我们如果使用vector连边、建边，跑Dijkstra的话，可以拿到和上面一样的好成绩，并且仍然MLE。

然后我们考虑换个连边方式，不在让每一条边的边权都是相同的了，考虑从一栋楼向其他的楼连边，边权为跳跃次数，妄图减少内存用量。

结果仍为失败。

我们不再考虑显式连边，而考虑在转移的时候再连边，这里就需要用SPFA了。

然后再跑一遍就可以过了。

``` cpp
#include <bits/stdc++.h>
#define ll long long
using namespace std;
const int N = 30010;
int n, m;
vector<int>e[N];
int dis[N], vis[N];
queue<int>q;
void spfa(int s)
{
	memset(dis, 63, sizeof(dis));
	dis[s] = 0, vis[s] = 1;
	q.push(s);
	while(!q.empty())
	{
		int u = q.front();
		q.pop();
		vis[u] = 0;
		for(auto i : e[u])
		{
			for(int j = 1; u + i * j < n; j++)
			{
				int v = u + i * j;
				if(dis[v] > dis[u] + j)
				{
					dis[v] = dis[u] + j;
					if(!vis[v])
					{
						q.push(v);
						vis[v] = 1;
					}
				}
			}
			for(int j = 1; u - i * j >= 0; j++)
			{
				int v = u - i * j;
				if(dis[v] > dis[u] + j)
				{
					dis[v] = dis[u] + j;
					if(!vis[v])
					{
						q.push(v);
						vis[v] = 1;
					}
				}
			}
		}
	}
}
int main()
{
	scanf("%d%d", &n, &m);
	int s, t;
	for(int i = 0; i < m; i++)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		if(i == 0)s = a;
		if(i == 1)
		{
			t = a;
			continue;
		}
		e[a].push_back(b);
	}
	spfa(s);
	if(dis[t] == 0x3f3f3f3f)puts("-1");
	else cout << dis[t] << endl;
	return 0;
}
```
