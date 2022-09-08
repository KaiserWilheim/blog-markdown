---
title: P4135 作诗 题解
date: 2022-08-22
tags:
	- 题解
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">作诗</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P4135">Luogu P4135</a></li><li><a href="https://loj.ac/p/3277">LibreOJ L6180</a></li></ul></div>

----

对于这道题，我们首先想到的就是分块了，因为看起来只能暴力做，没有什么好用的数据结构来维护。
我们对于每一个整段求出每一个颜色在其中的出现次数，每一次U型你问的时候只需要将其累加起来再统计即可。

但是我们颜色的种类数 $c$ 是与 $n$ 同阶的，我们枚举的时候难免会带一个较大的复杂度，达不到 $O(\sqrt{n})$ 的复杂度要求。

考虑不考虑这些整块内的颜色，只考虑散块内的颜色对整块内的答案的影响。因为散块内的颜色最多也只有 $O(\sqrt{n})$ 种。这样，我们需要枚举的颜色也就不超过 $O(\sqrt{n})$ 种了。

我们记录每一块的颜色后缀和 $cnt[bl][color]$，然后再记录一个整块形成的段的答案 $f[tl][tr]$。

统计的时候，假如说我们统计的块是从 $l$ 到 $r$ 的一段区间，而 $tl$ 和 $tr$ 分别是 $l$ 和 $r$ 所在的两个散块。
那么我们暴力统计出来 $tl$ 和 $tr$ 这两个散块中的颜色和 $num[color]$，同时拿一个什么东西记录下来所有出现过的颜色（我这里是用栈），然后拿出来 $f[tl][tr]$ 作为基准的答案。
对于每一个在散块内出现过的颜色，我们分六类讨论。
1. 该颜色在散块内出现奇数次，在整块内没有出现。
	此时该颜色对答案无影响。
2. 该颜色在散块内出现偶数次，在整块内没有出现。
	此时该颜色对答案有1的贡献。
3. 该颜色在散块内出现奇数次，在整块内出现奇数次。
	此时该颜色对答案有1的贡献。
4. 该颜色在散块内出现偶数次，在整块内出现奇数次。
	此时该颜色对答案没有贡献。
5. 该颜色在散块内出现奇数次，在整块内出现偶数次。
	此时该颜色对答案有-1的贡献。
6. 该颜色在散块内出现偶数次，在整块内出现偶数次。
	此时该颜色对答案没有贡献。

这六种情况中只有三种对答案有贡献，只需要判断是否满足条件即可。

前期处理的时间复杂度也不高，如果方法得当的话就是 $O(n \sqrt{n})$ 的，不会对后面的询问产生影响。

参考代码：

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = 440;
int n, m;
int c;
int S;
int lastans = 0;
int a[N];
int bl[N];
int cnt[M][N];
int f[M][M];
int main()
{
	scanf("%d%d%d", &n, &c, &m);
	S = sqrt(n);
	for(int i = 0; i < n; i++)
	{
		scanf("%d", &a[i]);
		bl[i] = i / S;
	}
	bl[n] = n;
	for(int i = 0; i <= bl[n - 1]; i++)
	{
		int t = 0;
		for(int j = i * S; j < n; j++)
		{
			cnt[i][a[j]]++;
			if(cnt[i][a[j]] & 1 && cnt[i][a[j]] > 1)t--;
			else if((cnt[i][a[j]] % 2) == 0)t++;
			if(bl[j] != bl[j + 1])f[i][bl[j]] = t;
		}
	}
	int num[N], sta[N], tt;
	memset(num, 0, sizeof(num));
	while(m--)
	{
		int l, r;
		scanf("%d%d", &l, &r);
		l = (l + lastans) % n;
		r = (r + lastans) % n;
		if(l > r)swap(l, r);
		int tl = bl[l], tr = bl[r];
		if(tl == tr)
		{
			tt = 0;
			for(int i = l; i <= r; i++)
			{
				num[a[i]]++;
				if(num[a[i]] == 1)sta[++tt] = a[i];
			}
			int res = 0;
			for(int i = 1; i <= tt; i++)
			{
				int k = sta[i];
				if(num[k] % 2 == 0)res++;
				num[k] = 0;
			}
			lastans = res;
			printf("%d\n", res);
		}
		else
		{
			tt = 0;
			int res = 0;
			if(tl + 1 <= tr - 1)res += f[tl + 1][tr - 1];
			for(int i = l; i < (tl + 1) * S; i++)
			{
				num[a[i]]++;
				if(num[a[i]] == 1)sta[++tt] = a[i];
			}
			for(int i = tr * S; i <= r; i++)
			{
				num[a[i]]++;
				if(num[a[i]] == 1)sta[++tt] = a[i];
			}
			for(int i = 1; i <= tt; i++)
			{
				int k = sta[i];
				if(num[k] % 2 == 1 && (cnt[tl + 1][k] - cnt[tr][k]) % 2 == 0 && (cnt[tl + 1][k] - cnt[tr][k]) > 0)res--;
				if(num[k] % 2 == 1 && (cnt[tl + 1][k] - cnt[tr][k]) % 2 == 1)res++;
				if(num[k] % 2 == 0 && (cnt[tl + 1][k] - cnt[tr][k]) == 0)res++;

				num[k] = 0;
			}
			lastans = res;
			printf("%d\n", res);
		}
	} 
	return 0;
}
```

