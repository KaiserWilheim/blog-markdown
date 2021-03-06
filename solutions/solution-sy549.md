---
title: '#549. 农夫奶牛的约翰们 题解'
date: 2022-04-12
tags:
	- 题解
	- 树链剖分
categories:
	题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">农夫奶牛的约翰们</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="http://192.168.0.111:1926/problem/549">SEZSY OJ #549</a></li></ul></div>

----

# 题意转化

这道题目要求我们求出树上两点 $x$、$y$ 之间最短路的长度。

首先，我们需要知道一个结论，就是树上连接两个点的简单路径有且只有一条，即为从一个点经过两个点的最近公共祖先再到另一个点的路径。

于是这道题目就被分解成为了两个小一点的问题：求两个点的LCA，和求出这两个点分别到LCA的距离。

# 分析做法

对于前者，我们已经有了一些选项：朴素、倍增或者树剖。

瞄一眼数据范围，$n \leq 10^6$，$m \leq 2 \times 10^5$。

（之前有人问了，那我就在这里讲一下）
（下面我这里使用形如 $O(\text{预处理}) \sim O(\text{单次询问})$ 的形式来描述复杂度）

朴素算法的时间复杂度是 $O(n) \sim \Theta(n)$，总体的时间复杂度（对于随机数据）大概在 $10^7$ 级别，优化一下应该可以过。
但是如果数据有意卡掉朴素算法，那么其时间复杂度会被卡到 $O(nm)$，大概在 $10^11$ 级别，这样子的运算量绝对是过不去的。
~~（所以为什么朴素算法可以过）~~

倍增算法的时间复杂度是 $O(n \log n) \sim O(\log n)$，总体的时间复杂度大概在 $10^8$ 级别，但是一般不会跑太满。
虽然看上去是挺唬人的，但是毕竟是优化过的算法，时间复杂度的稳定性也是比较高的，虽然说应付随机数据不如朴素好罢了。

树剖算法的时间复杂度是 $O(n) \sim O(\log n)$，总体的时间复杂度大概在 $10^6$ 级别左右，且常数一般较小。

综合比对了以下，看上去还是树剖比较靠谱。

----

对于后者，我们可以用一次 DFS 来求出所有节点到根节点的距离 $\operatorname{dis}(x)$。
由于这道题没有给出根节点，我这里习惯使用1号节点来当做根节点。
根节点的选择是不会影响算法的正确性的，两个点之间的路径盖世哪条还是哪条，不会因为根节点变了而改变的。

每次在询问的时候，我们对于给出的节点 $x$ 和 $y$，只需要回答 $\operatorname{dis}(x) + \operatorname{dis}(y) - 2 \times \operatorname{dis}(\operatorname{lca}(x,y))$ 即可。
因为 $x$ 到根的路径可以分为 $x$ 到 $\operatorname{lca}(x,y)$ 和 $\operatorname{lca}(x,y)$ 到根两段，$y$ 也同理。我们只需要回答 $x$ 到 $\operatorname{lca}(x,y)$ 和 $y$ 到 $\operatorname{lca}(x,y)$ 这两段路径的长度之和即可。

当然，我们可以选择在进行树剖的时候结合线段树或者树状数组维护区间和，同时进行一下边权转点权的方法来维护这个东西。

# 树剖？

树剖，全称为树链剖分，是一种通过将一棵树按照一定的规则分为若干条链的方案。

按照不同的法则，树链剖分可以分为重链剖分、长链剖分、实链剖分等等。一般比较常用的是重链剖分。

## 重链剖分

重链剖分的思想是，将一棵树按照“重边”来划分称为若干条“重链”。

### 重链

“重链”的定义是若干条首尾衔接的“重边”。
那么，如果想要了解重链，就需要先了解**重边**。

### 重边

“重边”的划分标准是它连接向一个重儿子。
这里需要注意的是，它只需要结束于一个重儿子，而无其他限制。
那么**重儿子**又是什么呢？

### 重儿子

“重儿子”是“重子节点”的别称。
我们判断一个子节点是否为重子节点的标准是它的子树大小。

对于一个节点的所有儿子，其中子树最大的那个儿子称为“重儿子”，其余的，则相对地称之为“轻儿子”。
连接某个节点和其重儿子的边叫做“重边”，而连接其与其轻儿子的边则相应地叫做“轻边”。

详细一点的解释可以看[OI-Wiki](https://oi-wiki.org/graph/hld/)或者[我的博客](https://kaiserwilheim.eu.org/OI/heavy-path-decomposition/)。

## 实现

树剖的实现是通过两个DFS进行的。

我们这次使用邻接表来存储树的边信息：

``` cpp
int w[N], h[N], e[M], ne[M], idx;
void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
```

第一个DFS记录了每个节点的父节点(fa)、深度(dep)、子树大小(sz)和重儿子(son)：

```cpp
int dep[N], sz[N], fa[N], son[N];
void dfs1(int p, int father, int depth)
{
	dep[p] = depth, fa[p] = father, sz[p] = 1;//初始化节点状态，记录其深度和父节点
	for(int i = h[p]; ~i; i = ne[i])//遍历其所有儿子
	{
		int j = e[i];
		if(j == father) continue;//防止遍历其父亲
		dfs1(j, p, depth + 1);//搜索当前儿子
		sz[p] += sz[j];//更新子树大小
		if(sz[son[p]] < sz[j]) son[p] = j;//判断是否为重儿子
	}
	return;
}
```

第二个DFS记录了每个节点所在重链的链顶节点(top)、dfs序(id)和重新定向的节点权值(nw)：

```cpp
int id[N], top[N], nw[N], cnt;
void dfs2(int p, int t)
{
	id[p] = ++cnt, nw[cnt] = w[p], top[p] = t;//初始化节点信息，记录其DFS序
	if(!son[p]) return;//是否为叶节点
	dfs2(son[p], t);//优先搜索在同一条重链上的重儿子
	for(int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if(j == fa[p] || j == son[p]) continue;
		dfs2(j, j);//搜索轻儿子，开一条新的重链，链顶为当前轻儿子
	}
	return;
}
```

此时我们将每一条重链都转化为了一段连续的区间，同时每一个节点的子树也转化为了一段连续的区间，这样子就会方便我们结合线段树或者树状数组等针对区间的数据结构进行维护。

这一点也是树剖对树上路径的问题很友好的原因所在。

----

这里我们实现的时候需要将边权转为点权，我们的方法是将每一条边的边权都转到其两个端点深度较大的那一个上面。

我们在第一遍DFS的时候，就首先将边权转到对应的点上去。
我们考虑记录一个`rw`数组，在第一个DFS里面遍历当前节点的所有子节点的时候加上一个`rw[j] = w[i]`就可以了。
而在第二遍DFS的时候，我们就把`rw`数组里面的值代替在正常情况下存储点权的`w`数组赋到`nw`里面去就可以了。

而在查询的时候，因为 $\operatorname{lca}(x,y)$ 这个点里面存的是它与 $fa[\operatorname{lca}(x,y)]$ 之间的这条边的边权，不在我们需要求的路径上，我们就不需要访问它了。

----

查询的时候，我们定义两个游标，初始位置分别在两个点上，然后不断向上跳至当前重链链顶的父亲，也就是`fa[top[p]]`，直到跳到同一个重链上，此时深度较小的那一个就是两个点的LCA。

同时我们在每一次游标跳转之前记录一下游标当前位置到游标当前所在重链链顶的这一段区间的边权和，我们使用树状数组进行统计。（因为线段树常数太大，无法通过此题）

查询的代码长这样：

``` cpp
int sumpath(int p, int q)
{
	int res = 0;
	while(top[p] != top[q])
	{
		if(dep[top[p]] < dep[top[q]])swap(p, q);//两个游标轮流跳，以防跳过了
		res += segsum(id[p]) - segsum(id[top[p]] - 1);
		p = fa[top[p]];
	}
	if(dep[p] < dep[q])swap(p, q);
	res += segsum(id[p]) - segsum(id[q]);
	return res;
}
```

我们在最后询问的时候只需要输出一个`sumpath(x,y)`即可。

参考代码见[这个提交记录](http://192.168.0.111:1926/submission/4463)。
加了个快读，但是影响不大。

----

如果我们选择放弃树状数组的话，我们就可以魔改一下第一次DFS的函数：

``` cpp
int dis[N];
void dfs1(int p, int father, int depth)
{
	dep[p] = depth, fa[p] = father, sz[p] = 1;
	for(int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if(j == father) continue;
		dis[j] = dis[p] + w[i];
		dfs1(j, p, depth + 1);
		sz[p] += sz[j];
		if(sz[son[p]] < sz[j]) son[p] = j;
	}
	return;
}
```

然后将查询的函数改一下：

``` cpp
int lca(int p, int q)
{
	while(top[p] != top[q])
	{
		if(dep[top[p]] < dep[top[q]])swap(p, q);
		p = fa[top[p]];
	}
	if(dep[p] < dep[q])swap(p, q);
	return q;
}
```

这样子就可以处理出来我们的 $\operatorname{dis}$ 和 $\operatorname{lca}(x,y)$，最后按照之前讲过的输出就可以了。

参考代码见[这个提交记录](http://192.168.0.111:1926/submission/4499)。这个没有加快读，但是也不慢。
