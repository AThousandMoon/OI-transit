# [题目描述](https://www.luogu.com.cn/problem/P4339)

求最小的识别 $m$ 进制中 $k$ 的倍数的 DFA 状态数。$T$ 组数据。 

$T\le3\cdot 10^5,m,k\le 10^{18}$。

# 题解

首先可以构造出 $k$ 个状态的 DFA，每个状态表示一个同余类。

根据某定理，每个 DFA 的最小化在同构意义下唯一。所以问题就是要将其最小化。

最小化的方法即为删除无用状态、合并等价状态。显然本题不会有无用状态，所以就是要合并等价状态。

定义 DFA $M=(S,\Sigma,d,s_0,F)$，状态 $a$ 与 $b$ 等价记作 $a\sim b$，当且仅当 $(a\in F\land b\in F)\lor(a\notin F\land b\notin F)$，且 $\forall c\in\Sigma,d(a,c)\sim d(b,c)$。

处理合并可以用并查集维护等价关系，每次将转移状态相同的状态合并，递归继续缩点。当然这题不能这样用。

但也是类似的方法。首先初始状态（也是接受状态）需要单独考虑，设其他状态为 $1,2,\dots,l$，设 $d=\gcd(m,k)$，$a,b$ 的转移状态相同即 $ma\equiv mb\pmod k$，也就是 $a\equiv b\pmod{\frac kd}$。

当 $d=1$ 或 $l\le\frac kd$ 时显然无法再缩点，直接返回。此时会有 $\frac md(k-l)$ 个位置能转移到接受状态，无法再递归缩点，所以递归到 $l'=\frac{k-m(k-l)}{d},k'=\frac kd$，当 $k<m(k-l)$ 时不递归。

时间复杂度 $O(T\log k)$。

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long LL;
template<typename T>
void read(T &x){
    int ch = getchar(); x = 0; bool f = false;
    for(;ch < '0' || ch > '9';ch = getchar());
    for(;ch >= '0' && ch <= '9';ch = getchar()) x = x * 10 + ch - '0';
    if(f) x = -x;
}
int t; LL m, k;
LL gcd(LL a, LL b){return b ? gcd(b, a % b) : a;}
LL calc(LL l, LL k){
	LL d = gcd(m, k);
	if(d == 1 || l <= k/d) return l;
	if(k <= (double)m*(k-l)) return k/d;
	return m/d*(k-l) + calc((k-m*(k-l))/d, k/d);
}
int main(){
	read(t);
	while(t --){
		read(m); read(k);
		printf("%lld\n", calc(k-1, k) + 1);
	}
}
```

