---
title: P3188 [HNOI2007] 梦幻岛宝珠 题解
date: 2022-07-10
tags:
	- 题解
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">梦幻岛宝珠</div>
<div id="problem-info-from">HNOI 2007</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3188">Luogu P3188</a></li><li><a href="https://www.acwing.com/problem/content/2149/">AcWing 2147</a></li><li><a href="https://darkbzoj.cc/problem/1190">BZOJ #1190</a></li></ul></div>

----

题目很明显是一个背包问题，但是其背包容量很大，是 $10^{10}$ 级别的。
这样明显不是普通背包算法可以应付得来的，但我们又不能不用背包来解决。
一定有什么地方可以优化。

我们观察题目给的条件，发现题目保证 $\sum n \leq 2000$，且每一个物品的重量都可以拆分为 $a \times 2^b$ 的形式，其中 $a \leq 10,b \leq 30$。

这样我们就可以将每一个物品按照 $b$ 来分组，对于每一个组内我们跑一个背包，然后再把这些组再跑一个背包就可以了。

对于每一组内跑背包的时候，我们背包的容量就是当前这一组内物品个数的十倍，因为所有的 $a$ 都不超过10，所以 $\sum a$ 肯定也是小于当前组内物品个数10倍的。

做组间背包的时候，我们从低位到高位做。
我们想，每一次我们都从当前位的容量内拿出来 $k$ 去给下面的部分做背包。对于剩余的容量做完背包之后，我们就将下面对容量为 $k$ 的背包做出来的结果直接加上来即可。

具体实现的时候，我们定义两个二维数组 $f$ 和 $g$。

$f[i][k]$ 代表的是对从低到高第 $i$ 位（或者说 $b=i$）的物品进行容量为 $k$ 的背包得出的结果。

求 $f$ 数组的时候我们就直接按照正常背包做就可以了。

而 $g[i][j]$ 代表的就是当使用 $b$ 为 $i$ 的物品进行容量为 $j$ 的背包时，$b$ 小于 $i$ 的所有物品占满了 $W$ 的后 $i$ 位所代表的容量的值。

举个例子，我们最后要输出的结果就是 $g[\lfloor \log_2{W} \rfloor][1]$ 的值。而这个代表我们对最高位上做背包时，背包容量为1（也只能是1），同时除了最高位以外的所有位代表的容量都被占满了的时候的最大值。

而在更新 $g$ 数组的时候，我们遵循下面的转移方程：

$$
g[i][j] = \max(g[i][j],f[i][j-k]+g[i-1][\min(10n,2k+(\frac{W}{2^{i-1}} \\& 1))])
$$

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 110, M = 40;
int n, c;
int w[N], v[N];
vector<pair<int, int> >h[M];
int f[M][N * M];
int g[M][N * M];
int lowbit(int x)
{
	return x & -x;
}
void init()
{
	memset(w, 0, sizeof(w));
	memset(v, 0, sizeof(v));
	memset(f, 0, sizeof(f));
	memset(g, 0, sizeof(g));
	for(int i = 0; i < M; i++)h[i].clear();
}
int main()
{
	scanf("%d%d", &n, &c);
	while(n != -1 && c != -1)
	{
		init();
		for(int i = 1; i <= n; i++)
		{
			scanf("%d%d", &w[i], &v[i]);
			int b = log2(lowbit(w[i]));
			h[b].push_back(make_pair(w[i] >> b, v[i]));
		}
		int cnt = log2(c);

		for(int t = 0; t <= cnt; t++)
			for(int i = 0; i < h[t].size(); i++)
				for(int j = 10 * h[t].size(); j >= h[t][i].first; j--)
					f[t][j] = max(f[t][j], f[t][j - h[t][i].first] + h[t][i].second);
		//给每一组内跑背包
		for(int i = 0; i <= 10 * h[0].size(); i++)
			g[0][i] = f[0][i];
		for(int t = 1; t <= cnt; t++)
			for(int j = 0; j <= 10 * n; j++)
				for(int i = 0; i <= j; i++)
					g[t][j] = max(g[t][j], f[t][j - i] + g[t - 1][min(10 * n, i * 2 + ((c >> (t - 1)) & 1))]);
		//跑组与组之间的背包
		printf("%d\n", g[cnt][1]);
		scanf("%d%d", &n, &c);
	}
	return 0;
}
```

