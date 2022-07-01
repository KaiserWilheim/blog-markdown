---
title: P5024 [NOIP2018 提高组] 保卫王国 题解
date: 2022-06-29
tags:
	- 题解
	- 树链剖分
	- 动态DP
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">保卫王国</div>
<div id="problem-info-from">NOIP-S 2018</div>
<div id="problem-info-difficulty">省选/NOI-</div>
<div id="problem-info-color">#9d3dcf</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P5024">Luogu P5024</a></li><li><a href="https://loj.ac/p/2955">LibreOJ L2955</a></li><li><a href="https://www.acwing.com/problem/content/538/">AcWing 536</a></li><li><a href="https://uoj.ac/problem/441">UOJ #441</a></li></ul></div>

----

保卫王国这道题是一道非常经典的动态DP题目，尽管其刚刚推出不到几年。

# 题意转化

首先我们做一下题意的转化。

题目要求在Z~~imbabwe~~国的每一个城市选择驻军或者不驻军，要求每一条道路两端至少有一个城市驻军。
驻军会产生正的费用，我们需要找出费用最小的方案。

对于上面的部分，我们只需要求一下这棵树的最小权点覆盖即可，也就是点权和减去最大权独立集。

然后我们还需要满足一系列要求，要求钦定两个城市强制驻军/不驻军，对每一个要求我们需要输出在当前要求下的最小权点覆盖。
询问之间互不影响。

对于这个情况的话，我们强制选点就可以将其权值改为 $w[i] - 10^{15}$，强制不选点的话就可以将其权值改为 $w[i] + 10^{15}$，最后再在答案里面消除影响即可。

然后就是如何快速求出答案了。

# 动态DP

动态DP是一类树上问题的统称，其一般源于一些简单的树上DP（就比如说让我们求树上的最大独立集），但是被加入了~丧心病狂~的修改点权操作。

我们先考虑一下正常的树上最大独立集怎么做。

我们定义 $f_{x,1}$ 为选中 $x$ 节点的最大结果，$f_{x,0}$ 为不选 $x$ 节点的最大结果，不难得到以下式子：

$$
\begin{align}
f_{i,0} &= \sum_j \max(f_{j,0},f_{j,1}) \\\\
f_{i,1} &= \sum_j f_{j,0} + a_i
\end{align}
$$

最后的答案就是 $max(f_{1,0},f_{1,1})$。

我们在没有修改点权的情况下一通 $O(n)$ 的DP就可以解决了。

但是对于修改点权的情况下我们无法（也不一定用）承担 $O(nm)$ 的时间复杂度。
我们再次看一眼上面的式子，发现只需要更新点权被更改了的节点到根节点的路径上的所有DP值即可。

这样做有一个风险，我们很可能遇到一条链的情况，这时候我们需要更新 $n$ 个节点，我们担不起这样的时间复杂度。
我们希望只需要更新 $O(\log n)$ 级别的节点……

tbl大神从上古论文中翻出来一个“全局平衡二叉树”（[题解在此](https://www.luogu.com.cn/blog/_post/46967)），（看起来）比单独树剖要好写，还不会被卡。
“全局平衡二叉树”固然好，但是Kaiser不会。
所以这里就只介绍树剖做法了。

树剖有一个性质，就是其重链个数不超过 $O(\log n)$ 条，我们最多需要更新的次数也不多于 $O(\log n)$ 次。

这样我们就可以将我们的复杂度降为 $O(m\log n)$ 级别的，看一下是可以过 $10^5$ 的数据的。

# 维护信息

然后我们考虑如何去维护这种信息。

同一条重链上的节点DFS序都是连续的，这让我们可以使用线段树等数据结构进行维护。

我们保持 $f$ 数组的定义不变，新建一个 $g$ 数组来迎合树剖剖出来的重儿子和轻儿子的概念。
我们定义 $g_{i,0}$ 代表 $i$ 号节点所有轻儿子都不取的结果，$g_{i,1}$ 代表 $i$ 号节点的轻儿子可取可不取的结果。
这样我们就可以大大简化我们的DP式子：

$$
\begin{align}
f_{i,0} &= g_{i,0} + \max(f_{son_i,0},f_{son_i,1}) \\\\
f_{i,1} &= g_{i,1} + a_i + f_{son_i,0}
\end{align}
$$

特殊地，对于叶子结点，$g_{i,0}=g_{i,1}=0$。

我们不如再合并一下，让 $g_{i,1}$ 直接代表只考虑轻儿子和自己的最大权独立集，相当于原来的 $g_{i,1}+a_i$。
这样我们的式子里面就只剩下 $f$ 和 $g$ 了。

这样子仍然不好维护。

## 矩阵

我们考虑像维护广义斐波那契数列那样维护信息，也就是定义一个矩阵和转移矩阵，用矩阵乘的方式来维护我们的信息。

我们大胆地定义一个新运算 $\odot$，定义 $\mathbf{A} \odot \mathbf{B}$ 的结果 $\mathbf{C}$ 为：

$$
\mathbf{C}\_{i,j} = \max_k (\mathbf{A}\_{i,k},\mathbf{B}\_{k,j})
$$

相当于就是把正常矩阵乘法里面的 $\sum$ 改为了 $\max$。
不知道什么原因，可能是取max和求和都具有结合律吧，这个操作就是满足结合律，我们就可以用矩阵乘法来维护它。

然后我们把我们的式子拆成类似这样的形式：

$$
\begin{align}
f_{i,0} &= \max(f_{son_i,0}+g_{i,0},f_{son_i,1}+g_{i,0}) \\\\
f_{i,1} &= \max(f_{son_i,0}+g_{i,1},-\infty)
\end{align}
$$

这样子我们就可以利用矩阵维护了。

我们确定我们的状态矩阵是长这个样子的：$[f_{i,0},f_{i,1}]$

然后我们需要找到一个转移矩阵 $\mathbf{U}$ 来使得 $[f_{son_i,0},f_{son_i,1}] \odot \mathbf{U} = [f_{i,0},f_{i,1}]$。

经过一番推导，我们可以得出我们的转移矩阵是 $\begin{bmatrix}g_{i,0}&g_{i,1}\\\\g_{i,0}&-\infty\end{bmatrix}$。
（比较简单我就不写了）

于是我们就可以开心维护了。

对于每一个节点，我们存储的是一个转移矩阵。在重链上的时候直接就求区间积，需要跳轻边的时候更新转移矩阵即可。

不过我们访问重链的时候是先访问链顶再访问链尾，我们的左右乘关系需要倒过来，整理一下可得
$$
\begin{bmatrix}g_{i,0}&g_{i,1}\\\\g_{i,0}&-\infty\end{bmatrix}
\odot
\begin{bmatrix}f_{son_i,0}\\\\f_{son_i,1}\end{bmatrix}=
\begin{bmatrix}f_{i,0}\\\\f_{i,1}\end{bmatrix}
$$

# 代码

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = 200010;
const ll INF = 1e15;
struct Matrix
{
	ll m[2][2];
	Matrix() { memset(m, -0x3f, sizeof(m)); }
	inline Matrix operator * (Matrix b)
	{
		Matrix c;
		for (int i = 0; i < 2; i++)
			for (int j = 0; j < 2; j++)
				for (int k = 0; k < 2; k++)
					c.m[i][j] = max(c.m[i][j], m[i][k] + b.m[k][j]);
		return c;
	}
};

int n, m;
ll a[N];
int h[N], e[M], ne[M], idx;
void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
int fa[N], son[N], sz[N], dep[N], top[N];
int id[N], dfn[N], ed[N], cnt;
ll f[N][2];
Matrix val[N];

struct SegTree
{
	int l, r;
	Matrix v;
}tr[N << 3];
void pushup(int p)
{
	tr[p].v = tr[p << 1].v * tr[p << 1 | 1].v;
}
void build(int p, int l, int r)
{
	tr[p].l = l, tr[p].r = r;
	if (l == r)
	{
		tr[p].v = val[dfn[l]];
		return;
	}
	int mid = (l + r) >> 1;
	build(p << 1, l, mid);
	build(p << 1 | 1, mid + 1, r);
	pushup(p);
}
void segadd(int p, int x)
{
	if (tr[p].l == tr[p].r)
	{
		tr[p].v = val[dfn[x]];
		return;
	}
	int mid = (tr[p].l + tr[p].r) >> 1;
	if (x <= mid)segadd(p << 1, x);
	else segadd(p << 1 | 1, x);
	pushup(p);
}
Matrix segsum(int p, int l, int r)
{
	if (tr[p].l == l && tr[p].r == r)return tr[p].v;
	int mid = (tr[p].l + tr[p].r) >> 1;
	if (r <= mid)
		return segsum(p << 1, l, r);
	else if (l > mid)
		return segsum(p << 1 | 1, l, r);
	else
		return segsum(p << 1, l, mid) * segsum(p << 1 | 1, mid + 1, r);
}

void dfs1(int p, int father)
{
	fa[p] = father, dep[p] = dep[father] + 1, sz[p] = 1;
	for (int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if (j == father)continue;
		dfs1(j, p);
		sz[p] += sz[j];
		if (sz[j] > sz[son[p]])son[p] = j;
	}
}
void dfs2(int p, int t)
{
	id[p] = ++cnt, dfn[cnt] = p, top[p] = t;
	ed[t] = max(ed[t], cnt);
	f[p][0] = 0, f[p][1] = a[p];
	val[p].m[0][0] = val[p].m[0][1] = 0;
	val[p].m[1][0] = a[p];
	if (son[p])
	{
		dfs2(son[p], t);
		f[p][0] += max(f[son[p]][0], f[son[p]][1]);
		f[p][1] += f[son[p]][0];
	}
	for (int i = h[p]; ~i; i = ne[i])
	{
		int j = e[i];
		if (j == fa[p] || j == son[p])continue;
		dfs2(j, j);
		f[p][0] += max(f[j][0], f[j][1]);
		f[p][1] += f[j][0];
		val[p].m[0][0] += max(f[j][0], f[j][1]);
		val[p].m[0][1] = val[p].m[0][0];
		val[p].m[1][0] += f[j][0];
	}
}
void addpath(int p, ll k)
{
	val[p].m[1][0] += k - a[p];
	a[p] = k;
	Matrix bef, aft;
	while (p)
	{
		bef = segsum(1, id[top[p]], ed[top[p]]);
		segadd(1, id[p]);
		aft = segsum(1, id[top[p]], ed[top[p]]);
		p = fa[top[p]];
		val[p].m[0][0] += max(aft.m[0][0], aft.m[1][0]) - max(bef.m[0][0], bef.m[1][0]);
		val[p].m[0][1] = val[p].m[0][0];
		val[p].m[1][0] += aft.m[0][0] - bef.m[0][0];
	}
}
char type[10];
ll sum;
int main()
{
	memset(h, 0, sizeof(h));
	scanf("%d%d", &n, &m);
	cin >> type;
	for (int i = 1; i <= n; i++)
	{
		scanf("%lld", &a[i]);
		sum += a[i];
	}
	for (int i = 1; i < n; i++)
	{
		int u, v;
		scanf("%d%d", &u, &v);
		add(u, v), add(v, u);
	}
	dfs1(1, 0);
	dfs2(1, 1);
	build(1, 1, n);
	for (int i = 1; i <= m; i++)
	{
		int u, x, v, y;
		scanf("%d%d%d%d", &u, &x, &v, &y);
		if ((x == 0 && y == 0) && (fa[u] == v || fa[v] == u))
		{
			puts("-1");
			continue;
		}
		ll v1 = a[u], v2 = a[v];
		addpath(u, a[u] + (x == 1 ? -INF : INF));
		addpath(v, a[v] + (y == 1 ? -INF : INF));
		Matrix ans = segsum(1, id[1], ed[1]);
		ll res = sum - max(ans.m[0][0], ans.m[1][0]) + (x == 1 ? 0 : INF) + (y == 1 ? 0 : INF);
		printf("%lld\n", res);
		addpath(u, v1);
		addpath(v, v2);
	}
	return 0;
}
```
