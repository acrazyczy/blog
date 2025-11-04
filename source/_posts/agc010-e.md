---
title: AGC010-E Rearranging
date: 2017-10-17 21:46:29
tags:
	- AtCoder
	- 贪心
	- 拓扑图
  	- DFS树
categories:
	- 解题报告
mathjax: true
---

## 题目大意

给定 $n$ 个整数 $a_i$，两个人要把这些数排成一个数列，他们进行如下操作：

$\bullet$ 第一个人先将这些数按照他想要的方式排成一个数列

$\bullet$ 第一个人排完之后，第二个人可以选择两个位置相邻的互质数交换，他可以执行该操作任意次。
第一个人想要使最后的数列字典序尽量小，第二个人想使最后的数列字典序尽量大。假设两个人都绝顶聪明。
求最后的数列。

$1\leq n\leq 2000,1\leq a_i\leq 10^8$
<!--more-->
---

## 题目分析

首先我们先找到第一个人排完之后第二个人的最优策略是什么：考虑两个不互质的数 $x$ 和 $y$，显然不管我怎么交换，$x$ 和 $y$ 的相对位置都不会发生改变。那我考虑枚举一个在前面的数 $a_i$，在后面的数 $a_j$，如果 $\gcd(a_i,a_j)=1$ 那么就从 $i$ 向 $j$ 连一条边，表示 $i$ 在最终序列必须在 $j$ 前面。

那么我们就可以贪心地构造出最终数列了。考虑对连成的图做拓扑排序，每次优先选择$a_x$大的出队加在已经生成的数列后面就好了。

回到问题本身，我们在上面的思路上继续扩展。考虑在所有不互质的 $i$ 和 $j$ 之间连无向边。那么问题就是给这个无向图定向成一个有向无环图，使得这个有向无环图进行上面贪心得到的序列字典序尽量小。

这个图可能会有多个连通块，显然，可以将它们分开来考虑。合并两个连通块的时候就是贪心地将两个已有的序列合并成一个字典序尽量大的序列，满足同一个序列的元素相对顺序不变。

那么现在我们只考虑连通图，第一个人肯定希望最小的元素严格作为第一个出队的，而这个一定存在至少一种方案：考虑以该点作为开始节点对图做一遍 $\mathrm{DFS}$，得到的 $\mathrm{DFS}$ 树就是我们定向的方案。

然后我们删掉这个元素，这幅图就又分裂成若干个连通块，剩下的过程肯定是和上一步类似的。但是这样我们会发现，如果我们选择了一个和删掉的点之间没有连边的，那么第二个人肯定可以先让这个出队，于是最小元素就不是第一个出队得了。因此我们只能选择和删除元素有连边的最小的（这样它肯定不会先于任何一个之前所选的出队）。

用一个比较简单的过程来描述，就是对于一个连通块我们从最小的点开始 $\mathrm{DFS}$。对于一个节点，我们按照从小到大顺序遍历儿子节点，然后将所有儿子节点的最终序列按照第二个人的贪心策略合并起来，最后在最前面加上当前节点。

总的时间复杂度是 $O(n^2\log {\max\{a_i\}})$的。

这题的贪心挺妙的啊，我拿掉最小元素之后就不会了啊。。。

---

## 代码实现

```cpp
#include <algorithm>
#include <iostream>
#include <cstdio>
#include <vector>

using namespace std;

typedef vector<int> V;

const int N=2005;
const int E=N*N;

V seq[N];

int tov[E],nxt[E];
int num[N],id[N];
bool vis[N];
int last[N];
int n,tot,rt;

inline bool cmp(int x,int y){return num[x]<num[y];}

inline void insert(int x,int y){tov[++tot]=y,nxt[tot]=last[x],last[x]=tot;}

int gcd(int x,int y){return y?gcd(y,x%y):x;}

inline void merge(V a,V b,V &ret)
{
	ret.resize(0),ret.reserve(a.size()+b.size());
	vector<int>::iterator ita=a.begin(),itb=b.begin();
	for (;ita!=a.end()||itb!=b.end();)
		if (ita==a.end()||itb!=b.end()&&num[*itb]>num[*ita]) ret.push_back(*itb),++itb;
		else ret.push_back(*ita),++ita;
}

void dfs(int x)
{
	V son;vis[x]=1;
	for (int i=last[x];i;i=nxt[i]) son.push_back(tov[i]);
	sort(son.begin(),son.end(),cmp);
	for (int i=0,y;i<(int)son.size();++i)
		if (!vis[y=son[i]]) dfs(y),merge(seq[y],seq[x],seq[x]);
	seq[x].insert(seq[x].begin(),x);
}

int main()
{
	freopen("rearranging.in","r",stdin),freopen("rearranging.out","w",stdout);
	scanf("%d",&n);
	for (int i=1;i<=n;++i) scanf("%d",&num[i]),id[i]=i;
	sort(id+1,id+1+n,cmp);
	for (int i=1;i<n;++i)
		for (int j=i+1;j<=n;++j)
			if (gcd(num[i],num[j])!=1) insert(i,j),insert(j,i);
	rt=0;
	for (int i=1,x;i<=n;++i)
		if (!vis[x=id[i]]) dfs(x),merge(seq[rt],seq[x],seq[x]),rt=x;
	for (int i=0;i<(int)seq[rt].size();++i) printf("%d%c",num[seq[rt][i]],i+1<(int)seq[rt].size()?' ':'\n');
	fclose(stdin),fclose(stdout);
	return 0;
}

```
