# 20261019

---

## ARC214 **C - Divide into 4 Teams**

https://atcoder.jp/contests/arc214/tasks/arc214_c

$N$ 个数字的序列 $P$，分成四个组，要求每个人必须有一个组，所有组至少一个人，且满足：

- $\sum_{i\in A} P_i = \sum_{i \in B} P_i$
- $\sum_{i\in C} P_i = \sum_{i \in D} P_i$

约束：$4 \le N \le 500$ $\sum P \le 10^5$

观察到 $\sum P$ 不是偶数一定无解，然后列方程：答案一定满足 $2(K+X) = \sum P$.

等价于从能构成 $\frac{1}{2}P$ 选法的情况里分割，就是 $K^2 - 2K$ 种，于是有：

```cpp
void solve() {
  ll N; rd(N);
  V<ll> A(N); rd(A);
  ll S = 0;
  rep(i, N) S += A[i];
  debug(S);
  if(S & 1) return wt(0);
  V<mm> f(S + 1, (mm)0);
  f[0] = 1;
  for(auto x : A) {
    rrep(i, x, S + 1) {
      f[i] += f[i - x];
    }
  }
  wt((f[S/2].pow(2) - f[S/2] * (mm)2).x);
}
```

