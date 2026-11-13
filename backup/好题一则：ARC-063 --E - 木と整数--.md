# 好题一则：ARC-063 **E - 木と整数**

---

https://atcoder.jp/contests/arc063/tasks/arc063_c

2198 ELO 相当

---

给定 $N$ 个点的一棵树，有 $K$ 个点上有数字，构造一种数字填写法，使得所有相临的点数字相差刚好 $\pm 1$.

---

可以很容易看出有 \(\text{dis}(u,v) < \text{abs}(u - v)\) 无解，并且满足奇偶性.

如果每一个点扩张之后都可以扩展自己的值域，那么其扩展的范围就是 $[L - 1, R + 1]$.

如果两个值域在某个点相遇，考虑合并其即可，因为是取小.

树上 DP 即可，但是细节有些多.

```cpp
void solve() {
  LL(N);
  V<V<ll>> E(N);
  rep(i, N - 1) {
    LL(u, v); -- u, -- v;
    E[u].push(v);
    E[v].push(u);
  }
  LL(K);
  V<pll> D(N, {-INF, INF});
  V<bool> vis(N, 0);
  ll root;
  rep(K) {
    LL(v, P); -- v;
    D[v] = {P, P};
    vis[v] = true;
    root = v;
  }

  auto meg = [&](int u, int v) -> bool { // fa, son
    D[u].fst = max(D[u].fst, D[v].fst - 1);
    D[u].snd = min(D[u].snd, D[v].snd + 1);
    return D[u].fst <= D[u].snd;
  };

  auto dfs = [&](auto&&dfs, int u, int fa, int p) -> void {
    if(vis[u]) {
      if(D[u].fst % 2 != p) {
        wt("No"); exit(0);
      }
    } else {
      if(p == 1) { // INF = 1 << 60, 转成奇数所以 + - 1
        D[u].fst ++;
        D[u].snd --;
      }
    }


    each(v, E[u]) if(v != fa) {
      dfs(dfs, v, u, p ^ 1);
      if(!meg(u, v)) {
        wt("No"); exit(0);
      }
    }
  };

  dfs(dfs, root, -1, D[root].fst % 2);
  debug(D); debug(root); debug(D[root]);
  wt("Yes");
  V<ll> ans(N, -1);
  ans[root] = D[root].fst;

  auto dfs1 = [&](auto&&dfs1, int u, int fa, int p) -> void {
    each(v, E[u]) if(v != fa) {
      if(p + 1 >= D[v].fst && p + 1 <= D[v].snd) {
        ans[v] = p + 1;
        dfs1(dfs1, v, u, p + 1);
      } else {
        ans[v] = p - 1;
        dfs1(dfs1, v, u, p - 1);
      }
    }
  };

  dfs1(dfs1, root, -1, ans[root]);
  each(x, ans) wt(x);
  debug(ans);

}
```

