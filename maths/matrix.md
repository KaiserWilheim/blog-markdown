---
title: 矩阵
date: 2022-05-07
tags:
	- 数学
	- 线性代数
categories:
	- 线性代数
mathjax: true
---

矩阵与矩阵相关运算。

<!-- more -->

<div id="problem-card-vis">false</div>

在数学中，矩阵是一种很重要的表现形式；在OI中，矩阵也是一种很重要的数据结构。
矩阵通常可以用在线性齐次递推式的加速上，比如说加速斐波那契数列的递推过程。
矩阵还可以让多个数据关联起来，并简便地进行区间维护。

# 基本概念

为了更好地区分矩阵与其他量，我们这里将代表矩阵的符号进行了特殊处理，就像这个样子：

$A \to \mathbf{A}$

## 矩阵是什么

由 $n \times m$ 个元素组成的，形如
$$
\mathbf{A} =
\begin{bmatrix}
a_{1,1} & a_{1,2} & a_{1,3} & \dots & a_{1,m} \\\\
a_{2,1} & a_{2,2} & a_{2,3} & \dots & a_{2,m} \\\\
a_{3,1} & a_{3,2} & a_{3,3} & \dots & a_{3,m} \\\\
\vdots & \vdots & \vdots & \ddots & \vdots \\\\
a_{n,1} & a_{n,2} & a_{n,3} & \dots & a_{n,m}
\end{bmatrix}
$$
的 $n$ 行 $m$ 列的数表，我们称之为大小为 $n \times m$ 的矩阵，可以简记为 $\mathbf{A_{\mathcal{n \times m}}}$ 。

## 特殊的矩阵

特殊的矩阵由很多种，比如单位矩阵、上三角矩阵等等。

### 行矩阵与列矩阵

行矩阵就是只有一行的矩阵，列矩阵就是只有一列的矩阵。

### 零矩阵

元素全为 $0$ 的矩阵。
零矩阵简记为 $\mathbf{0}$。

### 负矩阵

对于一个矩阵 $\mathbf{A}$ 的负矩阵 $-\mathbf{A}$，其中每个元素都与矩阵 $\mathbf{A}$ 内相同位置的元素互为相反数。

### 方阵

方阵指的就是正方形的矩阵，其行数与列数相等。
此时，其行数（或列数，反正他们相等）就可以被称作该矩阵的阶。

简单来说，一个 $n$ 阶方阵 $\mathbf{A_{\mathcal{n}}}$ 其实就是相当于一个 $n \times n$ 的矩阵 $\mathbf{A_{\mathcal{n \times n}}}$。

### 单位矩阵

单位矩阵就是指，在主对角线上的元素都是 $1$，其余元素都为 $0$ 的矩阵。
主对角线就是指 $(1,1)$ 到 $(n,n)$。这也说明单位矩阵都是正方形的。

单位矩阵简记为 $\mathbf{I}$。

单位矩阵也有其相应的阶，比如1到4阶的单位矩阵分别是这个样子的：$\mathbf{I_{\mathrm{1}}}=\begin{bmatrix}1\end{bmatrix}$，$\mathbf{I_{\mathrm{2}}}=\begin{bmatrix}1&\\\\&1\end{bmatrix}$，$\mathbf{I_{\mathrm{3}}}=\begin{bmatrix}1&&\\\\&1&\\\\&&1\end{bmatrix}$，$\mathbf{I_{\mathrm{4}}}=\begin{bmatrix}1&&&\\\\&1&&\\\\&&1&\\\\&&&1\end{bmatrix}$。

形象一点，就是这样：
（为 $0$ 的元素太多时一般将其省略）

$$
\mathbf{I} = 
\begin{bmatrix}
1& & & \\\\
 &1& & \\\\
 & & \ddots & \\\\
 & & &1
\end{bmatrix}
$$

单位矩阵的一个很重要的性质就是，任何矩阵乘以单位矩阵的结果还是其本身，即 $\mathbf{AI} = \mathbf{IA} = \mathbf{A}$。

有时候还可能将单位矩阵称为 $\mathbf{E}$，但是很少见，我目前还没有见过。

## 矩阵的运算

矩阵可以进行四则运算，但是与正常的数字还是有所不同。

### 加减

矩阵的加减要求两个矩阵必须行数列数均相同才可以进行。

加减的时候，每一对位置相同的元素相加或者相减。

形象一点就是这个样子：

$$
\mathbf{A} \pm \mathbf{B}
=
\begin{bmatrix}
a & b & c \\\\
d & e & f
\end{bmatrix}
\pm
\begin{bmatrix}
g & h & i \\\\
j & k & l
\end{bmatrix}
=
\begin{bmatrix}
a \pm g & b \pm h & c \pm i \\\\
d \pm j & e \pm k & f \pm l
\end{bmatrix}
$$

矩阵的加法满足交换律和结合律。

### 数乘

就是一个矩阵乘以一个数。

乘起来的时候，矩阵内的每一个元素都要乘以这个数。

形象一点就是这样：

$$
λ\mathbf{A}
=
λ \times
\begin{bmatrix}
a & b & c \\\\
d & e & f
\end{bmatrix}
= 
\begin{bmatrix}
λa & λb & λc \\\\
λd & λe & λf
\end{bmatrix}
$$

矩阵的数乘满足交换律、结合律和分配律。

如果将刚才这个过程反过来的话，就叫做矩阵提公因子。

### 点乘

点乘是矩阵运算中很重要的一部分。

点乘又叫做矩阵乘法，（基本上）是OI中矩阵的精髓。

矩阵乘法不满足交换律，是因为其需要满足左侧矩阵的列数与右侧矩阵的行数相等。

形象一点，就是 $\mathbf{A_{\mathcal{n \times p}}} \times \mathbf{B_{\mathcal{p \times m}}} = \mathbf{C_{\mathcal{n \times m}}}$。

具体操作的时候是这个样子的：

我们以 $\mathbf{A_{\mathrm{3 \times 3}}} \times \mathbf{B_{\mathrm{3 \times 2}}}$ 为例。

$$
\begin{align}
& \mathbf{A_{\mathrm{3 \times 3}}} \times \mathbf{B_{\mathrm{3 \times 2}}} \\\\
={} &
\begin{bmatrix}
a_{1,1} & a_{1,2} & a_{1,3} \\\\ a_{2,1} & a_{2,2} & a_{2,3} \\\\ a_{3,1} & a_{3,2} & a_{3,3}
\end{bmatrix}
\times
\begin{bmatrix}
b_{1,1} & b_{1,2} \\\\ b_{2,1} & b_{2,2} \\\\ b_{3,1} & b_{3,2}
\end{bmatrix}\\\\
={} &
\begin{bmatrix}
a_{1,1}b_{1,1}+a_{1,2}b_{2,1}+a_{1,3}b_{3,1} & a_{2,1}b_{1,1}+a_{2,2}b_{2,1}+a_{2,3}b_{3,1} & a_{3,1}b_{1,1}+a_{3,2}b_{2,1}+a_{3,3}b_{3,1} \\\\
a_{1,1}b_{1,2}+a_{1,2}b_{2,2}+a_{1,3}b_{3,2} & a_{2,1}b_{1,2}+a_{2,2}b_{2,2}+a_{2,3}b_{3,2} & a_{3,1}b_{1,2}+a_{3,2}b_{2,2}+a_{3,3}b_{3,2}
\end{bmatrix}
\end{align}
$$

除了交换律以外，矩阵乘法只满足结合律和左、右分配律。

### 幂

矩阵的幂运算与正常的幂运算一样，都是自乘多少次，但是由于矩阵乘法的特殊性质，我们只能给方阵求幂。

$\mathbf{A^{\mathcal{k}}} = \underbrace{\mathbf{AAAA \cdots AA}}_{\text{k of}}$

根据这种性质，我们可以像对数字一样进行快速幂，逻辑与之前一样。

### 转置

转置就是将一个矩阵顺时针旋转90度。

例：
$$
\begin{bmatrix}
1&1&4\\\\5&1&4
\end{bmatrix}^T
=
\begin{bmatrix}
5&1\\\\1&1\\\\4&4
\end{bmatrix}
$$

我们使用 $\mathbf{A}^T$ 来表示矩阵 $\mathbf{A}$ 的转置。

转置满足以下法则：
$$
\begin{align}
(\mathbf{A}^T)^T &= \mathbf{A} \\\\
(\lambda \mathbf{A})^T &= \lambda \mathbf{A}^T \\\\
(\mathbf{AB})^T &= \mathbf{B}^T \mathbf{A}^T
\end{align}
$$

# 实现

我们可以使用一个二维数组来存储矩阵。

就像这个样子：

``` cpp
struct Matrix
{
	int n, m;
	int a[N][N];
};
```

定义一个初始化函数：

``` cpp
Matrix() {};
Matrix(int n, int m) : n(n), m(m) { memset(a, 0, sizeof(a)) };
```

重载一下运算符：

加：

``` cpp
friend Matrix operator + (const Matrix &lhs, const Matrix &rhs)
{
	Matrix res(lhs.n, lhs.m);
	for(int i = 1; i <= lhs.n; i++)
		for(int j = 1; j <= lhs.m; j++)
			res.a[i][j] = lhs.a[i][j] + rhs.a[i][j];
	return res;
}
```

乘：

``` cpp
friend Matrix operator * (const Matrix &lhs, const Matrix &rhs)
{
	Matrix res(lhs.n, rhs.m);
	for(int i = 1; i <= lhs.n; i++)
		for(int j = 1; j <= rhs.m; j++)
			for(int k = 1; k <= lhs.m; k++)
				res.a[i][j] += lhs.a[i][k] * rhs.a[k][j];
	return res;
}
```

乘方（快速幂）：

``` cpp
friend Matrix operator ^ (Matrix rhs, int x)
{
	Matrix res(rhs.n, rhs.n);
	for(int i = 1; i <= rhs.n; i++)res.a[i][i] = 1;
	while(x)
	{
		if(x & 1) res = res * rhs;
		rhs = rhs * rhs;
		x >>= 1;
	}
	return res;
}
```

再继续加一些其他的重载运算符之后就是这个样子：

{% note default 矩阵结构体 %}
``` cpp
struct Matrix
{
	int n, m;
	int a[N][N];
	Matrix() {};
	Matrix(int n, int m) : n(n), m(m) { memset(a, 0, sizeof(a)) };
	friend Matrix operator + (const Matrix &lhs, const Matrix &rhs)
	{//加
		Matrix res(lhs.n, lhs.m);
		for(int i = 1; i <= lhs.n; i++)
			for(int j = 1; j <= lhs.m; j++)
				res.a[i][j] = lhs.a[i][j] + rhs.a[i][j];
		return res;
	}
	friend Matrix operator - (const Matrix &lhs, const Matrix &rhs)
	{//减
		Matrix res(lhs.n, lhs.m);
		for(int i = 1; i <= lhs.n; i++)
			for(int j = 1; j <= lhs.m; j++)
				res.a[i][j] = lhs.a[i][j] - rhs.a[i][j];
		return res;
	}
	Matrix operator -() const
	{//取反
		Matrix res(n, m);
		for(int i = 1; i <= n; i++)
			for(int j = 1; j <= m; j++)
				res.a[i][j] = -a[i][j];
		return res;
	}
	friend Matrix operator * (const Matrix &lhs, const Matrix &rhs)
	{//点乘
		Matrix res(lhs.n, rhs.m);
		for(int i = 1; i <= lhs.n; i++)
			for(int j = 1; j <= rhs.m; j++)
				for(int k = 1; k <= lhs.m; k++)
					res.a[i][j] += lhs.a[i][k] * rhs.a[k][j];
		return res;
	}
	friend Matrix operator * (const Matrix &lhs, int k)
	{//数乘
		Matrix res(lhs.n, lhs.m);
		for(int i = 1; i <= lhs.n; i++)
			for(int j = 1; j <= lhs.m; j++)
				res.a[i][j] = lhs.a[i][j] * k;
		return res;
	}
	friend Matrix operator ^ (Matrix lhs, int n)
	{//快速幂
		Matrix res(lhs.n, lhs.n);
		for(int i = 1; i <= lhs.n; i++)res.a[i][i] = 1;
		while(x)
		{
			if(n & 1) res = res * lhs;
			lhs = lhs * lhs;
			x >>= 1;
		}
		return res;
	}
};
```
{% endnote %}

但其实更常见的是这样子：
``` cpp
struct Matrix
{
	int n, m;
	int a[N][N];
	Matrix() {};
	Matrix(int n, int m) : n(n), m(m) { memset(a, 0, sizeof(a)) };
	friend Matrix operator + (const Matrix &lhs, const Matrix &rhs)
	{
		Matrix res(lhs.n, lhs.m);
		for(int i = 1; i <= lhs.n; i++)
			for(int j = 1; j <= lhs.m; j++)
				res.a[i][j] = lhs.a[i][j] + rhs.a[i][j];
		return res;
	}
	friend Matrix operator * (const Matrix &lhs, const Matrix &rhs)
	{
		Matrix res(lhs.n, rhs.m);
		for(int i = 1; i <= lhs.n; i++)
			for(int j = 1; j <= rhs.m; j++)
				for(int k = 1; k <= lhs.m; k++)
					res.a[i][j] += lhs.a[i][k] * rhs.a[k][j];
		return res;
	}
	friend Matrix operator ^ (Matrix lhs, int n)
	{
		Matrix res(lhs.n, lhs.n);
		for(int i = 1; i <= lhs.n; i++)res.a[i][i] = 1;
		while(x)
		{
			if(n & 1) res = res * lhs;
			lhs = lhs * lhs;
			x >>= 1;
		}
		return res;
	}
};
```

# 应用

## 矩阵加速递推

就以矩阵加速斐波那契数列递推为例吧。

我们将第 $n$ 项斐波那契数简写为 $F_n$。

我们知道 $F_n = F_{n-1} + F_{n-2}$，我们就考虑把斐波那契数列的相邻两项放在一个行（或者列，根据个人习惯）矩阵里面，就像这个样子：$\begin{bmatrix}F_i&F_{i-1}\end{bmatrix}$。

我们需要把 $\begin{bmatrix}F_i&F_{i-1}\end{bmatrix}$ 变成 $\begin{bmatrix}F_{i-1}+F_i&F_i\end{bmatrix}$，同时需要用到矩阵乘法。

因为
$$
\begin{bmatrix}
a & b
\end{bmatrix}
\times
\begin{bmatrix}
x & y\\\\
z & w
\end{bmatrix}
=
\begin{bmatrix}
ax+bz & ay+bw
\end{bmatrix}
$$

所以我们如果想让 $\begin{bmatrix}a&b\end{bmatrix}$ 变成 $\begin{bmatrix}a+b&a\end{bmatrix}$ 的话，我们就需要将其乘以 $\begin{bmatrix}1&1\\\\1&0\end{bmatrix}$。

然后就是矩阵快速幂了。
因为我们一开始的 $\begin{bmatrix}1&1\end{bmatrix}$ 是 $\begin{bmatrix}F_2&F_1\end{bmatrix}$，所以我们只需要将其乘以 $\begin{bmatrix}1&1\\\\1&0\end{bmatrix}^{n-2}$ 即可得到 $F_n$。

### 如何构建转移矩阵

我们考虑我们是如何从一个状态转移到下一个状态的。

对于一个函数 $f(x)$，我们假定它具有这样的转移式子：

$$
f(x) = f(x-1) + 2f(x-2) + 2^x + 2
$$

我们的状态矩阵就是这个样子的：$\begin{bmatrix}f(x)&f(x-1)&2^x&1\end{bmatrix}$。

然后我们考虑我们如何从 $\begin{bmatrix}f(x-1)&f(x-2)&2^{x-1}&1\end{bmatrix}$ 变为 $\begin{bmatrix}f(x)&f(x-1)&2^x&1\end{bmatrix}$

我们将 $\begin{bmatrix}f(x)&f(x-1)&2^x&1\end{bmatrix}$ 展开，得

$$
\begin{align}
f(x) &= f(x-1) \times 1 + f(x-2) \times 2 + 2^{x-1} \times 2 + 1 \times 2 \\\\
f(x-1) &= f(x-1) \times 1 + f(x-2) \times 0 + 2^{x-1} \times 0 + 1 \times 0 \\\\
2^x &= f(x-1) \times 0 + f(x-2) \times 0 + 2^{x-1} \times 2 + 1 \times 0 \\\\
1 &= f(x-1) \times 0 + f(x-2) \times 0 + 2^{x-1} \times 0 + 1 \times 1
\end{align}
$$

然后我们就可以得到我们的转移矩阵了：

$$
\begin{bmatrix}
1&1&0&0\\\\
2&0&0&0\\\\
2&0&2&0\\\\
2&0&0&1
\end{bmatrix}
$$

## 矩阵辅助维护信息

这种一般就是对于那种同时需要维护多种信息，还需要支持一大堆复杂的操作，但是推式子的时候不会超过一次的那种题，就比如说这一个：

[THUSC 大魔法师](/solutions/solution-l2980/)

我们可以发现其操作（$A_i = A_i + k$，$B_i = B_i \times k$，$C_i = k$）均未出现两个未知量相乘的情况，即可判定其可以利用矩阵乘法来维护。




