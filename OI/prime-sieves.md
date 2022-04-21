---
title: 质数筛法
zh-CN: true
date: 2021-09-16
tags:
- 数论
- 学习笔记
- 算法
categories: 
 学习笔记
mathjax: true
---
简介：质数筛

<!--more-->

**质数** 。
一类很奇异的数字。

质数的定义是：
**“在大于1的自然数中，除了1和它本身以外不再有其他因数的自然数。”**

于是我们就可以想到一个简单的算法来求 $n$ 是不是质数：

## 简单的算法

``` cpp
for(int i = 2; i <= sqrt(n); i++)
{
    if(n % i == 0)
    {
        cout << "いいえ" << endl;
        break;
    }
}
cout << "はい" << endl;
return 0;
```

这个算法的时间复杂度是 $O(\sqrt[]{n})$ 。

代码好写归好写，但是时间复杂度真的是太高了，有时候并不能满足我们的需要；
更何况我们经常需要的操作是找出 $n$以内的所有质数，而不是简单地判断一个数是否是质数。

从$[1,n]$ 范围内的所有整数里挑质数其实很简单。

很多人大概接触过这样一种方法：

+ 首先列出所有数字并按照从大到小的顺序排好：

![](https://i.loli.net/2021/09/16/9FUrb7uVhWJ3da2.png)

+ 然后拿出五颜六色的彩笔，开始对数据进行处理。
+ $1$ 既不是质数也不是合数，提前把它剔除

![](https://i.loli.net/2021/09/16/ZrR3XAqTa46xGHI.png)

+ 然后看数列的第一个数： $2$ 。拿笔圈出2，并且划掉2的所有倍数（也就是所有剩下的偶数）。

![](https://i.loli.net/2021/09/16/8R3H9v1YntmDB6N.png)

+ 数列的下一个数是 $3$ 。换个颜色的笔圈出3，并划掉所有剩下的3的倍数。

![](https://i.loli.net/2021/09/16/hDGu9gJq4dcfPsw.png)

+ 越过被划掉的4，下一个数是 $5$ 。操作我就不用说了吧。

![](https://i.loli.net/2021/09/16/p8vdWwcxgi32ZuV.png)

+ 下一个是 $7$ 。

![](https://i.loli.net/2021/09/16/nHqIJQ2uEKRs9if.png)

+ 越过8,9和10，我们来到了 $11$ 。由于11大于 $\sqrt[]{100}$ ，也就是10，我们就停止操作，换一个颜色的笔，圈出剩下的所有数。

![](https://i.loli.net/2021/09/16/XGn68kx5Cp9iVoe.png)

+ 擦掉所有划去的数，我们得到了100以内的所有质数。

![](https://i.loli.net/2021/09/16/cJADHQ4t73sPj61.png)

如果我们将这个操作用代码来实现，那就是：

## 埃氏筛

**埃氏筛** ，或者叫做 **埃拉托斯特尼筛法** ，利用的就是这种思想。

代码如下：

``` cpp
int n;
cin >> n;
bool visit[n];
memset(visit, false, n);
for(int i = 2; i <= n; i++)
{
    if(!visit[i])
    {
        if(n < i * i)break;//埃氏筛只需要剔除根号n以内的素数的倍数 
        for(int j = i * i; j <= n; j += i)visit[j] = true;//标记质数
    }
}
for(int i = 2; i <= n; i++)
{
    if(!visit[i])cout << i << endl;//输出 
}
```

埃氏筛法的时间复杂度只有 $O(n \log \log n)$ 。比起简单方法的 $O(\sqrt{n})$，埃氏筛法明显快了很多。

但是，埃氏筛法还可以进一步优化。

举个栗子：

数字 $223092870$ 可以被分解成 $2 \times 3 \times 5 \times 7 \times 11 \times 13 \times 17 \times 19 \times 23$ ，在埃氏筛法中会被搜索9次。

如果我们可以在这个数被搜索到的第一次就标记好，并且使得这个数在之后的过程中不会被再次搜索到，我们算法的速度就会快很多。

那么如何实现这一想法呢？

我们继续拿 $223092870$ 来举例。如果我们在开始判断之前就用 **&1** 的方法来去掉所有偶数呢？

显然会快很多。但是，类似 $4849845$  ( $3 \times 5 \times 7 \times 11 \times 13 \times 17 \times 19$ )

这样的数仍然会被不可避免地搜索到多次。

但是，如果我们保证 $4849845$ 这个数只在被分解成 $3 \times 1616615$ 的时候被搜索到并标记为合数，就可以实现这个想法。

于是，我们就得到了一种算法：

## 欧拉筛

**欧拉筛** ，又叫 **线性筛** ，在埃氏筛的基础上进行了优化，使得每个数被且只被搜索到一次。 

欧拉筛利用合数的 **最小质因子** 进行筛选。

我们可以打表看一下100以内的合数是怎么被推出来的。

![2~22](https://i.loli.net/2021/09/16/Vw8liybKC4zX6NO.png)

![23~50](https://i.loli.net/2021/09/16/fiZtvQesKYVyunW.png)、

当 $i>50$ 之后，因为 $i\times 2$ 已经超出100这个范围，无法推出任何更多的范围内的合数。

附上代码：

``` cpp
long long n;
cin >> n;
int prime[n];
bool visit[n];
memset(prime, 0, n);
memset(visit, false, n);
prime[0] = 0;
for(long long i = 2; i <= n; i++)
{
    if(!visit[i])
    {
        prime[prime[0] + 1] = i;
        prime[0]++;//记录质数 
    }
    for(long long j = 1; j <= prime[0] && i * prime[j] <= n; j++)
    {
        visit[i * prime[j]] = true;
        if(i % prime[j] == 0)
        {
            break;//判断是否是最小质因子 
        }
    }
}
for(long long i = 1; i <= prime[0]; i++)
{
    if(n % prime[i] == 0)
    {
        visit[n] = true;
    }//最后一个一般筛不出来，特判一下 
}
for(long long i = 2; i <= n; i++)
{
    if(!visit[i])
    {
        cout << i << endl;//输出 
    }
}
```

**欧拉筛** 的时间复杂度是 $O(n)$ （所以这就是为什么它也叫 **线性筛** ），在数据量很大的时候，速度会比埃氏筛快很多。

如果需要更直观一点的话，可以看这个动画：

![质数.gif](https://i.loli.net/2021/10/08/fT1mI9ulFgQUobs.gif)

2倍速版：

![质数2x.gif](https://i.loli.net/2021/10/08/Vw6ugo5ZEvmORB3.gif)


----

当 $n$ 很大的时候，找出$n$以内的所有质数已经不太现实了。但是，判断 $n$ 是否为质数仍然是一种 ~~（看起来）~~ 可行的操作。

这就用到了：

## $Miller-Rabin$ 算法

$Miller-Rabin$ 算法涉及了两个定理：

### 费马小定理

费马小定理是这样的：

> 对于质数 $p$ 和任意整数 $a$ ，有 $a^p\equiv a\pmod{p}$。

这里我们需要用的是它的逆定理，即

> 若满足 $a^p\equiv a\pmod{p}$ ， $p$ 为质数。
此时我们两边同时约去一个 $a$ ，得 $a^{p-1} \equiv 1\pmod{p}$。

很遗憾，费马小定理的逆定理不是个真命题，但满足条件的 $p$ 还是有很大概率是质数的。

但是如果我们选取不同的 $a$ 进行多次测试，那么我们几乎就可以断定这个数是一个质数了。

### 二次探测定理

> 如果 $p$ 是奇素数，则 $x^2\equiv 1\pmod{p}$ 的解为 $x\equiv 1$ 或 $x\equiv p-1\pmod{p}$ 。

这是很容易证明的：

$$ 
\begin{array}
x^2 \equiv 1 \pmod{p} \\\\
x^2-1 \equiv 0 \pmod{p} \\\\
(x-1)(x+1) \equiv 0 \pmod{p} 
\end{array} 
$$

> 又∵ $p$ 为奇素数，其因子只有 $1$ 与 $p$ 两个。
∴只有两解 $x\equiv 1$ 与 $x\equiv p-1 \pmod{p}$ 。

如果 $a^{n−1}\equiv 1\pmod{n}$ 成立，$Miller−Rabin$ 算法不是立即找另一个 $a$ 进行测试，而是看 $n−1$ 是不是偶数。如果 $n−1$ 是偶数，另 $u=\dfrac{n-1}{2}$ ，并检查是否满足二次探测定理即 $a^u\equiv 1$ 或 $a^u\equiv n−1 \pmod{n}$ 。

将这两条定理合起来，也就是最常见的 Miller−Rabin 测试 。

单次时间复杂度是 $O(\log n)$ ，总复杂度就是 $O(times\times \log n)$ 了。

参考代码：

``` cpp
#include<cstdio>
#define ll long long 
inline int read()
{
    char c = getchar(); int x = 0, f = 1;
    while(c < '0' || c > '9') { if(c == '-') f = -1; c = getchar(); }
    while(c >= '0' && c <= '9') x = x * 10 + c - '0', c = getchar();
    return x * f;
}
int N, M, Test[10] = { 2, 3, 5, 7, 11, 13, 17 };
int pow(int a, int p, int mod)
{
    int base = 1;
    for(; p; p >>= 1, a = (1ll * a * a) % mod)
        if(p & 1) base = (1ll * base * a) % mod;
    return base % mod;
}
bool Query(int P)
{
    if(P == 1) return 0;
    int t = P - 1, k = 0;
    while(!(t & 1)) k++, t >>= 1;
    for(int i = 0; i < 4; i++)
    {
        if(P == Test[i]) return 1;
        ll a = pow(Test[i], t, P), nxt = a;
        for(int j = 1; j <= k; j++)
        {
            nxt = (a * a) % P;
            if(nxt == 1 && a != 1 && a != P - 1) return 0;
            a = nxt;
        }
        if(a != 1) return 0;
    }
    return 1;
}
int main()
{
    N = read(); M = read();
    while(M--) puts(Query(read()) ? "Yes" : "No");
    return 0;
}
```
[$\blacktriangleright$](https://www.cnblogs.com/zwfymqz/p/8150969.html)
