### 跟 Nachia DP 博客学DP-9 : 树形 DP

遇到关于有根树 $T$ 的问题时，设定以下子问题

> [!tip]
>
> $dp[v] =$ （关于顶点 $v$ 问题的解）

该问题通常涉及顶点 $v$ 的子树问题，这里顶点 $v$ 的子树是指顶点集合为 $T$ 中 $v$ 的后代，且根为 $v$ 的 $T$ 的有根子树

仅从叶子向根方向（自底向上）或从根向叶子方向（自顶向下）之一扫描有根树的 DP 被称为树形 DP

树形 DP 有非常非常多变种，该文章仅讨论最简单的一类，其余的会在后续文章出现

对于树形 DP 入门，我更倾向于在例题的辅助下讲解，其本身并不具有什么可以讲的东西（与正常的 动态规划 相区分）

#### 讲解

##### 例题1:  [AtCoder EDPC:  P - Independent Set](https://atcoder.jp/contests/dp/tasks/dp_p)

[AtCoder EDPC:  P - Independent Set](https://atcoder.jp/contests/dp/tasks/dp_p)

> [!note]
>
> 一颗有 $N$ 个节点的树，编号从 $1,2,\dots,N$ ，对于每个 $i = 1,2,\dots,N-1$，第 $i$ 条边连接 $x_i$ 与 $y_i$ 两个点.   太郎打算把**每个点**染成**白**或**黑**色，相邻两个点**不能**都是黑色的.
>
> 找到染色方案数，对 $10^9 + 7$ 取模.
>
> **[约束]**
>
> - $1 \le N \le 10^5$
> - $1 \le x_i,y_i \le N$

这道题使用一个维度显然是不足以存储所有的元素的，因为该点是白还是黑色是影响其后续递推的，于是我们重新设 DP 表：$dp[v][color]=$ 以点 $v$ 为根且点 $v$ 是 $color$ 颜色时候的方案数，其中 $color = 0/1$

如果当前点是白色的话，其孩子可以是白或黑色的；如果当前点是黑色的话，其孩子仅可以是白色的，于是我们就可以写出来：

- $dp[u][white] = \prod_{v \in e[u]} (dp[v][white] + dp[v][black])$
- $dp[u][black] = \prod_{v \in e[u]} (dp[v][white])$

于是我们可以写出一个代码：

```cpp
auto Mainsol = [](){
  INT(N);
  v<v<I>> e(N);
  rep(N - 1) {
  	INT(x, y); -- x, -- y;
  	e[x].pb(y); e[y].pb(x);
  }
  v<a<mm ,2>> f(N, {0, 0});
  auto dfs = [&](auto&&dfs, int u, int fa = -1) -> void {
  	f[u][0] = f[u][1] = 1;
  	each(v, e[u]) {
  		if(v == fa) continue;
  		dfs(dfs, v, u);
  		f[u][0] *= f[v][0] + f[v][1];
  		f[u][1] *= f[v][0];
  	}
  };
  dfs(dfs, 0);
  out((f[0][0] + f[0][1]).x);
};
```

相同的题： [AtCoder ABC036: D - 涂色画](https://atcoder.jp/contests/abc036/tasks/abc036_d)

##### 例题2: [AtCoder TDPC:  N - 木](https://atcoder.jp/contests/tdpc/tasks/tdpc_tree)

[AtCoder TDPC:  N - 木](https://atcoder.jp/contests/tdpc/tasks/tdpc_tree)

> [!note]
>
> 给定你 $N - 1$ 条边，你可以以任意顺序选择边被绘画的先后，但是每一次绘画后，你要保证所有边是联通的，求出绘画方案，对 $10^9 + 7$ 取模
>
> **[约束]**
>
> - $2 \le N \le 1000$
> - $1 \le a_i,b_i \le N$

如果我们假设第一条绘画的边上有一个点是整体的根，那么对于所有的子树而言，第一次遇到一个新节点时，绘画的边必须包含子树的根，这样才能满足联通，但是一旦联通后，后续的所有边的选择顺序无所谓

所以我们知道，当边 $e_v$ 是连接 $v$ 和其父亲的边，我们必须把 $e_v$ 放在所有其他 $v$ 的子树中的边之前，而放在前面的概率是 $P(e_v) = \frac{1}{size[v]}$

那么所有非根节点的情况都满足的概率就是 ： $P = \prod_{v=2}^N \frac{1}{size[v]}$

$N - 1$ 条边的所有排列法就是 $(N - 1) !$，那么合法方案数就是 $P \cdot (N-1)! = \frac{(N-1)!}{\prod_{v=2}^N {size[v]}}$

于是我们先枚举以每个点为根的情况，在每个点为根的时候，预处理出其余点在当前情况下的子树大小即可，复杂度是 $O(N^2)$ 的

这道题的其实的动态规划其实在求子树大小！

- $size[u]=$ 以 $u$ 为根的子树的大小

显然有： $size[u] = 1 + \sum_{v \in to[u]} size[v]$

但是我们枚举每一个点之后会把相同的点计算两次贡献，所以答案要除以 $2$，这样就可以了，代码如下：

```cpp
auto Mainsol = [](){
  INT(N); 
  Comb C(N * N); // 预处理阶乘
  v<v<I>> e(N);
  rep(N - 1) {
  	INT(x, y); -- x, -- y;
  	e[x].pb(y), e[y].pb(x);
  }
  mm ans = 0, cons = C.fac(N - 1);
  dbg(cons.x);
  rep(root, N) {
  	v<ll> f(N, 0); // size
	auto dfs = [&](auto&&dfs, int u, int fa = -1) -> void{
		f[u] = 1;
		each(v, e[u]) if(v != fa) {
			dfs(dfs, v, u);
			f[u] += f[v];
		}
	};
	dfs(dfs, root);
	mm now = cons;
	rep(i, N) if(i != root) now *= ((mm)f[i]).inv();
  	ans += now;
  }
  out((ans/2).x);
};
```

其他的树形 DP，请期待后续的文章，谢谢！

#### 拓展阅读与例题

https://yukicoder.me/problems/no/417

http://tdpc.contest.atcoder.jp/tasks/tdpc_eel

http://codeforces.com/contest/855/problem/C

##### 拓展阅读

[树形 DP 教程 - Codeforces --- DP on Trees Tutorial - Codeforces](https://codeforces.com/blog/entry/20935)

[竞技编程中的树形 DP 问题总结 - 哈马扬哈马扬哈马扬 --- 競技プログラミングにおける木DP問題まとめ - はまやんはまやんはまやん](https://blog.hamayanhamayan.com/entry/2017/06/19/161741)