---
title: P3540 [POI2012] 超夸克 Squarks 题解
date: 2022-10-15
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
<div id="problem-info-name">超夸克 Squarks</div>
<div id="problem-info-from">POI 2012</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3540">Luogu P3540</a></li><li><a href="https://loj.ac/p/2698">LibreOJ L2698</a></li></ul></div>

----

题目想要给我们 $n$ 个数，但不是直接给出这些数字，而是给出了 $\frac{n(n-1)}{2}$ 个数字，代表其两两的和。
现在题目需要我们求出这 $n$ 个数字。

设这 $n$ 个数字从小到大分别为 $x_1,x_2,x_3,\dots,x_n$，$m=\frac{n(n-1)}{2}$ 个和从小到大分别为 $a_1,a_2,a_3,\dots,a_m$。

我们取这些和中最小的两个值 $a_1$ 和 $a_2$，其必定等于 $x_1+x_2$ 和 $x_1+x_3$。
这个就不需要证明了，自己手动举几个例子就好了。

然后我们只需要找出 $x_2+x_3$ 的值就可以找出 $x_1$、$x_2$ 和 $x_3$ 各自的值了。

但是 $a_3$ 不一定等于 $x_2+x_3$。

因为 $n \leq 300$，我们可以枚举哪一个 $a$ 是 $x_2+x_3$。
我们最多需要枚举 $n-4$ 个。

因为 $x_2+x_j(j>3)$ 和 $x_3+x_j(j>3)$ 一定是大于 $x_2+x_3$ 的，那么可能比 $x_2+x_3$ 小且比 $x_1+x_3$ 大的值只能是 $x_1+x_j(j>3)$ 了。
这种值最多有 $n-4$ 个。

再来看我们如何通过枚举到的一个 $x_2+x_3$ 得出一组可行解。

我们现在已经确认了 $x_2+x_3$ 的值了，也就相当于确认了 $x_1$、$x_2$ 和 $x_3$ 各自的值了。
那么我们剩下的值中的最小值一定就是 $x_1+x_4$，这个上面已经证过了。
我们已经有了 $x_1$ 的值了，那么我们就可以得到 $x_4$ 的值。
根据这个值，我们可以得出 $x_2+x_4$、$x_3+x_4$ 的值。
去除这些值之后，我们剩下的最小值一定就是 $x_1+x_5$ 了。
以此类推，我们如果一直这样进行下去，直到将所有的和都消除完了之后，就构造出了一组可行解。
否则就代表该组 $x_1,x_2,x_3$ 不可行。

时间复杂度是 $O(n^2)$ 的。

我们对于每一个枚举的 $x_2+x_3$ 求出一组可行解，总复杂度是 $O(n^3)$ 的。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1010, M = 1000010;
int n, m;
int v[M];
int x[N][N], ans;
map<int, int>dic;
int tot[M], cnt;
int cost[M];
bool chq(int a, int b, int c)
{
	if(a <= 0 || b <= 0 || c <= 0)return false;
	if(a > b || a > c || b > c)return false;
	if(x[ans][1] == a && x[ans][2] == b && x[ans][3] == c)return false;
	ans++;
	x[ans][1] = a, x[ans][2] = b, x[ans][3] = c;
	return true;
}
int get_x()
{
	memset(cost, 0, sizeof(cost));
	cost[dic[x[ans][1] + x[ans][2]]]++;
	cost[dic[x[ans][1] + x[ans][3]]]++;
	cost[dic[x[ans][2] + x[ans][3]]]++;
	int now = 3;
	for(int i = 4; i <= n; i++)
	{
		while(tot[dic[v[now]]] - cost[dic[v[now]]] == 0)now++;
		x[ans][i] = v[now] - x[ans][1];
		for(int j = 1; j < i; j++)
		{
			int pos = dic[x[ans][i] + x[ans][j]];
			cost[pos]++;
			if(cost[pos] > tot[pos])return 1;
		}
	}
	sort(x[ans] + 1, x[ans] + n + 1);
	bool flag = true;
	for(int i = 1; i <= n; i++)
		if(x[ans][i] != x[ans - 1][i])flag = false;
	if(flag)ans--;
	return 0;
}
int main()
{
	scanf("%d", &n);
	m = n * (n - 1) / 2;
	for(int i = 1; i <= m; i++)
		scanf("%d", &v[i]);
	sort(v + 1, v + 1 + m);
	for(int i = 1; i <= m; i++)
	{
		if(!dic[v[i]])dic[v[i]] = ++cnt;
		tot[dic[v[i]]]++;
	}
	for(int i = 3, a, b, c; i <= n; i++)
	{
		b = (v[i] - v[2] + v[1]) / 2;
		c = v[i] - b;
		a = v[1] - b;
		if(chq(a, b, c))ans -= get_x();
	}
	printf("%d\n", ans);
	for(int i = 1; i <= ans; i++)
	{
		for(int j = 1; j <= n; j++)
			printf("%d ", x[i][j]);
		putchar('\n');
	}
	return 0;
}
```

