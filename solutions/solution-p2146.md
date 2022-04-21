---
title: P2146 软件包管理器 题解
date: 2022-01-02
tags:
   - 题解
   - 树链剖分
categories: 
 题解
mathjax: true
---

Luogu P2146
LibreOJ #2130

2015 NOI

<!--more-->
----

树剖模板题。

首先我们看一下[题面](https://www.luogu.com.cn/problem/P2146)；

将题面简化一下，就是下面这样：

> 给定一棵大小为 $n$ 的，根节点为0的树，我们需要对这棵树进行两种操作，并求出有多少个节点的状态被改变：
> 1. 将根节点到给定节点的路径全部变成1，输入格式为 `install x` ；
> 2. 将给定节点及其子树全部变成0，输入格式为 `uninstall x` ；
> 初始情况下所有节点都是0。

对于我们的结果，我们直接可以对比操作前后整棵树的权值和（存在根节点里面），然后直接输出两数之差即可。

但是我们还需要注意一点：**默认懒标记值**。
我们之前一直设懒标记的默认值为0，而在此题的环境内，0代表的是一种状态，而非一个值。如果将懒标记值为0的时候直接跳过的话，就无法卸载所有需要卸载的软件。
所以我们在此将默认懒标记值定为-1。pushup的时候也是一样。

当然，本题要求的是区间覆盖而非区间增减，而这个就只将segadd和pushdown两个函数里面的`+=`换成`=`即可。
比如这样：

segadd：

``` cpp
void segadd(int p, int l, int r, int k)
{
    if((l <= tr[p].l) && (r >= tr[p].r))
    {
        tr[p].add = k;
        tr[p].sum = k * (tr[p].r - tr[p].l + 1);
        return;
    }
    pushdown(p);
    int mid = (tr[p].l + tr[p].r) >> 1;
    if(l <= mid) segadd(p << 1, l, r, k);
    if(r > mid) segadd(p << 1 | 1, l, r, k);
    pushup(p);
    return;
}
```

pushdown：

``` cpp
void pushdown(int p)
{
    auto &root = tr[p], &left = tr[p << 1], &rght = tr[p << 1 | 1];
    if(root.add!=-1)
    {
        left.add = root.add, left.sum = root.add * (left.r - left.l + 1);
        rght.add = root.add, rght.sum = root.add * (rght.r - rght.l + 1);
        root.add = -1;
    }
    return;
}
```

因为本题的根节点是0，所以我将所有节点的编号都加上了1，这样就与正常情况下差不多了。
完整代码：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
const int N = 100010, M = N << 1;
int n, m;
int h[N], e[M], ne[M], idx;
int id[N], cnt;
int dep[N], sz[N], top[N], fa[N], son[N];
struct segtree
{
    int l, r;
    ll add, sum;
}tr[N << 3];
void add(int a, int b)
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}
void dfs1(int p, int father, int depth)
{
    dep[p] = depth, fa[p] = father, sz[p] = 1;
    for(int i = h[p]; ~i; i = ne[i])
    {
        int j = e[i];
        if(j == father) continue;
        dfs1(j, p, depth + 1);
        sz[p] += sz[j];
        if(sz[son[p]] < sz[j]) son[p] = j;
    }
    return;
}
void dfs2(int p, int t)
{
    id[p] = ++cnt, top[p] = t;
    if(!son[p]) return;
    dfs2(son[p], t);
    for(int i = h[p]; ~i; i = ne[i])
    {
        int j = e[i];
        if(j == fa[p] || j == son[p]) continue;
        dfs2(j, j);
    }
    return;
}
void pushup(int p)
{
    tr[p].sum = tr[p << 1].sum + tr[p << 1 | 1].sum;
    return;
}
void pushdown(int p)
{
    auto &root = tr[p], &left = tr[p << 1], &rght = tr[p << 1 | 1];
    if(root.add!=-1)
    {
        left.add = root.add, left.sum = root.add * (left.r - left.l + 1);
        rght.add = root.add, rght.sum = root.add * (rght.r - rght.l + 1);
        root.add = -1;
    }
    return;
}
void build(int p, int l, int r)
{
    tr[p] = { l, r, -1, 0 };
    if(l == r) return;
    int mid = (l + r) >> 1;
    build(p << 1, l, mid);
    build(p << 1 | 1, mid + 1, r);
    //pushup(p);
    return;
}
void segadd(int p, int l, int r, int k)
{
    if((l <= tr[p].l) && (r >= tr[p].r))
    {
        tr[p].add = k;
        tr[p].sum = k * (tr[p].r - tr[p].l + 1);
        return;
    }
    pushdown(p);
    int mid = (tr[p].l + tr[p].r) >> 1;
    if(l <= mid) segadd(p << 1, l, r, k);
    if(r > mid) segadd(p << 1 | 1, l, r, k);
    pushup(p);
    return;
}
ll install(int p)
{
    ll prev = tr[1].sum;
    int v = 1;
    while(top[p] != top[v])
    {
        if(dep[top[p]] < dep[top[v]]) swap(p, v);
        segadd(1, id[top[p]], id[p], 1);
        p = fa[top[p]];
    }
    if(dep[p] < dep[v]) swap(p, v);
    segadd(1, id[v], id[p], 1);
    return tr[1].sum - 114514;
}
ll uninstall(int p)
{
    ll prev = tr[1].sum;
    segadd(1, id[p], id[p] + sz[p] - 1, 0);
    return 114514 - tr[1].sum;
}
int main()
{
    memset(h, -1, sizeof(h));
    scanf("%d", &n);
    for(int i = 2; i <= n; i++)
    {
        int a;
        scanf("%d", &a);
        a++;
        add(a, i);
        add(i, a);
    }
    dfs1(1, 1, 1);
    dfs2(1, 1);
    build(1, 1, n);
    scanf("%d", &m);
    while(m--)
    {
        string s;
        int x;
        cin >> s;
        scanf("%d", &x);
        if(s[0] == 'i')
        {
            printf("%lld\n", install(x + 1));
        }
        else if(s[0] == 'u')
        {
            printf("%lld\n", uninstall(x + 1));
        }
        else
        {
            puts("Youwike AK IOI!");
        }
    }
    return 114514;
}
```
