# NowCoder-SummerDay2-L & B solution

## B : [B-Bitwise Maximization_2026牛客暑期多校训练营2](https://ac.nowcoder.com/acm/contest/133877/B)

> [!note]
>
> $N$ 个数，分成两组 $A,B$，最大化 $\bigoplus_{i \in A} a_i + \bigoplus_{i \in B} a_i$ 
>
> **「约束」**
>
> - $1 \le N \le 5 \times 10^5$
> - $0 \le a_i \le 2^{30}$

如果我们分别认为前半和后半部分的异或和为 $X, Y$，答案可以写成：
$$
X + Y = X \oplus Y + (X \& Y) \times 2
$$
于是我们发现，这个新式子的前半是 $\bigoplus_{i \in N} a_i = sum$，所以我们只要最大化后半段即可！

而我们会发现，$sum$ 中是 $1$ 的位一定会被分成 $1$ 和 $0$，而其对第后半的式子贡献总是 $0$；而 $sum$ 中是 $0$ 的位可以被分成 $1$ 和 $1$，或是 $0$ 和 $0$ 。

那么取 $sum$ 中 $0$ 位置的 $a_i$ 贡献的最大异或和，我们可以通过线性基！

通过不断插入 $sum \oplus a_i$ ，然后取该线性基的最大异或和即可！

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
const ll N = 30, bit = 30;
 
ll p[35];
void insert(ll k) {
	for(ll i=bit;i>=0;i--) {
		if(!(k >> i & 1)) continue;
		if(!p[i]) return p[i] = k, void();
		k ^= p[i];
	}
}


int main() {
	int t; cin >> t;
	for(;t; t --) {
		memset(p, 0, sizeof p);
		int n; cin >> n;
		ll Sum = 0, ans1 = 0, ans2 = 0;
		ll a[n];
		for(int i = 0; i < n; i ++) {
			ll x; cin >> a[i]; x = a[i];
			Sum ^= x;
		}

		ll msk = ~Sum; // 取反
		for(int i = 0; i < n; i ++) insert(a[i] & msk);

		ll ans = 0;
		for(int i = bit; i >= 0; i --) {
				if((ans ^ p[i]) > ans) ans ^= p[i];
		}

		cout << (ans << 1) + (Sum) << '\n';

	}
	return 0;
}

```

## L : [L-Lazy Shuffling_2026牛客暑期多校训练营2](https://ac.nowcoder.com/acm/contest/133877/L)

> [!note]
>
> 给你一个排列 $P$，请你找到所有等长排列 $A$，在经过 $P$ 映射后，所有 $g(A, A_P)$ 中，满足最大 $g$ 的不同排列数，模 $998244353$
>
> 定义 $g(A, A_P) =$ $A$ 的逆序对数与 $A_P$ 的逆序对数之差
>
> **「约束」**：长度最大 22

对于固定的位置重排置换 $p = [p_1, p_2, \dots, p_n]$，任意排列 $A$ 被映射为 $A' = f_p(A) = [A_{p_1}, A_{p_2}, \dots, A_{p_n}]$。

我们需要找到使得 $\vert{}f(A) - f(A')\vert{}$ 达到**最大值**的排列 $A$ 的数量。

- $f(A)$ 表示排列 $A$ 的逆序对数。
- 当我们将 $A$ 中数字的相对大小关系设定好后，可以通过构造动态规划来寻找能够最大化 $f(A) - f(A')$ 或 $f(A') - f(A)$ 的 $A$ 的个数。

由于选择 $A$ 本质上等价于决定 $1, 2, \dots, n$ 这 $n$ 个数在 $A$ 中的放置位置（即从小到大依次把数字放到 $A$ 的某个位置 $i$）。

我们要**枚举所有的排列顺序**（也就是构造一个排列的“构造顺序”），计算得分，并统计得分最大时的方案数。

这是一个标准的 **Bit DP** 过程，而主要的点在于，新加入 $i$ 的时候，如何计算 $g$ 的增量？

我们可以考虑预处理这个填入 $i$ 之后会让 $f(A) - f(A')$ 的增量（当我们由小到大逐个将数值放入位置 $i$ 时，已放置的那些位置代表了比当前数**更小**的数），这里有两个情况：

- 记录所有在原数组中在 $i$ 之后 ($j > i$)，但在新数组 $A'$ 中排在 $i$ 之前 ($q[i] > q[j]$) 的位置 $j$。放置在 $i$ 会使 $f(A)$ 增加（贡献逆序），但使 $f(A')$ 减少（消除逆序），从而使 $f(A) - f(A')$ 增加。
- 与上面相反的情况，放置在 $i$ 会使 $f(A) - f(A')$ 减少。

于是我们可以写出代码了：

```cpp
auto Mainsol = [](){
  INT(N);
  VEC(I, p, N);
  each(x, p) x --;
  v<I> q(N);
  rep(i, N) q[p[i]] = i;

  v<int> pl(N, 0), mi(N, 0);
  rep(i, N) rep(j, N) {
  	if(j < i && q[j] > q[i])
  		mi[i] |= (1 << j);
  	if(j > i && q[i] > q[j])
  		pl[i] |= (1 << j);
  }

  int bit = 1 << N;
  v<ll> f(bit, -LINF);
  v<mm> cnt(bit, 0);

  f[0] = 0, cnt[0] = 1;
  rep(b, bit) {
  	ll nowf = f[b];
  	mm nowcnt = cnt[b];
  	rep(i, N) if(!(b & (1 << i))) {
  		int next = b | (1 << i);
  		ll val = popcnt(b & pl[i]) - popcnt(b & mi[i]);
  		val += nowf;
  		dbg(val, f[next]);
  		if(val > f[next]) f[next] = val, cnt[next] = nowcnt;
  		elif(val == f[next]) cnt[next] += nowcnt;
  	} 
  }

  ll ans = cnt[bit - 1].x;
  if(f[bit - 1] > 0) ans *= 2;
  out(ans % mod);
};
```

但是这个代码有一个地方会觉得有点奇怪对吧，为什么结尾要在判断后 乘2？

这里涉及到一个问题，$A$ 和 $A_P$ 都可以满足 $g$ 的条件计数，但是 DP 过程只会统计 $A$

那么有人会说，总乘2不就好了？

但是这里涉及 $P$ 刚好是原排列的情况，这时 $A = A_P$，那么就不能乘2了，而这里刚好满足，如果 $g \neq 0$，那么一定 $P$ 不是原排列！ 

 复杂度 $O(n \cdot 2^n)$，空间 $O(2 ^n)$









