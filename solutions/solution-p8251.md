---
title: P8251 [NOI Online 2022 提高组] 丹钓战 题解
date: 2022-06-30
tags:
	- 题解
	- 单调栈
	- 树状数组
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">丹钓战</div>
<div id="problem-info-from">NOI Online-S 2022</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P8251">Luogu P8251</a></li><li><a href="https://www.acwing.com/problem/content/4393/">AcWing 4390</a></li></ul></div>

----

看到题目，我们就知道这个题应该跟单调栈有关。

我们考虑每一次将一个二元组压入栈的时候，会弹出所有挡在其前面的二元组，直到碰到一个同时满足 $a_i \neq a_j$ 且 $b_i < b_j$ 的元素才会停下。

然后我们对于某个区间进行考虑。

题目是要求我们对每一个区间开一个单调栈的。
但是这样的话，时间复杂度绝对会超标。

所以我们考虑如何将其减小其时间复杂度。

考虑在当前区间左端点 $l$ 之前的二元组。
如果 $l$ 可以弹出这些二元组，那么后面一定是不用考虑的，毕竟后面再压入的时候，这些元素也是一定没有的，就不会对后面的元素产生影响。
如果 $l$ 不能弹出这些二元组，那么后面也是不用考虑的，毕竟当前区间内没有这个元素，也就不会对后面的元素产生影响了。

那么我们就考虑记录一下，对于每一个二元组，在其被压入单调栈之前所遇到的第一个不能将其弹出的二元组的下标。 
我们可以将其存储在一个数组里面，考虑将其命名为 $idx[]$。
那么，我们对于每一个区间，只需要看一下在这个区间内有多少个元素的 $idx[i] < l$就可以得出答案了。

这样就可以将整道题目分为两个部分：处理数据与回答询问。
其中处理数据的时间复杂度与单调栈类似，是 $O(n)$ 的。

但是，我们在回答询问的时候，时间复杂度是 $O(n^2)$ 的。
这样会大大影响我们的总用时。
（实测35秒左右）

然后我们可以考虑给他加一个 $\log$，比如使用一些数据结构什么的，这样就可以过掉这么大的数据。

于是我们就写了一个树状数组。
然后我们就可以将所有的询问离线下来，按照 $l$ 来分组解决了。

我们一开始在预处理完所有的 $idx[]$ 之后，就可以用一个桶来记录所有信息。
每一次我们扫到一个不同的 $l$ 的时候，将 $idx[i] < l$ 的所有二元组的编号扔进树状数组里面。
处理询问的时候直接在树状数组上面求区间和即可。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 500010;
int n, m;
int a[N], b[N];
int idx[N], tt;
int t[N];
int ans[N];
vector<int> tong[N];
vector<int> qus[N];
struct node
{
	int a, b, idx;
}sta[N];
struct nod1
{
	int l, r;
}q[N];
int lowbit(int x)
{
	return x & (-x);
}
void add(int x)
{
	for (int i = x; i <= n; i += lowbit(i))
		t[i]++;
}
int sum(int x)
{
	int ans = 0;
	for (int i = x; i; i -= lowbit(i))
		ans += t[i];
	return ans;
}
int main()
{
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= n; i++)
		scanf("%d", &a[i]);
	for (int i = 1; i <= n; i++)
		scanf("%d", &b[i]);
	sta[0] = { -1, 0, 0 };
	for (int i = 1; i <= n; i++)
	{
		while (((sta[tt].a == a[i]) || (sta[tt].b <= b[i])) && (tt > 0))
			tt--;
		idx[i] = sta[tt].idx;
		sta[++tt] = { a[i], b[i], i };
	}
	for (int i = 1; i <= n; i++)
		tong[idx[i]].push_back(i);
	for (int i = 1; i <= m; i++)
	{
		scanf("%d%d", &q[i].l, &q[i].r);
		qus[q[i].l].push_back(i);
	}
	for (int i = 1; i <= n; i++)
	{
		for (int k = 0; k < tong[i - 1].size(); k++)
			add(tong[i - 1][k]);
		for (auto now : qus[i])
		{
			int l = q[now].l, r = q[now].r;
			ans[now] = sum(r) - sum(l - 1);
		}
	}
	for (int i = 1; i <= m; i++)
		cout << ans[i] << endl;
	return 0;
}
```
