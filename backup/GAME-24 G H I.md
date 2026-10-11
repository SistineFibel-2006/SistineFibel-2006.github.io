# GAME-24 G H I

## G-回合制 Nim

https://atcoder.jp/contests/game-24/tasks/game_g

**结合和** 的题目，要求是每一轮 每个人都必须在 每一堆操作一次

所以如果考虑一堆石子，我们能知道石子数量 S 所对应的 必胜必败态 。

那么我们只需要考虑需要操作的次数最小的那堆石子 的 必胜必败态即可

操作次数就是 $\lfloor \frac{A}{B + 1}\rfloor$ ，先手必胜也就是有余数，所以我们只需要看 操作次数最小的 石子有没有余数即可。

```cpp
namespace XK {
  void solve() {
  	ll N; rd(N);
  	ll minx = LINF, ans = LINF;
  	rep(i, N) {
  		ll a, b; rd(a, b);
  		if(chmin(minx, a / (b + 1))) 
  			ans = LINF;
  		if(a / (b + 1) == minx) chmin(ans, a % (b + 1));
  	}
  	wt(ans ? "Alice" : "Bob");
  }
};
```

## H-字典序游戏

https://atcoder.jp/contests/game-24/tasks/game_h

其实类似 全状态转移，没有更好的写法，但是这里显式构造字符串显然复杂度不够，于是考虑使用 整数对 {rk[v], c} 来构造字符串，c 就是这一轮选择的， rk 就是之前所有的，这样就可以预处理出本轮的所有 新字符串，然后再将 c 融合入 rk 即可

比较简单的方法是 直接比这个二元组，你当然也可以 定义 key = rk[u] * 26 + (c - 'a')

因为 c 是小写字母，这样可以直接比较 key 的大小

使用一个 dp 数组记录每一条路的选法， 最后回溯即可

```cpp
namespace XK {
  void solve() {
  	ll N, M, K; rd(N, M, K);
  	V<V<int>> e(N);
  	V<pair<int, char>> ed(M);
  	rep(i, M) {
  		int u, v; char c; rd(u, v, c); -- u, -- v;
  		e[u].push_back(i);
  		ed[i] = {v, c};
  	}

  	V<V<int>> f(2 * K + 1, V<int>(N)); // hui su
  	V<int> rk(N), nrk(N), ord(N);
  	iota(all(ord), 0ll);
  	V<pair<int, char>> now(N);

  	Rep(t, 1, 2 * K + 1) {
  		bool alice = !(t & 1);
  		rep(u, N) {
  			int id = e[u][0];
  			now[u] = {rk[ed[id].fst], ed[id].snd};
  			f[t][u] = id;
  			for(auto eid : e[u]) {
  				auto [v, c] = ed[eid];
  				pair<int, char> can = {rk[v], c};
  				if((alice && can < now[u]) || (!alice && can > now[u])) {
  					now[u] = can;
  					f[t][u] = eid;
  				}
  			}
  		}

  		sort(all(ord), [now](int u, int v){
  			return now[u] < now[v];
  		});

  		nrk[ord[0]] = 0;
  		Rep(i, 1, N) {
  			int u = ord[i - 1], v = ord[i];
  			nrk[v] = nrk[u] + (now[u] != now[v]);
  		}
  		rk.swap(nrk);
  	}

  	string ans(2 * K, ' ');
  	int u = 0;
  	rrep(i, 1, 2 * K + 1) {
  		ans[i - 1] = ed[f[i][u]].snd;
  		u = ed[f[i][u]].fst;
  	}
  	wt(ans);
  }
};
```

## I-项链 2

https://atcoder.jp/contests/game-24/tasks/game_i

玩一下可以发现，想要游戏不断能玩下去的话，最终的操作一定会让其变成一个全是同一个字母的样子。

于是如果令 A,B,C 分别是 1,2,3 的话，任何一种组合操作都不会改变 序列本身的 异或和。

但是最终这个异或和如果是 0 的话，说明操作一定是 变成一个偶数长的 序列，反之则是 奇数长。

而只需要比较一开始和结尾的长度差就能判断操作数的奇偶，以此来判断谁能赢即可。

```cpp
namespace XK {
  void solve() {
  	ll N; string S; rd(N, S);
  	ll ans = 0;
  	rep(i, N) 
  		ans ^= (S[i] - 'A') + 1;
  	if(((N % 2) + !(ans == 0)) % 2) wt("Alice");
  	else wt("Bob");
  }
};
```

