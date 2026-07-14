### 跟 Nachia DP 博客学DP-15 回退 DP

**回退 DP （Restore DP / Rollback DP / Erasure DP）**是一种在计数 DP （尤其是背包 DP） 中非常经典且高效的技巧

在完全计算出 DP 后，通过部分地执行与原始 DP 相反的步骤来达成目标的解法，被称为“回退 DP”

它的核心思想是：**先用 $O(NM)$ 的时间求出包含所有元素的全局 DP 状态，然后利用“逆运算”，在 $O(M)$ 的时间内快速扣除（回退）某一个特定元素的影响**

如果没有这个技巧，我们为了求“去掉第 $i$ 个元素后的答案”，必须对剩余的 $N-1$ 个元素重新跑一次 $O(NM)$ 的背包，总复杂度会劣化到 $O(N^2 M)$。而使用回退 DP，总复杂度可以直接降到 $O(NM)$

#### 例题1

> [!note]
>
> 给定正整数 $N,M$ 与一个正整数序列 $A =(A_0,A_1,\ldots,A_{N-1})$ ，其中 $\sum_{i=0}^{N-1} A_i = M$。对于 $i = 0,1,\ldots,N-1$ 与 $x=1,2,\ldots,M$，将序列 $A$ 中仅移除第 $i$ 个元素 $A_i$ 后得到的序列记为 $B_i$，求 $B_i$ 的（不一定连续的）子序列中，元素总和为 $x$ 的选取方式的数量，并输出该数量除以 $998244353$ 的余数

设 $dp[x]$ 表示使用**全部元素**凑出和为 $x$ 的方案数

假设我们最后加入的元素是 $A_i$，那么在加入 $A_i$ 之前，凑出和为 $x$ 的方案数记为 $dp_{not}[x][i]$

那么根据转移，我们就有加入 $A_i$ 后的状态 $dp_{all}[x] = dp_{not}[x][i] + dp_{not}[x - A_i][i]$

这个转移其实很直观，凑出 $x$ 的方案，要么**不选 $A_i$**（就是前半段），要么**选 $A_i$**（就是之前凑出了 $x-A_i$ 的方案数，后半段）

而我们要求的就是 $dp_{not}[x][i]$ 对吧，就很简单的移项过来：
$$
dp_{not}[x][i] = dp_{all}[x]  - dp_{not}[x - A_i][i]
$$
这个公式就是回退 DP 的灵魂。它告诉我们：只要我们从能量/体积小到大（即 $x$ 从 $0$ 到 $M$）进行计算，就可以利用已经算好的较小状态，像推多米诺骨牌一样，逐一还原出没有 $A_i$ 时的状态。

- 对于 $x \lt A_i$：显然有 $dp_{not}[x][i] = dp_{all}[x]$
- 对于 $x \ge A_i$：就套现有的式子 $dp_{not}[x][i] = dp_{all}[x]  - dp_{not}[x - A_i][i]$

我们就可以写出这个题目了！

```cpp
	vector<long long> dp(M + 1, 0);
    dp[0] = 1;
    for (int i = 0; i < N; ++i) {
        for (int x = M; x >= A[i]; --x) {
            dp[x] = (dp[x] + dp[x - A[i]]) % MOD;
        }
    }
	for (int i = 0; i < N; ++i) {
        vector<long long> tmp(M + 1, 0);
        for (int x = 0; x <= M; ++x) {
            if (x < A[i]) {
                tmp[x] = dp[x];
            } else {
                tmp[x] = (dp[x] - tmp[x - A[i]] + MOD) % MOD;
            }
        }
        for (int x = 1; x <= M; ++x) {
            cout << tmp[x] << " ";
        }
        cout << "\n";
    }
```

而你可以看到，前半段是预处理出 $dp_{all}$，后半段是通过 $dp_{all}$ 回退出对应删去 $A_i$ 后的答案，这便是 **Rollback** ！

好的，现在让我们来看一道题吧：

#### 例题2：[AtCoder ABC321 F - #(subset sum = K) with Add and Erase](https://atcoder.jp/contests/abc321/tasks/abc321_f)

> [!note]
>
> 一个空盒子， $Q$ 次操作，两种情况：
>
> - 往箱子里放入一个值为 $x$ 的球
> - 从箱子里扔出一个值为 $x$ 的球，保证扔之前里面一定有
>
> 每一次操作完要输出一次询问：
>
> - 找到盒子中有多少种可以刚好凑成 $K$ 的方案数，对 $998244353$ 取模，所有球都认为是不同的
>
> **「约束」**
>
> - $1 \le Q \le 5000$
> - $1 \le K \le 5000$
> - $1 \le x \le 5000$

你会发现其实就是模板题！

插入其实就是背包更新，删除其实就是回滚更新：

```cpp
auto Mainsol = [](){
  INT(Q, K);
  v<mm> f(K + 1, 0); f[0] = 1;
  rep(Q) {
  	CHR(op); INT(x);
  	if(op == '+') {
  		rrep(v, x, K + 1)  // for(int i = K + 1; i -- > x; )
  			f[v] += f[v - x]; // 插入：标准背包更新，从大到小
  	} elif(op == '-') {
  		rep(v, x, K + 1) 
  			f[v] -= f[v - x]; // 删除：回退 DP 更新，从小到大
  	}
  	out(f[K].x);
  }
};
```



#### 例题与参考阅读

##### MID

[D - Multiset Mean](https://atcoder.jp/contests/arc104/tasks/arc104_d) 2200 ELO 

$\frac{\sum_{a\in S} a}{|S|} = x \Leftrightarrow \sum_{a \in S} |a - x| = 0$

```cpp
auto Mainsol = [](){
  INT(N, K); LL(M); mod = M;
  int S = K * N * (N + 1) / 2;
  v<v<mm>> f(N + 1, v<mm>(S + 1, 0));
  f[0][0] = 1;

  rep(i, 1, N + 1) {
  	rep(s, S + 1) {
  		f[i][s] = f[i - 1][s];
  		if(s >= i) f[i][s] += f[i][s - i];
  		if(s >= (K + 1) * i) f[i][s] -= f[i - 1][s - (K + 1) * i];
  	}
  }

  rep(x, 1, N + 1) {
  	ll top = min(K * x * (x - 1) / 2, K * (N - x) * (N - x + 1) / 2);
  	mm ans = 0;
  	rep(i, top + 1) ans += f[x - 1][i] * f[N - x][i];
  	ans *= (K + 1);
  	out((ans - 1).x);
  }
};
```

##### Hard

https://atcoder.jp/contests/arc102/tasks/arc102_c



 