---
title: "S2OJ #1551. 循环传送带 题解"
date: 2022-09-14
tags:
	- 题解
	- 构造
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">循环传送带</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1551">S2OJ #1551</a></li></ul></div>

----

# 题目简述

题目说是有一个长度为 $n$ 数组，我们需要借助一个「传送带」来把这个数组排好序。

传送带支持3种操作：

1. 将某些数字移上传送带。
2. 将某些数字移下传送带。
3. 等待传送带向后移动任意格。

假设我们这三种操作一共做了 $m$ 次，总共移动数字 $s$ 次。
题目对于 $m$ 和 $s$ 做了一些限制。
限制分两种，一种是 $s=2n$，另一种是 $m=64$。

# 移动数字次数受限

此处的限制是 $s=2n$，这意味着我们只能够将每一个数字移上移下一个来回。
同时 $m$ 的数量也不是很充裕，所以我们需要找到一个等待时间方案，使得每一个移动的距离都可以转化为一段区间的和。
考虑将所有移动的距离转化为 $ax+b$ 的形式，其中的 $a$ 和 $b$ 的最大值不超过 $\frac{m}{4}$，那么我们就可以构造出来一个合适的移动时间序列使得所有移动的距离都可以转化成为一段区间的和。
考虑使得 $x = \sqrt{n}$，此时 $x \leq 316$，可以满足我们 $x \leq \frac{m}{4}$ 的条件。

我们构造出来的移动时间序列是这样的：

$$
\underbrace{1,1,1,\cdots,1}\_{x \text{ of}},\overbrace{x,x,x,\cdots,x}^{x \text{ of}}
$$

这样我们通过将一个移动的距离转化成 $ax+b$ 的形式，可以将每一个移动距离进一步映射到 $[x-b,x+a]$ 的这样一个区间。

代码如下：

``` cpp 数字次数受限
void solve1()
{
	int k = sqrt(n) + 1;
	vector<int>in[400], out[400];
	for(int i = 0; i < n; i++)
	{
		if(a[i] == i)continue;
		int dis = (a[i] - i + n) % n;
		int x = dis / k, y = dis % k;
		in[k - y].push_back(a[i]);
		out[x].push_back(a[i]);
	}
	int tot = 2 * k;
	for(int i = 0; i <= k; i++)
	{
		if(!in[i].empty())tot++;
		if(!out[i].empty())tot++;
	}
	printf("%d\n", tot);
	for(int i = 0; i <= k; i++)
	{
		if(!in[i].empty())
		{
			printf("1 %d", in[i].size());
			for(auto v : in[i])
				printf(" %d", v);
			putchar('\n');
		}
		if(i != k)printf("3 1\n");
	}
	for(int i = 0; i <= k; i++)
	{
		if(!out[i].empty())
		{
			printf("2 %d", out[i].size());
			for(auto v : out[i])
				printf(" %d", v);
			putchar('\n');
		}
		if(i != k)printf("3 %d\n", k);
	}
}
```

# 操作次数受限

因为 $m=64$，所以我们考虑将每一个移动距离换一个拆分方式。
而 $s$ 到是十分充裕，所以我们考虑将拆分方式换为我们熟悉的二进制拆分。
$\log_2{n} \leq 16$，正好满足我们的条件。

那么就是熟悉的二进制拆分了。
因为套路十分常见，所以就直接放代码了。

``` cpp 操作次数受限
void solve2()
{
	unordered_set<int>bit[20];
	vector<int>in[20], out[20];
	int top = log2(n) + 1;
	for(int i = 0; i < n; i++)
	{
		if(a[i] == i)continue;
		int dis = (a[i] - i + n) % n;
		for(int k = 0; dis != 0; k++)
		{
			if(dis & 1)bit[k].insert(a[i]);
			dis >>= 1;
		}
	}
	if(!bit[0].empty())
	{
		for(auto v : bit[0])
			in[0].push_back(v);
	}
	for(int i = 1; i <= top; i++)
	{
		if(bit[i].empty())
		{
			if(!bit[i - 1].empty())
			{
				for(auto v : bit[i - 1])
					out[i - 1].push_back(v);
			}
		}
		else
		{
			if(bit[i - 1].empty())
			{
				for(auto v : bit[i])
					in[i].push_back(v);
			}
			else
			{
				for(auto v : bit[i - 1])
				{
					if(!bit[i].count(v))
						out[i - 1].push_back(v);
				}
				for(auto v : bit[i])
				{
					if(!bit[i - 1].count(v))
						in[i].push_back(v);
				}
			}
		}
	}
	int tot = top;
	for(int i = 0; i < top; i++)
	{
		if(!in[i].empty())tot++;
		if(!out[i].empty())tot++;
	}
	printf("%d\n", tot);
	for(int i = 0; i < top; i++)
	{
		if(!in[i].empty())
		{
			printf("1 %d", in[i].size());
			for(auto v : in[i])
				printf(" %d", v);
			putchar('\n');
		}
		printf("3 %d\n", (1 << i));
		if(!out[i].empty())
		{
			printf("2 %d", out[i].size());
			for(auto v : out[i])
				printf(" %d", v);
			putchar('\n');
		}
	}
}
```

# 全部代码

注意要先输出总操作次数。

``` cpp
#include<bits/stdc++.h>
#include<unordered_set>
using namespace std;
#define ll long long
const int N = 100010;
int n;
int m, s;
int a[N];
void solve1()
{
	int k = sqrt(n) + 1;
	vector<int>in[400], out[400];
	for(int i = 0; i < n; i++)
	{
		if(a[i] == i)continue;
		int dis = (a[i] - i + n) % n;
		int x = dis / k, y = dis % k;
		in[k - y].push_back(a[i]);
		out[x].push_back(a[i]);
	}
	int tot = 2 * k;
	for(int i = 0; i <= k; i++)
	{
		if(!in[i].empty())tot++;
		if(!out[i].empty())tot++;
	}
	printf("%d\n", tot);
	for(int i = 0; i <= k; i++)
	{
		if(!in[i].empty())
		{
			printf("1 %d", in[i].size());
			for(auto v : in[i])
				printf(" %d", v);
			putchar('\n');
		}
		if(i != k)printf("3 1\n");
	}
	for(int i = 0; i <= k; i++)
	{
		if(!out[i].empty())
		{
			printf("2 %d", out[i].size());
			for(auto v : out[i])
				printf(" %d", v);
			putchar('\n');
		}
		if(i != k)printf("3 %d\n", k);
	}
}
void solve2()
{
	unordered_set<int>bit[20];
	vector<int>in[20], out[20];
	int top = log2(n) + 1;
	for(int i = 0; i < n; i++)
	{
		if(a[i] == i)continue;
		int dis = (a[i] - i + n) % n;
		for(int k = 0; dis != 0; k++)
		{
			if(dis & 1)bit[k].insert(a[i]);
			dis >>= 1;
		}
	}
	if(!bit[0].empty())
	{
		for(auto v : bit[0])
			in[0].push_back(v);
	}
	for(int i = 1; i <= top; i++)
	{
		if(bit[i].empty())
		{
			if(!bit[i - 1].empty())
			{
				for(auto v : bit[i - 1])
					out[i - 1].push_back(v);
			}
		}
		else
		{
			if(bit[i - 1].empty())
			{
				for(auto v : bit[i])
					in[i].push_back(v);
			}
			else
			{
				for(auto v : bit[i - 1])
				{
					if(!bit[i].count(v))
						out[i - 1].push_back(v);
				}
				for(auto v : bit[i])
				{
					if(!bit[i - 1].count(v))
						in[i].push_back(v);
				}
			}
		}
	}
	int tot = top;
	for(int i = 0; i < top; i++)
	{
		if(!in[i].empty())tot++;
		if(!out[i].empty())tot++;
	}
	printf("%d\n", tot);
	for(int i = 0; i < top; i++)
	{
		if(!in[i].empty())
		{
			printf("1 %d", in[i].size());
			for(auto v : in[i])
				printf(" %d", v);
			putchar('\n');
		}
		printf("3 %d\n", (1 << i));
		if(!out[i].empty())
		{
			printf("2 %d", out[i].size());
			for(auto v : out[i])
				printf(" %d", v);
			putchar('\n');
		}
	}
}
int main()
{
	scanf("%d", &n);
	scanf("%d%d", &m, &s);
	for(int i = 0; i < n; i++)
	{
		scanf("%d", &a[i]);
	}
	if(m >= 2048)solve1();
	else solve2();
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>