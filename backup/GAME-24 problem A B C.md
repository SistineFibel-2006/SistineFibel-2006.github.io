# GAME-24 problem A B C

## A-Nim

https://atcoder.jp/contests/game-24/tasks/game_a

裸nim板子

## B-巧克力

https://atcoder.jp/contests/game-24/tasks/game_b

可以想到你不能切由 minx, maxx, miny, maxy 组成的巧克力，以外的行和列满足取石头的 Nim 模型，于是维护 minx, H - maxx, miny, W - maxy 组成的四个堆即可

```cpp
namespace XK {
  void solve() {
  	INT H, W, N; rd(H, W, N); -- H, -- W;
  	ll minx = LINF, maxx = -1, miny = LINF, maxy = -1;
  	rep(i, N) {
  		ll x, y; rd(x, y); -- x, -- y;
  		chmin(minx, x); chmin(miny, y);
  		chmax(maxx, x); chmax(maxy, y);
  	}
  	debug(minx, (H - maxx), miny, (W - maxy));
  	ll ans = minx ^ (H - maxx) ^ miny ^ (W - maxy);
  	if(ans) wt("Alice"); else wt("Bob");
  }
};
```

## C - 四格骨牌的多米诺 1

https://atcoder.jp/contests/game-24/tasks/game_c

可以按照博弈值的角度来理解，表格如下
Alice = 0, 2, 1/0, 1/1

Bob = 2, 0, 1/1, 0/1

所以博弈值分别是 -2, 2, -0.5, 0.5

于是只要满足 -2 * a + 2 * b - 0.5 * c + 0.5 * d > 0 的就是 Alice ，反之是 Bob

