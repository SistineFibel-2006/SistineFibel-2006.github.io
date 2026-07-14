### 跟 Nachia DP 博客学DP-13&14 多维 DP in-place DP 优化

#### 多维 DP （*d* 次元 DP）

*d* 中是一个具体的正整数，也就是通过 in-place 优化前的维数，该维数也是 DP **时间复杂度的下界**

##### *d* 的意义

其本质是：**为了满足「无后效性」，你最少需要用几个独立的变量来唯一确定一个子问题？**

常见的一维（*d* = 1）有：斐波那契数列、LIS 问题、最大子段和；状态往往只与一个线性位置或阶段有关

常见的二维（*d* = 2）有：01背包问题、LCS 问题、区间 DP、简单网格图路径；状态由两个独立变量约束（如：位置+容量，序列A长度+序列B长度，区间左端点+右端点）。

常见的三维（*d* = 3）有：二维费用背包、带有其他情况的状压 DP 等；在二维的基础上，额外增加了一个限制条件（如：次数限制、状态掩码、第二个背包容量）。

后续不再列举

##### *d* 的用处？

而其最大的作用是可以看出时间复杂度，如果你是一个三维 DP （$N,M,K$），那么无论你如何优化空间复杂度，其状态总数总是 $N*M*K$，只要转移是 $O(1)$ 的，其时间复杂度一定是 $O(N*M*K)$ 的，而你优化的只能是空间复杂度

同样地，被压缩掉的那个维度，总是 DP 的 阶段部分：在各类背包或序列问题中，被 in-place 移除的通常是“前 $i$ 个物品”、“前 $i$ 个字符”这一维。 这一维的本质是**拓扑序的推进器**。在空间优化后，这一维虽然在数组下标里消失了，但它变成了**外层循环的循环变量 $i$** 。

#### in-place 优化 （インライン DP）

> 这里开个坑吧，回头把DP优化法都更新一下 :(

在传统的 DP 中，我们往往会定义一个多维状态（例如 $dp[i][j]$），其中 $i$ 代表“处理到第 $i$ 个元素”这一阶段，$j$ 代表具体的决策状态。

而 **In-place DP** 的核心思想是：**我们直接在同一个全局数据结构（如一个数组、Segment Tree、Fenwick Tree 或 std::set）中进行动态维护和原地更新。** 这样不仅省去了空间上的第一维 $i$，更关键的是，通过配合高效的数据结构，能把原本需要 $O(N)$ 的转移代价直接优化到 $O(1)$ 或 $O(log N)$ 。

- *当然国内也有很多叫法，比如 线段树优化 DP 之类的词，这里我还是尊重一下原文*

当你在状态转移方程中总能看到 $dp[j] = min_{0 \le k \lt j} dp[k]$ 这样的东西时，你会发现这总是一个 RMQ 问题，其往往可以使用 Segtree 来维护，动态解决该 RMQ 问题，这便是 in-place DP 的关键

其实这句话应该更加严谨一点：当动态规划的状态转移方程中，转移来源可以抽象为**受到某种偏序条件约束或区间限制下的最值查询**（形如 $dp[j] = \min_{k \in \text{valid}(j)} dp[k]$）时，这往往可以转化为一个 **RMQ 问题**。我们通常可以利用 **线段树（Segtree）** 在 $O(\log N)$ 内动态维护并解决它；若仅涉及前缀最值且支持单调更新，也可以考虑更轻量的 **树状数组（Fenwick Tree）**。

比如 LIS 问题：其方程是 $dp[j] = min_{0 \le k \lt j, a_k \le a_j} dp[k]$ ，也就是说 $k \lt j$ 是前缀，但是多了 $a_k \lt a_j$ 这个大小关系的限制，这就是偏序约束下的 RMQ 问题了，你需要用数值 $a$ 作为线段树下标，在 $[1, a_j - 1]$ 区间内查询最小值。其第一个位置限制是转移过程自动处理的，也就是说 $j$ 位置的数值一定在 $k$ 位置后更新到，而我们只需要处理第二个偏序即可，也就是关于 $a$ 的 Segtree 即可维护！

当然有些可以用滑动窗口来解决，比如： $dp[j] = min_{j-L \le k \lt j} dp[k]$，左端点始终是 $j - K$，可以用单调队列解决

让我们看个例题吧：

##### 例题1：[AtCoder EDPC Q - Flowers](https://atcoder.jp/contests/dp/tasks/dp_q)

[Q - Flowers](https://atcoder.jp/contests/dp/tasks/dp_q)

> [!note]
>
> $N$ 个二元组 $(h_i, a_i)$，$h_i$ 保证两两不同，问 $h_i$ 单增的子序列中，$\sum_{i \in S} a_i$ 最大的值
>
> **「约束」**
>
> - $1 \le N \le 2 \times 10^5$
> - $1 \le h_i \le N$ ，$h_i$ 两两不同
> - $1 \le a_i \le 10^9$

这道题我们设置 $dp[i] :=$ 考虑前 $i$ 朵花，且第 $i$ 朵必选情况下的最大 $a_i$ 和

于是可以简单的列出一个转移方程，与 LIS 问题类似： 
$$
dp_j := max_{0 \lt k \lt j} \set{(dp_k + a_j)[h_k \lt h_j]}
$$
（其中 $[h_k \lt h_j]$ 是艾弗森括号）

于是可以把艾弗森括号提前，$a_j$ 提出来得到： $dp_j := a_j + max_{0 \lt k \lt j, h_k \lt h_j} dp_k$

这便是一个含偏序约束的 RMQ 问题，可以用 $h$ 为基准的 Segtree 动态修改查询！

于是可以写出代码：

```cpp
struct SegmentTree{ // zkw Segtree
    ll size = 1;
    vector<ll> data;
    SegmentTree(ll n){
        while(size < n) size *= 2;
        data.assign(size * 2, -LINF);
    }
    void update(ll at){
        while(at /= 2) data[at] = max(data[at * 2], data[at * 2 + 1]);
    }
    void set(ll at, ll val){
        at += size;
        chmax(data[at], val);
        update(at);
    }
    ll get(ll l, ll r){
        ll ans = -LINF;
        l += size; r += size;
        for(; l < r; l /= 2, r /= 2){
            if(l & 1) chmax(ans, data[l++]);
            if(r & 1) chmax(ans, data[--r]);
        }
        return ans;
    }
};

auto Mainsol = [](){
  INT(N);
  VEC(I, h, N);
  VEC(ll, a, N);
  SegmentTree H(N + 10);
  ll ans = a[0];
  H.set(h[0], ans);
  rep(i, 1, N) {
  	ll prev = H.get(0, h[i]);
  	if(prev <= 0) prev = 0;
  	ans = prev + a[i];
  	H.set(h[i], ans); 
  }
  out(H.get(0, N + 1)); // 这里注意，并不是最后的 ans 就是最优的，因为
    // 因为最大值不一定 一定包含 A_{n-1} ，所以需要查询 H \in [1, N] 里面的最值
};
```

#### 扩展阅读与例题

[动态规划中的空间复杂度削减技巧 - hogecoder --- 動的計画法における空間計算量削減テク - hogecoder](https://tsutaj.hatenablog.com/entry/2017/12/09/000000) “竞赛编程!!”竞赛编程 Advent Calendar 2017 第 9 天的文章

[关于“内联 DP”这一技巧 - skyaozora 的日记 - TopCoder 部 --- 「インラインDP」というテクニックに関して - skyaozoraの日記 - TopCoder部](https://topcoder-g-hatena-ne-jp.jag-icpc.org/skyaozora/20171212/1513084670.html)

##### EAZY

[AtCoder EDPC Q - Flowers](https://atcoder.jp/contests/dp/tasks/dp_q) 

##### MID

[K - ターゲット](https://atcoder.jp/contests/tdpc/tasks/tdpc_target)

化简式子之后变为 LIS 问题

```cpp
auto Mainsol = [](){
  INT(N);
  v<pll> a(N);
  v<ll> val(N);
  rep(i, N) {
  	LL(x, r);
  	a[i] = {x - r, x + r};
		val[i] = x + r;
  }
  uniq(val);
  ll M = sz(val);

  sor(a, [](auto A, auto B){
  	if(A.fst != B.fst) return A.fst < B.fst;
  	else return A.snd < B.snd;
  });
  SegmentTree H(M + 10);
  ll ans = 0;

  rep(i, N) {
  	ll r = a[i].snd;
		ll idx = lower_bound(all(val), r) - begin(val);
		ll prev = H.get(idx + 1, M + 2);
		chmax(prev, 0);
		H.set(idx, prev + 1);
		chmax(ans, prev + 1);
  }
  out(H.get(0, M + 1));
};
```



##### Hard

[AtCoder ARC073 F - Many Moves](https://atcoder.jp/contests/arc073/tasks/arc073_d)

```cpp
auto Mainsol = [](){
  LL(N, Q, A, B);
  v<ll> x(Q + 1);
  x[0] = A;
  rep(i, 1, Q + 1) {
  	in(x[i]);
  }

  SegmentTree stt(N + 1), stv1(N + 1), stv2(N + 1);
  stt.set(B, 0);
  stv1.set(B, 0 - B);
  stv2.set(B, B);

  ll add = 0;

  rep(i, 1, Q + 1) {
  	ll nowx = x[i], prex = x[i - 1];
		ll min1 = stv1.get(1, nowx + 1);
		ll res1 = (min1 == LINF) ? LINF : min1 + add + nowx;
		ll min2 = stv2.get(nowx + 1, N + 1);
		ll res2 = (min2 == LINF) ? LINF : min2 + add - nowx;
		ll V = min(res1, res2);
		add += abs(nowx - prex);
		if(V < LINF) {
			ll nxt = V - add;
			stt.set(prex, nxt);
			stv1.set(prex, nxt - prex);
			stv2.set(prex, nxt + prex);
		}
  }
  out(stt.get(1, N + 1) + add);

};
```

