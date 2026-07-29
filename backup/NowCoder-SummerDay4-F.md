# NowCoder-SummerDay4F: [23 Subsequences](https://ac.nowcoder.com/acm/contest/133879/F)

题目 ：[23 Subsequences](https://ac.nowcoder.com/acm/contest/133879/F)

> [!note]
>
> 定义一个序列 $B=(b_1,b_2,\ldots,b_M)$ 是好的，当且仅当对于每一个 $i=2,3,\ldots,M$，都有
> $$
> 2 \cdot b_{i-1} \le b_i \le 3 \cdot b_{i-1}
> $$
> 特别的，长度为 $1$ 的序列总是好的。
>
> 给你一个长度为 $N$ 的正整数序列 $A = (a_1, a_2, \ldots, a_N)$。你需要回答 $Q$ 次询问，每次询问给出一个区间 $[l, r]$ （$1 \le l \le r \le n$），请你找出在子序列 $A[l\ldots r]$，最长的好的子序列的长度。
>
> **「约束」**
>
> - $1 \le N, Q \le 2 \times 10^5$，$1 \le a_i \le 10^{18}$

这是一个**经典的**状态定义的技巧：

> [!tip]
>
> 定义 $f_{i,j} =$ 以 $i$ 开头，长度为 $j$ 的好子序列中，最小的结束下标值

而这里为什么第二位可以是好子序列长度呢？

因为你发现每一位数至少是前一个的两倍，于是我们有 $log 10^{18}$ 大概是 $60$ 左右，所以第二维度大概是 $60$ 左右，这应该是没有问题的！

接下来考虑初始状态，那么显然是： $f_{i, 1} = i$

那么状态转移呢？

$f_{i,j} = min_{i \lt k, 2a_i \le a_k \le 3a_i} f_{k, j - 1}$ 

就是这样！

其实是考虑对于每一个序列，然后在前面添加新的满足条件的数字，这样当然可以维护最小值！

正是 In-place DP 这样，第一维偏序关系正好就可以使用反向循环维护，而第二维刚好可以用线段树来维护。



```cpp
struct SegmentTree{
		ll size = 1;
		vector<ll> data;
		SegmentTree(ll n){
				while(size < n) size *= 2;
				data.assign(size * 2, LINF);
		}
		void update(ll at){
				while(at /= 2) data[at] = min(data[at * 2], data[at * 2 + 1]);
		}
		void set(ll at, ll val){
				at += size;
				if(data[at] <= val) return;
				data[at] = val;
				update(at);
		}
		ll get(ll l, ll r){
				ll ans = LINF;
				l += size; r += size;
				for(; l < r; l /= 2, r /= 2){
						if(l & 1) chmin(ans, data[l++]);
						if(r & 1) chmin(ans, data[--r]);
				}
				return ans;
		}
};

const int K = 60;

auto Mainsol = [](){
	INT(N, Q);
	VEC(ll, A, N);
	vec(ll, nums, N);
	rep(i, N) nums[i] = A[i];
	uniq(nums);
	int M = sz(nums);

	v<SegmentTree> seg; seg.reserve(K + 1);
	rep(len, K + 1) seg.pb(M);

	vector<a<int, K + 1>> f(N);
	rep(i, N) f[i].fill(N);

	rrep(i, N) {
		f[i][1] = i;

		ll lftv = 2 * A[i], rhtv = 3 * A[i];
		int lftpos = lower_bound(all(nums), lftv) - begin(nums);
		int rhtpos = upper_bound(all(nums), rhtv) - begin(nums);

		rep(len, 2, K + 1) if(lftpos < rhtpos) {
			ll res = seg[len - 1].get(lftpos, rhtpos);
			if(res < LINF) f[i][len] = res;
		}

		int vpos = lower_bound(all(nums), A[i]) - begin(nums);

		rep(len, 1, K + 1) if(f[i][len] < N) 
			seg[len].set(vpos, f[i][len]);
	}

	v<v<pii>> q(N);
	rep(i, Q) {
		INT(l, r); l --, r --;
		q[l].pb({r, i});
	}

	v<int> ans(Q), best(K + 1, N);

	rrep(l, N) {
		rep(len, 1, K + 1) chmin(best[len], f[l][len]);

		each(r, id, q[l]) {
			int res = 1;
			rep(len, 1, K + 1) if(best[len] <= r) res = len;
			ans[id] = res;
		}
	}

	each(c, ans) out(c);
};
```























