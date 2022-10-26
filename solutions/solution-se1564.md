---
title: "S2OJ #1564. 退休计划II 题解"
date: 2022-09-30
tags:
	- 题解
	- 生成树
	- DFS
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">退休计划II</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1564">S2OJ #1564</a></li></ul></div>

----

## 题目解读

题目要求我们找到一条路径，使得整个图分成三个块 $A$，$B$ 和路径本身，其中 $A$ 与 $B$ 大小相等，且 $A$ 与 $B$ 之间没有直接连边。
没有要求 $A$ $B$ 内部连通。

## 思想说明

一道结论题。

我们对于一张图进行DFS形成的DFS树，有着良好的一条性质，那就是其所拥有的非树边中不存在横叉边。
证明就直接略过。

我们回想一下DFS时候的过程。
在DFS的任意一个状态时，我们当前的图中存在三个点集：已经被遍历过且已经回溯的点，尚未遍历的点，和当前点到根的一条链。
同样是两个点集和一条链，只需要证明一下这种方案在某个时候可以满足要求即可。

首先证明除了链以外的两个点集之间不直接连边。
我们随便拿出来一个图的一个状态作为栗子：

![se1564-1.png](https://s2.loli.net/2022/09/30/pXNEao2LSeZdQ7A.png)

假设我们当前DFS到了12号节点，其子树已经DFS完成并回溯，下一步DFS的将是15号节点。

![se1564-2.png](https://s2.loli.net/2022/09/30/dsMLoxj1qZWQl2U.png)

我们标出了我们当前这一条链。

![se1564-3.png](https://s2.loli.net/2022/09/30/VeLdIu3SkRq4jta.png)

还有跨越点集的边。

因为我们遍历的顺序是从左到右，我们可以保证所有已经遍历过的点都在当前链的左边，尚未遍历的点都在当前链的右边或者下边，两者想要经过树边到达必须经过链，所以可以保证两者不会直接以树边相连。
再看非树边，其因为DFS树的特性只存在返祖边一种。
对于那些已经被回溯了的点，其沿着返祖边向上跳或是跳到另一个已经被回溯了的点，或是跳到当前的链上，可以保证不会跳到未被遍历的点。
要不然我们就会沿着这条边DFS下去了。

经过上面一大片不是十分严谨的证明，我们可以确定这样分是符合要求的。

再来看能否使得两个点集的大小相等。

我们DFS的时候只有两个操作，一是沿着边从当前点向下遍历其子节点，二是从当前点沿着边回溯到其父亲节点。
两种操作分别相当于是从未被遍历的点集内拿出一个点放到当前链上，二是从当前的链上拿出一个点放到已经被遍历过的点集内。
一个会将未被遍历的点集大小减一，一个会使已经遍历过的点集大小加一。
两种操作并不会同时出现，所以我们是可以将两个点集的大小变成相等的的。

## 代码实现

然后就是实现的部分了。

我们可以使用两个`vector`来维护当前的链和已经遍历完回溯过的点集。
未被遍历的点集不需要维护，因为其维护的话会过于麻烦，且大小可以通过前面维护的两个点集的大小计算得到。
每当我们向下遍历一个点的时候，我们将其放入代表当前链的`vector`的末尾。
而当我们从当前点回溯到其父节点的时候，我们将当前链的末尾`pop`掉，插入已经回溯过的点集之中。
每进行一个操作，我们都需要检查一下我们两个点集是否大小相等了。
一旦相等，我们就需要马上输出答案。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 200010, M = N << 1;
int n, m;
int h[N], e[M], ne[M], idx;
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
bool vis[N];
vector<int>a, b;
void chq()
{
	bool used[N];
	memset(used, 0, sizeof(used));
	printf("%d %d\n", b.size(), a.size());
	for(int i : b)
	{
		printf("%d ", i);
		used[i] = true;
	}
	putchar('\n');
	for(int i : a)
	{
		printf("%d ", i);
		used[i] = true;
	}
	putchar('\n');
	for(int i = 1; i <= n; i++)
	{
		if(!used[i])printf("%d ", i);
	}
	putchar('\n');
	exit(0);
}
void dfs(int p, int fa)
{
	b.push_back(p);
	vis[p] = true;
	if(a.size() == n - a.size() - b.size())
	{
		chq();
		return;
	}
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(vis[e[i]])continue;
		dfs(e[i], p);
		if(a.size() == n - a.size() - b.size())
		{
			chq();
			return;
		}
	}
	b.pop_back();
	a.push_back(p);
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= m; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		add(u, v), add(v, u);
	}
	for(int i = 1; i <= n; i++)
		if(!vis[i])dfs(i, 0);
	puts("-1");
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>

