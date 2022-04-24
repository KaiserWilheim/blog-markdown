---
title: P3644 [APIO2015] 八邻旁之桥 题解
date: 2022-04-20
tags:
	- 题解
categories:
	题解
mathjax: true
---

Luogu P3644
LibreOJ #2888

APIO 2015

<!-- more -->
----

题目大意：

现在有一条河，将巴邻旁市分成了 $A$、$B$ 两个区域。
河的两岸均有 $10^9+1$ 栋房子。每一栋房子均有其编号，从 $A_1,B_1$ 一直到 $A_{10^9+1},B_{10^9+1}$。其中，编号为 $A_i$ 的房子与编号与 $B_i$ 的房子正好隔河相望。
相邻两栋房子之间的距离为 $1$。这包括类似 $A_i,A_{i+1}$ 的情况，也包括 $A_i,B_i$ 的情况。

现在有 $n$ 条通勤线路，有一些是需要跨河才能到达的。
以前人们都坐船，但是现在政府决定建造 $k$ 座大桥来帮助市民进行日常的通勤，使得所有人都可以（且必须）开车来通勤。

现在政府交给了你这 $n$ 条通勤线路和桥梁的个数 $k$，要求你最小化所有人通勤所需要的时间之和。

所有的桥梁必须垂直于河流。

----

首先猛地一看可能没有什么太大的思路，总觉得是个贪心，或者是个结论题。

然后看一眼数据范围：

$1 \leq K \leq 2$
$1 \leq N \leq 10^5$

那就简单多了。

我们可以分类讨论：

# $K=1$

首先我们需要忽略所有不跨河的人。
这个在输入的时候就直接统计入答案了。

当我们只有一座桥的时候，就意味着**所有人都必须通过这座桥。**

那么我们就可以将这些所有的路线拆成三部分：在A岸的、在桥上的和在B岸的。
而且因为桥是垂直于河流的，那么我们完全可以把在B岸的和在A岸的放在一起统计。

那么我们的问题就可以转化为，找一个点，使得所有的点到这个点的距离之和最小。

容易得出我们需要求的就是所有数字的中位数。

统计答案即可。

代码：

``` cpp
#define int long long
int t[N], tot, ans;

scanf("%lld%lld", &k, &n);
for(int i = 1; i <= n; i++)
{
	scanf("%s%lld%s%lld", x, &u, y, &v);
	if(x[0] == y[0]) ans += abs(u - v);
	else
	{
		t[++tot] = u, t[++tot] = v;
		ans++;
	}
}
sort(t + 1, t + 1 + tot);
for(int i = 1; i <= tot; i++)
	ans += abs(t[tot >> 1] - t[i]);
printf("%lld\n", ans);
```

# $K=2$

我们考虑一下每一个路线的实际路程。

假设一个路线的端点的编号分别是 $i$ 和 $j$（$i \leq j$），那么：

- 如果 $i$、$j$ 分别在桥的两侧，其实际路程为 $j-i$。
- 如果 $i$、$j$ 夹在两个桥的中间，其实际路程与 $\frac{i+j}{2}$ 有关。

于是我们可以考虑按照 $i+j$ 来对所有的通勤路线进行排序。同时枚举一个划分的位置，左边的都走左边的桥，右边的都走右边的桥。

那么我们需要求的就是动态中位数问题了。

我们仍然考虑使用一个大根堆和一个小根堆来维护动态中位数，只不过我们统计答案的时候换个思路。

我们可以统计前缀和，并且把这个前缀和按照桥的位置分成左右两部分。

此时的最小距离之和就是桥右侧点的坐标和减去左侧点的坐标和。

桥左边的部分枚举的是前缀和，右边的部分枚举的是后缀和。

然后我们取最小值即可。

代码：

``` cpp
struct need
{
	int x, y;
	bool operator < (const need &a) const
	{
		return (x + y) < (a.x + a.y);
	}
}order[N];
priority_queue <int, vector<int>               > q1;
priority_queue <int, vector<int>, greater<int> > q2;
int n, k, u, v, cnt;
int ans1[N], ans2[N];
int res, sum1, sum2;
void exchange()
{
	if(q1.top() > q2.top())
	{
		int u = q1.top(), v = q2.top();
		q1.pop(), q1.push(v);
		q2.pop(), q2.push(u);
		sum1 += v - u, sum2 -= v - u;
	}
}
void clear()
{
	while(!q1.empty()) q1.pop();
	while(!q2.empty()) q2.pop();
	sum1 = sum2 = 0;
	u = v = 0;
}
void senhan()
{
	clear();
	for(int i = 1; i <= cnt; i++)
	{
		q1.push(order[i].x), q1.push(order[i].y);
		sum1 += order[i].x + order[i].y, sum1 -= q1.top(), sum2 += q1.top();
		q2.push(q1.top()); q1.pop();
		exchange();
		ans1[i] = sum2 - sum1;
	}
}
void gohan()
{
	clear();
	for(int i = cnt; i >= 1; i--)
	{
		q1.push(order[i].x), q1.push(order[i].y);
		sum1 += order[i].x + order[i].y, sum1 -= q1.top(), sum2 += q1.top();
		q2.push(q1.top()); q1.pop();
		exchange();
		ans2[i] = sum2 - sum1;
	}
}

sum1 = 0, sum2 = 0;
sort(order + 1, order + 1 + cnt);
senhan();
gohan();
res = 1e18;
for(int i = 0; i <= cnt; i++)
	res = min(res, ans1[i] + ans2[i + 1]);
printf("%lld\n", ans + res);
```

全部加起来：

{% note success 示例代码 %}
``` cpp
#define _CRT_SECURE_NO_WARNINGS
#include<bits/stdc++.h>
using namespace std;
#define int long long
const int N = 400010;
struct need
{
	int x, y;
	bool operator < (const need &a) const
	{
		return (x + y) < (a.x + a.y);
	}
}order[N];
priority_queue <int, vector<int>               > q1;
priority_queue <int, vector<int>, greater<int> > q2;
int n, k, ans, u, v, cnt;
int ans1[N], ans2[N];
char x[2], y[2];
int t[N], tot, res, sum1, sum2;
void exchange()
{
	if(q1.top() > q2.top())
	{
		int u = q1.top(), v = q2.top();
		q1.pop(), q1.push(v);
		q2.pop(), q2.push(u);
		sum1 += v - u, sum2 -= v - u;
	}
}
void clear()
{
	while(!q1.empty()) q1.pop();
	while(!q2.empty()) q2.pop();
	sum1 = sum2 = 0;
	u = v = 0;
}
void senhan()
{
	clear();
	for(int i = 1; i <= cnt; i++)
	{
		q1.push(order[i].x), q1.push(order[i].y);
		sum1 += order[i].x + order[i].y, sum1 -= q1.top(), sum2 += q1.top();
		q2.push(q1.top()); q1.pop();
		exchange();
		ans1[i] = sum2 - sum1;
	}
}
void gohan()
{
	clear();
	for(int i = cnt; i >= 1; i--)
	{
		q1.push(order[i].x), q1.push(order[i].y);
		sum1 += order[i].x + order[i].y, sum1 -= q1.top(), sum2 += q1.top();
		q2.push(q1.top()); q1.pop();
		exchange();
		ans2[i] = sum2 - sum1;
	}
}
signed main()
{
	scanf("%lld%lld", &k, &n);
	for(int i = 1; i <= n; i++)
	{
		scanf("%s%lld%s%lld", x, &u, y, &v);
		if(x[0] == y[0]) ans += abs(u - v);
		else
		{
			order[++cnt] = { u, v };
			t[++tot] = u, t[++tot] = v;
			ans++;
		}
	}
	if(k == 1)
	{
		sort(t + 1, t + 1 + tot);
		for(int i = 1; i <= tot; i++)
			ans += abs(t[tot >> 1] - t[i]);
		printf("%lld\n", ans);
	}
	else if(k == 2)
	{
		sum1 = 0, sum2 = 0;
		sort(order + 1, order + 1 + cnt);
		senhan();
		gohan();
		res = 1e18;
		for(int i = 0; i <= cnt; i++)
			res = min(res, ans1[i] + ans2[i + 1]);
		printf("%lld\n", ans + res);
	}
	return 0;
}
```
{% endnote %}

