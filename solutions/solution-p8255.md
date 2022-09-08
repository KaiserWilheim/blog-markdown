---
title: P8255 [NOI Online 2022 入门组] 数学游戏 题解
date: 2022-07-01
tags:
	- 题解
	- 数学
categories:
	- 题解
mathjax: true
---
<br>
<!-- more -->
<div id="problem-card-vis">true</div>
<div id="problem-info-name">数学游戏</div>
<div id="problem-info-from">NOI Online-J 2022</div>
<div id="problem-info-difficulty">提高+ /省选-</div>
<div id="problem-info-color">#3498db</div>
<div id="problem-info-submit"><ul><li><a href="https://www.luogu.com.cn/problem/P8255">Luogu P8255</a></li><li><a href="https://www.acwing.com/problem/content/4391/">AcWing 4388</a></li></ul></div>

----

题目大意就是，我们给定了 $z$ 和 $x$，我们需要找到一个对应的 $y$，使得 $z = x \times y \times \gcd(x,y)$。

当然，我们不一定保证给出的 $z$ 是有解的，所以我们需要快速判定出无解的情况。

我们观察一下 $z$ 的组成。
首先 $z$ 必须被 $x$ 整除。这个非常显而易见。
然后 $z$ 必须不为 $0$。这个也很显而易见。
这是目前仅靠观察能看出来的无解的情况。

然后我们考虑将 $z$ 分解掉。

设 $\gcd(x,y)=g$。
因为 $\gcd(x,y)$ 肯定能够整除 $x$ 和 $y$，这样我们就可以将 $x$ 和 $y$ 分解为

$$
\begin{cases}
x = x_1 \times g \\\\
y = y_1 \times g
\end{cases}
$$

这样我们的 $z$ 就可以分解为 $z = x_1 \times y_1 \times g^3$。

我们将 $z$ 除以一个 $x$，得到的就是 $y_1 \times g^2$。

这里有一个 $g^2$，如果我们可以通过 $\sqrt{\dfrac{z}{x}}$ 来得到 $g$ 就好了。
可惜并不能，因为 $\sqrt{y_1}$ 并不能忽略不计。

我们考虑怎么把 $g^2$ 给单独拎出来。
我们在别的地方尝试找出来一个 $g$……

$z$ 里面显然是不能再找了，我们只能找 $x$。
想一想，$x^2$ 可以分解为 $x_1^2 \times g^2$。
我们如果可以通过 $\gcd(x_1^2 \times g^2,y_1 \times g^2)$ 来得到 $g^2$ 就好了。

实际上还真的可以。

因为我们将 $x$ 和 $y$ 分解的时候，我们找到的 $x_1$ 和 $y_1$ 实际上已经是互质的了。
我们求gcd的时候，无法再从 $x_1^2$ 和 $y_1$ 中得到新的公因数了。

所以我们对 $x^2$ 和 $\dfrac{z}{x}$ 求一手gcd就可以得到 $g^2$ 了。

这时候我们并不一定得到了 $g^2$，可能与原来的数据有所偏差。
我们给 $g^2$ 开一个根，看 $(\lfloor \sqrt{g^2} \rfloor)^2$ 是否等于 $g^2$。
如果不等，那就说明不存在一个整数范围内的 $g$，但是 $g$ 却一定在整数范围内，那就一定无解。
否则就继续算下去就可以了。毕竟我们只关心有没有解。

最后我们的 $y$ 就是 $\dfrac{z}{x \times g}$ 了。
输出答案即可。

参考代码如下：

``` cpp
#include<bits/stdc++.h>
using namespace std;
#define ll long long
ll gcd(ll a, ll b)
{
	return b == 0 ? a : gcd(b, a % b);
}
int main()
{
	int T;
	scanf("%d", &T);
	while(T--)
	{
		ll x, z;
		scanf("%lld%lld", &x, &z);
		if(z % x != 0)
		{
			puts("-1");
			continue;
		}
		else if(z == 0)
		{
			puts("-1");
			continue;
		}
		ll yg = z / x;
		ll gg = gcd(x * x, yg);
		ll g = sqrt(gg);
		if(gg != g * g)
		{
			puts("-1");
			continue;
		}
		printf("%lld\n", yg / g);
	}
	return 0;
}
```

