---
title: Fast Walsh-Hadamard Transform (in Base $K$)
date: 2017-10-21 22:07:47
tags:
	- 矩阵
	- FFT/NTT/FWT
	- 构造
	- 数论
	- 原根
categories:
	- 学习小记
mathjax: true
---

## 简介
Fast Walsh-Hadamard Transform (FWT) 是用来解决一类二($K$)进制运算卷积问题的快速算法，可以理解为每一维大小为 $2$($K$) 的高维快速傅里叶变换。这类问题的一般形式是，给定 $A$ 和 $B$，求出 $C$ 满足：
$$
C_i=\sum_{j\otimes k=i}A_jB_k
$$ 其中 $\otimes$ 是一种二($K$)进制运算。

本文将从回顾 FFT 开始，深入理解 FFT 的原理，然后再引出二进制的 FWT。讨论 FWT 时，我们先会介绍 $\mathrm{or}$ 和 $\mathrm{and}$ 运算卷积两种较为简单的情况，然后进一步介绍 $\mathrm{xor}$ 的复杂情况。最后我们会将 $\mathrm{xor}$ 运算卷积推广至 $K$ 进制的情况。
<!--more-->
---

## FFT 原理

笔者之前在学习 FFT 的时候，存在一个理解误区，就是以为 FFT 实质上就是一种用来解决多项式乘法的特殊插值法：将 $\omega_n^0, \cdots, \omega_n^{n-1}$, 这个 $n$ 个主 $n$ 次单位复数根代入求点值，然后插值回来。实质上，我们知道多项式乘法得到的多项式次数界是相乘两个多项式之和的规模，如果用来求点值的数只有相乘多项式的次数界个数，怎么可能求得积的多项式呢？

相信天机清妙的读者都知道笔者犯了一个概念性错误，FFT实质上并不是在直接求解多项式相乘，它实质上是在做一个循环卷积，溢出的次数会自动累加到其模次数界对应的位置上。

为什么 FFT 是循环卷积呢？我们令 $\mathrm{DFT}(A)_i$ 表示 $A$ 这个多项式在 $\omega_n^i$ 处的点值，那么 FFT 的实质上是依赖于这样一个原理：
$$\mathrm{DFT}(A)_x\mathrm{DFT}(B)_x=\mathrm{DFT}(C)_x $$

令 $\omega=\omega_n^x$，我们将这条式子展开：
$$\begin{align}
\sum_{i=0}^{n-1}A_i\omega^i\sum_{j=0}^{n-1}B_j\omega^j&=\sum_{k=0}^{n-1}C_k\omega^k\\
&=\sum_{k=0}^{n-1}\sum_{(i+j)\ \mathrm{mod}\ n=k}A_iB_j\omega^k
\end{align}
$$

为了使对应项系数恒相等，我们需要有 $\omega^i\omega^j=\omega^k$。

由于我们选择的是主 $n$ 次单位复数根，正好 $\omega^i\omega^j=\omega^{i+j}=\omega_n^{(i+j)x}=\omega_n^{kx}=\omega^k$。

这就是 FFT 算法的一个重要依据。

---

## 快速变换的一般思路

有了上面 FFT 算法的思路，我们来考虑快速卷积运算的一般思路

定义二元函数 $f(i,j)$，定义
$$
\mathrm{trans}(A)_i=\sum_{j=0}^{n-1}A_jf(i,j)
$$

我们需要针对给定运算的规律，构造函数 $f$ 使其满足
$$
\mathrm{trans}(A)_x\mathrm{trans}(B)_x=\mathrm{trans}(C)_x $$

在 FFT 中，我们选择了主 $n$ 次单位复数根：$f(i,j)=\omega_n^{ij}$。

当然了，这个 $f$ 函数当然不只是能够满足上面条件就够了，还要方便我计算，不然在时间复杂度上就没有任何改观了。在 FFT 中，$n$ 个主 $n$ 次复数单位根的平方本质上恰好是 $n/2$ 个主 $n/2$ 次单位复数根，于是就可以通过分治策略将时间复杂度降至 $O(n\log n)$。但是不同运算有不同的性质，对于快速计算的方法我们必须具体情况具体分析。

---

## FWT 原理
我们先讨论二进制的情况。

### 或运算、与运算卷积

首先构造函数 $f$ 使其满足上面所讲的式子。

注意到一组比较显然的事实：
$$
\begin{align}
i\ \mathrm{or}\ k=k,j\ \mathrm{or}\ k=k&\Rightarrow \left(i\ \mathrm{or}\ j\right)\ \mathrm{or}\ k=k\\
i\ \mathrm{and}\ k=k,j\ \mathrm{and}\ k=k&\Rightarrow \left(i\ \mathrm{and}\ j\right)\ \mathrm{and}\ k=k
\end{align}
$$

对于与运算，我们定义 $f(i,j)=[i\ \mathrm{and}\ j=i]$；对于或运算我们定义 $f(i,j)=[i\ \mathrm{and}\ j=j]$。容易验证，这样的 $f$ 可以满足之前的式子。

考虑如何计算这个东西？其实它的本质就是给定 $\{a_n\}$，让你计算 $\{b_n\}$ 满足
$$
b_i=\sum_{i\subseteq j}a_j
$$或者
$$
b_i=\sum_{j\subseteq i}a_i
$$

这是一个很经典的问题。既然是二进制运算，我们就考虑按位分治。

假设我们现在要算出 $[l,r)$ 区间的答案，$[l,mid)$ 区间二进制首位都是 $0$，$[mid,r)$ 区间二进制首位都为 $1$。而右区间的某个数相比左区间的对应位置的二进制（在当前考虑的这么多位二进制下）只是在最高位多了 $1$，也就是说左边那个状态是右边的子集。然后我们就可以相应地把右（左）边的值加到左（右）边。

这样我们就可以在 $O(n\log n)$ 的时间复杂度下完成 FWT 的正过程 (DWT)，那么怎么做逆过程 (IDWT) 呢？很简单，正过程的时候我们是把其中一项加到另一项，逆过来的时候我们对应地将一项减去另一项就可以解出来了，大家可以自己推算一下。那我们要不要严格按照正过程的倒序（也就是先计算当前区间再递归两边）来执行呢？其实不用，因为这样我只不过是在逆过程时将高低位颠倒考虑而已，对最后结果没有影响，也就是我们只需要在正过程上稍加改动罢了。

总的时间复杂度就是 $O(n\log n)$ 的。注意到这个过程只有加减运算，因此在模意义下也不受模数限制。

代码实现：

```cpp
void DWT(int *a,int sig)  //sig:1(DWT)/-1(IDWT)
{
	for (int l=2;l<=n;l<<=1)
		for (int i=0,h=l>>1;i<n;i+=l)
			for (int j=0;j<h;++j)
			{
				int u=a[i+j],v=a[i+j+h];
				//and: a[i+j]=u+v*sig;
				//or: a[i+j+d]=v+u*sig;
			}
}
```

### 异或卷积
终于说到本博客的重点：异或卷积。异或卷积相比其它卷积构造相对复杂一些，但实现起来也是差不多的。

异或运算的 $f$ 基于这样一个事实：我们令 $\mathrm{bitcount}(s)$ 表示 $s$ 的二进制状态中 $1$ 的个数，那么一定有 $\mathrm{bitcount}(i\ \mathrm{and}\ k)$ 的奇偶性异或 $\mathrm{bitcount}(j\ \mathrm{and}\ k)$ 的奇偶性等于 $\mathrm{bitcount}\left((i\ \mathrm{xor}\ j)\ \mathrm{and}\ k\right)$
的奇偶性。

于是我们定义 $f(i,j)=(-1)^{\mathrm{bitcount}(i\ \mathrm{and}\ j)}$，这样我们的 $f$ 就满足要求了。

这个东西怎么快速计算呢？依然按位考虑，我们令 $f_0$ 表示一个元素在 $[l,mid)$ 中对应位置的值，$f_1$ 表示在 $[mid,r)$ 区间中对应位置的值。对于一个位于 $[l,mid)$ 的元素，那么它的 $f$ 显然等于 $f_0+f_1$，因为它最高位加入的数是 $0$，所以不论原本的值来自那一边，都不会影响奇偶性。而位于 $[mid,r)$ 的元素就是 $f=f_0-f_1$，因为在右半边两个二进制最高位都加入了 $1$，会导致奇偶性变化，而左边就不会。

至于逆变换，考虑到我们相当于知道了 $a+b$ 和 $a-b$，那么只需要简单的加减消元即可以解出两个值。

时间复杂度是 $O(n\log n)$ 的。注意到这个过程虽然有除以 $2$ 的运算，但是即使题目要求对一个一般的数取模，在通常情况下我们还是可以消除它的影响。因为 $\frac xa\ \mathrm{mod}\ p=\frac{x\ \mathrm{mod}\ pa}a$，我们直接将模数乘上次数界（也就是 $2$ 的某次幂），在做逆过程的时候不除 $2$，最后将答案除以次数界就好了。

时间复杂度 $O(n\log n)$。

代码实现（没有了除以 $2$ 正逆变换没有区别）：

```cpp
void DWT(int *a)
{
	for (int l=2;l<=n;l<<=1)
		for (int i=0,h=l>>1;i<n;i+=l)
			for (int j=0;j<h;++j)
			{
				int u=a[i+j],v=a[i+j+h];
				a[i+j]=u+v,a[i+j+h]=u-v;
			}
}
```

### $K$ 进制下的异或卷积
考虑沿袭二进制下的思路，对于二进制我们定义了 $f(i,j)=(-1)^{\mathrm{bitcount}(i\ \mathrm{and}\ j)}$。现在我们考虑一下这个东西在推广以后的本质，其实这里面的 $\mathrm{and}$ 就是 $K$ 进制下的不进位乘法，所谓 $\mathrm{bitcount}(s)$ 其实就是将所有的位加起来对$K$取模。然后我们希望能有一个相关的特征性的函数 $g$，能体现出这 $K$ 个可能的取值，满足 $g(i)g(j)=g\left((i+j)\ \mathrm{mod} K\right)$。

相信大家都发现 FFT 中的主 $n$ 次单位复数根可以胜任此位，也就是我们令 $\omega$ 为主 $K$ 次单位复数根，那么 $g(i)=\omega^i$，也就是 $f(i,j)=\omega^{\mathrm{bitcount}_k(i\ \mathrm{and}_k\ j)}$。

那我们这样怎么快速做正变换呢？和二进制其实是类似的，对于区间 $[l,r)$，我们按照最高位将其分成 $K$ 个长度相等的区间，然后考虑在这些区间中位于同样对应位置的数。

假设这些数是 $F_0,F_1,\dots,F_{K-2},F_{K-1}$，假设我们要的值是 $f_0,f_1,\dots,f_{K-2},f_{K-1}$，那么就有
$$
f_i=\sum_{j=0}^{K-1}F_j\omega^{ij}
$$

逆变换怎么做呢？我们考虑对于一个分治区间，实质上我们进行的是类似 FFT 过程的一个矩阵乘法：

$$
\begin{pmatrix}
1 & 1 & 1 & \cdots & 1 \\
1 & \omega & \omega^2 & \cdots & \omega^{K-1} \\
1 & \omega^2 & \omega^4 & \cdots & \omega^{2(K-1)} \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & \omega^{K-1} & \omega^{(K-1)2} & \cdots & \omega^{(K-1)(K-1)}
\end{pmatrix}\begin{pmatrix}
F_0\\
F_1\\
F_2\\
\vdots\\
F_{K-1}
\end{pmatrix}=\begin{pmatrix}
f_0\\
f_1\\
f_2\\
\vdots\\
f_{K-1}
\end{pmatrix}
$$

我们只需要在等式两边左乘一个系数矩阵的逆矩阵就好了，学过 FFT 的都知道，根据求和引理可以验证逆矩阵：

$$
\begin{pmatrix}
\frac 1K & \frac 1K & \frac 1K & \cdots & \frac 1K \\
\frac 1K & \frac{\omega^{-1}}K & \frac{\omega^{-2}}K & \cdots & \frac{\omega^{-(K-1)}}K \\
\frac 1K & \frac{\omega^{-2}}K & \frac{\omega^{-4}}K & \cdots & \frac{\omega^{-2(K-1)}}K \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
\frac 1K & \frac{\omega^{-(K-1)}}K & \frac{\omega^{-(K-1)2}}K & \cdots & \frac{\omega^{-(K-1)(K-1)}}K
\end{pmatrix}
$$

因此我们在逆过程的时候只需要把次幂取相反数，最后再除去次数界就好了。

相信看到这里大家都明白为什么说“（$K$ 进制下）FWT 为每一维大小为 $2$($K$) 的高维快速傅里叶变换了”。

可以用类似的实现过程，复杂度是 $T(n)=KT\left(\frac nK\right)+O(Kn)$，即 $O(Kn\log_K n)$。

对于（对质数）取模的情况，我们考虑原根 $g$。显然类似 NTT 那样取 $\omega=g^{\frac{P-1}K}$，就可以做到主$K$次单位复数根的效果。不过这样做比起 NTT 对模数的要求就还要更苛刻一些了，它必须是 $K$ 的某一个倍数加 $1$。$330301441$ 这个质数是能够对 $10$ 以内的所有 $K$ 做 $K$ 进制异或卷积的。

代码实现（这次贴一个比较完整的）：

```cpp
void DWT(int *a,int sig)
{
	for (int l=K;l<=len;l*=K)
	{
		for (int i=0;i<len;++i) b[i]=a[i];
		for (int i=0,h=l/K;i<len;i+=l)
			for (int j=0;j<h;++j)
				for (int k=0;k<K;++k)
				{
					a[i+j+k*h]=0;
					for (int k_=0;k_<K;++k_) (a[i+j+k*h]+=1ll*b[i+j+k_*h]*POW[(k*k_%K*sig+K)%K]%P)%=P;
				}
	}
}

void calc()
{
	for (len=1;len<=mx;len*=K);
	POW[0]=1,omega=quick_power(G,(P-1)/K);
	for (int i=1;i<=K;++i) POW[i]=1ll*POW[i-1]*omega%P;
	DWT(A,1),DWT(B,1);
	for (int i=0;i<len;++i) C[i]=1ll*A[i]*B[i]%P;
	DWT(C,-1);
	for (int i=0,inv=quick_power(len,P-2);i<len;++i) C[i]=1ll*C[i]*inv%P;
}
```

---

## 总结

这次笔者是为了做掉 CC October Long Challenge 2017 里面的那一道 XORTREEH 才突击学习一下 FWT 的，收获还挺大的，首先纠正了自己对 FFT 的错误认识，其次对于这一类快速变换算法的基本原理有了更加深刻的理解。将二进制的异或卷积推广到 $K$ 进制下的工作是笔者独立完成的，可以说是受益匪浅了，起码印象深刻以后都不怎么会忘记了，大家也可以试着自己搞一搞。

~~好久没有写过这么长的博客了……~~

---

## 参考资料

[neither_nor, FWT 详解 知识点](http://blog.csdn.net/neither_nor/article/details/60335099)

[_rqy, Fast Walsh-Hadamard Transform——快速沃尔什变换](https://www.cnblogs.com/y-clever/p/6875743.html)

[_rqy, Fast Walsh-Hadamard Transform——快速沃尔什变换（二）](https://www.cnblogs.com/y-clever/p/6979925.html)

[Picks, Fast Walsh-Hadamard Transform](http://picks.logdown.com/posts/179290-fast-walsh-hadamard-transform)

吕凯风, 集合幂级数的性质与应用及其快速算法, 2015年信息学奥林匹克中国国家队候选队员论文集
