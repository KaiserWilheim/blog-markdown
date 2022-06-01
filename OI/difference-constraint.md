---
title: 差分约束
date: 2022-03-30
tags:
	- 算法
categories:
	- 算法
mathjax: true
---

差分约束系统。

<!--more-->

我们经常会遇到一些问题，就是给出一堆未知量，再给出一堆类似“某一个未知量比另一个未知量最多（最少）大（小）多少”的问题，让我们从这一团繁杂的毛钱中拎出一根线头来，以求得这个毛线团的一组可行解。

比如说下面这个东西：

$$
\begin{cases}
x_1 \geq x_2 - 10 \\\\
x_1 \leq x_3 - 20 \\\\
x_3 \leq x_2 + 10 \\\\
x_4 \leq x_3 - 10 \\\\
x_2 \leq x_4 - 10
\end{cases}
$$

我们就基本上无从下手。

这时候，我们就需要引入一个新的算法——**差分约束**。

差分约束通过将不等式的问题转化为最短路（或最长路）问题来求解。

# 最短路？为什么？

回忆我们学习最短路的时候学到的知识。

假设对于两个点 $a$ 和 $b$，如果其间有一条边长为 $w$ 的有向边 $a \to b$，那么它们的 $dis$ 一定满足 $dis[b] \leq dis[b] + w$。

我们如果将每一个未知量 $x_k$ 与每一个点的 $dis[i]$ 联系起来的话，那么我们就可以得到形如 $x_b \leq x_a + w$ 的一堆不等式。

而我们刚好需要这些不等式。

于是我们就可以将我们手中的不等式组转化为一堆边，并将其放到图里面，建成一个有向图。

最后得出的每一组合法的最短距离，都对应了一组不等式组的解。

我们首先把所有的式子转化为 $x_u \leq x_v + w$ 的形式，再从每一个 $v$ 向 $u$ 建一条边权为 $w$ 的有向边。

这个有向图可以有环，毕竟环是不影响我们的求值环节的。
但是它不能有负环。

比如说下面这一组边：

$$
\begin{cases}
x_2 \leq x_1 + 10 \\\\
x_3 \leq x_2 + 10 \\\\
x_1 \leq x_3 - 30
\end{cases}
$$

建到图上就是这个样子：

![diffcon1.png](https://s2.loli.net/2022/03/30/d9ltROoBAY178Uy.png)

根据原始的不等式组，最终我们会得到 $x_1 \leq x_1 - 10$ 这样一个奇怪的式子，从而导致不等式组无解。

从图上看，这三条边构成了一个负环。

所以说，一个负环最终会导致出现一些奇怪的不等式，最终导致不等式组无解。
（当然，如果使用的是最长路的话就是正环）

那我们怎么判负环？（或者说正环）

SPFA！
（SPFA信徒狂喜）

# 实现

但是我们光靠这些边实际上是不能得出最终解的。
很多情况下，这张图根本不联通。
但是这样的图还是可以找到至少一组可行解的。

所以我们还需要建一个超级源点，向每一个点连一条长度为0的边。

# 应用

具体情况下，我们不仅有类似 $x_a \leq x_b + w$ 这样的式子，还有其他的一些奇奇怪怪的约束条件。

下面列出了一些常见的约束条件和解决办法：

| 约束条件 | 解决办法 |
| - | - |
| $x_a \leq w$ | 将源点到 $a$ 的边权从0改到 $w$。 |
| $x_a \geq w$ | 从 $a$ 向源点连一条长为 $-w$ 的边。 |
| $x_a = x_b + w$ | 将其拆分为 $x_a \leq x_b + w$ 和 $x_b \leq x_a + (-w)$。 |
| $x_a + x_b \leq w$ | 差分约束会寄。请注意一下题目中有没有可以利用的其他特殊性质。 |


如果在一个不等式组的约束下（不等式组有解），想求出 $x_i − x_j$ 的最大值呢？
首先一定有 $x_i − x_j \leq j$ 到 $i$ 的"最短" 路 $\operatorname{dis}(j,i)$ 。因为我可以先走到 $j$ ，然后走“$j$ 到 $i$ 的"最短路"”到 $i$ 。
然后我们证明 $x_i − x_j$ 可以取到这个值。
这相当于往不等式组中添加一个 $x_i − x_j \geq \operatorname{dis}(j,i)$ ，如果不等式组仍有解，$x_i − x_j$ 就能取到 $\operatorname{dis}(j,i)$ 。
这也相当于在图中添加一条边，从 $i$ 到 $j$ ，边权是 $−\operatorname{dis}(j,i)$ 。这样加边一定不会出现负环，因为 $\operatorname{dis}(j,i)$ 是 $j$ 到 $i$ 的最短路，要有负环的话就有别的路径长度 $< \operatorname{dis}(j,i)$ 了。
如果要求 $x_i − x_j$ 的最小值，就是求 $x_j − x_i$ 的最大值的相反数，即 $−\operatorname{dis}(i,j)$。
“$x_i − x_j$ 的最小值”$\leq$“$x_i − x_j$ 的最大值”，对应了 $−\operatorname{dis}(i,j) \leq \operatorname{dis}(j,i)$ ，也就是 $\operatorname{dis}(i,j) + \operatorname{dis}(j,i) \geq 0$ ，也就对应了图中没有包含 $i,j$ 的负环。

# 代码

洛谷[板子题](https://www.luogu.com.cn/problem/P5960)

参考代码：[`Luogu P5960`](https://gitee.com/kaiserwilheim/OIcodes/blob/master/Luogu/p5000-p5999/p5960/p5960.cpp)

# 例题

## Luogu P1993 小 K 的农场

题目链接：https://www.luogu.com.cn/problem/P1993

接近板子题。

我们可以使用我们刚刚学到的技巧来完成这道题目。

参考代码：[`Luogu P1993`](https://gitee.com/kaiserwilheim/OIcodes/blob/master/Luogu/p1000-p1999/p1993/p1993.cpp)

## Luogu P6145 [USACO20FEB] Timeline G

题目链接：https://www.luogu.com.cn/problem/P6145

由于保证有解，而且我们得到的约束条件又都形如“第 $b$ 次挤奶在第 $a$ 次挤奶结束至少 $x$ 天后进行”，所以我们得到的都是负权边。

我们可以进行 dp，也可以跑最短路。

参考代码：[`Luogu P6145`](https://gitee.com/kaiserwilheim/OIcodes/blob/master/Luogu/p6000+/p6145/p6145.cpp)

## Luogu P3275 [SCOI2011] 糖果

题目链接：https://www.luogu.com.cn/problem/P3275

我们遇到了新的约束条件：不带取等的不等式。

我们看一下题目的条件：**分糖果**。

由于糖果是一块一块的，我们不能分给小朋友们半块糖果或 $\lim_{m \to 0}$ 块糖果，所以我们可以尝试着更改一下约束条件。
我们可以将 $x_a > x_b$ 改为 $x_a \geq x_b + 1$。

这样就可以建图了。

参考代码：[`Luogu P3275`](https://gitee.com/kaiserwilheim/OIcodes/blob/master/Luogu/p3000-p3999/p3275/p3275.cpp)

