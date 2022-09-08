---
title: P3462 [POI2007] 砝码 ODW 题解
date: 2022-07-14
tags:
	- 题解
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">砝码ODW</div>
<div id="problem-info-from">POI 2007</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P3462">Luogu P3462</a></li><li><a href="https://loj.ac/p/2660">LibreOJ L2660</a></li><li><a href="https://www.acwing.com/problem/content/3693/">AcWing 3690</a></li><li><a href="https://darkbzoj.cc/problem/1110">BZOJ #1110</a></li></ul></div>

----

这道题要求我们将一堆砝码放进一堆容器里面，要求每个容器里面砝码的总重量不超过容器的容量。题目要求我们求出并最大化能装进容器内砝码数量。

首先我们可以确定一点，我们从最小的砝码开始装，一定是较优的。

但是题目给出的砝码总数和容器总数都是 $10^5$ 级别的，我们不能每一个砝码都搜一遍，不仅复杂度不对，还保证不了正确性。

但是，题目保证任何两个砝码，其中一个的重量必须是另一个的整数倍。
所以说，对于每一个砝码，剩余的砝码中一部分是它的整数倍，而剩余的能整除它。

这样我们就可以把这个转化为类似进制的东西，只不过不是严格的每两位之比相同的那种情况。

我们只需要找到最小的那个砝码，这就是我们进制的基数，我们设这个数为 $k_1$。
然后我们找到每一种不同的砝码重量，我们将其化为一个进制位。我们设这样的从小到大分别是 $k_2,k_3,k_4,\cdots$。

根据这一堆基数 $k$，我们将所有的容器拆成 $(a_0,a_1k_1,a_2k_2,a_3k_3,\cdots)$ 这样的形式。
因为我们所有的砝码都是 $k$ 的整数倍，所以这个 $a_0$ 就直接舍弃不要。剩下的同意一下所有容器中 $a_i$ 的和就可以了。

我们拆成这样的类 $k$ 进制了之后就不需要考虑装不下或者砝码跨容器的情况了。原因如下：

我们首先将所有的砝码从小到大排序。

然后我们从小到大放进DFS里面搜一下。
如果当前位还有剩余，就可以直接减掉。
如果没有剩余了，那就往上面搜，从前面的位借。若果借到最高位都借不到的话就输出当前砝码数量就可以了。
因为我们每一个砝码的重量一定对应一个进制位，我们就一定只需要在当前进制位为空的时候才会向上面借位，且我们不会向上进位，这样保证了我们不会让一个砝码利用两个不同容器的空间的情况。

这样我们就能在 $O(n\log{m})$ 的时间复杂度内解决这道题了。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 200010;
int n, m;
int a[N], b[N], num[N];
int c[40], cnt[40], tot;
bool dfs(int p)
{
	if(p > tot)return false;
	if(cnt[p])
	{
		cnt[p]--;
		return true;
	}
	if(dfs(p + 1))
	{
		cnt[p] += c[p + 1] / c[p];
		cnt[p]--;
		return true;
	}
	return false;
}
int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
		scanf("%d", &a[i]);
	for(int i = 1; i <= m; i++)
		scanf("%d", &b[i]);
	sort(b + 1, b + 1 + m);
	for(int i = 1; i <= m; i++)
	{
		if(b[i] != b[i - 1])c[++tot] = b[i];
		num[i] = tot;
	}
	for(int i = 1; i <= n; i++)
	{
		for(int j = tot; j; j--)
		{
			cnt[j] += a[i] / c[j];
			a[i] %= c[j];
		}
	}
	for(int i = 1; i <= m; i++)
	{
		if(!dfs(num[i]))
		{
			printf("%d\n", i - 1);
			break;
		}
		if(i == m)printf("%d\n", m);
	}
	return 0;
}
```
