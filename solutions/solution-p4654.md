---
title: P4654 [CEOI2017] Mousetrap 题解
date: 2022-07-08
tags:
	- 题解
	- 二分
	- 树形DP
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Mousetrap</div>
<div id="problem-info-from">CEOI 2017</div>
<div id="problem-info-difficulty">NOI / NOI+ / CTSC</div>
<div id="problem-info-color">#0e1d69</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4654">Luogu P4654</a></li><li><a href="https://loj.ac/p/2482">LibreOJ L2482</a></li></ul></div>

（蒟蒻的第一道黑题）

----

这是一棵无根树，不如我们先指定一个根节点，因为以1号点为根实在是想不到什么性质可以用了。
以老鼠所在节点为根也没有想到什么好性质，那不如就以陷阱房为根罢了。

这样做的话，老鼠想要去陷阱房必须向上走，不管他在哪里。

那么首先我们想一种特殊的情况，就是老鼠就在陷阱房旁边，也就是陷阱房的子节点，与其直接有边相连。
那么老鼠肯定不会向上走，他的最优策略肯定是向下走。
况且，老鼠在没有人清理走廊的情况下是无法走回头路的，所以其唯一能做的事就是找到一个很深很深的链，然后一头钻进去，再也出不来了。

这样的话，管理员必须采取反制措施。
那么现在管理员唯一能做的事，也只有将当前节点的重儿子堵住，才能让老鼠放弃这个决策，转而去次重儿子。

不对，上面还说是链呢，这里怎么就是重儿子了？

因为管理员需要将所有的支链堵住，才能最小化其操作次数；老鼠当然也知道这些，那么其就会向重儿子进发。
原因是这个样子的：
假设这里有一棵树：

<img src="https://s2.loli.net/2022/07/06/81HAXt27c63a45y.png" width="60%">

其中1号点是陷阱房，也是树根；老鼠一开始在二号房。
老鼠经过一番DFS，已经给每一个边标上了轻重：

<img src="https://s2.loli.net/2022/07/06/kr85l3wBzPnexvj.png" width="60%">

现在老鼠已经沿着次重边（2-15-19）到达了最后一个节点，同时管理员也堵上了所有老鼠可能经过的重边（2-4，15-16，19-20），现在他走不动了。
这是当前迷宫中的情况：

<img src="https://s2.loli.net/2022/07/06/L5yVWOGRHkYqNtv.png" width="60%">

假如我们擦干净15-19的话，老鼠就会走到15号点。
现在我们想要让老鼠继续向上到2号点，于是我们擦干净了2-15，然后……

老鼠钻进20号点了~~啊啊啊啊~~

如果20号点不是像我们给的图片中的那样光秃秃的，而是有很大的一棵子树（并且不如15和16号点的子树大），那么我们需要费的劲就更大了。

所以我们还需要有一个策略，就是在老鼠不能动的时候堵住所有支链，让老鼠只能沿着当前节点到根的链走。

于是我们堵住了15-20这条边，然后再擦干净2-15让老鼠上去，之后再堵住2-3，让老鼠不得不选择1-2，最后老鼠走到陷阱房，游戏结束。

这是老鼠初始的位置直接连接陷阱房的情况，我们假定了老鼠一开始绝对不会向上走。
那如果老鼠的初始位置不是与陷阱房直接相连呢？
也就是说，我们得考虑老鼠向上走一点，到达另一棵子树之后再一头扎下去的这种情况。

首先我们可以确定一点，就是我们不需要清理老鼠一开始向上走而污染的路径，因为我们在之后把老鼠再次赶上去的时候这里算是一个支链，需要堵住，而不是需要联通。

老鼠也很智能，他想让我们的操作数尽量多。
我们怎样对于一棵子树评估我们的操作数呢？

我们可以定义一个数组 $f_i$，代表我们将老鼠赶进 $i$ 的子树之后又将其赶到 $i$ 所需要的操作数。
老鼠肯定想选 $f$ 最大的子树，而我们会将其堵住，所以老鼠实际上选的是次大的子树，然后就这样一直走到底。

现在我们需要找出，哪一棵子树才是老鼠想要去的。
我们总不能枚举所有点，这个复杂度太高。

我们发现这个可以进行二分。
怎么二分呢？

我们可以记录一个 $sum_i$，代表 $i$ 号节点到根路径上所有支路都被堵上所需要的操作数。
其维护的方式是这样的：$sum_x = sum_{fa_x} + sz_x - [x \neq m]$。（根节点就不需要这个了）
其中 $sz_i$ 代表的是 $i$ 的子树个数，而不是子树大小。

然后我们考虑二分什么。
不用想，肯定是操作数。
假设我们给管理员设定了一个KPI，希望他能够在 $k$ 次操作内将老鼠赶到陷阱房。
而老鼠既然已经知道了管理员的这个目标，那么他只需要找到一颗 $sum > k$ 的子树一头钻进去就不需要出来了。
那么管理员就需要将所有 $sum > k$ 的子树封住。同时我们需要注意到封堵这个操作也是需要付出代价的，所以每一次我们封堵的时候 $k$ 都需要自减1。

如果管理员的手速不够快，老鼠钻进了一个 $sum > k$ 的子树，那么就宣告失败；
如果管理员封堵太多了，导致 $k < 0$ 了，那也宣告失败；
否则就算成功了。

下面是二分函数：

``` cpp
bool chq(int k)
{
	for (int i = m, cnt = 1; i != rt; i = fa[i], cnt++)
	{//向上遍历到根
		int tmp = 0;
		for (int j = h[i]; ~j; j = ne[j])
		{//枚举子节点
			int v = e[j];
			if (vis[v] || //会被老鼠弄脏，不需要堵
				sum[i] + f[v] <= k)//不会超出目标，暂时不需要堵
				continue;
			if (!cnt)return false;//管理员的手速不够快
			tmp++;
			cnt--;
		}
		k -= tmp;
	}
	return k >= 0;//需要封堵的太多了
}
```

至此，所有的东西我们就都分析完了。

示例代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010, M = 2000010;
int n, rt, m;
int h[N], e[M], ne[M], idx;
int sz[N], fa[N], f[N];
bool vis[N];
int sum[N];
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
void dfs(int p, int father)
{
	fa[p] = father, sz[p] = 0;
	int h1 = 0, h2 = 0;
	for (int i = h[p]; ~i; i = ne[i])//处理子树大小
		if (e[i] != father)sz[p]++;
	if (p != rt)//处理sum
		sum[p] = sum[father] + sz[p] - (p != m);
	for (int i = h[p]; ~i; i = ne[i])
	{//遍历子树，处理子树次大值
		int j = e[i];
		if (j == fa[p])continue;
		dfs(j, p);
		if (f[j] > h1)h2 = h1, h1 = f[j];
		else if (f[j] > h2)h2 = f[j];
	}
	f[p] = h2 + sz[p];
}
bool chq(int k)
{
	for (int i = m, cnt = 1; i != rt; i = fa[i], cnt++)
	{
		int tmp = 0;
		for (int j = h[i]; ~j; j = ne[j])
		{
			int v = e[j];
			if (vis[v] || sum[i] + f[v] <= k)continue;
			if (!cnt)return false;
			tmp++;
			cnt--;
		}
		k -= tmp;
	}
	return k >= 0;
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d%d", &n, &rt, &m);
	for (int i = 1; i < n; i++)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		add(a, b), add(b, a);
	}
	dfs(rt, 0);
	f[rt] = 0;
	for (int i = m; i; i = fa[i])//处理到根的路径，给它们打标记
		vis[i] = true;
	int l = 0, r = 1e8;
	while (l < r)
	{
		int mid = (l + r) >> 1;
		if (chq(mid))r = mid;
		else l = mid + 1;
	}
	printf("%d\n", r);
	return 0;
}
```

