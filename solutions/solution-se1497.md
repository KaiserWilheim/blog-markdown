---
title: "S2OJ #1497. 树 题解"
date: 2022-08-26
tags:
	- 题解
	- STL
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">树</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1497">S2OJ #1497</a></li></ul></div>

----

题目要求是“任意排列后可以形成回文串”，那么就是路径上最多只有一个颜色出现奇数次。
那我们可以将这个改为“成对的相同颜色可以消除对方的影响”，由此可以想到异或操作。

我们就可以每一次想办法搞出来链上的颜色异或和，看看最后结果是否为0或者在路径上出现过。
同时利用异或的特性，我们直接DFS一遍取每一个点到根的异或和，然后每一次询问都取出来两端点异或一下就是结果。

我们可以直接给每种颜色随一个权值，然后尝试卡过去。

因为颜色数量很多，所以我们这里选择`mt19937_64`来作为随机数生成器，生成的随机数是`unsigned long long`类型的。

同时为了保证时间小，我们直接用`std::set`来存所有出现过的权值，然后直接判断就可以了

这个方法的正确性不一定能够得到保证，但是碰到的几率太小了。
该方法发生错误，只能当且仅当有三个颜色 $a$，$b$ 和 $c$ 满足 $a \oplus b = c$ 且 $a$ 与 $b$ 两种颜色同时出现在同一条路径上，且出现次数为奇数才会发生碰撞导致结果错误。

下面是代码：

``` cpp
#include<bits/stdc++.h>
#include<random>
#include<chrono>
#define ull unsigned long long
using namespace std;
const int N = 1000010, M = N << 1;
const int L = 1010;
int n, m;
int h[N], e[M], ne[M], idx;
ull w[M];
void add(int a, int b, ull c)
{
	e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}
ull sum[N];
map<int, ull>dic;
set<ull>s;
void dfs(int p, int fa, ull now)
{
	sum[p] = sum[fa] ^ now;
	for(int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if(j == fa)continue;
		dfs(j, p, w[i]);
	}
}
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%d%d", &n, &m);
	mt19937_64 rand(chrono::system_clock::now().time_since_epoch().count());
	for(int i = 1; i < n; i++)
	{
		int u, v, w;
		scanf("%d%d%d", &u, &v, &w);
		if(!dic.count(w))
		{
			dic[w] = rand();
			s.insert(dic[w]);
		}
		ull k = dic[w];
		add(u, v, k), add(v, u, k);
	}
	dfs(1, 0, 0);
	long long a, b;
	scanf("%lld%lld", &a, &b);
	int ans = 0;
	while(m--)
	{
		int x = a % n + 1, y = b % n + 1;
		ull res = sum[x] ^ sum[y];
		if(res == 0 || s.count(res))ans++;
		a = (a * 666073) % 1000000007;
		b = (b * 233) % 998244353;
	}
	printf("%d\n", ans);
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>