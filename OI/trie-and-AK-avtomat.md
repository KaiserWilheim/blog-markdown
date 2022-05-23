---
title: Trie 与 AC自动机
date: 2022-04-05
tags:
	- 数据结构
	- 字符串
	- 自动机
categories:
	字符串
mathjax: true
---

Trie树（字典树）与AC自动机。

<!-- more -->

# Trie

Trie树，又称字典树，是一种数据结构。

Trie树可以存储大量不同的字符串，同时支持对其以 $O(|S|)$ 的复杂度进行查询。

首先我们可以放一张图上来：

![trie1.png](https://s2.loli.net/2022/04/06/DjtOJRCaSzxwZAB.png)

这就是我们向Trie中顺序插入了aa,aba,ba,bbc,bca,caba,cba,cc八个字符串之后的结果。

我们可以发现，Trie使用边来代表字母，而用点来表示字符串。
某一个点表示的字符串就是从根节点到该节点的路径上的字母顺序排列组合而成的字符串。
比如说17号节点就代表着cba这一个字符串。

![trie2.png](https://s2.loli.net/2022/04/06/udZE5v41asjfBYh.png)

每一个点虽说都代表着一个字符串，但是这个字符串不一定被插入过。我们将实际上有的字符串进行了标红。

## 实现

### 存储

我们将每一个节点存储到一个结构体里面：

{% tabs struct %}
<!-- tab 数组 -->
``` cpp
struct trie
{
	int s[26];
	int cnt;
}tr[N];
```
<!-- endtab -->

<!-- tab 指针 -->
``` cpp
struct trie
{
	trie *s[26];
	int cnt;
}tr[N];
```
<!-- endtab -->
{% endtabs %}

这里 `s` 数组的大小是根据字符集的大小来定的，一般情况下是小写字母集，所以这里填的是26。

### 插入

具体思想就是，如果当前节点的下一位所对应的儿子非空就跳到对应的儿子，是空的那就新建一个儿子。

{% tabs insert %}
<!-- tab 数组 -->
``` cpp
void insert(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else
		{
			tr[u].s[s[i] - 'a'] = ++idx;
			u = idx;
		}
	}
	tr[u].cnt++;
}
```
<!-- endtab -->

<!-- tab 指针 -->
``` cpp
void insert(string s)
{
	auto *u = &tr[0];
	for(int i = 0; i < s.length(); i++)
	{
		if(u->s[s[i] - 'a'] != 0)u = u->s[s[i] - 'a'];
		else
		{
			u->s[s[i] - 'a'] = &tr[++idx];
			u = u->s[s[i] - 'a'];
		}
	}
	u->cnt++;
}
```
<!-- endtab -->
{% endtabs %}

### 删除

删除这个操作使用的范围不是很广。具体思想就是将对应我们想要删除的字符串的点的`cnt`值减去1即可。

{% tabs loeschen %}
<!-- tab 数组 -->
``` cpp
void loeschen(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
		u = tr[u].s[s[i] - 'a'];
	tr[u].cnt--;
}
```
<!-- endtab -->

<!-- tab 指针 -->
``` cpp
void loeschen(string s)
{
	auto *u = &tr[0];
	for(int i = 0; i < s.length(); i++)
		u = u->s[s[i] - 'a'];
	u->cnt--;
}
```
<!-- endtab -->
{% endtabs %}

### 查询

查询操作的思想就是，不断从根节点沿着字符串跳，直到调到空节点或者跳完整个字符串为止。

{% tabs chq %}
<!-- tab 数组 -->
``` cpp
bool chq(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else return false;
	}
	return tr[u].cnt > 0;
}
```
<!-- endtab -->

<!-- tab 指针 -->
``` cpp
bool chq(string s)
{
	auto *u = &tr[0];
	for(int i = 0; i < s.length(); i++)
	{
		if(u->s[s[i] - 'a'] != 0)u = u->s[s[i] - 'a'];
		else return false;
	}
	return u->cnt > 0;
}
```
<!-- endtab -->
{% endtabs %}

### 全部加起来

{% tabs all %}
<!-- tab 数组 -->
``` cpp
struct trie
{
	int s[26];
	int cnt;
}tr[N];
int idx = 0;
void insert(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else
		{
			tr[u].s[s[i] - 'a'] = ++idx;
			u = idx;
		}
	}
	tr[u].cnt++;
}
void loeschen(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
		u = tr[u].s[s[i] - 'a'];
	tr[u].cnt--;
}
bool chq(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else return false;
	}
	return tr[u].cnt > 0;
}
```
<!-- endtab -->

<!-- tab 指针 -->
``` cpp
struct trie
{
	trie *s[26];
	int cnt;
}tr[N];
int idx = 0;
void insert(string s)
{
	auto *u = &tr[0];
	for(int i = 0; i < s.length(); i++)
	{
		if(u->s[s[i] - 'a'] != 0)u = u->s[s[i] - 'a'];
		else
		{
			u->s[s[i] - 'a'] = &tr[++idx];
			u = u->s[s[i] - 'a'];
		}
	}
	u->cnt++;
}
void loeschen(string s)
{
	auto *u = &tr[0];
	for(int i = 0; i < s.length(); i++)
		u = u->s[s[i] - 'a'];
	u->cnt--;
}
bool chq(string s)
{
	auto *u = &tr[0];
	for(int i = 0; i < s.length(); i++)
	{
		if(u->s[s[i] - 'a'] != 0)u = u->s[s[i] - 'a'];
		else return false;
	}
	return u->cnt > 0;
}
```
<!-- endtab -->

<!-- tab 数组（封装） -->
``` cpp
struct trie
{
	int s[N][26];
	int cnt[N];
	
	int idx = 0;
	void insert(string str)
	{
		int u = 0;
		for(int i = 0; i < str.length(); i++)
		{
			if(s[u][str[i] - 'a'] != 0)u = s[u][str[i] - 'a'];
			else
			{
				s[u][str[i] - 'a'] = ++idx;
				u = idx;
			}
		}
		cnt[u]++;
	}
	void loeschen(string str)
	{
		int u = 0;
		for(int i = 0; i < str.length(); i++)
			u = s[u][str[i] - 'a'];
		cnt[u]--;
	}
	bool chq(string str)
	{
		int u = 0;
		for(int i = 0; i < str.length(); i++)
		{
			if(s[u][str[i] - 'a'] != 0)u = str[u][s[i] - 'a'];
			else return false;
		}
		return cnt[u] > 0;
	}
};
```
<!-- endtab -->

<!-- tab 指针（封装） -->
不会写，咕了~
<!-- endtab -->
{% endtabs %}

## 例题

Luogu P2580：https://www.luogu.com.cn/problem/P2580

{% note success 参考代码 %}
{% tabs p2580 %}
<!-- tab 数组 -->
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010;

struct trie
{
	int s[26];
	int cnt;
	int vis;
}tr[N];
int idx = 0;

void insert(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else
		{
			tr[u].s[s[i] - 'a'] = ++idx;
			u = idx;
		}
	}
	tr[u].cnt++;
}

void chq(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else
		{
			puts("WRONG");
			return;
		}
	}
	if(tr[u].cnt < 1)puts("WRONG");
	else
	{
		if(tr[u].vis == tr[u].cnt)puts("REPEAT");
		else
		{
			puts("OK");
			tr[u].vis++;
		}
	}
	return;
}

int main()
{
	int n, m;
	cin >> n;
	for(int i = 1; i <= n; i++)
	{
		string s;
		cin >> s;
		insert(s);
	}
	cin >> m;
	for(int i = 1; i <= m; i++)
	{
		string s;
		cin >> s;
		chq(s);
	}
	return 0;
}
```
<!-- endtab -->

<!-- tab 数组（封装） -->
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010;

struct trie
{
	int s[N][26];
	int cnt[N];
	int vis[N];

	int idx = 0;
	void insert(string str)
	{
		int u = 0;
		for(int i = 0; i < str.length(); i++)
		{
			if(s[u][str[i] - 'a'] != 0)u = s[u][str[i] - 'a'];
			else
			{
				s[u][str[i] - 'a'] = ++idx;
				u = idx;
			}
		}
		cnt[u]++;
	}
	void chq(string str)
	{
		int u = 0;
		for(int i = 0; i < str.length(); i++)
		{
			if(s[u][str[i] - 'a'] != 0)u = s[u][str[i] - 'a'];
			else
			{
				puts("WRONG");
				return;
			}
		}
		if(cnt[u] < 1)puts("WRONG");
		else
		{
			if(vis[u] == cnt[u])puts("REPEAT");
			else
			{
				puts("OK");
				vis[u]++;
			}
		}
		return;
	}
}tr;

int main()
{
	int n, m;
	cin >> n;
	for(int i = 1; i <= n; i++)
	{
		string s;
		cin >> s;
		tr.insert(s);
	}
	cin >> m;
	for(int i = 1; i <= m; i++)
	{
		string s;
		cin >> s;
		tr.chq(s);
	}
	return 0;
}
```
<!-- endtab -->
{% endtabs %}
{% endnote %}

## 01-Trie

01-Trie是Trie的一种变体。其字符集不是一般见到的Trie的小写字符集，而是 $\lbrace 0,1 \rbrace$。

由于01-Trie的字符集仅限于0和1，所以它被人们用来处理一些与位运算有关的东西，比如说异或和或者集合内数字两两异或的极值什么的。

### 异或极值

我们假定我们需要维护这样一个操作：
给出一个集合 $A$，再给出一个数 $k$，我们需要在集合内找到一个数 $a_i$，使之与 $k$ 的异或值最大（或最小）。

因为异或操作就是将两个数按照二进制逐位进行比对，相同为0，不同为1，所以我们就可以将上文提到的那个集合 $A$ 内的所有数根据二进制位分解之后再插入Trie中。

然后我们就像查找字符串一样，将 $k$ 进行二进制分解后得到的01串放进去匹配。
如果我们需要的是异或最小值的话，就按照正常查找字符串的方式跳。
如果我们需要的是异或最大值的话，就让每一次跳儿子的时候跳到与当前位置不同的儿子。

我们还需要注意一点，就是我们需要保证这棵Trie的深度与数据范围的最大二进制位数一致。

由于我们是从高位到低位建的，而高位远比低位重要，所以我们就可以大胆贪心，能跳就跳，不能跳就跳到相反的那个儿子上。而且我们保证了Trie的高度是一定的，这就保证了我们一定可以跳到底。

{% note info 例题 %}
[Luogu P4551 最长异或路径](https://www.luogu.com.cn/problem/P4551)
``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 100010, M = N << 1;
int n;
struct TrieTree
{
	int s[2];
	int occ;
}tr[N * 32];
int tot = 1;
int w[M], h[N], e[M], ne[M], idx;
int dis[N];
int ans;
void add(int a, int b, int v)
{
	e[idx] = b;
	w[idx] = v;
	ne[idx] = h[a];
	h[a] = idx++;
	return;
}
void insert(int x)
{
	for(int i = 30, p = 1; i >= 0; i--)
	{
		int c = ((x >> i) & 1);
		if(!tr[p].s[c])tr[p].s[c] = ++tot;
		p = tr[p].s[c];
	}
	return;
}
void get(int x)
{
	int res = 0;
	for(int i = 30, p = 1; i >= 0; i--)
	{
		int c = ((x >> i) & 1);
		if(tr[p].s[c ^ 1])
		{
			p = tr[p].s[c ^ 1];
			res |= (1 << i);
		}
		else
		{
			p = tr[p].s[c];
		}
	}
	ans = max(ans, res);
	return;
}
void dfs(int p, int fa)
{
	insert(dis[p]);
	get(dis[p]);
	for(int i = h[p]; ~i; i = ne[i])
	{
		int v = e[i];
		if(v == fa)continue;
		dis[v] = dis[p] ^ w[i];
		dfs(v, p);
	}
	return;
}
int main()
{
	cin >> n;
	memset(h, -1, sizeof(h));
	for(int i = 1; i < n; i++)
	{
		int a, b, v;
		scanf("%d%d%d", &a, &b, &v);
		add(a, b, v);
		add(b, a, v);
	}
	dfs(1, 0);
	cout << ans << endl;
	return 0;
}
```
{% endnote %}

## 可持久化Trie

有的时候题目要求我们能够随时访问某个历史版本，或甚至回溯到某个历史版本并对其进行修改。

暴力的做法是每一次都开一个新的Trie。

这时候，我们就可以将Trie持久化以完成这些题目的要求。

之前我们每一次插入的时候都是从根节点开始的，添加的字符串也是一条从根节点开始的路径，所以我们每一次插入只需要增加新的一整条路径，并把这条路径与上一个版本中未被更新的部分连接上即可。

{% note info 例题 %}

https://www.acwing.com/problem/content/258/

``` cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 600010, M = N * 25;
struct trie
{
	int s[2];
	int maxid;
}tr[M];
int idx;
int root[N];
int a[N];
int n, m;
void insert(int i, int k, int p, int q);
//i=index,k=length,p=previous ver.,q=present ver.;
void init();
int query(int root, int c, int lim);
int main()
{
	scanf("%d%d", &n, &m);
	init();
	for(int i = 1; i <= n; i++)
	{
		int x;
		scanf("%d", &x);
		a[i] = a[i - 1] ^ x;
		root[i] = ++idx;
		insert(i, 23, root[i - 1], root[i]);
	}
	char op[2];
	int x, l, r;
	while(m--)
	{
		scanf("%s", op);
		if(*op == 'A')
		{
			scanf("%d", &x);
			n++;
			a[n] = a[n - 1] ^ x;
			root[n] = ++idx;
			insert(n, 23, root[n - 1], root[n]);
		}
		else if(*op == 'Q')
		{
			scanf("%d%d%d", &l, &r, &x);
			printf("%d\n", query(root[r - 1], a[n] ^ x, l - 1));
		}
		else
		{
			puts("Youwike AK IOI!");
		}
	}
	return 0;
}
void init()
{
	tr[0].maxid = -1;
	root[0] = ++idx;
	insert(0, 23, 0, root[0]);
	return;
}
void insert(int i, int k, int p, int q)
{
	if(k < 0)
	{
		tr[q].maxid = i;
		return;
	}
	int v = a[i] >> k & 1;
	if(p)tr[q].s[v ^ 1] = tr[p].s[v ^ 1];
	tr[q].s[v] = ++idx;
	insert(i, k - 1, tr[p].s[v], tr[q].s[v]);
	tr[q].maxid = max(tr[tr[q].s[0]].maxid, tr[tr[q].s[1]].maxid);
	return;
}
int query(int root, int c, int lim)
{
	int p = root;
	for(int i = 23; i >= 0; i--)
	{
		int v = c >> i & 1;
		if(tr[tr[p].s[v ^ 1]].maxid >= lim) p = tr[p].s[v ^ 1];
		else p = tr[p].s[v];
	}
	return c ^ a[tr[p].maxid];
}
```
{% endnote %}

# AC自动机

这个AC自动机啊，他是 Aho-Corasick Automaton，不是什么Automation。

（博客的标题来自于俄语 Ахо-Корасик автомат -> Aho-Korasik avtomat）

AC自动机就像是Trie和KMP的结合。
AC自动机基于的是Trie，同时其匹配字符串时的行为与KMP有异曲同工之妙。

## 思想

### 改造 Trie

这是一个简单的Trie结构体：

``` cpp
struct trie
{
	int s[26];
	int cnt;
}tr[N];
```

对于KMP算法，我们是根据前缀函数来对字符串匹配进行优化的。
而对于Trie呢？

### fail指针

我们考虑使用前缀函数的原因：

> 我们需要找到一个前缀，在保证其与某个后缀相等的前提下，使得当前匹配的进度最大地保留下来。

把同样的思想挪到Trie树上，我们需要找的就是：

> 我们需要找到一个节点，其代表的字符串是当前节点代表的字符串的最长后缀。

于是，我们就可以根据这样的情况来构建fail指针。

举个例子：

如下图，是依次向Trie树中添加he,his,him,her,hers,they,them,their,theirs,she的结果：

![AC1-1.png](https://s2.loli.net/2022/04/07/2QdAr9Jchl4tunG.png)

然后我们开始依次构建fail指针。

我们利用BFS来帮助我们构建fail指针。

考虑一个已经构建了fail指针的节点 $u$，其对应字符 $c$ 的儿子 $s[u][c]$ 的fail指针的构建遵循下列原则：

1. 如果 $s[fail[u]][c]$ 存在，则让 $s[u][c]$ 的fail指针指向 $s[fail[u]][c]$。这样相当于是在 $u$ 和 $fail[u]$ 后面加一个字符 $c$，分别对应 $s[u][c]$ 和 $s[fail[u]][c]$。
2. 如果 $s[fail[u]][c]$ 不存在，那么我们就递归寻找 $s[fail[fail[u]]][c]$，并重复上面的判断过程，直到跳到根节点。
3. 如果真的什么都没有了，那就让 $s[u][c]$ 的fail指针指向根节点。

下图中用不同的颜色来区分不同状态的边、点：

#FF9955 代表在队列中，等待构建fail指针的点；
#FFCC00 代表正在构建fail指针的点；
#AAD400 代表已经构建完fail指针的点；

#FF0000 代表正在构建的fail指针；
#FFDD55 代表已经构建完成的fail指针。

![AC2.gif](/pics/AC2.gif)

最终效果图：

![AC2-38.png](https://s2.loli.net/2022/04/08/up7tvA28h9R3E5a.png)

代码：

``` cpp
/*
struct trie
{
	int s[26];
	int cnt;
	int fail;
}tr[N];
*/
void get_fail()
{
	queue<int>q;
	for(int i = 0; i < 26; i++)
		if(tr[0].s[i])q.push(tr[u].s[i]);
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
```

上述代码可能与我们刚才的叙述有部分出入，原因是这份代码做了些许优化。
事实证明这些优化虽然会修改Trie的结构，使之由树变为图，但是对我们存储的信息是有利无害的。

我们首先将根节点的所有儿子入队。
如果我们第一步将根节点入队的话，会导致根节点的所有子节点的fail指针指向自己，而不是指向根节点。

然后是遍历在队列中的节点。
对于其非空的儿子，我们照常构建fail指针。由于当前节点的fail指针已经构建完毕，且沿着fail指针形成的通往根节点的链上的所有节点都已经构建完毕了，所以我们直接指向当前节点的fail指针指向的节点的所对应的儿子。如果有这个儿子的话就指向他，如果没有的话就相当于是指向根节点。

然后我们将当前节点的部分空儿子指向当前节点的fail指针指向的节点的对应儿子上。
原因就是，反正也是失配了，与其花时间判失配跳fail指针再访问，不如直接访问我们想要它到的节点。
所以我们在匹配的时候一直走（结构体意义上的）树边，直到整个跑完就可以了。

照这样建完fail指针之后，我们会将原先的Trie树变为一张“Trie图”。

由于边太多太密，我就不画了。可能会补上一张CSAcademy的图。
（太糊了，感觉会不久后撤下，就先存本地了）

![](/pics/AC3.png)

（这是原数据）

![](/pics/AC4.png)

（这是图）

## 匹配

匹配的思路是这个样子的：

1. 沿着Trie图的边跳，直到遍历完整个字符串为止。
	由于我们在之前已经将Trie树改造为了Trie图，且fail指针已经在某种意义上失去了其作为失配指针的意义，所以我们只需要跳Trie图内的边即可。
2. 每走到一个节点，就沿着当前节点的fail指针累加答案。
	由于fail边跳到的是当前字符串在Trie内存在的最长后缀，且如果当前的模式串出现在了文本串内的话，其子串也会出现在文本串内，所以我们可以沿着fail指针一路遍历当前模式串的所有后缀。

这里有一份对于多模式串匹配，且允许模式串多次被统计的，用来统计所有模式串的总出现次数的代码：

``` cpp
int query(string s)
{
	int u = 0;
	int res = 0;
	for(int i = 0; i < s.length(); i++)
	{
		u = tr[u].s[s[i] - 'a'];
		for(int j = u; j > 0; j = tr[j].fail)
			res += tr[j].cnt;
	}
	return res;
}
```

其中`res`是所有模式串的总出现次数。

## 例题

{% note info 板子1 %}

[P3808 【模板】 AC 自动机（简单版）](https://www.luogu.com.cn/problem/P3808)
``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 1000010;

struct trie
{
	int s[26];
	int cnt;
	int fail;
}tr[N];
int idx = 0;

void insert(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else
		{
			tr[u].s[s[i] - 'a'] = ++idx;
			u = idx;
		}
	}
	tr[u].cnt++;
}

void get_fail()
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

int query(string s)
{
	int u = 0;
	int res = 0;
	for(int i = 0; i < s.length(); i++)
	{
		u = tr[u].s[s[i] - 'a'];
		for(int j = u; j > 0 && tr[j].cnt > 0; j = tr[j].fail)
		{
			res += tr[j].cnt;
			tr[j].cnt = 0;
		}
	}
	return res;
}

int main()
{
	int n;
	cin >> n;
	for(int i = 1; i <= n; i++)
	{
		string s;
		cin >> s;
		insert(s);
	}
	get_fail();
	string t;
	cin >> t;
	cout << query(t) << endl;
	return 0;
}
```
{% endnote %}

{% note info 板子2 %}

[P3796 【模板】 AC 自动机（加强版)](https://www.luogu.com.cn/problem/P3796)

``` cpp
#include <bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010;

struct trie
{
	int s[26];
	int cnt;
	int fail;
}tr[N];
int idx = 0;
int vis[N];
vector<string>dic;

void insert(string s, int id)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		if(tr[u].s[s[i] - 'a'] != 0)u = tr[u].s[s[i] - 'a'];
		else
		{
			tr[u].s[s[i] - 'a'] = ++idx;
			u = idx;
		}
	}
	tr[u].cnt = id;
}

void get_fail()
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

void query(string s)
{
	int u = 0;
	for(int i = 0; i < s.length(); i++)
	{
		u = tr[u].s[s[i] - 'a'];
		for(int j = u; j > 0; j = tr[j].fail)
			vis[tr[j].cnt]++;
	}
	return;
}

void init()
{
	dic.clear();
	for(int i = 0; i <= idx; i++)
	{
		for(int j = 0; j < 26; j++)tr[i].s[j] = 0;
		tr[i].cnt = 0;
		tr[i].fail = 0;
	}
	idx = 0;
	memset(vis, 0, sizeof(vis));
}

int main()
{
	int n;
	cin >> n;
	while(n)
	{
		init();
		for(int i = 1; i <= n; i++)
		{
			string s;
			cin >> s;
			dic.push_back(s);
			insert(s, i);
		}
		get_fail();
		string t;
		cin >> t;
		query(t);
		int maxn = -1;
		for(int i = 1; i <= n; i++)
			maxn = max(maxn, vis[i]);
		cout << maxn << endl;
		for(int i = 1; i <= n; i++)
			if(vis[i] == maxn)
				cout << dic[i - 1] << endl;
		cin >> n;
	}
	return 0;
}
```
{% endnote %}



































