---
title: "S2OJ #1590. 网络 题解"
date: 2022-09-25
tags:
	- 题解
	- BFS
	- STL
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">网络</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1590">S2OJ #1590</a></li></ul></div>

----

题目要求我们对于给定的一张连通图，求出一棵生成树，使得其直径最小。
巧了，正好有一个算法针对这类问题，叫「最小直径生成树」。
其时间复杂度是 $O(n^3)$ 的，看起来好像不是很支持这一数据范围。

题目给的所有样例都可以使用BFS求出一个BFS树来水过去，这暗示着我们使用BFS。
~（这害的我只得了20分）~

考虑随便一条直径，我们可以找到其中点。
因为所有直径都是相交的，且至少相交于中点，所以我们只需要找到中点所在的位置就可以找到对应的一棵最小直径生成树了。

我们想要找到中点的话，可以进行 $n$ 次BFS来找到一个点，满足以这个点为根构造出来的一棵BFS树的最大深度在所有BFS树中最小。
此时，这一棵BFS树就是我们需要找的「最小直径生成树」了。
这种情况满足直径上的边数为偶数，点数为奇数。
（原题中给这种情况了20分）

当然，题目不可能这么简单。
我们还需要考虑一下中点在一条边上的情况。
假设我们中点所在的边是 $(u,v)$，那么此时以 $u$ 为根的BFS树和与 $v$ 为根的BFS树都满足最大深度在所有BFS树中最小这个条件。
而实际上的「最小直径生成树」应该是以「$(u,v)$ 的中点」为「根」，进行BFS得到的树。

我们考虑如何找出这种情况。
因为我们这个图有可能比较稠密，我们不能确定一个唯一的中心。
比如说构造出一个三元环和一堆孤立的点，这个三元环上的三个点与其他的所有点都直接连边，此时这三个点与其他所有点的距离均为1，都满足成为中心的条件。
但是我们可以发现，上述这种情况下，那些深度最大的点的集合是有部分重合的。
而如果中点在边上的情况下，我们对于以 $(u,v)$ 中点为起点来进行一遍BFS可以得到一个「深度最大的点的集合」。
假设这个深度是 $dis+0.5$（因为边的长度是1），那么我们可以推断出分别以 $u$ 和 $v$ 为起点构造出来的这种集合，其对应的深度是 $dis+1$。
继续研究可以发现，这两个集合并起来就是以 $(u,v)$ 为起点得到的集合，交起来是空集。
因为这些点可以被分为两类，一类在 $u$ 的子树中，与 $u$ 的距离为 $dis$，与 $v$ 的距离为 $dis+1$；另一类在 $v$ 的子树中，与 $u$ 的距离为 $dis+1$，与 $v$ 的距离为 $dis$。
此时前者会被分在 $v$ 的对应集合中，后者会被分在 $u$ 的对应集合中。
所以我们只需要一组 $(u,v)$，其满足之间直接连边的同时，深度最大的点的集合交集为空。
然后同时将 $u$ 和 $v$ 加到BFS的队列里面，进行一遍BFS即可。

这个算法的时间复杂度是 $O(n^3)$，仍然达不到我们想要的效果。

但是原算法中我们需要求的是全源最短路，这里我们利用了所有边权为1的性质将其替换成了更简单一些的 $n$ 次BFS。
使用类似的思想，我们可以使用`bitset`，通过做一些常数上面的优化来水过这道题。

我们如何使用`bitset`来优化建图？
我们建立 $n$ 个`bitset`，将其命名为`bitset<N>e[N]`。
就像邻接矩阵一样，我们如果在图中存在一条单向边 $(u,v)$ 的话，那么`e[u]`的第 $j$ 位就是1，否则为0。
我们可以使用`_Find_first()`和`_Find_next()`这两个函数，在 $O(\frac{n}{ω}+\operatorname{popcount}(k))$ 的时间复杂度内遍历bitset $k$ 所有为1的位置的下标。
抛弃掉布尔数组，我们也可以使用bitset来维护`vis`数组，并且可以利用按位与运算快速求出来当前点可以到达的位置。

总复杂度 $O(\frac{n^3}{ω})$。

代码如下：

``` cpp
#include<bits/stdc++.h>
#include<bitset>
using namespace std;
#define ll long long
const int N = 5010;
int n, m;
bitset<N>e[N], f[N];
bitset<N>vis;
int dis[N], far[N];
int bfs(int s)
{
	int res = 0;
	queue<int>q;
	vis.set();
	vis.flip(s);
	dis[s] = 0;
	q.push(s);
	while(!q.empty())
	{
		int u = q.front();
		q.pop();
		bitset<N> h = e[u] & vis;
		for(int i = h._Find_first(); i != h.size(); i = h._Find_next(i))
		{
			dis[i] = dis[u] + 1;
			res = max(res, dis[i]);
			q.push(i);
			vis.flip(i);
		}
	}
	return res;
}
int main()
{
	scanf("%d", &n);
	for(int i = 1; i <= n; i++)
	{
		char s[N];
		cin >> s + 1;
		for(int j = i + 1; j <= n; j++)
			if(s[j] == '1')e[i].flip(j), e[j].flip(i);
	}
	int minn = INT_MAX, minp = 0, minq = 0;
	for(int i = 1; i <= n; i++)
	{
		far[i] = bfs(i);
		if(far[i] < minn)minn = far[i], minp = i;
		for(int j = 1; j <= n; j++)
			if(dis[j] == far[i])f[i].flip(j);
	}
	queue<int>q;
	vis.set();
	for(int i = 1; i <= n; i++)
	{
		if(far[i] == minn)
		{
			for(int j = i + 1; j <= n; j++)
			{
				if(far[j] == minn && e[i][j] && (f[i] & f[j]).none())
				{
					minp = i, minq = j;
					break;
				}
			}
		}
		if(minq != 0)break;
	}
	vis.flip(minp);
	q.push(minp);
	if(minq != 0)
	{
		vis.flip(minq);
		q.push(minq);
		printf("%d %d\n", minp, minq);
	}
	while(!q.empty())
	{
		int u = q.front();
		q.pop();
		bitset<N> h = e[u] & vis;
		for(int i = h._Find_first(); i != h.size(); i = h._Find_next(i))
		{
			printf("%d %d\n", u, i);
			q.push(i);
			vis.flip(i);
		}
	}
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>