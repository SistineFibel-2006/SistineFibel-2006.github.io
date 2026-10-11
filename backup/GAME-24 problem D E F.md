# GAME-24 problem D E F

## D-并列Nim

考虑选择和，因为分别可以选不超过 B 个，但是组合起来。

考虑单个，是经典问题 A mod (B + 1)

于是把这个经典问题组合起来即可

## E-受限制Nim 1

因为说了是仅偶数，所以考虑必败态，并不是 全 0，而是 1 0 0 0 0 0

仅有一个 1，所以必败态的 SG 就是 异或和 是 1

## F-项链 1

还是一样的，考虑如果一个字符串状态 S 可以到达必败态，那么 S 就是必胜，反之如果 一个 S 的所有可达态都是 必胜，那么就是 必败态。

可能需要证明一下复杂度，这个我不太清楚如何证明。

```cpp
namespace XK {
  void solve() {
  	INT N; string S; rd(N, S);
  	unordered_map<string, bool> sg;
  	auto dfs = [&](auto&&dfs, string s) -> bool {
  		if(sg.find(s) == sg.end()) {
	  		rep(i, sz(s) - 1) if(s[i] != s[i + 1]) {
	  			if(dfs(dfs, s.substr(0, i) + s.substr(i + 2, sz(s) - i - 2)) == 0) {
	  				sg[s] = 1;
	  				break;
	  			}
	  		}
  		}
  		return sg[s];
  	};
  	bool ans = dfs(dfs, S);
  	// cerr << "?" << endl;
  	wt(ans ? "Alice" : "Bob");
  }
};
```









