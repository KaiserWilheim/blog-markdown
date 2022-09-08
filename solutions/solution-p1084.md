---
title: P1084 [NOIP2012 提高组] 疫情控制 题解
date: 2022-09-04
tags:
	- 题解
	- 贪心
	- 二分
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">疫情控制</div>
<div id="problem-info-from">NOIP 2012 提高组</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P1084">Luogu P1084</a></li><li><a href="https://loj.ac/p/2607">LibreOJ L2607</a></li><li><a href="https://www.acwing.com/problem/content/359/">AcWing 357</a></li></ul></div>

----

题目要求我们求出一个最小的时间，使得每一个叶子节点到根节点的路径上都有驻军。
同时根节点本身不能驻军。

# 二分答案

我们考虑一个极端情况，那就是我们将所有的军队分散在根节点的所有子节点中，这样就可以使得每一个叶子结点到根节点的路径上都有驻军。
（如果军队数量少于根节点的子节点数量的话直接无解）
假设我们对于这种方案取了一个时间花费最小的，我们可以看出当时间大于其的时候肯定是有解的，而时间小于其的则不一定有解，那么我们就可以二分答案了。

左端点肯定是0，右端点我们设成所有边的权值和。

然后就是判定是否能够控制疫情了。

# 判定

我们选用上面说的方案，也就是说将所有的军队尽量部署在根节点的所有子节点中。
当然，如果某一个子节点没有能被驻军，但是其到其子树中的所有叶子结点的路径上都有驻军也可以。

## 移动并初步部署

那我们每一个军队都贪心地尽量往上走，直到到达根节点的子节点为止。
为了节省时间，我们使用倍增优化一下移动。

``` cpp 倍增优化
int fa[N][32];
ll dis[N][32];
void getFa(int p)
{//DFS求出父亲节点
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa[p][0])continue;
		int j = e[i];
		fa[j][0] = p;
		dis[j][0] = w[i];
		getFa(j);
	}
}
void initFa()
{//倍增预处理
	for(int p = 1; p <= n; p++)
	{
		for(int i = 1; i <= t; i++)
		{
			fa[p][i] = fa[fa[p][i - 1]][i - 1];
			dis[p][i] = dis[p][i - 1] + dis[fa[p][i - 1]][i - 1];
		}
	}
}
```

``` cpp 移动
vector<pair<ll, int> >q;
for(int i = 1; i <= m; i++)
{
	int x = mil[i];
	ll cnt = 0;
	for(int j = t; j >= 0; j--)
	{
		if(fa[x][j] > 1 && cnt + dis[x][j] <= lim)
		{
			cnt += dis[x][j];
			x = fa[x][j];
		}
	}
	if(fa[x][0] == 1 && cnt + dis[x][0] <= lim)
		q.emplace_back(lim - cnt - dis[x][0], x);//存储闲置军队
	else scr[x] = true;//打上驻扎标记
}
```

移动完后，一些军队是已经再也不能向上走了，只能在此地驻扎，而另一些军队是已经走到了根节点的子节点，还有时间可以走。我们称处于后者的状态的军队为“闲置”的军队。
我们将这些军队存储起来，供以后使用。

## 进一步部署

我们从根节点的每一个子节点开始进行DFS，看一看其子树内是否有叶子结点到根节点的路径上没有驻军。
如果有的话，那么该节点就需要被驻军。

``` cpp DFS
bool scr[N], need[N];
bool isScrd(int p)
{
	bool isLeaf = true;//是否为叶子结点
	if(scr[p])return true;//节点本身被驻军
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa[p][0])continue;
		isLeaf = false;
		if(!isScrd(e[i]))return false;//有一个叶子结点没有被驻军
	}
	if(isLeaf)return false;//为叶子结点且本身未驻军
	return true;//不是叶子结点且子树内没有未被驻军的叶子结点
}
```

对于每一个有闲置军队且需要驻军的子节点，我们选取剩余时间最小的那个军队让其驻扎。
剩下的军队中，我们选出来一些军队让其继续闲置。
这些军队需要满足一个条件，就是可以从当前节点走到根节点再走回来，否则我们不如让其直接驻扎在当前节点。

``` cpp
for(auto u = q.begin(); u != q.end(); u++)
{
	if(need[u->second] && u->first < dis[u->second][0] * 2)need[u->second] = false;//驻扎
	else rmtm.push_back(u->first);//继续闲置
}
```

## 再部署

现在我们需要处理的节点只剩下那些没有被驻军且需要驻军的子节点了。
我们这时候可以不管哪个军队驻扎在那里了，只需要将其剩余时间与需要被驻扎的点与根节点的距离比较即可。

我们使用两个小根堆来存储这些信息，每一次取出堆顶比较并在适当时间弹出即可。

``` cpp
vector<ll>rmtm;
for(int i = h[1]; ~i; i = ne[i])
	if(!isScrd(e[i]))
		need[e[i]] = true;
for(auto u = q.begin(); u != q.end(); u++)
{
	if(need[u->second] && u->first < dis[u->second][0] * 2)need[u->second] = false;
	else rmtm.push_back(u->first);
}//书接上文
vector<ll>tbd;
for(int i = h[1]; ~i; i = ne[i])
	if(need[e[i]])tbd.push_back(dis[e[i]][0]);//压入需要驻军的节点
if(rmtm.size() < tbd.size())return false;//军队数量不够直接返回不可行
sort(rmtm.begin(), rmtm.end());
sort(tbd.begin(), tbd.end());//因为不需要随时取出栈顶比较，所以直接采用vector+sort()
vector<ll>::iterator i = tbd.begin(), j = rmtm.begin();
while(i != tbd.end() && j != rmtm.end())
{
	if(*i <= *j)i++, j++;//当前栈顶军队可以到达栈顶节点
	else j++;//栈顶军队不能到达任意一个节点，直接弃用
}
if(i == tbd.end())return true;//全部节点都驻上军了
else return false;
```

至此，判断时间上限是否合法的部分就结束了。

# 全都加起来

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 50010, M = 100010;
int n, m, t;
int mil[N];
int h[N], e[M], ne[M], w[M], idx;
void add(int a, int b, int c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
int fa[N][32];
ll dis[N][32];
void getFa(int p)
{
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa[p][0])continue;
		int j = e[i];
		fa[j][0] = p;
		dis[j][0] = w[i];
		getFa(j);
	}
}
void initFa()
{
	for(int p = 1; p <= n; p++)
	{
		for(int i = 1; i <= t; i++)
		{
			fa[p][i] = fa[fa[p][i - 1]][i - 1];
			dis[p][i] = dis[p][i - 1] + dis[fa[p][i - 1]][i - 1];
		}
	}
}
bool scr[N], need[N];
bool isScrd(int p)
{
	bool isLeaf = true;
	if(scr[p])return true;
	for(int i = h[p]; ~i; i = ne[i])
	{
		if(e[i] == fa[p][0])continue;
		isLeaf = false;
		if(!isScrd(e[i]))return false;
	}
	if(isLeaf)return false;
	return true;
}
bool chq(ll lim)
{
	memset(scr, 0, sizeof(scr));
	memset(need, 0, sizeof(need));
	vector<pair<ll, int> >q;
	for(int i = 1; i <= m; i++)
	{
		int x = mil[i];
		ll cnt = 0;
		for(int j = t; j >= 0; j--)
		{
			if(fa[x][j] > 1 && cnt + dis[x][j] <= lim)
			{
				cnt += dis[x][j];
				x = fa[x][j];
			}
		}
		if(fa[x][0] == 1 && cnt + dis[x][0] <= lim)
			q.emplace_back(lim - cnt - dis[x][0], x);
		else scr[x] = true;
	}
	sort(q.begin(), q.end());
	vector<ll>rmtm;
	for(int i = h[1]; ~i; i = ne[i])
		if(!isScrd(e[i]))
			need[e[i]] = true;
	for(auto u = q.begin(); u != q.end(); u++)
	{
		if(need[u->second] && u->first < dis[u->second][0] * 2)need[u->second] = false;
		else rmtm.push_back(u->first);
	}
	vector<ll>tbd;
	for(int i = h[1]; ~i; i = ne[i])
		if(need[e[i]])tbd.push_back(dis[e[i]][0]);
	if(rmtm.size() < tbd.size())return false;
	sort(rmtm.begin(), rmtm.end());
	sort(tbd.begin(), tbd.end());
	vector<ll>::iterator i = tbd.begin(), j = rmtm.begin();
	while(i != tbd.end() && j != rmtm.end())
	{
		if(*i <= *j)i++, j++;
		else j++;
	}
	if(i == tbd.end())return true;
	else return false;
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d", &n);
	t = log2(n) + 1;
	ll l = 0, r = 0;
	for(int i = 1; i < n; i++)
	{
		int u, v, w;
		scanf("%d%d%d", &u, &v, &w);
		add(u, v, w), add(v, u, w);
		r += w;
	}
	getFa(1);
	initFa();
	scanf("%d", &m);
	for(int i = 1; i <= m; i++)
		scanf("%d", &mil[i]);
	bool isValid = false;
	while(l < r)
	{
		ll mid = (l + r) >> 1;
		if(chq(mid))
		{
			r = mid;
			isValid = true;
		}
		else l = mid + 1;
		//printf("[%lld %lld]\n", l, r);
	}
	if(!isValid)puts("-1");
	else printf("%lld\n", l);
	return 0;
}
```

