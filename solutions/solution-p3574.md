---
title: P3574 [POI2014] FAR-FarmCraft 题解
date: 2022-06-08
tags:
	- 题解
	- 树形DP
	- 贪心
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">FarmCraft</div>
<div id="problem-info-from">POI 2014</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3574">Luogu P3574</a></li><li><a href="https://darkbzoj.cc/problem/3829">BZOJ #3829</a></li></ul></div>

----

题目需要我们从一号节点开始遍历整棵树，并给每一个节点记录下遍历到其的时间。
同时每一个节点又有一个自己的倒计时，当遍历到其的时候就开始计时。
我们需要求出所有节点中倒计时结束最慢的那一个的结束时间，并最小化之。

我们不难想象出一个简单的DP式子来求得最终的答案：

对于节点 $u$ 的子树 $v$，我们有如下的DP式子：

<center>$\operatorname{dp}[u]=\max(\operatorname{dp}[u],\operatorname{dp}[v]+\operatorname{sz}[u]+1)$</center>

其中 $\operatorname{dp}[u]$ 代表当前子树中最大的答案；
$\operatorname{sz}[u]$ 代表遍历过 $v$ 所在的子树之前已经经过的所有子树的大小之和再乘以2，这个在遍历完毕整个子树之后会更新为该子树的大小乘以2。

我们可以发现，我们最终的答案跟遍历子树的顺序有关，于是我们考虑对其进行排序。

对于一个节点 $x$ 的两个子树 $y$ 和 $z$，我们假设先遍历 $y$ 再遍历 $z$。
这样的话，我们的答案就是 $\max(\operatorname{dp}[y]+\operatorname{sz}[u]+1,\operatorname{dp}[z]+\operatorname{sz}[u]+\operatorname{sz}[y]+2+1)$。
我们假定这个方案比交换两个子树的遍历顺序得到的答案更优。
交换两个子树的遍历顺序之后得到的答案就是 $\max(\operatorname{dp}[z]+\operatorname{sz}[u]+1,\operatorname{dp}[y]+\operatorname{sz}[u]+\operatorname{sz}[z]+2+1)$。

我们最终得到如下式子：

<center>$\max(\operatorname{dp}[y]+\operatorname{sz}[u]+1,\operatorname{dp}[z]+\operatorname{sz}[u]+\operatorname{sz}[y]+2+1) > \max(\operatorname{dp}[z]+\operatorname{sz}[u]+1,\operatorname{dp}[y]+\operatorname{sz}[u]+\operatorname{sz}[z]+2+1)$</center>

我们将不等式左右两边同时约掉 $\operatorname{sz}[u]+1$，得到

<center>$\max(\operatorname{dp}[y],\operatorname{dp}[z]+\operatorname{sz}[y]+2) > \max(\operatorname{dp}[z],\operatorname{dp}[y]+\operatorname{sz}[z]+2)$</center>

因为 $\operatorname{dp}[y] < \operatorname{dp}[y]+\operatorname{sz}[z]+2$，$\operatorname{dp}[z] < \operatorname{dp}[z]+\operatorname{sz}[y]+2$，所以一定是 $\operatorname{dp}[y]+\operatorname{sz}[z]+2$ 与 $\operatorname{dp}[z]+\operatorname{sz}[y]+2$ 之间的差值导致了答案的变化。

因此，我们可以得到

<center>$\operatorname{dp}[y]-\operatorname{sz}[y] < \operatorname{dp}[z]-\operatorname{sz}[z]$</center>

然后我们就可以按照这样的方法排序了。

参考代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 500010;
template<typename T>
inline T read()
{
	T x = 0, f = 1; char c = getchar();
	while(!isdigit(c)) { if(c == '-')f = -1; c = getchar(); }
	while(isdigit(c))x = x * 10 + (c ^ 48), c = getchar();
	return x * f;
}
int n;
vector<int>e[N];
int val[N];
int f[N], sz[N];
bool cmp(const int a, const int b)
{
	return sz[a] - f[a] < sz[b] - f[b];
}
void dfs(int p, int fa)
{
	if(p != 1)f[p] = val[p];
	if(e[p].empty())return;
	for(auto i : e[p])
		if(i != fa)dfs(i, p);
	sort(e[p].begin(), e[p].end(), cmp);
	for(auto i : e[p])
	{
		if(i == fa)continue;
		f[p] = max(f[p], f[i] + sz[p] + 1);
		sz[p] += sz[i] + 2;
	}
}
int main()
{
	n = read<int>();
	for(int i = 1; i <= n; i++)
		val[i] = read<int>();
	for(int i = 1; i < n; i++)
	{
		int u = read<int>(), v = read<int>();
		e[u].push_back(v);
		e[v].push_back(u);
	}
	dfs(1, 0);
	printf("%d\n", max(f[1], sz[1] + val[1]));
	return 0;
}
```
