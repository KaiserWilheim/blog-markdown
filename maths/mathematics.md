---
title: 数学杂项整理
date: 2021-09-21 
tags:
- 数学
- 学习笔记
categories:
 学习笔记
mathjax: true
---

这里是一个数学杂项（也就是太短暂时写不成博客）的整理。

<!--more-->

<div id="problem-card-vis">false</div>

# 质数

参见 **陈卓裕质数三大公理** ([Link is here](https://kamome.moe/index.php/2021/09/26/they_changed_math/)) 。

在 $[1,n]$范围内，质数个数的数量级是 $\dfrac{n}{\log n}$ 的，

即， $\pi(n) \sim O(\dfrac{n}{\log n})$

（此处的$\log$指的是$\ln$）

# 组合计数

## 排列数

排列数，通常用 $A$ (Arrangement)来表示。
公式为：

$$
\begin{equation}
A^n_m = \frac{n!}{(n-m)!}
\end{equation}
$$

还有以下递推式：

$$
A_n^m = m \times A_{n-1}^{m-1} + A_{n-1}^m
$$

## 组合数

组合数，通常用 $C$ (Conbination)来表示。

常见的表示方法有$\binom{m}{n}$ 和 $C^n_m$ 两种。

我这里用的是后者。
（据说这个是苏联写法，已经在机房里面成为梗了）

公式为：

$$
\begin{equation}
C^n_m = \frac{A^n_m}{A^n_n} = \frac{n!}{(n-m)!m!}
\end{equation}
$$

还有以下递推式：

$$
C_n^m = C_{n-1}^{m-1} + C_{n-1}^m
$$

### 快速计算组合数

``` cpp
int C(int n, int m)
{
	if(m > n || m < 0) return 0;
	return fac[n] * ifac[m] % mod * ifac[n - m] % mod;
}//组合数
```

### 常见组合恒等式

- $\displaystyle C_n^m = C_n^{n-m}$

因为

$$
C_n^m = \frac{n!}{(n-m)!m!} = \frac{n!}{(n-(n-m))!(n-m)!} = C_n^{n-m}
$$

- $\displaystyle m \times C_n^m = n \times C_{n-1}^{m-1}$

因为

$$
m \times C_n^m = \frac{n!}{(n-m)!(m-1)!} = n \times \frac{(n-1)!}{(n-m)!(m-1)!} = n \times C_{n-1}^{m-1}
$$

- $\displaystyle \sum_{i=0}^n C_n^i = 2^n$

就类似于考虑每一个物品到底是选还是不选。

- $\displaystyle C_n^0 + C_n^2 + \cdots = C_n^1 + C_n^3 + \cdots = 2^{n-1} (n \geq 1)$

跟上面的那个差不多。

- $\displaystyle \sum_{i=0}^n (-1)^i C_n^i = C_n^0 - C_n^1 + C_n^2 - C_n^3 + \cdots = \[ n==0 \]$

上面那个的衍生。

- $\displaystyle \sum_{i=0}^k C_n^i \times C_m^{k-i} = C_{n+m}^k$

考虑将 $n+m$ 的一堆物品分为一堆 $n$ 的和一堆 $m$ 的，分别从两堆里面取。
然后分别考虑从两堆里面各挑出来几个物品。

- $\displaystyle \sum_{i=0}^n (C_n^i)^2 = C_{2n}^n$

上面的那个的特殊化。
因为 $C_n^i = C_n^{n-i}$，所以我们可以将其替换为这样的形式。

# 上升幂与下降幂

## 上升幂

$$
x^{\bar{k}} = \prod_{i=i}^k (x+i-1)
$$

## 下降幂

$$
x^{\underline{k}} = \prod_{i=1}^k (x-i+1)
$$





