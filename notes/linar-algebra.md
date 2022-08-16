---
title: 线性代数
date: 2022-07-18
tags:
    - 数学
categories:
    - 学习笔记
mathjax: true
---

线性代数。

<!--more-->

2022年7月暑假推倒重写。
<div id="problem-card-vis">false</div>

----

# 向量

线性代数基本上可以说是基于向量的。

## 概念

向量可以被描述成一条有向线段。

决定一个向量的两个量是其长度和其方向。
这就意味着，在保证其长度和方向不变的情况下，我们可以将其随意地平移而不改变其本身。

我们用一个有序数列来描述一个向量。

比如说下面的这个向量，我们就可以用 $\begin{bmatrix}2\\\\1\end{bmatrix}$ 来表示。

[pic here]

这个 $2$ 和 $1$ 分别代表的是，这个向量可以被表示为 $2 {\color[RGB]{125,180,100}\hat{\imath}} + 1 {\color[RGB]{255,100,80}\hat{\jmath}}$。
这样，每一个向量都可以与一个有序数列一一对应了。

## 向量的运算

这里先介绍两种重要的向量运算：加法和数乘。

我们这里首先以二维向量做例子。

### 加法

向量加法，就是高中课本里面的向量加法，可以用平行四边形法则的那个。

其满足交换律和结合律，所以我们可以将两个向量拆开来看。

对于两个向量 $\vec{a}$ 和 $\vec{b}$，我们将其表示为 $\begin{bmatrix}{\color[RGB]{125,180,100}x_a}\\\\{\color[RGB]{255,110,80}y_a}\end{bmatrix}$ 和 $\begin{bmatrix}{\color[RGB]{125,180,100}x_b}\\\\{\color[RGB]{255,110,80}y_b}\end{bmatrix}$。
那么如果令 $\vec{c} = \vec{a} + \vec{b}$ 的话，$\vec{c}$ 就可以表示为 $\begin{bmatrix}{\color[RGB]{125,180,100}x_a}\\\\{\color[RGB]{255,110,80}y_a}\end{bmatrix} + \begin{bmatrix}{\color[RGB]{125,180,100}x_b}\\\\{\color[RGB]{255,110,80}y_b}\end{bmatrix}$。
将两式拆分的话，可以得到 $\vec{c} = ({\color[RGB]{125,180,100}x_a \hat{\imath}} + {\color[RGB]{255,110,80}y_a \hat{\jmath}}) + ({\color[RGB]{125,180,100}x_b \hat{\imath}} + {\color[RGB]{255,110,80}y_b \hat{\jmath}})$，合并同类项可得 $\vec{c} = \begin{bmatrix}{\color[RGB]{125,180,100}x_a+x_b}\\\\{\color[RGB]{255,110,80}y_a+y_b}\end{bmatrix}$。

[pic here]

### 数乘

向量的数乘可以看作是对这个向量进行缩放。

还是对于向量 $\begin{bmatrix}2\\\\1\end{bmatrix}$，我们让他乘以一个数 $\lambda$，就相当于是将其缩放了 $\lambda$ 倍，最终得到的向量是 $\begin{bmatrix}2\lambda\\\\\lambda\end{bmatrix}$。

推广到一般情况下，对于一个向量 $\begin{bmatrix}{\color[RGB]{125,180,100}x}\\\\{\color[RGB]{255,110,80}y}\end{bmatrix}$，我们将其乘以 $\lambda$，得到的向量是 $\begin{bmatrix}{\color[RGB]{125,180,100}\lambda x}\\\\{\color[RGB]{255,110,80}\lambda y}\end{bmatrix}$。

[pic here]

## 基底，张成空间与线性相关

### 基底

我们刚才提到的东西里面有两个没有解释，是 $\color[RGB]{125,180,100}\hat{\imath}$ 和 $\color[RGB]{255,100,80}\hat{\jmath}$。
$\color[RGB]{125,180,100}\hat{\imath}$ 长度为1，方向沿 $x$ 轴；$\color[RGB]{255,100,80}\hat{\jmath}$ 长度为1，方向沿 $y$ 轴。

[pic here]

我们在数学书上面可以看到，这两个向量叫做 $x$ 轴和 $y$ 轴上的单位向量。
根据刚才的例子可以看出，我们可以使用这两个向量组成二维平面内的所有向量，也就是说，所有的向量都可以表示为 $a{\color[RGB]{125,180,100}\hat{\imath}} + b{\color[RGB]{255,100,80}\hat{\jmath}}$ 的形式。

我们就称这样的一组向量为这个二维平面的一组**基底**。

当前这个二维平面的基底不止有这一种，只要我们选定的一组向量可以**张成**这个二维平面的话，这组向量也可以算是一组基底。

### 张成空间

“张成”这个术语其实很简单。

我们随便考虑两个向量，比如说 $\color[RGB]{210,100,110}\vec{u} = \begin{bmatrix}2\\\\1\end{bmatrix}$ 和 $\color[RGB]{80,150,220}\vec{v} = \begin{bmatrix}3\\\\-1\end{bmatrix}$。

[pic here]

对于二维平面中的任意一个向量，如果我们都能将其写成 $a{\color[RGB]{210,100,110}\vec{u}} + b{\color[RGB]{80,150,220}\vec{v}}$ 的形式的话，我们就可以说 $\color[RGB]{210,100,110}\vec{u}$ 和 $\color[RGB]{80,150,220}\vec{v}$ 所张成的空间就是当前我们这个二维平面，$\color[RGB]{210,100,110}\vec{u}$ 和 $\color[RGB]{80,150,220}\vec{v}$ 就是当前二维平面的一组基底。
我们称这个让一个向量等于 $a{\color[RGB]{210,100,110}\vec{u}} + b{\color[RGB]{80,150,220}\vec{v}}$ 的组成向量的方式为**“线性组合”**或者**“线性表出”**。也就是说，由$\color[RGB]{210,100,110}\vec{u}$ 和 $\color[RGB]{80,150,220}\vec{v}$ 所张成的空间中的所有向量都是 $\color[RGB]{210,100,110}\vec{u}$ 和 $\color[RGB]{80,150,220}\vec{v}$ 的线性组合，或可以由 $\color[RGB]{210,100,110}\vec{u}$ 和 $\color[RGB]{80,150,220}\vec{v}$ 线性表出。

二维平面的一组基底最少需要两个向量。
但这两个向量可不是随便两个就可以的。

假如说选中的两个向量共线的话，这两个向量所能线性组合而成的向量就（相对）少多了。这些向量只会存在在这两个向量所在的直线上。那么，这两个向量所张成的空间就是这一条直线。

为什么两个共线的向量就不行了呢？

因为他们**线性相关**。

### 线性相关

我们定义一组向量 ${\bf{A}}:\\{ \vec{a_1},\vec{a_2},\vec{a_3},\cdots,\vec{a_m} \\}$ 线性相关，当且仅当存在一组不全为 $0$ 的数 $k_1,k_2,k_3,\cdots,k_m$，使得:
<center>$k_1\vec{a_1} + k_2\vec{a_2} + k_3\vec{a_3} + \cdots + k_m\vec{a_m} = \vec{0}$</center>

换句话说，这一组向量里面有至少一个向量可以被组内其他向量线性表出。

在我们刚才举的第一个例子中，两个向量不能被对方线性表出，我们就称其为**线性无关**的。
而在第二个例子中，两个共线的向量可以被对方线性表出，我们就称其为**线性相关**的。

# 矩阵

矩阵可以看作是一系列向量的组合。

一般我们在学矩阵的时候，我们首先认识到的是矩阵的代数意义。
这样不是很直观，且死记硬背的成分较多。

这里我们通过矩阵与线性变换的结合，可以将矩阵的几何意义表示出来。
这样就较为直观，并且可能可以加深对某些操作的理解。

## 矩阵与线性变换

何为**线性变换**？

我们仍然以二维平面为例。

对于这样一个平面直角坐标系，我们可以搞出来一个映射，使得这个二维平面内的每一个点都可以映射到二维平面内的一个点。
将点转化成为向量，我们的这个映射就相当于是对于一个向量的函数，其输入值和输出值的类型都是二维向量。

这就称为一个**变换**。

而**线性**代表的就是，我们需要保证，在原先平面内的每一条直线，在变换后的平面内其仍然为一条直线；同时原点位置不变。

在上面这两条要求的约束下，我们就可以利用基底来对变换进行定量的描述了。

我们仍然利用 $\color[RGB]{125,180,100}\hat{\imath}$ 和 $\color[RGB]{255,100,80}\hat{\jmath}$ 来说明。

我们对一个平面直角坐标系进行线性变换的同时，其基底 $\color[RGB]{125,180,100}\hat{\imath}$ 和 $\color[RGB]{255,100,80}\hat{\jmath}$ 同时也会发生变化。

[pic here]

基于这两个新的基底，我们按照原来的数量关系构建出来的向量和直接拿向量去变换得到的结果是吻合的。

[pic here]

基于上面这个结论，我们就可以用 $\color[RGB]{125,180,100}\hat{\imath}$ 和 $\color[RGB]{255,100,80}\hat{\jmath}$ 经过变换后的位置来描述这个变换了。

假设我们变换之后的 $\color[RGB]{125,180,100}\hat{\imath}$ 和 $\color[RGB]{255,100,80}\hat{\jmath}$ 分别为 $\color[RGB]{125,180,100}\begin{bmatrix}2\\\\1\end{bmatrix}$ 和 $\color[RGB]{255,100,80}\begin{bmatrix}1\\\\3\end{bmatrix}$。
我们将两个向量写到一块：$\begin{bmatrix}{\color[RGB]{125,180,100}2}&{\color[RGB]{255,100,80}1}\\\\{\color[RGB]{125,180,100}1}&{\color[RGB]{255,100,80}3}\end{bmatrix}$。

假设我们要对向量 $\begin{bmatrix}-1\\\\2\end{bmatrix}$ 做这个变换的话，其变换后可以看作是 $-1{\color[RGB]{125,180,100}\begin{bmatrix}2\\\\1\end{bmatrix}} + 2{\color[RGB]{255,100,80}\begin{bmatrix}1\\\\3\end{bmatrix}}$，也就是 $\begin{bmatrix}0\\\\5\end{bmatrix}$。

### 矩阵乘法

（这里有一个矩阵乘法的[可视化工具](http://matrixmultiplication.xyz/)）

将上面的东西推广到一般，可以得到下面的式子：

<center>$\begin{bmatrix}{\color[RGB]{125,180,100}a}&{\color[RGB]{255,100,80}b}\\{\color[RGB]{125,180,100}c}&{\color[RGB]{255,100,80}d}\end{bmatrix} \begin{bmatrix}e\\f\end{bmatrix} = \begin{bmatrix}{\color[RGB]{125,180,100}a}e+{\color[RGB]{255,100,80}b}f\\{\color[RGB]{125,180,100}c}e+{\color[RGB]{255,100,80}d}f\end{bmatrix}$</center>

~~根据矩阵乘法的定义~~，我们可以知道这样就相当于是给一个向量左乘了一个方阵。~~（其实上面写的就是）~~

对于一次做多种变换，比如说对于向量 $\vec{v}=\begin{bmatrix}-1\\\\2\end{bmatrix}$，我们先做一个 ${\bf{U}}\_1=\begin{bmatrix}{\color[RGB]{125,180,100}2}&{\color[RGB]{255,100,80}1}\\\\{\color[RGB]{125,180,100}1}&{\color[RGB]{255,100,80}3}\end{bmatrix}$，再做一个 ${\bf{U}}\_2=\begin{bmatrix}{\color[RGB]{125,180,100}2}&{\color[RGB]{255,100,80}1}\\\\{\color[RGB]{125,180,100}0}&{\color[RGB]{255,100,80}1}\end{bmatrix}$。
根据上面的式子，我们可以推出来结果是这个：$\begin{bmatrix}{\color[RGB]{125,180,100}2}&{\color[RGB]{255,100,80}1}\\\\{\color[RGB]{125,180,100}0}&{\color[RGB]{255,100,80}1}\end{bmatrix} \begin{bmatrix}{\color[RGB]{125,180,100}2}&{\color[RGB]{255,100,80}1}\\\\{\color[RGB]{125,180,100}1}&{\color[RGB]{255,100,80}3}\end{bmatrix} \begin{bmatrix}-1\\\\2\end{bmatrix}$。

按照我们上面的描述，我们的运算顺序是这个样子的：${\bf{U}}\_2({\bf{U}}\_1\vec{v})$。
如果我们将 ${\bf{U}}\_2{\bf{U}}\_1$ 合并成为一个矩阵的话，结果是


















































































