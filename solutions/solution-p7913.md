---
title: P7913 廊桥分配 题解
date: 2021-10-25
tags:
	- 题解
categories:
 题解
mathjax: true
---

Luogu P7913

2021 CSP-S T1

<!--more-->
----

这道题可以使用这样一种模拟的做法：

我们首先假定我们有无限多的廊桥可以使用。

每当一架飞机降落，我们就寻找可用的廊桥。

如果有多个可用的，我们选择编号最小的那个；而当没有可用的的时候，我们就新开一个廊桥（实际上思想与上一句类似）。

当所有的飞机全部安排好了之后，我们就记录下每一个廊桥接待了多少架飞机。

最后进行比较，求出所有分配情况下的最大接待数。

十分简单。

就拿样例2举个栗子：

```cpp
2 4 6
20 30
40 50
21 22
41 42
1 19
2 18
3 4
5 6
7 8
9 10
```

（这里进行了一下排序，但是由于样例2是已经排好序了的，所以没什么作用）

模拟过程：

{% note info 模拟过程 %}

**国内**航班降落于 $20$ ，编为 $\sharp 1$ ，将于 $30$ 时起飞。
廊桥 $\sharp 1$ 目前处于空闲状态，上一次使用结束时间为 $0$ 。
飞机 $\sharp 1$ 进入廊桥 $\sharp 1$ 。
**国内**航班降落于 $40$ ，编为 $\sharp 2$ ，将于 $50$ 时起飞。
廊桥 $\sharp 1$ 目前处于空闲状态，上一次使用结束时间为 $30$ 。
飞机 $\sharp 2$ 进入廊桥 $\sharp 1$ 。
**国内**航班降落于 $21$ ，编为 $\sharp 3$ ，将于 $22$ 时起飞。
廊桥 $\sharp 1$ 目前正在被使用，直至 $51$ 时才会空闲。
目前无空闲廊桥。新建廊桥 $\sharp 2$ 。
飞机 $\sharp 3$ 进入廊桥 $\sharp 2$ 。
**国内**航班降落于 $41$ ，编为 $\sharp 4$ ，将于 $42$ 时起飞。
廊桥 $\sharp 1$ 目前正在被使用，直至 $51$ 时才会空闲。
廊桥 $\sharp 2$ 目前处于空闲状态，上一次使用结束时间为 $22$ 。
飞机 $\sharp 4$ 进入廊桥 $\sharp 2$ 。
**国际**航班降落于 $1$ ，编为 $\sharp 1$ ，将于 $19$ 时起飞。
廊桥 $\sharp 1$ 目前处于空闲状态，上一次使用结束时间为 $0$ 。
飞机 $\sharp 1$ 进入廊桥 $\sharp 1$ 。
**国际**航班降落于 $2$ ，编为 $\sharp 2$ ，将于 $18$ 时起飞。
廊桥 $\sharp 1$ 目前正在被使用，直至 $20$ 时才会空闲。
目前无空闲廊桥。新建廊桥 $\sharp 2$ 。
飞机 $\sharp 2$ 进入廊桥 $\sharp 2$ 。
**国际**航班降落于 $3$ ，编为 $\sharp 3$ ，将于 $4$ 时起飞。
廊桥 $\sharp 1$ 目前正在被使用，直至 $20$ 时才会空闲。
廊桥 $\sharp 2$ 目前正在被使用，直至 $19$ 时才会空闲。
目前无空闲廊桥。新建廊桥 $\sharp 3$ 。
飞机 $\sharp 3$ 进入廊桥 $\sharp 3$ 。
**国际**航班降落于 $5$ ，编为 $\sharp 4$ ，将于 $6$ 时起飞。
廊桥 $\sharp 1$ 目前正在被使用，直至 $20$ 时才会空闲。
廊桥 $\sharp 2$ 目前正在被使用，直至 $19$ 时才会空闲。
廊桥 $\sharp 3$ 目前处于空闲状态，上一次使用结束时间为 $4$ 。
飞机 $\sharp 4$ 进入廊桥 $\sharp 3$ 。
**国际**航班降落于 $7$ ，编为 $\sharp 5$ ，将于 $8$ 时起飞。
廊桥 $\sharp 1$ 目前正在被使用，直至 $20$ 时才会空闲。
廊桥 $\sharp 2$ 目前正在被使用，直至 $19$ 时才会空闲。
廊桥 $\sharp 3$ 目前处于空闲状态，上一次使用结束时间为 $6$ 。
飞机 $\sharp 5$ 进入廊桥 $\sharp 3$ 。
**国际**航班降落于 $9$ ，编为 $\sharp 6$ ，将于 $10$ 时起飞。
廊桥 $\sharp 1$ 目前正在被使用，直至 $20$ 时才会空闲。
廊桥 $\sharp 2$ 目前正在被使用，直至 $19$ 时才会空闲。
廊桥 $\sharp 3$ 目前处于空闲状态，上一次使用结束时间为 $8$ 。
飞机 $\sharp 6$ 进入廊桥 $\sharp 3$ 。
**国内**廊桥 $\sharp 1$ 曾被 $2$ 架飞机使用，包括**国内**航班  $\sharp 1$ 和 $\sharp 2$ 。
**国内**廊桥 $\sharp 2$ 曾被 $2$ 架飞机使用，包括**国内**航班  $\sharp 3$ 和 $\sharp 4$ 。
**国际**廊桥 $\sharp 1$ 曾被 $1$ 架飞机使用，包括**国际**航班  $\sharp 1$ 。
**国际**廊桥 $\sharp 2$ 曾被 $1$ 架飞机使用，包括**国际**航班  $\sharp 2$ 。
**国际**廊桥 $\sharp 3$ 曾被 $4$ 架飞机使用，包括**国际**航班  $\sharp 3$ ， $\sharp 4$ ， $\sharp 5$ and $\sharp 6$ 。
分配 $0$ 个廊桥给**国内**航班，同时分配 $2$ 个给**国际**航班。 $0+2=2$ 。 结果更优。
现在的最大值是 $2$ 架。
分配 $1$ 个廊桥给**国内**航班，同时分配 $1$ 个给**国际**航班。 $2+1=3$ 。 结果更优。
现在的最大值是 $3$ 架。
分配 $2$ 个廊桥给**国内**航班，同时分配 $0$ 个给**国际**航班。 $4+0=4$ 。 结果更优。
现在的最大值是 $4$ 架。

{% endnote %}

然后我们再看一下样例3的模拟过程~~（如果你愿意的话）~~：

[样例3.docx](/example3.docx) 

我们可以清楚的看到，这样的模拟方法是可行的。

所以这里有一个样板程序：

{% note success 40分程序 %}

``` cpp
#pragma GCC optimize(2)
#include <bits/stdc++.h>
using namespace str;
const int N = 100010;
int patri[N], inter[N], ptnum[N], innum[N];
struct plane
{
	int st, ed;
}ptpl[N], inpl[N];
int idxpt = 1, idxin = 1;
int main()
{
	freopen("airport.in", "r", stdin);
	freopen("airport.out", "w", stdout);
	int n, pt, in;
	cin >> n >> pt >> in;
	for(int i = 1; i <= pt; i++)
	{
		cin >> ptpl[i].st >> ptpl[i].ed;
	}
	for(int i = 1; i <= in; i++)
	{
		cin >> inpl[i].st >> inpl[i].ed;
	}
	for(int i = 1; i <= pt; i++)
	{
		for(int j = i; j > 1; j--)
		{
			if(ptpl[j].st < ptpl[j - 1].st)
			{
				plane temp = ptpl[j];
				ptpl[j] = ptpl[j - 1];
				ptpl[j - 1] = temp;
			}
		}
	}
	for(int i = 1; i <= in; i++)
	{
		for(int j = i; j > 1; j--)
		{
			if(inpl[j].st < inpl[j - 1].st)
			{
				plane temp = inpl[j];
				inpl[j] = inpl[j - 1];
				inpl[j - 1] = temp;
			}
		}
	}
	for(int i = 1; i <= pt; i++)
	{
		bool flag = true;
		for(int j = 1; j <= idxpt; j++)
		{
			if(patri[j] < ptpl[i].st)
			{
				patri[j] = ptpl[i].ed;
				ptnum[j]++;
				flag = false;
				break;
			}
		}
		if(flag)
		{
			idxpt++;
			patri[idxpt] = ptpl[i].ed;
			ptnum[idxpt]++;
		}
	}
	for(int i = 1; i <= in; i++)
	{
		bool flag = true;
		for(int j = 1; j <= idxin; j++)
		{
			if(inter[j] < inpl[i].st)
			{
				inter[j] = inpl[i].ed;
				innum[j]++;
				flag = false;
				break;
			}
		}
		if(flag)
		{
			idxin++;
			inter[idxin] = inpl[i].ed;
			innum[idxin]++;
		}
	}
	for(int i = 1; i <= n; i++)ptnum[i] += ptnum[i - 1];
	for(int i = 1; i <= n; i++)innum[i] += innum[i - 1];
	int maxn = 0;
	for(int i = 0; i <= n; i++)
	{
		maxn = max(maxn, ptnum[i] + innum[n - i]);
	}
	cout << maxn << endl;
	return 0;
}
```
{% endnote %}
$\color[rgb]{1,1,0.0625}{φ}$

但是这样的时间复杂度太高，我们只能过8个点，得40分~~（不过40分好像也能进NOIP）~~。

这个算法的排序部分使用插入排序，时间复杂度为 $O(n^2)$ 级别，模拟部分时间复杂度为 $O(n^2)$ 级别，最后的差分序列计算和结果比较都是 $O(n)$ 级别的，所以这个算法的时间复杂度大概是 $O(n^2)$ 级别的，大概会在 $n=1e4$ 左右的时候超时。

我们这个时候看一下数据：$n \in [1,1e5]$ 。

绝对会超时。

（同时我们看一下第八和第九个点的数据：$n_8=3374,pt_8=2239,in_8=2756$ ； $n_9=68288,pt_9=37493,in_9=62505$ 。果然超时）

我们尝试优化一下：

我们可以使用 $set$ 来替代我们的插入排序：

同时，因为失去了数据之间的关联性，我们使用结构体来存储数据，这导致我们需要重载运算符来保证只按照降落时间来排序。

``` cpp
......
struct plane
{
	int st, ed;
	bool operator < (const plane &a) const { return a.st < st; }
};
......
int main()
{
    ......
    set<plane> ptpl;
	set<plane> inpl;
	set<plane>::iterator it;
    ......
}
```



同时，因为 $set$ 它排序的时候是按照从大到小的原则来排序的，所以我们需要稍微改一下运算符，使其按照从小到大的顺序排列。

上代码：
{% note success 我的最终解 %}
``` cpp
#include <bits/stdc++.h>
using namespace str;
const int N = 100010;
int patri[N], inter[N], ptnum[N], innum[N];
struct plane
{
	int st, ed;
	bool operator < (const plane &a) const { return a.st > st; }
};
int idxpt = 1, idxin = 1;
int main()
{
	freopen("airport.in", "r", stdin);
	freopen("airport.out", "w", stdout);
	int n, pt, in;
	cin >> n >> pt >> in;
	set<plane> ptpl;
	set<plane> inpl;
	set<plane>::iterator it;
	for(int i = 1; i <= pt; i++)
	{
		plane x;
		cin >> x.st >> x.ed;
		ptpl.insert(x);
	}
	for(int i = 1; i <= in; i++)
	{
		plane x;
		cin >> x.st >> x.ed;
		inpl.insert(x);
	}
	for(it = ptpl.begin(); it != ptpl.end(); it++)
	{
		bool flag = true;
		for(int j = 1; j <= idxpt; j++)
		{
			if(patri[j] < (*it).st)
			{
				patri[j] = (*it).ed;
				ptnum[j]++;
				flag = false;
				break;
			}
		}
		if(flag)
		{
			idxpt++;
			patri[idxpt] = (*it).ed;
			ptnum[idxpt]++;
		}
	}
	for(it = inpl.begin(); it != inpl.end(); it++)
	{
		bool flag = true;
		for(int j = 1; j <= idxin; j++)
		{
			if(inter[j] < (*it).st)
			{
				inter[j] = (*it).ed;
				innum[j]++;
				flag = false;
				break;
			}
		}
		if(flag)
		{
			idxin++;
			inter[idxin] = (*it).ed;
			innum[idxin]++;
		}
	}
	for(int i = 1; i <= n; i++)ptnum[i] += ptnum[i - 1];
	for(int i = 1; i <= n; i++)innum[i] += innum[i - 1];
	int maxn = 0;
	for(int i = 0; i <= n; i++)
	{
		maxn = max(maxn, ptnum[i] + innum[n - i]);
	}
	cout << maxn << endl;
	return 0;
}
```
{% endnote %}
$\color[rgb]{1,1,0.0625}{φ}$

这个就快一点。

这个在洛谷的民间数据测得的[结果](https://www.luogu.com.cn/record/60944920)是AC。后面用时平均在 $800ms$ 左右。

（当时要是我在考场上写出来了的话就好了）

不过这个据说是能被卡掉的。看到时候会不会有数据来卡吧。

（2022.3.31 发现没有更新）

会[被hack掉](https://www.luogu.com.cn/record/72846486)。

但是是 100 Unaccepted。















