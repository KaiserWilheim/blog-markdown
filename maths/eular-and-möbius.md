---
title: 欧拉和莫比乌斯
zh-CN: true
date: 2022-02-07
tags:
- 数学
categories:
 数学
mathjax: true
---

简介：欧拉定理，欧拉函数和莫比乌斯函数

<!--more-->

初稿写于2021-10-10，
再修改于2022-02-07

**Achtung: 本文章使用p来代指“任意质数”，请勿混淆。**

首先让我们膜拜一下莱昂哈德·欧拉(Leonhard Euler):

![eular](/pics/eular.jpg)

# 欧拉函数

欧拉函数 $φ(n)$ 的概念是在 $[0,n-1]$ 范围内有多少个整数与 $n$ 互质。

其中，我们规定 $φ(1)=1$，

且不难发现，$φ(p)=p-1$。

## 欧拉函数的一些性质

### 欧拉函数是积性函数

即 $φ(mn) = φ(m) φ(n)$

并且当 $n \bmod{2} \equiv 1$ 时，$φ(2n) = φ(n)$，

而当 $n \bmod{2} \equiv 0$（即 $2|n$)时，$φ(2n) = 2 φ(n)$

### $\displaystyle \sum_{m|n}^{n} m = n \frac{φ(n)}{2}$

$$
\sum_{m|n}^{n} m = n \frac{φ(n)}{2}
$$

易证。

### $\displaystyle \sum_{d|n} φ(d) = n$

$$
\sum_{d|n} φ(d) = n
$$

我们利用莫比乌斯反演的相关知识可以得出。

或者我们这样考虑：
如果 $\gcd(k,n) = d$ ，那么 $\gcd(\frac{k}{d} , \frac{n}{d}) = 1 (k < n)$ 。
我们如果设 $f(x)$ 为满足 $\gcd(k,n) = x$ 的数的个数，那么 $n = \sum_{i-1}^n f(i)$ 。
根据上面的证明，我们可以发现， $f(x) = φ(\frac{n}{x})$ ，从而得到 $n = \sum_{d|n} φ(\frac{n}{d})$ 。注意此时我们的约数 $d$ 和 $\frac{n}{d}$ 具有对称性，所以上面的式子可以化为 $n = \sum_{d|n} φ(d)$ 。

### $\displaystyle φ(p^k) = p^k - p^{k-1}$

若 $n = p^k$ ，那么 $φ(n) = p^k - p^{k-1}$ 。

由定义可知。
因为有 $n \perp p^k \iff p \not \mid n$ 。在 $\{ 0,1,p,\cdots ,p^k-1 \}$ 中， $p$ 的倍数是 $\{ 0,p,2p,\cdots ,p^k-p \}$ ，共有 $p^{k-1}$ 个。

### $\displaystyle φ(\prod_{i=1}^s p_i^{k_i}) = \prod_{i=1}^s p_i^{k_i - 1} · (p_i - 1)$

由唯一分解定理，设 $n = \prod_{i=1}^s p_i^{k_i}$ ，则有 $φ(n) = \prod_{i=1}^s \frac{p_i - 1}{p_i}$。

证明：

$$
\begin{align}
φ(n) &= \prod_{i=1}^s φ(p_i^{k_i}) \\\\
&= \prod_{i=1}^s p_i^{k_i} - p_i^{k_i - 1} \\\\
&= \prod_{i=1}^s (p_i - 1) \times p_i^{k_i - 1} \\\\
&= \prod_{i=1}^s (1 - \frac{1}{p_i}) p_i^{k_i} \\\\
&= n \prod_{i=1}^s (1 - \frac{1}{p_i})
\end{align}
$$

## 计算欧拉函数的值

### 单点求值

我们不难发现，当 $m$ 是一个质数的整数幂 $p^k$ 时，我们有
$$
φ(p^k)=p^k-p^{k-1}
$$
;
如果 $m>1$ 不是一个质数的整数幂，那我们可以把 $m$ 拆分成 $m=m_1 m_2$ ，其中 $m_1 \perp m_2$。这样这个数就可以在剩余系里表示为 $(n \bmod m_1 , n \bmod m_2)$ 。如果你不知道什么是剩余系，可以看我的博客：（咕了）。

根据
$$
k \perp m 且 k \perp n \iff k \perp mn
$$
和
$$
\gcd(m,n) = \gcd(n \bmod m,m)
$$
可得
$$
n \perp m \iff n \bmod m_1 \perp m_1 且 n \bmod m_2 \perp m_2
$$
，由此，我们可以推得
$$
φ(m)=φ(m_1)φ(m_2),m_1 \perp m_2
$$
所以，欧拉函数是一个积性函数。

欧拉函数的通项公式是
$$
φ(m)=\prod_{p|m} (p^{m_p}-p^{m_p-1})=m \prod_{p|m}(1-\frac{1}{p})
$$

上代码：

``` cpp
int euler_phi(int n)
{
	int m = int(sqrt(n + 0.5));
	int ans = n;
	for(int i = 2; i <= m; i++)
		if(n % i == 0)
		{
			ans = ans / i * (i - 1);
			while(n % i == 0) n /= i;
		}
	if(n > 1) ans = ans / n * (n - 1);
	return ans;
}
```

如果将代码改为下面的形式可以优化一些：

``` cpp
int euler_phi(int n)
{
	int ans = n;
	for(int i = 2; i * i <= n; i++)
		if(n % i == 0)
		{
			ans = ans / i * (i - 1);
			while(n % i == 0) n /= i;
		}
	if(n > 1) ans = ans / n * (n - 1);
	return ans;
}
```

### 区间筛

而我们可以使用筛法来求连续区间的欧拉函数值：

``` cpp
const int N = 1e7 + 5;
int phi[N], prime[N / 100];
bool visit[N];
void getphi(int n)
{
	phi[1] = 1;
	for(int i = 2; i <= n; i++)
	{
		if(!visit[i])
		{
			prime[prime[0] + 1] = i;
			prime[0]++;
			phi[i] = i - 1;
		}
		for(int j = 1; j <= prime[0] && i * prime[j] <= n; j++)
		{
			visit[i * prime[j]] = true;
			if(!(i % prime[j]))
			{
				phi[i * prime[j]] = phi[i] * prime[j];
				break;
			}
			else
			{
				phi[i * prime[j]] = phi[i] * (prime[j] - 1);
			}
		}
	}
}
```

还有OI Wiki上提供的代码：

{% note default from OI Wiki %}
``` cpp
void pre()
{
	memset(is_prime, 1, sizeof(is_prime));
	int cnt = 0;
	is_prime[1] = 0;
	phi[1] = 1;
	for(int i = 2; i <= 5000000; i++)
	{
		if(is_prime[i])
		{
			prime[++cnt] = i;
			phi[i] = i - 1;
		}
		for(int j = 1; j <= cnt && i * prime[j] <= 5000000; j++)
		{
			is_prime[i * prime[j]] = 0;
			if(i % prime[j])
				phi[i * prime[j]] = phi[i] * phi[prime[j]];
			else
			{
				phi[i * prime[j]] = phi[i] * prime[j];
				break;
			}
		}
	}
}
```
{% endnote %}

复杂度均为线性。

# 欧拉定理

我们先看一下费马小定理。

## 费马小定理

众所周知，费马小定理是：
$$
a^{p-1} \equiv 1 \pmod{p},a \perp p
$$
还有另一种形式，是这样的：
$$
\forall a \in \mathbb{N} ,a^p \equiv a \pmod{p}
$$
。

### 证明

我们首先取一个不为 $p$ 倍数的数 $a$ 。
构造一个序列： $A = \lbrace 1,2,3, \cdots , p - 1 \rbrace$ ，这个序列拥有这样的一个性质：
$$
\prod_{i=1}^n A_i \equiv \prod_{i=1}^n (A_i \times a) \pmod{p}
$$

证明：
$$
\because (A_i,p) = 1 , (A_i \times a,p) = 1
$$
又因为每一个 $A_i \times a \pmod{p}$ 都是独一无二的，且 $A_i \times a \pmod{p} < p$ ，得证（每一个 $A_i \times a$ 都对应了一个 $A_i$）。
设 $f = (p-1)!$，则
$$
\begin{align}
f &\equiv (a \times A_1)(a \times A_2)(a \times A_3) \cdots (a \times A_{p-1}) \pmod{p} \\\\
a^{p-1} · f &\equiv f \pmod{p} \\\\
a^{p-1} &\equiv 1 \pmod{p}
\end{align}
$$
证毕。

或者也可以使用归纳法证明：

显然 $1^p \equiv 1 \pmod{p}$。那么假设 $a^p \equiv a \pmod{p}$ 成立，那么通过二项式定理有
$$
(a+1)^p = a^p + C_p^1 a^{p-1} + C_p^2 a^{p-2} + \cdots + C_p^{p-1} a + 1
$$
因为 $C_p^k = \frac{p!}{(n-k)!k!}$ 对于 $k \in \[ 1,p-1 \]$ 成立，在模 $p$ 意义下 $C_p^1 \equiv C_p^2 \equiv \cdots \equiv C_p^{p-1} \equiv 0 \pmod{p}$，那么 $(a+1)^p \equiv a^p + 1 \pmod{p}$，将 $a^p \equiv a \pmod{p}$ 代入得 $(a+1)^p \equiv a+1 \pmod{p}$。
证毕。

## 欧拉定理

欧拉定理的内容如下：

若 $\gcd(a,m) = 1$，则 $a^{φ(m)} \equiv 1 \pmod{m}$。

### 证明

欧拉定理的证明过程与费马小定理的证明过程十分相似。
我们同样首先构造一个与 $m$ 互质的序列，再进行操作。

设 $r_1,r_2, \cdots ,r_{φ(m)}$ 为模 $m$ 意义下的一个简化剩余系，则 $ar_1,ar_2, \cdots ,ar_{φ(m)}$ 同样也为模 $m$ 意义下的一个简化剩余系。所以，$\prod_{i=1}^{φ(m)} r_i \equiv \prod_{i=1}^{φ(m)} ar_i \equiv a^{φ(m)} \prod_{i=1}^{φ(m)} r_i \pmod{m}$，约去 $\prod_{i=1}^{φ(m)} r_i$ 后可得 $a^{φ(m)} \equiv 1 \pmod{m}$。
证毕。

当 $m=p$ 时，由于 $φ(m) = m-1$，将之代入可得费马小定理。

## 扩展欧拉定理

扩展欧拉定理是这个样子的：

$$
a^b \equiv \begin{cases}a^{b \ \bmod \ φ(m)}, & \gcd(a,m) = 1,\\\\a^b, & \gcd(a,m) \neq 1, b < φ(m),\\\\a^{(b \ \bmod \ φ(m))+φ(m)}, & \gcd(a,m) \neq 1,b \geq φ(m). \end{cases} \pmod{m}
$$

证明见[OI Wiki](https://oi-wiki.org/math/number-theory/fermat/#_6)

# 莫比乌斯函数

## 定义

我们把一个数字分解质因数为 $n=p_1^{c_1}p_2^{c_2} \cdots p_k^{c_k}$，则
$$
μ(n)=
\begin{cases}
1 & n=1 \\\\
0 & \forall i \in [1,k] , c_i > 1 \\\\
(-1)^k & \forall i \in [1,k] , c_i = 1 
\end{cases}
$$

解释一下：
1. 当 $n=1$ 时， $μ(n)=1$；
2. 当 $n \neq 1$ 时：
    1. 当存在 $i\in [1,k]$，使得 $c_i > 1$ 时，$μ(n)=0$，也就是说只要某个质因子出现的次数超过一次，$μ(n)$ 就等于 $0$；
    2. 当任意 $i\in[1,k]$，都有 $c_i=1$ 时，$μ(n)=(-1)^k$，也就是说每个质因子都仅仅只出现过一次时，$μ(n)$ 等于 $-1$ 的 $k$ 次幂，此处 $k$ 指的便是仅仅只出现过一次的质因子的总个数。

## 性质

莫比乌斯函数不仅是积性函数，还有如下性质：
$$
\sum_{d|m} μ(d) = [m==1]
$$
其中 $[m==1]$ 代表 ```m==1?1:0``` 。

## 证明

设
$$
n=\prod_{i=1}^k{p_i}^{c_i},n'=\prod_{i=1}^k p_i
$$
那么
$$
\sum_{d\mid n}μ(d)=\sum_{d\mid n'}μ(d)=\sum_{i=0}^k C_k^i·(-1)^i=(1+(-1))^k
$$

## 莫比乌斯反演

如果两个的函数 $f(n)$ 与 $g(n)$ 满足
$$
f(n) = \sum_{d|n} g(d)
$$
则
$$
g(n) = \sum_{d|n} μ(d) f(\frac{n}{d})
$$

### 证明

{% note info 摘自《混凝土数学》 %}

#### 充分性

$$
\begin{align}
f(n) &= \sum_{d|n} g (d)\\\\
&= \sum_{d|n} g (\frac{n}{d})
\end{align}
$$

$$
\sum_{d|n} μ(d) f(\frac{n}{d}) = \sum_{d|n} μ(d) \sum_{d_1|\frac{n}{d}} g(d_1)
$$

$$
\begin{align}
\sum_{d|n} \sum_{d_1|\frac{n}{d}} μ(d) g (d_1) &= \sum_{d_1|n} g(d_1) \sum_{d|\frac{n}{d_1}} μ(d) \\\\
&= g(n)
\end{align}
$$

考虑到
$$
\sum_{d|\frac{n}{d_1}} μ(d) = 
\begin{cases}
1 & d_1 = n \\\\
0 & d_1 < n
\end{cases}
$$
因此
$$
\begin{align}
g(n) &= \sum_{d|n} μ(d) f(\frac{n}{d}) \\\\
&= \sum_{d|n} μ(\frac{n}{d}) f(d)
\end{align}
$$

#### 必要性

$$
\begin{align}
g(n) &= \sum_{d|n} μ(d) f(\frac{n}{d}) \\\\
&= \sum_{d|n} μ(\frac{n}{d}) f(d)
\end{align}
$$

$$
\begin{align}
\sum_{d|n} g(d) &= \sum_{d|n}g(\frac{n}{d}) \\\\
&= \sum_{d|n} \sum_{d_1|\frac{n}{d}} μ(\frac{n}{d d_1})f(d_1) \\\\
&= \sum_{d d_1|n} μ(\frac{n}{d d_1}) f(d_1) \\\\
&= \sum_{d_1|n} f(d_1) \sum_{d|\frac{n}{d}} μ(\frac{n}{d d_1}) \\\\
&= f(n)
\end{align}
$$

考虑到

$$
\begin{align}
\sum_{d|\frac{n}{d_1}} μ(\frac{n}{d d_1}) &= \sum_{d|\frac{n}{d_1}} μ(d) \\\\
&= \begin{cases} 1 & d_1 = n \\\\ 0 & d_1 < n \end{cases}
\end{align}
$$

因此

$$
\begin{align}
f(n) &= \sum_{d|n} g(d) \\\\
&= \sum_{d|n} g(\frac{n}{d})
\end{align}
$$

{% endnote %}

## 求莫比乌斯函数的值

### 单点求值

自己算。

### 区间筛法

``` cpp
void getMu()
{
	mu[1] = 1;
	for(int i = 2; i <= n; ++i)
	{
		if(!flg[i]) p[++tot] = i, mu[i] = -1;
		for(int j = 1; j <= tot && i * p[j] <= n; ++j)
		{
			flg[i * p[j]] = 1;
			if(i % p[j] == 0)
			{
				mu[i * p[j]] = 0;
				break;
			}
			mu[i * p[j]] = -mu[i];
		}
	}
}
```

## 莫比乌斯变换

设 $f(n)$，$g(n)$ 为两个数论函数。
如果有 $f(n) = \sum_{d|n} g(d)$ 那么有：

形式1： $g(n) = \sum_{d|n} μ(d) f(\frac{n}{d})$。

这种形式下，函数 $f(n)$ 被称为函数 $g(n)$ 的莫比乌斯变换，反之则称之为其的莫比乌斯逆变换（反演）。

形式2： $g(n) = \sum_{d|n} μ(\frac{d}{n}) f(d)$。

据说这种形式更常考一点。

### 证明

- 方法一：对原式做数论变换。

$$
\begin{align}
& \sum_{d\mid n}μ(d)f(\frac{n}{d}) \\\\
=& \sum_{d\mid n}μ(d)\sum_{k\mid \frac{n}{d}}g(k) \\\\
=& \sum_{k\mid n}g(k)\sum_{d\mid \frac{n}{k}}μ(d) \\\\
=& g(n)
\end{align}
$$

用 $\displaystyle\sum_{d\mid n}g(d)$ 来替换 $f(\dfrac{n}{d})$，再变换求和顺序。最后一步变换的依据：$\displaystyle\sum_{d\mid n}μ(d)=[n=1]$，因此在 $\dfrac{n}{k}=1$ 时第二个和式的值才为 $1$。此时 $n=k$，故原式等价于 $\displaystyle\sum_{k\mid n}[n=k]\cdot g(k)=g(n)$

- 方法二：运用卷积。

原问题为：已知 $f=g * 1$，证明 $g=f * μ$

易知如下转化：$f * μ=g * 1 * μ\implies f * μ=g$（其中 $1 * μ=ε$）。

对于第二种形式：

类似上面的方法一，我们考虑逆推这个式子。

$$
\begin{align}
& \sum_{n|d}{μ(\frac{d}{n})f(d)} \\\\
=& \sum_{k=1}^{+\infty}{μ(k)f(kn)} \\\\
=& \sum_{k=1}^{+\infty}{μ(k)\sum_{kn|d}{g(d)}} \\\\
=& \sum_{n|d}{g(d)\sum_{k|\frac{d}{n}}{μ(k)}} \\\\
=& \sum_{n|d}{g(d)ε(\frac{d}{n})} \\\\
=& g(n)
\end{align}
$$

我们把 $d$ 表示为 $kn$ 的形式，然后把 $f$ 的原定义代入式子。

发现枚举 $k$ 再枚举 $kn$ 的倍数可以转换为直接枚举 $n$ 的倍数再求出 $k$，发现后面那一块其实就是 $ε$, 整个式子只有在 $d=n$ 的时候才能取到值。

# 参考文献

《混凝土数学》
OI Wiki

