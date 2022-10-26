---
title: "S2OJ #1550. 假冒泡排序 题解"
date: 2022-09-14
tags:
	- 题解
	- S2OJ
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">假冒泡排序</div>
<div id="problem-info-from">none</div>
<div id="problem-info-difficulty">none</div>
<div id="problem-info-color"></div>
<div id="problem-info-submit"><ul><li><a href="https://sjzezoj.com/problem/1550">S2OJ #1550</a></li></ul></div>

----

题目的意思是，给定了一个1到n的排列，我们对其进行冒泡排序，但是每一次遍历一遍数组之后都会将数组中的最后一个数字与第一个数字进行比较，如果最后一个数字比第一个数字大的话就将两者交换位置。
问在冒泡排序的过程中可能出现的本质不同的版本的数量。

由于我们题目中的冒泡排序是顺序遍历的数组，所以我们在第一次遇到n的时候，冒泡排序实际上已经停止了，后面的动作只不过是将n从头搬到尾，然后将第一个数字放到最后罢了。

所以我们在第一次遇到n之前的所有交换都会产生一个本质不同的版本。

然后就是不断将n从头搬到尾的过程。
每这样循环一轮，其他元素中位置最靠前的元素都会被放到尾部。
在移动n的过程中，其他的元素顺序不变。
所以最终会产生 $n(n-1)$ 个不同的版本。

当然，这道题不会这么简单。
我们还会注意到一个地方，就是一种特殊情况。
当我们 $a_1=n-1$ 且 $a_n=n$ 的时候，我们最后一种情况会与一开始的这种情况重合，所以需要减去这种情况。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010;
int n;
int a[N], tmp[N];
int main()
{
	scanf("%d", &n);
	for(int i = 1; i <= n; i++)
	{
		scanf("%d", &a[i]);
		tmp[i] = a[i];
	}
	ll ans = 0;
	for(int i = 1; i < n; i++)
	{
		if(a[i] == n)break;
		if(a[i] > a[i + 1])
		{
			ans++;
			swap(a[i], a[i + 1]);
		}
	}
	ans += 1ll * n * (n - 1);
	ans -= (n >= 3 && tmp[1] == n - 1 && tmp[n] == n);
	printf("%lld\n", ans);
	return 0;
}
```

<script>
	getProblemCardInfo();
</script>