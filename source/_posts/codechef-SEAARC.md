---
title: CodeChef SEAARC
date: 2017-10-06 19:41:12
tags:
	- CodeChef
	- 树状数组
	- 阈值均衡
categories:
	- 解题报告
mathjax: true
---

## 题目大意

给定一个长度为 $n$ 的序列${a_n}$，对于任意 $x\neq y$，如果 $a_x=a_y$，则在 $x$ 和 $y$ 之间画一条弧。

称两条弧 $(x,y)$ 与 $(l,r)$ 相交当且仅当 $x<l<y<r$ 或者 $l<x<r<y$。求有多少条异色弧相交。

$1\leq n\leq 10^5,1\leq a_i\leq 10^5$
<!--more-->
---

## 题目分析

正难则反：考虑计算出所有的异色弧对数（枚举颜色用前缀和优化计算，线性），减去形如 AABB 的以及 ABBA 的就是答案了。

AABB 挺好算的，枚举位置，也是记录前缀和之类的东西就可以在线性时间内算完了。

关键就在于 ABBA 怎么算。考虑阈值均衡，设 $T$ 为阈值。出现次数小于等于 $T$ 的颜色称为小块($S$)，出现次数大于 $T$ 的颜色称为大块($L$)。


分类讨论：

对于形如 LSSL 的，我们枚举 $A$ 的颜色种类，以及第二个 $B$ 的位置。

用数组记录对于特定一种（小块）颜色，其所有已经枚举过的位置前面的 $A$ 这种颜色种类出现的次数之和，这个可以在枚举过程中更新。

将上面的信息乘上当前枚举的位置后面$A$这种颜色出现次数即可更新答案。

时间复杂度 $O\left(\frac{n^2}T\right)$。

对于形如 SLLS 以及 LLLL的（也就是 B 的颜色是大块的），我们枚举 B 的颜色种类以及第二个 A 的位置。

设 $\mathrm{pre}_i$ 表示前 $i$ 个位置B的出现次数，$A$ 存的是所有和 A 同色的出现的位置，我们分析 A 的某一种颜色带来的贡献
$$ \sum_{i=1}^{|A|}\sum_{j=1}^{i-1}{\mathrm{pre}_i-\mathrm{pre}_j\choose 2} $$

拆开来就是
$$
\frac12\sum_{i=1}^{|A|}\left(\mathrm{pre}^2_i(i-1)-2\mathrm{pre}_i\sum_{j=1}^{i-1}\mathrm{pre}_j+\sum_{j=1}^{i-1}\mathrm{pre}^2_j+\sum_{j=1}^{i-1}\mathrm{pre}_j-\mathrm{pre}_i(i-1)\right)
$$

维护一下 $\mathrm{pre}_j$ 以及 $\mathrm{pre}^2_j$ 的前缀和就好了。

时间复杂度也是 $O\left(\frac{n^2}T\right)$。

剩下的就是形如 SSSS 的。注意到枚举所有同色小块的一对位置的时间复杂度是 $O(nT)$ 的。

我们枚举第一个 B 的位置，对于已经枚举过的 A 的左端点，我们打一个 $+1$ 标记，对于这个左端点右边所有可能的右端点，我们都打一个 $-1$ 标记，统计答案的话我们再枚举第二个B的位置计算一下标记数组前缀和就能更新了。这个使用树状数组就可以了。

时间复杂度是 $O(nT\log n)$ 的。

总的时间复杂度，理论上 $T=\sqrt{\frac n{\log n}}$ 能达到 $O(n\sqrt{n\log n})$。不过由于树状数组常数贼小，其余部分的计算取模运算又比较多，因此适当调大一下 $T$ 的大小可以更快。

---

## 代码实现

```cpp
#include <algorithm>
#include <iostream>
#include <cstdio>
#include <cctype>
#include <vector>
#include <cmath>
 
using namespace std;
 
const int P=1000000007;
const int itwo=P+1>>1;
const int ifact=41666667;
const int N=100000;
 
inline int read()
{
	int x=0,f=1;
	char ch=getchar();
	while (!isdigit(ch)) f=ch=='-'?-1:f,ch=getchar();
	while (isdigit(ch)) x=x*10+ch-'0',ch=getchar();
	return x*f;
}
 
int a[N+5],cnt[N+5],f[N+5],g[N+5],pre[N+5],pre_[N+5];
vector<int> list[N+5];
int n,AABB,ABBA,all,ans,T,mx;
 
inline int C2(int n){return 1ll*n*(n-1)%P*itwo%P;}
inline int C4(int n){return 1ll*n*(n-1)%P*(n-2)%P*(n-3)%P*ifact%P;}
inline int sqr(int x){return 1ll*x*x%P;}
 
inline int lowbit(int x){return x&-x;}
 
struct Fenwick_tree
{
	int num[N+5];
 
	void modify(int x,int y){for (;x<=n;x+=lowbit(x)) (num[x]+=y)%=P;}
 
	int query(int x)
	{
		int ret=0;
		for (;x;x-=lowbit(x)) (ret+=num[x])%=P;
		return ret;
	}
}t;
 
void calc_AABB()
{
	int sum=0;
	for (int i=1;i<=mx;++i) pre[i]=0;
	for (int i=1;i<=n;++i)
	{
		(sum+=P-C2(pre[a[i]]++))%=P;
		(AABB+=1ll*sum*(cnt[a[i]]-pre[a[i]])%P)%=P;
		(sum+=C2(pre[a[i]]))%=P;
	}
}
 
void calc_ABBA()
{
	T=trunc(sqrt(n));
	//case 1: LSSL
	//iterator A's color and the position of B
	for (int i=1,tmp;i<=mx;++i)
		if (cnt[i]>T)
		{
			for (int j=1;j<=mx;++j) pre[j]=0;
			tmp=0;
			for (int j=1;j<=n;++j)
				if (a[j]==i) ++tmp;
				else if (cnt[a[j]]<=T) (ABBA+=1ll*pre[a[j]]*(cnt[i]-tmp)%P)%=P,(pre[a[j]]+=tmp)%=P;
		}
	//case 2: SLLS && LLLL
	//iterator B's color and the postion of A
	for (int i=1;i<=mx;++i)
		if (cnt[i]>T)
		{
			pre[0]=0;
			for (int j=1;j<=n;++j) pre[j]=pre[j-1]+(a[j]==i);
			for (int j=1;j<=mx;++j) f[j]=g[j]=pre_[j]=0;
			for (int j=1,tmp;j<=n;++j)
			{
				if (a[j]==i) continue;
				tmp=((((1ll*sqr(pre[j])*pre_[a[j]]%P-2ll*pre[j]*f[a[j]]%P+P)%P+f[a[j]])%P+g[a[j]])%P-1ll*pre[j]*pre_[a[j]]%P+P)%P;
				(ABBA+=1ll*tmp*itwo%P)%=P,(f[a[j]]+=pre[j])%=P,(g[a[j]]+=sqr(pre[j]))%=P;
				++pre_[a[j]];
			}
		}
	//case 3: SSSS
	for (int i=1;i<=n;++i)
	{
		if (cnt[a[i]]>T) continue;
		int ptr=cnt[a[i]]-1,rg=0;
		for (;list[a[i]][ptr]>i;--ptr) (ABBA+=t.query(list[a[i]][ptr]))%=P,t.modify(list[a[i]][ptr],-1),++rg;
		t.modify(i,rg);
	}
	for (int i=1;i<=mx;++i) if (cnt[i]<=T) (ABBA+=P-C4(cnt[i]))%=P;
}
 
int main()
{
	freopen("seaarc.in","r",stdin),freopen("seaarc.out","w",stdout);
	n=read();
	for (int i=1;i<=n;++i) ++cnt[a[i]=read()],list[a[i]].push_back(i),mx=max(mx,a[i]);
	for (int i=1,sum=0,tmp;i<=mx;++i) (all+=1ll*(tmp=C2(cnt[i]))*sum%P)%=P,(sum+=tmp)%=P;
	calc_AABB(),calc_ABBA(),ans=((all-AABB+P)%P-ABBA+P)%P;
	printf("%d\n",ans);
	fclose(stdin),fclose(stdout);
	return 0;
}
```