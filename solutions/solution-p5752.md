---
title: P5752 [NOI1999] 棋盘分割 题解
date: 2022-08-06
tags:
	- 题解
	- 搜索
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">棋盘分割</div>
<div id="problem-info-from">NOI 1999</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P5752">Luogu P5752</a></li><li><a href="https://www.acwing.com/problem/content/323/">AcWing 321</a></li></ul></div>

----

题目本身已经很简洁了，我这里就不再赘述了。

题目中说的“均方差” $\sigma$，其实就是标准差 $s$。

我们如果想要使标准差 $s$ 最小化，可以使方差 $s^2$ 最小化，效果是一样的。

我们可以观察到，不管我们怎么取值，我们 $\bar{x}$ 的值一定等于 $\dfrac{\sum\limits_{i=1}^8 \sum\limits_{j=1}^8 a_{i,j}}{n}$。所以我们可以提前预处理出来 $\bar{x}$，使用区间DP的方式来最小化 $\dfrac{(x_i - \bar{x})^2}{n}$。

我们设我们的DP数组为 $f[x_1][y_1][x_2][y_2][k]$，代表在已经切了 $k$ 刀时，左上角为 $(x_1,y_1)$，右下角为 $(x_2,y_2)$ 的这一个矩形所对应的DP值。

由于暴力枚举循环会很多，所以这里采用了记忆化搜索的方法。

每一次我们往下面的状态转移的时候，我们都有两种方案：横着切与竖着切。
两种切法都是枚举分割点，然后向下转移。

向下转移也有两种转移的方法，切成的两半都可以向下继续搜索，而剩下的部分就直接利用二维前缀和 $O(1)$ 求值就可以了。

当然我们还有另外一种方法，就是维护平方和 $\sum x_i^2$，使平方和最小。

代码如下：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 20, M = 10;
int n, m = 8;
int s[M][M];
double f[M][M][M][M][N];
double X;
double get(int x1, int y1, int x2, int y2)
{
	double delta = s[x2][y2] - s[x1 - 1][y2] - s[x2][y1 - 1] + s[x1 - 1][y1 - 1];
	delta = delta - X;
	return delta * delta;
}
double dp(int x1, int y1, int x2, int y2, int k)
{
	if(f[x1][y1][x2][y2][k] >= 0)return f[x1][y1][x2][y2][k];
	if(k == n)return f[x1][y1][x2][y2][k] = get(x1, y1, x2, y2);
	double t = 1e9;
	for(int i = x1; i < x2; i++)
	{
		t = min(t, dp(x1, y1, i, y2, k + 1) + get(i + 1, y1, x2, y2));
		t = min(t, dp(i + 1, y1, x2, y2, k + 1) + get(x1, y1, i, y2));
	}
	for(int i = y1; i < y2; i++)
	{
		t = min(t, dp(x1, y1, x2, i, k + 1) + get(x1, i + 1, x2, y2));
		t = min(t, dp(x1, i + 1, x2, y2, k + 1) + get(x1, y1, x2, i));
	}
	return f[x1][y1][x2][y2][k] = t;
}
int main()
{
	scanf("%d", &n);
	for(int i = 1; i <= m; i++)
		for(int j = 1; j <= m; j++)
			scanf("%d", &s[i][j]);
	for(int i = 1; i <= m; i++)
		for(int j = 1; j <= m; j++)
			s[i][j] += s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];
	memset(f, -1, sizeof(f));
	X = ( double )s[m][m] / n;
	printf("%.3lf\n", sqrt(dp(1, 1, m, m, 1) / n));
	return 0;
}
```

下面还有一个维护平方和的代码，也有相应的题目，是[洛谷P1436](https://www.luogu.com.cn/problem/P1436)。

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 20, M = 10;
int n, m = 8;
int s[M][M];
ll f[M][M][M][M][N];
ll x;
ll get(int x1, int y1, int x2, int y2)
{
	ll delta = s[x2][y2] - s[x1 - 1][y2] - s[x2][y1 - 1] + s[x1 - 1][y1 - 1];
	return delta * delta;
}
ll dp(int x1, int y1, int x2, int y2, int k)
{
	if(f[x1][y1][x2][y2][k] >= 0)return f[x1][y1][x2][y2][k];
	if(k == n)return f[x1][y1][x2][y2][k] = get(x1, y1, x2, y2);
	ll t = 1e15;
	for(int i = x1; i < x2; i++)
	{
		t = min(t, dp(x1, y1, i, y2, k + 1) + get(i + 1, y1, x2, y2));
		t = min(t, dp(i + 1, y1, x2, y2, k + 1) + get(x1, y1, i, y2));
	}
	for(int i = y1; i < y2; i++)
	{
		t = min(t, dp(x1, y1, x2, i, k + 1) + get(x1, i + 1, x2, y2));
		t = min(t, dp(x1, i + 1, x2, y2, k + 1) + get(x1, y1, x2, i));
	}
	return f[x1][y1][x2][y2][k] = t;
}
int main()
{
	scanf("%d", &n);
	for(int i = 1; i <= m; i++)
		for(int j = 1; j <= m; j++)
			scanf("%d", &s[i][j]);
	for(int i = 1; i <= m; i++)
		for(int j = 1; j <= m; j++)
			s[i][j] += s[i - 1][j] + s[i][j - 1] - s[i - 1][j - 1];
	memset(f, -1, sizeof(f));
	printf("%lld\n", dp(1, 1, m, m, 1));
	return 0;
}
```
