---
title: HackerRank - Moody's Analytics Fall University CodeSprint - Let's Play a Game
date: 2017-10-15 19:14:17
tags:
  - 贪心
  - 分类讨论
  - 二分图
  - 构造
  - HackerRank
  - 定理题
categories:
  - 解题报告
mathjax: true
---

## 题目大意

给定一个 $1\times n$ 的格子图。每一个格子都有着一种颜色（红蓝绿白之一）。每个格子上都有一定数量的硬币，不同的格子的硬币数一定是不同的。

你需要从格子图任选一个格子作为开始位置，每次你可以从一个格子跳到图中另外一个没有访问过的格子上。

跳格子有一个规则：从红色或者蓝色格子只能跳到绿色或白色格子上，从绿色或白色格子只能跳到红色或蓝色格子上。

你要一直跳到没有能够继续跳的格子为止。将你经过的所有格子的硬币数记录成一个序列，我们希望最大化这个序列的最长上升子序列长度。

$1\leq n\leq4\times10^5$
<!--more-->
---

## 题目分析

HackerRank 居然出论文题。

首先题目四种颜色本质就是两种颜色，问题变成在一个带点权完全二分图上找到一条路径使其最长上升子序列最长。

然后这个玩意儿居然有人研究……结论是：假设 $|X|\geq|Y|$ ，一定存在一个 $\mathrm{LIS}$ 包含了 $|Y|$ 部的所有点。

然后就很方便了，我们直接去构造这个 $\mathrm{LIS}$：把两个部的点都按权值排好序，然后把 $|X|$ 部的点尽可能多地插入到 $|Y|$ 部的相邻两个元素中就好了。

时间复杂度 $O(n\log n)$。

证明？咳咳比较复杂，限(wo)于(bi)篇(jiao)幅(lan)就不在这里展开了。可以看一下参考文献。

思路就是证明只要 $\mathrm{LIS}$ 没有完全包含 $|Y|$ 部的点，一定存在一种原本基础上的构造能够使新的 $\mathrm{LIS}$ 在 $Y$ 部的点数恰好多 $1$。

中间都是一些分类讨论和反证。

---

## 代码实现

```cpp
#include <algorithm>
#include <iostream>
#include <cstdio>
#include <cctype>

using namespace std;

inline int read()
{
	int x=0,f=1;
	char ch=getchar();
	while (!isdigit(ch)) f=ch=='-'?-1:f,ch=getchar();
	while (isdigit(ch)) x=x*10+ch-'0',ch=getchar();
	return x*f;
}

const int N=400005;

int nodes[2][N];
char str[N];
int a[N];
int n,ans,cnt;

void greedy()
{
	if (!nodes[0][0])
	{
		ans=nodes[1][0]>0;
		return;
	}
	ans=nodes[0][0];
	int head=1,tail=nodes[1][0];
	if (nodes[1][0]>nodes[0][0])
	{
		if (nodes[1][head]<nodes[0][1]) ++ans,++head;
		if (head<=tail&&nodes[1][tail]>nodes[0][nodes[0][0]]) ++ans,--tail;
	}
	else if (nodes[1][head]<nodes[0][1]) ++ans,++head;
	else if (nodes[1][tail]>nodes[0][nodes[0][0]]) ++ans,--tail;
	for (int i=1;i<nodes[0][0];++i)
	{
		for (;head<=tail&&nodes[1][head]<nodes[0][i];++head);
		ans+=head<=tail&&nodes[1][head]<nodes[0][i+1];
	}
}

int main()
{
	freopen("game.in","r",stdin),freopen("game.out","w",stdout);
	n=read(),scanf("%s",str),cnt=0;
	for (int i=0;i<n;++i) cnt+=str[i]=='B'||str[i]=='R';
	for (int i=1,x;i<=n;++i) x=(str[i-1]=='B'||str[i-1]=='R')^(cnt*2<=n),nodes[x][++nodes[x][0]]=read();
	for (int i=0;i<2;++i) sort(nodes[i]+1,nodes[i]+1+nodes[i][0]);
	greedy(),printf("%d\n",ans);
	fclose(stdin),fclose(stdout);
	return 0;
}
```

---

## 参考文献

```bibtex
@inproceedings{Lin2012FindingAL,
  title={Finding a Longest Increasing Subsequence from the Paths in a Complete Bipartite Graph},
  author={Guan-Yu Lin and Jia-Jie Liu and Yue-Li Wang},
  year={2012},
  url={https://api.semanticscholar.org/CorpusID:18850482}
}
```
