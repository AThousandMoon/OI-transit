# [题目描述](https://uoj.ac/problem/77)

给定 $6$ 个长为 $n$ 的非负整数序列 $a_i,b_i,w_i,l_i,r_i,p_i$，求当长为 $n$ 的序列 $c_i=0/1$ 时，
$$
\sum_{i=1}^n(b_ic_i+w_i(1-c_i)-p_ic_i[\exist j<i,l_i\le a_j\le r_i,c_j=0])
$$
的最大值。

$n\le 5000,b_i,w_i\le 2\cdot 10^5,a_i,l_i,r_i\le 10^9,p_i\le 3\cdot 10^5$。

# 题解

首先把 $l,a,r$ 离散化掉。

根据经验，这是网络流题，转化一下柿子：
$$
\sum_{i=1}^n(b_i+w_i)-\sum_{i=1}^n(b_i[c_i=0]+w_i[c_i=1]+p_ic_i[\exist j<i,l_i\le a_j\le r_i,c_j=0])
$$
最小化后面那一坨东西，于是要构造最小割。

只考虑 $b,w$ 的话 $(S,i,b_i),(i,T,w_i)$ 即可，割掉连源点的边表示 $c_i=0$，割掉连汇点的边表示 $c_i=1$。

考虑上 $p_i$ 就是 $\forall j<i,l_i\le a_j\le r_i,(i',j,+\infty)$，再加上 $(i,i',p_i)$。

用主席树优化连边，时间复杂度 $O(n^2\text{polylog}(n))$。

```cpp
#include<bits/stdc++.h>
#define PB emplace_back
#define MP make_pair
#define fi first
#define se second
using namespace std;
typedef long long LL;
typedef pair<int, int> pii;
const int _N = 5111, N = 1<<18, M = 1<<21;
template<typename T>
void read(T &x){
    int ch = getchar(); x = 0; bool f = false;
    for(;ch < '0' || ch > '9';ch = getchar());
    for(;ch >= '0' && ch <= '9';ch = getchar()) x = x * 10 + ch - '0';
    if(f) x = -x;
}
template<typename T>
bool chmax(T &a, const T &b){if(a < b) return a = b, 1; return 0;}
template<typename T>
bool chmin(T &a, const T &b){if(a > b) return a = b, 1; return 0;}
int n, len, cnt, S, T, a[_N], l[_N], r[_N], val[_N], head[N], to[M], nxt[M], cap[M];
void adde(int a, int b, int c){
	static int tot = 1;
	to[++tot] = b; nxt[tot] = head[a]; head[a] = tot; cap[tot] = c;
	to[++tot] = a; nxt[tot] = head[b]; head[b] = tot; cap[tot] = 0;
}
int rt[_N], ls[N], rs[N], ans;
int upd(int x, int L, int R, int p, int v){
	int now = ++cnt;
	if(L == R){adde(now, v, INT_MAX); if(x) adde(now, x, INT_MAX); return now;}
	int mid = L + R >> 1;
	if(p <= mid){ls[now] = upd(ls[x], L, mid, p, v); rs[now] = rs[x];}
	else {rs[now] = upd(rs[x], mid+1, R, p, v); ls[now] = ls[x];}
	if(ls[now]) adde(now, ls[now], INT_MAX);
	if(rs[now]) adde(now, rs[now], INT_MAX);
	return now;
}
void qry(int x, int L, int R, int l, int r, int v){
	if(!x) return; if(l <= L && R <= r){adde(v, x, INT_MAX); return;}
	int mid = L + R >> 1;
	if(l <= mid) qry(ls[x], L, mid, l, r, v);
	if(mid < r) qry(rs[x], mid+1, R, l, r, v);
}
int cur[N], dep[N], que[N], fr, re;
bool bfs(){
	memcpy(cur, head, sizeof cur);
	memset(dep, 0x3f, sizeof dep);
	dep[S] = fr = re = 0; que[re++] = S;
	while(fr < re){
		int u = que[fr++];
		for(int i = head[u];i;i = nxt[i])
			if(cap[i] > 0 && chmin(dep[to[i]], dep[u] + 1))
				que[re++] = to[i];
	} return dep[T] < N;
}
int dfs(int x, int lim){
	if(!lim || x == T) return lim;
	int flow = 0, f;
	for(int &i = cur[x];i;i = nxt[i])
		if(dep[to[i]] == dep[x] + 1 && (f = dfs(to[i], min(lim, cap[i])))){
			flow += f; lim -= f; cap[i] -= f; cap[i^1] += f; if(!lim) break;
		}
	return flow;
}
int main(){
	read(n); S = N-1; T = S-1; cnt = n<<1;
	for(int i = 1, b, w, p;i <= n;++ i){
		read(a[i]); read(b); read(w); read(l[i]); read(r[i]); read(p);
		val[i] = a[i]; adde(S, i, b); adde(i, T, w); adde(i, i+n, p); ans += b + w; 
	} sort(val + 1, val + n + 1); len = unique(val + 1, val + n + 1) - val - 1;
	for(int i = 1;i <= n;++ i){
		a[i] = lower_bound(val + 1, val + len + 1, a[i]) - val;
		l[i] = lower_bound(val + 1, val + len + 1, l[i]) - val;
		r[i] = upper_bound(val + 1, val + len + 1, r[i]) - val - 1;
		qry(rt[i-1], 1, len, l[i], r[i], i+n);
		rt[i] = upd(rt[i-1], 1, len, a[i], i);
	} while(bfs()) ans -= dfs(S, INT_MAX);
	printf("%d\n", ans);
}
```