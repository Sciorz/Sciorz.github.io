---
layout: article
title: 焦作现场赛部分题解
tags: 后缀数组 单调栈 字典树 哈希 
excerpt_type: html
---


题目链接： <https://codeforces.com/gym/102028>{:target="_blank"}

## H.Can You Solve the Harder Problem?
显然是个后缀数组，赛场上卡在了求区间每个前缀最大值的和的问题上。~~想了半天只想到一种莫队加单调栈的巨难写的搞法。最后几十分钟还在处理莫队转移的边界问题，最后没写完就结束了。~~

题解给的做法是这样的。首先枚举区间左端点，同时维护一棵线段树，假设当前枚举的左端点是$l$，线段树中$i$位置存放的值是$[l,i]$区间的最大值，这样对于我们所枚举的这个$l
$对于答案的贡献就可以在线段树里区间求和得到。

那么我们怎么在左端点不断改变的同时，使得线段树里的对应新的左端点呢？考虑从右向左枚举左端点，同时维护一个单调栈，这样每次新元素入栈时，把线段树上$[i,s.top()-1]$区间置为$a[i]$就可以了。


## K.Counting Failures on a Trie
~~比赛时两个字符串题一个都没过愧为字符串选手了~~

定义$next[i][j]$为从$i$这个节点开始从字典树的根匹配,第$2^j$次失败时匹配到了哪个字符。要倍增预处理出这个来。~~这些比赛时也都想到了:cry:~~

就是没有想到怎么求出失败一次到哪个字符，原来可以二分加哈希。把字典树每个前缀的哈希值存起来，二分匹配到字符串的最远位置，就完了。最后失败位置可以求出剩余字符串的哈希来，看一下字典树中的对应位置。




```cpp
// H代码
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int maxn=200050;
int a[maxn],b[maxn],t[maxn];
void deb(int *arr,int n) {
	for(int i = 1; i <= n; ++i) {
		cout<<arr[i]<<" ";
	}
	cout<<endl;
}
namespace sa {
	int n,m,sa[maxn],x[maxn],c[maxn],height[maxn],*rank=x;
	bool cmp(int u,int v,int l) {
		return x[u]==x[v]&&x[u+l]==x[v+l];
	}
	void init(int *str,int _n) {
		n=_n,x[n+1]=t[n+1]=str[n+1]=0;
		m=0;
		for(int i = 1; i <= n; ++i) m=max(m,str[i]);
		fill(c,c+m+1,0);
		for(int i = 1; i <= n; ++i) c[x[i]=str[i]]++;
		for(int i = 1; i <= m; ++i) c[i]+=c[i-1];
		for(int i = n; i >= 1; --i) sa[c[x[i]]--]=i;
		for(int l = 1; l <= n; l<<=1) {
			int cnt=0;
			for(int i = n-l+1; i <= n; ++i) t[++cnt]=i;
			for(int i = 1; i <= n; ++i) if(sa[i]>l) t[++cnt]=sa[i]-l;
			fill(c,c+1+m,0);
			for(int i = 1; i <= n; ++i) c[x[i]]++;
			for(int i = 1; i <= m; ++i) c[i]+=c[i-1];
			for(int i = n; i >= 1; --i) sa[c[x[t[i]]]--]=t[i];
			m=0,t[sa[1]]=++m;
			for(int i = 2; i <= n; ++i) {
				if(cmp(sa[i],sa[i-1],l)) t[sa[i]]=m;
				else t[sa[i]]=++m;
			}
			swap(x,t);
			if(m==n) break;
		}
		for(int i = 1; i <= n; ++i) rank[sa[i]]=i;
		int h=0;
		for(int i = 1; i <= n; ++i) {
			if(h) --h;
			int j=sa[rank[i]-1];
			while(str[i+h]==str[j+h]) ++h;
			height[rank[i]]=h;
		}
	}
}
ll sum[maxn*4],lazy[maxn*4];
void build(int l,int r,int id) {
	sum[id]=0,lazy[id]=-1;
	if(l==r) return;
	else {
		int mid=(l+r)>>1;
		build(l,mid,id<<1);
		build(mid+1,r,id<<1|1);
	}
}
void pushUp(int id,int l,int r) {
	int mid=(l+r)>>1;
	sum[id<<1]=lazy[id]*(mid-l+1),sum[id<<1|1]=lazy[id]*(r-mid);
	lazy[id<<1]=lazy[id<<1|1]=lazy[id];
	lazy[id]=-1;
}
void update(int x,int y,int val,int l,int r,int id) {
	if(x<=l && y>=r) sum[id]=(ll)val*(r-l+1),lazy[id]=val;
	else {
		if(lazy[id]!=-1) pushUp(id,l,r);
		int mid=(l+r)>>1;
		if(x<=mid) update(x,y,val,l,mid,id<<1);
		if(y>mid) update(x,y,val,mid+1,r,id<<1|1);
		sum[id]=sum[id<<1]+sum[id<<1|1];
	}
}
ll query(int x,int y,int l,int r,int id) {
	if(x<=l && y>=r) return sum[id];
	else {
		if(lazy[id]!=-1) pushUp(id,l,r);
		int mid=(l+r)>>1;
		ll ans=0;
		if(x<=mid) ans+=query(x,y,l,mid,id<<1);
		if(y>mid) ans+=query(x,y,mid+1,r,id<<1|1);
		return ans;
	}
}
int main() {
	int T;
	scanf("%d", &T);
	while(T--) {
		int n;
		scanf("%d", &n);
		for(int i = 1; i <= n; ++i) scanf("%d", a+i),t[i]=b[i]=a[i];
		sort(t+1,t+1+n);
		for(int i = 1; i <= n; ++i) {
			b[i]=lower_bound(t+1,t+1+n,b[i])-t;
		}
		sa::init(b,n);
		build(1,n,1);
		using sa::height;
		using sa::rank;
		stack<int> s;
		ll ans=0;
		for(int i = n; i >= 1; --i) {
			while(!s.empty() && a[s.top()]<=a[i]) s.pop();
			update(i,s.empty()?n:s.top()-1,a[i],1,n,1);
			s.push(i);
			int l=i+height[rank[i]],r=n;
			if(l<=r) ans+=query(l,r,1,n,1);
		}
		printf("%lld\n", ans);
	}
	return 0;
}
``` 
```cpp
// K代码
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const int maxn=100050;
const ll MOD=1e9+7;
int ch[maxn][26];
char str[maxn];
unordered_map<int,int> mp[maxn];
ll h[maxn],x=123,xp[maxn];
void dfs(int x,int par,int c,int dep) {
	h[x]=(h[par]*::x+c)%MOD,mp[dep][h[x]]=x;
	for(int i = 0; i < 26; ++i) {
		if(ch[x][i]) dfs(ch[x][i],x,i+'a',dep+1);
	}
}
int nxt[maxn][20];
int main() {
	xp[0]=1;
	for(int i = 1; i < maxn; ++i) xp[i]=xp[i-1]*x%MOD;
	int T;
	scanf("%d", &T);
	while(T--) {
		int n,m,q;
		scanf("%d%d%d", &n,&m,&q);
		for(int i = 0; i <= max(n,m); ++i) memset(ch[i],0,sizeof ch[i]),mp[i].clear();
		for(int i = 1; i <= n; ++i) {
			int x;
			char str[2];
			scanf("%d%s", &x,str);
			ch[x][str[0]-'a']=i;
		}
		dfs(0,0,0,0);
		scanf("%s", str+1);
		for(int i = 1; i <= m; ++i) {
			h[i]=(h[i-1]*x+str[i])%MOD;
		}
		for(int i = 1; i <= m; ++i) {
			int l=i,r=m,P=i;
			ll t=(h[m]-h[i-1]*xp[m-i+1])%MOD;
			if(t<0) t+=MOD;
			if(mp[m-i+1].count(t)) {
				nxt[i][0]=m+1;
				continue;
			}
			while(l<=r) {
				int mid=(l+r)>>1;
				ll t=(h[mid]-h[i-1]*xp[mid-i+1])%MOD;
				if(t<0) t+=MOD;
				if(mp[mid-i+1].count(t)==0) P=mid,r=mid-1;
				else l=mid+1;
			}
			nxt[i][0]=P;
		}
		for(int j = 1; j < 20; ++j) {
			for(int i = 1; i <= m; ++i) {
				if(nxt[i][j-1]<m) nxt[i][j]=nxt[nxt[i][j-1]+1][j-1];
				else nxt[i][j]=m+1;
			}
		}
		int D=0;
		while((1<<(D+1))<=m) ++D;
		D=19;
		while(q--) {
			int l,r;
			scanf("%d%d", &l,&r);
			int x=l;
			int cfail=0;
			for(int i = D; i >= 0; --i) {
				if(x<=m && nxt[x][i]<=r) cfail+=(1<<i),x=nxt[x][i]+1;
			}
			ll t=(h[r]-h[x-1]*xp[r-x+1])%MOD;
			if(t<0) t+=MOD;
			assert(mp[r-x+1].count(t));
			int dest=mp[r-x+1][t];
			assert(cfail<=m);
			printf("%d %d\n", cfail,dest);
		}
	}
	return 0;
}
``` 
