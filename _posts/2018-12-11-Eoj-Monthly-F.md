---
layout: article
title: Eoj 2018.12月赛 Ｆ.日落轨迹
tags: 后缀数组  
articles:
 excerpt_type: html
---

从这个题中学到了用后缀数组很快求出每个字符串每个后缀不同子串个数的方法。

从后往前挨个插入原字符串的每个后缀，求出它与字典序相邻的两个后缀的$lcp$就可以去重了。

```cpp
#include <bits/stdc++.h>
using namespace std;
const int maxn=200050;
typedef long long ll;
char str[maxn];
int nxt[maxn];
int getNxt(char *str,int n) {
	nxt[1]=0;
	for(int i = 2,u=0; i <= n; ++i) {
		while(u && str[i]!=str[u+1]) u=nxt[u];
		if(str[i]==str[u+1]) ++u;
		nxt[i]=u;
	}
	return n-nxt[n];
}
int sa[maxn],x[maxn],c[maxn],t[maxn],*Rank=x,height[maxn];
bool cmp(int u,int v,int l) {
	return x[u]==x[v]&&x[u+l]==x[v+l];
}
void sa_init(char *str,int n) {
	int m=255;
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
		swap(t,x);
		if(m==n) break;
	}
	for(int i = 1; i <= n; ++i) Rank[sa[i]]=i;
	int h=0;
	for(int i = 1; i <= n; ++i) {
		if(h) --h;
		int j=sa[Rank[i]-1];
		while(str[i+h]==str[j+h]) ++h;
		height[Rank[i]]=h;
	}
}
int f[maxn][20],Lg[maxn];
void rmq_init(int *arr,int n) {
	for(int i = 2; i < maxn; ++i) Lg[i]=Lg[i>>1]+1;
	for(int j = 0; (1<<j) <= n; ++j) {
		for(int i = 1; i+(1<<j)-1 <= n; ++i) {
			if(j==0) f[i][j]=arr[i];
			else f[i][j]=min(f[i][j-1],f[i+(1<<(j-1))][j-1]);
		}
	}
}
int Qmin(int x,int y) {
	int k=Lg[y-x+1];
	return min(f[x][k],f[y-(1<<k)+1][k]);
}
int n,q;
int lcp(int x,int y) {
	if(x==0 || y==0) return 0;
	if(x==y) return n*2-x+1;
	x=Rank[x],y=Rank[y];
	if(x>y) swap(x,y);
	return Qmin(x+1,y);
}
int L[maxn],R[maxn];
ll sum[maxn];
void deb(int *arr,int n) {
	for(int i = 1; i <= n; ++i) cout<<arr[i]<<" ";
	cout<<endl;
}
int main() {
	scanf("%d%d", &n,&q);
	scanf("%s", str+1);
	for(int i = 1; i <= n; ++i) str[i+n]=str[i];
	int L=getNxt(str,n*2);
	reverse(str+1,str+1+n*2);
	sa_init(str,n*2);
	rmq_init(height,n*2);
	stack<int> s;
	for(int i = 1; i <= n*2; ++i) {
		while(!s.empty() && sa[i]>sa[s.top()]) R[s.top()]=i,s.pop();
		if(s.size()) ::L[i]=s.top();
		s.push(i);
	}
	for(int i = n*2; i >= 1; --i) {
		int t=max(lcp(i,sa[::L[Rank[i]]]),lcp(i,sa[R[Rank[i]]]));
		sum[i]=sum[i+1]+(n*2-i+1-t);
	}
	reverse(sum+1,sum+1+n*2);
	while(q--) {
		int k;
		scanf("%d", &k);
		if(k<=n*2) printf("%lld\n", sum[k]);
		else printf("%lld\n", sum[n*2]+(ll)(k-n*2)*L);
	}
	return 0;
}
``` 
