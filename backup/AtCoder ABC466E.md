### [AtCoder ABC 466E - Range Flip](https://atcoder.jp/contests/abc466/tasks/abc466_e)

[E - Range Flip](https://atcoder.jp/contests/abc466/tasks/abc466_e)

#### 题目大意

给你 $N$ 张卡片，编号从 $1$ 到 $N$

第 $i$ 张卡正面写着 $A_i$，背面写着 $B_i$，一开始都是正面朝上

你可以进行下列操作至多 $K$ 次；

-  选择一个二元组 $(l, r)$ 满足 $1\le l \le r \le N$，对于所有 $l\le i\le r$ 的 $i$，翻转编号 $i$ 的卡，也就是正变反，反变正

进行完操作后，找到所有朝上数字之和的最大值

##### 约束

- $1\le N \le 2 \times 10^5$
- $1\le K \le 10$
- $1\le A_i,B_i \le 10^9$

#### 题解1

这个方法其实比较直观，如同之前的 Game DP 一样，我们设最大化变化值即可，如果定义 $d[i] = B[i] - A[i]$ 的话，那么我们要求 $max \set{\sum_{i=1}^N p \times d[i]}$，其中 $p = 0 / 1$

于是我们可以列出转移方程，设 $dp[i][j]=$ 进行了 $i$ 次操作，进行到第 $j$ 个数字时候的最大化变化值：

$dp[i][j] = max_{1 \le k \le j} \set{dp[i - 1][k - 1] + \sum_{i=k}^N d[i]}$

这个式子前缀和优化后可以变成

$dp[i][j] = max_{1 \le k \le j} \set{dp[i - 1][k - 1] + pre[j] - pre[k - 1]}$

而 $pre[j]$ 可以提出来变成

$dp[i][j] = pre[j] + max_{1 \le k \le j} \set{dp[i - 1][k - 1] - pre[k - 1]}$

而这个式子用一个 trick 就可以在 $O(NK)$ 的情况下维护了，也就是一直维护 max 值，代码如下：

```cpp
auto Mainsol = [](){
  INT(N, K);
  vec(ll, A, N); vec(ll, B, N);
  rep(i, N) in(A[i], B[i]);
  vec(ll, d, N);
  rep(i, N) d[i] = B[i] - A[i];
  vec(ll, pre, N + 1, 0);
  rep(i, N) pre[i + 1] = pre[i] + d[i];

  v<v<ll>> f(K + 1, v<ll>(N + 1, 0));
  rep(i, 1, K + 1) {
  	ll now = -LINF;
  	rep(j, 1, N + 1) {
  		chmax(now, f[i - 1][j - 1] - pre[j - 1]);
  		f[i][j] = f[i][j - 1];
  		chmax(f[i][j], pre[j] + now);
  	}
  }
  ll bst = 0;
  rep(i, N) bst += A[i];
  ll maxx = 0;
  rep(i, K + 1) chmax(maxx, f[i][N]);
  out(maxx + bst);
};

```

#### 题解2

这个做法需要一些观察，首先我们可以注意到，如果每一次选择的区间总是独立（也就是没有共用元素）的话，这样的选法总是优于（至少不劣）有共用元素的选法，于是我们可以注意到：

对于一个元素而言，其翻转次数最多 $K$ 次，也就是可达状态数 $2 \times K$ 种，于是我们有转移状态：

$dp[k] =$ 处理完前 $i$ 个元素后，进行了 $k$ 次切换能达到的最大总和

于是可以写出代码

每一次先尝试切换状态，如果前一个状态更优，那么就切换，然后根据奇偶加对应的结果即可，类似 状态机 DP！

```cpp
// code of potato167 
// https://atcoder.jp/contests/abc466/submissions/77364802
void solve(){
    int N, K;
    cin >> N >> K;
    vector<ll> A(N), B(N);
    rep(i, 0, N) cin >> A[i] >> B[i];
    vector<ll> dp(K * 2 + 1);
    rep(i, 0, N) {
        rep(j, 1, dp.size()) {
            chmax(dp[j], dp[j - 1]);
        }
        rep(j, 0, dp.size()) {
            if (j & 1) dp[j] += B[i];
            else dp[j] += A[i];
        }
    }
    cout << vec_max(dp) << "\n";
}
```

