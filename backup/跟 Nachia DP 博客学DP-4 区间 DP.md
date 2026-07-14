###  跟 Nachia DP 博客学DP-4 区间 DP

如果存在一个长度为 $N$ 的序列，那么将所有连续子序列（子串）作为计算对象的 DP 称为区间 DP。例如，关于序列 $A=(a_1,a_2,…,a_N)$ 的本问题中的子问题可以如下设定。

$dp[l][r] =$ 关于 $(l, r)$，特别是针对 $(a_l,a_{l+1},\ldots,a_{r-1})$ 所设定的子问题，$(l \le l \lt r \le N + 1)$

再比如，有一种模式是以长度为 $1$ 的连续子序列作为初始条件，逐步对更长的连续子序列进行计算，最终得到整个序列的计算结果。

#### 例题1：[L - Deque](https://atcoder.jp/contests/dp/tasks/dp_l)

[L - Deque](https://atcoder.jp/contests/dp/tasks/dp_l)

> [!note]
>
> 两个人玩游戏，分别取一个序列 $a=(a_1,a_2,\ldots,a_N)$ 两端的球，并使自己的得分增加对应的 $a$，$T$ 先手，分数是 $X$，$J$ 后手，分数是 $Y$。
>
> $T$ 的目的是最大化 $X - Y$，$J$ 的目的是最小化 $X-Y$，两者都以最优策略游玩，求最终的 $X - Y$ 的权值
>
> **「约束」**
>
> - $1\le N \le 3000$
> - $1\le a_i \le 10^9$

虽然是从两端往中间取，但是实际上从中间往外取计算会更加容易对吧！

设定 $dp[i][j]=$ 子串 $[i\ldots j]$ 取完之后 $X - Y$ 的结果，考虑如何转移：

显然子串 $[i\ldots j]$ 从差一点到这个状态转移，也就是： $dp[i][j] \Leftarrow dp[i + 1][j],dp[i][j - 1]$  

而这一步究竟要最大化还是最小化，实际上要看之前取了几步，于是我们可以写出转移方程了：

- 如果是 $T$ 的回合：$dp[i][j] = max \set{dp[i+1][j]+a[i],dp[i][j-1]+a[j]}$
- 如果是 $J$ 的回合：$dp[i][j]=min \set{dp[i+1][j] - a[i], dp[i][j-1] -a[j]}$

这里正好介绍出了典型的区间 DP 的写法：

- 先枚举区间长度
- 然后枚举区间起点
- 本题转移是 $O(1)$ 的

也就是 $O(N^2 \times 1)$ 的

 代码！

```cpp
void Mainsol() {
	INT(N);
	VEC(ll, A, N);
	
	ll f[3010][3010]{0};
	rep(i, N) f[i][i] = 0;

	rep(l, 1, N + 1) {
		rep(i, N - l + 1) {
			I j = i + l;
			if((N - l) % 2 == 0) { //hou
				f[i][j] = max(f[i + 1][j] + A[i], f[i][j - 1] + A[j - 1]);		
			} else {
				f[i][j] = min(f[i + 1][j] - A[i], f[i][j - 1] - A[j - 1]);
			}
		}
	}

	out(f[0][N]);
}
```

#### 例题2：[N - 史莱姆 --- N - Slimes](https://atcoder.jp/contests/dp/tasks/dp_n)

[N - 史莱姆 --- N - Slimes](https://atcoder.jp/contests/dp/tasks/dp_n)

> [!note]
>
> 石子合并问题

这道题的数据可以使用 $O(N^3)$ 的 区间 DP 来通过，本文采用左闭右开：

$dp[i][j] = min_{i \le k \lt j} \set{f[i][k] + f[k][j]} + cst(i, j)$

代码便是：

```cpp
auto Mainsol = [](){
	INT(N);
	VEC(ll, A, N);
	vec(ll, S, N + 1, 0);
	rep(i, N) S[i + 1] = S[i] + A[i];
	v<v<ll>> f(N + 1, v<ll>(N + 1, 0));
	rep(i, N) f[i][i + 1] = 0;
	rep(l, 2, N + 1) rep(i, N - l + 1) {
		int j = i + l;
		ll now = LINF;
		rep(k, i + 1, j) {
			chmin(now, f[i][k] + f[k][j]);
		}
		f[i][j] = now + S[j] - S[i];
	}
	out(f[0][N]);
};
```

这个显然是 $O(N^2 \times N)$ 的对吧哈哈！，因为转移是 $O(N)$ 的，枚举合并节点，而这里有一个很厉害的优化！

#### Monge 优化 （四边形不等式 优化）

满足 Monge 属性的二元函数可以使用 Monge 优化！如果有一个二元函数 $w(i, j)$ （通常是区间 $i$ 到 $j$ 的代价函数）满足：对于任意的 $i \le i' \lt j \le j'$，都有：
$$
w(i,j) + w(i',j') \le w(i,j')+w(i',j)
$$
那么我们就称函数 $w$ 满足 **Monge 属性** （或称满足**四边形不等式**），“交叉”区间的代价之和，小于或等于“包含”区间的代价之和。

同时区间 DP 通常还要求 $w$ 满足**区间单调性**，即如果区间被包含，代价更小：$w(i',j) \le w(i,j')$

该题目的代价函数 $w(i,j) = \sum_{k=i}^j a_k$ 满足四边形不等式（实际上是刚好取等），因此最优决策点 $s[i][j]$ 满足单调性： $s[i][j - 1] \le s[i][j] \le s[i+1][j]$

于是我们有优化：

```cpp
auto Mainsol = [](){
	INT(N);
	VEC(ll, A, N);
	vec(ll, S, N + 1, 0);
	rep(i, N) S[i + 1] = S[i] + A[i];
	v<v<ll>> f(N + 1, v<ll>(N + 1, 0));
	v<v<I>> s(N + 1, v<I>(N + 1, 0));
	rep(i, N) f[i][i + 1] = 0, s[i][i + 1] = i + 1; 
	rep(l, 2, N + 1) rep(i, N - l + 1) {
		int j = i + l;
		int L = max<int>(i + 1, s[i][j - 1]);
		int R = min<int>(j - 1, s[i + 1][j]);
		ll now = LINF, bstk = L;
		rep(k, L, R + 1) if(chmin(now, f[i][k] + f[k][j]))
			bstk = k;
		f[i][j] = now + S[j] - S[i];
		s[i][j] = bstk;
	}
	out(f[0][N]);
};
```

#### 例题3：[I - イウィ](https://atcoder.jp/contests/tdpc/tasks/tdpc_iwi)

> [!note]
>
> 一个字符串只有 `i` 或 `w`，在里面删除连续子串 `iwi` 后顺序链接剩余子串，不断执行该操作，问最多操作数？
>
> **「约束」**
>
> - $1 \le |S| \le 300$
> - 字符只有 `i` 或者 `w`

$dp[i][j] =$ 区间 $[i, j)$ 之中最多删除的字符数，有两种情况：

1. 总区间由两个小区间结果之和：$dp[i][j]= max_{i\lt k\lt j}(dp[i][k]+dp[k][j])$

2. 或者端点可以刚好被消光（$dp[i+1][j-1] == j - i + 2$），并且还要满足剩下的最左，最右，和中间那个字符刚好可以拼成 `iwi`，这时可以超级消除！也就是满足

   1. $s[i] ==$ `i`
   2. $s[k]==$`w`
   3. $s[j-1]==$`i`
   4. 除了 $k$ 以外的两部分也可以消光：$dp[i+1][k]==k-(i+1),dp[k+1][j-1]==(j-1)-(k+1)$

   而这时的答案呢？ $dp[i][j] = j - i$

于是可以写出代码

```cpp
auto Mainsol = [](){
  STR(S);
  int N = sz(S);
  v<v<ll>> f(N + 1, v<ll>(N + 1, 0));
  rep(i, N) f[i][i + 1] = 0;
  rep(l, 2, N + 1) rep(i, N - l + 1) {
  	int j = i + l;
  	ll now = 0;
  	rep(k, i + 1, j) {
  		chmax(now, f[i][k] + f[k][j]);
  		if(S[i] == 'i' && S[k] == 'w' && S[j - 1] == 'i')
  			if(f[i + 1][k] == k - (i + 1) && f[k + 1][j - 1] == (j - 1) - (k + 1))
  				now = j - i;
  	}
  	f[i][j] = now;
  }
  out(f[0][N] / 3);
};
```

好的，这就是 区间 DP 了！

实际上，区间 DP 中处理回文问题相当常见

下面是一些例题和拓展阅读

#### 例题与拓展阅读

[競技プログラミングにおける区間DP問題まとめ - はまやんはまやんはまやん](https://blog.hamayanhamayan.com/entry/2017/02/27/152226)

##### 例题

###### MID ++

 https://www.codechef.com/problems/PROC18D

https://leetcode.com/contest/weekly-contest-146/problems/minimum-cost-tree-from-leaf-values/

https://agc020.contest.atcoder.jp/tasks/agc020_e SPECIAL问题

http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=2035

https://onlinejudge.u-aizu.ac.jp/beta/room.html#ACPC2018Day3/problems/G 回文子序列

###### HARD

http://abc008.contest.atcoder.jp/tasks/abc008_4

http://codeforces.com/contest/888/problem/F

http://code-festival-2015-final-open.contest.atcoder.jp/tasks/codefestival_2015_final_g





