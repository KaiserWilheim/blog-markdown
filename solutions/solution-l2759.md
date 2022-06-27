---
title: L2759 蜜袋鼯（フクロモモンガ） 题解
date: 2022-01-16
tags:
	- 题解
	- 最短路
categories: 
	题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name"><ruby>蜜袋鼯<rt>フクロモモンガ</rt></ruby></div>
<div id="problem-info-from">JOI 2014 Final T4</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color">#0e1d69</div>
<div id="problem-info-submit"><ul><li><a href="https://loj.ac/p/2759">LibreOJ L2759</a></li></ul></div>

----

# 解析

首先我们考虑一下我们的策略。

无论给出何种方案，我们都需要遵守一个原则：非必要不爬升，且爬升时只爬升至足够飞过去的高度即可。

## 证明

对于前半句，我们这样分析：

我们假设可以突破高度只能大于等于0的限制。

那我们就可以持续进行飞行操作，直到到达终点再向上爬到树顶。

也就是说，我们将我们的操作从一系列的“飞跃-爬升”操作变成了一串连续的飞跃操作和一个爬升操作。

显然，只要我们选择的路径一定，我们最后爬升的路程一定是一定的。
我们不妨设其为 $h$ 。

如果我们在这种情况下，选择在中间进行一次爬升操作，假设我们爬升的高度为 $h'$ 。
那么，我们到达最后的高度就是$H_N - h + h'$。

然而，我们总共还是爬升了 $h-h'+h'=h$ 的高度，总共爬升操作所用的时间也是不变的。

更何况我们还会遇到到一棵树正上方时高度大于树高的时候，这时候我们就只能降低高度，而在最后多的时间来爬升刚才降低的那一段距离。
这会导致如果我们瞎爬升的话，结果会劣于非必要不爬升的结果。

那么对于后半句，我们这样分析：

假设一种极限情况，一路上的树一棵比一棵矮，且每一次从树顶开始飞行都会飞跃目标树。
那么如果我们选择爬升的高度太多，就会导致爬升高度的浪费。

我们完全可以选择一种极端的情况，那么就是让我们的高度保持为0即可。
我们在每次飞跃一条边（假设其为 $(u,v)$ ）时，我们如果之前已经保持了高度为0的话，我们就只需爬升 $w_{(u,v)}$ 的高度即可，即到达 $v$ 点时高度仍然为零。而如果我们仍有高度$x$的话，我们就爬升 ``` (x-w[(u,v)]>=0)?0:(w[(u,v)]-x) ``` 即可。
如果按照这样操作的话，我们只会在达到高度为0之前的时候才可能下降高度以适配较低的树高，最终得到的就是最优的结果。

----

之后对每一条边进行分析。

对于每一条边 $(u,v)$ ，会有以下四种情况：

1. 到达点正上方时的高度大于树高。
需要先向下爬到高度为 $h_v + w_{(u,v)}$ 的点才能从该边飞过，可以证明这是最优选择。
2. 在飞行途中不得不落地。
我们无论如何都无法通过这一条边，只能在向有向图中加边时忽略这一条边。
3. 不向上爬无法正常飞到下一个点。
根据上面的证明，我们只需向上爬升至高度 $w_{(u,v)}$ ，保证尽量靠近地面。
4. 可以正常飞到下一个点。
直接飞跃即可。

这样这个问题就变成了一个最短路问题。
而对于最短路的寻找，我们采用dijkstra算法。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100005;
const int M = 300005;
int head[N], ne[M << 1], to[M << 1], cost[M << 1], idx;
#define add(a,b,c) to[++idx]=b,ne[idx]=head[a],head[a]=idx,cost[idx]=c
#define bianli(x) for(int i=head[x];i;i=ne[i])
int val[N];
int n, m;
ll sum[N];
int H[N];
bool mark[N];
struct node
{
	ll v;
	int id;
	bool operator <(const node &A)const
	{
		return v > A.v;
	}
};
priority_queue<node> q;
void dijkstra(int at)
{
	for(int i=1;i<=n;i++)sum[i]=1e18;
	q.push({ 0,1 });
	H[1] = at;
	sum[1] = 0;
	ll v;
	while(!q.empty())
	{
		node now = q.top();
		q.pop();
		if(mark[now.id])continue;
		mark[now.id] = 1;
		int h = H[now.id];
		bianli(now.id)
		{
			if(h - cost[i] > val[to[i]])
			{
				v = sum[now.id] + h - cost[i] - val[to[i]] + cost[i];
				if(sum[to[i]] > v)
				{
					sum[to[i]] = v;
					H[to[i]] = val[to[i]];
					q.push({ sum[to[i]],to[i] });
				}
			}
			else if(h - cost[i] < 0)
			{
				v = sum[now.id] + cost[i] - h + cost[i];
				if(sum[to[i]] > v)
				{
					sum[to[i]] = v;
					H[to[i]] = 0;
					q.push({ sum[to[i]],to[i] });
				}
			}
			else
			{
				if(sum[to[i]] > sum[now.id] + cost[i])
				{
					sum[to[i]] = sum[now.id] + cost[i];
					H[to[i]] = h - cost[i];
					q.push({ sum[to[i]],to[i] });
				}
			}
		}
	}
}
int main()
{
	int at;
	scanf("%d%d%d", &n, &m, &at);
	for(int i = 1; i <= n; i++)scanf("%d", &val[i]);
	int a, b, c;
	while(m--)
	{
		scanf("%d%d%d", &a, &b, &c);
		if(val[a] >= c)add(a, b, c);
		if(val[b] >= c)add(b, a, c);
	}
	dijkstra(at);
	if(sum[n] == 1e18)puts("-1");
	else printf("%lld\n", sum[n] + val[n] - H[n]);
	return 0;
}
```
