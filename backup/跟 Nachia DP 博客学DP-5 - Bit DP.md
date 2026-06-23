### 跟 Nachia DP 博客学DP-5 : Bit DP

Nachia 原文： [DP 的俗称 | Mathenachia --- DP の俗称 | Mathenachia](https://www.mathenachia.blog/dp/)

状态压缩 DP 是一种常用于需要从 $n$ 个元素的排列（共有 $n!$ 中可能性）中寻找最优解的 DP .  常用的框架如下：

> [!tip]
>
> $dp[S] :=$ 对于全集 $U$ 的子集 $S$，在**顺序最佳化**后， $S$ 的最小成本
>
> 其中 $(S \subset U)$，$U$ 指的是 $n$ 个元素的排列.

常用的递推关系如下：

- $dp[S] = \min_{i\in S} (dp[S \setminus \set{i}] + cost(S \setminus \set{i}, i))$

该递推式子指的是：对于集合 $S$，我们希望计算其中元素经过顺序优化后的成本 $dp[S]$，但需要根据该顺序中 **最后的元素** 是什么来分类讨论：

- 当 $S$ 的最后一个元素是 $i$ 时，关于 $S$ 中除去最后一个 $i$ 的子集的最优解是 $dp[S -\setminus\set{i}]$ .  如果将 $S \setminus \set{i}$ 加上 $i$ 的成本记为 $cost(S \setminus \set i, i)$，那么最终的最小成本就是 $dp[S \setminus \set i] + cost(S \setminus \set i, i)$

之后，对于所有 $i \in S$ 尝试这个情况，然后采用成本最小的方案即可.  于是我们有：

> [!tip]
>
> **[转移方程]**： $dp[S] = \min_{i\in S} (dp[S \setminus \set{i}] + cost(S \setminus \set{i}, i))$
>
> **[初始值]**：$dp[0]=0$
>
> **[最终的最小成本（解）]**：$dp[(1<<n)-1]$

状压 DP 框架所解决的一个经典问题便是著名的 TSP 问题 （*Traveling salesman problem*, 旅行商问题）。该问题同样被广泛认为是不存在高效求解算法的问题，属于 NP-Hard 问题，所以说，其实状压 DP 是很好辨认的，通常通过数据范围来辨认.

#### TSP 问题 （*Traveling salesman problem*, 旅行商问题）

[TSP问题 | Aizu-Online-Judge](https://onlinejudge.u-aizu.ac.jp/courses/library/7/DPL/all/DPL_2_A)

> [!note]
>
> 现在有 $n$ 个城市，从任意城市出发，恰好访问一次所有城市，已访问过的城市不可再次前往，请你找到所用时间最短的路径，从城市 $i$ 到城市 $j$ 的所需时间 $dist[i][j]$ 是已知的.
>
> **[约束]**：
>
> - $1\le n\le 20$
> - $0\le dist[i][j] \le 10^3$
> - $dist[i][i] = 0$
> - $dist[i][j]=dist[j][i]$
>
> **[例子]**：
>
> $n=3$
>
> $dist = \\ 0\ 5\ 2\\ 5\ 0\ 4\\ 2\ 4\ 0$
>
> $ans = 6$：按照 0 -> 2 -> 1 的顺序，需要 6 秒
>
>  $n = 4$
>
> $dist = \\ 0\ 8\ 7\ 3\\ 8 \ 0\ 9\ 1\\ 7\ 9\ 0\ 4\\ 3\ 1\ 4\ 0$
>
> $ans = 11$：按照 1 -> 3 -> 0 -> 2 的顺序，需要 11 秒
>
> ![img](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F182963%2F801daa8c-bc73-d470-6789-cfee3bee76ea.jpeg?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=3eea2154b3b8189fc7a72d8d1fa70246)

这个问题的本质就是 **顺序最佳化** 遍历 $n$ 个城市（一共 $n!$ 中可能）的问题，所以可以用 Bit DP 解决了.

- $dp[S]:=$ 对于全集 $U = {0,1,\dots,n-1}$ 的子集 $S$，在 **优化顺序** 后， $S$ 内的最小成本.

考虑原本的模板递推式子： $dp[S] = \min_{i\in S} (dp[S \setminus \set{i}] + cost(S \setminus \set{i}, i))$

因此，我们想要计算 $cost(S \setminus \set{i}, i)$，但是这一部分取决于从什么城市到达 $i$，因此我们需要新增一维来获取这一信息.

- $dp[S][v]:=$ 对于全集 $U = {0,1,\dots,n-1}$ 的子集 $S$，在约束 “最后一个点是 $v$” 的情况下 **优化顺序** 后， $S$ 内的最小成本.

于是我们可以修改一点后得到该递推式：

- $dp[S][v] = \min_{u\in (S \setminus \set v)} (dp[S \setminus \set v][u] + dist[u][v])$

现在，因为已知 $S$ 的末尾是 $v$，因此根据 $S \setminus \set v$ 的末尾是什么来划分情况即可，现在只需要考虑初始值和解的取值即可：

> [!tip]
>
> **[初始值]**：$dp[1<<v][v] = 0,\ \ v \in U$ （从任意一个 $v$ 开始，且只经过了 $v$）
>
> **[最终的最小成本（解）]**：$\min_{v \in U} dp[(1<<n)-1][v]$ （所有点结尾的经过了所有点的最小值）

 于是我们可以用记忆化搜索来解决：

```cpp
void Mainsol() {
	INT(N, M);
	v<v<ll>> G(N, v<ll>(N, LINF));
	rep(M) {
		INT(x, y, c);
		G[x][y] = c;
	}
	v<v<ll>> f((1 << (N + 1)) - 1, v<ll>(N, 0));
	auto rec = [&](auto&&self, int S, int u) -> ll {
		if(S == 0 && u == 0) {
			return 0;
		} elif(S == 0) return LINF;
		if((S & (1 << u)) == 0) return LINF;
		ll ans = f[S][u];
		if(ans != 0) return ans;
		ans = LINF;
		rep(i, N)
			chmin(ans, self(self, S ^ (1 << u), i) + G[i][u]);
		return f[S][u] = ans;
	};
	ll ans = rec(rec, (1 << N) - 1, 0);
	if(ans == LINF) ans = -1;
	out(ans);
}
```

当然，如果你能注意到如果二进制数 $i, j$ 所表示的集合 $S(i) \sub S(j)$ 时，则 $i \le j$ 这一性质，即使采用分发式 DP 的循环更新，也可以解决:

```cpp
void Mainsol() {
	INT(N, M);
	vv(I, G, N, N, INF);
	vv(I, dp, 1 << N, N, INF);
  	rep(M) {
  		INT(s, t, d);
  		G[s][t] = d;
  	}
  	dp[0][0] = 0;
  	rep(S, 1 << N) rep(v, N) rep(u, N) {
  		if(S != 0 && !(S & (1 << u))) continue;
  		if((S & (1 << v)) == 0) if(v != u)
  			chmin(dp[S | (1 << v)][v], dp[S][u] + G[u][v]);
  	}
  	ll ans = dp[(1 << N) - 1][0];
  	if(ans == INF) ans = -1;
  	out(ans);
}
```

状压 DP 是一种框架，适用于在 $n$ 个元素的排列中寻找最优解的场景，可以把复杂度从 $O(n!)$ 降至 $O(n \cdot 2 ^ n)$ （在旅行商问题中是 $O(n^2 \cdot 2 ^ n)$）

实际上原本 $O(n!)$ 的搜索只能达到 $n \le 10$ 左右水平，而现在可以达到 $n \le 20$ 左右.

**所以其实辨别状压 DP 可以使用这样的方法，但是个人并不推荐，使用上述框架判断会更加准确！**

下面是一些拓展例题：

#### Atcoder-EDPC_O: Matching

[Atcoder-EDPC_O: Matching](https://atcoder.jp/contests/dp/tasks/dp_o)

> [!note]
>
> 有 $N$ 个男人，$N$ 个女人，都编号为 $1,2,\dots,N$.   有一个布尔配对表 $A_{i,j}$ 表示 $i$ 男子对 $j$ 女子是否可以匹配，你需要刚好匹配 $N$ 对伴侣，且每个人必须一一配对，查询刚好 $N$ 对的方案数，取模 $10^9 + 7$
>
> **[约束]**
>
> - $1\le N \le 21$
> - $a_{i,j}$ 是 $0/1$

考虑计数问题，我们先确定男人的顺序是 $1,2,\dots,N$，发现能匹配女人的顺序就有 $N!$ 中排列方式.

这里就出现关键框架了，在 $N!$ 中寻找 **顺序最优化** 的匹配，于是可以用 状压 DP.

于是我们可以定义：

- $dp[i][S]:=$ 当匹配到第 $i$ 个男生，且 $S$ 集合表示的女性都完成匹配时候的匹配数.

对于 $dp[i+1][*]$ 这一层的情况就可以由上一层转移过来，于是复杂度是 $O(N^2 \cdot 2^N)$，这是会 TL 的.

所以我们需要考虑如何更优化？我们会发现，当男生匹配 $i$ 个的时候，你的 $S$ 中也必须刚好有 $i$ 的女生匹配啊！于是我们就可以加一层判断来剪掉一层 内层的 $O(N)$  的循环，最终代码如下：

```cpp
void Mainsol() {
	INT(N);
	VV(bool, A, N, N);
	vv(mm, f, N + 1, 1 << 21); // mm 就是 modint
	f[0][0] = 1;
	rep(i, 0, N) rep(bi, 0, 1 << N) {
		// if(popcnt(bi) != i) continue; // 这句话就是关键剪枝
		rep(j, 0, N) if(!((1 << j) & bi))
			if(A[i][j]) f[i + 1][bi | (1 << j)] += f[i][bi];
	}
	out(f[N][(1 << N) - 1].x);
}
```

复杂度就是 $O(N\cdot 2^N)$ 的了，可以通过！

#### 拓展阅读与例题

##### EAZY

[yukicoder No.698 组队配对](https://yukicoder.me/problems/no/698)

[yukicoder No.357 商品排序 (Middle)](http://yukicoder.me/problems/no/357)

[CF CF454 Party](http://codeforces.com/contest/906/problem/C)

[CF ECR27 Guards-In-The-Storehouse](http://codeforces.com/contest/845/problem/F)

##### Middle

[ATC 天下一文字列集合](http://tenka1-2014-quala.contest.atcoder.jp/tasks/tenka1_2014_qualA_c)

[AOJ RUPC2018 Day2](https://onlinejudge.u-aizu.ac.jp/beta/room.html#RitsCamp18Day2/problems/I)

[ATC ARC078 Mole-and-Abandoned-Mine](http://arc078.contest.atcoder.jp/tasks/arc078_d)

[ATC APC001 XOR-Tree](https://beta.atcoder.jp/contests/apc001/tasks/apc001_f)

##### 拓展阅读

[巡回セールスマン問題の意味と2近似アルゴリズム | 高校数学の美しい物語](https://manabitimes.jp/math/1130)

[竞技编程中的 bitDP 问题总结 - はまやんはまやんはまやん --- 競技プログラミングにおけるbitDP問題まとめ - はまやんはまやんはまやん](https://blog.hamayanhamayan.com/entry/2017/07/16/130151)

[No.2733 Just K-times TSP - yukicoder](https://yukicoder.me/problems/no/2733)
