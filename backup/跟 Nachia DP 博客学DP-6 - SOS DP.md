### 跟 Nachia DP 博客学DP-6 : SOS DP

Nachia 原文： [DP 的俗称 | Mathenachia --- DP の俗称 | Mathenachia](https://www.mathenachia.blog/dp/)

也被称作 $3^n$ 的 DP （$O(3^n)$ 的 DP、子集 DP），SOS DP 的全称是 Sum over Subsets！

> [!tip]
>
> 在集合 $A$ 上的 Bit DP 中，在计算 $dp[B] \ (B \subset A)$ 时需要枚举并引用满足 $C \subsetneq B$ 的 $C$ 时，总的引用次数大约是 $3^{|A|}$ 
>
> 这里是证明：
>
> > [!note]
> >
> > 对于一个元素 $x \in A$，对于一个集合对 $(B, C)$ 满足 $(C \subsetneq B \subset A)$，这个 $x$ 只可能是下面3种互斥状态中的一个：
> >
> > 1. $x \in C$：因为 $C \subsetneq B$，所以一定有 $x \in B$
> > 2. $x \notin C, x \in B$：只在 $B$ 中
> > 3. $x \notin B$：不在 $B$ 中，那么肯定不在 $C$ 中
> >
> > 所以对于 $|A|$ 个元素中的每一个，都有可能是这三个状态中的一个，所以总的 $(B, C)$ 的配对总数就是： $3 \times 3 \times 3 \times \dots \times 3 = 3 ^ {|A|}$
> >
> > 所以总枚举数就是 $O(3 ^ n)$
>
> 这样的 DP 就被称为 $O(3^n) 的 DP$

例如在最小覆盖问题中，当需要从一个子集转移到另一个子集，且转移方向仅限于集合变大的时候，就可以通过 $O(3^n)$ 的算法解决.

#### 例题

##### [**AtCoder ABC187 F - Close Group**](https://atcoder.jp/contests/abc187/tasks/abc187_f)

> [!note]
>
> 给定一个 $N$ 个顶点与 $M$ 条边的简单无向图，标号 $1,2,\dots,N$，第 $i$ 个边连接顶点 $A_i$ 与 $B_i$
>
> 删除若干（可能是0）条边后，使得**下面条件成立的联通块（团）**的最小数量：
>
> - 对于任意一对顶点 $(a,b)$ 满足 $1\le a < b\le N$，如果顶点 $a$ 与 $b$ 在同一个联通块里，那么这两个点必定有一条**直接连接**这两个顶点的边
>
> **[约束]**
>
> - $1 \le N \le 18$
> - $0 \le M \le \frac{N(N-1)}{2}$
> - $1 \le A_i < B_i \le N$
> - $(A_i,B_i) \neq (A_j, B_j)$ for $i \neq j$

我们定义 $F(S)$ 为顶点集合 $S$ **完全分割成若干个完全图**，所需要的最少组数，考虑如何求 $F(S)$ :

1. 如果图 $G[S]$ 是完全图，$F(S) = 1$
2. 如果图 $G[S]$ 不是完全图，那么 $F(S) = \min_{\varnothing \neq T \subsetneq S} (F(T) + F(S \setminus T))$

而我们要求 $F(V)$，其中 $V$ 是全集，也就是全部 $N$ 个顶点所代表的集合.

这里就涉及到 SOS DP 的关键：

> [!note]
>
> 对于一个元素 $x \in V$ 来说，他在每一对 $(T, S)$ 中只有三种情况：
>
> 1. $x \in T$
> 2. $x\notin T, x\in S$
> 3. $x \notin S$
>
> 所以这个东西显然是可以 $O(3^N)$ DP 的！

而这个东西为什么可以 $O(3 ^ N)$，关键在于计算的顺序，因为我们总是按照 **集合从小到大** 的顺序去计算.

而如果不去这样算呢？我们可以计算一下，也就是 $T$ 与 $S$ 之间没有子集关系，那么就是 $O(2^N \cdot 2^N) = O(4^N)$，所以这里我们注意到了 $T$ 与 $S$ 的从属关系，于是有优化成 $O(3^N)$

所以代码思路大概就是这样的：

1. 先找出所有的完全图：标记哪些子集可以作为一个团
2. 用 SOS DP 来求 $F(V)$

```cpp
void Mainsol() {
	INT(N, M);
	v<v<I>> e(N);
	rep(M) {
		INT(a, b); -- a; -- b;
		e[a].pb(b); e[b].pb(a);
	}
	v<I> f(1 << N, INF); f[0] = 1;

	rep(i, N) rep(j, 1 << N) { // 计算所有的团
		if(f[j] != 1) continue;
		bool is_tuan = 1;
		rep(k, N) if(j & (1 << k)) {
			bool connect = 0;
			each(v, e[i]) if(v == k) {connect = 1; break;}
			if(!connect) {is_tuan = 0; break;}
		}
		if(is_tuan) f[j | (1 << i)] = 1;
	}

	rep(i, 1, 1 << N)
		for(int j = (i - 1) & i; j; j = (j - 1) & i) // 初始化 j 为 i的真子集 中最大的集合，循环结束是 j 转移为比 j 小的 i 的子集
			chmin(f[i], f[i ^ j] + f[j]);

	out(f[(1 << N) - 1]);
}
```

请记住这个 SOS DP 的这个经典结构，也就是找其真子集的过程：

```cpp
rep(i, 1, 1 << N)
		for(int j = (i - 1) & i; j; j = (j - 1) & i)
```

当然还有一种等价的更简洁写法：

```cpp
rep(i, 1, 1 << N) for(int j = i; -- j &= i; )
```

```cpp
#define rep(i, a, b) for(int i = (a); i < (b); i ++)
```

##### [AtCoder EDPC U - Grouping](https://atcoder.jp/contests/dp/tasks/dp_u)

> [!note]
>
> $N$ 个兔子，编号 $1, 2, \dots, N$
>
> 对于每一个点对 $1 \le i, j \le N$，兔子 $i, j$ 的适配点数就是 $a_{i,j}$，其中 $a_{i,i}=0, \ a_{i,j}=a_{j,i}$
>
> 他要把这 $N$ 个兔子划分成若干个组合，每一个兔子必须刚好分入一个组，一个组的价值是每一对 $1\le i, j \le N$ 其中 $i, j$ 兔子在同一个组，$val(S) = \sum a_{i,j}$ 
>
> **[约束]**
>
> - $1 \le N \le 16$
> - $|a_{i,j}| \le 10^9$
> -  $a_{i,i}=0, \ a_{i,j}=a_{j,i}$

这道题其实和上面那个是类似的，对于一个集合 $S$，其原本会有一个 $val(S)$，然后我们考虑能不能用其子集 $T$ 和这个差集 $S \setminus T$ 之和来更新即可，所以显然可以写出转移方程：

- $dp[S] = \max_{\varnothing \neq T \subsetneq S} (dp[T] + dp[S \setminus T])$

于是我们就经典两步走：

1. 先更新出每一个 $0 \le bit  \le (1 << N)$ 的初始 $val(bit)$，这一部分复杂度是 $(O(2^N \cdot N^2))$
2. 然后我们用 SOS DP 来考虑更新这个转移方程，复杂度是 $O(3 ^ N)$，论证就不说辣！

```cpp
void Mainsol() {
	INT(N);
	VV(ll, A, N, N);
	v<ll> f(1 << N), val(1 << N, 0);
	rep(bit, 1 << N) rep(i, N) rep(j, i + 1, N) if(bit & (1 << i)) if(bit & (1 << j))
		val[bit] += A[i][j];
	rep(i, 1, 1 << N) {
		f[i] = val[i];
		for(ll j = i; -- j &= i; ) {
			chmax(f[i], f[i ^ j] + val[j]);
		}
	}
	out(f[(1 << N) - 1]);
}
```

#### 拓展阅读与例题

##### EAZY

TODO

##### Middle

TODO

##### 拓展阅读

[SOS Dynamic Programming [Tutorial\] - Codeforces](https://codeforces.com/blog/entry/45223)

[SOS-DP 详解 | Eysiking](https://lineagestar.github.io/2025/08/17/Algorithm/SOSDP/index.html)









