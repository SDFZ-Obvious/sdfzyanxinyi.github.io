---
layout: post
title: "树状数组"
subtitle: "简单的数据结构初阶"
tags: "数据结构"
---
## 树状数组
（$\texttt{Binary Indexed Trees}$，简称 $\texttt{BIT}$，二进制索引树）是一种特殊的数据结构。主要用于解决**单点修改区间求和**的问题。

其插入和查询的操作时间复杂度都是 $\Theta(log\;n)$ 的。

## 对于这个问题：

给定一个 $n$ 个元素的的数组 $a1,a2,..,an$ ，对其进行 $m$ 次操作:
- 将数组中的第 $x$ 项加上 $d$
- 给定 $l$ 和 $r$ 求出 $\sum\limits_{i=l}^ra_i$ 

### 一、暴力做法
- 修改：直接修改元素，时间 $\Theta(1)$
- 求和：遍历累加，时间 $\Theta(n)$

最大时间复杂度 $\Theta(mn)$，太大了！
### 二、前缀和
- 修改：后面全部修改，时间 $\Theta(n)$
- 求和：做差，时间 $\Theta(1)$

最大时间复杂度 $\Theta(mn)$，还是太大了！

### 三、树状数组
通过观察前缀和的处理过程，我们发现为了表示一个区间，前缀和只用两个完整区间相减，这样修改一个点，就要将包含这个点的所有区间和修改一下，而往往区间右端点大于该点下标的都要修改。为了改进它，我们不妨让两个完整的区间变成几个小区间拼合起来的，这样几个小区间的和成为大区间的和，两和相减即可。这样在修改其中一个点时，就不必将它上方的区间全部修改，只需修改包含它的一部分就行了。

为了尽量方便的能用几个小区间拼出任意长度的大区间，我们想到二进制。

如图：
[![j0lYv9.png](https://s1.ax1x.com/2022/07/07/j0lYv9.png)](https://imgtu.com/i/j0lYv9)
我们将前缀和转化为树状数组：
[![j0lT2Q.png](https://s1.ax1x.com/2022/07/07/j0lT2Q.png)](https://imgtu.com/i/j0lT2Q)
显然，大大减少了修改的复杂度。

## 原理：

### 对于树状数组结构

下表为 $k$ 的点表示的就是它前面 $q$ 位数组元素的和。令 $q=k$ 二进制下末尾 $0$ 的个数

### 对于查询

我们要找 $l$ 和 $r$ 之间的区间和，首先需要求出 $\sum\limits_{i=1}^{l-1}a_i$ 和 $\sum\limits_{i=1}^ra_i$，然后做差，即:$\sum\limits_{i=l}^ra_i= \sum\limits_{i=1}^ra_i-\sum\limits_{i=1}^{l-1}a_i$ 。

要想求出从 $1$ 到一个数的和，我们需要找出所有组成它的小区间，然后全相加。

比如我要找到组成从 $1$ 到 $12$ 的所有小区间，可以发现，$12 = 8+4$，所以就先去找右端点为 $12$ 的小区间，$12-4=8$，我们再找右端点下标 $8$ 的， $8=8$，所以再找下标为 $0$ 的，而没有 $0$，所以查找结束。

这时候我们发现，每次找的右端点，都是上次找的下标的二进数的最低位的 $1$ 变成了 $0$。也就是减去了它的低位。

对于求这个低位，我们使用补码的性质，得到 `lowbit(x)=x & -x` 就可轻松求出。

对于找出的小区间，直接求和即可。

因为转化二进制，所以时间复杂度 $\Theta(log\;n)$。

### 对于修改

修改一个点，就必须把包含这个点的所有区间全修改一遍。

所以我们的任务是找到包含它的区间。

以上图为例，比如修改下标为 $3$ 的点，$3 = 2+1$，所以需要修改 下标为 $3,4$ 的，$4=4$, 所以需要修改下标为 $8$ 的。$8=8$，所以需要修改下标为 $16$ 的。$16$ 已经到达了上限，需要修改的就修改完了。

我们发现，需要修改的区间的右端点下标，就是不断加上原下标的 `lowbit`。

## 代码实现：

### lowbit：
```cpp
inline int lowbit(int x){return x&-x;}
```
### 对于查询

```cpp
inline int ask(int x){
	int res=0;
	while(x){
		res += c[x];
		x -= lowbit(x);
	}
	return res;
}
```


### 对于修改

```cpp
inline void add(int x,int val){
	while(x <= n){
		c[x] += val;
		x += lowbit(x);
	}
}
```

## 例题：

### [模板](https://www.luogu.com.cn/problem/P3374)
没什么好说的，套上就行
### [星星](https://www.luogu.com.cn/problem/T220389)
纵坐标排好序，以横坐标（值域）建桶，存放该横坐标上的星星个数，做树状数组，该星星的横坐标就是右端点，记录 $x$ 从 $1$ 到 $x$ 之间星星个数之和。
```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 5e5+5;
int n,m; 
int x,y,c[N],level[N];
inline int lowbit(int x){return x&-x;}
inline void add(int x,int val){
	while(x <=320001 ){
		c[x] += val;
		x += lowbit(x);
	}
}
inline int ask(int x){
	int res=0;
	while(x){
		res += c[x];
		x -= lowbit(x);
	}
	return res;
}
int main(){
	cin >> n;
	for(int i = 1;i <= n;i ++){
		cin >> x >> y;
		x ++;
		y ++;
		level[ask(x)]++;
		add(x,1);	
	}
	for(int i = 0;i <= n-1;i ++){
		cout<<level[i]<<endl;
	}
	return 0;
}
```