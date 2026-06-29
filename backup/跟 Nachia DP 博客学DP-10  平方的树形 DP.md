### 跟 Nachia DP 博客学DP-10  平方的树形 DP、$O(NK)$  的树形 DP

假设目标有根树的顶点数为 $n$，每个顶点有大小为 $1$ 的数据，数据的大小大概等同于子树中处理过的节点数目

父亲节点要把所有子节点的数据 **两两合并**

大致两种情形：

1. 经典的 $O(N^2)$ 树形 DP （“树上背包”的雏形）

   - **合并规则**：大小为 $a$ 和 $b$ 的两个包合并，代价是 $a \times b$，合并后新包大小是 $a + b$ （*其实有点像之前讲的卷积对吧哈哈*）

   - 为什么复杂度是 $O(N^2)$ ？

     表面上看，如果每个节点都把子树大小乘一遍，像是 $O(N^3)$.   但这里有个经典的**“子树大小平方和”**证明：

     - 代价 $a \times b$ 等价于 “在合并时，把 $a$ 包的点和 $b$ 包的每个点配对一次”.   整个树中，**任意两个节点只在他们的 LCA（最近公共祖先）处被配对一次**.   所以总配对数是 $\binom{N}{2}$，也就是 $O(N^2)$.   这就是 **平方树 DP** 名字的由来～

2. $O(NK)$ 的树形 DP（带容量限制的背包）

   - **合并规则**：合并代价仍然是 $a\times b$，但数据大小被**截断**了——大小是 $min(a+b,K)$.   这里的 $K$ 是一个常数，常见的有背包容量上限

   - 为什么复杂度是 $O(NK)$ ？

     因为大小超过 $K$ 的部分被“扔掉”了，每个节点上的数据大小最多只有 $K$

     这里我们可以用一个类似的分析，和上面一样；或者用更加精妙的分析，如果我们把大小小于 $K$ 的包称作 “小的”，大于的称作 “大的”，然后分类讨论：

     - **小与大合并**

       代价显然是 $size(small) \cdot K$

       每个元素作为 小-大 合并中的小元素被选中的次数最多是 1 次，因为一个小包和大包合并后总会变成一个大包.   此外，由于原始数量为 $N$，所以 $\sum(small)$ 总是比 $N$ 小的；于是我们有成本总和为 $O(NK)$

     - **大与大合并**

       两个大包合并一次的代价是：$K \times K = K ^ 2$

       如果一共有 $x$ 次合并，那么至少需要 $x + 1$ 个包，大包的总数最大是 $N / K$ 个，因为大包至少是 $K$ 大小的，而一共只有 $N$ 个元素

       所以合并次数 $\le N / K$，总代价 $\le (N/K) \times K^2 = NK$，是 $O(NK)$ 的

     - **小与小合并**

       *这个可能是最难讨论的一部分 :(*

       两个小包合并的时候，新包大小至多小于 $2K$（两个小于 $K$ 的合并，结果肯定小于 $2K$，之后如果继续让新包参与合并，那么就不在小小合并的讨论范畴了，因为其已经变成了大包）

       小小合并显然代价是 $O(size^2)$

       而所有包的大小要满足 $\sum_{i=1}^m size_i = N$，如果假设包一共有 $m$ 个，总代价就是 $O(\sum_{i=1}^m size_i^2)$

       那么这个总代价什么时候最大呢？显然是 $size$ 越大越好，于是也刚好是 $O(NK)$ ！

     这样以来，我们就证明了其复杂度是 $O(NK)$ 的

#### 例题

##### $O(N^2)$ 的 树形 DP

> [!note]
>
> 给定一颗有 $N$ 个点的树，标号从 $1$ 开始，求包含顶点 $1$ 且顶点数（树大小）刚好是 $K$ 的有根树个数
>
> **[约束]**
>
> - $1 \le K \le N \le 3000$

显然使用标准形状存储一维是不够的，于是扩展维度： $dp[v][i]=$ 以顶点 $v$ 为根的、顶点数是 $i$ 的有根树个数

于是可以写出一个 树上背包 的代码：

```cpp
auto Mainsol = [](){
  INT(N, K);
	v<v<I>> e(N);
	rep(N - 1) {
		INT(x, y); -- x, -- y;
		e[x].pb(y), e[y].pb(x);
	}
	v<v<ll>> f(N, v<ll>(N + 1, 0));
	v<I> siz(N, 0);
	auto dfs = [&](auto&&dfs, int u = 0, int fa = -1) -> void {
		siz[u] = 1;
		f[u][1] = 1; // 只选自己
		each(v, e[u]) if(v != fa) {
			dfs(dfs, v, u);
			::v<ll> now(siz[u] + siz[v] + 1, 0);
			rep(i, siz[u] + 1) rep(j, siz[v] + 1) now[i + j] += f[u][i] * f[v][j];
			siz[u] += siz[v];
			f[u] = move(now);
		}
		f[u][0] = 1;
	};
	dfs(dfs, 0);
	out(f[0][K]);
};
```

##### $O(NK)$ 的树上 DP

> [!note]
>
> 给定一颗有 $N$ 个点的树，标号从 $1$ 开始，求包含顶点 $1$ 且顶点数（树大小）刚好是 $K$ 的有根树个数
>
> **[约束]**
>
> - $1 \le N \le 10^5,\ \ 1\le K \le 500$

其实可以写出一个长得很像的代码，证明其复杂度是 $O(NK)$ 而不是 $O(NK^2)$ 的方法，其实在前面：

```cpp
auto Mainsol = [](){
  INT(N, K);
	v<v<I>> e(N);
	rep(N - 1) {
		INT(x, y); -- x, -- y;
		e[x].pb(y), e[y].pb(x);
	}
	v<v<ll>> f(N, v<ll>(N + 1, 0));
	v<I> siz(N, 0);
	auto dfs = [&](auto&&dfs, int u = 0, int fa = -1) -> void {
		siz[u] = 1;
		f[u][1] = 1; // 只选自己
		each(v, e[u]) if(v != fa) {
			dfs(dfs, v, u);
			::v<ll> now(min(siz[u] + siz[v], K) + 1, 0);
			rep(i, min(siz[u], K) + 1) rep(j, min(siz[v], K) + 1) 
                now[i + j] += f[u][i] * f[v][j];
			siz[u] += siz[v];
      		chmin(siz[u], K);
			f[u] = move(now);
      		f[u].resize(min(siz[u], K) + 1, 0);
		}
		f[u][0] = 1;
	};
	dfs(dfs, 0);
	out(f[0][K]);
};
```

这个代码之中其实就是多了一些剪枝，但是却可以让其在不同的约束下有不同的复杂度表现！

还有一个情况的树上 DP，请期待之后的文章！

#### 例题与其他阅读

##### 例题

###### EAZY

[P2014 [CTSC1997] 选课 - 洛谷](https://www.luogu.com.cn/problem/P2014)

$O(N^2)$ 或 $O(NK)$ 的树上 DP，与模板几乎相同！

```cpp
auto Mainsol = [](){
  INT(N, M);
  v<v<I>> e(N + 1);
  v<I> s(N + 1, 0);
  rep(i, N) {
  	INT(x); in(s[i + 1]);
  	e[x].pb(i + 1); e[i + 1].pb(x);
  }
  v<v<ll>> f(N + 1, v<ll>(N + 2, 0));
  v<I> siz(N + 1, 0);
  auto dfs = [&](auto&&dfs, int u, int fa = -1) -> void {
  	siz[u] = 1;
  	f[u][0] = 0;
  	f[u][1] = s[u];
  	each(v, e[u]) if(v != fa) {
  		dfs(dfs, v, u);
  		::v<ll> now(siz[u] + siz[v] + 1, 0);
  		rep(i, 1, siz[u] + 1) rep(j, siz[v] + 1) {
  			chmax(now[i + j], f[u][i] + f[v][j]);
  		}
  		siz[u] += siz[v];
  		rep(i, sz(now)) {
  			chmax(f[u][i], now[i]); 
  		}
  	}
  };
  dfs(dfs, 0);
  out(f[0][M + 1]);
};
```

[P2015 二叉苹果树 - 洛谷](https://www.luogu.com.cn/problem/P2015)

```cpp
auto Mainsol = [](){
  INT(N, Q);
  v<v<pll>> e(N);
  ll SS = 0;
  rep(N - 1) {
  	INT(x, y, v); -- x, -- y;
  	e[x].pb({y, v}), e[y].pb({x, v});
  	SS += v;
  }
  // dbg(SS);
  v<v<ll>> f(N, v<ll>(N + 100, 0));
  v<I> siz(N, 0);
  auto dfs = [&](auto&&dfs, int u, int fa = -1) -> void {
  	siz[u] = 0;
  	f[u][0] = 0;
  	each(V, e[u]) if(V.fst != fa) {
  		auto [v, w] = V;
  		dfs(dfs, v, u);
  		::v<ll> now(min(siz[u] + siz[v] + 1, Q) + 1, 0);
  		rep(i, min(siz[u], Q) + 1) rep(j, min(siz[v], Q) + 1) {
  			if(i + j <= Q) chmax(now[i], f[u][i]);
  			if(i + j < Q) chmax(now[i + j + 1], f[u][i] + f[v][j] + w);
  		}
  		siz[u] += siz[v] + 1;
  		rep(i, min(siz[u] + siz[v] + 1, Q) + 1)
  			f[u][i] = now[i];
  	}
  };
  dfs(dfs, 0);
  // dbg(f);
  out(f[0][Q]);
};

```

###### MID

[P - うなぎ](https://atcoder.jp/contests/tdpc/tasks/tdpc_eel)

树上 $O(NK)$ DP + NFA DP

> [!tip]
>
> **[提示1]**：考虑点的度数
>
> **[提示2]**：考虑点不同度数之间的转移

```cpp
auto Mainsol = [](){
  INT(N, K);
  v<v<I>> e(N);
  rep(N - 1) {
  	INT(x, y); -- x, -- y;
  	e[x].pb(y), e[y].pb(x);
  }
  v<v<v<mm>>> f(N, v<v<mm>>(K + 1, v<mm>(3, 0))); // f[N][size][0/1/2]
  v<I> siz(N, 0);
  auto dfs = [&](auto&&dfs, int u, int fa = -1) -> void {
  	siz[u] = 1;
  	f[u][0][0] = 1;
  	f[u][0][1] = 1;
  	f[u][0][2] = 0;
  	each(v, e[u]) if(v != fa) {
  		dfs(dfs, v, u);
  		int ssz = min(siz[u] + siz[v], K);
  		::v<::v<mm>> now(K + 1, ::v<mm>(3, 0));
  		rep(i, min(siz[u], K) + 1) rep(j, min(siz[v], K) + 1) if(i + j <= K) {
  			now[i + j][0] += f[u][i][0] * (f[v][j][0] + f[v][j][2]);
  			now[i + j][1] += f[u][i][1] * (f[v][j][0] + f[v][j][2]);
  			now[i + j][2] += f[u][i][2] * (f[v][j][0] + f[v][j][2]);
  			now[i + j][1] += f[u][i][0] * f[v][j][1];
  			if(i + j + 1 <= K) 
  				now[i + j + 1][2] += f[u][i][1] * f[v][j][1];
  		}
  		siz[u] += siz[v];
  		rep(i, min(siz[u], K) + 1) {
  			f[u][i][0] = now[i][0];
  			f[u][i][1] = now[i][1];
  			f[u][i][2] = now[i][2];
  		}
  	}
  };
  dfs(dfs, 0);
  out((f[0][K][0] + f[0][K][2]).x);
};
```
[CF Hello 2019 G](https://codeforces.com/contest/1097/problem/G)

###### HARD

[D - Zigzag Tree](https://atcoder.jp/contests/arc130/tasks/arc130_d)

##### 阅读

[树与计算量 前篇 〜O(N^2)与 O(NK)的树形 DP〜 - 如果有人问你“你是骗子吗”，请回答“YES”的博客 --- 木と計算量 前編 〜O(N^2)とO(NK)の木DP〜 - あなたは嘘つきですかと聞かれたら「YES」と答えるブログ](https://snuke.hatenablog.com/entry/2019/01/15/211812)

