---
title: P4643 [国家集训队] 阿狸和桃子的游戏 题解
date: 2022-04-24
tags:
	- 题解
	- 贪心
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">阿狸和桃子的游戏</div>
<div id="problem-info-from">国家集训队</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4643">Luogu P4643</a></li></ul></div>

----

题目大意是，我们需要从一个带有边权和点权的图上对点进行染色，并按照一个特定的规则计分，最终输出两人得分之差，要求差最大。

某一个人的得分是其染色的点权值之和，再加上两端点均被其染色的边权之和。

----

因为我们需要求的是两人得分之差，所以原本一条边的两端点分别被两人染色之后边权互不归属的情况可以看做两人各取一半。
同理，如果一条边的两个端点都被同一个人染了色之后，这条边的两半权值都被这个人拿走了，与我们题目里的情况符合。

那我们就可以将边的权值一分为二，分别加到两个端点上，然后将这道题目变成选择最大点权和。

因为两人均采取最优策略，所以桃子取到了最大值之后，阿狸就会取次大值，然后桃子去第三大的值，如此往复。

于是我们就可以将点按照权值降序两两一组，求所有组内点权之差的总和。

然后我们就可以输出答案了。

代码：

``` cpp
#include <bits/stdc++.h>
const int N = 10010;
int a[N];
using namespace std;
int main()
{
	int n, m;
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
	{
		int k;
		scanf("%d", &k);
		a[i] = k << 1;
	}
	for(int j = 1; j <= m; j++)
	{
		int x, y, z;
		scanf("%d%d%d", &x, &y, &z);
		a[x] += z;
		a[y] += z;
	}
	sort(a + 1, a + 1 + n);
	int ans = 0;
	for(int i = n; i >= 1; i -= 2)ans += a[i] - a[i - 1];
	printf("%d", ans / 2);
	return 0;
}
```

