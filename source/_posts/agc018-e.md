---
title: AGC018-E Sightseeing Plan
date: 2017-10-03 19:32:55
tags:
	- AtCoder
	- 计数类问题
	- 组合数
categories:
	- 解题报告
mathjax: true
---

## 题目大意

给定三个在平面直角坐标系的第一象限上矩形 $[x_1,y_1][x_2,y_2],[x_3,y_3][x_4,y_4],[x_5,y_5][x_6,y_6]$。

你要从第一个矩形内选出一个整点作为起始点，第二个中选出一个中转点，第三个中选出一个结束点，然后从起始点开始不断向上或右走，经过中转点最后到结束点。求不同的路径的总数量。

两条路径不同当且仅当它们的起始/中转/结束点存在不同或者路线不一样。

$1\leq x_1\lt x_2\lt x_3\lt x_4\lt x_5\lt x_6\leq10^6,1\leq y_1\lt y_2\lt y_3\lt y_4\lt y_5\lt y_6\leq10^6$
<!--more-->
---

## 题目分析

题目的条件太多了，考虑简化题目。

令 $C(x,y)$ 表示从 $(0,0)$ 走到 $(x,y)$ 的方案，我们考虑一条简单的式子：
$$
\sum_{y=0}^Y{C(X,y)}=C(X+1,Y)
$$

这个相信大家都会。再考虑使用两次将其扩展到二维的情况。
$$
\sum_{x=0}^X\sum_{y=0}^Y{C(x,y)}=C(X+1,Y+1)-1
$$

使用二维差分将其推广到对于一个一般的矩形的情况。
$$
\sum_{x=X_1}^{X_2}\sum_{y=Y_1}^{Y_2}{C(x,y)}=C(X_2+1,Y_2+1)-C(X_2+1,Y_1)-C(X_1,Y_2+1)+C(X_1,Y_1)
$$

由这条式子，我们可以看出，统计从一个点一直到一个矩形内所有点的方案数的问题，其实可以变成统计一个点到矩形四角上四个点的方案数问题。

考虑枚举第一个矩形的四个关键点之一，第三个矩形的关键点之一，然后对第二个矩形进行计算，现在问题是给定左下角和右上角的起点和终点，求经过矩形中每一个点的路径方案数的和。

可以发现，一条路径如果和矩形有 $\mathrm{len}$ 长度的相交，那么它对答案的贡献就要乘上 $\mathrm{len}$ 的系数。而这个 $\mathrm{len}$ 的数值可以由路径进入矩形的位置和离开矩形的位置来决定，具体而言，是两个 $x$ 坐标的差加上两个 $y$ 坐标的差。

考虑将路径贡献拆开来，枚举进入/离开矩形的位置，然后用路径总数乘上相应的系数（坐标之和，符号由进入/退出决定）加到答案里面。

最后的时间复杂度是 $O(\max X+\max Y)$。

感觉很妙啊。

---

## 代码实现
```cpp
#include <iostream>
#include <cstdio>
 
using namespace std;
 
const int P=1000000007;
const int N=2000000;
 
int fact[N+5],invf[N+5];
int f[4][3],g[4][3];
int X1,X2,X3,X4,X5,X6,Y1,Y2,Y3,Y4,Y5,Y6,ans;
 
inline int C(int n,int m){return 1ll*fact[n+m]*invf[n]%P*invf[m]%P;}
 
inline void add(int &x,int y){(x+=y)%=P;}
 
int quick_power(int x,int y)
{
	int ret=1;
	for (;y;y>>=1,x=1ll*x*x%P) if (y&1) ret=1ll*ret*x%P;
	return ret;
}
 
void pre()
{
	fact[0]=1;
	for (int i=1;i<=N;++i) fact[i]=1ll*fact[i-1]*i%P;
	invf[N]=quick_power(fact[N],P-2);
	for (int i=N;i>=1;--i) invf[i-1]=1ll*invf[i]*i%P;
}
 
int calc(int x1,int y1,int sig1,int x2,int y2,int sig2)
{
	int ret=0;
	for (int x=X3;x<=X4;++x) add(ret,P-1ll*C(x-x1,Y3-1-y1)*(x+Y3)%P*C(x2-x,y2-Y3)%P),add(ret,1ll*C(x2-x,y2-Y4-1)*(x+Y4+1)%P*C(x-x1,Y4-y1)%P);
	for (int y=Y3;y<=Y4;++y) add(ret,P-1ll*C(y-y1,X3-1-x1)*(y+X3)%P*C(y2-y,x2-X3)%P),add(ret,1ll*C(y2-y,x2-X4-1)*(y+X4+1)%P*C(y-y1,X4-x1)%P);
	return ((ret*=sig1*sig2)+=P)%=P;
}
 
int main()
{
	pre();
	freopen("sightseeing.in","r",stdin),freopen("sightseeing.out","w",stdout);
	scanf("%d%d%d%d%d%d",&X1,&X2,&X3,&X4,&X5,&X6),scanf("%d%d%d%d%d%d",&Y1,&Y2,&Y3,&Y4,&Y5,&Y6),ans=0;
	f[0][0]=1,f[0][1]=X1-1,f[0][2]=Y1-1;
	f[1][0]=-1,f[1][1]=X1-1,f[1][2]=Y2;
	f[2][0]=-1,f[2][1]=X2,f[2][2]=Y1-1;
	f[3][0]=1,f[3][1]=X2,f[3][2]=Y2;
	g[0][0]=1,g[0][1]=X6+1,g[0][2]=Y6+1;
	g[1][0]=-1,g[1][1]=X6+1,g[1][2]=Y5;
	g[2][0]=-1,g[2][1]=X5,g[2][2]=Y6+1;
	g[3][0]=1,g[3][1]=X5,g[3][2]=Y5;
	for (int i=0;i<4;++i)
		for (int j=0;j<4;++j)
			add(ans,calc(f[i][1],f[i][2],f[i][0],g[j][1],g[j][2],g[j][0]));
	printf("%d\n",ans);
	fclose(stdin),fclose(stdout);
	return 0;
}
```