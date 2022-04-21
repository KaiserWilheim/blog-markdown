---
title: P1061 Jam的计数法 题解
zh-CN: true
date: 2021-09-23
tags:
	- 题解
	- 模拟
categories:
 题解
mathjax: true
---

Luogu P1061

<!--more-->
----

这道题可以使用简单地使用模拟的方法来解决。

我们先把所有的字符转化成 $[1,26]$ 范围内的整数，便于运算；
接下来我们建立另外一个数组，存储 $[t-w+1,t]$ 范围内的整数，这代表某一位数字可以达到的最大值。
因为题目要求后面的数字严格大于前面的数字，所以我们降序存储的序列代表着最大的那个数。

接下来，进行判断。
如果最后一个数达到最大值，那就往前搜索，直到某一个数没有达到最大值为止。
之后将没有达到最大值的最低位+1，剩下的位顺序排列。

上代码：

``` cpp
#include <bits/stdc++.h>
using namespace str;
int a[30],b[30];
int main(){
	int s,t,w,st=1;
	cin>>s>>t>>w;
	for(int i=1;i<=w;i++){
		b[i]=t-(w-i);
	}
	for(int i=1;i<=w;i++){
		char c;
		cin>>c;
		a[i]=c-'a'+1;
		a[0]+=a[i];
	}
	for(int i=1;i<=5;i++){
		st=1;
		for(int j=w;j>=1;j--){
			if(a[j]>=b[j]){
				a[j]=-1;
			}else{
				break;
			}
		}
		for(int j=w;j>=1;j--){
			if((a[j]==-1)&&(a[j-1]!=-1)){
				st=0;
				a[j-1]++; 
				for(int k=j;k<=w;k++){
					a[k]=a[k-1]+1;
				}				
			}
		}
		if(st==1)a[w]++;
		for(int j=1;j<=w;j++){
			char c=a[j]+'a'-1;
			cout<<c;
		}
		cout<<endl;
		if(a[0]==(s+s-w+1)*w/2)return 0;
	}
	return 0;
}
```
$\color[rgb]{1,1,0.0625}{φ}$

感谢阅读。
