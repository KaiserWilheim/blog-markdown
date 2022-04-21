---
title: 网络流 做题记录
date: 2022-04-21
tags:
	- 题解
	- 网络流
categories:
	题解
mathjax: true
---

网络流相关做题记录。

<!-- more -->

# 目录

## 网络流24题

- LibreOJ #6000 / Luogu P2756 飞行员配对方案问题 [>](/solution-flow/#飞行员配对方案问题)
- LibreOJ #6007 / Luogu P2774 方格取数问题 [>](/solution-flow/#方格取数问题)
- LibreOJ #6008 / Luogu P1251 餐巾计划问题 [>](/solution-flow/#餐巾计划问题)
- LibreOJ #6011 / Luogu P4015 运输问题 [>](/solution-flow/#运输问题)
- LibreOJ #6012 / Luogu P4014 分配问题 [>](/solution-flow/#分配问题)
- LibreOJ #6013 / Luogu P4016 负载平衡问题 [>](/solution-flow/#负载平衡问题)
- LibreOJ #6015 / Luogu P2754 [CTSC1999] 家园 [>](/solution-flow/#CTSC1999-家园)
- LibreOJ #6122 / Luogu P2770 航空路线问题 [>](/solution-flow/#航空路线问题)
- LibreOJ #6224 / Luogu P4012 深海机器人问题 [>](/solution-flow/#深海机器人问题)

## 最大流

- Luogu P2936 [USACO09JAN] Total Flow S [>](/solution-flow/#USACO09JAN-Total-Flow-S)
- Luogu P1343 地震逃生 [>](/solution-flow/#地震逃生)
- Luogu P2472 [SCOI2007] 蜥蜴 [>](/solution-flow/#SCOI2007-蜥蜴)
- Luogu P2891 [USACO07OPEN] Dining G [>](/solution-flow/#USACO07OPEN-Dining-G)
  Luogu P1402 酒店之王 ^
  Luogu P1231 教辅的组成 ^

## 费用流

- Luogu P1004 [NOIP2000 提高组] 方格取数 [>](/solution-flow/#NOIP2000-提高组-方格取数)
- Luogu P1006 [NOIP2008 提高组] 传纸条 [>](/solution-flow/#NOIP2008-提高组-传纸条)
- Luogu P2457 [SDOI2006] 仓库管理员的烦恼 [>](/solution-flow/#SDOI2006-仓库管理员的烦恼)
- Luogu P2517 [HAOI2010] 订货 [>](/solution-flow/#HAOI2010-订货)
- Luogu P2153 [SDOI2009] 晨跑 [>](/solution-flow/#SDOI2009-晨跑)


# 网络流24题

## 飞行员配对方案问题

英国飞行员连向源点，外籍飞行员连向汇点，可以搭配的英国飞行员连向外籍飞行员。边权均为 $1$。
跑一遍最大流即可。

最后枚举所有边，如果当前边连接一个外籍飞行员和一个英国飞行员，且其容量为空，那么这条边就被流经过，说明这条边代表的一堆飞行员可以配对，输出两端点即可。

{% note success 示例代码 %}
{% tabs l6000 %}
<!-- tab Luogu -->
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1010, M = 251010, INF = 1e8;

int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T) return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs()) while(flow = find(S, INF)) r += flow;
	return r;
}

int main()
{
	scanf("%d%d", &m, &n);
	S = 0, T = n + 1;
	memset(h, -1, sizeof h);
	for(int i = 1; i <= m; i++) add(S, i, 1);
	for(int i = m + 1; i <= n; i++) add(i, T, 1);
	int a, b;
	while(cin >> a >> b, a != -1) add(a, b, 1);
	printf("%d\n", dinic());
	for(int i = 0; i < idx; i += 2)
		if(e[i] > m && e[i] <= n && !f[i])
			printf("%d %d\n", e[i ^ 1], e[i]);
	return 0;
}
```
<!-- endtab -->
<!-- tab LibreOJ -->
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1010, M = 251010, INF = 1e8;

int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T) return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs())
		while(flow = find(S, INF))
			r += flow;
	return r;
}

int main()
{
	scanf("%d%d", &n, &m);
	S = 0, T = n + 1;
	memset(h, -1, sizeof h);
	for(int i = 1; i <= m; i++) add(S, i, 1);
	for(int i = m + 1; i <= n; i++) add(i, T, 1);
	int a, b;
	while(cin >> a >> b) add(a, b, 1);
	printf("%d\n", dinic());
	return 0;
}
```
<!-- endtab -->
{% endtabs %}
{% endnote %}

## 方格取数问题

我们一旦取走一个数，那么这个数周围的四个格子里面的数就不能被取走了。

那我们可以转化一下，不考虑**允许**，而是考虑**禁止**。

那我们就可以按照权值和最小的原则来得到一种方案并取出，剩下的就是我们所要求的方案。

那我们按照国际象棋棋盘的染色方案进行染色，即对于位于第 $i$ 行 $j$ 列的点，如果 $i+j$ 为奇数则向源点连一条边权为当前格子权值的边，并同时向周围四个点连一条边权为无限大的边；否则连向汇点，边权为当前格子权值。

由于最小割最大流定理，我们在求出最大流之后，拿所有格子的权值总和减去最大流即使我们要的答案。

{% note success 示例代码 %}
``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 1010, M = 200010, INF = 1e8;
const int X[4] = { 0,0,1,-1 };
const int Y[4] = { 1,-1,0,0 };

int n, m, ans = 0, st, ed;

int h[N], e[M], ne[M], c[M], idx = 1;
void add(int a, int b, int v)
{
	e[++idx] = b, c[idx] = v, ne[idx] = h[a], h[a] = idx;
}

int num[N];
queue<int> q;
bool bfs()
{
	memset(num, 0, sizeof(num));
	num[st] = 1;
	q.push(st);
	while(!q.empty())
	{
		int x = q.front();
		for(int i = h[x]; ~i; i = ne[i])
		{
			int y = e[i];
			if(c[i] > 0 && num[y] == 0)
			{
				num[y] = num[x] + 1;
				q.push(y);
			}
		}
		q.pop();
	}
	if(num[ed]) return true;
	else return false;
}

int dfs(int x, int q)
{
	int s = 0, t;
	if(x == ed) return q;
	for(int i = h[x]; i; i = ne[i])
	{
		int y = e[i];
		if(c[i] && num[y] == num[x] + 1 && q > s)
		{
			s += (t = (dfs(y, min(q - s, c[i]))));
			c[i] -= t;
			c[i ^ 1] += t;
		}
	}
	if(!s) num[x] = 0;
	return s;
}

int dinic()
{
	int sum = 0;
	while(bfs())
		sum += dfs(st, INF);
	return sum;
}

bool chq(int t1, int t2)
{
	if(t1 <= 0)return false;
	if(t1 > n)return false;
	if(t2 <= 0)return false;
	if(t2 > m)return false;
	return true;
}
int main()
{
	int x, id;
	scanf("%d %d", &n, &m);
	st = 0, ed = n * m + 1;
	memset(h, -1, sizeof(h));
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= m; j++)
		{
			scanf("%d", &x);
			ans += x;
			id = (i - 1) * m + j;
			if((i + j) & 1)
			{
				add(st, id, x);
				add(id, st, 0);
			}
			else
			{
				add(id, ed, x);
				add(ed, id, 0);
			}
		}
	}
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= m; j++)
		{
			if((i + j) & 1)
			{
				for(int k = 0; k <= 3; k++)
				{
					int t1 = i + X[k], t2 = j + Y[k];
					if(!chq(t1, t2)) continue;
					id = (i - 1) * m + j;
					add(id, (t1 - 1) * m + t2, INF);
					add((t1 - 1) * m + t2, id, 0);
				}
			}
		}
	}
	printf("%d", ans - dinic());
	return 0;
}
```
{% endnote %}

## 餐巾计划问题

详细分析可以见之前写的[这篇博客](/solutions/solution-p1251/)。

我们这样连边：
1. 源点与餐厅输出连一条容量为当天用量，费用为0的边；
2. 汇点与餐厅输入连一条容量为当天用量，费用为0的边；
3. 餐厅输出与下一日的餐厅输出连一条容量为无限，费用为0的边。
4. 源点与餐厅输入连一条容量为无限，费用为 $p$ 的边；
5. 餐厅输出与 $n$ 天后的餐厅输入连一条容量为无限，费用为 $s$ 的边，代表快洗部；
6. 餐厅输出与 $m$ 天后的餐厅输入连一条容量为无限，费用为 $f$ 的边，代表慢洗部。

然后跑一个最小费用最大流即可，最终的费用就是答案。

{% note success 示例代码 %}
{% tabs l6008 %}
<!-- tab Luogu -->
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int p, x, xp, y, yp;
int need[N];
int s[N];
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a]; h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

int EK()
{
	int cost = 0;
	while(spfa())
	{
		int t = incf[T];
		cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
	return cost;
}

signed main()
{
	scanf("%lld", &n);
	S = 0, T = n * 2 + 1;
	memset(h, -1, sizeof h);
	for(int i = 1; i <= n; i++)scanf("%lld", &need[i]);
	scanf("%lld%lld%lld%lld%lld", &p, &x, &xp, &y, &yp);
	for(int i = 1; i <= n; i++)
	{
		int r = need[i];
		add(S, i, r, 0);
		add(n + i, T, r, 0);
		add(S, n + i, INF, p);
		if(i < n) add(i, i + 1, INF, 0);
		if(i + x <= n) add(i, n + i + x, INF, xp);
		if(i + y <= n) add(i, n + i + y, INF, yp);
	}
	printf("%lld\n", EK());
	return 0;
}

```
<!-- endtab -->
<!-- tab LibreOJ -->
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int p, x, xp, y, yp;
int need[N];
int s[N];
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a]; h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

int EK()
{
	int cost = 0;
	while(spfa())
	{
		int t = incf[T];
		cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
	return cost;
}

signed main()
{
	scanf("%lld", &n);
	scanf("%lld%lld%lld%lld%lld", &p, &x, &xp, &y, &yp);
	S = 0, T = n * 2 + 1;
	memset(h, -1, sizeof h);
	for(int i = 1; i <= n; i++)scanf("%lld", &need[i]);
	for(int i = 1; i <= n; i++)
	{
		int r = need[i];
		add(S, i, r, 0);
		add(n + i, T, r, 0);
		add(S, n + i, INF, p);
		if(i < n) add(i, i + 1, INF, 0);
		if(i + x <= n) add(i, n + i + x, INF, xp);
		if(i + y <= n) add(i, n + i + y, INF, yp);
	}
	printf("%lld\n", EK());
	return 0;
}
```
<!-- endtab -->
{% endtabs %}
{% endnote %}

## 运输问题

费用流。

源点向仓库连边，仓库向商店连边，商店向汇点连边。

跑费用流即可。

{% note success 示例代码 %}
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int s[N];
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a]; h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

int EK()
{
	int cost = 0;
	while(spfa())
	{
		int t = incf[T];
		cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
	return cost;
}

int main()
{
	scanf("%d%d", &m, &n);
	S = 0, T = m + n + 1;
	memset(h, -1, sizeof h);
	for(int i = 1; i <= m; i++)
	{
		int a;
		scanf("%d", &a);
		add(S, i, a, 0);
	}
	for(int i = 1; i <= n; i++)
	{
		int b;
		scanf("%d", &b);
		add(m + i, T, b, 0);
	}
	for(int i = 1; i <= m; i++)
		for(int j = 1; j <= n; j++)
		{
			int c;
			scanf("%d", &c);
			add(i, m + j, INF, c);
		}

	printf("%d\n", EK());
	for(int i = 0; i < idx; i += 2)
	{
		f[i] += f[i ^ 1], f[i ^ 1] = 0;
		w[i] = -w[i], w[i ^ 1] = -w[i ^ 1];
	}
	printf("%d\n", -EK());
	return 0;
}
```
{% endnote %}

## 分配问题

费用流。

源点向工人连边，工人向工件连边，工件向汇点连边。

跑费用流即可。

和上一道题惊人地相似。

{% note success 示例代码 %}
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int s[N];
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a]; h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

int EK()
{
	int cost = 0;
	while(spfa())
	{
		int t = incf[T];
		cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
	return cost;
}

int main()
{
	scanf("%d", &n);
	S = 0, T = n * 2 + 1;
	memset(h, -1, sizeof h);
	for(int i = 1; i <= n; i++)
	{
		add(S, i, 1, 0);
		add(n + i, T, 1, 0);
	}
	for(int i = 1; i <= n; i++)
		for(int j = 1; j <= n; j++)
		{
			int c;
			scanf("%d", &c);
			add(i, n + j, 1, c);
		}
	printf("%d\n", EK());
	for(int i = 0; i < idx; i += 2)
	{
		f[i] += f[i ^ 1], f[i ^ 1] = 0;
		w[i] = -w[i], w[i ^ 1] = -w[i ^ 1];
	}
	printf("%d\n", -EK());
	return 0;
}
```
{% endnote %}

## 负载平衡问题

首先每个点向左右两边的点连边，费用为1，然后：

如果这个点多于平均值，那就向源点连其权值与平均值之差；
如果这个点少于平均值，那就像汇点连其权值与平均值之差。

这两种边的费用为0。

然后跑费用流即可。

{% note success 示例代码 %}
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int s[N];
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a]; h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

int EK()
{
	int cost = 0;
	while(spfa())
	{
		int t = incf[T];
		cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
	return cost;
}

int main()
{
	scanf("%d", &n);
	S = 0, T = n + 1;
	memset(h, -1, sizeof h);
	int tot = 0;
	for(int i = 1; i <= n; i++)
	{
		scanf("%d", &s[i]);
		tot += s[i];
		add(i, i < n ? i + 1 : 1, INF, 1);
		add(i, i > 1 ? i - 1 : n, INF, 1);
	}
	tot /= n;
	for(int i = 1; i <= n; i++)
		if(tot < s[i])
			add(S, i, s[i] - tot, 0);
		else if(tot > s[i])
			add(i, T, tot - s[i], 0);
	printf("%d\n", EK());
	return 0;
}
```
{% endnote %}

## [CTSC1999]家园

首先我们需要判断一下地球和月球是否联通，这个只需要使用并查集即可。

然后我们枚举天数，直到某一天转移的人达到要求了之后就停止循环，并输出天数。

我们在枚举的时候，逐天按照飞船的行进路线加边。因为每一次飞船都需要花一天时间来走一次的路程，所以我们每一条边都是从某一天的某一个站点到下一天的下一个站点，容量为飞船容量。

然后每一天跑最大流，得到的最大流结果就是当天以及之前所有天的转移人数总和。

{% note success 示例代码 %}
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1101 * 22 + 10, M = (N + 1100 + 13 * 1101) + 10, INF = 1e8;

int n, m, k, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];
struct Ship
{
	int h, r, id[30];
}ships[30];
int p[30];

int find(int x)
{
	if(p[x] != x) p[x] = find(p[x]);
	return p[x];
}

int get(int i, int day)
{
	return day * (n + 2) + i;
}

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T) return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs()) while(flow = find(S, INF)) r += flow;
	return r;
}

int main()
{
	scanf("%d%d%d", &n, &m, &k);
	S = N - 2, T = N - 1;
	memset(h, -1, sizeof h);
	for(int i = 0; i < 30; i++) p[i] = i;
	for(int i = 0; i < m; i++)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		ships[i] = { a, b };
		for(int j = 0; j < b; j++)
		{
			int id;
			scanf("%d", &id);
			if(id == -1) id = n + 1;
			ships[i].id[j] = id;
			if(j)
			{
				int x = ships[i].id[j - 1];
				p[find(x)] = find(id);
			}
		}
	}
	if(find(0) != find(n + 1)) puts("0");
	else
	{
		add(S, get(0, 0), k);
		add(get(n + 1, 0), T, INF);
		int day = 1, res = 0;
		while(true)
		{
			add(get(n + 1, day), T, INF);
			for(int i = 0; i <= n + 1; i++)
				add(get(i, day - 1), get(i, day), INF);
			for(int i = 0; i < m; i++)
			{
				int r = ships[i].r;
				int a = ships[i].id[(day - 1) % r], b = ships[i].id[day % r];
				add(get(a, day - 1), get(b, day), ships[i].h);
			}
			res += dinic();
			if(res >= k) break;
			day++;
		}
		printf("%d\n", day);
	}
	return 0;
}
```
{% endnote %}

## 航空路线问题

题目要求我们找到一条从起点城市到重点城市再回到起点城市的路径，并要求除起点城市外每一个点只能经过一次。

我们考虑拆点，除起点和终点容量为2之外容量均为1，然后按照飞机航线来建边，容量为1。

当然这样我们是不可能得出最大经过的城市个数的，我们需要用到费用流的手段。

我们将每个城市拆点的时候建的边的费用赋为1，其他的边赋为0。

然后我们跑费用流即可。

然后题目还让我们输出一个可能的方案。

于是我们就直接大暴搜就可以了。
还要注意起点终点直通的情况。

{% note success 示例代码 %}
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, INF = 1e8;
int n, m;
map<string, int>dic;
struct edge
{
	int next, to, fl, v;
}e[N << 1];
int h[N], idx = 1;
int dist[N], pre[N], vis[N], minn[N];
queue<int>q;
int s, t, val, flag[N], check;
string ss[N];

void add(int from, int to, int fl, int v)
{
	e[++idx].to = to; e[idx].next = h[from]; e[idx].fl = fl; e[idx].v = v; h[from] = idx;
}

void update(int x, int flow)
{
	e[pre[x]].fl -= flow;
	e[pre[x] ^ 1].fl += flow;
	if(e[pre[x] ^ 1].to)update(e[pre[x] ^ 1].to, flow);
}

int spfa()
{
	memset(vis, 0, sizeof(vis));
	while(!q.empty())q.pop();
	for(int i = 1; i <= t; i++)dist[i] = -INF;
	minn[s] = INF; dist[s] = 0; q.push(s); vis[s] = 1;
	while(!q.empty())
	{
		int x = q.front(); q.pop(); vis[x] = 0;
		for(int i = h[x]; i; i = e[i].next)
		{
			if(dist[e[i].to] < dist[x] + e[i].v && e[i].fl)
			{
				dist[e[i].to] = dist[x] + e[i].v;
				pre[e[i].to] = i;
				minn[e[i].to] = min(minn[x], e[i].fl);
				if(!vis[e[i].to])
				{
					vis[e[i].to] = 1;
					q.push(e[i].to);
				}
			}
		}
	}
	if(dist[t] == -INF)return -INF;
	update(t, minn[t]); val += minn[t];
	return minn[t] * dist[t];
}

int EK()
{
	int sum = 0;
	while(1)
	{
		int x = spfa();
		if(x == -INF)return sum;
		sum += x;
	}
}

void dfs1(int x)
{
	cout << ss[x] << endl;
	vis[x] = 1;
	for(int i = h[x]; i; i = e[i].next)
	{
		if(e[i].to > n && e[i].to <= 2 * n && e[i].fl == 0) { dfs1(e[i].to - n); break; }//第一次dfs只找一条路径，找到就break
	}
}

void dfs2(int x)
{
	vis[x] = 1;
	for(int i = h[x]; i; i = e[i].next)
	{
		if(e[i].to > n && e[i].to <= 2 * n && e[i].fl == 0 && !vis[e[i].to - n]) { dfs2(e[i].to - n); }//不走第一次路径走过的点
	}
	cout << ss[x] << endl;
}

int main()
{
	scanf("%d%d", &n, &m);
	t = n * 2 + 1;
	for(int i = 1; i <= n; i++)
	{
		cin >> ss[i];
		dic[ss[i]] = i;
	}
	for(int i = 1; i <= m; i++)
	{
		string s1, s2;
		cin >> s1 >> s2;
		int x = dic[s1], y = dic[s2];
		if(x > y)swap(x, y);
		if(x == 1 && y == n)check = 1;
		add(x, y + n, 1, 0);
		add(y + n, x, 0, 0);
	}
	add(s, n + 1, INF, 0);
	add(n + 1, s, 0, 0);
	add(n, t, INF, 0);
	add(t, n, 0, 0);
	for(int i = 1; i <= n; i++)
	{
		if(i != 1 && i != n) { add(i + n, i, 1, 1), add(i, i + n, 0, -1); }
		else { add(i + n, i, 2, 1), add(i, i + n, 0, -1); }
	}
	int maxflow = EK();
	if(val == 2)
	{
		printf("%d\n", maxflow - 2);
	}
	else if(val == 1 && check)
	{
		printf("%d\n", 2);
		cout << ss[1] << endl << ss[n] << endl << ss[1] << endl;
		return 0;
	}
	else
	{
		printf("No Solution!\n");
		return 0;
	}
	memset(vis, 0, sizeof(vis));
	dfs1(1);
	dfs2(1);
	return 0;
}
```
{% endnote %}

## 深海机器人问题

这个就是很明显的最大费用最大流，我们只需要把生物标本的价值抽象成为费用即可。

按照题目要求，我们每一个点都向上和向右分别建两条边，一条代表采集了生物标本，容量为1价值为c，另外一条代表已经被采集完了，容量为无限价值为0。

把所有的可以出发的坐标连向源点，容量为 $k$；所有的可以作为目的地的坐标连向汇点，容量为 $r$。

然后跑一遍费用流即可。

{% note success 示例代码 %}
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];

int get(int x, int y)
{
	return x * (m + 1) + y;
}

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, -0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] < d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}
	return incf[T] > 0;
}

int EK()
{
	int cost = 0;
	while(spfa())
	{
		int t = incf[T];
		cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
	return cost;
}

int main()
{
	int A, B;
	scanf("%d%d%d%d", &A, &B, &n, &m);
	S = (n + 1) * (m + 1), T = S + 1;
	memset(h, -1, sizeof h);
	for(int i = 0; i <= n; i++)
		for(int j = 0; j < m; j++)
		{
			int c;
			scanf("%d", &c);
			add(get(i, j), get(i, j + 1), 1, c);
			add(get(i, j), get(i, j + 1), INF, 0);
		}
	for(int i = 0; i <= m; i++)
		for(int j = 0; j < n; j++)
		{
			int c;
			scanf("%d", &c);
			add(get(j, i), get(j + 1, i), 1, c);
			add(get(j, i), get(j + 1, i), INF, 0);
		}
	while(A--)
	{
		int k, x, y;
		scanf("%d%d%d", &k, &x, &y);
		add(S, get(x, y), k, 0);
	}
	while(B--)
	{
		int r, x, y;
		scanf("%d%d%d", &r, &x, &y);
		add(get(x, y), T, r, 0);
	}
	printf("%d\n", EK());
	return 0;
}
```
{% endnote %}

# 最大流

## [USACO09JAN] Total Flow S

最大流板子。
将字符转换成数字进行建图即可。

{% note success 示例代码 %}
``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 10010, M = 200010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T)  return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs()) while(flow = find(S, INF)) r += flow;
	return r;
}

map<char, int>dic;
int cnt = 0;
int chq(char c)
{
	if(dic.count(c) == 1)
	{
		return (*dic.find(c)).second;
	}
	else
	{
		dic.insert(make_pair(c, ++cnt));
		return cnt;
	}
}
int main()
{
	scanf("%d", &n);
	memset(h, -1, sizeof h);
	S = 0, T = n + 1;
	for(int i = 1; i <= n; i++)
	{
		char a, b;
		int w;
		cin >> a >> b >> w;
		int x = chq(a), y = chq(b);
		add(x, y, w);
	}
	add(S, chq('A'), INF);
	add(chq('Z'), T, INF);
	printf("%d\n", dinic());
	return 0;
}
```
{% endnote %}

## 地震逃生

也是最大流板子，只不过在跑之前先要用并查集来判断一下连通性。
当然也可以直接跑，判断最大流量是否为0。

{% note success 示例代码 %}
``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 10010, M = 200010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T)  return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs()) while(flow = find(S, INF)) r += flow;
	return r;
}

int main()
{
	int x;
	scanf("%d%d%d", &n, &m, &x);
	memset(h, -1, sizeof h);
	S = 0, T = n + 1;
	add(S, 1, INF);
	add(n, T, INF);
	for(int i = 1; i <= m; i++)
	{
		int a, b, c;
		scanf("%d%d%d", &a, &b, &c);
		add(a, b, c);
	}
	int y = dinic();
	if((y == 0) && (x >= y))
	{
		puts("Orz Ni Jinan Saint Cow!");
		return 0;
	}
	printf("%d %d\n", y, (x % y == 0) ? x / y : ( int )(x / y) + 1);
	return 0;
}
```
{% endnote %}

## [SCOI2007] 蜥蜴

这道题在做之前得首先解决**平面距离**是什么。

根据实际测试，本题中的**平面距离**其实指的就是**欧几里得距离**。

由于其 $d \leq 4$，我们就可以直接暴力枚举。

下面是一张图，代表着我们这道题某个点在不同的 $d$ 时需要连边的范围：

![flowsolu1.png](https://s2.loli.net/2022/04/21/5IUqMYC4FfsnXSD.png)

然后处理题目所给的信息：
石柱的高度就是当前点的容量，所以我们需要拆点；
跳出地图外面了就代表可以连向汇点；
有蜥蜴就代表着可以连向源点。

然后就跑最大流，得到的是可以逃离地图的蜥蜴个数的最大值，输出蜥蜴总数与其之差即可。

{% note success 示例代码 %}
``` cpp
#include<bits/stdc++.h>
using namespace std;
#define pfin (x-1)*m+y
#define pfout (x-1)*m+y+n*m
const int N = 1010, M = 200010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];
int dis, tot;
char reich[N][N], volke[N][N];

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T)  return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs())
		while(flow = find(S, INF)) r += flow;
	return r;
}

void link(int u, int x, int y)
{
	if((x < 1) || (x > n) || (y < 1) || (y > m))
	{
		add(u, T, INF);
		return;
	}
	if(reich[x][y] == '0')return;
	add(u, pfin, INF);
	return;
}
void connect(int x, int y)
{
	if(reich[x][y] == '0')return;
	add(pfin, pfout, reich[x][y] - '0');
	if(volke[x][y] == 'L')
	{
		add(S, pfin, 1);
		tot++;
	}
	if((x == 1) || (x == n) || (y == 1) || (y == m))
		add(pfout, T, INF);
	int u = pfout;
	if(dis >= 1)
	{
		link(u, x - 1, y);
		link(u, x + 1, y);
		link(u, x, y - 1);
		link(u, x, y + 1);
	}
	if(dis >= 2)
	{
		link(u, x - 2, y);
		link(u, x + 2, y);
		link(u, x, y - 2);
		link(u, x, y + 2);
		link(u, x - 1, y - 1);
		link(u, x + 1, y - 1);
		link(u, x - 1, y + 1);
		link(u, x + 1, y + 1);
	}
	if(dis >= 3)
	{
		link(u, x - 3, y);
		link(u, x + 3, y);
		link(u, x, y - 3);
		link(u, x, y + 3);
		link(u, x - 2, y - 1);
		link(u, x + 2, y - 1);
		link(u, x - 2, y + 1);
		link(u, x + 2, y + 1);
		link(u, x - 1, y - 2);
		link(u, x + 1, y - 2);
		link(u, x - 1, y + 2);
		link(u, x + 1, y + 2);
		link(u, x - 2, y - 2);
		link(u, x + 2, y - 2);
		link(u, x - 2, y + 2);
		link(u, x + 2, y + 2);
	}
	if(dis >= 4)
	{
		link(u, x - 4, y);
		link(u, x + 4, y);
		link(u, x, y - 4);
		link(u, x, y + 4);
		link(u, x - 3, y - 1);
		link(u, x + 3, y - 1);
		link(u, x - 3, y + 1);
		link(u, x + 3, y + 1);
		link(u, x - 1, y - 3);
		link(u, x + 1, y - 3);
		link(u, x - 1, y + 3);
		link(u, x + 1, y + 3);
		link(u, x - 3, y - 2);
		link(u, x + 3, y - 2);
		link(u, x - 3, y + 2);
		link(u, x + 3, y + 2);
		link(u, x - 2, y - 3);
		link(u, x + 2, y - 3);
		link(u, x - 2, y + 3);
		link(u, x + 2, y + 3);
	}
	return;
}

int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d%d", &n, &m, &dis);
	S = 0, T = n * m * 2 + 1;
	for(int i = 1; i <= n; i++)scanf("%s", reich[i] + 1);
	for(int i = 1; i <= n; i++)scanf("%s", volke[i] + 1);
	for(int i = 1; i <= n; i++)
		for(int j = 1; j <= m; j++)
			connect(i, j);
	printf("%d\n", tot - dinic());
	return 0;
}
```
{% endnote %}

## [USACO07OPEN] Dining G

P1402、P2891和P1231实质上是一类问题，都是三种物品进行匹配，求最大匹配个数。

对于这类问题，我们的思路就是拆点。

将匹配的主体（比如P2891的奶牛、P1402的顾客和P1231的练习册）拆成两个点，中间连一条容量为1的边，然后两个点分别与剩下的两种物品连边，然后跑最大流即可。

{% note success 示例代码 %}
{% tabs 2891 %}
<!-- tab P2891 -->
``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 510, M = 200010, INF = 1e8;
int n, f, d;
int h[N], e[M], c[M], ne[M], idx;
int num[N];
int minn, s, t, minflow, maxflow, tot;
queue<int>q;

void add(int u, int v, int a)
{
	e[++idx] = v, c[idx] = a, ne[idx] = h[u], h[u] = idx;
	e[++idx] = u, c[idx] = 0, ne[idx] = h[v], h[v] = idx;
}

bool add_num()
{
	while(!q.empty())q.pop();
	for(int i = s; i <= t + n; i++)num[i] = -1;
	num[s] = 1;
	q.push(s);
	while(!q.empty())
	{
		int now = q.front();
		q.pop();
		for(int i = h[now]; i; i = ne[i])
		{
			if(c[i] && num[e[i]] == -1)
			{
				num[e[i]] = num[now] + 1;
				q.push(e[i]);
			}
		}
	}
	if(num[t] == -1)return false;
	else return true;
}

int dfs(int u, int s)
{
	if(u == t)
	{
		return s;
	}
	for(int i = h[u]; i; i = ne[i])
	{
		if(c[i] && num[u] + 1 == num[e[i]] && (minflow = dfs(e[i], min(s, c[i]))))
		{
			c[i] -= minflow;
			c[i ^ 1] += minflow;
			return minflow;
		}
	}
	return 0;
}

void dinic()
{
	while(add_num())
		while((minn = dfs(1, INF)))
			maxflow += minn;
}

int main()
{
	scanf("%d%d%d", &n, &f, &d);
	idx = 1;
	s = 1;
	t = 1 + f + n + d + 1;
	for(int i = 1; i <= f; i++)add(s, 1 + i, 1);
	for(int i = 1; i <= d; i++)add(1 + f + n + i, t, 1);
	for(int i = 1; i <= n; i++)add(1 + f + i, 1 + f + n + d + 1 + i, 1);
	for(int i = 1; i <= n; i++)
	{
		int dn, fn;
		scanf("%d%d", &fn, &dn);
		for(int q = 1; q <= fn; q++)
		{
			int fi;
			scanf("%d", &fi);
			add(1 + fi, 1 + f + i, 1);
		}
		for(int q = 1; q <= dn; q++)
		{
			int di;
			scanf("%d", &di);
			add(1 + f + n + d + 1 + i, 1 + f + n + di, 1);
		}
	}
	dinic();
	printf("%d\n", maxflow);
	return 0;
}
```
<!-- endtab -->
<!-- tab P1402 -->
``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 10010, M = 200010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T)  return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs())
		while(flow = find(S, INF)) r += flow;
	return r;
}

int main()
{
	int room, meal;
	scanf("%d%d%d", &n, &room, &meal);
	memset(h, -1, sizeof h);
	S = 0, T = n * 2 + room + meal + 1;
	for(int i = 1; i <= room; i++)
		add(S, i, 1);
	for(int i = 1; i <= meal; i++)
		add(i + room + n * 2, T, 1);
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= room; j++)
		{
			int x;
			scanf("%d", &x);
			if(x == 1)
				add(j, i + room, 1);
		}
		add(i + room, i + room + n, 1);
	}
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= meal; j++)
		{
			int x;
			scanf("%d", &x);
			if(x == 1)
				add(i + room + n, j + room + n * 2, 1);
		}
	}
	printf("%d\n", dinic());
	return 0;
}
```
<!-- endtab -->
<!-- tab P1231 -->
``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 40010, M = 800010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], ne[M], idx;
int q[N], d[N], cur[N];

void add(int a, int b, int c)
{
	e[idx] = b, f[idx] = c, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, ne[idx] = h[b], h[b] = idx++;
}

bool bfs()
{
	int hh = 0, tt = 0;
	memset(d, -1, sizeof d);
	q[0] = S, d[S] = 0, cur[S] = h[S];
	while(hh <= tt)
	{
		int t = q[hh++];
		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(d[ver] == -1 && f[i])
			{
				d[ver] = d[t] + 1;
				cur[ver] = h[ver];
				if(ver == T)  return true;
				q[++tt] = ver;
			}
		}
	}
	return false;
}

int find(int u, int limit)
{
	if(u == T) return limit;
	int flow = 0;
	for(int i = cur[u]; ~i && flow < limit; i = ne[i])
	{
		cur[u] = i;
		int ver = e[i];
		if(d[ver] == d[u] + 1 && f[i])
		{
			int t = find(ver, min(f[i], limit - flow));
			if(!t) d[ver] = -1;
			f[i] -= t, f[i ^ 1] += t, flow += t;
		}
	}
	return flow;
}

int dinic()
{
	int r = 0, flow;
	while(bfs())
		while(flow = find(S, INF)) r += flow;
	return r;
}

int main()
{
	int prac, answ;
	scanf("%d%d%d", &n, &prac, &answ);
	memset(h, -1, sizeof h);
	S = 0, T = n * 2 + prac + answ + 1;
	for(int i = 1; i <= prac; i++)
		add(S, i, 1);
	for(int i = 1; i <= answ; i++)
		add(i + prac + n * 2, T, 1);
	int m;
	scanf("%d", &m);
	while(m--)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		add(b, a + prac, 1);
	}
	scanf("%d", &m);
	while(m--)
	{
		int a, b;
		scanf("%d%d", &a, &b);
		add(a + prac + n, b + n * 2 + prac, 1);
	}
	for(int i = 1; i <= n; i++)
		add(i + prac, i + prac + n, 1);
	printf("%d\n", dinic());
	return 0;
}
```
<!-- endtab -->
{% endtabs %}
{% endnote %}

# 费用流

## [NOIP2000 提高组] 方格取数

每一个点向其右侧和下侧的两个点分别连边，同时每个点拆点，容量为2，边权为方格中的数。
$(0,0)$ 连向源点，$(n,n)$ 连向汇点。

然后跑最大费用最大流即可。

{% note success 示例代码 %}
``` cpp
#include<bits/stdc++.h>
#define pfin (i-1)*n+j
#define pfout (i-1)*n+j+n*n
using namespace std;
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];
int nums[10][10];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;

		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}

	return incf[T] > 0;
}

void EK(int &flow, int &cost)
{
	flow = cost = 0;
	while(spfa())
	{
		int t = incf[T];
		flow += t, cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
}

int main()
{
	scanf("%d", &n);
	memset(h, -1, sizeof h);
	int a, b, c;
	while(scanf("%d%d%d", &a, &b, &c))
	{
		if((a == 0) && (b == 0) && (c == 0))break;
		nums[a][b] = c;
	}
	S = 0, T = 2 * n * n + 1;
	add(S, 1, 2, 0);
	add(2 * n * n, T, 2, 0);
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= n; j++)
		{
			add(pfin, pfout, 1, -nums[i][j]);
			add(pfin, pfout, 1, 0);
			if(j != n)add(pfout, pfin + 1, 2, 0);
			if(i != n)add(pfout, pfin + n, 2, 0);
		}
	}
	int flow, cost;
	EK(flow, cost);
	printf("%d\n", -cost);
	return 0;
}
```
{% endnote %}

## [NOIP2008 提高组] 传纸条

每一个点向其右侧和下侧的两个点分别连边，同时每个点拆点，容量为1，边权为好感度。
$(0,0)$ 连向源点，$(n,m)$ 连向汇点。

然后跑最大费用最大流即可。

{% note success 示例代码 %}
``` cpp
#include<bits/stdc++.h>
#define pfin (i-1)*m+j
#define pfout (i-1)*m+j+n*m
using namespace std;
const int N = 5010, M = 100010, INF = 1e8;
int n, m, S, T;
int h[N], e[M], f[M], w[M], ne[M], idx;
int q[N], d[N], pre[N], incf[N];
bool st[N];
int nums[60][60];

void add(int a, int b, int c, int d)
{
	e[idx] = b, f[idx] = c, w[idx] = d, ne[idx] = h[a], h[a] = idx++;
	e[idx] = a, f[idx] = 0, w[idx] = -d, ne[idx] = h[b], h[b] = idx++;
}

bool spfa()
{
	int hh = 0, tt = 1;
	memset(d, 0x3f, sizeof d);
	memset(incf, 0, sizeof incf);
	q[0] = S, d[S] = 0, incf[S] = INF;
	while(hh != tt)
	{
		int t = q[hh++];
		if(hh == N) hh = 0;
		st[t] = false;

		for(int i = h[t]; ~i; i = ne[i])
		{
			int ver = e[i];
			if(f[i] && d[ver] > d[t] + w[i])
			{
				d[ver] = d[t] + w[i];
				pre[ver] = i;
				incf[ver] = min(f[i], incf[t]);
				if(!st[ver])
				{
					q[tt++] = ver;
					if(tt == N) tt = 0;
					st[ver] = true;
				}
			}
		}
	}

	return incf[T] > 0;
}

void EK(int &flow, int &cost)
{
	flow = cost = 0;
	while(spfa())
	{
		int t = incf[T];
		flow += t, cost += t * d[T];
		for(int i = T; i != S; i = e[pre[i] ^ 1])
		{
			f[pre[i]] -= t;
			f[pre[i] ^ 1] += t;
		}
	}
}

int main()
{
	scanf("%d%d", &n, &m);
	memset(h, -1, sizeof h);
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= m; j++)
		{
			scanf("%d", &nums[i][j]);
		}
	}
	S = 0, T = 2 * n * m + 1;
	add(S, 1, 2, 0);
	add(2 * n * m, T, 2, 0);
	for(int i = 1; i <= n; i++)
	{
		for(int j = 1; j <= m; j++)
		{
			add(pfin, pfout, 1, -nums[i][j]);
			add(pfin, pfout, 1, 0);
			if(j != m)add(pfout, pfin + 1, 2, 0);
			if(i != n)add(pfout, pfin + m, 2, 0);
		}
	}
	int flow, cost;
	EK(flow, cost);
	printf("%d\n", -cost);
	return 0;
}
```
{% endnote %}

## [SDOI2006] 仓库管理员的烦恼



## [HAOI2010] 订货

## [SDOI2009] 晨跑

{% note success 示例代码 %}
``` cpp
```
{% endnote %}
