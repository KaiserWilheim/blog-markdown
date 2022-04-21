---
title: 扫描线
date: 2021-11-12
tags:
- 算法
categories:
 算法
mathjax: true
---

一种便携的用来求矩形面积并的算法。

<!--more-->

# 思想

顾名思义，扫描线就是要模拟一根线，扫过整个图形。至于方向什么的自己根据喜好，这里选用从左往右的方式。

# 过程

## 朴素做法

首先我们看一下这个例子：

<img src="https://i.loli.net/2021/11/12/ykmlLC6RpHTZbUf.png" alt="扫描线1.png" width="60%" />

我们有三个矩形，分别用红、蓝、绿三色标注了出来。
他们相交的面积用其他的颜色标了出来。

我们所得到的信息只有矩形的左下端点和右上端点的坐标。也就是我们标红的这几个点：

<img src="https://i.loli.net/2021/11/12/Naw9ZzMXnA6Jt7d.png" alt="扫描线2.png" width="60%" />

然后我们可以画出一根线来，扫过这个多边形的所有面积。
这根线从最左边的边开始，每遇到一条边，就停下来计算之前扫到的面积，即为距上一条边的距离与扫描线落在图形上的长度之积，再加到之前的和上面。
用动画来做就是这样的：

<img src="https://s2.loli.net/2021/12/29/zsg32tJnWbykfBo.gif" alt="扫描线3.gif" width="60%" />

但是这样做有一处需要注意的地方： **我们怎么才能知道每一次遇到一条边之后扫描线落在图像上的长度是多少** ？
如果我们每一次都去枚举、去找，我们程序的时间复杂度就达不到我们的要求了。
这就是扫描线算法的精髓之处。

## 线段树

### 分段

我们把 $y$ 轴上的一段区间按照每一个存在的矩形的边的高度值分一下块：

<img src="https://i.loli.net/2021/11/12/pkELge24OblfDxI.png" alt="扫描线4.png" width="60%" />

我们只需要计算每一段所对应的在 $x$ 轴上的长度就行了。

我们可以使用线段树这一强大的数据结构来帮助我们更快地进行区间修改。

<img src="https://i.loli.net/2021/11/12/W3OLxhVenapUIfH.png" alt="扫描线5.png" width="60%" />

### 标记

我们在遇到某个矩形的一条边的时候，我们需要看一下这两个东西：

1. 它的长度；
2. 他是起始边还是终止边。

如果是起始边的话，我们把他这条边所包含的所有区块都打上一个标记；如果是终止边的话，我们就把之前起始边打上的标记去掉。这样，只要有标记就是存在，不管有多少个；没有标记就是没有被覆盖，忽略不计。
所以，我们的线段树节点需要存储这个区段被标记的次数。同时，我们还需要存储基本的线段树信息，还有它的长度。
同时，我们用来存储矩形边的位置我们也需要改一下，使之能够存储这条边的上端点和下端点，还要标记这条边是起始边还是终止边。
最终我们使用动画模拟一下就是这个样子的：

<img src="https://s2.loli.net/2021/12/29/UEmq4XVFOWL56eo.gif" alt="扫描线6.gif" width="60%" />

# 代码

上代码。

```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 1000010;
int n, cnt = 0;
long long lf, rt, tp, bt, a[N << 1];
struct ScanLine
{
	long long l, r, h;
	int mark;
	bool operator < (const ScanLine &a) const
	{
		return h < a.h;
	}
} line[N << 1];
struct SegTree
{
	int l, r, sum;
	long long len;
} tree[N << 2];

void build(int p, int l, int r)
{
	tree[p].l = l;
	tree[p].r = r;
	tree[p].len = 0;
	tree[p].sum = 0;
	if(l == r)
		return;
	int mid = (l + r) >> 1;
	build(p << 1, l, mid);
	build(p << 1 | 1, mid + 1, r);
	return;
}

void pushup(int p)
{
	int l = tree[p].l, r = tree[p].r;
	if(tree[p].sum)
	{
		tree[p].len = a[r + 1] - a[l];
	}
	else
	{
		tree[p].len = tree[p << 1].len + tree[p << 1 | 1].len;
	}
}

void mark(int p, long long L, long long R, int c)
{
	int l = tree[p].l, r = tree[p].r;
	if(a[r + 1] <= L || R <= a[l])
		return;
	if(L <= a[l] && a[r + 1] <= R)
	{
		tree[p].sum += c;
		pushup(p);
		return;
	}
	mark(p << 1, L, R, c);
	mark(p << 1 | 1, L, R, c);
	pushup(p);
}

int main()
{
	scanf("%d", &n);
	for(int i = 1; i <= n; i++)
	{
		scanf("%lld%lld%lld%lld", &lf, &bt, &rt, &tp);
		a[2 * i - 1] = lf;
		a[2 * i] = rt;
		line[2 * i - 1] = { lf, rt, bt, 1 };
		line[2 * i] = { lf, rt, tp, -1 };
	}
	n <<= 1;
	sort(line + 1, line + n + 1);
	sort(a + 1, a + n + 1);
	int tot = unique(a + 1, a + n + 1) - a - 1;
	build(1, 1, tot - 1);
	long long ans = 0;
	for(int i = 1; i < n; i++)
	{
		mark(1, line[i].l, line[i].r, line[i].mark);
		ans += tree[1].len * (line[i + 1].h - line[i].h);
	}
	printf("%lld", ans);
	return 0;
}
```
