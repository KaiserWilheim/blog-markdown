---
title: 后缀数组
date: 2022-01-28
tags: 
	- 数据结构
  - 字符串
categories:
	字符串
mathjax: true
---

后缀数组(SA)。

<!--more-->

后缀数组主要维护的是两个数组：`sa[]`和`rk[]`。
其中，`sa[i]`表示将所有后缀进行排序之后第 $i$ 小的后缀的编号，`rk[i]`表示第 $i$ 个后缀的排名是多少。

我们容易看出，sa和rk两个数组互相指向对方，即`sa[rk[i]]=rk[sa[i]]=i`。

# 排序

我们知道了后缀数组维护的主要信息了，就可以开始排序了。
我们定义空字符比任何字符都要小，这样保证了长度不同的后缀可以进行排序。

## $O(n^2 \log n)$ 做法

我们可以直接使用`string`来存储数据，之后使用`sort`函数来进行排序。
由于比较两个字符串的时间复杂度是 $O(n)$ 的，所以总的时间复杂度是 $O(n^2 \log n)$ 的。

## $O(n \log^2 n)$ 做法

我们考虑倍增优化排序。

我们首先对长度较小的后缀进行排序，之后将已经排好序的子串当做第二关键字进行排序。每一次的第二关键字的长度是倍增的，所以我们可以优化出来一个log。

{% note default 参考代码 from OI Wiki %}
``` cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <iostream>
using namespace std;
const int N = 1000010;
char s[N];
int n, w, sa[N], rk[N << 1], oldrk[N << 1];
// 为了防止访问 rk[i+w] 导致数组越界，开两倍数组。
// 当然也可以在访问前判断是否越界，但直接开两倍数组方便一些。
int main() {
  int i, p;
  scanf("%s", s + 1);
  n = strlen(s + 1);
  for (i = 1; i <= n; ++i) sa[i] = i, rk[i] = s[i];
  for (w = 1; w < n; w <<= 1) {
    sort(sa + 1, sa + n + 1, [](int x, int y) {
      return rk[x] == rk[y] ? rk[x + w] < rk[y + w] : rk[x] < rk[y];});  // 这里用到了 lambda
    memcpy(oldrk, rk, sizeof(rk));
    // 由于计算 rk 的时候原来的 rk 会被覆盖，要先复制一份
    for (p = 0, i = 1; i <= n; ++i) {
      if (oldrk[sa[i]] == oldrk[sa[i - 1]] && oldrk[sa[i] + w] == oldrk[sa[i - 1] + w]) {
        rk[sa[i]] = p;
      } else {
        rk[sa[i]] = ++p;
      }  // 若两个子串相同，它们对应的 rk 也需要相同，所以要去重
    }
  }
  for (i = 1; i <= n; ++i) printf("%d ", sa[i]);

  return 0;
}
```
{% endnote %}

## $O(n \log n)$ 做法

我们尝试优化排序的速度。
在刚刚的代码中，排序的时间复杂度是 $O(n \log n)$ 的。如果我们将排序优化到 $O(n)$ 那么就可以减小总时间复杂度到 $O(n \log n)$ 。

我们尝试使用计数排序来优化我们的时间复杂度。
由于后缀数组排序时的关键字的排名，值域为 $O(n)$ ，而且是一个双关键字的排序，可以使用计数排序来优化至 $O(n)$ 。

同时我们考虑一些优化。

### 第二关键字无需计数排序

其实我们一开始已经排过序了。这些第二关键字相当于是一些原串的后缀。他们长度较小，也就优先被排过序。  

OI Wiki上还介绍了一些其他的优化。

{% note primary 其他优化 %}

- 优化计数排序的值域

每次对 $rk$ 进行去重之后，我们都计算了一个 $p$ ，这个 $p$ 即是 $rk$ 的值域，将值域改成它即可。

- 将 `rk[id[i]]` 存下来，减少不连续内存访问

这个优化在数据范围较大时效果非常明显。

- 用函数 cmp 来计算是否重复

同样是减少不连续内存访问，在数据范围较大时效果比较明显。

把 `oldrk[sa[i]] == oldrk[sa[i - 1]] && oldrk[sa[i] + w] == oldrk[sa[i - 1] + w]` 替换成 `cmp(sa[i], sa[i - 1], w)，bool cmp(int x, int y, int w) { return oldrk[x] == oldrk[y] && oldrk[x + w] == oldrk[y + w]; }`。

若排名都不相同可直接生成后缀数组
考虑新的`rk`数组，若其值域为 $[1,n]$ 那么每个排名都不同，此时无需再排序。
{% endnote %}

{% note default 参考代码 from OI Wiki %}
``` cpp
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <iostream>
using namespace std;
const int N = 1000010;
char s[N];
int n, sa[N], rk[N], oldrk[N << 1], id[N], px[N], cnt[N];
// px[i] = rk[id[i]]（用于排序的数组所以叫 px）
bool cmp(int x, int y, int w) {
  return oldrk[x] == oldrk[y] && oldrk[x + w] == oldrk[y + w];
}
int main() {
  int i, m = 300, p, w;
  scanf("%s", s + 1);
  n = strlen(s + 1);
  for (i = 1; i <= n; ++i) ++cnt[rk[i] = s[i]];
  for (i = 1; i <= m; ++i) cnt[i] += cnt[i - 1];
  for (i = n; i >= 1; --i) sa[cnt[rk[i]]--] = i;
  for (w = 1;; w <<= 1, m = p) {  // m=p 就是优化计数排序值域
    for (p = 0, i = n; i > n - w; --i) id[++p] = i;
    for (i = 1; i <= n; ++i)
      if (sa[i] > w) id[++p] = sa[i] - w;
    memset(cnt, 0, sizeof(cnt));
    for (i = 1; i <= n; ++i) ++cnt[px[i] = rk[id[i]]];
    for (i = 1; i <= m; ++i) cnt[i] += cnt[i - 1];
    for (i = n; i >= 1; --i) sa[cnt[px[i]]--] = id[i];
    memcpy(oldrk, rk, sizeof(rk));
    for (p = 0, i = 1; i <= n; ++i)
      rk[sa[i]] = cmp(sa[i], sa[i - 1], w) ? p : ++p;
    if (p == n) {
      for (int i = 1; i <= n; ++i) sa[rk[i]] = i;
      break;
    }
  }
  for (i = 1; i <= n; ++i) printf("%d ", sa[i]);
  return 0;
}
```
{% endnote %}

# `height` 数组

## 定义

height 数组的定义如下：
`height[i]=lcp(sa[i],sa[i-1])`，
即第 $i$ 名的后缀与其前一名的后缀的最长公共前缀的长度。

`height[1]`可以视作 $0$。

## $O(n)$ 求 `height` 数组需要的一个引理

$height[rk[i]]\ge height[rk[i-1]]-1$

证明：

当 $height[rk[i-1]]\le1$ 时，上式显然成立（右边小于等于 $0$）。

当 $height[rk[i-1]]>1$ 时：

设后缀 $i-1$ 为 $aAD$（$A$ 是一个长度为 $height[rk[i-1]]-1$ 的字符串），那么后缀 $i$ 就是 $AD$。设后缀 $sa[rk[i-1]-1]$ 为 $aAB$，那么 $lcp(i-1,sa[rk[i-1]-1])=aA$。由于后缀 $sa[rk[i-1]-1]+1$ 是 $AB$，一定排在后缀 $i$ 的前面，所以后缀 $sa[rk[i]-1]$ 一定含有前缀 $A$，所以 $lcp(i,sa[rk[i]-1])$ 至少是 $height[rk[i-1]]-1$。

简单来说：
$i-1$：$aAD$
$i$：$AD$
$sa[rk[i-1]-1]$：$aAB$
$sa[rk[i-1]-1]+1$：$AB$
$sa[rk[i]-1]$：$A[B/C]$
$lcp(i,sa[rk[i]-1])$：$AX$（$X$ 可能为空）

## $O(n)$ 求 `height` 数组的代码实现

利用上面这个引理暴力求即可：

```cpp
for (i = 1, k = 0; i <= n; ++i) {
  if (rk[i] == 0) continue;
  if (k) --k;
  while (s[i + k] == s[sa[rk[i] - 1] + k]) ++k;
  height[rk[i]] = k;
}
```

$k$ 不会超过 $n$，最多减 $n$ 次，所以最多加 $2n$ 次，总复杂度就是 $O(n)$。

## `height` 数组的应用

摘自 OI Wiki。

### 两子串最长公共前缀

$lcp(sa[i],sa[j])=\min\{height[i+1..j]\}$

感性理解：如果 $height$ 一直大于某个数，前这么多位就一直没变过；反之，由于后缀已经排好序了，不可能变了之后变回来。

严格证明可以参考[\[2004\]后缀数组 by. 徐智磊][1]。

有了这个定理，求两子串最长公共前缀就转化为了 [RMQ 问题](../topic/rmq.md)。

### 比较一个字符串的两个子串的大小关系

假设需要比较的是 $A=S[a..b]$ 和 $B=S[c..d]$ 的大小关系。

若 $lcp(a, c)\ge\min(|A|, |B|)$，$A < B\iff |A|<|B|$。

否则，$A < B\iff rk[a]< rk[c]$。

### 不同子串的数目

子串就是后缀的前缀，所以可以枚举每个后缀，计算前缀总数，再减掉重复。

“前缀总数”其实就是子串个数，为 $n(n+1)/2$。

如果按后缀排序的顺序枚举后缀，每次新增的子串就是除了与上一个后缀的 LCP 剩下的前缀。这些前缀一定是新增的，否则会破坏 $lcp(sa[i],sa[j])=\min \{ height[i+1..j]\}$ 的性质。只有这些前缀是新增的，因为 LCP 部分在枚举上一个前缀时计算过了。

所以答案为：

$\frac{n(n+1)}{2}-\sum\limits_{i=2}^nheight[i]$

### 出现至少 k 次的子串的最大长度

例题：[「USACO06DEC」Milk Patterns](https://www.luogu.com.cn/problem/P2852)。

{% note info 题解 %}
出现至少 $k$ 次意味着后缀排序后有至少连续 $k$ 个后缀的 LCP 是这个子串。
    
所以，求出每相邻 $k-1$ 个 $height$ 的最小值，再求这些最小值的最大值就是答案。
    
可以使用单调队列 $O(n)$ 解决，但使用其它方式也足以 AC。
{% endnote %}

{% note info 参考代码 %}
```cpp
#include <cstdio>
#include <cstring>
#include <iostream>
#include <set>
using namespace std;
const int N = 40010;
int n, k, a[N], sa[N], rk[N], oldrk[N], id[N], px[N], cnt[1000010], ht[N], ans;
multiset<int> t;  // multiset 是最好写的实现方式
bool cmp(int x, int y, int w) {
  return oldrk[x] == oldrk[y] && oldrk[x + w] == oldrk[y + w];
}
int main() {
  int i, j, w, p, m = 1000000;
  scanf("%d%d", &n, &k);
  --k;
  for (i = 1; i <= n; ++i) scanf("%d", a + i);  //求后缀数组
  for (i = 1; i <= n; ++i) ++cnt[rk[i] = a[i]];
  for (i = 1; i <= m; ++i) cnt[i] += cnt[i - 1];
  for (i = n; i >= 1; --i) sa[cnt[rk[i]]--] = i;
  for (w = 1; w < n; w <<= 1, m = p) {
    for (p = 0, i = n; i > n - w; --i) id[++p] = i;
    for (i = 1; i <= n; ++i)
      if (sa[i] > w) id[++p] = sa[i] - w;
    memset(cnt, 0, sizeof(cnt));
    for (i = 1; i <= n; ++i) ++cnt[px[i] = rk[id[i]]];
    for (i = 1; i <= m; ++i) cnt[i] += cnt[i - 1];
    for (i = n; i >= 1; --i) sa[cnt[px[i]]--] = id[i];
    memcpy(oldrk, rk, sizeof(rk));
    for (p = 0, i = 1; i <= n; ++i)
      rk[sa[i]] = cmp(sa[i], sa[i - 1], w) ? p : ++p;
  }
  for (i = 1, j = 0; i <= n; ++i) {  //求 height
    if (j) --j;
    while (a[i + j] == a[sa[rk[i] - 1] + j]) ++j;
    ht[rk[i]] = j;
  }
  for (i = 1; i <= n; ++i) {  //求所有最小值的最大值
    t.insert(ht[i]);
    if (i > k) t.erase(t.find(ht[i - k]));
    ans = max(ans, *t.begin());
  }
  cout << ans;
  return 0;
}
```
{% endnote %}

### 是否有某字符串在文本串中至少不重叠地出现了两次

可以二分目标串的长度 $|s|$，将 $h$ 数组划分成若干个连续 LCP 大于等于 $|s|$ 的段，利用 RMQ 对每个段求其中出现的数中最大和最小的下标，若这两个下标的距离满足条件，则一定有长度为 $|s|$ 的字符串不重叠地出现了两次。

### 连续的若干个相同子串

我们可以枚举连续串的长度 $|s|$，按照 $|s|$ 对整个串进行分块，对相邻两块的块首进行 LCP 与 LCS 查询，具体可见[\[2009\]后缀数组——处理字符串的有力工具][2]。

例题：[「NOI2016」优秀的拆分](https://loj.ac/p/2083)。

### 结合并查集

某些题目求解时要求你将后缀数组划分成若干个连续 LCP 长度大于等于某一值的段，亦即将 $h$ 数组划分成若干个连续最小值大于等于某一值的段并统计每一段的答案。如果有多次询问，我们可以将询问离线。观察到当给定值单调递减的时候，满足条件的区间个数总是越来越少，而新区间都是两个或多个原区间相连所得，且新区间中不包含在原区间内的部分的 $h$ 值都为减少到的这个值。我们只需要维护一个并查集，每次合并相邻的两个区间，并维护统计信息即可。

经典题目：[「NOI2015」品酒大会](https://uoj.ac/problem/131)

### 结合线段树

某些题目让你求满足条件的前若干个数，而这些数又在后缀排序中的一个区间内。这时我们可以用归并排序的性质来合并两个结点的信息，利用线段树维护和查询区间答案。

### 结合单调栈

例题：[「AHOI2013」差异](https://loj.ac/problem/2377)

{% note info 题解 %}
被加数的前两项很好处理，为 $n(n-1)(n+1)/2$（每个后缀都出现了 $n-1$ 次，后缀总长是 $n(n+1)/2$），关键是最后一项，即后缀的两两 LCP。
    
我们知道 $lcp(i,j)=k$ 等价于 $\min\{height[i+1..j]\}=k$。所以，可以把 $lcp(i,j)$ 记作 $\min\{x|i+1\le x\le j, height[x]=lcp(i,j)\}$ 对答案的贡献。
    
考虑每个位置对答案的贡献是哪些后缀的 LCP，其实就是从它开始向左若干个连续的 $height$ 大于它的后缀中选一个，再从向右若干个连续的 $height$ 不小于它的后缀中选一个。这个东西可以用单调栈计算。
    
单调栈部分类似于 [Luogu P2659 美丽的序列](https://www.luogu.com.cn/problem/P2659) 以及悬线法。
{% endnote %}

{% note default 参考代码 %}
``` cpp
#include <cstdio>
#include <cstring>
#include <iostream>
using namespace std;
const int N = 500010;
char s[N];
int n, sa[N], rk[N << 1], oldrk[N << 1], id[N], px[N], cnt[N], ht[N], sta[N],
    top, l[N];
long long ans;
bool cmp(int x, int y, int w) {
  return oldrk[x] == oldrk[y] && oldrk[x + w] == oldrk[y + w];
}
int main() {
  int i, k, w, p, m = 300;
  scanf("%s", s + 1);
  n = strlen(s + 1);
  ans = 1ll * n * (n - 1) * (n + 1) / 2;
  //求后缀数组
  for (i = 1; i <= n; ++i) ++cnt[rk[i] = s[i]];
  for (i = 1; i <= m; ++i) cnt[i] += cnt[i - 1];
  for (i = n; i >= 1; --i) sa[cnt[rk[i]]--] = i;
  for (w = 1; w < n; w <<= 1, m = p) {
    for (p = 0, i = n; i > n - w; --i) id[++p] = i;
    for (i = 1; i <= n; ++i)
      if (sa[i] > w) id[++p] = sa[i] - w;
    memset(cnt, 0, sizeof(cnt));
    for (i = 1; i <= n; ++i) ++cnt[px[i] = rk[id[i]]];
    for (i = 1; i <= m; ++i) cnt[i] += cnt[i - 1];
    for (i = n; i >= 1; --i) sa[cnt[px[i]]--] = id[i];
    memcpy(oldrk, rk, sizeof(rk));
    for (p = 0, i = 1; i <= n; ++i)
      rk[sa[i]] = cmp(sa[i], sa[i - 1], w) ? p : ++p;
  }
  //求 height
  for (i = 1, k = 0; i <= n; ++i) {
    if (k) --k;
    while (s[i + k] == s[sa[rk[i] - 1] + k]) ++k;
    ht[rk[i]] = k;
  }
  //维护单调栈
  for (i = 1; i <= n; ++i) {
    while (ht[sta[top]] > ht[i]) --top;  // top类似于一个指针
    l[i] = i - sta[top];
    sta[++top] = i;
  }
  //最后利用单调栈算 ans
  sta[++top] = n + 1;
  ht[n + 1] = -1;
  for (i = n; i >= 1; --i) {
    while (ht[sta[top]] >= ht[i]) --top;
    ans -= 2ll * ht[i] * l[i] * (sta[top] - i);
    sta[++top] = i;
  }
  cout << ans;
  return 0;
}
```
{% endnote %}
类似的题目：[「HAOI2016」找相同字符](https://loj.ac/problem/2064)。
