# 跟 Nachia 博客学 DP-17 耳 DP （状态 DP）

---

---

一般的结构是
$$
dp[i][p] = (位于第 i 项的时，到达自动机状态 p 的情况下的解)
$$
 [AtCoder 「みんなのプロコン 2019」 D – Ears](https://atcoder.jp/contests/yahoo-procon2019-qual/tasks/yahoo_procon2019_qual_d) 的类似问题有时被称为「耳 DP」。另一方面，被称为「耳 DP」的范围并不明确，个体差异很明显。

整理可行数组的条件，列举可适用于各点的基本条件（ 11 只耳朵中调整前的石子个数）时，可以构建一个恰好接受允许的基本条件切换方式的有限自动机。Ears 的解法就是利用这一点的 DP。这个有限自动机极其难以发现，在思考时容易被引导到其他方针，这是它的特征。

---

---

### https://atcoder.jp/contests/yahoo-procon2019-qual/tasks/yahoo_procon2019_qual_d

考虑有四种位置，最下端，起点，终点，最上端。

于是分别割成六个段：

- 贡献是 0
- 贡献是 非 0 偶数
- 贡献是 非 0 偶数
- 贡献是 非 0 奇数
- 贡献是 非 0 偶数
- 贡献是 0

第 1 和 2 个段可以合并，于是考虑自动机 DP，因为状态只能从小往大转移

```cpp
namespace XK {
  void solve() {
  	INT N; cin >> N;
  	V<ll> A(N); rep(i, N) cin >> A[i];

  	auto wk = [&](INT x, int i) -> int {
  		if(i == 0 || i == 4) return x;
  		if(i == 1 || i == 3) {
  			if(x != 0) return x % 2;
  			else return 2;
  		}
  		return !(x % 2);
  	};

  	AR<ll, 5> f; rep(i, 5) f[i] = 0;
  	rep(i, N) {
  		AR<ll, 5> nf; rep(i, 5) nf[i] = LINF;
  		ll pre = f[0];
  		rep(j, 5) {
  			chmin(pre, f[j]);
  			nf[j] = pre + wk(A[i], j);
  		}
  		f = move(nf);
  	}

  	ll ans = LINF;
  	rep(i, 5) chmin(ans, f[i]);
  	cout << ans;
  }
};
```

---

### https://atcoder.jp/contests/abc211/tasks/abc211_c

经典的自动机 DP，匹配到 第 j 位的时候，就会增加所有 j - 1 状态数

```cpp
namespace XK {
	const INT mod = 1e9 + 7;

  void solve() {
  	const string T = "chokudai";
  	string S; cin >> S;
  	INT N = sz(S);
  	AR<ll, 8> f{};
  	rep(i, N) {
  		AR<ll, 8> nf{};
  		rep(j, 8) nf[j] += f[j];
  		rep(j, 8) if(S[i] == T[j]) {
  			if(j == 0) (nf[j] += 1) %= mod;
  			else (nf[j] += f[j - 1]) %= mod;
  		}
  		// rep(j, 8) cout << nf[j] << " \n"[j == 7];
  		f = move(nf);
  	}
  	cout << f.back();
  }
};
```













