---
title: "S2OJ #636. 购物 题解"
date: 2022-09-03
tags:
	- 题解
	- 贪心
	- BFS
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">购物</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/636">S2OJ #636</a></li></ul></div>

----

我们可以将这个问题转化为一个反悔贪心问题。

我们可以将优惠券当做我们已经选了的一些物品，这些物品的 $p_i$ 与 $q_i$ 相等，也就是说可以不需要考虑其反悔造成的代价，从而产生了“当前物品使用了优惠券抵消了差价”的效果。而如果将优惠券转让给比热物品使用的话，我们就相当于是把之前的这个物品的差价再加回到答案上去，从而产生了转让优惠券的效果。

为此，我们在一般的反悔贪心的基础上，增加一步向堆中加入 $k$ 个0的操作，相当于是插入了这些优惠券。

然后直接做反悔贪心即可。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define dvsng scanf
#define int long long
const int N = 50010;
int n, m, k;
int a[N], b[N];
bool vis[N];
struct Node
{
	int v, id;
	bool operator < (const Node &rhs)const
	{
		return v > rhs.v;
	}
};
priority_queue<Node>q1, q2;
priority_queue<int, vector<int>, greater<int> >q;
signed main()
{
	scanf("%lld%lld%lld", &n, &k, &m);
	for(int i = 1; i <= n; i++)
		scanf("%lld%lld", &a[i], &b[i]);
	for(int i = 1; i <= k; i++)q.push(0);
	for(int i = 1; i <= n; i++)
	{
		q1.push({ a[i],i });
		q2.push({ b[i],i });
	}
	int ans = 0;
	while(m > 0 && ans < n)
	{
		while(q1.size() && vis[q1.top().id])q1.pop();
		while(q2.size() && vis[q2.top().id])q2.pop();
		if(q.top() + q2.top().v < q1.top().v)
		{
			m -= q2.top().v + q.top();
			if(m < 0)break;
			int id = q2.top().id;
			q.pop();
			q.push(a[id] - b[id]);
			vis[id] = true;
			q2.pop();
		}
		else
		{
			m -= q1.top().v;
			if(m < 0)break;
			vis[q1.top().id] = true;
			q1.pop();
		}
		ans++;
	}
	printf("%lld\n", ans);
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>