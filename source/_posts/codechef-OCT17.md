---
title: CodeChef October Long Challenge 2017 Solution
date: 2017-10-20 21:54:26
tags:
	- 贪心
	- 计数类问题
	- 普通动态规划
	- 线段树
	- FFT/NTT/FWT
	- 并查集
	- 构造
	- CodeChef
	- 生成树
	- 容斥原理
	- 数论
	- 单调栈
	- 暴力
categories:
	- 解题报告
mathjax: true
---

## 前言
弱鸡选手第一次认真打 CC 马拉松，然而多项式菜不会异或卷积，题答水平又低，于是只能做 $8$ 道题……

让我们跳过傻逼题
<!--more-->
---

## CHEFGP
首先你要发现一个性质，一定存在一种最优解使得 kiwi fruits 只会派给某一种水果的人。

然后就对着这个性质瞎构造了。

```cpp
#include <algorithm>
#include <iostream>
#include <cstring>
#include <cstdio>

using namespace std;

const int N=100005;

int cnt[N];
char str[N],res[N<<1];
int T,n,x,y,a,b,ans;

void work(bool sig)
{
	for (int i=0;i<=a;++i) cnt[i]=0;
	int b_=0;
	for (int i=1;b_<b&&i<=a;++i) if (!(i%x)) ++cnt[i],++b_;
	for (int cur=0;b_<b&&cur<=a;)
	{
		for (;cur<=a&&cnt[cur]==y;++cur);
		if (cur<=a) ++cnt[cur],++b_;
	}
	if (b_<b) return;
	int tot=0;
	for (int i=1,l=1;i<=a;++i)
	{
		tot+=i-l>=x;
		if (i-l>=x) l=i;
		if (cnt[i]) l=i+1;
	}
	if (tot<ans)
	{
		int len=0;
		for (int i=1;i<=cnt[0];++i) res[len++]=sig?'a':'b';
		for (int i=1,l=1;i<=a;++i)
		{
			if (i-l>=x) res[len++]='*';
			res[len++]=sig?'b':'a';
			if (i-l>=x) l=i;
			if (cnt[i]) l=i+1;
			for (int j=1;j<=cnt[i];++j) res[len++]=sig?'a':'b';
		}
		res[len]='\0',ans=tot;
	}
}

int main()
{
	freopen("chefgp.in","r",stdin),freopen("chefgp.out","w",stdout);
	for (scanf("%d",&T);T--;)
	{
		scanf("%s%d%d",str,&x,&y),a=b=0,n=strlen(str);
		for (int i=0;i<n;++i) a+=str[i]=='a',b+=str[i]=='b';
		ans=a+b,work(0),swap(a,b),swap(x,y),work(1),printf("%s\n",res);
	}
	fclose(stdin),fclose(stdout);
	return 0;
}
```

---

## CHEFCCYL
破环为链，前缀后缀搞一下。

注意不要被二元环坑了。

```cpp
#include <algorithm>
#include <iostream>
#include <cstdio>
#include <cctype>
#include <vector>

using namespace std;

inline int read()
{
	int x=0,f=1;
	char ch=getchar();
	while (!isdigit(ch)) f=ch=='-'?-1:f,ch=getchar();
	while (isdigit(ch)) x=x*10+ch-'0',ch=getchar();
	return x*f;
}

int buf[30];

inline void write(int x)
{
	if (x<0) putchar('-'),x=-x;
	for (;x;x/=10) buf[++buf[0]]=x%10;
	if (!buf[0]) buf[++buf[0]]=0;
	for (;buf[0];putchar('0'+buf[buf[0]--]));
}

const int N=100005;

int pre[N],suf[N],st[N],en[N],e[N];
vector<int> cir[N],sum[N];
int a[N];
int T,n,q;

inline int dist(int cid,int x,int y)
{
	if (x>y) swap(x,y);
	return min(sum[cid][y-1]-sum[cid][x-1],sum[cid][x-1]+sum[cid][a[cid]]-sum[cid][y-1]);
}

inline int dist(int stc,int x,int enc,int y)
{
	if (stc>enc) swap(stc,enc),swap(x,y);
	return min(pre[stc]+suf[enc]+e[n]+dist(stc,st[stc],x)+dist(enc,en[enc],y),pre[enc]-pre[stc]-dist(stc,st[stc],en[stc])+dist(stc,en[stc],x)+dist(enc,st[enc],y));
}

void prepare()
{
	pre[1]=0;
	for (int i=2;i<=n;++i) pre[i]=pre[i-1]+dist(i-1,st[i-1],en[i-1])+e[i-1];
	suf[n]=0;
	for (int i=n-1;i>=1;--i) suf[i]=suf[i+1]+dist(i+1,st[i+1],en[i+1])+e[i];
}

int main()
{
	freopen("chefccyl.in","r",stdin),freopen("chefccyl.out","w",stdout);
	for (T=read();T--;)
	{
		for (int i=1;i<=n;++i) cir[i].resize(0),sum[i].resize(0),cir[i].shrink_to_fit(),sum[i].shrink_to_fit();
		n=read(),q=read();
		for (int i=1;i<=n;++i)
		{
			a[i]=read(),cir[i].push_back(a[i]);
			for (int j=1;j<=a[i];++j) cir[i].push_back(read());
			sum[i].push_back(0);
			for (int j=1;j<=a[i];++j) sum[i].push_back(sum[i][j-1]+cir[i][j]);
		}
		for (int i=1;i<=n;++i) en[i]=read(),st[i%n+1]=read(),e[i]=read();
		prepare();
		for (int v1,v2,c1,c2;q--;putchar('\n')) v1=read(),c1=read(),v2=read(),c2=read(),write(dist(c1,v1,c2,v2));
	}
	fclose(stdin),fclose(stdout);
	return 0;
}
```

---

## MARRAYS
把绝对值拆开来考虑，一个一个序列 dp 就好了。

```cpp
#include <algorithm>
#include <iostream>
#include <climits>
#include <cstdio>
#include <cctype>

using namespace std;

typedef long long LL;

inline int read()
{
	int x=0,f=1;
	char ch=getchar();
	while (!isdigit(ch)) f=ch=='-'?-1:f,ch=getchar();
	while (isdigit(ch)) x=x*10+ch-'0',ch=getchar();
	return x*f;
}

int buf[30];

inline void write(LL x)
{
	if (x<0) putchar('-'),x=-x;
	for (;x;x/=10) buf[++buf[0]]=x%10;
	if (!buf[0]) buf[++buf[0]]=0;
	for (;buf[0];putchar('0'+buf[buf[0]--]));
}

const int N=1000005;
const LL INF=LLONG_MAX/2;

int a[N],id[N],len[N];
LL pre[N],suf[N];
LL f[N][2];
int T,n;
LL ans;

bool cmp(int x,int y){return a[x]<a[y];}

int main()
{
	freopen("marrays.in","r",stdin),freopen("marrays.out","w",stdout);
	for (T=read();T--;)
	{
		n=read(),len[0]=0;
		for (int i=1,cur=0;i<=n;cur+=len[i++])
		{
			len[0]+=len[i]=read();
			for (int j=cur+1;j<=cur+len[i];++j) a[j]=read();
		}
		for (int i=1;i<=len[1];++i) f[i][0]=-a[i],f[i][1]=a[i];
		for (int i=2,cur=len[1];i<=n;cur+=len[i++])
		{
			for (int j=cur-len[i-1]+1;j<=cur;++j) id[j]=j;
			sort(id+cur-len[i-1]+1,id+cur+1,cmp);
			pre[cur-len[i-1]]=-INF,suf[cur+1]=-INF;
			for (int j=cur-len[i-1]+1;j<=cur;++j) pre[j]=max(pre[j-1],f[id[j]][0]);
			for (int j=cur;j>=cur-len[i-1]+1;--j) suf[j]=max(suf[j+1],f[id[j]][1]);
			for (int j=cur+1;j<=cur+len[i];++j)
			{
				int ptr1=cur-len[i-1],ptr2=cur+1,l=cur-len[i-1]+1,r=cur;//ptr1 the last element smaller than(or equal to) a[j],ptr2 the first element greater than(or equal to) a[j]
				for (int mid;l<=r;)
				{
					mid=l+r>>1;
					if (a[id[mid]]<=a[j]) l=(ptr1=mid)+1;
					else r=mid-1;
				}
				l=cur-len[i-1]+1,r=cur;
				for (int mid;l<=r;)
				{
					mid=l+r>>1;
					if (a[id[mid]]>=a[j]) r=(ptr2=mid)-1;
					else l=mid+1;
				}
				LL x=max(1ll*(i-1)*a[j]+pre[ptr1],suf[ptr2]-1ll*(i-1)*a[j]);
				int j_=j-1==cur?cur+len[i]:j-1;
				f[j_][0]=x-1ll*i*a[j_],f[j_][1]=x+1ll*i*a[j_];
			}
		}
		ans=-INF;
		for (int i=len[0]-len[n]+1;i<=len[0];++i) ans=max(ans,f[i][0]+1ll*n*a[i]);
		printf("%lld\n",ans);
	}
	fclose(stdin),fclose(stdout);
	return 0;
}
```

---

## SHTARR
这不是在天朝烂大街的套路吗？

这题本质就是求一段区间组成的单调栈中，值在一定区间内的数的个数。然后我们可以差分一下。

然后就是楼房重建套路了。

时间复杂度 $O(q\log^2n)$。

```cpp
#include <iostream>
#include <cstdio>
#include <cctype>

using namespace std;

typedef double db;

inline int read()
{
	int x=0,f=1;
	char ch=getchar();
	while (!isdigit(ch)) f=ch=='-'?-1:f,ch=getchar();
	while (isdigit(ch)) x=x*10+ch-'0',ch=getchar();
	return x*f;
}

int buf[30];

inline void write(int x)
{
	if (x<0) putchar('-'),x=-x;
	for (;x;x/=10) buf[++buf[0]]=x%10;
	if (!buf[0]) buf[++buf[0]]=0;
	for (;buf[0];putchar('0'+buf[buf[0]--]));
}

const int N=1000005;

inline int max(int x,int y){return x>y?x:y;}

int a[N];
int T,n,q;

struct segment_tree
{
	int cnt[N<<2],mx[N<<2];

	int querymax(int x,int st,int en,int l,int r)
	{
		if (st==l&&en==r) return mx[x];
		int mid=l+r>>1;
		if (en<=mid) return querymax(x<<1,st,en,l,mid);
		else if (mid+1<=st) return querymax(x<<1|1,st,en,mid+1,r);
		else return max(querymax(x<<1,st,mid,l,mid),querymax(x<<1|1,mid+1,en,mid+1,r));
	}

	int querycnt(int x,int st,int en,int l,int r,db p)
	{
		if (l==r) return (mx[x]>p)+1;
		int mid=l+r>>1;bool cvr=st==l&&en==r;
		if (en<=mid) return querycnt(x<<1,st,en,l,mid,p);
		else if (mid+1<=st) return querycnt(x<<1|1,st,en,mid+1,r,p);
		else
		{
			int rmx=cvr?mx[x<<1|1]:querymax(x<<1|1,mid+1,en,mid+1,r);
			if (rmx>p) return (cvr?cnt[x]:querycnt(x<<1,st,mid,l,mid,rmx))+querycnt(x<<1|1,mid+1,en,mid+1,r,p)-1;
			else return querycnt(x<<1,st,mid,l,mid,p);
		}
	}

	void update(int x,int l,int r)
	{
		int mid=l+r>>1;
		mx[x]=max(mx[x<<1],mx[x<<1|1]),cnt[x]=querycnt(x<<1,l,mid,l,mid,mx[x<<1|1]);
	}

	void modify(int x,int y,int l,int r,int delta)
	{
		if (l==r)
		{
			mx[x]+=delta;
			return;
		}
		int mid=l+r>>1;
		if (y<=mid) modify(x<<1,y,l,mid,delta);
		else modify(x<<1|1,y,mid+1,r,delta);
		update(x,l,r);
	}

	void build(int x,int l,int r)
	{
		if (l==r)
		{
			mx[x]=a[n-l+1];
			return;
		}
		int mid=l+r>>1;
		build(x<<1,l,mid),build(x<<1|1,mid+1,r),update(x,l,r);
	}
}t;

int main()
{
	freopen("shtarr.in","r",stdin),freopen("shtarr.out","w",stdout);
	for (T=read();T--;)
	{
		n=read(),q=read();
		for (int i=1;i<=n;++i) a[i]=read();
		t.build(1,1,n);
		for (int i=1;i<=q;++i)
		{
			char opt=getchar();
			for (;opt!='?'&&opt!='+';opt=getchar());
			if (opt=='?')
			{
				int x=read(),l=read(),r=read();
				write(t.querycnt(1,1,n-x+1,1,n,l-.5)-t.querycnt(1,1,n-x+1,1,n,r-.5)+(t.querymax(1,1,n-x+1,1,n)>r-.5)),putchar('\n');
			}
			else
			{
				int x=read(),y=read();
				t.modify(1,n-x+1,1,n,y);
			}
		}
	}
	fclose(stdin),fclose(stdout);
	return 0;
}
```

---

## LKYEDGE
这题真的坑……

考虑枚举区间的左端点，然后计算每一条边第一次形成环的右端点最后更新 $f(x)$。计算这个我们可以对左端点右边的所有边按照编号做一遍最小生成树。

然后从小到大枚举非树边，将两点树上路径上所有边的最小右端点标记为枚举的右端点，然后使用并查集将这些路径压缩起来，后面我们不再更新这些边。

然后时间复杂度就是 $O(m^2\alpha(n))$ 的了。

于是你需要各种卡常特技，比如说我让左端点从右往左枚举，每次加入一条边，使用环切性质更新最小生成树。

```cpp
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>
//#include <ctime>

using namespace std;

typedef long long LL;

inline int read()
{
	int x=0,f=1;
	char ch=getchar();
	while (!isdigit(ch)) f=ch=='-'?-1:f,ch=getchar();
	while (isdigit(ch)) x=x*10+ch-'0',ch=getchar();
	return x*f;
}

int buf[30];

inline void write(LL x)
{
	if (x<0) putchar('-'),x=-x;
	for (;x;x/=10) buf[++buf[0]]=x%10;
	if (!buf[0]) buf[++buf[0]]=0;
	for (;buf[0];putchar('0'+buf[buf[0]--]));
}

const int N=10005;
const int E=N<<1;
const int M=5005;

int fa[N],anc[N],rk[N],eid[N],last[N],que[N],depth[N],top[N],vis[N],vis_[N],bid[N],mx[N];
int tov[E],pre[E],nxt[E],e[E];
int edge[M][2];
bool used[M];
LL f[M];
int T,m,n,tot,head,tail;

int getfather(int son){return !fa[son]?son:fa[son]=getfather(fa[son]);}

inline void insert(int x,int y,int z){tov[++tot]=y,e[tot]=z,nxt[tot]=last[x],pre[last[x]]=last[x]?tot:0,pre[last[x]=tot]=0;}

inline void erase(int x,int y){pre[y]?nxt[pre[y]]=nxt[y]:last[x]=nxt[y],nxt[y]?pre[nxt[y]]=pre[y]:0;}

inline int merge(int x,int y)
{
	if (rk[x]<rk[y]) x^=y^=x^=y;
	fa[y]=x,rk[x]+=rk[x]==rk[y];
	return top[x]=depth[top[x]]<depth[top[y]]?top[x]:top[y];
}

int main()
{
	freopen("lkyedge.in","r",stdin),freopen("lkyedge.out","w",stdout);
	//double tot1=0,tot2=0,tot3=0;clock_t tmp;
	for (T=read();T--;)
	{
		m=read(),n=0;
		for (int i=1;i<=m;++i) edge[i][0]=read(),edge[i][1]=read(),n=max(max(edge[i][0],edge[i][1]),n),f[i]=used[i]=0;
		memset(vis,0,sizeof vis),memset(vis_,0,sizeof vis_),tot=0,memset(last,0,sizeof last);
		for (int i=1;i<=n;++i) bid[i]=i;
		for (int i=m,u,v;i>=1;--i)
		{
			//tmp=clock();
			for (int j=i;j<=m;++j) top[edge[j][0]]=edge[j][0],top[edge[j][1]]=edge[j][1];
			u=edge[i][0],v=edge[i][1],used[i]=1;
			if (bid[u]==bid[v])
			{
				mx[que[tail=1]=u]=head=0,vis_[u]=i;
				for (int x,i_,y;head<tail;)
					for (i_=last[x=que[++head]];i_;i_=nxt[i_])
						if (vis_[y=tov[i_]]!=i) mx[que[++tail]=y]=e[mx[x]]>e[i_]?mx[x]:i_,vis_[y]=i;
				int i_=mx[v],i__=i_&1?i_+1:i_-1,a=tov[i__],b=tov[i_];
				erase(a,i_),erase(b,i__),used[e[i_]]=0;
			}
			insert(u,v,i),insert(v,u,i);
			//tot1+=((double)(clock()-tmp))/CLOCKS_PER_SEC,tmp=clock();
			//for (int x=1;x<=n;++x) top[x]=x;
			vis[u]=i,anc[que[tail=1]=u]=0,head=0,depth[u]=1;
			for (int x,i_,y;head<tail;)
				for (i_=last[x=que[++head]],bid[x]=u;i_;i_=nxt[i_])
					if (vis[y=tov[i_]]!=i) depth[y]=depth[anc[que[++tail]=y]=x]+1,eid[y]=e[i_],vis[y]=i;
			//tot2+=((double)(clock()-tmp))/CLOCKS_PER_SEC,tmp=clock();
			memset(fa,0,(sizeof (int))*(n+5)),memset(rk,0,(sizeof (int))*(n+5));
			for (int j=i,x,y;j<=m;++j)
			{
				if (used[j]) continue;
				x=top[getfather(edge[j][0])],y=top[getfather(edge[j][1])];
				for (;x!=y;)
				{
					if (depth[x]<depth[y]) x^=y^=x^=y;
					f[eid[x]]+=m-j+1,x=merge(getfather(x),getfather(anc[x]));
				}
				f[j]+=m-j+1;
			}
			//tot3+=((double)(clock()-tmp))/CLOCKS_PER_SEC;
		}
		for (int i=1;i<=m;++i) write(f[i]),putchar(i<m?' ':'\n');
	}
	//printf("%.5lf %.5lf %.5lf\n",tot1,tot2,tot3);
	fclose(stdin),fclose(stdout);
	return 0;
}
```

---

## XORTREEH
为了这道题突击学习了一下 FWT……

考虑计算出 $\mathrm{mex}$ 值为 $x$ 的概率 $f_x$。

这个可以用一个简单的容斥：$\mathrm{mex}=x$ 的子序列数 $=\mathrm{mex}\geq x-1$ 的子序列数 $-\mathrm{mex}\geq x$ 的子序列数。

然后接下来的事情就是求出选出 $X$ 个子序列之后 $\mathrm{mex}$ 值为 $x$ 的概率。考虑多选一个子序列，那么新的概率 $f’_x$ 其实就是对 $f$ 做一次异或卷积：

$$
f’_x=\sum_{i\ \mathrm{xor}_k\ j=x}f_i\times f_j
$$

然后我们只需要对 $f$ 数组做一次 $k$ 进制 DWT，然后对每个元素取 $X$ 次幂，最后再 IDWT 回来就好了。

模数十分友好，它减 $1$ 是 $K$ 以内所有正整数的倍数。由 WolframAlpha 得原根是 $22$。于是就可以做数论变换了。

时间复杂度 $O(Kn\log_Kn)$。

```cpp
#include <iostream>
#include <cstring>
#include <cstdio>
#include <cctype>

using namespace std;

typedef long long LL;

inline int read()
{
	int x=0,f=1;
	char ch=getchar();
	while (!isdigit(ch)) f=ch=='-'?-1:f,ch=getchar();
	while (isdigit(ch)) x=x*10+ch-'0',ch=getchar();
	return x*f;
}

const int P=330301441;
const int N=1000005;
const int V=100000;
const int G=22;

int quick_power(int x,int y)
{
	int ret=1;
	for (;y;y>>=1,x=1ll*x*x%P) if (y&1) ret=1ll*ret*x%P;
	return ret;
}

int POW2[N],POW[N],f[N],b[N],g[N],cnt[N];
int T,n,K,omega,mx,len,ans;
LL X;

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
	DWT(f,1);
	for (int i=0;i<len;++i) f[i]=quick_power(f[i],X%(P-1));
	DWT(f,-1);
	for (int i=0,inv=quick_power(len,P-2);i<len;++i) f[i]=1ll*f[i]*inv%P;
}

int main()
{
	freopen("xortreeh.in","r",stdin),freopen("xortreeh.out","w",stdout);
	POW2[0]=1;
	for (int i=1;i<=V;++i) POW2[i]=(POW2[i-1]<<1)%P;
	for (T=read();T--;)
	{
		memset(cnt,0,sizeof cnt),memset(f,0,sizeof f),memset(g,0,sizeof g);
 		n=read(),K=read(),scanf("%lld",&X),mx=0;
		for (int i=1,x;i<=n;++i) ++cnt[x=read()],mx=max(mx,x);
		++mx;
		for (int i=0;i<=mx;++i) g[i]=(POW2[cnt[i]]-1+P)%P;
		for (int i=1;i<=mx;++i) g[i]=1ll*g[i-1]*g[i]%P,(cnt[i]+=cnt[i-1])%=P;
		for (int i=1;i<=mx;++i) f[i]=(1ll*g[i-1]*POW2[n-cnt[i-1]]%P-1ll*g[i]*POW2[n-cnt[i]]%P+P)%P;
		f[0]=POW2[n-cnt[0]];
		calc(),ans=0;
		for (int i=0,div=quick_power(quick_power(POW2[n],X%(P-1)),P-2)%P;i<len;++i) (ans+=quick_power(1ll*quick_power(i,2)*quick_power(1ll*f[i]*div%P,3)%P,i))%=P;
		printf("%d\n",ans);
	}
	fclose(stdin),fclose(stdout);
	return 0;
}
```