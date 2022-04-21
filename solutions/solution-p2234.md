---
title: P2234 营业额统计 题解
date: 2022-01-08
tags:
	- 题解
	- STL
categories:
	题解
mathjax: true
---

Luogu P2234
LibreOJ #10143

HNOI 2002

<!--more-->
----

对于这种题目，我们考虑使用STL大法。

我们先插入两个哨兵，以防~~不会用`.begin()`和`.end()`~~越界。
这里把哨兵的值设置为了$\pm 1 \times 10^7$。

然后，边读入边判断。
如果这个数已经有过了，那么就直接+0就可以了。
如果这个数还没有出现过，那么将其加入set内，并查询其前驱与后继。
之后，取其与其前驱与后继值之差的较小值，并加入总和之中。
当然，如果其前驱与后继均为哨兵，那么就代表这个数是第一个，直接输出这个值。

代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
int tot = 0;
set<int>his;//历史记录
set<int>::iterator it;
int n;
int main()
{
	scanf("%d", &n);
	his.insert(-1000000), his.insert(1000000);//插入哨兵
	for(int i = 1; i <= n; i++)
	{
		int k;
		scanf("%d", &k);
		if(his.count(k))continue;//这个数出现过，直接跳过后续判断
		his.insert(k);
		int minn = 200020;
		it = his.find(k);
		it++;
		minn = min(minn, abs(*it - k));//这个数与其前驱之差
		it--, it--;
		minn = min(minn, abs(*it - k));//这个数与其后继之差
		tot += (minn>200000?k:minn);//是否为第一个数
	}
	printf("%lld\n", tot);
	return 0;
}
```

$\color[rgb]{1,1,0.0625}{φ}$

