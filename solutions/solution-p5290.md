---
title: P5290 [十二省联考 2019] 春节十二响 题解
date: 2022-06-08
tags:
	- 题解
	- 启发式合并
	- STL
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">春节十二响</div>
<div id="problem-info-from">十二省联考 2019</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P5290">Luogu P5290</a></li><li><a href="https://loj.ac/p/3052">LibreOJ L3052</a></li><li><a href="https://www.acwing.com/problem/content/3074/">AcWing 3071</a></li></ul></div>

----

在想不到正解之前，我们可以看到出题人对于树是一条链的情况给了15分。

我们可以先想想一条链上的做法，毕竟树可以被剖成若干条链嘛。

然后就是看根节点的儿子数量。
如果只有一个儿子的话就输出 $\sum M_i$，
如果有两个儿子的话就对其左右子树分别建立一个堆，每一次取出两个堆的堆顶进行比较，取$\max$之后加入答案。当一个堆取尽之后，把另一个对中的所有元素加入答案，最后加入 $M_1$ 即可。

然后考虑将这个推到树上去。

如果我们从下往上合并每一个节点的所有子树的话，其实还是几条链的合并，因为我们最终会将一棵树合并为一条链，忽略了其他对答案不产生影响的信息。

从而最终还是进行了类似上面的链与链之间的合并，正确性也是毋庸置疑的。

时间复杂度 $O(n^2)$，预期得分60分。

然后我们尝试优化复杂度。

对于两棵子树 $x$ 和 $y$，我们假定 $\operatorname{sz}(x) \geq \operatorname{sz}(y)$。那么，按照我们上面的做法，我们的复杂度是 $O(\operatorname{sz}(x))$ 的。

我们考虑将 $y$ 合并到 $x$ 内，这样就省去了再将 $x$ 中的多余节点加入新堆中的操作了，时间复杂度就可以优化为 $O(\operatorname{sz}(y))$。

最后总的复杂度就是 $O(n \log n)$。

代码如下：

``` cpp
#include<bits/stdc++.h>
#define ll long long
using namespace std;
const int N = 200010;
template <typename T>
inline T read()
{
	T x = 0, f = 1; char c = getchar();
	while(!isdigit(c)) { if(c == '-') f = -f; c = getchar(); }
	while(isdigit(c)) x = x * 10 + c - 48, c = getchar();
	return x * f;
}
int n, a[N], f;
vector<int> e[N], t;
priority_queue<int> tr[N];
inline void merge(int x, int y)
{
	if(tr[x].size() < tr[y].size()) swap(tr[x], tr[y]);
	while(tr[y].size())
	{
		t.push_back(max(tr[x].top(), tr[y].top()));
		tr[x].pop(), tr[y].pop();
	}
	while(t.size()) tr[x].push(t.back()), t.pop_back();
}
void dfs(int x)
{
	for(int i = 0; i < e[x].size(); i++)
		dfs(e[x][i]), merge(x, e[x][i]);
	tr[x].push(a[x]);
}
int main()
{
	n = read<int>();
	for(int i = 1; i <= n; i++)
		a[i] = read<int>();
	for(int i = 2; i <= n; i++)
	{
		f = read<int>();
		e[f].push_back(i);
	}
	dfs(1);
	ll ans = 0;
	while(tr[1].size()) ans += tr[1].top(), tr[1].pop();
	cout << ans << endl;
	return 0;
}
```

