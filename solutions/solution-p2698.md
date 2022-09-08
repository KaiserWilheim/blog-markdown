---
title: P2698 [USACO12MAR] Flowerpot 题解
date: 2022-08-18
tags:
	- 题解
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">Flowerpot</div>
<div id="problem-info-from">USACO 2012 March Silver</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2698">Luogu P2698</a></li></ul></div>

----

题目要求我们在 $x$ 轴上找到最短的一个区间，使得该区间内所有点的纵坐标的极差不小于给定的一个数 $D$。

我们首先可以想到的做法就是二分，将其转化成为一个判定问题。
因为一旦找到了一个合法区间，我们如果继续扩展区间端点的话肯定还是能满足条件的，所以可以二分。

那我们考虑怎么判断。

我们可以枚举左端点，然后找到一个最近的右端点使得其满足条件，然后判断区间长度是否小于等于当前二分的长度即可。
我们还可以发现，区间右端点是单调不降的，这就给了我们使用滑动窗口的机会。

然后我们就可以好好想一想了。

我们每一次做滑动窗口统计到区间长度的时候，为什么不记录下来然后取最小值呢？毕竟这些答案都是固定的。

于是我们就光荣舍弃掉了二分，虽然复杂度没什么变化，因为复杂度瓶颈是排序。

具体做法就是首先将所有点按照横坐标排个序，然后拿两个单调队列维护区间最大值和区间最小值，不断移动左右端点即可。

参考代码：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010;
int n;
struct Raindrop
{
	int x, y;
	bool operator < (const Raindrop &a)const
	{
		return x < a.x;
	}
}a[N];
int t[N], b[N];
int h1, t1, h2, t2;
int main()
{
	int d;
	scanf("%d%d", &n, &d);
	for(int i = 1; i <= n; i++)
	{
		scanf("%d%d", &a[i].x, &a[i].y);
	}
	sort(a + 1, a + 1 + n);

	int ans = 1e9;
	h1 = h2 = 1;
	for(int l = 1, r = 0; l <= n; l++)
	{
		while(h1 <= t1 && t[h1] < l)h1++;
		while(h2 <= t2 && b[h2] < l)h2++;
		while(a[t[h1]].y - a[b[h2]].y < d && r < n)
		{
			r++;
			while(h1 <= t1 && a[t[t1]].y < a[r].y)t1--;
			t[++t1] = r;
			while(h2 <= t2 && a[b[t2]].y > a[r].y)t2--;
			b[++t2] = r;
		}
		if(a[t[h1]].y - a[b[h2]].y >= d)
			ans = min(ans, a[r].x - a[l].x);
	}
	if(ans >= 1e9)ans = -1;
	printf("%d\n", ans);
	return 0;
}
```

