---
title: P2414 [NOI2011] 阿狸的打字机 题解
date: 2022-04-19
tags:
	- 题解
	- 字符串
	- AC自动机
categories:
	题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">阿狸的打字机</div>
<div id="problem-info-from">NOI 2011</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P2414">Luogu P2414</a></li><li><a href="https://loj.ac/p/2444">LibreOJ L2444</a></li><li><a href="https://www.acwing.com/problem/content/985/">AcWing 983</a></li></ul></div>

----

题目这个样子就相当于是构造了一个字符串的集合，同时询问我们某一个字符串在另一个字符串内出现了多少次。

首先看一下数据范围：
字符串数量 $n \leq 10^5$，询问数量 $m \leq 10^5$。

况且字符串的输入格式还特别奇怪：

> 阿狸喜欢收藏各种稀奇古怪的东西，最近他淘到一台老式的打字机。打字机上只有 $28$ 个按键，分别印有 $26$ 个小写英文字母和 `B`、`P` 两个字母。经阿狸研究发现，这个打字机是这样工作的：
>- 输入小写字母，打字机的一个凹槽中会加入这个字母(这个字母加在凹槽的最后)。
>- 按一下印有 `B` 的按键，打字机凹槽中最后一个字母会消失。
>- 按一下印有 `P` 的按键，打字机会在纸上打印出凹槽中现有的所有字母并换行，但凹槽中的字母不会消失。
>
>例如，阿狸输入 `aPaPBbP`，纸上被打印的字符如下：
>```
> a
> aa
> ab
>```
>我们把纸上打印出来的字符串从 $1$ 开始顺序编号，一直到 $n$。

然后我们就可以由此联想到Trie，以及以其为基础的AC自动机。

AC自动机是有字符串匹配的功能的，但是这样直接匹配还是有点慢。
其每一次询问的时间复杂度都是与文本串的长度成正相关的。

当然我们可以想到离线询问，对于处理相同文本串的询问确实是节省了时间。

同时我们考虑一个性质：

**我们的所有文本串均会出现在AC自动机中。**

或者说，我们相当于是拿模式串与模式串相比对。

这样我们就完全是在AC自动机内部做匹配，不需要考虑什么被抛弃的部分了。

----

如何解决询问？

我们刚才考虑过离线并按照文本串分组。

我们对于某一个文本串，要如何才能提溜出来其所有的子串呢？

我们考虑这样一个有关fail边的性质：

> 由于fail边跳到的是当前字符串在Trie内存在的最长后缀，且如果当前的模式串出现在了文本串内的话，其子串也会出现在文本串内，所以我们可以沿着fail指针一路遍历当前模式串的所有后缀。

所以对于某一个字符串 $s$，从根节点到代表 $s$ 的节点的这一条路径上所有节点及沿着其fail指针跳到根节点的所有路径上的点代表的字符串都在 $s$ 里面出现过。

这其中包含了 $s$ 的所有子串，因为其的某个子串一定是某一个后缀的某个前缀。

----

既然这个样子了，我们不如就不直接遍历整个AC自动机，而单独把fail指针提溜出来建成一张图遍历好了。

这里还附加了一个特殊性质，就是我们单独把fail指针拎出来之后会构建出来一棵树，而不是一张图。

然后我们考虑记录两个值：当前节点的DFS序和回溯到当前节点时的时间。

我们记录了这两个值以后，就可以单独把这个节点的子树给提溜出来了。

由于我们fail边指向的是当前字符串的最长后缀，那么我们fail树里面某一个节点绝对是其子树内所有点的某一个后缀。
那么如果这个节点出现过一次，那么这个节点代表的字符串在其子树内节点所代表的字符串内就必定出现过一次。
同理，如果这个节点的子树内的节点代表的字符串出现过一次，这个节点代表的字符串必定也出现过一次。
所以我们统计某一个字符串出现的次数的时候，我们需要统计该节点及其子树的所有信息。

那我们好像可以使用树状数组或线段树维护……

----

那我们怎么统计信息呢？

顺序遍历字符串即可。
同时还要沿着AC自动机跳。

每一次遇到一个新的字符的时候分类讨论：

- 如果是小写字母，那就沿着AC自动机跳，同时该节点出现的次数`+1`。
- 如果是 `P`，那就意味着（可能）有询问需要处理。遍历所有该字符串下的询问，并存储答案。
- 如果是 `B`，那就意味着上一个字符串遍历的所有信息就不算了，该节点出现的次数`-1`，同时跳到当前节点的父亲。

分析完毕。

----

然后是代码：

``` cpp
#include <bits/stdc++.h>
using namespace std;
const int N = 100010, M = N << 1;
int read()
{
	int x = 0, flag = 1; char c;
	while((c = getchar()) < '0' || c > '9') if(c == '-') flag = -1;
	while(c >= '0' && c <= '9') x = (x << 3) + (x << 1) + (c ^ 48), c = getchar();
	return x * flag;
}
int n, m;
char s[N];
int len;
int h[N], e[M], ne[M], idx;
void add(int a, int b)
{
	e[++idx] = b, ne[idx] = h[a], h[a] = idx;
}

int a[N];
int dfn[N], out[N], indx;

struct AC
{
	int s[26];
	int v, fail, fa;
}tr[N];
int cnt;
void insert()
{
	len = strlen(s);
	for(int i = 0, now = 0; i < len; i++)
	{
		if(s[i] >= 'a' && s[i] <= 'z')
		{
			if(!tr[now].s[s[i] - 'a'])
			{
				tr[now].s[s[i] - 'a'] = ++cnt;
				tr[cnt].fa = now;
			}
			now = tr[now].s[s[i] - 'a'];
		}
		if(s[i] == 'P')
			a[++n] = now;
		if(s[i] == 'B')
			now = tr[now].fa;
	}
}
void build_fail()
{
	queue<int>q;
	for(int i = 0; i < 26; i++)
		if(tr[0].s[i])q.push(tr[0].s[i]);
	while(!q.empty())
	{
		int u = q.front();
		q.pop();
		for(int i = 0; i < 26; i++)
		{
			if(tr[u].s[i])
			{
				tr[tr[u].s[i]].fail = tr[tr[u].fail].s[i];
				q.push(tr[u].s[i]);
			}
			else
			{
				tr[u].s[i] = tr[tr[u].fail].s[i];
			}
		}
	}
}

void dfs(int u, int p)
{
	dfn[u] = ++indx;
	for(int i = h[u]; ~i; i = ne[i])
	{
		int v = e[i];
		if(v ^ p)dfs(v, u);
	}
	out[u] = indx;
}

int lowbit(int x)
{
	return x & -x;
}
int c[N];
void segadd(int x, int v)
{
	for(int i = x; i <= indx; i += lowbit(i))
		c[i] += v;
}
int segsum(int x)
{
	int ans = 0;
	for(int i = x; i > 0; i -= lowbit(i))
		ans += c[i];
	return ans;
}

struct query
{
	int x, id;
};
vector<query>g[N];

int ans[N];
int main()
{
	memset(h, -1, sizeof(h));
	scanf("%s", s);
	insert();
	build_fail();
	for(int i = 1; i <= cnt; i++)
		add(i, tr[i].fail), add(tr[i].fail, i);
	dfs(0, 0);
	m = read();
	for(int i = 1; i <= m; i++)
	{
		int x = read(), y = read();
		g[y].push_back({ x,i });
	}
	for(int i = 0, now = 0, j = 0; i < len; i++)
	{
		if(s[i] == 'P')
		{
			j++;
			for(int k = 0; k < g[j].size(); k++)
			{
				int x = g[j][k].x, id = g[j][k].id;
				ans[id] = segsum(out[a[x]]) - segsum(dfn[a[x]] - 1);
			}
		}
		if(s[i] == 'B')
		{
			segadd(dfn[now], -1);
			now = tr[now].fa;
		}
		if(s[i] >= 'a' && s[i] <= 'z')
		{
			now = tr[now].s[s[i] - 'a'];
			segadd(dfn[now], 1);
		}
	}
	for(int i = 1; i <= m; i++)
		printf("%d\n", ans[i]);
	return 0;
}
```
