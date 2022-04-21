---
title: Link Cut Tree
zh-CN: true
date: 2022-02-11
tags:
	- 数据结构
categories:
	数据结构
mathjax: true
---

Link/Cut Tree.

供自己学习用。
（可能会清楚一点罢）

<!--more-->

Link/Cut Tree 又称 Link Cut Tree，简称 LCT，是一类用来解决动态树问题的数据结构。

Splay 是 LCT 的基础。但是 LCT 在使用 Splay 的时候，还将其进行了一些~~魔改~~扩展。

LCT 与 Splay 一样，包含了一(dà)些(liàng)函数来维持 LCT 的运转。

# 问题引入

假设我们需要维护一棵树，使其支持如下操作：

- 修改两点间路径权值。
- 查询两点间路径权值和。
- 修改某点子树权值。
- 查询某点子树权值和。

唔，看上去是一道树剖模版题。

那么我们再加一个操作：

- 断开并连接一些边，保证仍是一棵树。

要求在线求出上面的答案。

——这就需要动态树问题的解决方法：<ruby>リンク<rt>Link</rt></ruby><ruby>/<rt>/</rt></ruby><ruby>カット<rt>Cut</rt></ruby><ruby>ツリー<rt>Tree</rt></ruby>!

## 什么是动态树问题

维护一个森林，支持删除某条边，加入某条边，并保证加边，删边之后仍是森林。动态树问题要求我们要维护这个森林的一些信息。

一般的操作有询问两点连通性，询问两点路径权值和，连接两点或切断某条边、修改信息等。

## 路径……ん……树链剖分？

既然需要询问路径上的问题，那么就可以想到使用树剖来帮助我们解决这样的问题。

但是……
树剖对LCT的适用性怎么样？

### 从 LCT 的角度回顾一下树链剖分

首先我们对整棵树按子树大小进行了剖分，并重新按照dfs序标了号。

我们发现重新标号之后，在树上形成了一些以链为单位的连续区间，并且可以用线段树进行区间操作。

### 转向动态树问题

我们发现我们刚刚讲的树剖是以子树大小作为划分条件。那我们能不能重定义一种剖分，使它更适应我们的动态树问题呢？

考虑动态树问题需要什么链。

由于动态维护一个森林，显然我们希望这个链是我们指定的链，以便利用来求解。

这就需要……

### 实链剖分！

对于一个点连向它所有儿子的边，我们自己选择一条边进行剖分。
我们称被选择的边为实边，其他边则为虚边。

对于实边，我们称它所连接的儿子为实儿子。
对于一条由实边组成的链，我们同样称之为实链。

请记住我们选择实链剖分的最重要的原因：它是我们选择的，灵活且可变。
正是它的这种灵活可变性，我们便可以采用 Splay 来维护这些实链。

# LCT！

我们可以简单的把 LCT 理解成用一些 Splay 来维护动态的树链剖分，以期实现动态树上的区间操作。对于每条实链，我们建一个 Splay 来维护整个链区间的信息。

接下来，我们来学习 LCT 的具体结构。

## 辅助树

鉴于我们维护的是一个森林，那么我们就需要一些东西将所有的树连成一个整体来维护。

于是我们就推出了——辅助树！

我们可以认为，一些Splay构成了一棵辅助树，而一些辅助树构成了LCT，其维护的是整个森林。

1. 辅助树由多棵 Splay 组成，每棵 Splay 维护原树中的一条路径，且中序遍历这棵 Splay 得到的点序列，从前到后对应原树“从上到下”的一条路径。
2. 原树每个节点与辅助树的 Splay 节点一一对应。
3. 辅助树的各棵 Splay 之间并不是独立的。每棵 Splay 的根节点的父亲节点本应是空，但在 LCT 中每棵 Splay 的根节点的父亲节点指向原树中 **这条链** 的父亲节点（即链最顶端的点的父亲节点）。这类父亲链接与通常 Splay 的父亲链接区别在于儿子认父亲，而父亲不认儿子，对应原树的一条 **虚边**。因此，每个连通块恰好有一个点的父亲节点为空。
4. 由于辅助树的以上性质，我们维护任何操作都不需要维护原树，辅助树可以在任何情况下拿出一个唯一的原树，我们只需要维护辅助树即可。

### 考虑原树和辅助树的结构关系

- 原树中的实链 : 在辅助树中节点都在一棵 Splay 中。
- 原树中的虚链 : 在辅助树中，子节点所在 Splay 的 Father 指向父节点，但是父节点的两个儿子都不指向子节点。
- 注意：原树的根不等于辅助树的根。
- 原树的 Father 指向不等于辅助树的 Father 指向。
- 辅助树是可以在满足辅助树、Splay 的性质下任意换根的。
- 虚实链变换可以轻松在辅助树上完成，这也就是实现了动态维护树链剖分。


## 实现

这里我们以[洛谷板子题](https://www.luogu.com.cn/problem/P3690)为例。

### 接下来会用到的变量声明

- `s[N][2]` 左右儿子
- `fa[N]` 节点的父亲指向
- `sum[N]` 路径权值和
- `val[N]` 点权
- `rev[N]` 翻转标记
- `sz[N]` 辅助树上子树大小
- 其他可能用到的变量

其中，`s[2]`，`fa`，`val`，`rev`和`sum`我们定义为了一个结构体，就像这样：
``` cpp
struct Node
{
	int s[2], fa, val;
	int sum, rev;
}tr[N];
```

### 接下来会用到的函数声明

#### 一般数据结构函数

- `pushup(x)`
- `pushdown(x)`

#### Splay系函数

- `rotate(x)` 将 $x$ 向上旋转。
- `splay(x)` 通过与`rotate`联动来实现将 $x$ 旋转至**当前Splay的根**。

#### LCT独有的新函数

- `access(x)` 把从根到 $x$ 的所有点放在一条实链里，使根到 $x$ 成为一条实路径，并且在同一棵 Splay 里。**只有此操作是必须实现的，其他操作视题目而实现。**
- `isroot(x)` 判断 $x$ 是否是所在树的根。
- `update(x)` 在 `access` 操作之后，递归地从上到下 `pushdown` 更新信息。
- `makeroot(x)` 使 $x$ 点成为其所在树的根。
- `link(x, y)` 在 $x, y$ 两点间连一条边。
- `cut(x, y)` 把 $x, y$ 两点间边删掉。
- `findroot(x)` 找到 $x$ 所在树的根节点编号。
- `fix(x, v)` 修改 $x$ 的点权为 $v$。
- `split(x, y)` 提取出 $x, y$ 间的路径，方便做区间操作。

## 函数讲解

### `pushup()` 与 `pushdown()`

这两个函数具体根据你所需要的维护的信息来写。
在本题内就是这样的：

``` cpp
void pushup(int x)
{
	tr[x].sum = tr[tr[x].s[0]].sum ^ tr[x].val ^ tr[tr[x].s[1]].sum;
}
```

``` cpp
void pushdown(int x)
{
	if(tr[x].rev)
	{
		pushrev(tr[x].s[0]);
		pushrev(tr[x].s[1]);
		tr[x].rev = 0;
	}
}
```

同时我们因为需要进行翻转操作，于是就会对翻转标记进行一次pushdown。我们使用了函数来进行操作，但是这个函数可有可无：既可以放进其他函数的代码内，也可以单独拿出来。我们这里单独拿了出来，放在了pushup和pushdown的前面。函数如下：

``` cpp
void pushrev(int x)
{
	swap(tr[x].s[0], tr[x].s[1]);
	tr[x].rev ^= 1;
}
```

### `splay()` 与 `rotate()`

这两个是 Splay 系的函数。

有点不一样了呢。

```cpp
void rotate(int x)
{
	int y = tr[x].fa, z = tr[y].fa;
	int k = tr[y].s[1] == x;
	if(!isroot(y)) tr[z].s[tr[z].s[1] == y] = x;
	tr[x].fa = z;
	tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
	tr[x].s[k ^ 1] = y, tr[y].fa = x;
	pushup(y), pushup(x);
}
```
``` cpp
void splay(int x)
{
	int top = 0, r = x;
	stk[++top] = r;
	while(!isroot(r)) stk[++top] = r = tr[r].fa;
	while(top) pushdown(stk[top--]);
	while(!isroot(x))
	{
		int y = tr[x].fa, z = tr[y].fa;
		if(!isroot(y))
			if((tr[y].s[1] == x) ^ (tr[z].s[1] == y)) rotate(x);
			else rotate(y);
		rotate(x);
	}
}
```

相关解释见[Splay](/OI/splay)。

### `isroot()`

第一个 LCT 系函数。

``` cpp
bool isroot(int x)
{
	return (tr[tr[x].fa].s[0] != x) && (tr[tr[x].fa].s[1] != x);
}
```

在前面我们已经说过，LCT 具有 如果一个儿子不是实儿子，他的父亲找不到它的性质。
所以当一个点既不是它父亲的左儿子，又不是它父亲的右儿子，它就是当前 Splay 的根。

### `access()`

另一个 LCT 系函数。

``` cpp
void access(int x)
{
	int z = x;
	for(int y = 0; x; y = x, x = tr[x].fa)
	{
		splay(x);
		tr[x].s[1] = y, pushup(x);
	}
	splay(z);
}
```

access 是 LCT 的核心操作，试想我们像求解一条路径，而这条路径恰好就是我们当前的一棵 Splay，直接调用其信息即可。

`access()` 其实很容易，只有如下四步操作：

1. 把当前节点转到根。
2. 把儿子换成之前的节点。
3. 更新当前点的信息。
4. 把当前点换成当前点的父亲，继续操作。

### `makeroot()`

``` cpp
void makeroot(int x)
{
	access(x);
	pushrev(x);
}
```

### `link()`

link两个点其实很简单，先`makeroot(x)`，然后把 $x$ 的父亲指向 $y$ 即可。显然，这个操作肯定不能发生在同一棵树内，所以记得先判一下。

``` cpp
void link(int x, int y)
{
	makeroot(x);
	if(findroot(y) != x) tr[x].fa = y;
}
```

### `cut()`

`cut` 有两种情况，保证合法和不一定保证合法。（废话）

如果保证合法，直接 `split(x, y)`，这时候 $y$ 是根，$x$ 一定是它的儿子，双向断开即可。

如果是不保证合法，我们需要判断一下是否有边。
想要删边，必须要满足如下三个条件：

1. $x,y$ 连通。
2. $x$ 是 $y$ 的父亲。
3. $y$ 没有左儿子。

总结一下，上面三句话的意思就一个：$x,y$ 之间联通，且其间没有其他节点。

``` cpp
void cut(int x, int y)
{
	makeroot(x);
	if((findroot(y) == x) && (tr[y].fa == x) && (!tr[y].s[0]))
	{
		tr[x].s[1] = tr[y].fa = 0;
		pushup(x);
	}
}
```

### `split()`

`split` 操作意义很简单，就是拿出一棵splay，维护的是 $x$ 到 $y$ 的路径。
先 `makeroot(x)`，然后 `access(y)`。如果要 $y$ 做根，再 `splay(y)`。
另外split这三个操作直接可以把需要的路径拿出到 $y$ 的子树上，那不是随便干嘛咯。
这里不需要 $y$ 做根，所以就没有写 `splay(y)`。

``` cpp
void split(int x, int y)
{
    makeroot(x);
    access(y);
}
```

### `findroot()`

`findroot()` 其实就是找到当前辅助树的根。在 `access(y)` 后，再 `splay(y)`。这样根就是树里最小的那个，一直往左儿子走，沿途 `pushdown` 即可。
一直走到没有左儿子，非常简单。
注意，每次查询之后需要把查询到的答案对应的结点 `splay` 上去以保证复杂度。

``` cpp
int findroot(int x)
{
	access(x);
	while(tr[x].s[0]) pushdown(x), x = tr[x].s[0];
	splay(x);
	return x;
}
```

### 一些提醒

- 干点啥前一定要想一想需不需要 `pushup` 或者 `pushdown`，LCT由于特别灵活的原因，少 `pushdown` 或者 `pushup` 一次就可能把修改改到不该改的点上！
- LCT的 `rotate` 和 splay 的不太一样，`if(z)` 一定要放在前面。
- LCT的 `splay` 操作就是旋转到根，没有旋转到谁儿子的操作，因为不需要。

## 全部加起来

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;

int n, m;
struct Node
{
	int s[2], fa, val;
	int sum, rev;
}tr[N];
int stk[N];

void pushrev(int x)
{
	swap(tr[x].s[0], tr[x].s[1]);
	tr[x].rev ^= 1;
}

void pushup(int x)
{
	tr[x].sum = tr[tr[x].s[0]].sum ^ tr[x].val ^ tr[tr[x].s[1]].sum;
}

void pushdown(int x)
{
	if(tr[x].rev)
	{
		pushrev(tr[x].s[0]);
		pushrev(tr[x].s[1]);
		tr[x].rev = 0;
	}
}

bool isroot(int x)
{
	return (tr[tr[x].fa].s[0] != x) && (tr[tr[x].fa].s[1] != x);
}

void rotate(int x)
{
	int y = tr[x].fa, z = tr[y].fa;
	int k = (tr[y].s[1] == x);
	if(!isroot(y)) tr[z].s[tr[z].s[1] == y] = x;
	tr[x].fa = z;
	tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
	tr[x].s[k ^ 1] = y, tr[y].fa = x;
	pushup(y), pushup(x);
}

void splay(int x)
{
	int top = 0, r = x;
	stk[++top] = r;
	while(!isroot(r)) stk[++top] = r = tr[r].fa;
	while(top) pushdown(stk[top--]);
	while(!isroot(x))
	{
		int y = tr[x].fa, z = tr[y].fa;
		if(!isroot(y))
			if((tr[y].s[1] == x) ^ (tr[z].s[1] == y)) rotate(x);
			else rotate(y);
		rotate(x);
	}
}

void access(int x)  // 建立一条从根到x的路径，同时将x变成splay的根节点
{
	int z = x;
	for(int y = 0; x; y = x, x = tr[x].fa)
	{
		splay(x);
		tr[x].s[1] = y, pushup(x);
	}
	splay(z);
}

void makeroot(int x)  // 将x变成原树的根节点
{
	access(x);
	pushrev(x);
}

int findroot(int x)  // 找到x所在原树的根节点, 再将原树的根节点旋转到splay的根节点
{
	access(x);
	while(tr[x].s[0]) pushdown(x), x = tr[x].s[0];
	splay(x);
	return x;
}

void split(int x, int y)  // 给x和y之间的路径建立一个splay，其根节点是y
{
	makeroot(x);
	access(y);
}

void link(int x, int y)  // 如果x和y不连通，则加入一条x和y之间的边
{
	makeroot(x);
	if(findroot(y) != x) tr[x].fa = y;
}

void cut(int x, int y)  // 如果x和y之间存在边，则删除该边
{
	makeroot(x);
	if((findroot(y) == x) && (tr[y].fa == x) && (!tr[y].s[0]))
	{
		tr[x].s[1] = tr[y].fa = 0;
		pushup(x);
	}
}

int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++) scanf("%d", &tr[i].val);
	while(m--)
	{
		int t, x, y;
		scanf("%d%d%d", &t, &x, &y);
		if(t == 0)
		{
			split(x, y);
			printf("%d\n", tr[y].sum);
		}
		else if(t == 1) link(x, y);
		else if(t == 2) cut(x, y);
		else
		{
			splay(x);
			tr[x].val = y;
			pushup(x);
		}
	}
	return 0;
}
```

{% note default 封装好了的版本 %}

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010;

int n, m;
struct Link_Cut_Tree
{
	int s[N][2], fa[N], val[N];
	int sum[N], rev[N];

	int stk[N];

	void pushrev(int x)
	{
		swap(s[x][0], s[x][1]);
		rev[x] ^= 1;
	}

	void pushup(int x)
	{
		sum[x] = sum[s[x][0]] ^ val[x] ^ sum[s[x][1]];
	}

	void pushdown(int x)
	{
		if(rev[x])
		{
			pushrev(s[x][0]);
			pushrev(s[x][1]);
			rev[x] = 0;
		}
	}

	bool isroot(int x)
	{
		return (s[fa[x]][0] != x) && (s[fa[x]][1] != x);
	}

	void rotate(int x)
	{
		int y = fa[x], z = fa[y];
		int k = (s[y][1] == x);
		if(!isroot(y))s[z][s[z][1] == y] = x;
		fa[x] = z;
		s[y][k] = s[x][k ^ 1], fa[s[x][k ^ 1]] = y;
		s[x][k ^ 1] = y, fa[y] = x;
		pushup(y), pushup(x);
	}

	void splay(int x)
	{
		int top = 0, r = x;
		stk[++top] = r;
		while(!isroot(r)) stk[++top] = r = fa[r];
		while(top) pushdown(stk[top--]);
		while(!isroot(x))
		{
			int y = fa[x], z = fa[y];
			if(!isroot(y))
				if((s[y][1] == x) ^ (s[z][1] == y)) rotate(x);
				else rotate(y);
			rotate(x);
		}
	}

	void access(int x)
	{
		int z = x;
		for(int y = 0; x; y = x, x = fa[x])
		{
			splay(x);
			s[x][1] = y;
			pushup(x);
		}
		splay(z);
	}

	void makeroot(int x)
	{
		access(x);
		pushrev(x);
	}

	int findroot(int x)
	{
		access(x);
		while(s[x][0]) pushdown(x), x = s[x][0];
		splay(x);
		return x;
	}

	void split(int x, int y)
	{
		makeroot(x);
		access(y);
	}

	void link(int x, int y)
	{
		makeroot(x);
		if(findroot(y) != x) fa[x] = y;
	}

	void cut(int x, int y)
	{
		makeroot(x);
		if((findroot(y) == x) && (fa[y] == x) && (!s[y][0]))
		{
			s[x][1] = fa[y] = 0;
			pushup(x);
		}
	}
}tr;

int main()
{
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++) scanf("%d", &tr.val[i]);
	while(m--)
	{
		int t, x, y;
		scanf("%d%d%d", &t, &x, &y);
		if(t == 0)
		{
			tr.split(x, y);
			printf("%d\n", tr.sum[y]);
		}
		else if(t == 1) tr.link(x, y);
		else if(t == 2) tr.cut(x, y);
		else
		{
			tr.splay(x);
			tr.val[x] = y;
			tr.pushup(x);
		}
	}

	return 0;
}
```

{% endnote %}
























